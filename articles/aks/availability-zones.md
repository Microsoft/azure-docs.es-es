---
title: Uso de las zonas de disponibilidad en Azure Kubernetes Service (AKS)
description: Aprenda a crear un clúster que distribuya nodos a través de las zonas de disponibilidad en Azure Kubernetes Service (AKS)
services: container-service
ms.custom: fasttrack-edit, references_regions
ms.topic: article
ms.date: 09/04/2020
ms.openlocfilehash: b6162249592bf470c3b8e52686abd44b813d5606
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89489143"
---
# <a name="create-an-azure-kubernetes-service-aks-cluster-that-uses-availability-zones"></a>Creación de un clúster de Azure Kubernetes Service (AKS) que use zonas de disponibilidad

Un clúster de Azure Kubernetes Service (AKS) distribuye recursos como nodos y almacenamiento en secciones lógicas de infraestructura subyacente de Azure. Este modelo de implementación con zonas de disponibilidad garantiza que los nodos de una zona de disponibilidad determinada estén físicamente separados de los definidos en otra zona de disponibilidad. Los clústeres de AKS implementados con varias zonas de compatibilidad configuradas en un clúster proporcionan un alto nivel de disponibilidad para proteger frente a errores de hardware o eventos de mantenimiento planeados.

Si se definen los grupos de nodos de un clúster de modo que abarquen varias zonas, los nodos de un grupo de nodos determinado pueden seguir funcionando aunque una zona esté fuera de servicio. Las aplicaciones pueden seguir estando disponibles aunque se produzca un error físico en un único centro de datos, siempre que este esté orquestado para tolerar errores de un subconjunto de nodos.

En este artículo le mostraremos cómo crear un clúster de AKS y distribuir los componentes del nodo en las zonas de disponibilidad.

## <a name="before-you-begin"></a>Antes de empezar

Es preciso que esté instalada y configurada la versión 2.0.76 de la CLI de Azure, o cualquier otra posterior. Ejecute  `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte  [Install Azure CLI][install-azure-cli] (Instalación de la CLI de Azure).

## <a name="limitations-and-region-availability"></a>Limitaciones y disponibilidad de región

Los clústeres de AKS se pueden crear actualmente mediante zonas de disponibilidad en las siguientes regiones:

* Este de Australia
* Centro de EE. UU.
* Este de EE. UU. 2
* Este de EE. UU.
* Centro de Francia
* Japón Oriental
* Norte de Europa
* Sudeste de Asia
* Sur de Reino Unido
* Oeste de Europa
* Oeste de EE. UU. 2

Las siguientes limitaciones se aplican cuando crea un clúster de AKS mediante zonas de disponibilidad:

* Solo se pueden definir zonas de disponibilidad cuando se crea el clúster o grupo de nodos.
* La configuración de la zona de disponibilidad no se puede actualizar después de crear el clúster. Tampoco puede actualizar un clúster de zona sin disponibilidad para usar zonas de disponibilidad.
* El tamaño de nodo (SKU de VM) seleccionado debe estar disponible en todas las zonas de disponibilidad seleccionadas.
* Los clústeres con zonas de disponibilidad habilitadas requieren el uso de equilibradores de carga estándar de Azure para la distribución entre zonas. Este tipo de equilibrador de carga solo se puede definir en el momento de la creación del clúster. Para obtener más información y las limitaciones del equilibrador de carga estándar, consulte las [limitaciones de la SKU estándar del equilibrador de carga de Azure][standard-lb-limitations].

### <a name="azure-disks-limitations"></a>Limitaciones de los discos de Azure

Los volúmenes que usan discos administrados de Azure actualmente no son recursos con redundancia de zona. Los volúmenes no se pueden conectar entre zonas y deben estar ubicados en la misma zona que el nodo concreto que hospeda al pod de destino.

Si debe ejecutar cargas de trabajo con estado, use valores taint y toleration de grupo de nodos en las especificaciones de pods para agrupar la programación de pods en la misma zona que los discos. Asimismo, puede usar el almacenamiento basado en la red, como Azure Files, que se puede conectar a los pods según se programan entre zonas.

## <a name="overview-of-availability-zones-for-aks-clusters"></a>Introducción a las zonas de disponibilidad para los clústeres de AKS

Las zonas de disponibilidad son una oferta de alta disponibilidad que protege las aplicaciones y datos de los errores del centro de datos. Las zonas son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, hay un mínimo de tres zonas independientes en todas las regiones habilitadas para zonas. La separación física de las zonas de disponibilidad dentro de una región protege las aplicaciones y los datos frente a los errores del centro de datos.

Para más información, consulte [¿Qué son las zonas de disponibilidad en Azure?][az-overview]

Los clústeres de AKS que se implementan mediante zonas de disponibilidad que pueden distribuir nodos en varias zonas dentro de una sola región. Por ejemplo, un clúster en la región  *Este de EE. UU. 2* puede crear nodos en las tres zonas de disponibilidad del *Este de EE. UU. 2*. Esta distribución de los recursos del clúster de AKS mejora la disponibilidad del clúster, ya que son resistentes a los errores de una zona específica.

![Distribución de nodos de AKS en zonas de disponibilidad](media/availability-zones/aks-availability-zones.png)

Si una sola zona deja de estar disponible, las aplicaciones siguen ejecutándose siempre que el clúster esté distribuido entre varias zonas.

## <a name="create-an-aks-cluster-across-availability-zones"></a>Crear un clúster de AKS en zonas de disponibilidad

Cuando crea un clúster con el comando [az aks create][az-aks-create], el parámetro `--zones` define en qué zonas se implementan los nodos de agente. Los componentes del plano de control como etcd se distribuyen en tres zonas si se define el parámetro `--zones` en el momento de la creación del clúster. Las zonas específicas entre las que se han distribuido los componentes del plano de control son independientes de las zonas explícitas que se han seleccionado para el grupo de nodos inicial.

Si no se define ninguna zona para el grupo de agentes predeterminado al crear un clúster de AKS, no se garantiza que los componentes del plano de control se distribuyan entre zonas de disponibilidad. Puede agregar grupos de nodos adicionales mediante el comando [az aks nodepool add][az-aks-nodepool-add] y especificar `--zones` para los nuevos nodos, pero esto no cambia la forma en que se ha distribuido el plano de control entre zonas. La configuración de zonas de disponibilidad solo se puede definir en el momento de la creación del clúster o el grupo de nodos.

En el ejemplo siguiente se crea un clúster de AKS denominado *myAKSCluster* en el grupo de recursos denominado *myResourceGroup*. Se crean un total de *3* nodos: un agente en la zona *1*, uno en la *2* y otro en la *3*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus2

az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --generate-ssh-keys \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --node-count 3 \
    --zones 1 2 3
```

El clúster de AKS tarda unos minutos en crearse.

A la hora de decidir la zona a la que debe pertenecer un nodo nuevo, un grupo de nodos determinado de AKS usa la [mejor opción de equilibrio de zonas ofrecida por la instancia subyacente de Azure Virtual Machine Scale Sets][vmss-zone-balancing]. Un grupo de nodos dado de AKS se considera "equilibrado" si cada zona tiene el mismo número de máquinas virtuales o +\- 1 máquina virtual en todas las demás zonas del conjunto de escalado.

## <a name="verify-node-distribution-across-zones"></a>Comprobar la distribución de nodos entre zonas

Cuando el clúster esté listo, enumere los nodos de agente en el conjunto de escalado para ver en qué zona de disponibilidad se implementarán.

Primero, obtenga las credenciales del clúster de AKS mediante el comando [az aks get-credentials][az-aks-get-credentials]:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

A continuación, use el comando [kubectl describe][kubectl-describe] para enumerar los nodos del clúster y filtrar por el valor de *failure-domain.beta.kubernetes.io/zone*. El ejemplo siguiente es para un shell de Bash.

```console
kubectl describe nodes | grep -e "Name:" -e "failure-domain.beta.kubernetes.io/zone"
```

El siguiente resultado de ejemplo muestra los tres nodos distribuidos en la región especificada y las zonas de disponibilidad, como *eastus2-1* para la primera zona de disponibilidad y *eastus2-2* para la segunda zona de disponibilidad:

```console
Name:       aks-nodepool1-28993262-vmss000000
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000001
            failure-domain.beta.kubernetes.io/zone=eastus2-2
Name:       aks-nodepool1-28993262-vmss000002
            failure-domain.beta.kubernetes.io/zone=eastus2-3
```

A medida que agrega nodos adicionales a un grupo de agentes, la plataforma Azure distribuye automáticamente las VM subyacentes en las zonas de disponibilidad especificadas.

Tenga en cuenta que en las versiones más recientes de Kubernetes (1.17.0 y versiones posteriores), AKS usa la etiqueta `topology.kubernetes.io/zone` más reciente además de la `failure-domain.beta.kubernetes.io/zone` en desuso.

## <a name="verify-pod-distribution-across-zones"></a>Comprobar la distribución de pods entre zonas

Como se documenta en [Etiquetas, anotaciones y taints ampliamente conocidas][kubectl-well_known_labels], Kubernetes usa la etiqueta `failure-domain.beta.kubernetes.io/zone` para distribuir automáticamente los pods en un controlador o servicio de replicación en las diferentes zonas disponibles. Para probarlo, puede escalar verticalmente el clúster de 3 a 5 nodos para comprobar la propagación correcta del pod:

```azurecli-interactive
az aks scale \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 5
```

Cuando la operación de escalado se complete después de unos minutos, el comando `kubectl describe nodes | grep -e "Name:" -e "failure-domain.beta.kubernetes.io/zone"` de un shell de Bash debe proporcionar una salida similar a la de este ejemplo:

```console
Name:       aks-nodepool1-28993262-vmss000000
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000001
            failure-domain.beta.kubernetes.io/zone=eastus2-2
Name:       aks-nodepool1-28993262-vmss000002
            failure-domain.beta.kubernetes.io/zone=eastus2-3
Name:       aks-nodepool1-28993262-vmss000003
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000004
            failure-domain.beta.kubernetes.io/zone=eastus2-2
```

Ahora hay dos nodos adicionales en las zonas 1 y 2. Puede implementar una aplicación que consta de tres réplicas. Se va a usar NGINX como ejemplo:

```console
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=3
```

Si se miran los nodos en los que se están ejecutando los pods, se observa que estos se ejecutan en los nodos correspondientes a tres zonas de disponibilidad diferentes. Por ejemplo, con el comando `kubectl describe pod | grep -e "^Name:" -e "^Node:"` de un shell de Bash se obtendría una salida similar a esta:

```console
Name:         nginx-6db489d4b7-ktdwg
Node:         aks-nodepool1-28993262-vmss000000/10.240.0.4
Name:         nginx-6db489d4b7-v7zvj
Node:         aks-nodepool1-28993262-vmss000002/10.240.0.6
Name:         nginx-6db489d4b7-xz6wj
Node:         aks-nodepool1-28993262-vmss000004/10.240.0.8
```

Como puede ver en la salida anterior, el primer pod se ejecuta en el nodo 0, que se encuentra en la zona de disponibilidad `eastus2-1`. El segundo pod se ejecuta en el nodo 2, que corresponde a `eastus2-3`, y el tercero, en el nodo 4, en `eastus2-2`. Sin necesidad de configuración adicional, Kubernetes extiende los pods correctamente en las tres zonas de disponibilidad.

## <a name="next-steps"></a>Pasos siguientes

En este artículo se detalla cómo crear un clúster de AKS que usa zonas de disponibilidad. Para obtener más información sobre los clústeres de alta disponibilidad, consulte [Best practices for business continuity and disaster recovery in AKS][best-practices-bc-dr] (Procedimientos recomendados para la continuidad empresarial y recuperación ante desastres en AKS).

<!-- LINKS - internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-overview]: ../availability-zones/az-overview.md
[best-practices-bc-dr]: operator-best-practices-multi-region.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[standard-lb-limitations]: load-balancer-standard.md#limitations
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-nodepool-add]: /cli/azure/ext/aks-preview/aks/nodepool#ext-aks-preview-az-aks-nodepool-add
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[vmss-zone-balancing]: ../virtual-machine-scale-sets/virtual-machine-scale-sets-use-availability-zones.md#zone-balancing

<!-- LINKS - external -->
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-well_known_labels]: https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/
