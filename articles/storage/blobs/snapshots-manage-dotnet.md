---
title: Creación y administración de una instantánea de blob en .NET
titleSuffix: Azure Storage
description: Aprenda a usar la biblioteca cliente de .NET para crear una instantánea de solo lectura de un blob para hacer una copia de seguridad de los datos de blob en un momento determinado.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 08/27/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: fb57e7260a056afe023455149f610ff492c98c12
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226138"
---
# <a name="create-and-manage-a-blob-snapshot-in-net"></a>Creación y administración de una instantánea de blob en .NET

Una instantánea es una versión de solo lectura de un blob que se ha realizado en un momento dado. En este artículo se muestra cómo crear y administrar instantáneas de blob mediante la [biblioteca cliente para .NET de Azure Storage](/dotnet/api/overview/azure/storage?view=azure-dotnet).

Para obtener más información sobre las instantáneas de blob en Azure Storage, consulte [Instantáneas de blobs](snapshots-overview.md).

## <a name="create-a-snapshot"></a>Crear una instantánea

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Para crear una instantánea de un blob en bloques con la versión 12.x de la biblioteca de cliente de Azure Storage para .NET, use uno de los métodos siguientes:

- [CreateSnapshot](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.createsnapshot)
- [CreateSnapshotAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.createsnapshotasync)

En el ejemplo de código siguiente se muestra cómo crear una instantánea con la versión 12.x. Incluya una referencia a la biblioteca [Azure.Identity](https://www.nuget.org/packages/azure.identity) para usar sus credenciales de Azure AD para autorizar solicitudes al servicio.

```csharp
private static async Task CreateBlockBlobSnapshot(string accountName, string containerName, string blobName, Stream data)
{
    const string blobServiceEndpointSuffix = ".blob.core.windows.net";
    Uri containerUri = new Uri("https://" + accountName + blobServiceEndpointSuffix + "/" + containerName);

    // Get a container client object and create the container.
    BlobContainerClient containerClient = new BlobContainerClient(containerUri,
        new DefaultAzureCredential());
    await containerClient.CreateIfNotExistsAsync();

    // Get a blob client object.
    BlobClient blobClient = containerClient.GetBlobClient(blobName);

    try
    {
        // Upload text to create a block blob.
        await blobClient.UploadAsync(data);

        // Add blob metadata.
        IDictionary<string, string> metadata = new Dictionary<string, string>
        {
            { "ApproxBlobCreatedDate", DateTime.UtcNow.ToString() },
            { "FileType", "text" }
        };
        await blobClient.SetMetadataAsync(metadata);

        // Sleep 5 seconds.
        System.Threading.Thread.Sleep(5000);

        // Create a snapshot of the base blob.
        // You can specify metadata at the time that the snapshot is created.
        // If no metadata is specified, then the blob's metadata is copied to the snapshot.
        await blobClient.CreateSnapshotAsync();
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Para crear una instantánea de un blob en bloques con la versión 11.x de la biblioteca de cliente de Azure Storage para .NET, use uno de los métodos siguientes:

- [CreateSnapshot](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.createsnapshot)
- [CreateSnapshotAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.createsnapshotasync)

En el ejemplo de código siguiente se muestra cómo crear una instantánea con la versión 11.x. En este ejemplo se especifican metadatos adicionales para la instantánea cuando se crea.

```csharp
private static async Task CreateBlockBlobSnapshot(CloudBlobContainer container)
{
    // Create a new block blob in the container.
    CloudBlockBlob baseBlob = container.GetBlockBlobReference("sample-base-blob.txt");

    // Add blob metadata.
    baseBlob.Metadata.Add("ApproxBlobCreatedDate", DateTime.UtcNow.ToString());

    try
    {
        // Upload the blob to create it, with its metadata.
        await baseBlob.UploadTextAsync(string.Format("Base blob: {0}", baseBlob.Uri.ToString()));

        // Sleep 5 seconds.
        System.Threading.Thread.Sleep(5000);

        // Create a snapshot of the base blob.
        // You can specify metadata at the time that the snapshot is created.
        // If no metadata is specified, then the blob's metadata is copied to the snapshot.
        Dictionary<string, string> metadata = new Dictionary<string, string>();
        metadata.Add("ApproxSnapshotCreatedDate", DateTime.UtcNow.ToString());
        await baseBlob.CreateSnapshotAsync(metadata, null, null, null);
        Console.WriteLine(snapshot.SnapshotQualifiedStorageUri.PrimaryUri);
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

---

## <a name="delete-snapshots"></a>Eliminar instantáneas

Para eliminar un blob, primero debe eliminar todas las instantáneas de ese blob. Puede eliminar una instantánea individualmente o especificar que se eliminen todas las instantáneas cuando se elimina el blob de origen. Si intenta eliminar un blob que todavía tiene instantáneas, se producirá un error.

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Para eliminar un blob y sus instantáneas con la versión 12.x de la biblioteca de cliente de Azure Storage para .NET, use uno de los métodos siguientes e incluya la enumeración [DeleteSnapshotsOption](/dotnet/api/azure.storage.blobs.models.deletesnapshotsoption):

- [Eliminar](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.delete)
- [DeleteAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.deleteasync)
- [DeleteIfExists](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.deleteifexists)
- [DeleteIfExistsAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.deleteifexistsasync)

En el ejemplo de código siguiente se muestra cómo eliminar un blob y sus instantáneas en. NET, donde `blobClient` es un objeto del tipo [BlobClient](/dotnet/api/azure.storage.blobs.blobclient):

```csharp
await blobClient.DeleteIfExistsAsync(DeleteSnapshotsOption.IncludeSnapshots, null, default);
```

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Para eliminar un blob y sus instantáneas con la versión 11.x de la biblioteca de cliente de Azure Storage para .NET, use uno de los métodos de eliminación de blobs siguientes e incluya la enumeración [DeleteSnapshotsOption](/dotnet/api/microsoft.azure.storage.blob.deletesnapshotsoption):

- [Eliminar](/dotnet/api/microsoft.azure.storage.blob.cloudblob.delete)
- [DeleteAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.deleteasync)
- [DeleteIfExists](/dotnet/api/microsoft.azure.storage.blob.cloudblob.deleteifexists)
- [DeleteIfExistsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.deleteifexistsasync)

En el ejemplo de código siguiente se muestra cómo eliminar un blob y sus instantáneas en. NET, donde `blockBlob` es un objeto del tipo [CloudBlockBlob][dotnet_CloudBlockBlob]:

```csharp
await blockBlob.DeleteIfExistsAsync(DeleteSnapshotsOption.IncludeSnapshots, null, null, null);
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Instantáneas de blob](snapshots-overview.md)
- [Versiones de blob](versioning-overview.md)
- [Eliminación temporal para blobs](storage-blob-soft-delete.md)