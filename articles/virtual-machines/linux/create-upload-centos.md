---
title: Creación y carga de un disco duro virtual de Linux basado en CentOS
description: Aprenda a crear y cargar un disco duro virtual de Azure (VHD) que contiene un sistema operativo Linux basado en CentOS.
author: gbowerman
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 11/25/2019
ms.author: guybo
ms.openlocfilehash: 9097fb4aefe168ce36793d13f892fbbeab10ad56
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372746"
---
# <a name="prepare-a-centos-based-virtual-machine-for-azure"></a>Preparación de una máquina virtual basada en CentOS para Azure

Aprenda a crear y cargar un disco duro virtual de Azure (VHD) que contiene un sistema operativo Linux basado en CentOS.

* [Preparación de una máquina virtual CentOS 6.x para Azure](#centos-6x)
* [Preparación de una máquina virtual CentOS 7.0+ para Azure](#centos-70)


## <a name="prerequisites"></a>Prerequisites

En este artículo se supone que ya ha instalado un sistema operativo Linux CentOS (o un derivado similar) en un disco duro virtual. Existen varias herramientas para crear archivos .vhd; por ejemplo, una solución de virtualización como Hyper-V. Para obtener instrucciones, consulte [Instalación del rol de Hyper-V y configuración de una máquina Virtual](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

**Notas de instalación de CentOS**

* Consulte también [Notas generales sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes) para obtener más consejos sobre la preparación de Linux para Azure.
* El formato VHDX no se admite en Azure, solo el **VHD fijo**.  Puede convertir el disco al formato VHD con el Administrador de Hyper-V o el cmdlet Convert-VHD. Si utiliza VirtualBox, deberá seleccionar **Tamaño fijo** en lugar del tamaño predeterminado asignado dinámicamente al crear el disco.
* Al instalar el sistema Linux se *recomienda* utilizar las particiones estándar en lugar de un LVM (que a menudo viene de forma predeterminada en muchas instalaciones). De este modo se impedirá que el nombre del LVM entre en conflicto con las máquinas virtuales clonadas, especialmente si en algún momento hace falta adjuntar un disco de SO a otra máquina virtual idéntica para solucionar problemas. [LVM](configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) o [RAID](configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) se pueden utilizar en discos de datos.
* Se requiere la compatibilidad de kernel para el montaje de sistemas de archivos UDF. Al arrancar Azure la primera vez, la configuración de aprovisionamiento se pasa a la máquina virtual Linux a través de medios con formato UDF conectados al invitado. El agente Linux de Azure debe poder montar el sistema de archivos UDF para leer su configuración y aprovisionar la máquina virtual.
* Las versiones de kernel de Linux inferiores a la versión 2.6.37 no admiten NUMA en Hyper-V con tamaños de VM más grandes. Este problema afecta principalmente a las distribuciones anteriores que usan el kernel Red Hat 2.6.32 de canal de subida y se ha corregido en RHEL 6.6 (kernel-2.6.32-504). Los sistemas que ejecutan kernels personalizados cuyas versiones son anteriores a la versión 2.6.37, o bien kernels basados en RHEL cuyas versiones son anteriores a la versión 2.6.32-504, deben establecer el parámetro de inicio `numa=off` en la línea de comandos de kernel en grub.conf. Para obtener más información, consulte Red Hat [KB 436883](https://access.redhat.com/solutions/436883).
* No cree una partición de intercambio en el disco del SO. El agente de Linux se puede configurar para crear un archivo de intercambio en el disco de recursos temporal.  Puede encontrar más información al respecto en los pasos que vienen a continuación.
* En Azure, todos los discos duros virtuales deben tener un tamaño virtual alineado con 1 MB. Al convertir un disco sin procesar en un disco duro virtual, tiene que asegurarse de que su tamaño es un múltiplo de 1 MB antes de la conversión. Para más información, consulte [Notas sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes).

## <a name="centos-6x"></a>CentOS 6.x

1. En el Administrador de Hyper-V, seleccione la máquina virtual.

2. Haga clic en **Conectar** para abrir una ventana de consola de la máquina virtual.

3. En CentOS 6, NetworkManager puede interferir con el Agente de Linux de Azure. Desinstale el paquete ejecutando el comando siguiente:

    ```bash
    sudo rpm -e --nodeps NetworkManager
    ```

4. Cree o edite el archivo `/etc/sysconfig/network` y agregue el siguiente texto:

    ```console
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

5. Cree o edite el archivo `/etc/sysconfig/network-scripts/ifcfg-eth0` y agregue el siguiente texto:

    ```console
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

6. Modifique las reglas udev para impedir que se generen reglas estáticas para las interfaces Ethernet. Estas reglas pueden causar problemas al clonar una máquina virtual en Microsoft Azure o Hyper-V:

    ```bash
    sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```

7. Asegúrese de que el servicio de red se inicie en el arranque ejecutando el comando siguiente:

    ```bash
    sudo chkconfig network on
    ```

8. Si desea utilizar los espejos OpenLogic hospedados en los centros de datos de Azure, sustituya el archivo `/etc/yum.repos.d/CentOS-Base.repo` por los siguientes repositorios.  Así se agregará también el repositorio **[openlogic]** , que incluye paquetes adicionales como el Agente de Linux de Azure:

   ```console
   [openlogic]
   name=CentOS-$releasever - openlogic packages for $basearch
   baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
   enabled=1
   gpgcheck=0

   [base]
   name=CentOS-$releasever - Base
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #released updates
   [updates]
   name=CentOS-$releasever - Updates
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #additional packages that may be useful
   [extras]
   name=CentOS-$releasever - Extras
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #additional packages that extend functionality of existing packages
   [centosplus]
   name=CentOS-$releasever - Plus
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #contrib - packages by Centos Users
   [contrib]
   name=CentOS-$releasever - Contrib
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/contrib/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
   ```

    > [!Note]
    > En el resto de esta guía se supondrá que usa al menos el repositorio `[openlogic]`, que se utilizará para instalar el agente de Linux de Azure siguiente.

9. Agregue la línea siguiente a /etc/yum.conf:

    ```console
    http_caching=packages
    ```

10. Ejecute el siguiente comando para borrar los metadatos de yum actuales y actualizar el sistema con los últimos paquetes:

    ```bash
    yum clean all
    ```

    A menos que vaya a crear una imagen de una versión anterior de CentOS, se recomienda actualizar todos los paquetes a la versión más reciente:

    ```bash
    sudo yum -y update
    ```

    Después de ejecutar este comando es posible que sea preciso reiniciar el equipo.

11. (Opcional) Instale los controladores para los servicios de integración de Linux (LIS).

    > [!IMPORTANT]
    > El paso se **requiere** para CentOS 6.3 y las versiones anteriores, y opcional para las versiones posteriores.

    ```bash
    sudo rpm -e hypervkvpd  ## (may return error if not installed, that's OK)
    sudo yum install microsoft-hyper-v
    ```

    Como alternativa, puede seguir las instrucciones de instalación manual de la [página de descarga de LIS](https://www.microsoft.com/download/details.aspx?id=51612) para instalar el RPM en la máquina virtual.

12. Instale el Agente de Linux de Azure y sus dependencias. Inicie y habilite el servicio waagent:

    ```bash
    sudo yum install python-pyasn1 WALinuxAgent
    sudo service waagent start
    sudo chkconfig waagent on
    ```


    La instalación del paquete WALinuxAgent eliminará los paquetes NetworkManager y NetworkManager-gnome, si es que aún no se han eliminado, como se ha indicado en el paso 3.

13. Modifique la línea de arranque de kernel de su configuración grub para que incluya parámetros de kernel adicionales para Azure. Para ello, abra `/boot/grub/menu.lst` en un editor de texto y asegúrese de que el kernel predeterminado incluye los parámetros siguientes:

    ```console
    console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

    Así también se asegurará de que todos los mensajes de la consola se envían al primer puerto serie, lo que puede ayudar al soporte técnico de Azure con los problemas de depuración de errores.

    Además de lo anterior, se recomienda *quitar* los parámetros siguientes:

    ```console
    rhgb quiet crashkernel=auto
    ```

    Los arranques gráfico y silencioso no resultan útiles en un entorno de nube, donde queremos que todos los registros se envíen al puerto serie.  Es posible dejar la opción `crashkernel` configurada si así se desea, pero tenga en cuenta que este parámetro reducirá la cantidad de memoria disponible en la máquina virtual en 128 MB o más, lo cual puede resultar problemático en tamaños de VM más reducidos.

    > [!Important]
    > CentOS 6.5 y las versiones anteriores también deben establecer el parámetro `numa=off` del kernel. Consulte Red Hat [KB 436883](https://access.redhat.com/solutions/436883).

14. Asegúrese de que el servidor SSH se haya instalado y configurado para iniciarse en el tiempo de arranque.  Este es normalmente el valor predeterminado.

15. No cree espacio de intercambio en el disco del SO.

    El Agente de Linux de Azure puede configurar automáticamente un espacio de intercambio utilizando el disco de recursos local que se adjunta a la máquina virtual después de aprovisionarse en Azure. Tenga en cuenta que el disco de recursos local es un disco *temporal* que debe vaciarse cuando la máquina virtual se desaprovisiona. Después de instalar el Agente de Linux de Azure (vea el paso anterior), modifique los parámetros siguientes en `/etc/waagent.conf` de la forma adecuada:

    ```console
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048 ## NOTE: set this to whatever you need it to be.
    ```

16. Ejecute los comandos siguientes para desaprovisionar la máquina virtual y prepararla para aprovisionarse en Azure:

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

17. Haga clic en **Acción -> Apagar** en el Administrador de Hyper-V. El VHD de Linux ya está listo para cargarse en Azure.



## <a name="centos-70"></a>CentOS 7.0+

**Cambios en CentOS 7 (y derivados similares)**

La preparación de una máquina virtual CentOS 7 para Azure es muy similar a CentOS 6, aunque hay varias diferencias importantes que es necesario tener en cuenta:

* El paquete NetworkManager ya no entra en conflicto con el agente de Linux de Azure. Este paquete está instalado de manera predeterminada y recomendamos que no se quite.
* GRUB2 se utiliza ahora como cargador de arranque predeterminado; por ello, el proceso de edición de los parámetros de kernel ha cambiado (ver más adelante).
* XFS es ahora el sistema de archivos predeterminado. El sistema de archivos ext4 se puede seguir utilizando si así se desea.

**Pasos de configuración**

1. En el Administrador de Hyper-V, seleccione la máquina virtual.

2. Haga clic en **Conectar** para abrir una ventana de consola de la máquina virtual.

3. Cree o edite el archivo `/etc/sysconfig/network` y agregue el siguiente texto:

    ```console
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

4. Cree o edite el archivo `/etc/sysconfig/network-scripts/ifcfg-eth0` y agregue el siguiente texto:

    ```console
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

5. Modifique las reglas udev para impedir que se generen reglas estáticas para las interfaces Ethernet. Estas reglas pueden causar problemas al clonar una máquina virtual en Microsoft Azure o Hyper-V:

    ```bash
    sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    ```

6. Si desea utilizar los espejos OpenLogic hospedados en los centros de datos de Azure, sustituya el archivo `/etc/yum.repos.d/CentOS-Base.repo` por los siguientes repositorios.  De este modo se agregará también el repositorio **[openlogic]** que incluye paquetes para el agente de Linux de Azure:

   ```console
   [openlogic]
   name=CentOS-$releasever - openlogic packages for $basearch
   baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
   enabled=1
   gpgcheck=0
    
   [base]
   name=CentOS-$releasever - Base
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    
   #released updates
   [updates]
   name=CentOS-$releasever - Updates
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    
   #additional packages that may be useful
   [extras]
   name=CentOS-$releasever - Extras
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    
   #additional packages that extend functionality of existing packages
   [centosplus]
   name=CentOS-$releasever - Plus
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
   ```
    
   > [!Note]
   > En el resto de esta guía se supondrá que usa al menos el repositorio `[openlogic]`, que se utilizará para instalar el agente de Linux de Azure siguiente.

7. Ejecute el comando siguiente para borrar los metadatos de yum actuales e instalar las actualizaciones:

    ```bash
    sudo yum clean all
    ```

    A menos que vaya a crear una imagen de una versión anterior de CentOS, se recomienda actualizar todos los paquetes a la versión más reciente:

    ```bash
    sudo yum -y update
    ```

    Después de ejecutar este comando es posible que sea preciso reiniciar el equipo.

8. Modifique la línea de arranque de kernel de su configuración grub para que incluya parámetros de kernel adicionales para Azure. Para ello, abra `/etc/default/grub` en un editor de texto y edite el parámetro `GRUB_CMDLINE_LINUX`, por ejemplo:

    ```console
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   Así también se asegurará de que todos los mensajes de la consola se envían al primer puerto serie, lo que puede ayudar al soporte técnico de Azure con los problemas de depuración de errores. Esto también desactiva las nuevas convenciones de nomenclatura de CentOS 7 para NIC. Además de lo anterior, se recomienda *quitar* los parámetros siguientes:

    ```console
    rhgb quiet crashkernel=auto
    ```

    Los arranques gráfico y silencioso no resultan útiles en un entorno de nube, donde queremos que todos los registros se envíen al puerto serie. Es posible dejar la opción `crashkernel` configurada si así se desea, pero tenga en cuenta que este parámetro reducirá la cantidad de memoria disponible en la máquina virtual en 128 MB o más, lo cual puede resultar problemático en tamaños de VM más reducidos.

9. Una vez que termine de editar `/etc/default/grub` tal como se indicó antes, ejecute el comando siguiente para volver a compilar la configuración de GRUB:

    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

10. Si va a generar la imagen de **VMWare, VirtualBox o KVM**, asegúrese de que los controladores de Hyper-V se incluyen en initramfs:

    Edite `/etc/dracut.conf`y agregue contenido:

    ```console
    add_drivers+=" hv_vmbus hv_netvsc hv_storvsc "
    ```

    Vuelva a generar initramfs:

    ```bash
    sudo dracut -f -v
    ```

11. Instale el Agente de Linux de Azure y sus dependencias:

    ```bash
    sudo yum install python-pyasn1 WALinuxAgent
    sudo systemctl enable waagent
    ```

12. No cree espacio de intercambio en el disco del SO.

    El Agente de Linux de Azure puede configurar automáticamente un espacio de intercambio utilizando el disco de recursos local que se adjunta a la máquina virtual después de aprovisionarse en Azure. Tenga en cuenta que el disco de recursos local es un disco *temporal* que debe vaciarse cuando la máquina virtual se desaprovisiona. Después de instalar el Agente de Linux de Azure (vea el paso anterior), modifique los parámetros siguientes en `/etc/waagent.conf` de la forma adecuada:

    ```console
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

13. Ejecute los comandos siguientes para desaprovisionar la máquina virtual y prepararla para aprovisionarse en Azure:

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

14. Haga clic en **Acción -> Apagar** en el Administrador de Hyper-V. El VHD de Linux ya está listo para cargarse en Azure.

## <a name="next-steps"></a>Pasos siguientes

Ya está listo para usar el disco duro virtual de CentOS para crear nuevas máquinas virtuales de Azure. Si es la primera vez que carga el archivo .vhd en Azure, vea [Crear una VM Linux a partir de un disco personalizado](upload-vhd.md#option-1-upload-a-vhd).
