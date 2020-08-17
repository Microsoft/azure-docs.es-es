---
title: Creación y carga de un disco duro virtual de Linux
description: Aprenda a crear y cargar un disco duro virtual de Azure (VHD) que contiene un sistema operativo Linux.
author: gbowerman
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 10/08/2018
ms.author: guybo
ms.openlocfilehash: a80cc29f318cff8e5a4c665cd07ba1829d25d66d
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373392"
---
# <a name="information-for-non-endorsed-distributions"></a>Información para las distribuciones no aprobadas

El Acuerdo de Nivel de Servicio de la plataforma Azure se aplica a máquinas virtuales que ejecutan el SO Linux solo cuando cuentan con una de las [distribuciones aprobadas](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). En estas distribuciones aprobadas, las imágenes de Linux preconfiguradas se proporcionan en Azure Marketplace.

* [Linux en distribuciones aprobadas por Azure](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Compatibilidad con las imágenes de Linux en Microsoft Azure](https://support.microsoft.com/kb/2941892)

Todas las distribuciones que se ejecutan en Azure tienen varios requisitos previos. Este artículo no puede ser completo, dado que todas las distribuciones son diferentes. Incluso si cumple todos los criterios siguientes, puede que tenga que ajustar significativamente el sistema Linux para que se ejecute correctamente.

Se recomienda comenzar con una de las [distribuciones aprobadas de Linux en Azure](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Los artículos siguientes muestran cómo preparar las distintas distribuciones de Linux aprobadas que se admiten en Azure:

- [Distribuciones basadas en CentOS](create-upload-centos.md)
- [Debian Linux](debian-create-upload-vhd.md)
- [Flatcar Container Linux](flatcar-create-upload-vhd.md)
- [Oracle Linux](oracle-create-upload-vhd.md)
- [Red Hat Enterprise Linux](redhat-create-upload-vhd.md)
- [SLES y openSUSE](suse-create-upload-vhd.md)
- [Ubuntu](create-upload-ubuntu.md)

Este artículo se centra en ofrecer orientaciones generales para ejecutar su distribución de Linux en Azure.

## <a name="general-linux-installation-notes"></a>Notas generales sobre la instalación de Linux
* No se admite el formato de disco duro virtual de Hyper-V (VHDX) en Azure, solo *VHD fijo*.  Puede convertir el disco al formato VHD con el Administrador de Hyper-V o el cmdlet [Convert-VHD](/powershell/module/hyper-v/convert-vhd). Si usa VirtualBox, deberá seleccionar **Tamaño fijo** en lugar del tamaño predeterminado (asignado dinámicamente) al crear el disco.
* Azure admite máquinas virtuales de Gen1 (arranque del BIOS) y Gen2 (arranque UEFI).
* El tamaño máximo permitido para los discos duros virtuales es de 1023 GB.
* Al instalar el sistema Linux, se recomienda usar las particiones estándar en lugar del Administrador de volúmenes lógicos (LVM), que viene de forma predeterminada en muchas instalaciones. Usar particiones estándar impedirá que el nombre del LVM entre en conflicto con las VM clonadas, especialmente si en algún momento hace falta adjuntar un disco de SO a otra VM idéntica para solucionar problemas. [LVM](configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) o [RAID](configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) se pueden utilizar en discos de datos.
* Se requiere la compatibilidad de kernel para el montaje de sistemas de archivos UDF. Al arrancar Azure la primera vez, la configuración de aprovisionamiento se pasa a la VM Linux a través de medios con formato UDF conectados al invitado. El agente Linux de Azure debe montar el sistema de archivos UDF para leer su configuración y aprovisionar la VM.
* Las versiones de kernel de Linux inferiores a la versión 2.6.37 no admiten NUMA en Hyper-V con tamaños de VM más grandes. Este problema afecta principalmente a las distribuciones anteriores que usan el kernel Red Hat 2.6.32 de canal de subida, y se ha corregido en Red Hat Enterprise Linux (RHEL) 6.6 (kernel-2.6.32-504). Los sistemas que ejecutan kernels personalizados cuyas versiones son anteriores a la versión 2.6.37, o bien kernels basados en RHEL cuyas versiones son anteriores a la versión 2.6.32-504, deben establecer el parámetro de inicio `numa=off` en la línea de comandos de kernel en grub.conf. Para más información, consulte [Red Hat KB 436883](https://access.redhat.com/solutions/436883).
* No configure una partición de intercambio en el disco del SO. El agente de Linux se puede configurar para crear un archivo de intercambio en el disco de recursos temporal, como se describe en los pasos siguientes.
* En Azure, todos los discos duros virtuales deben tener un tamaño virtual alineado con 1 MB. Al convertir un disco sin formato en un disco duro virtual, tiene que asegurarse de que su tamaño sea un múltiplo de 1 MB antes de la conversión, como se describe en los pasos siguientes.

### <a name="installing-kernel-modules-without-hyper-v"></a>Instalación de los módulos de kernel sin Hyper-V
Azure se ejecuta en el hipervisor de Hyper-V, por lo que Linux requiere que ciertos módulos del kernel se ejecuten en Azure. Si tiene una VM que se ha creado fuera de Hyper-V, los instaladores de Linux pueden no incluir los controladores de Hyper-V en el disco RAM inicial (initrd o initramfs), a menos que la VM detecte que se está ejecutando en un entorno de Hyper-V. Cuando use otro sistema de virtualización (es decir, VirtualBox, KVM, etc.) para preparar la imagen de Linux, es posible que necesite recompilar el initrd para que al menos los módulos de kernel hv_vmbus y hv_storvsc estén disponibles en el disco RAM inicial.  Esto es un problema conocido en sistemas basados en el nivel superior de distribución de Red Hat y, posiblemente en otros.

El mecanismo para volver a generar la imagen initrd o initramfs puede variar dependiendo de la distribución. Consulte la documentación de distribución o el soporte para conocer el procedimiento adecuado.  Este es un ejemplo de cómo recompilar initrd mediante la utilidad `mkinitrd`:

1. Haga una copia de seguridad de la imagen initrd existente:

    ```
    cd /boot
    sudo cp initrd-`uname -r`.img  initrd-`uname -r`.img.bak
    ```

2. Recompile el initrd con los módulos kernel hv_vmbus y hv_storvsc:

    ```
    sudo mkinitrd --preload=hv_storvsc --preload=hv_vmbus -v -f initrd-`uname -r`.img `uname -r`
    ```

### <a name="resizing-vhds"></a>Cambio de tamaño de los discos duros virtuales
Las imágenes VHD en Azure deben tener un tamaño virtual alineado con 1 MB.  Normalmente, los discos duros virtuales creados con Hyper-V se alinean correctamente.  Si el disco duro virtual no está alineado correctamente, recibirá un mensaje de error similar al siguiente cuando intente crear una imagen desde el disco duro virtual.

* El disco duro virtual http:\//\<mystorageaccount>.blob.core.windows.net/vhds/MyLinuxVM.vhd tiene un tamaño virtual no admitido de 21475270656 bytes. El tamaño debe ser un número entero (en MB).

En este caso, puede cambiar el tamaño de la VM mediante la consola de administrador de Hyper-V o el del cmdlet de PowerShell [Resize-VHD](/powershell/module/hyper-v/resize-vhd?view=win10-ps).  Si no está trabajando en un entorno de Windows, se recomienda usar `qemu-img` para convertir (si es necesario) y cambiar el tamaño del disco duro virtual.

> [!NOTE]
> Hay un [problema conocido en versiones de qemu-img](https://bugs.launchpad.net/qemu/+bug/1490611) >= 2.2.1 que da como resultado un VHD con formato incorrecto. El problema se corrigió en QEMU 2.6. Se recomienda usar `qemu-img` 2.2.0 o inferior, o 2.6 o superior.
> 

1. Cambiar el tamaño del disco duro virtual directamente mediante herramientas como `qemu-img` o `vbox-manage` puede dar  como resultado un disco duro virtual que no puede arrancar.  Se recomienda convertir primero el VHD a una imagen de disco sin procesar.  Si la imagen de la VM se ha creado como imagen de disco sin procesar (el valor predeterminado para algunos hipervisores, como KVM), puede omitir este paso.
 
    ```
    qemu-img convert -f vpc -O raw MyLinuxVM.vhd MyLinuxVM.raw
    ```

1. Calcule el tamaño necesario de la imagen de disco para que el tamaño virtual esté alineado con 1 MB.  El siguiente script de shell de Bash usa `qemu-img info` para determinar el tamaño virtual de la imagen de disco y, después, calcula el tamaño con el bloque de 1 MB siguiente.

    ```bash
    rawdisk="MyLinuxVM.raw"
    vhddisk="MyLinuxVM.vhd"

    MB=$((1024*1024))
    size=$(qemu-img info -f raw --output json "$rawdisk" | \
    gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

    rounded_size=$(((($size+$MB-1)/$MB)*$MB))
    
    echo "Rounded Size = $rounded_size"
    ```

3. Cambie el tamaño del disco sin procesar con `$rounded_size`, como se estableció anteriormente.

    ```bash
    qemu-img resize MyLinuxVM.raw $rounded_size
    ```

4. Ahora, convierta el disco sin procesar a un VHD de tamaño fijo.

    ```bash
    qemu-img convert -f raw -o subformat=fixed -O vpc MyLinuxVM.raw MyLinuxVM.vhd
    ```

   O bien, con la versión de qemu 2.6 o posterior, incluya la opción `force_size`.

    ```bash
    qemu-img convert -f raw -o subformat=fixed,force_size -O vpc MyLinuxVM.raw MyLinuxVM.vhd
    ```

## <a name="linux-kernel-requirements"></a>Requisitos para el kernel de Linux

Los controladores de los Servicios de integración de Linux (LIS) para Hyper-V y Azure contribuyen directamente en el kernel de Linux del canal de subida. Muchas de las distribuciones que incluyen una versión reciente del kernel de Linux (como 3.x) ya tienen estos controladores disponibles, o de lo contrario ofrecerán versiones con modificaciones de versiones anteriores de estos controladores con sus kernels.  Estos controladores se actualizan constantemente en el kernel del canal de subida con nuevas correcciones y características; por ello, se recomienda cuando sea posible la ejecución de una [distribución aprobada](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) que incluya estas correcciones y actualizaciones.

Si ejecuta una variante de las versiones de 6.0 a 6.3 de Red Hat Enterprise Linux, tendrá que instalar los [controladores de LIS más recientes para Hyper-V](https://go.microsoft.com/fwlink/p/?LinkID=254263&clcid=0x409). A partir de RHEL 6.4+ (y derivados), los controladores de LIS ya se incluyen con el kernel; por tanto, no se necesitan paquetes de instalación adicionales.

Si se requiere un kernel personalizado, se recomienda usar una versión de kernel más reciente (por ejemplo, 3.8+). En el caso de las distribuciones o los proveedores que mantienen su propio kernel, tendrá que modificar los controladores de LIS con una versión más antigua del kernel del canal de subida al kernel personalizado.  Incluso si ya ejecuta una versión de kernel relativamente reciente, es altamente recomendable mantenerse al tanto de las correcciones ascendentes de los controladores de LIS y realizar modificaciones con versiones anteriores en estos cuando sea necesario. Las ubicaciones de los archivos de origen de controladores de LIS se especifican en el archivo [MAINTAINERS](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/MAINTAINERS) del árbol de origen del kernel de Linux:
```
    F:    arch/x86/include/asm/mshyperv.h
    F:    arch/x86/include/uapi/asm/hyperv.h
    F:    arch/x86/kernel/cpu/mshyperv.c
    F:    drivers/hid/hid-hyperv.c
    F:    drivers/hv/
    F:    drivers/input/serio/hyperv-keyboard.c
    F:    drivers/net/hyperv/
    F:    drivers/scsi/storvsc_drv.c
    F:    drivers/video/fbdev/hyperv_fb.c
    F:    include/linux/hyperv.h
    F:    tools/hv/
```
Las revisiones siguientes deben incluirse en el kernel. Esta lista no puede ser completa para todas las distribuciones.

* [ata_piix: Aplazar discos a los controladores de Hyper-V de manera predeterminada](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/drivers/ata/ata_piix.c?id=cd006086fa5d91414d8ff9ff2b78fbb593878e3c)
* [storvsc: cuenta para paquetes en tránsito en la ruta de RESET](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/drivers/scsi/storvsc_drv.c?id=5c1b10ab7f93d24f29b5630286e323d1c5802d5c)
* [storvsc: Evitar el uso de WRITE_SAME](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=3e8f4f4065901c8dfc51407e1984495e1748c090)
* [storvsc: deshabilita WRITE SAME para RAID y controladores del adaptador de host virtual](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=54b2b50c20a61b51199bedb6e5d2f8ec2568fb43)
* [storvsc: corrección de desreferenciación del puntero nulo](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=b12bb60d6c350b348a4e1460cd68f97ccae9822e)
* [storvsc: Los errores de búfer de anillo pueden dar lugar a una inmovilización de E/S](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=e86fb5e8ab95f10ec5f2e9430119d5d35020c951)
* [scsi_sysfs: Protección contra la ejecución doble de __scsi_remove_device](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/scsi_sysfs.c?id=be821fd8e62765de43cc4f0e2db363d0e30a7e9b)

## <a name="the-azure-linux-agent"></a>el Agente de Linux de Azure
El [agente de Linux de Azure](../extensions/agent-linux.md) `waagent` aprovisiona una máquina virtual Linux en Azure. Puede obtener la versión más reciente, dejar constancia de problemas o enviar solicitudes de extracción en el [repositorio GitHub del agente de Linux](https://github.com/Azure/WALinuxAgent).

* El agente de Linux se publica bajo licencia Apache 2.0. Muchas distribuciones ya proporcionan paquetes RPM o .deb para el agente, que se pueden instalar y actualizar con facilidad.
* El agente de Linux de Azure requiere Python v2.6+.
* El agente requiere también el módulo python-pyasn1. La mayoría de las distribuciones proporcionan este módulo como paquete independiente que se puede instalar.
* En algunos casos, el agente de Linux de Azure puede no ser compatible con NetworkManager. Muchos de los paquetes RPM o deb proporcionados por las distribuciones configuran NetworkManager como conflicto con el paquete waagent. En estos casos, se desinstalará NetworkManager al instalar el paquete del agente de Linux.
* El agente de Linux de Azure debe ser igual o mayor que la [versión mínima admitida](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

## <a name="general-linux-system-requirements"></a>Requisitos generales del sistema Linux

1. Modifique la línea de arranque de kernel en GRUB o GRUB2 para incluir los siguientes parámetros, de modo que todos los mensajes de la consola se envíen al primer puerto serie. Estos mensajes pueden ayudar al soporte técnico de Azure con la depuración de problemas.
    ```  
    console=ttyS0,115200n8 earlyprintk=ttyS0,115200 rootdelay=300
    ```
    También se recomienda *quitar* los parámetros siguientes, si existen.
    ```  
    rhgb quiet crashkernel=auto
    ```
    Los arranques gráfico y silencioso no resultan útiles en un entorno de nube, donde se quiere que todos los registros se envíen al puerto serie. Es posible dejar la opción `crashkernel` configurada si es necesario, pero tenga en cuenta que este parámetro reducirá la cantidad de memoria disponible en la VM, al menos, en 128 MB, lo cual puede resultar problemático en tamaños de VM más reducidos.

1. Instale el Agente de Linux de Azure.
  
    El agente de Linux de Azure se requiere para aprovisionar una imagen de Linux en Azure.  Muchas distribuciones proporcionan el agente como paquete RPM o .deb (el paquete suele llamarse WALinuxAgent o walinuxagent).  El agente también se puede instalar manualmente siguiendo los pasos de la [Guía del agente de Linux](../extensions/agent-linux.md).

1. Asegúrese de que el servidor SSH se haya instalado y configurado para iniciarse en el tiempo de arranque.  Esta configuración suele ser la predeterminada.

1. No cree un espacio de intercambio en el disco del sistema operativo.
  
    El Agente de Linux de Azure puede configurar automáticamente un espacio de intercambio utilizando el disco de recursos local que se adjunta a la máquina virtual después de aprovisionarse en Azure. El disco de recursos local es un disco *temporal* que podría tener que vaciarse cuando la máquina virtual se desaprovisiona. Después de instalar el agente de Linux de Azure (paso 2 anterior), modifique los parámetros siguientes en /etc/waagent.conf según sea necesario.
    ```  
        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: Set this to your desired size.
    ```
1. Ejecute los comandos siguientes para desaprovisionar la máquina virtual.
  
     ```
     sudo waagent -force -deprovision
     export HISTSIZE=0
     logout
     ```  
   > [!NOTE]
   > Es posible que en VirtualBox aparezca el siguiente error `[Errno 5] Input/output error` después de ejecutar `waagent -force -deprovision`. Este mensaje de error no es crítico, por lo que se puede omitir.

* Apague la máquina virtual y cargue el disco duro virtual en Azure.
