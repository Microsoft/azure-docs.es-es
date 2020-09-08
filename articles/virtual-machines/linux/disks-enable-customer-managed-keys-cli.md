---
title: 'CLI de Azure: Habilitación de claves administradas por el cliente con SSE para discos administrados'
description: Habilite las claves administradas por el cliente en los discos administrados con la CLI de Azure.
author: roygara
ms.date: 08/24/2020
ms.topic: how-to
ms.author: rogarana
ms.service: virtual-machines-linux
ms.subservice: disks
ms.openlocfilehash: 2acc95190454de2ffdd81a05a6e6cf166a9a6549
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88814148"
---
# <a name="use-the-azure-cli-to-enable-server-side-encryption-with-customer-managed-keys-for-managed-disks"></a>Uso de la CLI de Azure para habilitar el cifrado del lado servidor con claves administradas por el cliente para los discos administrados

Azure Disk Storage le permite administrar sus propias claves al usar el cifrado del lado servidor (SSE) para discos administrados, si así lo decide. Para obtener información conceptual sobre SSE con claves administradas por el cliente, así como otros tipos de cifrado de discos administrados, consulte la sección [Claves administradas por el cliente](disk-encryption.md#customer-managed-keys) de nuestro artículo sobre el cifrado de discos.

## <a name="restrictions"></a>Restricciones

Por ahora, las claves administradas por el cliente tienen las siguientes restricciones:

- Si esta característica está habilitada para el disco, no puede deshabilitarla.
    Si necesita encontrar una solución alternativa, debe [copiar todos los datos](disks-upload-vhd-to-managed-disk-cli.md#copy-a-managed-disk) en un disco administrado totalmente diferente que no use claves administradas por el cliente.
[!INCLUDE [virtual-machines-managed-disks-customer-managed-keys-restrictions](../../../includes/virtual-machines-managed-disks-customer-managed-keys-restrictions.md)]

## <a name="set-up-your-azure-key-vault-and-diskencryptionset"></a>Configuración de Azure Key Vault y DiskEncryptionSet

En primer lugar, debe configurar una instancia de Azure Key Vault y un recurso diskencryptionset.

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault](../../../includes/virtual-machines-disks-encryption-create-key-vault-cli.md)]

Ahora que ha creado y configurado estos recursos, puede usarlos para proteger los discos administrados. En los vínculos siguientes encontrará scripts de ejemplo, cada uno con un escenario correspondiente, que puede usar para proteger sus discos administrados.

## <a name="examples"></a>Ejemplos

### <a name="create-a-vm-using-a-marketplace-image-encrypting-the-os-and-data-disks-with-customer-managed-keys"></a>Creación de una máquina virtual con una imagen de Marketplace, cifrado del sistema operativo y de los discos de datos con claves administradas por el cliente

```azurecli
rgName=yourResourceGroupName
vmName=yourVMName
location=westcentralus
vmSize=Standard_DS3_V2
image=UbuntuLTS 
diskEncryptionSetName=yourDiskencryptionSetName

diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)

az vm create -g $rgName -n $vmName -l $location --image $image --size $vmSize --generate-ssh-keys --os-disk-encryption-set $diskEncryptionSetId --data-disk-sizes-gb 128 128 --data-disk-encryption-sets $diskEncryptionSetId $diskEncryptionSetId
```

### <a name="encrypt-existing-managed-disks"></a>Cifrado de los discos administrados existentes 

Los discos existentes no deben estar asociados a una máquina virtual en ejecución para que los cifre mediante el siguiente script:

```azurecli
rgName=yourResourceGroupName
diskName=yourDiskName
diskEncryptionSetName=yourDiskEncryptionSetName
 
az disk update -n $diskName -g $rgName --encryption-type EncryptionAtRestWithCustomerKey --disk-encryption-set $diskEncryptionSetId
```

### <a name="create-a-virtual-machine-scale-set-using-a-marketplace-image-encrypting-the-os-and-data-disks-with-customer-managed-keys"></a>Creación de un conjunto de escalado de máquinas virtuales con una imagen de Marketplace, cifrado del sistema operativo y de los discos de datos con claves administradas por el cliente

```azurecli
rgName=yourResourceGroupName
vmssName=yourVMSSName
location=westcentralus
vmSize=Standard_DS3_V2
image=UbuntuLTS 
diskEncryptionSetName=yourDiskencryptionSetName

diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)
az vmss create -g $rgName -n $vmssName --image UbuntuLTS --upgrade-policy automatic --admin-username azureuser --generate-ssh-keys --os-disk-encryption-set $diskEncryptionSetId --data-disk-sizes-gb 64 128 --data-disk-encryption-sets $diskEncryptionSetId $diskEncryptionSetId
```

### <a name="create-an-empty-disk-encrypted-using-server-side-encryption-with-customer-managed-keys-and-attach-it-to-a-vm"></a>Creación de un disco vacío cifrado con cifrado del lado servidor y claves administradas por el cliente y su conexión a una máquina virtual

```azurecli
vmName=yourVMName
rgName=yourResourceGroupName
diskName=yourDiskName
diskSkuName=Premium_LRS
diskSizeinGiB=30
location=westcentralus
diskLUN=2
diskEncryptionSetName=yourDiskEncryptionSetName


diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)

az disk create -n $diskName -g $rgName -l $location --encryption-type EncryptionAtRestWithCustomerKey --disk-encryption-set $diskEncryptionSetId --size-gb $diskSizeinGiB --sku $diskSkuName

diskId=$(az disk show -n $diskName -g $rgName --query [id] -o tsv)

az vm disk attach --vm-name $vmName --lun $diskLUN --ids $diskId 

```

### <a name="change-the-key-of-a-diskencryptionset-to-rotate-the-key-for-all-the-resources-referencing-the-diskencryptionset"></a>Cambiar la clave de DiskEncryptionSet para rotar la clave para todos los recursos que hacen referencia a DiskEncryptionSet

```azurecli

rgName=yourResourceGroupName
keyVaultName=yourKeyVaultName
keyName=yourKeyName
diskEncryptionSetName=yourDiskEncryptionSetName


keyVaultId=$(az keyvault show --name $keyVaultName--query [id] -o tsv)

keyVaultKeyUrl=$(az keyvault key show --vault-name $keyVaultName --name $keyName --query [key.kid] -o tsv)

az disk-encryption-set update -n keyrotationdes -g keyrotationtesting --key-url $keyVaultKeyUrl --source-vault $keyVaultId

```

### <a name="find-the-status-of-server-side-encryption-of-a-disk"></a>Buscar el estado de cifrado del lado servidor de un disco

[!INCLUDE [virtual-machines-disks-encryption-status-cli](../../../includes/virtual-machines-disks-encryption-status-cli.md)]

> [!IMPORTANT]
> Las claves administradas por el cliente dependen de identidades administradas para los recursos de Azure, una característica de Azure Active Directory (Azure AD). Al configurar claves administradas por el cliente, se asigna automáticamente una identidad administrada a los recursos en segundo plano. Si posteriormente mueve la suscripción, el grupo de recursos o el disco administrado de un directorio de Azure AD a otro, la identidad administrada asociada a los discos administrados no se transfiere al nuevo inquilino, por lo que es posible que las claves administradas por el cliente dejen de funcionar. Para obtener más información, consulte [Transferencia de una suscripción entre directorios de Azure AD](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).

## <a name="next-steps"></a>Pasos siguientes

- [Explore las plantillas de Azure Resource Manager para crear discos cifrados con claves administradas por el cliente](https://github.com/ramankumarlive/manageddiskscmkpreview)
- [Replicación de máquinas con discos habilitados para claves administradas por el cliente](../../site-recovery/azure-to-azure-how-to-enable-replication-cmk-disks.md)
- [Configuración de la recuperación ante desastres de máquinas virtuales de VMware en Azure con PowerShell](../../site-recovery/vmware-azure-disaster-recovery-powershell.md#replicate-vmware-vms)
- [Configuración de la recuperación ante desastres en Azure para máquinas virtuales de Hyper-V mediante PowerShell y Azure Resource Manager](../../site-recovery/hyper-v-azure-powershell-resource-manager.md#step-7-enable-vm-protection)