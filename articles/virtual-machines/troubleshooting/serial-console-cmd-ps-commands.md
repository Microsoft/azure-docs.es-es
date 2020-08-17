---
title: CMD y PowerShell en máquinas virtuales Windows de Azure | Microsoft Docs
description: Uso de CMD y comandos de PowerShell dentro de SAC en máquinas virtuales Windows de Azure
services: virtual-machines-windows
documentationcenter: ''
author: alsin
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/14/2018
ms.author: alsin
ms.openlocfilehash: 571df9c07e71682e2be51a73e3837c79cb074c3a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87028471"
---
# <a name="windows-commands---cmd-and-powershell"></a>Comandos de Windows: CMD y PowerShell

En esta sección se incluyen ejemplos de comandos para realizar tareas comunes en escenarios donde puede ser necesario usar SAC para acceder a la máquina virtual Windows, como cuando necesita solucionar errores de conexión de RDP.

SAC se ha incluido en todas las versiones de Windows desde Windows Server 2003, pero está deshabilitado de forma predeterminada. SAC se basa en el controlador de kernel `sacdrv.sys`, el servicio `Special Administration Console Helper` (`sacsvr`) y el proceso `sacsess.exe`. Para más información, consulte [Emergency Management Services Tools and Settings](/previous-versions/windows/it-pro/windows-server-2003/cc787940(v%3dws.10)) (Herramientas y configuraciones de Servicios de administración de emergencias).

SAC le permite conectarse a su sistema operativo a través del puerto serie. Cuando se inicia CMD desde SAC, `sacsess.exe` inicia `cmd.exe` dentro del sistema operativo en ejecución. Esto se puede ver en Task Manager si se conecta mediante RDP a su máquina virtual al mismo tiempo que está conectado a SAC mediante la característica de la consola serie. El CMD al que se accede a través de SAC es el mismo `cmd.exe` que usa cuando se conecta mediante RDP. Están disponibles las mismas herramientas y comandos, incluida la posibilidad de iniciar PowerShell desde esa instancia de CMD. Hay una diferencia importante entre SAC y el Entorno de recuperación de Windows (WinRE) y es que SAC le permite administrar su sistema operativo en ejecución, mientras que WinRE se inicia en un sistema operativo mínimo diferente. Aunque las máquinas virtuales no admiten la posibilidad de acceder a WinRE, con la característica de la consola serie las máquinas virtuales de Azure se pueden administrar mediante SAC.

Dado que SAC está limitado a un búfer de pantalla de 80 x 24 sin desplazamiento, agregue `| more` a los comandos para mostrar la salida en una página cada vez. Use `<spacebar>` para ver la página siguiente o `<enter>` para ver la línea siguiente.

`SHIFT+INSERT` es el método abreviado de pegar de la ventana de la consola serie.

Como el búfer de pantalla de SAC está limitado, puede que sea más fácil escribir los comandos más largos en un editor de texto local y luego pegarlos en SAC.

## <a name="view-and-edit-windows-registry-settings"></a>Visualización y edición de la configuración del Registro de Windows
### <a name="verify-rdp-is-enabled"></a>Comprobar que RDP está habilitado
`reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections`

`reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fDenyTSConnections`

La segunda clave (en \Policies) solo existe si está configurada la opción de directiva de grupo pertinente.

### <a name="enable-rdp"></a>Habilitar RDP
`reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0`

`reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fDenyTSConnections /t REG_DWORD /d 0`

La segunda clave (en \Policies) solo se necesitaría si se ha configurado la opción de directiva de grupo pertinente. El valor se volverá a escribir en la siguiente actualización de directiva de grupo si está configurado en la directiva de grupo.

## <a name="manage-windows-services"></a>Administración de los servicios de Windows

### <a name="view-service-state"></a>Ver el estado del servicio
`sc query termservice`
###  <a name="view-service-logon-account"></a>Ver la cuenta de inicio de sesión del servicio
`sc qc termservice`
### <a name="set-service-logon-account"></a>Establecer la cuenta de inicio de sesión del servicio
`sc config termservice obj= "NT Authority\NetworkService"`

Se requiere un espacio después del signo igual.
### <a name="set-service-start-type"></a>Establecer el tipo de inicio del servicio
`sc config termservice start= demand`

Se requiere un espacio después del signo igual. Los valores de inicio posibles son `boot`, `system`, `auto`, `demand`, `disabled`, `delayed-auto`.
### <a name="set-service-dependencies"></a>Establecer las dependencias del servicio
`sc config termservice depend= RPCSS`

Se requiere un espacio después del signo igual.
### <a name="start-service"></a>Iniciar el servicio
`net start termservice`

or

`sc start termservice`
### <a name="stop-service"></a>Detener el servicio
`net stop termservice`

or

`sc stop termservice`
## <a name="manage-networking-features"></a>Administración de las características de red
### <a name="show-nic-properties"></a>Mostrar las propiedades de NIC
`netsh interface show interface`
### <a name="show-ip-properties"></a>Mostrar las propiedades de IP
`netsh interface ip show config`
### <a name="show-ipsec-configuration"></a>Mostrar la configuración de IPSec
`netsh nap client show configuration`
### <a name="enable-nic"></a>Habilitar NIC
`netsh interface set interface name="<interface name>" admin=enabled`
### <a name="set-nic-to-use-dhcp"></a>Establecer NIC para usar DHCP
`netsh interface ip set address name="<interface name>" source=dhcp`

Para más información sobre `netsh`, [haga clic aquí](/windows-server/networking/technologies/netsh/netsh-contexts).

Las máquinas virtuales de Azure se deben configurar siempre en el sistema operativo invitado para usar DHCP para obtener una dirección IP. La configuración de IP estática de Azure sigue usando DHCP para proporcionar la dirección IP estática a la máquina virtual.
### <a name="ping"></a>Ping
`ping 8.8.8.8`
### <a name="port-ping"></a>Ping de puerto
Instalar al cliente de Telnet

`dism /online /Enable-Feature /FeatureName:TelnetClient`

Comprobación de la conectividad

`telnet bing.com 80`

Quitar al cliente de Telnet

`dism /online /Disable-Feature /FeatureName:TelnetClient`

Cuando se limita a los métodos disponibles en Windows de forma predeterminada, PowerShell puede ser un enfoque mejor para probar la conectividad del puerto. Consulte la sección de PowerShell a continuación para ver ejemplos.
### <a name="test-dns-name-resolution"></a>Probar la resolución de nombres DNS
`nslookup bing.com`
### <a name="show-windows-firewall-rule"></a>Mostrar la regla de Firewall de Windows
`netsh advfirewall firewall show rule name="Remote Desktop - User Mode (TCP-In)"`
### <a name="disable-windows-firewall"></a>Deshabilitar el Firewall de Windows
`netsh advfirewall set allprofiles state off`

Puede usar este comando al solucionar problemas para excluir temporalmente el Firewall de Windows. Se habilitará en el siguiente reinicio o cuando lo habilite mediante el siguiente comando. No detenga el servicio Firewall de Windows (MPSSVC) o el servicio Motor de filtrado de base (BFE) como forma de excluir el Firewall de Windows. Si lo hace, se bloqueará toda la conectividad.
### <a name="enable-windows-firewall"></a>Habilitar el Firewall de Windows
`netsh advfirewall set allprofiles state on`
## <a name="manage-users-and-groups"></a>Administración de usuarios y grupos
### <a name="create-local-user-account"></a>Crear una cuenta de usuario local
`net user /add <username> <password>`
### <a name="add-local-user-to-local-group"></a>Agregar un usuario local a un grupo local
`net localgroup Administrators <username> /add`
### <a name="verify-user-account-is-enabled"></a>Comprobar que la cuenta de usuario está habilitada
`net user <username> | find /i "active"`

El nombre de la cuenta de administrador local de las máquinas virtuales de Azure creadas a partir de la imagen generalizada tomará el especificado durante el aprovisionamiento de la máquina virtual. Así que normalmente no será `Administrator`.
### <a name="enable-user-account"></a>Habilitar la cuenta de usuario
`net user <username> /active:yes`
### <a name="view-user-account-properties"></a>Ver las propiedades de la cuenta de usuario
`net user <username>`

Líneas de ejemplo de interés de una cuenta de administrador local:

`Account active Yes`

`Account expires Never`

`Password expires Never`

`Workstations allowed All`

`Logon hours allowed All`

`Local Group Memberships *Administrators`

### <a name="view-local-groups"></a>Ver los grupos locales
`net localgroup`
## <a name="manage-the-windows-event-log"></a>Administración del registro de eventos de Windows
### <a name="query-event-log-errors"></a>Consultar los errores del registro de eventos
`wevtutil qe system /c:10 /f:text /q:"Event[System[Level=2]]" | more`

Cambie `/c:10` por el número deseado de eventos que se devolverán o muévalo para que se devuelvan todos los eventos que coincidan con el filtro.
### <a name="query-event-log-by-event-id"></a>Consultar el registro de eventos por identificador de evento
`wevtutil qe system /c:1 /f:text /q:"Event[System[EventID=11]]" | more`
### <a name="query-event-log-by-event-id-and-provider"></a>Consultar el registro de eventos por identificador de evento y proveedor
`wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11]]" | more`
### <a name="query-event-log-by-event-id-and-provider-for-the-last-24-hours"></a>Consultar el registro de eventos por identificador de evento y proveedor durante las últimas 24 horas
`wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11 and TimeCreated[timediff(@SystemTime) <= 86400000]]]"`

Use `604800000` para recuperar 7 días en lugar de 24 horas.
### <a name="query-event-log-by-event-id-provider-and-eventdata-in-the-last-7-days"></a>Consultar el registro de eventos por identificador de evento, proveedor y datos de evento en los últimos 7 días
`wevtutil qe security /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-Security-Auditing'] and EventID=4624 and TimeCreated[timediff(@SystemTime) <= 604800000]] and EventData[Data[@Name='TargetUserName']='<username>']]" | more`
## <a name="view-or-remove-installed-applications"></a>Visualización o eliminación de las aplicaciones instaladas
### <a name="list-installed-applications"></a>Mostrar las aplicaciones instaladas
`wmic product get Name,InstallDate | sort /r | more`

El comando `sort /r` ordena de forma descendente por fecha de instalación para que sea fácil ver lo que se ha instalado recientemente. Use `<spacebar>` para avanzar a la página siguiente de salida o `<enter>` para avanzar una línea.
### <a name="uninstall-an-application"></a>Desinstalar una aplicación
`wmic path win32_product where name="<name>" call uninstall`

Reemplace `<name>` por el nombre devuelto en el comando anterior en la aplicación que quiere quitar.

## <a name="file-system-management"></a>Administración del sistema de archivos
### <a name="get-file-version"></a>Obtener la versión del archivo
`wmic datafile where "drive='C:' and path='\\windows\\system32\\drivers\\' and filename like 'netvsc%'" get version /format:list`

En este ejemplo se devuelve la versión del archivo del controlador NIC virtual, que es netvsc.sys, netvsc63.sys o netvsc60.sys, según la versión de Windows.
### <a name="scan-for-system-file-corruption"></a>Examinar los archivos del sistema en busca de daños
`sfc /scannow`

Consulte también [Repair a Windows Image](/windows-hardware/manufacture/desktop/repair-a-windows-image) (Reparación de una imagen de Windows).
### <a name="scan-for-system-file-corruption"></a>Examinar los archivos del sistema en busca de daños
`dism /online /cleanup-image /scanhealth`

Consulte también [Repair a Windows Image](/windows-hardware/manufacture/desktop/repair-a-windows-image) (Reparación de una imagen de Windows).
### <a name="export-file-permissions-to-text-file"></a>Exportar los permisos de archivo a un archivo de texto
`icacls %programdata%\Microsoft\Crypto\RSA\MachineKeys /t /c > %temp%\MachineKeys_permissions_before.txt`
### <a name="save-file-permissions-to-acl-file"></a>Guardar los permisos de archivo en el archivo ACL
`icacls %programdata%\Microsoft\Crypto\RSA\MachineKeys /save %temp%\MachineKeys_permissions_before.aclfile /t`
### <a name="restore-file-permissions-from-acl-file"></a>Restaurar los permisos de archivo del archivo ACL
`icacls %programdata%\Microsoft\Crypto\RSA /save %temp%\MachineKeys_permissions_before.aclfile /t`

La ruta de acceso al usar `/restore` debe ser la carpeta primaria de la carpeta que especificó al utilizar `/save`. En este ejemplo, `\RSA` es la carpeta primaria de la carpeta `\MachineKeys` especificada en el ejemplo anterior con `/save`.
### <a name="take-ntfs-ownership-of-a-folder"></a>Tomar la propiedad de NTFS de una carpeta
`takeown /f %programdata%\Microsoft\Crypto\RSA\MachineKeys /a /r`
### <a name="grant-ntfs-permissions-to-a-folder-recursively"></a>Conceder permisos NTFS a una carpeta de forma recursiva
`icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "BUILTIN\Administrators:(F)"`
## <a name="manage-devices"></a>Administración de dispositivos
### <a name="remove-non-present-pnp-devices"></a>Quitar dispositivos Plug and Play no existentes
`%windir%\System32\RUNDLL32.exe %windir%\System32\pnpclean.dll,RunDLL_PnpClean /Devices /Maxclean`
## <a name="manage-group-policy"></a>Administración de directiva de grupo
### <a name="force-group-policy-update"></a>Forzar la actualización de la directiva de grupo
`gpupdate /force /wait:-1`
## <a name="miscellaneous-tasks"></a>Herramientas varias
### <a name="show-os-version"></a>Mostrar la versión del sistema operativo
`ver`

or

`wmic os get caption,version,buildnumber /format:list`

or

`systeminfo  find /i "os name"`

`systeminfo | findstr /i /r "os.*version.*build"`
### <a name="view-os-install-date"></a>Ver la fecha de instalación del sistema operativo
`systeminfo | find /i "original"`

or

`wmic os get installdate`
### <a name="view-last-boot-time"></a>Ver la hora del último arranque
`systeminfo | find /i "system boot time"`
### <a name="view-time-zone"></a>Ver la zona horaria
`systeminfo | find /i "time zone"`

or

`wmic timezone get caption,standardname /format:list`
### <a name="restart-windows"></a>Reiniciar Windows
`shutdown /r /t 0`

Al agregar `/f` se forzará a que las aplicaciones en ejecución se cierren sin avisar a los usuarios.
### <a name="detect-safe-mode-boot"></a>Detectar el modo de arranque seguro
`bcdedit /enum | find /i "safeboot"`

## <a name="windows-commands---powershell"></a>Comandos de Windows: PowerShell

Para ejecutar PowerShell en SAC, después de llegar a un símbolo del sistema CMD, escriba:

`powershell <enter>`

> [!CAUTION]
> Quite el módulo PSReadLine de la sesión de PowerShell antes de ejecutar otros comandos de PowerShell. Existe un problema conocido en el que se pueden introducir caracteres adicionales en el texto pegado desde el portapapeles si PSReadLine se ejecuta en una sesión de PowerShell en SAC.

En primer lugar, compruebe si PSReadLine está cargado. Se carga de forma predeterminada en Windows Server 2016, Windows 10 y versiones posteriores de Windows. Solo existiría en versiones anteriores de Windows si se hubiera instalado manualmente.

Si este comando devuelve un símbolo del sistema sin ninguna salida, significa que el módulo no se ha cargado y puede seguir usando la sesión de PowerShell en SAC de la forma habitual.

`get-module psreadline`

Si el comando anterior devuelve la versión del módulo PSReadLine, ejecute el siguiente comando para descargarlo. Este comando no elimina ni desinstala el módulo, solo lo descarga de la sesión actual de PowerShell.

`remove-module psreadline`

## <a name="view-and-edit-windows-registry-settings"></a>Visualización y edición de la configuración del Registro de Windows
### <a name="verify-rdp-is-enabled"></a>Comprobar que RDP está habilitado
`get-itemproperty -path 'hklm:\system\curRentcontrolset\control\terminal server' -name 'fdenytsconNections'`

`get-itemproperty -path 'hklm:\software\policies\microsoft\windows nt\terminal services' -name 'fdenytsconNections'`

La segunda clave (en \Policies) solo existe si está configurada la opción de directiva de grupo pertinente.
### <a name="enable-rdp"></a>Habilitar RDP
`set-itemproperty -path 'hklm:\system\curRentcontrolset\control\terminal server' -name 'fdenytsconNections' 0 -type dword`

`set-itemproperty -path 'hklm:\software\policies\microsoft\windows nt\terminal services' -name 'fdenytsconNections' 0 -type dword`

La segunda clave (en \Policies) solo se necesitaría si se ha configurado la opción de directiva de grupo pertinente. El valor se volverá a escribir en la siguiente actualización de directiva de grupo si está configurado en la directiva de grupo.
## <a name="manage-windows-services"></a>Administración de los servicios de Windows
### <a name="view-service-details"></a>Ver los detalles del servicio
`get-wmiobject win32_service -filter "name='termservice'" |  format-list Name,DisplayName,State,StartMode,StartName,PathName,ServiceType,Status,ExitCode,ServiceSpecificExitCode,ProcessId`

Aunque se puede usar `Get-Service`, no incluye la cuenta de inicio de sesión del servicio. `Get-WmiObject win32-service` sí la incluye.
### <a name="set-service-logon-account"></a>Establecer la cuenta de inicio de sesión del servicio
`(get-wmiobject win32_service -filter "name='termservice'").Change($null,$null,$null,$null,$null,$false,'NT Authority\NetworkService')`

Cuando se use una cuenta de servicio distinta de `NT AUTHORITY\LocalService`, `NT AUTHORITY\NetworkService` o `LocalSystem`, especifique la contraseña de la cuenta como el último argumento (octavo) después del nombre de cuenta.
### <a name="set-service-startup-type"></a>Establecer el tipo de inicio del servicio
`set-service termservice -startuptype Manual`

`Set-service` acepta `Automatic`, `Manual` o `Disabled` como tipo de inicio.
### <a name="set-service-dependencies"></a>Establecer las dependencias del servicio
`Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\TermService' -Name DependOnService -Value @('RPCSS','TermDD')`
### <a name="start-service"></a>Iniciar el servicio
`start-service termservice`
### <a name="stop-service"></a>Detener el servicio
`stop-service termservice`
## <a name="manage-networking-features"></a>Administración de las características de red
### <a name="show-nic-properties"></a>Mostrar las propiedades de NIC
`get-netadapter | where {$_.ifdesc.startswith('Microsoft Hyper-V Network Adapter')} |  format-list status,name,ifdesc,macadDresS,driverversion,MediaConNectState,MediaDuplexState`

or

`get-wmiobject win32_networkadapter -filter "servicename='netvsc'" |  format-list netenabled,name,macaddress`

`Get-NetAdapter` está disponible en 2012+; para 2008R2, use `Get-WmiObject`.
### <a name="show-ip-properties"></a>Mostrar las propiedades de IP
`get-wmiobject Win32_NetworkAdapterConfiguration -filter "ServiceName='netvsc'" |  format-list DNSHostName,IPAddress,DHCPEnabled,IPSubnet,DefaultIPGateway,MACAddress,DHCPServer,DNSServerSearchOrder`
### <a name="enable-nic"></a>Habilitar NIC
`get-netadapter | where {$_.ifdesc.startswith('Microsoft Hyper-V Network Adapter')} | enable-netadapter`

or

`(get-wmiobject win32_networkadapter -filter "servicename='netvsc'").enable()`

`Get-NetAdapter` está disponible en 2012+; para 2008R2, use `Get-WmiObject`.
### <a name="set-nic-to-use-dhcp"></a>Establecer NIC para usar DHCP
`get-netadapter | where {$_.ifdesc.startswith('Microsoft Hyper-V Network Adapter')} | Set-NetIPInterface -DHCP Enabled`

`(get-wmiobject Win32_NetworkAdapterConfiguration -filter "ServiceName='netvsc'").EnableDHCP()`

`Get-NetAdapter` está disponible en 2012+. Para 2008R2, use `Get-WmiObject`. Las máquinas virtuales de Azure se deben configurar siempre en el sistema operativo invitado para usar DHCP para obtener una dirección IP. La configuración de IP estática de Azure sigue usando DHCP para proporcionar la dirección IP a la máquina virtual.
### <a name="ping"></a>Ping
`test-netconnection`

> [!NOTE]
> Es posible que el cmdlet Write-Progress no funcione con este comando. Como medida de mitigación, puede ejecutar el comando `$ProgressPreference = "SilentlyContinue"` en PowerShell para deshabilitar la barra de progreso.

or

`get-wmiobject Win32_PingStatus -Filter 'Address="8.8.8.8"' | format-table -autosize IPV4Address,ReplySize,ResponseTime`

`Test-Netconnection` sin ningún parámetro intentará hacer ping a `internetbeacon.msedge.net`. Está disponible en 2012+. Para 2008R2, use `Get-WmiObject` como en el segundo ejemplo.
### <a name="port-ping"></a>Ping de puerto
`test-netconnection -ComputerName bing.com -Port 80`

or

`(new-object Net.Sockets.TcpClient).BeginConnect('bing.com','80',$null,$null).AsyncWaitHandle.WaitOne(300)`

`Test-NetConnection` está disponible en 2012+. Para 2008R2, use `Net.Sockets.TcpClient`.
### <a name="test-dns-name-resolution"></a>Probar la resolución de nombres DNS
`resolve-dnsname bing.com`

or

`[System.Net.Dns]::GetHostAddresses('bing.com')`

`Resolve-DnsName` está disponible en 2012+. Para 2008R2, use `System.Net.DNS`.
### <a name="show-windows-firewall-rule-by-name"></a>Mostrar la regla de Firewall de Windows por nombre
`get-netfirewallrule -name RemoteDesktop-UserMode-In-TCP`
### <a name="show-windows-firewall-rule-by-port"></a>Mostrar la regla de Firewall de Windows por puerto
`get-netfirewallportfilter | where {$_.localport -eq 3389} | foreach {Get-NetFirewallRule -Name $_.InstanceId} | format-list Name,Enabled,Profile,Direction,Action`

or

`(new-object -ComObject hnetcfg.fwpolicy2).rules | where {$_.localports -eq 3389 -and $_.direction -eq 1} | format-table Name,Enabled`

`Get-NetFirewallPortFilter` está disponible en 2012+. Para 2008R2, use el objeto COM `hnetcfg.fwpolicy2`.
### <a name="disable-windows-firewall"></a>Deshabilitar el Firewall de Windows
`Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False`

`Set-NetFirewallProfile` está disponible en 2012+. Para 2008R2, use `netsh advfirewall` como se hace referencia en la sección de CMD anterior.
## <a name="manage-users-and-groups"></a>Administración de usuarios y grupos
### <a name="create-local-user-account"></a>Crear una cuenta de usuario local
`new-localuser <name>`
### <a name="verify-user-account-is-enabled"></a>Comprobar que la cuenta de usuario está habilitada
`(get-localuser | where {$_.SID -like "S-1-5-21-*-500"}).Enabled`

or

`(get-wmiobject Win32_UserAccount -Namespace "root\cimv2" -Filter "SID like 'S-1-5-%-500'").Disabled`

`Get-LocalUser` está disponible en 2012+. Para 2008R2, use `Get-WmiObject`. En este ejemplo se muestra la cuenta de administrador local integrada, que siempre tiene el SID `S-1-5-21-*-500`. El nombre de la cuenta de administrador local de las máquinas virtuales de Azure creadas a partir de la imagen generalizada tomará el especificado durante el aprovisionamiento de la máquina virtual. Así que normalmente no será `Administrator`.
### <a name="add-local-user-to-local-group"></a>Agregar un usuario local a un grupo local
`add-localgroupmember -group Administrators -member <username>`
### <a name="enable-local-user-account"></a>Habilitar la cuenta de usuario local
`get-localuser | where {$_.SID -like "S-1-5-21-*-500"} | enable-localuser`

En este ejemplo se habilita la cuenta de administrador local integrada, que siempre tiene el SID `S-1-5-21-*-500`. El nombre de la cuenta de administrador local de las máquinas virtuales de Azure creadas a partir de la imagen generalizada tomará el especificado durante el aprovisionamiento de la máquina virtual. Así que normalmente no será `Administrator`.
### <a name="view-user-account-properties"></a>Ver las propiedades de la cuenta de usuario
`get-localuser | where {$_.SID -like "S-1-5-21-*-500"} | format-list *`

or

`get-wmiobject Win32_UserAccount -Namespace "root\cimv2" -Filter "SID like 'S-1-5-%-500'" |  format-list Name,Disabled,Status,Lockout,Description,SID`

`Get-LocalUser` está disponible en 2012+. Para 2008R2, use `Get-WmiObject`. En este ejemplo se muestra la cuenta de administrador local integrada, que siempre tiene el SID `S-1-5-21-*-500`.
### <a name="view-local-groups"></a>Ver los grupos locales
`(get-localgroup).name | sort` `(get-wmiobject win32_group).Name | sort`

`Get-LocalUser` está disponible en 2012+. Para 2008R2, use `Get-WmiObject`.
## <a name="manage-the-windows-event-log"></a>Administración del registro de eventos de Windows
### <a name="query-event-log-errors"></a>Consultar los errores del registro de eventos
`get-winevent -logname system -maxevents 1 -filterxpath "*[System[Level=2]]" | more`

Cambie `/c:10` por el número deseado de eventos que se devolverán o muévalo para que se devuelvan todos los eventos que coincidan con el filtro.
### <a name="query-event-log-by-event-id"></a>Consultar el registro de eventos por identificador de evento
`get-winevent -logname system -maxevents 1 -filterxpath "*[System[EventID=11]]" | more`
### <a name="query-event-log-by-event-id-and-provider"></a>Consultar el registro de eventos por identificador de evento y proveedor
`get-winevent -logname system -maxevents 1 -filterxpath "*[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11]]" | more`
### <a name="query-event-log-by-event-id-and-provider-for-the-last-24-hours"></a>Consultar el registro de eventos por identificador de evento y proveedor durante las últimas 24 horas
`get-winevent -logname system -maxevents 1 -filterxpath "*[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11 and TimeCreated[timediff(@SystemTime) <= 86400000]]]"`

Use `604800000` para recuperar 7 días en lugar de 24 horas. |
### <a name="query-event-log-by-event-id-provider-and-eventdata-in-the-last-7-days"></a>Consultar el registro de eventos por identificador de evento, proveedor y datos de evento en los últimos 7 días
`get-winevent -logname system -maxevents 1 -filterxpath "*[System[Provider[@Name='Microsoft-Windows-Security-Auditing'] and EventID=4624 and TimeCreated[timediff(@SystemTime) <= 604800000]] and EventData[Data[@Name='TargetUserName']='<username>']]" | more`
## <a name="view-or-remove-installed-applications"></a>Visualización o eliminación de las aplicaciones instaladas
### <a name="list-installed-software"></a>Mostrar el software instalado
`get-wmiobject win32_product | select installdate,name | sort installdate -descending | more`
### <a name="uninstall-software"></a>Desinstalar el software
`(get-wmiobject win32_product -filter "Name='<name>'").Uninstall()`
## <a name="file-system-management"></a>Administración del sistema de archivos
### <a name="get-file-version"></a>Obtener la versión del archivo
`(get-childitem $env:windir\system32\drivers\netvsc*.sys).VersionInfo.FileVersion`

En este ejemplo se devuelve la versión del archivo del controlador NIC virtual, que se llama netvsc.sys, netvsc63.sys o netvsc60.sys, según la versión de Windows.
### <a name="download-and-extract-file"></a>Descargar y extraer el archivo
`$path='c:\bin';md $path;cd $path;(new-object net.webclient).downloadfile( ('htTp:/'+'/download.sysinternals.com/files/SysinternalsSuite.zip'),"$path\SysinternalsSuite.zip");(new-object -com shelL.apPlication).namespace($path).CopyHere( (new-object -com shelL.apPlication).namespace("$path\SysinternalsSuite.zip").Items(),16)`

En este ejemplo se crea una carpeta `c:\bin`, y luego se descarga y se extrae el conjunto de herramientas de Sysinternals en `c:\bin`.
## <a name="miscellaneous-tasks"></a>Herramientas varias
### <a name="show-os-version"></a>Mostrar la versión del sistema operativo
`get-wmiobject win32_operatingsystem | format-list caption,version,buildnumber`
### <a name="view-os-install-date"></a>Ver la fecha de instalación del sistema operativo
`(get-wmiobject win32_operatingsystem).converttodatetime((get-wmiobject win32_operatingsystem).installdate)`
### <a name="view-last-boot-time"></a>Ver la hora del último arranque
`(get-wmiobject win32_operatingsystem).lastbootuptime`
### <a name="view-windows-uptime"></a>Ver el tiempo de actividad de Windows
`"{0:dd}:{0:hh}:{0:mm}:{0:ss}.{0:ff}" -f ((get-date)-(get-wmiobject win32_operatingsystem).converttodatetime((get-wmiobject win32_operatingsystem).lastbootuptime))`

Devuelve el tiempo de actividad como `<days>:<hours>:<minutes>:<seconds>:<milliseconds>`, por ejemplo `49:16:48:00.00`.
### <a name="restart-windows"></a>Reiniciar Windows
`restart-computer`

Al agregar `-force` se forzará a que las aplicaciones en ejecución se cierren sin avisar a los usuarios.
## <a name="instance-metadata"></a>Metadatos de instancia

Puede consultar los metadatos de instancia de Azure desde la máquina virtual de Azure para ver detalles como osType, Location, vmSize, vmId, name, resourceGroupName, subscriptionId, privateIpAddress y publicIpAddress.

Para consultar los metadatos de instancia se requiere una buena conectividad de la red del invitado, dado que se realiza una llamada REST a través del host de Azure al servicio de metadatos de instancia. De modo que, si puede consultar los metadatos de instancia, significa que su invitado puede comunicarse a través de la red a un servicio hospedado por Azure.

Para más información, consulte [Servicio de metadatos de instancia de Azure](../windows/instance-metadata-service.md).

### <a name="instance-metadata"></a>Metadatos de instancia
`$im = invoke-restmethod -headers @{"metadata"="true"} -uri http://169.254.169.254/metadata/instance?api-version=2017-08-01 -method get`

`$im | convertto-json`
### <a name="os-type-instance-metadata"></a>Tipo de sistema operativo (metadatos de instancia)
`$im.Compute.osType`
### <a name="location-instance-metadata"></a>Ubicación (metadatos de instancia)
`$im.Compute.Location`
### <a name="size-instance-metadata"></a>Tamaño (metadatos de instancia)
`$im.Compute.vmSize`
### <a name="vm-id-instance-metadata"></a>Identificador de máquina virtual (metadatos de instancia)
`$im.Compute.vmId`
### <a name="vm-name-instance-metadata"></a>Nombre de máquina virtual (metadatos de instancia)
`$im.Compute.name`
### <a name="resource-group-name-instance-metadata"></a>Nombre del grupo de recursos (metadatos de instancia)
`$im.Compute.resourceGroupName`
### <a name="subscription-id-instance-metadata"></a>Identificador de suscripción (metadatos de instancia)
`$im.Compute.subscriptionId`
### <a name="tags-instance-metadata"></a>Etiquetas (metadatos de instancia)
`$im.Compute.tags`
### <a name="placement-group-id-instance-metadata"></a>Identificador de grupo de selección de ubicación (metadatos de instancia)
`$im.Compute.placementGroupId`
### <a name="platform-fault-domain-instance-metadata"></a>Dominio de error de plataforma (metadatos de instancia)
`$im.Compute.platformFaultDomain`
### <a name="platform-update-domain-instance-metadata"></a>Dominio de actualización de plataforma (metadatos de instancia)
`$im.Compute.platformUpdateDomain`
### <a name="ipv4-private-ip-address-instance-metadata"></a>Dirección IP privadas de IPv4 (metadatos de instancia)
`$im.network.interface.ipv4.ipAddress.privateIpAddress`
### <a name="ipv4-public-ip-address-instance-metadata"></a>Dirección IP pública de IPv4 (metadatos de instancia)
`$im.network.interface.ipv4.ipAddress.publicIpAddress`
### <a name="ipv4-subnet-address--prefix-instance-metadata"></a>Dirección/prefijo de subred IPv4 (metadatos de instancia)
`$im.network.interface.ipv4.subnet.address`

`$im.network.interface.ipv4.subnet.prefix`
### <a name="ipv6-ip-address-instance-metadata"></a>Dirección IP de IPv6 (metadatos de instancia)
`$im.network.interface.ipv6.ipAddress`
### <a name="mac-address-instance-metadata"></a>Dirección MAC (metadatos de instancia)
`$im.network.interface.macAddress`

## <a name="next-steps"></a>Pasos siguientes
* La página principal de la documentación de Windows de la consola serie se encuentra [aquí](serial-console-windows.md).
* La consola serie también está disponible para máquinas virtuales [Linux](serial-console-linux.md).
* Obtenga más información sobre [diagnósticos de arranque](boot-diagnostics.md).
