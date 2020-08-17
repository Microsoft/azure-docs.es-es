---
title: 'Solución del error de detención de Windows: error de inicialización del servicio de directorio'
description: Resuelva los problemas en que una máquina virtual (VM) de controlador de dominio de Active Directory se queda atascada en un bucle que indica que debe reiniciarse.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 3396f8fe-7573-4a15-a95d-a1e104c6b76d
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 05/05/2020
ms.author: v-miegge
ms.openlocfilehash: 909481964f8aa3272715e235fa011562225a9422
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87028369"
---
# <a name="troubleshoot-windows-stop-error--directory-service-initialization-failure"></a>Solución del error de detención de Windows: error de inicialización del servicio de directorio

En este artículo se incluyen los pasos para resolver los problemas en que una máquina virtual (VM) de controlador de dominio de Active Directory se queda atascada en un bucle e indica que debe reiniciarse.

## <a name="symptom"></a>Síntoma

Si usa [Diagnósticos de arranque](./boot-diagnostics.md) para ver la captura de pantalla de la VM, en la captura de pantalla se muestra que la VM debe reiniciarse debido a un error, con el código de detención **0xC00002E1** en Windows Server 2008 R2 o **0xC00002E2** en Windows Server 2012 o posterior.

![La pantalla de inicio de Windows Server 2012 indica "Se ha producido un problema en su PC y necesita reiniciarse. Vamos a recopilar información sobre el error y después se reiniciará automáticamente".](./media/troubleshoot-directory-service-initialization-failure/1.png)

## <a name="cause"></a>Causa

El código de error **0xC00002E2** representa **STATUS_DS_INIT_FAILURE** y el código de error **0xC00002E1** representa **STATUS_DS_CANT_START**. Ambos errores se producen cuando hay un problema con el servicio de directorio.

A medida que arranque el sistema operativo, el servidor de autenticación de seguridad local forzará el reinicio automático (**LSASS.exe**), que autentica los inicios de sesión de usuario. La autenticación no puede producirse si el sistema operativo de la VM es un controlador de dominio que no tiene acceso de lectura y escritura a su base de datos de Active Directory local. Debido a una falta de acceso a **Active Directory (AD)** , no se puede autenticar LSASS.exe y se fuerza el reinicio del sistema operativo.

Este error puede deberse a cualquiera de las siguientes condiciones:

- No hay acceso al disco que contiene la base de datos de AD local (**NTDS.DIT**).
- El disco que contiene la base de datos de AD local (NTDS.DIT) se ha quedado sin espacio disponible.
- Falta la base de datos de AD local (NTDS.DIT).
- La VM tiene varios discos y la directiva de red de área de almacenamiento (SAN) está configurada de forma incorrecta. La directiva de SAN no está establecida en **ONLINEALL** y los discos que no son de sistema operativo están conectados en modo sin conexión en el administrador de discos.
- La base de datos de AD local (NTDS.DIT) está dañada.

## <a name="solution"></a>Solución

### <a name="process-overview"></a>Información general del proceso:

1. Cree una VM de reparación y acceda a ella.
1. Libere espacio en el disco.
1. Compruebe que la unidad que contiene la base de datos de AD esté conectada.
1. Habilite el modo de restauración de servicios de directorio.
1. **Recomendado**: Antes de volver a compilar la VM, habilite Serial Console y la recopilación de volcado de memoria.
1. Vuelva a compilar la VM.
1. Vuelva a configurar la directiva de SAN.

> [!NOTE]
> Si se produce este error, significa que el sistema operativo invitado no está operativo. Solucionará el problema en el modo sin conexión.

### <a name="create-and-access-a-repair-vm"></a>Creación de una máquina virtual de reparación y acceso a ella

1. Siga los [pasos 1 a 3 de los comandos de reparación de VM](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) para preparar una VM de reparación.
1. Use Conexión a Escritorio remoto para conectarse a la VM de reparación.

### <a name="free-up-space-on-disk"></a>Liberación de espacio en el disco

Dado que el disco ahora está conectado a una VM de reparación, compruebe que el disco que contiene la base de datos interna de Active Directory tenga suficiente espacio para ejecutarse correctamente.

1. Compruebe si el disco está lleno. Para ello, haga clic con el botón derecho en la unidad y seleccione **Propiedades**.
1. Si el disco tiene menos de 300 Mb de espacio libre, [expándalo a un máximo de 1 TB con PowerShell](../windows/expand-os-disk.md).
1. Si el disco ha alcanzado 1 Tb de espacio usado, ejecute una limpieza de disco.

   1. Use PowerShell para [desconectar el disco de datos](../windows/detach-disk.md#detach-a-data-disk-using-powershell) de la VM rota.
   1. Una vez desconectado de la VM rota, [conecte el disco de datos](../windows/attach-disk-ps.md#attach-an-existing-data-disk-to-a-vm) a una VM en funcionamiento.
   1. Utilice la [herramienta Liberar espacio en disco](https://support.microsoft.com/help/4026616/windows-10-disk-cleanup) para liberar más espacio.

1. **Opcional**: si se necesita más espacio, abra una instancia de CMD y escriba el comando `defrag <LETTER ASSIGNED TO THE OS DISK>: /u /x /g` para realizar una desfragmentación en la unidad:

  * En el comando, reemplace `<LETTER ASSIGNED TO THE OS DISK>` por la letra del disco del sistema operativo. Por ejemplo, si la letra del disco es `F:`, el comando debe ser `defrag F: /u /x /g`.

  * En función del nivel de fragmentación, la desfragmentación puede tardar varias horas.

Si hay espacio suficiente en el disco, continúe con la siguiente tarea.

### <a name="check-that-the-drive-containing-the-active-directory-database-is-attached"></a>Compruebe que la unidad que contiene la base de datos de Active Directory esté conectada.

1. Abra una instancia de CMD con privilegios elevados y ejecute los siguientes comandos:

   1. Cargue el archivo del Registro:

      `REG LOAD HKLM\BROKENSYSTEM f:\windows\system32\config\SYSTEM`

      La designación `f:` supone que el disco es la unidad `F:`. Use la letra de unidad de la unidad que contiene el disco del sistema operativo.

   1. Determine la letra de unidad y la carpeta de **NTDS.DIT**:

      ```
      REG QUERY "HKLM\BROKENSYSTEM\ControlSet001\Services\NTDS\parameters" /v "DSA Working Directory"
      REG QUERY "HKLM\BROKENSYSTEM\ControlSet001\Services\NTDS\parameters" /v "DSA Database file"
      REG QUERY "HKLM\BROKENSYSTEM\ControlSet001\Services\NTDS\parameters" /v "Database backup path"
      REG QUERY "HKLM\BROKENSYSTEM\ControlSet001\Services\NTDS\parameters" /v "Database log files path"
      ```

   1. Descargue el archivo de Registro:

      `REG UNLOAD HKLM\BROKENSYSTEM`

1. Use Azure Portal para comprobar que la unidad donde está configurado NTDS.DIT se ha agregado a la VM.
1. Mediante la consola de Administración de discos del sistema operativo invitado, compruebe que el disco que contiene NTDS.DIT esté en línea.
   1. La herramienta Administración de discos se puede encontrar en **Herramientas administrativas > Administración de equipos > Almacenamiento**, o bien se puede acceder a esta mediante el comando `diskmgmt.msc` en una instancia de CMD.
1. Si el disco no está conectado a la VM, vuelva a conectar el disco de datos para corregir el problema.

   Si el disco se conectó con normalidad, continúe con la siguiente tarea.

### <a name="enable-directory-services-restore-mode"></a>Habilitación del modo de restauración de servicios de directorio

Configure la VM para que arranque en **Modo de restauración de servicios de directorio (DSRM)** para omitir la comprobación de la existencia del archivo NTDS.DIT durante el arranque.

1. Antes de continuar, compruebe que ha completado las tareas anteriores para conectar el disco a una VM de reparación y que ha determinado el disco donde se encuentra el archivo NTDS.DIT.
1. Con una instancia de CMD con privilegios elevados, enumere los datos de la partición de arranque en ese almacén para encontrar el identificador de la partición activa:

   `bcdedit /store <Drive Letter>:\boot\bcd /enum`

   Reemplace `< Drive Letter >` por la letra determinada en los pasos anteriores.

   ![En la captura de pantalla se muestra una instancia de CMD con privilegios elevados después de escribir el comando "bcdedit /store <Drive Letter>:\boot\bcd /enum", que muestra la Administración de arranque de Windows con el identificador.](./media/troubleshoot-directory-service-initialization-failure/2.png)

1. Habilite la marca `safeboot DsRepair` en la partición de arranque:

   `bcdedit /store <Drive Letter>:\boot\bcd /set {<Identifier>} safeboot dsrepair`

   Reemplace `< Drive Letter >` y `< Identifier >` por los valores determinados en los pasos anteriores.

1. Vuelva a consultar las opciones de arranque para asegurarse de que el cambio se estableció correctamente.

   ![En la captura de pantalla se muestra una instancia de CMD con privilegios elevados después de habilitar la marca safeboot DsRepair.](./media/troubleshoot-directory-service-initialization-failure/3.png)

### <a name="recommended-before-you-rebuild-the-vm-enable-serial-console-and-memory-dump-collection"></a>Recomendado: Antes de volver a compilar la VM, habilite Serial Console y la recopilación del volcado de memoria.

Para habilitar la recopilación del volcado de memoria y Serial console, ejecute el siguiente script mediante la apertura de una sesión de símbolo del sistema con privilegios elevados (Ejecutar como administrador) y ejecute los siguientes comandos.

1. Habilitación de Serial console:

  ```
  bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
  bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
  ```

1. Compruebe que el espacio disponible en el disco del sistema operativo sea igual al tamaño de memoria (RAM) de la VM.

  1. Si no hay suficiente espacio en el disco del sistema operativo, cambie la ubicación donde se creará el archivo de volcado de memoria y remítala a los discos de datos conectados a la VM que tengan espacio libre suficiente.

     Para cambiar la ubicación, reemplace `%SystemRoot%` por la letra de unidad (por ejemplo, `F:`) del disco de datos en los comandos siguientes.

  #### <a name="the-following-configuration-is-suggested-to-enable-os-dump"></a>Se sugiere la siguiente configuración para habilitar el volcado del sistema operativo:

  **Cargar disco del sistema operativo roto**:

  `REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM`

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

  `REG UNLOAD HKLM\BROKENSYSTEM`

### <a name="rebuild-the-vm"></a>Recompilación de la máquina virtual

1. Siga el [paso 5 de los comandos de reparación de la VM](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) para volver a ensamblar la VM.

### <a name="reconfigure-the-storage-area-network-policy"></a>Reconfiguración de la directiva de red de área de almacenamiento

1. Al arrancar en modo DSRM, el único usuario disponible para iniciar sesión es el administrador de recuperación, que se usó cuando se promovió la VM a un controlador de dominio. Todos los demás usuarios mostrarán un error de autenticación.

   1. Si no hay ningún otro controlador de dominio disponible, debe iniciar sesión localmente con `.\administrator` o `machinename\administrator` y la contraseña de DSRM.

1. Configure la directiva de SAN para que todos los discos estén en línea.

   1. Abra una instancia CMD con privilegios elevados y escriba `DISKPART`.
   1. Consulte la lista de discos.

      `DISKPART> list disk`

   1. Escriba los siguientes comandos para seleccionar el disco que debe conectarse y cambie la directiva de SAN:

      ```
      DISKPART> select disk 1
      Disk 1 is now the selected disk.

      DISKPART> attributes disk clear readonly
      Disk attributes cleared successfully.

      DISKPART> attributes disk
      Current Read-only State : No
      Read-only : No
      Boot Disk : No
      Pagefile Disk : No
      Hibernation File Disk : No
      Crashdump Disk : No
      Clustered Disk : No

      DISKPART> online disk
      DiskPart successfully onlined the selected disk.

      DISKPART> san
      SAN Policy : Online All
      ```

1. Una vez corregido el problema, asegúrese de que se ha quitado la marca `DsRepair safeboot`:

   `bcdedit /deletevalue {default} safeboot dsrepair`

1. Reinicie la máquina virtual.

   > [!NOTE]
   > Si la VM se acaba de migrar desde un entorno local y desea migrar más controladores de dominio del entorno local a Azure, debe tener en cuenta los pasos descritos en el artículo siguiente para evitar que este problema se produzca en migraciones futuras:
   >
   > [Cómo cargar los controladores de dominio de Hyper-V locales existentes a Azure con Azure PowerShell](https://support.microsoft.com/help/2904015)
