---
title: Usar Azure Media Content Moderator para detectar contenido para adultos | Documentos de Microsoft
description: El procesador multimedia Azure Media Content Moderator ayuda a detectar posible contenido para adultos en los vídeos.
services: media-services
documentationcenter: ''
author: sanjeev3
manager: mikemcca
editor: ''
ms.assetid: a245529f-3150-4afc-93ec-e40d8a6b761d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/14/2019
ms.author: sajagtap
ms.custom: devx-track-csharp
ms.openlocfilehash: 395d2239970e7efbf6973d6262df0e049306584b
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266704"
---
# <a name="use-azure-media-content-moderator-to-detect-possible-adult-and-racy-content"></a>Usar Azure Media Content Moderator para detectar contenido para adultos

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Se va a retirar el procesador de multimedia de **Azure Media Content Moderator**. Para conocer la fecha de retirada, consulte el tema sobre los [componentes heredados](legacy-components.md).

## <a name="overview"></a>Información general
El procesador multimedia (MP) **Azure Media Content Moderator** le permite usar la moderación automatizada en sus vídeos. Por ejemplo, puede usarlo para detectar posible contenido para adultos en los vídeos y revisar el contenido que hayan indicado sus equipos de moderadores.

Tenga en cuenta que el procesador multimedia **Azure Media Content Moderator** está actualmente en versión preliminar.

En este artículo se proporcionan diversos detalles acerca de **Azure Media Content Moderator** y muestra cómo se usa con el SDK de Media Services para .NET.

## <a name="content-moderator-input-files"></a>Archivos de entrada de Content Moderator
Archivos de vídeo. Actualmente, se admiten los siguientes formatos: MP4, MOV y WMV.

## <a name="content-moderator-output-files"></a>Archivos de salida de Content Moderator
Los archivos de salida moderados en formato JSON incluyen fotogramas clave y capturas que se han detectado de forma automática. Los fotogramas clave se devuelven con una serie de puntuaciones de confianza para indicar el posible contenido para adultos. Asimismo, también se incluye una marca booleana que indica si es recomendable realizar una revisión. Se asignan valores a la marca de recomendación para realizar la revisión en función de los umbrales internos establecidos según las puntuaciones del contenido para adultos.

## <a name="elements-of-the-output-json-file"></a>Elementos del archivo JSON de salida

El trabajo genera un archivo de salida JSON que contiene metadatos acerca de las capturas y fotogramas clave detectados e indica si existe contenido para adultos.

La salida JSON incluye los siguientes elementos:

### <a name="root-json-elements"></a>Elementos raíz JSON

| Elemento | Descripción |
| --- | --- |
| version |La versión de Content Moderator. |
| timescale |"Tics" por segundo del vídeo. |
| offset |La diferencia horaria de las marcas de tiempo. En la versión 1.0 de las Video API, el valor será siempre 0. Tenga en cuenta que este valor puede cambiar en el futuro. |
| framerate |Fotogramas por segundo del vídeo. |
| width |El ancho del fotograma de vídeo de salida, en píxeles.|
| height |La altura del fotograma de vídeo de salida, en píxeles.|
| totalDuration |La duración de la entrada de vídeo, en "tics". |
| [fragments](#fragments-json-elements) |Los metadatos se separan en diferentes segmentos denominados fragmentos. Cada fragmento se compone de una captura detectada automáticamente que consta de un inicio, una duración, un número de intervalo y unos eventos. |

### <a name="fragments-json-elements"></a>Elementos JSON de fragmentos

|Elemento|Descripción|
|---|---|
| start |La hora de inicio del primer evento, en "tics". |
| duration |La longitud del fragmento, en "tics". |
| interval |El intervalo de cada entrada de evento dentro del fragmento, en "tics". |
| [eventos](#events-json-elements) |Cada evento representa un clip y cada clip contiene los fotogramas clave que se hayan detectado y seguido dentro del período de tiempo especificado. Es una matriz de eventos. La matriz externa representa un intervalo de tiempo. La matriz interna consta de 0 o más eventos que ocurrieron en ese momento en el tiempo.|

### <a name="events-json-elements"></a>Elementos de eventos JSON

|Elemento|Descripción|
|---|---|
| reviewRecommended | `true` o `false`, dependiendo de si **adultScore** o **racyScore** superan los umbrales internos. |
| adultScore | Puntuación de confianza del posible contenido para adultos, en una escala del 0,00 al 0,99. |
| racyScore | Puntuación de confianza del posible contenido obsceno, en una escala del 0,00 al 0,99. |
| índice | Índice del marco en una escala que va desde el primer hasta el último índice de fotograma. |
| timestamp | La ubicación del marco en "tics". |
| shotIndex | El índice de la captura principal. |


## <a name="content-moderation-quickstart-and-sample-output"></a>Guía de inicio rápido de Content Moderation y salida de ejemplos

### <a name="task-configuration-preset"></a>Configuración de tareas (valor predeterminado)
Cuando cree una tarea con **Azure Media Content Moderator**, tiene que especificar una configuración preestablecida. El siguiente valor predeterminado de configuración es solo para la moderación de contenido.

```json
{
    "version":"2.0"
}
```

### <a name="net-code-sample"></a>Ejemplo de código de .NET

En el siguiente ejemplo de código de .NET se usa el SDK de .NET de Media Services para ejecutar un trabajo de Content Moderator. Es necesario tener un recurso de Media Services a modo de entrada que contenga el vídeo que se va a moderar.
Consulte la [guía de inicio rápido para vídeos de Content Moderator](../../cognitive-services/Content-Moderator/video-moderation-api.md) para obtener el código fuente y el proyecto de Visual Studio completos.


```csharp
/// <summary>
/// Run the Content Moderator job on the designated Asset from local file or blob storage
/// </summary>
/// <param name="asset"></param>
static void RunContentModeratorJob(IAsset asset)
{
    // Grab the presets
    string configuration = File.ReadAllText(CONTENT_MODERATOR_PRESET_FILE);

    // grab instance of Azure Media Content Moderator MP
    IMediaProcessor mp = _context.MediaProcessors.GetLatestMediaProcessorByName(MEDIA_PROCESSOR);

    // create Job with Content Moderator task
    IJob job = _context.Jobs.Create(String.Format("Content Moderator {0}",
               asset.AssetFiles.First() + "_" + Guid.NewGuid()));

    ITask contentModeratorTask = job.Tasks.AddNew("Adult and racy classifier task",
                                                  mp, configuration,
                                                  TaskOptions.None);
    contentModeratorTask.InputAssets.Add(asset);
    contentModeratorTask.OutputAssets.AddNew("Adult and racy classifier output",
                                             AssetCreationOptions.None);

    job.Submit();


    // Create progress printing and querying tasks
    Task progressPrintTask = new Task(() =>
    {
        IJob jobQuery = null;
        do
        {
            var progressContext = _context;
            jobQuery = progressContext.Jobs
                .Where(j => j.Id == job.Id)
                .First();
            Console.WriteLine(string.Format("{0}\t{1}",
                                            DateTime.Now,
                                            jobQuery.State));
            Thread.Sleep(10000);
        }
        while (jobQuery.State != JobState.Finished &&
               jobQuery.State != JobState.Error &&
               jobQuery.State != JobState.Canceled);
    });
    progressPrintTask.Start();

    Task progressJobTask = job.GetExecutionProgressTask(
                           CancellationToken.None);
    progressJobTask.Wait();

    // If job state is Error, the event handling 
    // method for job progress should log errors.  Here we check 
    // for error state and exit if needed.
    if (job.State == JobState.Error)
    {
        ErrorDetail error = job.Tasks.First().ErrorDetails.First();
        Console.WriteLine(string.Format("Error: {0}. {1}",
                          error.Code,
                          error.Message));
    }

    DownloadAsset(job.OutputMediaAssets.First(), OUTPUT_FOLDER);
}
```

Para obtener el código fuente y el proyecto de Visual Studio completos, consulte la [guía de inicio rápido para vídeos de Content Moderator](../../cognitive-services/Content-Moderator/video-moderation-api.md).

### <a name="json-output"></a>Salida de JSON

En el siguiente ejemplo se muestra un elemento de salida JSON de Content Moderator que se truncó.

> [!NOTE]
> Ubicación de un fotograma clave en segundos = marca de tiempo/escala temporal

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

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>Vínculos relacionados
[Información general de análisis de Azure Media Services](media-services-analytics-overview.md)

[Demostraciones de Azure Media Analytics](https://azuremedialabs.azurewebsites.net/demos/Analytics.html)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información acerca de la [solución para la moderación y revisión de vídeos](../../cognitive-services/Content-Moderator/video-moderation-human-review.md) de Content Moderator.

Para obtener el código fuente y el proyecto de Visual Studio completos, consulte la [guía de inicio rápido para la moderación de vídeos](../../cognitive-services/Content-Moderator/video-moderation-api.md). 

Aprenda a crear [revisiones de vídeos](../../cognitive-services/Content-Moderator/video-reviews-quickstart-dotnet.md) a partir de los elementos salida moderados y las [transcripciones moderadas](../../cognitive-services/Content-Moderator/video-transcript-reviews-quickstart-dotnet.md) en. NET.

Consulte el [tutorial de moderación y revisión de vídeos](../../cognitive-services/Content-Moderator/video-transcript-moderation-review-tutorial-dotnet.md) de .NET. 
