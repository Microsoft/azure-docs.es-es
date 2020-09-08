---
title: 'Conceptos: Redes en Azure Kubernetes Service (AKS)'
description: Obtenga más información sobre las redes en Azure Kubernetes Service (AKS), incluidas las redes de kubenet y Azure CNI, los controladores de entrada, los equilibradores de carga y las direcciones IP estáticas.
ms.topic: conceptual
ms.date: 06/11/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: edb195fae2e05a1f746c10482576f7e0b1bff7c9
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88243911"
---
# <a name="network-concepts-for-applications-in-azure-kubernetes-service-aks"></a>Conceptos de redes de aplicaciones en Azure Kubernetes Service (AKS)

En una aproximación de los microservicios basados en contenedores al desarrollo de aplicaciones, los componentes de la aplicación deben funcionar juntos para procesar sus tareas. Kubernetes proporciona diversos recursos que permiten esta comunicación con la aplicación. Puede conectarse a aplicaciones y exponer estas de forma interna o externa. Para compilar aplicaciones de alta disponibilidad, puede equilibrar la carga de sus aplicaciones. Las aplicaciones más complejas pueden requerir la configuración del tráfico de entrada para la terminación SSL/TLS o el enrutamiento de varios componentes. Por motivos de seguridad, es posible que deba restringir el flujo de tráfico de red entre pods y nodos.

En este artículo se presentan los conceptos básicos que proporcionan redes para sus aplicaciones en AKS:

- [Servicios](#services)
- [Redes virtuales de Azure](#azure-virtual-networks)
- [Controladores de entrada](#ingress-controllers)
- [Directivas de red](#network-policies)

## <a name="kubernetes-basics"></a>Aspectos básicos de Kubernetes

Para permitir el acceso a las aplicaciones o para que los componentes de las aplicaciones se comuniquen entre sí, Kubernetes proporciona una capa de abstracción para la red virtual. Los nodos de Kubernetes están conectados a una red virtual y pueden proporcionar conectividad de entrada y salida para los pods. El componente *kube-proxy* se ejecuta en cada nodo para proporcionar estas características de red.

En Kubernetes, los *servicios* agrupan lógicamente los pods para permitir el acceso directo a través de una dirección IP o nombre DNS y en un puerto específico. También puede distribuir el tráfico mediante un *equilibrador de carga*. También se puede lograr un enrutamiento del tráfico de la aplicación más complejo mediante los *controladores de entradas*. La seguridad y el filtrado del tráfico de red de los pods es posible con las *directivas de red* de Kubernetes.

La plataforma Azure también ayuda a simplificar las redes virtuales de los clústeres de AKS. Cuando se crea un equilibrador de carga de Kubernetes, se crea y se configura el recurso de equilibrador de carga de Azure subyacente. Cuando abre los puertos de red a los pods, se configuran las reglas de los grupos de seguridad de red de Azure correspondientes. Para el enrutamiento de aplicaciones HTTP, Azure también puede configurar un *DNS externo* a medida que se configuran nuevas rutas de entrada.

## <a name="services"></a>Servicios

Para simplificar la configuración de red de las cargas de trabajo de la aplicación, Kubernetes usa *servicios* para agrupar lógicamente un conjunto de pods y proporcionar conectividad de red. Están disponibles los siguientes tipos de servicio:

- **Dirección IP de clúster**: crea una dirección IP interna para su uso dentro del clúster de AKS. Es útil solo para aplicaciones internas que admiten otras cargas de trabajo dentro del clúster.

    ![Diagrama que muestra el flujo de tráfico de Dirección IP de clúster en un clúster de AKS][aks-clusterip]

- **NodePort**: crea una asignación de puerto en el nodo subyacente que permite que se pueda acceder a la aplicación directamente con la dirección IP del nodo y el puerto.

    ![Diagrama que muestra el flujo de tráfico de NodePort en un clúster de AKS][aks-nodeport]

- **LoadBalancer**: crea un recurso de equilibrador de carga de Azure, configura una dirección IP externa y conecta los pods solicitados al grupo de back-end del equilibrador de carga. Para permitir que el tráfico de los clientes llegue a la aplicación, se crean reglas de equilibrio de carga en los puertos deseados. 

    ![Diagrama que muestra el flujo de tráfico de LoadBalancer en un clúster de AKS][aks-loadbalancer]

    Para mayor control y el enrutamiento del tráfico entrante, puede usar en su lugar un [controlador de entrada](#ingress-controllers).

- **ExternalName**: crea una entrada DNS específica para facilitar el acceso a la aplicación.

La dirección IP de los equilibradores de carga y de los servicios se puede asignar dinámicamente, o puede especificar una dirección IP estática existente para su uso. Se pueden asignar direcciones IP estáticas internas y externas. Esta dirección IP estática existente está a menudo asociada a una entrada DNS.

Se pueden crear equilibradores de carga *internos* y *externos*. Solo se asigna una dirección IP privada a los equilibradores de carga internos, por lo que no se puede acceder a ellos desde Internet.

## <a name="azure-virtual-networks"></a>Redes virtuales de Azure

En AKS, puede implementar un clúster que use uno de los dos siguientes modelos de red:

- Red de *kubenet*: los recursos de la red normalmente se crean y se configuran cuando se implementa el clúster de AKS.
- Red de *Azure Container Networking Interface (CNI)* : el clúster de AKS se conecta a los recursos y configuraciones de la red virtual existentes.

### <a name="kubenet-basic-networking"></a>Red (básica) de kubenet

La opción de red de *kubenet* es la configuración predeterminada para la creación del clúster de AKS. Con *kubenet*, los nodos obtienen una dirección IP de una subred de la red virtual de Azure. Los pods reciben una dirección IP de un espacio de direcciones lógicamente distinto a la subred de red virtual de Azure de los nodos. A continuación, se configura la traducción de direcciones de red (NAT) para que los pods puedan acceder a los recursos en la red virtual de Azure. La dirección IP de origen del tráfico se somete a un proceso NAT hacia la dirección IP principal del nodo.

Los nodos usan el complemento [kubenet][kubenet] de Kubernetes. Puede dejar que la plataforma de Azure cree y configure las redes virtuales por usted o puede implementar el clúster de AKS en una subred de red virtual existente. Nuevamente, solo los nodos reciben una dirección IP enrutable y los pods usan NAT para comunicarse con otros recursos fuera del clúster de AKS. Este enfoque reduce enormemente el número de direcciones IP que se deben reservar en el espacio de red para que los pods las usen.

Para más información, consulte el artículo de [Configuración de la red de kubenet para un clúster de AKS][aks-configure-kubenet-networking].

### <a name="azure-cni-advanced-networking"></a>Redes (avanzadas) de Azure CNI

Con Azure CNI, cada pod obtiene una dirección IP de la subred y se puede acceder a ella directamente. Estas direcciones IP deben ser únicas en el espacio de red y deben planearse de antemano. Cada nodo tiene un parámetro de configuración para el número máximo de pods que admite. Luego, el número equivalente de direcciones IP por nodo se reserva por adelantado para ese nodo. Este enfoque requiere más planificación, ya que de otro modo puede conducir al agotamiento de direcciones IP o a la necesidad de recompilar los clústeres en una subred mayor, a medida que crecen las exigencias de la aplicación.

A diferencia de kubenet, el tráfico hacia los puntos de conexión de la misma red virtual no traduce las direcciones de red a la dirección IP principal del nodo. La dirección de origen para el tráfico dentro de la red virtual es la dirección IP del pod. El tráfico externo a la red virtual sigue traduciendo las direcciones de red a la dirección IP principal del nodo.

Los nodos usan el complemento [Azure Container Networking Interface (CNI)][cni-networking] de Kubernetes.

![Diagrama que muestra dos nodos con puentes que conectan cada uno a una única red virtual de Azure][advanced-networking-diagram]

Para más información, consulte el artículo de [Configuración de Azure CNI para un clúster de AKS][aks-configure-advanced-networking].

### <a name="compare-network-models"></a>Comparación de modelos de red

Kubenet y Azure CNI proporcionan conectividad de red para los clústeres de AKS. Sin embargo, cada uno tiene sus ventajas y sus desventajas. En general, se aplican las siguientes consideraciones:

* **kubenet**
    * Conserva el espacio de direcciones IP.
    * Usa el equilibrador de carga interno o externo de Kubernetes para llegar a los pods desde fuera del clúster.
    * Debe administrar y mantener las rutas definidas por el usuario (UDR) de manera manual.
    * 400 nodos por clúster como máximo.
* **Azure CNI**
    * Los pods obtienen conectividad de red virtual completa y se pueden alcanzar directamente a través de su dirección IP privada desde redes conectadas.
    * Requiere más espacio de direcciones IP.

Existen las siguientes diferencias de comportamiento entre kubenet y Azure CNI:

| Capacidad                                                                                   | Kubenet   | CNI de Azure |
|----------------------------------------------------------------------------------------------|-----------|-----------|
| Implementación del clúster en la red virtual existente o en una nueva                                            | Se admite, las UDR se aplican manualmente | Compatible |
| Conectividad entre pods                                                                         | Compatible | Compatible |
| Conectividad entre la máquina virtual y el pod; con la máquina virtual en la misma red virtual                                          | Funciona cuando lo inicia el pod | Funciona en ambas direcciones |
| Conectividad entre la máquina virtual y el pod; con la máquina virtual en una red virtual del mismo nivel                                            | Funciona cuando lo inicia el pod | Funciona en ambas direcciones |
| Acceso local mediante VPN o ExpressRoute                                                | Funciona cuando lo inicia el pod | Funciona en ambas direcciones |
| Acceso a los recursos protegidos por puntos de conexión de servicio                                             | Compatible | Compatible |
| Expone los servicios de Kubernetes mediante un servicio de equilibrador de carga, App Gateway o un controlador de entrada | Compatible | Compatible |
| Azure DNS y zonas privadas predeterminadas                                                          | Compatible | Compatible |

Con respecto a DNS, con los complementos kubenet y Azure CNI, DNS se ofrece mediante CoreDNS, una implementación que se ejecuta en AKS con su propio escalador automático. Para más información sobre CoreDNS en Kubernetes, consulte [Personalización del servicio DNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/). CoreDNS se configura de forma predeterminada para reenviar dominios desconocidos a los servidores DNS de nodo, es decir, a la funcionalidad DNS de la red virtual de Azure donde se ha implementado el clúster de AKS. Por lo tanto, las zonas privadas y Azure DNS funcionarán para los pods que se ejecutan en AKS.

### <a name="support-scope-between-network-models"></a>Ámbito de compatibilidad entre los modelos de red

Independientemente del modelo de red que use, kubenet y Azure CNI pueden implementarse de una de las siguientes maneras:

* La plataforma Azure puede crear y configurar automáticamente los recursos de red virtual al crear un clúster de AKS.
* Puede crear y configurar los recursos de red virtual manualmente y conectarse a ellos al crear el clúster de AKS.

Aunque se admiten funcionalidades como puntos de conexión de servicio o UDR tanto con kubenet como con Azure CNI, las [directivas de compatibilidad para AKS][support-policies] definen los cambios que se pueden realizar. Por ejemplo:

* Si crea manualmente los recursos de red virtual para un clúster de AKS, tendrá soporte técnico para configurar sus propios puntos de conexión de servicio o UDR.
* Si la plataforma de Azure crea automáticamente los recursos de red virtual para el clúster de AKS, no se admite cambiar manualmente esos recursos administrados de AKS para configurar sus propios UDR o puntos de conexión de servicio.

## <a name="ingress-controllers"></a>Controladores de entrada

Cuando se crea un servicio del tipo LoadBalancer, se crea un recurso subyacente de equilibrador de carga de Azure. El equilibrador de carga está configurado para distribuir el tráfico a los pods en su servicio en un puerto determinado. LoadBalancer solo funciona en la capa 4: el servicio no conoce las aplicaciones reales y no puede realizar ninguna consideración de enrutamiento adicional.

Los *controladores de entrada* funcionan en la capa 7 y puede usar reglas más inteligentes para distribuir el tráfico de la aplicación. Un uso habitual de un controlador de entrada es enrutar el tráfico HTTP a diferentes aplicaciones basadas en la URL de entrada.

![Diagrama que muestra el flujo de tráfico de entrada en un clúster de AKS][aks-ingress]

En AKS, puede crear un recurso de entrada con algo parecido a NGINX, o usar la característica de enrutamiento de aplicación HTTP de AKS. Cuando se habilita el enrutamiento de aplicación HTTP para un clúster de AKS, la plataforma Azure crea el controlador de entrada y un controlador de *DNS externo*. Cuando se crean nuevos recursos de entrada en Kubernetes, se crean los registros DNS A necesarios en una zona DNS específica del clúster. Para más información, consulte el artículo sobre la [implementación del enrutamiento de aplicaciones HTTP][aks-http-routing].

El complemento del controlador de entrada de Application Gateway (AGIC) permite que los clientes de AKS aprovechen la tecnología nativa del equilibrador de carga de nivel 7 de Application Gateway para exponer el software en la nube a Internet. AGIC supervisa el clúster de Kubernetes en el que se hospeda y actualiza continuamente una instancia de Application Gateway para que los servicios seleccionados se expongan a Internet. Para más información sobre el complemento de AGIC para AKS, consulte [¿Qué es controlador de entrada de Application Gateway?][agic-overview]

Otra característica común de los controladores de entrada es la terminación SSL/TLS. En aplicaciones web de gran tamaño, a las que se accede a través de HTTPS, la terminación TLS puede controlarse mediante el recurso de entrada en lugar de en la propia aplicación. Para proporcionar una configuración y generación automática de certificados TLS, puede configurar el recurso de entrada para que use proveedores como Let's Encrypt. Para más información sobre cómo configurar un controlador de entrada de NGINX con Let's Encrypt, consulte el artículo de [Entrada y TLS][aks-ingress-tls].

También puede configurar el controlador de entrada para conservar la dirección IP de origen de cliente en las solicitudes a los contenedores en el clúster de AKS. Si la solicitud de un cliente se enruta a un contenedor del clúster de AKS mediante el controlador de entrada, la dirección IP de origen de la solicitud no estará disponible para el contenedor de destino. Al habilitar la *conservación de la dirección IP de origen de cliente*, la dirección IP de origen para el cliente está disponible en el encabezado de solicitud en *X-Forwarded-For*. Si usa la conservación de la dirección IP de origen de cliente en el controlador de entrada, no puede usar TLS de paso a través. La conservación de la dirección IP de origen de cliente y TLS de paso a través pueden usarse con otros servicios, como de tipo *LoadBalancer*.

## <a name="network-security-groups"></a>Grupos de seguridad de red

Un grupo de seguridad de red filtra el tráfico de las máquinas virtuales como, por ejemplo, el de los nodos de AKS. Al crear los servicios, como LoadBalancer, la plataforma Azure configura automáticamente las reglas de grupo de seguridad de red que son necesarias. No configure manualmente las reglas del grupo de seguridad de red para filtrar el tráfico de los pods en un clúster de AKS. Defina los puertos necesarios y el reenvío como parte de los manifiestos de servicio de Kubernetes y permita que la plataforma Azure cree o actualice las reglas correspondientes. También puede usar las directivas de red, tal como se describe en la sección siguiente, para aplicar automáticamente las reglas de filtro de tráfico a los pods.

## <a name="network-policies"></a>Directivas de red

De forma predeterminada, todos los pods de un clúster de AKS pueden enviar y recibir tráfico sin limitaciones. Para mejorar la seguridad, puede definir reglas que controlan el flujo de tráfico. Las aplicaciones de back-end solo se suelen exponer en los servicios de front-end, o los componentes de base de datos solo son accesibles para las capas de aplicación que se conectan a ellos.

La directiva de red es una característica de Kubernetes disponible en AKS que permite controlar el flujo de tráfico entre pods. Puede permitir o denegar el tráfico según la configuración como etiquetas asignadas, espacio de nombres o puerto de tráfico. Los grupos de seguridad de red se usan más para los nodos de AKS y no para los pods. El uso de directivas de red proporciona una manera más adecuada y nativa en la nube para controlar el flujo de tráfico. Como los pods se crean dinámicamente en un clúster de AKS, se pueden aplicar automáticamente las directivas de red necesarias.

Para más información, consulte [Protección del tráfico entre pods mediante directivas de red en Azure Kubernetes Service (AKS)][use-network-policies].

## <a name="next-steps"></a>Pasos siguientes

Para empezar a trabajar con las redes de AKS, cree y configure un clúster de AKS con sus propios intervalos de direcciones IP mediante [kubenet][aks-configure-kubenet-networking] o [CNI de Azure][aks-configure-advanced-networking].

Para los procedimientos recomendados asociados, consulte [Procedimientos recomendados con la conectividad de red y la seguridad en Azure Kubernetes Service (AKS)][operator-best-practices-network].

Para obtener más información sobre los conceptos básicos de Kubernetes y AKS, consulte los artículos siguientes:

- [Clústeres y cargas de trabajo de Kubernetes/AKS][aks-concepts-clusters-workloads]
- [Acceso a Kubernetes/AKS e identidad][aks-concepts-identity]
- [Seguridad de Kubernetes/AKS][aks-concepts-security]
- [Almacenamiento de Kubernetes/AKS][aks-concepts-storage]
- [Escala de Kubernetes/AKS][aks-concepts-scale]

<!-- IMAGES -->
[aks-clusterip]: ./media/concepts-network/aks-clusterip.png
[aks-nodeport]: ./media/concepts-network/aks-nodeport.png
[aks-loadbalancer]: ./media/concepts-network/aks-loadbalancer.png
[advanced-networking-diagram]: ./media/concepts-network/advanced-networking-diagram.png
[aks-ingress]: ./media/concepts-network/aks-ingress.png

<!-- LINKS - External -->
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet

<!-- LINKS - Internal -->
[aks-http-routing]: http-application-routing.md
[aks-ingress-tls]: ./ingress-tls.md
[aks-configure-kubenet-networking]: configure-kubenet.md
[aks-configure-advanced-networking]: configure-azure-cni.md
[aks-concepts-clusters-workloads]: concepts-clusters-workloads.md
[aks-concepts-security]: concepts-security.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-storage]: concepts-storage.md
[aks-concepts-identity]: concepts-identity.md
[agic-overview]: ../application-gateway/ingress-controller-overview.md
[use-network-policies]: use-network-policies.md
[operator-best-practices-network]: operator-best-practices-network.md
[support-policies]: support-policies.md
