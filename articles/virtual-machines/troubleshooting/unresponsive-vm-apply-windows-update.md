---
title: La máquina virtual de Azure no responde y aparece el error C01A001D al aplicar Windows Update
description: En este artículo se proporcionan los pasos necesarios para resolver los problemas en los que Windows Update genera un error y deja de responder en una máquina virtual de Azure.
services: virtual-machines-windows
documentationcenter: ''
author: TobyTu
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: a97393c3-351d-4324-867d-9329e31b3528
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 03/31/2020
ms.author: v-mibufo
ms.openlocfilehash: 76c3f729a8520c7bff7b49a1d2200d7950f8a9f4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074298"
---
# <a name="vm-is-unresponsive-with-c01a001d-error-when-applying-windows-update"></a>La máquina virtual no responde y aparece el error C01A001D al aplicar Windows Update

En este artículo se proporcionan los pasos necesarios para resolver los problemas en los que Windows Update (KB) genera un error y deja de responder en una máquina virtual de Azure.

## <a name="symptoms"></a>Síntomas

Cuando se usan [diagnósticos de arranque](./boot-diagnostics.md) para ver la captura de pantalla de la máquina virtual, se muestra que Windows Update (KB) está en curso, pero se produce un error, cuyo código es: 'C01A001D'.

![Windows Update no responde](./media/unresponsive-vm-apply-windows-update/unresponsive-windows-update.png)

## <a name="cause"></a>Causa

No se puede crear un archivo de volcado del proceso en el sistema de archivos. El sistema operativo no puede escribir archivos en el disco.

## <a name="resolution"></a>Solución

### <a name="process-overview"></a>Información general del proceso

1. [Creación de una máquina virtual de reparación y acceso a ella](#create-and-access-a-repair-vm).
2. [Liberación de espacio en el disco duro](#free-up-space-on-the-hard-disk).
3. [se recomienda: antes de recompilar la máquina virtual, habilite la consola serie y la recopilación del volcado de memoria](#recommended-before-rebuilding-the-vm-enable-serial-console-and-memory-dump-collection).
4. [Recompilación de la máquina virtual](#rebuild-the-vm).

> [!NOTE]
> Cuando se produce este error, el sistema operativo invitado no funciona. Este problema solo se puede solucionar en el modo sin conexión.

### <a name="create-and-access-a-repair-vm"></a>Creación de una máquina virtual de reparación y acceso a ella

1. Para preparar una máquina virtual de reparación, siga los [pasos 1 a 3 de los comandos de reparación de máquinas virtuales](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md).
2. Conéctese a la máquina virtual de reparación mediante Conexión a Escritorio remoto.

### <a name="free-up-space-on-the-hard-disk"></a>Liberación de espacio en el disco duro

Si el disco no tiene 1 TB, debe cambiar su tamaño. Una vez que el disco tenga 1 TB, realice una limpieza del disco y una desfragmentación de la unidad.

1. Compruebe si el disco está lleno. Si el disco no llega a 1 TB, [expándalo hasta un máximo de 1 TB mediante PowerShell](../windows/expand-os-disk.md?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json).
2. Una vez que el disco tenga 1 TB, realice una limpieza del disco.
    - [Desconecte el disco de datos de la máquina virtual rota](../windows/detach-disk.md).
    - [Conecte el disco de datos a una máquina virtual que funcione](../windows/attach-disk-ps.md#attach-an-existing-data-disk-to-a-vm).
    - Use la [herramienta Liberador de espacio en disco](https://support.microsoft.com/help/4026616/windows-10-disk-cleanup) para liberar espacio.
3. Después de cambiar el tamaño del disco y liberar espacio en él, desfragmente la unidad:

    ```
    defrag <LETTER ASSIGN TO THE OS DISK>: /u /x /g
    ```
    En función del nivel de fragmentación, esta operación podría tardar horas.

### <a name="recommended-before-rebuilding-the-vm-enable-serial-console-and-memory-dump-collection"></a>Se recomienda: antes de recompilar la máquina virtual, habilite la consola serie y la recopilación del volcado de memoria

1. Abra una sesión de símbolo del sistema con privilegios elevados (Ejecutar como administrador).
2. Ejecute los comandos siguientes:

    Habilitación de la consola serie:

    ```
    bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
    bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
    ```
3. asegúrese de que el espacio disponible en el disco del sistema operativo sea al menos igual que el de la memoria de la máquina virtual (RAM).

    Si no hay suficiente espacio en el disco del sistema operativo, cambie la ubicación en que se creará el archivo de volcado de memoria y remítalo a un discos de datos conectado a la máquina virtual con espacio libre suficiente. Para cambiar la ubicación, reemplace `%SystemRoot%` por la letra de la unidad (por ejemplo, "F:") del disco de datos en los comandos siguientes:

    **Habilitación de la configuración para el volcado del sistema operativo:**

    Cargar el disco del sistema operativo roto:

    ```
    REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM
    ```

    Habilitar en ControlSet001:

    ```
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
    ```

    Habilitar en ControlSet002:

    ```
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
    ```

    Descargar disco del sistema operativo roto:

    ```
    REG UNLOAD HKLM\BROKENSYSTEM
    ```

### <a name="rebuild-the-vm"></a>Recompilación de la máquina virtual

Siga el [paso 5 de los comandos de reparación de máquinas virtuales](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) para volver a ensamblar la máquina virtual.
