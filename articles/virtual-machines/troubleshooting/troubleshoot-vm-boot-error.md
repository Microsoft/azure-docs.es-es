---
title: Arranque de máquinas virtuales Linux en Rescate de GRUB
description: La máquina virtual no puede arrancar porque ha entrado en una consola de rescate.
services: virtual-machines-windows
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: troubleshooting
ms.date: 08/28/2019
ms.author: tiag
ms.openlocfilehash: 5a2fd7fcfdae8559bfb39bffff7c73c7082a86aa
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543289"
---
# <a name="linux-vm-boots-to-grub-rescue"></a>Arranque de máquinas virtuales Linux en Rescate de GRUB

Hemos detectado que la máquina virtual ha entrado en una consola de rescate. Este problema se produce cuando se han realizado cambios recientemente en el kernel de una máquina virtual Linux (por ejemplo, una actualización del kernel) y la máquina virtual ya no se inicia correctamente porque durante el proceso de arranque se producen errores en el kernel. Durante el proceso de arranque, cuando el cargador de arranque intenta localizar el kernel de Linux y le cede el control de arranque, si se produce un error al realizar esta cesión, la máquina virtual entra en una consola de rescate.

Si en el futuro no puede conectarse a una máquina virtual, puede consultar una captura de pantalla de la máquina virtual en la hoja de diagnóstico de arranque de Azure Portal. Esto puede ayudar a diagnosticar el problema y determinar si la causa es un error de inicio similar.

## <a name="recommended-steps"></a>Pasos recomendados

Siga los pasos de mitigación que se indican a continuación en función del error que reciba:

### <a name="error---unknown-filesystem"></a>Error: sistema de archivos desconocido

* Si recibe un error similar a **sistema de archivos desconocido**, puede deberse a que el sistema de archivos está dañado en la partición de arranque o a una configuración incorrecta del kernel.

   * Si los problemas provienen del sistema de archivos, siga los pasos que se indican en el artículo [Recuperación de Linux: no se puede utilizar SSH en una máquina virtual Linux debido a errores en el sistema de archivos (fsck, inodes)](/archive/blogs/linuxonazure/linux-recovery-cannot-ssh-to-linux-vm-due-to-file-system-errors-fsck-inodes).
   * Si tiene problemas con el kernel, siga los pasos descritos en el artículo [Cómo recuperar una máquina virtual Linux de Azure de problemas de arranque relacionados con el kernel](https://support.microsoft.com/help/4091524/how-recover-azure-linux-vm-from-kernel-related-boot-related-issues) o [Recuperación de Linux: corrección de problemas relacionados con el kernel que no afectan al arranque mediante chroot](http://linuxonazure.azurewebsites.net/linux-recovery-fixing-non-boot-issues-related-to-kernel-problems-using-chroot/).
   
### <a name="error---file-not-found"></a>Error: archivo no encontrado

* Si aparece un error similar a **Error 15: no se encontró el archivo o el disco de RAM inicial**  o bien **no se encontró el archivo initrd/initramfs**, siga estos pasos.

    * Si el archivo que falta es `/boot/grub2/grub.cfg` o `initrd/initramfs`, siga estos pasos:

    1. Asegúrese de que `/etc/default/grub` existe y tiene la configuración correcta o deseada. Si no sabe cuál es la configuración predeterminada, puede probar con una máquina virtual en funcionamiento.

    2. A continuación, ejecute el siguiente comando para volver a generar la configuración: `grub2-mkconfig -o /boot/grub2/grub.cfg`

   * Si el archivo que falta es `/boot/grub/menu.lst`, este error está relacionado con las versiones anteriores del sistema operativo (**RHEL 6.x**, **Centos 6.x** y **Ubuntu 14.04**), por lo que los comandos pueden variar. Tendrá que preparar un servidor antiguo y probarlo allí para asegurarse de los comandos proporcionados son correctos.

### <a name="error---no-such-partition"></a>Error: no se trata de una partición

* Si aparece un error similar a **No se trata de una partición**, consulte [Caso práctico: aparece el error "no se trata de una partición" al iniciar la máquina virtual después de intentar ampliar la unidad del sistema operativo](/archive/blogs/shwetanayak/case-scenario-no-such-partition-error-while-trying-to-start-the-vm-after-attempting-to-extend-the-os-drive).

### <a name="error---grubcfg-file-not-found"></a>Error: no se encontró el archivo grub.cfg

* Si aparece un error similar a **No se encontró el archivo /boot/grub2/grub.cfg**, siga estos pasos.

    * Si el archivo que falta es `/boot/grub2/grub.cfg` o `initrd/initramfs`, siga estos pasos:

    1. Asegúrese de que `/etc/default/grub` existe y tiene la configuración correcta o deseada. Si no sabe cuál es la configuración predeterminada, puede probar con una máquina virtual en funcionamiento.

    2. A continuación, ejecute el siguiente comando para volver a generar la configuración: `grub2-mkconfig -o /boot/grub2/grub.cfg`.

   * Si el archivo que falta es `/boot/grub/menu.lst`, este error está relacionado con las versiones anteriores del sistema operativo (**RHEL 6.x**, **Centos 6.x** y **Ubuntu 14.04**), por lo que los comandos pueden variar. Prepare un servidor antiguo y pruébelo allí para asegurarse de los comandos proporcionados son correctos.

## <a name="next-steps"></a>Pasos siguientes

* [Información general del agente de máquina virtual de Azure](../extensions/agent-windows.md)
* [Características y extensiones de las máquinas virtuales para Windows](../extensions/features-windows.md)
