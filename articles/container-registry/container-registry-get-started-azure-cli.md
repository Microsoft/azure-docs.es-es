---
title: 'Inicio rápido: Creación de un registro (CLI de Azure)'
description: Aprenda rápidamente a crear un registro de contenedor privado de Docker con la CLI de Azure.
ms.topic: quickstart
ms.date: 06/12/2020
ms.custom: seodec18, H1Hack27Feb2017, mvc, devx-track-azurecli
ms.openlocfilehash: f4e69616d30c6a7b853c5cc854adee147ebde206
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87486551"
---
# <a name="quickstart-create-a-private-container-registry-using-the-azure-cli"></a>Inicio rápido: Creación de un registro de contenedor privado con la CLI de Azure

Azure Container Registry es un servicio de registro de contenedores de Docker administrado usado para almacenar imágenes de contenedor de Docker privadas. En esta guía se detalla la creación de una instancia de Azure Container Registry mediante la CLI de Azure. A continuación, utilice los comandos de Docker para insertar una imagen de contenedor en el registro y, finalmente, extraiga y ejecute la imagen desde el registro.

En este inicio rápido es preciso que ejecute la CLI de Azure (se recomienda la versión 2.0.55 u otra posterior). Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli].

También debe tener instalado Docker localmente. Docker proporciona paquetes que permiten configurar Docker fácilmente en cualquier sistema [macOS][docker-mac], [Windows][docker-windows] o [Linux][docker-linux].

Dado que Azure Cloud Shell no incluye todos los componentes necesarios de Docker (como por ejemplo el demonio `dockerd`), no se puede usar Cloud Shell en este tutorial de inicio rápido.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create][az-group-create]. Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurecli
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-container-registry"></a>Creación de un Registro de contenedor

En este inicio rápido se crea un registro *Básico*, que es una opción rentable para los desarrolladores que aprenden sobre Azure Container Registry. Para más información sobre los niveles de servicio disponibles, consulte [SKU de Azure Container Registry][container-registry-skus].

Creación de una instancia de ACR mediante el comando [az acr create][az-acr-create]. El nombre del registro debe ser único dentro de Azure y contener entre 5 y 50 caracteres alfanuméricos. En el ejemplo siguiente, se usa *myContainerRegistry007*. Actualice este valor a uno único.

```azurecli
az acr create --resource-group myResourceGroup \
  --name myContainerRegistry007 --sku Basic
```

Cuando se crea el registro, el resultado es similar al siguiente:

```json
{
  "adminUserEnabled": false,
  "creationDate": "2019-01-08T22:32:13.175925+00:00",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myContainerRegistry007",
  "location": "eastus",
  "loginServer": "mycontainerregistry007.azurecr.io",
  "name": "myContainerRegistry007",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

Tome nota de `loginServer` en la salida, que es el nombre de registro completo (todo en minúsculas). En el resto de este inicio rápido, `<registry-name>` es el marcador de posición del nombre del registro de contenedor y `<login-server>` es el marcador de posición del nombre del servidor de inicio de sesión del registro.

## <a name="log-in-to-registry"></a>Iniciar sesión en el registro

Antes de insertar y extraer imágenes de contenedor, debe iniciar sesión en el registro. Para ello, utilice el comando [az acr login][az-acr-login].

```azurecli
az acr login --name <registry-name>
```

El comando devolverá un mensaje `Login Succeeded` una vez completado.

[!INCLUDE [container-registry-quickstart-docker-push](../../includes/container-registry-quickstart-docker-push.md)]

## <a name="list-container-images"></a>Lista de imágenes de contenedor

En el ejemplo siguiente se enumeran los repositorios del registro:

```azurecli
az acr repository list --name <registry-name> --output table
```

Salida:

```
Result
----------------
hello-world
```

En el ejemplo siguiente se muestran las etiquetas del repositorio **hello-world**.

```azurecli
az acr repository show-tags --name <registry-name> --repository hello-world --output table
```

Salida:

```
Result
--------
v1
```

[!INCLUDE [container-registry-quickstart-docker-pull](../../includes/container-registry-quickstart-docker-pull.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar el comando [az group delete][az-group-delete] para quitar el grupo de recursos, el registro de contenedor y las imágenes de contenedor almacenadas allí.

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una instancia de Azure Container Registry con la CLI de Azure, ha insertado una imagen de contenedor en el registro y ha extraído y ejecutado la imagen del registro. Siga los tutoriales de Azure Container Registry para información más detallada de ACR.

> [!div class="nextstepaction"]
> [Tutoriales de Azure Container Registry][container-registry-tutorial-prepare-registry]

> [!div class="nextstepaction"]
> [Tutoriales de Azure Container Registry Tasks][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli]: /cli/azure/install-azure-cli
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
[container-registry-skus]: container-registry-skus.md
[container-registry-tutorial-prepare-registry]: container-registry-tutorial-prepare-registry.md
