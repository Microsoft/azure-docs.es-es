---
title: Cifrado de discos para conjuntos de escalado de Azure con Azure PowerShell
description: Obtenga información sobre cómo usar Azure PowerShell para cifrar las instancias de máquina virtual y los discos conectados en un conjunto de escalado de máquinas virtuales Windows
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 10/15/2019
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: 70995575e745b74561745363a1290564c9ce3002
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87029644"
---
# <a name="encrypt-os-and-attached-data-disks-in-a-virtual-machine-scale-set-with-azure-powershell"></a>Cifrado de discos de datos conectados y de sistema operativo en un conjunto de escalado de máquinas virtuales con Azure PowerShell

El módulo de Azure PowerShell se usa para crear y administrar recursos de Azure desde la línea de comandos de PowerShell o en scripts.  En este artículo se muestra cómo usar Azure PowerShell para crear y cifrar un conjunto de escalado de máquinas virtuales. Para más información sobre cómo aplicar Azure Disk Encryption a un conjunto de escalado de máquinas virtuales, consulte [Azure Disk Encryption para conjuntos de escalado de máquinas virtuales](disk-encryption-overview.md).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-an-azure-key-vault-enabled-for-disk-encryption"></a>Creación de una instancia de Azure Key Vault habilitada para el cifrado de discos

Azure Key Vault puede almacenar claves, secretos o contraseñas que permiten su implementación segura en las aplicaciones y los servicios. Las claves criptográficas se almacenan en Azure Key Vault con protección de software, o puede importar o generar las claves en módulos de seguridad de hardware (HSM) certificados conforme a las normas FIPS 140-2 de nivel 2. Las claves criptográficas se usan para cifrar y descifrar los discos virtuales conectados a la máquina virtual. Estas claves criptográficas se pueden controlar y se puede auditar su uso.

Cree una instancia de Key Vault con [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault). Para permitir que la instancia de Key Vault se use para el cifrado de discos, establezca el parámetro *EnabledForDiskEncryption*. En el ejemplo siguiente también se definen las variables para el nombre del grupo de recursos, el nombre de la instancia de Key Vault y la ubicación. Proporcione su propio nombre de Key Vault:

```azurepowershell-interactive
$rgName="myResourceGroup"
$vaultName="myuniquekeyvault"
$location = "EastUS"

New-AzResourceGroup -Name $rgName -Location $location
New-AzKeyVault -VaultName $vaultName -ResourceGroupName $rgName -Location $location -EnabledForDiskEncryption
```

### <a name="use-an-existing-key-vault"></a>Uso de una instancia existente de Key Vault

Este paso solo es necesario si tiene una instancia de Key Vault que desea usar con el cifrado de discos. Omita este paso si creó una instancia de Key Vault en la sección anterior.

Puede habilitar una instancia de Key Vault existente en la misma suscripción y región que el conjunto de escalado para el cifrado de discos con [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/Set-AzKeyVaultAccessPolicy). Defina el nombre de la instancia existente de Key Vault en la variable *$vaultName* como se indica a continuación:


```azurepowershell-interactive
$vaultName="myexistingkeyvault"
Set-AzKeyVaultAccessPolicy -VaultName $vaultName -EnabledForDiskEncryption
```

## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado

En primer lugar, establezca un nombre de usuario de administrador y una contraseña para las instancias de máquina virtual con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1):

```azurepowershell-interactive
$cred = Get-Credential
```

Ahora, cree un conjunto de escalado de máquinas virtuales con [New-AzVmss](/powershell/module/az.compute/new-azvmss). Para distribuir el tráfico a las instancias individuales de VM, también se crea un equilibrador de carga. El equilibrador de carga incluye reglas para distribuir el tráfico en el puerto TCP 80, así como también permite el tráfico de Escritorio remoto en el puerto TCP 3389 y la conexión remota de PowerShell en el puerto TCP 5985:

```azurepowershell-interactive
$vmssName="myScaleSet"

New-AzVmss `
    -ResourceGroupName $rgName `
    -VMScaleSetName $vmssName `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -PublicIpAddressName "myPublicIPAddress" `
    -LoadBalancerName "myLoadBalancer" `
    -UpgradePolicy "Automatic" `
    -Credential $cred
```

## <a name="enable-encryption"></a>Habilitación del cifrado

Para cifrar las instancias de VM de un conjunto de escalado, primero obtenga información sobre el identificador de recurso y el URI de Key Vault con [Get-AzKeyVault](/powershell/module/az.keyvault/Get-AzKeyVault). Estas variables se usan luego para iniciar el proceso de cifrado con [Set-AzVmssDiskEncryptionExtension](/powershell/module/az.compute/Set-AzVmssDiskEncryptionExtension):


```azurepowershell-interactive
$diskEncryptionKeyVaultUrl=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).VaultUri
$keyVaultResourceId=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).ResourceId

Set-AzVmssDiskEncryptionExtension -ResourceGroupName $rgName -VMScaleSetName $vmssName `
    -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $keyVaultResourceId -VolumeType "All"
```

Cuando se le solicite, escriba *y* para continuar el proceso de cifrado de disco en las instancias de VM del conjunto de escalado.

### <a name="enable-encryption-using-kek-to-wrap-the-key"></a>Habilitación del cifrado con KEK para ajustar la clave

También puede usar una clave de cifrado de claves para mayor seguridad al cifrar el conjunto de escalado de máquinas virtuales.

```azurepowershell-interactive
$diskEncryptionKeyVaultUrl=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).VaultUri
$keyVaultResourceId=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).ResourceId
$keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $vaultName -Name $keyEncryptionKeyName).Key.kid;

Set-AzVmssDiskEncryptionExtension -ResourceGroupName $rgName -VMScaleSetName $vmssName `
    -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $keyVaultResourceId `
    -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $keyVaultResourceId -VolumeType "All"
```

> [!NOTE]
>  La sintaxis para el valor del parámetro disk-encryption-keyvault es la cadena de identificador completa:</br>
/subscriptions/[guid-id-suscripción]/resourceGroups/[nombre-grupo-recursos]/providers/Microsoft.KeyVault/vaults/[nombre-almacén-claves]</br></br>
> La sintaxis para el valor del parámetro key-encryption-key es el identificador URI completo de la clave de cifrado de claves, como se muestra a continuación:</br>
https://[nombre-almacén-claves].vault.azure.net/keys/[nombre-clave-cifrado]/[id-exclusivo-clave-cifrado]

## <a name="check-encryption-progress"></a>Comprobación del progreso del cifrado

Para comprobar el estado del cifrado de disco, use [Get-AzVmssDiskEncryption](/powershell/module/az.compute/Get-AzVmssDiskEncryption):


```azurepowershell-interactive
Get-AzVmssDiskEncryption -ResourceGroupName $rgName -VMScaleSetName $vmssName
```

Cuando las instancias de VM están cifradas, el código *EncryptionSummary* informa *ProvisioningState/succeeded*, como se muestra en la salida de ejemplo a continuación:

```powershell
ResourceGroupName            : myResourceGroup
VmScaleSetName               : myScaleSet
EncryptionSettings           :
  KeyVaultURL                : https://myuniquekeyvault.vault.azure.net/
  KeyEncryptionKeyURL        :
  KeyVaultResourceId         : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.KeyVault/vaults/myuniquekeyvault
  KekVaultResourceId         :
  KeyEncryptionAlgorithm     :
  VolumeType                 : All
  EncryptionOperation        : EnableEncryption
EncryptionSummary[0]         :
  Code                       : ProvisioningState/succeeded
  Count                      : 2
EncryptionEnabled            : True
EncryptionExtensionInstalled : True
```

## <a name="disable-encryption"></a>Deshabilitación del cifrado

Si ya no quiere usar los discos de instancias de VM cifrados, puede deshabilitar el cifrado con [Disable-AzVmssDiskEncryption](/powershell/module/az.compute/Disable-AzVmssDiskEncryption), como se indica a continuación:


```azurepowershell-interactive
Disable-AzVmssDiskEncryption -ResourceGroupName $rgName -VMScaleSetName $vmssName
```

## <a name="next-steps"></a>Pasos siguientes

- En este artículo, usó Azure PowerShell para cifrar un conjunto de escalado de máquinas virtuales. También puede usar la [CLI de Azure](disk-encryption-cli.md) o [plantillas de Azure Resource Manager](disk-encryption-azure-resource-manager.md).
- Si quiere aplicar Azure Disk Encryption después de aprovisionar otra extensión, puede usar la [secuenciación de extensiones](virtual-machine-scale-sets-extension-sequencing.md).
