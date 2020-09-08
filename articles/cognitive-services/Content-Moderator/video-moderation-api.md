---
title: 'Análisis de contenido de vídeo para detectar material inapropiado en C#: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Cómo analizar el contenido de vídeo para detectar diverso material inapropiado mediante el SDK de Content Moderator para .NET
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: how-to
ms.date: 05/18/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 9b389841bdba107ba27371387d4a6e5d1f009d41
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919359"
---
# <a name="analyze-video-content-for-objectionable-material-in-c"></a>Análisis de contenido de vídeo para detectar material inapropiado en C#

En este artículo se proporcionan información y ejemplos de código que le ayudarán a empezar a usar el [SDK de Content Moderator para .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) para examinar contenido de vídeo a fin de detectar contenido para adultos o atrevido.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar. 

## <a name="prerequisites"></a>Prerequisites
- Cualquier edición de [Visual Studio 2015 o 2017](https://www.visualstudio.com/downloads/).

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

La funcionalidad de moderación de vídeo de Content Moderator está disponible de forma gratuita como un **procesador de multimedia** (en versión preliminar pública) para Azure Media Services (AMS). Azure Media Services es un servicio de Azure especializado para almacenar y transmitir contenido de vídeo. 

### <a name="create-an-azure-media-services-account"></a>Creación de una cuenta de Azure Media Services

Para suscribirse a ASM y crear una cuenta de almacenamiento de Azure asociada, siga las instrucciones que se indican en [Creación de una cuenta de Azure Media Services](https://docs.microsoft.com/azure/media-services/media-services-portal-create-account). En esa cuenta de almacenamiento, cree un contenedor de almacenamiento de blobs.

### <a name="create-an-azure-active-directory-application"></a>Creación de una aplicación de Azure Active Directory

Vaya a la nueva suscripción de AMS en Azure Portal y seleccione **Acceso de API** en el menú lateral. Seleccione **Connect to Azure Media Services with service principal** (Conectarse a Azure Media Services con la entidad de servicio). Anote el valor del campo **Punto de conexión de la API de REST**; la necesitará más adelante.

En la sección **Aplicación de Azure AD**, seleccione **Crear nueva** y asigne un nombre al nuevo registro de aplicación de Azure AD, por ejemplo "VideoModADApp". Haga clic en **Guardar** y espere unos minutos a que se configure la aplicación. A continuación, verá el nuevo registro de aplicación en la sección **Aplicación de Azure AD** de la página.

Seleccione el registro de aplicación y haga clic en el botón **Administrar aplicación** que hay debajo. Anote el valor del campo **Id. de aplicación**; lo necesitará más adelante. Seleccione **Configuración** > **Claves** y escriba una descripción para una nueva clave (por ejemplo, "VideoModKey"). Haga clic en **Guardar** y, luego, observe el nuevo valor de clave. Copie esta cadena y guárdela en un lugar seguro.

Si desea un tutorial más detallado del proceso anterior, consulte [Introducción a la autenticación de Azure AD](https://docs.microsoft.com/azure/media-services/media-services-portal-get-started-with-aad).

Una vez hecho esto, puede usar el procesador de multimedia de moderación de vídeo de dos maneras diferentes.

## <a name="use-azure-media-services-explorer"></a>Usar el explorador de Azure Media Services

Azure Media Services Explorer es un front-end fácil de usar de AMS. Úselo para examinar la cuenta de AMS, cargar vídeos y analizar el contenido con el procesador de multimedia de Content Moderator. Descárguelo e instálelo desde [GitHub](https://github.com/Azure/Azure-Media-Services-Explorer/releases), o consulte la [entrada de blog de Azure Media Services Explorer](https://azure.microsoft.com/blog/managing-media-workflows-with-the-new-azure-media-services-explorer-tool/) para más información.

![Explorador de Azure Media Services con Content Moderator](images/ams-explorer-content-moderator.PNG)

## <a name="create-the-visual-studio-project"></a>Creación del proyecto de Visual Studio

1. En Visual Studio, cree un proyecto de **Aplicación de consola (.NET Framework)** y asígnele el nombre **VideoModeration**. 
1. Si hay otros proyectos en la solución, seleccione este como proyecto de inicio único.
1. Obtenga los paquetes NuGet requeridos. Haga clic con el botón derecho en el proyecto en el Explorador de soluciones y seleccione **Administrar paquetes NuGet**; a continuación, busque e instale los siguientes paquetes:
    - windowsazure.mediaservices
    - windowsazure.mediaservices.extensions

## <a name="add-video-moderation-code"></a>Incorporación del código de moderación de vídeo

A continuación, copiará y pegará el código de esta guía en el proyecto para implementar un escenario básico de moderación de contenido.

### <a name="update-the-programs-using-statements"></a>Actualizar las instrucciones using del programa

Agregue las siguientes instrucciones `using` al principio del archivo _Program.cs_.

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.IO;
using System.Threading;
using Microsoft.WindowsAzure.Storage.Blob;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Auth;
using System.Collections.Generic;
```

### <a name="set-up-resource-references"></a>Configuración de las referencias de recursos

Agregue los siguientes campos estáticos a la clase **Program** en _Program.cs_. Estos campos contienen la información necesaria para conectarse a su suscripción de AMS. Rellénelos con los valores que obtuvo en los pasos anteriores. Tenga en cuenta que `CLIENT_ID` es el valor de **Id. de aplicación** de la aplicación de Azure AD, y que `CLIENT_SECRET` es el valor de "VideoModKey" que ha creado para esa aplicación.

```csharp
// declare constants and globals
private static CloudMediaContext _context = null;
private static CloudStorageAccount _StorageAccount = null;

// Azure Media Services (AMS) associated Storage Account, Key, and the Container that has 
// a list of Blobs to be processed.
static string STORAGE_NAME = "YOUR AMS ASSOCIATED BLOB STORAGE NAME";
static string STORAGE_KEY = "YOUR AMS ASSOCIATED BLOB STORAGE KEY";
static string STORAGE_CONTAINER_NAME = "YOUR BLOB CONTAINER FOR VIDEO FILES";

private static StorageCredentials _StorageCredentials = null;

// Azure Media Services authentication. 
private const string AZURE_AD_TENANT_NAME = "microsoft.onmicrosoft.com";
private const string CLIENT_ID = "YOUR CLIENT ID";
private const string CLIENT_SECRET = "YOUR CLIENT SECRET";

// REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".      
private const string REST_API_ENDPOINT = "YOUR API ENDPOINT";

// Content Moderator Media Processor Nam
private const string MEDIA_PROCESSOR = "Azure Media Content Moderator";

// Input and Output files in the current directory of the executable
private const string INPUT_FILE = "VIDEO FILE NAME";
private const string OUTPUT_FOLDER = "";

// JSON settings file
private static readonly string CONTENT_MODERATOR_PRESET_FILE = "preset.json";

```

Si quiere usar un archivo de vídeo local (caso más sencillo), agréguelo al proyecto y escriba su ruta de acceso como el valor `INPUT_FILE` (las rutas de acceso relativas son relativas al directorio de ejecución).

También deberá crear el archivo _preset.json_ en el directorio actual y usarlo para especificar un número de versión. Por ejemplo:

```JSON
{
    "version": "2.0"
}
```

### <a name="load-the-input-videos"></a>Carga del vídeo de entrada

El método **Main** de la clase **Program** crea un contexto multimedia de Azure y luego un contexto de almacenamiento de Azure (en caso de que los vídeos estén en Blob Storage). El código restante examina un vídeo de una carpeta local, blob o varios blobs dentro de un contenedor de almacenamiento de Azure. Puede probar todas las opciones si decide comentar las otras líneas de código.

```csharp
// Create Azure Media Context
CreateMediaContext();

// Create Storage Context
CreateStorageContext();

// Use a file as the input.
IAsset asset = CreateAssetfromFile();

// -- OR ---

// Or a blob as the input
// IAsset asset = CreateAssetfromBlob((CloudBlockBlob)GetBlobsList().First());

// Then submit the asset to Content Moderator
RunContentModeratorJob(asset);

//-- OR ----

// Just run the content moderator on all blobs in a list (from a Blob Container)
// RunContentModeratorJobOnBlobs();
```

### <a name="create-an-azure-media-context"></a>Creación de un contexto multimedia de Azure

Agregue el método siguiente a la clase **Program**. Se usarán sus credenciales de AMS para permitir la comunicación con AMS.

```csharp
// Creates a media context from azure credentials
static void CreateMediaContext()
{
    // Get Azure AD credentials
    var tokenCredentials = new AzureAdTokenCredentials(AZURE_AD_TENANT_NAME,
        new AzureAdClientSymmetricKey(CLIENT_ID, CLIENT_SECRET),
        AzureEnvironments.AzureCloudEnvironment);

    // Initialize an Azure AD token
    var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

    // Create a media context
    _context = new CloudMediaContext(new Uri(REST_API_ENDPOINT), tokenProvider);
}
```

### <a name="add-the-code-to-create-an-azure-storage-context"></a>Agregar el código para crear un contexto de Azure Storage

Agregue el método siguiente a la clase **Program**. Usará el contexto de almacenamiento, creado con las credenciales de almacenamiento, para acceder al almacenamiento de blobs.

```csharp
// Creates a storage context from the AMS associated storage name and key
static void CreateStorageContext()
{
    // Get a reference to the storage account associated with a Media Services account. 
    if (_StorageCredentials == null)
    {
        _StorageCredentials = new StorageCredentials(STORAGE_NAME, STORAGE_KEY);
    }
    _StorageAccount = new CloudStorageAccount(_StorageCredentials, false);
}
```

### <a name="add-the-code-to-create-azure-media-assets-from-local-file-and-blob"></a>Agregar el código para crear recursos de Azure Media a partir de un archivo local y un blob

El procesador de multimedia de Content Moderator ejecuta trabajos en la opción **Recursos** de la plataforma Azure Media Services.
Estos métodos crean los recursos de un archivo local o un blob asociado.

```csharp
// Creates an Azure Media Services Asset from the video file
static IAsset CreateAssetfromFile()
{
    return _context.Assets.CreateFromFile(INPUT_FILE, AssetCreationOptions.None); ;
}

// Creates an Azure Media Services asset from your blog storage
static IAsset CreateAssetfromBlob(CloudBlockBlob Blob)
{
    // Create asset from the FIRST blob in the list and return it
    return _context.Assets.CreateFromBlob(Blob, _StorageCredentials, AssetCreationOptions.None);
}
```

### <a name="add-the-code-to-scan-a-collection-of-videos-as-blobs-within-a-container"></a>Agregar el código para examinar una colección de vídeos (como blobs) en un contenedor

```csharp
// Runs the Content Moderator Job on all Blobs in a given container name
static void RunContentModeratorJobOnBlobs()
{
    // Get the reference to the list of Blobs. See the following method.
    var blobList = GetBlobsList();

    // Iterate over the Blob list items or work on specific ones as needed
    foreach (var sourceBlob in blobList)
    {
        // Create an Asset
        IAsset asset = _context.Assets.CreateFromBlob((CloudBlockBlob)sourceBlob,
                            _StorageCredentials, AssetCreationOptions.None);
        asset.Update();

        // Submit to Content Moderator
        RunContentModeratorJob(asset);
    }
}

// Get all blobs in your container
static IEnumerable<IListBlobItem> GetBlobsList()
{
    // Get a reference to the Container within the Storage Account
    // that has the files (blobs) for moderation
    CloudBlobClient CloudBlobClient = _StorageAccount.CreateCloudBlobClient();
    CloudBlobContainer MediaBlobContainer = CloudBlobClient.GetContainerReference(STORAGE_CONTAINER_NAME);

    // Get the reference to the list of Blobs 
    var blobList = MediaBlobContainer.ListBlobs();
    return blobList;
}
```

### <a name="add-the-method-to-run-the-content-moderator-job"></a>Agregar el método para ejecutar el trabajo de Content Moderator

```csharp
// Run the Content Moderator job on the designated Asset from local file or blob storage
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

### <a name="add-helper-functions"></a>Incorporación de funciones auxiliares

Estos métodos descargan el archivo de salida (JSON) de Content Moderator del recurso de Azure Media Services, y le permiten hacer un seguimiento del estado del trabajo de moderación para que el programa pueda registrar el estado de ejecución en la consola.

```csharp
static void DownloadAsset(IAsset asset, string outputDirectory)
{
    foreach (IAssetFile file in asset.AssetFiles)
    {
        file.Download(Path.Combine(outputDirectory, file.Name));
    }
}

// event handler for Job State
static void StateChanged(object sender, JobStateChangedEventArgs e)
{
    Console.WriteLine("Job state changed event:");
    Console.WriteLine("  Previous state: " + e.PreviousState);
    Console.WriteLine("  Current state: " + e.CurrentState);
    switch (e.CurrentState)
    {
        case JobState.Finished:
            Console.WriteLine();
            Console.WriteLine("Job finished.");
            break;
        case JobState.Canceling:
        case JobState.Queued:
        case JobState.Scheduled:
        case JobState.Processing:
            Console.WriteLine("Please wait...\n");
            break;
        case JobState.Canceled:
            Console.WriteLine("Job is canceled.\n");
            break;
        case JobState.Error:
            Console.WriteLine("Job failed.\n");
            break;
        default:
            break;
    }
}
```

### <a name="run-the-program-and-review-the-output"></a>Ejecutar el programa y revisar la salida

Una vez completado el trabajo de moderación de contenido, analice la respuesta JSON. Esta consta de estos elementos:

- Resumen de información del vídeo
- **Capturas**  denominadas "**fragmentos**".
- **Fotogramas clave**  denominados "**eventos**" con una marca **reviewRecommended" (= true o false)"** basada en puntuaciones de tipo **Adultos** e **Inapropiado**.
- Los valores **start**, **duration** , **totalDuration** y **timestamp** están entrecomillados. Divida con el valor **timescale** para obtener el número en segundos.
 
> [!NOTE]
> - `adultScore` representa la posible presencia y la puntuación de predicción de contenido que se pueda considerar sexualmente explícito o para adultos en ciertas situaciones.
> - `racyScore` representa la posible presencia y la puntuación de predicción de contenido que se pueda considerar sexualmente insinuante o para adultos en ciertas situaciones.
> - `adultScore` y `racyScore` están entre 0 y 1. Cuanto más alta sea la puntuación, mayor será la predicción del modelo sobre la aplicabilidad de la categoría. Esta versión preliminar usa un modelo estadístico, en lugar de resultados codificados manualmente. Se recomienda realizar pruebas con contenido propio para determinar cómo se alinea cada categoría en función de sus propios requisitos.
> - `reviewRecommended` es true o false según los umbrales de puntuación internos. Los clientes deben evaluar si quieren usar este valor o decidir si desean umbrales personalizados basados en sus directivas de contenido.

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

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo generar [reseñas de vídeos](video-reviews-quickstart-dotnet.md) desde la salida de moderación.

Agregue la [moderación de transcripciones](video-transcript-moderation-review-tutorial-dotnet.md) a sus reseñas de vídeos.

Consulte el tutorial detallado sobre cómo crear una [solución completa de moderación de vídeos y transcripciones ](video-transcript-moderation-review-tutorial-dotnet.md).

[Descargue la solución de Visual Studio](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator) para este y otras guías de inicio rápido de Content Moderator para .NET.
