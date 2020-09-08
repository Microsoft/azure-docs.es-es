---
title: Autorización del acceso a datos con una identidad administrada
titleSuffix: Azure Storage
description: Use identidades administradas para recursos de Azure para autorizar el acceso a datos de blobs y colas desde aplicaciones que se ejecutan en máquinas virtuales de Azure, aplicaciones de funciones y otras.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/04/2019
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 2087e5f8ec397123df504e9d30d351a0ba79b4a5
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018763"
---
# <a name="authorize-access-to-blob-and-queue-data-with-managed-identities-for-azure-resources"></a>Autorización del acceso a datos de blobs y colas con identidades administradas para los recursos de Azure

El almacenamiento de blobs y colas de Azure admite la autenticación de Azure Active Directory (Azure AD) con [identidades administradas para los recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md). Las identidades administradas para recursos de Azure pueden autorizar el acceso a datos de blobs y colas con credenciales de Azure AD desde aplicaciones que se ejecutan en máquinas virtuales (VM) de Azure, aplicaciones de función, conjuntos de escalado de máquinas virtuales y otros servicios. Si usa identidades administradas para recursos de Azure junto con autenticación de Azure AD, puede evitar el almacenamiento de credenciales con las aplicaciones que se ejecutan en la nube.  

En este artículo se muestra cómo autorizar el acceso a los datos de blobs o colas desde una máquina virtual de Azure mediante identidades administradas para recursos de Azure. También se describe cómo probar el código en el entorno de desarrollo.

## <a name="enable-managed-identities-on-a-vm"></a>Habilitación de identidades administradas en una máquina virtual

Para poder usar identidades administradas para recursos de Azure a fin de autorizar el acceso a blobs y colas desde la máquina virtual, primero debe habilitar las identidades administradas para los recursos de Azure en la máquina virtual. Para aprender a habilitar las identidades administradas para los recursos de Azure, consulte uno de estos artículos:

- [Azure Portal](https://docs.microsoft.com/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm)
- [Azure PowerShell](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [CLI de Azure](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Plantilla de Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Bibliotecas cliente de Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

Para más información sobre las identidades administradas, consulte [Identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md).

## <a name="authenticate-with-the-azure-identity-library"></a>Autenticación con la biblioteca de identidades de Azure

La biblioteca cliente de identidades de Azure proporciona a Azure compatibilidad con la autenticación de token de Azure AD para [Azure SDK](https://github.com/Azure/azure-sdk). Las versiones más recientes de las bibliotecas cliente de Azure Storage para .NET, Java, Python y JavaScript se integran con la biblioteca de identidades de Azure para proporcionar un medio sencillo y seguro de obtener un token de OAuth 2.0 para la autorización de las solicitudes de Azure Storage.

Una ventaja de la biblioteca cliente de identidades de Azure es que permite usar el mismo código para autenticar tanto si la aplicación se ejecuta en el entorno de desarrollo como en Azure. La biblioteca cliente de identidades de Azure para .NET autentica una entidad de seguridad. Cuando el código se ejecuta en Azure, la entidad de seguridad es una identidad administrada para los recursos de Azure. En el entorno de desarrollo, la identidad administrada no existe, por lo que la biblioteca cliente autentica tanto el usuario como una entidad de servicio con fines de prueba.

Después de la autenticación, la biblioteca cliente de identidades de Azure obtiene una credencial de token. Esta credencial de token se encapsula después en el objeto de cliente de servicio que se crea para realizar operaciones en Azure Storage. La biblioteca administra este escenario sin problemas mediante la obtención de la credencial de token adecuada.

Para más información sobre la biblioteca cliente de identidades de Azure, consulte [Biblioteca cliente de identidades de Azure para .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity). Para consultar documentación de referencia sobre la biblioteca cliente de identidades de Azure, consulte [Espacio de nombres Azure.Identity](/dotnet/api/azure.identity).

### <a name="assign-azure-roles-for-access-to-data"></a>Asignación de roles de Azure para el acceso a los datos

Cuando una entidad de seguridad de Azure AD intenta acceder a los datos de blobs o colas, dicha entidad debe tener permisos para el recurso. Si la entidad de seguridad es una identidad administrada de Azure o una cuenta de usuario de Azure AD que ejecuta código en el entorno de desarrollo, se le debe asignar un rol de Azure que conceda acceso a los datos de blobs o colas en Azure Storage. Para información sobre la asignación de permisos a través de RBAC, consulte la sección titulada **Asignación de roles de Azure para derechos de acceso** en [Autorización del acceso a blobs y colas de Azure con Azure Active Directory](../common/storage-auth-aad.md#assign-azure-roles-for-access-rights).

### <a name="authenticate-the-user-in-the-development-environment"></a>Autenticación del usuario en el entorno de desarrollo

Cuando el código se ejecuta en el entorno de desarrollo, la autenticación se puede administrar automáticamente o puede requerir el inicio de sesión en un explorador, según las herramientas que use. Por ejemplo, Microsoft Visual Studio admite el inicio de sesión único (SSO), de forma que la cuenta de usuario de Azure AD activa se utiliza automáticamente para la autenticación. Para más información sobre el inicio de sesión único, consulte [Inicio de sesión único en las aplicaciones](../../active-directory/manage-apps/what-is-single-sign-on.md).

Otras herramientas de desarrollo pueden pedirle que inicie sesión mediante un explorador web.

### <a name="authenticate-a-service-principal-in-the-development-environment"></a>Autenticación de una entidad de servicio en el entorno de desarrollo

Si el entorno de desarrollo no admite el inicio de sesión único a través de un explorador web, puede usar una entidad de servicio para autenticarse desde el entorno de desarrollo.

#### <a name="create-the-service-principal"></a>Creación de la entidad de servicio

Para crear una entidad de servicio con la CLI de Azure y asignar el rol de Azure, llame al comando [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac). Proporcione un rol de acceso a datos Azure Storage para asignar a la nueva entidad de servicio. Además, proporcione el ámbito de la asignación de roles. Para más información sobre los roles integrados que se proporcionan para Azure Storage, consulte [Roles integrados de Azure](../../role-based-access-control/built-in-roles.md).

Si no tiene permisos suficientes para asignar un rol a la entidad de servicio, puede que tenga que pedir al propietario o administrador de la cuenta que realice la asignación de roles.

En el ejemplo siguiente se usa la CLI de Azure para crear una nueva entidad de servicio y asignarle el rol **lector de datos de blob de almacenamiento** con el ámbito de cuenta.

```azurecli-interactive
az ad sp create-for-rbac \
    --name <service-principal> \
    --role "Storage Blob Data Reader" \
    --scopes /subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>
```

El comando `az ad sp create-for-rbac` devuelve una lista de propiedades de entidad de servicio en formato JSON. Copie estos valores para poder usarlos con el fin de crear las variables de entorno necesarias en el paso siguiente.

```json
{
    "appId": "generated-app-ID",
    "displayName": "service-principal-name",
    "name": "http://service-principal-uri",
    "password": "generated-password",
    "tenant": "tenant-ID"
}
```

> [!IMPORTANT]
> La propagación de las asignaciones de roles de Azure pueden tardar unos minutos.

#### <a name="set-environment-variables"></a>Establecimiento de variables de entorno

La biblioteca cliente de Azure Identity lee valores de tres variables de entorno en tiempo de ejecución para autenticar la entidad de servicio. En la tabla siguiente se describe el valor que se debe establecer para cada variable de entorno.

|Variable de entorno|Value
|-|-
|`AZURE_CLIENT_ID`|Id. de la aplicación de la entidad de servicio
|`AZURE_TENANT_ID`|Id. de inquilino de Azure AD de la entidad de servicio
|`AZURE_CLIENT_SECRET`|Contraseña generada para la entidad de servicio

> [!IMPORTANT]
> Después de establecer las variables de entorno, cierre y vuelva a abrir la ventana de la consola. Si usa Visual Studio u otro entorno de desarrollo, es posible que tenga que reiniciar el entorno de desarrollo para que registre las nuevas variables de entorno.

Para más información, consulte [Creación de una identidad para aplicaciones de Azure en el portal](../../active-directory/develop/howto-create-service-principal-portal.md).

[!INCLUDE [storage-install-packages-blob-and-identity-include](../../../includes/storage-install-packages-blob-and-identity-include.md)]

## <a name="net-code-example-create-a-block-blob"></a>Ejemplo de código .NET: creación de un blob en bloques

Agregue las siguientes directivas `using` al código para usar las bibliotecas de cliente de Azure Storage y Azure Identity.

```csharp
using Azure;
using Azure.Identity;
using Azure.Storage.Blobs;
using System;
using System.IO;
using System.Text;
using System.Threading.Tasks;
```

Para obtener una credencial de token que el código pueda usar para autorizar solicitudes para Azure Storage, cree una instancia de la clase [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential). En el ejemplo de código siguiente se muestra cómo obtener la credencial de token autenticada y usarla para crear un objeto de cliente de servicio y, luego, usar el cliente de servicio para cargar un nuevo blob:

```csharp
async static Task CreateBlockBlobAsync(string accountName, string containerName, string blobName)
{
    // Construct the blob container endpoint from the arguments.
    string containerEndpoint = string.Format("https://{0}.blob.core.windows.net/{1}",
                                                accountName,
                                                containerName);

    // Get a credential and create a client object for the blob container.
    BlobContainerClient containerClient = new BlobContainerClient(new Uri(containerEndpoint),
                                                                    new DefaultAzureCredential());

    try
    {
        // Create the container if it does not exist.
        await containerClient.CreateIfNotExistsAsync();

        // Upload text to a new block blob.
        string blobContents = "This is a block blob.";
        byte[] byteArray = Encoding.ASCII.GetBytes(blobContents);

        using (MemoryStream stream = new MemoryStream(byteArray))
        {
            await containerClient.UploadBlobAsync(blobName, stream);
        }
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

> [!NOTE]
> Para autorizar solicitudes en datos de blob o cola con Azure AD, debe usar HTTPS para esas solicitudes.

## <a name="next-steps"></a>Pasos siguientes

- [Administrar los derechos de acceso a los datos de almacenamiento con RBAC](storage-auth-aad-rbac.md).
- [Usar Azure AD con aplicaciones de almacenamiento](storage-auth-aad-app.md).
- [Ejecutar la CLI de Azure o comandos de PowerShell con credenciales de Azure AD para acceder a datos de blob o de cola](authorize-active-directory-powershell.md).
