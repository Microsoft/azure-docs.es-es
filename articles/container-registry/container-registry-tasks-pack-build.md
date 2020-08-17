---
title: Compilar imagen con Cloud Native Buildpack
description: Use el comando az acr pack build para compilar una imagen de contenedor desde una aplicación e insertarla en Azure Container Registry, sin usar ningún documento Dockerfile.
ms.topic: article
ms.date: 10/24/2019
ms.custom: devx-track-javascript
ms.openlocfilehash: f99175165e01a9287a66c59957fc7a239b00393a
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88077227"
---
# <a name="build-and-push-an-image-from-an-app-using-a-cloud-native-buildpack"></a>Compilación e inserción de una imagen desde una aplicación mediante una instancia de Cloud Native Buildpacks

El comando de la CLI de Azure `az acr pack build` usa la herramienta de la CLI [`pack`](https://github.com/buildpack/pack), desde [Buildpacks](https://buildpacks.io/), para compilar una aplicación e insertar su imagen en un registro de contenedor de Azure. Esta característica proporciona una opción para compilar rápidamente una imagen de contenedor a partir del código fuente de la aplicación en Node.js, Java y otros lenguajes sin tener que definir ningún documento Dockerfile.

Puede usar Azure Cloud Shell o una instalación local de la CLI de Azure para ejecutar los ejemplos de este artículo. Si quiere usarla de forma local, necesitará la versión 2.0.70 u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

> [!IMPORTANT]
> Esta funcionalidad actualmente está en su versión preliminar. Las versiones preliminares están a su disposición con la condición de que acepte los [términos de uso adicionales][terms-of-use]. Es posible que algunos de los aspectos de esta característica cambien antes de ofrecer disponibilidad general.

## <a name="use-the-build-command"></a>Ejecución del comando de compilación

Para compilar e instalar una imagen de contenedor mediante Cloud Native Buildpacks, ejecute el comando [az acr pack build][az-acr-pack-build]. Mientras que el comando [az acr build][az-acr-build] compila e inserta una imagen a partir de un código fuente de Dockerfile y otro código relacionado, con `az acr pack build` puede especificar directamente un árbol del código fuente de la aplicación.

Como mínimo, especifique lo siguiente cuando ejecute `az acr pack build`:

* Un registro de contenedor de Azure en el que ejecutar el comando
* Un nombre de imagen y una etiqueta para la imagen resultante
* Una de las [ubicaciones de contexto compatibles](container-registry-tasks-overview.md#context-locations) con ACR Tasks, como un directorio local, un repositorio de GitHub o un tarball remoto
* El nombre de una imagen de compilador de buildpacks adecuada para la aplicación. Azure Container Registry copia en caché imágenes del compilador, como `cloudfoundry/cnb:0.0.34-cflinuxfs3`, para agilizar las compilaciones.  

El comando `az acr pack build` admite otras características de los comandos de ACR Tasks, incluidas las [variables de ejecución](container-registry-tasks-reference-yaml.md#run-variables) y los [registros de ejecución de tareas](container-registry-tasks-logs.md) que se transmiten y también se guardan para poderse recuperar posteriormente.

## <a name="example-build-nodejs-image-with-cloud-foundry-builder"></a>Ejemplo: Compilación de la imagen de Node.js con el compilador de Cloud Foundry

En el ejemplo siguiente se crea una imagen de contenedor a partir de la aplicación Node.js en el repositorio [Azure-Samples/nodejs-docs-hello-world](https://github.com/Azure-Samples/nodejs-docs-hello-world), con el compilador `cloudfoundry/cnb:0.0.34-cflinuxfs3`. Azure Container Registry copia en caché este compilador, por lo que no se requiere un parámetro `--pull`:

```azurecli
az acr pack build \
    --registry myregistry \
    --image {{.Run.Registry}}/node-app:1.0 \
    --builder cloudfoundry/cnb:0.0.34-cflinuxfs3 \
    https://github.com/Azure-Samples/nodejs-docs-hello-world.git
```

En este ejemplo se crea la imagen `node-app` con la etiqueta `1.0` y se inserta en el registro de contenedor *myregistry*. En este ejemplo, el nombre del registro de destino se antepone explícitamente al nombre de la imagen. Si no se especifica, el nombre del servidor de inicio de sesión del registro se antepone automáticamente al nombre de la imagen.

La salida del comando muestra el progreso de la compilación e inserción de la imagen. 

Una vez que la imagen se crea correctamente, puede ejecutarla con Docker, si lo tiene instalado. En primer lugar, inicie sesión en el registro:

```azurecli
az acr login --name myregistry
```

Ejecute la imagen:

```console
docker run --rm -p 1337:1337 myregistry.azurecr.io/node-app:1.0
```

Vaya a `localhost:1337` en su explorador favorito para ver la aplicación web de ejemplo. Presione `[Ctrl]+[C]` para detener el contenedor.

## <a name="example-build-java-image-with-heroku-builder"></a>Ejemplo: Compilación de una imagen de Java con el compilador de Heroku

En el ejemplo siguiente se crea una imagen de contenedor desde la aplicación Java en el repositorio [buildpack/sample-java-app](https://github.com/buildpack/sample-java-app), mediante el compilador `heroku/buildpacks:18`. El parámetro `--pull` especifica que el comando debe extraer la última imagen del compilador. 

```azurecli
az acr pack build \
    --registry myregistry \
    --image java-app:{{.Run.ID}} \
    --pull --builder heroku/buildpacks:18 \
    https://github.com/buildpack/sample-java-app.git
```

En este ejemplo se crea la imagen `java-app` etiquetada con el identificador de ejecución del comando y se inserta en el registro de contenedor *myregistry*.

La salida del comando muestra el progreso de la compilación e inserción de la imagen. 

Una vez que la imagen se crea correctamente, puede ejecutarla con Docker, si lo tiene instalado. En primer lugar, inicie sesión en el registro:

```azurecli
az acr login --name myregistry
```

Para ejecutar la imagen, puede sustituir la etiqueta de la imagen por *runid*:

```console
docker run --rm -p 8080:8080 myregistry.azurecr.io/java-app:runid
```

Vaya a `localhost:8080` en su explorador favorito para ver la aplicación web de ejemplo. Presione `[Ctrl]+[C]` para detener el contenedor.


## <a name="next-steps"></a>Pasos siguientes

Después de compilar e insertar una imagen de contenedor con `az acr pack build`, puede implementarla como cualquier imagen en un destino de su elección. Entre las opciones de implementación de Azure, se incluye su ejecución en [App Service](../app-service/tutorial-custom-container.md) o [Azure Kubernetes Service](../aks/tutorial-kubernetes-deploy-cluster.md), entre otros.

Para obtener más información sobre las características de ACR Tasks, vea el artículo [Automatización de compilaciones y mantenimiento de imágenes de contenedor con ACR Tasks](container-registry-tasks-overview.md).


<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr/task
[az-acr-pack-build]: /cli/azure/acr/pack#az-acr-pack-build
