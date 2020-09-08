---
title: Uso de .NET para administrar propiedades y metadatos de un contenedor de blobs
titleSuffix: Azure Storage
description: Aprenda a establecer y recuperar propiedades del sistema y a almacenar metadatos personalizados de contenedores de blobs en su cuenta de Azure Storage mediante la biblioteca cliente de .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/01/2020
ms.author: tamram
ms.custom: devx-track-csharp
ms.openlocfilehash: 115ab9767c23afd5876eb23821bd1283777cdb5e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018933"
---
# <a name="manage-container-properties-and-metadata-with-net"></a>Administración de propiedades y metadatos de contenedor con .NET

Los contenedores de blobs admiten propiedades del sistema y metadatos definidos por el usuario, además de los datos que contienen. En este artículo se muestra cómo administrar propiedades del sistema y metadatos definidos por el usuario con la [biblioteca cliente de Azure Storage para .NET](/dotnet/api/overview/azure/storage?view=azure-dotnet).

## <a name="about-properties-and-metadata"></a>Acerca de las propiedades y los metadatos

- **Propiedades del sistema**: en cada recurso de almacenamiento de blobs existen propiedades del sistema. Algunas se pueden leer o establecer, mientras que otras son de solo lectura. En segundo plano, algunas propiedades del sistema corresponden a ciertos encabezados HTTP estándar. La biblioteca cliente de Azure Storage para .NET mantiene estas propiedades automáticamente.

- **Metadatos definidos por el usuario**: los metadatos definidos por el usuario se componen de uno o más pares nombre-valor que especifica para un recurso de almacenamiento de blobs. Puede usar metadatos para almacenar valores adicionales con el recurso. Los valores de metadatos se proporcionan para uso personal y no afectan a cómo se comporta el recurso.

Los pares de nombre/valor de metadatos son encabezados HTTP válidos y, por tanto, deben cumplir todas las restricciones que controlan los encabezados HTTP. Los nombres de los metadatos deben ser nombres válidos de encabezado HTTP e identificadores de C# válidos, y solo pueden contener caracteres ASCII y no distinguir entre mayúsculas y minúsculas. Los valores de los metadatos que contengan caracteres distintos de ASCII deben estar codificados en Base64 o con direcciones URL.

## <a name="retrieve-container-properties"></a>Recuperación de las propiedades del contenedor

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Para recuperar las propiedades del contenedor, llame a uno de los métodos siguientes:

- [GetProperties](/dotnet/api/azure.storage.blobs.blobcontainerclient.getproperties)
- [GetPropertiesAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getpropertiesasync)

En el ejemplo de código siguiente se capturan las propiedades del sistema de un contenedor y se escriben algunos valores de propiedad en una ventana de la consola:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Metadata.cs" id="Snippet_ReadContainerProperties":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

El proceso de recuperación de los valores de propiedad y metadatos de un recurso de almacenamiento de blobs consta de dos pasos. Para poder leer estos valores, tiene que capturarlos explícitamente llamando al método **FetchAttributes** o **FetchAttributesAsync**. La excepción a esta regla es que los métodos **Exists** y **ExistsAsync** llaman al método **FetchAttributes** en segundo plano. Cuando se llama a uno de estos métodos, no es necesario llamar también a **FetchAttributes**.

> [!IMPORTANT]
> Si descubre que no se han rellenado los valores de propiedad ni metadatos para un recurso de almacenamiento, compruebe que su código llama al método **FetchAttributes** o **FetchAttributesAsync**.

Para recuperar las propiedades del contenedor, llame a uno de los métodos siguientes:

- [FetchAttributes](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.fetchattributes)
- [FetchAttributesAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.fetchattributesasync)

En el ejemplo de código siguiente se capturan las propiedades del sistema de un contenedor y se escriben algunos valores de propiedad en una ventana de la consola:

```csharp
private static async Task ReadContainerPropertiesAsync(CloudBlobContainer container)
{
    try
    {
        // Fetch some container properties and write out their values.
        await container.FetchAttributesAsync();
        Console.WriteLine("Properties for container {0}", container.StorageUri.PrimaryUri);
        Console.WriteLine("Public access level: {0}", container.Properties.PublicAccess);
        Console.WriteLine("Last modified time in UTC: {0}", container.Properties.LastModified);
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

---

## <a name="set-and-retrieve-metadata"></a>Establecimiento y recuperación de metadatos

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Puede especificar metadatos como uno o más pares nombre-valor en un recurso de blob o contenedor. Para establecer los metadatos, agregue pares nombre-valor a un objeto [IDictionary](/dotnet/api/system.collections.idictionary) del recurso y, luego, llame a uno de los métodos siguientes para escribir los valores:

- [SetMetadata](/dotnet/api/azure.storage.blobs.blobcontainerclient.setmetadata)
- [SetMetadataAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.setmetadataasync)

El nombre de los metadatos debe cumplir las convenciones de nomenclatura para los identificadores de C#. Los nombres de los metadatos conservan las mayúsculas y minúsculas con las que se crearon, pero no las distinguen cuando se establecen o se leen. Si se envían dos o más encabezados de metadatos con el mismo nombre para un recurso, Blob Storage separa los dos valores por comas, los concatena y devuelve el código de respuesta HTTP 200 (correcto).

El ejemplo de código siguiente establece los metadatos en un contenedor.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Metadata.cs" id="Snippet_AddContainerMetadata":::

Para recuperar metadatos, invoque uno de los métodos siguientes:

- [GetProperties](/dotnet/api/azure.storage.blobs.blobcontainerclient.getproperties)
- [GetPropertiesAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getpropertiesasync).

A continuación, copie los valores como se muestra en el ejemplo siguiente.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Metadata.cs" id="Snippet_ReadContainerMetadata":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Puede especificar metadatos como uno o más pares nombre-valor en un recurso de blob o contenedor. Para establecer los metadatos, agregue pares nombre-valor a la colección **Metadatos** del recurso y, luego, llame a uno de los métodos siguientes para escribir los valores:

- [SetMetadata](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.setmetadata)
- [SetMetadataAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.setmetadataasync)

El nombre de los metadatos debe cumplir las convenciones de nomenclatura para los identificadores de C#. Los nombres de los metadatos conservan las mayúsculas y minúsculas con las que se crearon, pero no las distinguen cuando se establecen o se leen. Si se envían dos o más encabezados de metadatos con el mismo nombre para un recurso, Blob Storage separa los dos valores por comas, los concatena y devuelve el código de respuesta HTTP 200 (correcto).

El ejemplo de código siguiente establece los metadatos en un contenedor. Un valor se establece mediante el método **Agregar** de la colección. El otro valor se establece mediante la sintaxis implícita de clave/valor. Ambos son válidos.

```csharp
public static async Task AddContainerMetadataAsync(CloudBlobContainer container)
{
    try
    {
        // Add some metadata to the container.
        container.Metadata.Add("docType", "textDocuments");
        container.Metadata["category"] = "guidance";

        // Set the container's metadata.
        await container.SetMetadataAsync();
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

Para recuperar metadatos, llame al método **FetchAttributes** o **FetchAttributesAsync** en el blob o contenedor para rellenar la colección **Metadata** y después lea los valores tal como se muestra en el ejemplo siguiente.

```csharp
public static async Task ReadContainerMetadataAsync(CloudBlobContainer container)
{
    try
    {
        // Fetch container attributes in order to populate the container's properties and metadata.
        await container.FetchAttributesAsync();

        // Enumerate the container's metadata.
        Console.WriteLine("Container metadata:");
        foreach (var metadataItem in container.Metadata)
        {
            Console.WriteLine("\tKey: {0}", metadataItem.Key);
            Console.WriteLine("\tValue: {0}", metadataItem.Value);
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

---

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="see-also"></a>Consulte también

- [Operación Get Container Properties](/rest/api/storageservices/get-container-properties)
- [Operación Set Container Metadata](/rest/api/storageservices/set-container-metadata)
- [Operación Get Container Metadata](/rest/api/storageservices/get-container-metadata)
