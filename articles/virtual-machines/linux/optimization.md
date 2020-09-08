---
title: Optimización de la máquina virtual Linux en Azure
description: Aprenda algunas sugerencias de optimización para asegurarse de que ha configurado la máquina virtual Linux para obtener un rendimiento óptimo en Azure.
author: rickstercdn
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 09/06/2016
ms.author: rclaus
ms.subservice: disks
ms.openlocfilehash: eff512c9d050eb293391233848fcece83e845680
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654198"
---
# <a name="optimize-your-linux-vm-on-azure"></a>Optimización de la máquina virtual Linux en Azure
Crear una máquina virtual con Linux es muy sencillo desde la línea de comandos o desde el Portal. Este tutorial muestra cómo asegurarse de que está configurada para optimizar su rendimiento en la Plataforma Microsoft Azure. Este tema usa una VM de servidor Ubuntu, pero también puede crear máquinas virtuales Linux mediante [sus propias imágenes como plantillas](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).  

## <a name="prerequisites"></a>Prerrequisitos
En este tema se da por supuesto que ya tiene una suscripción de Azure activa ([suscripción de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/)) y ya ha aprovisionado una VM en su suscripción de Azure. Asegúrese de que dispone de la [CLI de Azure](/cli/azure/install-az-cli2) más reciente instalada y de que ha iniciado sesión en su suscripción de Azure con [az login](/cli/azure/reference-index) antes de [crear una máquina virtual](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="azure-os-disk"></a>Disco de sistema operativo de Azure
Al crear la VM Linux en Azure, tiene dos discos asociados a ella. **/dev/sda** es el disco del SO y **/dev/sdb** es el disco temporal.  No utilice el disco del SO principal ( **/dev/sda**) salvo para el mismo sistema operativo, ya que se ha optimizado para un tiempo de arranque rápido de la VM y no proporcionará un buen rendimiento para sus cargas de trabajo. Querrá conectar uno o más discos a la VM con el fin de obtener un almacenamiento optimizado y persistente para sus datos. 

## <a name="adding-disks-for-size-and-performance-targets"></a>Adición de discos para objetivos de rendimiento y tamaño
En función del tamaño de la VM, puede conectar hasta 16 discos adicionales en una máquina de serie A, 32 discos en una de serie D y 64 discos en una máquina de serie G, cada una de ellas con hasta 32 TB de tamaño. Agregue discos adicionales según sea necesario en función del espacio y los requisitos de IOPS. Cada disco tiene un objetivo de rendimiento de 500 IOPS para Standard Storage y de hasta 20 000 IOPS por disco para Premium Storage.

Para alcanzar el máximo valor de IOPS en los discos de Premium Storage con la configuración de caché como **ReadOnly** o **None**, debe deshabilitar las **barreras** al montar el sistema de archivos en Linux. No necesita las barreras porque las escrituras en los discos de Premium Storage de copia de seguridad son duraderas para esta configuración de caché.

* Si utiliza **reiserFS**, deshabilite las barreras mediante la opción de montaje `barrier=none` (para habilitar las barreras, use `barrier=flush`)
* Si utiliza **ext3/ext4**, deshabilite las barreras mediante la opción de montaje `barrier=0` (para habilitar las barreras, use `barrier=1`)
* Si utiliza **XFS**, deshabilite las barreras mediante la opción de montaje `nobarrier` (para habilitar las barreras, use la opción `barrier`)

## <a name="unmanaged-storage-account-considerations"></a>Consideraciones de la cuenta de almacenamiento no administrada
La acción predeterminada al crear una máquina virtual con la CLI de Azure consiste en usar Azure Managed Disks.  Estos discos se controlan mediante la plataforma de Azure y no requieren preparativos ni ubicación para el almacenamiento.  Los discos no administrados requieren una cuenta de almacenamiento y tienen algunas consideraciones de rendimiento adicionales.  Para más información acerca de los discos administrados, consulte [Azure Managed Disks overview](../managed-disks-overview.md) (Introducción a los discos administrados de Azure).  En la siguiente sección se describen consideraciones de rendimiento solo cuando se usan discos no administrados.  Una vez más, la solución de almacenamiento predeterminada y recomendada consiste en utilizar discos administrados.

Si crea una VM con discos no administrados, asegúrese de que conecta los discos desde las cuentas de almacenamiento que residen en la misma región que la VM para garantizar la proximidad y minimizar la latencia de red.  Cada cuenta de almacenamiento estándar tiene un máximo de IOPS de 20 k y una capacidad de tamaño de 500 TB.  Este límite nos da aproximadamente 40 discos muy utilizados, incluido el disco del SO y cualquier disco de datos que haya creado. Para las cuentas de Premium Storage, no hay ningún límite máximo de IOPS, pero hay un límite de tamaño de 32 TB. 

Al tratar con cargas de trabajo de IOPS elevadas y de haber elegido Standard Storage para los discos, debe dividir los discos entre varias cuentas de almacenamiento para asegurarse de no alcanzar el límite de 20 000 IOPS para las cuentas de Standard Storage. La máquina virtual puede contener una combinación de discos de las diferentes cuentas de almacenamiento y de los tipos de cuentas de almacenamiento para alcanzar la configuración óptima.
 

## <a name="your-vm-temporary-drive"></a>Su unidad temporal de máquina virtual
De forma predeterminada, cuando se crea una VM, Azure proporciona un disco de SO ( **/dev/sda**) y un disco temporal ( **/dev/sdb**).  Todos los discos adicionales que se agreguen se mostrarán como **/dev/sdc**, **/dev/sdd**, **/dev/sde**, etc. Todos los datos del disco temporal ( **/dev/sdb**) no son duraderos y se pueden perder si eventos específicos, como el cambio de tamaño de la VM, la reimplementación o el mantenimiento, fuerzan un reinicio de la VM.  El tamaño y el tipo de disco temporal está relacionado con el tamaño de memoria virtual que seleccionó en el momento de la implementación. En todas las VM de tamaño premium (series DS, G y DS_V2) se realizará una copia de seguridad de la unidad temporal en una unidad SSD local para obtener un rendimiento adicional de hasta 48 000 IOPS. 

## <a name="linux-swap-partition"></a>Partición de intercambio de Linux
Si la VM de Azure procede de una imagen de Ubuntu o CoreOS, puede usar CustomData para enviar una cloud-config a cloud-init. Si [cargó una imagen personalizada de Linux](upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) que usa cloud-init, configure también particiones de intercambio mediante cloud-init.

En las imágenes de nube de Ubuntu, debe usar cloud-init para configurar la partición de intercambio. Para más información, consulte [AzureSwapPartitions](https://wiki.ubuntu.com/AzureSwapPartitions).

En el caso de las imágenes sin compatibilidad con cloud-init, las imágenes de máquina virtual implementadas desde Azure Marketplace tienen un agente Linux de máquina virtual integrado con el sistema operativo. Este agente permite que la máquina virtual interactúe con diversos servicios de Azure. Suponiendo que ha implementado una imagen estándar desde Azure Marketplace, deberá hacer lo siguiente para configurar correctamente los valores del archivo de intercambio de Linux:

Busque y modifique dos entradas en el archivo **/etc/waagent.conf** . Estas entradas controlan la existencia de un archivo de intercambio dedicado y el tamaño del archivo de intercambio. Los parámetros que debe comprobar son `ResourceDisk.EnableSwap` y `ResourceDisk.SwapSizeMB` 

Para habilitar un disco habilitado correctamente y un archivo de intercambio montado, asegúrese de que los parámetros tengan la siguiente configuración:

* ResourceDisk.EnableSwap=Y
* ResourceDisk.SwapSizeMB={tamaño en MB que satisfaga sus requisitos} 

Una vez haya realizado el cambio, debe reiniciar waagent o la VM Linux con el fin de reflejar dichos cambios.  Sabrá que se han implementado los cambios y que se ha creado un archivo de intercambio cuando use el comando `free` para ver el espacio libre. En el ejemplo siguiente tiene un archivo de intercambio de 512 MB creado como resultado de modificar el archivo **waagent.conf**:

```bash
azuseruser@myVM:~$ free
            total       used       free     shared    buffers     cached
Mem:       3525156     804168    2720988        408       8428     633192
-/+ buffers/cache:     162548    3362608
Swap:       524284          0     524284
```

## <a name="io-scheduling-algorithm-for-premium-storage"></a>Algoritmo de programación de E/S para Premium Storage
Con el kernel de Linux 2.6.18, el algoritmo de programación de E/S predeterminado se cambió de Deadline (Fecha límite) a CFQ (Algoritmo de cola completamente razonable). Para patrones de E/S de acceso aleatorio, hay una diferencia insignificante en diferencias de rendimiento entre CFQ y Deadline.  En el caso de los discos SSD donde el patrón de E/S del disco es principalmente secuencial, el cambio al algoritmo NOOP o al de Deadline puede lograr un mejor rendimiento de E/S.

### <a name="view-the-current-io-scheduler"></a>Visualización del programador de E/S actual
Use el comando siguiente:  

```bash
cat /sys/block/sda/queue/scheduler
```

Se mostrará la salida siguiente, que indica cuál es el programador actual.  

```bash
noop [deadline] cfq
```

### <a name="change-the-current-device-devsda-of-io-scheduling-algorithm"></a>Cambiar el dispositivo actual (/dev/sda) del algoritmo de programación de E/S
Use los comandos siguientes:  

```bash
azureuser@myVM:~$ sudo su -
root@myVM:~# echo "noop" >/sys/block/sda/queue/scheduler
root@myVM:~# sed -i 's/GRUB_CMDLINE_LINUX=""/GRUB_CMDLINE_LINUX_DEFAULT="quiet splash elevator=noop"/g' /etc/default/grub
root@myVM:~# update-grub
```

> [!NOTE]
> Aplicar este valor solo para **/dev/sda** no es útil. Establézcalo en todos los discos de datos donde la E/S secuencial domina el patrón de E/S.  

Debería mostrarse el resultado siguiente, que indica que **grub.cfg** se ha vuelto a compilar correctamente y que el programador predeterminado se ha actualizado a NOOP.  

```bash
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.13.0-34-generic
Found initrd image: /boot/initrd.img-3.13.0-34-generic
Found linux image: /boot/vmlinuz-3.13.0-32-generic
Found initrd image: /boot/initrd.img-3.13.0-32-generic
Found memtest86+ image: /memtest86+.elf
Found memtest86+ image: /memtest86+.bin
done
```

Para la familia de distribución Red Hat, solo necesita el siguiente comando:   

```bash
echo 'echo noop >/sys/block/sda/queue/scheduler' >> /etc/rc.local
```

Ubuntu 18.04 con el kernel optimizado para Azure usa programadores de E/S de varias colas. En ese escenario, `none` es la selección adecuada, en lugar de `noop`. Para obtener más información, consulte los [programadores de E/S de Ubuntu](https://wiki.ubuntu.com/Kernel/Reference/IOSchedulers).

## <a name="using-software-raid-to-achieve-higher-iops"></a>Uso del software RAID para alcanzar mayores IOPS
Si las cargas de trabajo necesitan más IOPS de lo que puede proporcionar un único disco, debe utilizar una configuración de software RAID de varios discos. Como Azure ya realiza la resistencia de disco en el nivel de tejido local, obtendrá el máximo nivel de rendimiento en una configuración de creación de bandas RAID-0.  Aprovisione y cree discos en el entorno de Azure y conéctelos a la VM Linux antes de la creación de particiones, del formato y del montaje de las unidades.  Para más información sobre cómo configurar el software RAID en la máquina virtual Linux en Azure, consulte el documento **[Configuración del software RAID en Linux](configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)** .

Como alternativa a una configuración de RAID tradicional, también puede optar por instalar el administrador de volúmenes lógicos (LVM) para configurar una serie de discos físicos en un único volumen de almacenamiento lógico seccionado. Con esta configuración, las lecturas y escrituras se distribuyan en varios discos del grupo de volúmenes (similar a RAID0). Por motivos de rendimiento, es probable que deba seccionar los volúmenes lógicos para que las lecturas y escrituras utilicen todos los discos de datos conectados.  Para obtener más información sobre cómo configurar un volumen lógico seccionado en la máquina virtual Linux en Azure, vea el documento **[Configuración del LVM en una máquina virtual Linux en Azure](configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)** .

## <a name="next-steps"></a>Pasos siguientes
Recuerde que, como en todos los debates sobre optimización, debe realizar pruebas antes y después de cada cambio para cuantificar el impacto que tienen dichos cambios.  La optimización es un proceso paso a paso que tiene resultados diferentes en las distintas máquinas en su entorno.  Lo que funciona para una configuración puede no funcionar para otras.

Algunos vínculos útiles a recursos adicionales:

* [Guía de usuario del Agente de Linux de Azure](../extensions/agent-linux.md)
* [Configuración del software RAID en Linux](configure-raid.md)
