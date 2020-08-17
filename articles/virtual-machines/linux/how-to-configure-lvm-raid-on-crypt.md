---
title: 'Configuración de LVM y RAID en dispositivos cifrados: Azure Disk Encryption'
description: En este artículo se proporcionan instrucciones para configurar LVM y RAID en dispositivos cifrados para VM Linux.
author: jofrance
ms.service: security
ms.topic: how-to
ms.author: jofrance
ms.date: 03/17/2020
ms.custom: seodec18
ms.openlocfilehash: 746243336d74aefc55df48872fe9dd21e9cd99a5
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87268227"
---
# <a name="configure-lvm-and-raid-on-encrypted-devices"></a>Configuración de LVM y RAID en dispositivos cifrados

En este artículo encontrará un proceso paso a paso para la Administración de volúmenes lógicos (LVM) y RAID en dispositivos cifrados. El proceso se aplica a los entornos siguientes:

- Distribuciones de Linux
    - RHEL 7.6 y versiones posteriores
    - Ubuntu 18.04 y versiones posteriores
    - SUSE 12 y versiones posteriores
- Extensión de pase único de Azure Disk Encryption
- Extensión de pase doble de Azure Disk Encryption


## <a name="scenarios"></a>Escenarios

Los procedimientos descritos en este artículo admiten los escenarios siguientes:  

- Configuración de LVM sobre dispositivos cifrados (LVM-on-crypt)
- Configuración de RAID sobre dispositivos cifrados (RAID-on-crypt)

Después de que se han cifrado los dispositivos subyacentes, puede crear las estructuras LVM o RAID sobre esa capa cifrada. 

Los volúmenes físicos (PV) se crean sobre la capa cifrada. Los volúmenes físicos se usan para crear el grupo de volúmenes. Cree los volúmenes y agregue las entradas necesarias en /etc/fstab. 

![Diagrama de las capas de las estructuras de LVM](./media/disk-encryption/lvm-raid-on-crypt/000-lvm-raid-crypt-diagram.png)

De forma similar, el dispositivo RAID se crea sobre la capa cifrada en los discos. Se crea un sistema de archivos sobre el dispositivo RAID y se agrega a /etc/fstab como dispositivo normal.

## <a name="considerations"></a>Consideraciones

Se recomienda utilizar LVM-on-crypt. RAID es una opción cuando LVM no se puede utilizar debido a limitaciones específicas del entorno o de la aplicación.

Usará la opción **EncryptFormatAll**. Para obtener más información acerca de esta opción, consulte [Uso de la característica EncryptFormatAll para discos de datos en máquinas virtuales Linux](./disk-encryption-linux.md#use-encryptformatall-feature-for-data-disks-on-linux-vms).

Si bien puede usar este método también al cifrar el sistema operativo, aquí solo se van a cifrar unidades de datos.

En los procedimientos se supone que ya ha revisado los requisitos previos en [Escenarios de Azure Disk Encryption en máquinas virtuales Linux](./disk-encryption-linux.md) y en [Inicio rápido: Creación y cifrado de una máquina virtual Linux con la CLI de Azure](./disk-encryption-cli-quickstart.md).

La versión de pase doble de Azure Disk Encryption quedará en desuso próximamente y ya no se debe usar en los nuevos cifrados.

## <a name="general-steps"></a>Pasos generales

Cuando use las configuraciones "on-crypt", siga el proceso que se describe en los procedimientos a continuación.

>[!NOTE] 
>Vamos a usar variables a lo largo de todo el artículo. Reemplace los valores en consecuencia.

### <a name="deploy-a-vm"></a>Implementación de una máquina virtual 
Los siguientes comandos son opcionales, pero se recomienda aplicarlos en una máquina virtual (VM) recién implementada.

PowerShell:

```powershell
New-AzVm -ResourceGroupName ${RGNAME} `
-Name ${VMNAME} `
-Location ${LOCATION} `
-Size ${VMSIZE} `
-Image ${OSIMAGE} `
-Credential ${creds} `
-Verbose
```
CLI de Azure:

```bash
az vm create \
-n ${VMNAME} \
-g ${RGNAME} \
--image ${OSIMAGE} \
--admin-username ${username} \
--admin-password ${password} \
-l ${LOCATION} \
--size ${VMSIZE} \
-o table
```
### <a name="attach-disks-to-the-vm"></a>Conexión de discos a la VM
Repita los siguientes comandos para `$N` número de discos nuevos que desee conectar a la VM.

PowerShell:

```powershell
$storageType = 'Standard_LRS'
$dataDiskName = ${VMNAME} + '_datadisk0'
$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $LOCATION -CreateOption Empty -DiskSizeGB 5
$dataDisk1 = New-AzDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName ${RGNAME}
$vm = Get-AzVM -Name ${VMNAME} -ResourceGroupName ${RGNAME} 
$vm = Add-AzVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 0
Update-AzVM -VM ${VM} -ResourceGroupName ${RGNAME}
```

CLI de Azure:

```bash
az vm disk attach \
-g ${RGNAME} \
--vm-name ${VMNAME} \
--name ${VMNAME}datadisk1 \
--size-gb 5 \
--new \
-o table
```

### <a name="verify-that-the-disks-are-attached-to-the-vm"></a>Verificación de que los discos están conectados a la VM
PowerShell:
```powershell
$VM = Get-AzVM -ResourceGroupName ${RGNAME} -Name ${VMNAME}
$VM.StorageProfile.DataDisks | Select-Object Lun,Name,DiskSizeGB
```
![Lista de discos conectados en PowerShell](./media/disk-encryption/lvm-raid-on-crypt/001-lvm-raid-check-disks-powershell.png)

CLI de Azure:

```bash
az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.dataDisks -o table
```
![Lista de discos conectados en la CLI de Azure](./media/disk-encryption/lvm-raid-on-crypt/002-lvm-raid-check-disks-cli.png)

Portal:

![Lista de discos conectados en el portal](./media/disk-encryption/lvm-raid-on-crypt/003-lvm-raid-check-disks-portal.png)

Sistema operativo:

```bash
lsblk 
```
![Lista de discos conectados en el sistema operativo](./media/disk-encryption/lvm-raid-on-crypt/004-lvm-raid-check-disks-os.png)

### <a name="configure-the-disks-to-be-encrypted"></a>Configuración de los discos que se van a cifrar
Esta configuración se realiza a nivel del sistema operativo. Los discos correspondientes se configuran para un cifrado tradicional a través de Azure Disk Encryption:

- Los sistemas de archivos se crean sobre los discos.
- Se crean puntos de montaje temporales para montar los sistemas de archivos.
- Los sistemas de archivos se configuran en /etc/fstab para montarse durante el arranque.

Compruebe la letra del dispositivo asignada a los nuevos discos. En este ejemplo, se van a usar cuatro discos de datos.

```bash
lsblk 
```
![Discos de datos conectados al sistema operativo](./media/disk-encryption/lvm-raid-on-crypt/004-lvm-raid-check-disks-os.png)

### <a name="create-a-file-system-on-top-of-each-disk"></a>Creación de un sistema de archivos sobre cada disco
Este comando itera la creación de un sistema de archivos ext4 en cada disco, definido en la parte "in" del ciclo "for".

```bash
for disk in c d e f; do echo mkfs.ext4 -F /dev/sd${disk}; done |bash
```
![Creación de un sistema de archivos ext4](./media/disk-encryption/lvm-raid-on-crypt/005-lvm-raid-create-temp-fs.png)

Busque el identificador único universal (UUID) de los sistemas de archivos recién creados, cree una carpeta temporal, agregue las entradas correspondientes en /etc/fstab y monte todos los sistemas de archivos.

Este comando también recorre en iteración cada disco definido en la parte "in" del ciclo "for":

```bash
for disk in c d e f; do diskuuid="$(blkid -s UUID -o value /dev/sd${disk})"; \
mkdir /tempdata${disk}; \
echo "UUID=${diskuuid} /tempdata${disk} ext4 defaults,nofail 0 0" >> /etc/fstab; \
mount -a; \
done
``` 

### <a name="verify-that-the-disks-are-mounted-properly"></a>Verificación de que los discos se han montado correctamente
```bash
lsblk
```
![Lista de sistemas de archivos temporales montados](./media/disk-encryption/lvm-raid-on-crypt/006-lvm-raid-verify-temp-fs.png)

Verifique también que los discos se han configurado:

```bash
cat /etc/fstab
```
![Información de configuración a través de fstab](./media/disk-encryption/lvm-raid-on-crypt/007-lvm-raid-verify-temp-fstab.png)

### <a name="encrypt-the-data-disks"></a>Cifrado de los discos de datos
PowerShell con una clave de cifrado de claves (KEK):

```powershell
$sequenceVersion = [Guid]::NewGuid() 
Set-AzVMDiskEncryptionExtension -ResourceGroupName $RGNAME `
-VMName ${VMNAME} `
-DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl `
-DiskEncryptionKeyVaultId $KeyVaultResourceId `
-KeyEncryptionKeyUrl $keyEncryptionKeyUrl `
-KeyEncryptionKeyVaultId $KeyVaultResourceId `
-VolumeType 'DATA' `
-EncryptFormatAll `
-SequenceVersion $sequenceVersion `
-skipVmBackup;
```

CLI de Azure mediante una KEK:

```bash
az vm encryption enable \
--resource-group ${RGNAME} \
--name ${VMNAME} \
--disk-encryption-keyvault ${KEYVAULTNAME} \
--key-encryption-key ${KEYNAME} \
--key-encryption-keyvault ${KEYVAULTNAME} \
--volume-type "DATA" \
--encrypt-format-all \
-o table
```
### <a name="verify-the-encryption-status"></a>Comprobación del estado de cifrado

Continúe con el paso siguiente solo cuando todos los discos estén cifrados.

PowerShell:

```powershell
Get-AzVmDiskEncryptionStatus -ResourceGroupName ${RGNAME} -VMName ${VMNAME}
```
![Estado de cifrado en PowerShell](./media/disk-encryption/lvm-raid-on-crypt/008-lvm-raid-verify-encryption-status-ps.png)

CLI de Azure:

```bash
az vm encryption show -n ${VMNAME} -g ${RGNAME} -o table
```
![Estado de cifrado en la CLI de Azure](./media/disk-encryption/lvm-raid-on-crypt/009-lvm-raid-verify-encryption-status-cli.png)

Portal:

![Estado de cifrado en el portal](./media/disk-encryption/lvm-raid-on-crypt/010-lvm-raid-verify-encryption-status-portal.png)

Nivel del sistema operativo:

```bash
lsblk
```
![Estado de cifrado en el sistema operativo](./media/disk-encryption/lvm-raid-on-crypt/011-lvm-raid-verify-encryption-status-os.png)

La extensión agregará los sistemas de archivos a /var/lib/azure_disk_encryption_config/azure_crypt_mount (un cifrado anterior), o bien a /etc/crypttab (cifrados nuevos).

>[!NOTE] 
>No modifique ninguno de estos archivos.

Este archivo se encargará de activar estos discos durante el proceso de arranque para que LVM o RAID pueda usarlos más adelante. 

No se preocupe por los puntos de montaje de este archivo. Azure Disk Encryption perderá la capacidad de montar los discos como sistema de archivos normal después de crear un volumen físico o un dispositivo RAID sobre esos dispositivos cifrados. (Esto quitará el formato del sistema de archivos que usamos durante el proceso de preparación).

### <a name="remove-the-temporary-folders-and-temporary-fstab-entries"></a>Eliminación de las carpetas temporales y las entradas fstab temporales
Desmonte los sistemas de archivos en los discos que se usarán como parte de LVM.

```bash
for disk in c d e f; do unmount /tempdata${disk}; done
```
Y quite las entradas /etc/fstab:

```bash
vi /etc/fstab
```
### <a name="verify-that-the-disks-are-not-mounted-and-that-the-entries-on-etcfstab-were-removed"></a>Compruebe que los discos no están montados y que se han quitado las entradas de /etc/fstab

```bash
lsblk
```
![Verificación de que los sistemas de archivos temporales se han desmontado](./media/disk-encryption/lvm-raid-on-crypt/012-lvm-raid-verify-disks-not-mounted.png)

Y verifique que los discos se han configurado:
```bash
cat /etc/fstab
```
![Verificación de que se han quitado las entradas fstab temporales](./media/disk-encryption/lvm-raid-on-crypt/013-lvm-raid-verify-fstab-temp-removed.png)

## <a name="steps-for-lvm-on-crypt"></a>Pasos para LVM-on-crypt
Ahora que los discos subyacentes están cifrados, puede crear las estructuras de LVM.

En lugar de usar el nombre del dispositivo, use las rutas de acceso /dev/mapper para cada uno de los discos a fin de crear un volumen físico (en la capa de cifrado sobre el disco, no en el propio disco).

### <a name="configure-lvm-on-top-of-the-encrypted-layers"></a>Configuración de LVM sobre las capas cifradas
#### <a name="create-the-physical-volumes"></a>Creación de los volúmenes físicos
Recibirá una advertencia que le pregunta si es correcto borrar la firma del sistema de archivos. Para continuar, escriba **y** o use **echo "y"** como se muestra a continuación:

```bash
echo "y" | pvcreate /dev/mapper/c49ff535-1df9-45ad-9dad-f0846509f052
echo "y" | pvcreate /dev/mapper/6712ad6f-65ce-487b-aa52-462f381611a1
echo "y" | pvcreate /dev/mapper/ea607dfd-c396-48d6-bc54-603cf741bc2a
echo "y" | pvcreate /dev/mapper/4159c60a-a546-455b-985f-92865d51158c
```
![Verificación de que se ha creado un volumen físico](./media/disk-encryption/lvm-raid-on-crypt/014-lvm-raid-pvcreate.png)

>[!NOTE] 
>Aquí, los nombres de /dev/mapper/device se deben reemplazar por los valores reales en función de la salida de **lsblk**.

#### <a name="verify-the-information-for-physical-volumes"></a>Verificación de la información de los volúmenes físicos
```bash
pvs
```

![Información de los volúmenes físicos](./media/disk-encryption/lvm-raid-on-crypt/015-lvm-raid-pvs.png)

#### <a name="create-the-volume-group"></a>Creación del grupo de volúmenes
Cree el grupo de volúmenes con los mismos dispositivos ya inicializados:

```bash
vgcreate vgdata /dev/mapper/
```

### <a name="check-the-information-for-the-volume-group"></a>Comprobación de la información del grupo de volúmenes

```bash
vgdisplay -v vgdata
```
```bash
pvs
```
![Información del grupo de volúmenes](./media/disk-encryption/lvm-raid-on-crypt/016-lvm-raid-pvs-on-vg.png)

#### <a name="create-logical-volumes"></a>Cree los volúmenes lógicos.

```bash
lvcreate -L 10G -n lvdata1 vgdata
lvcreate -L 7G -n lvdata2 vgdata
``` 

#### <a name="check-the-created-logical-volumes"></a>Comprobación de los volúmenes lógicos creados

```bash
lvdisplay
lvdisplay vgdata/lvdata1
lvdisplay vgdata/lvdata2
```
![Información de los volúmenes lógicos](./media/disk-encryption/lvm-raid-on-crypt/017-lvm-raid-lvs.png)

#### <a name="create-file-systems-on-top-of-the-structures-for-logical-volumes"></a>Cree sistemas de archivos sobre las estructuras de los volúmenes lógicos

```bash
echo "yes" | mkfs.ext4 /dev/vgdata/lvdata1
echo "yes" | mkfs.ext4 /dev/vgdata/lvdata2
```

#### <a name="create-the-mount-points-for-the-new-file-systems"></a>Cree los puntos de montaje para los nuevos sistemas de archivos

```bash
mkdir /data0
mkdir /data1
```

#### <a name="add-the-new-file-systems-to-etcfstab-and-mount-them"></a>Agregue los nuevos sistemas de archivos a /etc/fstab y móntelos

```bash
echo "/dev/mapper/vgdata-lvdata1 /data0 ext4 defaults,nofail 0 0" >>/etc/fstab
echo "/dev/mapper/vgdata-lvdata2 /data1 ext4 defaults,nofail 0 0" >>/etc/fstab
mount -a
```

#### <a name="verify-that-the-new-file-systems-are-mounted"></a>Verificación de que los nuevos sistemas de archivos se han montado

```bash
lsblk -fs
df -h
```
![Información para sistemas de archivos montados](./media/disk-encryption/lvm-raid-on-crypt/018-lvm-raid-lsblk-after-lvm.png)

En esta variación de **lsblk**, se enumeran los dispositivos que muestran las dependencias en orden inverso. Esta opción ayuda a identificar los dispositivos agrupados por el volumen lógico, en lugar de los nombres de dispositivo de /dev/sd[disk] originales.

Es importante asegurarse de que se agrega la opción **nofail** a las opciones de punto de montaje de los volúmenes LVM creados sobre un dispositivo cifrado mediante Azure Disk Encryption. Esto evita que el sistema operativo se bloquee durante el proceso de arranque (o en modo de mantenimiento).

Si no utiliza la opción **nofail**:

- El sistema operativo nunca llegará a la fase en la que se inicia Azure Disk Encryption y los discos de datos se desbloquean y se montan. 
- Los discos cifrados se desbloquearán al final del proceso de arranque. Los volúmenes de LVM y los sistemas de archivos estarán montados automáticamente hasta que Azure Disk Encryption los desbloquee. 

Puede probar reiniciar la VM y validar que los sistemas de archivos también se monten de forma automática después del tiempo de arranque. Este proceso puede tardar varios minutos, según el número y el tamaño de los sistemas de archivos.

#### <a name="reboot-the-vm-and-verify-after-reboot"></a>Reinicio de la máquina virtual y comprobación después del reinicio

```bash
shutdown -r now
```
```bash
lsblk
df -h
```
## <a name="steps-for-raid-on-crypt"></a>Pasos para RAID-on-crypt
Ahora que los discos subyacentes están cifrados, puede continuar para crear las estructuras de RAID. El proceso es el mismo que el de LVM, pero, en lugar de usar el nombre del dispositivo, use las rutas de acceso de /dev/mapper para cada disco.

#### <a name="configure-raid-on-top-of-the-encrypted-layer-of-the-disks"></a>Configuración de RAID sobre la capa cifrada de los discos
```bash
mdadm --create /dev/md10 \
--level 0 \
--raid-devices=4 \
/dev/mapper/c49ff535-1df9-45ad-9dad-f0846509f052 \
/dev/mapper/6712ad6f-65ce-487b-aa52-462f381611a1 \
/dev/mapper/ea607dfd-c396-48d6-bc54-603cf741bc2a \
/dev/mapper/4159c60a-a546-455b-985f-92865d51158c
```
![Información para RAID configurado mediante el comando mdadm](./media/disk-encryption/lvm-raid-on-crypt/019-lvm-raid-md-creation.png)

>[!NOTE] 
>Aquí, los nombres de /dev/mapper/device se deben reemplazar por los valores reales en función de la salida de **lsblk**.

### <a name="checkmonitor-raid-creation"></a>Comprobación/supervisión de la creación de RAID
```bash
watch -n1 cat /proc/mdstat
mdadm --examine /dev/mapper/[]
mdadm --detail /dev/md10
```
![Estado de RAID](./media/disk-encryption/lvm-raid-on-crypt/020-lvm-raid-md-details.png)

### <a name="create-a-file-system-on-top-of-the-new-raid-device"></a>Creación de un sistema de archivos sobre el nuevo dispositivo RAID
```bash
mkfs.ext4 /dev/md10
```

Cree un punto de montaje para el sistema de archivos, agregue el nuevo sistema de archivos a /etc/fstab y móntelo:

```bash
for device in md10; do diskuuid="$(blkid -s UUID -o value /dev/${device})"; \
mkdir /raiddata; \
echo "UUID=${diskuuid} /raiddata ext4 defaults,nofail 0 0" >> /etc/fstab; \
mount -a; \
done
```

Verifique que el nuevo sistema de archivos se ha montado:

```bash
lsblk -fs
df -h
```
![Información para sistemas de archivos montados](./media/disk-encryption/lvm-raid-on-crypt/021-lvm-raid-lsblk-md-details.png)

Es importante asegurarse de que se agrega la opción **nofail** a las opciones de punto de montaje de los volúmenes RAID creados sobre un dispositivo cifrado mediante Azure Disk Encryption. Esto evita que el sistema operativo se bloquee durante el proceso de arranque (o en modo de mantenimiento).

Si no utiliza la opción **nofail**:

- El sistema operativo nunca llegará a la fase en la que se inicia Azure Disk Encryption y los discos de datos se desbloquean y se montan.
- Los discos cifrados se desbloquearán al final del proceso de arranque. Los volúmenes de RAID y los sistemas de archivos estarán montados automáticamente hasta que Azure Disk Encryption los desbloquee.

Puede probar reiniciar la VM y validar que los sistemas de archivos también se monten de forma automática después del tiempo de arranque. Este proceso puede tardar varios minutos, según el número y el tamaño de los sistemas de archivos.

```bash
shutdown -r now
```

Y cuando pueda iniciar sesión:

```bash
lsblk
df -h
```
## <a name="next-steps"></a>Pasos siguientes

- [Solución de problemas de Azure Disk Encryption](disk-encryption-troubleshooting.md)
