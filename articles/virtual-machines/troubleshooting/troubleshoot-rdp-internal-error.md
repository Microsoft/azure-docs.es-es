---
title: Se produce un error interno al crear una conexión RDP a Azure Virtual Machines | Microsoft Docs
description: Aprenda a solucionar errores internos de RDP en Microsoft Azure. | Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/22/2018
ms.author: genli
ms.openlocfilehash: 4831a084153c28576cca7c40dfefeb8c5ff3c4e6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87036393"
---
#  <a name="an-internal-error-occurs-when-you-try-to-connect-to-an-azure-vm-through-remote-desktop"></a>Se produce un error interno al intentar conectarse a una máquina virtual de Azure a través de Escritorio remoto

En este artículo se describe un error que puede experimentar al intentar conectarse a una máquina virtual (VM) en Microsoft Azure.


## <a name="symptoms"></a>Síntomas

No se puede conectar a una máquina virtual de Azure mediante el Protocolo de escritorio remoto (RDP). La conexión se queda bloqueada en la sección "Configurando remoto" o recibe el siguiente mensaje de error:

- RDP internal error (Error interno de RDP)
- An internal error has occurred (Se ha producido un error interno)
- This computer can't be connected to the remote computer (Este equipo no se puede conectar al equipo remoto). Try connecting again (Intente conectarse de nuevo). Si el problema no desaparece, póngase en contacto con el propietario del equipo remoto o con el administrador de red


## <a name="cause"></a>Causa

Este problema puede ocurrir por los siguientes motivos:

- No se puede acceder a las claves de cifrado RSA locales.
- El protocolo TLS está deshabilitado.
- El certificado está dañado o ha expirado.

## <a name="solution"></a>Solución

Antes de seguir estos pasos, tome una instantánea del disco del sistema operativo de la máquina virtual afectada como copia de seguridad. Para obtener más información, consulte [Instantánea de un disco](../windows/snapshot-copy-managed-disk.md).

Para solucionar este problema, utilice la consola serie o [repare la máquina virtual sin conexión](#repair-the-vm-offline) mediante la conexión del disco del sistema operativo de la máquina virtual a una máquina virtual de recuperación.


### <a name="use-serial-control"></a>Uso del control serie

Conéctese a una [consola serie y abra una instancia de PowerShell](./serial-console-windows.md#use-cmd-or-powershell-in-serial-console
). Si la consola serie no está habilitada en la máquina virtual, vaya a la sección de [reparación de la máquina virtual sin conexión](#repair-the-vm-offline).

#### <a name="step-1-check-the-rdp-port"></a>Paso: 1 Comprobar el puerto del RDP

1. En una instancia de PowerShell, use [NETSTAT](/windows-server/administration/windows-commands/netstat) para comprobar si otras aplicaciones utilizan el puerto 8080:

    ```powershell
    Netstat -anob |more
    ```

2. Si Termservice.exe usa dicho puerto, vaya al paso 2. Si otro servicio o aplicación que no sea Termservice.exe usa el puerto 8080, siga estos pasos:

    1. Detenga el servicio para la aplicación que utiliza el servicio 3389:

        ```powershell
        Stop-Service -Name <ServiceName> -Force
        ```

    2. Inicie el servicio del terminal:

        ```powershell
        Start-Service -Name Termservice
        ```

2. Si la aplicación no se puede detener o si este método no le resulta válido, cambie el puerto de RDP:

    1. Cambie el puerto:

        ```powershell
        Set-ItemProperty -Path 'HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name PortNumber -value <Hexportnumber>

        Stop-Service -Name Termservice -Force

        Start-Service -Name Termservice
        ```

    2. Establezca el firewall del nuevo puerto:

        ```powershell
        Set-NetFirewallRule -Name "RemoteDesktop-UserMode-In-TCP" -LocalPort <NEW PORT (decimal)>
        ```

    3. [Actualice el grupo de seguridad de red del nuevo puerto](../../virtual-network/security-overview.md) en el puerto RDP de Azure Portal.

#### <a name="step-2-set-correct-permissions-on-the-rdp-self-signed-certificate"></a>Paso 2: Establecer los permisos correctos en el certificado autofirmado de RDP

1. En una instancia de PowerShell, ejecute los siguientes comandos, uno a uno, para renovar el certificado autofirmado de RDP:

    ```powershell
    Import-Module PKI

    Set-Location Cert:\LocalMachine 

    $RdpCertThumbprint = 'Cert:\LocalMachine\Remote Desktop\'+((Get-ChildItem -Path 'Cert:\LocalMachine\Remote Desktop\').thumbprint) 

    Remove-Item -Path $RdpCertThumbprint

    Stop-Service -Name "SessionEnv"

    Start-Service -Name "SessionEnv"
    ```

2. Si no puede renovarlo mediante este método, pruebe a hacerlo de forma remota:

    1. Desde una máquina virtual que tenga conectividad con la máquina con problemas, escriba **mmc** en el cuadro **Ejecutar** para abrir Microsoft Management Console.
    2. En el menú **Archivo**, seleccione **Agregar o quitar complemento**, seleccione **Certificados**y, después, seleccione **Agregar**.
    3. Seleccione **Cuentas de equipo** , seleccione **Otro equipo** y, después, agregue la dirección IP de la máquina virtual con problemas.
    4. Vaya a la carpeta **Escritorio remoto/Certificados**, haga clic con el botón derecho en el certificado y seleccione **Eliminar**.
    5. En una instancia de PowerShell, desde la consola serie, reinicie el servicio de configuración de Escritorio remoto:

        ```powershell
        Stop-Service -Name "SessionEnv"

        Start-Service -Name "SessionEnv"
        ```

3. Restablezca el permiso para la carpeta MachineKeys.

    ```powershell
    remove-module psreadline icacls

    md c:\temp

    icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\BeforeScript_permissions.txt 

    takeown /f "C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys" /a /r

    icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\System:(F)"

    icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\NETWORK SERVICE:(R)"

    icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "BUILTIN\Administrators:(F)"

    icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\AfterScript_permissions.txt 

    Restart-Service TermService -Force
    ```

4. Reinicie la máquina virtual e intente iniciar una conexión mediante Escritorio remoto a la máquina virtual. Si el error no desaparece, vaya al paso siguiente.

#### <a name="step-3-enable-all-supported-tls-versions"></a>Paso 3: Habilitar todas las versiones de TLS compatibles

El cliente de RDP usa TLS 1.0 como protocolo predeterminado. Sin embargo, es posible cambiarlo por TLS 1.1, que se ha convertido en el nuevo estándar. Si TLS 1.1 está deshabilitado en la máquina virtual, se producirá un error en la conexión.

1. En una instancia de CMD, habilite el protocolo TLS:

    ```console
    reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f
    ```

2. Para evitar que la directiva de AD sobrescriba los cambios, detenga temporalmente la actualización de la directiva de grupo:

    ```console
    REG add "HKLM\SYSTEM\CurrentControlSet\Services\gpsvc" /v Start /t REG_DWORD /d 4 /f
    ```

3. Reinicie la máquina virtual para que los cambios surtan efecto. Si se resuelve el problema, ejecute el siguiente comando para volver a habilitar la directiva de grupo:

    ```console
    sc config gpsvc start= auto sc start gpsvc

    gpupdate /force
    ```

    Si el cambio se revierte, significa que hay una directiva de Active Directory en el dominio de la empresa. Para que el problema no vuelva a aparecer tiene que cambiar dicha política.

### <a name="repair-the-vm-offline"></a>Reparación de la máquina virtual sin conexión

#### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Conecte el disco del sistema operativo a una máquina virtual de recuperación.

1. [Conecte el disco del sistema operativo a una máquina virtual de recuperación](./troubleshoot-recovery-disks-portal-windows.md).
2. Después de que el disco del sistema operativo esté conectado a una máquina virtual de recuperación, asegúrese de que el disco aparece marcado como **En línea** en la consola de administración de discos. Anote la letra de unidad que se asigna al disco del sistema operativo conectado.
3. Inicie una conexión mediante el Escritorio remoto a la máquina virtual de recuperación.

#### <a name="enable-dump-log-and-serial-console"></a>Habilitación del registro de volcado de memoria y Serial Console

Para habilitar el registro de volcado de memoria y Serial Console, ejecute el siguiente script.

1. Abra una sesión de símbolo del sistema con privilegios elevados (**Ejecutar como administrador**).
2. Ejecute el siguiente script:

    En este script, se supone que la letra de unidad que se asigna al disco del sistema operativo conectado es F. Reemplácela por el valor apropiado para su máquina virtual.

    ```console
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

#### <a name="reset-the-permission-for-machinekeys-folder"></a>Restablezca el permiso para la carpeta MachineKeys

1. Abra una sesión de símbolo del sistema con privilegios elevados (**Ejecutar como administrador**).
2. Ejecute el siguiente script. En este script, se supone que la letra de unidad que se asigna al disco del sistema operativo conectado es F. Reemplácela por el valor apropiado para su máquina virtual.

    ```console
    Md F:\temp

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\BeforeScript_permissions.txt

    takeown /f "F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys" /a /r

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\System:(F)"

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\NETWORK SERVICE:(R)"

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "BUILTIN\Administrators:(F)"

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\AfterScript_permissions.txt
    ```

#### <a name="enable-all-supported-tls-versions"></a>Habilitación de todas las versiones de TLS compatibles

1. Abra una sesión de símbolo del sistema con privilegios elevados (**Ejecutar como administrador**) y ejecute los siguientes comandos. En el siguiente script se supone que la letra de unidad que se asigna al disco del sistema operativo conectado es F. Reemplácela por el valor apropiado para su máquina virtual.
2. Compruebe que TLS está habilitado:

    ```console
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWO
    ```

3. Si la clave no existe o su valor es **0**, habilite el protocolo, para lo que debe ejecutar los siguientes scripts:

    ```console
    REM Enable TLS 1.0, TLS 1.1 and TLS 1.2

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f
    ```

4. Habilite NLA:

    ```console
    REM Enable NLA

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD /d 1 /f reg unload HKLM\BROKENSYSTEM
    ```

5. [Desasocie el disco del sistema operativo y vuelva a crear la máquina virtual](./troubleshoot-recovery-disks-portal-windows.md) y, a continuación, compruebe si se ha resuelto el problema.
