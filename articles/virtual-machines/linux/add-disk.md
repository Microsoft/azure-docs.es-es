---
title: Agregue un disco de datos a la máquina virtual Linux mediante la CLI de Azure
description: Aprenda a agregar un disco de datos persistente a una máquina virtual de Linux con la CLI de Azure
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 08/20/2020
ms.author: cynthn
ms.subservice: disks
ms.openlocfilehash: 7098744fe012c994e311696a376cd7ed0dc9ac53
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89076623"
---
# <a name="add-a-disk-to-a-linux-vm"></a>Adición de un disco a una máquina virtual de Linux

En este artículo se muestra cómo conectar un disco persistente a la máquina virtual para que se puedan conservar los datos, incluso si la máquina virtual se vuelve a aprovisionar por mantenimiento o cambio de tamaño.


## <a name="attach-a-new-disk-to-a-vm"></a>Conexión de un nuevo disco a una máquina virtual

Si quiere agregar un nuevo disco de datos en la VM, use el comando [az vm disk attach](/cli/azure/vm/disk?view=azure-cli-latest) con el parámetro `--new`. Si la máquina virtual está en una zona de disponibilidad, el disco se crea automáticamente en la misma zona que la máquina virtual. Para obtener más información, consulte [Introducción a las zonas de disponibilidad](../../availability-zones/az-overview.md). En el ejemplo siguiente se crea un disco denominado *myDataDisk* que tiene un tamaño de 50 Gb:

```azurecli
az vm disk attach \
   -g myResourceGroup \
   --vm-name myVM \
   --name myDataDisk \
   --new \
   --size-gb 50
```

## <a name="attach-an-existing-disk"></a>un disco existente

Para conectar un disco existente, busque el identificador del disco y páselo al comando [az vm disk attach](/cli/azure/vm/disk?view=azure-cli-latest). El ejemplo siguiente consulta un disco llamado *myDataDisk* en *myResourceGroup* y, a continuación, lo asocia a la máquina virtual denominada *myVM*:

```azurecli
diskId=$(az disk show -g myResourceGroup -n myDataDisk --query 'id' -o tsv)

az vm disk attach -g myResourceGroup --vm-name myVM --name $diskId
```

## <a name="format-and-mount-the-disk"></a>Formato y montaje del disco

Para la partición, el formato y el montaje del disco nuevo para que la máquina virtual con Linux pueda usarlo, necesita SSH en la VM. Para más información, consulte [Creación de claves SSH en Linux y Mac para máquinas virtuales de Linux en Azure](mac-create-ssh-keys.md). El ejemplo siguiente se conecta a una máquina virtual con la dirección IP pública *10.123.123.25* y el nombre de usuario *azureuser*:

```bash
ssh azureuser@10.123.123.25
```

### <a name="find-the-disk"></a>Buscar el disco

Una vez conectado a la máquina virtual, debe buscar el disco. En este ejemplo, se usa `lsblk` para enumerar los discos. 

```bash
lsblk -o NAME,HCTL,SIZE,MOUNTPOINT | grep -i "sd"
```

La salida es similar a la del ejemplo siguiente:

```bash
sda     0:0:0:0      30G
├─sda1             29.9G /
├─sda14               4M
└─sda15             106M /boot/efi
sdb     1:0:1:0      14G
└─sdb1               14G /mnt
sdc     3:0:0:0      50G
```

Aquí, `sdc` es el disco que queremos, porque es de 50 GB. Si no está seguro de qué disco se trata según el tamaño, puede ir a la página de la máquina virtual en el portal, seleccionar **Discos** y comprobar el número LUN del disco en **Discos de datos**. 


### <a name="format-the-disk"></a>Formato del disco

Formatee el disco con `parted`. Si el tamaño del disco es de 2 tebibytes (TiB) u otro mayor, deberá usar la creación de particiones de GPT. Si es menor, podrá usar la de MBR o la de GPT. 

> [!NOTE]
> Se recomienda usar la versión más reciente de `parted` que esté disponible para su distribución.
> Si el tamaño del disco es de 2 tebibytes (TiB) o superior, debe usar la creación de particiones de GPT. Si el tamaño del disco es de menos de 2 TiB, puede usar la creación de particiones de MBR o GPT.  


En el ejemplo siguiente se usa `parted` en `/dev/sdc`, que es la ubicación en la que normalmente estará el primer disco de datos de la mayoría de las máquinas virtuales. Reemplace `sdc` por la opción correcta para el disco. También se formateará con el sistema de archivos [XFS](https://xfs.wiki.kernel.org/).

```bash
sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo mkfs.xfs /dev/sdc1
sudo partprobe /dev/sdc1
```

Use la utilidad [`partprobe`](https://linux.die.net/man/8/partprobe) para garantizar que el kernel reconozca inmediatamente la nueva partición y el sistema de archivos. Si no se usa `partprobe`, los comandos blkid o lslbk no podrán devolver de forma inmediata el UUID del nuevo sistema de archivos.


### <a name="mount-the-disk"></a>Montaje del disco

Ahora, cree un directorio para montar el sistema de archivos mediante `mkdir`. El ejemplo siguiente permite crear un directorio en `/datadrive`:

```bash
sudo mkdir /datadrive
```

A continuación, use `mount` para montar el sistema de archivos. En el ejemplo siguiente, se monta la partición `/dev/sdc1` en el punto de montaje `/datadrive`:

```bash
sudo mount /dev/sdc1 /datadrive
```

### <a name="persist-the-mount"></a>Hacer persistente el montaje

Para asegurarse de que la unidad se vuelve a montar automáticamente después de reiniciar, debe agregarse al archivo */etc/fstab*. Además, se recomienda usar el UUID (identificador único global) en */etc/fstab* para hacer referencia a la unidad, en lugar de solo el nombre del dispositivo (por ejemplo, */dev/sdc1*). Si el SO detecta un error de disco durante el arranque, con el UUID se evita que se monte el disco incorrecto en una ubicación especificada. A los discos de datos restantes se les asignarán después esos mismos identificadores de dispositivo. Para buscar el UUID de la unidad nueva, use la utilidad `blkid`:

```bash
sudo blkid
```

El resultado es similar al ejemplo siguiente:

```bash
/dev/sda1: LABEL="cloudimg-rootfs" UUID="11111111-1b1b-1c1c-1d1d-1e1e1e1e1e1e" TYPE="ext4" PARTUUID="1a1b1c1d-11aa-1234-1a1a1a1a1a1a"
/dev/sda15: LABEL="UEFI" UUID="BCD7-96A6" TYPE="vfat" PARTUUID="1e1g1cg1h-11aa-1234-1u1u1a1a1u1u"
/dev/sdb1: UUID="22222222-2b2b-2c2c-2d2d-2e2e2e2e2e2e" TYPE="ext4" TYPE="ext4" PARTUUID="1a2b3c4d-01"
/dev/sda14: PARTUUID="2e2g2cg2h-11aa-1234-1u1u1a1a1u1u"
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="xfs" PARTLABEL="xfspart" PARTUUID="c1c2c3c4-1234-cdef-asdf3456ghjk"
```

> [!NOTE]
> La edición incorrecta del archivo **/etc/fstab** puede tener como resultado un sistema que no se pueda arrancar. Si no está seguro, consulte la documentación de distribución para obtener información sobre cómo editar correctamente ese archivo. También se recomienda realizar una copia de seguridad del archivo /etc/fstab antes de editarlo.

Después, abra el archivo */etc/fstab* en un editor de texto como se indica a continuación:

```bash
sudo nano /etc/fstab
```

En este ejemplo, use el valor de UUID para el dispositivo `/dev/sdc1` que se creó en los pasos anteriores y el punto de montaje `/datadrive`. Agregue la línea siguiente al final del archivo `/etc/fstab`:

```bash
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   xfs   defaults,nofail   1   2
```

En este ejemplo, usamos el editor nano, así que cuando haya terminado de editar el archivo, use `Ctrl+O` para escribir el archivo y `Ctrl+X` para salir del editor.

> [!NOTE]
> Después, la eliminación de un disco de datos sin editar fstab podría provocar un error en el inicio de la máquina virtual. La mayoría de las distribuciones proporcionan las opciones de fstab *nofail* y/o *nobootwait*. Estas opciones permiten que el sistema arranque incluso si no se monta el disco en el momento del arranque. Consulte la documentación de su distribución para obtener más información sobre estos parámetros.
>
> La opción *nofail* garantiza que la máquina virtual se inicia incluso si el sistema de archivos está dañado o el disco no existe en tiempo de arranque. Sin esta opción, puede encontrarse con el comportamiento que se describe en [Cannot SSH to Linux VM due to FSTAB errors](/archive/blogs/linuxonazure/cannot-ssh-to-linux-vm-after-adding-data-disk-to-etcfstab-and-rebooting) (No se puede conectar mediante SSH a una máquina virtual Linux debido a errores de FSTAB).
>
> La consola serie de máquina virtual de Azure puede usarse para el acceso de consola a la máquina virtual si al modificar fstab se ha producido un error de arranque. Puede encontrar más información en la [documentación de la consola serie](../troubleshooting/serial-console-linux.md).

### <a name="trimunmap-support-for-linux-in-azure"></a>Compatibilidad de TRIM/UNMAP con Linux en Azure
Algunos kernels de Linux admiten operaciones TRIM/UNMAP para descartar bloques no usados del disco. Esta característica es especialmente útil en el almacenamiento estándar para informar a Azure de que las páginas eliminadas ya no son válidas y se pueden descartar. Además, le permitirá ahorrar dinero si crea archivos de gran tamaño y luego los elimina.

Hay dos maneras de habilitar la compatibilidad con TRIM en su máquina virtual Linux. Como es habitual, consulte la documentación de distribución para ver el enfoque recomendado:

* Use la opción de montaje `discard` en */etc/fstab*, por ejemplo:

    ```bash
    UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   xfs   defaults,discard   1   2
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

## <a name="troubleshooting"></a>Solución de problemas

[!INCLUDE [virtual-machines-linux-lunzero](../../../includes/virtual-machines-linux-lunzero.md)]

## <a name="next-steps"></a>Pasos siguientes

* Revise las recomendaciones para [optimizar el rendimiento de la máquina Linux](optimization.md) para asegurarse de que la máquina virtual Linux está configurada correctamente.
* Amplíe la capacidad de almacenamiento mediante la adición de discos adicionales y [configure RAID](configure-raid.md) para obtener un mayor rendimiento.
