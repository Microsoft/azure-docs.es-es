---
title: 'Solución de un error de arranque: error en la lectura de disco'
description: En este artículo se proporcionan los pasos para resolver problemas en que el disco no se puede leer en una VM de Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 817c9c5c-6a81-4b42-a6ad-0a0a11858b84
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 06/01/2020
ms.author: v-miegge
ms.openlocfilehash: f59903ed111be1fe414f4b3ded250d754c91d323
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87069146"
---
# <a name="troubleshoot-boot-error---disk-read-error-occurred"></a>Solución de un error de arranque: error de lectura de disco

En este artículo se proporcionan los pasos para resolver problemas en que el disco no se puede leer en una máquina virtual (VM) de Azure.

## <a name="symptoms"></a>Síntomas

Cuando use los [diagnósticos de arranque](./boot-diagnostics.md) para ver la captura de pantalla de la VM, verá que en dicha captura de pantalla se muestra el mensaje siguiente: "Error de lectura de disco". Presione Ctrl+Alt+Supr para reiniciar".

   ![Mensaje de error: Error de lectura de disco. Presione Control + Alt + Supr para reiniciar.](./media/disk-read-error-occurred/1.png)

## <a name="cause"></a>Causa

Este mensaje de error indica que la estructura del disco está dañada y es ilegible. Si usa una máquina virtual de generación 1, también es posible que la partición de disco que contiene los datos de configuración de arranque no esté establecida en el estado **Activo**.

## <a name="solution"></a>Solución

### <a name="process-overview"></a>Información general del proceso

1. Cree una VM de reparación y acceda a ella.
1. Seleccione una solución:
   - [Establecimiento del estado de la partición en Activo](#set-partition-status-to-active)
   - [Corrección de la partición de disco](#fix-the-disk-partition)
1. Habilite Serial console y la recopilación del volcado de memoria.
1. Vuelva a compilar la VM.

> [!NOTE]
> Cuando se detecta este error de arranque, el sistema operativo invitado (SO) no está operativo. Ahora solucionará el problema en el modo sin conexión.

### <a name="create-and-access-a-repair-vm"></a>Creación de una máquina virtual de reparación y acceso a ella

1. Siga los pasos 1 a 3 de los [comandos de reparación de VM](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) para preparar una VM de reparación.
1. Use Conexión a Escritorio remoto para conectarse a la VM de reparación.

### <a name="set-partition-status-to-active"></a>Establecimiento del estado de la partición en Activo

Las VM de generación 1 deben verificar primero que la partición del sistema operativo, que contiene el almacén BCD, está marcada con el estado **Activo**. Si tiene una VM de generación 2, vaya a [Corrección de la partición de disco](#fix-the-disk-partition), ya que la marca Estado quedó en desuso en la generación posterior.

1. Abra un símbolo del sistema con privilegios elevados (cmd.exe).
1. Escriba **diskpart** para iniciar la herramienta **DISKPART**.
1. Escriba **list disk** para enumerar los discos del sistema e identificar el disco duro virtual del sistema operativo conectado.
1. Cuando encuentre el VHD del sistema operativo conectado, escriba **sel disk #** para seleccionarlo. Vea la imagen siguiente para obtener un ejemplo en el que disco 1 es el disco duro virtual del sistema operativo conectado.

   ![Ventana diskpart con la salida del comando **list disk**, en que se muestran los discos 0 y 1 en la tabla. También muestra la salida del comando **sel disk 1**, en que el disco 1 es el disco seleccionado.](./media/disk-read-error-occurred/2.png)

1. Una vez seleccionado el disco, escriba **list partition** para mostrar las particiones del disco seleccionado.
1. Una vez identificada la partición de arranque, escriba **sel partition #** para seleccionar la partición. La partición de arranque suele tener un tamaño aproximado de 350 MB.  Vea la siguiente imagen, por ejemplo, en que la partición 1 es la partición de arranque.

   ![Ventana diskpart con la salida del comando **list partition**, en que se muestran las particiones 1 y 2 en la tabla. También se muestra la salida del comando *sel partition 1*, en que la partición 1 es el disco seleccionado.](./media/disk-read-error-occurred/3.png)

1. Escriba **detail partition** para comprobar el estado de la partición. Consulte las siguientes capturas de pantalla para obtener ejemplos de la partición que se está configurando en **Active: No** (Activo: no) o **Activo: Yes** (Activa: sí).

   **Active: No** (Activo: no)

   ![Ventana diskpart con la salida del comando **detail partition**, en que la partición 1 está establecida en **Active: No** (Activo: no).](./media/disk-read-error-occurred/4.png)

   **Active: Yes** (Activo: sí)

   ![Ventana diskpart con la salida del comando *detail partition*, en que la partición 1 está definida como *Active: Yes** (Activo: sí).](./media/disk-read-error-occurred/5.png)

1. Si la partición no está establecida en **Activo**, escriba **active** para cambiar la marca Activo.
1. Escriba **partición de detalles** para comprobar que el cambio de estado se completó correctamente y verifique que la salida incluye **Active: Yes** (Activo: sí). 
1. Escriba **exit** para cerrar la herramienta DISKPART y guardar los cambios de configuración.

### <a name="fix-the-disk-partition"></a>Corrección de la partición de disco

1. Abra un símbolo del sistema con privilegios elevados (cmd.exe).
1. Use el comando siguiente para ejecutar **CHKDSK** en los discos y corregir los errores:

   `chkdsk <DRIVE LETTER>: /f`

   Al agregar la opción de comando **/f**, se corregirán los errores en el disco. Asegúrese de reemplazar **< DRIVE LETTER >** por la letra del VHD del sistema operativo conectado.

### <a name="enable-the-serial-console-and-memory-dump-collection"></a>Habilitación de la consola serie y de la recopilación del volcado de memoria

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

   **Cargue el subárbol del Registro desde el disco del sistema operativo roto:**

   ```
   REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM
   ```

   **Habilitar en ControlSet001:**

   ```
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f 
   ```

   **Habilitar en ControlSet002:**

   ```
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f 
   ```

   **Descargar disco del sistema operativo roto:**

   ```
   REG UNLOAD HKLM\BROKENSYSTEM
   ```
   
### <a name="rebuild-the-vm"></a>Recompilación de la máquina virtual

Realice el [paso 5 de los comandos de reparación de VM](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) para volver a crear la VM.
