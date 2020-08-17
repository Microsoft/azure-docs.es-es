---
title: Azure Disk Encryption con máquinas virtuales IaaS de Linux de aplicación de Azure AD (versión anterior)
description: En este artículo se proporcionan instrucciones sobre cómo habilitar Microsoft Azure Disk Encryption para máquinas virtuales IaaS de Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 03/15/2019
ms.custom: seodec18
ms.openlocfilehash: fa01c4a595a08ffdba56d777128431946540eee5
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372678"
---
# <a name="enable-azure-disk-encryption-with-azure-ad-on-linux-vms-previous-release"></a>Habilitación de Azure Disk Encryption con Azure AD en máquinas virtuales Linux (versión anterior)

La nueva versión de Azure Disk Encryption elimina el requisito de tener que proporcionar un parámetro de aplicación de Azure Active Directory (Azure AD) para habilitar el cifrado de disco de la máquina virtual. Con la nueva versión, ya no es necesario proporcionar credenciales de Azure AD durante el paso de habilitar el cifrado. Todas las máquinas virtuales nuevas se deben cifrar sin los parámetros de aplicación de Azure AD con la nueva versión. Para ver las instrucciones sobre cómo habilitar el cifrado de disco de máquina virtual con la nueva versión, consulte [Azure Disk Encryption para máquinas virtuales Linux](disk-encryption-linux.md). Las máquinas virtuales que ya se han cifrado con parámetros de la aplicación de Azure AD siguen siendo compatibles y deben continuar manteniéndose con la sintaxis de AAD.

Puede habilitar varios escenarios de cifrado de disco y los pasos pueden variar según el escenario. En las secciones siguientes se tratan con más detalle los escenarios de máquinas virtuales de infraestructura como servicio (IaaS) de Linux. El cifrado de disco solo se puede aplicar a las máquinas virtuales que tengan [tamaños y sistemas operativos compatibles](disk-encryption-overview.md#supported-vms-and-operating-systems). También es preciso que se cumplan los siguientes requisitos previos:

- [Requisitos adicionales de las máquinas virtuales](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Redes y directiva de grupo](disk-encryption-overview-aad.md#networking-and-group-policy)
- [Requisitos de almacenamiento de la clave de cifrado](disk-encryption-overview-aad.md#encryption-key-storage-requirements)

Tome una [instantánea](snapshot-copy-managed-disk.md), realice una copia de seguridad o ambas cosas antes de cifrar los discos. Las copias de seguridad garantizan que, en caso de un error inesperado durante el cifrado, sea posible una opción de recuperación. Las máquinas virtuales con discos administrados requieren una copia de seguridad antes del cifrado. Después de hacer la copia de seguridad, puede usar el cmdlet Set-AzVMDiskEncryptionExtension para cifrar los discos administrados especificando el parámetro -skipVmBackup. Para más información sobre cómo realizar la copia de seguridad y restauración de máquinas virtuales cifradas, consulte [Azure Backup](../../backup/backup-azure-vms-encryption.md). 

>[!WARNING]
 > - Si ya ha usado [Azure Disk Encryption con la aplicación de Azure AD](disk-encryption-overview-aad.md) para cifrar una máquina virtual, debe seguir usando esta opción para cifrar la VM. No puede usar [Azure Disk Encryption](disk-encryption-overview.md) en esta máquina virtual cifrada, ya que no es un escenario compatible, lo que significa que aún no se admite la conmutación desde la aplicación de Azure AD a esta máquina virtual cifrada.
 > - Para garantizar que los secretos de cifrado no traspasen los límites regionales, Azure Disk Encryption necesita que el almacén de claves y las máquinas virtuales se ubiquen en la misma región. Cree y use un almacén de claves que se encuentre en la misma región que la máquina virtual que se va a cifrar.
 > - El proceso de cifrado de volúmenes del sistema operativo Linux puede tardar unas horas. Es normal que los volúmenes del sistema operativo Linux tarden más en cifrarse que los volúmenes de datos.
> - Al cifrar los volúmenes del sistema operativo Linux, la máquina virtual se debe considerar como no disponible. Se recomienda encarecidamente evitar los inicios de sesión de SSH mientras el cifrado está en curso para evitar que se bloqueen los archivos abiertos a los que se debe tener acceso durante el proceso de cifrado. Para comprobar el progreso, use los comandos [Get-AzVMDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus) o [vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). Este proceso puede tardar unas horas si trabaja con un volumen de sistema operativo de 30 GB; además, deberá tener en cuenta el tiempo necesario para realizar el cifrado de los volúmenes de datos. El tiempo de cifrado del volumen de datos será proporcional al tamaño y la cantidad de los volúmenes de datos, a menos que se use la opción **encrypt format all**. 
 > - Solo se puede deshabilitar el cifrado en máquinas virtuales Linux para volúmenes de datos. No se admite en volúmenes de datos o del sistema operativo si el volumen del sistema operativo se ha cifrado. 

 

## <a name="enable-encryption-on-an-existing-or-running-iaas-linux-vm"></a><a name="bkmk_RunningLinux"> </a> Habilitación del cifrado en una máquina virtual IaaS de Linux existente o en ejecución

En este escenario, puede habilitar el cifrado mediante la plantilla de Azure Resource Manager, los cmdlets de PowerShell o los comandos de la CLI. 

>[!IMPORTANT]
 >Es obligatorio crear una instantánea o una copia de seguridad de una instancia de máquina virtual basada en un disco administrado fuera de Azure Disk Encryption y antes de habilitar esta característica. Puede tomar una instantánea del disco administrado desde Azure Portal o puede usar [Azure Backup](../../backup/backup-azure-vms-encryption.md). Las copias de seguridad garantizan que es posible disponer de una opción de recuperación en el caso de que se produzca un error inesperado durante el cifrado. Después de hacer la copia de seguridad, use el cmdlet Set-AzVMDiskEncryptionExtension para cifrar los discos administrados mediante la especificación del parámetro -skipVmBackup. El comando Set-AzVMDiskEncryptionExtension no se ejecuta en las máquinas virtuales basadas en un disco administrado hasta que se haya hecho una copia de seguridad y se especifique este parámetro. 
>
>Cifrar o deshabilitar el cifrado puede provocar el reinicio de la máquina virtual. 
>

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-by-using-the-azure-cli"></a><a name="bkmk_RunningLinuxCLI"> </a>Habilitación del cifrado en una máquina virtual Linux existente o en ejecución mediante la CLI de Azure 
Puede habilitar el cifrado de disco en el disco duro virtual cifrado mediante la instalación y el uso de la herramienta de la línea de comandos [CLI de Azure 2.0](/cli/azure). Se puede usar en el explorador con [Azure Cloud Shell](../../cloud-shell/overview.md), o lo puede instalar en el equipo local y usarlo en cualquier sesión de PowerShell. Para habilitar el cifrado en máquinas virtuales IaaS de Linux existentes o en ejecución en Azure, use los siguientes comandos:

Use el comando [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) para habilitar el cifrado en una máquina virtual IaaS en ejecución en Azure.

-  **Cifrado de una máquina virtual en ejecución mediante un secreto de cliente:**
    
     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
     ```

- **Cifrado de una máquina virtual en ejecución mediante KEK para encapsular el secreto de cliente:**
    
     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

   >[!NOTE]
   > La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name].</br> </br> La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id].

- **Comprobación del cifrado de los discos:** para comprobar el estado de cifrado de una máquina virtual IaaS, use el comando [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). 

     ```azurecli-interactive
         az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

- **Deshabilitar el cifrado:** para deshabilitar el cifrado, use el comando [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). La deshabilitación del cifrado solo se permite en volúmenes de datos de máquinas virtuales Linux.
    
     ```azurecli-interactive
         az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type DATA
     ```

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-by-using-powershell"></a><a name="bkmk_RunningLinuxPSH"> </a> Habilitación del cifrado en una máquina virtual Linux existente o en ejecución mediante PowerShell
Use el cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) para habilitar el cifrado en una máquina virtual IaaS en ejecución en Azure. Tome una [instantánea](snapshot-copy-managed-disk.md) o realice una copia de seguridad de la máquina virtual con [Azure Backup](../../backup/backup-azure-vms-encryption.md) antes de cifrar los discos. El parámetro - skipVmBackup ya está especificado en los scripts de PowerShell para cifrar una máquina virtual que ejecuta Linux.

- **Cifrado de una máquina virtual en ejecución mediante un secreto de cliente:** el siguiente script inicializa las variables y ejecuta el cmdlet Set-AzVMDiskEncryptionExtension. El grupo de recursos, la máquina virtual, el almacén de claves, la aplicación de Azure AD y el secreto de cliente deben haberse creado ya como requisitos previos. Reemplace MyVirtualMachineResourceGroup, MyKeyVaultResourceGroup, MySecureVM, MySecureVault, My-AAD-client-ID y My-AAD-client-secret por sus propios valores. Modifique el parámetro -VolumeType para especificar qué discos está cifrando.

     ```azurepowershell
         $VMRGName = 'MyVirtualMachineResourceGroup';
         $KVRGname = 'MyKeyVaultResourceGroup';
         $vmName = 'MySecureVM';
         $aadClientID = 'My-AAD-client-ID';
         $aadClientSecret = 'My-AAD-client-secret';
         $KeyVaultName = 'MySecureVault';
         $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
         $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
         $KeyVaultResourceId = $KeyVault.ResourceId;
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```
- **Cifrado de una máquina virtual en ejecución mediante KEK para encapsular el secreto de cliente:** Azure Disk Encryption le permite especificar una clave existente en el almacén de claves para encapsular los secretos de cifrado de disco que se generaron al habilitar el cifrado. Cuando se especifica una clave de cifrado de claves, Azure Disk Encryption usa esa clave para encapsular los secretos de cifrado antes de escribirlos en el almacén de claves. Modifique el parámetro -VolumeType para especificar qué discos está cifrando. 

     ```azurepowershell
         $KVRGname = 'MyKeyVaultResourceGroup';
         $VMRGName = 'MyVirtualMachineResourceGroup';
         $aadClientID = 'My-AAD-client-ID';
         $aadClientSecret = 'My-AAD-client-secret';
         $KeyVaultName = 'MySecureVault';
         $keyEncryptionKeyName = 'MyKeyEncryptionKey';
         $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
         $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
         $KeyVaultResourceId = $KeyVault.ResourceId;
         $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```

  >[!NOTE]
  > La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[KVresource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name].</br> </br>
  La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id]. 
    
- **Comprobación del cifrado de los discos:** para comprobar el estado de cifrado de una máquina virtual de IaaS, use el cmdlet [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus). 
    
     ```azurepowershell-interactive 
         Get-AzVmDiskEncryptionStatus -ResourceGroupName MyVirtualMachineResourceGroup -VMName MySecureVM
     ```
    
- **Deshabilitar el cifrado de disco:** para deshabilitar el cifrado, use el cmdlet [Disable-AzureRmVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). La deshabilitación del cifrado solo se permite en volúmenes de datos de máquinas virtuales Linux.
     
     ```azurepowershell-interactive 
         Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```


### <a name="enable-encryption-on-an-existing-or-running-iaas-linux-vm-with-a-template"></a><a name="bkmk_RunningLinux"> </a> Habilitación del cifrado en una máquina virtual IaaS de Linux existente o en ejecución con una plantilla

Puede habilitar el cifrado de disco en una máquina virtual IaaS con Linux existente o en ejecución en Azure mediante la [plantilla de Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm).

1. En la plantilla de inicio rápido de Azure, seleccione **Deploy to Azure** (Implementar en Azure).

2. Seleccione la suscripción, el grupo de recursos, la ubicación del grupo de recursos, los parámetros, los términos legales y el contrato. Seleccione **Crear** para habilitar el cifrado en la máquina virtual IaaS existente o en ejecución.

En la tabla siguiente figuran los parámetros de la plantilla de Resource Manager para máquinas virtuales existentes o en ejecución que usan un identificador de cliente de Azure AD:

| Parámetro | Descripción |
| --- | --- |
| AADClientID | Identificador de cliente de la aplicación de Azure AD que tiene permisos para escribir secretos en el almacén de claves. |
| AADClientSecret | Secreto de cliente de la aplicación de Azure AD que tiene permisos para escribir secretos en el almacén de claves. |
| keyVaultName | Nombre del almacén de claves donde se debe cargar la clave. Puede obtenerlo mediante el comando `az keyvault show --name "MySecureVault" --query KVresourceGroup` de la CLI de Azure. |
|  keyEncryptionKeyURL | Dirección URL de la clave de cifrado de claves que se usa para cifrar la clave generada. Este parámetro es opcional si selecciona **nokek** en la lista desplegable de **UseExistingKek**. Si selecciona **kek** en la lista desplegable de **UseExistingKek**, debe proporcionar el valor de _keyEncryptionKeyURL_. |
| volumeType | Tipo de volumen en que se realiza la operación de cifrado. Los valores válidos admitidos son _OS_ o _All_. (Consulte las distribuciones de Linux admitidas y sus versiones para los discos de sistema operativo y de datos en la sección anterior de requisitos previos). |
| sequenceVersion | Versión de la secuencia de la operación de BitLocker. Incremente el número de versión cada vez que se realice una operación de cifrado de disco en la misma máquina virtual. |
| vmName | Nombre de la máquina virtual en que se va a realizar la operación de cifrado. |
| frase de contraseña | Escriba una frase de contraseña segura como clave de cifrado de datos. |



## <a name="use-the-encryptformatall-feature-for-data-disks-on-linux-iaas-vms"></a><a name="bkmk_EFA"> </a>Uso de la característica EncryptFormatAll para discos de datos en máquinas virtuales IaaS de Linux
El parámetro EncryptFormatAll reduce el tiempo que se tarda en cifrar los discos de datos de Linux. Las particiones que cumplen determinados criterios se formatean (con su actual sistema de archivos). Luego, se vuelven a montar donde estaban antes de la ejecución del comando. Si quiere excluir un disco de datos que cumple los criterios, puede desmontarlo antes de ejecutar el comando.

 Después de ejecutar este comando, se formatean las unidades que se han montado anteriormente. Luego se inicia la capa de cifrado sobre la unidad ahora vacía. Cuando se selecciona esta opción, también se cifra el disco temporal asociado a la máquina virtual. Si la unidad efímera se restablece, la solución Azure Disk Encryption la vuelve a formatear y a cifrar para la máquina virtual en la próxima oportunidad.

>[!WARNING]
> EncryptFormatAll no debe usarse cuando hay datos necesarios en los volúmenes de datos de una máquina virtual. Para excluir discos del cifrado, puede desmontarlos. Pruebe primero el parámetro EncryptFormatAll en una VM de prueba para entender el parámetro de característica y su implicación antes de probarlo en la VM de producción. La opción EncryptFormatAll formatea el disco de datos, por lo que todos los datos que contiene se pierden. Antes de continuar, compruebe que los discos que quiere excluir estén desmontados correctamente. </br></br>
 >Si establece este parámetro mientras actualiza la configuración de cifrado, podría provocar un reinicio antes del cifrado real. En este caso, también quiere quitar el disco que no quiere que se formatee a partir del archivo fstab. De igual forma, debe agregar la partición que quiere cifrar formateada al archivo fstab antes de iniciar la operación de cifrado. 

### <a name="encryptformatall-criteria"></a><a name="bkmk_EFACriteria"> </a>Criterios de EncryptFormatAll
El parámetro recorre todas las particiones y las cifra siempre que cumplan *todos* los criterios siguientes: 
- No es una partición raíz, de arranque o del sistema operativo
- Ya no está cifrada
- No es un volumen BEK
- No es un volumen RAID
- No es un volumen LVM
- Está montada

Cifre los discos que componen el volumen LVM o RAID en lugar de los volúmenes LVM o RAID.

### <a name="use-the-encryptformatall-parameter-with-a-template"></a><a name="bkmk_EFATemplate"> </a> Uso del parámetro EncryptFormatAll con una plantilla
Para usar la opción EncryptFormatAll, use cualquier plantilla de Azure Resource Manager ya existente que cifre una máquina virtual Linux y cambie el campo **EncryptionOperation** del recurso AzureDiskEncryption.

1. Por ejemplo, use la [plantilla de Resource Manager para cifrar una máquina virtual IaaS de Linux en ejecución](https://github.com/vermashi/azure-quickstart-templates/tree/encrypt-format-running-linux-vm/201-encrypt-running-linux-vm). 
2. En la plantilla de inicio rápido de Azure, seleccione **Deploy to Azure** (Implementar en Azure).
3. Cambie el campo **EncryptionOperation** de **EnableEncryption** a **EnableEncryptionFormatAl**.
4. Seleccione la suscripción, el grupo de recursos, la ubicación del grupo de recursos, otros parámetros, los términos legales y el contrato. Seleccione **Crear** para habilitar el cifrado en la máquina virtual IaaS existente o en ejecución.


### <a name="use-the-encryptformatall-parameter-with-a-powershell-cmdlet"></a><a name="bkmk_EFAPSH"> </a>Uso del parámetro EncryptFormatAll con un cmdlet de PowerShell
Use el cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) con el parámetro EncryptFormatAll.

**Cifrado de una máquina virtual en ejecución mediante un secreto de cliente y EncryptFormatAll:** por ejemplo, el siguiente script inicializa las variables y ejecuta el cmdlet Set-AzVMDiskEncryptionExtension con el parámetro EncryptFormatAll. El grupo de recursos, la máquina virtual, el almacén de claves, la aplicación de Azure AD y el secreto de cliente deben haberse creado ya como requisitos previos. Reemplace MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM, MySecureVault, My-AAD-client-ID y My-AAD-client-secret por sus propios valores.
  
   ```azurepowershell
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup'; 
     $aadClientID = 'My-AAD-client-ID';
     $aadClientSecret = 'My-AAD-client-secret';
     $KeyVaultName = 'MySecureVault';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
      
     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -EncryptFormatAll
   ```


### <a name="use-the-encryptformatall-parameter-with-logical-volume-manager-lvm"></a><a name="bkmk_EFALVM"> </a>Uso del parámetro EncryptFormatAll con el Administrador de volúmenes lógicos (LVM) 
Se recomienda una configuración LVM-on-crypt. En todos los ejemplos siguientes, reemplace la ruta de acceso de dispositivo y los puntos de montaje por lo que se adapte a su caso de uso. Esta configuración se puede realizar de la manera siguiente:

- Agregue los discos de datos que componen la máquina virtual.
- Formatee, monte y agregue estos discos al archivo fstab.

    1. Formatee el disco recién agregado. Aquí se usarán symlinks generados por Azure. El uso de symlinks evita los problemas relacionados con el cambio de los nombres de dispositivo. Para más información, consulte [Solución de problemas de nombres de dispositivo](../troubleshooting/troubleshoot-device-names-problems.md).
    
        ```console
        mkfs -t ext4 /dev/disk/azure/scsi1/lun0
        ```

    2. Monte los discos.

        ```console
        mount /dev/disk/azure/scsi1/lun0 /mnt/mountpoint
        ```

    3. Agréguelos a fstab.

        ```console
        echo "/dev/disk/azure/scsi1/lun0 /mnt/mountpoint ext4 defaults,nofail 1 2" >> /etc/fstab
        ```

    4. Ejecute el cmdlet Set-AzVMDiskEncryptionExtension PowerShell con -EncryptFormatAll para cifrar estos discos.

       ```azurepowershell-interactive
        Set-AzVMDiskEncryptionExtension -ResourceGroupName "MySecureGroup" -VMName "MySecureVM" -DiskEncryptionKeyVaultUrl "https://mykeyvault.vault.azure.net/" -EncryptFormatAll
        ```

    5. Configure LVM encima de estos nuevos discos. Tenga en cuenta que las unidades cifradas se desbloquean después de que la máquina virtual ha terminado de arrancar. Por lo tanto, el montaje de LVM también tendrá que retrasarse.




## <a name="new-iaas-vms-created-from-customer-encrypted-vhd-and-encryption-keys"></a><a name="bkmk_VHDpre"> </a>Nuevas máquinas virtuales IaaS creadas a partir de discos duros virtuales cifrados por el cliente y claves de cifrado
En este escenario, se puede habilitar el cifrado mediante la plantilla de Resource Manager, cmdlets de PowerShell o comandos de la CLI. Las siguientes secciones explican con más detalle la plantilla de Resource Manager y los comandos de la CLI. 

Use las instrucciones del apéndice para preparar imágenes previamente cifradas que se pueden usar en Azure. Una vez creada la imagen, puede seguir los pasos de la sección siguiente para crear una VM cifrada de Azure.

* [Preparación de un VHD con Linux precifrado](disk-encryption-sample-scripts.md)

>[!IMPORTANT]
 >Es obligatorio crear una instantánea o una copia de seguridad de una instancia de máquina virtual basada en un disco administrado fuera de Azure Disk Encryption y antes de habilitar esta característica. Puede tomar una instantánea del disco administrado desde el portal o puede usar [Azure Backup](../../backup/backup-azure-vms-encryption.md). Las copias de seguridad garantizan que es posible disponer de una opción de recuperación en el caso de que se produzca un error inesperado durante el cifrado. Después de hacer la copia de seguridad, use el cmdlet Set-AzVMDiskEncryptionExtension para cifrar los discos administrados mediante la especificación del parámetro -skipVmBackup. El comando Set-AzVMDiskEncryptionExtension no se ejecuta en las máquinas virtuales basadas en un disco administrado hasta que se haya hecho una copia de seguridad y se especifique este parámetro. 
>
>Cifrar o deshabilitar el cifrado puede provocar el reinicio de la máquina virtual.



### <a name="use-azure-powershell-to-encrypt-iaas-vms-with-pre-encrypted-vhds"></a><a name="bkmk_VHDprePSH"> </a> Uso de Azure PowerShell para cifrar máquinas virtuales IaaS con discos duros virtuales previamente cifrados 
Puede habilitar el cifrado de disco en el disco duro virtual cifrado mediante el cmdlet [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk#examples) de PowerShell. En el ejemplo siguiente se ofrecen algunos parámetros comunes. 

```powershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>Habilitación del cifrado en un disco de datos recién agregado
Puede agregar un nuevo disco de datos mediante [az vm disk attach](add-disk.md) o [Azure Portal](attach-disk-portal.md). Para poder cifrarlo, deberá montar primero el disco de datos recién asociado. Dado que la unidad de datos no se puede usar mientras el cifrado está en curso, deberá solicitar el cifrado de la unidad. 

### <a name="enable-encryption-on-a-newly-added-disk-with-the-azure-cli"></a>Habilitación del cifrado en un disco recién agregado con la CLI de Azure
 Si la máquina virtual se cifró previamente con "All", el parámetro --volume-type debe permanecer como "All". All incluye los discos de datos y del SO. Si la máquina virtual se cifró previamente con el tipo de volumen "OS", el parámetro --volume-type se debe cambiar a "All", con el fin de que se incluyan tanto el nuevo disco de datos como el sistema operativo. Si la VM solo se cifró con el tipo de volumen de "Data", puede permanecer como "Data", tal y como se muestra aquí. Para preparar el cifrado, no es suficiente con agregar y conectar un nuevo disco de datos a una máquina virtual. El disco recién conectado también se debe formatear y montar correctamente en la máquina virtual antes de habilitar el cifrado. En Linux, el disco se debe montar en /etc/fstab con un [nombre de dispositivo de bloque persistente](../troubleshooting/troubleshoot-device-names-problems.md). 

A diferencia de la sintaxis de PowerShell, la CLI no requiere que el usuario proporcione ninguna versión de secuencia única cuando se habilita el cifrado. La CLI genera y usa su propio valor de versión de secuencia único automáticamente.

-  **Cifrado de una máquina virtual en ejecución mediante un secreto de cliente:** 
    
     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type "Data"
     ```

- **Cifrado de una máquina virtual en ejecución mediante KEK para encapsular el secreto de cliente:**

     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "Data"
     ```

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Habilitación del cifrado en un disco recién agregado con Azure PowerShell
 Al usar PowerShell para cifrar un nuevo disco para Linux, se debe especificar una nueva versión de secuencia. La versión de secuencia debe ser única. El siguiente script genera un GUID para la versión de secuencia. 
 

-  **Cifrado de una máquina virtual en ejecución mediante un secreto de cliente:** el siguiente script inicializa las variables y ejecuta el cmdlet Set-AzVMDiskEncryptionExtension. El grupo de recursos, la máquina virtual, el almacén de claves, la aplicación de Azure AD y el secreto de cliente deben haberse creado ya como requisitos previos. Reemplace MyVirtualMachineResourceGroup, MyKeyVaultResourceGroup, MySecureVM, MySecureVault, My-AAD-client-ID y My-AAD-client-secret por sus propios valores. El parámetro - VolumeType se establece en discos de datos y no en el disco del sistema operativo. Si la máquina virtual se cifró previamente con los tipos de volumen "OS" o "All", el parámetro -VolumeType se debe cambiar a "All", con el fin de que se incluyan tanto el nuevo disco de datos como el sistema operativo.

     ```azurepowershell
         $KVRGname = 'MyKeyVaultResourceGroup';
         $VMRGName = 'MyVirtualMachineResourceGroup'; 
         $vmName = 'MySecureVM';
         $aadClientID = 'My-AAD-client-ID';
         $aadClientSecret = 'My-AAD-client-secret';
         $KeyVaultName = 'MySecureVault';
         $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
         $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
         $KeyVaultResourceId = $KeyVault.ResourceId;
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion;
     ```
- **Cifrado de una máquina virtual en ejecución mediante KEK para encapsular el secreto de cliente:** Azure Disk Encryption le permite especificar una clave existente en el almacén de claves para encapsular los secretos de cifrado de disco que se generaron al habilitar el cifrado. Cuando se especifica una clave de cifrado de claves, Azure Disk Encryption usa esa clave para encapsular los secretos de cifrado antes de escribirlos en el almacén de claves. El parámetro - VolumeType se establece en discos de datos y no en el disco del sistema operativo. Si la máquina virtual se cifró previamente con los tipos de volumen "OS" o "All", el parámetro -VolumeType se debe cambiar a "All", con el fin de que se incluyan tanto el nuevo disco de datos como el sistema operativo.

     ```azurepowershell
         $KVRGname = 'MyKeyVaultResourceGroup';
         $VMRGName = 'MyVirtualMachineResourceGroup';
         $vmName = 'MyExtraSecureVM';
         $aadClientID = 'My-AAD-client-ID';
         $aadClientSecret = 'My-AAD-client-secret';
         $KeyVaultName = 'MySecureVault';
         $keyEncryptionKeyName = 'MyKeyEncryptionKey';
         $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
         $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
         $KeyVaultResourceId = $KeyVault.ResourceId;
         $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion;
     ```


>[!NOTE]
> La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]. </br> </br>
La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id].

## <a name="disable-encryption-for-linux-vms"></a>Deshabilitación del cifrado para máquinas virtuales Linux
Puede deshabilitar el cifrado con Azure PowerShell, la CLI de Azure o una plantilla de Resource Manager. 

>[!IMPORTANT]
>Solo se puede deshabilitar el cifrado con Azure Disk Encryption en máquinas virtuales Linux para volúmenes de datos. No se admite en volúmenes de datos o del sistema operativo si el volumen del sistema operativo se ha cifrado. 

- **Deshabilitar el cifrado de disco con Azure PowerShell:** para deshabilitar el cifrado, use el cmdlet [Disable-AzureRmVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). 
     ```azurepowershell-interactive
         Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM' [--volume-type {ALL, DATA, OS}]
     ```

- **Deshabilitar el cifrado con la CLI de Azure:** para deshabilitar el cifrado, use el comando [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). 
     ```azurecli-interactive
         az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type [ALL, DATA, OS]
     ```
- **Deshabilitar el cifrado con una plantilla de Resource Manager:** use la plantilla [Deshabilitación del cifrado en una máquina virtual Linux en ejecución](https://aka.ms/decrypt-linuxvm) para deshabilitar el cifrado.
     1. Seleccione **Implementar en Azure**.
     2. Seleccione la suscripción, el grupo de recursos, la ubicación, la máquina virtual, los términos legales y el contrato.
     3. Seleccione **Comprar** para deshabilitar el cifrado de disco en una máquina virtual Windows en ejecución. 


## <a name="next-steps"></a>Pasos siguientes

- [Introducción a Azure Disk Encryption para Linux](disk-encryption-overview-aad.md)
- [Creación y configuración de un almacén de claves para Azure Disk Encryption con Azure AD (versión anterior)](disk-encryption-key-vault-aad.md)
