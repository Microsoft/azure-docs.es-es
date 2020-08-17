---
title: 'Desconexión de un disco de datos de una máquina virtual Linux: Azure'
description: Información sobre cómo desconectar un disco de datos de una máquina virtual de Azure mediante la CLI de Azure o Azure Portal.
author: roygara
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 07/18/2018
ms.author: rogarana
ms.subservice: disks
ms.custom: devx-track-azurecli
ms.openlocfilehash: 41f70a476e5cc562207614c4e04658d5f0fc6d09
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499524"
---
# <a name="how-to-detach-a-data-disk-from-a-linux-virtual-machine"></a>Desconexión de un disco de datos de una máquina virtual Linux

Cuando ya no necesite un disco de datos que se encuentra conectado a una máquina virtual, puede desconectarlo fácilmente. Esto quita el disco de la máquina virtual, pero no lo quita del almacenamiento. En este artículo se trabaja con una distribución de Ubuntu LTS 16.04. Si usa una distribución diferente, las instrucciones para desmontar el disco pueden variar.

> [!WARNING]
> Si se desconecta un disco, no se elimina automáticamente. Si se ha suscrito a Almacenamiento premium, seguirá acumulando cargos de almacenamiento por el disco. Para más información, vea [Precios y facturación al utilizar Premium Storage](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

Si desea volver a usar los datos existentes en el disco, puede acoplarlo de nuevo a la misma máquina virtual (o a otra).  


## <a name="connect-to-the-vm-to-unmount-the-disk"></a>Conexión a la máquina virtual para desmontar el disco

Para poder desconectar el disco mediante la CLI o el portal, debe desmontar el disco y quitar las referencias a él en el archivo fstab.

Conéctese a la máquina virtual. En este ejemplo, la dirección IP pública de la máquina virtual es *10.0.1.4* con el nombre de usuario *azureuser*: 

```bash
ssh azureuser@10.0.1.4
```

En primer lugar, busque el disco de datos que quiere desconectar. En el ejemplo siguiente se usa dmesg para filtrar por los discos SCSI:

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

En este caso, *sdc* es el disco que se quiere desconectar. También debe obtener el UUID del disco.

```bash
sudo -i blkid
```

La salida es similar a la siguiente:

```bash
/dev/sda1: UUID="11111111-1b1b-1c1c-1d1d-1e1e1e1e1e1e" TYPE="ext4"
/dev/sdb1: UUID="22222222-2b2b-2c2c-2d2d-2e2e2e2e2e2e" TYPE="ext4"
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="ext4"
```


Edite el archivo */etc/fstab* para quitar las referencias al disco. 

> [!NOTE]
> La edición incorrecta del archivo **/etc/fstab** puede tener como resultado un sistema que no se pueda arrancar. Si no está seguro, consulte la documentación de distribución para obtener información sobre cómo editar correctamente ese archivo. También se recomienda realizar una copia de seguridad del archivo /etc/fstab antes de editarlo.

Abra el archivo */etc/fstab* en un editor de texto tal y como se indica a continuación:

```bash
sudo vi /etc/fstab
```

En este ejemplo, debe eliminarse la siguiente línea del archivo */etc/fstab*:

```bash
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext4   defaults,nofail   1   2
```

Utilice `umount` para desmontar el disco. En el ejemplo siguiente se desmonta la partición */dev/sdc1* del punto de montaje */datadrive*:

```bash
sudo umount /dev/sdc1 /datadrive
```


## <a name="detach-a-data-disk-using-azure-cli"></a>Desconexión de un disco de datos con la CLI de Azure 

En este ejemplo se desconecta el disco *myDataDisk* de la máquina virtual denominada *myVM* en *myResourceGroup*.

```azurecli
az vm disk detach \
    -g myResourceGroup \
    --vm-name myVm \
    -n myDataDisk
```

El disco permanece en el almacenamiento pero ya no está acoplado a una máquina virtual.


## <a name="detach-a-data-disk-using-the-portal"></a>Desconexión de un disco de datos mediante el portal

1. En el menú de la izquierda, seleccione **Máquinas virtuales**.
1. En la hoja de la máquina virtual, seleccione **Discos**.
1. En la parte superior de la hoja **Discos**, seleccione **Editar**.
1. En la hoja **Discos**, en el extremo derecho del disco de datos que desea desconectar, haga clic en el botón de desconexión ![imagen del botón de desconexión](./media/detach-disk/detach.png).
1. Después de quitar el disco, haga clic en **Guardar** en la parte superior de la hoja.

El disco permanece en el almacenamiento pero ya no está acoplado a una máquina virtual.



## <a name="next-steps"></a>Pasos siguientes
Si desea reutilizar el disco de datos, basta con que lo [conecte a otra máquina virtual](add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

