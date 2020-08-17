---
title: Recuperación de VM de Linux con chroot mediante LVM (Administrador de volúmenes lógicos) -VM de Azure
description: Recuperación de VM de Linux con LVM.
services: virtual-machines-linux
documentationcenter: ''
author: vilibert
manager: spogge
editor: ''
tags: Linux chroot LVM
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 11/24/2019
ms.author: vilibert
ms.openlocfilehash: 03e6f51d2ab7138675f7d79c04faa2e4dffec60c
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87825691"
---
# <a name="troubleshooting-a-linux-vm-when-there-is-no-access-to-the-azure-serial-console-and-the-disk-layout-is-using-lvm-logical-volume-manager"></a>Solución de problemas de una VM de Linux cuando no se tiene acceso a la consola serie de Azure y el diseño de disco usa LVM (Administrador de volúmenes lógicos)

Esta guía de solución de problemas es útil en escenarios en los que una VM de Linux no se inicia, si no es posible usar el protocolo SSH y si el diseño del sistema de archivos subyacente se configura con LVM (Administrador de volúmenes lógicos).

## <a name="take-snapshot-of-the-failing-vm"></a>Realizar una instantánea de la VM con errores

Haga una instantánea de la VM con afectada. 

La instantánea se adjuntará a una VM de **rescate**. Siga las instrucciones [aquí](../linux/snapshot-copy-managed-disk.md#use-azure-portal) sobre cómo hacer una **instantánea**.

## <a name="create-a-rescue-vm"></a>Cree una VM de rescate
Normalmente, se recomienda usar una VM de rescate de la misma versión o de un sistema operativo similar. Use la misma **región** y **grupo de recursos** de la VM afectada.

## <a name="connect-to-the-rescue-vm"></a>Conexión a la VM de rescate
Conéctese mediante SSH a la VM de **rescate**. Eleve los privilegios y conviértase en un super usuario usando lo siguiente:

`sudo su -`

## <a name="attach-the-disk"></a>Adjuntar el disco
Conecte un disco a la VM de **rescate** creada a partir de la instantánea tomada previamente.

Azure Portal > seleccione la VM de **rescate**-> **Discos** 

![Creación de disco](./media/chroot-logical-volume-manager/create-disk-from-snap.png)

Rellene los campos. Asigne un nombre al nuevo disco, seleccione el mismo grupo de recursos que la instantánea, la VM afectada y la VM de rescate.

El **tipo de origen** es **Instantánea**.
La **instantánea de origen** es el nombre de la **instantánea** que se creó anteriormente.

![crear disco 2](./media/chroot-logical-volume-manager/create-disk-from-snap-2.png)

Cree un punto de montaje para el disco adjunto.

`mkdir /rescue`

Ejecute el comando **fdisk-l** para comprobar que se ha adjuntado el disco de instantáneas y mostrar todos los dispositivos y las particiones disponibles.

`fdisk -l`

En la mayoría de los escenarios, el disco de instantáneas adjunto se verá como **/dev/sdc** y mostrará dos particiones **/dev/sdc1** y **/dev/sdc2**.

![Fdisk](./media/chroot-logical-volume-manager/fdisk-output-sdc.png)

El **\*** indica una partición de inicio, ya que ambas particiones se van a montar.

Ejecute el comando **lsblk** para ver los LVM de la VM afectada.

`lsblk`

![Ejecute lsblk](./media/chroot-logical-volume-manager/lsblk-output-mounted.png)


Compruebe si se muestran LVM de la VM afectada.
Si no es así, use los comandos siguientes para habilitarlos y volver a ejecutar **lsblk**.
Asegúrese de que los LVM del disco adjunto sean visibles antes de continuar.

```
vgscan --mknodes
vgchange -ay
lvscan
mount –a
lsblk
```

Busque la ruta de acceso para montar el volumen lógico que contiene la partición / (raíz). Recuerde que este tiene archivos de configuración como /etc/default/grub.

En este ejemplo, la salida del comando **lsblk** anterior **rootvg-rootlv** es la **raíz** correcta de LV que se va a montar y que se puede usar en el siguiente comando.

La salida del siguiente comando mostrará la ruta de acceso que se va a montar en la **raíz** de LV.

`pvdisplay -m | grep -i rootlv`

![Rootlv](./media/chroot-logical-volume-manager/locate-rootlv.png)

Continúe para montar este dispositivo en el directorio /rescue.

`mount /dev/rootvg/rootlv /rescue`

Monte la partición que tiene la **marca de inicio** establecida en /rescue/boot.

`
mount /dev/sdc1 /rescue/boot
`

Compruebe que los sistemas de archivos del disco adjunto se montan correctamente mediante el comando **lsblk**.

![Ejecute lsblk](./media/chroot-logical-volume-manager/lsblk-output-1.png)

o el comando **df -Th**

![Df](./media/chroot-logical-volume-manager/df-output.png)

## <a name="gaining-chroot-access"></a>Obtener acceso a chroot

Obtenga acceso a **chroot** y podrá realizar varias correcciones; tenga en cuenta que existen pequeñas variaciones en cada distribución de Linux.

```
 cd /rescue
 mount -t proc proc proc
 mount -t sysfs sys sys/
 mount -o bind /dev dev/
 mount -o bind /dev/pts dev/pts/
 chroot /rescue
```

Si se produce un error como el siguiente:

**chroot: no se pudo ejecutar el comando "/bin/bash": no se encontró el archivo o directorio**,

intente montar el volumen lógico **usr**.

`
mount  /dev/mapper/rootvg-usrlv /rescue/usr
`

> [!TIP]
> Al ejecutar comandos en un entorno de **chroot**, tenga en cuenta que se ejecutan en el disco del sistema operativo adjunto y no en el de la VM de **rescate** local. 

Los comandos se pueden usar para instalar, quitar y actualizar software. Solucionar problemas de VM para corregir errores.


Ejecute el comando lsblk; /rescue es ahora / y /rescue/boot es /boot ![Chrooted](./media/chroot-logical-volume-manager/chrooted.png).

## <a name="perform-fixes"></a>Realización de correcciones

### <a name="example-1---configure-the-vm-to-boot-from-a-different-kernel"></a>Ejemplo 1: configurar la VM para que se inicie desde un kernel diferente.

Un escenario común es forzar una VM para que se inicie desde un kernel anterior, ya que es posible que el kernel que tenga instalado se haya dañado o que no se haya completado correctamente una actualización.


```
cd /boot/grub2

grep -i linux grub.cfg

grub2-editenv list

grub2-set-default "CentOS Linux (3.10.0-1062.1.1.el7.x86_64) 7 (Core)"

grub2-editenv list

grub2-mkconfig -o /boot/grub2/grub.cfg
```

*walkthrough*

El comando **grep** muestra los kernels que **grub.cfg** conoce.
![Kernels](./media/chroot-logical-volume-manager/kernels.png)

**grub2-editenv list** muestra el kernel que se cargará en el siguiente inicio. ![Valor predeterminado del kernel](./media/chroot-logical-volume-manager/kernel-default.png).

**grub2-set-default** se usa para cambiar a otro kernel. ![Conjunto de grub2](./media/chroot-logical-volume-manager/grub2-set-default.png).

La lista **grub2-editenv** muestra qué kernel se cargará en el siguiente inicio. ![Nuevo kernel](./media/chroot-logical-volume-manager/kernel-new.png)

**grub2-mkconfig** vuelve a generar grub.cfg mediante las versiones necesarias. ![grub2 mkconfig](./media/chroot-logical-volume-manager/grub2-mkconfig.png)



### <a name="example-2---upgrade-packages"></a>Ejemplo 2: actualizar paquetes

Una actualización de kernel con error puede hacer que la VM no se pueda iniciar.
Monte todos los volúmenes lógicos para permitir la eliminación o reinstalación de los paquetes.

Ejecute el comando **lvs** para comprobar qué **LV** están disponibles para el montaje; cada VM que se migró o que proviene de otro proveedor de nube, variará en la configuración.

Salga del entorno **chroot** y monte el **LV** necesario.

![Avanzado](./media/chroot-logical-volume-manager/advanced.png)

A continuación, acceda nuevamente al entorno **chroot** ejecutando:

`chroot /rescue`

Todos los LV deben ser visibles como particiones montadas.

![Avanzado](./media/chroot-logical-volume-manager/chroot-all-mounts.png)

Consulte el **kernel** instalado.

![Avanzado](./media/chroot-logical-volume-manager/rpm-kernel.png)

Si es necesario, elimine o actualice el **kernel**
![Avanzado](./media/chroot-logical-volume-manager/rpm-remove-kernel.png)


### <a name="example-3---enable-serial-console"></a>Ejemplo 3: habilitar la consola serie
Si no se ha podido obtener acceso a la consola serie de Azure, compruebe los parámetros de configuración de GRUB para la VM de Linux y corríjalos. Puede encontrar información detallada [en este documento](./serial-console-grub-proactive-configuration.md).

### <a name="example-4---kernel-loading-with-problematic-lvm-swap-volume"></a>Ejemplo 4: carga del kernel con un volumen de intercambio LVM problemático

Es posible que una máquina virtual no arranque completamente y caiga en el símbolo del sistema de **Dracut**.
Puede encontrar más detalles sobre el error en la consola serie de Azure o ir a Azure Portal -> Diagnósticos de arranque -> Registro serie


Puede aparecer un error similar al siguiente:

```
[  188.000765] dracut-initqueue[324]: Warning: /dev/VG/SwapVol does not exist
         Starting Dracut Emergency Shell...
Warning: /dev/VG/SwapVol does not exist
```

El archivo grub.cfg se ha configurado en este ejemplo para cargar una LV con el nombre de **rd.lvm.lv=VG/SwapVol** y la máquina virtual no puede encontrarla. Esta línea muestra cómo se carga el kernel haciendo referencia a LV SwapVol

```
[    0.000000] Command line: BOOT_IMAGE=/vmlinuz-3.10.0-1062.4.1.el7.x86_64 root=/dev/mapper/VG-OSVol ro console=tty0 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0 biosdevname=0 crashkernel=256M rd.lvm.lv=VG/OSVol rd.lvm.lv=VG/SwapVol nodmraid rhgb quiet
[    0.000000] e820: BIOS-provided physical RAM map:
```

 Quite la línea LV infractora de la configuración de /etc/default/grub y recompile grub2.cfg


## <a name="exit-chroot-and-swap-the-os-disk"></a>Salga de chroot e intercambie el disco del sistema operativo.

Después de reparar el problema, desmonte y desasocie el disco de la VM de rescate; gracias a ello, podrá intercambiar el disco del sistema operativo de la VM afectada.

```
exit
cd /
umount /rescue/proc/
umount /rescue/sys/
umount /rescue/dev/pts
umount /rescue/dev/
umount /rescue/boot
umount /rescue
```

Desasocie el disco de la VM de rescate y realice un intercambio de disco.

Seleccione la VM del portal **Discos** y seleccione **Desasociar**
![Desasociar el disco](./media/chroot-logical-volume-manager/detach-disk.png) 

Guarde los cambios en ![Guardar elemento desasociado](./media/chroot-logical-volume-manager/save-detach.png). 

El disco estará ahora disponible, gracias a lo cual podrá intercambiarlo con el disco del sistema operativo original de la VM afectada.

Vaya a Azure Portal y busque la VM con errores; a continuación, seleccione **Discos** -> **Intercambiar disco del sistema operativo**
![Intercambiar disco](./media/chroot-logical-volume-manager/swap-disk.png). 

Complete los campos para **elegir disco**, que será wl disco de instantánea que se desasoció en el paso anterior. También es necesario escribir el nombre de la VM afectada; a continuación, seleccione **Aceptar**.

![Nuevo disco del sistema operativo](./media/chroot-logical-volume-manager/new-osdisk.png) 

Si la VM se está ejecutando, el intercambio de disco se desactivará y se reiniciará la VM una vez completada la operación de intercambio de disco.


## <a name="next-steps"></a>Pasos siguientes
Más información acerca de

 [Azure Serial Console]( ./serial-console-linux.md)

[Modo de usuario único](./serial-console-grub-single-user-mode.md)