---
title: Uso de .NET para crear una SAS de delegación de usuarios para un contenedor o un blob
titleSuffix: Azure Storage
description: Aprenda a crear una SAS de delegación de usuarios con credenciales de Azure Active Directory en Azure Storage mediante la biblioteca cliente de .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/18/2019
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: blobs
ms.openlocfilehash: e499b27124ed5723fb19fecbe4091f17364aea5a
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533995"
---
# <a name="create-a-user-delegation-sas-for-a-container-or-blob-with-net"></a>Creación de una SAS de delegación de usuarios para un contenedor o blob con .NET

[!INCLUDE [storage-auth-sas-intro-include](../../../includes/storage-auth-sas-intro-include.md)]

En este artículo se muestra cómo usar credenciales de Azure Active Directory (Azure AD) para crear una SAS de delegación de usuarios para un contenedor o blob con la biblioteca cliente de Azure Storage para .NET versión 12.

[!INCLUDE [storage-auth-user-delegation-include](../../../includes/storage-auth-user-delegation-include.md)]

## <a name="assign-azure-roles-for-access-to-data"></a>Asignación de roles de Azure para el acceso a los datos

Cuando una entidad de seguridad de Azure AD intenta acceder a los datos de blobs, dicha entidad debe tener permisos para el recurso. Si la entidad de seguridad es una identidad administrada en Azure o una cuenta de usuario de Azure AD que ejecuta código en el entorno de desarrollo, se le debe asignar un rol de Azure que conceda acceso a los datos de blobs en Azure Storage. Para información sobre la asignación de permisos a través de RBAC, consulte la sección titulada **Asignación de roles de Azure para derechos de acceso** en [Autorización del acceso a blobs y colas de Azure con Azure Active Directory](../common/storage-auth-aad.md#assign-azure-roles-for-access-rights).

[!INCLUDE [storage-install-packages-blob-and-identity-include](../../../includes/storage-install-packages-blob-and-identity-include.md)]

Para más información sobre cómo autenticarse con la biblioteca de cliente de Azure Identity desde Azure Storage, consulte la sección titulada **Autenticación con la biblioteca de Azure Identity** en [Autorización del acceso a blobs y colas con Azure Active Directory e identidades administradas para los recursos de Azure](../common/storage-auth-aad-msi.md?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json#authenticate-with-the-azure-identity-library).

## <a name="add-using-directives"></a>Adición de directivas using

Agregue las siguientes directivas `using` al código para usar las bibliotecas de cliente de Azure Storage y Azure Identity.

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Azure;
using Azure.Identity;
using Azure.Storage.Sas;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
```

## <a name="get-an-authenticated-token-credential"></a>Obtención de una credencial de token autenticada

Para obtener una credencial de token que el código pueda usar para autorizar solicitudes para Azure Storage, cree una instancia de la clase [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential).

En el fragmento de código siguiente se muestra cómo obtener la credencial de token autenticada y usarla para crear un cliente de servicio para Blob Storage:

```csharp
// Construct the blob endpoint from the account name.
string blobEndpoint = string.Format("https://{0}.blob.core.windows.net", accountName);

// Create a new Blob service client with Azure AD credentials.
BlobServiceClient blobClient = new BlobServiceClient(new Uri(blobEndpoint),
                                                     new DefaultAzureCredential());
```

## <a name="get-the-user-delegation-key"></a>Obtención de la clave de delegación de usuario

Cada SAS se firma con una clave. Para crear una SAS de delegación de usuarios, primero debe solicitar una clave de delegación de usuario, que se usará para firmar la SAS. La clave de delegación de usuarios es análoga a la clave de cuenta usada para firmar una SAS de servicio o una SAS de cuenta, excepto en que se basa en las credenciales de Azure AD. Cuando un cliente solicita una clave de delegación de usuario mediante un token de OAuth 2.0, Azure Storage devuelve la clave de delegación de usuario en nombre del usuario.

Una vez que tenga la clave de delegación de usuario, puede usar esa clave para crear cualquier número de firmas de acceso compartido de delegación de usuarios durante la vigencia de la clave. La clave de delegación de usuario es independiente del token de OAuth 2.0 que se usa para adquirirla, por lo que no es necesario renovar el token siempre y cuando la clave siga siendo válida. Puede especificar que la clave sea válida para un período de hasta 7 días.

Use uno de los métodos siguientes para solicitar la clave de delegación de usuario:

- [GetUserDelegationKey](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.getuserdelegationkey)
- [GetUserDelegationKeyAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.getuserdelegationkeyasync)

El siguiente fragmento de código obtiene la clave de delegación de usuario y escribe sus propiedades:

```csharp
// Get a user delegation key for the Blob service that's valid for seven days.
// You can use the key to generate any number of shared access signatures over the lifetime of the key.
UserDelegationKey key = await blobClient.GetUserDelegationKeyAsync(DateTimeOffset.UtcNow,
                                                                   DateTimeOffset.UtcNow.AddDays(7));

// Read the key's properties.
Console.WriteLine("User delegation key properties:");
Console.WriteLine("Key signed start: {0}", key.SignedStartsOn);
Console.WriteLine("Key signed expiry: {0}", key.SignedExpiresOn);
Console.WriteLine("Key signed object ID: {0}", key.SignedObjectId);
Console.WriteLine("Key signed tenant ID: {0}", key.SignedTenantId);
Console.WriteLine("Key signed service: {0}", key.SignedService);
Console.WriteLine("Key signed version: {0}", key.SignedVersion);
```

## <a name="create-the-sas-token"></a>Creación del token de SAS

En el fragmento de código siguiente se muestra cómo crear un nuevo [BlobSasBuilder](/dotnet/api/azure.storage.sas.blobsasbuilder) y proporcionar los parámetros para la SAS de delegación de usuarios. Después, el fragmento de código llama a [ToSasQueryParameters](/dotnet/api/azure.storage.sas.blobsasbuilder.tosasqueryparameters) para obtener la cadena del token de SAS. Por último, el código genera el URI completo, incluida la dirección de recursos y el token de SAS.

```csharp
// Create a SAS token that's valid for one hour.
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = containerName,
    BlobName = blobName,
    Resource = "b",
    StartsOn = DateTimeOffset.UtcNow,
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
};

// Specify read permissions for the SAS.
sasBuilder.SetPermissions(BlobSasPermissions.Read);

// Use the key to get the SAS token.
string sasToken = sasBuilder.ToSasQueryParameters(key, accountName).ToString();

// Construct the full URI, including the SAS token.
UriBuilder fullUri = new UriBuilder()
{
    Scheme = "https",
    Host = string.Format("{0}.blob.core.windows.net", accountName),
    Path = string.Format("{0}/{1}", containerName, blobName),
    Query = sasToken
};
```

## <a name="example-get-a-user-delegation-sas"></a>Ejemplo: Obtención de una SAS de delegación de usuarios

En el siguiente método de ejemplo se muestra el código completo para autenticar la entidad de seguridad y crear la SAS de delegación de usuarios:

```csharp
async static Task<Uri> GetUserDelegationSasBlob(string accountName, string containerName, string blobName)
{
    // Construct the blob endpoint from the account name.
    string blobEndpoint = string.Format("https://{0}.blob.core.windows.net", accountName);

    // Create a new Blob service client with Azure AD credentials.  
    BlobServiceClient blobClient = new BlobServiceClient(new Uri(blobEndpoint), 
                                                            new DefaultAzureCredential());

    // Get a user delegation key for the Blob service that's valid for seven days.
    // You can use the key to generate any number of shared access signatures over the lifetime of the key.
    UserDelegationKey key = await blobClient.GetUserDelegationKeyAsync(DateTimeOffset.UtcNow, 
                                                                        DateTimeOffset.UtcNow.AddDays(7));

    // Read the key's properties.
    Console.WriteLine("User delegation key properties:");
    Console.WriteLine("Key signed start: {0}", key.SignedStartsOn);
    Console.WriteLine("Key signed expiry: {0}", key.SignedExpiresOn);
    Console.WriteLine("Key signed object ID: {0}", key.SignedObjectId);
    Console.WriteLine("Key signed tenant ID: {0}", key.SignedTenantId);
    Console.WriteLine("Key signed service: {0}", key.SignedService);
    Console.WriteLine("Key signed version: {0}", key.SignedVersion);
    Console.WriteLine();

    // Create a SAS token that's valid for one hour.
    BlobSasBuilder sasBuilder = new BlobSasBuilder()
    {
        BlobContainerName = containerName,
        BlobName = blobName,
        Resource = "b",
        StartsOn = DateTimeOffset.UtcNow,
        ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
    };

    // Specify read permissions for the SAS.
    sasBuilder.SetPermissions(BlobSasPermissions.Read);

    // Use the key to get the SAS token.
    string sasToken = sasBuilder.ToSasQueryParameters(key, accountName).ToString();

    // Construct the full URI, including the SAS token.
    UriBuilder fullUri = new UriBuilder()
    {
        Scheme = "https",
        Host = string.Format("{0}.blob.core.windows.net", accountName),
        Path = string.Format("{0}/{1}", containerName, blobName),
        Query = sasToken
    };

    Console.WriteLine("User delegation SAS URI: {0}", fullUri);
    Console.WriteLine();
    return fullUri.Uri;
}
```

## <a name="example-read-a-blob-with-a-user-delegation-sas"></a>Ejemplo: Lectura de un blob con una SAS de delegación de usuarios

En el ejemplo siguiente se prueba la SAS de delegación de usuarios creada en el ejemplo anterior desde una aplicación cliente simulada. Si la SAS es válida, la aplicación cliente puede leer el contenido del blob. Si la SAS no es válida, por ejemplo, si ha expirado, Azure Storage devuelve el código de error 403 (prohibido).

```csharp
private static async Task ReadBlobWithSasAsync(Uri sasUri)
{
    // Try performing blob operations using the SAS provided.

    // Create a blob client object for blob operations.
    BlobClient blobClient = new BlobClient(sasUri, null);

    // Download and read the contents of the blob.
    try
    {
        // Download blob contents to a stream and read the stream.
        BlobDownloadInfo blobDownloadInfo = await blobClient.DownloadAsync();
        using (StreamReader reader = new StreamReader(blobDownloadInfo.Content, true))
        {
            string line;
            while ((line = reader.ReadLine()) != null)
            {
                Console.WriteLine(line);
            }
        }

        Console.WriteLine();
        Console.WriteLine("Read operation succeeded for SAS {0}", sasUri);
        Console.WriteLine();
    }
    catch (RequestFailedException e)
    {
        // Check for a 403 (Forbidden) error. If the SAS is invalid,
        // Azure Storage returns this error.
        if (e.Status == 403)
        {
            Console.WriteLine("Read operation failed for SAS {0}", sasUri);
            Console.WriteLine("Additional error information: " + e.Message);
            Console.WriteLine();
        }
        else
        {
            Console.WriteLine(e.Message);
            Console.ReadLine();
            throw;
        }
    }
}
```

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="see-also"></a>Consulte también

- [Grant limited access to Azure Storage resources using shared access signatures (SAS)](../common/storage-sas-overview.md) (Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido [SAS])
- [Obtener la operación de clave de delegación de usuario](/rest/api/storageservices/get-user-delegation-key)
- [Creación de una SAS de delegación de usuario (API de REST)](/rest/api/storageservices/create-user-delegation-sas)
