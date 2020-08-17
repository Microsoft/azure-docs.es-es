---
title: Procedimientos recomendados de seguridad de la red
titleSuffix: Azure Kubernetes Service
description: Procedimientos recomendados con los recursos de red virtual y la conectividad para el operador del clúster en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: conceptual
ms.date: 12/10/2018
ms.openlocfilehash: fc839fd69e3b574c47aa7bb712583dfc0b9c711d
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542711"
---
# <a name="best-practices-for-network-connectivity-and-security-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados con la conectividad de red y la seguridad en Azure Kubernetes Service (AKS)

Al crear y administrar clústeres en Azure Kubernetes Service (AKS), se proporciona conectividad de red para los nodos y las aplicaciones. Estos recursos de red incluyen los intervalos de direcciones IP, los equilibradores de carga y los controladores de entrada. Para mantener un servicio de alta calidad para las aplicaciones, debe planear y configurar estos recursos.

Este artículo de procedimientos recomendados se centra en la conectividad de red y la seguridad para los operadores del clúster. En este artículo aprenderá a:

> [!div class="checklist"]
> * Comparar los modos de red de kubenet y de Azure CNI en AKS
> * Planear la conectividad y el direccionamiento IP necesarios
> * Distribuir el tráfico mediante equilibradores de carga, controladores de entrada o un firewall de aplicaciones web (WAF)
> * Conectarse de forma segura a los nodos del clúster

## <a name="choose-the-appropriate-network-model"></a>Elección del modelo de red adecuado

**Guía de procedimiento recomendado**: para la integración con redes virtuales existentes o redes locales, use las redes de Azure CNI en AKS. Este modelo de red también permite mayor separación de los recursos y los controles en un entorno empresarial.

Las redes virtuales proporcionan conectividad básica para que los nodos de AKS y los clientes accedan a las aplicaciones. Hay dos maneras diferentes de implementar los clústeres de AKS en redes virtuales:

* **Redes de kubenet**: cuando el clúster se implementa y usa el complemento [kubenet][kubenet] de Kubernetes, Azure administra los recursos de red virtual.
* **Redes de Azure CNI**: se implementan en una red virtual existente y usan el complemento [Azure Container Networking Interface (CNI)][cni-networking] de Kubernetes. Los pods reciben direcciones IP individuales que se pueden enrutar a otros servicios de red o recursos locales.

Container Networking Interface (CNI) es un protocolo independiente del proveedor que permite que el entorno de ejecución del contenedor realice solicitudes a un proveedor de red. Azure CNI asigna direcciones IP a los nodos y los pods, y proporciona características de administración de direcciones IP(IPAM) cuando se conecta a redes virtuales de Azure existentes. Cada recurso de nodo y pod recibe una dirección IP en la red virtual de Azure y no se necesita más enrutamiento para la comunicación con otros servicios o recursos.

![Diagrama que muestra dos nodos con puentes que conectan cada uno a una única red virtual de Azure](media/operator-best-practices-network/advanced-networking-diagram.png)

En las implementaciones de producción, tanto kubenet como Azure CNI son opciones válidas.

Una ventaja importante de las redes de Azure CNI para producción es que el modelo de red permite separar el control y la administración de los recursos. Desde una perspectiva de seguridad, se suele preferir que distintos equipos administren y protejan los recursos. Las redes de Azure CNI permiten la conexión directa a recursos de Azure existentes, a recursos locales o a otros servicios mediante direcciones IP asignadas a cada pod.

Al usar redes de Azure CNI, el recurso de red virtual se encuentra en un grupo de recursos independiente del clúster de AKS. Delegue permisos en la entidad de servicio de AKS para acceder y administrar estos recursos. La entidad de servicio usada por el clúster de AKS debe tener al menos permisos de [colaborador de la red](../role-based-access-control/built-in-roles.md#network-contributor) en la subred de la red virtual. Si quiere definir un [rol personalizado](../role-based-access-control/custom-roles.md) en lugar de usar el rol integrado de colaborador de red, se requieren los permisos siguientes:
  * `Microsoft.Network/virtualNetworks/subnets/join/action`
  * `Microsoft.Network/virtualNetworks/subnets/read`

Para obtener más información sobre la delegación en entidad de servicio de AKS, consulte [Delegación del acceso a otros recursos de Azure][sp-delegation]. En lugar de una entidad de servicio, también puede usar la identidad administrada asignada por el sistema para los permisos. Para más información, consulte [Uso de identidades administradas](use-managed-identity.md).

Como cada pod y cada nodo recibe su propia dirección IP, planee los intervalos de direcciones para las subredes de AKS. La subred debe ser lo suficientemente grande como para proporcionar direcciones IP para cada nodo, pod y recurso de red que implemente. Cada clúster de AKS se debe colocar en su propia subred. Para permitir la conectividad con redes locales o emparejadas en Azure, no use intervalos de direcciones IP que se superpongan con recursos de red existentes. El número de pods que cada nodo ejecuta con las redes de kubenet y de Azure CNI tiene un límite predeterminado. Para controlar los eventos de escalado horizontal o las actualizaciones de clúster, también es necesario que haya otras direcciones IP disponibles para utilizarlas en la subred asignada. Este espacio de direcciones adicional es especialmente importante si usa contenedores de Windows Server, como aquellos grupos de nodos que requieren una actualización para aplicar las revisiones de seguridad más recientes. Para obtener más información sobre los nodos de Windows Server, consulte [Actualización de un grupo de nodos en AKS][nodepool-upgrade].

Para calcular la dirección IP necesaria, consulte [Configuración de redes de Azure CNI en AKS][advanced-networking].

### <a name="kubenet-networking"></a>Redes de kubenet

Aunque las redes de kubenet no requieren configuración de las redes virtuales antes de la implementación del clúster, existen algunos inconvenientes:

* Los nodos y los pods se colocan en subredes IP diferentes. El enrutamiento definido por el usuario (UDR) y el reenvío de IP se usan para enrutar el tráfico entre los pods y los nodos. Este enrutamiento adicional puede reducir el rendimiento de las redes.
* Las conexiones a redes locales existentes o el emparejamiento con otras redes virtuales de Azure pueden ser complejos.

Kubenet es adecuado para implementaciones pequeñas o cargas de trabajo de prueba, ya que no es necesario crear la red virtual y las subredes independientemente del clúster de AKS. Los sitios web sencillos con poco tráfico o que sirven para levantar y desplazar cargas de trabajo en contenedores también se pueden beneficiar de la sencillez de los clústeres de AKS que se implementan con redes de kubenet. Para la mayoría de las implementaciones de producción, debe planificar y usar redes de Azure CNI. También puede [configurar sus propios intervalos de direcciones IP y redes virtuales con kubenet][aks-configure-kubenet-networking].

## <a name="distribute-ingress-traffic"></a>Distribución del tráfico de entrada

**Guía de procedimiento recomendado**: para distribuir el tráfico HTTP o HTTPS a las aplicaciones, use los recursos y los controladores de entrada. Los controladores de entrada proporcionan características adicionales a las de un equilibrador de carga de Azure común y se pueden administrar como recursos nativos de Kubernetes.

Un equilibrador de carga de Azure puede distribuir el tráfico de cliente a las aplicaciones en el clúster de AKS, pero su conocimiento sobre ese tráfico es limitado. Un recurso de equilibrador de carga funciona en la capa 4 y distribuye el tráfico en función de protocolos o puertos. La mayoría de las aplicaciones web que usan HTTP o HTTPS deben usar recursos y controladores de entrada de Kuberenetes, que funcionan en la capa 7. En la entrada se puede distribuir el tráfico en función de la dirección URL de la aplicación y administrar la terminación TLS/SSL. Esta capacidad también reduce el número de direcciones IP que se exponen y se asignan. Con un equilibrador de carga, cada aplicación normalmente necesita una dirección IP pública asignada y a ella y al servicio del clúster de AKS. Con un recurso de entrada, una única dirección IP puede distribuir tráfico a varias aplicaciones.

![Diagrama que muestra el flujo de tráfico de entrada en un clúster de AKS](media/operator-best-practices-network/aks-ingress.png)

 En la entrada existen dos componentes:

 * un *recurso* de entrada y
 * un *controlador* de entrada

El recurso de entrada es un manifiesto YAML de `kind: Ingress` que define el host, los certificados y las reglas para enrutar el tráfico a los servicios que se ejecutan en el clúster de AKS. El siguiente ejemplo de manifiesto de YAML distribuye el tráfico de *myapp.com* a uno de los dos servicios, *blogservice* o *storeservice*. El cliente se dirige a un servicio o al otro en función de la dirección URL a la que acceden.

```yaml
kind: Ingress
metadata:
 name: myapp-ingress
   annotations: kubernetes.io/ingress.class: "PublicIngress"
spec:
 tls:
 - hosts:
   - myapp.com
   secretName: myapp-secret
 rules:
   - host: myapp.com
     http:
      paths:
      - path: /blog
        backend:
         serviceName: blogservice
         servicePort: 80
      - path: /store
        backend:
         serviceName: storeservice
         servicePort: 80
```

Un controlador de entrada es un demonio que se ejecuta en un nodo de AKS y supervisa las solicitudes entrantes. A continuación, el tráfico se distribuye según las reglas definidas en el recurso de entrada. El controlador de entrada más común se basa en [NGINX]. AKS no restringe al usuario a un controlador específico, por lo que puede usar otros, como [Contour][contour], [HAProxy][haproxy] o [Traefik][traefik].

El controlador de entrada también debe programarse en un nodo de Linux. Los nodos de Windows Server no deben ejecutar el controlador de entrada. Use un selector de nodo en el manifiesto YAML o implementación de gráfico de Helm para indicar que el recurso debe ejecutarse en un nodo basado en Linux. Para obtener más información, consulte [Uso de selectores de nodo para controlar dónde se programan los pods en AKS][concepts-node-selectors].

Hay muchos escenarios de entrada, incluidas las siguientes guías paso a paso:

* [Creación de un controlador de entrada básico con conectividad de red externa][aks-ingress-basic]
* [Creación de un controlador de entrada que use una red privada interna y una dirección IP][aks-ingress-internal]
* [Crear un controlador de entrada que usa sus propios certificados TLS][aks-ingress-own-tls]
* Creación de un controlador de entrada que use Let's Encrypt para generar certificados TLS de forma automática [con una dirección IP pública dinámica][aks-ingress-tls] o [con una dirección IP pública estática][aks-ingress-static-tls]

## <a name="secure-traffic-with-a-web-application-firewall-waf"></a>Protección del tráfico con un firewall de aplicaciones web (WAF)

**Guía de procedimientos recomendados**: para analizar el tráfico entrante en busca de posibles ataques, use un firewall de aplicaciones web (WAF) como [Barracuda WAF para Azure][barracuda-waf] o Azure Application Gateway. Estos recursos de red más avanzados también pueden enrutar el tráfico más allá de las conexiones HTTP y HTTPS sencillas o la terminación TLS básica.

Un controlador de entrada que distribuye el tráfico a servicios y aplicaciones suele ser un recurso de Kubernetes en el clúster de AKS. El controlador se ejecuta como demonio en un nodo de AKS y consume algunos de los recursos del nodo, como la CPU, la memoria y el ancho de banda de red. En entornos mayores, con frecuencia se prefiere descargar algo de tráfico mediante el enrutamiento o la terminación TLS a un recurso de red externo al clúster de AKS. También puede analizar el tráfico entrante en busca de posibles ataques.

![Un firewall de aplicaciones web (WAF), como Azure Application Gateway, puede proteger y distribuir el tráfico del clúster de AKS.](media/operator-best-practices-network/web-application-firewall-app-gateway.png)

Un firewall de aplicaciones web (WAF) proporciona una capa adicional de seguridad mediante el filtrado del tráfico entrante. Open Web Application Security Project (OWASP) proporciona un conjunto de reglas para detectar ataques, como el cross site scripting o el envenenamiento de cookies. [Azure Application Gateway][app-gateway] (actualmente en versión preliminar en AKS) es una solución WAF que se puede integrar con los clústeres de AKS para proporcionar estas características de seguridad antes de que el tráfico llegue al clúster y a las aplicaciones de AKS. Otras soluciones de terceros también realizan estas funciones, por lo que puede seguir usando las inversiones existentes o la experiencia en un producto determinado.

Los recursos de entrada o el equilibrador de carga continúan ejecutándose en el clúster de AKS para refinar aún más la distribución del tráfico. Application Gateway se puede administrar de manera centralizada como controlador de entrada con definición de recurso. Para empezar, [cree un controlador de entrada de Application Gateway][app-gateway-ingress].

## <a name="control-traffic-flow-with-network-policies"></a>Control del flujo de tráfico con directivas de red

**Guía de procedimientos recomendados**: use directivas de red para permitir o denegar el tráfico a los pods. De forma predeterminada, se permite todo el tráfico entre los pods dentro de un clúster. Para mejorar la seguridad, defina reglas que limiten la comunicación del pod.

La directiva de red es una característica de Kubernetes que permite controlar el flujo de tráfico entre pods. Puede permitir o denegar el tráfico según la configuración como etiquetas asignadas, espacio de nombres o puerto de tráfico. El uso de directivas de red proporciona una manera nativa en la nube para controlar el flujo de tráfico. Como los pods se crean dinámicamente en un clúster de AKS, se pueden aplicar automáticamente las directivas de red necesarias. No use grupos de seguridad de red de Azure para controlar el tráfico de pod a pod, use las directivas de red.

Para usar la directiva de red, la característica debe habilitarse al crear un clúster de AKS. No se puede habilitar la directiva de red en un clúster de AKS existente. Planee con antelación para asegurarse de que habilita las directivas de red en los clústeres y de que puede usarlas según sea necesario. La directiva de red solo se debe usar para los pods y los nodos basados en Linux de AKS.

Se crea una directiva de red como un recurso de Kubernetes mediante un manifiesto YAML. Las directivas se aplican a los pods definidos y, a continuación, las reglas de entrada o salida definen cómo puede fluir el tráfico. El ejemplo siguiente aplica una directiva de red a los pods que tienen la etiqueta *app: backend*. La regla de entrada solo permite el tráfico desde los pods con la etiqueta *app: frontend*:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

Para empezar a usar directivas, consulte [Protección del tráfico entre pods mediante directivas de red en Azure Kubernetes Service (AKS)][use-network-policies].

## <a name="securely-connect-to-nodes-through-a-bastion-host"></a>Conexión segura a los nodos mediante una pasarela de aplicaciones

**Guía de procedimiento recomendado**: no exponga la conectividad remota a los nodos de AKS. Cree una pasarela de aplicaciones, o host jump, en una red virtual de administración. Utilice la pasarela de aplicaciones para enrutar el tráfico de forma segura en el clúster de AKS para las tareas de administración remota.

La mayoría de las operaciones en AKS puede realizarse con las herramientas de administración de Azure o mediante el servidor de API de Kubernetes. Los nodos de AKS no están conectados a la red pública de internet y solo están disponibles en una red privada. Para conectarse a los nodos y realizar tareas de mantenimiento o solucionar problemas, enrute las conexiones mediante una pasarela de aplicaciones o host de salto. Este host debe estar en una red virtual de administración independiente emparejada de forma segura a la red virtual del clúster de AKS.

![Conexión a los nodos de AKS mediante una pasarela de aplicaciones o host de salto](media/operator-best-practices-network/connect-using-bastion-host-simplified.png)

La red de administración para la pasarela de aplicaciones también debe protegerse. Use [Azure ExpressRoute][expressroute] o [VPN Gateway][vpn-gateway] para conectarse a una red local y controlar el acceso con grupos de seguridad de red.

## <a name="next-steps"></a>Pasos siguientes

Este artículo se centra en la conectividad de red y la seguridad. Para obtener más información sobre los conceptos básicos de red en Kubernetes, consulte [Conceptos de redes de aplicaciones en Azure Kubernetes Service (AKS)][aks-concepts-network].

<!-- LINKS - External -->
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet
[app-gateway-ingress]: https://github.com/Azure/application-gateway-kubernetes-ingress
[nginx]: https://www.nginx.com/products/nginx/kubernetes-ingress-controller
[contour]: https://github.com/heptio/contour
[haproxy]: https://www.haproxy.org
[traefik]: https://github.com/containous/traefik
[barracuda-waf]: https://www.barracuda.com/products/webapplicationfirewall/models/5

<!-- INTERNAL LINKS -->
[aks-concepts-network]: concepts-network.md
[sp-delegation]: kubernetes-service-principal.md#delegate-access-to-other-azure-resources
[expressroute]: ../expressroute/expressroute-introduction.md
[vpn-gateway]: ../vpn-gateway/vpn-gateway-about-vpngateways.md
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-own-tls]: ingress-own-tls.md
[app-gateway]: ../application-gateway/overview.md
[use-network-policies]: use-network-policies.md
[advanced-networking]: configure-azure-cni.md
[aks-configure-kubenet-networking]: configure-kubenet.md
[concepts-node-selectors]: concepts-clusters-workloads.md#node-selectors
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool