---
title: Restablecimiento de la contraseña local de Linux en máquinas virtuales de Azure | Microsoft Docs
description: Introducción de los pasos para restablecer la contraseña local de Linux en la máquina virtual de Azure
services: virtual-machines-linux
documentationcenter: ''
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
ms.date: 08/20/2019
ms.author: delhan
ms.openlocfilehash: c6bfd5b9ff3626593916533f27c5c2755cebcb13
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87028488"
---
# <a name="how-to-reset-local-linux-password-on-azure-vms"></a>Restablecimiento de las contraseñas locales de Linux en máquinas virtuales de Azure

En este artículo se presentan varios métodos para restablecer las contraseñas locales de Linux en una máquina virtual. Si la cuenta de usuario ha expirado o simplemente desea crear una cuenta, puede usar los métodos siguientes para crear una cuenta de administrador local y recuperar el acceso a la máquina virtual.

## <a name="symptoms"></a>Síntomas

No puede iniciar sesión en la máquina virtual y recibe un mensaje que indica que la contraseña que usó es incorrecta. Además, no puede usar VMAgent para restablecer la contraseña en Azure Portal.

## <a name="manual-password-reset-procedure"></a>Procedimiento de restablecimiento de contraseñas manual

> [!NOTE]
> Los siguientes pasos no se aplican a la máquina virtual con un disco no administrado.

1. Realice una instantánea del disco del sistema operativo de la máquina virtual afectada, cree un disco a partir de la instantánea y luego conecte el disco a una máquina virtual de solución de problemas. Para más información, consulte [Solución de problemas de una máquina virtual Windows mediante la conexión del disco del sistema operativo a una máquina virtual de recuperación mediante Azure Portal](troubleshoot-recovery-disks-portal-linux.md).

2. Conéctese a la máquina virtual de solución de problemas mediante Escritorio remoto.

3.  Ejecute el siguiente comando SSH en la máquina virtual de solución de problemas para convertirse en superusuario.

    ```bash
    sudo su
    ```

4.  Ejecute **fdisk -l** o examine los registros del sistema para encontrar el disco recién conectado. Busque el nombre de unidad que va a montar. A continuación, en la máquina virtual temporal, busque en el archivo de registro correspondiente.

    ```bash
    grep SCSI /var/log/kern.log (ubuntu)
    grep SCSI /var/log/messages (centos, suse, oracle)
    ```

    Ejemplo de salida del comando grep:

    ```bash
    kernel: [ 9707.100572] sd 3:0:0:0: [sdc] Attached SCSI disk
    ```

5.  Cree un punto de montaje denominado **tempmount**.

    ```bash
    mkdir /tempmount
    ```

6.  Monte el disco del sistema operativo en el punto de montaje. Generalmente, es necesario montar *sdc1* o *sdc2*. En función de la partición de hospedaje en el directorio */etc* del disco de la máquina dañada.

    ```bash
    mount /dev/sdc1 /tempmount
    ```

7.  Cree copias de los principales archivos de credenciales antes de realizar cambios:

    ```bash
    cp /etc/passwd /etc/passwd_orig    
    cp /etc/shadow /etc/shadow_orig    
    cp /tempmount/etc/passwd /etc/passwd
    cp /tempmount/etc/shadow /etc/shadow 
    cp /tempmount/etc/passwd /tempmount/etc/passwd_orig
    cp /tempmount/etc/shadow /tempmount/etc/shadow_orig
    ```

8.  Restablezca la contraseña del usuario que necesite:

    ```bash
    passwd <<USER>> 
    ```

9.  Mueva los archivos modificados a la ubicación correcta del disco dañado de la máquina.

    ```bash
    cp /etc/passwd /tempmount/etc/passwd
    cp /etc/shadow /tempmount/etc/shadow
    cp /etc/passwd_orig /etc/passwd
    cp /etc/shadow_orig /etc/shadow
    ```

10. Vuelva a la raíz y desmonte el disco.

    ```bash
    cd /
    umount /tempmount
    ```

11. En Azure Portal, desasocie el disco de la máquina virtual de solución de problemas.

12. [Cambie el disco del sistema operativo de la máquina virtual afectada](troubleshoot-recovery-disks-portal-linux.md#swap-the-os-disk-for-the-vm).

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de máquinas virtuales de Azure mediante la conexión de un disco de SO a otra máquina virtual de Azure](https://social.technet.microsoft.com/wiki/contents/articles/18710.troubleshoot-azure-vm-by-attaching-os-disk-to-another-azure-vm.aspx)

* [CLI de Azure: cómo eliminar y volver a implementar una máquina virtual desde VHD](/archive/blogs/linuxonazure/azure-cli-how-to-delete-and-re-deploy-a-vm-from-vhd)
