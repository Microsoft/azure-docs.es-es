---
title: 'Tutorial de Kubernetes en Azure: Implementación de un clúster'
description: En este tutorial de Azure Kubernetes Service (AKS), se crea un clúster de AKS y se usa kubectl para conectarse al nodo maestro de Kubernetes.
services: container-service
ms.topic: tutorial
ms.date: 02/25/2020
ms.custom: mvc
ms.openlocfilehash: 80393042191abc2a8eb74182cf18581d252222a5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87056469"
---
# <a name="tutorial-deploy-an-azure-kubernetes-service-aks-cluster"></a>Tutorial: Implementación de un clúster de Azure Kubernetes Service (AKS)

Kubernetes proporciona una plataforma distribuida para aplicaciones en contenedores. Con AKS, puede crear rápidamente un clúster de Kubernetes listo para producción. En este tutorial, la tercera parte de siete, se implementa un clúster de Kubernetes en AKS. Aprenderá a:

> [!div class="checklist"]
> * Implementación de un clúster de Kubernetes AKS que puede autenticarse en un registro de contenedor de Azure
> * Instalar la CLI de Kubernetes (kubectl)
> * Configurar kubectl para conectarse a un clúster de AKS

En tutoriales adicionales, la aplicación Azure Vote se implementa en el clúster, se escala y se actualiza.

## <a name="before-you-begin"></a>Antes de empezar

En los tutoriales anteriores, se creó una imagen de contenedor y se actualizó en una instancia de Azure Container Registry. Si no ha realizado estos pasos, y desea continuar, inicie el [Tutorial 1: Creación de imágenes de contenedor][aks-tutorial-prepare-app].

Para realizar este tutorial es necesario ejecutar la versión 2.0.53, o superior, de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

## <a name="create-a-kubernetes-cluster"></a>Creación de un clúster de Kubernetes

Los clústeres de AKS pueden usar los controles de acceso basado en roles (RBAC) de Kubernetes. Dichos controles permiten definir el acceso a los recursos en función de los roles asignados a los usuarios. Los permisos se combinan si a un usuario se le asignan varios roles y los permisos se pueden limitar a un espacio de nombres único o a todo el clúster. De forma predeterminada, la CLI de Azure habilita automáticamente RBAC al crear un clúster de AKS.

Cree un clúster de AKS mediante [az aks create][]. En el ejemplo siguiente se crea un clúster denominado *myAKSCluster* en el grupo de recursos denominado *myResourceGroup*. Este grupo de recursos se creó en el [tutorial anterior][aks-tutorial-prepare-acr] en la región *eastus*. En el ejemplo siguiente no se especifica una región, por lo que el clúster de AKS también se crea en la región *eastus*. Para más información sobre los límites de recursos y la disponibilidad de regiones para AKS, consulte [Cuotas, restricciones de tamaño de máquina virtual y disponibilidad de regiones en Azure Kubernetes Service (AKS)][quotas-skus-regions].

Para permitir que un clúster de AKS interactúe con otros recursos de Azure, se crea automáticamente una entidad de servicio de Azure Active Directory, dado que no se especificó ninguna. Aquí, a esta entidad de servicio se le [concede el derecho a extraer imágenes][container-registry-integration] de la instancia de Azure Container Registry (ACR) que creó en el tutorial anterior. Tenga en cuenta que puede usar una [identidad administrada](use-managed-identity.md) en lugar de una entidad de servicio para facilitar la administración.

```azurecli
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 2 \
    --generate-ssh-keys \
    --attach-acr <acrName>
```

También puede configurar manualmente una entidad de servicio para que extraiga imágenes de ACR. Para más información, consulte [Autenticación de ACR con entidades de servicio](../container-registry/container-registry-auth-service-principal.md) o [Autenticación desde Kubernetes con un secreto de extracción](../container-registry/container-registry-auth-kubernetes.md).

Al cabo de varios minutos, la implementación se completa y devuelve información en formato JSON sobre la implementación de AKS.

> [!NOTE]
> Para asegurarse de que el clúster funciona de forma confiable, debe ejecutar al menos 2 (dos) nodos.

## <a name="install-the-kubernetes-cli"></a>Instalación de la CLI de Kubernetes

Para conectarse al clúster de Kubernetes desde su equipo local, use [kubectl][kubectl], el cliente de la línea de comandos de Kubernetes.

Si usa Azure Cloud Shell, `kubectl` ya está instalado. También lo puede instalar localmente. Para ello debe usar el comando [az aks install-cli][]:

```azurecli
az aks install-cli
```

## <a name="connect-to-cluster-using-kubectl"></a>Conexión a un clúster mediante kubectl

Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials][]. En el ejemplo siguiente se obtienen las credenciales del clúster de AKS llamado *myAKSCluster* en el grupo de recursos *myResourceGroup*:

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Para comprobar la conexión al clúster, use ejecute el comando [kubectl get nodes][kubectl-get] para devolver una lista de los nodos del clúster:

```
$ kubectl get nodes

NAME                       STATUS   ROLES   AGE   VERSION
aks-nodepool1-12345678-0   Ready    agent   32m   v1.14.8
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se ha implementado un clúster de Kubernetes en AKS y se ha configurado `kubectl` para conectarse a él. Ha aprendido a:

> [!div class="checklist"]
> * Implementación de un clúster de Kubernetes AKS que puede autenticarse en un registro de contenedor de Azure
> * Instalar la CLI de Kubernetes (kubectl)
> * Configurar kubectl para conectarse a un clúster de AKS

Vaya al siguiente tutorial para aprender a implementar una aplicación en el clúster.

> [!div class="nextstepaction"]
> [Implementación de una aplicación en Kubernetes][aks-tutorial-deploy-app]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- LINKS - internal -->
[aks-tutorial-deploy-app]: ./tutorial-kubernetes-deploy-application.md
[aks-tutorial-prepare-acr]: ./tutorial-kubernetes-prepare-acr.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[az ad sp create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az acr show]: /cli/azure/acr#az-acr-show
[az role assignment create]: /cli/azure/role/assignment#az-role-assignment-create
[az aks create]: /cli/azure/aks#az-aks-create
[az aks install-cli]: /cli/azure/aks#az-aks-install-cli
[az aks get-credentials]: /cli/azure/aks#az-aks-get-credentials
[azure-cli-install]: /cli/azure/install-azure-cli
[container-registry-integration]: ./cluster-container-registry-integration.md
[quotas-skus-regions]: quotas-skus-regions.md
