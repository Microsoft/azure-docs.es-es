---
title: 'Autenticar el acceso a los blobs y colas con identidades administradas para Azure resources: almacenamiento de Azure | Microsoft Docs'
description: Almacenamiento de blobs y colas de Azure admiten la autenticación de Azure Active Directory con identidades administradas para los recursos de Azure. Puede usar identidades administradas para los recursos de Azure para autenticar el acceso a blobs y colas desde aplicaciones que se ejecutan en máquinas virtuales de Azure, aplicaciones de función, conjuntos de escalado de máquinas virtuales y otros.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/21/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: f7525c3e125010bb4db9655bc214861e22dc8875
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787975"
---
# <a name="authenticate-access-to-blobs-and-queues-with-azure-active-directory-and-managed-identities-for-azure-resources"></a>Autenticar el acceso a los blobs y colas con Azure Active Directory y las identidades administradas para los recursos de Azure

El almacenamiento de blobs y colas de Azure admite la autenticación de Azure Active Directory (Azure AD) con [identidades administradas para los recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md). Administra identidades para los recursos de Azure pueden autorizar el acceso al blob y datos de la cola mediante credenciales de Azure AD desde aplicaciones que se ejecutan en máquinas virtuales (VM), aplicaciones de función, conjuntos de escalado de máquinas virtuales y otros servicios. Mediante el uso de identidades administradas para los recursos de Azure junto con la autenticación de Azure AD, puede evitar almacenar las credenciales con las aplicaciones que se ejecutan en la nube.  

En este artículo se muestra cómo autorizar el acceso a datos blob o cola con una identidad administrada desde una máquina virtual de Azure. 

## <a name="enable-managed-identities-on-a-vm"></a>Habilitación de identidades administradas en una máquina virtual

Para poder usar las identidades administradas para los recursos de Azure para autorizar el acceso a los blobs y colas de la máquina virtual, primero debe habilitar las identidades administradas para los recursos de Azure en la máquina virtual. Para aprender a habilitar las identidades administradas para los recursos de Azure, consulte uno de estos artículos:

- [Azure Portal](https://docs.microsoft.com/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm)
- [Azure PowerShell](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [CLI de Azure](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Plantilla de Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [SDK de Azure](../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

## <a name="grant-permissions-to-an-azure-ad-managed-identity"></a>Conceder permisos a una identidad administrada de Azure AD

Para autorizar una solicitud al servicio Blob o cola de una identidad administrada en la aplicación de Azure Storage, primero configure acceso basado en roles (RBAC) de control para esa identidad administrada. Almacenamiento de Azure define los roles RBAC que abarcan los permisos para los datos de blob y cola. Cuando el rol de RBAC se asigna a una identidad administrada, la identidad administrada se concede esos permisos a los datos de blob o cola en el ámbito adecuado. 

Para obtener más información sobre la asignación de roles de RBAC, consulte uno de los siguientes artículos:

- [Conceder acceso a datos blob y cola de Azure con RBAC en Azure portal](storage-auth-aad-rbac-portal.md)
- [Conceder acceso a datos de blob y cola de Azure con RBAC, mediante la CLI de Azure](storage-auth-aad-rbac-cli.md)
- [Conceder acceso a datos de blob y cola de Azure con RBAC con PowerShell](storage-auth-aad-rbac-powershell.md)

## <a name="authorize-with-a-managed-identity-access-token"></a>Autorizar con un token de acceso de identidad administrada

Para autorizar las solicitudes en el almacenamiento de Blob y cola con una identidad administrada, la aplicación o el script debe adquirir un token de OAuth. El [autenticación de la aplicación de Microsoft Azure](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) biblioteca de cliente para .NET (versión preliminar) simplifica el proceso de adquisición y renovación de un token desde el código.

La biblioteca de autenticación de la aplicación cliente administra la autenticación automáticamente. La biblioteca usa las credenciales del desarrollador para autenticarse durante el desarrollo local. El uso de credenciales de desarrollador durante el desarrollo local es más seguro porque no es necesario crear credenciales de Azure AD o compartir credenciales entre los programadores. Cuando la solución se implementa posteriormente a Azure, la biblioteca cambia automáticamente al uso de credenciales de la aplicación.

Para usar la biblioteca de autenticación de la aplicación en una aplicación de Azure Storage, instale el paquete de versión preliminar más reciente de [Nuget](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication), así como la versión más reciente de la [biblioteca de cliente común de Azure Storage para .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/) y el [bibliotecas de cliente de almacenamiento de blobs de Azure para .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/). Agregue el siguiente **mediante** instrucciones en el código:

```csharp
using Microsoft.Azure.Services.AppAuthentication;
using Microsoft.Azure.Storage.Auth;
using Microsoft.Azure.Storage.Blob;
```

La biblioteca de autenticación de la aplicación proporciona el **AzureServiceTokenProvider** clase. Una instancia de esta clase se puede pasar a una devolución de llamada que se obtiene un token y, a continuación, se renueva el token antes de que expire.

En el ejemplo siguiente se obtiene un token y lo usa para crear un nuevo blob, a continuación, se usa el mismo token para leer el blob.

```csharp
const string blobName = "https://storagesamples.blob.core.windows.net/sample-container/blob1.txt";

// Get the initial access token and the interval at which to refresh it.
AzureServiceTokenProvider azureServiceTokenProvider = new AzureServiceTokenProvider();
var tokenAndFrequency = TokenRenewerAsync(azureServiceTokenProvider, 
                                            CancellationToken.None).GetAwaiter().GetResult();

// Create storage credentials using the initial token, and connect the callback function 
// to renew the token just before it expires
TokenCredential tokenCredential = new TokenCredential(tokenAndFrequency.Token, 
                                                        TokenRenewerAsync,
                                                        azureServiceTokenProvider, 
                                                        tokenAndFrequency.Frequency.Value);

StorageCredentials storageCredentials = new StorageCredentials(tokenCredential);

// Create a blob using the storage credentials.
CloudBlockBlob blob = new CloudBlockBlob(new Uri(blobName), 
                                            storageCredentials);

// Upload text to the blob.
blob.UploadTextAsync(string.Format("This is a blob named {0}", blob.Name));

// Continue to make requests against Azure Storage. 
// The token is automatically refreshed as needed in the background.
do
{
    // Read blob contents
    Console.WriteLine("Time accessed: {0} Blob Content: {1}", 
                        DateTimeOffset.UtcNow, 
                        blob.DownloadTextAsync().Result);

    // Sleep for ten seconds, then read the contents of the blob again.
    Thread.Sleep(TimeSpan.FromSeconds(10));
} while (true);
```

El método de devolución de llamada comprueba la fecha de expiración del token y lo renueva según sea necesario:

```csharp
private static async Task<NewTokenAndFrequency> TokenRenewerAsync(Object state, CancellationToken cancellationToken)
{
    // Specify the resource ID for requesting Azure AD tokens for Azure Storage.
    const string StorageResource = "https://storage.azure.com/";  

    // Use the same token provider to request a new token.
    var authResult = await ((AzureServiceTokenProvider)state).GetAuthenticationResultAsync(StorageResource);

    // Renew the token 5 minutes before it expires.
    var next = (authResult.ExpiresOn - DateTimeOffset.UtcNow) - TimeSpan.FromMinutes(5);
    if (next.Ticks < 0)
    {
        next = default(TimeSpan);
        Console.WriteLine("Renewing token...");
    }

    // Return the new token and the next refresh time.
    return new NewTokenAndFrequency(authResult.AccessToken, next);
}
```

Para obtener más información acerca de la biblioteca de autenticación de la aplicación, consulte [autenticación de servicio a servicio en Azure Key Vault mediante .NET](../../key-vault/service-to-service-authentication.md). 

Para obtener más información sobre cómo adquirir un token de acceso, consulte [cómo usar las identidades administradas para los recursos de Azure en una máquina virtual de Azure para adquirir un token de acceso](../../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md).

> [!NOTE]
> Para autorizar las solicitudes con los datos de blob o cola con Azure AD, debe usar HTTPS para dichas solicitudes.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información acerca de los roles RBAC para el almacenamiento de Azure, consulte [administrar derechos de acceso a los datos de almacenamiento con RBAC](storage-auth-aad-rbac.md).
- Para información sobre la autorización de acceso a los contenedores y las colas desde las aplicaciones de almacenamiento, consulte el artículo sobre el [uso de Azure AD con aplicaciones de almacenamiento](storage-auth-aad-app.md).
- Para obtener información sobre cómo ejecutar comandos de PowerShell y CLI de Azure con credenciales de Azure AD, consulte [comandos ejecutar la CLI de Azure o PowerShell con credenciales de Azure AD para tener acceso a datos blob o cola](storage-auth-aad-script.md).