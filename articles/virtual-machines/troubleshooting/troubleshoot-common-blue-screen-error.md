---
title: Errores de pantalla azul al iniciar una máquina virtual de Azure | Microsoft Docs
description: Más información sobre cómo solucionar el problema que el error de pantalla azul recibe durante el inicio | Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 09/28/2018
ms.author: genli
ms.openlocfilehash: b382efc4d283d64ce0f833bde9104fa2e3bc973a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088552"
---
# <a name="windows-shows-blue-screen-error-when-booting-an-azure-vm"></a>Windows muestra un error de pantalla azul al iniciar una máquina virtual de Azure.
En este artículo se describen los errores de pantalla azul que pueden surgir al iniciar una máquina virtual (VM) de Windows en Microsoft Azure. Se proporcionan los pasos para ayudarle a recopilar datos de una incidencia de soporte técnico. 


## <a name="symptom"></a>Síntoma 

Una máquina virtual de Windows no se inicia. Al revisar las capturas de pantalla de inicio en [Diagnósticos de arranque](./boot-diagnostics.md), ve uno de los mensajes de error siguientes en una pantalla azul:

- Se produjo un problema en nuestro equipo y se debe reiniciar. Solo vamos a recopilar alguna información de error y, a continuación, podrá reiniciar.
- Se produjo un problema con su PC y se debe reiniciar.

En esta sección se muestran los mensajes de error habituales que pueden producirse al administrar máquinas virtuales:

## <a name="cause"></a>Causa

Se puede producir un error de detención por diversos motivos. Las causas más comunes son:

- Problema con un controlador
- Memoria o archivo de sistema dañados
- Una aplicación tiene acceso a un sector prohibido de la memoria

## <a name="collect-memory-dump-file"></a>Recopilación de un archivo de volcado de memoria

Para resolver este problema, primero debe recopilar el archivo de volcado del bloqueo y ponerse en contacto con el soporte técnico en relación al archivo de volcado de memoria. Para recopilar el archivo de volcado de memoria, siga estos pasos:

### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Conecte el disco del sistema operativo a una máquina virtual de recuperación.

1. Tome una instantánea del disco del sistema operativo de la máquina virtual afectada como copia de seguridad. Para obtener más información, consulte [Instantánea de un disco](../windows/snapshot-copy-managed-disk.md).
2. [Conecte el disco del sistema operativo a una máquina virtual de recuperación](./troubleshoot-recovery-disks-portal-windows.md). 
3. Escritorio remoto a la máquina virtual de recuperación.

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

    ```config-reg
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f

    reg unload HKLM\BROKENSYSTEM
    ```

3. [Desconecte el disco del sistema operativo y, a continuación, vuelva a adjuntarlo a la máquina virtual afectada](./troubleshoot-recovery-disks-portal-windows.md).
4. Inicie la máquina virtual para reproducir el problema y, a continuación, se generará un archivo de volcado de memoria.
5. Conecte el disco del sistema operativo a una máquina virtual de recuperación, recopile un archivo de volcado de memoria y, a continuación, [envíe una incidencia de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) con el archivo de volcado de memoria.
