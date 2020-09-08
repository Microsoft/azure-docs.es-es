---
title: Carga de archivos en una cuenta de Media Services mediante .NET | Microsoft Docs
description: Aprenda a obtener contenido multimedia en Media Services mediante la creación y carga de recursos.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: c9c86380-9395-4db8-acea-507c52066f73
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 51f6b40fc4cafad16ef6f6ca92fafa7ba1378c4a
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89255945"
---
# <a name="upload-files-into-a-media-services-account-using-net"></a>Cargar archivos en una cuenta de Media Services mediante .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

En Media Services, cargará (o ingerirá) los archivos digitales en un recurso. La entidad **Asset** puede contener archivos de vídeo, audio, imágenes, colecciones de miniaturas, pistas de texto y subtítulos (y los metadatos sobre estos archivos).  Una vez cargados los archivos, el contenido se almacena de forma segura en la nube para un posterior procesamiento y streaming.

Los archivos del recurso se denominan **archivos de recursos**. La instancia de **AssetFile** y el archivo multimedia real son dos objetos distintos. La instancia de AssetFile contiene metadatos sobre el archivo multimedia, mientras que el archivo multimedia contiene el contenido multimedia real.

## <a name="considerations"></a>Consideraciones

Se aplican las siguientes consideraciones:
 
 * Media Services usa el valor de la propiedad IAssetFile.Name al generar direcciones URL para el contenido de streaming (por ejemplo, http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters.) Por esta razón, no se permite la codificación porcentual. El valor de la propiedad **Name**no puede tener ninguno de los siguientes [caracteres reservados para la codificación porcentual](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters):!*'();:@&=+$,/?%#[]" Además, solo puede haber un '.' para la extensión del nombre de archivo.
* La longitud del nombre no debe ser superior a 260 caracteres.
* Existe un límite máximo de tamaño de archivo admitido para el procesamiento en Media Services. Consulte [este](media-services-quotas-and-limitations.md) artículo para obtener información más detallada acerca de la limitación de tamaño de archivo.
* Hay un límite de 1 000 000 directivas para diferentes directivas de AMS (por ejemplo, para la directiva de localizador o ContentKeyAuthorizationPolicy). Debe usar el mismo identificador de directiva si siempre usa los mismos permisos de acceso y días, por ejemplo, directivas para localizadores que vayan a aplicarse durante mucho tiempo (directivas distintas a carga). Para obtener más información, consulte [este](media-services-dotnet-manage-entities.md#limit-access-policies) artículo.

Al crear recursos, puede especificar las siguientes opciones de cifrado:

* **Ninguno** : no se utiliza cifrado. Este es el valor predeterminado. Cuando se utiliza esta opción, el contenido no está protegido en tránsito o en reposo en el almacenamiento.
  Si tiene previsto entregar un MP4 mediante una descarga progresiva, use esta opción. 
* **CommonEncryption** : utilice esta opción si va a cargar contenido que ya se ha cifrado y protegido con cifrado común o DRM de PlayReady (por ejemplo, Smooth Streaming protegido con DRM de PlayReady).
* **EnvelopeEncrypted** : utilice esta opción si va a cargar HLS cifrado con AES. Tenga en cuenta que los archivos deben haberse codificado y cifrado con Transform Manager.
* **StorageEncrypted** : cifra el contenido no cifrado localmente mediante el cifrado AES de 256 bits y, a continuación, lo carga en Azure Storage donde se almacena cifrado en reposo. Los recursos protegidos con el cifrado de almacenamiento se descifran automáticamente y se colocan en un sistema de archivos cifrados antes de la codificación y, opcionalmente, se vuelven a cifrar antes de volver a cargarlos como un nuevo recurso de salida. El caso de uso principal para el cifrado de almacenamiento es cuando desea proteger los archivos multimedia de entrada de alta calidad con un sólido cifrado en reposo en disco.
  
    Media Services proporciona cifrado de almacenamiento en disco para sus recursos, no por cable como Digital Rights Manager (DRM).
  
    Si el recurso tiene el almacenamiento cifrado, asegúrese de configurar la directiva de entrega de recursos. Para obtener más información, consulte [Configuración de directivas de entrega de recursos](media-services-dotnet-configure-asset-delivery-policy.md).

Si especifica que el recurso se cifre con una opción **CommonEncrypted** o una opción **EnvelopeEncrypted**, debe asociar el recurso a un elemento **ContentKey**. Para obtener más información, consulte [Creación de una ContentKey](media-services-dotnet-create-contentkey.md). 

Si especifica que el recurso se cifre con una opción **StorageEncrypted**, el SDK de Media Services para .NET crea un elemento **ContentKey** de **StorageEncrypted** para el recurso.

En este artículo se muestra cómo usar el SDK de Media Services para .NET, así como las extensiones del SDK de Media Services para .NET para cargar archivos en un recurso de Media Services.

## <a name="upload-a-single-file-with-media-services-net-sdk"></a>Carga de un solo archivo con el SDK .NET de Media Services

El código siguiente usa .NET para cargar un único archivo. Se crean y se destruyen las instancias AccessPolicy y Locator mediante la función Upload. 

```csharp
        static public IAsset CreateAssetAndUploadSingleFile(AssetCreationOptions assetCreationOptions, string singleFilePath)
        {
            if (!File.Exists(singleFilePath))
            {
                Console.WriteLine("File does not exist.");
                return null;
            }

            var assetName = Path.GetFileNameWithoutExtension(singleFilePath);
            IAsset inputAsset = _context.Assets.Create(assetName, assetCreationOptions);

            var assetFile = inputAsset.AssetFiles.Create(Path.GetFileName(singleFilePath));

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            return inputAsset;
        }
```


## <a name="upload-multiple-files-with-media-services-net-sdk"></a>Carga de varios archivos con el SDK .NET de Media Services
El código siguiente muestra cómo crear un recurso y cargar varios archivos.

El código hace lo siguiente:

* Crea un recurso vacío mediante el método CreateEmptyAsset definido en el paso anterior.
* Crea una instancia de **AccessPolicy** que define los permisos y la duración del acceso al recurso.
* Crea una instancia de **Locator** que proporciona acceso al recurso.
* Crea una instancia de **BlobTransferClient** . Este tipo representa a un cliente que funciona en los blobs de Azure. En este ejemplo, el cliente supervisa el progreso de la carga. 
* Enumera los archivos en el directorio especificado y crea una instancia de **AssetFile** para cada archivo.
* Carga los archivos en los Media Services con el método **UploadAsync** . 

> [!NOTE]
> Use el método UploadAsync para asegurarse de que las llamadas no provocan un bloqueo y los archivos se cargan en paralelo.
> 
> 

```csharp
        static public IAsset CreateAssetAndUploadMultipleFiles(AssetCreationOptions assetCreationOptions, string folderPath)
        {
            var assetName = "UploadMultipleFiles_" + DateTime.UtcNow.ToString();

            IAsset asset = _context.Assets.Create(assetName, assetCreationOptions);

            var accessPolicy = _context.AccessPolicies.Create(assetName, TimeSpan.FromDays(30),
                                                                AccessPermissions.Write | AccessPermissions.List);

            var locator = _context.Locators.CreateLocator(LocatorType.Sas, asset, accessPolicy);

            var blobTransferClient = new BlobTransferClient();
            blobTransferClient.NumberOfConcurrentTransfers = 20;
            blobTransferClient.ParallelTransferThreadCount = 20;

            blobTransferClient.TransferProgressChanged += blobTransferClient_TransferProgressChanged;

            var filePaths = Directory.EnumerateFiles(folderPath);

            Console.WriteLine("There are {0} files in {1}", filePaths.Count(), folderPath);

            if (!filePaths.Any())
            {
                throw new FileNotFoundException(String.Format("No files in directory, check folderPath: {0}", folderPath));
            }

            var uploadTasks = new List<Task>();
            foreach (var filePath in filePaths)
            {
                var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
                Console.WriteLine("Created assetFile {0}", assetFile.Name);

                // It is recommended to validate AssetFiles before upload. 
                Console.WriteLine("Start uploading of {0}", assetFile.Name);
                uploadTasks.Add(assetFile.UploadAsync(filePath, blobTransferClient, locator, CancellationToken.None));
            }

            Task.WaitAll(uploadTasks.ToArray());
            Console.WriteLine("Done uploading the files");

            blobTransferClient.TransferProgressChanged -= blobTransferClient_TransferProgressChanged;

            locator.Delete();
            accessPolicy.Delete();

            return asset;
        }

    static void  blobTransferClient_TransferProgressChanged(object sender, BlobTransferProgressChangedEventArgs e)
    {
        if (e.ProgressPercentage > 4) // Avoid startup jitter, as the upload tasks are added.
        {
            Console.WriteLine("{0}% upload competed for {1}.", e.ProgressPercentage, e.LocalFile);
        }
    }
```


Al cargar un número elevado de recursos, tenga en cuenta lo siguiente:

* Cree un nuevo objeto **CloudMediaContext** por subproceso. La clase **CloudMediaContext** no es segura para subprocesos.
* Aumente NumberOfConcurrentTransfers desde el valor predeterminado de 2 a un valor superior como 5. La configuración de esta propiedad afecta a todas las instancias de **CloudMediaContext**. 
* Mantenga ParallelTransferThreadCount en el valor predeterminado de 10.

## <a name="ingesting-assets-in-bulk-using-media-services-net-sdk"></a><a id="ingest_in_bulk"></a>Ingesta de activos en bloque con SDK .NET de Media Services
La carga de archivos de recursos de gran tamaño puede ser un obstáculo durante la creación de recursos. La ingesta de recursos en masa o "Ingesta en masa" implica la separación de la creación de recursos del proceso de carga. Para adoptar un enfoque de ingesta en masa, cree un manifiesto (IngestManifest) que describa el recurso y sus archivos asociados. A continuación, use el método de carga que prefiera para cargar los archivos asociados al contenedor de blobs del manifiesto. Los Microsoft Azure Media Services ven el contenedor de blobs asociado al manifesto. Una vez que se carga un archivo en el contenedor de blobs, los Microsoft Azure Media Services completan la creación de recursos según la configuración del recurso en el manifiesto (IngestManifestAsset).

Para crear un nuevo manifiesto IngestManifest, llame al método Create expuesto por la colección de IngestManifests en CloudMediaContext. Este método crea un nuevo manifiesto IngestManifest con el nombre de manifiesto proporcionado.

```csharp
    IIngestManifest manifest = context.IngestManifests.Create(name);
```

Cree los recursos que están asociados con el manifiesto IngestManifest en masa. Configure las opciones de cifrado deseadas en el recurso para la ingesta en masa.

```csharp
    // Create the assets that will be associated with this bulk ingest manifest
    IAsset destAsset1 = _context.Assets.Create(name + "_asset_1", AssetCreationOptions.None);
    IAsset destAsset2 = _context.Assets.Create(name + "_asset_2", AssetCreationOptions.None);
```

Un IngestManifestAsset permite asociar un recurso con un IngestManifest en masa para la ingesta en masa. También asocia los elementos AssetFile que constituirán cada recurso. Para crear un IngestManifestAsset, use el método Create en el contexto del servidor.

En el ejemplo siguiente se agregan dos nuevos IngestManifestAssets que asocian los dos recursos creados anteriormente al manifesto de ingesta en masa. Además, cada IngestManifestAsset asocia un conjunto de archivos que se cargan para cada recurso durante la ingesta en masa.  

```csharp
    string filename1 = _singleInputMp4Path;
    string filename2 = _primaryFilePath;
    string filename3 = _singleInputFilePath;

    IIngestManifestAsset bulkAsset1 =  manifest.IngestManifestAssets.Create(destAsset1, new[] { filename1 });
    IIngestManifestAsset bulkAsset2 =  manifest.IngestManifestAssets.Create(destAsset2, new[] { filename2, filename3 });
```

Puede usar cualquier aplicación cliente de alta velocidad capaz de cargar los archivos de recursos en el URI del contenedor de almacenamiento de blobs proporcionado por la propiedad **IIngestManifest.BlobStorageUriForUpload** de IngestManifest. 

El código siguiente muestra cómo usar el SDK de .NET para cargar los archivos de recursos.

```csharp
    static void UploadBlobFile(string containerName, string filename)
    {
        Task copytask = new Task(() =>
        {
            var storageaccount = new CloudStorageAccount(new StorageCredentials(_storageAccountName, _storageAccountKey), true);
            CloudBlobClient blobClient = storageaccount.CreateCloudBlobClient();
            CloudBlobContainer blobContainer = blobClient.GetContainerReference(containerName);

            string[] splitfilename = filename.Split('\\');
            var blob = blobContainer.GetBlockBlobReference(splitfilename[splitfilename.Length - 1]);

            using (var stream = System.IO.File.OpenRead(filename))
                blob.UploadFromStream(stream);

            lock (consoleWriteLock)
            {
                Console.WriteLine("Upload for {0} completed.", filename);
            }
        });

        copytask.Start();
    }
```

El código para cargar los archivos de recursos en el ejemplo usado en este artículo se muestra en el ejemplo de código siguiente:

```csharp
    UploadBlobFile(manifest.BlobStorageUriForUpload, filename1);
    UploadBlobFile(manifest.BlobStorageUriForUpload, filename2);
    UploadBlobFile(manifest.BlobStorageUriForUpload, filename3);
```

Puede determinar el progreso de la ingesta en bloque de todos los recursos asociados con un manifiesto **IngestManifest** mediante el sondeo de la propiedad Statistics de **IngestManifest**. Para actualizar la información de progreso, debe usar un **CloudMediaContext** nuevo cada vez que sondee la propiedad Statistics.

En el ejemplo siguiente se muestra cómo sondear un **IngestManifest**por su Id.

```csharp
    static void MonitorBulkManifest(string manifestID)
    {
       bool bContinue = true;
       while (bContinue)
       {
          CloudMediaContext context = GetContext();
          IIngestManifest manifest = context.IngestManifests.Where(m => m.Id == manifestID).FirstOrDefault();

          if (manifest != null)
          {
             lock(consoleWriteLock)
             {
                Console.WriteLine("\nWaiting on all file uploads.");
                Console.WriteLine("PendingFilesCount  : {0}", manifest.Statistics.PendingFilesCount);
                Console.WriteLine("FinishedFilesCount : {0}", manifest.Statistics.FinishedFilesCount);
                Console.WriteLine("{0}% complete.\n", (float)manifest.Statistics.FinishedFilesCount / (float)(manifest.Statistics.FinishedFilesCount + manifest.Statistics.PendingFilesCount) * 100);

                if (manifest.Statistics.PendingFilesCount == 0)
                {
                   Console.WriteLine("Completed\n");
                   bContinue = false;
                }
             }

             if (manifest.Statistics.FinishedFilesCount < manifest.Statistics.PendingFilesCount)
                Thread.Sleep(60000);
          }
          else // Manifest is null
             bContinue = false;
       }
    }
```


## <a name="upload-files-using-net-sdk-extensions"></a>Cargar archivos con extensiones del SDK de .NET
En el ejemplo siguiente se muestra cómo cargar un solo archivo con extensiones del SDK de .NET. En este caso, se usa el método **CreateFromFile**, pero también está disponible la versión asincrónica (**CreateFromFileAsync**). El método **CreateFromFile** le permite especificar el nombre de archivo, la opción de cifrado y una devolución de llamada para notificar el progreso de carga del archivo.

```csharp
    static public IAsset UploadFile(string fileName, AssetCreationOptions options)
    {
        IAsset inputAsset = _context.Assets.CreateFromFile(
            fileName,
            options,
            (af, p) =>
            {
                Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
            });

        Console.WriteLine("Asset {0} created.", inputAsset.Id);

        return inputAsset;
    }
```

En el ejemplo siguiente se llama a la función UploadFile y se especifica el cifrado de almacenamiento como la opción de creación de recursos.  

```csharp
    var asset = UploadFile(@"C:\VideoFiles\BigBuckBunny.mp4", AssetCreationOptions.StorageEncrypted);
```

## <a name="next-steps"></a>Pasos siguientes

Ahora puede codificar los recursos cargados. Para más información, consulte [Encode an asset using Media Encoder Standard with the Azure portal](media-services-portal-encode.md)(Codificación de recursos mediante el estándar de codificador multimedia con Azure Portal).

También puede usar Azure Functions para desencadenar un trabajo de codificación basado en un archivo que llega al contenedor configurado. Para más información, consulte [este ejemplo](https://azure.microsoft.com/resources/samples/media-services-dotnet-functions-integration/ ).

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-step"></a>Paso siguiente
Ahora que ha cargado un recurso en Media Services, vaya al artículo [Obtención de un procesador multimedia][How to Get a Media Processor].

[How to Get a Media Processor]: media-services-get-media-processor.md
