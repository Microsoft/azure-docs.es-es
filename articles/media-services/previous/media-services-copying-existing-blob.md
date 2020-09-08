---
title: Copiado de blobs desde una cuenta de almacenamiento a un recurso de Azure Media Services | Microsoft Docs
description: En este tema se muestra cómo copiar un blob existente en un recurso de Media Services. El ejemplo usa las extensiones del SDK de Azure Media Services para .NET.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 4ac981fb3aa17cbc242b5cc51cd579f926bad034
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89257983"
---
# <a name="copying-existing-blobs-into-a-media-services-asset"></a>Copiado de un blob existente en un recurso de Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

En este tema se muestra cómo copiar blobs de una cuenta de almacenamiento a un nuevo recurso de Azure Media Services (AMS) mediante [Extensiones del SDK de Azure Media Services para .NET](https://github.com/Azure/azure-sdk-for-media-services-extensions/).

No debe intentar cambiar el contenido de los contenedores de blobs que generó Media Services sin usar las API de Media Services.

Los métodos de extensión funcionan con:

- Recursos habituales.
- Recursos de archivo dinámico (formato FragBlob).
- Recursos de origen y de destino que pertenecen a diferentes cuentas de Media Services (incluso a través de centros de datos diferentes). Sin embargo, se podrían cobrar gastos por hacerlo. Para obtener más información sobre los precios, vea [Transferencias de datos](https://azure.microsoft.com/pricing/#header-11).

En el artículo se muestran dos ejemplos de código:

1. Copia de blobs de un recurso de una cuenta de AMS en un recurso nuevo de otra cuenta de AMS.
2. Copia de blobs de una cuenta de almacenamiento en un recurso nuevo de una cuenta de AMS.

## <a name="copy-blobs-between-two-ams-accounts"></a>Copia de blobs entre dos cuentas de AMS  

### <a name="prerequisites"></a>Prerrequisitos

Dos cuentas de Media Services. Vea el artículo [Cómo crear una cuenta de Media Services](media-services-portal-create-account.md).

### <a name="download-sample"></a>Descarga de un ejemplo
Puede seguir los pasos descritos en este artículo o puede descargar un ejemplo que contiene el código que se describe en este artículo desde [aquí](https://azure.microsoft.com/documentation/samples/media-services-dotnet-copy-blob-into-asset/).

### <a name="set-up-your-project"></a>Configurar su proyecto

1. Configure el entorno de desarrollo, como se describe en [Desarrollo de Media Services con .NET](media-services-dotnet-how-to-use.md). 
2. Agregue la sección appSettings al archivo .config y actualice los valores en función de las cuentas de Media Services, la cuenta de almacenamiento de destino y el identificador del recurso de origen.  

```xml
<appSettings>
    <add key="AMSSourceAADTenantDomain" value="tenant"/>
    <add key="AMSSourceRESTAPIEndpoint" value="endpoint"/>

    <add key="SourceAMSClientId" value="clientID"/>
    <add key="SourceAMSClientSecret" value="clientSecret"/>

    <add key="SourceAssetID" value="nb:cid:UUID:<id>"/>

    <add key="AMSDestAADTenantDomain" value="tenant"/>
    <add key="AMSDestRESTAPIEndpoint" value="endpoint"/>

    <add key="DestAMSClientId" value="clientID"/>
    <add key="DestAMSClientSecret" value="clientSecret"/>

    <add key="DestStorageAccountName" value="name"/>
    <add key="DestStorageAccountKey" value="key"/>

</appSettings>
```

### <a name="copy-blobs-from-an-asset-in-one-ams-account-into-an-asset-in-another-ams-account"></a>Copia de blobs de un recurso de una cuenta de AMS en un recurso de otra cuenta de AMS

El código siguiente utiliza el método **IAsset.Copy** de extensión para copiar todos los archivos del recurso de origen en el recurso de destino mediante una única extensión.

```csharp
using System;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Linq;
using System.Configuration;
using Microsoft.WindowsAzure.Storage.Auth;

namespace CopyExistingBlobsIntoAsset
{
    class Program
    {
        static string _sourceAADTenantDomain = 
            ConfigurationManager.AppSettings["AMSSourceAADTenantDomain"];
        static string _sourceRESTAPIEndpoint = 
            ConfigurationManager.AppSettings["AMSSourceRESTAPIEndpoint"];
        static string _sourceClientId = 
            ConfigurationManager.AppSettings["SourceAMSClientId"];
        static string _sourceClientSecret = 
            ConfigurationManager.AppSettings["SourceAMSClientSecret"];

        static string _destAADTenantDomain = 
            ConfigurationManager.AppSettings["AMSDestAADTenantDomain"];
        static string _destRESTAPIEndpoint = 
            ConfigurationManager.AppSettings["AMSDestRESTAPIEndpoint"];
        static string _destClientId = 
            ConfigurationManager.AppSettings["DestAMSClientId"];
        static string _destClientSecret = 
            ConfigurationManager.AppSettings["DestAMSClientSecret"];

        static string _destStorageAccountName = 
            ConfigurationManager.AppSettings["DestStorageAccountName"];
        static string _destStorageAccountKey = 
            ConfigurationManager.AppSettings["DestStorageAccountKey"];
        static string _sourceAssetID = 
            ConfigurationManager.AppSettings["SourceAssetID"];

        private static CloudMediaContext _sourceContext = null;
        private static CloudMediaContext _destContext = null;

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials1 = new AzureAdTokenCredentials(_sourceAADTenantDomain,
                   new AzureAdClientSymmetricKey(_sourceClientId, _sourceClientSecret),
                   AzureEnvironments.AzureCloudEnvironment);

            AzureAdTokenCredentials tokenCredentials2 = new AzureAdTokenCredentials(_destAADTenantDomain,
                   new AzureAdClientSymmetricKey(_destClientId, _destClientSecret),
                   AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider1 = new AzureAdTokenProvider(tokenCredentials1);
            var tokenProvider2 = new AzureAdTokenProvider(tokenCredentials2);

            // Create the context for your source Media Services account.
            _sourceContext = new CloudMediaContext(new Uri(_sourceRESTAPIEndpoint), tokenProvider1);

            // Create the context for your destination Media Services account.
            _destContext = new CloudMediaContext(new Uri(_destRESTAPIEndpoint), tokenProvider2);

            // Get the credentials of the default Storage account bound to your destination Media Services account.
            StorageCredentials destinationStorageCredentials =
                new StorageCredentials(_destStorageAccountName, _destStorageAccountKey);

            // Get a reference to the source asset in the source context.
            IAsset sourceAsset = _sourceContext.Assets.Where(asset => asset.Id == _sourceAssetID).First();

            // Create an empty destination asset in the destination context.
            IAsset destinationAsset = _destContext.Assets.Create(sourceAsset.Name, AssetCreationOptions.None);

            // Copy the files in the source asset instance into the destination asset instance.
            // There is an additional overload with async support.
            sourceAsset.Copy(destinationAsset, destinationStorageCredentials);

            Console.WriteLine("Done");
        }
    }
}
```

## <a name="copy-blobs-from-a-storage-account-into-an-ams-account"></a>Copia de blobs de una cuenta de almacenamiento a una cuenta de AMS 

### <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de almacenamiento desde la que se van a copiar los blobs.
- Una cuenta de AMS en la que se van a copiar los blobs.

### <a name="set-up-your-project"></a>Configurar su proyecto

1. Configure el entorno de desarrollo, como se describe en [Desarrollo de Media Services con .NET](media-services-dotnet-how-to-use.md). 
2. Agregue la sección appSettings al archivo .config y actualice los valores en función de las cuentas de almacenamiento de origen y las cuentas de AMS de destino.

```xml
<appSettings>
    <add key="SourceStorageAccountName" value="name" />
    <add key="SourceStorageAccountKey" value="key" />
    <add key="NameOfBlobContainerYouWantToCopy" value="BlobContainerName"/>
    
    <add key="AMSAADTenantDomain" value="tenant"/>
    <add key="AMSRESTAPIEndpoint" value="endpoint"/>
    <add key="AMSClientId" value="clientID"/>
    <add key="AMSClientSecret" value="clientSecret"/>
    <add key="AMSStorageAccountName" value="name"/>
    <add key="AMSStorageAccountKey" value="key"/>
</appSettings>
```

### <a name="copy-blobs-from-some-storage-account-into-a-new-asset-in-an-ams-account"></a>Copia de blobs de una cuenta de almacenamiento en un recurso nuevo de una cuenta de AMS.

El código siguiente copia blobs de una cuenta de almacenamiento a un recurso de Media Services. 

>[!NOTE]
>Hay un límite de 1 000 000 directivas para diferentes directivas de AMS (por ejemplo, para la directiva de localizador o ContentKeyAuthorizationPolicy). Debe usar el mismo identificador de directiva si siempre usa los mismos permisos de acceso y días, por ejemplo, directivas para localizadores que vayan a aplicarse durante mucho tiempo (directivas distintas a carga). Para obtener más información, consulte [este](media-services-dotnet-manage-entities.md#limit-access-policies) artículo.

```csharp
using System;
using System.Configuration;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
    
namespace CopyExistingBlobsIntoAsset
{
    class Program
    {
        // Read values from the App.config file.
        private static readonly string _sourceStorageAccountName =
            ConfigurationManager.AppSettings["SourceStorageAccountName"];
        private static readonly string _sourceStorageAccountKey =
            ConfigurationManager.AppSettings["SourceStorageAccountKey"];
        private static readonly string _NameOfBlobContainerYouWantToCopy =
            ConfigurationManager.AppSettings["NameOfBlobContainerYouWantToCopy"];

        private static readonly string _AMSAADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _AMSRESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];
        private static readonly string _AMSStorageAccountName =
            ConfigurationManager.AppSettings["AMSStorageAccountName"];
        private static readonly string _AMSStorageAccountKey =
            ConfigurationManager.AppSettings["AMSStorageAccountKey"];

        // Field for service context.
        private static CloudMediaContext _context = null;
        private static CloudStorageAccount _sourceStorageAccount = null;
        private static CloudStorageAccount _destinationStorageAccount = null;

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials = new AzureAdTokenCredentials(_AMSAADTenantDomain,
               new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
               AzureEnvironments.AzureCloudEnvironment);
            
            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            // Create the context for your source Media Services account.
            _context = new CloudMediaContext(new Uri(_AMSRESTAPIEndpoint), tokenProvider);

            _sourceStorageAccount =
                new CloudStorageAccount(new StorageCredentials(_sourceStorageAccountName,
                    _sourceStorageAccountKey), true);

            _destinationStorageAccount =
                new CloudStorageAccount(new StorageCredentials(_AMSStorageAccountName,
                    _AMSStorageAccountKey), true);

            CloudBlobClient sourceCloudBlobClient =
                _sourceStorageAccount.CreateCloudBlobClient();
            CloudBlobContainer sourceContainer =
                sourceCloudBlobClient.GetContainerReference(_NameOfBlobContainerYouWantToCopy);

            CreateAssetFromExistingBlobs(sourceContainer);

            Console.WriteLine("Done");
        }

        static private IAsset CreateAssetFromExistingBlobs(CloudBlobContainer sourceBlobContainer)
        {
            CloudBlobClient destBlobStorage = _destinationStorageAccount.CreateCloudBlobClient();

            // Create a new asset. 
            IAsset asset = _context.Assets.Create("NewAsset_" + Guid.NewGuid(), AssetCreationOptions.None);

            IAccessPolicy writePolicy = _context.AccessPolicies.Create("writePolicy",
                TimeSpan.FromHours(24), AccessPermissions.Write);

            ILocator destinationLocator =
                _context.Locators.CreateLocator(LocatorType.Sas, asset, writePolicy);

            // Get the asset container URI and Blob copy from mediaContainer to assetContainer. 
            CloudBlobContainer destAssetContainer =
                destBlobStorage.GetContainerReference((new Uri(destinationLocator.Path)).Segments[1]);

            if (destAssetContainer.CreateIfNotExists())
            {
                destAssetContainer.SetPermissions(new BlobContainerPermissions
                {
                    PublicAccess = BlobContainerPublicAccessType.Blob
                });
            }

            var blobList = sourceBlobContainer.ListBlobs();

            foreach (CloudBlockBlob sourceBlob in blobList)
            {
                var assetFile = asset.AssetFiles.Create((sourceBlob as ICloudBlob).Name);

                ICloudBlob destinationBlob = destAssetContainer.GetBlockBlobReference(assetFile.Name);

                CopyBlob(sourceBlob, destAssetContainer);
                
                sourceBlob.FetchAttributes();
                assetFile.ContentFileSize = (sourceBlob as ICloudBlob).Properties.Length;
                assetFile.Update();
                Console.WriteLine("File {0} is of {1} size", assetFile.Name, assetFile.ContentFileSize);
            }

            asset.Update();

            destinationLocator.Delete();
            writePolicy.Delete();

            // Set the primary asset file.
            // If, for example, we copied a set of Smooth Streaming files, 
            // set the .ism file to be the primary file. 
            // If we, for example, copied an .mp4, then the mp4 would be the primary file. 
            var ismAssetFile = asset.AssetFiles.ToList().
                Where(f => f.Name.EndsWith(".ism", StringComparison.OrdinalIgnoreCase)).FirstOrDefault();

            // The following code assigns the first .ism file as the primary file in the asset.
            // An asset should have one .ism file.  
            if (ismAssetFile != null)
            {
                ismAssetFile.IsPrimary = true;
                ismAssetFile.Update();
            }

            return asset;
        }

        /// <summary>
        /// Copies the specified blob into the specified container.
        /// </summary>
        /// <param name="sourceBlob">The source container.</param>
        /// <param name="destinationContainer">The destination container.</param>
        static private void CopyBlob(ICloudBlob sourceBlob, CloudBlobContainer destinationContainer)
        {
            var signature = sourceBlob.GetSharedAccessSignature(new SharedAccessBlobPolicy
            {
                Permissions = SharedAccessBlobPermissions.Read,
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24)
            });

            var destinationBlob = destinationContainer.GetBlockBlobReference(sourceBlob.Name);

            if (destinationBlob.Exists())
            {
                Console.WriteLine(string.Format("Destination blob '{0}' already exists. Skipping.", destinationBlob.Uri));
            }
            else
            {

                // Display the size of the source blob.
                Console.WriteLine(sourceBlob.Properties.Length);

                Console.WriteLine(string.Format("Copy blob '{0}' to '{1}'", sourceBlob.Uri, destinationBlob.Uri));
                destinationBlob.StartCopy(new Uri(sourceBlob.Uri.AbsoluteUri + signature));

                while (true)
                {
                    // The StartCopyFromBlob is an async operation, 
                    // so we want to check if the copy operation is completed before proceeding. 
                    // To do that, we call FetchAttributes on the blob and check the CopyStatus. 
                    destinationBlob.FetchAttributes();
                    if (destinationBlob.CopyState.Status != CopyStatus.Pending)
                    {
                        break;
                    }
                    //It's still not completed. So wait for some time.
                    System.Threading.Thread.Sleep(1000);
                }

                // Display the size of the destination blob.
                Console.WriteLine(destinationBlob.Properties.Length);

            }
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

Ahora puede codificar los recursos cargados. Para más información, consulte [Encode an asset using Media Encoder Standard with the Azure portal](media-services-portal-encode.md)(Codificación de recursos mediante el estándar de codificador multimedia con Azure Portal).

También puede usar Azure Functions para desencadenar un trabajo de codificación basado en un archivo que llega al contenedor configurado. Para más información, consulte [este ejemplo](https://azure.microsoft.com/resources/samples/media-services-dotnet-functions-integration/ ).

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
