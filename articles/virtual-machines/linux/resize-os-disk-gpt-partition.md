---
title: Cambio del tamaño de un disco de SO con una partición GPT | Microsoft Docs
description: En este artículo se proporcionan instrucciones sobre cómo cambiar el tamaño de un disco de SO con una partición GPT.
services: virtual-machines-linux
documentationcenter: ''
author: kailashmsft
manager: dcscontentpm
editor: ''
tags: ''
ms.service: security
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.devlang: azurecli
ms.date: 05/03/2020
ms.author: kaib
ms.custom: seodec18
ms.openlocfilehash: 30a960c3ed76788158b15022947fec49a95ae299
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89375217"
---
# <a name="resize-an-os-disk-that-has-a-gpt-partition"></a>Cambio del tamaño de un disco de SO con una partición GPT

> [!NOTE]
> Este escenario se aplica únicamente a los discos de SO que tienen una partición de tabla de particiones de GUID (GPT).

En este artículo se describe cómo aumentar el tamaño de un disco del SO que tiene una partición GPT en Linux. 

## <a name="identify-whether-the-os-disk-has-an-mbr-or-gpt-partition"></a>Identificar si el disco del SO tiene una partición MBR o GPT

Use el comando `parted` para identificar si la partición de disco se ha creado con una partición de registro de arranque maestro (MBR) o una partición GPT.

### <a name="mbr-partition"></a>Partición MBR

En la salida siguiente, la **tabla de particiones** muestra el valor **msdos**. Este valor identifica una partición MBR.

```
[user@myvm ~]# parted -l /dev/sda
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 107GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Number  Start   End     Size    Type     File system  Flags
1       1049kB  525MB   524MB   primary  ext4         boot
2       525MB   34.4GB  33.8GB  primary  ext4
[user@myvm ~]#
```

### <a name="gpt-partition"></a>Partición GPT

En la salida siguiente, la **tabla de particiones** muestra el valor **gpt**. Este valor identifica una partición GPT.

```
[user@myvm ~]# parted -l /dev/sda
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 68.7GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name                  Flags
1       1049kB  525MB   524MB   fat16        EFI System Partition  boot
2       525MB   1050MB  524MB   xfs
3       1050MB  1052MB  2097kB                                     bios_grub
4       1052MB  68.7GB  67.7GB                                     lvm
```

Si la máquina virtual (VM) tiene una partición GPT en el disco del SO, aumente el tamaño del disco de SO.

## <a name="increase-the-size-of-the-os-disk"></a>Aumento de tamaño del disco de SO

Las instrucciones siguientes se aplican a las distribuciones aprobadas por Linux.

> [!NOTE]
> Antes de continuar, haga una copia de seguridad de la VM o realice una instantánea del disco de SO.

### <a name="ubuntu"></a>Ubuntu

Para aumentar el tamaño del disco de SO en Ubuntu 16.x y 18.x:

1. Pare la VM.
1. Aumente el tamaño del disco de SO desde el portal.
1. Reinicie la VM y, a continuación, inicie sesión en la VM como usuario **raíz**.
1. Compruebe que el disco de SO ahora presenta un tamaño mayor del sistema de archivos.

Como se muestra en el ejemplo siguiente, se ha cambiado el tamaño del disco de SO del portal a 100 GB. El sistema de archivos **/dev/sda1** montado en **/** muestra ahora 97 GB.

```
user@myvm:~# df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  314M     0  314M   0% /dev
tmpfs          tmpfs      65M  2.3M   63M   4% /run
/dev/sda1      ext4       97G  1.8G   95G   2% /
tmpfs          tmpfs     324M     0  324M   0% /dev/shm
tmpfs          tmpfs     5.0M     0  5.0M   0% /run/lock
tmpfs          tmpfs     324M     0  324M   0% /sys/fs/cgroup
/dev/sda15     vfat      105M  3.6M  101M   4% /boot/efi
/dev/sdb1      ext4       20G   44M   19G   1% /mnt
tmpfs          tmpfs      65M     0   65M   0% /run/user/1000
user@myvm:~#
```

### <a name="suse"></a>SUSE

Para aumentar el tamaño del disco de SO en SUSE 12 SP4, SUSE SLES 12 para SAP, SUSE SLES 15 y SUSE SLES 15 para SAP:

1. Pare la VM.
1. Aumente el tamaño del disco de SO desde el portal.
1. Reinicie la VM.

Una vez reiniciada la VM, siga los pasos que se describen a continuación:

1. Acceda a la VM como usuario **raíz** con el comando siguiente:

   ```
   # sudo -i
   ```

1. Use el siguiente comando para instalar el paquete **growpart**, que se usará para cambiar el tamaño de la partición:

   ```
   # zypper install growpart
   ```

1. Use el comando `lsblk` para encontrar la partición montada en la raíz del sistema de archivos ("/"). En este caso, vemos que la partición 4 del SDA del dispositivo está montada en /:

   ```
   # lsblk
   NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda      8:0    0   48G  0 disk
   ├─sda1   8:1    0    2M  0 part
   ├─sda2   8:2    0  512M  0 part /boot/efi
   ├─sda3   8:3    0    1G  0 part /boot
   └─sda4   8:4    0 28.5G  0 part /
   sdb      8:16   0    4G  0 disk
   └─sdb1   8:17   0    4G  0 part /mnt/resource
   ```

1. Cambie el tamaño de la partición necesaria con el comando `growpart` mediante el número de partición que se encuentra en el paso anterior.

   ```
   # growpart /dev/sda 4
   CHANGED: partition=4 start=3151872 old: size=59762655 end=62914527 new: size=97511391 end=100663263
   ```

1. Vuelva a ejecutar el comando `lsblk` para comprobar si ha aumentado la partición.

   El resultado siguiente muestra que se ha cambiado el tamaño de la partición **/dev/sda4** a 46,5 GB.
   
   ```
   linux:~ # lsblk
   NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda      8:0    0   48G  0 disk
   ├─sda1   8:1    0    2M  0 part
   ├─sda2   8:2    0  512M  0 part /boot/efi
   ├─sda3   8:3    0    1G  0 part /boot
   └─sda4   8:4    0 46.5G  0 part /
   sdb      8:16   0    4G  0 disk
   └─sdb1   8:17   0    4G  0 part /mnt/resource
   ```

1. Identifique el tipo de sistema de archivos del disco del sistema operativo mediante el comando `lsblk` con la marca `-f`:

   ```
   linux:~ # lsblk -f
   NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
   sda
   ├─sda1
   ├─sda2 vfat   EFI   AC67-D22D                            /boot/efi
   ├─sda3 xfs    BOOT  5731a128-db36-4899-b3d2-eb5ae8126188 /boot
   └─sda4 xfs    ROOT  70f83359-c7f2-4409-bba5-37b07534af96 /
   sdb
   └─sdb1 ext4         8c4ca904-cd93-4939-b240-fb45401e2ec6 /mnt/resource
   ```

1. Según el tipo del sistema de archivos, use los comandos adecuados para cambiar el tamaño del sistema de archivos.
   
   Para **xfs**, use el comando siguiente:
   
   ```
   #xfs_growfs /
   ```
   
   Salida de ejemplo:
   
   ```
   linux:~ # xfs_growfs /
   meta-data=/dev/sda4              isize=512    agcount=4, agsize=1867583 blks
            =                       sectsz=512   attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0 rmapbt=0
            =                       reflink=0
   data     =                       bsize=4096   blocks=7470331, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal               bsize=4096   blocks=3647, version=2
            =                       sectsz=512   sunit=0 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 7470331 to 12188923
   ```
   
   Para **ext4**, use el comando siguiente:
   
   ```
   #resize2fs /dev/sda4
   ```
   
1. Compruebe el mayor tamaño del sistema de archivos para **df -Th** con el siguiente comando:
   
   ```
   #df -Thl
   ```
   
   Salida de ejemplo:
   
   ```
   linux:~ # df -Thl
   Filesystem     Type      Size  Used Avail Use% Mounted on
   devtmpfs       devtmpfs  445M  4.0K  445M   1% /dev
   tmpfs          tmpfs     458M     0  458M   0% /dev/shm
   tmpfs          tmpfs     458M   14M  445M   3% /run
   tmpfs          tmpfs     458M     0  458M   0% /sys/fs/cgroup
   /dev/sda4      xfs        47G  2.2G   45G   5% /
   /dev/sda3      xfs      1014M   86M  929M   9% /boot
   /dev/sda2      vfat      512M  1.1M  511M   1% /boot/efi
   /dev/sdb1      ext4      3.9G   16M  3.7G   1% /mnt/resource
   tmpfs          tmpfs      92M     0   92M   0% /run/user/1000
   tmpfs          tmpfs      92M     0   92M   0% /run/user/490
   ```
   
   En el ejemplo anterior, podemos ver que se ha aumentado el tamaño del sistema de archivos para el disco de SO.

### <a name="rhel"></a>RHEL

Para aumentar el tamaño del disco de SO en RHEL 7.x con LVM:

1. Pare la VM.
1. Aumente el tamaño del disco de SO desde el portal.
1. Inicie la máquina virtual.

Una vez reiniciada la VM, siga los pasos que se describen a continuación:

1. Acceda a la VM como usuario **raíz** con el comando siguiente:
 
   ```
   #sudo su
   ```

1. Instale el paquete **gptfdisk**, que es necesario para aumentar el tamaño del disco de SO.

   ```
   #yum install gdisk -y
   ```

1. Para ver el sector más grande disponible en el disco, ejecute el siguiente comando:

   ```
   #sgdisk -e /dev/sda
   ```

1. Cambie el tamaño de la partición sin eliminarla mediante el comando siguiente. El comando **parted** tiene una opción denominada **resizepart** para cambiar el tamaño de la partición sin eliminarla. El número 4 después de **resizepart** indica el cambio de tamaño de la cuarta partición.

   ```
   #parted -s /dev/sda "resizepart 4 -1" quit
   ```
    
1. Ejecute el siguiente comando para comprobar que la partición ha aumentado:

   ```
   #lsblk
   ```

   El resultado siguiente muestra que se ha cambiado el tamaño de la partición **/dev/sda4** a 99 GB.

   ```
   [user@myvm ~]# lsblk
   NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   fd0                 2:0    1    4K  0 disk
   sda                 8:0    0  100G  0 disk
   ├─sda1              8:1    0  500M  0 part /boot/efi
   ├─sda2              8:2    0  500M  0 part /boot
   ├─sda3              8:3    0    2M  0 part
   └─sda4              8:4    0   99G  0 part
   ├─rootvg-tmplv    253:0    0    2G  0 lvm  /tmp
   ├─rootvg-usrlv    253:1    0   10G  0 lvm  /usr
   ├─rootvg-optlv    253:2    0    2G  0 lvm  /opt
   ├─rootvg-homelv   253:3    0    1G  0 lvm  /home
   ├─rootvg-varlv    253:4    0    8G  0 lvm  /var
   └─rootvg-rootlv   253:5    0    2G  0 lvm  /
   sdb                 8:16   0   50G  0 disk
   └─sdb1              8:17   0   50G  0 part /mnt/resource
   ```

1. Use el siguiente comando para cambiar el tamaño del volumen físico (PV):

   ```
   #pvresize /dev/sda4
   ```

   La salida siguiente muestra que se ha cambiado el tamaño de PV a 99,02 GB.

   ```
   [user@myvm ~]# pvresize /dev/sda4
   Physical volume "/dev/sda4" changed
   1 physical volume(s) resized or updated / 0 physical volume(s) not resized

   [user@myvm ~]# pvs
   PV         VG     Fmt  Attr PSize   PFree
   /dev/sda4  rootvg lvm2 a--  <99.02g <74.02g
   ```

1. En el ejemplo siguiente, se cambia el tamaño de **/dev/mapper/rootvg-rootlv** de 2 GB a 12 GB (un aumento de 10 GB) mediante el siguiente comando. Este comando también cambiará el tamaño del sistema de archivos.

   ```
   #lvresize -r -L +10G /dev/mapper/rootvg-rootlv
   ```

   Salida de ejemplo:

   ```
   [user@myvm ~]# lvresize -r -L +10G /dev/mapper/rootvg-rootlv
   Size of logical volume rootvg/rootlv changed from 2.00 GiB (512 extents) to 12.00 GiB (3072 extents).
   Logical volume rootvg/rootlv successfully resized.
   meta-data=/dev/mapper/rootvg-rootlv isize=512    agcount=4, agsize=131072 blks
            =                       sectsz=4096  attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0
   data     =                       bsize=4096   blocks=524288, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal               bsize=4096   blocks=2560, version=2
            =                       sectsz=4096  sunit=1 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 524288 to 3145728
   ```
         
1. Compruebe si **/dev/mapper/rootvg-rootlv** tiene un mayor tamaño del sistema de archivos mediante el comando siguiente:

   ```
   #df -Th /
   ```

   Salida de ejemplo:

   ```
   [user@myvm ~]# df -Th /
   Filesystem                Type  Size  Used Avail Use% Mounted on
   /dev/mapper/rootvg-rootlv xfs    12G   71M   12G   1% /
   [user@myvm ~]#
   ```

> [!NOTE]
> Para usar el mismo procedimiento para cambiar el tamaño de cualquier otro volumen lógico, cambie el nombre de **lv** en el paso 7.

## <a name="next-steps"></a>Pasos siguientes

- [Cambiar el tamaño del disco](expand-disks.md)
