---
title: Creación de imágenes de contenedor de Service Fabric en Azure
description: En este tutorial, aprenderá a crear imágenes de contenedor para una aplicación de Service Fabric con varios contenedores.
author: suhuruli
ms.topic: tutorial
ms.date: 07/22/2019
ms.author: suhuruli
ms.custom: mvc
ms.openlocfilehash: fe06da759a1ad42ef5cef888f98c440cdfb9569c
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "78252781"
---
# <a name="tutorial-create-container-images-on-a-linux-service-fabric-cluster"></a>Tutorial: Creación de imágenes de contenedor en un clúster de Service Fabric de Linux

Este tutorial es la primera parte de una serie de tutoriales en los que se muestra cómo usar los contenedores en un clúster de Service Fabric en Linux. En este tutorial, se prepara una aplicación con varios contenedores para usarla con Service Fabric. En los tutoriales posteriores, estas imágenes se usan como parte de una aplicación de Service Fabric. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Clonar el origen de la aplicación de GitHub
> * Crear una imagen de contenedor a partir del origen de la aplicación
> * Implementar una instancia de Azure Container Registry (ACR)
> * Etiquetar una imagen de contenedor para ACR
> * Cargar la imagen a ACR

En esta serie de tutoriales, se aprende a:

> [!div class="checklist"]
> * Crear imágenes de contenedor para Service Fabric
> * [Creación y ejecución de una aplicación de Service Fabric con Containers](service-fabric-tutorial-package-containers.md)
> * [Cómo se administran la conmutación por error y el escalado en Service Fabric](service-fabric-tutorial-containers-failover.md)

## <a name="prerequisites"></a>Prerequisites

* Configuración del entorno de desarrollo de Linux para Service Fabric. Siga las instrucciones que se indican [aquí](service-fabric-get-started-linux.md) para configurar el entorno de Linux.
* Para realizar este tutorial es necesario que ejecute la versión 2.0.4 o superior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).
* Además, requiere que tenga una suscripción de Azure disponible. Para obtener más información sobre una versión de evaluación gratuita, vaya [aquí](https://azure.microsoft.com/free/).

## <a name="get-application-code"></a>Obtención del código de la aplicación

La aplicación de ejemplo que se usa en este tutorial es una aplicación para votar. La aplicación consta de un componente web front-end y de una instancia back-end de Redis. Los componentes se empaquetan en imágenes de contenedor.

Use git para descargar una copia de la aplicación en su entorno de desarrollo.

```bash
git clone https://github.com/Azure-Samples/service-fabric-containers.git

cd service-fabric-containers/Linux/container-tutorial/
```

La solución contiene dos carpetas y un archivo "docker-compose.yml". La carpeta "azure-vote" contiene el servicio de front-end de Python junto con el Dockerfile que se usa para generar la imagen. El directorio "Voting" contiene el paquete de aplicación de Service Fabric que se implementa en el clúster. Estos directorios contienen los recursos necesarios para este tutorial.

## <a name="create-container-images"></a>Creación de imágenes de contenedor

En el directorio **azure-vote**, ejecute el siguiente comando para generar la imagen para el componente web de front-end. Este comando usa el archivo Dockerfile de este directorio para generar la imagen.

```bash
docker build -t azure-vote-front .
```
> [!Note]
> Si se deniega su permiso, siga [esta](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user) documentación sobre cómo trabajar con Docker sin sudo.

Este comando puede tardar algún tiempo porque todas las dependencias necesarias tienen que extraerse de Docker Hub. Cuando haya finalizado, use el comando [docker images](https://docs.docker.com/engine/reference/commandline/images/) para ver las imágenes creadas.

```bash
docker images
```

Tenga en cuenta que se han descargado o creado dos imágenes. La imagen *azure-vote-front* contiene la aplicación. Se derivó de una imagen de *Python* de Docker Hub.

```bash
REPOSITORY                   TAG                 IMAGE ID            CREATED              SIZE
azure-vote-front             latest              052c549a75bf        About a minute ago   708MB
tiangolo/uwsgi-nginx-flask   python3.6           590e17342131        5 days ago           707MB

```

## <a name="deploy-azure-container-registry"></a>Implementación de Azure Container Registry

Primero, ejecute el comando **az login** para iniciar sesión en su cuenta de Azure.

```azurecli
az login
```

A continuación, use el comando **az account** para elegir la suscripción para crear Azure Container Registry. Tendrá que escribir el identificador de suscripción de su suscripción de Azure en lugar de <subscription_id>.

```azurecli
az account set --subscription <subscription_id>
```

Para implementar Azure Container Registry, necesita tener antes un grupo de recursos. Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Para crear un grupo de recursos, use el comando **az group create**. En este ejemplo, se crea un grupo de recursos denominado *myResourceGroup* en la región *westus*.

```azurecli
az group create --name <myResourceGroup> --location westus
```

Cree una instancia de Azure Container Registry con el comando **az acr create**. Reemplace \<acrName> por el nombre del registro de contenedor que quiere crear en la suscripción. Este nombre debe ser alfanumérico y único.

```azurecli
az acr create --resource-group <myResourceGroup> --name <acrName> --sku Basic --admin-enabled true
```

En el resto de este tutorial, usamos "acrName" como un marcador de posición para el nombre del registro del contenedor que eligió. Anote este valor.

## <a name="sign-in-to-your-container-registry"></a>Inicio de sesión en el registro de contenedor

Inicie sesión en la instancia de ACR antes de insertar imágenes en ella. Use el comando **az acr login** para completar la operación. Proporcione el nombre único que se especificó para el registro de contenedor cuando se creó.

```azurecli
az acr login --name <acrName>
```

Al finalizar, el comando devuelve un mensaje que indica que el inicio de sesión se ha realizado correctamente.

## <a name="tag-container-images"></a>Etiquetado de imágenes de contenedor

Es preciso etiquetar cada imagen de contenedor con el nombre loginServer del registro. Esta etiqueta se usa para el enrutamiento al insertar imágenes de contenedor en un registro de imágenes.

Para ver una lista de imágenes actual, use el comando [docker images](https://docs.docker.com/engine/reference/commandline/images/).

```bash
docker images
```

Salida:

```bash
REPOSITORY                   TAG                 IMAGE ID            CREATED              SIZE
azure-vote-front             latest              052c549a75bf        About a minute ago   708MB
tiangolo/uwsgi-nginx-flask   python3.6           590e17342131        5 days ago           707MB
```

Para obtener el nombre de loginServer, ejecute el comando siguiente:

```azurecli
az acr show --name <acrName> --query loginServer --output table
```

Esta acción genera una tabla con los siguientes resultados. Este resultado se usará para etiquetar la imagen **azure-vote-front** antes de insertarla en el registro de contenedor en el paso siguiente.

```output
Result
------------------
<acrName>.azurecr.io
```

Ahora, etiquete la imagen *azure-vote-front* con el loginServer de su registro de contenedor. Además, agregue `:v1` al final del nombre de la imagen. Esta etiqueta indica la versión de la imagen.

```bash
docker tag azure-vote-front <acrName>.azurecr.io/azure-vote-front:v1
```

Una vez etiquetada, ejecute "docker images" para comprobar la operación.

Salida:

```output
REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
azure-vote-front                       latest              052c549a75bf        23 minutes ago      708MB
<acrName>.azurecr.io/azure-vote-front   v1                  052c549a75bf       23 minutes ago      708MB
tiangolo/uwsgi-nginx-flask             python3.6           590e17342131        5 days ago          707MB

```

## <a name="push-images-to-registry"></a>Inserción de imágenes en el registro

Inserte la imagen *azure-vote-front* en el registro. 

En el siguiente ejemplo, reemplace el nombre loginServer de ACR por el loginServer de su entorno.

```bash
docker push <acrName>.azurecr.io/azure-vote-front:v1
```

Los comandos de inserción de Docker tardan un par de minutos en completarse.

## <a name="list-images-in-registry"></a>Lista de imágenes en el registro

Para que se devuelva una lista de las imágenes que se han insertado en la instancia de Azure Container Registry, el usuario debe usar el comando [az acr repository list](/cli/azure/acr/repository). Actualice el comando con el nombre de instancia de ACR.

```azurecli
az acr repository list --name <acrName> --output table
```

Salida:

```output
Result
----------------
azure-vote-front
```

Al finalizar el tutorial, la imagen de contenedor se ha almacenado en una instancia privada de Azure Container Registry. Esta imagen se implementa desde ACR en un clúster de Service Fabric en tutoriales posteriores.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se extrajo una aplicación de GitHub y se crearon e insertaron imágenes de contenedor en un registro. Se han completado los siguientes pasos:

> [!div class="checklist"]
> * Clonar el origen de la aplicación de GitHub
> * Crear una imagen de contenedor a partir del origen de la aplicación
> * Implementar una instancia de Azure Container Registry (ACR)
> * Etiquetar una imagen de contenedor para ACR
> * Cargar la imagen a ACR

Avance hasta el siguiente tutorial para obtener información acerca del empaquetado de contenedores en una aplicación de Service Fabric mediante Yeoman.

> [!div class="nextstepaction"]
> [Empaquetar e implementar contenedores como aplicación de Service Fabric](service-fabric-tutorial-package-containers.md)
