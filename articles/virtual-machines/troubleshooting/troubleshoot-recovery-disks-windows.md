---
title: Use a Windows troubleshooting VM with Azure PowerShell (Uso de una máquina virtual Windows de solución de problemas con Azure PowerShell) | Microsoft Docs
description: Aprenda a solucionar problemas de la máquina virtual Windows en Azure mediante la conexión del disco del sistema operativo a una máquina virtual de recuperación mediante Azure PowerShell.
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 08/09/2018
ms.author: genli
ms.openlocfilehash: 1c7b6ee73513c7e51e3bb18c3835a1d395a4c516
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87028268"
---
# <a name="troubleshoot-a-windows-vm-by-attaching-the-os-disk-to-a-recovery-vm-using-azure-powershell"></a>Solución de problemas de una máquina virtual Windows mediante la conexión del disco del sistema operativo a una máquina virtual de recuperación mediante Azure PowerShell
Si la máquina virtual Windows de Azure se encuentra un error de disco o de arranque, deberá realizar los pasos para solucionar problemas en el propio disco. Un ejemplo habitual sería una actualización de aplicación con error que impide que la máquina virtual se pueda arrancar correctamente. En este artículo se detalla cómo utilizar Azure PowerShell para conectar el disco a otra máquina virtual Windows para solucionar los errores y, posteriormente, reparar la máquina virtual original. 

> [!Important]
> Los scripts de este artículo solo se aplican a las máquinas virtuales que usan [Managed Disks](../windows/managed-disks-overview.md). 

 

## <a name="recovery-process-overview"></a>Introducción al proceso de recuperación
Ahora podemos usar Azure PowerShell para cambiar el disco del sistema operativo de una máquina virtual. No es necesario eliminar ni volver a crear la máquina virtual.

El proceso de solución de problemas es el siguiente:

1. Detenga la máquina virtual afectada.
2. Cree una instantánea del disco del sistema operativo de la máquina virtual.
3. Cree un disco de la instantánea del disco del sistema operativo.
4. Conecte el disco como un disco de datos a una máquina virtual de recuperación.
5. Conéctese a la máquina virtual de recuperación. Edite los archivos o ejecute cualquier herramienta necesaria para solucionar los problemas del disco copiado del sistema operativo.
6. Desmonte y desasocie el disco de la máquina virtual de recuperación.
7. Cambie el disco del sistema operativo de la máquina virtual afectada.

Puede usar los comandos de reparación de máquina virtual para automatizar los pasos 1, 2, 3, 4, 6 y 7. Para obtener más documentación e instrucciones, vea [Reparación de una máquina virtual Windows mediante los comandos de reparación de máquinas virtuales de Azure](repair-windows-vm-using-azure-virtual-machine-repair-commands.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Asegúrese de que tiene [la Azure PowerShell más reciente](/powershell/azure/) instalada y con la sesión iniciada en su suscripción:

```powershell
Connect-AzAccount
```

En los ejemplos siguientes, reemplace los nombres de parámetros por los suyos propios. 

## <a name="determine-boot-issues"></a>Determinación de los problemas de arranque
Puede ver una captura de pantalla de la máquina virtual de Azure para ayudar a solucionar problemas de arranque. Esta captura de pantalla puede ayudar a identificar por qué no arranca correctamente una máquina virtual. En el ejemplo siguiente se obtiene la captura de pantalla de la máquina virtual Windows llamada `myVM` en el grupo de recursos `myResourceGroup`:

```powershell
Get-AzVMBootDiagnosticsData -ResourceGroupName myResourceGroup `
    -Name myVM -Windows -LocalPath C:\Users\ops\
```

Revise la captura de pantalla para determinar por qué la máquina virtual no arranca correctamente. Tenga en cuenta todos los mensajes de error o códigos de error específicos proporcionados.

## <a name="stop-the-vm"></a>Parada de la máquina virtual

En el ejemplo siguiente se detiene la máquina virtual llamada `myVM` del grupo de recursos `myResourceGroup`:

```powershell
Stop-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
```

Espere a que la máquina virtual haya finalizado la detención antes de continuar en el paso siguiente.


## <a name="create-a-snapshot-from-the-os-disk-of-the-vm"></a>Creación de una instantánea del disco del sistema operativo de la máquina virtual

En el ejemplo siguiente se crea una instantánea llamada `mySnapshot` del disco del sistema operativo de la máquina virtual "myVM". 

```powershell
$resourceGroupName = 'myResourceGroup' 
$location = 'eastus' 
$vmName = 'myVM'
$snapshotName = 'mySnapshot'  

#Get the VM
$vm = get-azvm `
-ResourceGroupName $resourceGroupName `
-Name $vmName

#Create the snapshot configuration for the OS disk
$snapshot =  New-AzSnapshotConfig `
-SourceUri $vm.StorageProfile.OsDisk.ManagedDisk.Id `
-Location $location `
-CreateOption copy

#Take the snapshot
New-AzSnapshot `
   -Snapshot $snapshot `
   -SnapshotName $snapshotName `
   -ResourceGroupName $resourceGroupName 
```

Una instantánea es una copia completa de solo lectura de un disco duro virtual. No se puede adjuntar a una máquina virtual. En el siguiente paso, crearemos un disco a partir de esta instantánea.

## <a name="create-a-disk-from-the-snapshot"></a>Creación de un disco a partir de la instantánea

Este script crea un disco administrado llamado `newOSDisk` desde la instantánea denominada `mysnapshot`.  

```powershell
#Set the context to the subscription Id where Managed Disk will be created
#You can skip this step if the subscription is already selected

$subscriptionId = 'yourSubscriptionId'

Select-AzSubscription -SubscriptionId $SubscriptionId

#Provide the name of your resource group
$resourceGroupName ='myResourceGroup'

#Provide the name of the snapshot that will be used to create Managed Disks
$snapshotName = 'mySnapshot' 

#Provide the name of the Managed Disk
$diskName = 'newOSDisk'

#Provide the size of the disks in GB. It should be greater than the VHD file size.
$diskSize = '128'

#Provide the storage type for Managed Disk. Premium_LRS or Standard_LRS.
$storageType = 'Standard_LRS'

#Provide the Azure region (e.g. westus) where Managed Disks will be located.
#This location should be same as the snapshot location
#Get all the Azure location using command below:
#Get-AzLocation
$location = 'eastus'

$snapshot = Get-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName 
 
$diskConfig = New-AzDiskConfig -AccountType $storageType -Location $location -CreateOption Copy -SourceResourceId $snapshot.Id
 
New-AzDisk -Disk $diskConfig -ResourceGroupName $resourceGroupName -DiskName $diskName
```
Ahora ya tiene una copia del disco del sistema operativo original. Puede montar este disco en otra máquina virtual Windows con el fin de solucionar problemas.

## <a name="attach-the-disk-to-another-windows-vm-for-troubleshooting"></a>Conexión del disco a otra máquina virtual de Windows para solucionar problemas

Ahora vamos a conectar la copia del disco del sistema operativo original a una máquina virtual como un disco de datos. Este proceso le permite corregir errores de configuración o revisar archivos de registro adicionales de la aplicación o sistema en el disco. En el ejemplo siguiente se conecta el disco denominado `newOSDisk` a la máquina virtual `RecoveryVM`.

> [!NOTE]
> Para conectar el disco, la copia del disco del sistema operativo original y la máquina virtual de recuperación deben estar en la misma ubicación.

```powershell
$rgName = "myResourceGroup"
$vmName = "RecoveryVM"
$location = "eastus" 
$dataDiskName = "newOSDisk"
$disk = Get-AzDisk -ResourceGroupName $rgName -DiskName $dataDiskName 

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName 

$vm = Add-AzVMDataDisk -CreateOption Attach -Lun 0 -VM $vm -ManagedDiskId $disk.Id

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

## <a name="connect-to-the-recovery-vm-and-fix-issues-on-the-attached-disk"></a>Conexión a la máquina virtual de recuperación y corrección de problemas en el disco conectado

1. RDP en la máquina virtual de recuperación con las credenciales apropiadas. En el siguiente ejemplo se descarga el archivo de conexión RDP de la máquina virtual `RecoveryVM` del grupo de recursos `myResourceGroup`, y lo hace en `C:\Users\ops\Documents`"

    ```powershell
    Get-AzRemoteDesktopFile -ResourceGroupName "myResourceGroup" -Name "RecoveryVM" `
        -LocalPath "C:\Users\ops\Documents\myVMRecovery.rdp"
    ```

2. El disco de datos se detecta y conecta automáticamente. Consulte la lista de volúmenes conectados para determinar la letra de unidad de la manera siguiente:

    ```powershell
    Get-Disk
    ```

    La siguiente salida de ejemplo muestra el disco conectado a un disco **2**. (también puede usar `Get-Volume` para ver la letra de unidad):

    ```powershell
    Number   Friendly Name   Serial Number   HealthStatus   OperationalStatus   Total Size   Partition
                                                                                             Style
    ------   -------------   -------------   ------------   -----------------   ----------   ----------
    0        Virtual HD                                     Healthy             Online       127 GB MBR
    1        Virtual HD                                     Healthy             Online       50 GB MBR
    2        newOSDisk                                  Healthy             Online       127 GB MBR
    ```

Una vez montada la copia del disco del sistema operativo original, puede realizar todos los pasos de mantenimiento y solución de problemas según sea necesario. Una vez que se han resuelto los problemas, continúe con los pasos siguientes.

## <a name="unmount-and-detach-original-os-disk"></a>Desmontaje y desconexión del disco duro del sistema operativo
Una vez resueltos los errores, desmonte y desconecte el disco existente de la máquina virtual de recuperación. No se podrá usar el disco en ninguna otra máquina virtual hasta que se libere la concesión que conecta el disco a la máquina virtual de recuperación.

1. Desde dentro de la sesión RDP, desmonte el disco de datos de la máquina virtual de recuperación. Necesita el número de disco del cmdlet `Get-Disk` anterior. A continuación, use `Set-Disk` para establecer el disco como sin conexión:

    ```powershell
    Set-Disk -Number 2 -IsOffline $True
    ```

    Confirme que el disco está actualmente establecido como sin conexión de nuevo con `Get-Disk`. La siguiente salida de ejemplo muestra que el disco está actualmente establecido como sin conexión:

    ```powershell
    Number   Friendly Name   Serial Number   HealthStatus   OperationalStatus   Total Size   Partition
                                                                                             Style
    ------   -------------   -------------   ------------   -----------------   ----------   ----------
    0        Virtual HD                                     Healthy             Online       127 GB MBR
    1        Virtual HD                                     Healthy             Online       50 GB MBR
    2        Msft Virtu...                                  Healthy             Offline      127 GB MBR
    ```

2. Salga de la sesión RDP. En la sesión de Azure PowerShell, elimine el disco denominado `newOSDisk` de la máquina virtual "RecoveryVM".

    ```powershell
    $myVM = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "RecoveryVM"
    Remove-AzVMDataDisk -VM $myVM -Name "newOSDisk"
    Update-AzVM -ResourceGroup "myResourceGroup" -VM $myVM
    ```

## <a name="change-the-os-disk-for-the-affected-vm"></a>Cambio del disco del sistema operativo de la máquina virtual afectada

Puede usar Azure PowerShell para intercambiar los discos del sistema operativo. No es necesario eliminar ni volver a crear la máquina virtual.

En este ejemplo se detiene la máquina virtual `myVM` y se asigna el disco denominado `newOSDisk` como nuevo disco del sistema operativo. 

```powershell
# Get the VM 
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM 

# Make sure the VM is stopped\deallocated
Stop-AzVM -ResourceGroupName myResourceGroup -Name $vm.Name -Force

# Get the new disk that you want to swap in
$disk = Get-AzDisk -ResourceGroupName myResourceGroup -Name newDisk

# Set the VM configuration to point to the new disk  
Set-AzVMOSDisk -VM $vm -ManagedDiskId $disk.Id -Name $disk.Name  -sto

# Update the VM with the new OS disk. Possible values of StorageAccountType include: 'Standard_LRS' and 'Premium_LRS'
Update-AzVM -ResourceGroupName myResourceGroup -VM $vm -StorageAccountType <Type of the storage account >

# Start the VM
Start-AzVM -Name $vm.Name -ResourceGroupName myResourceGroup
```

## <a name="verify-and-enable-boot-diagnostics"></a>Comprobación y habilitación del diagnóstico de arranque

En el ejemplo siguiente se habilita la extensión de diagnóstico en la máquina virtual denominada `myVMDeployed` en el grupo de recursos `myResourceGroup`:

```powershell
$myVM = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVMDeployed"
Set-AzVMBootDiagnostics -ResourceGroupName myResourceGroup -VM $myVM -enable
Update-AzVM -ResourceGroup "myResourceGroup" -VM $myVM
```

## <a name="next-steps"></a>Pasos siguientes
Si tiene problemas para conectarse a la máquina virtual, consulte [Solución de problemas de conexiones RDP a una máquina virtual de Azure](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Para problemas con el acceso a aplicaciones que se ejecutan en su máquina virtual, consulte [Solucionar problemas de conectividad de aplicaciones en una máquina virtual Windows](troubleshoot-app-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Para más información sobre el uso de Resource Manager, consulte [Información general de Azure Resource Manager](../../azure-resource-manager/management/overview.md).
