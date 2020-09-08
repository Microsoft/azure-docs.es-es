---
title: Censura de rostros con Azure Media Analytics | Microsoft Docs
description: Azure Media Redactor es un procesador multimedia (MP) de Azure Media Analytics que ofrece censura de rostros escalable en la nube. En este artículo se muestra cómo censurar caras con Azure Media Analytics.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 9a562e31a26cc6cf7188dd635273bd9fa8c57213
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269478"
---
# <a name="redact-faces-with-azure-media-analytics"></a>Censura de rostros con Azure Media Analytics

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Información general
**Redactor multimedia de Azure** es un procesador multimedia (MP) de [Análisis multimedia de Azure](media-services-analytics-overview.md) que ofrece censura de rostros escalable en la nube. La censura de rostros le permite modificar un vídeo con el fin de difuminar las caras de personas seleccionadas. Puede usar el servicio de censura de rostros en escenarios de seguridad pública y de noticias en los medios de comunicación. Unos minutos de material de archivo que contenga varias caras puede tardar horas en censurarse manualmente, pero con este servicio, el proceso de censura de caras requiere solamente unos pocos pasos sencillos. Para más información, consulte [este blog](https://azure.microsoft.com/blog/azure-media-redactor/).

En este artículo se proporcionan detalles sobre **Azure Media Redactor** y se muestra cómo se usa con el SDK de Media Services para .NET.

## <a name="face-redaction-modes"></a>Modos de censura de rostros
La censura facial funciona detectando caras en cada fotograma de vídeo y realizando un seguimiento del objeto de cara tanto hacia delante como hacia atrás en el tiempo, para que la imagen de la misma persona pueda difuminarse también desde otros ángulos. El proceso de censura automatizada es complejo y no siempre genera los resultados deseados al 100 %. Por este motivo, Azure Media Analytics le ofrece un par de métodos para modificar el resultado final.

Además de un modo totalmente automático, hay un flujo de trabajo de dos pasos que permite la selección y deselección de caras encontradas a través de una lista de identificadores. Igualmente, para realizar ajustes arbitrarios por fotograma, el procesador multimedia utiliza un archivo de metadatos en formato JSON. Este flujo de trabajo se divide en los modos **Analyze** (Analizar) y **Redact** (Censurar). Puede combinar los dos modos en un único paso que ejecuta las tareas de un trabajo; este modo se denomina **Combinado**.

### <a name="combined-mode"></a>Modo combinado
Esto genera un archivo .mp4 censurado automáticamente sin necesidad de intervención manual.

| Fase | Nombre de archivo | Notas |
| --- | --- | --- |
| Recurso de entrada |foo.bar |Vídeo en formato WMV, MOV o MP4 |
| Configuración de entrada |Configuración predeterminada de trabajo |{'version':'1.0', 'options': {'mode':'combined'}} |
| Recurso de salida |foo_redacted.mp4 |Vídeo con difuminado aplicado |

#### <a name="input-example"></a>Ejemplo de entrada:
[Vea este vídeo](https://ampdemo.azureedge.net/?url=https%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fed99001d-72ee-4f91-9fc0-cd530d0adbbc%2FDancing.mp4)

#### <a name="output-example"></a>Ejemplo de salida:
[Vea este vídeo](https://ampdemo.azureedge.net/?url=https%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc6608001-e5da-429b-9ec8-d69d8f3bfc79%2Fdance_redacted.mp4)

### <a name="analyze-mode"></a>Modo Analyze (Análisis)
El paso **analizar** del flujo de trabajo de dos pasos toma una entrada de vídeo y genera un archivo JSON de ubicaciones de rostros e imágenes jpg de cada rostro detectado.

| Fase | Nombre de archivo | Notas |
| --- | --- | --- |
| Recurso de entrada |foo.bar |Vídeo en formato WMV, MOV o MP4 |
| Configuración de entrada |Configuración predeterminada de trabajo |{'version':'1.0', 'options': {'mode':'analyze'}} |
| Recurso de salida |foo_annotations.JSON |Datos de anotación de ubicaciones de rostros en formato JSON. El usuario puede editarlo para modificar los rectángulos de selección para el difuminado. Vea el ejemplo a continuación. |
| Recurso de salida |foo_thumb%06d.jpg [foo_thumb000001.jpg, foo_thumb000002.jpg] |Un jpg recortado de cada rostro detectado, donde el número indica el identificador (labelId) de la cara |

#### <a name="output-example"></a>Ejemplo de salida:

```json
    {
      "version": 1,
      "timescale": 24000,
      "offset": 0,
      "framerate": 23.976,
      "width": 1280,
      "height": 720,
      "fragments": [
        {
          "start": 0,
          "duration": 48048,
          "interval": 1001,
          "events": [
            [],
            [],
            [],
            [],
            [],
            [],
            [],
            [],
            [],
            [],
            [],
            [],
            [],
            [
              {
                "index": 13,
                "id": 1138,
                "x": 0.29537,
                "y": -0.18987,
                "width": 0.36239,
                "height": 0.80335
              },
              {
                "index": 13,
                "id": 2028,
                "x": 0.60427,
                "y": 0.16098,
                "width": 0.26958,
                "height": 0.57943
              }
            ],

    … truncated
```

### <a name="redact-mode"></a>Modo Redact (Censurar)
El segundo paso del flujo de trabajo tiene un mayor número de entradas que tienen que combinarse en un solo recurso.

Esto incluye una lista de identificadores para difuminar, el vídeo original y las anotaciones JSON. Este modo usa las anotaciones para aplicar difuminado en el vídeo de entrada.

El resultado de la fase de análisis no incluye el vídeo original. El vídeo tiene que estar cargado en el recurso de entrada para la tarea de modo Redact (Censurar) y seleccionado como el archivo principal.

| Fase | Nombre de archivo | Notas |
| --- | --- | --- |
| Recurso de entrada |foo.bar |Vídeo en formato WMV, MOV o MP4. El mismo vídeo que en el paso 1. |
| Recurso de entrada |foo_annotations.JSON |archivo de metadatos de anotaciones de la fase uno, con modificaciones opcionales. |
| Recurso de entrada |foo_IDList.txt (opcional) |Nueva lista opcional separada por líneas de identificadores de rostro para censurar. Si se deja en blanco, se difuminarán todas las caras. |
| Configuración de entrada |Configuración predeterminada de trabajo |{'version':'1.0', 'options': {'mode':'analyze'}} |
| Recurso de salida |foo_redacted.mp4 |Vídeo con difuminado aplicado en base a las anotaciones |

#### <a name="example-output"></a>Salida de ejemplo
Este es el resultado de una lista de identificadores con un identificador seleccionado.

[Vea este vídeo](https://ampdemo.azureedge.net/?url=https%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fad6e24a2-4f9c-46ee-9fa7-bf05e20d19ac%2Fdance_redacted1.mp4)

foo_IDList.txt de ejemplo
 
```output
1
2
3
```

## <a name="blur-types"></a>Tipos de desenfoque

En el modo **Combined** (Combinado) o **Redact** (Censurar), hay cinco modos de desenfoque diferentes entre los que puede elegir en la configuración de la entrada JSON: **Bajo**, **Medio**, **Alto**, **Box** y **Negro**. Se usa **Medio** de forma predeterminada.

Puede encontrar ejemplos de los tipos de desenfoque a continuación.

### <a name="example-json"></a>Ejemplo de JSON:

```json
    {'version':'1.0', 'options': {'Mode': 'Combined', 'BlurType': 'High'}}
```

#### <a name="low"></a>Bajo

![Bajo](./media/media-services-face-redaction/blur1.png)
 
#### <a name="med"></a>Medio

![Medio](./media/media-services-face-redaction/blur2.png)

#### <a name="high"></a>Alto

![Alto](./media/media-services-face-redaction/blur3.png)

#### <a name="box"></a>Box

![Box](./media/media-services-face-redaction/blur4.png)

#### <a name="black"></a>Negro

![Negro](./media/media-services-face-redaction/blur5.png)

## <a name="elements-of-the-output-json-file"></a>Elementos del archivo JSON de salida

El procesador multimedia de censura proporciona detección de ubicación y seguimiento de rostros de alta precisión que puede detectar hasta 64 caras humanas en un fotograma de vídeo. Las caras de frente ofrecen los mejores resultados, mientras que las que se encuentran de lado y las caras pequeñas (inferiores o iguales a 24x24 píxeles) podrían no ser tan precisas.

[!INCLUDE [media-services-analytics-output-json](../../../includes/media-services-analytics-output-json.md)]

## <a name="net-sample-code"></a>Código de ejemplo de .NET

El programa siguiente muestra cómo:

1. Crear un recurso y cargar un archivo multimedia en él.
2. Crear un trabajo con una tarea de censura de rostros basándose en un archivo de configuración que contiene el siguiente valor predeterminado de JSON: 

    ```json
            {
                'version':'1.0',
                'options': {
                    'mode':'combined'
                }
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

namespace FaceRedaction
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

            // Run the FaceRedaction job.
            var asset = RunFaceRedactionJob(@"C:\supportFiles\FaceRedaction\SomeFootage.mp4",
                        @"C:\supportFiles\FaceRedaction\config.json");

            // Download the job output asset.
            DownloadAsset(asset, @"C:\supportFiles\FaceRedaction\Output");
        }

        static IAsset RunFaceRedactionJob(string inputMediaFilePath, string configurationFile)
        {
            // Create an asset and upload the input media file to storage.
            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
            "My Face Redaction Input Asset",
            AssetCreationOptions.None);

            // Declare a new job.
            IJob job = _context.Jobs.Create("My Face Redaction Job");

            // Get a reference to Azure Media Redactor.
            string MediaProcessorName = "Azure Media Redactor";

            var processor = GetLatestMediaProcessorByName(MediaProcessorName);

            // Read configuration from the specified file.
            string configuration = File.ReadAllText(configurationFile);

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("My Face Redaction Task",
            processor,
            configuration,
            TaskOptions.None);

            // Specify the input asset.
            task.InputAssets.Add(asset);

            // Add an output asset to contain the results of the job.
            task.OutputAssets.AddNew("My Face Redaction Output Asset", AssetCreationOptions.None);

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

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>Vínculos relacionados
[Información general de análisis de Azure Media Services](media-services-analytics-overview.md)

[Demostraciones de Azure Media Analytics](https://azuremedialabs.azurewebsites.net/demos/Analytics.html)

