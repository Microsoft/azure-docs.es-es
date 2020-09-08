---
title: Detección de movimientos con Análisis multimedia de Azure | Microsoft Docs
description: El procesador de multimedia (MP) Detector de movimiento multimedia de Azure permite identificar de manera eficaz las secciones de interés dentro de un vídeo que, de lo contrario, sería extenso y monótono.
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
ms.date: 03/19/2019
ms.author: juliako
ms.reviewer: milanga
ms.custom: devx-track-csharp
ms.openlocfilehash: 5e9a978cd96da47e91434c7146eb3e258d77d8ee
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89261349"
---
# <a name="detect-motions-with-azure-media-analytics"></a>Detección de movimientos con Análisis multimedia de Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> El procesador de multimedia **Azure Media Motion Detector** se va a retirar. Para ver la fecha de retirada, consulte el tema sobre los [componentes heredados](legacy-components.md).
 
## <a name="overview"></a>Información general

El procesador de multimedia (MP) **Detector de movimiento multimedia de Azure** permite identificar de manera eficaz las secciones de interés dentro de un vídeo que, de lo contrario, sería extenso y monótono. La detección de movimiento se puede usar en grabaciones con cámara estática para identificar las secciones del vídeo donde se produjo el movimiento. Genera un archivo JSON que contiene metadatos con marcas de tiempo y la región delimitadora donde se produjo el evento.

Orientada a las fuentes de vídeo de seguridad, esta tecnología puede clasificar el movimiento en eventos relevantes y falsos positivos, como sombras y cambios en la iluminación. Esto le permite generar alertas de seguridad a partir de las fuentes de las cámaras sin recibir correos no deseados con una cantidad interminable de eventos irrelevantes, a la vez que puede extraer momentos de interés a partir de vídeos de vigilancia considerablemente extensos.

El MP **Detector de movimiento multimedia de Azure** está actualmente en versión preliminar.

Este artículo proporciona detalles sobre el **Azure Media Motion Detector** y muestra cómo se usa con el SDK de Media Services para .NET

## <a name="motion-detector-input-files"></a>Archivos de entrada del Detector de movimiento
Archivos de vídeo. Actualmente, se admiten los siguientes formatos: MP4, MOV y WMV.

## <a name="task-configuration-preset"></a>Configuración de tareas (valor predeterminado)
Cuando cree una tarea con **Azure Media Motion Detector**, tiene que especificar una configuración preestablecida. 

### <a name="parameters"></a>Parámetros
Puede usar los siguientes parámetros:

| Nombre | Opciones | Descripción | Valor predeterminado |
| --- | --- | --- | --- |
| sensitivityLevel |String: 'bajo', 'medio', 'alto' |Establece el nivel de sensibilidad que se usa para notificar los movimientos. Es necesario ajustarlo bien para controlar el número de falsos positivos. |'medio' |
| frameSamplingValue |Un número entero positivo |Establece la frecuencia con la que se ejecuta el algoritmo. 1 es en cada fotograma, 2 significa en uno de cada dos fotogramas y así sucesivamente. |1 |
| detectLightChange |Boolean: 'true', 'false' |Establece si se notifican los cambios de luz en los resultados |'False' |
| mergeTimeThreshold |Xs-time: hh:mm:ss<br/>Ejemplo: 00:00:03 |Especifica el período de tiempo entre eventos de movimiento donde 2 eventos se combinan y se notifican como 1. |00:00:00 |
| detectionZones |Una matriz de zonas de detección:<br/>-La zona de detección es un array de 3 o más puntos<br/>-Punto es una coordenada x e y de 0 a 1. |Describe la lista de zonas de detección poligonal que se usan.<br/>Los resultados aparecerán con las zonas como un identificador, siendo el primero 'id': 0 |Zona única, que abarca todo el marco. |

### <a name="json-example"></a>Ejemplo JSON

```json
    {
      "version": "1.0",
      "options": {
        "sensitivityLevel": "medium",
        "frameSamplingValue": 1,
        "detectLightChange": "False",
        "mergeTimeThreshold":
        "00:00:02",
        "detectionZones": [
          [
            {"x": 0, "y": 0},
            {"x": 0.5, "y": 0},
            {"x": 0, "y": 1}
           ],
          [
            {"x": 0.3, "y": 0.3},
            {"x": 0.55, "y": 0.3},
            {"x": 0.8, "y": 0.3},
            {"x": 0.8, "y": 0.55},
            {"x": 0.8, "y": 0.8},
            {"x": 0.55, "y": 0.8},
            {"x": 0.3, "y": 0.8},
            {"x": 0.3, "y": 0.55}
          ]
        ]
      }
    }
```

## <a name="motion-detector-output-files"></a>Archivos de salida del Detector de movimiento
Un trabajo de detección de movimiento devuelve un archivo JSON en el recurso de salida, que describe las alertas de movimiento y sus categorías dentro del vídeo. El archivo contiene información sobre el tiempo y la duración del movimiento detectado en el vídeo.

La API del Detector de movimiento brinda indicadores una vez que hay objetos en movimiento en un vídeo de fondo fijo (por ejemplo, un vídeo de vigilancia). El Detector de movimiento está entrenado para disminuir las alarmas falsas, como cambios de iluminación y sombras. Las limitaciones actuales de los algoritmos incluyen vídeos con visión nocturna, objetos semitransparentes y objetos pequeños.

### <a name="elements-of-the-output-json-file"></a><a id="output_elements"></a>Elementos del archivo JSON de salida
> [!NOTE]
> En la versión más reciente, el formato JSON de salida ha cambiado y puede representar un cambio importante para algunos clientes.
> 
> 

En la tabla siguiente, se describen elementos del archivo JSON de salida.

| Elemento | Descripción |
| --- | --- |
| version |Esto se refiere a la versión de la API de vídeo. La versión actual es 2. |
| timescale |"Tics" por segundo del vídeo. |
| offset |La diferencia de tiempo para las marcas de tiempo en "tics". En la versión 1.0 de las API de vídeo, será siempre 0. En los escenarios futuros que se admitan, este valor puede cambiar. |
| framerate |Fotogramas por segundo del vídeo. |
| width, height |Se refiere al ancho y alto del vídeo en píxeles. |
| start |La marca de tiempo de inicio en "tics". |
| duration |La longitud del evento, en "tics". |
| interval |El intervalo de cada entrada del evento, en "tics". |
| events |Cada fragmento de evento contiene el movimiento detectado dentro de esa duración. |
| type |En la versión actual, este valor siempre es "2" para el movimiento genérico. Esta etiqueta brinda a las API de vídeo la flexibilidad para clasificar el movimiento en las versiones futuras. |
| regionId |Tal como se explicó anteriormente, este valor siempre será 0 en esta versión. Esta etiqueta brinda a la API de vídeo la flexibilidad para encontrar el movimiento en diversas regiones en las versiones futuras. |
| regions |Se refiere al área del vídeo donde le interesa el movimiento. <br/><br/>-"id" representa el área de la región: en esta versión es solo una, Id. 0. <br/>-"tipo" representa la forma de la región que le interesa para un movimiento. Actualmente, se admiten los valores "rectángulo" y "polígono".<br/> Si especifica "rectángulo", las dimensiones de la región son X, Y, ancho y alto. Las coordenadas X e Y representan las coordenadas XY del lado superior izquierdo de la región en una escala normalizada de 0,0 a 1,0. El ancho y el alto representan el tamaño de la región en una escala normalizada de 0,0 a 1,0. En la versión actual, X, Y, ancho y alto son valores fijos siempre en 0, 0 y 1, 1. <br/>Si especifica "polígono", la región tiene dimensiones en puntos. <br/> |
| fragments |Los metadatos se separan en diferentes segmentos denominados fragmentos. Cada fragmento contiene un inicio, una duración, un número de intervalo y eventos. Un fragmento sin eventos significa que no se detectó movimiento durante esa hora de inicio y la duración. |
| corchetes [] |Cada corchete representa un intervalo del evento. Si ese intervalo contiene corchetes vacíos, significa que no se detectó movimiento. |
| locations |Esta nueva entrada de eventos muestra la ubicación donde se produjo el movimiento. Se trata de un valor más específico que las zonas de detección. |

En el siguiente ejemplo de JSON se muestra la salida:

```json
    {
      "version": 2,
      "timescale": 23976,
      "offset": 0,
      "framerate": 24,
      "width": 1280,
      "height": 720,
      "regions": [
        {
          "id": 0,
          "type": "polygon",
          "points": [{'x': 0, 'y': 0},
            {'x': 0.5, 'y': 0},
            {'x': 0, 'y': 1}]
        }
      ],
      "fragments": [
        {
          "start": 0,
          "duration": 226765
        },
        {
          "start": 226765,
          "duration": 47952,
          "interval": 999,
          "events": [
            [
              {
                "type": 2,
                "typeName": "motion",
                "locations": [
                  {
                    "x": 0.004184,
                    "y": 0.007463,
                    "width": 0.991667,
                    "height": 0.985185
                  }
                ],
                "regionId": 0
              }
            ],
```

## <a name="limitations"></a>Limitaciones
* Los formatos de vídeo de entrada admitidos incluyen MP4, MOV y WMV.
* La detección de movimiento está optimizada para los vídeos de fondo fijos. El algoritmo se centra en disminuir las alarmas falsas, como cambios en la iluminación y sombras.
* Es posible que no se detecten ciertos movimientos debido a desafíos técnicos; por ejemplo, vídeos con visión nocturna, objetos semitransparentes y objetos pequeños.

## <a name="net-sample-code"></a>Código de ejemplo de .NET

El programa siguiente muestra cómo:

1. Crear un recurso y cargar un archivo multimedia en él.
2. Crear un trabajo con una tarea de detección de movimiento de vídeo basada en un archivo de configuración que contiene el siguiente valor predeterminado JSON: 
   
    ```json
            {
            "Version": "1.0",
            "Options": {
                "SensitivityLevel": "medium",
                "FrameSamplingValue": 1,
                "DetectLightChange": "False",
                "MergeTimeThreshold":
                "00:00:02",
                "DetectionZones": [
                [
                    {"x": 0, "y": 0},
                    {"x": 0.5, "y": 0},
                    {"x": 0, "y": 1}
                ],
                [
                    {"x": 0.3, "y": 0.3},
                    {"x": 0.55, "y": 0.3},
                    {"x": 0.8, "y": 0.3},
                    {"x": 0.8, "y": 0.55},
                    {"x": 0.8, "y": 0.8},
                    {"x": 0.55, "y": 0.8},
                    {"x": 0.3, "y": 0.8},
                    {"x": 0.3, "y": 0.55}
                ]
                ]
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

namespace VideoMotionDetection
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

            // Run the VideoMotionDetection job.
            var asset = RunVideoMotionDetectionJob(@"C:\supportFiles\VideoMotionDetection\BigBuckBunny.mp4",
                                        @"C:\supportFiles\VideoMotionDetection\config.json");

            // Download the job output asset.
            DownloadAsset(asset, @"C:\supportFiles\VideoMotionDetection\Output");
        }

        static IAsset RunVideoMotionDetectionJob(string inputMediaFilePath, string configurationFile)
        {
            // Create an asset and upload the input media file to storage.
            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
                "My Video Motion Detection Input Asset",
                AssetCreationOptions.None);

            // Declare a new job.
            IJob job = _context.Jobs.Create("My Video Motion Detection Job");

            // Get a reference to Azure Media Motion Detector.
            string MediaProcessorName = "Azure Media Motion Detector";

            var processor = GetLatestMediaProcessorByName(MediaProcessorName);

            // Read configuration from the specified file.
            string configuration = File.ReadAllText(configurationFile);

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("My Video Motion Detection Task",
                processor,
                configuration,
                TaskOptions.None);

            // Specify the input asset.
            task.InputAssets.Add(asset);

            // Add an output asset to contain the results of the job.
            task.OutputAssets.AddNew("My Video Motion Detection Output Asset", AssetCreationOptions.None);

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
[Blog de Azure Media Services Motion Detector](https://azure.microsoft.com/blog/motion-detector-update/)

[Información general de análisis de Azure Media Services](media-services-analytics-overview.md)

[Demostraciones de Azure Media Analytics](https://azuremedialabs.azurewebsites.net/demos/Analytics.html)

