---
title: 'Contenedores de Docker: LUIS'
titleSuffix: Azure Cognitive Services
description: El contenedor de LUIS carga la aplicación entrenada o publicada en un contenedor de Docker y proporciona acceso a las predicciones de consulta de los puntos de conexión de la API del contenedor.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 6a2208fac98d3cd8e4ddcea887d9b8cf30fb6482
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86524512"
---
# <a name="install-and-run-luis-docker-containers"></a>Instalar y ejecutar contenedores de docker de LUIS

El contenedor Language Understanding (LUIS) carga el modelo de Language Understanding entrenado o publicado. Como [aplicación de LUIS](https://www.luis.ai), el contenedor de Docker proporciona acceso a las predicciones de consulta desde los puntos de conexión de API del contenedor. Puede recopilar registros de consultas del contenedor y cargarlos de nuevo en la aplicación de Language Understanding para mejorar la precisión de predicción de la aplicación.

En el siguiente vídeo, se explica cómo se utiliza este contenedor.

[![Demostración de contenedores de Cognitive Services](./media/luis-container-how-to/luis-containers-demo-video-still.png)](https://aka.ms/luis-container-demo)

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para ejecutar el contenedor de LUIS, tenga en cuenta los siguientes requisitos previos:

|Obligatorio|Propósito|
|--|--|
|Motor de Docker| Necesita que el motor de Docker esté instalado en un [equipo host](#the-host-computer). Docker dispone de paquetes que configuran el entorno de Docker en [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) y [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Docker debe configurarse para permitir que los contenedores se conecten con Azure y envíen datos de facturación a dicho servicio. <br><br> **En Windows**, Docker también debe estar configurado de forma que admita los contenedores de Linux.<br><br>|
|Conocimientos sobre Docker | Debe tener conocimientos básicos sobre los conceptos de Docker, como los registros, los repositorios, los contenedores y las imágenes de contenedor, así como conocer los comandos `docker` básicos.|
|Recurso de Azure `Cognitive Services` y archivo de [aplicación empaquetada](luis-how-to-start-new-app.md) de LUIS |Para poder usar el contenedor, debe tener:<br><br>* Un recurso de Azure _Cognitive Services_ y la clave de facturación asociada del URI del punto de conexión de facturación. Ambos valores están disponibles en las páginas de introducción y claves del recurso y son necesarios para iniciar el contenedor. <br>* Una aplicación entrenada o publicada como una entrada montada en el contenedor junto con su identificador de aplicación asociado. Puede obtener el archivo empaquetado en el portal de LUIS o las API de creación. Si va a obtener la aplicación empaquetada de LUIS desde las [API de creación](#authoring-apis-for-package-file), también necesitará su _clave de creación_.<br><br>Estos requisitos se usan para pasar argumentos de la línea de comandos a las siguientes variables:<br><br>**{AUTHORING_KEY}** : esta clave se usa para obtener la aplicación empaquetada del servicio LUIS en la nube y cargar los registros de consultas en la nube. El formato es `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`.<br><br>**{APP_ID}** : este identificador se usa para seleccionar la aplicación. El formato es `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.<br><br>**{API_KEY}** : Esta clave se usa para iniciar el contenedor. Puede encontrar la clave de punto de conexión en dos lugares. El primero de ellos es Azure Portal, en la lista de claves del recurso de _Cognitive Services_. La clave de punto de conexión también está disponible en el portal de LUIS, en la página de configuración de puntos de conexión y claves. No utilice la clave de inicio.<br><br>**{ENDPOINT_URI}** : el punto de conexión tal como se proporciona en la página Información general.<br><br>El propósito de la [clave de creación y la clave de punto de conexión](luis-limits.md#key-limits) es diferente. No deben utilizarse indistintamente. |

[!INCLUDE [Gathering required container parameters](../containers/includes/container-gathering-required-parameters.md)]

### <a name="authoring-apis-for-package-file"></a>API de creación del archivo de paquete

API de creación de las aplicaciones empaquetadas:

* [API del paquete publicado](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagepublishedapplicationasgzip)
* [API del paquete no publicado, solo entrenado](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagetrainedapplicationasgzip)

### <a name="the-host-computer"></a>El equipo host

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Recomendaciones y requisitos del contenedor

En la tabla siguiente se muestran los valores mínimos y los recomendados para el host de contenedor. Los requisitos pueden variar en función del volumen de tráfico.

|Contenedor| Mínima | Recomendado | TPS<br>(mínimo, máximo)|
|-----------|---------|-------------|--|
|LUIS|1 núcleo, 2 GB de memoria|1 núcleos, 4 GB de memoria|20, 40|

* Cada núcleo debe ser de 2,6 gigahercios (GHz) como mínimo.
* TPS: transacciones por segundo

El núcleo y la memoria se corresponden con los valores de `--cpus` y `--memory` que se usan como parte del comando `docker run`.

## <a name="get-the-container-image-with-docker-pull"></a>Obtención de la imagen del contenedor con `docker pull`

Utilice el comando [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) para descargar la imagen de un contenedor del repositorio `mcr.microsoft.com/azure-cognitive-services/luis`:

```
docker pull mcr.microsoft.com/azure-cognitive-services/luis:latest
```

Para obtener una descripción completa de las etiquetas disponibles, como la etiqueta `latest` que se utilizó en el comando anterior, consulte [LUIS](https://go.microsoft.com/fwlink/?linkid=2043204) en Docker Hub.

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

## <a name="how-to-use-the-container"></a>Uso del contenedor

Una vez que el contenedor esté en el [equipo host](#the-host-computer), utilice el siguiente proceso para trabajar con el contenedor.

![Proceso para utilizar el contenedor de Language Understanding (LUIS)](./media/luis-container-how-to/luis-flow-with-containers-diagram.jpg)

1. [Exporte el paquete](#export-packaged-app-from-luis) del contenedor desde el portal de LUIS o las API de LUIS.
1. Mueva el archivo del paquete al directorio de **entrada** correspondiente del [equipo host](#the-host-computer). No debe modificar el archivo del paquete de LUIS, sobrescribirlo, descomprimirlo ni cambiar su nombre.
1. [Ejecute el contenedor](#run-the-container-with-docker-run) con la configuración de facturación y el _montaje de entrada_. Hay más [ejemplos](luis-container-configuration.md#example-docker-run-commands) del comando `docker run` disponibles.
1. [Consulte el punto de conexión de predicción del contenedor](#query-the-containers-prediction-endpoint).
1. Cuando haya terminado con el contenedor, [importe los registros de punto de conexión](#import-the-endpoint-logs-for-active-learning) desde la salida de montaje en el portal de LUIS y [detenga](#stop-the-container) el contenedor.
1. En el portal de LUIS, utilice el [aprendizaje activo ](luis-how-to-review-endpoint-utterances.md) de la página sobre la **revisión de expresiones de puntos de conexión** para mejorar la aplicación.

La aplicación que se ejecuta en el contenedor no se puede modificar. Con el fin de cambiar la aplicación en el contenedor, debe cambiar la aplicación en el servicio LUIS con el portal de [LUIS](https://www.luis.ai) o usar las [API de creación](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c2f) de LUIS. Después de realizar el entrenamiento o la publicación, descargue un nuevo paquete y ejecute de nuevo el contenedor.

La aplicación de LUIS que está dentro del contenedor no puede exportarse de nuevo al servicio de LUIS. Solo se pueden cargar los registros de consulta.

## <a name="export-packaged-app-from-luis"></a>Exportación de la aplicación empaquetada desde LUIS

El contenedor de LUIS necesita una aplicación de LUIS que esté entrenada o publicada para responder a consultas de predicción de expresiones de los usuarios. Para obtener la aplicación de LUIS, utilice la API del paquete publicado o entrenado.

La ubicación predeterminada es el subdirectorio `input` del lugar donde se ejecuta el comando `docker run`.

Sitúe el archivo del paquete en un directorio y, cuando ejecute el contenedor de Docker, haga referencia a este directorio estableciéndolo como montaje de entrada.

### <a name="package-types"></a>Tipos de paquetes

El directorio de montaje de entrada puede contener los modelos de **producción**, de **ensayo** y con **versiones** de la aplicación al mismo tiempo. Todos los paquetes se montan.

|Tipo de paquete|API de punto de conexión de consulta|Disponibilidad de consultas|Formato del nombre de archivo del paquete|
|--|--|--|--|
|Con versiones|GET, POST|Solo el contenedor|`{APP_ID}_v{APP_VERSION}.gz`|
|Ensayo|GET, POST|Azure y el contenedor|`{APP_ID}_STAGING.gz`|
|Producción|GET, POST|Azure y el contenedor|`{APP_ID}_PRODUCTION.gz`|

> [!IMPORTANT]
> No debe modificar los archivo del paquete de LUIS, descomprimirlos, sobrescribirlos ni cambiar su nombre.

### <a name="packaging-prerequisites"></a>Requisitos previos de empaquetado

Para poder empaquetar una aplicación de LUIS, debe tener lo siguiente:

|Requisitos de empaquetado|Detalles|
|--|--|
|Instancia de recurso de Azure _Cognitive Services_|Las regiones admitidas son<br><br>Oeste de EE. UU. (`westus`)<br>Oeste de Europa (`westeurope`)<br>Este de Australia (`australiaeast`)|
|Aplicación de LUIS entrenada o publicada|Sin [dependencias no compatibles][unsupported-dependencies]. |
|Acceso al sistema de archivos del [equipo host](#the-host-computer) |El equipo host debe permitir un [montaje de entrada](luis-container-configuration.md#mount-settings).|

### <a name="export-app-package-from-luis-portal"></a>Exportación del paquete de aplicación desde el portal de LUIS

El [portal](https://www.luis.ai) de LUIS brinda la posibilidad de exportar el paquete de la aplicación publicada o entrenada.

### <a name="export-published-apps-package-from-luis-portal"></a>Exportación del paquete de la aplicación publicada desde el portal de LUIS

El paquete de la aplicación publicada está disponible en la página **My Apps** (Mis aplicaciones).

1. Inicie sesión en el [portal](https://www.luis.ai) de LUIS.
1. Active la casilla situada a la izquierda del nombre de la aplicación de la lista.
1. Seleccione el elemento **Export** (Exportar) en la barra de herramientas contextual situada encima de la lista.
1. Seleccione **Export for container (GZIP)** [Exportar para contenedor (GZIP)].
1. Seleccione el entorno **Production slot** (Espacio de producción) o **Staging slot** (Espacio de ensayo).
1. El paquete se descarga desde el explorador.

![Exportación del paquete publicado del contenedor desde el menú Export (Exportar) de la página de la aplicación](./media/luis-container-how-to/export-published-package-for-container.png)

### <a name="export-versioned-apps-package-from-luis-portal"></a>Exportación del paquete de la aplicación con versiones desde el portal de LUIS

El paquete de la aplicación con versiones está disponible en la página de listas **Versions** (Versiones).

1. Inicie sesión en el [portal](https://www.luis.ai) de LUIS.
1. Seleccione la aplicación en la lista.
1. Seleccione **Manage** (Administrar) en la barra de navegación de la aplicación.
1. Seleccione **Versions** (Versiones) en la barra de navegación de la izquierda.
1. Active la casilla situada a la izquierda del nombre de la versión de la lista.
1. Seleccione el elemento **Export** (Exportar) en la barra de herramientas contextual situada encima de la lista.
1. Seleccione **Export for container (GZIP)** [Exportar para contenedor (GZIP)].
1. El paquete se descarga desde el explorador.

![Exportación del paquete entrenado del contenedor desde el menú Export (Exportar) de la página de versiones](./media/luis-container-how-to/export-trained-package-for-container.png)

### <a name="export-published-apps-package-from-api"></a>Exportación del paquete de la aplicación publicada desde la API

Utilice el siguiente método de la API REST para empaquetar una aplicación de LUIS que ya esté [publicada](luis-how-to-publish-app.md). Utilice sus proprios valores para sustituir los marcadores de posición de la llamada API; para ello, utilice la tabla situada bajo la especificación HTTP.

```http
GET /luis/api/v2.0/package/{APP_ID}/slot/{SLOT_NAME}/gzip HTTP/1.1
Host: {AZURE_REGION}.api.cognitive.microsoft.com
Ocp-Apim-Subscription-Key: {AUTHORING_KEY}
```

| Marcador de posición | Value |
|-------------|-------|
| **{APP_ID}** | Identificador de la aplicación de LUIS publicada. |
| **{SLOT_NAME}** | Entorno de la aplicación de LUIS publicada. Utilice uno de los valores siguientes:<br/>`PRODUCTION`<br/>`STAGING` |
| **{AUTHORING_KEY}** | Clave de creación de la cuenta de LUIS para la aplicación de LUIS publicada.<br/>Puede obtener la clave de creación en la página **User Settings** (Configuración del usuario) del portal de LUIS. |
| **{AZURE_REGION}** | Región de Azure que corresponda:<br/><br/>`westus`: Oeste de EE. UU.<br/>`westeurope`: Oeste de Europa<br/>`australiaeast`: Este de Australia |

Para descargar el paquete publicado, consulte la [documentación de la API aquí][download-published-package]. Si se descarga correctamente, la respuesta será un archivo de paquete de LUIS. Guarde el archivo en la ubicación de almacenamiento especificada para el montaje de entrada del contenedor.

### <a name="export-versioned-apps-package-from-api"></a>Exportación del paquete de la aplicación con versiones desde la API

Utilice el siguiente método de la API REST para empaquetar una aplicación de LUIS que ya esté [entrenada](luis-how-to-train.md). Utilice sus proprios valores para sustituir los marcadores de posición de la llamada API; para ello, utilice la tabla situada bajo la especificación HTTP.

```http
GET /luis/api/v2.0/package/{APP_ID}/versions/{APP_VERSION}/gzip HTTP/1.1
Host: {AZURE_REGION}.api.cognitive.microsoft.com
Ocp-Apim-Subscription-Key: {AUTHORING_KEY}
```

| Marcador de posición | Value |
|-------------|-------|
| **{APP_ID}** | Identificador de la aplicación de LUIS entrenada. |
| **{APP_VERSION}** | Versión de la aplicación de LUIS entrenada. |
| **{AUTHORING_KEY}** | Clave de creación de la cuenta de LUIS para la aplicación de LUIS publicada.<br/>Puede obtener la clave de creación en la página **User Settings** (Configuración del usuario) del portal de LUIS. |
| **{AZURE_REGION}** | Región de Azure que corresponda:<br/><br/>`westus`: Oeste de EE. UU.<br/>`westeurope`: Oeste de Europa<br/>`australiaeast`: Este de Australia |

Para descargar el paquete con versiones, consulte la [documentación de la API aquí][download-versioned-package]. Si se descarga correctamente, la respuesta será un archivo de paquete de LUIS. Guarde el archivo en la ubicación de almacenamiento especificada para el montaje de entrada del contenedor.

## <a name="run-the-container-with-docker-run"></a>Ejecute el contenedor con `docker run`.

Utilice el comando [docker run](https://docs.docker.com/engine/reference/commandline/run/) para ejecutar el contenedor. Consulte [Recopilación de los parámetros obligatorios](#gathering-required-parameters) para más información sobre cómo obtener los valores de `{ENDPOINT_URI}` y `{API_KEY}`.

Hay disponibles [ejemplos](luis-container-configuration.md#example-docker-run-commands) del comando `docker run`.

```console
docker run --rm -it -p 5000:5000 ^
--memory 4g ^
--cpus 2 ^
--mount type=bind,src=c:\input,target=/input ^
--mount type=bind,src=c:\output\,target=/output ^
mcr.microsoft.com/azure-cognitive-services/luis ^
Eula=accept ^
Billing={ENDPOINT_URI} ^
ApiKey={API_KEY}
```

* En este ejemplo se usa el directorio que está en la unidad `C:` para evitar conflictos con los permisos de Windows. Si necesita usar un directorio específico como directorio de entrada, tal vez tenga que conceder permiso al servicio de Docker.
* No cambie el orden de los argumentos a menos que esté familiarizado con los contenedores de Docker.
* Si usa otro sistema operativo, utilice la consola o el terminal, la sintaxis de carpeta de los montajes y el carácter de continuación de línea que sean adecuados para su sistema. En estos ejemplos se supone que se usa una consola de Windows con un carácter de continuación de línea `^`. Dado que el contenedor es un sistema operativo Linux, el montaje de destino usa una sintaxis de carpeta basada en Linux.

Este comando:

* Ejecuta un contenedor desde la imagen de contenedor de LUIS.
* Carga la aplicación de LUIS desde el montaje de entrada de *C:\input*, situado en el host del contenedor.
* Asigna dos núcleos de la CPU y 4 gigabytes (GB) de memoria.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Guarda los registros de LUIS y el contenedor en el montaje de salida de *C:\output*, situado en el host del contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

Hay más [ejemplos](luis-container-configuration.md#example-docker-run-commands) del comando `docker run` disponibles.

> [!IMPORTANT]
> Para poder ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará.  Para obtener más información, vea [Facturación](#billing).
> El valor de ApiKey es la **clave** de la página de **Azure Resources** del portal de LUIS y también está disponible en la página de claves del recurso `Cognitive Services` de Azure.

[!INCLUDE [Running multiple containers on the same host](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

## <a name="endpoint-apis-supported-by-the-container"></a>API de punto de conexión compatibles con el contenedor

Las versiones V2 y [V3](luis-migration-api-v3.md) de la API están disponibles con el contenedor.

## <a name="query-the-containers-prediction-endpoint"></a>Consulta del punto de conexión de predicción del contenedor

El contenedor proporciona varias API de puntos de conexión de predicción de consultas basadas en REST. Los puntos de conexión de las aplicaciones publicadas (de ensayo o producción) tienen una ruta _diferente_ a la de los puntos de conexión de las aplicaciones con versiones.

Utilice el host, `http://localhost:5000`, con las API de contenedor.

# <a name="v3-prediction-endpoint"></a>[Punto de conexión de predicción de V3](#tab/v3)

|Tipo de paquete|Verbo HTTP|Enrutar|Parámetros de consulta|
|--|--|--|--|
|Publicado|GET, POST|`/luis/v3.0/apps/{appId}/slots/{slotName}/predict?`|`query={query}`<br>[`&verbose`]<br>[`&log`]<br>[`&show-all-intents`]|
|Con versiones|GET, POST|`/luis/v3.0/apps/{appId}/versions/{versionId}/predict?`|`query={query}`<br>[`&verbose`]<br>[`&log`]<br>[`&show-all-intents`]|

Los parámetros de consulta determinan cómo y qué se devuelve en la respuesta de la consulta:

|Parámetro de consulta|Tipo|Propósito|
|--|--|--|
|`query`|string|Expresión del usuario.|
|`verbose`|boolean|Valor booleano que indica si se van a devolver todos los metadatos de los modelos de predicción. El valor predeterminado es False.|
|`log`|boolean|Registra las consultas, lo que puede utilizarse después para el [aprendizaje activo](luis-how-to-review-endpoint-utterances.md). El valor predeterminado es False.|
|`show-all-intents`|boolean|Valor booleano que indica si se devuelven todas las intenciones o solo la intención de puntuación superior. El valor predeterminado es False.|

# <a name="v2-prediction-endpoint"></a>[Punto de conexión de predicción de V2](#tab/v2)

|Tipo de paquete|Verbo HTTP|Enrutar|Parámetros de consulta|
|--|--|--|--|
|Publicado|[GET](https://westus.dev.cognitive.microsoft.com/docs/services/5819c76f40a6350ce09de1ac/operations/5819c77140a63516d81aee78), [POST](https://westus.dev.cognitive.microsoft.com/docs/services/5819c76f40a6350ce09de1ac/operations/5819c77140a63516d81aee79)|`/luis/v2.0/apps/{appId}?`|`q={q}`<br>`&staging`<br>[`&timezoneOffset`]<br>[`&verbose`]<br>[`&log`]<br>|
|Con versiones|GET, POST|`/luis/v2.0/apps/{appId}/versions/{versionId}?`|`q={q}`<br>[`&timezoneOffset`]<br>[`&verbose`]<br>[`&log`]|

Los parámetros de consulta determinan cómo y qué se devuelve en la respuesta de la consulta:

|Parámetro de consulta|Tipo|Propósito|
|--|--|--|
|`q`|string|Expresión del usuario.|
|`timezoneOffset`|number|timezoneOffset le permite [cambiar la zona horaria](luis-concept-data-alteration.md#change-time-zone-of-prebuilt-datetimev2-entity) que se utiliza en la entidad datetimeV2 pregenerada.|
|`verbose`|boolean|Si está establecido en true, devuelve todas las intenciones y sus puntuaciones. El valor predeterminado es false, donde solo se devuelve la intención principal.|
|`staging`|boolean|Si está establecido en true, devuelve la consulta a partir de los resultados del entorno de ensayo. |
|`log`|boolean|Registra las consultas, lo que puede utilizarse después para el [aprendizaje activo](luis-how-to-review-endpoint-utterances.md). El valor predeterminado es true.|

***

### <a name="query-the-luis-app"></a>Consulta de la aplicación de LUIS

A continuación, se muestra un ejemplo de un comando CURL para consultar el contenedor de una aplicación publicada:

# <a name="v3-prediction-endpoint"></a>[Punto de conexión de predicción de V3](#tab/v3)

Para consultar un modelo en un espacio, use la siguiente API:

```bash
curl -G \
-d verbose=false \
-d log=true \
--data-urlencode "query=turn the lights on" \
"http://localhost:5000/luis/v3.0/apps/{APP_ID}/slots/production/predict"
```

Para realizar consultas en el entorno de **ensayo**, reemplace `production` en la ruta por `staging`:

`http://localhost:5000/luis/v3.0/apps/{APP_ID}/slots/staging/predict`

Para consultar un modelo con versiones, use la siguiente API:

```bash
curl -G \
-d verbose=false \
-d log=false \
--data-urlencode "query=turn the lights on" \
"http://localhost:5000/luis/v3.0/apps/{APP_ID}/versions/{APP_VERSION}/predict"
```

# <a name="v2-prediction-endpoint"></a>[Punto de conexión de predicción de V2](#tab/v2)

Para consultar un modelo en un espacio, use la siguiente API:

```bash
curl -X GET \
"http://localhost:5000/luis/v2.0/apps/{APP_ID}?q=turn%20on%20the%20lights&staging=false&timezoneOffset=0&verbose=false&log=true" \
-H "accept: application/json"
```
Para realizar consultas en el entorno de **ensayo**, cambie el valor del parámetro de la cadena de consulta **staging** a true:

`staging=true`

Para consultar un modelo con versiones, use la siguiente API:

```bash
curl -X GET \
"http://localhost:5000/luis/v2.0/apps/{APP_ID}/versions/{APP_VERSION}?q=turn%20on%20the%20lights&timezoneOffset=0&verbose=false&log=true" \
-H "accept: application/json"
```
El nombre de la versión tiene un máximo de 10 caracteres y solamente contiene los caracteres que se permiten en las direcciones URL.

***

## <a name="import-the-endpoint-logs-for-active-learning"></a>Importación de los registros de punto de conexión para el aprendizaje activo

Si se especifica un montaje de salida para el contenedor de LUIS, los archivos de registro de consultas de la aplicación se guardan en el directorio de salida, donde `{INSTANCE_ID}` es el identificador del contenedor. El registro de consultas de la aplicación contiene la consulta, la respuesta y las marcas de tiempo de cada consulta de predicción enviada al contenedor de LUIS.

En la siguiente ubicación, se muestra la estructura anidada de directorios de los archivos de registro del contenedor.
```
/output/luis/{INSTANCE_ID}/
```

En el portal de LUIS, seleccione la aplicación y la opción **Import endpoint logs** (Importar registros de puntos de conexión) para cargar estos registros.

![Importación de los archivos de registro del contenedor para el aprendizaje activo](./media/luis-container-how-to/upload-endpoint-log-files.png)

Después de cargar el registro, [revise las expresiones del punto de conexión](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-review-endpoint-utterances) en el portal de LUIS.

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Detención del contenedor

Para apagar el contenedor, en el entorno de la línea de comandos en la que se ejecuta el contenedor, presione **Ctrl + C**.

## <a name="troubleshooting"></a>Solución de problemas

Si ejecuta el contenedor con un [montaje](luis-container-configuration.md#mount-settings) de salida y el registro habilitados, el contenedor genera archivos de registro que resultan útiles para solucionar problemas que se producen al iniciar o ejecutar el contenedor.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturación

El contenedor de LUIS envía información de facturación a Azure mediante un recurso de _Cognitive Services_ de la cuenta de Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Para obtener más información acerca de estas opciones, consulte [Configure containers](luis-container-configuration.md) (Configuración de contenedores).

<!--blogs/samples/video courses -->
[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Resumen

En este artículo, ha aprendido algunos conceptos y ha visto el flujo de trabajo de Language Understanding (LUIS) para descargar, instalar y ejecutar contenedores. En resumen:

* Language Understanding (LUIS) proporciona un contenedor de Linux para Docker que permite hacer predicciones de consultas de puntos de conexión de expresiones.
* Las imágenes del contenedor se descargan desde Microsoft Container Registry (MCR).
* Las imágenes de contenedor se ejecutan en Docker.
* Puede usar la API REST para consultar los puntos de conexión del contenedor especificando el URI que tiene el contenedor en el host.
* Debe especificar la información de facturación al crear una instancia de un contenedor.

> [!IMPORTANT]
> Los contenedores de Cognitive Services no tienen licencia para ejecutarse sin estar conectados a Azure para realizar mediciones. Los clientes tienen que habilitar los contenedores para comunicar la información de facturación con el servicio de medición en todo momento. Los contenedores de Cognitive Services no envían datos de los clientes (por ejemplo, la imagen o el texto que se está analizando) a Microsoft.

## <a name="next-steps"></a>Pasos siguientes

* Revise [Configuración de contenedores](luis-container-configuration.md) para ver las opciones de configuración.
* Consulte las [limitaciones de los contenedores de LUIS](luis-container-limitations.md) para conocer las restricciones de funcionalidad conocidas.
* Consulte [Solución de problemas](troubleshooting.md) para resolver los problemas relacionados con la funcionalidad de LUIS.
* Uso de [Contenedores de Cognitive Services](../cognitive-services-container-support.md)

<!-- Links - external -->
[download-published-package]: https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagepublishedapplicationasgzip
[download-versioned-package]: https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagetrainedapplicationasgzip

[unsupported-dependencies]: luis-container-limitations.md#unsupported-dependencies-for-latest-container
