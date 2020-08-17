---
title: Habilitar el complemento Controlador de entrada para un nuevo clúster de AKS con una nueva instancia de Azure Application Gateway
description: Aprenda a usar la CLI de Azure para habilitar el complemento Controlador de entrada para el nuevo clúster de AKS con una nueva instancia de Application Gateway.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 06/10/2020
ms.author: caya
ms.openlocfilehash: 4634421829cf71c0c5b9476f8ff3d08b9caa7dbd
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87117340"
---
# <a name="tutorial-enable-the-ingress-controller-add-on-preview-for-a-new-aks-cluster-with-a-new-application-gateway-instance"></a>Tutorial: Habilitar el complemento Controlador de entrada (versión preliminar) para un nuevo clúster de AKS con una nueva instancia de Application Gateway

Puede usar la CLI de Azure para habilitar el complemento [Controlador de entrada de Application Gateway (AGIC)](ingress-controller-overview.md) para un clúster de [Azure Kubernetes Services (AKS)](https://azure.microsoft.com/services/kubernetes-service/). El complemento está actualmente en versión preliminar.

En este tutorial, creará un clúster de AKS con el complemento AGIC habilitado. Al crear el clúster, se creará automáticamente una instancia de Azure Application Gateway que se va a usar. Luego, implementará una aplicación de ejemplo que usará el complemento para exponer la aplicación a través de Application Gateway. 

El complemento proporciona una manera mucho más rápida de implementar AGIC para el clúster de AKS que [previamente a través de Helm](ingress-controller-overview.md#difference-between-helm-deployment-and-aks-add-on). También ofrece una experiencia totalmente administrada.    

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Cree un grupo de recursos. 
> * Crear un nuevo clúster de AKS con el complemento AGIC habilitado. 
> * Implementar una aplicación de ejemplo con AGIC para la entrada en el clúster de AKS.
> * Comprobar que la aplicación es accesible a través de Application Gateway.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Si decide instalar y usar la CLI de forma local, en este tutorial es necesario que ejecute la CLI de Azure versión 2.0.4 o posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Registre la marca de características de *AKS-IngressApplicationGatewayAddon* con el comando [az feature register](https://docs.microsoft.com/cli/azure/feature#az-feature-register), tal y como se muestra en el ejemplo siguiente. Solo tendrá que hacer esto una vez por cada suscripción mientras el complemento aún esté en versión preliminar.
```azurecli-interactive
az feature register --name AKS-IngressApplicationGatewayAddon --namespace Microsoft.ContainerService
```

El estado puede tardar unos minutos en mostrar `Registered`. Puede comprobar el estado del registro con el comando [az feature list](https://docs.microsoft.com/cli/azure/feature#az-feature-register):
```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AKS-IngressApplicationGatewayAddon')].{Name:name,State:properties.state}"
```

Cuando esté listo, actualice el registro del proveedor de recursos Microsoft.ContainerService con el comando [az provider register](https://docs.microsoft.com/cli/azure/provider#az-provider-register):
```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

Instale o actualice la extensión aks-preview para este tutorial. Use los siguientes comandos de la CLI de Azure:
```azurecli-interactive
az extension add --name aks-preview
az extension list
```
```azurecli-interactive
az extension update --name aks-preview
az extension list
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

En Azure, puede asignar recursos relacionados a un grupo de recursos. Para crear un grupo de recursos, use [az group create](/cli/azure/group#az-group-create). En el ejemplo siguiente se crea un grupo de recursos denominado *myResourceGroup* en la ubicación de *canadacentral* (región): 

```azurecli-interactive
az group create --name myResourceGroup --location canadacentral
```

## <a name="deploy-an-aks-cluster-with-the-add-on-enabled"></a>Implementar un clúster de AKS con el complemento habilitado

Ahora implementará un nuevo clúster de AKS con el complemento AGIC habilitado. Si no proporciona una instancia de Application Gateway existente para usarla en este proceso, crearemos y configuraremos automáticamente una nueva instancia de Application Gateway para servir el tráfico al clúster de AKS.  

> [!NOTE]
> El complemento Controlador de entrada de Application Gateway admite *solo* SKU de Application Gateway V2 (Standard y WAF) y *no* las SKU de Application Gateway v1. Al implementar una nueva instancia de Application Gateway a través del complemento AGIC, solo puede implementar una SKU de Application Gateway Standard_v2. Si desea habilitar el complemento para una SKU de Application Gateway WAF_v2, use cualquiera de estos métodos:
>
> - Habilite WAF en Application Gateway a través del portal. 
> - Cree la instancia del WAF_v2 de Application Gateway primero y luego, siga las instrucciones sobre cómo [habilitar el complemento de AGIC con un clúster de AKS existente y la instancia de Application Gateway existente](tutorial-ingress-controller-add-on-existing.md). 

En el ejemplo siguiente, implementará un nuevo clúster de AKS denominado *myCluster* con [Azure CNI](https://docs.microsoft.com/azure/aks/concepts-network#azure-cni-advanced-networking) e [identidades administradas](https://docs.microsoft.com/azure/aks/use-managed-identity). El complemento AGIC se habilitará en el grupo de recursos que creó, *myResourceGroup*. 

La implementación de un nuevo clúster de AKS con el complemento de AGIC habilitado sin especificar una instancia de Application Gateway existente hará referencia a la creación automática de una SKU de Standard_v2 de Application Gateway. Por lo tanto, también especificará el nombre y el espacio de direcciones de subred de la instancia de Application Gateway. El nombre de la instancia de Application Gateway será *myApplicationGateway* y el espacio de direcciones de subred que usamos es 10.2.0.0/16. Asegúrese de que agregó o actualizó la extensión aks-preview al principio de este tutorial. 

```azurecli-interactive
az aks create -n myCluster -g myResourceGroup --network-plugin azure --enable-managed-identity -a ingress-appgw --appgw-name myApplicationGateway --appgw-subnet-prefix "10.2.0.0/16" 
```

Para configurar parámetros adicionales para el comando `az aks create`, vea [estas referencias](https://docs.microsoft.com/cli/azure/aks?view=azure-cli-latest#az-aks-create). 

> [!NOTE]
> El clúster de AKS que creó aparecerá en el grupo de recursos que creó, *myResourceGroup*. Pero la instancia de Application Gateway creada automáticamente estará en el grupo de recursos del nodo, donde se encuentran los grupos de agentes. El grupo de recursos de nodo se denomina *MC_resource-group-name_cluster-name_location* de manera predeterminada, pero se puede modificar. 

## <a name="deploy-a-sample-application-by-using-agic"></a>Implementar una aplicación de ejemplo con AGIC

Ahora implementará una aplicación de ejemplo en el clúster de AKS que creó. La aplicación usará el complemento AGIC para la entrada y conectará la instancia de Application Gateway al clúster de AKS. 

En primer lugar, obtenga las credenciales para el clúster de AKS con la ejecución del comando `az aks get-credentials`: 

```azurecli-interactive
az aks get-credentials -n myCluster -g myResourceGroup
```

Ahora que tiene las credenciales, ejecute el siguiente comando para configurar una aplicación de ejemplo que use AGIC para la entrada al clúster. AGIC actualizará la instancia de Application Gateway que configuró anteriormente con las reglas de enrutamiento correspondientes a la nueva aplicación de ejemplo que implementó.  

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml 
```

## <a name="check-that-the-application-is-reachable"></a>Comprobar que la aplicación es accesible

Ahora que la instancia de Application Gateway está configurada para atender el tráfico al clúster de AKS, vamos a comprobar que la aplicación es accesible. En primer lugar, obtenga la dirección IP de la entrada: 

```azurecli-interactive
kubectl get ingress
```

Para comprobar que la aplicación de ejemplo que creó se ejecute, haga lo siguiente:

- Visite la dirección IP de la instancia de Application Gateway que tenía al ejecutar el comando anterior.
- Use `curl`. 

Application Gateway puede tardar un minuto en obtener la actualización. Si Application Gateway sigue en el estado **Actualizando** en el portal, deje que termine antes de intentar llegar a la dirección IP. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, quite el grupo de recursos, la instancia de Application Gateway y todos los recursos relacionados:

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes
* [Obtener información sobre cómo deshabilitar el complemento AGIC](./ingress-controller-disable-addon.md)
* [Más información sobre las anotaciones que se admiten con AGIC](./ingress-controller-annotations.md)
* [Solución de problemas con AGIC](./ingress-controller-troubleshoot.md)

