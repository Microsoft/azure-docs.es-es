---
title: Habilitación y administración del control de versiones de blob (versión preliminar)
titleSuffix: Azure Storage
description: Aprenda a habilitar el control de versiones de blob (versión preliminar) desde Azure Portal o con una plantilla de Azure Resource Manager.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 08/10/2020
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: 85e8ccd03bd20ed9bb572d482dbc7a06b8af725c
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067280"
---
# <a name="enable-and-manage-blob-versioning-preview"></a>Habilitación y administración del control de versiones de blob (versión preliminar)

Puede habilitar el control de versiones de Blob Storage (versión preliminar) para conservar automáticamente las versiones anteriores de un objeto.  Cuando el control de versiones de blobs está habilitado, puede restaurar una versión anterior de un blob para recuperar los datos si se modifican o eliminan por error.

En este artículo se muestra cómo habilitar o deshabilitar el control de versiones de blobs para la cuenta de almacenamiento mediante Azure Portal o una plantilla de Azure Resource Manager.

Antes de habilitar el control de versiones de blobs, debe registrarse en la versión preliminar. Para más información sobre el control de versiones de blobs, incluida la forma de registrarse en la versión preliminar, consulte [Control de versiones de blobs (versión preliminar)](versioning-overview.md).

## <a name="enable-blob-versioning"></a>Habilitación del control de versiones de blob

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para habilitar el control de versiones de blobs en Azure Portal:

1. Vaya a la cuenta de almacenamiento desde el portal.
1. En **Blob service**, elija **Protección de datos**.
1. En la sección **Control de versiones**, seleccione **Habilitado**.

:::image type="content" source="media/versioning-enable/portal-enable-versioning.png" alt-text="Captura de pantalla que muestra cómo habilitar el control de versiones de blobs en Azure Portal":::

# <a name="template"></a>[Plantilla](#tab/template)

Para habilitar el control de versiones de blobs con una plantilla, cree una plantilla con la propiedad **IsVersioningEnabled** definida como **true**. En los siguientes pasos se muestra cómo crear una plantilla en Azure Portal.

1. En Azure Portal, elija **Crear un recurso**.
1. En **Buscar en Marketplace**, escriba **implementación de plantillas** y, después, presione **ENTRAR**.
1. Elija **Template Deployment**, elija **Crear** y, luego, **Cree su propia plantilla en el editor**.
1. En el editor de plantillas, pegue el código JSON siguiente. Reemplace el marcador de posición `<accountName>` por el nombre de la cuenta de almacenamiento.
1. Guarde la plantilla.
1. Especifique el grupo de recursos de la cuenta y, luego, elija el botón **Comprar** para implementar la plantilla y habilitar el control de versiones de blob.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {},
        "resources": [
            {
                "type": "Microsoft.Storage/storageAccounts/blobServices",
                "apiVersion": "2019-06-01",
                "name": "<accountName>/default",
                "properties": {
                    "IsVersioningEnabled": true
                }
            }
        ]
    }
    ```

Para obtener más información sobre la implementación de recursos con plantillas en Azure Portal, vea [Implementación de recursos con Azure Portal](../../azure-resource-manager/templates/deploy-portal.md).

---

## <a name="modify-a-blob-to-trigger-a-new-version"></a>Modificar un blob para desencadenar una nueva versión

En el ejemplo de código siguiente se muestra cómo desencadenar la creación de una nueva versión con la biblioteca de cliente de Azure Storage para .NET versión [12.5.0-preview.5](https://www.nuget.org/packages/Azure.Storage.Blobs/12.5.0-preview.5) o posterior. Antes de ejecutar este ejemplo, asegúrese de que ha habilitado el control de versiones para la cuenta de almacenamiento.

En el ejemplo se crea un blob en bloques y, a continuación, se actualizan los metadatos del blob. La actualización de los metadatos del blob desencadena la creación de una nueva versión. En el ejemplo se recupera la versión inicial y la versión actual, y se muestra que solo la versión actual incluye los metadatos.

```csharp
public static async Task UpdateVersionedBlobMetadata(string containerName, string blobName)
{
    // Create a new service client from the connection string.
    BlobServiceClient blobServiceClient = new BlobServiceClient(ConnectionString);

    // Create a new container client.
    BlobContainerClient containerClient = blobServiceClient.GetBlobContainerClient(containerName);

    try
    {
        // Create the container.
        await containerClient.CreateIfNotExistsAsync();

        // Upload a block blob.
        BlockBlobClient blockBlobClient = containerClient.GetBlockBlobClient(blobName);

        string blobContents = string.Format("Block blob created at {0}.", DateTime.Now);
        byte[] byteArray = Encoding.ASCII.GetBytes(blobContents);

        string initalVersionId;
        using (MemoryStream stream = new MemoryStream(byteArray))
        {
            Response<BlobContentInfo> uploadResponse = await blockBlobClient.UploadAsync(stream, null, default);

            // Get the version ID for the current version.
            initalVersionId = uploadResponse.Value.VersionId;
        }

        // Update the blob's metadata to trigger the creation of a new version.
        Dictionary<string, string> metadata = new Dictionary<string, string>
        {
            { "key", "value" },
            { "key1", "value1" }
        };

        Response<BlobInfo> metadataResponse = await blockBlobClient.SetMetadataAsync(metadata);

        // Get the version ID for the new current version.
        string newVersionId = metadataResponse.Value.VersionId;

        // Request metadata on the previous version.
        BlockBlobClient initalVersionBlob = blockBlobClient.WithVersion(initalVersionId);
        Response<BlobProperties> propertiesResponse = await initalVersionBlob.GetPropertiesAsync();
        PrintMetadata(propertiesResponse);

        // Request metadata on the current version.
        BlockBlobClient newVersionBlob = blockBlobClient.WithVersion(newVersionId);
        Response<BlobProperties> newPropertiesResponse = await newVersionBlob.GetPropertiesAsync();
        PrintMetadata(newPropertiesResponse);
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
    finally
    {
        await containerClient.DeleteAsync();
    }
}

static void PrintMetadata(Response<BlobProperties> propertiesResponse)
{
    if (propertiesResponse.Value.Metadata.Count > 0)
    {
        Console.WriteLine("Metadata values for version {0}:", propertiesResponse.Value.VersionId);
        foreach (var item in propertiesResponse.Value.Metadata)
        {
            Console.WriteLine("Key:{0}  Value:{1}", item.Key, item.Value);
        }
    }
    else
    {
        Console.WriteLine("Version {0} has no metadata.", propertiesResponse.Value.VersionId);
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

- [Versiones de blobs (versión preliminar)](versioning-overview.md)
- [Eliminación temporal de blobs de Azure Storage](soft-delete-overview.md)
