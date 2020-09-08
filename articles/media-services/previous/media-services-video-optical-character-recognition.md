---
title: Digitalización de texto con OCR de Análisis multimedia de Azure | Microsoft Docs
description: Gracias al OCR (reconocimiento óptico de caracteres) de Análisis multimedia de Azure, podrá convertir el contenido de texto de archivos de vídeo en texto digital modificable y utilizable en búsquedas.  De este modo, se puede automatizar la extracción de metadatos significativos de la señal de vídeo de los elementos multimedia.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 307c196e-3a50-4f4b-b982-51585448ffc6
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 4f89215525232f9722cd95cc4dd85a4bd19118de
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269002"
---
# <a name="use-azure-media-analytics-to-convert-text-content-in-video-files-into-digital-text"></a>Uso de Análisis multimedia de Azure para convertir el contenido de texto de archivos de vídeo en texto digital 

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

> [!NOTE]
> El procesador de multimedia **Azure Media OCR** se va a retirar. Para ver la fecha de retirada, consulte el tema sobre los [componentes heredados](legacy-components.md).

## <a name="overview"></a>Información general
Si necesita extraer el contenido de texto de los archivos de vídeo y generar un texto digital que se pueda editar y en el que se pueda buscar, utilice OCR (reconocimiento óptico de caracteres) de Análisis multimedia de Azure. Este procesador multimedia de Azure detecta contenido de texto en los archivos de vídeo y genera archivos de texto para su uso. El OCR le permite automatizar la extracción de metadatos significativos a partir de la señal de vídeo de los elementos multimedia.

Cuando se utiliza junto con un motor de búsqueda, puede indexar fácilmente el contenido multimedia por texto y mejorar la detectabilidad de su contenido. Esto es muy útil en vídeos con gran cantidad de texto, como una grabación de vídeo o una captura de pantalla de una presentación de diapositivas. El procesador multimedia de OCR de Azure está optimizado para texto digital.

El procesador multimedia **Azure Media OCR** está actualmente en versión preliminar.

En este artículo se proporcionan detalles sobre **Azure Media OCR** y se muestra cómo se usa con el SDK de Media Services para .NET. Para obtener más información y ejemplos, consulte [este blog](https://azure.microsoft.com/blog/announcing-video-ocr-public-preview-new-config/).

## <a name="ocr-input-files"></a>Archivos de entrada para OCR
Archivos de vídeo. Actualmente, se admiten los siguientes formatos: MP4, MOV y WMV.

## <a name="task-configuration"></a>Configuración de tareas
Configuración de tareas (valor predeterminado) Cuando se crea una tarea con **Azure Media OCR**, debe especificar una configuración preestablecida mediante JSON o XML. 

>[!NOTE]
>El motor de OCR solo toma una área de imagen con un mínimo de 40 píxeles hasta un máximo de 32 000 píxeles como entrada válida en los valores alto y ancho.
>

### <a name="attribute-descriptions"></a>Descripciones de atributos
| Nombre del atributo | Descripción |
| --- | --- |
|AdvancedOutput| Si establece AdvancedOutput en true, la salida JSON contendrá datos posicionales para cada palabra única (además de frases y regiones). Si no desea ver estos detalles, establezca la marca en false. El valor predeterminado es false. Para más información, vea [este blog](https://azure.microsoft.com/blog/azure-media-ocr-simplified-output/).|
| Idioma |(Opcional) Describe el idioma del texto que desea buscar. Uno de los siguientes: Detección automática (predeterminado), alemán, árabe, chino (simplificado y tradicional), checo, coreano, danés, español, finés, francés, griego, húngaro, inglés, italiano, japonés, neerlandés, noruego, polaco, portugués, rumano, ruso, serbio cirílico, serbio latino, eslovaco, sueco, turco. |
| TextOrientation |(Opcional) Describe la orientación del texto que desea buscar.  "Izquierda" significa que la parte superior de todas las letras apunta a la izquierda.  El texto predeterminado (similar al que se encuentra en un libro) se puede denominar orientado "hacia arriba".  Uno de los siguientes: detección automática (valor predeterminado), arriba, derecha, abajo, izquierda. |
| TimeInterval |(Opcional) Describe la frecuencia de muestreo.  El valor predeterminado es cada 1/2 segundo.<br/>Formato JSON: HH:mm:ss.SSS (predeterminado 00:00:00.500)<br/>Formato XML – primitiva de duración W3C XSD (valor predeterminado PT0.5) |
| DetectRegions |(Opcional) Una matriz de objetos DetectRegion que especifica regiones dentro del marco de vídeo en el que se va a detectar el texto.<br/>Un objeto DetectRegion consta de los siguientes cuatro valores enteros:<br/>Izquierda: píxeles desde el margen izquierdo<br/>Parte superior: píxeles desde el margen superior<br/>Ancho: ancho de la región en píxeles<br/>Alto: el alto de la región en píxeles |

#### <a name="json-preset-example"></a>Ejemplo de JSON preestablecido

```json
    {
        "Version":1.0, 
        "Options": 
        {
            "AdvancedOutput":"true",
            "Language":"English", 
            "TimeInterval":"00:00:01.5",
            "TextOrientation":"Up",
            "DetectRegions": [
                    {
                       "Left": 10,
                       "Top": 10,
                       "Width": 100,
                       "Height": 50
                    }
             ]
        }
    }
```

#### <a name="xml-preset-example"></a>Ejemplo de XML preestablecido

```xml
    <?xml version=""1.0"" encoding=""utf-16""?>
    <VideoOcrPreset xmlns:xsi=""https://www.w3.org/2001/XMLSchema-instance"" xmlns:xsd=""https://www.w3.org/2001/XMLSchema"" Version=""1.0"" xmlns=""https://www.windowsazure.com/media/encoding/Preset/2014/03"">
      <Options>
         <AdvancedOutput>true</AdvancedOutput>
         <Language>English</Language>
         <TimeInterval>PT1.5S</TimeInterval>
         <DetectRegions>
             <DetectRegion>
                   <Left>10</Left>
                   <Top>10</Top>
                   <Width>100</Width>
                   <Height>50</Height>
            </DetectRegion>
       </DetectRegions>
       <TextOrientation>Up</TextOrientation>
      </Options>
    </VideoOcrPreset>
```

## <a name="ocr-output-files"></a>Archivos de salida OCR
La salida del procesador multimedia OCR es un archivo JSON.

### <a name="elements-of-the-output-json-file"></a>Elementos del archivo JSON de salida
La salida de vídeo OCR proporciona datos segmentados en tiempo en los caracteres que se encuentran en el vídeo.  Puede utilizar atributos tales como el idioma o la orientación para afinar exactamente las palabras en las que está interesado para analizar. 

La salida contiene los siguientes atributos:

| Elemento | Descripción |
| --- | --- |
| Timescale |"Tics" por segundo del vídeo |
| Offset |Diferencia de tiempo para las marcas de tiempo En la versión 1.0 de las API de vídeo, será siempre 0. |
| Framerate |Fotogramas por segundo del vídeo |
| width |Ancho del vídeo en píxeles |
| height |Alto del vídeo en píxeles |
| Fragments |Matriz de fragmentos de vídeo basados en tiempo en los que están fragmentados los metadatos |
| start |Hora de inicio de un fragmento en "tics" |
| duration |Duración de un fragmento en "tics" |
| interval |Intervalo de cada evento en el fragmento determinado |
| events |La matriz que contiene las regiones |
| region |Objeto que representa las palabras o frases detectadas |
| language |Idioma del texto detectado en una región |
| orientation |Orientación del texto detectado en una región |
| lines |Matriz de líneas de texto detectadas en una región |
| text |El texto real |

### <a name="json-output-example"></a>Ejemplo de salida JSON
En el siguiente ejemplo de salida contiene la información general de vídeo y varios fragmentos de vídeo. Cada fragmento de vídeo contiene todas las regiones, que las detecta el módulo de administración del reconocimiento óptico de caracteres, con el idioma y la orientación del texto. La región también contiene las líneas de palabras en esta región con el texto de la línea, la posición de la línea y cualquier otra información de las palabras (contenido de palabras, posición y confianza) de esta línea. El siguiente es un ejemplo y he agregado algunos comentarios entre líneas.

```json
    {
        "version": 1, 
        "timescale": 90000, 
        "offset": 0, 
        "framerate": 30, 
        "width": 640, 
        "height": 480,  // general video information
        "fragments": [
            {
                "start": 0, 
                "duration": 180000, 
                "interval": 90000,  // the time information about this fragment
                "events": [
                    [
                       { 
                            "region": { // the detected region array in this fragment 
                                "language": "English",  // region language
                                "orientation": "Up",  // text orientation
                                "lines": [  // line information array in this region, including the text and the position
                                    {
                                        "text": "One Two", 
                                        "left": 10, 
                                        "top": 10, 
                                        "right": 210, 
                                        "bottom": 110, 
                                        "word": [  // word information array in this line
                                            {
                                                "text": "One", 
                                                "left": 10, 
                                                "top": 10, 
                                                "right": 110, 
                                                "bottom": 110, 
                                                "confidence": 900
                                            }, 
                                            {
                                                "text": "Two", 
                                                "left": 110, 
                                                "top": 10, 
                                                "right": 210, 
                                                "bottom": 110, 
                                                "confidence": 910
                                            }
                                        ]
                                    }
                                ]
                            }
                        }
                    ]
                ]
            }
        ]
    }
```

## <a name="net-sample-code"></a>Código de ejemplo de .NET

El programa siguiente muestra cómo:

1. Crear un recurso y cargar un archivo multimedia en él.
2. Cree un trabajo con un archivo de configuración o valores preestablecidos de OCR.
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

namespace OCR
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

            // Run the OCR job.
            var asset = RunOCRJob(@"C:\supportFiles\OCR\presentation.mp4",
                                        @"C:\supportFiles\OCR\config.json");

            // Download the job output asset.
            DownloadAsset(asset, @"C:\supportFiles\OCR\Output");
        }

        static IAsset RunOCRJob(string inputMediaFilePath, string configurationFile)
        {
            // Create an asset and upload the input media file to storage.
            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
                "My OCR Input Asset",
                AssetCreationOptions.None);

            // Declare a new job.
            IJob job = _context.Jobs.Create("My OCR Job");

            // Get a reference to Azure Media OCR.
            string MediaProcessorName = "Azure Media OCR";

            var processor = GetLatestMediaProcessorByName(MediaProcessorName);

            // Read configuration from the specified file.
            string configuration = File.ReadAllText(configurationFile);

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("My OCR Task",
                processor,
                configuration,
                TaskOptions.None);

            // Specify the input asset.
            task.InputAssets.Add(asset);

            // Add an output asset to contain the results of the job.
            task.OutputAssets.AddNew("My OCR Output Asset", AssetCreationOptions.None);

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

