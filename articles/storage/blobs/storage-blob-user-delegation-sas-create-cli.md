---
title: Uso de la CLI de Azure para crear una SAS de delegación de usuarios para un contenedor o blob
titleSuffix: Azure Storage
description: Aprenda a crear una SAS de delegación de usuarios con credenciales de Azure Active Directory mediante la CLI de Azure.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/18/2019
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: blobs
ms.custom: devx-track-azurecli
ms.openlocfilehash: ddfa12bd758ffc362c51e10f63800665109d54a6
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534080"
---
# <a name="create-a-user-delegation-sas-for-a-container-or-blob-with-the-azure-cli"></a>Creación de una SAS de delegación de usuarios para un contenedor o blob con la CLI de Azure

[!INCLUDE [storage-auth-sas-intro-include](../../../includes/storage-auth-sas-intro-include.md)]

En este artículo se muestra cómo usar las credenciales de Azure Active Directory (Azure AD) para crear una SAS de delegación de usuarios para un contenedor o blob con la CLI de Azure.

[!INCLUDE [storage-auth-user-delegation-include](../../../includes/storage-auth-user-delegation-include.md)]

## <a name="install-the-latest-version-of-the-azure-cli"></a>Instalar la versión más reciente de la CLI de Azure

Para usar la CLI de Azure para asegurar una SAS con credenciales de Azure AD, primero asegúrese de haber instalado la última versión de la CLI de Azure. Para obtener más información sobre cómo instalar la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Para crear una SAS de delegación de usuarios mediante la CLI de Azure, asegúrese de que ha instalado la versión 2.0.78 o posterior. Para comprobar la versión instalada, use el comando `az --version`.

## <a name="sign-in-with-azure-ad-credentials"></a>Iniciar sesión con las credenciales de Azure AD

Inicie sesión en la CLI de Azure con sus credenciales de Azure AD. Para obtener más información, consulte [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli).

## <a name="assign-permissions-with-rbac"></a>Asignación de permisos con RBAC

Para crear una SAS de delegación de usuarios desde Azure PowerShell, se debe asignar a la cuenta de Azure AD utilizada para iniciar sesión en la CLI de Azure un rol que incluya la acción **Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey**. Gracias a este permiso, la cuenta de Azure AD puede solicitar la *clave de delegación de usuarios*. La clave de delegación de usuarios se usa para firmar la SAS de delegación de los usuarios. El rol que proporciona la acción **Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey** debe asignarse en el nivel de la cuenta de almacenamiento, el grupo de recursos o la suscripción.

Si no tiene permisos suficientes para asignar roles de Azure a la entidad de seguridad de Azure AD, puede que tenga que pedir al propietario o administrador de la cuenta que asigne los permisos necesarios.

En el ejemplo siguiente se asigna el rol de **colaborador de datos de Storage Blob**, que incluye la acción **Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey**. El ámbito del rol se encuentra en el nivel de la cuenta de almacenamiento.

No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

Para obtener más información sobre los roles integrados que incluyen la acción **Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey**, consulte los [roles integrados de Azure](../../role-based-access-control/built-in-roles.md).

## <a name="use-azure-ad-credentials-to-secure-a-sas"></a>Uso de credenciales de Azure AD para proteger una SAS

Cuando se crea una SAS de delegación de usuarios con la CLI de Azure, la clave de delegación de usuarios que se usa para firmar la SAS se crea de forma implícita. La hora de inicio y la hora de expiración que especifique para la SAS también se usan como hora de inicio y hora de expiración para la clave de delegación de usuarios.

Dado que el intervalo máximo de validez de la clave de delegación de usuarios es de siete días a partir de la fecha de inicio, debe especificar una hora de expiración para la SAS que está en el plazo de siete días a partir de la hora de inicio. La SAS no es válida después de que expire la clave de delegación de usuarios, por lo que una SAS con un tiempo de expiración de más de siete días seguirá siendo válida solo durante siete días.

Al crear una SAS de delegación de usuarios, se requieren `--auth-mode login` y `--as-user parameters`. Especifique un *inicio de sesión* para el parámetro `--auth-mode` y que así las solicitudes realizadas a Azure Storage estén autorizadas con sus credenciales de Azure AD. Especifique el parámetro `--as-user` para indicar que la SAS devuelta debe ser una SAS de delegación de usuarios.

### <a name="create-a-user-delegation-sas-for-a-container"></a>Creación de una SAS de delegación de usuarios para un contenedor

Para crear una SAS de delegación de usuario para un contenedor con la CLI de Azure, llame al comando [az storage container generate-sas](/cli/azure/storage/container#az-storage-container-generate-sas).

Los permisos admitidos para una SAS de delegación de usuarios en un contenedor incluyen Agregar, Crear, Eliminar, Enumerar, Leer y Escribir. Los permisos se pueden especificar individualmente o combinados. Para obtener más información sobre estos permisos, consulte [Crear una SAS de delegación de usuarios](/rest/api/storageservices/create-user-delegation-sas).

En el ejemplo siguiente se devuelve un token de SAS de delegación de usuarios para un contenedor. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```azurecli-interactive
az storage container generate-sas \
    --account-name <storage-account> \
    --name <container> \
    --permissions acdlrw \
    --expiry <date-time> \
    --auth-mode login \
    --as-user
```

El token de SAS de delegación de usuarios devuelto será similar al siguiente:

```
se=2019-07-27&sp=r&sv=2018-11-09&sr=c&skoid=<skoid>&sktid=<sktid>&skt=2019-07-26T18%3A01%3A22Z&ske=2019-07-27T00%3A00%3A00Z&sks=b&skv=2018-11-09&sig=<signature>
```

### <a name="create-a-user-delegation-sas-for-a-blob"></a>Creación de una SAS de delegación de usuarios para un blob

Para crear una SAS de delegación de usuario para un blob con la CLI de Azure, llame al comando [az storage blob generate-sas](/cli/azure/storage/blob#az-storage-blob-generate-sas).

Los permisos admitidos para una SAS de delegación de usuarios en un blob incluyen Agregar, Crear, Eliminar, Leer y Escribir. Los permisos se pueden especificar individualmente o combinados. Para obtener más información sobre estos permisos, consulte [Crear una SAS de delegación de usuarios](/rest/api/storageservices/create-user-delegation-sas).

La siguiente sintaxis devuelve una SAS de delegación de usuarios para un blob. En el ejemplo se especifica el parámetro `--full-uri`, que devuelve el URI del blob con el token de SAS anexado. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```azurecli-interactive
az storage blob generate-sas \
    --account-name <storage-account> \
    --container-name <container> \
    --name <blob> \
    --permissions acdrw \
    --expiry <date-time> \
    --auth-mode login \
    --as-user
    --full-uri
```

El URI del token de SAS de delegación de usuarios devuelto será similar al siguiente:

```
https://storagesamples.blob.core.windows.net/sample-container/blob1.txt?se=2019-08-03&sp=rw&sv=2018-11-09&sr=b&skoid=<skoid>&sktid=<sktid>&skt=2019-08-02T2
2%3A32%3A01Z&ske=2019-08-03T00%3A00%3A00Z&sks=b&skv=2018-11-09&sig=<signature>
```

> [!NOTE]
> Una SAS de delegación de usuarios no admite la definición de permisos con una directiva de acceso almacenada.

## <a name="revoke-a-user-delegation-sas"></a>Revocación de una SAS de delegación de usuarios

Para revocar una SAS de delegación de usuario desde la CLI de Azure, llame al comando [az storage account revoke-delegation-keys](/cli/azure/storage/account#az-storage-account-revoke-delegation-keys). Este comando revoca todas las claves de delegación de usuarios asociadas a la cuenta de almacenamiento especificada. Se invalidan todas las firmas de acceso compartido asociadas a esas claves.

No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

```azurecli-interactive
az storage account revoke-delegation-keys \
    --name <storage-account> \
    --resource-group <resource-group>
```

> [!IMPORTANT]
> La clave de delegación de usuario y las asignaciones de roles de Azure se almacenan en la caché de Azure Storage, por lo que puede haber un retraso entre el momento en que se inicia el proceso de revocación y el momento en que una SAS de delegación de usuario existente deja de ser válida.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una SAS de delegación de usuario (API de REST)](/rest/api/storageservices/create-user-delegation-sas)
- [Obtener la operación de clave de delegación de usuario](/rest/api/storageservices/get-user-delegation-key)
