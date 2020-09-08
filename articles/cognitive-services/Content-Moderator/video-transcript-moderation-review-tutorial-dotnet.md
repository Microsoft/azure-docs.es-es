---
title: 'Tutorial: Moderación de vídeos y transcripciones en .NET: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Este tutorial le ayudará a aprender a crear una solución completa de moderación de vídeo y transcripción con moderación asistida por máquina y la creación de la revisión humana en bucle.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: tutorial
ms.date: 08/05/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 1f5a68bcf0069663d8ef1101407bea7ee26e9e8b
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919297"
---
# <a name="tutorial-video-and-transcript-moderation"></a>Tutorial: Moderación de vídeos y transcripciones

En este tutorial, aprenderá a crear una solución completa de moderación de vídeo y transcripción con moderación asistida por máquina e integración de revisión humana.

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> - Comprimir los vídeos dE entrada para un procesamiento más rápido
> - Moderar el vídeo para obtener capturas y fotogramas con información
> - Utilizar las marcas de tiempo de los fotogramas para crear miniaturas (imágenes)
> - Enviar marcas de tiempo y miniaturas para crear revisiones de vídeos
> - Convertir la voz del vídeo en texto (transcripción) con la API de Media Indexer
> - Moderar la transcripción con el servicio de moderación de texto
> - Agregar la transcripción moderada a la revisión del vídeo

## <a name="prerequisites"></a>Prerrequisitos

- Regístrese en el sitio web [Herramienta de revisión de Content Moderator](https://contentmoderator.cognitive.microsoft.com/) y cree etiquetas personalizadas. Consulte[Uso de etiquetas](Review-Tool-User-Guide/tags.md) si necesita ayuda con este paso.

    ![captura de pantalla de etiquetas personalizadas de moderación de vídeo](images/video-tutorial-custom-tags.png)
- Para ejecutar la aplicación de ejemplo, necesitará una cuenta de Azure, un recurso de Azure Media Services, un recurso de Azure Content Moderator y credenciales de Azure Active Directory. Para instrucciones sobre cómo obtener estos recursos, consulte la guía de la [API Video Moderation](video-moderation-api.md).
- Descargue la [aplicación de consola de revisión de vídeo](https://github.com/MicrosoftContentModerator/VideoReviewConsoleApp) desde GitHub.

## <a name="enter-credentials"></a>Escribir credenciales

Edite el archivo `App.config` y agregue el nombre del inquilino de Active Directory, los puntos de conexión de servicio y las claves de la suscripción que indica `#####`. Necesita la siguiente información:

|Clave|Descripción|
|-|-|
|`AzureMediaServiceRestApiEndpoint`|Punto de conexión para la API de Azure Media Services|
|`ClientSecret`|Clave de suscripción de Azure Media Services|
|`ClientId`|Identificador de cliente para Azure Media Services|
|`AzureAdTenantName`|Nombre de inquilino de Active Directory que representa su organización|
|`ContentModeratorReviewApiSubscriptionKey`|Clave de suscripción para la API de revisión de Content Moderator|
|`ContentModeratorApiEndpoint`|Punto de conexión para Content Moderator API|
|`ContentModeratorTeamId`|Identificador de equipo del moderador de contenido|

## <a name="examine-the-main-code"></a>Examinar el código principal

La clase `Program` en `Program.cs` es el punto de entrada principal para la aplicación de moderación de vídeo.

### <a name="methods-of-program-class"></a>Métodos de la clase Program

|Método|Descripción|
|-|-|
|`Main`|Analiza la línea de comandos, recopila la información del usuario y empieza a procesar.|
|`ProcessVideo`|Comprime, carga, modera y crea las revisiones del vídeo.|
|`CreateVideoStreamingRequest`|Crea una secuencia para cargar un vídeo|
|`GetUserInputs`|Recopila información proporcionada por el usuario; se usa cuando no hay ninguna opción de línea de comandos|
|`Initialize`|Inicializa los objetos necesarios para el proceso de moderación|

### <a name="the-main-method"></a>El método Main

`Main()` es donde comienza la ejecución, por lo que es el lugar para empezar a entender el proceso de moderación de vídeo.

[!code-csharp[Main](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/Program.cs?range=20-24,42-52,54-74)]

`Main()` controla los argumentos de línea de comandos siguientes:

- La ruta de acceso a un directorio que contiene archivos de vídeo MPEG-4 que deben enviarse para moderación. Todos los archivos `*.mp4` de este directorio y sus subdirectorios se envían para moderación.
- Si lo desea, un marcador booleano (true/false) que indica si deben generarse transcripciones de texto con el fin de moderar audio.

Si no hay argumentos de línea de comandos, `Main()` llama a `GetUserInputs()`. Este método solicita al usuario que escriba la ruta de acceso a un único archivo de vídeo y especifique si se debe generar una transcripción de texto.

> [!NOTE]
> La aplicación de consola utiliza la [API de Azure Media Indexer](https://docs.microsoft.com/azure/media-services/media-services-process-content-with-indexer2) para generar las transcripciones a partir de la pista de audio del vídeo cargado. Los resultados se muestran en formato WebVTT. Para obtener más información sobre este formato, vea [Web Video Text Tracks Format](https://developer.mozilla.org/docs/Web/API/WebVTT_API) (Formato de pistas de texto de vídeo web).

### <a name="initialize-and-processvideo-methods"></a>Métodos Initialize y ProcessVideo

Independientemente de si las opciones del programa proceden de la línea de comandos o de la entrada interactiva del usuario, `Main()` llama a `Initialize()` para crear las instancias siguientes:

|Clase|Descripción|
|-|-|
|`AMSComponent`|Comprime los archivos de vídeo antes de enviarlos para moderación.|
|`AMSconfigurations`|Interfaz para los datos de configuración de la aplicación, que se encuentra en `App.config`.|
|`VideoModerator`| Carga, codificación, cifrado y moderación con el SDK de Azure Media Services|
|`VideoReviewApi`|Administra las revisiones de vídeo en el servicio Content Moderator.|

Estas clases (además de `AMSConfigurations`, que es sencilla) se tratan con más detalle en las próximas secciones de este tutorial.

Por último, los archivos de vídeo se procesan uno a uno mediante una llamada a `ProcessVideo()` para cada uno.

[!code-csharp[ProcessVideo](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/Program.cs?range=76-118)]

El método `ProcessVideo()` es bastante sencillo. Realiza las operaciones siguientes en orden:

- Comprime el vídeo
- Carga el vídeo en un recurso de Azure Media Services
- Crea un trabajo de Azure Media Services para moderar el vídeo
- Crea una revisión del vídeo en Content Moderator

En las siguientes secciones se examinan con más detalle algunos de los procesos individuales invocados por `ProcessVideo()`. 

## <a name="compress-the-video"></a>Comprimir el vídeo

Para minimizar el tráfico de red, la aplicación convierte archivos de vídeo a formato H.264 (MPEG-4 AVC) y les aplica una escala de un ancho máximo de 640 píxeles. Se recomienda el códec H.264 por su alta eficacia (velocidad de compresión). La compresión se realiza mediante la herramienta de línea de comandos `ffmpeg` gratuita, que se incluye en la carpeta `Lib` de la solución de Visual Studio. Los archivos de entrada pueden ser de cualquier formato admitido por `ffmpeg`, incluidos los códecs y formatos de archivo de vídeo más usados.

> [!NOTE]
> Cuando se inicia el programa con las opciones de línea de comandos, se especifica un directorio que contiene los archivos de vídeo que se envían para moderación. Se procesan todos los archivos de este directorio con la extensión de nombre de archivo `.mp4`. Para procesar otras extensiones de nombre de archivo, actualice el método `Main()` en `Program.cs` para incluir las extensiones deseadas.

El código que comprime un solo archivo de vídeo es la clase `AmsComponent` en `AMSComponent.cs`. El método responsable de esta funcionalidad es `CompressVideo()`, que se muestra aquí.

[!code-csharp[CompressVideo](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/AMSComponent.cs?range=31-59)]

Este código realiza los pasos siguientes:

- Realiza comprobaciones para asegurarse de que la configuración de `App.config` contiene todos los datos necesarios
- Realiza comprobaciones para asegurarse de que el binario `ffmpeg` está presente
- Genera el nombre de archivo de salida mediante la anexión de `_c.mp4` al nombre base del archivo (como `Example.mp4` -> `Example_c.mp4`)
- Genera una cadena de línea de comandos para llevar a cabo la conversión
- Inicia un proceso `ffmpeg` mediante la línea de comandos
- Espera a que el vídeo se procese

> [!NOTE]
> Si sabe con certeza que sus vídeos ya se han comprimido con H.264 y tienen las dimensiones adecuadas, puede volver a escribir `CompressVideo()` para omitir la compresión.

El método devuelve el nombre del archivo de salida comprimido.

## <a name="upload-and-moderate-the-video"></a>Cargar y moderar el vídeo

El vídeo debe almacenarse en Azure Media Services para que el servicio Content Moderation lo pueda procesar. La clase `Program` en `Program.cs` tiene un método breve `CreateVideoStreamingRequest()` que devuelve un objeto que representa la solicitud de streaming usada para cargar el vídeo.

[!code-csharp[CreateVideoStreamingRequest](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/Program.cs?range=120-133)]

El objeto `UploadVideoStreamRequest` resultante se define en `UploadVideoStreamRequest.cs` (y su elemento primario, `UploadVideoRequest`, en `UploadVideoRequest.cs`). Estas clases no se muestran aquí; son breves y solo sirven para almacenar los datos de vídeo comprimidos y la información sobre estos. Otra clase de solo datos, `UploadAssetResult` (`UploadAssetResult.cs`) se utiliza para almacenar los resultados del proceso de carga. Ahora es posible comprender estas líneas en `ProcessVideo()`:

[!code-csharp[ProcessVideoSnippet](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/Program.cs?range=91-104)]

Estas líneas realizan las siguientes tareas:

- Crean un `UploadVideoStreamRequest` para cargar el vídeo comprimido
- Establecen la marca `GenerateVTT` de la solicitud si el usuario ha solicitado una transcripción de texto
- Llaman a `CreateAzureMediaServicesJobToModerateVideo()` para realizar la carga y recibir el resultado

## <a name="examine-video-moderation-code"></a>Examinar el código de moderación de vídeo

El método `CreateAzureMediaServicesJobToModerateVideo()` está en `VideoModerator.cs`, que contiene la mayor parte del código que interactúa con Azure Media Services. El código fuente del método se muestra en el siguiente extracto.

[!code-csharp[CreateAzureMediaServicesJobToModerateVideo](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/VideoModerator.cs?range=230-283)]

Este código realiza las tareas siguientes:

- Crea un trabajo de AMS para que se realice el procesamiento
- Agrega tareas para codificar el archivo de vídeo, moderándolo y generando una transcripción de texto
- Envía el trabajo, cargando el archivo e iniciando el procesamiento
- Recupera los resultados de la moderación, la transcripción de texto (si se solicita) y otra información

## <a name="sample-video-moderation-output"></a>Salida de moderación de vídeo de ejemplo

El resultado del trabajo de moderación de vídeo (vea el [inicio rápido de moderación de vídeo](video-moderation-api.md) es una estructura de datos JSON que contiene los resultados de la moderación. Estos resultados incluyen un desglose de los fragmentos (capturas) en el vídeo, y cada uno de ellos contiene eventos (clips) con fotogramas clave que se han marcado para su revisión. Cada fotograma clave se puntúa sobre la probabilidad de que tenga contenido para adultos. En el ejemplo siguiente se muestra una respuesta JSON:

```json
{
    "version": 2,
    "timescale": 90000,
    "offset": 0,
    "framerate": 50,
    "width": 1280,
    "height": 720,
    "totalDuration": 18696321,
    "fragments": [
    {
        "start": 0,
        "duration": 18000
    },
    {
        "start": 18000,
        "duration": 3600,
        "interval": 3600,
        "events": [
        [
        {
            "reviewRecommended": false,
            "adultScore": 0.00001,
            "racyScore": 0.03077,
            "index": 5,
            "timestamp": 18000,
            "shotIndex": 0
        }
        ]
    ]
    },
    {
        "start": 18386372,
        "duration": 119149,
        "interval": 119149,
        "events": [
        [
        {
            "reviewRecommended": true,
            "adultScore": 0.00000,
            "racyScore": 0.91902,
            "index": 5085,
            "timestamp": 18386372,
            "shotIndex": 62
        }
    ]
    ]
    }
]
}
```

También se produce una transcripción de audio del vídeo cuando está configurada la marca `GenerateVTT`.

> [!NOTE]
> La aplicación de consola utiliza la [API de Azure Media Indexer](https://docs.microsoft.com/azure/media-services/media-services-process-content-with-indexer2) para generar las transcripciones a partir de la pista de audio del vídeo cargado. Los resultados se muestran en formato WebVTT. Para obtener más información sobre este formato, vea [Web Video Text Tracks Format](https://developer.mozilla.org/docs/Web/API/WebVTT_API) (Formato de pistas de texto de vídeo web).

## <a name="create-a-human-review"></a>Crear una revisión humana

El proceso de moderación devuelve una lista de fotogramas clave del vídeo, junto con una transcripción de sus pistas de audio. El siguiente paso es crear una revisión en la herramienta de revisión de Content Moderator para moderadores humanos. Volviendo al método `ProcessVideo()` en `Program.cs`, verá la llamada al método `CreateVideoReviewInContentModerator()`. Este método está en la clase `videoReviewApi`, que se encuentra en `VideoReviewAPI.cs` y se muestra aquí.

[!code-csharp[CreateVideoReviewInContentModerator](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/VideoReviewAPI.cs?range=42-69)]

`CreateVideoReviewInContentModerator()` llama a otros métodos para llevar a cabo las siguientes tareas:

> [!NOTE]
> La aplicación de consola utiliza la biblioteca [FFmpeg](https://ffmpeg.org/) para generar miniaturas. Estas miniaturas (imágenes) se corresponden con las marcas de tiempo de los fotogramas en la salida de moderación de vídeo.

|Tarea|Métodos|Archivo|
|-|-|-|
|Extraer los fotogramas clave del vídeo y crear imágenes en miniatura a partir de ellos|`CreateVideoFrames()`<br>`GenerateFrameImages()`|`FrameGeneratorServices.cs`|
|Analizar la transcripción de texto, si está disponible, para buscar el audio para adultos|`GenerateTextScreenProfanity()`| `VideoReviewAPI.cs`|
|Preparar y enviar una solicitud de revisión de vídeo para su inspección humana|`CreateReviewRequestObject()`<br> `ExecuteCreateReviewApi()`<br>`CreateAndPublishReviewInContentModerator()`|`VideoReviewAPI.cs`|

La captura de pantalla siguiente muestra los resultados de los pasos anteriores.

![Vista predeterminada de la revisión de vídeo](images/video-tutorial-default-view.PNG)

## <a name="process-the-transcript"></a>Procesar la transcripción

Hasta ahora, el código presentado en este tutorial se ha centrado en el contenido visual. La revisión del contenido de voz es un proceso independiente y opcional que, como se ha mencionado, usa una transcripción generada a partir del audio. Ahora es el momento de revisar cómo se crean las transcripciones de texto y cómo se usan en el proceso de revisión. La tarea de generar la transcripción recae en el servicio [Azure Media Indexer](https://docs.microsoft.com/azure/media-services/media-services-index-content).

La aplicación realiza las tareas siguientes:

|Tarea|Métodos|Archivo|
|-|-|-|
|Determinar si se deben generar transcripciones de texto|`Main()`<br>`GetUserInputs()`|`Program.cs`|
|Si es así, enviar un trabajo de transcripción como parte de la moderación|`ConfigureTranscriptTask()`|`VideoModerator.cs`|
|Obtener una copia local de la transcripción|`GenerateTranscript()`|`VideoModerator.cs`|
|Marcar los fotogramas del vídeo que contienen audio inapropiado|`GenerateTextScreenProfanity()`<br>`TextScreen()`|`VideoReviewAPI.cs`|
|Agregar los resultados a la revisión|`UploadScreenTextResult()`<br>`ExecuteAddTranscriptSupportFile()`|`VideoReviewAPI.cs`|

### <a name="task-configuration"></a>Configuración de tareas

Vamos a ver cómo se envía el trabajo de transcripción. `CreateAzureMediaServicesJobToModerateVideo()` (ya descrito) llama a `ConfigureTranscriptTask()`.

[!code-csharp[ConfigureTranscriptTask](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/VideoModerator.cs?range=295-304)]

La configuración de la tarea de transcripción se lee desde el archivo `MediaIndexerConfig.json` en la carpeta `Lib` de la solución. Se crean recursos de AMS para el archivo de configuración y para la salida del proceso de transcripción. Cuando se ejecuta el trabajo de AMS, esta tarea crea una transcripción de texto a partir de la pista de audio del archivo de vídeo.

> [!NOTE]
> La aplicación de ejemplo reconoce solo voz en inglés de Estados Unidos.

### <a name="transcript-generation"></a>Generación de transcripción

La transcripción se publica como un recurso de AMS. Para examinar si existe contenido inapropiado en la transcripción, la aplicación descarga el recurso de Azure Media Services. `CreateAzureMediaServicesJobToModerateVideo()` llama a `GenerateTranscript()`, como se muestra aquí, para recuperar el archivo.

[!code-csharp[GenerateTranscript](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/VideoModerator.cs?range=351-370)]

Después de algunas configuraciones necesarias de Azure Media Services, la descarga se realiza efectivamente llamando a `DownloadAssetToLocal()`, una función genérica que copia un recurso de Azure Media Services en un archivo local.

## <a name="moderate-the-transcript"></a>Moderar la transcripción

Con la transcripción a mano, se analiza y se utiliza en la revisión. La creación de la revisión pertenece al ámbito de `CreateVideoReviewInContentModerator()`, que llama a `GenerateTextScreenProfanity()` para realizar el trabajo. A su vez, este método llama a `TextScreen()`, que contiene la mayor parte de la funcionalidad.

`TextScreen()` realiza las siguientes tareas:

- Analizar la transcripción para ver si hay marcas de tiempo y títulos
- Enviar cada título a la moderación de texto
- Marcar los fotogramas que pueden tener contenido de voz inapropiado

Vamos a examinar cada una estas tareas con más detalle:

### <a name="initialize-the-code"></a>Inicialización del código

En primer lugar, inicialice todas las variables y colecciones.

[!code-csharp[TextScreen](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/VideoReviewAPI.cs?range=515-527)]

### <a name="parse-the-transcript-for-captions"></a>Análisis de la transcripción para detectar títulos

A continuación, analice la transcripción en formato VTT para ver si contiene títulos y marcas de tiempo. La herramienta de revisión muestra estos títulos en la pestaña de transcripción en la pantalla de revisión de vídeo. Las marcas de tiempo se usan para sincronizar los títulos con los fotogramas de vídeo correspondientes.

[!code-csharp[TextScreen2](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/VideoReviewAPI.cs?range=528-567)]

### <a name="moderate-captions-with-the-text-moderation-service"></a>Moderación de los títulos con el servicio de moderación de texto

A continuación, se analizan los títulos de texto analizados con la API de texto de Content Moderator.

> [!NOTE]
> La clave de servicio de Content Moderator tiene un límite de frecuencia de solicitudes por segundo (RPS). Si se supera el límite, el SDK produce una excepción con un código de error 429.
>
> Una clave de un plan gratuito tiene un límite de una solicitud por segundo.

[!code-csharp[TextScreen3](~/VideoReviewConsoleApp/Microsoft.ContentModerator.AMSComponent/AMSComponentClient/VideoReviewAPI.cs?range=568-653)]

### <a name="text-moderation-breakdown"></a>Desglose de la moderación de texto

`TextScreen()` es un método considerable, así que vamos a verlo por partes.

1. En primer lugar, el método lee el archivo de transcripción línea por línea. Omite las líneas en blanco y las líneas que contienen un `NOTE` con una puntuación de confianza. Extrae las marcas de tiempo y los elementos de texto de las *indicaciones* del archivo. Una indicación representa el texto de la pista de audio e incluye las horas de inicio y finalización. Una indicación comienza con la línea de la marca de tiempo con la cadena `-->`. Le sigue una o más líneas de texto.

1. Las instancias de `CaptionScreentextResult` (definidas en `TranscriptProfanity.cs`) se utilizan para contener la información que se analiza de cada indicación.  Cuando se detecta una nueva línea de marca de tiempo o se alcanza una longitud máxima de texto de 1024 caracteres, se agrega un nuevo `CaptionScreentextResult` a `csrList`. 

1. A continuación, el método envía cada indicación a la API de moderación de texto. Llama a `ContentModeratorClient.TextModeration.DetectLanguageAsync()` y `ContentModeratorClient.TextModeration.ScreenTextWithHttpMessagesAsync()`, que se definen en el ensamblado `Microsoft.Azure.CognitiveServices.ContentModerator`. Para evitar la limitación de velocidad, el método hace una pausa de un segundo antes de enviar cada indicación.

1. Después de recibir los resultados del servicio de moderación de texto, el método los analiza para ver si cumplen los umbrales de confianza. Estos valores se establecen en `App.config` como `OffensiveTextThreshold`, `RacyTextThreshold` y `AdultTextThreshold`. Por último, también se almacenan los propios términos inapropiados. Todos los fotogramas dentro de intervalo de tiempo de la indicación se marcan como que contienen material ofensivo o para adultos.

1. `TextScreen()` devuelve una instancia de `TranscriptScreenTextResult` que contiene el resultado de moderación de texto a partir del vídeo como un todo. Este objeto incluye marcas y puntuaciones para los distintos tipos de contenido inapropiado, junto con una lista de todos los términos inapropiados. El autor de la llamada, `CreateVideoReviewInContentModerator()`, llama a `UploadScreenTextResult()` para asociar esta información a la revisión para que esté disponible para los revisores humanos.

La siguiente pantalla muestra el resultado de los pasos de generación y moderación de la transcripción.

![Vista de la transcripción de la moderación de vídeo](images/video-tutorial-transcript-view.PNG)

## <a name="program-output"></a>Salida del programa

La siguiente salida de línea de comandos del programa muestra las distintas tareas a medida que se completan. Además, el resultado de moderación (en formato JSON) y la transcripción de voz están disponibles en el mismo directorio que los archivos de vídeo originales.

```console
Microsoft.ContentModerator.AMSComponentClient
Enter the fully qualified local path for Uploading the video :
"Your File Name.MP4"
Generate Video Transcript? [y/n] : y

Video compression process started...
Video compression process completed...

Video moderation process started...
Video moderation process completed...

Video review process started...
Video Frames Creation inprogress...
Frames(83) created successfully.
Review Created Successfully and the review Id 201801va8ec2108d6e043229ba7a9e6373edec5
Video review successfully completed...

Total Elapsed Time: 00:05:56.8420355
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, configuró una aplicación que modera el contenido de vídeo, &mdash;incluido el contenido de transcripción&mdash;, y crea revisiones en la herramienta de revisión. A continuación, obtenga más información sobre los detalles de la moderación de vídeo.

> [!div class="nextstepaction"]
> [Moderación de vídeo](./video-moderation-human-review.md)
