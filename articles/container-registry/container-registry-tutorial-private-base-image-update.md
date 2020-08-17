---
title: 'Tutorial: Desencadenamiento de la compilación de imágenes mediante la actualización de imágenes de base'
description: En este tutorial se configura una tarea de Azure Container Registry que desencadena automáticamente compilaciones de imágenes de contenedor en la nube cuando se actualiza una imagen de base en otro registro de contenedor de Azure privado.
ms.topic: tutorial
ms.date: 01/22/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: e28dc65f44f3d9997a59e6204451cc84b0e41e40
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407139"
---
# <a name="tutorial-automate-container-image-builds-when-a-base-image-is-updated-in-another-private-container-registry"></a>Tutorial: Automatización de las compilaciones de imágenes de contenedor al actualizarse una imagen de base en otro registro de contenedor privado 

ACR Tasks admite las compilaciones automatizadas de imágenes cuando se [actualiza una imagen de base](container-registry-tasks-base-images.md) del contenedor, como cuando se revisa el sistema operativo o el marco de trabajo de la aplicación de una de las imágenes de base. 

En este tutorial aprenderá a crear una tarea en ACR que desencadene una compilación en la nube cuando se inserte una imagen de base de contenedor en otro registro de contenedor de Azure. También puede probar un tutorial para crear una tarea en ACR que desencadene una compilación de imagen cuando se inserte una imagen de base en el [mismo registro de contenedor de Azure](container-registry-tutorial-base-image-update.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear la imagen de base en un registro de base
> * Crear una tarea de compilación de aplicaciones en otro registro para realizar el seguimiento de la imagen de base 
> * Actualización de la imagen base para que desencadene una tarea de imagen de aplicación
> * Mostrar la tarea desencadenada
> * Comprobación de la imagen de aplicación actualizada

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si quiere usar la CLI de Azure de forma local, debe tener la versión **2.0.68** de esta u otra posterior instalada. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure][azure-cli].

## <a name="prerequisites"></a>Prerrequisitos

### <a name="complete-the-previous-tutorials"></a>Complete los tutoriales anteriores.

En este tutorial se da por supuesto que ya ha completado los pasos de los dos primeros tutoriales de la serie, en los cuales puede realizar la:

* Creación de una instancia de Azure Container Registry
* Bifurcación del repositorio de ejemplo
* Clonación del repositorio de ejemplo
* Creación de un token de acceso personal de GitHub

Si aún no lo ha hecho, complete los primeros tutoriales antes de continuar:

[Compilación de imágenes de contenedor en la nube con Azure Container Registry Tasks](container-registry-tutorial-quick-task.md)

[Automatización de compilaciones de imágenes de contenedor con Azure Container Registry Tasks](container-registry-tutorial-build-task.md)

Además del registro de contenedor creado para los tutoriales anteriores, debe crear un registro para almacenar las imágenes de base. Si lo desea, cree el segundo registro en una ubicación diferente a la del registro original.

### <a name="configure-the-environment"></a>Configuración del entorno

Rellene estas variables de entorno de shell con valores adecuados para el entorno. Este paso no es estrictamente necesario, pero hace que la ejecución de los comandos de varias líneas de la CLI de Azure en este tutorial sea un poco más fácil. Si no rellena estas variables de entorno, debe reemplazar manualmente cada valor siempre que aparezca en los comandos de ejemplo.

```azurecli-interactive
BASE_ACR=<base-registry-name>   # The name of your Azure container registry for base images
ACR_NAME=<registry-name>        # The name of your Azure container registry for application images
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the second tutorial
```

### <a name="base-image-update-scenario"></a>Escenario de actualización de imagen base

Este tutorial le guía a través de un escenario de actualización de imagen base. Este escenario refleja un flujo de trabajo de desarrollo para administrar las imágenes de base en un registro de contenedor privado común al crear imágenes de aplicación en otros registros. Las imágenes de base pueden especificar sistemas operativos y marcos de trabajo comunes utilizados por un equipo o incluso componentes de servicio comunes.

Por ejemplo, los desarrolladores que desarrollan imágenes de aplicación en sus propios registros pueden acceder a un conjunto de imágenes de base que se mantienen en el registro de base común. El registro de base puede estar en otra región o incluso se puede usar la replicación geográfica con él.

El [código de ejemplo][code-sample] incluye dos archivos Dockerfile: una imagen de aplicación y una imagen especificada como su base. En las secciones siguientes creará una tarea de ACR que desencadene automáticamente una compilación de la imagen de aplicación cuando se inserte una nueva versión de la imagen de base en un registro de contenedor de Azure distinto.

* [Dockerfile-app][dockerfile-app]: una pequeña aplicación web de Node.js que representa una página web estática que muestra la versión de Node.js en la que se basa. Se simula la cadena de versión: se muestra el contenido de una variable de entorno, `NODE_VERSION`, que se definió en la imagen base.

* [Dockerfile-base][dockerfile-base]: la imagen que `Dockerfile-app` especifica como su base. Está basada en una imagen de [Node][base-node] e incluye la variable de entorno `NODE_VERSION`.

En las secciones siguientes, va a crear una tarea, actualizar el valor `NODE_VERSION` en el archivo de Docker de la imagen base y, finalmente, a usar ACR Tasks para compilar la imagen base. Cuando ACR Tasks inserta la nueva imagen base en el registro, este desencadena automáticamente una compilación de la imagen de aplicación. Si lo desea, puede ejecutar la imagen de contenedor de la aplicación localmente para ver las diferentes cadenas de versión de las imágenes compiladas.

En este tutorial, ACR Tasks compila e inserta una imagen de contenedor de aplicación especificada en un archivo Dockerfile. Azure Container Registry Tasks también puede ejecutar [tareas de varios pasos](container-registry-tasks-multi-step.md) mediante un archivo YAML que define los pasos para compilar, insertar y, opcionalmente, probar varios contenedores.

## <a name="build-the-base-image"></a>Compilación de la imagen base

Empiece por compilar la imagen de base con una *tarea rápida* de ACR Tasks mediante [az acr build][az-acr-build]. Como se ha descrito en el [primer tutorial](container-registry-tutorial-quick-task.md) de la serie, este proceso no solo sirve para compilar la imagen, si no también para insertarla en el registro de contenedor si la compilación es correcta. En este ejemplo la imagen se inserta en el registro de la imagen de base.

```azurecli-interactive
az acr build --registry $BASE_ACR --image baseimages/node:9-alpine --file Dockerfile-base .
```

## <a name="create-a-task-to-track-the-private-base-image"></a>Creación de una tarea para realizar el seguimiento de la imagen de base privada

A continuación, cree una tarea en el registro de imagen de aplicación con [az acr task create][az-acr-task-create] y habilite una [identidad administrada](container-registry-tasks-authentication-managed-identity.md). Esta identidad administrada se usará en pasos posteriores para que la tarea se autentique con el registro de la imagen de base. 

En este ejemplo se usa una identidad asignada por el sistema, pero puede crear y habilitar una identidad administrada asignada por el usuario para determinados escenarios. Para más detalles, [Autenticación entre registros en una tarea de ACR mediante una identidad administrada por Azure](container-registry-tasks-cross-registry-authentication.md).

```azurecli-interactive
az acr task create \
    --registry $ACR_NAME \
    --name taskhelloworld \
    --image helloworld:{{.Run.ID}} \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git \
    --file Dockerfile-app \
    --git-access-token $GIT_PAT \
    --arg REGISTRY_NAME=$BASE_ACR.azurecr.io \
    --assign-identity
```


Esta tarea es similar a la creada en el [tutorial anterior](container-registry-tutorial-build-task.md). Indica a ACR Tasks que desencadene una compilación de la imagen cuando se inserten confirmaciones en el repositorio especificado por `--context`. Mientras que el archivo de Dockerfile que se usa para compilar la imagen en el tutorial anterior especifica una imagen base pública (`FROM node:9-alpine`), el archivo Dockerfile de esta tarea ([Dockerfile-app][dockerfile-app]) especifica una imagen de base en el registro de imagen de base:

```Dockerfile
FROM ${REGISTRY_NAME}/baseimages/node:9-alpine
```

Esta configuración facilita simular una revisión del marco de trabajo en la imagen base más adelante en este tutorial.

## <a name="give-identity-pull-permissions-to-base-registry"></a>Concesión a la identidad de permisos de extracción del registro de base

Para conceder permisos de identidad administrada de la tarea para extraer imágenes del registro de imagen de base, primero ejecute [az acr task show][az-acr-task-show] para obtener el identificador de la entidad de servicio de la identidad. Después, ejecute [az acr show][az-acr-show] para obtener el identificador de recurso del registro de base:

```azurecli-interactive
# Get service principal ID of the task
principalID=$(az acr task show --name taskhelloworld --registry $ACR_NAME --query identity.principalId --output tsv) 

# Get resource ID of the base registry
baseregID=$(az acr show --name $BASE_ACR --query id --output tsv) 
```
 
Asígnele los permisos de extracción de identidad administrada al registro mediante la ejecución de [az role assignment create][az-role-assignment-create]:

```azurecli-interactive
az role assignment create \
  --assignee $principalID \
  --scope $baseregID --role acrpull 
```

## <a name="add-target-registry-credentials-to-the-task"></a>Incorporación de las credenciales del registro de destino a la tarea

Ejecute [az acr task credential add][az-acr-task-credential-add] para agregar las credenciales a la tarea. Pase el parámetro `--use-identity [system]` para indicar que la identidad administrada asignada por el sistema de la tarea puede acceder a las credenciales.

```azurecli-interactive
az acr task credential add \
  --name taskhelloworld \
  --registry $ACR_NAME \
  --login-server $BASE_ACR.azurecr.io \
  --use-identity [system] 
```

## <a name="manually-run-the-task"></a>Ejecución manual de la tarea

Use [az acr task run][az-acr-task-run] para desencadenar manualmente la tarea y compilar la imagen de aplicación. Este paso es necesario para que la tarea supervise la dependencia de la imagen de la aplicación en la imagen de base.

```azurecli-interactive
az acr task run --registry $ACR_NAME --name taskhelloworld
```

Una vez finalizada la tarea, tome nota del **identificador de ejecución** (por ejemplo, "da6") si desea completar el siguiente paso opcional.

### <a name="optional-run-application-container-locally"></a>Opcional: Ejecutar contenedor de aplicación localmente

Si está trabajando de forma local (no en Cloud Shell), y tiene Docker instalado, ejecute el contenedor para ver la aplicación representada en un explorador web antes de recompilar su imagen base. Si va a usar Cloud Shell, omita esta sección (Cloud Shell no admite `az acr login` ni `docker run`).

En primer lugar, autentíquese en el registro de contenedor con [az acr login][az-acr-login]:

```azurecli
az acr login --name $ACR_NAME
```

Ahora, ejecute el contenedor localmente con `docker run`. Reemplace **\<run-id\>** por el identificador de ejecución de la salida del paso anterior (por ejemplo, "da6"). En este ejemplo el contenedor se denomina `myapp` e incluye el parámetro `--rm` para quitar el contenedor cuando lo detenga.

```bash
docker run -d -p 8080:80 --name myapp --rm $ACR_NAME.azurecr.io/helloworld:<run-id>
```

Vaya a `http://localhost:8080` en el explorador, allí verá el número de versión de Node.js representado en la página web de manera parecida a la siguiente. En un paso posterior, cambiará la versión agregando una "a" a la cadena de versión.

![Captura de pantalla de una aplicación de ejemplo representada en el explorador][base-update-01]

Ejecute el siguiente comando para detener y eliminar el contenedor:

```bash
docker stop myapp
```

## <a name="list-the-builds"></a>Lista de las compilaciones

A continuación, muestre las tareas que ACR Tasks ha completado para el registro mediante el comando [az acr task list-runs][az-acr-task-list-runs]:

```azurecli-interactive
az acr task list-runs --registry $ACR_NAME --output table
```

Si completó el tutorial anterior (y no eliminó el registro), debería ver una salida parecida a la siguiente. Tome nota del número de ejecuciones de tareas y del identificador de ejecución más reciente, para que pueda comparar la salida después de actualizar la imagen base en la sección siguiente.

```console
$ az acr task list-runs --registry $ACR_NAME --output table

RUN ID    TASK            PLATFORM    STATUS     TRIGGER     STARTED               DURATION
--------  --------------  ----------  ---------  ----------  --------------------  ----------
da6       taskhelloworld  Linux       Succeeded  Manual      2018-09-17T23:07:22Z  00:00:38
da5                       Linux       Succeeded  Manual      2018-09-17T23:06:33Z  00:00:31
da4       taskhelloworld  Linux       Succeeded  Git Commit  2018-09-17T23:03:45Z  00:00:44
da3       taskhelloworld  Linux       Succeeded  Manual      2018-09-17T22:55:35Z  00:00:35
da2       taskhelloworld  Linux       Succeeded  Manual      2018-09-17T22:50:59Z  00:00:32
da1                       Linux       Succeeded  Manual      2018-09-17T22:29:59Z  00:00:57
```

## <a name="update-the-base-image"></a>Actualización de la imagen base

Aquí puede simular una revisión de la plataforma en la imagen base. Edite **Dockerfile-base** y agregue una "a" después del número de versión definido en `NODE_VERSION`:

```Dockerfile
ENV NODE_VERSION 9.11.2a
```

Ejecute una tarea rápida para compilar la imagen base modificada. Tome nota del **identificador de ejecución** de la salida.

```azurecli-interactive
az acr build --registry $BASE_ACR --image baseimages/node:9-alpine --file Dockerfile-base .
```

Una vez que haya finalizado la compilación y que ACR Tasks haya insertado la nueva imagen base en el registro, este desencadena una compilación de la imagen de aplicación. Es posible que la tarea que creó anteriormente tarde unos instantes en desencadenar la compilación de la imagen de aplicación, ya que debe detectar la imagen base recién compilada e insertada.

## <a name="list-updated-build"></a>Lista de compilaciones actualizadas

Ahora que ha actualizado la imagen base, vuelva a enumerar las ejecuciones de tareas para compararlas con la lista anterior. Si al principio la salida no es diferente, ejecute periódicamente el comando hasta ver cómo aparece la nueva ejecución de tarea en la lista.

```azurecli-interactive
az acr task list-runs --registry $ACR_NAME --output table
```

La salida es similar a la siguiente. El DESENCADENADOR de la última compilación ejecutada debe ser "Image Update", lo cual indica que la tarea la inició la tarea rápida de la imagen base.

```console
$ az acr task list-runs --registry $ACR_NAME --output table

Run ID    TASK            PLATFORM    STATUS     TRIGGER       STARTED               DURATION
--------  --------------  ----------  ---------  ------------  --------------------  ----------
da8       taskhelloworld  Linux       Succeeded  Image Update  2018-09-17T23:11:50Z  00:00:33
da7                       Linux       Succeeded  Manual        2018-09-17T23:11:27Z  00:00:35
da6       taskhelloworld  Linux       Succeeded  Manual        2018-09-17T23:07:22Z  00:00:38
da5                       Linux       Succeeded  Manual        2018-09-17T23:06:33Z  00:00:31
da4       taskhelloworld  Linux       Succeeded  Git Commit    2018-09-17T23:03:45Z  00:00:44
da3       taskhelloworld  Linux       Succeeded  Manual        2018-09-17T22:55:35Z  00:00:35
da2       taskhelloworld  Linux       Succeeded  Manual        2018-09-17T22:50:59Z  00:00:32
da1                       Linux       Succeeded  Manual        2018-09-17T22:29:59Z  00:00:57
```

Si quiere realizar el siguiente paso opcional y ejecutar el contenedor recién compilado para ver el número de versión actualizado, anote el valor del **identificador de ejecución** de la compilación que desencadenó "Image Update" (en la salida anterior es "da8").

### <a name="optional-run-newly-built-image"></a>Opcional: Ejecución de la imagen recién compilada

Si está trabajando de forma local (no en Cloud Shell), y tiene Docker instalado, ejecute la nueva imagen de aplicación después de terminar su compilación. Sustituya `<run-id>` por el identificador de ejecución que obtuvo en el paso anterior. Si va a usar Cloud Shell, omita esta sección (Cloud Shell no admite `docker run`).

```bash
docker run -d -p 8081:80 --name updatedapp --rm $ACR_NAME.azurecr.io/helloworld:<run-id>
```

Vaya a http://localhost:8081 en el explorador, allí verá el número de versión de Node.js actualizado (con la "a") en la página web:

![Captura de pantalla de una aplicación de ejemplo representada en el explorador][base-update-02]

Es importante observar que ha actualizado la imagen **base** con un nuevo número de versión, pero que también la imagen de **aplicación** recién compilada muestra la nueva versión. ACR Tasks recopiló el cambio en la imagen base y recompiló automáticamente la imagen de aplicación.

Ejecute el siguiente comando para detener y eliminar el contenedor:

```bash
docker stop updatedapp
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a usar una tarea para desencadenar automáticamente compilaciones de imágenes de contenedor cuando se actualiza la imagen base de la imagen. Ahora, pase al siguiente tutorial para aprender a desencadenar tareas según una programación definida.

> [!div class="nextstepaction"]
> [Ejecutar una tarea según una programación](container-registry-tasks-scheduled.md)

<!-- LINKS - External -->
[base-alpine]: https://hub.docker.com/_/alpine/
[base-dotnet]: https://hub.docker.com/r/microsoft/dotnet/
[base-node]: https://hub.docker.com/_/node/
[base-windows]: https://hub.docker.com/r/microsoft/nanoserver/
[code-sample]: https://github.com/Azure-Samples/acr-build-helloworld-node
[dockerfile-app]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-app
[dockerfile-base]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-base

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-update]: /cli/azure/acr/task#az-acr-task-update
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task-show]: /cli/azure/acr/task#az-acr-task-show
[az-acr-task-credential-add]: /cli/azure/acr/task/credential#az-acr-task-credential-add
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-task-list-runs]: /cli/azure/acr/task#az-acr-task-list-runs
[az-acr-task]: /cli/azure/acr#az-acr-task
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create

<!-- IMAGES -->
[base-update-01]: ./media/container-registry-tutorial-base-image-update/base-update-01.png
[base-update-02]: ./media/container-registry-tutorial-base-image-update/base-update-02.png
