---
title: Acceso anónimo a contenedores y blobs públicos con .NET
titleSuffix: Azure Storage
description: Use la biblioteca cliente de Azure Storage para .NET para acceder a contenedores y blobs públicos de forma anónima.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 08/02/2020
ms.author: tamram
ms.reviewer: fryu
ms.custom: devx-track-csharp
ms.openlocfilehash: 1ba36c7128f72f5181f3fed53f5615ab464e3325
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89013544"
---
# <a name="access-public-containers-and-blobs-anonymously-with-net"></a>Acceso anónimo a contenedores y blobs públicos con .NET

Azure Storage admite el acceso de lectura público opcional a contenedores y blobs. Los clientes pueden acceder de forma anónima a contenedores y blobs públicos mediante el uso de las bibliotecas cliente de Azure Storage, así como de otras herramientas y utilidades que admiten el acceso a datos en Azure Storage.

En este artículo se muestra cómo obtener acceso a un contenedor público o a un blob desde .NET. Para obtener información sobre cómo configurar el acceso de lectura anónimo en un contenedor, vea [Configuración de acceso de lectura público anónimo a contenedores y blobs](anonymous-read-access-configure.md). Para obtener información sobre cómo impedir el acceso anónimo a una cuenta de almacenamiento, vea [Impedir el acceso de lectura público anónimo a contenedores y blobs](anonymous-read-access-prevent.md).

Un cliente que tiene acceso a contenedores y blobs de forma anónima puede utilizar constructores que no requieren credenciales. En los ejemplos siguientes se muestran varias maneras diferentes de hacer referencia a contenedores y blobs de forma anónima.

## <a name="create-an-anonymous-client-object"></a>Creación de un objeto de cliente anónimo

Puede crear un nuevo objeto de cliente de servicio para el acceso anónimo proporcionando el punto de conexión de almacenamiento de Blob para la cuenta. Sin embargo, también debe conocer el nombre de un contenedor en esa cuenta que esté disponible para el acceso anónimo.

# <a name="net-v12-sdk"></a>[\.SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Security.cs" id="Snippet_CreateAnonymousBlobClient":::

# <a name="net-v11-sdk"></a>[\.SDK de .NET, versión 11](#tab/dotnet11)

```csharp
public static void CreateAnonymousBlobClient()
{
    // Create the client object using the Blob storage endpoint for your account.
    CloudBlobClient blobClient = new CloudBlobClient(
        new Uri(@"https://storagesamples.blob.core.windows.net"));

    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");

    // Read the container's properties. 
    // Note this is only possible when the container supports full public read access.
    container.FetchAttributes();
    Console.WriteLine(container.Properties.LastModified);
    Console.WriteLine(container.Properties.ETag);
}
```

---

## <a name="reference-a-container-anonymously"></a>Referencia a un contenedor de forma anónima

Si tiene la dirección URL de un contenedor que está disponible de forma anónima, puede utilizarla para hacer referencia al contenedor directamente.

# <a name="net-v12-sdk"></a>[\.SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Security.cs" id="Snippet_ListBlobsAnonymously":::

# <a name="net-v11-sdk"></a>[\.SDK de .NET, versión 11](#tab/dotnet11)

```csharp
public static void ListBlobsAnonymously()
{
    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = new CloudBlobContainer(
        new Uri(@"https://storagesamples.blob.core.windows.net/sample-container"));

    // List blobs in the container.
    // Note this is only possible when the container supports full public read access.
    foreach (IListBlobItem blobItem in container.ListBlobs())
    {
        Console.WriteLine(blobItem.Uri);
    }
}
```

---

## <a name="reference-a-blob-anonymously"></a>Referencia a un blob de forma anónima

Si tiene la dirección URL a un blob que está disponible para el acceso anónimo, puede hacer referencia al blob directamente con esa dirección URL:

# <a name="net-v12-sdk"></a>[\.SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Security.cs" id="Snippet_DownloadBlobAnonymously":::

# <a name="net-v11-sdk"></a>[\.SDK de .NET, versión 11](#tab/dotnet11)

```csharp
public static void DownloadBlobAnonymously()
{
    CloudBlockBlob blob = new CloudBlockBlob(
        new Uri(@"https://storagesamples.blob.core.windows.net/sample-container/logfile.txt"));
    blob.DownloadToFile(@"C:\Temp\logfile.txt", FileMode.Create);
}
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de acceso de lectura público anónimo a contenedores y blobs](anonymous-read-access-configure.md)
- [Impedir el acceso de lectura público anónimo a contenedores y blobs](anonymous-read-access-prevent.md)
- [Autorización del acceso a Azure Storage](../common/storage-auth.md)
