---
title: Escenarios de Azure Disk Encryption en máquinas virtuales Linux
description: En este artículo se proporcionan las instrucciones necesarias para habilitar Microsoft Azure Disk Encryption para máquinas virtuales Linux en varios escenarios
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: b7d19d782e9cb29cfc917293f084b78830db89bc
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797622"
---
# <a name="azure-disk-encryption-scenarios-on-linux-vms"></a>Escenarios de Azure Disk Encryption en máquinas virtuales Linux


Azure Disk Encryption para máquinas virtuales Linux usa la característica DM-Crypt de Linux para proporcionar un cifrado completo tanto del disco del sistema operativo como de los discos de datos. Además, proporciona cifrado del disco temporal cuando se usa la característica EncryptFormatAll.

Azure Disk Encryption [se integra con Azure Key Vault](disk-encryption-key-vault.md) para ayudarle a controlar y administrar las claves y los secreto de cifrado de discos. Para obtener información general del servicio, consulte [Azure Disk Encryption para máquinas virtuales Linux](disk-encryption-overview.md).

El cifrado de disco solo se puede aplicar a las máquinas virtuales que tengan [tamaños y sistemas operativos compatibles](disk-encryption-overview.md#supported-vms-and-operating-systems). También es preciso que se cumplan los siguientes requisitos previos:

- [Requisitos adicionales de las máquinas virtuales](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Requisitos de red](disk-encryption-overview.md#networking-requirements)
- [Requisitos de almacenamiento de la clave de cifrado](disk-encryption-overview.md#encryption-key-storage-requirements)

En todos los casos es preciso [hacer una instantánea](snapshot-copy-managed-disk.md) o crear una copia de seguridad antes de cifrar los discos. Las copias de seguridad garantizan que, en caso de un error inesperado durante el cifrado, sea posible una opción de recuperación. Las máquinas virtuales con discos administrados requieren una copia de seguridad antes del cifrado. Una vez realizada la copia de seguridad, puede usar el [cmdlet Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) para cifrar los discos administrados mediante la especificación del parámetro -skipVmBackup. Para más información sobre cómo realizar la copia de seguridad y restauración de máquinas virtuales cifradas, consulte el artículo sobre [Azure Backup](../../backup/backup-azure-vms-encryption.md). 

>[!WARNING]
> - Si ya ha usado Azure Disk Encryption con Azure AD para cifrar una VM, debe seguir usando esta opción para cifrar la VM. Para más información, consulte [Azure Disk Encryption con Azure AD (versión anterior)](disk-encryption-overview-aad.md). 
>
> - Al cifrar los volúmenes del sistema operativo Linux, la máquina virtual se debe considerar como no disponible. Se recomienda encarecidamente evitar los inicios de sesión de SSH mientras el cifrado está en curso para evitar que se bloqueen los archivos abiertos a los que se debe tener acceso durante el proceso de cifrado. Para comprobar el progreso, use el cmdlet [Get-AzVMDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus) de PowerShell o el comando [vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show) de la CLI. Este proceso puede tardar unas horas si trabaja con un volumen de sistema operativo de 30 GB; además, deberá tener en cuenta el tiempo necesario para realizar el cifrado de los volúmenes de datos. El tiempo de cifrado del volumen de datos será proporcional al tamaño y la cantidad de los volúmenes de datos, a menos que se use la opción "all" del formato de cifrado. 
> - Solo se puede deshabilitar el cifrado en máquinas virtuales Linux para volúmenes de datos. No se admite en volúmenes de datos o volúmenes del sistema operativo si el volumen del sistema operativo se ha cifrado.  

## <a name="install-tools-and-connect-to-azure"></a>Instalación de herramientas y conexión a Azure

Azure Disk Encryption se puede habilitar y administrar mediante la [CLI de Azure](/cli/azure) y [Azure PowerShell](/powershell/azure/new-azureps-module-az). Para ello, es preciso instalar las herramientas de forma local y conectarse a su suscripción de Azure.

### <a name="azure-cli"></a>Azure CLI

La [CLI de Azure 2.0](/cli/azure) es una herramienta de línea de comandos para administrar recursos de Azure. La CLI está diseñada para flexibilizar los datos de consulta, admitir operaciones de larga duración (como los procesos sin bloqueo) y facilitar la realización de scripts. Para instalarla localmente, siga los pasos que puede encontrar en [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

 

Para [iniciar sesión en su cuenta de Azure con la CLI de Azure](/cli/azure/authenticate-azure-cli), use el comando [az login](/cli/azure/reference-index?view=azure-cli-latest#az-login).

```azurecli
az login
```

Si quiere seleccionar un inquilino con el que iniciar sesión, use:
    
```azurecli
az login --tenant <tenant>
```

Si tiene varias suscripciones y quiere especificar una en concreto, use [az account list](/cli/azure/account#az-account-list) para obtener la lista de suscripciones y [az account set](/cli/azure/account#az-account-set) para especificar cuál.
     
```azurecli
az account list
az account set --subscription "<subscription name or ID>"
```

Para más información, consulte [Service Fabric y CLI de Azure 2.0](/cli/azure/get-started-with-azure-cli). 

### <a name="azure-powershell"></a>Azure PowerShell
El [módulo az de Azure PowerShell](/powershell/azure/new-azureps-module-az) ofrece un conjunto de cmdlets que usan el modelo de [Azure Resource Manager](../../azure-resource-manager/management/overview.md) para administrar los recursos de Azure. Se puede usar en el explorador con [Azure Cloud Shell](../../cloud-shell/overview.md), o bien se puede instalar en una máquina local, para lo que hay que seguir las instrucciones de [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). 

Si ya lo tiene instalado localmente, asegúrese de que usa la versión más reciente de la versión del SDK de Azure PowerShell para configurar Azure Disk Encryption. Descargue la versión más reciente de [Azure PowerShell](https://github.com/Azure/azure-powershell/releases).

Para [iniciar sesión en una cuenta de Azure con Azure PowerShell](/powershell/azure/authenticate-azureps?view=azps-2.5.0), utilice el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0).

```powershell
Connect-AzAccount
```

Si tiene varias suscripciones y desea especificar una de ellas, use el cmdlet [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription) para enumerarlas y, después, el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext?view=azps-2.5.0):

```powershell
Set-AzContext -Subscription -Subscription <SubscriptionId>
```

La ejecución del cmdlet [Get-AzContext](/powershell/module/Az.Accounts/Get-AzContext) verificará que se ha seleccionado la suscripción correcta.

Para confirmar que están instalados los cmdlets de Azure Disk Encryption, use el cmdlet [Get-command](/powershell/module/microsoft.powershell.core/get-command?view=powershell-6):
     
```powershell
Get-command *diskencryption*
```
Para más información, consulte [Introducción a los cmdlets de Azure PowerShell](/powershell/azure/get-started-azureps). 

## <a name="enable-encryption-on-an-existing-or-running-linux-vm"></a>Habilitación del cifrado en una máquina virtual Linux existente o en ejecución
En este escenario, puede habilitar el cifrado mediante la plantilla de Resource Manager, los cmdlets de PowerShell o los comandos de la CLI. Si necesita información de esquema para la extensión de máquina virtual, consulte el artículo [Extensión de Azure Disk Encryption para Linux](../extensions/azure-disk-enc-linux.md).

>[!IMPORTANT]
 >Es obligatorio crear una instantánea o una copia de seguridad de una instancia de máquina virtual basada en un disco administrado fuera de Azure Disk Encryption y antes de habilitar esta característica. Se puede tomar una instantánea del disco administrado desde el portal o mediante [Azure Backup](../../backup/backup-azure-vms-encryption.md). Las copias de seguridad garantizan que es posible disponer de una opción de recuperación en el caso de que se produzca un error inesperado durante el cifrado. Una vez que se realiza una copia de seguridad, el cmdlet Set-AzVMDiskEncryptionExtension puede usarse para cifrar los discos administrados mediante la especificación del parámetro -skipVmBackup. El comando Set-AzVMDiskEncryptionExtension produce un error en las máquinas virtuales basadas en un disco administrado hasta que se realice una copia de seguridad y se especifique este parámetro. 
>
>Cifrar o deshabilitar el cifrado puede provocar el reinicio de la máquina virtual. 
>

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-using-azure-cli"></a>Habilitación del cifrado en una máquina virtual Linux existente o en ejecución mediante la CLI de Azure 

Parr habilitar el cifrado de disco en un disco duro virtual cifrado, instale y use la herramienta de línea de comandos [CLI de Azure](/cli/azure/?view=azure-cli-latest). Se puede usar en el explorador con [Azure Cloud Shell](../../cloud-shell/overview.md), o lo puede instalar en el equipo local y usarlo en cualquier sesión de PowerShell. Para habilitar el cifrado en máquinas virtuales Linux existentes o en ejecución en Azure, use los siguientes comandos de la CLI:

Use el comando [az vm encryption enable](/cli/azure/vm/encryption?view=azure-cli-latest#az-vm-encryption-show) para habilitar el cifrado en una máquina virtual en ejecución en Azure.

- **Cifrado de una máquina virtual en ejecución:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
     ```

- **Cifrado de una máquina virtual en ejecución mediante KEK:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

    >[!NOTE]
    > La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br>
La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

- **Comprobar que los discos están cifrados:** Para comprobar el estado de cifrado de una máquina virtual, use el comando [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

- **Deshabilitar el cifrado:** para deshabilitar el cifrado, use el comando [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). La deshabilitación del cifrado solo se permite en volúmenes de datos de máquinas virtuales Linux.

     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type "data"
     ```

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-using-powershell"></a>Habilitación del cifrado en una máquina virtual Linux existente o en ejecución mediante PowerShell
Use el cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) para habilitar el cifrado en una máquina virtual en ejecución en Azure. Tome una [instantánea](snapshot-copy-managed-disk.md) o realice una copia de seguridad de la máquina virtual con [Azure Backup](../../backup/backup-azure-vms-encryption.md) antes de cifrar los discos. El parámetro - skipVmBackup ya está especificado en los scripts de PowerShell para cifrar una máquina virtual que ejecuta Linux.

-  **Cifrado de una máquina virtual en ejecución:** el siguiente script inicializa las variables y ejecuta el cmdlet Set-AzVMDiskEncryptionExtension. El grupo de recursos, la máquina virtual y el almacén de claves se han creado como requisitos previos. Reemplace MyVirtualMachineResourceGroup, MySecureVM y MySecureVault por los valores deseados. Modifique el parámetro -VolumeType para especificar qué discos está cifrando.

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $sequenceVersion = [Guid]::NewGuid();  

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```
- **Cifrado de una máquina virtual en ejecución mediante KEK:** Es posible que deba agregar el parámetro -VolumeType si va a cifrar discos de datos y no el disco del sistema operativo. 

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

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```

    >[!NOTE]
    > La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 
    
- **Comprobar que los discos están cifrados:** Para comprobar el estado de cifrado de una máquina virtual, use el cmdlet [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus). 
    
     ```azurepowershell-interactive 
     Get-AzVmDiskEncryptionStatus -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```
    
- **Deshabilitar el cifrado de disco:** para deshabilitar el cifrado, use el cmdlet [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). La deshabilitación del cifrado solo se permite en volúmenes de datos de máquinas virtuales Linux.
     
     ```azurepowershell-interactive 
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-with-a-template"></a>Habilitación del cifrado en una máquina virtual Linux existente o en ejecución con una plantilla

Para habilitar el cifrado de disco en una máquina virtual Linux existente o en ejecución en Azure, utilice la [plantilla de Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm-without-aad).

1. En la plantilla de inicio rápido de Azure, haga clic en **Deploy to Azure** (Implementar en Azure).

2. Seleccione la suscripción, el grupo de recursos, la ubicación del grupo de recursos, los parámetros, los términos legales y el contrato. Haga clic en **Crear** para habilitar el cifrado en la máquina virtual existente o en ejecución.

En la tabla siguiente figuran los parámetros de la plantilla de Resource Manager para máquinas virtuales existentes o en ejecución:

| Parámetro | Descripción |
| --- | --- |
| vmName | Nombre de la máquina virtual para ejecutar la operación de cifrado. |
| keyVaultName | Nombre del almacén de claves en el que se debe cargar la clave de cifrado. Para obtenerlo, use el cmdlet `(Get-AzKeyVault -ResourceGroupName <MyKeyVaultResourceGroupName>). Vaultname` o el comando de la CLI de Azure `az keyvault list --resource-group "MyKeyVaultResourceGroupName"`.|
| keyVaultResourceGroup | Nombre del grupo de recursos que contiene el almacén de claves. |
|  keyEncryptionKeyURL | Dirección URL de la clave de cifrado de claves que se usa para cifrar la clave de cifrado. Este parámetro es opcional si selecciona **nokek** en la lista desplegable de UseExistingKek. Si selecciona **kek** en la lista desplegable de UseExistingKek, debe proporcionar el valor de _keyEncryptionKeyURL_. |
| volumeType | Tipo de volumen en que se realiza la operación de cifrado. Los valores válidos son _SO_, _Datos_ y _Todo_. 
| forceUpdateTag | Cada vez que la operación tenga que ejecutarse, pase un valor único como GUID. |
| ubicación | Ubicación para todos los recursos. |

Para más información sobre la configuración de la plantilla de cifrado de discos de máquina virtual, consulte [Azure Disk Encryption para Linux](../extensions/azure-disk-enc-linux.md).

## <a name="use-encryptformatall-feature-for-data-disks-on-linux-vms"></a>Uso de la característica EncryptFormatAll para discos de datos en máquinas virtuales Linux

El parámetro **EncryptFormatAll** reduce el tiempo que se tardan en cifrar los discos de datos de Linux. Se formatearán las particiones que cumplan ciertos criterios, junto con sus sistemas de archivos actuales, y se volverán a montar en el lugar en que estaban antes de la ejecución del comando. Si quiere excluir un disco de datos que cumple los criterios, puede desmontarlo antes de ejecutar el comando.

 Después de ejecutar este comando, se dará formato a todas las unidades que se montaron previamente y el nivel de cifrado se iniciará encima de la unidad vacía. Cuando se selecciona esta opción, también se cifrará el disco temporal asociado a la máquina virtual. Si el disco temporal se restablece, la solución Azure Disk Encryption lo vuelve a formatear y cifrar para la máquina virtual en la siguiente oportunidad. Una vez que el disco de recursos se cifra, el [agente de Linux de Microsoft Azure](../extensions/agent-linux.md) no podrá administrarlo ni habilitar el archivo de intercambio, pero puede configurar este último manualmente.

>[!WARNING]
> EncryptFormatAll no debe usarse cuando hay datos necesarios en los volúmenes de datos de una máquina virtual. Para excluir discos del cifrado, puede desmontarlos. Debe probar primero EncryptFormatAll en una máquina virtual de prueba y comprender el parámetro de característica y su implicación antes de intentar usarlo en la máquina virtual de producción. La opción EncryptFormatAll formatea el disco de datos, y todos los datos que contiene se pierden. Antes de continuar, compruebe que los discos que quiere excluir estén desmontados correctamente. </br></br>
 >Si configura este parámetro mientras actualiza la configuración de cifrado, podría provocar un reinicio antes del cifrado real. En este caso, también querrá quitar el disco que no quiere que se formatee del archivo fstab. De igual forma, debe agregar la partición que quiere cifrar formateada al archivo fstab antes de iniciar la operación de cifrado. 

### <a name="encryptformatall-criteria"></a>Criterios de EncryptFormatAll
El parámetro recorre todas las particiones y las cifra siempre que cumplan **todos** los siguientes criterios:
- No es una partición raíz, de arranque o del sistema operativo
- Ya no está cifrada
- No es un volumen BEK
- No es un volumen RAID
- No es un volumen LVM
- Está montada

Cifre los discos que componen el volumen LVM o RAID en lugar de los volúmenes LVM o RAID.

### <a name="use-the-encryptformatall-parameter-with-azure-cli"></a>Uso del parámetro EncryptFormatAll con la CLI de Azure
Use el comando [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) para habilitar el cifrado en una máquina virtual en ejecución en Azure.

-  **Cifrar una máquina virtual en ejecución mediante EncryptFormatAll:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "data" --encrypt-format-all
     ```

### <a name="use-the-encryptformatall-parameter-with-a-powershell-cmdlet"></a>Uso del parámetro EncryptFormatAll con un cmdlet de PowerShell
Use el cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) con el parámetro EncryptFormatAll. 

**Cifrar una máquina virtual en ejecución mediante EncryptFormatAll:** como ejemplo, el siguiente script inicializa las variables y ejecuta el cmdlet Set-AzVMDiskEncryptionExtension con el parámetro EncryptFormatAll. El grupo de recursos, la máquina virtual y el almacén de claves se han creado como requisitos previos. Reemplace MyVirtualMachineResourceGroup, MySecureVM y MySecureVault por los valores deseados.
  
```azurepowershell
$KVRGname = 'MyKeyVaultResourceGroup';
$VMRGName = 'MyVirtualMachineResourceGroup';
$vmName = 'MySecureVM';
$KeyVaultName = 'MySecureVault';
$KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;

Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "data" -EncryptFormatAll
```


### <a name="use-the-encryptformatall-parameter-with-logical-volume-manager-lvm"></a>Uso del parámetro EncryptFormatAll con el Administrador de volúmenes lógicos (LVM) 
Se recomienda una configuración LVM-on-crypt. En todos los ejemplos siguientes, reemplace la ruta de acceso de dispositivo y los puntos de montaje por lo que se adapte a su caso de uso. Esta configuración se puede realizar de la manera siguiente:

1.  Agregue los discos de datos que componen la máquina virtual.

1. Formatee, monte y agregue estos discos al archivo fstab.

1. Elija una partición estándar, cree una partición que abarque toda la unidad y, a continuación, formatee la partición. Aquí se usarán symlinks generados por Azure. El uso de symlinks evita los problemas relacionados con el cambio de los nombres de dispositivo. Para más información, consulte el artículo [Solución de problemas de nombres de dispositivo](../troubleshooting/troubleshoot-device-names-problems.md).
    
    ```bash
    parted /dev/disk/azure/scsi1/lun0 mklabel gpt
    parted -a opt /dev/disk/azure/scsi1/lun0 mkpart primary ext4 0% 100%
    
    mkfs -t ext4 /dev/disk/azure/scsi1/lun0-part1
    ```

1. Monte los discos:

    ```bash
    mount /dev/disk/azure/scsi1/lun0-part1 /mnt/mountpoint
    ````
    
    Agréguelos al archivo fstab:

    ```bash
    echo "/dev/disk/azure/scsi1/lun0-part1 /mnt/mountpoint ext4 defaults,nofail 0 2" >> /etc/fstab
    ```
    
1. Ejecute el cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension?view=azps-3.8.0) con -EncryptFormatAll para cifrar estos discos.

    ```azurepowershell-interactive
    $KeyVault = Get-AzKeyVault -VaultName "MySecureVault" -ResourceGroupName "MySecureGroup"
    
    Set-AzVMDiskEncryptionExtension -ResourceGroupName "MySecureGroup" -VMName "MySecureVM" -DiskEncryptionKeyVaultUrl $KeyVault.VaultUri -DiskEncryptionKeyVaultId $KeyVault.ResourceId -EncryptFormatAll -SkipVmBackup -VolumeType Data
    ```

    Si quiere usar una clave de cifrado de claves (KEK), pase el URI de su KEK y el ResourceID de su almacén de claves a los parámetros -KeyEncryptionKeyUrl y -KeyEncryptionKeyVaultId, respectivamente:

    ```azurepowershell-interactive
    $KeyVault = Get-AzKeyVault -VaultName "MySecureVault" -ResourceGroupName "MySecureGroup"
    $KEKKeyVault = Get-AzKeyVault -VaultName "MyKEKVault" -ResourceGroupName "MySecureGroup"
    $KEK = Get-AzKeyVaultKey -VaultName "myKEKVault" -KeyName "myKEKName"
    
    Set-AzVMDiskEncryptionExtension -ResourceGroupName "MySecureGroup" -VMName "MySecureVM" -DiskEncryptionKeyVaultUrl $KeyVault.VaultUri -DiskEncryptionKeyVaultId $KeyVault.ResourceId -EncryptFormatAll -SkipVmBackup -VolumeType Data -KeyEncryptionKeyUrl $$KEK.id -KeyEncryptionKeyVaultId $KEKKeyVault.ResourceId
    ```

1. Configure LVM encima de estos nuevos discos. Tenga en cuenta que las unidades cifradas se desbloquean después de que la máquina virtual ha terminado de arrancar. Por lo tanto, el montaje de LVM también tendrá que retrasarse.

## <a name="new-vms-created-from-customer-encrypted-vhd-and-encryption-keys"></a>Nuevas máquinas virtuales creadas a partir de discos duros virtuales cifrados por el cliente y claves de cifrado
En este escenario, puede habilitar el cifrado mediante los cmdlets de PowerShell o los comandos de la CLI. 

Use las instrucciones de los mismos scripts de Azure Disk Encryption para preparar imágenes previamente cifradas que se pueden usar en Azure. Una vez creada la imagen, puede seguir los pasos de la sección siguiente para crear una VM cifrada de Azure.

* [Preparación de un VHD con Linux precifrado](disk-encryption-sample-scripts.md#prepare-a-pre-encrypted-linux-vhd)

>[!IMPORTANT]
 >Es obligatorio crear una instantánea o una copia de seguridad de una instancia de máquina virtual basada en un disco administrado fuera de Azure Disk Encryption y antes de habilitar esta característica. Puede tomar una instantánea del disco administrado desde el portal o se puede usar [Azure Backup](../../backup/backup-azure-vms-encryption.md). Las copias de seguridad garantizan que es posible disponer de una opción de recuperación en el caso de que se produzca un error inesperado durante el cifrado. Una vez que se realiza una copia de seguridad, el cmdlet Set-AzVMDiskEncryptionExtension puede usarse para cifrar los discos administrados mediante la especificación del parámetro -skipVmBackup. El comando Set-AzVMDiskEncryptionExtension produce un error en las máquinas virtuales basadas en un disco administrado hasta que se realice una copia de seguridad y se especifique este parámetro. 
>
> Cifrar o deshabilitar el cifrado puede provocar el reinicio de la máquina virtual. 



### <a name="use-azure-powershell-to-encrypt-vms-with-pre-encrypted-vhds"></a>Uso de Azure PowerShell para cifrar máquinas virtuales con discos duros virtuales previamente cifrados 
Puede habilitar el cifrado de disco en el disco duro virtual cifrado mediante el cmdlet [Set-AzVMOSDisk](/powershell/module/Az.Compute/Set-AzVMOSDisk#examples) de PowerShell. En el ejemplo siguiente se proporcionan algunos parámetros comunes. 

```azurepowershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Linux -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>Habilitación del cifrado en un disco de datos recién agregado

Puede agregar un nuevo disco de datos mediante [az vm disk attach](add-disk.md), o [Azure Portal](attach-disk-portal.md). Para poder cifrarlo, deberá montar primero el disco de datos recién asociado. Dado que la unidad de datos no se puede usar mientras el cifrado está en curso, deberá solicitar el cifrado de la unidad. 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-cli"></a>Habilitación del cifrado en un disco recién agregado con la CLI de Azure

 Si la máquina virtual se cifró previamente con "All", el parámetro --volume-type debe permanecer como "All". All incluye los discos de datos y del SO. Si la máquina virtual se cifró previamente con el tipo de volumen "OS", el parámetro--volume-type se debe cambiar a "All", con el fin de que se incluyan tanto el nuevo disco de datos como el sistema operativo. Si la VM se cifró con el tipo de volumen de "Data", entonces puede permanecer como "Data", tal y como se muestra a continuación. Para preparar el cifrado, no es suficiente con agregar y asociar un nuevo disco de datos a una VM. El disco recién conectado también se debe formatear y montar correctamente en la VM antes de habilitar el cifrado. En Linux, el disco se debe montar en /etc/fstab con un [nombre de dispositivo de bloque persistente](../troubleshooting/troubleshoot-device-names-problems.md).  

A diferencia de la sintaxis de PowerShell, la CLI no requiere que el usuario proporcione ninguna versión de secuencia única cuando se habilita el cifrado. La CLI genera y usa su propio valor de versión de secuencia único automáticamente.

-  **Cifrar los volúmenes de datos de una VM en ejecución:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "Data"
     ```

- **Cifrar los volúmenes de datos de una VM con KEK:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "Data"
     ```

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Habilitación del cifrado en un disco recién agregado con Azure PowerShell
 Al usar PowerShell para cifrar un nuevo disco para Linux, se debe especificar una nueva versión de secuencia. La versión de secuencia debe ser única. El siguiente script genera un GUID para la versión de secuencia. Tome una [instantánea](snapshot-copy-managed-disk.md) o realice una copia de seguridad de la máquina virtual con [Azure Backup](../../backup/backup-azure-vms-encryption.md) antes de cifrar los discos. El parámetro - skipVmBackup ya está especificado en los scripts de PowerShell para cifrar un disco de datos recién agregado.
 

-  **Cifrar los volúmenes de datos de una VM en ejecución:** el siguiente script inicializa las variables y ejecuta el cmdlet Set-AzVMDiskEncryptionExtension. Ya se deben haber satisfecho los requisitos previos de crear el grupo de recursos, la máquina virtual y el almacén de claves. Reemplace MyVirtualMachineResourceGroup, MySecureVM y MySecureVault por los valores deseados. Los valores aceptables para el parámetro -VolumeType son All, OS y Data. Si la máquina virtual se cifró previamente con los tipos de volumen "OS" o "All", el parámetro --VolumeType se debe cambiar a "All", con el fin de que se incluyan tanto el nuevo disco de datos como el sistema operativo.

      ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $sequenceVersion = [Guid]::NewGuid();

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion -skipVmBackup;
      ```
- **Cifrar los volúmenes de datos de una VM con KEK:** Los valores aceptables para el parámetro -VolumeType son All, OS y Data. Si la VM se cifró previamente con un tipo de volumen de "OS" o "All", entonces el parámetro --VolumeType se debe cambiar a All, de modo que se incluyan el nuevo disco de datos y el SO.

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

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion -skipVmBackup;
     ```

    >[!NOTE]
    > La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[KVresource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 


## <a name="disable-encryption-for-linux-vms"></a>Deshabilitación del cifrado para máquinas virtuales Linux
[!INCLUDE [disk-encryption-disable-encryption-cli](../../../includes/disk-encryption-disable-cli.md)]

## <a name="unsupported-scenarios"></a>Escenarios no admitidos

Azure Disk Encryption no funciona en los siguientes escenarios, características y tecnologías de Linux:

- VM de cifrado de nivel básico o VM creadas a través del método de creación de VM clásico.
- Deshabilitación del cifrado en una unidad de SO o datos de una VM Linux cuando se cifra la unidad de SO.
- Cifrado de la unidad del sistema operativo para los conjuntos de escalado de máquinas virtuales Linux.
- Cifrado de imágenes personalizadas en VM Linux.
- Integración con un sistema de administración de claves local.
- Azure Files (sistema de archivos compartido).
- Network File System (NFS).
- Volúmenes dinámicos.
- Discos de sistema operativo efímero.
- Cifrado de sistemas de archivos compartidos o distribuidos como (pero no limitados a): DFS, GFS, DRDB y CephFS.
- Traslado de una máquina virtual cifrada a otra suscripción o región.
- Creación de una imagen o instantánea de una máquina virtual cifrada y su uso para implementar máquinas virtuales adicionales.
- Volcado de memoria de kernel (kdump).
- Oracle ACFS (ASM Cluster File System).
- Máquinas virtuales de Gen2 (consulte: [Compatibilidad con máquinas virtuales de generación 2 en Azure](generation-2.md#generation-1-vs-generation-2-capabilities)).
- Discos de NVMe de las máquinas virtuales de serie Lsv2 (consulte: [Serie Lsv2](../lsv2-series.md)).
- Una máquina virtual con "puntos de montaje anidados", es decir, varios puntos de montaje en una sola ruta de acceso (como "/1stmountpoint/data/2stmountpoint").
- Una máquina virtual con una unidad de datos montada en la parte superior de una carpeta de sistema operativo.
- Máquinas virtuales de la serie M con discos de Acelerador de escritura.
- Aplicación de ADE a una máquina virtual que tiene discos cifrados con el [cifrado del lado servidor con claves administradas por el cliente](disk-encryption.md) (SSE + CMK). La aplicación de SSE + CMK a un disco de datos en una máquina virtual cifrada con ADE tampoco es un escenario admitido.
- Migración de una máquina virtual cifrada con ADE, o que **alguna vez** haya estado cifrada con ADE, al [cifrado del lado servidor con claves administradas por el cliente](disk-encryption.md).

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a Azure Disk Encryption](disk-encryption-overview.md)
- [Scripts de ejemplo de Azure Disk Encryption](disk-encryption-sample-scripts.md)
- [Solución de problemas de Azure Disk Encryption](disk-encryption-troubleshooting.md)
