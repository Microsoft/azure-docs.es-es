---
title: Codificación de un recurso con Media Encoder Standard mediante .NET | Microsoft Docs
description: En este artículo se muestra cómo usar .NET para codificar un recurso mediante Media Encoder Standard.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 03431b64-5518-478a-a1c2-1de345999274
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.reviewer: anilmur
ms.custom: devx-track-csharp
ms.openlocfilehash: 90b4680c725f3b5b7fcb6b988e52410c572b2cfa
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89262416"
---
# <a name="encode-an-asset-with-media-encoder-standard-using-net"></a>Codificación de un recurso mediante Estándar de codificador multimedia

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

Los trabajos de Encoding son una de las operaciones de procesamiento más habituales en Media Services. Los trabajos de codificación se crean para convertir archivos multimedia de una codificación a otra. Al codificar, puede usar Media Encoder, el codificador integrado de Media Services. También puede usar un codificador proporcionado por un asociado de Media Services; hay codificadores de terceros disponibles en Azure Marketplace. 

En este tema se muestra cómo usar .NET para codificar sus recursos con Media Encoder Standard (MES). Media Encoder Standard se configura mediante uno de los valores preestablecidos de codificador descritos [aquí](https://go.microsoft.com/fwlink/?linkid=618336&clcid=0x409).

Se recomienda codificar siempre los archivos de origen en un conjunto de archivos MP4 de velocidad de bits adaptable y, a continuación, convertirlo al formato deseado con el [empaquetado dinámico](media-services-dynamic-packaging-overview.md). 

Si el recurso de salida tiene el almacenamiento cifrado, asegúrese de configurar la directiva de entrega de recursos. Para obtener más información, consulte [Configuración de directivas de entrega de recursos](media-services-dotnet-configure-asset-delivery-policy.md).

> [!NOTE]
> MES genera un archivo de salida con un nombre que contiene los 32 primeros caracteres del nombre del archivo de entrada. El nombre se basa en lo que se especifica en el archivo de valores preestablecidos. Por ejemplo, "FileName": "{nombreBase}_{índice}{extensión}". {Basename} se reemplaza por los 32 primeros caracteres del nombre del archivo de entrada.
> 
> 

### <a name="mes-formats"></a>Formatos de MES
[Códecs y formatos](media-services-media-encoder-standard-formats.md)

### <a name="mes-presets"></a>Valores preestablecidos de MES
Media Encoder Standard se configura mediante uno de los valores preestablecidos de codificador descritos [aquí](https://go.microsoft.com/fwlink/?linkid=618336&clcid=0x409).

### <a name="input-and-output-metadata"></a>Metadatos de entrada y salida
Al codificar un recurso de entrada (o recursos) con MES, obtendrá un recurso de salida en la correcta realización de dicha tarea de codificación. El recurso de salida contiene vídeo, audio, miniaturas, manifiestos, etc., basados en el valor predeterminado de codificación que utilice.

El recurso de salida también contiene un archivo con metadatos sobre el recurso de entrada. El nombre del archivo XML de metadatos tiene el formato siguiente: <asset_id>_metadata.xml (por ejemplo, 41114ad3-eb5e-4c57-8d92-5354e2b7d4a4_metadata.xml), donde <asset_id> es el valor AssetId del recurso de entrada. El esquema de este archivo XML de metadatos de entrada se describe [aquí](media-services-input-metadata-schema.md).

El recurso de salida también contiene un archivo con metadatos sobre el recurso de salida. El nombre del archivo XML de metadatos tiene el formato siguiente: <nombre_de_archivo_de_origen>_manifest.xml (por ejemplo, BigBuckBunny_manifest.xml). El esquema de este archivo XML de metadatos de salida se describe [aquí](media-services-output-metadata-schema.md).

Si desea examinar cualquiera de los dos archivos de metadatos, puede crear un localizador de SAS y descargar el archivo en el equipo local. Puede encontrar un ejemplo sobre cómo crear un localizador SAS y descargar un archivo con extensiones del SDK de .NET de Media Services.

## <a name="download-sample"></a>Descarga de un ejemplo
Puede obtener y ejecutar un ejemplo que muestra cómo codificar con MES [aquí](https://azure.microsoft.com/documentation/samples/media-services-dotnet-on-demand-encoding-with-media-encoder-standard/).

## <a name="net-sample-code"></a>Código de ejemplo de .NET

En el ejemplo de código siguiente se usa el último SDK para .NET de Media Services para realizar las siguientes tareas:

* Crear un trabajo de codificación.
* Obtener una referencia al codificador Codificador multimedia estándar.
* Especificar que se utilice el valor predeterminado [Streaming adaptable](media-services-autogen-bitrate-ladder-with-mes.md). 
* Agregar una única tarea de codificación al trabajo. 
* Especificar el recurso de entrada que se va a codificar.
* Crear un recurso de salida que contendrá el recurso codificado.
* Agregar un controlador de eventos para comprobar el progreso del trabajo.
* Envíe el trabajo.

#### <a name="create-and-configure-a-visual-studio-project"></a>Creación y configuración de un proyecto de Visual Studio

Configure el entorno de desarrollo y rellene el archivo app.config con la información de la conexión, como se describe en [Desarrollo de Media Services con .NET](media-services-dotnet-how-to-use.md). 

#### <a name="example"></a>Ejemplo 

```csharp
using System;
using System.Linq;
using System.Configuration;
using System.IO;
using System.Threading;
using Microsoft.WindowsAzure.MediaServices.Client;

namespace MediaEncoderStandardSample
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

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);


            // Get an uploaded asset.
            var asset = _context.Assets.FirstOrDefault();

            // Encode and generate the output using the "Adaptive Streaming" preset.
            EncodeToAdaptiveBitrateMP4Set(asset);

            Console.ReadLine();
        }

        static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset asset)
        {
            // Declare a new job.
            IJob job = _context.Jobs.Create("Media Encoder Standard Job");
            // Get a media processor reference, and pass to it the name of the 
            // processor to use for the specific task.
            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

            // Create a task with the encoding details, using a string preset.
            // In this case "Adaptive Streaming" preset is used.
            ITask task = job.Tasks.AddNew("My encoding task",
                processor,
                "Adaptive Streaming",
                TaskOptions.None);

            // Specify the input asset to be encoded.
            task.InputAssets.Add(asset);
            // Add an output asset to contain the results of the job. 
            // This output is specified as AssetCreationOptions.None, which 
            // means the output asset is not encrypted. 
            task.OutputAssets.AddNew("Output asset",
                AssetCreationOptions.None);

            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
            job.Submit();
            job.GetExecutionProgressTask(CancellationToken.None).Wait();

            return job.OutputMediaAssets[0];
        }

        private static void JobStateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine("Job state changed event:");
            Console.WriteLine("  Previous state: " + e.PreviousState);
            Console.WriteLine("  Current state: " + e.CurrentState);
            switch (e.CurrentState)
            {
                case JobState.Finished:
                    Console.WriteLine();
                    Console.WriteLine("Job is finished. Please wait while local tasks or downloads complete...");
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

## <a name="advanced-encoding-features-to-explore"></a>Características avanzadas de codificación para explorar
* [Cómo generar vistas en miniatura](media-services-dotnet-generate-thumbnail-with-mes.md)
* [Generación de miniaturas durante la codificación](media-services-dotnet-generate-thumbnail-with-mes.md#example-of-generating-a-thumbnail-while-encoding)
* [Recorte de vídeos durante la codificación](media-services-crop-video.md)
* [Personalización de valores preestablecidos de codificación](media-services-custom-mes-presets-with-dotnet.md)
* [Superposición o marca de agua de un vídeo con una imagen](media-services-advanced-encoding-with-mes.md#overlay)

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Pasos siguientes
[Cómo generar vistas en miniatura mediante Media Encoder Standard con .NET](media-services-dotnet-generate-thumbnail-with-mes.md)
[Introducción a Media Services Encoding](media-services-encode-asset.md)

