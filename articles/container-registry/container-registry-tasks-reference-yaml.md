---
title: 'Referencia de YAML: ACR Tasks'
description: Referencia para definir tareas en YAML para ACR Tasks, como propiedades de tareas, tipos de pasos, propiedades de pasos y variables integradas.
ms.topic: article
ms.date: 07/08/2020
ms.openlocfilehash: 4710afe0d10a81f2a84437a335d3a012f3bac326
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87479785"
---
# <a name="acr-tasks-reference-yaml"></a>Referencia de ACR Tasks: YAML

La definición de tareas de varios pasos en ACR Tasks proporciona un primitivo de proceso centrado en el contenedor enfocado a compilar y probar contenedores, y aplicarles revisiones. En este artículo se tratan los comandos, parámetros, propiedades y sintaxis de los archivos YAML que definen las tareas de varios pasos.

Este artículo contiene la referencia para crear archivos YAML de tarea de varios pasos para ACR Tasks. Para una introducción a ACR Tasks, consulte la [introducción a ACR Tasks](container-registry-tasks-overview.md).

## <a name="acr-taskyaml-file-format"></a>formato de archivo acr-task.yaml

ACR Tasks admite la declaración de tareas de varios pasos en la sintaxis estándar de YAML. Puede definir los pasos de una tarea en un archivo YAML. Luego, puede ejecutar manualmente la tarea si pasa el archivo al comando [az acr run][az-acr-run]. O bien, use el archivo para crear una tarea con [az acr task create][az-acr-task-create] que se desencadena automáticamente en una confirmación de GIT, en una actualización de imagen base o en una programación. Aunque en este artículo se hace referencia a `acr-task.yaml` como el archivo que contiene los pasos, ACR Tasks admite cualquier nombre de archivo válido con una [extensión admitida](#supported-task-filename-extensions).

Los primitivos `acr-task.yaml` de nivel superior son **propiedades de tareas**, **tipos de pasos** y **propiedades de pasos**:

* Las [propiedades de tareas](#task-properties) se aplican a todos los pasos a lo largo de la ejecución de tareas. Existen varias propiedades de tareas globales, como:
  * `version`
  * `stepTimeout`
  * `workingDirectory`
* Los [tipos de pasos de tareas](#task-step-types) representan los tipos de acciones que se pueden realizar en una tarea. Hay tres tipos de pasos:
  * `build`
  * `push`
  * `cmd`
* Las [propiedades de pasos de tareas](#task-step-properties) son parámetros que se aplican a un paso individual. Hay varias propiedades de pasos, como:
  * `startDelay`
  * `timeout`
  * `when`
  * y muchas más.

Le sigue el formato base de un archivo `acr-task.yaml`, incluidas algunas propiedades de pasos comunes. Aunque no es una representación exhaustiva de todo el uso de propiedades de pasos o tipos de pasos, proporciona una información general rápida del formato de archivo básico.

```yml
version: # acr-task.yaml format version.
stepTimeout: # Seconds each step may take.
steps: # A collection of image or container actions.
  - build: # Equivalent to "docker build," but in a multi-tenant environment
  - push: # Push a newly built or retagged image to a registry.
    when: # Step property that defines either parallel or dependent step execution.
  - cmd: # Executes a container, supports specifying an [ENTRYPOINT] and parameters.
    startDelay: # Step property that specifies the number of seconds to wait before starting execution.
```

### <a name="supported-task-filename-extensions"></a>Extensiones de nombre de archivo de tareas admitidas

ACR Tasks ha reservado varias extensiones de nombre de archivo, como `.yaml`, que se procesarán como un archivo de tarea. Cualquier extensión que *no* se encuentre en la siguiente lista se considera un archivo Dockerfile en ACR Tasks: .yaml, .yml, .toml, .json, .sh, .bash, .zsh,. ps1, .ps, .cmd, .bat, TS, .js, .php, .py,. RB, .lua.

YAML es el único formato de archivo compatible actualmente con ACR Tasks. Las demás extensiones de nombre de archivo están reservadas para posible compatibilidad futura.

## <a name="run-the-sample-tasks"></a>Ejecución de las tareas de ejemplo

Son varios los archivos de tareas de ejemplo a los que se hace referencia en las siguientes secciones de este artículo. Las tareas de ejemplo están en un repositorio público de GitHub, [Azure-Samples/acr-tasks][acr-tasks]. Puede ejecutarlas con el comando [az acr run][az-acr-run] de la CLI de Azure. Los comandos de ejemplo son parecidos a:

```azurecli
az acr run -f build-push-hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

En el formato de los comandos de ejemplo se da por hecho que ha configurado un registro de forma predeterminada en la CLI de Azure, por lo que se omite el parámetro `--registry`. Para configurar un registro predeterminado, use el comando [az configure][az-configure] con el parámetro `--defaults`, que acepta un valor `acr=REGISTRY_NAME`.

Por ejemplo, para configurar la CLI de Azure con un registro predeterminado llamado "myregistry":

```azurecli
az configure --defaults acr=myregistry
```

## <a name="task-properties"></a>Propiedades de tareas

Las propiedades de tareas suelen aparecer en la parte superior de un archivo `acr-task.yaml` y son propiedades globales que se aplican a lo largo de la ejecución completa de los pasos de una tarea. Algunas de estas propiedades globales se pueden reemplazar dentro de un paso individual.

| Propiedad | Tipo | Opcional | Descripción | Invalidación admitida | Valor predeterminado |
| -------- | ---- | -------- | ----------- | ------------------ | ------------- |
| `version` | string | Sí | La versión del archivo `acr-task.yaml` analizada por el servicio ACR Tasks. Si bien ACR Tasks se esfuerza por mantener la compatibilidad con versiones anteriores, este valor permite que ACR Tasks mantenga la compatibilidad dentro de una versión definida. Si no se especifica, se establece de manera predeterminada en la versión más reciente. | No | None |
| `stepTimeout` | int (segundos) | Sí | El número máximo de segundos que se puede ejecutar un paso. Si la propiedad se especifica en una tarea, establece la propiedad `timeout` predeterminada de todos los pasos. Si la propiedad `timeout` se especifica en un paso, invalida la propiedad que la tarea proporciona. | Sí | 600 (10 minutos) |
| `workingDirectory` | string | Sí | El directorio de trabajo del contenedor durante el tiempo de ejecución. Si la propiedad se especifica en una tarea, establece la propiedad `workingDirectory` predeterminada de todos los pasos. Si se especifica en un paso, invalida la propiedad que la tarea proporciona. | Sí | `/workspace` |
| `env` | [string, string, ...] | Sí |  Matriz de cadenas en formato `key=value` que define las variables de entorno de la tarea. Si la propiedad se especifica en una tarea, establece la propiedad `env` predeterminada de todos los pasos. Si se especifica en un paso, invalida las variables de entorno heredadas de la tarea. | Sí | None |
| `secrets` | [secret, secret, ...] | Sí | Matriz de objetos de [secreto](#secret). | No | None |
| `networks` | [network, network, ...] | Sí | Matriz de objetos de [red](#network). | No | None |
| `volumes` | [volume, volume, ...] | Sí | Matriz de objetos [volume](#volume). Especifica volúmenes con contenido de origen que se van a montar en un paso. | No | None |

### <a name="secret"></a>secret

El objeto de secreto tiene estas propiedades.

| Propiedad | Tipo | Opcional | Descripción | Valor predeterminado |
| -------- | ---- | -------- | ----------- | ------- |
| `id` | string | No | El identificador del secreto. | None |
| `keyvault` | string | Sí | La dirección URL del secreto de Azure Key Vault. | None |
| `clientID` | string | Sí | El identificador de cliente de la [identidad administrada asignada por el usuario](container-registry-tasks-authentication-managed-identity.md) para los recursos de Azure. | None |

### <a name="network"></a>red

El objeto de red tiene estas propiedades.

| Propiedad | Tipo | Opcional | Descripción | Valor predeterminado |
| -------- | ---- | -------- | ----------- | ------- | 
| `name` | string | No | El nombre de la red. | None |
| `driver` | string | Sí | El controlador para administrar la red. | None |
| `ipv6` | bool | Sí | Si la red IPv6 está habilitada. | `false` |
| `skipCreation` | bool | Sí | Si se omite la creación de la red. | `false` |
| `isDefault` | bool | Sí | Si la red es una red predeterminada proporcionada con Azure Container Registry. | `false` |

### <a name="volume"></a>volumen

El objeto volume tiene las siguientes propiedades.

| Propiedad | Tipo | Opcional | Descripción | Valor predeterminado |
| -------- | ---- | -------- | ----------- | ------- | 
| `name` | string | No | Nombre del volumen que se va a montar. Solo puede contener caracteres alfanuméricos, "-" y "_". | None |
| `secret` | map[string]string | No | Cada clave del mapa es el nombre de un archivo creado y rellenado en el volumen. Cada valor es la versión de cadena del secreto. Los valores del secreto deben estar codificados con Base64. | None |

## <a name="task-step-types"></a>Tipos de pasos de tareas

ACR Tasks admite tres tipos de pasos. Cada tipo de paso admite varias propiedades, que se detallan en la sección para cada tipo de paso.

| Tipo de paso | Descripción |
| --------- | ----------- |
| [`build`](#build) | Compila una imagen de contenedor mediante la conocida sintaxis `docker build`. |
| [`push`](#push) | Ejecuta una acción `docker push` de imágenes recién compiladas o etiquetadas de nuevo en un registro de contenedor. Se admite Azure Container Registry, otros registros privados y Docker Hub público. |
| [`cmd`](#cmd) | Ejecuta un contenedor como un comando, con parámetros que se pasan al elemento `[ENTRYPOINT]` del contenedor. El tipo de paso `cmd` admite parámetros como `env`, `detach` y otras opciones de comando `docker run` conocidas, que permiten pruebas unitarias y funcionales con la ejecución simultánea del contenedor. |

## <a name="build"></a>build

Compila una imagen de contenedor. El tipo de paso `build` representa un medio multiinquilino seguro de ejecutar `docker build` en la nube como un primitivo de primera clase.

### <a name="syntax-build"></a>Sintaxis: build

```yml
version: v1.1.0
steps:
  - [build]: -t [imageName]:[tag] -f [Dockerfile] [context]
    [property]: [value]
```

El tipo de paso `build` admite los parámetros de la tabla siguiente. El tipo de paso `build` también es compatible con todas las opciones del comando [compilación de docker](https://docs.docker.com/engine/reference/commandline/build/), como `--build-arg` para establecer las variables en tiempo de compilación.

| Parámetro | Descripción | Opcional |
| --------- | ----------- | :-------: |
| `-t` &#124; `--image` | Define el elemento completo `image:tag` de la imagen compilada.<br /><br />Como se pueden usar imágenes para las validaciones de tareas internas, no todas las imágenes requieren la acción `push` en un registro. Sin embargo, para crear una instancia de una imagen dentro de una ejecución de tareas, la imagen necesita un nombre al que hacer referencia.<br /><br />A diferencia de `az acr build`, la ejecución de ACR Tasks no proporciona un comportamiento de inserción predeterminado. Con ACR Tasks, en el escenario predeterminado se da por hecho la posibilidad de compilar, validar y luego insertar una imagen. Consulte [push](#push) para ver como insertar opcionalmente imágenes compiladas. | Sí |
| `-f` &#124; `--file` | Especifica el archivo Dockerfile que se pasa a `docker build`. Si no se especifica, se da por hecho el archivo Dockerfile predeterminado en la raíz del contexto. Para especificar un archivo Dockerfile, pase el nombre de archivo relativo a la raíz del contexto. | Sí |
| `context` | El directorio raíz pasado a `docker build`. El directorio raíz de cada tarea se establece en un directorio [workingDirectory](#task-step-properties) compartido, e incluye la raíz del directorio clonado de Git asociado. | No |

### <a name="properties-build"></a>Propiedades: build

El tipo de paso `build` admite las siguientes propiedades. Encuentre detalles de estas propiedades en la sección [Propiedades de pasos de tareas](#task-step-properties) de este artículo.

| Propiedades | Tipo | Obligatorio |
| -------- | ---- | -------- |
| `detach` | bool | Opcional |
| `disableWorkingDirectoryOverride` | bool | Opcional |
| `entryPoint` | string | Opcional |
| `env` | [string, string, ...] | Opcional |
| `expose` | [string, string, ...] | Opcional |
| `id` | string | Opcional |
| `ignoreErrors` | bool | Opcional |
| `isolation` | string | Opcional |
| `keep` | bool | Opcional |
| `network` | object | Opcional |
| `ports` | [string, string, ...] | Opcional |
| `pull` | bool | Opcional |
| `repeat` | int | Opcional |
| `retries` | int | Opcional |
| `retryDelay` | int (segundos) | Opcional |
| `secret` | object | Opcional |
| `startDelay` | int (segundos) | Opcional |
| `timeout` | int (segundos) | Opcional |
| `volumeMount` | object | Opcional |
| `when` | [string, string, ...] | Opcional |
| `workingDirectory` | string | Opcional |

### <a name="examples-build"></a>Ejemplos: build

#### <a name="build-image---context-in-root"></a>Imagen de compilación: contexto en raíz

```azurecli
az acr run -f build-hello-world.yaml https://github.com/AzureCR/acr-tasks-sample.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/build-hello-world.yaml -->
[!code-yml[task](~/acr-tasks/build-hello-world.yaml)]

#### <a name="build-image---context-in-subdirectory"></a>Imagen de compilación: contexto en el subdirectorio

```yml
version: v1.1.0
steps:
  - build: -t $Registry/hello-world -f hello-world.dockerfile ./subDirectory
```

## <a name="push"></a>push

Inserta una o varias imágenes compiladas o etiquetadas de nuevo en un registro de contenedor. Admite la inserción en registros privados como Azure Container Registry o Docker Hub público.

### <a name="syntax-push"></a>Sintaxis: push

El tipo de paso `push` admite una colección de imágenes. La sintaxis de la colección YAML admite formatos alineados y anidados. La inserción de una sola imagen se representa normalmente mediante sintaxis alineada:

```yml
version: v1.1.0
steps:
  # Inline YAML collection syntax
  - push: ["$Registry/hello-world:$ID"]
```

Para una mejor legibilidad, use sintaxis anidada al insertar varias imágenes:

```yml
version: v1.1.0
steps:
  # Nested YAML collection syntax
  - push:
    - $Registry/hello-world:$ID
    - $Registry/hello-world:latest
```

### <a name="properties-push"></a>Propiedades: push

El tipo de paso `push` admite las siguientes propiedades. Encuentre detalles de estas propiedades en la sección [Propiedades de pasos de tareas](#task-step-properties) de este artículo.

| Propiedad | Tipo | Obligatorio |
| -------- | ---- | -------- |
| `env` | [string, string, ...] | Opcional |
| `id` | string | Opcional |
| `ignoreErrors` | bool | Opcional |
| `startDelay` | int (segundos) | Opcional |
| `timeout` | int (segundos) | Opcional |
| `when` | [string, string, ...] | Opcional |

### <a name="examples-push"></a>Ejemplos: push

#### <a name="push-multiple-images"></a>Insertar varias imágenes

```azurecli
az acr run -f build-push-hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/build-push-hello-world.yaml -->
[!code-yml[task](~/acr-tasks/build-push-hello-world.yaml)]

#### <a name="build-push-and-run"></a>Compilar, insertar y ejecutar

```azurecli
az acr run -f build-run-hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/build-run-hello-world.yaml -->
[!code-yml[task](~/acr-tasks/build-run-hello-world.yaml)]

## <a name="cmd"></a>cmd

El tipo de paso `cmd` ejecuta un contenedor.

### <a name="syntax-cmd"></a>Sintaxis: cmd

```yml
version: v1.1.0
steps:
  - [cmd]: [containerImage]:[tag (optional)] [cmdParameters to the image]
```

### <a name="properties-cmd"></a>Propiedades: cmd

El tipo de paso `cmd` admite las siguientes propiedades:

| Propiedad | Tipo | Obligatorio |
| -------- | ---- | -------- |
| `detach` | bool | Opcional |
| `disableWorkingDirectoryOverride` | bool | Opcional |
| `entryPoint` | string | Opcional |
| `env` | [string, string, ...] | Opcional |
| `expose` | [string, string, ...] | Opcional |
| `id` | string | Opcional |
| `ignoreErrors` | bool | Opcional |
| `isolation` | string | Opcional |
| `keep` | bool | Opcional |
| `network` | object | Opcional |
| `ports` | [string, string, ...] | Opcional |
| `pull` | bool | Opcional |
| `repeat` | int | Opcional |
| `retries` | int | Opcional |
| `retryDelay` | int (segundos) | Opcional |
| `secret` | object | Opcional |
| `startDelay` | int (segundos) | Opcional |
| `timeout` | int (segundos) | Opcional |
| `volumeMount` | object | Opcional |
| `when` | [string, string, ...] | Opcional |
| `workingDirectory` | string | Opcional |

Puede encontrar detalles de estas propiedades en la sección [Propiedades de pasos de tareas](#task-step-properties) de este artículo.

### <a name="examples-cmd"></a>Ejemplos: cmd

#### <a name="run-hello-world-image"></a>Ejecutar la imagen hello world

Este comando ejecuta el archivo de tarea `hello-world.yaml`, que hace referencia a la imagen [hello world](https://hub.docker.com/_/hello-world/) de Docker Hub.

```azurecli
az acr run -f hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.yaml -->
[!code-yml[task](~/acr-tasks/hello-world.yaml)]

#### <a name="run-bash-image-and-echo-hello-world"></a>Ejecutar imagen de Bash y repetir "hello world"

Este comando ejecuta el archivo de tarea `bash-echo.yaml`, que hace referencia a la imagen [bash](https://hub.docker.com/_/bash/) de Docker Hub.

```azurecli
az acr run -f bash-echo.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/bash-echo.yaml -->
[!code-yml[task](~/acr-tasks/bash-echo.yaml)]

#### <a name="run-specific-bash-image-tag"></a>Ejecutar etiqueta de imagen de Bash específica

Para ejecutar una versión de imagen concreta, especifique la etiqueta en `cmd`.

Este comando ejecuta el archivo de tarea `bash-echo-3.yaml`, que hace referencia a la imagen [bash:3.0](https://hub.docker.com/_/bash/) de Docker Hub.

```azurecli
az acr run -f bash-echo-3.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/bash-echo-3.yaml -->
[!code-yml[task](~/acr-tasks/bash-echo-3.yaml)]

#### <a name="run-custom-images"></a>Ejecutar imágenes personalizadas

El tipo de paso `cmd` hace referencia a las imágenes mediante el formato estándar `docker run`. Se da por hecho que las imágenes que no comienzan con un registro se originan en docker.io. El ejemplo anterior se podría representar igualmente como:

```yml
version: v1.1.0
steps:
  - cmd: docker.io/bash:3.0 echo hello world
```

Mediante la convención de referencia de imágenes `docker run` estándar, `cmd` puede ejecutar imágenes de cualquier registro privado o en Docker Hub público. Si va a hacer referencia a imágenes que se encuentran en el mismo registro donde se ejecutan las tareas de ACR, no es necesario especificar credenciales de registro.

* Ejecute una imagen proveniente de Azure Container Registry. En el ejemplo siguiente se da por supuesto que tiene un registro llamado `myregistry` y una imagen personalizada llamada `myimage:mytag`.

    ```yml
    version: v1.1.0
    steps:
        - cmd: myregistry.azurecr.io/myimage:mytag
    ```

* Generalice la referencia del registro con una variable de ejecución o un alias.

    En lugar de codificar de forma rígida el nombre del registro en un archivo `acr-task.yaml`, puede hacer que sea más fácil de trasladar mediante una [variable de ejecución](#run-variables) o [alias](#aliases). La variable `Run.Registry` o el alias `$Registry` se expanden en tiempo de ejecución al nombre del registro en el que se ejecuta la tarea.

    Por ejemplo, para generalizar la tarea anterior de modo que funcione en cualquier registro de contenedor de Azure, haga referencia a la variable $Registry en el nombre de imagen:

    ```yml
    version: v1.1.0
    steps:
      - cmd: $Registry/myimage:mytag
    ```

#### <a name="access-secret-volumes"></a>Obtener acceso a volúmenes secretos

La propiedad `volumes` permite que los volúmenes y el contenido de sus secretos se especifiquen para los pasos `build` y `cmd` de una tarea. En cada paso, una propiedad `volumeMounts` opcional muestra los volúmenes y las rutas de acceso al contenedor que se van a montar en el contenedor en ese paso. Los secretos se proporcionan como archivos en la ruta de acceso de montaje de cada volumen.

Ejecute una tarea y monte dos secretos en un paso: uno almacenado en un almacén de claves y el otro especificado en la línea de comandos:

```azurecli
az acr run -f mounts-secrets.yaml --set-secret mysecret=abcdefg123456 https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/mounts-secrets.yaml -->
<!-- [!code-yml[task](~/acr-tasks/mounts-secrets.yaml)] -->

```yml
# This template demonstrates mounting a custom volume into a container at a CMD step
secrets:
  - id: sampleSecret
    keyvault: https://myacbvault2.vault.azure.net/secrets/SampleSecret

volumes:
  - name: mysecrets
    secret:
      mysecret1: {{.Secrets.sampleSecret | b64enc}}
      mysecret2: {{.Values.mysecret | b64enc}}

steps:
  - cmd: bash cat /run/test/mysecret1 /run/test/mysecret2
    volumeMounts:
      - name: mysecrets
        mountPath: /run/test
```

## <a name="task-step-properties"></a>Propiedades de pasos de tareas

Cada tipo de paso admite varias propiedades adecuadas para su tipo. En la tabla siguiente se definen todas las propiedades de pasos disponibles. No todos los tipos de pasos admiten todas las propiedades. Para ver cuál de estas propiedades está disponible para cada tipo de paso, consulte las acciones de referencia de tipos de pasos [cmd](#cmd), [build](#build) y [push](#push).

| Propiedad | Tipo | Opcional | Descripción | Valor predeterminado |
| -------- | ---- | -------- | ----------- | ------- |
| `detach` | bool | Sí | Si el contenedor se debe desasociar cuando se ejecuta. | `false` |
| `disableWorkingDirectoryOverride` | bool | Sí | Si se deshabilita la funcionalidad de invalidación de `workingDirectory`. Use en combinación con `workingDirectory` para tener el control total sobre el directorio de trabajo del contenedor. | `false` |
| `entryPoint` | string | Sí | Invalida el elemento `[ENTRYPOINT]` del contenedor de un paso. | None |
| `env` | [string, string, ...] | Sí | Matriz de cadenas en formato `key=value` que define las variables de entorno del paso. | None |
| `expose` | [string, string, ...] | Sí | La matriz de los puertos que están expuestos desde el contenedor. |  None |
| [`id`](#example-id) | string | Sí | Identifica el paso de forma única dentro de la tarea. Otros pasos de la tarea pueden hacer referencia al elemento `id` del paso, por ejemplo, para la comprobación de dependencias con `when`.<br /><br />`id` también es el nombre del contenedor en ejecución. Los procesos que se ejecutan en otros contenedores de la tarea pueden hacer referencia a `id` como su nombre de host DNS, o para acceder a él con registros de docker [id], por ejemplo. | `acb_step_%d`, donde `%d` es el índice basado en 0 del orden descendente de los pasos en el archivo YAML. |
| `ignoreErrors` | bool | Sí | Si el paso se debe marcar como correcto, independientemente de que hubo un error en la ejecución de un contenedor. | `false` |
| `isolation` | string | Sí | El nivel de aislamiento del contenedor. | `default` |
| `keep` | bool | Sí | Si se debe mantener el contenedor del paso después de la ejecución. | `false` |
| `network` | object | Sí | Identifica una red en el que se ejecuta el contenedor. | None |
| `ports` | [string, string, ...] | Sí | Matriz de los puertos que se publican desde el contenedor al host. |  None |
| `pull` | bool | Sí | Si se debe forzar una extracción del contenedor antes de ejecutarlo para evitar cualquier comportamiento de almacenamiento en caché. | `false` |
| `privileged` | bool | Sí | Si se debe ejecutar el contenedor en modo con privilegios. | `false` |
| `repeat` | int | Sí | El número de reintentos para repetir la ejecución de un contenedor. | 0 |
| `retries` | int | Sí | El número de reintentos si se produce un error en la ejecución de un contenedor. Solo se realiza un reintento si el código de salida de un contenedor es distinto de cero. | 0 |
| `retryDelay` | int (segundos) | Sí | El retraso en segundos entre los reintentos de la ejecución de un contenedor. | 0 |
| `secret` | object | Sí | Identifica un secreto de Azure Key Vault o una [identidad administrada para los recursos de Azure](container-registry-tasks-authentication-managed-identity.md). | None |
| `startDelay` | int (segundos) | Sí | Número de segundos para retrasar la ejecución de un contenedor. | 0 |
| `timeout` | int (segundos) | Sí | Número máximo de segundos que se puede ejecutar un paso antes de terminar. | 600 |
| [`when`](#example-when) | [string, string, ...] | Sí | Configura la dependencia de un paso de uno o varios pasos dentro de la tarea. | None |
| `user` | string | Sí | El nombre de usuario o UID de un contenedor. | None |
| `volumeMounts` | object | No | Matriz de objetos [volumeMount](#volumemount). | None |
| `workingDirectory` | string | Sí | Establece el directorio de trabajo de un paso. De forma predeterminada, ACR Tasks crea un directorio raíz como directorio de trabajo. Sin embargo, si la compilación tiene varios pasos, los pasos anteriores pueden compartir artefactos con los pasos posteriores mediante la especificación del mismo directorio de trabajo. | `/workspace` |

### <a name="volumemount"></a>volumeMount

El objeto volumeMount tiene estas propiedades.

| Propiedad | Tipo | Opcional | Descripción | Valor predeterminado |
| -------- | ---- | -------- | ----------- | ------- | 
| `name` | string | No | Nombre del volumen que se va a montar. Debe coincidir exactamente con el nombre de una propiedad `volumes`. | None |
| `mountPath`   | string | no | Ruta de acceso absoluta para montar archivos en el contenedor.  | None |

### <a name="examples-task-step-properties"></a>Ejemplos: Propiedades de pasos de tareas

#### <a name="example-id"></a>Ejemplo: id

Compila dos imágenes mediante la creación de una instancia de una imagen de prueba funcional. Cada paso se identifica mediante un valor de `id` único al que otros pasos de la tarea hacen referencia en su propiedad `when`.

```azurecli
az acr run -f when-parallel-dependent.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-parallel-dependent.yaml -->
[!code-yml[task](~/acr-tasks/when-parallel-dependent.yaml)]

#### <a name="example-when"></a>Ejemplo: when

La propiedad `when` especifica la dependencia de un paso de otros pasos dentro de la tarea. Admite dos valores de parámetro:

* `when: ["-"]`: no indica ninguna dependencia de otros pasos. Un paso que especifica `when: ["-"]` comenzará inmediatamente la ejecución y permitirá la ejecución de pasos simultáneos.
* `when: ["id1", "id2"]`: indica que el paso depende de los pasos con `id` "id1" e `id` "id2". Este paso no se ejecuta hasta que los pasos "id1" e "id2" se han completado.

Si `when` no se especifica en un paso, este paso depende de la finalización del paso anterior en el archivo `acr-task.yaml`.

Ejecución secuencial de pasos sin `when`:

```azurecli
az acr run -f when-sequential-default.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-sequential-default.yaml -->
[!code-yml[task](~/acr-tasks/when-sequential-default.yaml)]

Ejecución secuencial de pasos con `when`:

```azurecli
az acr run -f when-sequential-id.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-sequential-id.yaml -->
[!code-yml[task](~/acr-tasks/when-sequential-id.yaml)]

Compilación de imágenes paralelas:

```azurecli
az acr run -f when-parallel.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-parallel.yaml -->
[!code-yml[task](~/acr-tasks/when-parallel.yaml)]

Compilación de imágenes paralelas y pruebas de dependencia:

```azurecli
az acr run -f when-parallel-dependent.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-parallel-dependent.yaml -->
[!code-yml[task](~/acr-tasks/when-parallel-dependent.yaml)]

## <a name="run-variables"></a>Variables de ejecución

ACR Tasks incluye un conjunto predeterminado de variables que están disponibles para los pasos de la tarea cuando se ejecutan. Se puede acceder a estas variables con el formato `{{.Run.VariableName}}`, donde `VariableName` es uno de los siguientes:

* `Run.ID`
* `Run.SharedVolume`
* `Run.Registry`
* `Run.RegistryName`
* `Run.Date`
* `Run.OS`
* `Run.Architecture`
* `Run.Commit`
* `Run.Branch`
* `Run.TaskName`

Normalmente, los nombres de variable se explican por sí solos. A continuación se indican los detalles de las variables de uso frecuente. A partir de la versión de YAML `v1.1.0`, puede usar un [alias de tarea](#aliases) abreviado predefinido en el lugar de la mayoría de las variables de ejecución. Por ejemplo, en lugar de `{{.Run.Registry}}`, utilice el alias `$Registry`.

### <a name="runid"></a>Run.ID

Cada ejecución, mediante `az acr run`, o la ejecución basada en desencadenador de las tareas creadas por medio de `az acr task create`, tiene un identificador único. El identificador representa la ejecución actualmente en marcha.

Se usa normalmente para etiquetar de forma única una imagen:

```yml
version: v1.1.0
steps:
    - build: -t $Registry/hello-world:$ID .
```

### <a name="runregistry"></a>Run.Registry

El nombre completo de servidor del registro. Se usa normalmente para hacer referencia genérica al registro donde se ejecuta la tarea.

```yml
version: v1.1.0
steps:
  - build: -t $Registry/hello-world:$ID .
```

### <a name="runregistryname"></a>Run.RegistryName

Nombre del registro de contenedor. Se suele usar en pasos de tarea que no requieren un nombre de servidor completo. Por ejemplo, pasos de `cmd` que ejecutan comandos de la CLI de Azure en los registros.

```yml
version 1.1.0
steps:
# List repositories in registry
- cmd: az login --identity
- cmd: az acr repository list --name $RegistryName
```

### <a name="rundate"></a>Run.Date

La hora UTC actual a la que comenzó la ejecución.

### <a name="runcommit"></a>Run.Commit

Para una tarea desencadenada por una confirmación en un repositorio de GitHub, el identificador de la confirmación.

### <a name="runbranch"></a>Run.Branch

Para una tarea desencadenada por una confirmación en un repositorio de GitHub, el nombre de la rama.

## <a name="aliases"></a>Alias

A partir de la versión `v1.1.0`, ACR Tasks admite alias que están disponibles para los pasos de la tarea cuando se ejecutan. Los alias son similares en concepto a los alias (métodos abreviados de comandos) que se admiten en Bash y otros shell de comandos. 

Con un alias, puede iniciar cualquier comando o grupo de comandos (incluidas las opciones y los nombres de archivo) al escribir una sola palabra.

ACR Tasks admite varios alias predefinidos y también los alias personalizados que cree.

### <a name="predefined-aliases"></a>Alias predefinidos

Los siguientes alias de tarea están disponibles para usarse en lugar de [variables de ejecución](#run-variables):

| Alias | Variable de ejecución |
| ----- | ------------ |
| `ID` | `Run.ID` |
| `SharedVolume` | `Run.SharedVolume` |
| `Registry` | `Run.Registry` |
| `RegistryName` | `Run.RegistryName` |
| `Date` | `Run.Date` |
| `OS` | `Run.OS` |
| `Architecture` | `Run.Architecture` |
| `Commit` | `Run.Commit` |
| `Branch` | `Run.Branch` |

En los pasos de la tarea, anteponga la directiva `$` a un alias, como en este ejemplo:

```yml
version: v1.1.0
steps:
  - build: -t $Registry/hello-world:$ID -f hello-world.dockerfile .
```

### <a name="image-aliases"></a>Alias de imagen

Cada uno de los siguientes alias apunta a una imagen estable en Microsoft Container Registry (MCR). Puede hacer referencia a cada uno de ellas en la sección `cmd` de un archivo de tarea sin usar una directiva.

| Alias | Imagen |
| ----- | ----- |
| `acr` | `mcr.microsoft.com/acr/acr-cli:0.1` |
| `az` | `mcr.microsoft.com/acr/azure-cli:a80af84` |
| `bash` | `mcr.microsoft.com/acr/bash:a80af84` |
| `curl` | `mcr.microsoft.com/acr/curl:a80af84` |

En la tarea de ejemplo siguiente se usan varios alias para [purgar](container-registry-auto-purge.md) las etiquetas de imagen con una antigüedad mayor a siete días en el repositorio `samples/hello-world` del registro de ejecución:

```yml
version: v1.1.0
steps:
  - cmd: acr tag list --registry $RegistryName --repository samples/hello-world
  - cmd: acr purge --registry $RegistryName --filter samples/hello-world:.* --ago 7d
```

### <a name="custom-alias"></a>Alias personalizados

Defina un alias personalizado en el archivo YAML y úselo tal y como se muestra en el ejemplo siguiente. Un alias solo puede contener caracteres alfanuméricos. La directiva predeterminada para expandir un alias es el carácter `$`.

```yml
version: v1.1.0
alias:
  values:
    repo: myrepo
steps:
  - build: -t $Registry/$repo/hello-world:$ID -f Dockerfile .
```

Puede vincularse a un archivo YAML local o remoto para las definiciones de alias personalizadas. En el ejemplo siguiente se vincula a un archivo YAML en Azure Blob Storage:

```yml
version: v1.1.0
alias:
  src:  # link to local or remote custom alias files
    - 'https://link/to/blob/remoteAliases.yml?readSasToken'
[...]
```

## <a name="next-steps"></a>Pasos siguientes

Para información general sobre las tareas de varios pasos, consulte [Ejecución de tareas de varios pasos de compilación, prueba y aplicación de revisiones en ACR Tasks](container-registry-tasks-multi-step.md).

Para compilaciones paso a paso, consulte la [Introducción a ACR Tasks](container-registry-tasks-overview.md).



<!-- IMAGES -->

<!-- LINKS - External -->
[acr-tasks]: https://github.com/Azure-Samples/acr-tasks

<!-- LINKS - Internal -->
[az-acr-run]: /cli/azure/acr#az-acr-run
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-configure]: /cli/azure/reference-index#az-configure
