---
title: Establecimiento y recuperación de propiedades y metadatos de objetos en Azure Storage | Microsoft Docs
description: Almacenamiento de metadatos personalizados en los objetos de Azure Storage y establecimiento y recuperación de propiedades del sistema.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/03/2019
ms.author: tamram
ms.openlocfilehash: e8a85319a12f04a11e3914716d9ff84cdb6de8d8
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787874"
---
# <a name="set-and-retrieve-properties-and-metadata"></a>Establecimiento y recuperación de propiedades y metadatos

Objetos en las propiedades de compatibilidad de sistema de Azure Storage y metadatos definidos por el usuario, además de los datos que contienen. En este artículo se describen las propiedades del sistema de administración y los metadatos definidos por el usuario con la [biblioteca de cliente de Azure Storage para .NET](/dotnet/api/overview/azure/storage/client).

* **Propiedades del sistema**:  En cada recurso de almacenamiento existen propiedades del sistema. Algunas se pueden leer o establecer, mientras que otras son de solo lectura. En segundo plano, algunas propiedades del sistema corresponden a ciertos encabezados HTTP estándar. Las bibliotecas cliente de Azure Storage mantienen estas propiedades por usted.

* **Los metadatos definidos por el usuario**: Los metadatos definidos por el usuario constan de uno o más pares de nombre-valor que especifique para un recurso de almacenamiento de Azure. Puede usar metadatos para almacenar valores adicionales con un recurso. Los valores de metadatos se proporcionan para uso personal y no afectan a cómo se comporta el recurso.

El proceso de recuperación de los valores de propiedad y metadatos de un recurso de almacenamiento consta de dos pasos. Para poder leer estos valores, tiene que capturarlos explícitamente llamando al método **FetchAttributes** o **FetchAttributesAsync**. La excepción es si llama al método **Exists** o **ExistsAsync** en un recurso. Cuando se llama a uno de estos métodos, Azure Storage llama al método **FetchAttributes** adecuado en segundo plano como parte de la llamada al método **Exists**.

> [!IMPORTANT]
> Si descubre que no se han rellenado los valores de propiedad ni metadatos para un recurso de almacenamiento, compruebe que su código llama al método **FetchAttributes** o **FetchAttributesAsync**.
>
> Pares de nombre/valor de metadatos son encabezados HTTP válidos y, por lo que deben cumplir todas las restricciones que rigen los encabezados HTTP. Los nombres de los metadatos deben ser nombres válidos de encabezado HTTP válido y C# identificadores, puede contener únicamente caracteres ASCII y debe tratarse como mayúsculas y minúsculas. Los valores de metadatos que contiene caracteres no ASCII deben estar codificados en Base64 o codificados de dirección URL.

## <a name="setting-and-retrieving-properties"></a>Establecimiento y recuperación de propiedades
Para recuperar valores de propiedad, llame al método **FetchAttributesAsync** en el blob o contenedor para rellenar las propiedades y, a continuación, lea los valores.

Para establecer propiedades en un objeto, especifique el valor de propiedad y, a continuación, llame al método **SetProperties** .

En el ejemplo de código siguiente se crea un contenedor y se escriben algunos de sus valores de propiedad en una ventana de la consola.

```csharp
//Parse the connection string for the storage account.
const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key";
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

//Create the service client object for credentialed access to the Blob service.
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

// Retrieve a reference to a container.
CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

// Create the container if it does not already exist.
container.CreateIfNotExists();

// Fetch container properties and write out their values.
await container.FetchAttributesAsync();
Console.WriteLine("Properties for container {0}", container.StorageUri.PrimaryUri.ToString());
Console.WriteLine("LastModifiedUTC: {0}", container.Properties.LastModified.ToString());
Console.WriteLine("ETag: {0}", container.Properties.ETag);
Console.WriteLine();
```

## <a name="setting-and-retrieving-metadata"></a>Establecer y recuperar metadatos
Puede especificar metadatos como uno o más pares nombre-valor en un recurso de blob o contenedor. Para establecer los metadatos, agregue pares nombre-valor a la colección **Metadata** del recurso y, después, llame al método **SetMetadata** o **SetMetadataAsync** para guardar los valores en el servicio.

> [!NOTE]
> El nombre de los metadatos debe cumplir las convenciones de nomenclatura para los identificadores de C#.
>
>

El ejemplo de código siguiente establece los metadatos en un contenedor. Un valor se establece mediante el método **Agregar** de la colección. El otro valor se establece mediante la sintaxis implícita de clave/valor. Ambos son válidos.

```csharp
public static async Task AddContainerMetadataAsync(CloudBlobContainer container)
{
    // Add some metadata to the container.
    container.Metadata.Add("docType", "textDocuments");
    container.Metadata["category"] = "guidance";

    // Set the container's metadata.
    await container.SetMetadataAsync();
}
```

Para recuperar metadatos, llame al método **FetchAttributes** o **FetchAttributesAsync** en el blob o contenedor para rellenar la colección **Metadata** y después lea los valores tal como se muestra en el ejemplo siguiente.

```csharp
public static async Task ListContainerMetadataAsync(CloudBlobContainer container)
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
```

## <a name="next-steps"></a>Pasos siguientes
* [Biblioteca de cliente de Azure Storage para referencia de .NET](/dotnet/api/?term=Microsoft.Azure.Storage)
* [Biblioteca de cliente de almacenamiento de Azure Blob para el paquete .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/)
* [Biblioteca de cliente de almacenamiento de Azure cola para el paquete .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Queue/)
* [Biblioteca de cliente de almacenamiento de Azure archivo para el paquete .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.File/)

