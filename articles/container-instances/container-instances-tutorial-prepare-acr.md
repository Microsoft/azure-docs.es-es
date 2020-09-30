---
title: 'Tutorial: Preparación del registro de contenedor para implementar una imagen'
description: 'Tutorial de Azure Container Instances, parte 2 de 3: Preparación de una instancia de Azure Container Registry e inserción de una imagen'
ms.topic: tutorial
ms.date: 12/18/2019
ms.custom: seodec18, mvc
ms.openlocfilehash: 44a7f21c067897b046413851ef5a2c73bfccc24f
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90708043"
---
# <a name="tutorial-create-an-azure-container-registry-and-push-a-container-image"></a>Tutorial: Creación de una instancia de Azure Container Registry e inserción de una imagen de contenedor

Esta es la segunda parte de un tutorial de tres partes. En la [primera parte](container-instances-tutorial-prepare-app.md) del tutorial se ha creado una imagen de contenedor de Docker para una aplicación web de Node.js. En este tutorial, la imagen se inserta en Azure Container Registry. Si no ha creado todavía la imagen de contenedor, vuelva a [Tutorial 1: Creación de una imagen de contenedor](container-instances-tutorial-prepare-app.md).

Azure Container Registry es su registro de Docker privado en Azure. En este tutorial, segunda parte de la serie, se realizan las siguientes operaciones:

> [!div class="checklist"]
> * Creación de una instancia de Azure Container Registry con la CLI de Azure
> * Etiquetar una imagen de contenedor para Azure Container Registry
> * Cargar la imagen en el registro

En el artículo siguiente, el último tutorial de la serie, va a implementar el contenedor del registro privado en Azure Container Instances.

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [container-instances-tutorial-prerequisites](../../includes/container-instances-tutorial-prerequisites.md)]

[!INCLUDE [container-instances-create-registry](../../includes/container-instances-create-registry.md)]

## <a name="tag-container-image"></a>Etiquetado de la imagen de contenedor

Para insertar una imagen de contenedor en un registro privado como Azure Container Registry, primero debe etiquetar la imagen con el nombre completo del servidor de inicio de sesión del registro.

En primer lugar, obtenga el nombre del servidor de inicio de sesión completo de la instancia de Azure Container Registry. Ejecute el siguiente comando [az acr show][az-acr-show] y reemplace `<acrName>` por el nombre del registro que acaba de crear:

```azurecli
az acr show --name <acrName> --query loginServer --output table
```

Por ejemplo, si el registro se denomina *mycontainerregistry082*:

```azurecli
az acr show --name mycontainerregistry082 --query loginServer --output table
```

```output
Result
------------------------
mycontainerregistry082.azurecr.io
```

Ahora, muestre la lista de imágenes locales con el comando [docker images][docker-images]:

```bash
docker images
```

Junto con cualquier otra imagen que tenga en su máquina, debería ver la imagen *aci-tutorial-app* que creó en el [ tutorial anterior](container-instances-tutorial-prepare-app.md):

```console
$ docker images
REPOSITORY          TAG       IMAGE ID        CREATED           SIZE
aci-tutorial-app    latest    5c745774dfa9    39 minutes ago    68.1 MB
```

Etiquete la imagen *aci-tutorial-app* con el servidor de inicio de sesión del registro de contenedor. Además, agregue la etiqueta `:v1` al final del nombre de imagen para indicar el número de versión de la imagen. Reemplace `<acrLoginServer>` por el resultado del comando [az acr show][az-acr-show] que ha ejecutado antes.

```bash
docker tag aci-tutorial-app <acrLoginServer>/aci-tutorial-app:v1
```

Vuelva a ejecutar `docker images` para comprobar la operación de etiquetado:

```console
$ docker images
REPOSITORY                                            TAG       IMAGE ID        CREATED           SIZE
aci-tutorial-app                                      latest    5c745774dfa9    39 minutes ago    68.1 MB
mycontainerregistry082.azurecr.io/aci-tutorial-app    v1        5c745774dfa9    7 minutes ago     68.1 MB
```

## <a name="push-image-to-azure-container-registry"></a>Inserción de imágenes en Azure Container Registry

Ahora que ha etiquetado la imagen *aci-tutorial-app* con el nombre del servidor de inicio de sesión completo del registro privado, puede insertarla en el registro con el comando [docker push][docker-push]. Reemplace `<acrLoginServer>` por el nombre del servidor de inicio de sesión completo obtenido en el paso anterior.

```bash
docker push <acrLoginServer>/aci-tutorial-app:v1
```

La operación `push` debería tardar de unos segundos a unos minutos según la conexión a Internet, y el resultado ser similar al siguiente:

```console
$ docker push mycontainerregistry082.azurecr.io/aci-tutorial-app:v1
The push refers to a repository [mycontainerregistry082.azurecr.io/aci-tutorial-app]
3db9cac20d49: Pushed
13f653351004: Pushed
4cd158165f4d: Pushed
d8fbd47558a8: Pushed
44ab46125c35: Pushed
5bef08742407: Pushed
v1: digest: sha256:ed67fff971da47175856505585dcd92d1270c3b37543e8afd46014d328f05715 size: 1576
```

## <a name="list-images-in-azure-container-registry"></a>Lista de imágenes en Azure Container Registry

Para comprobar que la imagen que acaba de insertar está efectivamente en el registro de contenedor de Azure, enumere las imágenes del registro con el comando [az acr repository list][az-acr-repository-list]. Reemplace `<acrName>` por el nombre de Registro de contenedor.

```azurecli
az acr repository list --name <acrName> --output table
```

Por ejemplo:

```azurecli
az acr repository list --name mycontainerregistry082 --output table
```

```output
Result
----------------
aci-tutorial-app
```

Para ver las *etiquetas* de una imagen concreta, use el comando [az acr repository show-tags][az-acr-repository-show-tags].

```azurecli
az acr repository show-tags --name <acrName> --repository aci-tutorial-app --output table
```

Debería ver una salida similar a esta:

```console
az acr repository show-tags --name mycontainerregistry082 --repository aci-tutorial-app --output table
Result
--------
v1
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha preparado una instancia de Azure Container Registry para usarla con Azure Container Instances y ha insertado una imagen de contenedor en el registro. Se han completado los siguientes pasos:

> [!div class="checklist"]
> * Creación de una instancia de Azure Container Registry con la CLI de Azure
> * Etiquetado de una imagen de contenedor para Azure Container Registry
> * Carga de una imagen en Azure Container Registry

Vaya al tutorial siguiente para obtener información sobre cómo implementar el contenedor en Azure con Azure Container Instances:

> [!div class="nextstepaction"]
> [Implementación de un contenedor en Azure Container Instances](container-instances-tutorial-deploy-app.md)

<!-- LINKS - External -->
[docker-build]: https://docs.docker.com/engine/reference/commandline/build/
[docker-get-started]: https://docs.docker.com/get-started/
[docker-hub-nodeimage]: https://store.docker.com/images/node
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[nodejs]: https://nodejs.org

<!-- LINKS - Internal -->
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-repository-list]: /cli/azure/acr/repository
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az-acr-repository-show-tags
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-group-create]: /cli/azure/group#az-group-create
[azure-cli-install]: /cli/azure/install-azure-cli
