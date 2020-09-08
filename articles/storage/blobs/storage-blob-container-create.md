---
title: 'Creación o eliminación de un contenedor de blobs en .NET: Azure Storage'
description: Obtenga información sobre cómo crear o eliminar un contenedor de blobs en la cuenta de Azure Storage mediante la biblioteca cliente .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/22/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: a17ab97dbfa1819154695f4c287b59db90f34334
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019001"
---
# <a name="create-or-delete-a-container-in-azure-storage-with-net"></a>Creación o eliminación de un contenedor en Azure Storage con .NET

Los blobs de Azure Storage se organizan en contenedores. Antes de poder cargar un blob, primero debe crear un contenedor. En este artículo se muestra cómo crear y eliminar contenedores con [la biblioteca cliente de Azure Storage para .NET](/dotnet/api/overview/azure/storage?view=azure-dotnet).

## <a name="name-a-container"></a>Asignación de un nombre para un contenedor

Un nombre de contenedor debe ser un nombre DNS válido, ya que forma parte del URI único que se usa para direccionar el contenedor o sus blobs. Siga estas reglas al asignar un nombre a un contenedor:

- Los nombres de contenedor pueden tener entre 3 y 63 caracteres.
- Los nombres de contenedor deben comenzar por una letra o un número, y solo pueden contener letras en minúscula, números y el carácter de guión (-).
- En los nombres de contenedor no se permiten dos o más guiones consecutivos.

El URI de un contenedor tiene este formato:

`https://myaccount.blob.core.windows.net/mycontainer`

## <a name="create-a-container"></a>Crear un contenedor

Para crear un contenedor, llame a uno de los métodos siguientes:

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

- [Crear](/dotnet/api/azure.storage.blobs.blobcontainerclient.create)
- [CreateAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.createasync)
- [CreateIfNotExists](/dotnet/api/azure.storage.blobs.blobcontainerclient.createifnotexists)
- [CreateIfNotExistsAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.createifnotexistsasync)

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

- [Crear](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.create)
- [CreateAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createasync)
- [CreateIfNotExists](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createifnotexists)
- [CreateIfNotExistsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createifnotexistsasync)
---

Los métodos **Create** y **CreateAsync** inician una excepción si ya existe un contenedor con el mismo nombre.

Los métodos **CreateIfNotExists** y **CreateIfNotExistsAsync** devuelven un valor booleano que indica si se ha creado el contenedor. Si ya existe un contenedor con el mismo nombre, estos métodos devuelven **False** para indicar que no se ha creado un contenedor nuevo.

Los contenedores se crean inmediatamente debajo de la cuenta de almacenamiento. No es posible anidar un contenedor debajo de otro.

En el ejemplo siguiente se crea un contenedor de forma asincrónica:

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Containers.cs" id="CreateSampleContainerAsync":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
private static async Task<CloudBlobContainer> CreateSampleContainerAsync(CloudBlobClient blobClient)
{
    // Name the sample container based on new GUID, to ensure uniqueness.
    // The container name must be lowercase.
    string containerName = "container-" + Guid.NewGuid();

    // Get a reference to a sample container.
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);

    try
    {
        // Create the container if it does not already exist.
        bool result = await container.CreateIfNotExistsAsync();
        if (result == true)
        {
            Console.WriteLine("Created container {0}", container.Name);
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
    }

    return container;
}
```
---

## <a name="create-the-root-container"></a>Creación del contenedor raíz

Un contenedor raíz sirve como contenedor predeterminado para la cuenta de almacenamiento. Cada cuenta de almacenamiento puede tener un contenedor raíz, que se debe denominar *$root*. El contenedor raíz debe crearse o eliminarse explícitamente.

Puede hacer referencia a un blob almacenado en el contenedor raíz sin incluir el nombre del contenedor raíz. El contenedor raíz permite hacer referencia a un blob en el nivel superior de la jerarquía de la cuenta de almacenamiento. Por ejemplo, se puede hacer referencia a un blob que esté en el contenedor raíz de la siguiente manera:

`https://myaccount.blob.core.windows.net/default.html`

En el ejemplo siguiente se crea el contenedor raíz de forma sincrónica:

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Containers.cs" id="CreateRootContainer":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
private static void CreateRootContainer(CloudBlobClient blobClient)
{
    try
    {
        // Create the root container if it does not already exist.
        CloudBlobContainer container = blobClient.GetContainerReference("$root");
        if (container.CreateIfNotExists())
        {
            Console.WriteLine("Created root container.");
        }
        else
        {
            Console.WriteLine("Root container already exists.");
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
    }
}
```
---

## <a name="delete-a-container"></a>Eliminación de un contenedor

Para eliminar un contenedor de .NET, use uno de los métodos siguientes:

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

- [Eliminar](/dotnet/api/azure.storage.blobs.blobcontainerclient.delete)
- [DeleteAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.deleteasync)
- [DeleteIfExists](/dotnet/api/azure.storage.blobs.blobcontainerclient.deleteifexists)
- [DeleteIfExistsAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.deleteifexistsasync)

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

- [Eliminar](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.delete)
- [DeleteAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteasync)
- [DeleteIfExists](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteifexists)
- [DeleteIfExistsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteifexistsasync)
---

Los métodos **Delete** y **DeleteAsync** inician una excepción si el contenedor no existe.

Los métodos **DeleteIfExists** y **DeleteIfExistsAsync** devuelven un valor booleano que indica si se ha eliminado el contenedor. Si el contenedor especificado no existe, estos métodos devuelven **False** para indicar que el contenedor no se ha eliminado.

Después de eliminar un contenedor, no se puede crear otro con el mismo nombre durante *al menos* 30 segundos. Un intento de crear un contenedor con el mismo nombre producirá un error con el código de error de HTTP 409 (conflicto). Cualquier otra operación en el contenedor o en los blobs que contiene producirá un error con el código de error HTTP 404 (no encontrado).

En el ejemplo siguiente se elimina el contenedor especificado y se controla la excepción si el contenedor no existe:

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Containers.cs" id="DeleteSampleContainerAsync":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
private static async Task DeleteSampleContainerAsync(CloudBlobClient blobClient, string containerName)
{
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);

    try
    {
        // Delete the specified container and handle the exception.
        await container.DeleteAsync();
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

En el ejemplo siguiente se muestra cómo eliminar todos los contenedores que comienzan con un prefijo especificado.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Containers.cs" id="DeleteContainersWithPrefixAsync":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
private static async Task DeleteContainersWithPrefixAsync(CloudBlobClient blobClient, string prefix)
{
    Console.WriteLine("Delete all containers beginning with the specified prefix");
    try
    {
        foreach (var container in blobClient.ListContainers(prefix))
        {
            Console.WriteLine("\tContainer:" + container.Name);
            await container.DeleteAsync();
        }

        Console.WriteLine();
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

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="see-also"></a>Consulte también

- [Operación de creación de contenedor](/rest/api/storageservices/create-container)
- [Operación Eliminar contenedor](/rest/api/storageservices/delete-container)
