---
title: Uso de una máquina virtual Linux de solución de problemas con la CLI de Azure | Microsoft Docs
description: Aprenda a solucionar problemas de la máquina virtual Linux mediante la conexión del disco del sistema operativo a una máquina virtual de recuperación mediante la CLI de Azure.
services: virtual-machines-linux
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/16/2017
ms.author: genli
ms.openlocfilehash: 191ea575ed8ce84d2d96227bf93cc4890edd00de
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400490"
---
# <a name="troubleshoot-a-linux-vm-by-attaching-the-os-disk-to-a-recovery-vm-with-the-azure-cli"></a>Solución de problemas de una máquina virtual Linux mediante la conexión del disco del sistema operativo a una máquina virtual de recuperación mediante la CLI de Azure
Si la máquina virtual Linux se encuentra un error de disco o de arranque, deberá realizar los pasos para solucionar problemas en el propio disco duro virtual. Un ejemplo habitual sería una entrada no válida en `/etc/fstab` que impide que la máquina virtual se pueda arrancar correctamente. En este artículo se detalla cómo utilizar la CLI de Azure para conectar el disco duro virtual a otra máquina virtual Linux para solucionar los errores y, posteriormente, volver a crear la máquina virtual original. 

## <a name="recovery-process-overview"></a>Introducción al proceso de recuperación
El proceso de solución de problemas es el siguiente:

1. Detenga la máquina virtual afectada.
1. Realice una instantánea del disco del sistema operativo de la máquina virtual.
1. Cree un disco de la instantánea del disco del sistema operativo.
1. Conecte y monte el nuevo disco del sistema operativo en otra máquina virtual Linux con el fin de solucionar problemas.
1. Conéctese a la máquina virtual de solución de problemas. Edite los archivos o ejecute cualquier herramienta necesaria para solucionar los problemas del nuevo disco del sistema operativo.
1. Desmonte y desconecte el disco duro virtual de la máquina virtual de solución de problemas.
1. Cambie el disco del sistema operativo de la máquina virtual afectada.

Para realizar estos pasos para la solución de problemas, es preciso tener instalada la [CLI de Azure](/cli/azure/install-az-cli2) más reciente y haber iniciado sesión en una cuenta de Azure mediante [az login](/cli/azure/reference-index).

> [!Important]
> Los scripts de este artículo solo se aplican a las máquinas virtuales que usan [Managed Disks](../managed-disks-overview.md). 

En los ejemplos siguientes, reemplace los nombres de parámetros por los suyos propios, como `myResourceGroup` y `myVM`.

## <a name="determine-boot-issues"></a>Determinación de los problemas de arranque
Examine la salida de serie para determinar por qué la máquina virtual no es capaz de arrancar correctamente. Un ejemplo habitual es una entrada no válida en `/etc/fstab` o el disco duro virtual subyacente que se va a eliminar o mover.

Obtenga los registros de arranque con [az vm boot-diagnostics get-boot-log](/cli/azure/vm/boot-diagnostics). En el ejemplo siguiente se obtiene la salida de serie de la máquina virtual llamada `myVM` en el grupo de recursos `myResourceGroup`:

```azurecli
az vm boot-diagnostics get-boot-log --resource-group myResourceGroup --name myVM
```

Revise la salida de serie para determinar por qué la máquina virtual no arranca correctamente. Si la salida de serie no proporciona ninguna indicación, debe revisar los archivos de registro en `/var/log` una vez que tenga el disco duro virtual conectado a una máquina virtual de solución de problemas.

## <a name="stop-the-vm"></a>Parada de la máquina virtual

En el ejemplo siguiente se detiene la máquina virtual llamada `myVM` del grupo de recursos `myResourceGroup`:

```azurecli
az vm stop --resource-group MyResourceGroup --name MyVm
```
## <a name="take-a-snapshot-from-the-os-disk-of-the-affected-vm"></a>Realización de una instantánea del disco del sistema operativo de la máquina virtual afectada

Una instantánea es una copia completa de solo lectura de un disco duro virtual. No se puede adjuntar a una máquina virtual. En el siguiente paso, crearemos un disco a partir de esta instantánea. En el ejemplo siguiente se crea una instantánea llamada `mySnapshot` del disco del sistema operativo de la máquina virtual "myVM". 

```azurecli
#Get the OS disk Id 
$osdiskid=(az vm show -g myResourceGroup -n myVM --query "storageProfile.osDisk.managedDisk.id" -o tsv)

#creates a snapshot of the disk
az snapshot create --resource-group myResourceGroupDisk --source "$osdiskid" --name mySnapshot
```
## <a name="create-a-disk-from-the-snapshot"></a>Creación de un disco a partir de la instantánea

Este script crea un disco administrado llamado `myOSDisk` desde la instantánea denominada `mySnapshot`.  

```azurecli
#Provide the name of your resource group
$resourceGroup="myResourceGroup"

#Provide the name of the snapshot that will be used to create Managed Disks
$snapshot="mySnapshot"

#Provide the name of the Managed Disk
$osDisk="myNewOSDisk"

#Provide the size of the disks in GB. It should be greater than the VHD file size.
$diskSize=128

#Provide the storage type for Managed Disk. Premium_LRS or Standard_LRS.
$storageType="Premium_LRS"

#Provide the OS type
$osType="linux"

#Get the snapshot Id 
$snapshotId=(az snapshot show --name $snapshot --resource-group $resourceGroup --query id -o tsv)

# Create a new Managed Disks using the snapshot Id.

az disk create --resource-group $resourceGroup --name $osDisk --sku $storageType --size-gb $diskSize --source $snapshotId

```

Si el grupo de recursos y la instantánea de origen no están en la misma región, recibirá un error de que no se encuentra el recurso al ejecutar `az disk create`. En este caso, debe especificar `--location <region>` para crear el disco en la misma región que la instantánea de origen.

Ahora ya tiene una copia del disco del sistema operativo original. Puede montar este nuevo disco en otra máquina virtual Windows con el fin de solucionar problemas.

## <a name="attach-the-new-virtual-hard-disk-to-another-vm"></a>Conexión del nuevo disco duro virtual a otra máquina virtual
Para los pasos siguientes, se usa otra máquina virtual con el fin de solucionar problemas. Conecte el disco a esta máquina virtual de solución de problemas para examinar y modificar el contenido del disco. Este proceso permite corregir los errores de configuración o revisar otros archivos de registro de la aplicación o el sistema.

Este script asocia el disco `myNewOSDisk` a la máquina virtual `MyTroubleshootVM`:

```azurecli
# Get ID of the OS disk that you just created.
$myNewOSDiskid=(az disk show -g $resourceGroup -n $osDisk --query id -o tsv)

# Attach the disk to the troubleshooting VM
az vm disk attach --disk $myNewOSDiskid --resource-group $resourceGroup --size-gb $diskSize --sku $storageType --vm-name MyTroubleshootVM
```
## <a name="mount-the-attached-data-disk"></a>Montaje del disco de datos conectado

> [!NOTE]
> Los siguientes ejemplos detallan los pasos necesarios en una máquina virtual Ubuntu. Si usas una distribución de Linux diferente como Red Hat Enterprise Linux o SUSE, el registro de ubicaciones del archivo de registro y los comandos `mount` pueden ser ligeramente diferentes. Consulte la documentación para su distribución específica para los cambios apropiados en los comandos.

1. SSH en la máquina virtual de solución de problemas con las credenciales apropiadas. Si este disco es el primer disco de datos conectado a la máquina virtual de solución de problemas, es probable que el disco se conecte a `/dev/sdc`. Use `dmesg` para ver los discos conectados:

    ```bash
    dmesg | grep SCSI
    ```

    La salida es similar a la del ejemplo siguiente:

    ```bash
    [    0.294784] SCSI subsystem initialized
    [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
    [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
    [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
    [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
    ```

    En el ejemplo anterior, el disco del sistema operativo está en `/dev/sda` y el disco temporal que se proporciona para cada máquina virtual está en `/dev/sdb`. Si hubiera varios discos de datos, deben estar en `/dev/sdd`, `/dev/sde`, y así sucesivamente.

2. Cree un directorio para montar el disco duro virtual existente. El ejemplo siguiente permite crear un directorio llamado `troubleshootingdisk`:

    ```bash
    sudo mkdir /mnt/troubleshootingdisk
    ```

3. Si tiene varias particiones en el disco duro virtual existente, monte la partición requerida. En el ejemplo siguiente, se monta la primera partición principal en `/dev/sdc1`:

    ```bash
    sudo mount /dev/sdc1 /mnt/troubleshootingdisk
    ```

    > [!NOTE]
    > El procedimiento recomendado consiste en montar los discos de datos en máquinas virtuales en Azure con el identificador único universal (UUID) del disco duro virtual. En este breve escenario de solución de problemas, no es necesario montar el disco duro virtual con el UUID. Sin embargo, en circunstancias normales, editar `/etc/fstab` para montar los discos duros virtuales mediante el nombre de dispositivo en lugar del UUID puede provocar que la máquina virtual no se pueda arrancar.


## <a name="fix-issues-on-the-new-os-disk"></a>Corrección de problemas en el nuevo disco del sistema operativo
Con el disco duro virtual existente montado, ahora puede realizar todos los pasos de mantenimiento y solución de problemas según sea necesario. Una vez que se han resuelto los problemas, continúe con los pasos siguientes.


## <a name="unmount-and-detach-the-new-os-disk"></a>Desmontaje y desconexión del nuevo disco del sistema operativo
Una vez resueltos los errores, desmonte y desconecte el disco duro virtual existente de la máquina virtual de solución de problemas. No se podrá usar el disco duro virtual en ninguna otra máquina virtual hasta que se libere la concesión que conecta el disco duro virtual a la máquina virtual de solución de problemas.

1. Desde la sesión SSH a la máquina virtual de solución de problemas, desmonte el disco duro virtual existente. Cambie primero el directorio primario del punto de montaje:

    ```bash
    cd /
    ```

    Ahora, desmonte el disco duro virtual existente. El siguiente ejemplo desmonta el dispositivo en `/dev/sdc1`:

    ```bash
    sudo umount /dev/sdc1
    ```

2. Ahora, desconecte el disco duro virtual de la máquina virtual. Salga de la sesión SSH a la máquina virtual de solución de problemas:

    ```azurecli
    az vm disk detach -g MyResourceGroup --vm-name MyTroubleShootVm --name myNewOSDisk
    ```

## <a name="change-the-os-disk-for-the-affected-vm"></a>Cambio del disco del sistema operativo de la máquina virtual afectada

Puede usar la CLI de Azure para intercambiar los discos del sistema operativo. No es necesario eliminar ni volver a crear la máquina virtual.

En este ejemplo se detiene la máquina virtual `myVM` y se asigna el disco denominado `myNewOSDisk` como nuevo disco del sistema operativo.

```azurecli
# Stop the affected VM
az vm stop -n myVM -g myResourceGroup

# Get ID of the OS disk that is repaired.
$myNewOSDiskid=(az vm show -g $resourceGroup -n $osDisk --query id -o tsv)

# Change the OS disk of the affected VM to "myNewOSDisk"
az vm update -g myResourceGroup -n myVM --os-disk $myNewOSDiskid

# Start the VM
az vm start -n myVM -g myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes
Si tiene problemas para conectarse a la máquina virtual, consulte [Solución de problemas de conexiones SSH a una máquina virtual Linux de Azure](troubleshoot-ssh-connection.md). Para problemas con el acceso a aplicaciones que se ejecutan en su máquina virtual, consulte [Solucionar problemas de conectividad de aplicaciones en una máquina virtual de Linux en Azure](troubleshoot-app-connection.md).

