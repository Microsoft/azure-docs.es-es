---
title: Creación y carga de un VHD de SUSE Linux en Azure
description: Aprenda a crear y cargar un disco duro virtual de Azure (VHD) que contiene un sistema operativo SUSE Linux.
author: gbowerman
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 03/12/2018
ms.author: guybo
ms.openlocfilehash: 73e07c612486d5f48b1ad3eca8044a561549092b
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87292125"
---
# <a name="prepare-a-sles-or-opensuse-virtual-machine-for-azure"></a>Preparación de una máquina virtual SLES u openSUSE para Azure


En este artículo se supone que ya ha instalado un sistema operativo Linux SUSE u openSUSE en un disco duro virtual. Existen varias herramientas para crear archivos .vhd; por ejemplo, una solución de virtualización como Hyper-V. Para obtener instrucciones, consulte [Instalación del rol de Hyper-V y configuración de una máquina Virtual](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

## <a name="sles--opensuse-installation-notes"></a>Notas sobre la instalación de SLES/openSUSE
* Consulte también [Notas generales sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes) para obtener más consejos sobre la preparación de Linux para Azure.
* El formato VHDX no se admite en Azure, solo el **VHD fijo**.  Puede convertir el disco al formato VHD con el Administrador de Hyper-V o el cmdlet Convert-VHD.
* Al instalar el sistema Linux se recomienda utilizar las particiones estándar en lugar de un LVM (que a menudo viene de forma predeterminada en muchas instalaciones). De este modo se impedirá que el nombre del LVM entre en conflicto con las máquinas virtuales clonadas, especialmente si en algún momento hace falta adjuntar un disco de SO a otra máquina virtual para solucionar problemas. [LVM](configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) o [RAID](configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) se pueden utilizar en discos de datos si así se prefiere.
* No cree una partición de intercambio en el disco del SO. El agente de Linux se puede configurar para crear un archivo de intercambio en el disco de recursos temporal.  Puede encontrar más información al respecto en los pasos que vienen a continuación.
* En Azure, todos los discos duros virtuales deben tener un tamaño virtual alineado con 1 MB. Al convertir un disco sin procesar en un disco duro virtual, tiene que asegurarse de que su tamaño es un múltiplo de 1 MB antes de la conversión. Para más información, consulte [Notas sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes).

## <a name="use-suse-studio"></a>Uso de SUSE Studio
[SUSE Studio](https://studioexpress.opensuse.org/) puede crear y administrar fácilmente sus imágenes de SLES y openSUSE para Hyper-V y Azure. Este es el enfoque recomendado para personalizar sus propias imágenes SLES y openSUSE.

Como alternativa a la creación de su propio VHD, SUSE también publica imágenes de BYOS ("traiga su propia suscripción") para SLES en [VMDepot](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/04/using-and-contributing-vms-to-vm-depot.pdf).

## <a name="prepare-suse-linux-enterprise-server-11-sp4"></a>Preparación de SUSE Linux Enterprise Server 11 SP4
1. Seleccione la máquina virtual en el panel central del Administrador de Hyper-V.
2. Haga clic en **Conectar** para abrir la ventana de la máquina virtual.
3. Registre el sistema de SUSE Linux Enterprise para permitir que descargue actualizaciones e instale paquetes.
4. Actualice el sistema con las revisiones más recientes:

    ```console
    # sudo zypper update
    ```

1. Instale el agente de Linux de Azure desde el repositorio de SLES (SLE11-Public-Cloud-Module):

    ```console
    # sudo zypper install python-azure-agent
    ```

1. Compruebe si waagent está establecido en "on" en chkconfig y, en caso contrario, puede habilitarlo para que se inicie automáticamente:

    ```console
    # sudo chkconfig waagent on
    ```

7. Compruebe si el servicio waagent se está ejecutando y, en caso contrario, inícielo: 

    ```console
    # sudo service waagent start
    ```

8. Modifique la línea de arranque de kernel de su configuración grub para que incluya parámetros de kernel adicionales para Azure. Para ello, abra "/boot/grub/menu.lst" en un editor de texto y asegúrese de que el kernel predeterminado incluye los parámetros siguientes:

    ```config-grub
    console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

    Así se asegurará de que todos los mensajes de la consola se envían al primer puerto serie, lo que puede ayudar al soporte técnico de Azure con los problemas de depuración de errores.
9. Confirme que tanto /boot/grub/menu.lst como /etc/fstab hacen referencia a su UUID (by-uuid) en lugar de al identificador del disco (by-id). 
   
    Obtención del UUID del disco

    ```console
    # ls /dev/disk/by-uuid/
    ```

    Si se usa /dev/disk/by-id/, actualice /boot/grub/menu.lst y /etc/fstab con el valor by-uuid adecuado.
   
    Antes del cambio
   
    `root=/dev/disk/by-id/SCSI-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx-part1`
   
    Después del cambio
   
    `root=/dev/disk/by-uuid/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

10. Modifique las reglas udev para impedir que se generen reglas estáticas para las interfaces Ethernet. Estas reglas pueden causar problemas al clonar una máquina virtual en Microsoft Azure o Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```

11. Se recomienda editar el archivo "/etc/sysconfig/network/dhcp" y cambiar el parámetro `DHCLIENT_SET_HOSTNAME` por lo siguiente:

    ```config
    DHCLIENT_SET_HOSTNAME="no"
    ```

12. En "/etc/sudoers", convierta en comentario o quite las líneas siguientes, si existen:

    ```text
    Defaults targetpw   # ask for the password of the target user i.e. root
    ALL    ALL=(ALL) ALL   # WARNING! Only use this together with 'Defaults targetpw'!
    ```

13. Asegúrese de que el servidor SSH se haya instalado y configurado para iniciarse en el tiempo de arranque.  Este es normalmente el valor predeterminado.
14. No cree espacio de intercambio en el disco del SO.
    
    El Agente de Linux de Azure puede configurar automáticamente un espacio de intercambio utilizando el disco de recursos local que se adjunta a la máquina virtual después de aprovisionarse en Azure. Tenga en cuenta que el disco de recursos local es un disco *temporal* que debe vaciarse cuando la máquina virtual se desaprovisiona. Después de instalar el Agente de Linux de Azure (consulte el paso anterior), modifique apropiadamente los parámetros siguientes en /etc/waagent.conf:

    ```config-conf
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

15. Ejecute los comandos siguientes para desaprovisionar la máquina virtual y prepararla para aprovisionarse en Azure:

    ```console
    # sudo waagent -force -deprovision
    # export HISTSIZE=0
    # logout
    ```
16. Haga clic en **Acción -> Apagar** en el Administrador de Hyper-V. El VHD de Linux ya está listo para cargarse en Azure.

---
## <a name="prepare-opensuse-131"></a>Preparación de openSUSE 13.1+
1. Seleccione la máquina virtual en el panel central del Administrador de Hyper-V.
2. Haga clic en **Conectar** para abrir la ventana de la máquina virtual.
3. En el shell, ejecute el comando '`zypper lr`'. Si este comando devuelve una salida similar a la siguiente, los repositorios están configurados según lo esperado y no es necesario ningún ajuste (tenga en cuenta que los números de versión pueden variar):

   | # | Alias                 | Nombre                  | habilitado | Actualizar
   | - | :-------------------- | :-------------------- | :------ | :------
   | 1 | Cloud:Tools_13.1      | Cloud:Tools_13.1      | Sí     | Sí
   | 2 | openSUSE_13.1_OSS     | openSUSE_13.1_OSS     | Sí     | Sí
   | 3 | openSUSE_13.1_Updates | openSUSE_13.1_Updates | Sí     | Sí

    Si el comando devuelve el mensaje "No repositories defined...", utilice los comandos siguientes para agregar estos repositorios:

    ```console
    # sudo zypper ar -f http://download.opensuse.org/repositories/Cloud:Tools/openSUSE_13.1 Cloud:Tools_13.1
    # sudo zypper ar -f https://download.opensuse.org/distribution/13.1/repo/oss openSUSE_13.1_OSS
    # sudo zypper ar -f http://download.opensuse.org/update/13.1 openSUSE_13.1_Updates
    ```

    Puede verificar entonces que se han agregado los repositorios al volver a ejecutar el comando '`zypper lr`'. En caso de que no haya un repositorio de actualizaciones relevante habilitado, habilítelo con el comando siguiente:

    ```console
    # sudo zypper mr -e [NUMBER OF REPOSITORY]
    ```

4. Actualice el kernel a la versión más reciente disponible:

    ```console
    # sudo zypper up kernel-default
    ```

    O para actualizar el sistema con todos los parches más recientes:

    ```console
    # sudo zypper update
    ```

5. Instale el Agente de Linux de Azure.

    ```console
    # sudo zypper install WALinuxAgent
    ```

6. Modifique la línea de arranque de kernel de su configuración grub para que incluya parámetros de kernel adicionales para Azure. Para ello, abra "/boot/grub/menu.lst" en un editor de texto y asegúrese de que el kernel predeterminado incluye los parámetros siguientes:

    ```config-grub
     console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

   Así se asegurará de que todos los mensajes de la consola se envían al primer puerto serie, lo que puede ayudar al soporte técnico de Azure con los problemas de depuración de errores. Además, elimine los parámetros siguientes de la línea de arranque de kernel, si es que están:

    ```config-grub
     libata.atapi_enabled=0 reserve=0x1f0,0x8
    ```

7. Se recomienda editar el archivo "/etc/sysconfig/network/dhcp" y cambiar el parámetro `DHCLIENT_SET_HOSTNAME` por lo siguiente:

    ```config
     DHCLIENT_SET_HOSTNAME="no"
    ```

8. **Importante:** En "/etc/sudoers", convierta en comentario o quite las líneas siguientes, si existen:

    ```text
    Defaults targetpw   # ask for the password of the target user i.e. root
    ALL    ALL=(ALL) ALL   # WARNING! Only use this together with 'Defaults targetpw'!
    ```

9. Asegúrese de que el servidor SSH se haya instalado y configurado para iniciarse en el tiempo de arranque.  Este es normalmente el valor predeterminado.
10. No cree espacio de intercambio en el disco del SO.

    El Agente de Linux de Azure puede configurar automáticamente un espacio de intercambio utilizando el disco de recursos local que se adjunta a la máquina virtual después de aprovisionarse en Azure. Tenga en cuenta que el disco de recursos local es un disco *temporal* que debe vaciarse cuando la máquina virtual se desaprovisiona. Después de instalar el Agente de Linux de Azure (consulte el paso anterior), modifique apropiadamente los parámetros siguientes en /etc/waagent.conf:

    ```config-conf
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

11. Ejecute los comandos siguientes para desaprovisionar la máquina virtual y prepararla para aprovisionarse en Azure:

    ```console
    # sudo waagent -force -deprovision
    # export HISTSIZE=0
    # logout
    ```

12. Asegúrese de que el Agente de Linux de Azure se ejecute al inicio:

    ```console
    # sudo systemctl enable waagent.service
    ```

13. Haga clic en **Acción -> Apagar** en el Administrador de Hyper-V. El VHD de Linux ya está listo para cargarse en Azure.

## <a name="next-steps"></a>Pasos siguientes
Ya está listo para usar el disco duro virtual de SUSE para crear nuevas máquinas virtuales de Azure. Si es la primera vez que carga el archivo .vhd en Azure, vea [Crear una VM Linux a partir de un disco personalizado](upload-vhd.md#option-1-upload-a-vhd).
