---
title: Scripts de ejemplo de Azure Disk Encryption
description: Este artículo es el apéndice de Microsoft Azure Disk Encryption para máquinas virtuales Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: how-to
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18, devx-track-azurepowershell
ms.openlocfilehash: abf805d24d164ba31daa4d77d6360629632bfcf0
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89072713"
---
# <a name="azure-disk-encryption-sample-scripts-for-linux-vms"></a>Scripts de ejemplo de Azure Disk Encryption para máquinas virtuales Linux

En este artículo se proporcionan scripts de ejemplo tanto para preparar los discos duros virtuales previamente cifrados como para realizar otras tareas.

 

## <a name="sample-powershell-scripts-for-azure-disk-encryption"></a>Scripts de ejemplo de PowerShell para Azure Disk Encryption 

- **Enumerar todas las máquinas virtuales cifradas en la suscripción**

     ```azurepowershell-interactive
     $osVolEncrypted = {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).OsVolumeEncrypted}
     $dataVolEncrypted= {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).DataVolumesEncrypted}
     Get-AzVm | Format-Table @{Label="MachineName"; Expression={$_.Name}}, @{Label="OsVolumeEncrypted"; Expression=$osVolEncrypted}, @{Label="DataVolumesEncrypted"; Expression=$dataVolEncrypted}
     ```

- **Enumerar todos los secretos de cifrado de disco usados para cifrar las máquinas virtuales en un almacén de claves** 

     ```azurepowershell-interactive
     Get-AzKeyVaultSecret -VaultName $KeyVaultName | where {$_.Tags.ContainsKey('DiskEncryptionKeyFileName')} | format-table @{Label="MachineName"; Expression={$_.Tags['MachineName']}}, @{Label="VolumeLetter"; Expression={$_.Tags['VolumeLetter']}}, @{Label="EncryptionKeyURL"; Expression={$_.Id}}
     ```

### <a name="using-the-azure-disk-encryption-prerequisites-powershell-script"></a>Uso del script de PowerShell de requisitos previos de Azure Disk Encryption
Si ya está familiarizado con los requisitos previos para Azure Disk Encryption, puede usar el [script de PowerShell de requisitos previos de Azure Disk Encryption](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/AzureDiskEncryptionPreRequisiteSetup.ps1 ). Para un ejemplo de uso de este script de PowerShell, consulte la [guía de inicio rápido para el cifrado de máquinas virtuales](disk-encryption-powershell-quickstart.md). Puede quitar los comentarios de una sección del script, a partir de la línea 211, para cifrar todos los discos de las máquinas virtuales de un grupo de recursos existente. 

En la siguiente tabla se muestran los parámetros que se pueden usar en el script de PowerShell: 


|Parámetro|Descripción|¿Obligatorio?|
|------|------|------|
|$resourceGroupName| Nombre del grupo de recursos al que pertenece la instancia de KeyVault.  Si no existe, se creará un grupo de recursos con este nombre.| True|
|$keyVaultName|Nombre de la instancia de KeyVault en donde se colocarán las claves de cifrado. Si no existe, se creará un almacén con este nombre.| True|
|$location|Ubicación de la instancia de KeyVault. Asegúrese de que la instancia de KeyVault y las máquinas virtuales que se van a cifrar están en la misma ubicación. Obtenga una lista de ubicaciones con `Get-AzLocation`.|True|
|$subscriptionId|Identificador de la suscripción de Azure que se va a utilizar.  El identificador de la suscripción se puede obtener con `Get-AzSubscription`.|True|
|$aadAppName|Nombre de la aplicación de Azure AD que se va a utilizar para escribir secretos en KeyVault. Si no existe, se creará una aplicación con este nombre. Si esta aplicación ya existe, pasará el parámetro aadClientSecret al script.|False|
|$aadClientSecret|Secreto de cliente de la aplicación de Azure AD que se creó anteriormente.|False|
|$keyEncryptionKeyName|Nombre de la clave de cifrado de clave opcional en KeyVault. Si no existe, se creará una clave con este nombre.|False|


### <a name="encrypt-or-decrypt-vms-without-an-azure-ad-app"></a>Cifrado o descifrado de máquinas virtuales sin aplicación de Azure AD

- [Habilitar el cifrado de disco en una máquina virtual Linux existente o en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm-without-aad)  
- [Deshabilitar el cifrado en una máquina virtual Linux en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-linux-vm-without-aad) 
    - La deshabilitación del cifrado solo se permite en volúmenes de datos de máquinas virtuales Linux.  

### <a name="encrypt-or-decrypt-vms-with-an-azure-ad-app-previous-release"></a>Cifrado o descifrado de máquinas virtuales con aplicación de Azure AD (versión anterior) 
 
- [Habilitar el cifrado de disco en una máquina virtual Linux existente o en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm)    


-  [Deshabilitar el cifrado en una máquina virtual Linux en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-linux-vm) 
    - La deshabilitación del cifrado solo se permite en volúmenes de datos de máquinas virtuales Linux. 


- [Crear un disco administrado cifrado a partir de un blob de almacenamiento o disco duro virtual previamente cifrado](https://github.com/Azure/azure-quickstart-templates/tree/master/201-create-encrypted-managed-disk)
    - Crea un disco administrado cifrado a partir de un disco duro virtual previamente cifrado y su correspondiente configuración de cifrado.





## <a name="encrypting-an-os-drive-on-a-running-linux-vm"></a>Cifrado de la unidad del sistema operativo en una máquina virtual con Linux en ejecución

### <a name="prerequisites-for-os-disk-encryption"></a>Requisitos previos para el cifrado de disco del sistema operativo

* La máquina virtual debe usar una distribución compatible con el cifrado del disco del sistema operativo, como se muestra en el artículo [Sistemas operativos compatibles con Azure Disk Encryption](disk-encryption-overview.md#supported-vms) 
* La máquina virtual debe crearse a partir de la imagen de Marketplace en Azure Resource Manager.
* La máquina virtual de Azure con al menos 4 GB de RAM (el tamaño recomendado es 7 GB).
* (Para RHEL y CentOS) Deshabilite SELinux. Para deshabilitar SELinux, consulte "4.4.2. Disabling SELinux" (Deshabilitar SELinux) en el documento [SELinux User's and Administrator's Guide](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SELinux_Users_and_Administrators_Guide/sect-Security-Enhanced_Linux-Working_with_SELinux-Changing_SELinux_Modes.html#sect-Security-Enhanced_Linux-Enabling_and_Disabling_SELinux-Disabling_SELinux) (Guía del administrador y del usuario de SELinux) en la máquina virtual.
* Después de deshabilitar SELinux, reinicie la máquina virtual al menos una vez.

### <a name="steps"></a>Pasos
1. Cree una máquina virtual mediante una de las distribuciones especificadas anteriormente.

   Para CentOS 7.2, se admite el cifrado de disco del sistema operativo a través de una imagen especial. Para usar esta imagen, especifique "7.2n" como SKU al crear la máquina virtual:

   ```powershell
    Set-AzVMSourceImage -VM $VirtualMachine -PublisherName "OpenLogic" -Offer "CentOS" -Skus "7.2n" -Version "latest"
   ```
2. Configure la máquina virtual según sus necesidades. Si va a cifrar todas las unidades (sistema operativo y datos), las unidades de datos tienen que especificarse y montarse desde /etc/fstab.

   > [!NOTE]
   > Use UUID=... para especificar unidades de datos en /etc/fstab en lugar de especificar el nombre del dispositivo de bloque (por ejemplo, /dev/sdb1). Durante el cifrado, se cambia el orden de las unidades en la máquina virtual. Si la máquina virtual depende de un orden específico de dispositivos de bloque, se producirá un error de montaje después del cifrado.

3. Cierre las sesiones de SSH.

4. Para cifrar el sistema operativo, especifique volumeType como **All** u **OS** al habilitar el cifrado.

   > [!NOTE]
   > Todos los procesos de espacio de usuario que no se ejecutan como servicios `systemd` deben terminarse con `SIGKILL`. Reinicie la máquina virtual. Al habilitar el cifrado del disco de sistema operativo en una máquina virtual en ejecución, tenga en cuenta el tiempo de inactividad de la máquina virtual.

5. Supervise periódicamente el progreso del cifrado con las instrucciones de la [sección siguiente](#monitoring-os-encryption-progress).

6. Cuando Get-AzVmDiskEncryptionStatus muestre "VMRestartPending", inicie sesión en la máquina virtual para reiniciarla o hágalo mediante el portal, PowerShell o la CLI.
    ```powershell
    C:\> Get-AzVmDiskEncryptionStatus  -ResourceGroupName $ResourceGroupName -VMName $VMName
    -ExtensionName $ExtensionName

    OsVolumeEncrypted          : VMRestartPending
    DataVolumesEncrypted       : NotMounted
    OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
    ProgressMessage            : OS disk successfully encrypted, reboot the VM
    ```
   Antes de reiniciar, se recomienda que guarde los [diagnósticos de arranque](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/) de la máquina virtual.

## <a name="monitoring-os-encryption-progress"></a>Supervisión del progreso de cifrado del sistema operativo
Hay tres maneras de supervisar el progreso de cifrado del sistema operativo:

* Use el cmdlet `Get-AzVmDiskEncryptionStatus` e inspeccione el campo ProgressMessage:
    ```powershell
    OsVolumeEncrypted          : EncryptionInProgress
    DataVolumesEncrypted       : NotMounted
    OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
    ProgressMessage            : OS disk encryption started
    ```
  Una vez que la máquina virtual llega al mensaje de inicio del cifrado de disco del sistema operativo, el proceso tarda aproximadamente entre 40 y 50 minutos en una máquina virtual de almacenamiento Premium.

  Debido al [problema 388](https://github.com/Azure/WALinuxAgent/issues/388) de WALinuxAgent, `OsVolumeEncrypted` y `DataVolumesEncrypted` se muestran como `Unknown` en algunas distribuciones. En WALinuxAgent versión 2.1.5 y versiones posteriores, este problema se ha corregido automáticamente. En caso de que se vea `Unknown` en la salida, puede comprobar el estado de cifrado del disco mediante el Explorador de recursos de Azure.

  Vaya al [Explorador de recursos de Azure](https://resources.azure.com/) y expanda esta jerarquía en el panel de selección a la izquierda:

  ~~~~
  |-- subscriptions
     |-- [Your subscription]
          |-- resourceGroups
               |-- [Your resource group]
                    |-- providers
                         |-- Microsoft.Compute
                              |-- virtualMachines
                                   |-- [Your virtual machine]
                                        |-- InstanceView
  ~~~~                

  En el objeto InstanceView, desplácese hacia abajo para ver el estado de cifrado de las unidades de disco.

  ![Vista de instancia de máquina virtual](./media/disk-encryption/vm-instanceview.png)

* Fíjese en los [diagnósticos de arranque](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/). Los mensajes de la extensión ADE deben llevar el prefijo `[AzureDiskEncryption]`.

* Inicie sesión en la máquina virtual a través de SSH y obtenga el registro de la extensión de:

    /var/log/azure/Microsoft.Azure.Security.AzureDiskEncryptionForLinux

  Se recomienda que no inicie sesión en la máquina virtual con el cifrado del sistema operativo en curso. Copie los registros solo cuando los otros dos métodos no den resultado.

## <a name="prepare-a-pre-encrypted-linux-vhd"></a>Preparación de un VHD con Linux precifrado
La preparación de disco duros virtuales previamente cifrados puede variar dependiendo de la distribución. Se pueden encontrar ejemplos sobre cómo preparar Ubuntu 16, openSUSE 13.2 y CentOS 7. 

### <a name="ubuntu-16"></a>Ubuntu 16
Configure el cifrado durante la instalación de la distribución; para ello, siga estos pasos:

1. Seleccione **Configure encrypted volumes** (Configurar volúmenes cifrados) al realizar la partición de los discos.

   ![Configuración de Ubuntu 16.04: configuración de volúmenes cifrados](./media/disk-encryption/ubuntu-1604-preencrypted-fig1.png)

2. Cree una unidad de arranque independiente, que no debe estar cifrada. Cifre la unidad raíz.

   ![Configuración de Ubuntu 16.04: selección de dispositivos para cifrar](./media/disk-encryption/ubuntu-1604-preencrypted-fig2.png)

3. Proporcione una frase de contraseña. Se trata de la frase de contraseña que se carga en el almacén de claves.

   ![Configuración de Ubuntu 16.04: suministro de la frase de contraseña](./media/disk-encryption/ubuntu-1604-preencrypted-fig3.png)

4. Finalice la partición.

   ![Configuración de Ubuntu 16.04: finalización de la creación de particiones](./media/disk-encryption/ubuntu-1604-preencrypted-fig4.png)

5. Cuando arranque la máquina virtual y se le pida una frase de contraseña, utilice la frase de contraseña que especificó en el paso 3.

   ![Configuración de Ubuntu 16.04: suministro de la frase de contraseña en el arranque](./media/disk-encryption/ubuntu-1604-preencrypted-fig5.png)

6. Prepare la máquina virtual para su carga en Azure con [estas instrucciones](./create-upload-ubuntu.md?toc=/azure/virtual-machines/linux/toc.json). No ejecute todavía el último paso (desaprovisionamiento de la máquina virtual).

Configure el cifrado para que funcione con Azure de esta manera:

1. Cree un archivo en /usr/local/sbin/azure_crypt_key.sh con el contenido del script siguiente. Preste atención a KeyFileName, porque es el nombre que Azure ha asignado al archivo de frase de contraseña.

    ```bash
    #!/bin/sh
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint
    modprobe vfat >/dev/null 2>&1
    modprobe ntfs >/dev/null 2>&1
    sleep 2
    OPENED=0
    cd /sys/block
    for DEV in sd*; do

        echo "> Trying device: $DEV ..." >&2
        mount -t vfat -r /dev/${DEV}1 $MountPoint >/dev/null||
        mount -t ntfs -r /dev/${DEV}1 $MountPoint >/dev/null
        if [ -f $MountPoint/$KeyFileName ]; then
                cat $MountPoint/$KeyFileName
                umount $MountPoint 2>/dev/null
                OPENED=1
                break
        fi
        umount $MountPoint 2>/dev/null
    done

      if [ $OPENED -eq 0 ]; then
        echo "FAILED to find suitable passphrase file ..." >&2
        echo -n "Try to enter your password: " >&2
        read -s -r A </dev/console
        echo -n "$A"
     else
        echo "Success loading keyfile!" >&2
    fi
   ```

2. Cambie la configuración de cifrado en */etc/crypttab*. Debería ser parecido a este:
   ```
    xxx_crypt uuid=xxxxxxxxxxxxxxxxxxxxx none luks,discard,keyscript=/usr/local/sbin/azure_crypt_key.sh
    ```

4. Agregue permisos de ejecución al script:
   ```
    chmod +x /usr/local/sbin/azure_crypt_key.sh
   ```
5. Edite */etc/initramfs-tools/modules* anexando líneas:
   ```
    vfat
    ntfs
    nls_cp437
    nls_utf8
    nls_iso8859-1
   ```
6. Ejecute `update-initramfs -u -k all` para actualizar initramfs a fin de que `keyscript` surta efecto.

7. Ahora puede desaprovisionar la máquina virtual.

   ![Configuración de Ubuntu 16.04: update-initramfs](./media/disk-encryption/ubuntu-1604-preencrypted-fig6.png)

8. Continúe con el paso siguiente y cargue el disco duro virtual en Azure.

### <a name="opensuse-132"></a>openSUSE 13.2
Para configurar el cifrado durante la instalación de distribución, siga estos pasos:
1. Al crear particiones de los discos, seleccione **Encrypt Volume Group** (Cifrar el grupo de volúmenes) y, a continuación, escriba una contraseña. Esta es la contraseña que se cargará en el almacén de claves.

   ![Configuración de openSUSE 13.2: cifrado del grupo de volúmenes](./media/disk-encryption/opensuse-encrypt-fig1.png)

2. Arranque la máquina virtual con la contraseña.

   ![Configuración de openSUSE 13.2: suministro de la frase de contraseña en el arranque](./media/disk-encryption/opensuse-encrypt-fig2.png)

3. Prepare la máquina virtual para cargarla en Azure con las instrucciones de [Preparación de una máquina virtual SLES u openSUSE para Azure](./suse-create-upload-vhd.md?toc=/azure/virtual-machines/linux/toc.json#prepare-opensuse-131). No ejecute todavía el último paso (desaprovisionamiento de la máquina virtual).

Para configurar el cifrado de manera que funcione con Azure, siga estos pasos:
1. Edite /etc/dracut.conf y agregue la línea siguiente:
    ```
    add_drivers+=" vfat ntfs nls_cp437 nls_iso8859-1"
    ```
2. Agregue comentarios a estas líneas al final del archivo /usr/lib/dracut/modules.d/90crypt/module-setup.sh:
   ```bash
    #        inst_multiple -o \
    #        $systemdutildir/system-generators/systemd-cryptsetup-generator \
    #        $systemdutildir/systemd-cryptsetup \
    #        $systemdsystemunitdir/systemd-ask-password-console.path \
    #        $systemdsystemunitdir/systemd-ask-password-console.service \
    #        $systemdsystemunitdir/cryptsetup.target \
    #        $systemdsystemunitdir/sysinit.target.wants/cryptsetup.target \
    #        systemd-ask-password systemd-tty-ask-password-agent
    #        inst_script "$moddir"/crypt-run-generator.sh /sbin/crypt-run-generator
   ```

3. Agregue la siguiente línea al principio del archivo /usr/lib/dracut/modules.d/90crypt/parse-crypt.sh:
   ```bash
    DRACUT_SYSTEMD=0
   ```
   Y cambie todas las apariciones de:
   ```bash
    if [ -z "$DRACUT_SYSTEMD" ]; then
   ```
   a:
   ```bash
    if [ 1 ]; then
   ```
4. Edite /usr/lib/dracut/modules.d/90crypt/cryptroot-ask.sh and append it to "# Open LUKS device":

    ```bash
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint >&2
    modprobe vfat >/dev/null >&2
    modprobe ntfs >/dev/null >&2
    for SFS in /dev/sd*; do
    echo "> Trying device:$SFS..." >&2
    mount ${SFS}1 $MountPoint -t vfat -r >&2 ||
    mount ${SFS}1 $MountPoint -t ntfs -r >&2
    if [ -f $MountPoint/$KeyFileName ]; then
        echo "> keyfile got..." >&2
        cp $MountPoint/$KeyFileName /tmp-keyfile >&2
        luksfile=/tmp-keyfile
        umount $MountPoint >&2
        break
    fi
    done
    ```
5. Ejecute `/usr/sbin/dracut -f -v` para actualizar el initrd.

6. Ahora puede desaprovisionar la máquina virtual y cargar el disco duro virtual en Azure.

### <a name="centos-7-and-rhel-7"></a>CentOS 7 y RHEL 7

Para configurar el cifrado durante la instalación de distribución, siga estos pasos:
1. Seleccione **Encrypt my data** (Cifrar mis datos) al crear particiones en discos.

   ![Configuración de CentOS 7: destino de la instalación](./media/disk-encryption/centos-encrypt-fig1.png)

2. Asegúrese de que **Encrypt** (Cifrar) está seleccionado para la partición raíz.

   ![Configuración de CentOS 7: selección del cifrado para la partición raíz](./media/disk-encryption/centos-encrypt-fig2.png)

3. Proporcione una frase de contraseña. Se trata de la frase de contraseña que va a cargar en el almacén de claves.

   ![Configuración de CentOS 7: suministro de la frase de contraseña](./media/disk-encryption/centos-encrypt-fig3.png)

4. Cuando arranque la máquina virtual y se le pida una frase de contraseña, utilice la frase de contraseña que especificó en el paso 3.

   ![Configuración de CentOS 7: introducción de la frase de contraseña en el arranque](./media/disk-encryption/centos-encrypt-fig4.png)

5. Prepare la máquina virtual para cargarla en Azure con las instrucciones de "CentOS 7.0+" en [Preparación de una máquina virtual basada en CentOS para Azure](./create-upload-centos.md?toc=/azure/virtual-machines/linux/toc.json#centos-70). No ejecute todavía el último paso (desaprovisionamiento de la máquina virtual).

6. Ahora puede desaprovisionar la máquina virtual y cargar el disco duro virtual en Azure.

Para configurar el cifrado de manera que funcione con Azure, siga estos pasos:

1. Edite /etc/dracut.conf y agregue la línea siguiente:
    ```
    add_drivers+=" vfat ntfs nls_cp437 nls_iso8859-1"
    ```

2. Agregue comentarios a estas líneas al final del archivo /usr/lib/dracut/modules.d/90crypt/module-setup.sh:
   ```bash
    #        inst_multiple -o \
    #        $systemdutildir/system-generators/systemd-cryptsetup-generator \
    #        $systemdutildir/systemd-cryptsetup \
    #        $systemdsystemunitdir/systemd-ask-password-console.path \
    #        $systemdsystemunitdir/systemd-ask-password-console.service \
    #        $systemdsystemunitdir/cryptsetup.target \
    #        $systemdsystemunitdir/sysinit.target.wants/cryptsetup.target \
    #        systemd-ask-password systemd-tty-ask-password-agent
    #        inst_script "$moddir"/crypt-run-generator.sh /sbin/crypt-run-generator
   ```

3. Agregue la siguiente línea al principio del archivo /usr/lib/dracut/modules.d/90crypt/parse-crypt.sh:
   ```bash
    DRACUT_SYSTEMD=0
   ```
   Y cambie todas las apariciones de:
   ```bash
    if [ -z "$DRACUT_SYSTEMD" ]; then
   ```
   to
   ```bash
    if [ 1 ]; then
   ```
4. Edite /usr/lib/dracut/modules.d/90crypt/cryptroot-ask.sh y anexe lo siguiente después de "# Open LUKS device":
    ```bash
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint >&2
    modprobe vfat >/dev/null >&2
    modprobe ntfs >/dev/null >&2
    for SFS in /dev/sd*; do
    echo "> Trying device:$SFS..." >&2
    mount ${SFS}1 $MountPoint -t vfat -r >&2 ||
    mount ${SFS}1 $MountPoint -t ntfs -r >&2
    if [ -f $MountPoint/$KeyFileName ]; then
        echo "> keyfile got..." >&2
        cp $MountPoint/$KeyFileName /tmp-keyfile >&2
        luksfile=/tmp-keyfile
        umount $MountPoint >&2
        break
    fi
    done
    ```    
5. Ejecute "/usr/sbin/dracut -f -v" para actualizar initrd.

    ![Configuración de CentOS 7: run /usr/sbin/dracut -f -v](./media/disk-encryption/centos-encrypt-fig5.png)

## <a name="upload-encrypted-vhd-to-an-azure-storage-account"></a>Carga de un VHD cifrado a una cuenta de almacenamiento de Azure
Una vez que se cifre DM-Crypt, el disco duro virtual cifrado local debe cargarse en la cuenta de almacenamiento.
```powershell
    Add-AzVhd [-Destination] <Uri> [-LocalFilePath] <FileInfo> [[-NumberOfUploaderThreads] <Int32> ] [[-BaseImageUriToPatch] <Uri> ] [[-OverWrite]] [ <CommonParameters>]
```
## <a name="upload-the-secret-for-the-pre-encrypted-vm-to-your-key-vault"></a>Carga del secreto de la máquina virtual previamente cifrada en el almacén de claves
Al cifrar mediante una aplicación de Azure AD (versión anterior), el secreto de cifrado del disco que obtuvo con anterioridad se debe cargar como secreto en el almacén de claves. El almacén de claves debe tener habilitados el cifrado de discos y los permisos para el cliente de Azure AD.

```powershell 
 $AadClientId = "My-AAD-Client-Id"
 $AadClientSecret = "My-AAD-Client-Secret"

 $key vault = New-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -Location $Location

 Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -ServicePrincipalName $AadClientId -PermissionsToKeys all -PermissionsToSecrets all
 Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -EnabledForDiskEncryption
``` 

### <a name="disk-encryption-secret-not-encrypted-with-a-kek"></a>Secreto del cifrado de disco no cifrado con una KEK
Para configurar el secreto en el almacén de claves, use [Set-AzKeyVaultSecret](/powershell/module/az.keyvault/set-azkeyvaultsecret). La frase de contraseña se codifica en forma de cadena base64 y, después, se carga en el almacén de claves. Además, asegúrese de que se establecen las siguientes etiquetas al crear el secreto en el almacén de claves.

```powershell

 # This is the passphrase that was provided for encryption during the distribution installation
 $passphrase = "contoso-password"

 $tags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
 $secretName = [guid]::NewGuid().ToString()
 $secretValue = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($passphrase))
 $secureSecretValue = ConvertTo-SecureString $secretValue -AsPlainText -Force

 $secret = Set-AzKeyVaultSecret -VaultName $KeyVaultName -Name $secretName -SecretValue $secureSecretValue -tags $tags
 $secretUrl = $secret.Id
```


Use `$secretUrl` en el paso siguiente para [conectar el disco del sistema operativo sin usar KEK](#without-using-a-kek).

### <a name="disk-encryption-secret-encrypted-with-a-kek"></a>Secreto del cifrado de disco cifrado con una KEK
Antes de cargar el secreto en el almacén de claves, puede cifrarlo si lo desea mediante una clave de cifrado de claves. Utilice la [API](/rest/api/keyvault/wrapkey) de encapsulamiento para cifrar por primera vez el secreto mediante la clave de cifrado de claves. El resultado de esta operación de encapsulamiento es una cadena codificada en URL como base64 que luego se carga como secreto con el cmdlet [`Set-AzKeyVaultSecret`](/powershell/module/az.keyvault/set-azkeyvaultsecret).

```powershell
    # This is the passphrase that was provided for encryption during the distribution installation
    $passphrase = "contoso-password"

    Add-AzKeyVaultKey -VaultName $KeyVaultName -Name "keyencryptionkey" -Destination Software
    $KeyEncryptionKey = Get-AzKeyVaultKey -VaultName $KeyVault.OriginalVault.Name -Name "keyencryptionkey"

    $apiversion = "2015-06-01"

    ##############################
    # Get Auth URI
    ##############################

    $uri = $KeyVault.VaultUri + "/keys"
    $headers = @{}

    $response = try { Invoke-RestMethod -Method GET -Uri $uri -Headers $headers } catch { $_.Exception.Response }

    $authHeader = $response.Headers["www-authenticate"]
    $authUri = [regex]::match($authHeader, 'authorization="(.*?)"').Groups[1].Value

    Write-Host "Got Auth URI successfully"

    ##############################
    # Get Auth Token
    ##############################

    $uri = $authUri + "/oauth2/token"
    $body = "grant_type=client_credentials"
    $body += "&client_id=" + $AadClientId
    $body += "&client_secret=" + [Uri]::EscapeDataString($AadClientSecret)
    $body += "&resource=" + [Uri]::EscapeDataString("https://vault.azure.net")
    $headers = @{}

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $access_token = $response.access_token

    Write-Host "Got Auth Token successfully"

    ##############################
    # Get KEK info
    ##############################

    $uri = $KeyEncryptionKey.Id + "?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token}

    $response = Invoke-RestMethod -Method GET -Uri $uri -Headers $headers

    $keyid = $response.key.kid

    Write-Host "Got KEK info successfully"

    ##############################
    # Encrypt passphrase using KEK
    ##############################

    $passphraseB64 = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($Passphrase))
    $uri = $keyid + "/encrypt?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"alg" = "RSA-OAEP"; "value" = $passphraseB64}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $wrappedSecret = $response.value

    Write-Host "Encrypted passphrase successfully"

    ##############################
    # Store secret
    ##############################

    $secretName = [guid]::NewGuid().ToString()
    $uri = $KeyVault.VaultUri + "/secrets/" + $secretName + "?api-version=" + $apiversion
    $secretAttributes = @{"enabled" = $true}
    $secretTags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"value" = $wrappedSecret; "attributes" = $secretAttributes; "tags" = $secretTags}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method PUT -Uri $uri -Headers $headers -Body $body

    Write-Host "Stored secret successfully"

    $secretUrl = $response.id
```

Use `$KeyEncryptionKey` y `$secretUrl` en el paso siguiente para [conectar el disco del sistema operativo usando KEK](#using-a-kek).

##  <a name="specify-a-secret-url-when-you-attach-an-os-disk"></a>Especificación de una dirección URL de secreto al adjuntar un disco del sistema operativo

###  <a name="without-using-a-kek"></a>Sin utilizar una KEK
Mientras esté adjuntando el disco del sistema operativo, tiene que pasar `$secretUrl`. La dirección URL se generó en la sección "Secreto de cifrado de disco no cifrado con una KEK".
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $VhdUri `
            -VhdUri $OSDiskUri `
            -Linux `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl
```
### <a name="using-a-kek"></a>Uso de una KEK
Cuando conecte un disco del sistema operativo, pase `$KeyEncryptionKey` y `$secretUrl`. La dirección URL se generó en la sección "Secreto del cifrado de disco cifrado con KEK".
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $CopiedTemplateBlobUri `
            -VhdUri $OSDiskUri `
            -Linux `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl `
            -KeyEncryptionKeyVaultId $KeyVault.ResourceId `
            -KeyEncryptionKeyURL $KeyEncryptionKey.Id
```
