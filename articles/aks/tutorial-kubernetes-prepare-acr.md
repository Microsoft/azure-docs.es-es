---
title: 'Kubernetes en el tutorial de Azure: Creación de un registro de contenedor'
description: En este tutorial de Azure Kubernetes Service (AKS) se crea una instancia de Azure Container Registry y se carga una imagen del contenedor de aplicaciones de ejemplo.
services: container-service
ms.topic: tutorial
ms.date: 12/19/2018
ms.custom: mvc
ms.openlocfilehash: 197e5c7bed569e67376f9c28fe0d2e050016cce8
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87922411"
---
# <a name="tutorial-deploy-and-use-azure-container-registry"></a>Tutorial: Implementación y uso de Azure Container Registry

Azure Container Registry (ACR) es un registro privado para imágenes de contenedor. Un registro de contenedor privado permite compilar e implementar aplicaciones y código personalizado de forma segura. En este tutorial, que es la segunda de las siete partes, se implementa una instancia de ACR y se insertar una imagen de contenedor en ella. Aprenderá a:

> [!div class="checklist"]
> * Crear una instancia de Azure Container Registry (ACR)
> * Etiquetar una imagen de contenedor para ACR
> * Cargar la imagen a ACR
> * Ver las imágenes del registro

En tutoriales adicionales, esta instancia de ACR se integra con un clúster de Kubernetes en AKS y se implementa una aplicación a partir de la imagen.

## <a name="before-you-begin"></a>Antes de empezar

En el [tutorial anterior][aks-tutorial-prepare-app], se creó una imagen de contenedor para una aplicación sencilla de Azure Voting. Si no ha creado la imagen de la aplicación de Azure Voting, vuelva al [Tutorial 1: creación de imágenes de contenedor][aks-tutorial-prepare-app].

Para realizar este tutorial es necesario ejecutar la versión 2.0.53, o superior, de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

## <a name="create-an-azure-container-registry"></a>Creación de una instancia de Azure Container Registry

Para crear una instancia de Azure Container Registry, es preciso tener antes un grupo de recursos. Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Para crear un grupo de recursos, use el comando [az group create][az-group-create]. En el siguiente ejemplo, se crea un grupo de recursos denominado *myResourceGroup* en la región *eastus*:

```azurecli
az group create --name myResourceGroup --location eastus
```

Cree una instancia de Azure Container Registry con el comando [crear az acr][az-acr-create] y especifique su propio nombre de registro. El nombre del registro debe ser único dentro de Azure y contener entre 5 y 50 caracteres alfanuméricos. En el resto del tutorial, se usa `<acrName>` como marcador de posición del nombre del registro de contenedor. Proporcione su propio nombre único de registro. La SKU *básica* es un punto de entrada optimizado para costo con fines de desarrollo que proporciona un equilibrio entre almacenamiento y rendimiento.

```azurecli
az acr create --resource-group myResourceGroup --name <acrName> --sku Basic
```

## <a name="log-in-to-the-container-registry"></a>Inicio de sesión en el registro de contenedor

Para usar la instancia de ACR, primero debe iniciar sesión. Use el comando [az acr login][az-acr-login] y especifique el nombre único que dio al registro de contenedor en el paso anterior.

```azurecli
az acr login --name <acrName>
```

Al finalizar, el comando devuelve un mensaje que indica que el *inicio de sesión se ha realizado correctamente*.

## <a name="tag-a-container-image"></a>Etiquetado de una imagen de contenedor

Para ver una lista de las imágenes locales actuales, use el comando [docker images][docker-images]:

```azurecli
$ docker images
```
La salida del comando anterior muestra una lista de las imágenes locales actuales:

```
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
azure-vote-front             latest              4675398c9172        13 minutes ago      694MB
redis                        latest              a1b99da73d05        7 days ago          106MB
tiangolo/uwsgi-nginx-flask   flask               788ca94b2313        9 months ago        694MB
```

Para usar la imagen de contenedor *azure-vote-front* con ACR, se debe etiquetar con la dirección del servidor de inicio de sesión del registro de imagen. Esta etiqueta se usa para el enrutamiento al insertar imágenes de contenedor en un registro de imágenes.

Para obtener la dirección del servidor de inicio de sesión, use el comando [az acr list][az-acr-list] y consulte *loginServer* como se indica a continuación:

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

Ahora, etiquete la imagen *azure-vote-front* local con la dirección *acrLoginServer* del registro de contenedor. Para indicar la versión de la imagen, agregue *:v1* al final de su nombre:

```console
docker tag azure-vote-front <acrLoginServer>/azure-vote-front:v1
```

Para comprobar que se aplican las etiquetas, vuelva a ejecutar [docker images][docker-images]. 

```azurecli
$ docker images
```

Una imagen se etiqueta con la dirección de la instancia de ACR y un número de versión.

```
REPOSITORY                                           TAG           IMAGE ID            CREATED             SIZE
azure-vote-front                                     latest        eaf2b9c57e5e        8 minutes ago       716 MB
mycontainerregistry.azurecr.io/azure-vote-front      v1            eaf2b9c57e5e        8 minutes ago       716 MB
redis                                                latest        a1b99da73d05        7 days ago          106MB
tiangolo/uwsgi-nginx-flask                           flask         788ca94b2313        8 months ago        694 MB
```

## <a name="push-images-to-registry"></a>Inserción de imágenes en el registro

Con la imagen generada y etiquetada, inserte la imagen *azure-vote-front* en la instancia de ACR. Use [docker push][docker-push] y especifique su propia dirección *acrLoginServer* para el nombre de la imagen, tal como se muestra a continuación:

```console
docker push <acrLoginServer>/azure-vote-front:v1
```

La inserción de la imagen en ACR puede tardar varios minutos en completarse.

## <a name="list-images-in-registry"></a>Lista de imágenes en el registro

Para que se devuelva una lista de las imágenes que se han insertado en la instancia de ACR, el usuario debe utilizar el comando [az acr repository list][az-acr-repository-list]. Escriba su propio `<acrName>` como se indica a continuación:

```azurecli
az acr repository list --name <acrName> --output table
```

En la siguiente salida de ejemplo se muestra la imagen *azure-vote-front* como disponible en el registro:

```
Result
----------------
azure-vote-front
```

Para ver las etiquetas de una imagen concreta, use el comando [az acr repository show-tags][az-acr-repository-show-tags] como se indica a continuación:

```azurecli
az acr repository show-tags --name <acrName> --repository azure-vote-front --output table
```

La salida de ejemplo siguiente muestra la imagen de *v1* etiquetada en un paso anterior:

```
Result
--------
v1
```

Ya hay una imagen de contenedor almacenada en una instancia de Azure Container Registry privada. En el siguiente tutorial, dicha imagen se implementa desde ACR en un clúster de Kubernetes.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una instancia de Azure Container Registry y ha insertado una imagen para su uso en un clúster de AKS. Ha aprendido a:

> [!div class="checklist"]
> * Crear una instancia de Azure Container Registry (ACR)
> * Etiquetar una imagen de contenedor para ACR
> * Cargar la imagen a ACR
> * Ver las imágenes del registro

Vaya al siguiente tutorial para aprender a implementar un clúster de Kubernetes en Azure.

> [!div class="nextstepaction"]
> [Implementación de un clúster de Kubernetes][aks-tutorial-deploy-cluster]

<!-- LINKS - external -->
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/

<!-- LINKS - internal -->
[az-acr-create]: /cli/azure/acr
[az-acr-list]: /cli/azure/acr
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-list]: /cli/azure/acr#az-acr-list
[az-acr-repository-list]: /cli/azure/acr/repository
[az-acr-repository-show-tags]: /cli/azure/acr/repository
[az-group-create]: /cli/azure/group#az-group-create
[azure-cli-install]: /cli/azure/install-azure-cli
[aks-tutorial-deploy-cluster]: ./tutorial-kubernetes-deploy-cluster.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
