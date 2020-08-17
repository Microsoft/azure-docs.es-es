---
title: 'Tutorial: Preparación de la imagen de contenedor para la implementación'
description: 'Tutorial de Azure Container Instances, parte 1 de 3: preparación de una aplicación en una imagen de contenedor para su implementación en Azure Container Instances'
ms.topic: tutorial
ms.date: 03/21/2018
ms.custom: seodec18, mvc, devx-track-javascript
ms.openlocfilehash: 3a73d68f99c7e2ec4c71e408abbdbc7eec7ee95d
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407173"
---
# <a name="tutorial-create-a-container-image-for-deployment-to-azure-container-instances"></a>Tutorial: Creación de una imagen de contenedor para su implementación en Azure Container Instances

Azure Container Instances permite la implementación de contenedores de Docker en una infraestructura de Azure sin necesidad de aprovisionar ninguna máquina virtual o adoptar un servicio de nivel superior. En este tutorial, se empaqueta una pequeña aplicación web de Node.js en una imagen del contenedor que se puede ejecutar mediante Azure Container Instances.

En este artículo, la primera parte de la serie, podrá:

> [!div class="checklist"]
> * Clonar el código fuente de la aplicación desde GitHub
> * Crear una imagen de contenedor desde el origen de la aplicación
> * Probar la imagen en un entorno local de Docker

En las partes dos y tres del tutorial, se carga la imagen de Azure Container Registry y, después, se implementa en Azure Container Instances.

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [container-instances-tutorial-prerequisites](../../includes/container-instances-tutorial-prerequisites.md)]

## <a name="get-application-code"></a>Obtención del código de la aplicación

La aplicación de ejemplo de este tutorial es una aplicación web sencilla compilada en [Node.js][nodejs]. La aplicación sirve una página HTML estática y es similar a la captura de pantalla siguiente:

![Aplicación del tutorial tal como se muestra en un explorador][aci-tutorial-app]

Use Git para clonar el repositorio de la aplicación de ejemplo:

```bash
git clone https://github.com/Azure-Samples/aci-helloworld.git
```

También puede [descargar el archivo ZIP][aci-helloworld-zip] directamente de GitHub.

## <a name="build-the-container-image"></a>Compilación de la imagen del contenedor

El archivo Dockerfile de la aplicación de ejemplo muestra cómo se compila el contenedor. Se comienza con una [imagen oficial de Node.js][docker-hub-nodeimage] basada en [Alpine Linux][alpine-linux], una pequeña distribución muy apropiada para usarla con contenedores. Luego copia los archivos de aplicación en el contenedor, instala las dependencias mediante Node Package Manager y, por último, inicia la aplicación.

```Dockerfile
FROM node:8.9.3-alpine
RUN mkdir -p /usr/src/app
COPY ./app/ /usr/src/app/
WORKDIR /usr/src/app
RUN npm install
CMD node /usr/src/app/index.js
```

Use el comando [docker build][docker-build] para crear la imagen del contenedor y etiquetarla como *aci-tutorial-app*:

```bash
docker build ./aci-helloworld -t aci-tutorial-app
```

La salida del comando [docker build][docker-build] es similar a la siguiente (truncada para favorecer la legibilidad):

```console
$ docker build ./aci-helloworld -t aci-tutorial-app
Sending build context to Docker daemon  119.3kB
Step 1/6 : FROM node:8.9.3-alpine
8.9.3-alpine: Pulling from library/node
88286f41530e: Pull complete
84f3a4bf8410: Pull complete
d0d9b2214720: Pull complete
Digest: sha256:c73277ccc763752b42bb2400d1aaecb4e3d32e3a9dbedd0e49885c71bea07354
Status: Downloaded newer image for node:8.9.3-alpine
 ---> 90f5ee24bee2
...
Step 6/6 : CMD node /usr/src/app/index.js
 ---> Running in f4a1ea099eec
 ---> 6edad76d09e9
Removing intermediate container f4a1ea099eec
Successfully built 6edad76d09e9
Successfully tagged aci-tutorial-app:latest
```

Use el comando [docker images][docker-images] para ver la imagen compilada:

```bash
docker images
```

La imagen recién compilada debería aparecer en la lista:

```console
$ docker images
REPOSITORY          TAG       IMAGE ID        CREATED           SIZE
aci-tutorial-app    latest    5c745774dfa9    39 seconds ago    68.1 MB
```

## <a name="run-the-container-locally"></a>Ejecute el contenedor localmente

Antes de implementar el contenedor en Azure Container Instances, use [docker run][docker-run] para ejecutarlo localmente y confirmar que funciona. El modificador `-d` permite que el contenedor se ejecute en segundo plano, mientras que `-p` permite asignar un puerto arbitrario del equipo al puerto 80 del contenedor.

```bash
docker run -d -p 8080:80 aci-tutorial-app
```

La salida del comando `docker run` muestra el identificador del contenedor en ejecución si el comando se realizó correctamente:

```console
$ docker run -d -p 8080:80 aci-tutorial-app
a2e3e4435db58ab0c664ce521854c2e1a1bda88c9cf2fcff46aedf48df86cccf
```

Ahora, vaya a `http://localhost:8080` en el explorador para confirmar que el contenedor está en ejecución. Debería ver una página web similar a la siguiente:

![Ejecución local de la aplicación en el explorador][aci-tutorial-app-local]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una imagen de contenedor que se puede implementar en Azure Container Instances y ha comprobado qu8e funciona localmente. Hasta ahora, esto es lo que ha hecho:

> [!div class="checklist"]
> * Se ha clonado el origen de la aplicación desde GitHub
> * Ha creado una imagen de contenedor a partir del origen de la aplicación
> * El contenedor se ha probado localmente

Pase al siguiente tutorial de la serie para aprender a almacenar imágenes de contenedor en Azure Container Registry:

> [!div class="nextstepaction"]
> [Inserción de imágenes en Azure Container Registry](container-instances-tutorial-prepare-acr.md)

<!--- IMAGES --->
[aci-tutorial-app]:./media/container-instances-quickstart/aci-app-browser.png
[aci-tutorial-app-local]: ./media/container-instances-tutorial-prepare-app/aci-app-browser-local.png

<!-- LINKS - External -->
[aci-helloworld-zip]: https://github.com/Azure-Samples/aci-helloworld/archive/master.zip
[alpine-linux]: https://alpinelinux.org/
[docker-build]: https://docs.docker.com/engine/reference/commandline/build/
[docker-get-started]: https://docs.docker.com/get-started/
[docker-hub-nodeimage]: https://store.docker.com/images/node
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[nodejs]: https://nodejs.org

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
