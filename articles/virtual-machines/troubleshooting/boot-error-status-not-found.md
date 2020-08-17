---
title: 'Solución del error de Administración de arranque de Windows: 0xC0000225 "Estado no encontrado"'
description: Pasos para resolver los problemas en los que se produce el código de error 0xC0000225 en una VM de Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: c17899a4-b270-4725-9530-0dcd829b178c
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 05/11/2020
ms.author: v-miegge
ms.openlocfilehash: 3677d67f55cfccdc80245b2ec870ffa76b0a1940
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088672"
---
# <a name="troubleshoot-windows-boot-manager-error----0xc0000225-status-not-found"></a>Solución del error de Administración de arranque de Windows: 0xC0000225 "Estado no encontrado"
 
En este artículo se proporcionan los pasos para resolver problemas en los que se produce el código de error 0xC0000225 en una VM de Azure. Este error indica que no se encuentra el estado o el objeto.

## <a name="symptoms"></a>Síntomas

Si usa [Diagnósticos de arranque](./boot-diagnostics.md) para ver la captura de pantalla de la VM, verá que se muestra un error que indica que Windows no puede iniciar con el código *0xc0000225*.

El archivo asociado a este código de error le informará de los pasos que se deben realizar para resolver el problema. Busque el texto de la sección **File:** para determinar el curso de acción adecuado.

### <a name="drivers-os-related-or-third-party"></a>Controladores, relacionados con el sistema operativo o de terceros

Si el archivo está presente pero hace referencia a un controlador (como se muestra), está relacionado con el sistema operativo o es de terceros, siga los pasos descritos en la sección [Reparación del archivo del sistema](#repair-the-system-file).
 
En la siguiente imagen, Administración de arranque de Windows indica "Windows no puede iniciar. Esto se puede deber a un cambio reciente en el hardware o software". La imagen también muestra Status como "0xc0000225", **File:** como `\windows\System32\drivers\atapi.sys` e **Info:** como "No se pudo cargar el sistema operativo porque no se encuentra un controlador de sistema crítico o está dañado".

![Administración de arranque de Windows indica "Windows no puede iniciar. Esto se puede deber a un cambio reciente en el hardware o software". También muestra Status como "0xc0000225", File como "\windows\System32\drivers\atapi.sys" y, en la sección Info, indica: "No se pudo cargar el sistema operativo porque no se encuentra un controlador de sistema crítico o está dañado".](./media/troubleshoot-boot-error-status-not-found/1.png)

### <a name="no-file"></a>No hay archivo

Si se presenta el código de estado, pero no se muestra ningún archivo, siga los pasos que se describen en la sección [Adición de la variable OSDEVICE](#add-the-osdevice-variable).

En la siguiente imagen, Administración de arranque de Windows indica "Windows no puede iniciar. Esto se puede deber a un cambio reciente en el hardware o software". La imagen también muestra Status como "0xc0000225" e **Info:** como "Error al seleccionar el arranque; no se puede tener acceso a un dispositivo requerido".

![Administración de arranque de Windows indica "Windows no puede iniciar. Esto se puede deber a un cambio reciente en el hardware o software". También muestra Status como "0xc0000225" y en la sección Info indica: "Error al seleccionar el arranque; no se puede tener acceso a un dispositivo requerido".](./media/troubleshoot-boot-error-status-not-found/2.png)

### <a name="registry-file"></a>Archivo del Registro

Si hace referencia a cualquiera de los archivos del Registro, como \windows\system32\config\system, siga los pasos que se describen en la sección [Creación de una incidencia de soporte técnico](#contact-support).
 
En la siguiente imagen, Administración de arranque de Windows indica "Windows no puede iniciar. Esto se puede deber a un cambio reciente en el hardware o software". La imagen también muestra Status como "0xc0000225", File como `\windows\System32\config\system` e **Info:** como "No se pudo cargar el sistema operativo porque el archivo de Registro del sistema no se encuentra o está dañado".

![Administración de arranque de Windows indica "Windows no puede iniciar. Esto se puede deber a un cambio reciente en el hardware o software". También muestra Status como "0xc0000225", File como "\windows\System32\config\system" y, en la sección Info, indica: "No se pudo cargar el sistema operativo porque el archivo de Registro del sistema no se encuentra o está dañado".](./media/troubleshoot-boot-error-status-not-found/3.png)

En la imagen siguiente, la pantalla de recuperación indica "El equipo o dispositivo necesita reparación. No se pudo cargar el sistema operativo porque el archivo de Registro del sistema no se encuentra o está dañado". La imagen también muestra el código de error como "0xc0000225" y File como `\windows\System32\config\system`.

![La pantalla de recuperación indica "El equipo o dispositivo necesita reparación. No se pudo cargar el sistema operativo porque el archivo de Registro del sistema no se encuentra o está dañado". También se muestra el código de error "0xc0000225" y el archivo "\windows\System32\config\system".](./media/troubleshoot-boot-error-status-not-found/4.png)

## <a name="causes"></a>Causas

### <a name="missing-binary"></a>Binario ausente

Es posible que se encuentre un binario ausente o dañado en el archivo del sistema **(.sys)** .

### <a name="bcd-corruption-or-improper-vhd-migration"></a>Daños en BCD o migración de VHD incorrecta

En este caso, los **datos de la configuración de arranque (BCD)** están dañados o la **unidad de disco duro virtual (VHD)** se migró del entorno local, pero no se preparó correctamente. El resultado es que falta la variable **OSDEVICE** y que se deberá agregar.

### <a name="registry-hive-corruption"></a>Daños en el subárbol del Registro

Un subárbol del Registro dañado puede deberse a:

- Un error del subárbol.
- El subárbol se monta, pero está vacío.
- El subárbol no se cerró correctamente.
## <a name="solution"></a>Solución

### <a name="process-overview"></a>Información general del proceso

1. Cree una VM de reparación y acceda a esta.
1. Seleccione una solución:
   - [Reparación del archivo del sistema](#repair-the-system-file)
   - [Adición de la variable OSDevice](#add-the-osdevice-variable)
   - [Creación de una incidencia de soporte técnico](#contact-support)
1. Habilite Serial console y la recopilación del volcado de memoria.
1. Vuelva a compilar la VM.

### <a name="create-and-access-a-repair-vm"></a>Creación de una VM de reparación y acceso a ella

1. Siga los pasos 1 a 3 de los [comandos de reparación de VM](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) para preparar una VM de reparación.
1. Use Conexión a Escritorio remoto para conectarse a la VM de reparación.

### <a name="select-a-solution"></a>Selección de una solución

1. Abra un símbolo del sistema con privilegios elevados.
1. Según el síntoma que haya identificado anteriormente, siga los pasos de la solución correspondiente. Puede omitir los pasos de las otras soluciones, ya que no se aplicarán a su problema:

- [Reparación del archivo del sistema](#repair-the-system-file)
- [Adición de la variable OSDevice](#add-the-osdevice-variable)
- [Creación de una incidencia de soporte técnico](#contact-support)

### <a name="repair-the-system-file"></a>Reparación del archivo del sistema

1. Con el disco duro virtual conectado, navegue hasta la ubicación de archivo del binario que se muestra en la captura de pantalla de la máquina virtual (VM).
1. Haga clic con el botón derecho en el archivo, seleccione **Propiedades** y, a continuación, seleccione la pestaña **Detalles** para ver información sobre el archivo.
   1. Anote la versión del archivo, tal como se muestra en la imagen siguiente:

      ![La ventana Propiedades del archivo "cng.sys", con la versión de archivo resaltada.](./media/troubleshoot-boot-error-status-not-found/5.png)

1. Cambie el nombre del archivo a **< BINARY.SYS >.old** y reemplace **< BINARY.SYS >** por el nombre del archivo.

   Para la imagen del paso anterior, el nombre del archivo **cng.sys** cambiaría a **cng.sys.old**.

   > [!NOTE]
   > Si intenta cambiar el nombre del archivo y recibe el mensaje "El archivo está dañado o es ilegible", [póngase en contacto con el soporte técnico para obtener asistencia](https://azure.microsoft.com/support/create-ticket/), ya que esta solución no funcionará.

1. Ahora que se ha cambiado el nombre del archivo dañado, restáurelo desde el repositorio interno para corregirlo.
   1. Inicie una sesión de **CMD**.
   1. Vaya a **\windows\winsxs**.

   1. Busque el archivo binario del principio de esta sección con el siguiente comando:

      `dir <BINARY WITH ".SYS" EXTENSION>  /s`

      Este comando enumera todas las versiones del archivo que tiene la máquina, lo que le proporciona el historial de rutas de acceso de ese componente.
      
      Por ejemplo, el nombre del archivo **dir cng.sys** cambiaría a **dir cng.sys /s**

   1. Elija la versión más reciente del archivo en la lista (o cualquiera que prefiera) y copie el archivo en la carpeta **windows\system32** con la ruta de acceso anterior y el comando siguiente:

      `copy <drive>:\Windows\WinSxS\<DIRECTORY WHERE FILE IS>\<BINARY WITH ".SYS" EXTENSION> <DRIVE>:\Windows\System32\Drivers\`

      > [!NOTE]
      > Si el archivo binario más reciente no funcionó, pruebe una versión anterior a esa, o cualquier otra que sepa que es estable, como una versión anterior a una revisión.

      Por ejemplo, si el binario que busca es **cmimcext.sys**, la unidad defectuosa es la unidad **F:** y acaba de ejecutar una búsqueda de la versión más reciente, verá la siguiente imagen, donde una consulta en el símbolo del sistema de `dir cmim* /s` busca la versión más reciente del archivo cmimcext.sys.

      ![Consulta en el símbolo del sistema de "dir cmim* /s" para buscar la versión más reciente del archivo cmimcext.sys.](./media/troubleshoot-boot-error-status-not-found/6.png)

      En la imagen de ejemplo anterior, la consulta se realizó en **C:** , mientras que la letra de unidad debería ser la de la unidad defectuosa, **F:** , que es el disco del sistema operativo conectado como disco de datos en la VM de reparación.

      El comando resultante para copiar el archivo sería: `copy F:\Windows\WinSxS\amd64_xxxxxx\cmimcext.sys F:\Windows\System32\Drivers`.

Una vez completada esta tarea, pase a [Habilitación de Serial console y de la recopilación del volcado de memoria](#enable-the-serial-console-and-memory-dump-collection).

### <a name="add-the-osdevice-variable"></a>Adición de la variable OSDEVICE

Recopile la información de configuración de arranque actual y anote el identificador en la partición activa. Posteriormente, usará esta información para agregar la variable **OSDEVICE**, siguiendo las instrucciones para la generación de la VM.

Si esta recopilación de información genera un error que indica que no hay ningún archivo **\boot\bcd**, siga las instrucciones que se indican en [Reparación del archivo del sistema](#repair-the-system-file) en su lugar. 

1. Para las VM de la generación 1, abra un símbolo del sistema con privilegios elevados como administrador y ejecute el siguiente comando:

   `bcdedit /store <LETTER OF BOOT FOLDER>:\boot\bcd /enum`

   Esta imagen muestra el cargador de arranque de Windows en una VM de la generación 1, con el atributo de identificador resaltado. El atributo identificador resaltado muestra una cadena alfanumérica única.

   ![Cargador de arranque de Windows en una VM de la generación 1 con el atributo identificador resaltado. El atributo identificador resaltado muestra una cadena alfanumérica única.](./media/troubleshoot-boot-error-status-not-found/7.png)

   Anote el identificador del cargador de arranque de Windows, cuya ruta de acceso es **\windows\system32\winload.exe**.

1. En el caso de las VM de la generación 2, compruebe que el disco del sistema operativo está en línea y que se han asignado las letras de unidad de la partición. Cuando lo haya comprobado, recopile la información de configuración de arranque.
   1. En **Windows Search**, escriba **Administración de discos** y abra la consola de Administración de discos. Use esta consola para identificar el número de disco conectado a la VM de reparación y la partición de Extensible Firmware Interface (EFI) que contiene el almacén BCD.

   En la imagen siguiente, el disco 2 es el número de disco conectado a la VM de reparación. La imagen también muestra la partición del sistema EFI en el disco 2, que tiene un tamaño de 100 MB y no tiene una letra asignada.

   ![El disco 2 se muestra como el número de disco conectado a la VM de reparación. También muestra la partición del sistema EFI en el disco 2, que es de 100 MB y no tiene una letra asignada.](./media/troubleshoot-boot-error-status-not-found/8.png)

   1. Abra un símbolo del sistema con privilegios elevados como administrador y ejecute los siguientes comandos:
      1. Abra la **HERRAMIENTA DISKPART** mediante el comando `diskpart`.
      1. Enumere todos los discos y, a continuación, seleccione el disco conectado identificado en el paso anterior:
      
         ```
         list disk
         sel disk <DISK #>
         ```

         La siguiente imagen muestra los resultados de enumeración y selección de un disco. Se enumeran el disco 0 (127 GB / en línea), el disco 1 (32 GB / en línea) y el disco 2 (127 GB / en línea), y el disco 2 se selecciona mediante el comando `sel disk 2`.

         ![Resultados de enumerar y, a continuación, seleccionar un disco. Se enumeran el disco 0 (127 GB | en línea), el disco 1 (32 GB | en línea) y el disco 2 (127 GB | en línea), y el disco 2 se selecciona.](./media/troubleshoot-boot-error-status-not-found/9.png)

      1. Enumere las particiones y seleccione la partición del sistema EFI identificada en el paso anterior:
      
         ```
         list partition
         sel partition <PARTITION #>
         ```

         En la imagen siguiente se muestran los resultados de la lista y la selección de una partición. Se enumeran la partición 1 (reservada / 16 MB), la partición 2 (sistema / 100 MB) y la partición 3 (primaria / 126 GB), y se selecciona la partición 2 con el comando `sel part 2`.

         ![Resultados de enumerar y, a continuación, seleccionar una partición. Se enumeran la partición 1 (reservada | 16 MB), la partición 2 (sistema | 100 MB) y la partición 3 (primaria | 126 GB), y se selecciona la partición 2.](./media/troubleshoot-boot-error-status-not-found/10.png)

      1. Asigne una letra a la partición EFI mediante el comando `assign`.

         En la imagen siguiente, el comando `assign` y la nueva unidad **SYSTEM (F:)** se pueden ver en el Explorador de archivos.

         ![El comando assign y la nueva unidad SYSTEM (F:) se pueden ver en el Explorador de archivos.](./media/troubleshoot-boot-error-status-not-found/11.png)

      1. Enumere los datos del almacén BCD con el siguiente comando:
      
         `bcdedit /store <LETTER OF EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /enum`

         En la imagen siguiente, el cargador de arranque de Windows se encuentra en una VM de la generación 2 con el atributo identificador resaltado. El atributo identificador resaltado tiene un valor de **{default}** .

         ![El cargador de arranque de Windows se muestra en una VM de la generación 2 con el atributo identificador resaltado. El atributo identificador resaltado muestra el valor default.](./media/troubleshoot-boot-error-status-not-found/12.png)

         Anote el identificador en el cargador de arranque de Windows, cuya ruta de acceso es **\windows\system32\winload.efi**.

1. Observe que falta la variable OSDEVICE en la partición activa:

   ![Los atributos de Administración de arranque de Windows y del cargador de arranque de Windows se enumeran en el símbolo del sistema, pero falta el atributo de dispositivo del sistema operativo.](./media/troubleshoot-boot-error-status-not-found/13.png)
   
   En esta imagen, los atributos de Administración de arranque de Windows y del cargador de arranque de Windows se enumeran en el símbolo del sistema, pero falta el atributo OSDEVICE.

1. Agregue la variable OSDEVICE en función de la siguiente información:

   En el caso de los discos del sistema operativo de una sola partición, agregue `BOOT`.

   > [!NOTE]
   > La carpeta de arranque se encontrará en la misma partición que la carpeta de Windows (**carpeta \windows**).
   > - La carpeta de arranque para las VM de generación 1 es **carpeta \boot\bcd**.
   > - La carpeta de arranque para las VM de generación 2 es **EFI\Microsoft\boot\bcd**.

   En el caso de las VM de generación 1, escriba el siguiente comando:

   `bcdedit /store <LETTER OF BOOT FOLDER>:\boot\bcd /set {<IDENTIFIER>} OSDEVICE BOOT`

   En el caso de las VM de generación 2, escriba el siguiente comando:

   `bcdedit /store <LETTER OF EFI FOLDER>:EFI\Microsoft\boot\bcd /set {<IDENTIFIER>} OSDEVICE BOOT`

   En el caso de discos del sistema operativo de varias particiones, agregue `PARTITION=<LETTER OF WINDOWS FOLDER>:`.

   > [!NOTE]
   > Lo más probable es que la carpeta de arranque esté en una partición diferente que la carpeta de Windows (**carpeta \windows**).
   > - La carpeta de arranque para las VM de generación 1 es **carpeta \boot\bcd**.
   > - La carpeta de arranque para las VM de generación 2 es **EFI\Microsoft\boot\bcd**.

   En el caso de las VM de generación 1, escriba el siguiente comando:

   `bcdedit /store <LETTER OF BOOT FOLDER>:\boot\bcd /set {<IDENTIFIER>} OSDEVICE partition=<LETTER OF WINDOWS FOLDER>:`

   En el caso de las VM de generación 2, escriba el siguiente comando:

   `bcdedit /store <LETTER OF EFI FOLDER>:EFI\Microsoft\boot\bcd /set {< IDENTIFIER>} OSDEVICE partition=<LETTER OF WINDOWS FOLDER>:`

Una vez completada esta tarea, pase a [Habilitación de Serial console y de la recopilación del volcado de memoria](#enable-the-serial-console-and-memory-dump-collection).

### <a name="contact-support"></a>Ponerse en contacto con el servicio de soporte técnico

El error **Archivo del Registro** tiene una solución, pero tendrá que [crear una incidencia de soporte técnico](https://azure.microsoft.com/support/create-ticket/) para obtener más ayuda.

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
