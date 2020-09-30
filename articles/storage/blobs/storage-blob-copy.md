---
title: 'Copia de un blob con .NET: Azure Storage'
description: Obtenga información sobre cómo copiar un blob en la cuenta de Azure Storage mediante la biblioteca cliente .NET.
services: storage
author: mhopkins-msft
ms.author: mhopkins
ms.date: 09/01/2020
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: a7ca195bdfb05baff6100b3481903f9ca0841dc6
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89320671"
---
# <a name="copy-a-blob-with-net"></a>Copia de un blob con .NET

En este artículo se muestra cómo copiar un blob con una cuenta de Azure Storage. También se muestra cómo anular una operación de copia asincrónica. El código de ejemplo usa la [biblioteca cliente de Azure Storage para .NET](/dotnet/api/overview/azure/storage).

## <a name="about-copying-blobs"></a>Acerca de la copia de blobs

Cuando se copia un blob en la misma cuenta de almacenamiento, se trata de una operación sincrónica. Cuando la copia se realiza entre cuentas, se trata de una operación asincrónica.

El blob de origen para una operación de copia puede ser un blob en bloques, un blob en anexos, un blob en páginas o una instantánea. Si el blob de destino ya existe, debe ser del mismo tipo que el blob de origen. Si existe un blob de destino, se sobrescribirá.

No se puede modificar el blob de destino mientras haya una operación de copia en curso. Un blob de destino solo puede tener una operación de copia de blob pendiente. En otras palabras, un blob no puede ser el destino de varias operaciones de copia pendientes.

Siempre se copia el blob o el archivo de origen completo, ya que no se admite la copia de un intervalo de bytes o un conjunto de bloques.

Cuando se copia un blob, las propiedades del sistema al que pertenece se copian en el blob de destino con los mismos valores.

Una operación de copia puede tomar cualquiera de las siguientes formas:

- Puede copiar un blob de origen en un blob de destino con un nombre diferente. El blob de destino puede ser un blob existente del mismo tipo (en bloques, anexos o páginas), o puede ser un nuevo blob creado por la operación de copia.
- Puede copiar un blob de origen en un blob de destino con el mismo nombre. De este modo, se reemplazará de forma efectiva el blob de destino. Este tipo de operación de copia quita los bloques sin confirmar y sobrescribe los metadatos del blob de destino.
- Puede copiar un archivo de origen del servicio de Azure File en un blob de destino. El blob de destino puede ser un blob en bloques existente o un nuevo blob de ese tipo creado por la operación de copia. No se admite la copia de archivos a blobs en páginas o blobs en anexos.
- Puede copiar una instantánea sobre su blob base. Si se mueve una instantánea a la posición del blob, puede restaurar una versión anterior de un blob.
- Puede copiar una instantánea en un blob de destino con un nombre diferente. El blob resultante de destino es un blob en el que se puede escribir y no una instantánea.

## <a name="copy-a-blob"></a>Copia de un blob

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Para crear un blob, llame a uno de los métodos siguientes:

- [StartCopyFromUri](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.startcopyfromuri)
- [StartCopyFromUriAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.startcopyfromuriasync)

Los métodos **StartCopyFromUri** y **StartCopyFromUriAsync** devuelven un objeto [CopyFromUriOperation](/dotnet/api/azure.storage.blobs.models.copyfromurioperation) que contiene información sobre la operación de copia.

En el ejemplo de código siguiente se obtiene un objeto [BlobClient](/dotnet/api/azure.storage.blobs.blobclient) que representa un blob creado previamente y se copia en un nuevo blob del mismo contenedor:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CopyBlob.cs" id="Snippet_CopyBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Para crear un blob, llame a uno de los métodos siguientes:

- [StartCopy](/dotnet/api/microsoft.azure.storage.blob.cloudblob.startcopy)
- [StartCopyAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.startcopyasync)

Los métodos **StartCopy** y **StartCopyAsync** devuelven un valor de Id. de copia que se usa para comprobar el estado de la operación de copia o anularla.

En el ejemplo de código siguiente se obtiene una referencia a un blob creado previamente y se copia en un nuevo blob del mismo contenedor:

```csharp
private static async Task CopyBlockBlobAsync(CloudBlobContainer container)
{
    CloudBlockBlob sourceBlob = null;
    CloudBlockBlob destBlob = null;
    string leaseId = null;

    try
    {
        // Get a block blob from the container to use as the source.
        sourceBlob = container.ListBlobs().OfType<CloudBlockBlob>().FirstOrDefault();

        // Lease the source blob for the copy operation 
        // to prevent another client from modifying it.
        // Specifying null for the lease interval creates an infinite lease.
        leaseId = await sourceBlob.AcquireLeaseAsync(null);

        // Get a reference to a destination blob (in this case, a new blob).
        destBlob = container.GetBlockBlobReference("copy of " + sourceBlob.Name);

        // Ensure that the source blob exists.
        if (await sourceBlob.ExistsAsync())
        {
            // Get the ID of the copy operation.
            string copyId = await destBlob.StartCopyAsync(sourceBlob);

            // Fetch the destination blob's properties before checking the copy state.
            await destBlob.FetchAttributesAsync();

            Console.WriteLine("Status of copy operation: {0}", destBlob.CopyState.Status);
            Console.WriteLine("Completion time: {0}", destBlob.CopyState.CompletionTime);
            Console.WriteLine("Bytes copied: {0}", destBlob.CopyState.BytesCopied.ToString());
            Console.WriteLine("Total bytes: {0}", destBlob.CopyState.TotalBytes.ToString());
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
    finally
    {
        // Break the lease on the source blob.
        if (sourceBlob != null)
        {
            await sourceBlob.FetchAttributesAsync();

            if (sourceBlob.Properties.LeaseState != LeaseState.Available)
            {
                await sourceBlob.BreakLeaseAsync(new TimeSpan(0));
            }
        }
    }
}
```

---

## <a name="abort-a-blob-copy-operation"></a>Anulación de una operación de copia de blobs

Al anular una operación de copia, se obtiene un blob de destino con una longitud cero para los blobs en bloques, anexos y páginas. Sin embargo, los nuevos valores de los metadatos del blob de destino se copiarán del blob de origen o se establecerán explícitamente en la llamada a la operación de copia. Para conservar los metadatos originales anteriores a la copia, cree una instantánea del blob de destino antes de llamar a uno de los métodos de copia.

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Puede comprobar la propiedad [BlobProperties.CopyStatus](/dotnet/api/azure.storage.blobs.models.blobproperties.copystatus) en el blob de destino para obtener el estado de la operación de copia. El blob final se confirmará cuando se complete la copia.

Cuando se anula una operación de copia de blobs en curso, el estado de copia del blob se establece en [CopyState.Aborted](/dotnet/api/microsoft.azure.storage.blob.copystatus).

Los métodos [AbortCopyFromUri](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.abortcopyfromuri) y [AbortCopyFromUriAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.abortcopyfromuriasync) cancelan una operación de copia de blobs en curso.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CopyBlob.cs" id="Snippet_StopBlobCopy":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Puede comprobar la propiedad [CopyState.Status](/dotnet/api/microsoft.azure.storage.blob.copystate.status) en el blob de destino para obtener el estado de la operación de copia. El blob final se confirmará cuando se complete la copia.

Cuando se anula una operación de copia de blobs en curso, el estado de copia del blob se establece en [CopyState.Aborted](/dotnet/api/microsoft.azure.storage.blob.copystatus).

Los métodos [AbortCopy](/dotnet/api/microsoft.azure.storage.blob.cloudblob.abortcopy) y [AbortCopyAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.abortcopyasync) cancelan una operación de copia de blobs en curso.

```csharp
// Fetch the destination blob's properties before checking the copy state.
await destBlob.FetchAttributesAsync();

// Check the copy status. If it is still pending, abort the copy operation.
if (destBlob.CopyState.Status == CopyStatus.Pending)
{
    await destBlob.AbortCopyAsync(copyId);
    Console.WriteLine("Copy operation {0} has been aborted.", copyId);
}
```

---

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Los temas siguientes contienen información sobre la copia de blobs y la anulación de las operaciones de copia en curso mediante las API REST de Azure.

- [Copy Blob](/rest/api/storageservices/copy-blob)
- [Abort Copy Blob](/rest/api/storageservices/abort-copy-blob)
