---
title: Administración de activos y entidades relacionadas con el SDK de Media Services para .NET
description: Obtenga información acerca de cómo administrar los recursos y las entidades relacionadas con el SDK de Media Services para .NET.
author: juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 1bd8fd42-7306-463d-bfe5-f642802f1906
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/01/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: b615e6526eb4187d354c4e276147743156b10bc1
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266024"
---
# <a name="managing-assets-and-related-entities-with-media-services-net-sdk"></a>Administración de activos y entidades relacionadas con el SDK de Media Services para .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-manage-entities.md)
> * [REST](media-services-rest-manage-entities.md)
> 
> 

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

En este tema se muestra cómo administrar entidades de Azure Media Services con. NET.

A partir del 1 de abril de 2017, se eliminarán automáticamente los registros de trabajo de más de 90 días de su cuenta, junto con los registros de tarea asociados, aunque el número total de registros no llegue a la cuota máxima. Por ejemplo, el 1 de abril de 2017, todos los registros de trabajo de la cuenta que sean anteriores al 31 de diciembre de 2016 se eliminarán automáticamente. Si desea archivar la información del trabajo o la tarea, puede usar el código que se describe en este tema.

## <a name="prerequisites"></a>Prerrequisitos

Configure el entorno de desarrollo y rellene el archivo app.config con la información de la conexión, como se describe en [Desarrollo de Media Services con .NET](media-services-dotnet-how-to-use.md). 

## <a name="get-an-asset-reference"></a>Obtención de una referencia de recurso
Una tarea frecuente es obtener una referencia a un recurso existente en Media Services. En el ejemplo de código siguiente se muestra cómo puede obtener una referencia de recurso de la colección de recursos en el objeto de contexto del servidor mediante un Id. de recurso. En el ejemplo de código siguiente se usa una consulta Linq para obtener una referencia a un objeto IAsset existente.

```csharp
    static IAsset GetAsset(string assetId)
    {
        // Use a LINQ Select query to get an asset.
        var assetInstance =
            from a in _context.Assets
            where a.Id == assetId
            select a;
        // Reference the asset as an IAsset.
        IAsset asset = assetInstance.FirstOrDefault();

        return asset;
    }
```

## <a name="list-all-assets"></a>Lista de todos los recursos
A medida que crece el número de recursos de almacenamiento, resulta útil mostrar una lista de los recursos. En el ejemplo de código siguiente se muestra cómo iterar a través de la colección de recursos en el objeto de contexto del servidor. Con cada recurso, el ejemplo de código también escribe algunos de sus valores de propiedad en la consola. Por ejemplo, cada recurso puede contener muchos archivos multimedia. El ejemplo de código escribe todos los archivos asociados con cada recurso.

```csharp
    static void ListAssets()
    {
        string waitMessage = "Building the list. This may take a few "
            + "seconds to a few minutes depending on how many assets "
            + "you have."
            + Environment.NewLine + Environment.NewLine
            + "Please wait..."
            + Environment.NewLine;
        Console.Write(waitMessage);

        // Create a Stringbuilder to store the list that we build. 
        StringBuilder builder = new StringBuilder();

        foreach (IAsset asset in _context.Assets)
        {
            // Display the collection of assets.
            builder.AppendLine("");
            builder.AppendLine("******ASSET******");
            builder.AppendLine("Asset ID: " + asset.Id);
            builder.AppendLine("Name: " + asset.Name);
            builder.AppendLine("==============");
            builder.AppendLine("******ASSET FILES******");

            // Display the files associated with each asset. 
            foreach (IAssetFile fileItem in asset.AssetFiles)
            {
                builder.AppendLine("Name: " + fileItem.Name);
                builder.AppendLine("Size: " + fileItem.ContentFileSize);
                builder.AppendLine("==============");
            }
        }

        // Display output in console.
        Console.Write(builder.ToString());
    }
```

## <a name="get-a-job-reference"></a>Obtención de una referencia de trabajo

Cuando se trabaja con tareas de procesamiento en el código de Media Services, a menudo necesitará obtener una referencia a un trabajo existente basado en un identificador. En el ejemplo de código siguiente se muestra cómo obtener una referencia a un objeto IJob de la colección de trabajos.

Puede ser necesario obtener una referencia de trabajo cuando se inicia un trabajo de codificación que tarda mucho en ejecutarse y debe comprobar el estado del trabajo en un subproceso. En casos como éste, cuando el método devuelve un subproceso, deberá recuperar una referencia actualizada a un trabajo.

```csharp
    static IJob GetJob(string jobId)
    {
        // Use a Linq select query to get an updated 
        // reference by Id. 
        var jobInstance =
            from j in _context.Jobs
            where j.Id == jobId
            select j;
        // Return the job reference as an Ijob. 
        IJob job = jobInstance.FirstOrDefault();

        return job;
    }
```

## <a name="list-jobs-and-assets"></a>Lista de trabajos y recursos
Una tarea relacionada importante es enumerar los recursos con sus respectivos trabajos asociados en Media Services. En el ejemplo de código siguiente se muestra cómo enumerar cada objeto IJob y, a continuación, para cada trabajo, muestra propiedades acerca del trabajo, todas las tareas relacionadas, todos los recursos de entrada y todos los recursos de salida. El código de este ejemplo puede ser útil para muchas otras tareas. Por ejemplo, si desea mostrar una lista de los recursos de salida de uno o más trabajos de codificación que ejecutó anteriormente, este código muestra cómo obtener acceso a los recursos de salida. Cuando tenga una referencia a un recurso de salida, puede entregar el contenido a otros usuarios o aplicaciones descargándolo o proporcionando direcciones URL. 

Para obtener más información sobre las opciones de entrega de recursos, consulte [Entrega de recursos con el SDK de Media Services para .NET](media-services-deliver-streaming-content.md).

```csharp
    // List all jobs on the server, and for each job, also list 
    // all tasks, all input assets, all output assets.

    static void ListJobsAndAssets()
    {
        string waitMessage = "Building the list. This may take a few "
            + "seconds to a few minutes depending on how many assets "
            + "you have."
            + Environment.NewLine + Environment.NewLine
            + "Please wait..."
            + Environment.NewLine;
        Console.Write(waitMessage);

        // Create a Stringbuilder to store the list that we build. 
        StringBuilder builder = new StringBuilder();

        foreach (IJob job in _context.Jobs)
        {
            // Display the collection of jobs on the server.
            builder.AppendLine("");
            builder.AppendLine("******JOB*******");
            builder.AppendLine("Job ID: " + job.Id);
            builder.AppendLine("Name: " + job.Name);
            builder.AppendLine("State: " + job.State);
            builder.AppendLine("Order: " + job.Priority);
            builder.AppendLine("==============");


            // For each job, display the associated tasks (a job  
            // has one or more tasks). 
            builder.AppendLine("******TASKS*******");
            foreach (ITask task in job.Tasks)
            {
                builder.AppendLine("Task Id: " + task.Id);
                builder.AppendLine("Name: " + task.Name);
                builder.AppendLine("Progress: " + task.Progress);
                builder.AppendLine("Configuration: " + task.Configuration);
                if (task.ErrorDetails != null)
                {
                    builder.AppendLine("Error: " + task.ErrorDetails);
                }
                builder.AppendLine("==============");
            }

            // For each job, display the list of input media assets.
            builder.AppendLine("******JOB INPUT MEDIA ASSETS*******");
            foreach (IAsset inputAsset in job.InputMediaAssets)
            {

                if (inputAsset != null)
                {
                    builder.AppendLine("Input Asset Id: " + inputAsset.Id);
                    builder.AppendLine("Name: " + inputAsset.Name);
                    builder.AppendLine("==============");
                }
            }

            // For each job, display the list of output media assets.
            builder.AppendLine("******JOB OUTPUT MEDIA ASSETS*******");
            foreach (IAsset theAsset in job.OutputMediaAssets)
            {
                if (theAsset != null)
                {
                    builder.AppendLine("Output Asset Id: " + theAsset.Id);
                    builder.AppendLine("Name: " + theAsset.Name);
                    builder.AppendLine("==============");
                }
            }

        }

        // Display output in console.
        Console.Write(builder.ToString());
    }
```

## <a name="list-all-access-policies"></a>Lista de todas las directivas de acceso
En Media Services, puede definir una directiva de acceso en un recurso o sus archivos. Una directiva de acceso define los permisos de un archivo o un recurso (tipo de acceso y la duración). En el código de Media Services, normalmente se define una directiva de acceso mediante la creación de un objeto IAccessPolicy y, a continuación, su asociación a un recurso existente. A continuación, cree un objeto ILocator, que permite proporcionar acceso directo a los recursos de Media Services. El proyecto de Visual Studio que acompaña a esta serie de documentación contiene varios ejemplos de código que muestran cómo crear y asignar directivas de acceso y localizadores a los activos.

En el ejemplo de código siguiente se muestra cómo enumerar todas las directivas de acceso del servidor y se muestra el tipo de permisos asociado a cada uno. Otra manera útil para ver las directivas de acceso es enumerar todos los objetos de ILocator en el servidor y, a continuación, para cada localizador, puede enumerar su directiva de acceso asociada mediante su propiedad AccessPolicy.

```csharp
    static void ListAllPolicies()
    {
        foreach (IAccessPolicy policy in _context.AccessPolicies)
        {
            Console.WriteLine("");
            Console.WriteLine("Name:  " + policy.Name);
            Console.WriteLine("ID:  " + policy.Id);
            Console.WriteLine("Permissions: " + policy.Permissions);
            Console.WriteLine("==============");

        }
    }
```
    
## <a name="limit-access-policies"></a>Directivas de limitación del acceso 

>[!NOTE]
> Hay un límite de 1 000 000 directivas para diferentes directivas de AMS (por ejemplo, para la directiva de localizador o ContentKeyAuthorizationPolicy). Debe usar el mismo identificador de directiva si siempre usa los mismos permisos de acceso y días, por ejemplo, directivas para localizadores que vayan a aplicarse durante mucho tiempo (directivas distintas a carga). 

Por ejemplo, puede crear un conjunto genérico de directivas con el siguiente código que se ejecutaría solo una vez en la aplicación. Puede registrar identificadores en un archivo de registro para usarlo más adelante:

```csharp
    double year = 365.25;
    double week = 7;
    IAccessPolicy policyYear = _context.AccessPolicies.Create("One Year", TimeSpan.FromDays(year), AccessPermissions.Read);
    IAccessPolicy policy100Year = _context.AccessPolicies.Create("Hundred Years", TimeSpan.FromDays(year * 100), AccessPermissions.Read);
    IAccessPolicy policyWeek = _context.AccessPolicies.Create("One Week", TimeSpan.FromDays(week), AccessPermissions.Read);

    Console.WriteLine("One year policy ID is: " + policyYear.Id);
    Console.WriteLine("100 year policy ID is: " + policy100Year.Id);
    Console.WriteLine("One week policy ID is: " + policyWeek.Id);
```

Luego, puede usar los identificadores existentes en un código similar al siguiente:

```csharp
    const string policy1YearId = "nb:pid:UUID:2a4f0104-51a9-4078-ae26-c730f88d35cf";


    // Get the standard policy for 1 year read only
    var tempPolicyId = from b in _context.AccessPolicies
                       where b.Id == policy1YearId
                       select b;
    IAccessPolicy policy1Year = tempPolicyId.FirstOrDefault();

    // Get the existing asset
    var tempAsset = from a in _context.Assets
                where a.Id == assetID
                select a;
    IAsset asset = tempAsset.SingleOrDefault();

    ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
        policy1Year,
        DateTime.UtcNow.AddMinutes(-5));
    Console.WriteLine("The locator base path is " + originLocator.BaseUri.ToString());
```

## <a name="list-all-locators"></a>Enumerar todos los localizadores
Un localizador es una dirección URL que proporciona una ruta de acceso directo para tener acceso a un recurso, junto con los permisos para el recurso definidos por la directiva de acceso asociada del localizador. Cada activo puede tener una colección de objetos ILocator asociados a él en su propiedad Locators. El contexto de servidor también tiene una colección de localizadores que contiene todos los localizadores.

En el ejemplo de código siguiente se enumeran todos los localizadores del servidor. Para cada localizador, muestra el identificador para la directiva de acceso y recursos relacionado. También muestra el tipo de permisos, la fecha de caducidad y la ruta de acceso completa al recurso.

Tenga en cuenta que una ruta de acceso del localizador a un recurso solo es una dirección URL base para el recurso. Para crear una ruta directa a archivos individuales a los que podría desplazarse un usuario o una aplicación, el código debe agregar la ruta de acceso del archivo específico a la ruta del localizador. Para obtener más información sobre cómo hacerlo, consulte el tema [Entrega de recursos con el SDK de Media Services para .NET](media-services-deliver-streaming-content.md).

```csharp
    static void ListAllLocators()
    {
        foreach (ILocator locator in _context.Locators)
        {
            Console.WriteLine("***********");
            Console.WriteLine("Locator Id: " + locator.Id);
            Console.WriteLine("Locator asset Id: " + locator.AssetId);
            Console.WriteLine("Locator access policy Id: " + locator.AccessPolicyId);
            Console.WriteLine("Access policy permissions: " + locator.AccessPolicy.Permissions);
            Console.WriteLine("Locator expiration: " + locator.ExpirationDateTime);
            // The locator path is the base or parent path (with included permissions) to access  
            // the media content of an asset. To create a full URL to a specific media file, take 
            // the locator path and then append a file name and info as needed.  
            Console.WriteLine("Locator base path: " + locator.Path);
            Console.WriteLine("");
        }
    }
```

## <a name="enumerating-through-large-collections-of-entities"></a>Enumeración de grandes colecciones de entidades
Al consultar entidades, hay un límite de 1000 entidades devueltas a la vez, porque la REST v2 pública limita los resultados de consulta a 1000. Debe usar Skip y Take al enumerar grandes colecciones de entidades. 

La siguiente función recorre todos los trabajos en la cuenta de Media Services proporcionada. Media Services devuelve 1000 trabajos en la colección de trabajos. La función usa Skip y Take para asegurarse de que se enumeran todos los trabajos (en el caso de que tenga más de 1000 trabajos en su cuenta).

```csharp
    static void ProcessJobs()
    {
        try
        {

            int skipSize = 0;
            int batchSize = 1000;
            int currentBatch = 0;

            while (true)
            {
                // Loop through all Jobs (1000 at a time) in the Media Services account
                IQueryable _jobsCollectionQuery = _context.Jobs.Skip(skipSize).Take(batchSize);
                foreach (IJob job in _jobsCollectionQuery)
                {
                    currentBatch++;
                    Console.WriteLine("Processing Job Id:" + job.Id);
                }

                if (currentBatch == batchSize)
                {
                    skipSize += batchSize;
                    currentBatch = 0;
                }
                else
                {
                    break;
                }
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
    }
```

## <a name="delete-an-asset"></a>Eliminación de un recurso
En el ejemplo siguiente se elimina un recurso.

```csharp
    static void DeleteAsset( IAsset asset)
    {
        // delete the asset
        asset.Delete();

        // Verify asset deletion
        if (GetAsset(asset.Id) == null)
            Console.WriteLine("Deleted the Asset");

    }
```

## <a name="delete-a-job"></a>Eliminación de un trabajo
Para eliminar un trabajo, debe comprobar el estado del trabajo, como se indica en la propiedad State. Los trabajos finalizados o cancelados pueden eliminarse, mientras que los trabajos que se encuentran en otros estados, por ejemplo, en cola, programado o en proceso, se deben cancelar primero y, a continuación, se pueden eliminar.

En el ejemplo de código siguiente se muestra un método para eliminar un trabajo; para ello, comprueba el estado de los trabajos y, a continuación, los elimina cuando el estado es finalizado o cancelado. Este código depende de la sección anterior de este tema para obtener una referencia a un trabajo: Obtención de una referencia de trabajo.

```csharp
    static void DeleteJob(string jobId)
    {
        bool jobDeleted = false;

        while (!jobDeleted)
        {
            // Get an updated job reference.  
            IJob job = GetJob(jobId);

            // Check and handle various possible job states. You can 
            // only delete a job whose state is Finished, Error, or Canceled.   
            // You can cancel jobs that are Queued, Scheduled, or Processing,  
            // and then delete after they are canceled.
            switch (job.State)
            {
                case JobState.Finished:
                case JobState.Canceled:
                case JobState.Error:
                    // Job errors should already be logged by polling or event 
                    // handling methods such as CheckJobProgress or StateChanged.
                    // You can also call job.DeleteAsync to do async deletes.
                    job.Delete();
                    Console.WriteLine("Job has been deleted.");
                    jobDeleted = true;
                    break;
                case JobState.Canceling:
                    Console.WriteLine("Job is cancelling and will be deleted "
                        + "when finished.");
                    Console.WriteLine("Wait while job finishes canceling...");
                    Thread.Sleep(5000);
                    break;
                case JobState.Queued:
                case JobState.Scheduled:
                case JobState.Processing:
                    job.Cancel();
                    Console.WriteLine("Job is scheduled or processing and will "
                        + "be deleted.");
                    break;
                default:
                    break;
            }

        }
    }
```


## <a name="delete-an-access-policy"></a>Eliminación de una directiva de acceso
En el ejemplo de código siguiente se muestra cómo obtener una referencia a una directiva de acceso basada en un Id. de directiva y, a continuación, eliminar la directiva.

```csharp
    static void DeleteAccessPolicy(string existingPolicyId)
    {
        // To delete a specific access policy, get a reference to the policy.  
        // based on the policy Id passed to the method.
        var policyInstance =
                from p in _context.AccessPolicies
                where p.Id == existingPolicyId
                select p;
        IAccessPolicy policy = policyInstance.FirstOrDefault();

        policy.Delete();

    }
```


## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
