---
title: 'Comprobación del estado de cifrado para Linux: Azure Disk Encryption'
description: En este artículo se proporcionan instrucciones para comprobar el estado de cifrado del nivel de plataforma y del sistema operativo.
author: kailashmsft
ms.service: security
ms.topic: how-to
ms.author: kaib
ms.date: 03/11/2020
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 1d99b2a69f1cfd31216ca1058d5bc6825be83bcd
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503521"
---
# <a name="verify-encryption-status-for-linux"></a>Comprobación del estado de cifrado para Linux 

El ámbito de este artículo es comprobar el estado de cifrado de una máquina virtual mediante distintos métodos: Azure Portal, PowerShell, la CLI de Azure o el sistema operativo de la máquina virtual (VM). 

Puede comprobar el estado de cifrado durante o después del cifrado, para ello puede:

- Comprobar los discos conectados a una máquina virtual determinada. 
- Consultar la configuración de cifrado en cada disco tanto si el disco está conectado o como si no.

Este escenario se aplica a las extensiones de pase doble y de pase único de Azure Disk Encryption. Las distribuciones de Linux son el único entorno para este escenario.

>[!NOTE] 
>Vamos a usar variables a lo largo de todo el artículo. Reemplace los valores en consecuencia.

## <a name="portal"></a>Portal

En Azure Portal, dentro de la sección **Extensiones**, seleccione en la lista la extensión Azure Disk Encryption. La información del **Mensaje de estado** indica el estado de cifrado actual:

![Comprobación del portal con el estado, la versión y el mensaje de estado resaltados](./media/disk-encryption/verify-encryption-linux/portal-check-001.png)

En la lista de extensiones, verá la versión de la extensión de Azure Disk Encryption correspondiente. La versión 0. x corresponde a Azure Disk Encryption de pase doble y la versión 1. x corresponde a Azure Disk Encryption de pase único.

Puede obtener más detalles seleccionando primero la extensión y después **Ver estado detallado**. Se mostrará un estado más detallado del proceso de cifrado en formato JSON.

![Comprobación del portal con el vínculo "Ver estado detallado" resaltado](./media/disk-encryption/verify-encryption-linux/portal-check-002.png)

![Estado detallado en formato JSON](./media/disk-encryption/verify-encryption-linux/portal-check-003.png)

Otra manera de validar el estado de cifrado es examinar la sección **Configuración del disco**.

![Estado de cifrado del disco del sistema operativo y los discos de datos](./media/disk-encryption/verify-encryption-linux/portal-check-004.png)

>[!NOTE] 
> Este estado significa que los discos tienen una configuración de cifrado marcada, no que se cifraron realmente en el nivel de sistema operativo.
>
> Intencionadamente, los discos se marcan primero y se cifran más adelante. Si se produce un error en el proceso de cifrado, los discos pueden acabar marcados pero no cifrados. 
>
> Para confirmar que los discos están realmente cifrados, puede comprobar el cifrado de cada disco en el nivel de sistema operativo.

## <a name="powershell"></a>PowerShell

Puede validar el estado de cifrado *general* de una máquina virtual cifrada mediante los siguientes comandos de PowerShell:

```azurepowershell
   $VMNAME="VMNAME"
   $RGNAME="RGNAME"
   Get-AzVmDiskEncryptionStatus -ResourceGroupName  ${RGNAME} -VMName ${VMNAME}
```
![Estado de cifrado general en PowerShell](./media/disk-encryption/verify-encryption-linux/verify-status-ps-01.png)

Puede capturar la configuración de cifrado de cada disco mediante los siguientes comandos de PowerShell.

### <a name="single-pass"></a>Pase único
En un pase único, la configuración de cifrado se marca en cada uno de los discos (sistema operativo y datos). Puede capturar la configuración de cifrado de un disco del sistema operativo en un pase único, como se indica a continuación:

``` powershell
$RGNAME = "RGNAME"
$VMNAME = "VMNAME"

$VM = Get-AzVM -Name ${VMNAME} -ResourceGroupName ${RGNAME}  
 $Sourcedisk = Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName $VM.StorageProfile.OsDisk.Name
 Write-Host "============================================================================================================================================================="
 Write-Host "Encryption Settings:"
 Write-Host "============================================================================================================================================================="
 Write-Host "Enabled:" $Sourcedisk.EncryptionSettingsCollection.Enabled
 Write-Host "Version:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettingsVersion
 Write-Host "Source Vault:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SourceVault.Id
 Write-Host "Secret URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl
 Write-Host "Key URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl
 Write-Host "============================================================================================================================================================="
```
![Configuración de cifrado para un disco de sistema operativo](./media/disk-encryption/verify-encryption-linux/verify-os-single-ps-001.png)

Si el disco no tiene la configuración de cifrado marcada, la salida estará vacía:

![Salida vacía](./media/disk-encryption/verify-encryption-linux/os-encryption-settings-2.png)

Use los siguientes comandos para capturar la configuración de cifrado para los discos de datos:

```azurepowershell
$RGNAME = "RGNAME"
$VMNAME = "VMNAME"

$VM = Get-AzVM -Name ${VMNAME} -ResourceGroupName ${RGNAME}
 clear
 foreach ($i in $VM.StorageProfile.DataDisks|ForEach-Object{$_.Name})
 {
 Write-Host "============================================================================================================================================================="
 Write-Host "Encryption Settings:"
 Write-Host "============================================================================================================================================================="
 Write-Host "Checking Disk:" $i
 $Disk=(Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName $i)
 Write-Host "Encryption Enable: " $Sourcedisk.EncryptionSettingsCollection.Enabled
 Write-Host "Encryption KeyEncryptionKey: " $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl;
 Write-Host "Encryption DiskEncryptionKey: " $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl;
 Write-Host "============================================================================================================================================================="
 }
```
![Configuración de cifrado para discos de datos](./media/disk-encryption/verify-encryption-linux/verify-data-single-ps-001.png)

### <a name="dual-pass"></a>Pase doble
En el pase doble, la configuración de cifrado se marca en el modelo de máquina virtual y no en cada disco individual.

Para comprobar que la configuración de cifrado se ha marcado en un pase doble, puede usar los siguientes comandos:

```azurepowershell
$RGNAME = "RGNAME"
$VMNAME = "VMNAME"

$vm = Get-AzVm -ResourceGroupName ${RGNAME} -Name ${VMNAME};
$Sourcedisk = Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName $VM.StorageProfile.OsDisk.Name
clear
Write-Host "============================================================================================================================================================="
Write-Host "Encryption Settings:"
Write-Host "============================================================================================================================================================="
Write-Host "Enabled:" $Sourcedisk.EncryptionSettingsCollection.Enabled
Write-Host "Version:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettingsVersion
Write-Host "Source Vault:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SourceVault.Id
Write-Host "Secret URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl
Write-Host "Key URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl
Write-Host "============================================================================================================================================================="
```
![Configuración de cifrado en un pase doble](./media/disk-encryption/verify-encryption-linux/verify-dual-ps-001.png)

### <a name="unattached-disks"></a>Discos sin conectar

Compruebe la configuración de cifrado de los discos que no están conectados a una máquina virtual.

### <a name="managed-disks"></a>Discos administrados
```powershell
$Sourcedisk = Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName ${TARGETDISKNAME}
Write-Host "============================================================================================================================================================="
Write-Host "Encryption Settings:"
Write-Host "============================================================================================================================================================="
Write-Host "Enabled:" $Sourcedisk.EncryptionSettingsCollection.Enabled
Write-Host "Version:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettingsVersion
Write-Host "Source Vault:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SourceVault.Id
Write-Host "Secret URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl
Write-Host "Key URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl
Write-Host "============================================================================================================================================================="
```
## <a name="azure-cli"></a>Azure CLI

Puede comprobar el estado de cifrado *general* de una máquina virtual cifrada con los siguientes comandos de la CLI de Azure:

```bash
VMNAME="VMNAME"
RGNAME="RGNAME"
az vm encryption show --name ${VMNAME} --resource-group ${RGNAME} --query "substatus"
```
![Estado general de cifrado desde la CLI de Azure ](./media/disk-encryption/verify-encryption-linux/verify-gen-cli.png)

### <a name="single-pass"></a>Pase único
Puede validare la configuración de cifrado de cada disco mediante los siguientes comandos de la CLI de Azure:

```bash
az vm encryption show -g ${RGNAME} -n ${VMNAME} --query "disks[*].[name, statuses[*].displayStatus]"  -o table
```

![Configuración del cifrado de datos](./media/disk-encryption/verify-encryption-linux/data-encryption-settings-2.png)

>[!IMPORTANT]
> Si el disco no tiene la configuración de cifrado marcada, verá el texto **El disco no está cifrado**.

Use los comandos siguientes para obtener la configuración detallada de estado y cifrado.

Disco del sistema operativo:

```bash
RGNAME="RGNAME"
VMNAME="VNAME"

disk=`az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.osDisk.name -o tsv`
for disk in $disk; do \
echo "============================================================================================================================================================="
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${disk} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
done
```

![Configuración detallada de estado y cifrado para el disco del sistema operativo](./media/disk-encryption/verify-encryption-linux/os-single-cli.png)

Discos de datos:

```bash
RGNAME="RGNAME"
VMNAME="VMNAME"
az vm encryption show --name ${VMNAME} --resource-group ${RGNAME} --query "substatus"

for disk in `az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.dataDisks[].name -o tsv`; do \
echo "============================================================================================================================================================="; \
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${disk} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
done
```

![Configuración detallada de estado y cifrado para los discos de datos](./media/disk-encryption/verify-encryption-linux/data-single-cli.png)

### <a name="dual-pass"></a>Pase doble

``` bash
az vm encryption show --name ${VMNAME} --resource-group ${RGNAME} -o table
```

![Configuración general de cifrado para el pase doble mediante la CLI de Azure](./media/disk-encryption/verify-encryption-linux/verify-gen-dual-cli.png)

También puede comprobar la configuración de cifrado en el perfil de almacenamiento del modelo de máquina virtual del disco del sistema operativo:

```bash
disk=`az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.osDisk.name -o tsv`
for disk in $disk; do \
echo "============================================================================================================================================================="; \
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${disk} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
done
```

![Perfil de máquina virtual para el pase doble mediante la CLI de Azure](./media/disk-encryption/verify-encryption-linux/verify-vm-profile-dual-cli.png)

### <a name="unattached-disks"></a>Discos sin conectar

Compruebe la configuración de cifrado de los discos que no están conectados a una máquina virtual.

### <a name="managed-disks"></a>Discos administrados

```bash
RGNAME="RGNAME"
TARGETDISKNAME="DISKNAME"
echo "============================================================================================================================================================="
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
```
### <a name="unmanaged-disks"></a>Discos no administrados

Los discos no administrados son archivos VHD almacenados como blobs en páginas en cuentas de Azure Storage.

Para obtener los detalles de un disco específico, tiene que proporcionar:

- El identificador de la cuenta de almacenamiento que contenga el disco.
- Una cadena de conexión para esa cuenta de almacenamiento particular.
- El nombre del contenedor que almacena el disco.
- El nombre del disco.

Este comando muestra todos los identificadores de todas las cuentas de almacenamiento:

```bash
az storage account list --query [].[id] -o tsv
```
Los identificadores de la cuenta de almacenamiento se muestran en este formato:

/subscriptions/\<subscription id>/resourceGroups/\<resource group name>/providers/Microsoft.Storage/storageAccounts/\<storage account name>

Seleccione el identificador adecuado y almacénelo en una variable:
```bash
id="/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/<storage account name>"
```

Este comando obtiene la cadena de conexión para una cuenta de almacenamiento concreta y la almacena en una variable:

```bash
ConnectionString=$(az storage account show-connection-string --ids $id --query connectionString -o tsv)
```

El siguiente comando muestra todos los contenedores en una cuenta de almacenamiento:
```bash
az storage container list --connection-string $ConnectionString --query [].[name] -o tsv
```
El contenedor que se usa para los discos se denomina normalmente "vhds".

Almacene el nombre del contenedor en una variable: 
```bash
ContainerName="name of the container"
```

Use este comando para enumerar todos los blobs de un contenedor determinado:
```bash 
az storage blob list -c ${ContainerName} --connection-string $ConnectionString --query [].[name] -o tsv
```
Elija el disco que desea consultar y almacene su nombre en una variable:
```bash
DiskName="diskname.vhd"
```
Consulte la configuración de cifrado de disco:
```bash
az storage blob show -c ${ContainerName} --connection-string ${ConnectionString} -n ${DiskName} --query metadata.DiskEncryptionSettings
```

## <a name="operating-system"></a>Sistema operativo
Compruebe si las particiones de disco de datos están cifradas (y si el disco del sistema operativo no lo está).

Cuando una partición o un disco están cifrados, se muestra como un tipo **crypt**. Cuando no lo están, se muestra como un tipo **part/disk**.

``` bash
lsblk
```

![Nivel de cifrado de sistema operativo para una partición](./media/disk-encryption/verify-encryption-linux/verify-os-crypt-layer.png)

Puede obtener más detalles con la siguiente variante **lsblk**. 

Verá una capa de tipo **crypt** que está montada por la extensión. En el ejemplo siguiente se muestran los volúmenes lógicos y los discos normales con **crypto\_LUKS FSTYPE**.

```bash
lsblk -o NAME,TYPE,FSTYPE,LABEL,SIZE,RO,MOUNTPOINT
```
![Capa de cifrado del sistema operativo para volúmenes lógicos y discos normales](./media/disk-encryption/verify-encryption-linux/verify-os-crypt-layer-2.png)

Como paso adicional, también puede comprobar si el disco de datos tiene claves cargadas:

``` bash
cryptsetup luksDump /dev/VGNAME/LVNAME
```

``` bash
cryptsetup luksDump /dev/sdd1
```

Y puede comprobar qué dispositivos **dm** se muestran como **crypt**:

```bash
dmsetup ls --target crypt
```

## <a name="next-steps"></a>Pasos siguientes

- [Solución de problemas de Azure Disk Encryption](disk-encryption-troubleshooting.md)
