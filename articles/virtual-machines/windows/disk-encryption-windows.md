---
title: Escenarios de Azure Disk Encryption en máquinas virtuales Windows
description: En este artículo se proporcionan las instrucciones necesarias para habilitar Microsoft Azure Disk Encryption para máquinas virtuales Windows en varios escenarios
author: msmbaldwin
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: how-to
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: 8b2a8d552a2b9a1d6d3bb02bf02be95af031a5e4
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87291962"
---
# <a name="azure-disk-encryption-scenarios-on-windows-vms"></a>Escenarios de Azure Disk Encryption en máquinas virtuales Windows

Azure Disk Encryption para máquinas virtuales Windows usa la característica BitLocker de Windows para proporcionar el cifrado de disco completo del disco de sistema operativo y los discos de datos. Además, proporciona cifrado del disco temporal cuando el parámetro VolumeType es All.

Azure Disk Encryption [se integra con Azure Key Vault](disk-encryption-key-vault.md) para ayudarle a controlar y administrar las claves y los secreto de cifrado de discos. Para obtener información general del servicio, consulte [Azure Disk Encryption para máquinas virtuales Windows](disk-encryption-overview.md).

El cifrado de disco solo se puede aplicar a las máquinas virtuales que tengan [tamaños y sistemas operativos compatibles](disk-encryption-overview.md#supported-vms-and-operating-systems). También es preciso que se cumplan los siguientes requisitos previos:

- [Requisitos de red](disk-encryption-overview.md#networking-requirements)
- [Requisitos de la directiva de grupo](disk-encryption-overview.md#group-policy-requirements)
- [Requisitos de almacenamiento de la clave de cifrado](disk-encryption-overview.md#encryption-key-storage-requirements)

>[!IMPORTANT]
> - Si ya ha usado Azure Disk Encryption con Azure AD para cifrar una VM, debe seguir usando esta opción para cifrar la VM. Para más información, consulte [Azure Disk Encryption con Azure AD (versión anterior)](disk-encryption-overview-aad.md). 
>
> - Es preciso [hacer una instantánea](snapshot-copy-managed-disk.md) o crear una copia de seguridad antes de cifrar los discos. Las copias de seguridad garantizan que, en caso de un error inesperado durante el cifrado, sea posible una opción de recuperación. Las máquinas virtuales con discos administrados requieren una copia de seguridad antes del cifrado. Una vez realizada la copia de seguridad, puede usar el [cmdlet Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) para cifrar los discos administrados mediante la especificación del parámetro -skipVmBackup. Para más información sobre cómo realizar la copia de seguridad y la restauración de máquinas virtuales cifradas, consulte [Copia de seguridad y restauración de máquinas virtuales de Azure cifradas](../../backup/backup-azure-vms-encryption.md). 
>
> - Cifrar o deshabilitar el cifrado puede provocar el reinicio de una máquina virtual.

## <a name="install-tools-and-connect-to-azure"></a>Instalación de herramientas y conexión a Azure

[!INCLUDE [disk-encryption-install-cli-powershell](../../../includes/disk-encryption-install-cli-powershell.md)]

## <a name="enable-encryption-on-an-existing-or-running-windows-vm"></a>Habilitación del cifrado en una máquina virtual Windows existente o en ejecución
En este escenario, puede habilitar el cifrado mediante la plantilla de Resource Manager, los cmdlets de PowerShell o los comandos de la CLI. Si necesita información de esquema para la extensión de máquina virtual, consulte el artículo sobre la [extensión de Azure Disk Encryption para Windows](../extensions/azure-disk-enc-windows.md).

### <a name="enable-encryption-on-existing-or-running-vms-with-azure-powershell"></a>Habilitación del cifrado en máquinas virtuales existentes o en ejecución con Azure PowerShell 
Use el cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) para habilitar el cifrado en una máquina virtual IaaS en ejecución en Azure. 

-  **Cifrado de una máquina virtual en ejecución:** el siguiente script inicializa las variables y ejecuta el cmdlet Set-AzVMDiskEncryptionExtension. Ya se deben haber satisfecho los requisitos previos de crear el grupo de recursos, la máquina virtual y el almacén de claves. Reemplace MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM y MySecureVault por los valores deseados.

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;
    ```
- **Cifrado de una máquina virtual en ejecución mediante KEK:** 

     ```azurepowershell
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup';
     $vmName = 'MyExtraSecureVM';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;

     ```
     
   >[!NOTE]
   > La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

- **Comprobar que los discos están cifrados:** para comprobar el estado de cifrado de una máquina virtual de IaaS, use el cmdlet [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus). 
     ```azurepowershell-interactive
     Get-AzVmDiskEncryptionStatus -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```
    
- **Deshabilitar el cifrado de disco:** para deshabilitar el cifrado, use el cmdlet [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). La deshabilitación del cifrado de disco de datos en la VM de Windows cuando se han cifrado tanto los discos de datos como del SO no funciona según lo previsto. Deshabilite del cifrado en todos los discos en su lugar.

     ```azurepowershell-interactive
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```

### <a name="enable-encryption-on-existing-or-running-vms-with-the-azure-cli"></a>Habilitación del cifrado en máquinas virtuales existentes o en ejecución con la CLI de Azure
Use el comando [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) para habilitar el cifrado en una máquina virtual IaaS en ejecución en Azure.

- **Cifrado de una máquina virtual en ejecución:**

    ```azurecli-interactive
    az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
    ```

- **Cifrado de una máquina virtual en ejecución mediante KEK:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

     >[!NOTE]
     > La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name] </br> La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

- **Comprobar que los discos están cifrados:** para comprobar el estado de cifrado de una máquina virtual IaaS, use el comando [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

- **Deshabilitar el cifrado:** para deshabilitar el cifrado, use el comando [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). La deshabilitación del cifrado de disco de datos en la VM de Windows cuando se han cifrado tanto los discos de datos como del SO no funciona según lo previsto. Deshabilite del cifrado en todos los discos en su lugar.

     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type [ALL, DATA, OS]
     ```

### <a name="using-the-resource-manager-template"></a>Uso de la plantilla de Resource Manager

Puede habilitar el cifrado de disco en máquinas virtuales IaaS de Windows existentes o en ejecución en Azure mediante la [plantilla de Resource Manager para cifrar una máquina virtual Windows en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm-without-aad).


1. En la plantilla de inicio rápido de Azure, haga clic en **Deploy to Azure** (Implementar en Azure).

2. Seleccione la suscripción, el grupo de recursos, la ubicación, la configuración, los términos legales y el contrato. Haga clic en **Comprar** para habilitar el cifrado en la máquina virtual IaaS existente o en ejecución.

En la tabla siguiente figuran los parámetros de la plantilla de Resource Manager para máquinas virtuales existentes o en ejecución:

| Parámetro | Descripción |
| --- | --- |
| vmName | Nombre de la máquina virtual para ejecutar la operación de cifrado. |
| keyVaultName | Nombre del almacén de claves en el que se que debe cargar la clave de BitLocker. Puede obtenerlo mediante el cmdlet `(Get-AzKeyVault -ResourceGroupName <MyKeyVaultResourceGroupName>). Vaultname` o el comando de la CLI de Azure `az keyvault list --resource-group "MyKeyVaultResourceGroup"`|
| keyVaultResourceGroup | Nombre del grupo de recursos que contiene el almacén de claves|
|  keyEncryptionKeyURL | La dirección URL de la clave de cifrado de claves con el formato https://&lt;nombreDelAlmacénDeClaves&gt;.vault.azure.net/key/&lt;nombreDeClave&gt;. Si no desea usar una KEK, deje este campo en blanco. |
| volumeType | Tipo de volumen en que se realiza la operación de cifrado. Los valores válidos son _SO_, _Datos_ y _Todo_. 
| forceUpdateTag | Cada vez que la operación tenga que ejecutarse, pase un valor único como GUID. |
| resizeOSDisk | Si se debería cambiar el tamaño de la partición del sistema operativo para ocupar el VHD del sistema operativo completo antes de dividir el volumen del sistema. |
| ubicación | Ubicación para todos los recursos. |

## <a name="enable-encryption-on-nvme-disks-for-lsv2-vms"></a>Habilitación del cifrado en discos de NVMe para VM Lsv2

En este escenario se describe cómo habilitar Azure Disk Encryption en discos NVMe para VM de la serie Lsv2.  La serie Lsv2 incluye almacenamiento local NVMe. Los discos NVMe locales son efímeros y los datos se perderán en estos discos si se detiene o desasigna la VM (Consulte: [Serie Lsv2](../lsv2-series.md)).

Para habilitar el cifrado en discos NVMe:

1. Inicialice los discos NVMe y cree volúmenes NTFS.
1. Habilite el cifrado en la VM con el parámetro VolumeType establecido en All. Esto habilitará el cifrado para todos los discos de sistemas operativos y datos, incluidos los volúmenes respaldados por discos NVMe. Para más información, consulte [Habilitación del cifrado en una VM Windows existente o en ejecución](#enable-encryption-on-an-existing-or-running-windows-vm).

El cifrado se conservará en los discos NVMe en los escenarios siguientes:
- Reinicio de máquina virtual
- Restablecimiento de imagen inicial de VMSS
- Cambio de sistema operativo

Se anulará la inicialización de los discos NVMe en los siguientes escenarios:

- Inicio de la VM después de la desasignación
- Recuperación del servicio
- Copia de seguridad

En estos escenarios, los discos NVMe deben inicializarse después de que se inicie la VM. Para habilitar el cifrado en los discos NVMe, ejecute el comando para volver a habilitar Azure Disk Encryption después de inicializar los discos NVMe.

Además de los escenarios que aparecen en la sección [Escenarios no admitidos](#unsupported-scenarios), no se admite el cifrado de discos NVMe para:

- VM cifradas con Azure Disk Encryption con AAD (versión anterior)
- Discos NVMe con espacios de almacenamiento
- Azure Site Recovery de SKU con discos NVMe (consulte [Matriz de compatibilidad para la recuperación ante desastres de VM de Azure entre regiones de Azure: Máquinas replicadas: almacenamiento](../../site-recovery/azure-to-azure-support-matrix.md#replicated-machines---storage)).

## <a name="new-iaas-vms-created-from-customer-encrypted-vhd-and-encryption-keys"></a>Máquinas virtuales IaaS creadas a partir de discos duros virtuales cifrados por el cliente y claves de cifrado

En este escenario, puede crear una máquina virtual a partir de un disco duro virtual previamente cifrado y las claves de cifrado asociadas mediante cmdlets de PowerShell o comandos de la CLI. 

Siga las instrucciones que encontrará en [Preparación de un disco duro virtual previamente cifrado](disk-encryption-sample-scripts.md#prepare-a-pre-encrypted-windows-vhd). Una vez creada la imagen, puede seguir los pasos de la sección siguiente para crear una VM cifrada de Azure.


### <a name="encrypt-vms-with-pre-encrypted-vhds-with-azure-powershell"></a>Cifrado de máquinas virtuales con discos duros virtuales previamente cifrados con Azure PowerShell
Puede habilitar el cifrado de disco en el disco duro virtual cifrado mediante el cmdlet [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk#examples) de PowerShell. En el ejemplo siguiente se proporcionan algunos parámetros comunes. 

```azurepowershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myKVresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>Habilitación del cifrado en un disco de datos recién agregado
También puede [agregar un nuevo disco a una máquina virtual Windows con PowerShell](attach-disk-ps.md) o [mediante Azure Portal](attach-managed-disk-portal.md). 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Habilitación del cifrado en un disco recién agregado con Azure PowerShell
 Si se usa PowerShell para cifrar un nuevo disco para máquinas virtuales Windows, se debe especificar una nueva versión de la secuencia. La versión de secuencia debe ser única. El siguiente script genera un GUID para la versión de secuencia. En algunos casos, un disco de datos recién agregado se podría cifrar automáticamente mediante la extensión Azure Disk Encryption. El cifrado automático suele producirse cuando se reinicia la máquina virtual después de que el nuevo disco se pone en línea. Esto se suele producir porque se especificó "All" para el tipo de volumen cuando el cifrado de disco se ejecutó anteriormente en la máquina virtual. Si el cifrado automático se produce en un disco de datos recién agregado, se recomienda volver a ejecutar el cmdlet Set-AzVmDiskEncryptionExtension con la nueva versión de secuencia. Si el nuevo disco de datos es de cifrado automático y no desea cifrarlo, descifre todas las unidades en primer lugar y, a continuación, vuelva a cifrarlas con una nueva versión de secuencia que especifique el sistema operativo para el tipo de volumen. 
  
 

-  **Cifrado de una máquina virtual en ejecución:** el siguiente script inicializa las variables y ejecuta el cmdlet Set-AzVMDiskEncryptionExtension. Ya se deben haber satisfecho los requisitos previos de crear el grupo de recursos, la máquina virtual y el almacén de claves. Reemplace MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM y MySecureVault por los valores deseados. Este ejemplo utiliza "All" para el parámetro -VolumeType, que incluye los volúmenes de sistema operativo y de datos. Si solo quiere cifrar el volumen del sistema operativo, use "OS" para el parámetro VolumeType. 

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $sequenceVersion = [Guid]::NewGuid();

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "All" –SequenceVersion $sequenceVersion;
    ```
- **Cifrado de una máquina virtual en ejecución mediante KEK:** Este ejemplo utiliza "All" para el parámetro -VolumeType, que incluye los volúmenes de sistema operativo y de datos. Si solo quiere cifrar el volumen del sistema operativo, use "OS" para el parámetro VolumeType.

     ```azurepowershell
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup';
     $vmName = 'MyExtraSecureVM';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
     $sequenceVersion = [Guid]::NewGuid();

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "All" –SequenceVersion $sequenceVersion;

     ```

    >[!NOTE]
    > La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-cli"></a>Habilitación del cifrado en un disco recién agregado con la CLI de Azure
 El comando de la CLI de Azure proporciona automáticamente una nueva versión de secuencia al ejecutarlo para habilitar el cifrado. El ejemplo usa "All" para el parámetro VolumeType. Es posible que deba cambiar el parámetro VolumeType para el sistema operativo si solo está cifrando el disco del sistema operativo. A diferencia de la sintaxis de PowerShell, la CLI no requiere que el usuario especifique ninguna versión de secuencia única cuando se habilita el cifrado. La CLI genera y usa su propio valor de versión de secuencia único automáticamente.   

-  **Cifrado de una máquina virtual en ejecución:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "All"
     ```

- **Cifrado de una máquina virtual en ejecución mediante KEK:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "All"
     ```


## <a name="disable-encryption"></a>Deshabilitación del cifrado
[!INCLUDE [disk-encryption-disable-encryption-powershell](../../../includes/disk-encryption-disable-powershell.md)]

## <a name="unsupported-scenarios"></a>Escenarios no admitidos

Azure Disk Encryption no funciona para los siguientes escenarios, características ni tecnologías:

- VM de cifrado de nivel básico o VM creadas a través del método de creación de VM clásico.
- VM de cifrado configuradas con sistemas RAID basados en software.
- VM de cifrado configuradas con espacios de almacenamiento directo (S2D), o bien versiones de Windows Server anteriores a 2016 configuradas con espacios de almacenamiento de Windows.
- Integración con un sistema de administración de claves local.
- Azure Files (sistema de archivos compartido).
- Network File System (NFS).
- Volúmenes dinámicos.
- Contenedores de Windows Server, que crean volúmenes dinámicos para cada contenedor.
- Discos de sistema operativo efímero.
- Cifrado de sistemas de archivos compartidos o distribuidos como DFS, GFS, DRDB, and CephFS (sin limitarse solo a estos).
- Traslado de unas máquinas virtuales cifradas a otra suscripción o región.
- Creación de una imagen o instantánea de una máquina virtual cifrada y su uso para implementar máquinas virtuales adicionales.
- Máquinas virtuales de Gen2 (consulte: [Compatibilidad con máquinas virtuales de generación 2 en Azure](generation-2.md#generation-1-vs-generation-2-capabilities))
- Máquinas virtuales de la serie M con discos de Acelerador de escritura.
- Aplicación de ADE a una VM que tiene un disco de datos cifrado con el [cifrado del lado del servidor con claves administradas por el cliente](disk-encryption.md) (SSE + CMK) o la aplicación de SSE + CMK a un disco de datos en una VM cifrada con ADE.
- Migración de una máquina virtual cifrada con ADE a [cifrado del lado servidor con claves administradas por el cliente](disk-encryption.md).


## <a name="next-steps"></a>Pasos siguientes

- [Introducción a Azure Disk Encryption](disk-encryption-overview.md)
- [Scripts de ejemplo de Azure Disk Encryption](disk-encryption-sample-scripts.md)
- [Solución de problemas de Azure Disk Encryption](disk-encryption-troubleshooting.md)
