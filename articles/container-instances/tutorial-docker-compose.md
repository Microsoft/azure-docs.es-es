---
title: 'Tutorial: uso de Docker Compose para implementar un grupo de varios contenedores'
description: Use Docker Compose para compilar y ejecutar una aplicación de varios contenedores y, a continuación, abrir la aplicación en Azure Container Instances.
ms.topic: tutorial
ms.date: 09/14/2020
ms.custom: ''
ms.openlocfilehash: 1e8a5cd856358a0dc3e9c356cb3a55f75db29c86
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90708300"
---
# <a name="tutorial-deploy-a-multi-container-group-using-docker-compose"></a>Tutorial: Implementación de un grupo de varios contenedores mediante Docker Compose 

En este tutorial, usará [Docker Compose](https://docs.docker.com/compose/) para definir y ejecutar una aplicación de varios contenedores de forma local y, a continuación, implementarla como un [grupo de contenedores](container-instances-container-groups.md) en Azure Container Instances. 

Ejecute los contenedores en Azure Container Instances a petición cuando desarrolle aplicaciones nativas en la nube y quiera cambiar sin problemas entre el desarrollo local y la implementación en la nube. La [integración entre Docker y Azure](https://docs.docker.com/engine/context/aci-integration/) habilita esta capacidad. Puede usar los comandos nativos de Docker para ejecutar [una única instancia de contenedor](quickstart-docker-cli.md) o un grupo de varios contenedores en Azure.

> [!IMPORTANT]
> No se admiten todas las características de Azure Container Instances. Proporcione comentarios sobre la integración entre Docker y Azure mediante la creación de una incidencia en el repositorio de GitHub de [integración de Docker ACI](https://github.com/docker/aci-integration-beta).

> [!TIP]
> Puede usar la [extensión de Docker para Visual Studio Code](https://aka.ms/VSCodeDocker) para obtener una experiencia integrada para desarrollar, ejecutar y administrar contenedores, imágenes y contextos.

En este artículo:

> [!div class="checklist"]
> * Creación de una instancia de Azure Container Registry
> * Clonar el código fuente de la aplicación desde GitHub
> * Usar Docker Compose para compilar una imagen y ejecutar una aplicación de varios contenedores de forma local.
> * Inserir la imagen de la aplicación en el registro de contenedor.
> * Crear un contexto de Azure para Docker.
> * Llevar la aplicación a Azure Container Instances.

## <a name="prerequisites"></a>Requisitos previos

* **CLI de Azure**: debe tener la CLI de Azure instalada en el equipo local. Se recomienda la versión 2.10.1 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

* **Docker Desktop**: debe usar Docker Desktop versión 2.3.0.5 o posterior, disponible para [Windows](https://desktop.docker.com/win/edge/Docker%20Desktop%20Installer.exe) o [macOS](https://desktop.docker.com/mac/edge/Docker.dmg). O bien, instale la [CLI de integración de Docker ACI para Linux](https://docs.docker.com/engine/context/aci-integration/#install-the-docker-aci-integration-cli-on-linux).

[!INCLUDE [container-instances-create-registry](../../includes/container-instances-create-registry.md)]

## <a name="get-application-code"></a>Obtención del código de la aplicación

La aplicación de ejemplo que se usa en este tutorial es una aplicación básica para votar. La aplicación consta de un componente web front-end y de una instancia back-end de Redis. El componente web se empaqueta en una imagen de contenedor personalizada. La instancia de Redis usa una imagen sin modificar de Docker Hub.

Use [git](https://git-scm.com/downloads) para clonar la aplicación en su entorno de desarrollo:

```console
git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
```

Cambie al directorio clonado.

```console
cd azure-voting-app-redis
```

En el directorio se encuentra el código fuente de la aplicación, un archivo de Docker Compose creado previamente y un archivo docker-compose.yaml.

## <a name="modify-docker-compose-file"></a>Modificación del archivo de Docker Compose

Abra el archivo docker-compose.yaml en un editor de texto. El archivo configura los servicios `azure-vote-back` y `azure-vote-front`.

```yml
version: '3'
services:
  azure-vote-back:
    image: redis
    container_name: azure-vote-back
    ports:
        - "6379:6379"

  azure-vote-front:
    build: ./azure-vote
    image: azure-vote-front
    container_name: azure-vote-front
    environment:
      REDIS: azure-vote-back
    ports:
        - "8080:80"
```

En la configuración `azure-vote-front`, realice los dos cambios siguientes:

1. Actualice la propiedad `image` en el servicio `azure-vote-front`. Use como prefijo del nombre de la imagen el nombre del servidor de inicio de sesión de Azure Container Registry, \<acrName\>.azurecr.io. Por ejemplo, si el registro se denomina *myregistry*, el nombre del servidor de inicio de sesión es *myregistry.azurecr.io* (todo en minúsculas) y la propiedad de imagen es `myregistry.azurecr.io/azure-vote-front`.
1. Cambie la asignación de `ports` a `80:80`. Guarde el archivo.

El archivo actualizado debe ser parecido al siguiente:

```yml
version: '3'
services:
  azure-vote-back:
    image: redis
    container_name: azure-vote-back
    ports:
        - "6379:6379"

  azure-vote-front:
    build: ./azure-vote
    image: myregistry.azurecr.io/azure-vote-front
    container_name: azure-vote-front
    environment:
      REDIS: azure-vote-back
    ports:
        - "80:80"
```

Al realizar estas sustituciones, la imagen de `azure-vote-front` que se crea en el paso siguiente se etiqueta para Azure Container Registry y se puede extraer para ejecutarse en Azure Container Instances.

> [!TIP]
> No tiene que usar Azure Container Registry para este escenario. Por ejemplo, puede elegir un repositorio privado en Docker Hub para hospedar la imagen de la aplicación. Si elige un registro diferente, actualice la propiedad de imagen correctamente.

## <a name="run-multi-container-application-locally"></a>Ejecución local de una aplicación de varios contenedores

Ejecute el archivo [docker-compose up](https://docs.docker.com/compose/reference/up/), que usa el archivo `docker-compose.yaml` para compilar la imagen de contenedor, descargue la imagen de Redis e inicie la aplicación:

```console
docker-compose up --build -d
```

Cuando haya finalizado, use el comando [docker images](https://docs.docker.com/engine/reference/commandline/images/) para ver las imágenes creadas. Se han descargado o creado tres imágenes. La imagen `azure-vote-front` contiene la aplicación de front-end y usa la imagen `uwsgi-nginx-flask` como base. La imagen `redis` se usa para iniciar una instancia de Redis.

```
$ docker images

REPOSITORY                                TAG        IMAGE ID            CREATED             SIZE
myregistry.azurecr.io/azure-vote-front    latest     9cc914e25834        40 seconds ago      944MB
redis                                     latest     a1b99da73d05        7 days ago          104MB
tiangolo/uwsgi-nginx-flask                python3.6  788ca94b2313        9 months ago        9444MB
```

Ejecute el comando [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) para ver los contenedores en ejecución:

```
$ docker ps

CONTAINER ID        IMAGE                                   COMMAND                  CREATED             STATUS              PORTS                           NAMES
82411933e8f9        myregistry.azurecr.io/azure-vote-front  "/entrypoint.sh /sta…"   57 seconds ago      Up 30 seconds       443/tcp, 0.0.0.0:80->80/tcp   azure-vote-front
b68fed4b66b6        redis                                   "docker-entrypoint.s…"   57 seconds ago      Up 30 seconds       0.0.0.0:6379->6379/tcp          azure-vote-back
```

Para ver la aplicación en ejecución, escriba `http://localhost:80` en un explorador web local. Se carga la aplicación de ejemplo, como se muestra en el ejemplo siguiente:

:::image type="content" source="media/tutorial-docker-compose/azure-vote.png" alt-text="Imagen de la aplicación de votación":::

Después de probar la aplicación local, ejecute [docker-compose down](https://docs.docker.com/compose/reference/down/) para detener la aplicación y quitar los contenedores.

```console
docker-compose down
```

## <a name="push-image-to-container-registry"></a>Inserción de imágenes en Container Registry

Para implementar la aplicación en Azure Container Instances, es preciso que inserte la imagen de `azure-vote-front` en el registro de contenedor. Ejecute [docker-compose push](https://docs.docker.com/compose/reference/push) para insertar la imagen:

```console
docker-compose push
```

La inserciones en el registro puede tardar unos minutos.

Para comprobar que la imagen está almacenada en el registro, ejecute el comando [az acr repository show](/cli/azure/acr/repository#az-acr-repository-show):

```azurecli
az acr repository show --name <acrName> --repository azure-vote-front
```

[!INCLUDE [container-instances-create-docker-context](../../includes/container-instances-create-docker-context.md)]

## <a name="deploy-application-to-azure-container-instances"></a>Implementación de una aplicación en Azure Container Instances

A continuación, cambie al contexto de ACI. Los comandos de Docker subsiguientes se ejecutarán en este contexto.

```console
docker context use myacicontext
```

Ejecute `docker compose up` para iniciar la aplicación en Azure Container Instances. La imagen de `azure-vote-front` se extrae del registro de contenedor y el grupo de contenedores se crea en Azure Container Instances.

```console
docker compose up
```

> [!NOTE]
> Los comandos de Docker Compose disponibles actualmente en un contexto de ACI son `docker compose up` y `docker compose down`. No hay ningún guion entre `docker` y `compose` en estos comandos.

En poco tiempo, se implementa el grupo de contenedores. Salida del ejemplo:

```
[+] Running 3/3
 ⠿ Group azurevotingappredis  Created                          3.6s
 ⠿ azure-vote-back            Done                             10.6s
 ⠿ azure-vote-front           Done                             10.6s
```

Ejecute `docker ps` para ver los contenedores en ejecución y la dirección IP asignada al grupo de contenedores.

```console
docker ps
```

Salida del ejemplo:

```
CONTAINER ID                           IMAGE                                    COMMAND             STATUS              PORTS
azurevotingappredis_azure-vote-back    redis                                                        Running             52.179.23.131:6379->6379/tcp
azurevotingappredis_azure-vote-front   myregistry.azurecr.io/azure-vote-front                       Running             52.179.23.131:80->80/tcp
```

Para ver la aplicación en ejecución en la nube, escriba la dirección IP mostrada en un explorador web local. En este ejemplo, escriba `52.179.23.131`. Se carga la aplicación de ejemplo, como se muestra en el ejemplo siguiente:

:::image type="content" source="media/tutorial-docker-compose/azure-vote-aci.png" alt-text="Imagen de la aplicación de votación en ACI":::

Para ver los registros del contenedor de front-end, ejecute el comando [docker logs](https://docs.docker.com/engine/reference/commandline/logs). Por ejemplo:

```console
docker logs azurevotingappredis_azure-vote-front
```

También puede usar Azure Portal u otras herramientas de Azure para ver las propiedades y el estado del grupo de contenedores que ha implementado.

Cuando termine de probar la aplicación, detenga la aplicación y los contenedores con `docker compose down`:

```console
docker compose down
```

Este comando elimina el grupo de contenedores de Azure Container Instances.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha usado Docker Compose para pasar de ejecutar una aplicación de varios contenedores localmente a ejecutarla en Azure Container Instances. Ha aprendido a:

> [!div class="checklist"]
> * Creación de una instancia de Azure Container Registry
> * Clonar el código fuente de la aplicación desde GitHub
> * Usar Docker Compose para compilar una imagen y ejecutar una aplicación de varios contenedores de forma local.
> * Inserir la imagen de la aplicación en el registro de contenedor.
> * Crear un contexto de Azure para Docker.
> * Llevar la aplicación a Azure Container Instances.

También puede usar la [extensión de Docker para Visual Studio Code](https://aka.ms/VSCodeDocker) para obtener una experiencia integrada para desarrollar, ejecutar y administrar contenedores, imágenes y contextos.

Si quiere aprovechar más características de Azure Container Instances, use las herramientas de Azure para especificar un grupo de varios contenedores. Por ejemplo, consulte los tutoriales para implementar un grupo de contenedores mediante la CLI de Azure con un [archivo YAML](container-instances-multi-container-yaml.md), o bien realice la implementación con una [plantilla de Azure Resource Manager](container-instances-multi-container-group.md). 