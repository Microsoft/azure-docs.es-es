---
title: Conexión de un disco de datos a una máquina virtual Linux
description: Use el portal para conectar un disco de datos nuevo o existente a una máquina virtual Linux.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 07/12/2018
ms.author: cynthn
ms.subservice: disks
ms.openlocfilehash: 8bafde086febce78ca2715ab77c8a071467b986b
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87825470"
---
# <a name="use-the-portal-to-attach-a-data-disk-to-a-linux-vm"></a>Uso del portal para conectar un disco de datos a una máquina virtual Linux 
En este artículo se muestra cómo adjuntar discos nuevos y existentes a una máquina virtual Linux a través del Portal de Azure. También puede [adjuntar un disco de datos a una máquina virtual con Windows en Azure Portal](../windows/attach-managed-disk-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). 

Antes de conectar discos a la VM, revise estas sugerencias:

* El tamaño de la máquina virtual controla cuántos discos de datos puede conectar. Para obtener más información, consulte [Tamaños de máquinas virtuales](../sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
* Los discos conectados a las máquinas virtuales son, en realidad, archivos .vhd almacenados en Azure. Para más información, consulte [Introducción a Azure Managed Disks](managed-disks-overview.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
* Después de adjuntar el disco, deberá [conectarse a la VM de Linux para montar el nuevo disco](#connect-to-the-linux-vm-to-mount-the-new-disk).


## <a name="find-the-virtual-machine"></a>Búsqueda de la máquina virtual
1. Vaya a [Azure Portal](https://portal.azure.com/) para encontrar la máquina virtual. Busque y seleccione **Máquinas virtuales**.
2. Seleccione la VM en la lista.
3. En la barra lateral de la página **Máquinas virtuales**, en **configuración**, elija **Discos**.
   
    ![Abrir configuración de disco](./media/attach-disk-portal/find-disk-settings.png)


## <a name="attach-a-new-disk"></a>Conexión de un disco nuevo

1. En el panel **Discos**, haga clic en **+ Add data disk** (+ Agregar disco de datos).
2. Haga clic en el menú desplegable para el **nombre** y seleccione **Crear disco**:

    ![Creación de disco administrado de Azure](./media/attach-disk-portal/create-new-md.png)

3. Escriba un nombre para el disco administrado. Revise la configuración predeterminada, actualice según sea necesario y, luego, haga clic en **Crear**.
   
   ![Revisar configuración de disco](./media/attach-disk-portal/create-new-md-settings.png)

4. Haga clic en **Guardar** para crear el disco administrado y actualizar la configuración de VM:

   ![Guardado del nuevo Azure Managed Disk](./media/attach-disk-portal/confirm-create-new-md.png)

5. Una vez que Azure crea el disco y lo adjunta a la máquina virtual, el nuevo disco aparece en la configuración de disco de la máquina virtual en el apartado **Discos de datos**. Como los discos administrados son un recurso de nivel superior, el disco aparece en la raíz del grupo de recursos:

   ![Azure Managed Disk en grupo de recursos](./media/attach-disk-portal/view-md-resource-group.png)

## <a name="attach-an-existing-disk"></a>un disco existente
1. En el panel **Discos**, haga clic en **+ Add data disk** (+ Agregar disco de datos).
2. Haga clic en el menú desplegable de **nombre** para ver una lista de los discos administrados existentes a los que puede obtener acceso con la suscripción de Azure. Seleccione el disco administrado que va a conectar:

   ![Conexión de un disco administrado de Azure existente](./media/attach-disk-portal/select-existing-md.png)

3. Haga clic en **Guardar** para conectar el disco administrado existente y actualizar la configuración de la VM:
   
   ![Guardado de las actualizaciones del disco administrado de Azure](./media/attach-disk-portal/confirm-attach-existing-md.png)

4. Una vez que Azure adjunta el disco a la máquina virtual,este aparece en la configuración del disco de la máquina virtual en el apartado **Discos de datos**.

## <a name="connect-to-the-linux-vm-to-mount-the-new-disk"></a>Conexión a la máquina virtual con Linux para montar el nuevo disco
Para la partición, el formato y el montaje del disco nuevo para que la máquina virtual con Linux pueda usarlo, necesita SSH en la VM. Para más información, consulte [Creación de claves SSH en Linux y Mac para máquinas virtuales de Linux en Azure](mac-create-ssh-keys.md). En el siguiente ejemplo se realiza la conexión a una máquina virtual con la entrada DNS pública de *mypublicdns.westus.cloudapp.azure.com* con el nombre de usuario *azureuser*: 

```bash
ssh azureuser@mypublicdns.westus.cloudapp.azure.com
```

Una vez conectado a la máquina virtual, está listo para conectar un disco. Primero, busquemos el disco con `dmesg` (el método usado para detectar el nuevo disco puede variar). En el ejemplo siguiente se usa dmesg para filtrar por los discos *SCSI*:

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

En este caso, *sdc* es el disco que queremos. 

### <a name="partition-a-new-disk"></a>Partición de un disco nuevo
Si usa un disco existente que contiene datos, omita el paso de montaje de disco. Si está conectando un disco nuevo, deberá crear particiones en el disco.

> [!NOTE]
> Se recomienda usar las versiones más recientes de fdisk o parted que estén disponibles para su distribución.

Cree particiones del disco con `fdisk`. Si el tamaño del disco es 2 tebibytes (TiB) o mayor, debe usar particionamiento GPT; puede usar `parted` para las particiones GPT. Si el tamaño del disco es de menos de 2 TiB, puede usar particionamiento MBR o GPT. Conviértalo en un disco principal en la partición 1 y acepte los demás valores predeterminados. En el ejemplo siguiente se inicia el proceso `fdisk` en */dev/sdc*:

```bash
sudo fdisk /dev/sdc
```

Use el comando `n` para agregar una nueva partición. En este ejemplo, también elegimos `p` para una partición principal y aceptamos el resto de los valores predeterminados. El resultado será similar al ejemplo siguiente:

```bash
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0x2a59b123.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-10485759, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-10485759, default 10485759):
Using default value 10485759
```

Para imprimir la tabla de particiones, escriba `p` y luego use `w` para escribir la tabla en el disco y salga. El resultado debería similar al ejemplo siguiente:

```bash
Command (m for help): p

Disk /dev/sdc: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders, total 10485760 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x2a59b123

   Device Boot      Start         End      Blocks   Id  System
/dev/sdc1            2048    10485759     5241856   83  Linux

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

Ahora, escriba un sistema de archivos en la partición con el comando `mkfs`. Especifique el tipo de sistema de archivos y el nombre del dispositivo. En el ejemplo siguiente se crea un sistema de archivos *ext4* en la partición */dev/sdc1* que se creó en los pasos anteriores:

```bash
sudo mkfs -t ext4 /dev/sdc1
```

La salida es similar a la del ejemplo siguiente:

```bash
mke2fs 1.42.9 (4-Feb-2014)
Discarding device blocks: done
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
327680 inodes, 1310464 blocks
65523 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=1342177280
40 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
    32768, 98304, 163840, 229376, 294912, 819200, 884736
Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

#### <a name="alternate-method-using-parted"></a>Método alternativo con parted
La utilidad fdisk requiere una entrada interactiva y, por lo tanto, no resulta idónea para usarse en scripts de automatización. Sin embargo, se puede generar un script para la utilidad [parted](https://www.gnu.org/software/parted/) y, por lo tanto, resulta mejor en los escenarios de automatización. La utilidad parted se puede utilizar para crear particiones y formatear un disco de datos. En el tutorial siguiente, usamos un nuevo disco de datos /dev/sdc y lo formateamos con el sistema de archivos [XFS](https://xfs.wiki.kernel.org/).
```bash
sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo mkfs.xfs /dev/sdc1
partprobe /dev/sdc1
```
Como se indicó anteriormente, usamos la utilidad [partprobe](https://linux.die.net/man/8/partprobe) para garantizar que el kernel reconozca inmediatamente la nueva partición y el sistema de archivos. Si no se usa partprobe, los comandos blkid o lslbk no podrán devolver de forma inmediata el UUID del nuevo sistema de archivos.

### <a name="mount-the-disk"></a>Montaje del disco
Cree un directorio para montar el sistema de archivos mediante `mkdir`. En el ejemplo se crea un directorio en */datadrive*:

```bash
sudo mkdir /datadrive
```

A continuación, use `mount` para montar el sistema de archivos. En el ejemplo siguiente, se monta la partición */dev/sdc1* en el punto de montaje */datadrive*:

```bash
sudo mount /dev/sdc1 /datadrive
```

Para asegurarse de que la unidad se vuelve a montar automáticamente después de reiniciar, debe agregarse al archivo */etc/fstab*. Además, se recomienda encarecidamente que se use el UUID (identificador único global) en */etc/fstab* para hacer referencia a la unidad en lugar de solo el nombre del dispositivo (por ejemplo, */dev/sdc1*). Si el SO detecta un error de disco durante el arranque, con el UUID se evita que se monte el disco incorrecto en una ubicación especificada. A los discos de datos restantes se les asignarán después esos mismos identificadores de dispositivo. Para buscar el UUID de la unidad nueva, use la utilidad `blkid`:

```bash
sudo -i blkid
```

La salida es similar a la siguiente:

```bash
/dev/sda1: UUID="11111111-1b1b-1c1c-1d1d-1e1e1e1e1e1e" TYPE="ext4"
/dev/sdb1: UUID="22222222-2b2b-2c2c-2d2d-2e2e2e2e2e2e" TYPE="ext4"
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="ext4"
```

> [!NOTE]
> La edición incorrecta del archivo **/etc/fstab** puede tener como resultado un sistema que no se pueda arrancar. Si no está seguro, consulte la documentación de distribución para obtener información sobre cómo editar correctamente ese archivo. También se recomienda realizar una copia de seguridad del archivo /etc/fstab antes de editarlo.

Después, abra el archivo */etc/fstab* en un editor de texto como se indica a continuación:

```bash
sudo vi /etc/fstab
```

En este ejemplo, use el valor de UUID para el dispositivo */dev/sdc1* que se creó en los pasos anteriores y el punto de montaje */datadrive*. Agregue la siguiente línea al final del archivo */etc/fstab* :

```bash
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext4   defaults,nofail   1   2
```
Cuando haya terminado, guarde el archivo */etc/fstab* y reinicie el sistema.
> [!NOTE]
> Después, la eliminación de un disco de datos sin editar fstab podría provocar un error en el inicio de la máquina virtual. La mayoría de las distribuciones proporcionan las opciones de fstab *nofail* y/o *nobootwait*. Estas opciones permiten que el sistema arranque incluso si no se monta el disco en el momento del arranque. Consulte la documentación de su distribución para obtener más información sobre estos parámetros.
> 
> La opción *nofail* garantiza que la máquina virtual se inicia incluso si el sistema de archivos está dañado o el disco no existe en tiempo de arranque. Sin esta opción, puede encontrarse con el comportamiento que se describe en [Cannot SSH to Linux VM due to FSTAB errors](/archive/blogs/linuxonazure/cannot-ssh-to-linux-vm-after-adding-data-disk-to-etcfstab-and-rebooting) (No se puede conectar mediante SSH a una máquina virtual Linux debido a errores de FSTAB).

### <a name="trimunmap-support-for-linux-in-azure"></a>Compatibilidad de TRIM/UNMAP con Linux en Azure
Algunos kernels de Linux admiten operaciones TRIM/UNMAP para descartar bloques no usados del disco. Esta característica es especialmente útil en el almacenamiento estándar para informar a Azure de que las páginas eliminadas ya no son válidas y se pueden descartar. Además, le permitirá ahorrar dinero si crea archivos de gran tamaño y luego los elimina.

Hay dos maneras de habilitar la compatibilidad con TRIM en su máquina virtual Linux. Como es habitual, consulte la documentación de distribución para ver el enfoque recomendado:

* Use la opción de montaje `discard` en */etc/fstab*, por ejemplo:

    ```bash
    UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext4   defaults,discard   1   2
    ```
* En algunos casos, la opción `discard` podría tener afectar al rendimiento. Como alternativa, puede ejecutar el comando `fstrim` manualmente desde la línea de comandos o agregarlo a su crontab para ejecutar con regularidad:
  
    **Ubuntu**
  
    ```bash
    sudo apt-get install util-linux
    sudo fstrim /datadrive
    ```
  
    **RHEL/CentOS**

    ```bash
    sudo yum install util-linux
    sudo fstrim /datadrive
    ```

## <a name="next-steps"></a>Pasos siguientes
También puede [conectar un disco de datos](add-disk.md) mediante la CLI de Azure.