---
title: 'Versión preliminar: creación de una versión de imagen cifrada con claves propias'
description: Cree una versión de imagen en una galería de imágenes compartidas mediante claves de cifrado administradas por el cliente.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 08/11/2020
ms.author: cynthn
ms.openlocfilehash: 91f485d03717ab80bac26abd16da165d7b0dead7
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291932"
---
# <a name="preview-use-customer-managed-keys-for-encrypting-images"></a>Vista previa: uso de claves administradas por el cliente para el cifrado de imágenes

Las imágenes de la galería se almacenan como discos administrados, por lo que se cifran de forma automática mediante cifrado del lado servidor. El cifrado del lado servidor usa el [cifrado AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) de 256 bits, uno de los cifrados de bloques más seguros que existen, compatible con FIPS 140-2. Para más información sobre de los módulos criptográficos subyacentes en los discos administrados de Azure, consulte [Cryptography API: Next Generation](/windows/desktop/seccng/cng-portal)

Puede confiar en las claves administradas por la plataforma para el cifrado de las imágenes, usar sus propias claves o usar ambas conjuntamente, para un cifrado doble. Si opta por administrar el cifrado con claves propias, puede especificar una *clave administrada por el cliente* que se usará para cifrar y descifrar todos los discos de las imágenes. 

El cifrado del lado servidor mediante claves administradas por el cliente usa Azure Key Vault. Puede importar [las claves RSA](../key-vault/keys/hsm-protected-keys.md) a su instancia de Key Vault o generar nuevas claves RSA en Azure Key Vault.

## <a name="prerequisites"></a>Requisitos previos

Para este artículo, es necesario que ya disponga de un conjunto de cifrado de disco para su imagen.

- Para usar solo una clave administrada por el cliente, consulte **Habilitación de claves administradas por el cliente con el cifrado del lado servidor** mediante [Azure Portal](./disks-enable-customer-managed-keys-portal.md) o [PowerShell](./windows/disks-enable-customer-managed-keys-powershell.md#set-up-your-azure-key-vault-and-diskencryptionset).

- Para usar las claves administradas por la plataforma y las administradas por el cliente (para el cifrado doble), consulte **Habilitación del cifrado doble en reposo** mediante [Azure Portal](./disks-enable-double-encryption-at-rest-portal.md) o [PowerShell](./windows/disks-enable-double-encryption-at-rest-powershell.md).
    > [!IMPORTANT]
    > Debe usar este vínculo [https://aka.ms/diskencryptionupdates](https://aka.ms/diskencryptionupdates) para tener acceso a Azure Portal. El cifrado doble en reposo no está visible actualmente en Azure Portal público sin usar el vínculo.

## <a name="limitations"></a>Limitaciones

Hay varias limitaciones al usar claves administradas por el cliente para cifrar imágenes de la galería de imágenes compartidas:  

- Los conjuntos de claves de cifrado deben estar en la misma suscripción y región que la imagen.

- No se pueden compartir imágenes que usan claves administradas por el cliente. 

- Las imágenes que usan claves administradas por el cliente no se pueden replicar en otras regiones.

- Una vez que haya usado claves propias para cifrar un disco o una imagen, no podrá volver a usar claves administradas por la plataforma para cifrar esos discos o imágenes.


> [!IMPORTANT]
> El cifrado mediante claves administradas por el cliente se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="powershell"></a>PowerShell

En la versión preliminar pública, primero debe registrar la característica.

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName SIGEncryption -ProviderNamespace Microsoft.Compute
```

El proceso de registro tarda unos minutos en completarse. Use Get-AzProviderFeature para comprobar el estado del registro de características.

```azurepowershell-interactive
Get-AzProviderFeature -FeatureName SIGEncryption -ProviderNamespace Microsoft.Compute
```

Cuando RegistrationState devuelve Registered (Registrado), puede continuar al paso siguiente.

Compruebe el registro del proveedor. Asegúrese de que devuelve `Registered`.

```azurepowershell-interactive
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute | Format-table -Property ResourceTypes,RegistrationState
```

Si no devuelve `Registered`, use lo siguiente para registrar los proveedores:

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
```

Para especificar un conjunto de cifrado de disco para una versión de imagen, use [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion) con el parámetro `-TargetRegion`. 

```azurepowershell-interactive

$sourceId = <ID of the image version source>

$osDiskImageEncryption = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet'}

$dataDiskImageEncryption1 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet1';Lun=1}

$dataDiskImageEncryption2 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet2';Lun=2}

$dataDiskImageEncryptions = @($dataDiskImageEncryption1,$dataDiskImageEncryption2)

$encryption1 = @{OSDiskImage=$osDiskImageEncryption;DataDiskImages=$dataDiskImageEncryptions}

$region1 = @{Name='West US';ReplicaCount=1;StorageAccountType=Standard_LRS;Encryption=$encryption1}

$targetRegion = @($region1)


# Create the image
New-AzGalleryImageVersion `
   -ResourceGroupName $rgname `
   -GalleryName $galleryName `
   -GalleryImageDefinitionName $imageDefinitionName `
   -Name $versionName -Location $location `
   -SourceImageId $sourceId `
   -ReplicaCount 2 `
   -StorageAccountType Standard_LRS `
   -PublishingProfileEndOfLifeDate '2020-12-01' `
   -TargetRegion $targetRegion
```

### <a name="create-a-vm"></a>Crear una VM

Puede crear una máquina virtual a partir de una galería de imágenes compartidas y usar claves administradas por el cliente para cifrar los discos. La sintaxis es la misma que para crear una máquina virtual [generalizada](vm-generalized-image-version-powershell.md) o [especializada](vm-specialized-image-version-powershell.md) a partir de una imagen; debe usar el conjunto de parámetros extendidos y agregar `Set-AzVMOSDisk -Name $($vmName +"_OSDisk") -DiskEncryptionSetId $diskEncryptionSet.Id -CreateOption FromImage` a la configuración de la máquina virtual.

En el caso de los discos de datos, debe agregar el parámetro `-DiskEncryptionSetId $setID` al usar [Add-AzVMDataDisk](/powershell/module/az.compute/add-azvmdatadisk).


## <a name="cli"></a>CLI 

En la versión preliminar pública, primero debe registrar la característica.

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name SIGEncryption
```

Compruebe el estado del registro de la característica.

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name SIGEncryption | grep state
```

Cuando esto devuelva `"state": "Registered"`, puede avanzar al paso siguiente.

Compruebe el registro.

```azurecli-interactive
az provider show -n Microsoft.Compute | grep registrationState
```

Si no se muestra como registrada, ejecute lo siguiente:

```azurecli-interactive
az provider register -n Microsoft.Compute
```


Para especificar un disco para una versión de imagen, use [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create) con el parámetro `--target-region-encryption`. El formato de `--target-region-encryption` es una lista de claves separadas por comas para cifrar los discos de sistema operativo y de datos. Debería ser parecido a este: `<encryption set for the OS disk>,<Lun number of the data disk>,<encryption set for the data disk>,<Lun number for the second data disk>,<encryption set for the second data disk>`. 

Si el origen del disco del sistema operativo es un disco administrado o una máquina virtual, use `--managed-image` para especificar el origen de la versión de la imagen. En este ejemplo, el origen es una imagen administrada que tiene un disco del sistema operativo y un disco de datos en LUN 0. El disco del sistema operativo se cifrará con DiskEncryptionSet1 y el de datos con DiskEncryptionSet2.

```azurecli-interactive
az sig image-version create \
   -g MyResourceGroup \
   --gallery-image-version 1.0.0 \
   --location westus \
   --target-regions westus=2=standard_lrs \
   --target-region-encryption DiskEncryptionSet1,0,DiskEncryptionSet2 \
   --gallery-name MyGallery \
   --gallery-image-definition MyImage \
   --managed-image "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage"
```

Si el origen del disco del sistema operativo es una instantánea, use `--os-snapshot` para especificarlo. Si hay instantáneas de disco de datos que también deben formar parte de la versión de la imagen, agréguelas mediante `--data-snapshot-luns` para especificar el LUN y `--data-snapshots` para especificar las instantáneas.

En este ejemplo, los orígenes son instantáneas de disco. Hay un disco del sistema operativo y también uno de datos en LUN 0. El disco del sistema operativo se cifrará con DiskEncryptionSet1 y el de datos con DiskEncryptionSet2.

```azurecli-interactive
az sig image-version create \
   -g MyResourceGroup \
   --gallery-image-version 1.0.0 \
   --location westus\
   --target-regions westus=2=standard_lrs \
   --target-region-encryption DiskEncryptionSet1,0,DiskEncryptionSet2 \
   --os-snapshot "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/snapshots/myOSSnapshot" \
   --data-snapshot-luns 0 \
   --data-snapshots "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/snapshots/myDDSnapshot" \
   --gallery-name MyGallery \
   --gallery-image-definition MyImage 
   
```

### <a name="create-the-vm"></a>Creación de la máquina virtual

Puede crear una máquina virtual a partir de una galería de imágenes compartidas y usar claves administradas por el cliente para cifrar los discos. La sintaxis es la misma que para crear una máquina virtual [generalizada](vm-generalized-image-version-cli.md) o [especializada](vm-specialized-image-version-cli.md) a partir de una imagen; solo hay que agregar el parámetro `--os-disk-encryption-set` con el identificador del conjunto de cifrado. En el caso de los discos de datos, agregue `--data-disk-encryption-sets` con una lista delimitada por espacios de los conjuntos de cifrado de disco para los discos de datos.


## <a name="portal"></a>Portal

Al crear la versión de la imagen en el portal, puede usar la pestaña **Cifrado** para especificar la información sobre los conjuntos de cifrado de almacenamiento.

> [!IMPORTANT]
> Para usar el cifrado doble, debe usar este vínculo [https://aka.ms/diskencryptionupdates](https://aka.ms/diskencryptionupdates) para tener acceso a Azure Portal. El cifrado doble en reposo no está visible actualmente en Azure Portal público sin usar el vínculo.


1. En la página **Crear una versión de imagen**, seleccione la pestaña **Cifrado**.
2. En **Tipo de cifrado**, seleccione **Cifrado en reposo con una clave administrada por el cliente** o **Cifrado doble con claves administradas por el cliente y la plataforma**. 
3. En cada disco de la imagen, seleccione en la lista desplegable el **conjunto de cifrado de disco** que se va a usar. 

### <a name="create-the-vm"></a>Creación de la máquina virtual

Puede crear una máquina virtual a partir de una versión de imagen y usar claves administradas por el cliente para cifrar los discos. Al crear la máquina virtual en el portal, en la pestaña **Discos**, seleccione **Cifrado en reposo con claves administradas por el cliente** o **Cifrado doble con claves administradas por el cliente y la plataforma** para **Tipo de cifrado**. Después puede seleccionar el conjunto de cifrado en la lista desplegable.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre [cifrado de disco del lado servidor](./windows/disk-encryption.md).

Para obtener información sobre cómo proporcionar información del plan de compra, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](marketplace-images.md).
