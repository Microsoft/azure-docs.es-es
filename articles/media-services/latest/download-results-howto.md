---
title: 'Descarga de los resultados de un trabajo: Azure Media Services'
description: En este artículo se muestra cómo descargar los resultados de un trabajo.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: b3decdd4a2855382b27cb426d7fb030b46a2458c
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89298035"
---
# <a name="download-the-results-of-a-job"></a>Descarga de los resultados de un trabajo

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En Azure Media Services, cuando se procesan los vídeos (por ejemplo, para su codificación o análisis) debe crear un [recurso](assets-concept.md) de salida para almacenar el resultado del [trabajo](transforms-jobs-concept.md). Después, puede almacenar dichos resultados en una carpeta local mediante las API de Media Service y Storage. 

En este artículo se muestra cómo descargar los resultados mediante los SDK de Java y .NET.

## <a name="java"></a>Java

```java
/**
    * Use Media Service and Storage APIs to download the output files to a local folder
    * @param manager               The entry point of Azure Media resource management
    * @param resourceGroup         The name of the resource group within the Azure subscription
    * @param accountName           The Media Services account name
    * @param assetName             The asset name
    * @param outputFolder          The output folder for downloaded files.
    * @throws StorageException
    * @throws URISyntaxException
    * @throws IOException
    */
private static void downloadResults(MediaManager manager, String resourceGroup, String accountName,
        String assetName, File outputFolder) throws StorageException, URISyntaxException, IOException {
    ListContainerSasInput parameters = new ListContainerSasInput()
        .withPermissions(AssetContainerPermission.READ)
        .withExpiryTime(DateTime.now().plusHours(1));
    AssetContainerSas assetContainerSas = manager.assets()
        .listContainerSasAsync(resourceGroup, accountName, assetName, parameters).toBlocking().first();
    
    String strSas = assetContainerSas.assetContainerSasUrls().get(0);
    CloudBlobContainer container = new CloudBlobContainer(new URI(strSas));

    File directory = new File(outputFolder, assetName);
    directory.mkdir();

    ArrayList<ListBlobItem>  blobs = container.listBlobsSegmented(null, true, EnumSet.noneOf(BlobListingDetails.class), 200, null, null, null).getResults();

    for (ListBlobItem blobItem: blobs) {
        if (blobItem instanceof CloudBlockBlob) {
            CloudBlockBlob blob = (CloudBlockBlob)blobItem;
            File downloadTo = new File(directory, blob.getName());

            blob.downloadToFile(downloadTo.getPath());
        }
    }

    System.out.println("Download complete.");
}
```

Consulte el ejemplo de código completo: [EncodingWithMESPredefinedPreset](https://github.com/Azure-Samples/media-services-v3-java/blob/master/VideoEncoding/EncodingWithMESPredefinedPreset/src/main/java/sample/EncodingWithMESPredefinedPreset.java)

## <a name="net"></a>.NET

```csharp
/// <summary>
/// Use Media Service and Storage APIs to download the output files to a local folder
/// </summary>
/// <param name="client">The Media Services client.</param>
/// <param name="resourceGroupName">The name of the resource group within the Azure subscription.</param>
/// <param name="accountName">The Media Services account name.</param>
/// <param name="assetName">The asset name.</param>
/// <param name="resultsFolder">The output folder name for downloaded files.</param>
/// <returns>A task.</returns>
private async static Task DownloadResults(IAzureMediaServicesClient client, string resourceGroupName, string accountName, string assetName, string resultsFolder)
{
    AssetContainerSas assetContainerSas = client.Assets.ListContainerSas(
                    resourceGroupName, 
                    accountName, 
                    assetName,
                    permissions: AssetContainerPermission.Read, 
                    expiryTime: DateTime.UtcNow.AddHours(1).ToUniversalTime()
                    );

    Uri containerSasUrl = new Uri(assetContainerSas.AssetContainerSasUrls.FirstOrDefault());
    CloudBlobContainer container = new CloudBlobContainer(containerSasUrl);

    string directory = Path.Combine(resultsFolder, assetName);
    Directory.CreateDirectory(directory);

    Console.WriteLine("Downloading results to {0}.", directory);
    
    var blobs = container.ListBlobsSegmentedAsync(null,true, BlobListingDetails.None,200,null,null,null).Result;
    
    foreach (var blobItem in blobs.Results)
    {
        if (blobItem is CloudBlockBlob)
        {
            CloudBlockBlob blob = blobItem as CloudBlockBlob;
            string filename = Path.Combine(directory, blob.Name);

            await blob.DownloadToFileAsync(filename, FileMode.Create);
        }
    }

    Console.WriteLine("Download complete.");
}
```

Consulte el ejemplo de código completo: [EncodingWithMESPredefinedPreset](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/master/VideoEncoding/EncodingWithMESPredefinedPreset/Program.cs)

## <a name="next-steps"></a>Pasos siguientes

[Creación de una entrada de trabajo a partir de una dirección URL de HTTP(s)](job-input-from-http-how-to.md).
