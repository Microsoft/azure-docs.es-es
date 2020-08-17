---
title: Uso de PowerShell para configurar las claves administradas por el cliente
titleSuffix: Azure Storage
description: Configurar las claves que administra el cliente para el cifrado de Azure Storage desde PowerShell
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/13/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: a3fdde755a5e024efead5c8861a1d5cd769b6d23
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87036835"
---
# <a name="configure-customer-managed-keys-with-azure-key-vault-by-using-powershell"></a>Configuración de claves administradas por el cliente con Azure Key Vault mediante PowerShell

[!INCLUDE [storage-encryption-configure-keys-include](../../../includes/storage-encryption-configure-keys-include.md)]

En este artículo se muestra cómo configurar Azure Key Vault con claves administradas por el cliente mediante PowerShell. Para aprender a crear un almacén de claves mediante la CLI de Azure, consulte [Inicio rápido: Establecimiento y recuperación de un secreto de Azure Key Vault mediante PowerShell](../../key-vault/secrets/quick-create-powershell.md).

## <a name="assign-an-identity-to-the-storage-account"></a>Asignación de una identidad a la cuenta de almacenamiento

Para habilitar claves que administra el cliente para la cuenta de almacenamiento, debe asignar primero una identidad administrada que haya asignado el sistema a la cuenta de almacenamiento. Usará esta identidad administrada para conceder los permisos de la cuenta de almacenamiento para obtener acceso al almacén de claves.

Para asignar una identidad administrada con PowerShell, llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount). No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```powershell
$storageAccount = Set-AzStorageAccount -ResourceGroupName <resource_group> `
    -Name <storage-account> `
    -AssignIdentity
```

Para obtener información sobre cómo configurar identidades administradas que el sistema ha asignado con PowerShell, consulte [Configurar identidades administradas para recursos de Azure en una máquina virtual de Azure con PowerShell](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md).

## <a name="create-a-new-key-vault"></a>Creación de un almacén de claves nuevo

Para crear un nuevo almacén de claves mediante PowerShell, instale la versión 2.0.0 o posterior del módulo [Az.KeyVault](https://www.powershellgallery.com/packages/Az.KeyVault/2.0.0) de PowerShell. Luego llame a [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault) para crear un nuevo almacén de claves.

El almacén de claves que se usará para almacenar las claves que administre el cliente para el cifrado de Azure Storage debe tener dos configuraciones de protección de claves habilitadas: la **eliminación temporal** y la opción de **no purgar**. En la versión 2.0.0 y posteriores del módulo Az.KeyVault, la eliminación temporal está habilitada de forma predeterminada cuando se crea un nuevo almacén de claves.

En el ejemplo siguiente se crea un nuevo almacén de claves con las propiedades **Eliminación temporal** y **No purgar** habilitadas. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```powershell
$keyVault = New-AzKeyVault -Name <key-vault> `
    -ResourceGroupName <resource_group> `
    -Location <location> `
    -EnablePurgeProtection
```

Para obtener información sobre cómo habilitar **Eliminación temporal** y **No purgar** en un almacén de claves existente con PowerShell, consulte las secciones tituladas **Habilitar la eliminación temporal** y **Habilitación de la protección de purgas** del artículo sobre [uso de la eliminación temporal con PowerShell](../../key-vault/general/soft-delete-powershell.md).

## <a name="configure-the-key-vault-access-policy"></a>Configuración de la directiva de acceso del almacén de claves

A continuación, configure la directiva de acceso del almacén de claves, para que la cuenta de almacenamiento tenga permiso para acceder a él. En este paso, usará la identidad administrada que asignó previamente a la cuenta de almacenamiento.

Para establecer la directiva de acceso del almacén de claves, llame a [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy). Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```powershell
Set-AzKeyVaultAccessPolicy `
    -VaultName $keyVault.VaultName `
    -ObjectId $storageAccount.Identity.PrincipalId `
    -PermissionsToKeys wrapkey,unwrapkey,get
```

## <a name="create-a-new-key"></a>Creación de una nueva clave

Después, cree una clave nueva en el almacén de claves. Para crear una clave nueva, llame a [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey). Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```powershell
$key = Add-AzKeyVaultKey -VaultName $keyVault.VaultName -Name <key> -Destination 'Software'
```

El cifrado de almacenamiento de Azure admite claves RSA y RSA-HSM de los tamaños 2048, 3072 y 4096. Para más información acerca de las claves, consulte la sección **Claves en Key Vault** en [Información acerca de claves, secretos y certificados de Azure Key Vault](../../key-vault/about-keys-secrets-and-certificates.md#key-vault-keys).

## <a name="configure-encryption-with-customer-managed-keys"></a>Configuración del cifrado con claves que administra el cliente

De forma predeterminada, el cifrado de Azure Storage usa claves que administra Microsoft. En este paso, configure la cuenta de Azure Storage para usar las claves administradas por el cliente con Azure Key Vault, y especifique la clave para asociar a la cuenta de almacenamiento.

Al configurar el cifrado con claves administradas por el cliente, puede optar por rotar automáticamente la clave que se usa para el cifrado cuando la versión cambie en el almacén de claves asociado. Como alternativa, puede especificar explícitamente una versión de clave que se usará para el cifrado hasta que la versión de la clave se actualice manualmente.

### <a name="configure-encryption-for-automatic-rotation-of-customer-managed-keys"></a>Configuración del cifrado para la rotación automática de claves administradas por el cliente

Para configurar el cifrado para la rotación automática de claves administradas por el cliente, instale el módulo [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage), versión 2.0.0 o posterior.

Para rotar automáticamente las claves administradas por el cliente, omita la versión de la clave al configurar las claves administradas por el cliente para la cuenta de almacenamiento. Llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) para actualizar la configuración de cifrado de la cuenta de almacenamiento, como se muestra en el ejemplo siguiente, e incluya la opción **-KeyvaultEncryption** para habilitar las claves administradas por el cliente para la cuenta de almacenamiento. Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -KeyvaultEncryption `
    -KeyName $key.Name `
    -KeyVaultUri $keyVault.VaultUri
```

### <a name="configure-encryption-for-manual-rotation-of-key-versions"></a>Configuración del cifrado para la rotación manual de las versiones de clave

Para especificar de manera explícita una versión de la clave que se va a usar para el cifrado, indique la versión de la clave al configurar el cifrado con las claves administradas por el cliente para la cuenta de almacenamiento. Llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) para actualizar la configuración de cifrado de la cuenta de almacenamiento, como se muestra en el ejemplo siguiente, e incluya la opción **-KeyvaultEncryption** para habilitar las claves administradas por el cliente para la cuenta de almacenamiento. Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -KeyvaultEncryption `
    -KeyName $key.Name `
    -KeyVersion $key.Version `
    -KeyVaultUri $keyVault.VaultUri
```

Al rotar manualmente la versión de la clave, deberá actualizar la configuración de cifrado de la cuenta de almacenamiento para usar la nueva versión. En primer lugar, llame a [Get-AzKeyVaultKey](/powershell/module/az.keyvault/get-azkeyvaultkey) para obtener la versión más reciente de la clave. A continuación, llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) para actualizar la configuración de cifrado de la cuenta de almacenamiento y así poder usar la nueva versión de la clave, tal como se muestra en el ejemplo anterior.

## <a name="use-a-different-key"></a>Uso de una clave distinta

Para cambiar la clave usada para el cifrado de Azure Storage, llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) tal y como se muestra en [Configuración del cifrado con claves que administra el cliente](#configure-encryption-with-customer-managed-keys) y proporcione el nombre y la versión de la nueva clave. Si la nueva clave se encuentra en otro almacén de claves, actualice también el URI del almacén de claves.

## <a name="revoke-customer-managed-keys"></a>Revocación de claves administradas por el cliente

Para revocar las claves administradas por el cliente, quite la directiva de acceso del almacén de claves. Para revocar una clave administrada por el cliente, llame al comando [Remove-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/remove-azkeyvaultaccesspolicy), como se muestra en el siguiente ejemplo. Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```powershell
Remove-AzKeyVaultAccessPolicy -VaultName $keyVault.VaultName `
    -ObjectId $storageAccount.Identity.PrincipalId `
```

## <a name="disable-customer-managed-keys"></a>Deshabilitación de claves administradas por el cliente

Cuando las claves administradas por el cliente se deshabilitan, la cuenta de almacenamiento se vuelve a cifrar con claves administradas por Microsoft. Para deshabilitar las claves administradas por el cliente, llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) con la opción `-StorageEncryption` en, tal y como se muestra en el ejemplo siguiente. Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -StorageEncryption  
```

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md)
- [¿Qué es Azure Key Vault?](https://docs.microsoft.com/azure/key-vault/key-vault-overview)
