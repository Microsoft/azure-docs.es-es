---
title: Uso de PowerShell para crear una SAS de delegación de usuarios para un contenedor o blob
titleSuffix: Azure Storage
description: Aprenda a cómo crear una SAS de delegación de usuarios con credenciales de Azure Active Directory mediante PowerShell.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/18/2019
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: blobs
ms.openlocfilehash: 97eb3f4cbb4ac76823ebe43126db6b5c2a10010b
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533978"
---
# <a name="create-a-user-delegation-sas-for-a-container-or-blob-with-powershell"></a>Creación de una SAS de delegación de usuarios para un contenedor o blob con PowerShell

[!INCLUDE [storage-auth-sas-intro-include](../../../includes/storage-auth-sas-intro-include.md)]

En este artículo se muestra cómo usar las credenciales de Azure Active Directory (Azure AD) para crear una SAS de delegación de usuarios para un contenedor o blob con Azure PowerShell.

[!INCLUDE [storage-auth-user-delegation-include](../../../includes/storage-auth-user-delegation-include.md)]

## <a name="install-the-powershell-module"></a>Instalación del módulo de PowerShell

Para crear una SAS de delegación de usuarios con PowerShell, instale la versión 1.10.0 o posterior del módulo Az.Storage. Siga estos pasos para instalar la versión más reciente del módulo:

1. Desinstale las instalaciones anteriores de Azure PowerShell:

    - Quite cualquier instalación anterior de Azure PowerShell desde Windows usando el ajuste **Aplicaciones y características** en **Configuración**.
    - Quite todos los módulos de **Azure** desde `%Program Files%\WindowsPowerShell\Modules`.

1. Asegúrese de tener instalada la versión más reciente de PowerShellGet. Abra una ventana de Windows PowerShell y ejecute el siguiente comando para instalar la versión más reciente:

    ```powershell
    Install-Module PowerShellGet –Repository PSGallery –Force
    ```

1. Cierre y vuelva a abrir la ventana de PowerShell después de instalar PowerShellGet.

1. Instale la versión más reciente de Azure PowerShell:

    ```powershell
    Install-Module Az –Repository PSGallery –AllowClobber
    ```

1. Asegúrese de que tiene instalada la versión 3.2.0 o posterior de Azure PowerShell. Ejecute el siguiente comando para instalar la versión más reciente del módulo de PowerShell para Azure Storage.

    ```powershell
    Install-Module -Name Az.Storage -Repository PSGallery -Force
    ```

1. Cierre y vuelva a abrir la ventana de PowerShell.

Para comprobar qué versión del módulo Az.Storage está instalada, ejecute el siguiente comando:

```powershell
Get-Module -ListAvailable -Name Az.Storage -Refresh
```

Para más información sobre cómo instalar Azure PowerShell, consulte [Instalación de Azure PowerShell con PowerShellGet](/powershell/azure/install-az-ps).

## <a name="sign-in-to-azure-powershell-with-azure-ad"></a>Inicio de sesión en Azure PowerShell con Azure AD

Llame al comando [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) para iniciar sesión con la cuenta de Azure AD:

```powershell
Connect-AzAccount
```

Para obtener más información sobre cómo iniciar sesión con PowerShell, consulte [Inicio de sesión con Azure PowerShell](/powershell/azure/authenticate-azureps).

## <a name="assign-permissions-with-rbac"></a>Asignación de permisos con RBAC

Para crear una SAS de delegación de usuarios desde Azure PowerShell, se debe asignar a la cuenta de Azure AD utilizada para iniciar sesión en PowerShell un rol que incluya la acción **Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey**. Gracias a este permiso, la cuenta de Azure AD puede solicitar la *clave de delegación de usuarios*. La clave de delegación de usuarios se usa para firmar la SAS de delegación de los usuarios. El rol que proporciona la acción **Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey** debe asignarse en el nivel de la cuenta de almacenamiento, el grupo de recursos o la suscripción. Para más información sobre los permisos de RBAC para crear una SAS de delegación de usuarios, consulte la sección **Asignación de permisos con RBAC** de [Create a user delegation SAS](/rest/api/storageservices/create-user-delegation-sas) (Creación de una SAS de delegación de usuarios).

Si no tiene permisos suficientes para asignar roles de Azure a la entidad de seguridad de Azure AD, puede que tenga que pedir al propietario o administrador de la cuenta que asigne los permisos necesarios.

En el ejemplo siguiente se asigna el rol de **colaborador de datos de Storage Blob**, que incluye la acción **Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey**. El ámbito del rol se encuentra en el nivel de la cuenta de almacenamiento.

No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

```powershell
New-AzRoleAssignment -SignInName <email> `
    -RoleDefinitionName "Storage Blob Data Contributor" `
    -Scope  "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

Para obtener más información sobre los roles integrados que incluyen la acción **Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey**, consulte los [roles integrados de Azure](../../role-based-access-control/built-in-roles.md).

## <a name="use-azure-ad-credentials-to-secure-a-sas"></a>Uso de credenciales de Azure AD para proteger una SAS

Cuando se crea una SAS de delegación de usuarios con Azure PowerShell, la clave de delegación de usuarios que se usa para firmar la SAS se crea de forma implícita. La hora de inicio y la hora de expiración que especifique para la SAS también se usan como hora de inicio y hora de expiración para la clave de delegación de usuarios. 

Dado que el intervalo máximo de validez de la clave de delegación de usuarios es de siete días a partir de la fecha de inicio, debe especificar una hora de expiración para la SAS que está en el plazo de siete días a partir de la hora de inicio. La SAS no es válida después de que expire la clave de delegación de usuarios, por lo que una SAS con un tiempo de expiración de más de siete días seguirá siendo válida solo durante siete días.

Para crear una SAS de delegación de usuarios para un contenedor o blob con Azure PowerShell, primero debe crear un nuevo objeto de contexto de Azure Storage especificando el parámetro `-UseConnectedAccount`. El parámetro `-UseConnectedAccount` especifica que el comando crea el objeto de contexto en la cuenta Azure AD con la que ha iniciado sesión.

No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

```powershell
$ctx = New-AzStorageContext -StorageAccountName <storage-account> -UseConnectedAccount
```

### <a name="create-a-user-delegation-sas-for-a-container"></a>Creación de una SAS de delegación de usuarios para un contenedor

Para devolver un token de SAS de delegación de usuarios para un contenedor, llame al comando [New-AzStorageContainerSASToken](/powershell/module/az.storage/new-azstoragecontainersastoken) y pase el objeto de contexto de Azure Storage que creó anteriormente.

En el ejemplo siguiente se devuelve un token de SAS de delegación de usuarios para un contenedor. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```powershell
New-AzStorageContainerSASToken -Context $ctx `
    -Name <container> `
    -Permission racwdl `
    -ExpiryTime <date-time>
```

El token de SAS de delegación de usuarios devuelto será similar al siguiente:

```output
?sv=2018-11-09&sr=c&sig=<sig>&skoid=<skoid>&sktid=<sktid>&skt=2019-08-05T22%3A24%3A36Z&ske=2019-08-07T07%3A
00%3A00Z&sks=b&skv=2018-11-09&se=2019-08-07T07%3A00%3A00Z&sp=rwdl
```

### <a name="create-a-user-delegation-sas-for-a-blob"></a>Creación de una SAS de delegación de usuarios para un blob

Para devolver un token de SAS de delegación de usuarios para un blob, llame al comando [New-AzStorageBlobSASToken](/powershell/module/az.storage/new-azstorageblobsastoken) y pase el objeto de contexto de Azure Storage que creó anteriormente.

La siguiente sintaxis devuelve una SAS de delegación de usuarios para un blob. En el ejemplo se especifica el parámetro `-FullUri`, que devuelve el URI del blob con el token de SAS anexado. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```powershell
New-AzStorageBlobSASToken -Context $ctx `
    -Container <container> `
    -Blob <blob> `
    -Permission racwd `
    -ExpiryTime <date-time>
    -FullUri
```

El URI del token de SAS de delegación de usuarios devuelto será similar al siguiente:

```output
https://storagesamples.blob.core.windows.net/sample-container/blob1.txt?sv=2018-11-09&sr=b&sig=<sig>&skoid=<skoid>&sktid=<sktid>&skt=2019-08-06T21%3A16%3A54Z&ske=2019-08-07T07%3A00%3A00Z&sks=b&skv=2018-11-09&se=2019-08-07T07%3A00%3A00Z&sp=racwd
```

> [!NOTE]
> Una SAS de delegación de usuarios no admite la definición de permisos con una directiva de acceso almacenada.

## <a name="revoke-a-user-delegation-sas"></a>Revocación de una SAS de delegación de usuarios

Para revocar una SAS de delegación de usuarios de Azure PowerShell, llame al comando **Revoke-AzStorageAccountUserDelegationKeys**. Este comando revoca todas las claves de delegación de usuarios asociadas a la cuenta de almacenamiento especificada. Se invalidan todas las firmas de acceso compartido asociadas a esas claves.

No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

```powershell
Revoke-AzStorageAccountUserDelegationKeys -ResourceGroupName <resource-group> `
    -StorageAccountName <storage-account>
```

> [!IMPORTANT]
> La clave de delegación de usuario y las asignaciones de roles de Azure se almacenan en la caché de Azure Storage, por lo que puede haber un retraso entre el momento en que se inicia el proceso de revocación y el momento en que una SAS de delegación de usuario existente deja de ser válida.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una SAS de delegación de usuario (API de REST)](/rest/api/storageservices/create-user-delegation-sas)
- [Obtener la operación de clave de delegación de usuario](/rest/api/storageservices/get-user-delegation-key)
