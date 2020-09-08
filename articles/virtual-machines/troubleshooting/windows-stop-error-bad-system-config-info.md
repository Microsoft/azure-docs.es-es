---
title: 'Error de detención de Windows: 0x00000074 la información de configuración del sistema no es correcta'
description: En este artículo se proporcionan los pasos para resolver problemas en los que Windows no se puede arrancar y debe reiniciarse debido a una información de configuración del sistema incorrecta en una máquina virtual de Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: a1e47f6a-c7d5-4327-a7b0-ad48ed543641
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 08/24/2020
ms.author: v-miegge
ms.openlocfilehash: 071b5786127af31a2ad3266c128dbfb7cacad656
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88942015"
---
# <a name="windows-stop-error---0x00000074-bad-system-config-info"></a>Error de detención de Windows: 0x00000074 la información de configuración del sistema no es correcta

En este artículo se proporcionan los pasos para resolver problemas en los que Windows no se puede arrancar y debe reiniciarse debido a una información de configuración del sistema incorrecta en una máquina virtual de Azure.

## <a name="symptom"></a>Síntoma

Si usa [Diagnósticos de arranque](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/boot-diagnostics) para ver la captura de pantalla de la máquina virtual, verá que la captura muestra el código de detención de Windows **#0x00000074** o **BAD_SYSTEM_CONFIG_INFO**.

*Se produjo un problema con su equipo y se debe reiniciar. Puede reiniciar.* 
*Para más información sobre este problema y posibles soluciones, visite http://windows.com/stopcode* 
*Si se llama al departamento de soporte técnico, aporte esta información:* 
*Código de detención: BAD_SYSTEM_CONFIG_INFO*

  ![El código de detención de Windows 0x00000074, que también aparece como "BAD_SYSTEM_CONFIG_INFO". Windows informa al usuario de que el equipo ha tenido problemas y debe reiniciarse.](./media/windows-stop-error-bad-system-config-info/1.png)

## <a name="cause"></a>Causa

El código de detención **BAD_SYSTEM_CONFIG_INFO** se produce si el subárbol del registro **SYSTEM** parece estar dañado. Este error puede producirse por cualquiera de estos motivos:

- El subárbol del registro no se ha cerrado correctamente.
- El subárbol del registro está dañado.
- Faltan claves del Registro o valores.

## <a name="solution"></a>Solución

### <a name="process-overview"></a>Información general del proceso:

1. Cree una VM de reparación y acceda a ella.
1. Compruebe si hay daños en el subárbol.
1. Habilite Serial console y la recopilación del volcado de memoria.
1. Vuelva a compilar la VM.

> [!NOTE]
> Cuando se detecta este error, el sistema operativo invitado no está operativo. Este problema solo se puede solucionar en el modo sin conexión.

### <a name="create-and-access-a-repair-vm"></a>Creación de una máquina virtual de reparación y acceso a ella

1. Siga los pasos 1 a 3 de los [comandos de reparación de VM](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/repair-windows-vm-using-azure-virtual-machine-repair-commands) para preparar una VM de reparación.
1. Compruebe si hay daños en el subárbol.
1. Use Conexión a Escritorio remoto para conectarse a la máquina virtual de reparación.
1. Copie la carpeta `\windows\system32\config` y guárdela en la partición de disco correcta o en otra ubicación segura. Realice una copia de seguridad de esta carpeta como precaución, ya que modificará los archivos de registro críticos.

> [!NOTE]
> Realice una copia de la carpeta `\windows\system32\config` como copia de seguridad en caso de que necesite revertir los cambios que realice en el registro.

### <a name="check-for-hive-corruption"></a>Comprobación para ver si hay daños en el subárbol

Las instrucciones siguientes le ayudarán a determinar si la causa se debe a daños en el subárbol o si este no se ha cerrado correctamente. Si el subárbol no se ha cerrado correctamente, podrá desbloquear el archivo y arreglar la máquina virtual.

1. En la máquina virtual de reparación, abra la aplicación **Editor del Registro**. Escriba "REGEDIT" en la barra de búsqueda de Windows para encontrarlo.
1. En Editor del Registro, seleccione **HKEY_LOCAL_MACHINE** para resaltarlo y, a continuación, seleccione **Archivo > Cargar subárbol...** en el menú.
1. Vaya a `\windows\system32\config\SYSTEM` y seleccione **Abrir**.
1. Cuando se le pida que escriba un nombre, escriba **BROKENSYSTEM**.

   1. Si no se puede abrir el subárbol, o si está vacío, el subárbol está dañado. Si el subárbol está dañado, [abra una incidencia de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

     ![Se produce un error que indica que Editor del Registro no puede cargar el subárbol.](./media/windows-stop-error-bad-system-config-info/2.png)

   1. Si el subárbol se abre normalmente, el subárbol no se cerró correctamente. Continúe con el paso 5.

1. Para corregir un subárbol que no se ha cerrado correctamente, resalte **BROKENSYSTEM** y seleccione **Archivo > Descargar subárbol...** para desbloquear el archivo.

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

Realice el [paso 5 de los comandos de reparación de VM](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/repair-windows-vm-using-azure-virtual-machine-repair-commands#repair-process-example) para volver a crear la VM.
