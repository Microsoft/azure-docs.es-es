---
title: Codificación avanzada con el flujo de trabajo de Media Encoder premium | Microsoft Docs
description: Aprenda a codificar con el flujo de trabajo del Codificador multimedia Premium. Los ejemplos de código están escritos en C# y utilizan el SDK de Media Services para .NET.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 29a889fc15b1889711d08d3386920652a8bc8f57
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269053"
---
# <a name="advanced-encoding-with-media-encoder-premium-workflow"></a>Codificación avanzada con el flujo de trabajo del Codificador multimedia Premium

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> El procesador de multimedia de Media Encoder Premium Workflow del que se habla en este tema no está disponible en China.
>
>

## <a name="overview"></a>Información general
Microsoft Azure Media Services presenta el procesador multimedia de **Flujo de trabajo de Media Encoder Premium** . Este procesador ofrece funciones de codificación avanzadas para sus flujos de trabajo de Premium a petición.

Los siguientes temas describen los detalles relacionados con el **flujo de trabajo premium de Media Encoder**:

* [Formatos que admite el flujo de trabajo del Codificador multimedia Premium](media-services-premium-workflow-encoder-formats.md) : trata los formatos de archivo y los códecs que admite el **flujo de trabajo del Codificador multimedia Premium**.
* En [Información general y comparación de codificadores multimedia a petición de Azure](media-services-encode-asset.md) se comparan las funcionalidades de codificación de **Media Encoder Premium Workflow** y **Media Encoder Standard**.

En este artículo se muestra cómo codificar con **Media Encoder Premium Workflow** con .NET.

Las tareas de codificación del **flujo de trabajo de Media Encoder Premium** requieren un archivo de configuración independiente, denominado archivo de flujo de trabajo. Estos archivos tienen una extensión .workflow y se crean con la herramienta [Diseñador de flujo de trabajo](media-services-workflow-designer.md) .

Los archivos de flujo de trabajo predeterminados también se pueden obtener [aquí](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows). La carpeta también contiene la descripción de estos archivos.

Los archivos de flujo de trabajo deben cargarse en su cuenta de Media Services como un recurso, y el recurso se debe transferir a la tarea de codificación.

## <a name="create-and-configure-a-visual-studio-project"></a>Creación y configuración de un proyecto de Visual Studio

Configure el entorno de desarrollo y rellene el archivo app.config con la información de la conexión, como se describe en [Desarrollo de Media Services con .NET](media-services-dotnet-how-to-use.md). 

## <a name="encoding-example"></a>Ejemplo de codificación

En el ejemplo siguiente se muestra cómo codificar con el **flujo de trabajo del Codificador multimedia Premium**.

Hay que seguir estos pasos:

1. Crear un recurso y cargar un archivo de flujo de trabajo.
2. Crear un recurso y cargar un archivo multimedia de origen.
3. Obtenga el procesador multimedia "Media Encoder Premium Workflow".
4. Crear un trabajo y una tarea.

    En la mayoría de los casos, la cadena de configuración de la tarea está vacía (como en el ejemplo siguiente). Hay algunos escenarios avanzados (que requieren que establezca propiedades de tiempo de ejecución dinámicamente) en los que debería proporcionar una cadena XML para la tarea de codificación. Ejemplos de estos escenarios son la creación de una superposición, la unión paralela o secuencial multimedia y el subtitulado.
5. Agregar dos recursos de entrada a la tarea.

   1. 1\.º: el recurso de flujo de trabajo.
   2. 2\.º: el recurso de vídeo.

      >[!NOTE]
      >El recurso de flujo de trabajo debe agregarse a la tarea antes que el recurso multimedia.
      La cadena de configuración para esta tarea debe estar vacía.
   
6. Envíe el trabajo de codificación.

```csharp
using System;
using System.Linq;
using System.Configuration;
using System.IO;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.MediaServices.Client;

namespace MediaEncoderPremiumWorkflowSample
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

        private static readonly string _supportFiles =
            Path.GetFullPath(@"../..\Media");

        private static readonly string _workflowFilePath =
            Path.GetFullPath(_supportFiles + @"\H264 Progressive Download MP4.workflow");

        private static readonly string _singleMP4InputFilePath =
            Path.GetFullPath(_supportFiles + @"\BigBuckBunny.mp4");

        static void Main(string[] args)
        {

            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            var workflowAsset = CreateAssetAndUploadSingleFile(_workflowFilePath);
            var videoAsset = CreateAssetAndUploadSingleFile(_singleMP4InputFilePath);
            IAsset outputAsset = CreateEncodingJob(workflowAsset, videoAsset);

        }

        static public IAsset CreateAssetAndUploadSingleFile(string singleFilePath)
        {
            var assetName = "UploadSingleFile_" + DateTime.UtcNow.ToString();
            var asset = _context.Assets.Create(assetName, AssetCreationOptions.None);

            var fileName = Path.GetFileName(singleFilePath);

            var assetFile = asset.AssetFiles.Create(fileName);

            Console.WriteLine("Created assetFile {0}", assetFile.Name);

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            return asset;
        }

        static public IAsset CreateEncodingJob(IAsset workflow, IAsset video)
        {
            // Declare a new job.
            IJob job = _context.Jobs.Create("Premium Workflow encoding job");
            // Get a media processor reference, and pass to it the name of the
            // processor to use for the specific task.
            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Premium Workflow");

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("Premium Workflow encoding task",
                processor,
                "",
                TaskOptions.None);

            // Specify the input asset to be encoded.
            task.InputAssets.Add(workflow);
            task.InputAssets.Add(video); // we add one asset
                                         // Add an output asset to contain the results of the job.
                                         // This output is specified as AssetCreationOptions.None, which
                                         // means the output asset is not encrypted.
            task.OutputAssets.AddNew("Output asset",
                AssetCreationOptions.None);

            // Use the following event handler to check job progress.  
            job.StateChanged += new
                    EventHandler<JobStateChangedEventArgs>(StateChanged);

            // Launch the job.
            job.Submit();

            // Check job execution and wait for job to finish.
            Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
            progressJobTask.Wait();

            // If job state is Error the event handling
            // method for job progress should log errors.  Here we check
            // for error state and exit if needed.
            if (job.State == JobState.Error)
            {
                throw new Exception("\nExiting method due to job error.");
            }

            return job.OutputMediaAssets[0];
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
        private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
        {
            var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
            ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

            if (processor == null)
                throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

            return processor;
        }
    }
}
```

## <a name="need-help"></a>¿Necesita ayuda?

Puede abrir una incidencia de soporte técnico si se desplaza a la [nueva solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
