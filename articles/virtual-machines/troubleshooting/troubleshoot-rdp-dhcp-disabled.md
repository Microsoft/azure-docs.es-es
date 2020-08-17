---
title: No se puede establecer conexión con Azure Virtual Machines porque DHCP está deshabilitado | Microsoft Docs
description: Aprenda a solucionar problemas de RDP que se producen cuando el servicio cliente DHCP está deshabilitado en Microsoft Azure.| Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 11/13/2018
ms.author: genli
ms.openlocfilehash: 945f8896a844e7a73107df44d03abc7290f4e3fc
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86999145"
---
#  <a name="cannot-rdp-to-azure-virtual-machines-because-the-dhcp-client-service-is-disabled"></a>No se puede utilizar RDP en Azure Virtual Machines porque el servicio cliente DHCP está deshabilitado

En este artículo, se describe un problema que impide utilizar Escritorio remoto con Azure Windows Virtual Machines (VM) cuando el servicio cliente DHCP está deshabilitado en la máquina virtual.


## <a name="symptoms"></a>Síntomas
No se puede establecer conexión mediante RDP con una máquina virtual de Azure porque el servicio cliente DHCP está deshabilitado en la máquina virtual. Al comprobar la captura de pantalla de los [Diagnósticos de arranque](../troubleshooting/boot-diagnostics.md) en Azure portal, puede ver que la máquina virtual se inicia de manera normal y espera las credenciales en la pantalla de inicio de sesión. Puede ver de forma remota los registros de eventos de la máquina virtual con el Visor de eventos. Verá que el servicio cliente DHCP no se inicia correctamente. A continuación, se muestra un registro de ejemplo:

**Nombre de registro**: Sistema </br>
**Origen**: Administrador de control de servicios </br>
**Fecha**: 16/12/2015 11:19:36 a. m. </br>
**Id. de evento**: 7022 </br>
**Categoría de tarea**: None </br>
**Nivel**: Error </br>
**Palabras clave**: Clásico</br>
**Usuario**: N/D </br>
**Equipo**: myvm.cosotos.com</br>
**Descripción**: el servicio cliente DHCP no respondió después de iniciar.</br>

En las máquinas virtuales de Resource Manager, pude utilizar la consola de acceso de serie para consultar los registros de eventos 7022 con el comando siguiente:

```console
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Service Control Manager'] and EventID=7022 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
```

En las máquinas virtuales clásicas, es necesario trabajar en modo SIN CONEXIÓN y recopilar los registros manualmente.

## <a name="cause"></a>Causa

El servicio cliente DHCP no se está ejecutando en la máquina virtual.

> [!NOTE]
> Este artículo solamente es aplicable al servicio cliente DHCP, no al servidor DHCP.

## <a name="solution"></a>Solución

Antes de seguir estos pasos, tome una instantánea del disco del sistema operativo de la máquina virtual afectada como copia de seguridad. Para obtener más información, consulte [Instantánea de un disco](../windows/snapshot-copy-managed-disk.md).

Para resolver este problema, use el control de serie a fin de habilitar DHCP o [restablecer la interfaz de red](reset-network-interface.md) en la máquina virtual.

### <a name="use-serial-control"></a>Uso del control serie

1. Conéctese a una [consola serie y abra la instancia de CMD](serial-console-windows.md#use-cmd-or-powershell-in-serial-console).
). Si la consola serie no está habilitada en la máquina virtual, consulte [Restablecimiento de la interfaz de red](reset-network-interface.md).
2. Compruebe si el protocolo DHCP está deshabilitado en la interfaz de red:

    ```console
    sc query DHCP
    ```

3. Si DHCP se ha detenido, intente iniciar el servicio:

    ```console
    sc start DHCP
    ```

4. Envíe una consulta el servicio de nuevo para asegurarse de que el servicio se ha iniciado correctamente.

    ```console
    sc query DHCP
    ```

    Intente conectarse a la máquina virtual y compruebe si el problema se ha resuelto.
5. Si el servicio no se inicia, elija a continuación la solución que corresponda en función del mensaje de error que recibió:

    | Error  |  Solución |
    |---|---|
    | 5- ACCESS DENIED  | Consulte [El servicio cliente DHCP se ha detenido debido a un error de denegación de acceso](#dhcp-client-service-is-stopped-because-of-an-access-denied-error).  |
    |1053 - ERROR_SERVICE_REQUEST_TIMEOUT   | Consulte [El servicio cliente DHCP se bloquea o deja de responder](#dhcp-client-service-crashes-or-hangs).  |
    | 1058 - ERROR_SERVICE_DISABLED  | Consulte [El servicio cliente DHCP no se está ejecutando](#dhcp-client-service-is-disabled).  |
    | 1059 - ERROR_CIRCULAR_DEPENDENCY  |[Póngase en contacto con el equipo de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.   |
    | 1067 - ERROR_PROCESS_ABORTED |Consulte [El servicio cliente DHCP se bloquea o deja de responder](#dhcp-client-service-crashes-or-hangs).   |
    |1068 - ERROR_SERVICE_DEPENDENCY_FAIL   | [Póngase en contacto con el equipo de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.  |
    |1069 - ERROR_SERVICE_LOGON_FAILED   |  Consulte [Se produce un error de inicio de sesión en el servicio cliente DHCP](#dhcp-client-service-fails-because-of-logon-failure) |
    | 1070 - ERROR_SERVICE_START_HANG  | Consulte [El servicio cliente DHCP se bloquea o deja de responder](#dhcp-client-service-crashes-or-hangs).  |
    | 1077 - ERROR_SERVICE_NEVER_STARTED  | Consulte [El servicio cliente DHCP no se está ejecutando](#dhcp-client-service-is-disabled).  |
    |1079 - ERROR_DIFERENCE_SERVICE_ACCOUNT   | [Póngase en contacto con el equipo de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.  |
    |1053 | [Póngase en contacto con el equipo de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.  |


#### <a name="dhcp-client-service-is-stopped-because-of-an-access-denied-error"></a>El servicio cliente DHCP se ha detenido debido a un error de denegación de acceso

1. Conéctese a [Serial Console](serial-console-windows.md) y abra una instancia de PowerShell.
2. Descargue la herramienta Monitor de procesos ejecutando el siguiente script:

   ```powershell
   remove-module psreadline
   $source = "https://download.sysinternals.com/files/ProcessMonitor.zip"
   $destination = "c:\temp\ProcessMonitor.zip"
   $wc = New-Object System.Net.WebClient
   $wc.DownloadFile($source,$destination)
   ```
3. Ahora inicie un seguimiento de **procmon**:

   ```
   procmon /Quiet /Minimized /BackingFile c:\temp\ProcMonTrace.PML
   ```
4. Reproduzca el problema iniciando el servicio que está generando el mensaje de **Acceso denegado**:

   ```
   sc start DHCP
   ```

   Cuando se produzca un error, finalice el seguimiento de Monitor de procesos:

   ```
   procmon /Terminate
   ```
5. Recopile el archivo**c:\temp\ProcMonTrace.PML**:

    1. [Conecte un disco de datos a la máquina virtual](../windows/attach-managed-disk-portal.md
).
    2. Use la consola serie para copiar el archivo a la nueva unidad. Por ejemplo, `copy C:\temp\ProcMonTrace.PML F:\`. En este comando, F es la letra de unidad del disco de datos conectado. Reemplace la letra por el valor que corresponda.
    3. Desconecte la unidad de datos y conéctela a una máquina virtual en ejecución que tenga instalado el Monitor de procesos.

6. Abra **ProcMonTrace.PML** con el Monitor de procesos de la máquina virtual en ejecución. A continuación, filtre por  **El resultado es Acceso denegado**, tal y como se muestra en la captura de pantalla siguiente:

    ![Filtro por resultado en el Monitor de procesos](./media/troubleshoot-remote-desktop-services-issues/process-monitor-access-denined.png)

7. Corrija las claves del registro, carpetas o archivos que están en la salida. Normalmente, este problema se produce cuando la cuenta de inicio de sesión usada en el servicio no tiene permisos de ACL para acceder a estos objetos. Para determinar cuáles son los permisos de ACL correctos para la cuenta de inicio de sesión, consulte una máquina virtual que tenga un estado correcto.

#### <a name="dhcp-client-service-is-disabled"></a>El servicio cliente DHCP está deshabilitado

1. Restaure el servicio a su valor de inicio predeterminado:

   ```
   sc config DHCP start= auto
   ```

2. Inicie el servicio:

   ```
   sc start DHCP
   ```

3. Consulte de nuevo el estado del servicio para asegurarse de que está en ejecución:

   ```
   sc query DHCP
   ```

4. Intente conectarse a la máquina virtual mediante Escritorio remoto.

#### <a name="dhcp-client-service-fails-because-of-logon-failure"></a>Se produce un error de inicio de sesión en el servicio cliente DHCP

1. Como este problema se produce si se ha modificado la cuenta de inicio del servicio, revierta la cuenta a su estado predeterminado:

    ```console
    sc config DHCP obj= 'NT Authority\Localservice'
    ```

2. Inicie el servicio:

    ```console
    sc start DHCP
    ```

3. Intente conectarse a la máquina virtual mediante Escritorio remoto.

#### <a name="dhcp-client-service-crashes-or-hangs"></a>El servicio cliente DHCP se bloquea o deja de responder

1. Si el servicio está bloqueado en el estado **Iniciando** o **Deteniendo**, intente detener el servicio:

    ```console
    sc stop DHCP
    ```

2. Aisle el servicio en su propio contenedor "svchost":

    ```console
    sc config DHCP type= own
    ```

3. Inicie el servicio:

    ```console
    sc start DHCP
    ```

4. Si el servicio sigue sin iniciarse, [póngase en contacto con el departamento de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

### <a name="repair-the-vm-offline"></a>Reparación de la máquina virtual sin conexión

#### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Conecte el disco del sistema operativo a una máquina virtual de recuperación.

1. [Conecte el disco del sistema operativo a una máquina virtual de recuperación](./troubleshoot-recovery-disks-portal-windows.md).
2. Inicie una conexión mediante el Escritorio remoto a la máquina virtual de recuperación. Asegúrese de que el disco asociado aparece marcado como **En línea** en la consola de Administración de discos. Anote la letra de unidad que está asignada al disco del sistema operativo conectado.
3.  Abra una instancia del símbolo del sistema con privilegios elevados (**Ejecutar como administrador**). A continuación, ejecute el siguiente script. En este script, se presupone que la letra de unidad asignada al disco del sistema operativo conectado es **F**. Reemplace la letra por el valor que corresponda a su máquina virtual.

    ```
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM

    REM Set default values back on the broken service
    reg add "HKLM\BROKENSYSTEM\ControlSet001\services\DHCP" /v start /t REG_DWORD /d 2 /f
    reg add "HKLM\BROKENSYSTEM\ControlSet001\services\DHCP" /v ObjectName /t REG_SZ /d "NT Authority\LocalService" /f
    reg add "HKLM\BROKENSYSTEM\ControlSet001\services\DHCP" /v type /t REG_DWORD /d 16 /f
    reg add "HKLM\BROKENSYSTEM\ControlSet002\services\DHCP" /v start /t REG_DWORD /d 2 /f
    reg add "HKLM\BROKENSYSTEM\ControlSet002\services\DHCP" /v ObjectName /t REG_SZ /d "NT Authority\LocalService" /f
    reg add "HKLM\BROKENSYSTEM\ControlSet002\services\DHCP" /v type /t REG_DWORD /d 16 /f

    reg unload HKLM\BROKENSYSTEM
    ```

4. [Desconecte el disco del sistema operativo y vuelva a crear la máquina virtual](./troubleshoot-recovery-disks-portal-windows.md). A continuación, compruebe si se ha resuelto el problema.

## <a name="next-steps"></a>Pasos siguientes

Si sigue necesitando ayuda, [póngase en contacto con el servicio de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema.
