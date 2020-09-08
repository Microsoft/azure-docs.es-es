---
title: 'Enumeración de contenedores de blob con .NET: Azure Storage'
description: Obtenga información sobre cómo enumerar los contenedores de blobs en la cuenta de Azure Storage mediante la biblioteca cliente .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 01/06/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: f443cd5603e6ca0f60dc0e69b734bfa46138d476
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018950"
---
# <a name="list-blob-containers-with-net"></a>Enumeración de contenedores de blob con .NET

Al enumerar los contenedores de una cuenta de Azure Storage desde el código, puede especificar una serie de opciones para administrar cómo se devuelven los resultados de Azure Storage. En este artículo se muestra cómo enumerar contenedores con [la biblioteca cliente de Azure Storage para .NET](/dotnet/api/overview/azure/storage?view=azure-dotnet).  

## <a name="understand-container-listing-options"></a>Descripción de las opciones de enumeración de contenedores

Para enumerar los contenedores de la cuenta de almacenamiento, llame a uno de los métodos siguientes:

- [ListContainersSegmented](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listcontainerssegmented)
- [ListContainersSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listcontainerssegmentedasync)

Las sobrecargas de estos métodos proporcionan opciones adicionales para administrar cómo devuelve los contenedores la operación de enumeración. Estas opciones se describen en las secciones siguientes.

### <a name="manage-how-many-results-are-returned"></a>Administración del número de resultados que se devuelven

De forma predeterminada, una operación de enumeración devuelve hasta 5000 resultados a la vez. Para devolver un conjunto de resultados más pequeño, proporcione un valor distinto de cero para el parámetro `maxresults` al llamar a uno de los métodos **ListContainerSegmented**.

Si la cuenta de almacenamiento contiene más de 5000 contenedores, o si ha especificado un valor para `maxresults` de forma que la operación de enumeración devuelva un subconjunto de los contenedores de la cuenta de almacenamiento, Azure Storage devuelve un *token de continuación* con la lista de contenedores. Un token de continuación es un valor opaco que se puede usar para recuperar el siguiente conjunto de resultados de Azure Storage.

En el código, compruebe el valor del token de continuación para determinar si es NULL. Cuando el token de continuación es NULL, el conjunto de resultados está completo. Si el token de continuación no es NULL, vuelva a llamar a **ListContainersSegmented** o **ListContainersSegmentedAsync**, y pase el token de continuación para recuperar el siguiente conjunto de resultados, hasta que el token de continuación sea NULL.

### <a name="filter-results-with-a-prefix"></a>Filtrado de los resultados con un prefijo

Para filtrar la lista de contenedores, especifique una cadena para el parámetro `prefix`. La cadena de prefijo puede incluir uno o varios caracteres. Después, Azure Storage solo devuelve los contenedores cuyos nombres empiecen por ese prefijo.

### <a name="return-metadata"></a>Devolución de metadatos

Para devolver metadatos del contenedor con los resultados, especifique el valor **Metadata** con la enumeración [ContainerListingDetails](/dotnet/api/microsoft.azure.storage.blob.containerlistingdetails). Azure Storage incluye metadatos con cada contenedor que se devuelve, por lo que no es necesario llamar a uno de los métodos **FetchAttributes** para recuperar los metadatos del contenedor.

## <a name="example-list-containers"></a>Ejemplo: Enumerar contenedores

En el ejemplo siguiente se enumeran de forma asincrónica los contenedores de una cuenta de almacenamiento que comienzan con un prefijo especificado. En el ejemplo se enumeran los contenedores en incrementos de cinco resultados a la vez, y se usa el token de continuación para obtener el siguiente segmento de resultados. En el ejemplo también se devuelven metadatos de contenedor con los resultados.

```csharp
private static async Task ListContainersWithPrefixAsync(CloudBlobClient blobClient,
                                                        string prefix)
{
    Console.WriteLine("List all containers beginning with prefix {0}, plus container metadata:", prefix);

    try
    {
        ContainerResultSegment resultSegment = null;
        BlobContinuationToken continuationToken = null;

        do
        {
            // List containers beginning with the specified prefix, returning segments of 5 results each.
            // Passing null for the maxResults parameter returns the max number of results (up to 5000).
            // Requesting the container's metadata with the listing operation populates the metadata,
            // so it's not necessary to also call FetchAttributes() to read the metadata.
            resultSegment = await blobClient.ListContainersSegmentedAsync(
                prefix, ContainerListingDetails.Metadata, 5, continuationToken, null, null);

            // Enumerate the containers returned.
            foreach (var container in resultSegment.Results)
            {
                Console.WriteLine("\tContainer:" + container.Name);

                // Write the container's metadata keys and values.
                foreach (var metadataItem in container.Metadata)
                {
                    Console.WriteLine("\t\tMetadata key: " + metadataItem.Key);
                    Console.WriteLine("\t\tMetadata value: " + metadataItem.Value);
                }
            }

            // Get the continuation token. If not null, get the next segment.
            continuationToken = resultSegment.ContinuationToken;

        } while (continuationToken != null);
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0} : {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
    }
}
```

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="see-also"></a>Consulte también

[List Containers](/rest/api/storageservices/list-containers2)
[Enumeración de recursos de blob](/rest/api/storageservices/enumerating-blob-resources)
