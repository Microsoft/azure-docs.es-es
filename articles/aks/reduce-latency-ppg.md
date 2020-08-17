---
title: Uso de grupos con ubicación por proximidad para disminuir la latencia de los clústeres de Azure Kubernetes Service (AKS)
description: Aprenda a usar los grupos con ubicación por proximidad para disminuir la latencia de las cargas de trabajo de clústeres de AKS.
services: container-service
manager: gwallace
ms.topic: article
ms.date: 07/10/2020
author: jluk
ms.openlocfilehash: 5b3dc3803cfb89f4a74d082b5913e69df1d03a00
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87986719"
---
# <a name="reduce-latency-with-proximity-placement-groups-preview"></a>Reducción de la latencia con grupos con ubicación por proximidad (versión preliminar)

> [!Note]
> Cuando se usan los grupos con ubicación por proximidad en AKS, la coubicación solo se aplica a los nodos agente. Se mejora la latencia nodo a nodo y la latencia pod a pod hospedado correspondiente. La coubicación no afecta la selección de ubicación del plano de control de un clúster.

Al implementar la aplicación en Azure, la propagación de instancias de máquina virtual (VM) entre regiones o zonas de disponibilidad crea una latencia de red, lo que puede afectar al rendimiento general de la aplicación. Un grupo con ubicación por proximidad es una agrupación lógica que se usa para asegurarse de que los recursos de proceso de Azure se encuentran físicamente cercanos entre sí. Algunas aplicaciones como juegos, simulaciones de ingeniería y operaciones bursátiles de alta frecuencia (HFT) requieren una latencia baja y tareas que se completen rápidamente. En escenarios de informática de alto rendimiento (HPC), como estos, considere la posibilidad de usar [grupos con ubicación por proximidad](../virtual-machines/linux/co-location.md#proximity-placement-groups) (PPG) en los grupos de nodos del clúster.

## <a name="limitations"></a>Limitaciones

* Un grupo con ubicación por proximidad se puede asignar a una zona de disponibilidad como máximo.
* Un grupo de nodos debe usar Virtual Machine Scale Sets para asociar un grupo con ubicación por proximidad.
* Un grupo de nodos puede asociar un grupo con ubicación por proximidad solo en el momento de la creación del grupo de nodos.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="before-you-begin"></a>Antes de empezar

Debe tener instalados los siguientes recursos:

- La extensión aks-preview 0.4.53

### <a name="set-up-the-preview-feature-for-proximity-placement-groups"></a>Configuración de la característica en vista previa (GB) para los grupos con ubicación por proximidad

> [!IMPORTANT]
> Cuando se usan grupos con ubicación por proximidad con grupos de nodos de AKS, la coubicación solo se aplica a los nodos agente. Se mejora la latencia nodo a nodo y la latencia pod a pod hospedado correspondiente. La coubicación no afecta la selección de ubicación del plano de control de un clúster.

```azurecli-interactive
# register the preview feature
az feature register --namespace "Microsoft.ContainerService" --name "ProximityPlacementGroupPreview"
```

Puede tardar varios minutos en registrarse. Use el siguiente comando para comprobar que la característica está registrada:

```azurecli-interactive
# Verify the feature is registered:
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/ProximityPlacementGroupPreview')].{Name:name,State:properties.state}"
```

Durante la versión preliminar, necesita la extensión *aks-preview* de la CLI para usar los grupos con ubicación por proximidad. Use el comando [az extension add][az-extension-add] y, a continuación, busque las actualizaciones disponibles mediante el comando [az extension update][az-extension-update]:

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

## <a name="node-pools-and-proximity-placement-groups"></a>Grupos de nodos y grupos con ubicación por proximidad

El primer recurso que se implementa con un grupo con ubicación por proximidad se adjunta a un centro de datos específico. Los recursos adicionales implementados con el mismo grupo con ubicación por proximidad se colocan en el mismo centro de datos. Una vez que todos los recursos que usan el grupo con ubicación por proximidad se han detenido (desasignado) o eliminado, ya no se adjuntan.

* Muchos grupos de nodos pueden asociarse a un solo grupo con ubicación por proximidad.
* Un grupo de nodos solo se puede asociar a un solo grupo con ubicación por proximidad.

### <a name="configure-proximity-placement-groups-with-availability-zones"></a>Configuración de grupos con ubicación por proximidad con zonas de disponibilidad

> [!NOTE]
> Aunque los grupos con ubicación por proximidad necesitan un grupo de nodos para poder usar como máximo una zona de disponibilidad, el [Acuerdo de Nivel de Servicio de máquina virtual de Azure de línea de base del 99,9 %](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_9/) sigue vigente para las máquinas virtuales de una sola zona.

Los grupos de ubicación por proximidad son un concepto de grupo de nodos y están asociados a cada grupo de nodos individual. El uso de un recurso PPG no afecta a la disponibilidad del plano de control de AKS. Sí puede afectar al modo en que un clúster debe diseñarse con zonas. Para que un clúster se distribuya entre varias zonas, se recomienda el diseño siguiente.

* Aprovisione un clúster con el primer grupo del sistema usando tres zonas y sin grupo de ubicación por proximidad asociado. De esta forma, se garantiza que los pods del sistema aterrizan en un grupo de nodos dedicado que se distribuirá entre varias zonas.
* Agregue grupos de nodos de usuario adicionales con una única zona y un grupo de ubicación por proximidad asociado a cada grupo. Por ejemplo, nodepool1 en la zona 1 y PPG1, nodepool2 en la zona 2 y PPG2 o nodepool3 en la zona 3 con PPG3. Así, se tiene la seguridad de que, en el nivel de clúster, los nodos se reparten entre varias zonas y cada grupo de nodos individual se ubica en la zona designada con un recurso de PPG dedicado.

## <a name="create-a-new-aks-cluster-with-a-proximity-placement-group"></a>Creación de un clúster de AKS con un grupo con ubicación por proximidad

El ejemplo siguiente usa el comando [az group create][az-group-create] para crear un grupo de recursos denominado *myResourceGroup* en la región *centralus*. Se crea un clúster de AKS denominado *myAKSCluster* mediante el comando [az aks create][az-aks-create].

Las redes aceleradas mejoran en gran medida el rendimiento de red de las máquinas virtuales. Idealmente, use grupos con ubicación por proximidad junto con redes aceleradas. De forma predeterminada, AKS usa redes aceleradas en las [instancias de máquinas virtuales admitidas](../virtual-network/create-vm-accelerated-networking-cli.md?toc=/azure/virtual-machines/linux/toc.json#limitations-and-constraints), que incluyen la mayoría de las máquinas virtuales de Azure con dos o más CPU virtuales.

Cree un clúster de AKS con un grupo de ubicación por proximidad asociado al primer grupo de nodos del sistema:

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location centralus
```
Ejecute el comando siguiente y almacene el id. que se devuelve:

```azurecli-interactive
# Create proximity placement group
az ppg create -n myPPG -g myResourceGroup -l centralus -t standard
```

El comando genera una salida, que incluye el valor *id* que necesita para los comandos siguientes de la CLI:

```output
{
  "availabilitySets": null,
  "colocationStatus": null,
  "id": "/subscriptions/yourSubscriptionID/resourceGroups/myResourceGroup/providers/Microsoft.Compute/proximityPlacementGroups/myPPG",
  "location": "centralus",
  "name": "myPPG",
  "proximityPlacementGroupType": "Standard",
  "resourceGroup": "myResourceGroup",
  "tags": {},
  "type": "Microsoft.Compute/proximityPlacementGroups",
  "virtualMachineScaleSets": null,
  "virtualMachines": null
}
```

Use el id. de recurso del grupo con ubicación por proximidad para el valor *myPPGResourceID* en el comando siguiente:

```azurecli-interactive
# Create an AKS cluster that uses a proximity placement group for the initial system node pool only. The PPG has no effect on the cluster control plane.
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --ppg myPPGResourceID
```

## <a name="add-a-proximity-placement-group-to-an-existing-cluster"></a>Incorporación de un grupo con ubicación por proximidad a un clúster existente

Puede agregar un grupo con ubicación por proximidad a un clúster existente mediante la creación de un grupo de nodos nuevo. Si lo desea, puede migrar las cargas de trabajo existentes al grupo de nodos nuevo y, a continuación, eliminar el grupo de nodos original.

Use el mismo grupo con ubicación por proximidad que creó anteriormente y esto garantizará que los nodos de agente de ambos grupos de nodos del clúster de AKS estén físicamente ubicados en el mismo centro de datos.

Use el id. de recurso del grupo con ubicación por proximidad que creó anteriormente y agregue un grupo de nodos nuevo con el comando [`az aks nodepool add`][az-aks-nodepool-add]:

```azurecli-interactive
# Add a new node pool that uses a proximity placement group, use a --node-count = 1 for testing
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-count 1 \
    --ppg myPPGResourceID
```

## <a name="clean-up"></a>Limpieza

Para eliminar el clúster, use el comando [`az group delete`][az-group-delete] para eliminar el grupo de recursos de AKS:

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre los [grupos con ubicación por proximidad][proximity-placement-groups].

<!-- LINKS - Internal -->
[azure-ad-rbac]: azure-ad-rbac.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-get-upgrades]: /cli/azure/aks#az-aks-get-upgrades
[az-aks-upgrade]: /cli/azure/aks#az-aks-upgrade
[az-aks-show]: /cli/azure/aks#az-aks-show
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[proximity-placement-groups]: ../virtual-machines/linux/co-location.md#proximity-placement-groups
[az-aks-create]: /cli/azure/aks#az-aks-create
[system-pool]: ./use-system-pools.md
[az-aks-nodepool-add]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-add
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
