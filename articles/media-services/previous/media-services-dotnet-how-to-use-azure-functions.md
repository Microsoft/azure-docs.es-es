---
title: Desarrollo de Azure Functions con Media Services
description: En este tema se muestra cómo empezar a desarrollar Azure Functions con Media Services mediante Azure Portal.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 51bdcb01-1846-4e1f-bd90-70020ab471b0
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 43e16d15c48471d75a7ab5ce23898f1161a60153
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87052825"
---
# <a name="develop-azure-functions-with-media-services"></a>Desarrollo de Azure Functions con Media Services

En este artículo se muestra cómo empezar a crear instancias de Azure Functions que usan Media Services. La función de Azure definida en este artículo supervisa un contenedor de la cuenta de almacenamiento llamado **input** para los nuevos archivos MP4. Una vez que un archivo se coloca en el contenedor de almacenamiento, el desencadenador de blobs ejecuta la función. Puede consultar las funciones de Azure en la [Introducción](../../azure-functions/functions-overview.md) y otros temas relacionados en la sección **Azure Functions**.

Si desea explorar e implementar instancias de Azure Functions existentes que usan Azure Media Services, visite [Azure Functions de Media Services](https://github.com/Azure-Samples/media-services-dotnet-functions-integration). Este repositorio incluye ejemplos que usan Azure Media Services para mostrar flujos de trabajo relacionados con la ingesta directa de contenido desde Blob Storage, la codificación y la escritura de contenido de nuevo en Blob Storage. También incluye ejemplos de cómo supervisar las notificaciones de trabajo por medio de webhooks y colas de Azure. También puede desarrollar sus funciones a partir de los ejemplos del repositorio [Azure Functions de Media Services](https://github.com/Azure-Samples/media-services-dotnet-functions-integration). Para implementar las funciones, presione el botón **Implementar en Azure**.

## <a name="prerequisites"></a>Prerrequisitos

- Para poder crear la primera función, es necesario tener una cuenta de Azure activa. Si aún no tiene una cuenta de Azure, [tiene a su disposición la creación de una cuenta gratis](https://azure.microsoft.com/free/).
- Si va a crear instancias de Azure Functions que llevan a cabo acciones en su cuenta de Azure Media Services (AMS) o escuchan eventos enviados por Media Services, debe crear una cuenta de AMS, tal como se describe [aquí](media-services-portal-create-account.md).
    
## <a name="create-a-function-app"></a>Creación de una aplicación de función

1. Vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con su cuenta de Azure.
2. Cree una aplicación de función como se describe [aquí](../../azure-functions/functions-create-function-app-portal.md).

>[!NOTE]
> Una cuenta de almacenamiento especificada en la variable de entorno **StorageConnection** (vea el paso siguiente) debe estar en la misma región que su aplicación.

## <a name="configure-function-app-settings"></a>Configuración de la aplicación de función

Cuando desarrolle funciones de Media Services, es útil agregar variables de entorno que se usarán en todas sus funciones. Para definir la configuración de la aplicación, haga clic en el vínculo Configurar las opciones de la aplicación. Para obtener más información, consulte [Configuración de Azure Function App](../../azure-functions/functions-how-to-use-azure-function-app-settings.md). 

La función, definida en este artículo, da por hecho que tiene las siguientes variables de entorno en la configuración de la aplicación:

**AMSAADTenantDomain**: Punto de conexión de inquilino de Azure AD. Para más información sobre cómo conectarse a la API de AMS, consulte [este](media-services-use-aad-auth-to-access-ams-api.md) artículo.

**AMSRESTAPIEndpoint**:  identificador URI que representa el punto de conexión de API REST. 

**AMSClientId**: Identificador de cliente de la aplicación de Azure AD.

**AMSClientSecret**: Secreto de cliente de la aplicación de Azure AD.

**StorageConnection**: conexión de almacenamiento de la cuenta asociada a la cuenta de Media Services. Este valor se utiliza en los archivos **function.json** y **run.csx** (descritos a continuación).

## <a name="create-a-function"></a>Creación de una función

Una vez implementada su instancia de Function App, puede encontrarla entre Azure Functions de **App Services**.

1. Seleccione la aplicación de función y haga clic en **Nueva función**.
2. Elija el lenguaje **C#** y el escenario **Procesamiento de datos**.
3. Elija la plantilla **BlobTrigger**. Esta función se desencadena si se carga un blob en el contenedor **input**. El nombre **input** se especifica en **Path** (Ruta de acceso), en el paso siguiente.

    ![files](./media/media-services-azure-functions/media-services-azure-functions004.png)

4. Una vez que seleccione **BlobTrigger**, aparecerán algunos controles más en la página.

    ![files](./media/media-services-azure-functions/media-services-azure-functions005.png)

4. Haga clic en **Crear**. 

## <a name="files"></a>Archivos

La función de Azure está asociada a archivos de código y otros archivos que se describen en esta sección. Cuando se usa Azure Portal para crear una función, **function.json** y **run.csx** se crean automáticamente. Debe agregar o cargar un archivo **project.json**. El resto de esta sección proporciona una breve explicación de cada archivo y muestra sus definiciones.

![files](./media/media-services-azure-functions/media-services-azure-functions003.png)

### <a name="functionjson"></a>function.json

El archivo function.json define los enlaces de función y otras opciones de configuración. Este archivo se usa en tiempo de ejecución para determinar los eventos que se supervisarán y cómo pasar datos y devolverlos al ejecutarse una función. Para obtener más información, consulte [Enlaces HTTP y webhook en Azure Functions](../../azure-functions/functions-reference.md#function-code).

>[!NOTE]
>Establezca la propiedad **disabled** en **true** para impedir que se ejecute la función. 

Reemplace el contenido del archivo function.json existente por el código siguiente:

```json
{
  "bindings": [
    {
      "name": "myBlob",
      "type": "blobTrigger",
      "direction": "in",
      "path": "input/{filename}.mp4",
      "connection": "ConnectionString"
    }
  ],
  "disabled": false
}
```

### <a name="projectjson"></a>project.json

El archivo project.json contiene dependencias. Este es un ejemplo de archivo **project.json** que incluye los paquetes de Azure Media Services de .NET de NuGet. Tenga en cuenta que los números de versión cambian con las últimas actualizaciones a los paquetes, por lo que debe confirmar las versiones más recientes. 

Agregue la siguiente definición a project.json. 

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "windowsazure.mediaservices": "4.0.0.4",
        "windowsazure.mediaservices.extensions": "4.0.0.4",
        "Microsoft.IdentityModel.Clients.ActiveDirectory": "3.13.1",
        "Microsoft.IdentityModel.Protocol.Extensions": "1.0.2.206221351"
      }
    }
   }
}

```
    
### <a name="runcsx"></a>run.csx

Este es el código C# para la función.  La función definida a continuación supervisa un contenedor de la cuenta de almacenamiento llamado **input** (que es lo que se especificó en la ruta de acceso) para los nuevos archivos MP4. Una vez que un archivo se coloca en el contenedor de almacenamiento, el desencadenador de blobs ejecuta la función.
    
En el ejemplo definido en esta sección se muestra 

1. cómo incluir un recurso en una cuenta de Media Services (copiando un blob en un recurso de AMS) y 
2. cómo enviar un trabajo de codificación que usa "Streaming adaptable" preestablecido de Media Encoder Standard.

En un escenario real, lo más probable es que quisiera realizar un seguimiento del progreso del trabajo y, a continuación, publicar el recurso codificado. Para obtener más información, consulte [Uso de Azure WebHooks para supervisar las notificaciones de trabajo de Media Services](media-services-dotnet-check-job-progress-with-webhooks.md). Para obtener más ejemplos, consulte [Azure Functions de Media Services](https://github.com/Azure-Samples/media-services-dotnet-functions-integration).  

Reemplace el contenido del archivo run.csx existente por el código siguiente: Una vez que haya terminado de definir la función, haga clic en **Guardar y ejecutar**.

```csharp
#r "Microsoft.WindowsAzure.Storage"
#r "Newtonsoft.Json"
#r "System.Web"

using System;
using System.Net;
using System.Net.Http;
using Newtonsoft.Json;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.IO;
using System.Web;
using Microsoft.Azure;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.Azure.WebJobs;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
  
// Read values from the App.config file.

static readonly string _AADTenantDomain = Environment.GetEnvironmentVariable("AMSAADTenantDomain");
static readonly string _RESTAPIEndpoint = Environment.GetEnvironmentVariable("AMSRESTAPIEndpoint");
 
static readonly string _mediaservicesClientId = Environment.GetEnvironmentVariable("AMSClientId");
static readonly string _mediaservicesClientSecret = Environment.GetEnvironmentVariable("AMSClientSecret");

static readonly string _connectionString = Environment.GetEnvironmentVariable("ConnectionString");  

private static CloudMediaContext _context = null;
private static CloudStorageAccount _destinationStorageAccount = null;

public static void Run(CloudBlockBlob myBlob, string fileName, TraceWriter log)
{
    // NOTE that the variables {fileName} here come from the path setting in function.json
    // and are passed into the  Run method signature above. We can use this to make decisions on what type of file
    // was dropped into the input container for the function. 

    // No need to do any Retry strategy in this function, By default, the SDK calls a function up to 5 times for a 
    // given blob. If the fifth try fails, the SDK adds a message to a queue named webjobs-blobtrigger-poison.

    log.Info($"C# Blob trigger function processed: {fileName}.mp4");
    log.Info($"Media Services REST endpoint : {_RESTAPIEndpoint}");

    try
    {
        AzureAdTokenCredentials tokenCredentials = new AzureAdTokenCredentials(_AADTenantDomain,
                            new AzureAdClientSymmetricKey(_mediaservicesClientId, _mediaservicesClientSecret),
                            AzureEnvironments.AzureCloudEnvironment);
 
        AzureAdTokenProvider tokenProvider = new AzureAdTokenProvider(tokenCredentials);
 
        _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

        IAsset newAsset = CreateAssetFromBlob(myBlob, fileName, log).GetAwaiter().GetResult();

        // Step 2: Create an Encoding Job

        // Declare a new encoding job with the Standard encoder
        IJob job = _context.Jobs.Create("Azure Function - MES Job");

        // Get a media processor reference, and pass to it the name of the 
        // processor to use for the specific task.
        IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

        // Create a task with the encoding details, using a custom preset
        ITask task = job.Tasks.AddNew("Encode with Adaptive Streaming",
            processor,
            "Adaptive Streaming",
            TaskOptions.None); 

        // Specify the input asset to be encoded.
        task.InputAssets.Add(newAsset);

        // Add an output asset to contain the results of the job. 
        // This output is specified as AssetCreationOptions.None, which 
        // means the output asset is not encrypted. 
        task.OutputAssets.AddNew(fileName, AssetCreationOptions.None);

        job.Submit();
        log.Info("Job Submitted");

    }
    catch (Exception ex)
    {
        log.Error("ERROR: failed.");
        log.Info($"StackTrace : {ex.StackTrace}");
        throw ex;
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

public static async Task<IAsset> CreateAssetFromBlob(CloudBlockBlob blob, string assetName, TraceWriter log){
    IAsset newAsset = null;

    try{
        Task<IAsset> copyAssetTask = CreateAssetFromBlobAsync(blob, assetName, log);
        newAsset = await copyAssetTask;
        log.Info($"Asset Copied : {newAsset.Id}");
    }
    catch(Exception ex){
        log.Info("Copy Failed");
        log.Info($"ERROR : {ex.Message}");
        throw ex;
    }

    return newAsset;
}

/// <summary>
/// Creates a new asset and copies blobs from the specifed storage account.
/// </summary>
/// <param name="blob">The specified blob.</param>
/// <returns>The new asset.</returns>
public static async Task<IAsset> CreateAssetFromBlobAsync(CloudBlockBlob blob, string assetName, TraceWriter log)
{
     //Get a reference to the storage account that is associated with the Media Services account. 
    _destinationStorageAccount = CloudStorageAccount.Parse(_connectionString);

    // Create a new asset. 
    var asset = _context.Assets.Create(blob.Name, AssetCreationOptions.None);
    log.Info($"Created new asset {asset.Name}");

    IAccessPolicy writePolicy = _context.AccessPolicies.Create("writePolicy",
    TimeSpan.FromHours(4), AccessPermissions.Write);
    ILocator destinationLocator = _context.Locators.CreateLocator(LocatorType.Sas, asset, writePolicy);
    CloudBlobClient destBlobStorage = _destinationStorageAccount.CreateCloudBlobClient();

    // Get the destination asset container reference
    string destinationContainerName = (new Uri(destinationLocator.Path)).Segments[1];
    CloudBlobContainer assetContainer = destBlobStorage.GetContainerReference(destinationContainerName);

    try{
    assetContainer.CreateIfNotExists();
    }
    catch (Exception ex)
    {
    log.Error ("ERROR:" + ex.Message);
    }

    log.Info("Created asset.");

    // Get hold of the destination blob
    CloudBlockBlob destinationBlob = assetContainer.GetBlockBlobReference(blob.Name);

    // Copy Blob
    try
    {
    using (var stream = await blob.OpenReadAsync()) 
    {            
        await destinationBlob.UploadFromStreamAsync(stream);          
    }

    log.Info("Copy Complete.");

    var assetFile = asset.AssetFiles.Create(blob.Name);
    assetFile.ContentFileSize = blob.Properties.Length;
    assetFile.IsPrimary = true;
    assetFile.Update();
    asset.Update();
    }
    catch (Exception ex)
    {
    log.Error(ex.Message);
    log.Info (ex.StackTrace);
    log.Info ("Copy Failed.");
    throw;
    }

    destinationLocator.Delete();
    writePolicy.Delete();

    return asset;
}
```

## <a name="test-your-function"></a>Probar la función

Para probar la función, debe cargar un archivo MP4 en el contenedor **input** de la cuenta de almacenamiento que especificó en la cadena de conexión.  

1. Seleccione la cuenta de almacenamiento que especificó en la variable de entorno **StorageConnection**.
2. Haga clic en **Blobs**.
3. Haga clic en **+ Contenedor**. Asigne el nombre **input** al contenedor.
4. Presione **Cargar** y busque un archivo .mp4 que quiera cargar.

>[!NOTE]
> Al usar un desencadenador de blobs en un plan de consumo, puede haber un retraso de hasta 10 minutos en el procesamiento de nuevos blobs después de que una aplicación de función quede inactiva. Después de que se ejecute la aplicación de función, los blobs se procesan inmediatamente. Para más información, consulte [Desencadenadores y enlaces de Blob Storage](../../azure-functions/functions-bindings-storage-blob.md).

## <a name="next-steps"></a>Pasos siguientes

En este punto, está listo para iniciar el desarrollo de una aplicación de Media Services. 
 
Para más detalles, ejemplos completos y soluciones del uso de Azure Functions y Logic Apps con Azure Media Services para crear flujos de trabajo de creación de contenido personalizado, consulte el [ejemplo de integración de funciones de .NET con Media Services en GitHub](https://github.com/Azure-Samples/media-services-dotnet-functions-integration).

Además, consulte [Uso de Azure WebHooks para supervisar las notificaciones de trabajo de Media Services con .NET](media-services-dotnet-check-job-progress-with-webhooks.md). 

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
