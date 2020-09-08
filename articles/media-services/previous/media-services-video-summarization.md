---
title: Usar Miniaturas de vídeo multimedia de Azure para crear un resumen de vídeo | Microsoft Docs
description: El resumen de vídeo puede ayudarle a crear resúmenes de vídeos largos al seleccionar automáticamente fragmentos interesantes del vídeo original. Esto es útil si quiere proporcionar una rápida descripción de lo que se va a encontrar en un vídeo largo.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: a245529f-3150-4afc-93ec-e40d8a6b761d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.reviewer: milanga
ms.custom: devx-track-csharp
ms.openlocfilehash: e6810e20373aa6fcaab447d6b6be51a47f400e24
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89268594"
---
# <a name="use-azure-media-video-thumbnails-to-create-a-video-summarization"></a>Usar Miniaturas de vídeo multimedia de Azure para crear un resumen de vídeo

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> El procesador de multimedia **Azure Media Video Thumbnails** se va a retirar. Para ver la fecha de retirada, consulte el tema sobre los [componentes heredados](legacy-components.md).

## <a name="overview"></a>Información general

El procesador de multimedia (PM) **Miniaturas de vídeo multimedia de Azure** le permite crear un resumen de un vídeo útil para aquellos clientes que solo quieren ver un resumen de un vídeo largo. Por ejemplo, es posible que los clientes quieran ver un breve "vídeo de resumen" al mantener el mouse sobre una miniatura. Al ajustar los parámetros de **Miniaturas de vídeo multimedia de Azure** mediante un valor predeterminado de configuración, puede usar la eficaz tecnología de detección de tomas y concatenación del PM para generar de forma algorítmica un subclip descriptivo.  

El PM **Miniatura de vídeo multimedia de Azure** está actualmente en versión preliminar.

En este artículo se proporcionan detalles sobre **Miniatura de vídeo multimedia de Azure** y se muestra cómo se usa con el SDK de Media Services para .NET.

## <a name="limitations"></a>Limitaciones

En algunos casos, si el vídeo no está formado por diferentes escenas, el resultado solo será una fotografía.

## <a name="video-summary-example"></a>Ejemplo de resumen de vídeo
Estos son algunos ejemplos de lo que puede hacer el procesador de multimedia Miniaturas de vídeo multimedia de Azure:

### <a name="original-video"></a>Vídeo original
[Vídeo original](https://ampdemo.azureedge.net/azuremediaplayer.html?url=httpss%3A%2F%2Fnimbuscdn-nimbuspm.streaming.mediaservices.windows.net%2Faed33834-ec2d-4788-88b5-a4505b3d032c%2FMicrosoft%27s%20HoloLens%20Live%20Demonstration.ism%2Fmanifest)

### <a name="video-thumbnail-result"></a>Resultado de miniaturas de vídeo
[Resultado de miniaturas de vídeo](https://ampdemo.azureedge.net/azuremediaplayer.html?url=https%3A%2F%2Fnimbuscdn-nimbuspm.streaming.mediaservices.windows.net%2Ff5c91052-4232-41d4-b531-062e07b6a9ae%2FHololens%2520Demo_VideoThumbnails_MotionThumbnail.mp4)

## <a name="task-configuration-preset"></a>Configuración de tareas (valor predeterminado)
Al crear una tarea de miniatura de vídeo con **Miniaturas de vídeo multimedia de Azure**, debe especificar un valor predeterminado de configuración. El ejemplo de miniatura anterior se creó con la siguiente configuración básica de JSON:

```json
    {
        "version":"1.0"
    }
```

En este momento puede modificar los siguientes parámetros:

| Parámetro | Descripción |
| --- | --- |
| outputAudio |Especifica si el vídeo resultante contiene audio o no. <br/>Los valores permitidos son: True o False. El valor predeterminado es True. |
| fadeInFadeOut |Especifica si se usan transiciones de fundido entre las distintas miniaturas de movimiento.  <br/>Los valores permitidos son: True o False.  El valor predeterminado es True. |
| maxMotionThumbnailDurationInSecs |Entero que especifica la duración de todo el vídeo resultante.  El valor predeterminado depende de la duración del vídeo original. |

En la tabla siguiente se describe la duración predeterminada cuando no se usa **maxMotionThumbnailInSecs** .

|  | Pequeña | Mayor |
| --- | --- | --- |
| **Duración del vídeo** |d < 3 min |3 min < d < 15 min |
| **Duración de la miniatura** |15 s (2-3 escenas) |30 s (3-5 escenas) |

El siguiente JSON establece los parámetros disponibles.

```json
    {
        "version": "1.0",
        "options": {
            "outputAudio": "true",
            "maxMotionThumbnailDurationInSecs": "10",
            "fadeInFadeOut": "true"
        }
    }
```

## <a name="net-sample-code"></a>Código de ejemplo de .NET

El programa siguiente muestra cómo:

1. Crear un recurso y cargar un archivo multimedia en él.
2. Crea un trabajo con una tarea de miniatura de vídeo basada en un archivo de configuración que contiene el siguiente valor predeterminado de JSON: 
    
    ```json
            {                
                "version": "1.0",
                "options": {
                    "outputAudio": "true",
                    "maxMotionThumbnailDurationInSecs": "30",
                    "fadeInFadeOut": "false"
                }
            }
    ```

3. Descargar los archivos de salida 

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

    namespace VideoSummarization
    {
        class Program
        {
            // Read values from the App.config file.
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


                // Run the thumbnail job.
                var asset = RunVideoThumbnailJob(@"C:\supportFiles\VideoThumbnail\BigBuckBunny.mp4",
                                            @"C:\supportFiles\VideoThumbnail\config.json");

                // Download the job output asset.
                DownloadAsset(asset, @"C:\supportFiles\VideoThumbnail\Output");
            }

            static IAsset RunVideoThumbnailJob(string inputMediaFilePath, string configurationFile)
            {
                // Create an asset and upload the input media file to storage.
                IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
                    "My Video Thumbnail Input Asset",
                    AssetCreationOptions.None);

                // Declare a new job.
                IJob job = _context.Jobs.Create("My Video Thumbnail Job");

                // Get a reference to Azure Media Video Thumbnails.
                string MediaProcessorName = "Azure Media Video Thumbnails";

                var processor = GetLatestMediaProcessorByName(MediaProcessorName);

                // Read configuration from the specified file.
                string configuration = File.ReadAllText(configurationFile);

                // Create a task with the encoding details, using a string preset.
                ITask task = job.Tasks.AddNew("My Video Thumbnail Task",
                    processor,
                    configuration,
                    TaskOptions.None);

                // Specify the input asset.
                task.InputAssets.Add(asset);

                // Add an output asset to contain the results of the job.
                task.OutputAssets.AddNew("My Video Thumbnail Output Asset", AssetCreationOptions.None);

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

### <a name="video-thumbnail-output"></a>Resultado de miniaturas de vídeo
[Resultado de miniaturas de vídeo](https://ampdemo.azureedge.net/azuremediaplayer.html?url=https%3A%2F%2Fnimbuscdn-nimbuspm.streaming.mediaservices.windows.net%2Fd06f24dc-bc81-488e-a8d0-348b7dc41b56%2FHololens%2520Demo_VideoThumbnails_MotionThumbnail.mp4)

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>Vínculos relacionados
[Información general de análisis de Azure Media Services](media-services-analytics-overview.md)

[Demostraciones de Azure Media Analytics](https://azuremedialabs.azurewebsites.net/demos/Analytics.html)

