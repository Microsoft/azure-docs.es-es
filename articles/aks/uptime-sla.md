---
title: Azure Kubernetes Service (AKS) con acuerdo de nivel de servicio de tiempo de actividad
description: Obtenga información sobre la oferta opcional de acuerdo de nivel de servicio de tiempo de actividad para el servidor de API de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 06/24/2020
ms.custom: references_regions
ms.openlocfilehash: 6523e16bfe3bf0592b78da544d7d52dc3d969af4
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88749121"
---
# <a name="azure-kubernetes-service-aks-uptime-sla"></a>Acuerdo de nivel de servicio de tiempo de actividad de Azure Kubernetes Service (AKS)

El acuerdo de nivel de servicio (SLA) de tiempo de actividad es una característica opcional para habilitar un mejor acuerdo de nivel de servicio con respaldo financiero para un clúster. El acuerdo de nivel de servicio de tiempo de actividad garantiza una disponibilidad del 99,95 % del punto de conexión del servidor de API de Kubernetes para los clústeres que usan [zonas de disponibilidad][availability-zones], y una disponibilidad del 99,9 % de los clústeres que no utilizan zonas de disponibilidad. AKS usa réplicas de nodo maestro en los dominios de actualización y de error para garantizar que se cumplan los requisitos del acuerdo de nivel de servicio.

Los clientes que necesiten un acuerdo de nivel de servicio para alcanzar los requisitos de cumplimiento o que requieren extender un acuerdo de nivel de servicio a sus usuarios finales deben habilitar esta característica. Los clientes con cargas de trabajo críticas que se beneficien de un mayor acuerdo de nivel de servicio de tiempo de actividad también pueden beneficiarse. El uso de la característica de acuerdo de nivel de servicio de tiempo de actividad con zonas de disponibilidad permite una mayor disponibilidad para el tiempo de actividad del servidor de API de Kubernetes.  

Los clientes todavía pueden crear un número ilimitado de clústeres gratuitos con un objetivo de nivel de servicio (SLO) del 99,5 %, y optar por el tiempo de actividad preferido del SLO o SLA según sea necesario.

> [!Important]
> En el caso de los clústeres con bloqueo de salida, consulte la [limitación del tráfico de salida](limit-egress-traffic.md) para abrir los puertos adecuados.

## <a name="region-availability"></a>Disponibilidad en regiones

El Acuerdo de Nivel de Servicio de tiempo de actividad está disponible en las regiones públicas y de Azure Government en las que [se admite AKS](https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service).

* Azure China 21Vianet no se admite actualmente.

## <a name="limitations"></a>Limitaciones

* Por el momento no se admiten los clústeres privados.

## <a name="sla-terms-and-conditions"></a>Términos y condiciones del acuerdo de nivel de servicio

El acuerdo de nivel de servicio de tiempo de actividad es una característica de pago y se habilita para cada clúster. El precio del acuerdo de nivel de servicio de tiempo de actividad viene determinado por el número de clústeres discretos, no por el tamaño de los clústeres individuales. Puede ver los [Detalles de precios de los acuerdos de nivel de servicio de tiempo de actividad](https://azure.microsoft.com/pricing/details/kubernetes-service/) para obtener más información.

## <a name="before-you-begin"></a>Antes de empezar

* Instale la versión 2.8.0 de la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest), o cualquier versión posterior.

## <a name="creating-a-new-cluster-with-uptime-sla"></a>Creación de un clúster con un Acuerdo de Nivel de Servicio de tiempo de actividad

> [!NOTE]
> Actualmente, si habilita el Acuerdo de Nivel de Servicio de tiempo de actividad, no hay forma de quitarlo de un clúster.

Para crear un nuevo clúster con el acuerdo de nivel de servicio de tiempo de actividad, use la CLI de Azure.

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
# Create a resource group
az group create --name myResourceGroup --location eastus
```
Use el comando [`az aks create`][az-aks-create] para crear un clúster de AKS. En el siguiente ejemplo se crea un clúster denominado *myAKSCluster* con un nodo. La operación tarda varios minutos en completarse:

```azurecli-interactive
# Create an AKS cluster with uptime SLA
az aks create --resource-group myResourceGroup --name myAKSCluster --uptime-sla --node-count 1
```
Transcurridos unos minutos, el comando se completa y devuelve información en formato JSON sobre el clúster. El siguiente fragmento JSON muestra el nivel de pago de la SKU, que indica que el clúster tiene habilitado el Acuerdo de Nivel de Servicio de tiempo de actividad.

```output
  },
  "sku": {
    "name": "Basic",
    "tier": "Paid"
  },
```

## <a name="modify-an-existing-cluster-to-use-uptime-sla"></a>Modificación de un clúster existente para usar un Acuerdo de Nivel de Servicio de tiempo de actividad

Opcionalmente, puede actualizar los clústeres existentes para usar el Acuerdo de Nivel de Servicio de tiempo de actividad.

Si creó un clúster de AKS con los pasos anteriores, elimine el grupo de recursos:

```azurecli-interactive
# Delete the existing cluster by deleting the resource group 
az group delete --name myResourceGroup --yes --no-wait
```

Cree un nuevo grupo de recursos:

```azurecli-interactive
# Create a resource group
az group create --name myResourceGroup --location eastus
```

Cree un nuevo clúster y no use el Acuerdo de Nivel de Servicio de tiempo de actividad:

```azurecli-interactive
# Create a new cluster without uptime SLA
az aks create --resource-group myResourceGroup --name myAKSCluster--node-count 1
```

Use el comando [`az aks update`][az-aks-nodepool-update] para actualizar el clúster existente:

```azurecli-interactive
# Update an existing cluster to use Uptime SLA
 az aks update --resource-group myResourceGroup --name myAKSCluster --uptime-sla
 ```

 El siguiente fragmento JSON muestra el nivel de pago de la SKU, que indica que el clúster tiene habilitado el Acuerdo de Nivel de Servicio de tiempo de actividad.

 ```output
  },
  "sku": {
    "name": "Basic",
    "tier": "Paid"
  },
  ```

## <a name="clean-up"></a>Limpieza

Para evitar cargos, limpie los recursos que haya creado. Para eliminar el clúster, use el comando [`az group delete`][az-group-delete] para eliminar el grupo de recursos de AKS:

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```


## <a name="next-steps"></a>Pasos siguientes

Use [Availability Zones][availability-zones] para aumentar la alta disponibilidad con las cargas de trabajo del clúster de AKS.

Configure el clúster para [limitar el tráfico de salida](limit-egress-traffic.md).

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
[region-availability]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service

<!-- LINKS - Internal -->
[vm-skus]: ../virtual-machines/sizes.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[faq]: ./faq.md
[availability-zones]: ./availability-zones.md
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[limit-egress-traffic]: ./limit-egress-traffic.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-nodepool-update]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-update
[az-group-delete]: /cli/azure/group#az-group-delete
