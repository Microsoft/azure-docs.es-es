---
title: 'Error de detención de Windows: estado, Sin memoria'
description: En este artículo se indican los pasos de resolución de problemas cuando Windows no se inicia y muestra el estado "estado, Sin memoria".
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: d29c7e49-4578-43c8-b829-831da4e48932
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 06/26/2020
ms.author: v-mibufo
ms.openlocfilehash: 33b4c59e14301e496d0eddafa7bdfdf201b7aa29
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87005912"
---
# <a name="windows-stop-error---status-no-memory"></a>Error de detención de Windows: estado, Sin memoria

En este artículo se indican los pasos de resolución de problemas cuando Windows no se inicia y muestra el estado o código de error #0xC0000017, también conocido como "estado, Sin memoria".

## <a name="symptom"></a>Síntoma

Cuando use [Diagnósticos de arranque](./boot-diagnostics.md) para ver la captura de pantalla de la máquina virtual (VM), verá que muestra el código de error: `0xC0000017`. En función de la versión de Windows que ejecute, es posible que vea este código en **Administración de arranque de Windows** o en la **pantalla Recuperación**.

   **Administración de arranque de Windows**

   ![Administración de arranque de Windows indica "Windows no puede iniciar. Esto se puede deber a un cambio reciente en el hardware o software". Al desplazarse hacia abajo, verá el código de estado "0xC0000017", además de información que indica que "No se puede cargar la aplicación o el sistema operativo porque falta un archivo necesario o este contiene errores".](./media/troubleshoot-windows-stop-error/1.png)

   **Pantalla Recuperación**
 
   ![Pantalla Recuperación que indica que "El equipo o dispositivo necesita reparación. No hay memoria suficiente para crear el dispositivo Ramdisk". También debería ver el código de error "0xC0000017".](./media/troubleshoot-windows-stop-error/2.png)

## <a name="cause"></a>Causa

El disco del sistema operativo está lleno o demasiado fragmentado o el sistema operativo (SO) no puede acceder a la memoria o al archivo de paginación, o a ninguno de los dos.

## <a name="solution"></a>Solución

### <a name="process-overview"></a>Información general del proceso:

1. Creación de una VM de reparación y acceso a ella
1. Liberación de espacio en el disco
1. Limpieza de la memoria incorrecta del almacén BCD
1. Restauración del archivo de paginación en su ubicación predeterminada
1. Habilitación de la consola serie y recopilación de volcados de memoria
1. Recompilación de la máquina virtual

> [!NOTE]
> Si se produce este error, significa que el sistema operativo invitado no está operativo. Ahora solucionará el problema en el modo sin conexión.

### <a name="create-and-access-a-repair-vm"></a>Creación de una máquina virtual de reparación y acceso a ella

1. Siga los [pasos 1 a 3 de los comandos de reparación de VM](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) para preparar una VM de reparación.
1. Use Conexión a Escritorio remoto para conectarse a la VM de reparación.

### <a name="for-generation-2-vms-assign-a-letter-to-the-extensible-firmware-interface-efi-partition"></a>En el caso de las máquinas virtuales de generación 2, asigne una letra a la partición Extensible Firmware Interface (EFI):

Si utiliza una máquina virtual de generación 2, es posible que la partición EFI del disco conectado no tenga una letra asignada. Tendrá que seguir los pasos que se indican a continuación para asignar una letra a la partición antes de continuar con esta guía de solución de problemas.

1. En la búsqueda de Windows, escriba `diskmgmt` y abra la **consola de Administración de discos**.
1. Identifique el disco dañado conectado a la máquina virtual de reparación. Normalmente, este disco aparece en último lugar en la consola y tiene el valor numérico más alto.
1. Tenga en cuenta que, si en ese disco hay una partición que contiene la **partición de sistema EFI**, que tampoco tiene asignado un valor de letra (por ejemplo, unidad *F:* ). Si todas las particiones están asignadas, puede pasar directamente a liberar espacio en el disco. De lo contrario, continúe con la asignación de letra a este disco.

   ![La consola de Administración de discos, con el disco conectado "Disco 2", así como la partición sin asignar que es de 100 MB y es la "Partición de sistema EFI".](./media/troubleshoot-windows-stop-error/3.png)

1. Como administrador, abra un símbolo del sistema con privilegios elevados y escriba `diskpart` para iniciar la herramienta **DISKPART**.
1. Escriba los siguientes comandos:

   ```
   list disk
   sel disk <NUMBER OF THE ATTACHED DISK>
   list partition
   sel partition <NUMBER OF THE SYSTEM (EFI) PARTION>
   assign
   ```
   
   - En el comando, reemplace `<NUMBER OF THE ATTACHED DISK>` por el número del disco que identificó en el paso 2.
   - En el comando, reemplace `<NUMBER OF THE SYSTEM PARTION>` por el número de partición de la partición de sistema EFI. Todavía no se ha asignado una letra a esta partición, pero debe ser del tipo **Sistema** y tener un tamaño de aproximadamente 100 MB.

   > [!NOTE]
   > La comparación de las particiones en la consola de Administración de discos con las mostradas en la herramienta DISKPART puede resultar útil para determinar qué número de partición corresponde a la partición de sistema EFI del disco conectado.

1. Cierre la ventana del símbolo del sistema.

### <a name="free-up-space-on-the-disk"></a>Liberación de espacio en el disco

Ahora que el disco dañado está conectado a la máquina virtual de reparación, debe comprobar que el sistema operativo del disco tiene suficiente espacio para funcionar correctamente. 

1. Compruebe si el disco está lleno. Para ello, haga clic con el botón derecho en la unidad y seleccione **Propiedades**.
1. Si el disco tiene **menos de 300 MB de espacio libre**, [expándalo a un máximo de 1 TB con PowerShell](../windows/expand-os-disk.md).
1. Cuando el tamaño del disco sea **1 TB**, tendrá que realizar una limpieza de disco. Puede usar la [herramienta Liberador de espacio en disco](https://support.microsoft.com/help/4026616/windows-10-disk-cleanup) para liberar espacio.
1. Abra una instancia de símbolo del sistema con privilegios elevados (ejecutar como administrador) y realice una desfragmentación en la unidad:

   ``
   defrag <LETTER ASSIGNED TO THE OS DISK>: /u /x /g
   ``
   
   - En función del nivel de fragmentación, la desfragmentación puede tardar varias horas.
   - En el comando, reemplace `<LETTER ASSIGNED TO THE OS DISK>` por la letra del disco del sistema operativo (por ejemplo, unidad *F:* ).

### <a name="clean-out-bad-memory-from-the-boot-configuration-data-bcd-store"></a>Limpieza de la memoria incorrecta del almacén de datos de la configuración de arranque (BCD)

1. Abra un símbolo del sistema con privilegios elevados (ejecutar como administrador).
1. Consulte el archivo de configuración de arranque en busca de marcas de memoria incorrecta con el siguiente comando:

   ``
   bcdedit /store <LETTER ASSIGNED TO THE OS DISK>:\boot\bcd /enum {badmemory}
   ``
   
   - En el comando, reemplace `<LETTER ASSIGNED TO THE OS DISK>` por la letra del disco del sistema operativo (por ejemplo, unidad *F:* ).

1. Si la consulta no muestra bloques de memoria incorrecta, vaya a la siguiente tarea. De lo contrario, continúe con el paso 4.
1. Si se identifican bloques de memoria incorrecta, estos bloques impiden la creación de un disco RAM y deben eliminarse con el siguiente comando:

   ``
   bcdedit /store <LETTER ASSIGNED TO THE OS DISK>:\boot\bcd /deletevalue {badmemory} badmemorylist
   ``
   
   - En el comando, reemplace `<LETTER ASSIGNED TO THE OS DISK>` por la letra del disco del sistema operativo (por ejemplo, unidad *F:* ).

### <a name="restore-the-page-file-to-its-default-location"></a>Restauración del archivo de paginación en su ubicación predeterminada

El archivo de paginación almacena datos que no se pueden mantener en la memoria de acceso aleatorio (RAM) del equipo como una forma de desbordamiento o copia de seguridad. Es posible que el archivo esté hospedado en un disco duro virtual (VHD) y no en la unidad temporal, que es la ubicación predeterminada de Azure. De ser así, es posible que el archivo no sea accesible y haya que restaurarlo en la ubicación predeterminada.

Antes de realizar ningún paso, debe crear una copia de la carpeta **\windows\system32\config** en un disco correcto. Este paso garantiza que puede deshacer los cambios no deseados. Trabajará con archivos importantes del sistema, así que esta precaución es muy recomendable.

1. En la búsqueda de Windows, escriba **regedit** y abra la aplicación Editor del Registro.
1. En el Editor del Registro, resalte la clave **HKEY_LOCAL_MACHINE** y seleccione **Archivo > Cargar subárbol...** en el menú.

   ![Menú Cargar subárbol del Editor del Registro.](./media/troubleshoot-windows-stop-error/4.png)

1. En el cuadro de diálogo Cargar subárbol, seleccione **\windows\system32\config\SYSTEM** y haga clic en Abrir.
   1. Se le pedirá un nombre, y debe especificar **BROKENSYSTEM**. Este nombre le ayudará a diferenciar los subárboles afectados mientras está solucionando problemas.
   1. Expanda **HKEY_LOCAL_MACHINE** para ver la nueva clave BROKENSYSTEM que ha agregado.
1. Con el Editor del Registro, determine en qué ControlSet se va a arrancar el equipo.
   1. Vaya a **HKEY_LOCAL_MACHINE >> BROKENSYSTEM >> Select**.
   1. En las claves que aparecen, tenga en cuenta el valor de datos de Current. Por ejemplo, si este valor es **1** o **0x00000001 (1)** , el conjunto de controles sería ControlSet001.
1. Compruebe la ubicación en la que está configurada la creación del archivo de paginación.
   1. En HKEY_LOCAL_MACHINE \BROKENSYSTEM, expanda el directorio que coincida con el número de ControlSet que identificó en el paso 4, como **ControlSet001**.
   1. Vaya a **Control >> Administrador de sesiones Administración de memoria** y anote la ubicación de la clave de **ExistingPageFiles**.
   1. Esta clave debe estar en la ubicación predeterminada de Azure de la unidad temporal. Si no está ahí y se encuentra en un disco duro virtual en otra ubicación, como la unidad de disco de datos o la unidad del sistema operativo, habrá que eliminarla.
   1. Vaya a esa ubicación en el Explorador de archivos y elimine el archivo **pagefile.sys**.

### <a name="enable-the-serial-console-and-memory-dump-collection"></a>Habilitación de Serial console y de la recopilación del volcado de memoria

**Recomendado**: Antes de volver a compilar la VM, habilite Serial console y la recopilación del volcado de memoria mediante la ejecución del siguiente script:

Para habilitar la recopilación de volcado de memoria y Serial Console, ejecute el siguiente script:

1. Abra una sesión de símbolo del sistema con privilegios elevados como administrador.
1. Muestre los datos del almacén BCD y determine el identificador del cargador de arranque, que va a usar en el paso siguiente.

   1. En el caso de una máquina virtual de generación 1, escriba el siguiente comando y anote el identificador que se muestra:
   
      ``
      bcdedit /store <BOOT PARTITON>:\boot\bcd /enum
      ``
   
   - En el comando, reemplace `<BOOT PARTITON>` por la letra de la partición del disco conectado que contiene la carpeta de arranque.

      ![Salida de la lista del almacén BCD en una máquina virtual de generación 1, que muestra en el cargador de arranque de Windows el número de identificación.](./media/troubleshoot-windows-stop-error/5.png)

   1. En el caso de una máquina virtual de generación 2, escriba el siguiente comando y anote el identificador que se muestra:
   
      ``
      bcdedit /store <LETTER OF THE EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /enum
      ``
   
   - En el comando, reemplace `<LETTER OF THE EFI SYSTEM PARTITION>` por la letra de la partición de sistema EFI.
   - Puede resultar útil iniciar la consola de Administración de discos para identificar la partición del sistema adecuada etiquetada como **Partición de sistema EFI**.
   - El identificador puede ser un GUID único o puede ser el valor predeterminado **bootmgr**.

1. Ejecute los siguientes comandos para habilitar Serial Console:

   ```
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON 
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
   ```
   
   - En el comando, reemplace `<VOLUME LETTER WHERE THE BCD FOLDER IS>` por la letra de la carpeta BCD.
   - En el comando, reemplace `<BOOT LOADER IDENTIFIER>` por el identificador que encontró en el paso anterior.

1. Compruebe que el espacio disponible en el disco del sistema operativo sea mayor que el tamaño de memoria (RAM) de la VM.

   Si no hay suficiente espacio en el disco del sistema operativo, cambie la ubicación donde se creará el archivo de volcado de memoria y remita dicha ubicación a cualquier disco de datos conectado a la VM que tenga espacio libre suficiente. Para cambiar la ubicación, reemplace **%SystemRoot%** por la letra de unidad del disco de datos (por ejemplo, unidad **F:** ) en los comandos siguientes.

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
