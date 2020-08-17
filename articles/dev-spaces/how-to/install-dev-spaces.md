---
title: Habilitación de Azure Dev Spaces en AKS e instalación de las herramientas de cliente
services: azure-dev-spaces
ms.date: 07/24/2019
ms.topic: conceptual
description: Obtenga información sobre el proceso de habilitación de Azure Dev Spaces en un clúster de AKS y la instalación de las herramientas de cliente.
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores, Helm, service mesh, enrutamiento de service mesh, kubectl, k8s
ms.openlocfilehash: ac010a466f7db7b829cc3d6f0687dbdbefdd7b6c
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407904"
---
# <a name="enable-azure-dev-spaces-on-an-aks-cluster-and-install-the-client-side-tools"></a>Habilitación de Azure Dev Spaces en un clúster de AKS e instalación de las herramientas de cliente

En este artículo se muestran varias maneras de habilitar Azure Dev Spaces en un clúster de AKS, así como de instalar las herramientas de cliente.

## <a name="enable-azure-dev-spaces-using-the-cli"></a>Habilitación de Azure Dev Spaces con la CLI

Para poder habilitar Dev Spaces mediante la CLI, necesitará lo siguiente:
* Suscripción a Azure. Si no tiene una suscripción a Azure, puede crear una [cuenta gratuita][az-portal-create-account].
* [La CLI de Azure instalada][install-cli].
* [Un clúster de AKS][create-aks-cli] en una [región admitida][supported-regions].

Use el comando `use-dev-spaces` para habilitar Dev Spaces en el clúster de AKS y siga las indicaciones.

```azurecli
az aks use-dev-spaces -g myResourceGroup -n myAKSCluster
```

El comando anterior habilita Dev Spaces en el clúster *myAKSCluster* del grupo *myResourceGroup* y crea un espacio de desarrollo *predeterminado*.

```console
'An Azure Dev Spaces Controller' will be created that targets resource 'myAKSCluster' in resource group 'myResourceGroup'. Continue? (y/N): y

Creating and selecting Azure Dev Spaces Controller 'myAKSCluster' in resource group 'myResourceGroup' that targets resource 'myAKSCluster' in resource group 'myResourceGroup'...2m 24s

Select a dev space or Kubernetes namespace to use as a dev space.
 [1] default
Type a number or a new name: 1

Kubernetes namespace 'default' will be configured as a dev space. This will enable Azure Dev Spaces instrumentation for new workloads in the namespace. Continue? (Y/n): Y

Configuring and selecting dev space 'default'...3s

Managed Kubernetes cluster 'myAKSCluster' in resource group 'myResourceGroup' is ready for development in dev space 'default'. Type `azds prep` to prepare a source directory for use with Azure Dev Spaces and `azds up` to run.
```

El comando `use-dev-spaces` también instala la CLI de Azure Dev Spaces.

## <a name="install-the-client-side-tools"></a>Instalación de las herramientas de cliente

Puede usar las herramientas de cliente de Azure Dev Spaces para interactuar con Dev Spaces en un clúster de AKS desde la máquina local. Hay varias maneras de instalar las herramientas de cliente:

* En [Visual Studio Code][vscode], instale la [extensión de Azure Dev Spaces][vscode-extension].
* En [Visual Studio 2019][visual-studio], instale la carga de trabajo de desarrollo de Azure.
* Descargue e instale la CLI de [Windows][cli-win], [Mac][cli-mac] o [Linux][cli-linux].

## <a name="remove-azure-dev-spaces-using-the-cli"></a>Eliminación de Azure Dev Spaces con la CLI

Para quitar Azure Dev Spaces del clúster de AKS, use el comando `azds remove`.

```azurecli
azds remove -g MyResourceGroup -n MyAKS
```

La salida del ejemplo siguiente muestra cómo quitar Azure Dev Spaces del clúster de *MyAKS*.

```azurecli
$ azds remove -g MyResourceGroup -n MyAKS
Azure Dev Spaces Controller 'MyAKS' in resource group 'MyResourceGroup' that targets resource 'MyAKS' in resource group 'MyResourceGroup' will be deleted. This will remove Azure Dev Spaces instrumentation from the target resource for new workloads. Continue? (y/N): y

Deleting Azure Dev Spaces Controller 'MyAKS' in resource group 'MyResourceGroup' that targets resource 'MyAks' in resource group 'MyResourceGroup' (takes a few minutes)...
```

Los espacios de nombres creados con Azure Dev Spaces se conservan junto con sus cargas de trabajo, pero las nuevas cargas de trabajo de esos espacios de nombres no se instrumentan con Azure Dev Spaces. Además, si reinicia los pods existentes instrumentados con Azure Dev Spaces, pueden aparecer errores. Esos pods deben volver a implementarse sin herramientas de Azure Dev Spaces. Para quitar Azure Dev Spaces por completo del clúster, elimine todos los pods de todos los espacios de nombres en los que se haya habilitado Azure Dev Spaces.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de la forma en que Azure Dev Spaces le ayuda a desarrollar aplicaciones más complejas en varios contenedores y cómo puede simplificar el desarrollo en colaboración mediante el uso de distintas versiones o bifurcaciones del código en distintos espacios.

> [!div class="nextstepaction"]
> [Desarrollo en equipo en Azure Dev Spaces][team-development-qs]

[create-aks-cli]: ../../aks/kubernetes-walkthrough.md#create-a-resource-group
[install-cli]: /cli/azure/install-azure-cli?view=azure-cli-latest
[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[team-development-qs]: ../quickstart-team-development.md

[az-portal]: https://portal.azure.com
[az-portal-create-account]: https://azure.microsoft.com/free
[cli-linux]: https://aka.ms/get-azds-linux
[cli-mac]: https://aka.ms/get-azds-mac
[cli-win]: https://aka.ms/get-azds-windows
[visual-studio]: https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs
[visual-studio-k8s-tools]: https://aka.ms/get-vsk8stools
[vscode]: https://code.visualstudio.com/download
[vscode-extension]: https://marketplace.visualstudio.com/items?itemName=azuredevspaces.azds
