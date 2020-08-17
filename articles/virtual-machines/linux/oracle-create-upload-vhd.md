---
title: Creación y carga de un disco duro virtual de Oracle Linux
description: Aprenda a crear y cargar un disco duro virtual de Azure (VHD) que contiene el sistema operativo Oracle Linux.
author: gbowerman
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 12/10/2019
ms.author: guybo
ms.openlocfilehash: 97fd61614eff05ddc542dbe17cf199663ab4cf1b
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371913"
---
# <a name="prepare-an-oracle-linux-virtual-machine-for-azure"></a>Preparación de una máquina virtual Oracle Linux para Azure

En este artículo se supone que ya ha instalado un sistema operativo Oracle Linux en un disco duro virtual. Existen varias herramientas para crear archivos .vhd; por ejemplo, una solución de virtualización como Hyper-V. Para obtener instrucciones, consulte [Instalación del rol de Hyper-V y configuración de una máquina Virtual](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

## <a name="oracle-linux-installation-notes"></a>Notas sobre la instalación de Oracle Linux
* Consulte también [Notas generales sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes) para obtener más consejos sobre la preparación de Linux para Azure.
* Hyper-V y Azure admiten Oracle Linux tanto con el kernel de empresa ininterrumpible (UEK) como con el kernel compatible con Red Hat.
* El UEK2 de Oracle no se admite en Hyper-V y Azure porque no incluye los controladores requeridos.
* El formato VHDX no se admite en Azure, solo el **VHD fijo**.  Puede convertir el disco al formato VHD con el Administrador de Hyper-V o el cmdlet Convert-VHD.
* Al instalar el sistema Linux se recomienda utilizar las particiones estándar en lugar de un LVM (que a menudo viene de forma predeterminada en muchas instalaciones). De este modo se impedirá que el nombre del LVM entre en conflicto con las máquinas virtuales clonadas, especialmente si en algún momento hace falta adjuntar un disco de SO a otra máquina virtual para solucionar problemas. [LVM](configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) o [RAID](configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) se pueden utilizar en discos de datos si así se prefiere.
* Las versiones de kernel de Linux inferiores a la versión 2.6.37 no admiten NUMA en Hyper-V con tamaños de VM más grandes. Este problema afecta principalmente a las distribuciones anteriores que usan el kernel Red Hat 2.6.32 de canal de subida y se ha corregido en Oracle Linux 6.6 y las versiones posteriores.
* No cree una partición de intercambio en el disco del SO. El agente de Linux se puede configurar para crear un archivo de intercambio en el disco de recursos temporal.  Puede encontrar más información al respecto en los pasos que vienen a continuación.
* En Azure, todos los discos duros virtuales deben tener un tamaño virtual alineado con 1 MB. Al convertir un disco sin procesar en un disco duro virtual, tiene que asegurarse de que su tamaño es un múltiplo de 1 MB antes de la conversión. Para más información, consulte [Notas sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes).
* Asegúrese de que el repositorio `Addons` está habilitado. Edite el archivo `/etc/yum.repos.d/public-yum-ol6.repo`(Oracle Linux 6) o `/etc/yum.repos.d/public-yum-ol7.repo`(Oracle Linux 7) y cambie la línea `enabled=0` a `enabled=1` en **[ol6_addons]** o **[ol7_addons]** en este archivo.

## <a name="oracle-linux-64-and-later"></a>Oracle Linux 6.4 y posterior
Debe completar los pasos de configuración específicos del sistema operativo para que la máquina virtual se ejecute en Azure.

1. Seleccione la máquina virtual en el panel central del Administrador de Hyper-V.
2. Haga clic en **Conectar** para abrir la ventana de la máquina virtual.
3. Desinstale NetworkManager ejecutando el comando siguiente:

    ```console
    # sudo rpm -e --nodeps NetworkManager
    ```

    **Nota:** si el paquete todavía no está instalado, se producirá un mensaje de error en este comando. Se espera que esto sea así.
4. Cree un archivo llamado **network** in the `/etc/sysconfig/` que contenga el texto siguiente:

    ```config   
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

5. Cree un archivo llamado **ifcfg-eth0** in the `/etc/sysconfig/network-scripts/` que contenga el texto siguiente:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

6. Modifique las reglas udev para impedir que se generen reglas estáticas para las interfaces Ethernet. Estas reglas pueden causar problemas al clonar una máquina virtual en Microsoft Azure o Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```

7. Asegúrese de que el servicio de red se inicie en el arranque ejecutando el comando siguiente:

    ```console
    # chkconfig network on
    ```

8. Instale python-pyasn1 ejecutando el comando siguiente:

    ```console
    # sudo yum install python-pyasn1
    ```

9. Modifique la línea de arranque de kernel de su configuración grub para que incluya parámetros de kernel adicionales para Azure. Para ello, abra "/boot/grub/menu.lst" en un editor de texto y asegúrese de que el kernel incluye los parámetros siguientes:

    ```config-grub
    console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

   Así se asegurará de que todos los mensajes de la consola se envían al primer puerto serie, lo que puede ayudar al soporte técnico de Azure con los problemas de depuración de errores.
   
   Además de lo anterior, se recomienda *quitar* los parámetros siguientes:

    ```config-grub
    rhgb quiet crashkernel=auto
    ```

   Los arranques gráfico y silencioso no resultan útiles en un entorno de nube, donde queremos que todos los registros se envíen al puerto serie.
   
   Es posible dejar la opción `crashkernel` configurada si así se desea, pero tenga en cuenta que este parámetro reducirá la cantidad de memoria disponible en la máquina virtual en 128 MB o más, lo cual puede resultar problemático en tamaños de VM más reducidos.
10. Asegúrese de que el servidor SSH se haya instalado y configurado para iniciarse en el tiempo de arranque.  Este es normalmente el valor predeterminado.
11. Instale el Agente de Linux de Azure ejecutando el comando siguiente: La versión más reciente es la 2.0.15.

    ```console
    # sudo yum install WALinuxAgent
    ```

    Tenga en cuanta que al instalar el paquete WALinuxAgent se eliminarán los paquetes NetworkManager y NetworkManager-gnome, si es que aún no se han eliminado como se indica en el paso 2.
12. No cree espacio de intercambio en el disco del SO.
    
    El Agente de Linux de Azure puede configurar automáticamente un espacio de intercambio utilizando el disco de recursos local que se adjunta a la máquina virtual después de aprovisionarse en Azure. Tenga en cuenta que el disco de recursos local es un disco *temporal* que debe vaciarse cuando la máquina virtual se desaprovisiona. Después de instalar el Agente de Linux de Azure (consulte el paso anterior), modifique apropiadamente los parámetros siguientes en /etc/waagent.conf:

    ```config-conf
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

13. Ejecute los comandos siguientes para desaprovisionar la máquina virtual y prepararla para aprovisionarse en Azure:

    ```console
    # sudo waagent -force -deprovision
    # export HISTSIZE=0
    # logout
    ```

14. Haga clic en **Acción -> Apagar** en el Administrador de Hyper-V. El VHD de Linux ya está listo para cargarse en Azure.

---
## <a name="oracle-linux-70-and-later"></a>Oracle Linux 7.0 y posterior
**Cambios en Oracle Linux 7**

La preparación de una máquina virtual Oracle Linux 7 para Azure es muy similar a Oracle Linux 6, aunque hay varias diferencias importantes que es necesario tener en cuenta:

* Azure admite Oracle Linux tanto con el kernel de empresa ininterrumpible (UEK) como con el kernel compatible con Red Hat. Se recomienda Oracle Linux con UEK.
* El paquete NetworkManager ya no entra en conflicto con el agente de Linux de Azure. Este paquete está instalado de manera predeterminada y recomendamos que no se quite.
* GRUB2 se utiliza ahora como cargador de arranque predeterminado; por ello, el proceso de edición de los parámetros de kernel ha cambiado (ver más adelante).
* XFS es ahora el sistema de archivos predeterminado. El sistema de archivos ext4 se puede seguir utilizando si así se desea.

**Pasos de configuración**

1. En el Administrador de Hyper-V, seleccione la máquina virtual.
2. Haga clic en **Conectar** para abrir una ventana de consola de la máquina virtual.
3. Cree un archivo llamado **network** in the `/etc/sysconfig/` que contenga el texto siguiente:

    ```config
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

4. Cree un archivo llamado **ifcfg-eth0** in the `/etc/sysconfig/network-scripts/` que contenga el texto siguiente:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

5. Modifique las reglas udev para impedir que se generen reglas estáticas para las interfaces Ethernet. Estas reglas pueden causar problemas al clonar una máquina virtual en Microsoft Azure o Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    ```

6. Asegúrese de que el servicio de red se inicie en el arranque ejecutando el comando siguiente:

    ```console
    # sudo chkconfig network on
    ```

7. Instale el paquete python-pyasn1 ejecutando el comando siguiente:

    ```console
    # sudo yum install python-pyasn1
    ```

8. Ejecute el comando siguiente para borrar los metadatos de yum actuales e instalar las actualizaciones:

    ```console 
    # sudo yum clean all
    # sudo yum -y update
    ```

9. Modifique la línea de arranque de kernel de su configuración grub para que incluya parámetros de kernel adicionales para Azure. Para ello, abra "/etc/default/grub" en un editor de texto y edite el parámetro `GRUB_CMDLINE_LINUX` ; por ejemplo:

    ```config-grub
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   Así también se asegurará de que todos los mensajes de la consola se envían al primer puerto serie, lo que puede ayudar al soporte técnico de Azure con los problemas de depuración de errores. También desactiva las convenciones de nomenclatura para los adaptadores de red en Oracle Linux 7 con el kernel de empresa ininterrumpible. Además de lo anterior, se recomienda *quitar* los parámetros siguientes:

    ```config-grub
       rhgb quiet crashkernel=auto
    ```
 
   Los arranques gráfico y silencioso no resultan útiles en un entorno de nube, donde queremos que todos los registros se envíen al puerto serie.
   
   Es posible dejar la opción `crashkernel` configurada si así se desea, pero tenga en cuenta que este parámetro reducirá la cantidad de memoria disponible en la máquina virtual en 128 MB o más, lo cual puede resultar problemático en tamaños de VM más reducidos.
10. Una vez que haya finalizado de editar "/etc/default/grub" como se indica anteriormente, ejecute el comando siguiente para volver a compilar la configuración de grub:

    ```console
    # sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

11. Asegúrese de que el servidor SSH se haya instalado y configurado para iniciarse en el tiempo de arranque.  Este es normalmente el valor predeterminado.
12. Instale el Agente de Linux de Azure ejecutando el comando siguiente:

    ```console
    # sudo yum install WALinuxAgent
    # sudo systemctl enable waagent
    ```

13. No cree espacio de intercambio en el disco del SO.
    
    El Agente de Linux de Azure puede configurar automáticamente un espacio de intercambio utilizando el disco de recursos local que se adjunta a la máquina virtual después de aprovisionarse en Azure. Tenga en cuenta que el disco de recursos local es un disco *temporal* que debe vaciarse cuando la máquina virtual se desaprovisiona. Después de instalar el Agente de Linux de Azure (consulte el paso anterior), modifique apropiadamente los parámetros siguientes en /etc/waagent.conf:

    ```config-conf
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

14. Ejecute los comandos siguientes para desaprovisionar la máquina virtual y prepararla para aprovisionarse en Azure:
    
    ```console
    # sudo waagent -force -deprovision
    # export HISTSIZE=0
    # logout
    ```

15. Haga clic en **Acción -> Apagar** en el Administrador de Hyper-V. El VHD de Linux ya está listo para cargarse en Azure.

## <a name="next-steps"></a>Pasos siguientes
Ya está listo para usar el archivo .vhd de Oracle Linux para crear nuevas máquinas virtuales en Azure. Si es la primera vez que carga el archivo .vhd en Azure, vea [Crear una VM Linux a partir de un disco personalizado](upload-vhd.md#option-1-upload-a-vhd).
