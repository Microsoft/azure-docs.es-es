---
title: Configuración de redes de Azure CNI en Azure Kubernetes Service (AKS)
description: Aprenda a configurar redes de Azure CNI (avanzadas) en Azure Kubernetes Service (AKS), incluida la implementación de un clúster de AKS en una red virtual existente y una subred.
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 10/11/2018
ms.author: iainfou
ms.openlocfilehash: 9006590583f0ef52bbce716529534f8bce6f47c5
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780364"
---
# <a name="configure-azure-cni-networking-in-azure-kubernetes-service-aks"></a>Configuración de redes de Azure CNI en Azure Kubernetes Service (AKS)

De forma predeterminada, los clústeres de AKS usan [kubenet][kubenet], y una red virtual y una subred se crean automáticamente. Con *kubenet*, los nodos obtienen una dirección IP de una subred de la red virtual. Luego, la traducción de direcciones de red (NAT) se configura en los nodos, y los pods reciben una dirección IP "oculta" detrás de la dirección IP del nodo. Este enfoque reduce el número de direcciones IP que se deben reservar en el espacio de red para su uso por parte de los pods.

Con [Azure Container Networking Interface (CNI)][cni-networking], cada pod obtiene una dirección IP de la subred, y se puede acceder directamente. Estas direcciones IP deben ser únicas en el espacio de red y deben planearse de antemano. Cada nodo tiene un parámetro de configuración para el número máximo de pods que admite. Luego, el número equivalente de direcciones IP por nodo se reserva por adelantado para ese nodo. Este enfoque requiere más planeación y a menudo lleva al agotamiento de direcciones IP o a la necesidad de volver a generar los clústeres en una subred mayor, a medida que crezcan las exigencias de la aplicación.

En este artículo se muestra cómo usar las redes de *Azure CNI* para crear y usar la subred una red virtual con un clúster de AKS. Para más información sobre las opciones y consideraciones de red, consulte el artículo sobre los [conceptos de red para Kubernetes y AKS][aks-network-concepts].

## <a name="prerequisites"></a>Requisitos previos

* La red virtual del clúster AKS debe permitir la conectividad saliente de Internet.
* No cree más de un clúster AKS en la misma subred.
* Los clústeres AKS no pueden usar `169.254.0.0/16`, `172.30.0.0/16` o `172.31.0.0/16` para el intervalo de direcciones del servicio de Kubernetes.
* La entidad de servicio usada por el clúster de AKS debe tener al menos permisos de [colaborador de la red](../role-based-access-control/built-in-roles.md#network-contributor) en la subred de la red virtual. Si quiere definir un [rol personalizado](../role-based-access-control/custom-roles.md) en lugar de usar el rol integrado de colaborador de red, se requieren los permisos siguientes:
  * `Microsoft.Network/virtualNetworks/subnets/join/action`
  * `Microsoft.Network/virtualNetworks/subnets/read`

## <a name="plan-ip-addressing-for-your-cluster"></a>Planeamiento de direccionamiento IP del clúster

Los clústeres configurados con redes de Azure CNI requieren planeamiento adicional. Los tamaños de red virtual y de subred deben ajustarse al número de pods que tiene previsto ejecutar, así como al número de nodos del clúster.

Se asignan direcciones IP para los pods y los nodos del clúster desde la subred especificada dentro de la red virtual. Cada nodo se configura con una dirección IP principal. De forma predeterminada, Azure CNI configura previamente treinta direcciones IP adicionales que se asignan a los pods programados en el nodo. Cuando se escala horizontalmente el clúster, cada nodo del mismo modo se configura de modo similar con direcciones IP de la subred. También puede ver el [número máximo de pods por nodo](#maximum-pods-per-node).

> [!IMPORTANT]
> El número de direcciones IP necesarias debe incluir consideraciones sobre las operaciones de actualización y escalabilidad. Si establece el intervalo de direcciones IP para que solo se admita un número fijo de nodos, no se podrá actualizar o escalar el clúster.
>
> - Al **actualizar** el clúster de AKS, se implementa un nuevo nodo en el clúster. Los servicios y las cargas de trabajo comienzan a ejecutarse en el nuevo nodo y el nodo antiguo se quita del clúster. Para realizar este proceso de actualización gradual es necesario que haya disponible un bloque adicional de direcciones IP. El recuento de nodos es entonces `n + 1`.
>
> - Al **escalar** un clúster de AKS, se implementa un nuevo nodo en el clúster. Los servicios y las cargas de trabajo comienzan a ejecutarse en el nuevo nodo. El intervalo de direcciones IP debe tener en cuenta cómo podría escalar verticalmente el número de nodos y los pods que el clúster puede admitir. También se debe incluir un nodo adicional para las operaciones de actualización. El recuento de nodos es entonces `n + number-of-additional-scaled-nodes-you-anticipate + 1`.

Si espera que los nodos ejecuten el número máximo de pods, y destruye e implementa pods con regularidad, también debe contar con algunas direcciones IP adicionales por nodo. Estas direcciones IP adicionales tienen en cuenta que la eliminación de un servicio y la implementación de la dirección IP liberada para un nuevo servicio pueden tardar unos segundos, y adquieren la dirección.

El plan de direcciones IP de un clúster AKS consta de una red virtual, al menos una subred para los nodos y pods, y un intervalo de direcciones del servicio de Kubernetes.

| Intervalo de direcciones / recurso de Azure | Límites y tamaño |
| --------- | ------------- |
| Virtual network | El tamaño de Azure Virtual Network puede ser de /8, pero se limita a 65 536 direcciones IP configuradas. |
| Subred | Debe ser lo suficientemente grande como para dar cabida a los nodos, los pods y a todos los recursos de Kubernetes y de Azure que se pueden aprovisionar en el clúster. Por ejemplo, si implementa una instancia de Azure Load Balancer interna, sus direcciones IP de front-end se asignan desde la subred del clúster, no las direcciones IP públicas. El tamaño de la subred también debe tener en cuenta las operaciones de actualización o las necesidades futuras de escalabilidad.<p />Para calcular el tamaño *mínimo* de subred, incluido un nodo adicional para operaciones de actualización: `(number of nodes + 1) + ((number of nodes + 1) * maximum pods per node that you configure)`<p/>Ejemplo de un clúster de 50 nudos: `(51) + (51  * 30 (default)) = 1,581` (/21 o superior)<p/>Ejemplo de un clúster de 50 nodos que también incluye aprovisionamiento para escalar verticalmente 10 nodos adicionales: `(61) + (61 * 30 (default)) = 1,891` (/21 o superior)<p>Si no especifica un número máximo de pods por nodo al crear el clúster, el número máximo de pods por nodo se establece en *30*. El número mínimo de direcciones IP requeridas se basa en ese valor. Si se calculan los requisitos mínimos de dirección IP según otro valor máximo distinto, vea [cómo configurar el número máximo de pods por nodo](#configure-maximum---new-clusters) para establecer este valor cuando se implementa el clúster. |
| Intervalo de direcciones del servicio de Kubernetes | Este intervalo no lo debe usar ningún elemento de red de esta red virtual o que esté conectado a ella. El CIDR de la dirección del servicio debe ser menor que /12. |
| Dirección IP del servicio DNS de Kubernetes | Dirección IP del intervalo de direcciones del servicio de Kubernetes que se usará en la detección de servicios de clúster (kube-dns). No use la primera dirección IP en el intervalo de direcciones, como .1. La primera dirección del intervalo de la subred se usa para la dirección *kubernetes.default.svc.cluster.local*. |
| Dirección de puente de Docker | Dirección IP (en notación CIDR) que se usa como la dirección IP del puente de Docker en los nodos. El valor predeterminado es 172.17.0.1/16. |

## <a name="maximum-pods-per-node"></a>Pods máximos por nodo

El número máximo de pods por nodo en un clúster de AKS es 250. El número máximo *predeterminado* de pods por nodo varía entre las redes de *kubenet*y de *Azure CNI*, y el método de implementación del clúster.

| Método de implementación | Kubenet predeterminado | Azure CNI predeterminado | Configurable en la implementación |
| -- | :--: | :--: | -- |
| Azure CLI | 110 | 30 | Sí (hasta 250) |
| Plantilla de Resource Manager | 110 | 30 | Sí (hasta 250) |
| Portal | 110 | 30 | No |

### <a name="configure-maximum---new-clusters"></a>Configurar máximo: nuevos clústeres

Puede configurar el número máximo de pods por nodo *solo en tiempo de implementación del clúster*. Si implementa con la CLI de Azure o con una plantilla de Resource Manager, puede establecer los pods máximos por cada valor de nodo que se necesiten en la siguiente `maxPods` directrices:

| Redes | Mínimo | Máximo |
| -- | :--: | :--: |
| Azure CNI | 30 | 250 |
| Kubenet | 30 | 110 |

> [!NOTE]
> El valor mínimo en la tabla anterior se aplica estrictamente por el servicio AKS.
No se puede establecer un valor maxPods inferior al valor mínimo que se muestra como si lo hace puede impedir que el clúster de inicio.

* **CLI de Azure**: especifique el argumento `--max-pods` cuando implemente un clúster con el comando [az aks create][az-aks-create]. El valor máximo es 250.
* **Plantilla de Resource Manager**: especifique la propiedad `maxPods` del objeto [ManagedClusterAgentPoolProfile] cuando implemente un clúster con una plantilla de Resource Manager. El valor máximo es 250.
* **Portal de Azure**: no puede modificar el número máximo de pods por nodo cuando implemente un clúster con Azure Portal. Los clústeres de redes de Azure CNI están limitados a 30 pods por nodo cuando se implementan mediante Azure Portal.

### <a name="configure-maximum---existing-clusters"></a>Configurar máximo: clústeres existentes

No se puede cambiar el número máximo de pods por nodo en un clúster de AKS existente. Solo puede ajustar el número cuando se implementa inicialmente el clúster.

## <a name="deployment-parameters"></a>Parámetros de implementación

Cuando crea un clúster de AKS, los parámetros siguientes son configurables para redes de Azure CNI:

**Red virtual**: la red virtual en la que desea implementar el clúster de Kubernetes. Si desea crear una red virtual nueva para el clúster, seleccione *Crear nueva* y siga los pasos descritos en la sección *Creación de red virtual*. Para información acerca de límites y cuotas para Azure Virtual Network, vea [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits).

**Subred**: la subred dentro de la red virtual en la que desea implementar el clúster. Si desea crear una nueva subred en la red virtual para el clúster, seleccione *Crear nueva* y siga los pasos descritos en la sección *Creación de subred*. Para la conectividad híbrida, el intervalo de direcciones no debe solaparse con ninguna otra red virtual de su entorno.

**Intervalo de direcciones de servicio de Kubernetes**: Este es el conjunto de direcciones IP virtuales que Kubernetes asigna a [servicios][services] internos en el clúster. Puede usar cualquier intervalo de direcciones privado que cumpla los requisitos siguientes:

* No debe estar dentro del intervalo de direcciones IP de la red virtual del clúster.
* No deben superponerse con ninguna otra red virtual del mismo nivel que la red virtual del clúster.
* No debe superponerse con ninguna dirección IP local.
* No debe estar dentro de los intervalos `169.254.0.0/16`, `172.30.0.0/16` o `172.31.0.0/16`

Aunque es técnicamente posible especificar un intervalo de direcciones de servicio en la misma red virtual que el clúster, no se recomienda. Puede producirse un comportamiento impredecible si se usan intervalos IP superpuestos. Para más información, consulte la sección [P+F](#frequently-asked-questions) de este artículo. Para más información sobre los servicios de Kubernetes, consulte [Servicios][services] en la documentación de Kubernetes.

**Dirección IP del servicio DNS de Kubernetes**:  la dirección IP del servicio DNS del clúster. Esta dirección debe estar dentro del  *intervalo de direcciones del servicio Kubernetes*. No use la primera dirección IP en el intervalo de direcciones, como .1. La primera dirección del intervalo de la subred se usa para la dirección *kubernetes.default.svc.cluster.local*.

**Dirección del puente de Docker**: la dirección IP y la máscara de red que se van a asignar al puente de Docker. El puente de Docker permite que los nodos de AKS se comuniquen con la plataforma de administración subyacente. Esta dirección IP no debe estar dentro del rango de direcciones IP de red virtual del clúster, y no debe superponerse con otros rangos de direcciones en uso en la red.

## <a name="configure-networking---cli"></a>Configuración de redes: CLI

Al crear un clúster de AKS con la CLI de Azure, también puede configurar redes de Azure CNI. Use los siguientes comandos para crear un nuevo clúster de AKS con las redes de Azure CNI habilitadas.

En primer lugar, obtenga el identificador de recursos de subred para la subred existente a la que se unirá el clúster de AKS:

```console
$ az network vnet subnet list \
    --resource-group myVnet \
    --vnet-name myVnet \
    --query [].id --output tsv

/subscriptions/<guid>/resourceGroups/myVnet/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/default
```

Use el comando [crear az aks][az-aks-create] con el argumento `--network-plugin azure` para crear un clúster con redes avanzadas. Actualice el valor `--vnet-subnet-id` con el identificador de subred recopilado en el paso anterior:

```azurecli
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --network-plugin azure \
    --vnet-subnet-id <subnet-id> \
    --docker-bridge-address 172.17.0.1/16 \
    --dns-service-ip 10.2.0.10 \
    --service-cidr 10.2.0.0/24
```

## <a name="configure-networking---portal"></a>Configuración de redes: portal

La siguiente captura de pantalla de Azure Portal muestra un ejemplo de la configuración de estos valores durante la creación del clúster de AKS:

![Configuración de redes avanzada en Azure Portal][portal-01-networking-advanced]

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

Las siguientes preguntas y respuestas se aplican a la configuración de red de **Azure CNI**.

* *¿Puedo implementar máquinas virtuales en la subred del clúster?*

  No. No se admite la implementación de máquinas virtuales en la subred usada por el clúster Kubernetes. Las máquinas virtuales se pueden implementar en la misma red virtual, pero en una subred diferente.

* *¿Puedo configurar las directivas de red por pod?*

  Sí, la directiva de red de Kubernetes está disponible en AKS. Para empezar, vea [proteger el tráfico entre pods mediante el uso de directivas de redes en AKS][network-policy].

* *¿Es configurable el número máximo de pods que se puede implementar en un nodo ?*

  Sí, al implementar un clúster con la CLI de Azure o una plantilla de Resource Manager. Consulte [Pods máximos por nodo](#maximum-pods-per-node).

  No se puede cambiar el número máximo de pods por nodo en un clúster existente.

* *¿Cómo se pueden configurar propiedades adicionales para la subred que he creado durante la creación del clúster de AKS? Por ejemplo, los puntos de conexión de servicio.*

  La lista completa de propiedades de la red virtual y las subredes que se crean durante la creación del clúster de AKS puede configurarse en la página de configuración de red virtual estándar en Azure Portal.

* *¿Puedo usar una subred diferente dentro de mi red virtual de clúster como* **intervalo de direcciones de servicio de Kubernetes**?

  Aunque no se recomienda, esta configuración es posible. El rango de direcciones de servicio es un conjunto de direcciones IP virtuales (VIP) que Kubernetes asigna a los servicios internos del clúster. Las redes de Azure no tiene ninguna visibilidad sobre el intervalo de direcciones IP de servicio del clúster de Kubernetes. Debido a ello, es posible crear posteriormente una nueva subred en la red virtual del clúster que se superponga con el intervalo de direcciones de servicio. Si se produce una superposición de este tipo, Kubernetes podría asignar a un servicio una dirección IP que ya esté en uso por otro recurso de la subred, lo que provocaría un comportamiento impredecible o errores. Al tener la seguridad de usar un intervalo de direcciones que se encuentra fuera de la red virtual del clúster puede evitar este riesgo de superposición.

## <a name="next-steps"></a>Pasos siguientes

### <a name="networking-in-aks"></a>Redes en AKS

Más información acerca de las redes en AKS en los siguientes artículos:

- [Uso de una dirección IP estática con el equilibrador de carga de Azure Kubernetes Service (AKS)](static-ip.md)
- [Uso de un equilibrador de carga interno con Azure Container Service (AKS)](internal-lb.md)

- [Crear un controlador de entrada básico con conectividad de red externa][aks-ingress-basic]
- [Habilitar el complemento de enrutamiento de aplicación HTTP][aks-http-app-routing]
- [Crear un controlador de entrada que usa una red privada interna y una dirección IP][aks-ingress-internal]
- [Crear un controlador de entrada con una dirección IP pública dinámica y configurar Let's Encrypt para generar certificados TLS de forma automática][aks-ingress-tls]
- [Crear un controlador de entrada con una dirección IP pública estática y configurar Let's Encrypt para generar certificados TLS de forma automática][aks-ingress-static-tls]

### <a name="aks-engine"></a>Motor de AKS

[Azure Kubernetes Service Engine (AKS Engine)][aks-engine] es un proyecto de código abierto que genera plantillas de Azure Resource Manager que puede usar para implementar clústeres de Kubernetes en Azure.

Los clústeres de Kubernetes creados con AKS Engine admiten los complementos [kubenet][kubenet] y [Azure CNI][cni-networking]. Por lo tanto, ambos escenarios de redes son compatibles con AKS Engine.

<!-- IMAGES -->
[advanced-networking-diagram-01]: ./media/networking-overview/advanced-networking-diagram-01.png
[portal-01-networking-advanced]: ./media/networking-overview/portal-01-networking-advanced.png

<!-- LINKS - External -->
[aks-engine]: https://github.com/Azure/aks-engine
[services]: https://kubernetes.io/docs/concepts/services-networking/service/
[portal]: https://portal.azure.com
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet

<!-- LINKS - Internal -->
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[aks-ssh]: ssh.md
[ManagedClusterAgentPoolProfile]: /azure/templates/microsoft.containerservice/managedclusters#managedclusteragentpoolprofile-object
[aks-network-concepts]: concepts-network.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-internal]: ingress-internal-ip.md
[network-policy]: use-network-policies.md
