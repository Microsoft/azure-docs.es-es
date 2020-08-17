---
title: Creación y carga de un VHD de Ubuntu Linux en Azure
description: Aprenda a crear y cargar un disco duro virtual de Azure (VHD) que contiene el sistema operativo Ubuntu Linux.
author: danielsollondon
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 06/06/2020
ms.author: danis
ms.openlocfilehash: 8b34e266214285f6483acca59050780810e62345
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373358"
---
# <a name="prepare-an-ubuntu-virtual-machine-for-azure"></a>Preparación de una máquina virtual Ubuntu para Azure


Ahora Ubuntu publica discos duros virtuales de Azure oficiales que se pueden descargar en [https://cloud-images.ubuntu.com/](https://cloud-images.ubuntu.com/). Si necesita crear su propia imagen de Ubuntu especializada para Azure, en lugar de usar el siguiente procedimiento manual se recomienda comenzar con estos VHD conocidos y personalizarlos según sea necesario. Las versiones más recientes de las imágenes siempre se encuentran en las siguientes ubicaciones:

* Ubuntu 16.04/Xenial: [ubuntu-16.04-server-cloudimg-amd64-disk1.vmdk](https://cloud-images.ubuntu.com/releases/xenial/release/ubuntu-16.04-server-cloudimg-amd64-disk1.vmdk)
* Ubuntu 18.04/Bionic: [bionic-server-cloudimg-amd64.vmdk](https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.vmdk)

## <a name="prerequisites"></a>Requisitos previos
En este artículo se supone que ya ha instalado un sistema operativo Ubuntu Linux en un disco duro virtual. Existen varias herramientas para crear archivos .vhd; por ejemplo, una solución de virtualización como Hyper-V. Para obtener instrucciones, consulte [Instalación del rol de Hyper-V y configuración de una máquina Virtual](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

**Notas de instalación de Ubuntu**

* Consulte también [Notas generales sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes) para obtener más consejos sobre la preparación de Linux para Azure.
* El formato VHDX no se admite en Azure, solo el **VHD fijo**.  Puede convertir el disco al formato VHD mediante el Administrador de Hyper-V o el cmdlet `Convert-VHD`.
* Al instalar el sistema Linux se recomienda utilizar las particiones estándar en lugar de un LVM (que a menudo viene de forma predeterminada en muchas instalaciones). De este modo se impedirá que el nombre del LVM entre en conflicto con las máquinas virtuales clonadas, especialmente si en algún momento hace falta adjuntar un disco de SO a otra máquina virtual para solucionar problemas. [LVM](configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) o [RAID](configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) se pueden utilizar en discos de datos si así se prefiere.
* No cree una partición ni un archivo de intercambio en el disco del SO. El agente de aprovisionamiento de cloud-init se puede configurar para crear una partición o archivo de intercambio en el disco de recursos temporal. Puede encontrar más información al respecto en los pasos que vienen a continuación.
* En Azure, todos los discos duros virtuales deben tener un tamaño virtual alineado con 1 MB. Al convertir un disco sin procesar en un disco duro virtual, tiene que asegurarse de que su tamaño es un múltiplo de 1 MB antes de la conversión. Para más información, consulte [Notas sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes).

## <a name="manual-steps"></a>Pasos manuales
> [!NOTE]
> Antes de intentar crear su propia imagen personalizada de Ubuntu para Azure, considere la posibilidad de utilizar las imágenes pregeneradas y probadas de [https://cloud-images.ubuntu.com/](https://cloud-images.ubuntu.com/) en su lugar.
> 
> 

1. Seleccione la máquina virtual en el panel central del Administrador de Hyper-V.

2. Haga clic en **Conectar** para abrir la ventana de la máquina virtual.

3. Sustituya los repositorios actuales de la imagen para utilizar el repositorio de Azure de Ubuntu.

    Antes de editar `/etc/apt/sources.list` se recomienda realizar una copia de seguridad:

    ```console
    # sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
    ```

    Ubuntu 16.04 y Ubuntu 18.04:

    ```console
    # sudo sed -i 's/http:\/\/archive\.ubuntu\.com\/ubuntu\//http:\/\/azure\.archive\.ubuntu\.com\/ubuntu\//g' /etc/apt/sources.list
    # sudo sed -i 's/http:\/\/[a-z][a-z]\.archive\.ubuntu\.com\/ubuntu\//http:\/\/azure\.archive\.ubuntu\.com\/ubuntu\//g' /etc/apt/sources.list
    # sudo apt-get update
    ```

4. Ahora, las imágenes de Ubuntu de Azure usan el [kernel adaptado a Azure](https://ubuntu.com/blog/microsoft-and-canonical-increase-velocity-with-azure-tailored-kernel). Actualice el sistema operativo al kernel más reciente adaptado a Azure e instale las herramientas de Linux de Azure (incluidas las dependencias de Hyper-V) mediante la ejecución de los siguientes comandos:

    Ubuntu 16.04 y Ubuntu 18.04:

    ```console
    # sudo apt update
    # sudo apt install linux-azure linux-image-azure linux-headers-azure linux-tools-common linux-cloud-tools-common linux-tools-azure linux-cloud-tools-azure
    (recommended) # sudo apt full-upgrade

    # sudo reboot
    ```

5. Modifique la línea de arranque de kernel para que Grub incluya los parámetros de kernel adicionales para Azure. Para ello, abra `/etc/default/grub` en un editor de texto, busque la variable llamada `GRUB_CMDLINE_LINUX_DEFAULT` (o agréguela si fuera necesario) y edítela para incluir los parámetros siguientes:

    ```text
    GRUB_CMDLINE_LINUX_DEFAULT="console=tty1 console=ttyS0,115200n8 earlyprintk=ttyS0,115200 rootdelay=300 quiet splash"
    ```

    Guarde este archivo y ciérrelo, y ejecute `sudo update-grub`. Así se asegurará de que todos los mensajes de la consola se envían al primer puerto serie, lo que puede ayudar al soporte técnico de Azure con los problemas de depuración de errores.

6. Asegúrese de que el servidor SSH se haya instalado y configurado para iniciarse en el tiempo de arranque.  Este es normalmente el valor predeterminado.

7. Instale cloud-init (el agente de aprovisionamiento) y el agente de Linux de Azure (el controlador de extensiones de invitado). Cloud-init usa netplan para configurar la red del sistema durante el aprovisionamiento y cada arranque posterior.

    ```console
    # sudo apt update
    # sudo apt install cloud-init netplan.io walinuxagent && systemctl stop walinuxagent
    ```

   > [!Note]
   >  El paquete `walinuxagent` puede eliminar los paquetes `NetworkManager` y `NetworkManager-gnome`, en caso de que estén instalados.

8. Quite las configuraciones predeterminadas de cloud-init y los artefactos de netplan sobrantes que puedan entrar en conflicto con el aprovisionamiento de cloud-init en Azure:

    ```console
    # rm -f /etc/cloud/cloud.cfg.d/50-curtin-networking.cfg /etc/cloud/cloud.cfg.d/curtin-preserve-sources.cfg
    # rm -f /etc/cloud/ds-identify.cfg
    # rm -f /etc/netplan/*.yaml
    ```

9. Configure cloud-init para aprovisionar el sistema mediante el origen de datos de Azure:

    ```console
    # cat > /etc/cloud/cloud.cfg.d/90_dpkg.cfg << EOF
    datasource_list: [ Azure ]
    EOF

    # cat > /etc/cloud/cloud.cfg.d/90-azure.cfg << EOF
    system_info:
       package_mirrors:
         - arches: [i386, amd64]
           failsafe:
             primary: http://archive.ubuntu.com/ubuntu
             security: http://security.ubuntu.com/ubuntu
           search:
             primary:
               - http://azure.archive.ubuntu.com/ubuntu/
             security: []
         - arches: [armhf, armel, default]
           failsafe:
             primary: http://ports.ubuntu.com/ubuntu-ports
             security: http://ports.ubuntu.com/ubuntu-ports
    EOF

    # cat > /etc/cloud/cloud.cfg.d/10-azure-kvp.cfg << EOF
    reporting:
      logging:
        type: log
      telemetry:
        type: hyperv
    EOF
    ```

10. Configure el agente de Linux de Azure para que se base en cloud-init para realizar el aprovisionamiento. Consulte el [proyecto de WALinuxAgent](https://github.com/Azure/WALinuxAgent) para obtener más información sobre estas opciones.

    ```console
    sed -i 's/Provisioning.Enabled=y/Provisioning.Enabled=n/g' /etc/waagent.conf
    sed -i 's/Provisioning.UseCloudInit=n/Provisioning.UseCloudInit=y/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf

    cat >> /etc/waagent.conf << EOF
    # For Azure Linux agent version >= 2.2.45, this is the option to configure,
    # enable, or disable the provisioning behavior of the Linux agent.
    # Accepted values are auto (default), waagent, cloud-init, or disabled.
    # A value of auto means that the agent will rely on cloud-init to handle
    # provisioning if it is installed and enabled, which in this case it will.
    Provisioning.Agent=auto
    EOF
    ```

11. Limpie cloud-init y los registros y artefactos en tiempo de ejecución del agente de Linux de Azure:

    ```console
    # sudo cloud-init clean --logs --seed
    # sudo rm -rf /var/lib/cloud/
    # sudo systemctl stop walinuxagent.service
    # sudo rm -rf /var/lib/waagent/
    # sudo rm -f /var/log/waagent.log
    ```

12. Ejecute los comandos siguientes para desaprovisionar la máquina virtual y prepararla para aprovisionarse en Azure:

    > [!NOTE]
    > El comando `sudo waagent -force -deprovision+user` intentará limpiar el sistema y hacer que sea apto para el reaprovisionamiento. La opción `+user` elimina la última cuenta de usuario aprovisionada y los datos asociados.

    > [!WARNING]
    > El desaprovisionamiento mediante el comando superior no garantiza que se haya borrado toda información confidencial de la imagen y que se pueda redistribuir.

    ```console
    # sudo waagent -force -deprovision+user
    # rm -f ~/.bash_history
    # export HISTSIZE=0
    # logout
    ```

13. Haga clic en **Acción -> Apagar** en el Administrador de Hyper-V.

14. Azure solo acepta VHD de tamaño fijo. Si el disco del sistema operativo de la VM no es un VHD de tamaño fijo, use el cmdlet `Convert-VHD` de PowerShell y especifique la opción `-VHDType Fixed`. Eche un vistazo a los documentos de `Convert-VHD` aquí: [Convert-VHD](/powershell/module/hyper-v/convert-vhd?view=win10-ps).


## <a name="next-steps"></a>Pasos siguientes
Ya está listo para usar el disco duro virtual de Ubuntu para crear nuevas máquinas virtuales de Azure. Si es la primera vez que carga el archivo .vhd en Azure, vea [Crear una VM Linux a partir de un disco personalizado](upload-vhd.md#option-1-upload-a-vhd).
