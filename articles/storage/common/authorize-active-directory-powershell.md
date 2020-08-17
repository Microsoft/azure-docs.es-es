---
title: Ejecución de comandos de PowerShell con credenciales de Azure AD para acceder a datos de blob o de cola
titleSuffix: Azure Storage
description: PowerShell admite el inicio de sesión con credenciales de Azure AD para ejecutar comandos en los datos de blob y de cola de Azure Storage. Se proporciona un token de acceso para la sesión y se usa para autorizar operaciones de llamada. Los permisos dependen del rol de Azure asignado a la entidad de seguridad de Azure AD.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/30/2019
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: a99b2841441e43ac02688494b1324fa5d630dcd5
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534930"
---
# <a name="run-powershell-commands-with-azure-ad-credentials-to-access-blob-or-queue-data"></a>Ejecución de comandos de PowerShell con credenciales de Azure AD para acceder a datos de blob o de cola

Azure Storage proporciona extensiones para PowerShell que le permiten iniciar sesión y ejecutar comandos de scripting con credenciales de Azure Active Directory (Azure AD). Al iniciar sesión en PowerShell con credenciales de Azure AD, se devuelve un token de acceso OAuth 2.0. PowerShell usa automáticamente ese token para autorizar las operaciones de datos posteriores en Blob Storage o Queue Storage. Para las operaciones admitidas, ya no necesita pasar una clave de cuenta o token SAS con el comando.

Puede asignar permisos en los datos de blob y de cola a una entidad de seguridad de Azure AD a través del control de acceso basado en rol (RBAC). Para más información sobre los roles de Azure en Azure Storage, vea [Administración de los derechos de acceso a los datos de Azure Storage con RBAC](storage-auth-aad-rbac.md).

## <a name="supported-operations"></a>Operaciones compatibles

Las extensiones de Azure Storage se admiten para las operaciones en datos de blob o de cola. Las operaciones a las que podrá llamar dependerán de los permisos que se concedan a la entidad de seguridad de Azure AD con la que inicie sesión en PowerShell. Los permisos para los contenedores o colas de Azure Storage se asignan mediante RBAC. Por ejemplo, si se le asigna el rol de **lector de datos de blob**, puede ejecutar comandos de scripting que lean datos de un contenedor o cola. Si se le asigna el rol de **colaborador de datos de blob**, podrá ejecutar comandos de scripting que lean, escriban o eliminen un contenedor o cola, o los datos que estos contienen.

Para más información sobre los permisos requeridos para cada operación de Azure Storage en un contenedor o una cola, consulte el artículo de [Llamada a las operaciones de almacenamiento con tokens de OAuth](/rest/api/storageservices/authorize-with-azure-active-directory#call-storage-operations-with-oauth-tokens).  

## <a name="call-powershell-commands-using-azure-ad-credentials"></a>Llamada a comandos de PowerShell mediante credenciales de Azure AD

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Si quiere usar Azure PowerShell para iniciar sesión y ejecutar operaciones posteriores en Azure Storage mediante credenciales de Azure AD, cree un contexto de almacenamiento para hacer referencia a la cuenta de almacenamiento e incluya el parámetro `-UseConnectedAccount`.

En el ejemplo siguiente se muestra cómo crear un contenedor en una nueva cuenta de almacenamiento desde Azure PowerShell mediante las credenciales de Azure AD. No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

1. Inicie sesión en su cuenta de Azure con el comando [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount):

    ```powershell
    Connect-AzAccount
    ```

    Para obtener más información sobre cómo iniciar sesión en Azure con PowerShell, consulte [Inicio de sesión con Azure PowerShell](/powershell/azure/authenticate-azureps).

1. Cree un grupo de recursos de Azure mediante una llamada a [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). 

    ```powershell
    $resourceGroup = "sample-resource-group-ps"
    $location = "eastus"
    New-AzResourceGroup -Name $resourceGroup -Location $location
    ```

1. Cree una cuenta de almacenamiento mediante una llamada a [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount).

    ```powershell
    $storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
      -Name "<storage-account>" `
      -SkuName Standard_LRS `
      -Location $location `
    ```

1. Obtenga el contexto de la cuenta de almacenamiento que especifica la nueva cuenta de almacenamiento mediante una llamada a [New-AzStorageContext](/powershell/module/az.storage/new-azstoragecontext). Cuando actúe en una cuenta de almacenamiento, puede hacer referencia al contexto en lugar de proporcionar varias veces las credenciales. Incluya el parámetro `-UseConnectedAccount` para llamar a las operaciones de datos posteriores con sus credenciales de Azure AD:

    ```powershell
    $ctx = New-AzStorageContext -StorageAccountName "<storage-account>" -UseConnectedAccount
    ```

1. Antes de crear el contenedor, asígnese a sí mismo el rol [Colaborador de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor). Aunque sea el propietario de la cuenta, necesita permisos explícitos para realizar operaciones de datos en la cuenta de almacenamiento. Para más información sobre la asignación de roles de Azure, vea [Conceder acceso a los datos de blobs y colas de Azure con RBAC en Azure Portal](storage-auth-aad-rbac.md).

    > [!IMPORTANT]
    > La propagación de las asignaciones de roles de Azure puede tardar unos minutos.

1. Cree un contenedor mediante una llamada a [New-AzStorageContainer](/powershell/module/az.storage/new-azstoragecontainer). Dado que esta llamada usa el contexto creado en los pasos anteriores, el contenedor se crea con sus credenciales de Azure AD.

    ```powershell
    $containerName = "sample-container"
    New-AzStorageContainer -Name $containerName -Context $ctx
    ```

## <a name="next-steps"></a>Pasos siguientes

- [Uso de PowerShell para asignar un rol de Azure para el acceso a datos de blobs y colas](storage-auth-aad-rbac-powershell.md)
- [Autorización del acceso a datos de blobs y colas con identidades administradas para los recursos de Azure](storage-auth-aad-msi.md)
