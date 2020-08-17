---
title: Codificación y transmisión de un archivo remoto con Azure Media Services v3
description: Siga los pasos de este tutorial para codificar un archivo según una dirección URL y transmitir su contenido con Azure Media Services mediante REST.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
ms.custom: mvc
ms.date: 03/16/2020
ms.author: juliako
ms.openlocfilehash: f12771e55ced3b8783b6c7497b83e6b041c66b75
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074475"
---
# <a name="tutorial-encode-a-remote-file-based-on-url-and-stream-the-video---rest"></a>Tutorial: Codificación de un archivo remoto según una dirección URL y transmisión del vídeo: REST

Azure Media Services permite codificar los archivos multimedia en formatos que se pueden reproducir en una gran variedad de exploradores y dispositivos. Por ejemplo, puede que quiera transmitir su contenido en los formatos HLS o MPEG DASH de Apple. Antes de la transmisión, primero debe codificar su archivo de medios digitales de alta calidad. Para obtener instrucciones acerca de la codificación, consulte [El concepto de codificación](encoding-concept.md).

En este tutorial se muestra cómo codificar un archivo según una dirección URL y transmitir el vídeo con Azure Media Services mediante REST. 

![Reproducción del vídeo](./media/stream-files-tutorial-with-api/final-video.png)

En este tutorial se muestra cómo realizar las siguientes acciones:    

> [!div class="checklist"]
> * Creación de una cuenta de Media Services
> * Acceso a la API de Media Services
> * Descarga de archivos Postman
> * Configuración de Postman
> * Envío de solicitudes mediante Postman
> * Prueba de la URL de streaming
> * Limpieza de recursos

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

- [Cree una cuenta de Media Services](./create-account-howto.md).

    Asegúrese de recordar los valores que usó para el nombre de la cuenta de Media Services y el nombre del grupo de recursos

- Instale el cliente de REST de [Postman](https://www.getpostman.com/) para ejecutar las API de REST mostradas en algunos de los tutoriales de REST de AMS. 

    Usamos **Postman** pero cualquier herramienta de REST sería adecuada. Otras alternativas son: **Visual Studio Code** con el complemento de REST o **Telerik Fiddler**. 

## <a name="download-postman-files"></a>Descarga de archivos Postman

Clone un repositorio de GitHub que contenga los archivos de recopilación y entorno de Postman.

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-rest-postman.git
 ```

## <a name="access-api"></a>Acceso a la API

Para más información, consulte [Obtención de credenciales para acceder a Media Services API](access-api-howto.md)

## <a name="configure-postman"></a>Configuración de Postman

### <a name="configure-the-environment"></a>Configuración del entorno 

1. Abra la aplicación **Postman**.
2. A la derecha de la pantalla, seleccione la opción **Manage Environments** (Administrar entornos).

    ![Administración del entorno](./media/develop-with-postman/postman-import-env.png)
4. En el cuadro de diálogo **Manage Environments** (Administrar entornos), haga clic en **Import** (Importar).
2. Vaya al archivo `Azure Media Service v3 Environment.postman_environment.json` que se descargó cuando clonó `https://github.com/Azure-Samples/media-services-v3-rest-postman.git`.
6. Se agrega el entorno **Azure Media Service v3 Environment**.

    > [!Note]
    > Actualice las variables de acceso con los valores que obtuvo de la sección **Acceso a la API de Media Services** anterior.

7. Haga doble clic en el archivo seleccionado y escriba los valores que obtuvo al seguir los pasos descritos en [acceso a la API](#access-api).
8. Cierre el cuadro de diálogo.
9. Seleccione el entorno **Azure Media Service v3 Environment** en la lista desplegable.

    ![Selección del entorno](./media/develop-with-postman/choose-env.png)
   
### <a name="configure-the-collection"></a>Configuración de la colección

1. Haga clic en **Import** (Importar) para importar el archivo de la colección.
1. Vaya al archivo `Media Services v3.postman_collection.json` que se descargó cuando clonó `https://github.com/Azure-Samples/media-services-v3-rest-postman.git`
3. Elija el archivo **Media Services v3.postman_collection.json**.

    ![Importación de un archivo](./media/develop-with-postman/postman-import-collection.png)

## <a name="send-requests-using-postman"></a>Envío de solicitudes mediante Postman

En esta sección se enviarán solicitudes que son significativas para codificar y crear direcciones URL, de modo que pueda hacer streaming del archivo. En concreto, se envían las solicitudes siguientes:

1. Obtención del token de Azure AD para la autenticación de la entidad de servicio
1. Inicio de un punto de conexión de streaming
2. Creación de un recurso de salida
3. Cree una transformación
4. Creación de un trabajo
5. Creación de un objeto StreamingLocator
6. Enumeración de las rutas de acceso de StreamingLocator

> [!Note]
>  En este tutorial se da por hecho que va a crear todos los recursos con nombres únicos.  

### <a name="get-azure-ad-token"></a>Obtención del token de Azure AD 

1. En la ventana izquierda de la aplicación Postman, seleccione "Step 1: Get AAD Auth token" (Paso 1: Obtención del token de autorización de AAD).
2. A continuación, seleccione "Get Azure AD Token for Service Principal Authentication" (Obtener token de Azure AD para la autenticación de la entidad de servicio).
3. Presione **Enviar**.

    Se envía la siguiente operación **POST**.

    ```
    https://login.microsoftonline.com/:tenantId/oauth2/token
    ```

4. La respuesta regresa con el token y establece la variable de entorno "AccessToken" en el valor del token. Para ver el código que establece "AccessToken", haga clic en la pestaña **Tests** (Pruebas). 

    ![Obtención del token de AAD](./media/develop-with-postman/postman-get-aad-auth-token.png)


### <a name="start-a-streaming-endpoint"></a>Inicio de un punto de conexión de streaming

Para iniciar la transmisión del vídeo, primero debe iniciar el [punto de conexión de streaming](./streaming-endpoint-concept.md) desde el que quiere hacerlo.

> [!NOTE]
> Solo se le cobrará cuando el punto de conexión de streaming esté en estado de ejecución.

1. En la ventana izquierda de la aplicación Postman, seleccione "Streaming and Live" (Streaming y en vivo).
2. A continuación, seleccione "Start StreamingEndpoint" (Iniciar punto de conexión de streaming).
3. Presione **Enviar**.

    * Se envía la siguiente operación **POST**:

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaservices/:accountName/streamingEndpoints/:streamingEndpointName/start?api-version={{api-version}}
        ```
    * Si la solicitud es correcta, se devuelve `Status: 202 Accepted`.

        Este estado significa que se ha aceptado el procesamiento de la solicitud, pero no se ha completado. Puede consultar el estado de la operación según el valor del encabezado de respuesta `Azure-AsyncOperation`.

        Por ejemplo, la siguiente operación GET devuelve el estado de la operación:
        
        `https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/<resourceGroupName>/providers/Microsoft.Media/mediaservices/<accountName>/streamingendpointoperations/1be71957-4edc-4f3c-a29d-5c2777136a2e?api-version=2018-07-01`

        En el artículo [Seguimiento de las operaciones asincrónicas de Azure](../../azure-resource-manager/management/async-operations.md) se explica de forma detallada cómo realizar un seguimiento del estado de las operaciones asincrónicas de Azure mediante los valores devueltos en la respuesta.

### <a name="create-an-output-asset"></a>Creación de un recurso de salida

El [recurso](/rest/api/media/assets) de salida almacena el resultado del trabajo de codificación. 

1. En la ventana izquierda de la aplicación Postman, seleccione "Assets" (Recursos).
2. A continuación, seleccione "Create or update an Asset" (Crear o actualizar un recurso).
3. Presione **Enviar**.

    * Se envía la siguiente operación **PUT**:

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/assets/:assetName?api-version={{api-version}}
        ```
    * La operación tiene el siguiente cuerpo:

        ```json
        {
        "properties": {
            "description": "My Asset",
            "alternateId" : "some GUID"
         }
        }
        ```

### <a name="create-a-transform"></a>Creación de una transformación

Cuando se codifica o procesa contenido en Media Services, es un patrón común configurar los ajustes de codificación como una receta. Después, podría enviar un **trabajo** para aplicar esa receta a un vídeo. Al enviar nuevos trabajos en cada nuevo vídeo, está aplicando dicha receta a todos los vídeos de la biblioteca. Una receta en Media Services se llama **transformación**. Para obtener más información, consulte [Transformaciones y trabajos](./transforms-jobs-concept.md). El ejemplo descrito en este tutorial define una receta que codifica el vídeo para transmitirlo a varios dispositivos iOS y Android. 

Al crear una nueva instancia de la [transformación](/rest/api/media/transforms), debe especificar qué desea originar como salida. El parámetro requerido es un objeto **TransformOutput**. Cada objeto **TransformOutput** contiene un **valor preestablecido**. El **valor preestablecido** describe las instrucciones paso a paso de las operaciones de procesamiento de vídeo o audio que se utilizarán para generar el objeto **TransformOutput** deseado. El ejemplo descrito en este artículo utiliza un valor preestablecido integrado denominado **AdaptiveStreaming**. El valor preestablecido codifica el vídeo de entrada en una escala de velocidad de bits generada automáticamente (pares resolución-velocidad de bits) basada en la resolución y la velocidad de bits, y produce archivos ISO MP4 con vídeo H.264 y audio AAC correspondiente a cada par resolución-velocidad de bits. Para más información sobre este valor preestablecido, consulte el artículo sobre la [generación automática de la escala de velocidad de bits](autogen-bitrate-ladder.md).

Puede usar un valor de EncoderNamedPreset integrado o valores preestablecidos personalizados. 

> [!Note]
> Al crear una [transformación](/rest/api/media/transforms), debe comprobar primero si ya existe una con el método **Get**. En este tutorial se da por hecho que va a crear la transformación con un nombre único.

1. En la ventana izquierda de la aplicación Postman, seleccione "Encoding and Analysis" (Codificación y análisis).
2. A continuación, seleccione "Create Transform" (Crear transformación).
3. Presione **Enviar**.

    * Se envía la siguiente operación **PUT**.

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/transforms/:transformName?api-version={{api-version}}
        ```
    * La operación tiene el siguiente cuerpo:

        ```json
        {
            "properties": {
                "description": "Standard Transform using an Adaptive Streaming encoding preset from the library of built-in Standard Encoder presets",
                "outputs": [
                    {
                    "onError": "StopProcessingJob",
                "relativePriority": "Normal",
                    "preset": {
                        "@odata.type": "#Microsoft.Media.BuiltInStandardEncoderPreset",
                        "presetName": "AdaptiveStreaming"
                    }
                    }
                ]
            }
        }
        ```

### <a name="create-a-job"></a>Creación de un trabajo

Un [trabajo](/rest/api/media/jobs) es la solicitud real a Media Services para aplicar la **transformación** creada a un determinado contenido de vídeo o audio. El **trabajo** especifica información como la ubicación del vídeo de entrada y la ubicación de la salida.

En este ejemplo, la entrada del trabajo se basa en una dirección URL HTTPS ("https:\//nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/").

1. En la ventana izquierda de la aplicación Postman, seleccione "Encoding and Analysis" (Codificación y análisis).
2. A continuación, seleccione "Create or Update Job" (Crear o actualizar trabajo).
3. Presione **Enviar**.

    * Se envía la siguiente operación **PUT**.

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/transforms/:transformName/jobs/:jobName?api-version={{api-version}}
        ```
    * La operación tiene el siguiente cuerpo:

        ```json
        {
        "properties": {
            "input": {
            "@odata.type": "#Microsoft.Media.JobInputHttp",
            "baseUri": "https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/",
            "files": [
                    "Ignite-short.mp4"
                ]
            },
            "outputs": [
            {
                "@odata.type": "#Microsoft.Media.JobOutputAsset",
                "assetName": "testAsset1"
            }
            ]
        }
        }
        ```

El trabajo tarda algún tiempo en completarse y cuando lo hace querrá recibir una notificación. Para ver el progreso del trabajo, se recomienda usar Event Grid. Event Grid está diseñado para conseguir alta disponibilidad, rendimiento coherente y escalado dinámico. Con Event Grid, sus aplicaciones pueden escuchar y reaccionar a eventos de casi todos los servicios de Azure y de orígenes personalizados. El control de eventos sencillo y reactivo basado en HTTP le ayuda a crear soluciones eficaces con filtrado y enrutamiento de eventos inteligente.  Consulte [Enrutamiento de eventos a un punto de conexión web personalizado](job-state-events-cli-how-to.md).

El **trabajo** pasa normalmente por los siguientes estados: **Programado**, **En cola**, **Procesando**, **Finalizado** (el estado final). Si el trabajo ha encontrado un error, obtendrá el estado **Error**. Si el trabajo está en proceso de cancelación, obtendrá **Cancelando** y **Cancelado** cuando haya terminado.

#### <a name="job-error-codes"></a>Códigos de error de trabajo

Consulte [Códigos de error](/rest/api/media/jobs/get#joberrorcode).

### <a name="create-a-streaming-locator"></a>Creación de un localizador de streaming

Una vez finalizado el trabajo de codificación, el siguiente paso es poner el vídeo del **recurso** de salida a disposición de los clientes para su reproducción. Puede hacerlo en dos pasos: primero, cree un objeto [StreamingLocator](/rest/api/media/streaminglocators) y, en segundo lugar, cree las direcciones URL de streaming que pueden usar los clientes. 

El proceso de creación de un localizador de streaming se denomina publicación. De forma predeterminada, el localizador de streaming es válido inmediatamente después de realizar las llamadas a la API y dura hasta que se elimina, salvo que configure las horas de inicio y de finalización opcionales. 

Al crear un objeto [StreamingLocator](/rest/api/media/streaminglocators), debe especificar el objeto **StreamingPolicyName** deseado. En este ejemplo, va a transmitir contenido no cifrado, de modo que se puede usar la directiva de streaming sin cifrar predefinida, "Predefined_ClearStreamingOnly".

> [!IMPORTANT]
> Al utilizar el objeto [StreamingPolicy](/rest/api/media/streamingpolicies) personalizado, debe diseñar un conjunto limitado de dichas directivas para su cuenta de Media Service y reutilizarlas para sus objetos StreamingLocator siempre que se necesiten las mismas opciones y protocolos de cifrado. 

La cuenta de Media Service tiene una cuota para el número de entradas de **Streaming Policy**. No debe crear un objeto **Streaming Policy** para cada localizador de streaming.

1. En la ventana izquierda de la aplicación Postman, seleccione "Streaming Policies and Locators" (Directivas de streaming y localizadores).
2. A continuación, seleccione "Create a Streaming Locator (clear)" (Crear un localizador de streaming [sin cifrado]).
3. Presione **Enviar**.

    * Se envía la siguiente operación **PUT**.

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/streamingLocators/:streamingLocatorName?api-version={{api-version}}
        ```
    * La operación tiene el siguiente cuerpo:

        ```json
        {
          "properties": {
            "streamingPolicyName": "Predefined_ClearStreamingOnly",
            "assetName": "testAsset1",
            "contentKeys": [],
            "filters": []
         }
        }
        ```

### <a name="list-paths-and-build-streaming-urls"></a>Enumeración de rutas de acceso y creación de direcciones URL de streaming

#### <a name="list-paths"></a>Enumeración de rutas de acceso

Ahora que se ha creado el objeto [StreamingLocator](/rest/api/media/streaminglocators), puede obtener las direcciones URL de streaming.

1. En la ventana izquierda de la aplicación Postman, seleccione "Streaming Policies" (Directivas de streaming).
2. A continuación, seleccione "List Paths" (Enumerar rutas de acceso).
3. Presione **Enviar**.

    * Se envía la siguiente operación **POST**.

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/streamingLocators/:streamingLocatorName/listPaths?api-version={{api-version}}
        ```
        
    * La operación no tiene cuerpo:
        
4. Anote una de las rutas de acceso que desea usar con el streaming, la utilizará en la sección siguiente. En este caso, se devolvieron las rutas de acceso siguientes:
    
    ```
    "streamingPaths": [
        {
            "streamingProtocol": "Hls",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=m3u8-aapl)"
            ]
        },
        {
            "streamingProtocol": "Dash",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=mpd-time-csf)"
            ]
        },
        {
            "streamingProtocol": "SmoothStreaming",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest"
            ]
        }
    ]
    ```

#### <a name="build-the-streaming-urls"></a>Creación de las direcciones URL de streaming

En esta sección, crearemos una dirección URL de streaming de HLS. Las direcciones URL constan de los siguientes valores:

1. El protocolo a través del cual se envían datos. En este caso, "https".

    > [!NOTE]
    > Si el reproductor está hospedado en un sitio https, asegúrese de actualizar la dirección URL a "https".

2. Nombre de host de StreamingEndpoint. En este caso, el nombre es "amsaccount-usw22.streaming.media.azure.net".

    Para obtener el nombre de host puede utilizar la siguiente operación GET:
    
    ```
    https://management.azure.com/subscriptions/00000000-0000-0000-0000-0000000000000/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/streamingEndpoints/default?api-version={{api-version}}
    ```
    
3. Una ruta de acceso que obtuvo en la sección anterior (rutas de lista).  

Como resultado, se creó la siguiente dirección URL de HLS

```
https://amsaccount-usw22.streaming.media.azure.net/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=m3u8-aapl)
```

## <a name="test-the-streaming-url"></a>Prueba de la URL de streaming


> [!NOTE]
> Asegúrese de que el **punto de conexión de streaming** desde el que va a hacer streaming del contenido esté en ejecución.

Para probar el streaming, este artículo usa Azure Media Player. 

1. Abra un explorador web y vaya a [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/).
2. En el cuadro **URL:** , pegue la dirección URL que ha creado. 
3. Presione **Actualizar Player**.

Azure Media Player puede usarse para realizar pruebas, pero no debe usarse en un entorno de producción. 

## <a name="clean-up-resources-in-your-media-services-account"></a>Limpieza de los recursos en su cuenta de Media Services

Por lo general, debe limpiar todo excepto los objetos que piensa reutilizar (típicamente, reutilizará **transformaciones** y conservará los objetos **StreamingLocator**, etc.). Si desea que la cuenta esté limpia después de la experimentación, debe eliminar los recursos que no piensa volver a usar.  

Para eliminar un recurso, seleccione la operación "Eliminar..." en el recurso que desea eliminar.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita ninguno de los recursos del grupo de recursos, incluida las cuentas de almacenamiento y de Media Services que ha creado en este tutorial, elimine el grupo de recursos que ha creado antes.  

Ejecute el siguiente comando de la CLI:

```azurecli
az group delete --name amsResourceGroup
```

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo cargar, codificar y transmitir el vídeo, consulte el siguiente artículo: 

> [!div class="nextstepaction"]
> [Análisis de vídeos](analyze-videos-tutorial-with-api.md)
