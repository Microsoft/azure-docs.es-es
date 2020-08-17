---
title: Indización de archivos multimedia con el Indizador multimedia de Azure
description: El Indizador multimedia de Azure permite que el contenido de los archivos multimedia se puedan buscar y genera una transcripción de texto completo para las palabras clave y subtítulos. En este tema se muestra cómo usar el Indizador multimedia.
services: media-services
documentationcenter: ''
author: Asolanki
manager: femila
editor: ''
ms.assetid: 827a56b2-58a5-4044-8d5c-3e5356488271
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 09/22/2019
ms.author: juliako
ms.reviewer: johndeu
ms.openlocfilehash: 8b9f0e824352126204c7a6f1cfe08d3e7c6c29a9
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87826065"
---
# <a name="indexing-media-files-with-azure-media-indexer"></a>Indización de archivos multimedia con el Indizador multimedia de Azure

> [!NOTE]
> El procesador de multimedia **Azure Media Indexer** se va a retirar. Para ver las fechas de retirada, consulte el tema [componentes heredados](legacy-components.md). [Video Indexer de Azure Media Services ](../video-indexer/index.yml) reemplaza a su procesador multimedia heredado. Para más información, consulte [Migración de Azure Media Indexer y Azure Media Indexer 2 a Video Indexer de Azure Media Services](migrate-indexer-v1-v2.md).

El Indizador multimedia de Azure permite que el contenido de los archivos multimedia se puedan buscar y genera una transcripción de texto completo para las palabras clave y subtítulos. Puede procesar uno o varios archivos multimedia en un lote.  

Al indexar contenido, asegúrese de usar archivos multimedia que tengan una voz muy clara (sin música de fondo, ruido, efectos ni silbido de micrófono). Algunos ejemplos de contenido adecuado son: reuniones, conferencias o presentaciones grabadas. Es posible que el siguiente contenido no sea adecuado para la indización: películas, programas de TV, cualquier elemento con audio y efectos de sonido mezclados o contenido mal gragado con ruido de fondo (silbido).

Un trabajo de indización puede generar las siguientes salidas:

* Archivos de subtítulos en los siguientes formatos: **TTML** y **WebVTT**.
  
    Entre los archivos de subtítulos se incluye una etiqueta denominada Recognizability que puntúa un trabajo de indización en función de lo reconocible que resulta la voz en el vídeo de origen.  Puede usar el valor de Recognizability para filtrar los archivos de salida por facilidad de uso. Una puntuación baja significa unos resultados de indización pobres debido a la calidad del audio.
* Archivo de palabras clave (XML).

En este artículo se muestra cómo crear trabajos de indexación para **indexar un recurso** e **indexar varios archivos**.

## <a name="using-configuration-and-manifest-files-for-indexing-tasks"></a>Uso de archivos de manifiesto y de manifiesto para tareas de indización
Puede especificar más detalles de las tareas de indización mediante la configuración de tarea. Por ejemplo, puede especificar los metadatos que se usarán para el archivo multimedia. Estos metadatos los usa el motor de lenguaje para ampliar su vocabulario y mejora considerablemente la precisión del reconocimiento de voz.  También es posible especificar los archivos de salida deseados.

También puede procesar varios archivos multimedia a la vez mediante un archivo de manifiesto.

Para obtener más información, consulte [Valores preestablecidos de tarea para Azure Media Indexer](./media-services-analytics-overview.md).

## <a name="index-an-asset"></a>Indización de un recurso
El método siguiente carga un archivo multimedia como un recurso y crea un trabajo para indizarlo.

Si no se especifica ningún archivo de configuración, el archivo multimedia se indexa con todos los valores predeterminados.

```csharp
    static bool RunIndexingJob(string inputMediaFilePath, string outputFolder, string configurationFile = "")
    {
        // Create an asset and upload the input media file to storage.
        IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
            "My Indexing Input Asset",
            AssetCreationOptions.None);

        // Declare a new job.
        IJob job = _context.Jobs.Create("My Indexing Job");

        // Get a reference to the Azure Media Indexer.
        string MediaProcessorName = "Azure Media Indexer";
        IMediaProcessor processor = GetLatestMediaProcessorByName(MediaProcessorName);

        // Read configuration from file if specified.
        string configuration = string.IsNullOrEmpty(configurationFile) ? "" : File.ReadAllText(configurationFile);

        // Create a task with the encoding details, using a string preset.
        ITask task = job.Tasks.AddNew("My Indexing Task",
            processor,
            configuration,
            TaskOptions.None);

        // Specify the input asset to be indexed.
        task.InputAssets.Add(asset);

        // Add an output asset to contain the results of the job.
        task.OutputAssets.AddNew("My Indexing Output Asset", AssetCreationOptions.None);

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
            Console.WriteLine("Exiting method due to job error.");
            return false;
        }

        // Download the job outputs.
        DownloadAsset(task.OutputAssets.First(), outputFolder);

        return true;
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
```

<!-- __ -->
### <a name="output-files"></a><a id="output_files"></a>Archivos de salida
De forma predeterminada, el trabajo de indización genera los siguientes archivos de salida. Los archivos se almacenan en el primer recurso de salida.

Cuando hay más de un archivo multimedia de entrada, Indexer genera un archivo de manifiesto para las salidas del trabajo denominado "JobResult.txt". Para cada archivo de elementos multimedia de entrada, los archivos de palabras clave, TTML y WebVTT resultantes se numeran secuencialmente y se les asigna un nombre usando el "Alias".

| Nombre de archivo | Descripción |
| --- | --- |
| **InputFileName.ttml**<br/>**InputFileName.vtt** |Archivos de subtítulos en los formatos TTML y WebVTT.<br/><br/>Se pueden usar para crear archivos de audio y vídeo accesibles para personas con discapacidades auditivas.<br/><br/>Entre los archivos de subtítulos, se incluye una etiqueta denominada <b>Recognizability</b> que puntúa un trabajo de indexación en función de lo reconocible que resulta la voz en el vídeo de origen.  Puede usar el valor de <b>Recognizability</b> para filtrar los archivos de salida por facilidad de uso. Una puntuación baja significa unos resultados de indización pobres debido a la calidad del audio. |
| **InputFileName.kw.xml<br/>InputFileName.info** |Archivos de información y palabras clave. <br/><br/>El archivo de palabras clave es un archivo XML que contiene las palabras clave que se extraen del contenido de voz, con información de frecuencia y desplazamiento. <br/><br/>El archivo de información es un archivo de texto sin formato que contiene información detallada acerca de cada término reconocido. La primera línea es especial y contiene la puntuación de reconocimiento. Cada línea siguiente es una lista separada por tabulaciones de los datos siguientes: hora de inicio, hora de finalización, palabra o frase y confianza. Las horas se proporcionan en segundos y la confianza se expresa como un número entre 0-1. <br/><br/>Línea de ejemplo: "1.20    1.45    word    0.67" <br/><br/>Estos archivos se pueden usar para varios propósitos, por ejemplo, para realizar análisis de voz, o para exponerse a motores de búsqueda como Bing, Google o Microsoft SharePoint para hacer que los archivos multimedia sean más reconocibles, o incluso para proporcionar anuncios más pertinentes. |
| **JobResult.txt** |Manifiesto de salida que solo aparece al indexar varios archivos y que contiene la información siguiente:<br/><br/><table border="1"><tr><th>InputFile</th><th>Alias</th><th>MediaLength</th><th>Error</th></tr><tr><td>a.mp4</td><td>Media_1</td><td>300</td><td>0</td></tr><tr><td>b.mp4</td><td>Media_2</td><td>0</td><td>3000</td></tr><tr><td>c.mp4</td><td>Media_3</td><td>600</td><td>0</td></tr></table><br/> |

Si no se indexan correctamente todos los archivos multimedia de entrada, el trabajo de indexación falla con el código de error 4000. Para obtener más información, consulte [Códigos de error](#error_codes).

## <a name="index-multiple-files"></a>Indización de varios archivos
El método siguiente carga varios archivos multimedia como un recurso y crea un trabajo para indizar todos esos archivos en un lote.

Se crea un archivo de manifiesto con la extensión ".lst" y se carga en el recurso. El archivo de manifiesto contiene la lista de todos los archivos del recurso. Para obtener más información, consulte [Valores preestablecidos de tarea para Azure Media Indexer](./media-services-analytics-overview.md).

```csharp
    static bool RunBatchIndexingJob(string[] inputMediaFiles, string outputFolder)
    {
        // Create an asset and upload to storage.
        IAsset asset = CreateAssetAndUploadMultipleFiles(inputMediaFiles,
            "My Indexing Input Asset - Batch Mode",
            AssetCreationOptions.None);

        // Create a manifest file that contains all the asset file names and upload to storage.
        string manifestFile = "input.lst";            
        File.WriteAllLines(manifestFile, asset.AssetFiles.Select(f => f.Name).ToArray());
        var assetFile = asset.AssetFiles.Create(Path.GetFileName(manifestFile));
        assetFile.Upload(manifestFile);

        // Declare a new job.
        IJob job = _context.Jobs.Create("My Indexing Job - Batch Mode");

        // Get a reference to the Azure Media Indexer.
        string MediaProcessorName = "Azure Media Indexer";
        IMediaProcessor processor = GetLatestMediaProcessorByName(MediaProcessorName);

        // Read configuration.
        string configuration = File.ReadAllText("batch.config");

        // Create a task with the encoding details, using a string preset.
        ITask task = job.Tasks.AddNew("My Indexing Task - Batch Mode",
            processor,
            configuration,
            TaskOptions.None);

        // Specify the input asset to be indexed.
        task.InputAssets.Add(asset);

        // Add an output asset to contain the results of the job.
        task.OutputAssets.AddNew("My Indexing Output Asset - Batch Mode", AssetCreationOptions.None);

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
            Console.WriteLine("Exiting method due to job error.");
            return false;
        }

        // Download the job outputs.
        DownloadAsset(task.OutputAssets.First(), outputFolder);

        return true;
    }

    private static IAsset CreateAssetAndUploadMultipleFiles(string[] filePaths, string assetName, AssetCreationOptions options)
    {
        IAsset asset = _context.Assets.Create(assetName, options);

        foreach (string filePath in filePaths)
        {
            var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
            assetFile.Upload(filePath);
        }

        return asset;
    }
```

### <a name="partially-succeeded-job"></a>Trabajo parcialmente correcto
Si no se indizan correctamente todos los archivos multimedia de entrada, el trabajo de indización fallará con el código de error 4000. Para obtener más información, consulte [Códigos de error](#error_codes).

Se generan las mismas salidas (como trabajos realizados correctamente). Puede consultar el archivo de manifiesto de salida para ver qué archivos de entrada son erróneos, según los valores de columna Error. En el caso de los archivos de entrada erróneos, no se generarán los archivos de palabras clave, TTML y WebVTT resultantes.

### <a name="task-preset-for-azure-media-indexer"></a><a id="preset"></a> Valores predefinidos de tarea para Azure Media Indexer
Para personalizar el procesamiento de Azure Media Indexer, puede proporcionar valores predefinidos de tarea junto con la tarea.  A continuación se describe el formato de este xml de configuración.

| Nombre | Necesario | Descripción |
| --- | --- | --- |
| **input** |false |Archivos de recursos que desea indexar.</p><p>Azure Media Indexer es compatible con los siguientes formatos de archivo multimedia: MP4, WMV, MP3, M4A, WMA, AAC, WAV.</p><p>Puede especificar el nombre del archivo o archivos en el atributo **name** o **list** del elemento **input** (como se muestra a continuación). Si no especifica qué archivo de recurso desea indexar, se selecciona el principal. Si no hay ningún archivo de recurso principal establecido, se indexa el primer recurso de entrada.</p><p>Para especificar explícitamente el nombre de archivo de recurso, haga:<br/>`<input name="TestFile.wmv">`<br/><br/>También puede indexar varios archivos de recursos al mismo tiempo (hasta 10). Para ello, siga estos pasos:<br/><br/><ol class="ordered"><li><p>Cree un archivo de texto (archivo de manifiesto) y asígnele una extensión .lst. </p></li><li><p>Agregue una lista de todos los nombres de archivos de recursos en el recurso de entrada a este archivo de manifiesto. </p></li><li><p>Agregue (cargue) el archivo de manifiesto al recurso.  </p></li><li><p>Especifique el nombre del archivo de manifiesto en el atributo list de la entrada.<br/>`<input list="input.lst">`</li></ol><br/><br/>Nota: Si agrega más de 10 archivos al archivo de manifiesto, el trabajo de indexación producirá un error con el código 2006. |
| **metadata** |false |Metadatos de los archivos de recurso especificados usados para la adaptación de vocabulario.  Resulta útil para preparar el indexador para reconocer palabras de vocabulario no estándar tales como nombres propios.<br/>`<metadata key="..." value="..."/>` <br/><br/>Puede proporcionar **valores** para **claves** predefinidas. Actualmente se admiten las siguientes claves:<br/><br/>"title" y "description", que se usan para que la adaptación de vocabulario ajuste el modelo de lenguaje a su trabajo y mejorar así la precisión del reconocimiento de voz.  Los valores se incluyen en las búsquedas de Internet para encontrar documentos de texto relevantes para el contexto, y el contenido se usa para aumentar el diccionario interno durante el tiempo que dura la tarea de indexación.<br/>`<metadata key="title" value="[Title of the media file]" />`<br/>`<metadata key="description" value="[Description of the media file] />"` |
| **features** <br/><br/> Agregado en la versión 1.2. Actualmente solo se admite la característica de reconocimiento de voz ("ASR"). |false |La característica de reconocimiento de voz tiene las siguientes claves de configuración:<table><tr><th><p>Clave</p></th>        <th><p>Descripción</p></th><th><p>Valor de ejemplo</p></th></tr><tr><td><p>Idioma</p></td><td><p>El lenguaje natural que se reconocerá en el archivo multimedia.</p></td><td><p>Inglés, español</p></td></tr><tr><td><p>CaptionFormats</p></td><td><p>Una lista separada con punto y coma de los formatos de subtítulo de salida que desee (si existen)</p></td><td><p>ttml;webvtt</p></td></tr><tr><td><p></p></td><td><p> </p></td><td><p>True; False</p></td></tr><tr><td><p>GenerateKeywords</p></td><td><p>Una marca booleana que especifica si se requiere un archivo XML de palabras clave o no.</p></td><td><p>True; False. </p></td></tr><tr><td><p>ForceFullCaption</p></td><td><p>Una marca booleana que especifica si se debe o no forzar leyendas completas (independientemente del nivel de confianza).  </p><p>El valor predeterminado es false, en cuyo caso las palabras y frases que tienen un nivel de confianza de un 50% menos se omite en las salidas de la leyenda final y se reemplazan por puntos suspensivos ("...").  Los puntos suspensivos son útiles para el control de calidad de las leyendas y la auditoría.</p></td><td><p>True; False. </p></td></tr></table> |

### <a name="error-codes"></a><a id="error_codes"></a>Códigos de error
En caso de error, Azure Media Indexer debe notificar uno de los siguientes códigos de error:

| Código | Nombre | Razones posibles |
| --- | --- | --- |
| 2000 |Configuración no válida |Configuración no válida |
| 2001 |Recursos de entrada no válidos |Faltan recursos de entrada o están vacíos. |
| 2002 |Manifiesto no válido |El manifiesto está vacío o contiene elementos no válidos. |
| 2003 |No se pudo descargar el archivo multimedia |Dirección URL no válida en el archivo de manifiesto. |
| 2004 |Protocolo no admitido |No se admite el protocolo de dirección URL de los medios. |
| 2005 |Tipo de archivo no admitido |No se admite el tipo de archivo multimedia de entrada. |
| 2006 |Hay demasiados archivos de entrada |Hay más de 10 archivos en el manifiesto de entrada. |
| 3000 |No se pudo descodificar el archivo multimedia |Códec multimedia no compatible <br/>or<br/> El archivo multimedia está dañado. <br/>or<br/> No hay ninguna secuencia de audio en el archivo multimedia de entrada. |
| 4000 |Indización de lote parcialmente correcta |Algunos de los archivos multimedia de entrada no se pudieron indizar. Para obtener más información, consulte <a href="#output_files">Archivos de salida</a>. |
| Otros |Errores internos |Póngase en contacto con el equipo de soporte técnico. indexer@microsoft.com |

## <a name="supported-languages"></a><a id="supported_languages"></a>Idiomas admitidos
Actualmente, se admiten los idiomas inglés y español.  

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>Vínculos relacionados
[Información general de análisis de Azure Media Services](media-services-analytics-overview.md)

[Indización de archivos multimedia con Azure Media Indexer 2 Preview](media-services-process-content-with-indexer2.md)