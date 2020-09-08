---
title: Detección de caras y emociones con Análisis multimedia de Azure | Microsoft Docs
description: Este tema muestra cómo detectar caras y emociones con Análisis multimedia de Azure.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 5ca4692c-23f1-451d-9d82-cbc8bf0fd707
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.reviewer: milanga
ms.custom: devx-track-csharp
ms.openlocfilehash: ac27fa5773e266d546b6511a24c6a5e14c5fda7f
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89257847"
---
# <a name="detect-face-and-emotion-with-azure-media-analytics"></a>Detección de caras y emociones con Análisis multimedia de Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> El procesador de multimedia **Azure Media Face Detector** se va a retirar. Para ver la fecha de retirada, consulte el tema sobre los [componentes heredados](legacy-components.md).

## <a name="overview"></a>Información general

El procesador de multimedia (MP) **Azure Media Face Detector** (Detector de caras multimedia de Azure) le permite contar y seguir los movimientos, e incluso evaluar la participación y reacción de la audiencia a través de expresiones faciales. Este servicio contiene dos características: 

* **Detección de caras**
  
    La detección de caras busca y realiza el seguimiento de caras humanas dentro de un vídeo. Se pueden detectar varias caras y posteriormente realizar un seguimiento de las mismas a medida que se mueven, con los metadatos de tiempo y ubicación devueltos en un archivo JSON. Durante el seguimiento, intentará dar un identificador constante a la misma cara mientras la persona se mueve alrededor de la pantalla, aunque se tape o salga brevemente del fotograma.
  
  > [!NOTE]
  > Este servicio no lleva a cabo el reconocimiento facial. Si una persona abandona el fotograma o se tapa durante demasiado tiempo se le asigna un nuevo identificador cuando regresa.
  > 
  > 
* **Detección de emociones**
  
    Detección de emociones es un componente opcional del procesador de multimedia de detección de caras que devuelve análisis sobre varios atributos emocionales a partir de las caras detectadas, como felicidad, tristeza, temor, enojo, etc. 

El procesador de multimedia **Azure Media Face Detector** (Detector de caras multimedia de Azure) está actualmente en versión preliminar.

En este artículo se proporcionan detalles sobre el **Azure Media Face Detector** y muestra cómo se usa con el SDK de Media Services para .NET.

## <a name="face-detector-input-files"></a>Archivos de entrada de Face Detector (Detector de caras)
Archivos de vídeo. Actualmente, se admiten los siguientes formatos: MP4, MOV y WMV.

## <a name="face-detector-output-files"></a>Archivos de salida de Face Detector (Detector de caras)
La API de detección y seguimiento de caras proporciona detección y seguimiento de caras de alta precisión que puede detectar hasta 64 caras humanas en un vídeo. Las caras de frente ofrecen los mejores resultados, mientras que las caras de lado y las caras pequeñas (inferiores o iguales a 24x24 píxeles) podrían no ser tan precisas.

Las caras detectadas y seguidas se devuelven con coordenadas (izquierda, superior, ancho y alto) que indican la ubicación de caras en la imagen en píxeles, así como un número de identificación de cara que indica el seguimiento de esa persona. Los números de identificación de cara son propensos a restablecerse en circunstancias en las que la cara de frente se pierde o se superpone en el fotograma, lo que provoca que a algunas personas se les asigne varios identificadores.

## <a name="elements-of-the-output-json-file"></a><a id="output_elements"></a>Elementos del archivo JSON de salida

[!INCLUDE [media-services-analytics-output-json](../../../includes/media-services-analytics-output-json.md)]

Face Detector (Detector de caras) usa técnicas de fragmentación (donde los metadatos se pueden dividir en fragmentos según el tiempo y puede descargar solo lo que necesita) y segmentación (donde los eventos se dividen en caso de que aumenten demasiado). Algunos cálculos sencillos pueden ayudarle a transformar los datos. Por ejemplo, si un evento se inició en 6300 (tics), con una escala de tiempo de 2997 (tics/segundo) y una velocidad de fotogramas de 29,97 (fotogramas/segundo), entonces:

* Inicio/Escala de tiempo = 2,1 segundos
* Segundos x Framerate = 63 marcos

## <a name="face-detection-input-and-output-example"></a>Ejemplo de entrada y salida de detección de caras
### <a name="input-video"></a>Vídeo de entrada
[Vídeo de entrada](https://ampdemo.azureedge.net/azuremediaplayer.html?url=httpss%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc8834d9f-0b49-4b38-bcaf-ece2746f1972%2FMicrosoft%20Convergence%202015%20%20Keynote%20Highlights.ism%2Fmanifest&amp;autoplay=false)

### <a name="task-configuration-preset"></a>Configuración de tareas (valor predeterminado)
Al crear una tarea con **Azure Media Face Detector**(Detector de caras multimedia de Azure), debe especificar un valor predeterminado de configuración. El siguiente valor predeterminado de configuración es solo para detección de caras.

```json
    {
      "version":"1.0",
      "options":{
          "TrackingMode": "Fast"
      }
    }
```

#### <a name="attribute-descriptions"></a>Descripciones de atributos
| Nombre del atributo | Descripción |
| --- | --- |
| Mode |Rápido: procesamiento rápido, pero menos preciso (valor predeterminado).|

### <a name="json-output"></a>Salida de JSON
El siguiente ejemplo de salida de JSON se truncó.

```json
    {
    "version": 1,
    "timescale": 30000,
    "offset": 0,
    "framerate": 29.97,
    "width": 1280,
    "height": 720,
    "fragments": [
        {
        "start": 0,
        "duration": 60060
        },
        {
        "start": 60060,
        "duration": 60060,
        "interval": 1001,
        "events": [
            [
            {
                "id": 0,
                "x": 0.519531,
                "y": 0.180556,
                "width": 0.0867188,
                "height": 0.154167
            }
            ],
            [
            {
                "id": 0,
                "x": 0.517969,
                "y": 0.181944,
                "width": 0.0867188,
                "height": 0.154167
            }
            ],
            [
            {
                "id": 0,
                "x": 0.517187,
                "y": 0.183333,
                "width": 0.0851562,
                "height": 0.151389
            }
            ],
```


## <a name="emotion-detection-input-and-output-example"></a>Ejemplo de entrada y salida de detección de emociones
### <a name="input-video"></a>Vídeo de entrada
[Vídeo de entrada](https://ampdemo.azureedge.net/azuremediaplayer.html?url=httpss%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc8834d9f-0b49-4b38-bcaf-ece2746f1972%2FMicrosoft%20Convergence%202015%20%20Keynote%20Highlights.ism%2Fmanifest&amp;autoplay=false)

### <a name="task-configuration-preset"></a>Configuración de tareas (valor predeterminado)
Al crear una tarea con **Azure Media Face Detector**(Detector de caras multimedia de Azure), debe especificar un valor predeterminado de configuración. El siguiente valor predeterminado de configuración especifica la creación de JSON basándose en la detección de emociones.

```json
    {
      "version": "1.0",
      "options": {
        "aggregateEmotionWindowMs": "987",
        "mode": "aggregateEmotion",
        "aggregateEmotionIntervalMs": "342"
      }
    }
```


#### <a name="attribute-descriptions"></a>Descripciones de atributos
| Nombre del atributo | Descripción |
| --- | --- |
| Mode |Faces: Solo detección de caras.<br/>PerFaceEmotion: Devuelve emociones independientemente para cada detección de cara.<br/>AggregateEmotion: Devuelve valores de emociones medios para todas las caras del fotograma. |
| AggregateEmotionWindowMs |Utilizar si se selecciona el modo AggregateEmotion. Especifica la longitud del vídeo que se utiliza para generar cada resultado agregado, en milisegundos. |
| AggregateEmotionIntervalMs |Utilizar si se selecciona el modo AggregateEmotion. Especifica con qué frecuencia se van a producir resultados agregados. |

#### <a name="aggregate-defaults"></a>Agregar valores predeterminados
A continuación figuran valores recomendados para la configuración de ventana e intervalo agregada. AggregateEmotionWindowMs debe ser mayor que AggregateEmotionIntervalMs.

|| Valores predeterminados | Máximos | Mínimos |
|--- | --- | --- | --- |
| **AggregateEmotionWindowMs** |0.5 |2 |0,25|
| **AggregateEmotionIntervalMs** |0.5 |1 |0,25|

### <a name="json-output"></a>Salida de JSON
Salida de JSON para la emoción agregada (truncada):

```json
    {
     "version": 1,
     "timescale": 30000,
     "offset": 0,
     "framerate": 29.97,
     "width": 1280,
     "height": 720,
     "fragments": [
       {
         "start": 0,
         "duration": 60060,
         "interval": 15015,
         "events": [
           [
             {
               "windowFaceDistribution": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               },
               "windowMeanScores": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               }
             }
           ],
           [
             {
               "windowFaceDistribution": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               },
               "windowMeanScores": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               }
             }
           ],
           [
             {
               "windowFaceDistribution": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               },
               "windowMeanScores": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               }
             }
           ],
           [
             {
               "windowFaceDistribution": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               },
               "windowMeanScores": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               }
             }
           ]
         ]
       },
       {
         "start": 60060,
         "duration": 60060,
         "interval": 15015,
         "events": [
           [
             {
               "windowFaceDistribution": {
                 "neutral": 1,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               },
               "windowMeanScores": {
                 "neutral": 0.688541,
                 "happiness": 0.0586323,
                 "surprise": 0.227184,
                 "sadness": 0.00945675,
                 "anger": 0.00592107,
                 "disgust": 0.00154993,
                 "fear": 0.00450447,
                 "contempt": 0.0042109
               }
             }
           ],
           [
             {
               "windowFaceDistribution": {
                 "neutral": 1,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
```

## <a name="limitations"></a>Limitaciones
* Los formatos de vídeo de entrada admitidos incluyen MP4, MOV y WMV.
* El intervalo de tamaños de cara detectable es 24x24 a 2048x2048 píxeles. Las caras que se encuentren fuera de este intervalo no se detectarán.
* Para cada vídeo, el número máximo de caras devuelto es 64.
* Es posible que algunas caras no se detecten debido a los desafíos técnicos; por ejemplo, ángulos de cara muy grandes (poses) y oclusión grande. Las caras de frente y de frente cercanas tienen los mejores resultados.

## <a name="net-sample-code"></a>Código de ejemplo de .NET

El programa siguiente muestra cómo:

1. Crear un recurso y cargar un archivo multimedia en él.
2. Crear un trabajo con una tarea de detección de caras basándose en un archivo de configuración que contiene el siguiente valor predeterminado de JSON: 

    ```json
            {
                "version": "1.0"
            }
    ```
3. Descargar los archivos JSON de salida. 

#### <a name="create-and-configure-a-visual-studio-project"></a>Creación y configuración de un proyecto de Visual Studio

Configure el entorno de desarrollo y rellene el archivo app.config con la información de la conexión, como se describe en [Desarrollo de Media Services con .NET](media-services-dotnet-how-to-use.md). 

#### <a name="example"></a>Ejemplo

```csharp
using System;
using System.Configuration;
using System.IO;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Threading;
using System.Threading.Tasks;

namespace FaceDetection
{
    class Program
    {
        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        // Field for service context.
        private static CloudMediaContext _context = null;

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            // Run the FaceDetection job.
            var asset = RunFaceDetectionJob(@"C:\supportFiles\FaceDetection\BigBuckBunny.mp4",
                                        @"C:\supportFiles\FaceDetection\config.json");

            // Download the job output asset.
            DownloadAsset(asset, @"C:\supportFiles\FaceDetection\Output");
        }

        static IAsset RunFaceDetectionJob(string inputMediaFilePath, string configurationFile)
        {
            // Create an asset and upload the input media file to storage.
            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
                "My Face Detection Input Asset",
                AssetCreationOptions.None);

            // Declare a new job.
            IJob job = _context.Jobs.Create("My Face Detection Job");

            // Get a reference to Azure Media Face Detector.
            string MediaProcessorName = "Azure Media Face Detector";

            var processor = GetLatestMediaProcessorByName(MediaProcessorName);

            // Read configuration from the specified file.
            string configuration = File.ReadAllText(configurationFile);

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("My Face Detection Task",
                processor,
                configuration,
                TaskOptions.None);

            // Specify the input asset.
            task.InputAssets.Add(asset);

            // Add an output asset to contain the results of the job.
            task.OutputAssets.AddNew("My Face Detection Output Asset", AssetCreationOptions.None);

            // Use the following event handler to check job progress.  
            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);

            // Launch the job.
            job.Submit();

            // Check job execution and wait for job to finish.
            Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);

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
                return null;
            }

            return job.OutputMediaAssets[0];
        }

        static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
        {
            IAsset asset = _context.Assets.Create(assetName, options);

            var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
            assetFile.Upload(filePath);

            return asset;
        }

        static void DownloadAsset(IAsset asset, string outputDirectory)
        {
            foreach (IAssetFile file in asset.AssetFiles)
            {
                file.Download(Path.Combine(outputDirectory, file.Name));
            }
        }

        static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
        {
            var processor = _context.MediaProcessors
                .Where(p => p.Name == mediaProcessorName)
                .ToList()
                .OrderBy(p => new Version(p.Version))
                .LastOrDefault();

            if (processor == null)
                throw new ArgumentException(string.Format("Unknown media processor",
                                                           mediaProcessorName));

            return processor;
        }

        static private void StateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine("Job state changed event:");
            Console.WriteLine("  Previous state: " + e.PreviousState);
            Console.WriteLine("  Current state: " + e.CurrentState);

            switch (e.CurrentState)
            {
                case JobState.Finished:
                    Console.WriteLine();
                    Console.WriteLine("Job is finished.");
                    Console.WriteLine();
                    break;
                case JobState.Canceling:
                case JobState.Queued:
                case JobState.Scheduled:
                case JobState.Processing:
                    Console.WriteLine("Please wait...\n");
                    break;
                case JobState.Canceled:
                case JobState.Error:
                    // Cast sender as a job.
                    IJob job = (IJob)sender;
                    // Display or log error details as needed.
                    // LogJobStop(job.Id);
                    break;
                default:
                    break;
            }
        }
    }
}
```

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>Vínculos relacionados
[Información general de análisis de Azure Media Services](media-services-analytics-overview.md)

[Demostraciones de Azure Media Analytics](https://amslabs.azurewebsites.net/demos/Analytics.html)

