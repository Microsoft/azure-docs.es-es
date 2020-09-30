---
title: Inicio y detención de Azure Kubernetes Service (AKS)
description: Aprenda a iniciar y detener un clúster de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 09/18/2020
author: palma21
ms.openlocfilehash: a743a6c30d5ce8bcaf275bf1a658f8343de4d4fb
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931995"
---
# <a name="stop-and-start-an-azure-kubernetes-service-aks-cluster-preview"></a>Inicio y detención de un clúster de Azure Kubernetes Service (AKS) (versión preliminar)

Es posible que las cargas de trabajo de AKS no tengan que ejecutarse continuamente, por ejemplo, en el caso de un clúster de desarrollo que se use solo durante el horario comercial. Esto se traduce en momentos en que el clúster de Azure Kubernetes Service (AKS) podría estar inactivo, sin ejecutar más que los componentes del sistema. Para reducir la superficie del clúster, [escale todos los grupos de nodos `User` a 0](scale-cluster.md#scale-user-node-pools-to-0), aunque sigue siendo necesario que el [grupo `System`](use-system-pools.md) ejecute los componentes del sistema mientras el clúster está en ejecución. Para optimizar aún más los costos durante estos períodos, puede desactivar por completo (detener) el clúster. Esta acción detiene el plano de control y los nodos del agente, lo que permite ahorrar en todos los costos de proceso, a la vez que se mantienen todos los objetos y el estado del clúster almacenados para cuando se inicie de nuevo. Esto permite continuar justo donde se ha dejado después de un fin de semana o que el clúster se ejecute solo mientras se ejecutan los trabajos por lotes.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

### <a name="install-the-aks-preview-azure-cli"></a>Instalación de la CLI de Azure `aks-preview` 

También se necesita la versión 0.4.64 o posterior de la extensión de la CLI de Azure *aks-preview*. Instale la extensión de la CLI de Azure *aks-preview* mediante el comando [az extension add][az-extension-add]. También puede instalar las actualizaciones disponibles mediante el comando [az extension update][az-extension-update].

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
``` 

> [!WARNING]
> El estado de clúster de un clúster de AKS detenido se conserva durante un máximo de 12 meses. Si el clúster se detiene durante más de 12 meses, no se puede recuperar su estado. Para obtener más información, vea [Directivas de soporte técnico para AKS](support-policies.md).
> Solo puede iniciar o eliminar un clúster de AKS detenido. Para realizar cualquier operación, como escalado o actualización, primero inicie el clúster.


### <a name="register-the-startstoppreview-preview-feature"></a>Registro de la característica en vista previa (GB) `StartStopPreview`

Para usar la característica de inicio o detención de un clúster, debe habilitar la marca de característica `StartStopPreview` en la suscripción.

Registre la marca de característica `StartStopPreview` con el comando [az feature register][az-feature-register], como se muestra en el siguiente ejemplo:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "StartStopPreview"
```

Tarda unos minutos en que el estado muestre *Registrado*. Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/StartStopPreview')].{Name:name,State:properties.state}"
```

Cuando haya terminado, actualice el registro del proveedor de recursos *Microsoft.ContainerService* con el comando [az provider register][az-provider-register]:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## <a name="stop-an-aks-cluster"></a>Detención de un clúster de AKS

Puede usar el comando `az aks stop` para detener los nodos y el plano de control de un clúster de AKS en ejecución. En el siguiente ejemplo se detiene un clúster denominado *myAKSCluster*:

```azurecli-interactive
az aks stop --name myAKSCluster --resource-group myResourceGroup
```

Para comprobar que el clúster se ha detenido, use el comando [az aks show][az-aks-show] y confirme que `powerState` aparece como `Stopped`, como en la siguiente salida:

```json
{
[...]
  "nodeResourceGroup": "MC_myResourceGroup_myAKSCluster_westus2",
  "powerState":{
    "code":"Stopped"
  },
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
[...]
}
```

Si `provisioningState` muestra `Stopping`, significa que el clúster aún no se ha detenido por completo.

> [!IMPORTANT]
> Si usa [presupuestos de interrupciones de pods](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/), la operación de detención puede tardar más tiempo, ya que el proceso de purga tarda más en finalizar.


## <a name="start-an-aks-cluster"></a>Inicio de un clúster de AKS

Puede usar el comando `az aks start` para iniciar los nodos y el plano de control de un clúster de AKS detenido. El clúster se reinicia con el estado del plano de control y el número de nodos de agente anteriores.  
En el siguiente ejemplo se inicia un clúster denominado *myAKSCluster*:

```azurecli-interactive
az aks start --name myAKSCluster --resource-group myResourceGroup
```

Para comprobar que el clúster se ha iniciado, use el comando [az aks show][az-aks-show] y confirme que `powerState` aparece como `Running`, como en la siguiente salida:

```json
{
[...]
  "nodeResourceGroup": "MC_myResourceGroup_myAKSCluster_westus2",
  "powerState":{
    "code":"Running"
  },
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
[...]
}
```

Si `provisioningState` muestra `Starting`, significa que el clúster aún no se ha iniciado por completo.


## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre cómo escalar grupos `User` a 0, vea [Escalado de grupos `User` a 0](scale-cluster.md#scale-user-node-pools-to-0).
- Para obtener información sobre cómo ahorrar costos mediante instancias de acceso puntual, vea [Incorporación de un grupo de nodos de acceso puntual a AKS](spot-node-pool.md).
- Para obtener más información sobre las directivas de soporte técnico de AKS, vea [Directivas de soporte técnico de AKS](support-policies.md).

<!-- LINKS - external -->

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli&preserve-view=true
[az-extension-add]: /cli/azure/extension?view=azure-cli-latest#az-extension-add&preserve-view=true
[az-extension-update]: /cli/azure/extension?view=azure-cli-latest#az-extension-update&preserve-view=true
[az-feature-register]: /cli/azure/feature?view=azure-cli-latest#az-feature-register&preserve-view=true
[az-feature-list]: /cli/azure/feature?view=azure-cli-latest#az-feature-list&preserve-view=true
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest#az-provider-register&preserve-view=true