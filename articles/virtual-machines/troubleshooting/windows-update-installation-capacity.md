---
title: 'Solución de problemas de inicio del sistema operativo: capacidad de la instalación de Windows Update'
description: Pasos para resolver problemas en los que Windows Update (KB) obtiene un error y deja de responder en una VM de Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 6359e486-7b02-4c1e-995c-ef6d505f85f4
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 05/11/2020
ms.author: v-miegge
ms.openlocfilehash: 596303223554589ef26938486ccfd2281ccd46f5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86999112"
---
# <a name="troubleshoot-os-start-up--windows-update-installation-capacity"></a>Solución de problemas de inicio del sistema operativo: capacidad de la instalación de Windows Update

En este artículo se proporcionan los pasos necesarios para resolver problemas en una máquina virtual (VM) de Azure en que Windows Update (KB) obtiene un error y deja de responder.

## <a name="symptom"></a>Síntoma

Si usa Diagnósticos de arranque para ver la captura de pantalla de la VM, verá que esta muestra Windows Update (KB) en curso, pero genera un error con el código: **C01A001D**. La imagen siguiente muestra Windows Update (KB) atascado con el mensaje "Error C01A001D al aplicar la actualización ##### de ##### (######)":

![Windows Update (KB) está atascado con el mensaje "error C01A001D al aplicar la actualización X de Y (Z)".](./media/troubleshoot-windows-update-installation-capacity/1.png)

## <a name="cause"></a>Causa

En esta situación, el sistema operativo (SO) no puede completar una instalación de Windows Update (KB) porque no se puede crear un archivo principal en el sistema de archivos. Según este error, el sistema operativo no puede escribir ningún archivo en el disco.

## <a name="solution"></a>Solución

### <a name="process-overview"></a>Información general del proceso:

1. Cree una VM de reparación y acceda a esta.
1. Libere espacio en el disco.
1. Habilite Serial console y la recopilación del volcado de memoria.
1. Vuelva a compilar la VM.

> [!NOTE]
> Este error indica que el sistema operativo invitado no está operativo. Resuelva este problema en el modo sin conexión.

### <a name="create-and-access-a-repair-vm"></a>Creación de una VM de reparación y acceso a ella

1. Siga los pasos 1 a 3 de los [comandos de reparación de VM](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) para preparar una VM de reparación.
1. Use Conexión a Escritorio remoto para conectarse a la VM de reparación.

### <a name="free-up-space-on-the-disk"></a>Liberación de espacio en el disco

Para resolver el problema:

- Cambie el tamaño del disco al máximo de 1 TB si no tiene ya este tamaño.
- Realice un limpieza de disco.
- Desfragmente la unidad.

1. Compruebe si el disco está lleno. Si el tamaño del disco no llega a 1 TB, expándalo hasta un máximo de 1 TB [mediante PowerShell](../windows/expand-os-disk.md).
1. Si el disco ya tiene 1 TB, tendrá que realizar una limpieza de disco.
   1. Desconecte el disco de datos [de la VM rota](../windows/detach-disk.md).
   1. Conecte el disco de datos [a una VM que funcione](../windows/attach-disk-ps.md#attach-an-existing-data-disk-to-a-vm).
   1. Use la [herramienta Liberador de espacio en disco](https://support.microsoft.com/help/4026616/windows-10-disk-cleanup) para liberar espacio.
1. Una vez finalizado el cambio de tamaño y la limpieza, desfragmente la unidad con el siguiente comando:

   ```
   defrag <LETTER ASSIGN TO THE OS DISK>: /u /x /g
   ```
   
En función del nivel de fragmentación, la desfragmentación puede tardar varias horas.

### <a name="enable-the-serial-console-and-memory-dump-collection"></a>Habilitación de Serial console y de la recopilación del volcado de memoria

**Recomendado**: Antes de volver a compilar la VM, habilite Serial console y la recopilación del volcado de memoria mediante la ejecución del siguiente script:

1. Abra una sesión de símbolo del sistema con privilegios elevados como administrador.
1. Ejecute los comandos siguientes:

   **Habilitación de Serial console**:
   
   ```
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON 
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
   ```

1. Compruebe que el espacio disponible en el disco del sistema operativo sea mayor que el tamaño de memoria (RAM) de la VM.

   Si no hay suficiente espacio en el disco del sistema operativo, cambie la ubicación donde se creará el archivo de volcado de memoria y remita dicha ubicación a cualquier disco de datos conectado a la VM que tenga espacio libre suficiente. Para cambiar la ubicación, reemplace **%SystemRoot%** por la letra de unidad del disco de datos (por ejemplo, **F:** ) en los comandos siguientes.

   Configuración sugerida para habilitar el volcado del sistema operativo:

    **Cargar el disco del sistema operativo roto:**

   ```
   REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM 
   ```
   
   **Habilitar en ControlSet001**:

   ```
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
   ```
   
   **Habilitar en ControlSet002**:

   ```
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
   ```
   
   **Descargar disco del sistema operativo roto**:

   ```
   REG UNLOAD HKLM\BROKENSYSTEM
   ```
   
### <a name="rebuild-the-vm"></a>Recompilación de la máquina virtual

Realice el [paso 5 de los comandos de reparación de VM](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) para volver a crear la VM.
