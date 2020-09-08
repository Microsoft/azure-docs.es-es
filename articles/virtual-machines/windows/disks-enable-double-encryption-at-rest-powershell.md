---
title: Azure PowerShell - Habilitación del cifrado doble en reposo - discos administrados
description: Habilite el cifrado doble en reposo para los datos de discos administrados mediante Azure PowerShell.
author: roygara
ms.date: 08/24/2020
ms.topic: how-to
ms.author: rogarana
ms.service: virtual-machines-windows
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 6cb0bf9ae12169f9431cf221431f442b5870d932
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816514"
---
# <a name="use-the-azure-powershell-module-to-enable-double-encryption-at-rest-for-managed-disks"></a>Uso del módulo de Azure PowerShell para habilitar el cifrado doble en reposo para discos administrados

Azure Disk Storage admite el cifrado doble en reposo para los discos administrados. Para obtener información conceptual sobre el cifrado doble en reposo, así como otros tipos de cifrado de discos administrados, consulte la sección [Doble cifrado en reposo](disk-encryption.md#double-encryption-at-rest) de nuestro artículo sobre el cifrado de discos.

## <a name="supported-regions"></a>Regiones admitidas

[!INCLUDE [virtual-machines-disks-double-encryption-at-rest-regions](../../../includes/virtual-machines-disks-double-encryption-at-rest-regions.md)]

## <a name="prerequisites"></a>Requisitos previos

Instale la [versión de Azure PowerShell](/powershell/azure/install-az-ps)e inicie sesión en una cuenta de Azure con [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-4.3.0).

## <a name="getting-started"></a>Introducción

1. Cree una instancia de Azure Key Vault y la clave de cifrado.

    Al crear la instancia de Key Vault, debe habilitar la eliminación temporal y la protección de purgas. La eliminación temporal garantiza que la instancia de Key Vault conserva una clave eliminada durante un período de retención determinado (valor predeterminado de 90 días). La protección de purgas garantiza que una clave eliminada no se puede eliminar permanentemente hasta que transcurra el período de retención. Esta configuración le protege contra la pérdida de datos debido a la eliminación accidental. Estos valores son obligatorios cuando se usa una instancia de Key Vault para cifrar discos administrados.
    
    ```powershell
    $ResourceGroupName="yourResourceGroupName"
    $LocationName="westus2"
    $keyVaultName="yourKeyVaultName"
    $keyName="yourKeyName"
    $keyDestination="Software"
    $diskEncryptionSetName="yourDiskEncryptionSetName"

    $keyVault = New-AzKeyVault -Name $keyVaultName -ResourceGroupName $ResourceGroupName -Location $LocationName -EnableSoftDelete -EnablePurgeProtection

    $key = Add-AzKeyVaultKey -VaultName $keyVaultName -Name $keyName -Destination $keyDestination  
    ```

1.  Cree un elemento DiskEncryptionSet con encryptionType establecido como EncryptionAtRestWithPlatformAndCustomerKeys. Use la versión de API **2020-05-01** en la plantilla Azure Resource Manager (ARM). 
    
    ```powershell
    New-AzResourceGroupDeployment -ResourceGroupName $ResourceGroupName `
    -TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/DoubleEncryption/CreateDiskEncryptionSetForDoubleEncryption.json" `
    -diskEncryptionSetName $diskEncryptionSetName `
    -keyVaultId $keyVault.ResourceId `
    -keyVaultKeyUrl $key.Key.Kid `
    -encryptionType "EncryptionAtRestWithPlatformAndCustomerKeys" `
    -region $LocationName
    ```

1. Conceda al recurso DiskEncryptionSet acceso al almacén de claves.

    > [!NOTE]
    > Azure puede tardar unos minutos en crear la identidad del elemento DiskEncryptionSet en Azure Active Directory. Si recibe un error similar a "No se encuentra el objeto en Active Directory" al ejecutar el siguiente comando, espere unos minutos y vuelva a intentarlo.

    ```powershell  
    $des=Get-AzDiskEncryptionSet -name $diskEncryptionSetName -ResourceGroupName $ResourceGroupName
    Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ObjectId $des.Identity.PrincipalId -PermissionsToKeys wrapkey,unwrapkey,get
    ```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado y configurado estos recursos, puede usarlos para proteger los discos administrados. En los vínculos siguientes encontrará scripts de ejemplo, cada uno con un escenario correspondiente, que puede usar para proteger sus discos administrados.

- [Azure PowerShell: habilitación de claves administradas por el cliente con el cifrado del lado servidor para discos administrados](disks-enable-customer-managed-keys-powershell.md)
- [Ejemplos de plantillas de Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/DoubleEncryption)