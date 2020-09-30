---
title: La máquina virtual no responde durante la aplicación de la directiva de configuración de directivas de auditorías
description: En este artículo se proporcionan los pasos para resolver aquellos problemas en los que la máquina virtual no responde durante la aplicación de la directiva de configuración de directivas de auditorías, lo cual impide el inicio de una máquina virtual de Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 3f6383b5-81fa-49ea-9434-2fe475e4cbef
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 08/24/2020
ms.author: v-miegge
ms.openlocfilehash: bc41783bf977806b5f9bba5b953f1f581ad07f18
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299531"
---
# <a name="virtual-machine-is-unresponsive-while-applying-audit-policy-configuration-policy"></a>La máquina virtual no responde durante la aplicación de la directiva de configuración de directivas de auditorías

En este artículo se proporcionan los pasos para resolver aquellos problemas en los que la máquina virtual no responde durante la aplicación de la directiva de configuración de directivas de auditorías, lo cual impide el inicio de una máquina virtual de Azure.

## <a name="symptom"></a>Síntoma

Si usa [Diagnósticos de arranque](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/boot-diagnostics) para ver la captura de pantalla de la máquina virtual, verá que en la captura de pantalla se muestra que el sistema operativo no respondía durante el arranque con el mensaje **Aplicando directiva de configuración de directivas de auditoría**.

  ![Sistema operativo arrancando con el mensaje: "Aplicando directiva de configuración de directivas de auditoría"](./media/vm-unresponsive-applying-audit-configuration-policy/1.png)

  ![Sistema operativo arrancando en Windows Server 2012 con el mensaje: "Aplicando directiva de configuración de directivas de auditoría"](./media/vm-unresponsive-applying-audit-configuration-policy/2.png)

## <a name="cause"></a>Causa

Hay bloqueos en conflicto cuando la directiva intenta limpiar los perfiles de usuario antiguos.

> [!NOTE]
> Esto solo se aplica a Windows Server 2012 y Windows Server 2012 R2.

Esta es la directiva problemática: *Configuración del equipo\Directivas\Plantillas administrativas\Sistema/Perfiles de usuario\Delete user profiles older than a specified number of days on system restart (Eliminar perfiles de usuario anteriores al número de días especificado en el reinicio del sistema).*

## <a name="solution"></a>Solución

### <a name="process-overview"></a>Información general del proceso

1. Cree una VM de reparación y acceda a ella.
1. Deshabilite la directiva.
1. Habilite Serial console y la recopilación del volcado de memoria.
1. Vuelva a compilar la VM.
1. Recopile el archivo de volcado de memoria y envíe una incidencia de soporte técnico.

### <a name="create-and-access-a-repair-vm"></a>Creación de una máquina virtual de reparación y acceso a ella

1. Siga los pasos 1 a 3 de los [comandos de reparación de VM](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/repair-windows-vm-using-azure-virtual-machine-repair-commands) para preparar una VM de reparación.
1. Use Conexión a Escritorio remoto para conectarse a la VM de reparación.

### <a name="disable-the-policy"></a>Deshabilite la directiva.

1. En la máquina virtual de reparación, abra el **Editor del Registro**.
1. Busque la clave **HKEY_LOCAL_MACHINE** y seleccione **Archivo > Cargar subárbol** en el menú.

   ![Navegación en el Editor del Registro para cargar un subárbol.](./media/vm-unresponsive-applying-audit-configuration-policy/3.png)

   - Puede usar Cargar subárbol para cargar las claves del Registro desde un sistema sin conexión. En este caso, el sistema es el disco roto conectado a la máquina virtual de reparación.
   - La configuración de todo el sistema se almacena en **HKEY_LOCAL_MACHINE** y se puede abreviar como **HKLM**.

1. En el disco conectado, abra el archivo `\windows\system32\config\SOFTWARE`.

   - Cuando se le pida un nombre, escriba **BROKENSOFTWARE**.
   - Para comprobar que se cargó **BROKENSOFTWARE**, expanda **HKEY_LOCAL_MACHINE** y busque la clave **BROKENSOFTWARE** agregada.

1. Vaya a **BROKENSOFTWARE** y compruebe si la clave **CleanupProfiles** existe en el subárbol cargado.

   - Si la clave existe, se establece la directiva **CleanupProfiles**. Su valor representa la directiva de retención medida en días.
   - Si la clave no existe, no se establece la directiva **CleanupProfiles**. En esta situación, vaya directamente a [enviar una incidencia de soporte técnico con un archivo de volcado de memoria](#collect-the-memory-dump-file-and-submit-a-support-ticket).

1. Elimine la clave **CleanupProfiles** con este comando:

   `reg delete "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows\System" /v CleanupProfiles /f`

1. Descargue el subárbol **BROKENSOFTWARE** con este comando:

   `reg unload HKLM\BROKENSOFTWARE`

### <a name="enable-the-serial-console-and-memory-dump-collection"></a>Habilitación de la consola serie y de la recopilación del volcado de memoria

**Recomendado**: Antes de volver a compilar la VM, habilite Serial console y la recopilación del volcado de memoria mediante la ejecución del siguiente script:

1. Abra una sesión de símbolo del sistema con privilegios elevados como administrador.
1. Enumere los datos del almacén de BCD y determine el identificador del cargador de arranque, que usará en el paso siguiente.

   1. En el caso de una máquina virtual de la Generación 1, escriba el siguiente comando y anote el identificador que se muestra:

      `bcdedit /store <BOOT PARTITON>:\boot\bcd /enum`

      - En el comando, reemplace `<BOOT PARTITON>` por la letra de la partición del disco conectado que contiene la carpeta de arranque.

        ![Figura 4 en la que aparece la salida de la lista del almacén de BCD en una máquina virtual de la Generación 1, que muestra el número de identificador bajo el cargador de arranque de Windows.](./media/vm-unresponsive-applying-audit-configuration-policy/4.png)

   1. En el caso de una máquina virtual de generación 2, escriba el siguiente comando y anote el identificador que se muestra:

      `bcdedit /store <LETTER OF THE EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /enum`

      - En el comando, reemplace `<LETTER OF THE EFI SYSTEM PARTITION>` por la letra de la partición de sistema EFI.
      - Puede resultar útil iniciar la consola de Administración de discos para identificar la partición del sistema adecuada etiquetada como **Partición de sistema EFI**.
      - El identificador puede ser un GUID único o puede ser el valor predeterminado **bootmgr**.

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
   
### <a name="rebuild-the-virtual-machine"></a>Recompilación de la máquina virtual

1. Realice el [paso 5 de los comandos de reparación de VM](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/repair-windows-vm-using-azure-virtual-machine-repair-commands#repair-process-example) para volver a crear la VM.

1. Pruebe si la máquina virtual se inicia normalmente para ver si el problema se ha solucionado.

   - Si el problema no se ha corregido, continúe con [recopilar un archivo de volcado de memoria y enviar una incidencia de soporte técnico](#collect-the-memory-dump-file-and-submit-a-support-ticket).
   - Si se ha corregido el problema, no es necesario realizar más pasos.

Si el problema se resolvió, la directiva se ha deshabilitado ahora localmente. En el caso de una solución permanente, no use la directiva CleanupProfiles en las máquinas virtuales, ya que se eliminarán automáticamente los perfiles de usuario. Use un método diferente para realizar limpiezas de perfiles, como una tarea programada o un script.

**No use esta directiva:** 
*Equipo\Plantillas administrativas\Sistema\Perfiles de usuario\Delete user profiles older than a specified number of days on system restart (Eliminar perfiles de usuario anteriores al número de días especificado en el reinicio del sistema).*

### <a name="the-issue-should-now-be-fixed"></a>Ahora se debería haber solucionado ya el problema.

Pruebe la máquina virtual para asegurarse de que funciona de la manera habitual. Si sigue teniendo problemas, puede continuar con la siguiente sección para obtener más ayuda.

### <a name="collect-the-memory-dump-file-and-submit-a-support-ticket"></a>Recopilación del archivo de volcado de memoria y envío de una incidencia de soporte técnico

Para resolver este problema, primero debe recopilar el archivo de volcado de memoria del bloqueo y ponerse en contacto con el soporte técnico en relación a ese archivo. Para recopilar el archivo de volcado de memoria, siga estos pasos:

#### <a name="attach-the-os-disk-to-a-new-repair-vm"></a>Conecte el disco del sistema operativo a una nueva VM de recuperación

1. Siga los pasos 1 a 3 de los [comandos de reparación de la máquina virtual](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/repair-windows-vm-using-azure-virtual-machine-repair-commands) para preparar una nueva máquina virtual de reparación.
1. Use Conexión a Escritorio remoto para conectarse a la VM de reparación.

#### <a name="locate-the-dump-file-and-submit-a-support-ticket"></a>Busque el archivo de volcado de memoria y envíe una incidencia de soporte técnico

1. En la VM de reparación, vaya a la carpeta de Windows en el disco del sistema operativo conectado. Si la letra de unidad que se asigna al disco del sistema operativo conectado es *F*, deberá ir a `F:\Windows`.
1. Busque el archivo `memory.dmp` y, a continuación, [envíe una incidencia de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) con el archivo de volcado de memoria.
1. Si tiene problemas para encontrar el archivo `memory.dmp`, use en su lugar [llamadas de interrupción no enmascarable (NMI) en la consola serie](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/serial-console-windows#use-the-serial-console-for-nmi-calls). Siga la guía para [generar aquí un archivo de volcado de memoria completo mediante llamadas NMI](https://docs.microsoft.com/windows/client-management/generate-kernel-or-complete-crash-dump).
