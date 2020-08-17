---
title: El inicio de la máquina virtual está detenido en "Preparando Windows. No apague el equipo" en Azure | Microsoft Docs
description: Introducción a los pasos de solución del problema donde el inicio de la máquina virtual se bloquea en "Preparando Windows. No apague el equipo."
services: virtual-machines-windows
documentationcenter: ''
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: article
ms.date: 09/18/2018
ms.author: delhan
ms.openlocfilehash: d7e56fe36af3d841cfd888dd6c1bf05502837cdd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079849"
---
# <a name="vm-startup-is-stuck-on-getting-windows-ready-dont-turn-off-your-computer-in-azure"></a>El inicio de la máquina virtual está detenido en "Preparando Windows. No apague el equipo"en Azure

En este artículo se describen las pantallas "En preparación" y "Getting Windows ready" (Preparando Windows) que pueden aparecer al iniciar una máquina virtual Windows en Microsoft Azure. Se proporcionan los pasos para ayudarle a recopilar datos de una incidencia de soporte técnico.

 

## <a name="symptoms"></a>Síntomas

No se inicia una máquina virtual Windows. Cuando use **Diagnósticos de arranque** para obtener la captura de pantalla de la máquina virtual, puede ver que la máquina virtual muestra el mensaje "En preparación" o "Getting Windows ready" (Preparando Windows).

![Ejemplo de mensaje para Windows Server 2012 R2](./media/troubleshoot-vm-configure-update-boot/message1.png)

![Mensaje de ejemplo](./media/troubleshoot-vm-configure-update-boot/message2.png)

## <a name="cause"></a>Causa

Normalmente, este problema se produce cuando el servidor está realizando el último reinicio después de que la configuración ha cambiado. El cambio en la configuración se puede inicializar mediante actualizaciones de Windows o cambios en los roles o las características del servidor. En el caso de Windows Update, si el tamaño de las actualizaciones es grande, el sistema operativo necesitará más tiempo para reconfigurar los cambios.

## <a name="collect-an-os-memory-dump"></a>Recopilación de un volcado de memoria del sistema operativo

Si el problema no se soluciona después de esperar a que se hayan procesado los cambios, deberá recopilar un archivo de volcado de memoria y ponerse en contacto con el soporte técnico. Para recopilar el archivo de volcado de memoria, siga estos pasos:

### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Conecte el disco del sistema operativo a una máquina virtual de recuperación.

1. Tome una instantánea del disco del sistema operativo de la máquina virtual afectada como copia de seguridad. Para obtener más información, consulte [Instantánea de un disco](../windows/snapshot-copy-managed-disk.md).
2. [Conecte el disco del sistema operativo a una máquina virtual de recuperación](./troubleshoot-recovery-disks-portal-windows.md).
3. Escritorio remoto a la máquina virtual de recuperación. 
4. Si el disco del sistema operativo está cifrado, debe desactivar el cifrado antes de pasar al siguiente paso. Para más información, consulte [Descifrado del disco de sistema cifrado en la máquina virtual que no arranca](troubleshoot-bitlocker-boot-error.md#solution).

### <a name="locate-dump-file-and-submit-a-support-ticket"></a>Busque el archivo de volcado de memoria y envíe una incidencia de soporte técnico.

1. En la máquina virtual de recuperación, vaya a la carpeta de Windows en el disco del sistema operativo conectado. Si la letra de unidad que se asigna al disco del sistema operativo conectado es F, deberá ir a F:\Windows.
2. Busque el archivo memory.dmp y, a continuación, [envíe una incidencia de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) con el archivo de volcado de memoria. 

Si no encuentra el archivo de volcado de memoria, vaya el paso siguiente para habilitar el registro de volcado de memoria y Serial Console.

### <a name="enable-dump-log-and-serial-console"></a>Habilitación del registro de volcado de memoria y Serial Console

Para habilitar el registro de volcado de memoria y Serial Console, ejecute el siguiente script.

1. Abra la sesión de símbolo del sistema con privilegios elevados (Ejecutar como administrador).
2. Ejecute el siguiente script:

    En este script, se supone que la letra de unidad que se asigna al disco del sistema operativo conectado es F. Reemplácela por el valor adecuado en la máquina virtual.

    ```powershell
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REM Enable Serial Console
    bcdedit /store F:\boot\bcd /set {bootmgr} displaybootmenu yes
    bcdedit /store F:\boot\bcd /set {bootmgr} timeout 5
    bcdedit /store F:\boot\bcd /set {bootmgr} bootems yes
    bcdedit /store F:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
    bcdedit /store F:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200

    REM Suggested configuration to enable OS Dump
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    reg unload HKLM\BROKENSYSTEM
    ```

    1. Asegúrese de que hay suficiente espacio en disco para asignar tanta memoria como la RAM, que depende del tamaño que se seleccione para esta máquina virtual.
    2. Si no hay espacio suficiente o se trata de un máquina virtual de gran tamaño (serie G, GS o E), puede cambiar la ubicación en la que se creará este archivo y que haga referencia a cualquier otro disco de datos asociado a la máquina virtual. Para ello, deberá cambiar la clave siguiente:
    
        ```console
        reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

        REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f
        REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f

        reg unload HKLM\BROKENSYSTEM
        ```

3. [Desconecte el disco del sistema operativo y, a continuación, vuelva a adjuntarlo a la máquina virtual afectada](./troubleshoot-recovery-disks-portal-windows.md).
4. Inicie la máquina virtual y acceda a la consola serie.
5. Seleccione **Enviar interrupción no enmascarable (NMI)** para desencadenar el volcado de memoria.
    ![Imagen de dónde enviar una interrupción no enmascarable](./media/troubleshoot-vm-configure-update-boot/run-nmi.png)
6. Vuelva a conectar el disco del sistema operativo a una máquina virtual de recuperación y recopile el archivo de volcado.

## <a name="contact-microsoft-support"></a>Consultar al soporte técnico de Microsoft

Después de recopilar el archivo de volcado de memoria, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para analizar la causa principal.
