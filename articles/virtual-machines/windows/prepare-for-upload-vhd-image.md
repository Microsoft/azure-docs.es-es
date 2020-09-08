---
title: Preparación de un disco duro virtual de Windows para cargar en Azure
description: Aprenda a preparar un VHD o un VHDX de Windows para cargarlo en Azure
author: glimoli
manager: dcscontentpm
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: troubleshooting
ms.date: 04/28/2020
ms.author: genli
ms.openlocfilehash: 8b5124a0336773412ae9c36a32a0f6f86da62a31
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056251"
---
# <a name="prepare-a-windows-vhd-or-vhdx-to-upload-to-azure"></a>Preparación de un VHD o un VHDX de Windows antes de cargarlo en Azure

Antes de cargar una máquina virtual Windows desde un entorno local en Azure, debe preparar el disco duro virtual (VHD o VHDX). Azure admite máquinas virtuales de generación 1 y de generación 2 que estén en el formato de archivo VHD y tengan un disco de tamaño fijo. El tamaño máximo permitido del disco duro virtual del sistema operativo en una máquina virtual de generación 1 es de 2 TB.

Puede convertir un archivo VHDX en VHD y también convertir un disco de expansión dinámica en un disco de tamaño fijo, pero no puede cambiar la generación de una máquina virtual. Para obtener más información, consulte [¿Debería crear una máquina virtual de generación 1 o 2 en Hyper-V?](/windows-server/virtualization/hyper-v/plan/Should-I-create-a-generation-1-or-2-virtual-machine-in-Hyper-V) y [Compatibilidad para máquinas virtuales de generación 2 en Azure](generation-2.md).

Para información sobre la directiva de soporte de software de servidor de Microsoft ejecutado en Azure, vea [Soporte técnico del software de servidor de Microsoft para máquinas virtuales de Microsoft Azure](https://support.microsoft.com/help/2721672/).

> [!NOTE]
> Las instrucciones de este artículo se aplican a:
>
> - La versión de Windows Server 2008 R2 de 64 bits y versiones posteriores de los sistemas operativos de Windows Server. Para más información sobre cómo ejecutar un sistema operativo de 32 bits en Azure, vea [Compatibilidad de sistemas operativos de 32 bits en máquinas virtuales de Azure](https://support.microsoft.com/help/4021388/).
> - Si se va a usar una herramienta de recuperación ante desastres para migrar la carga de trabajo, como Azure Site Recovery o Azure Migrate, sigue siendo necesario este proceso en el sistema operativo invitado para preparar la imagen antes de la migración.

## <a name="convert-the-virtual-disk-to-a-fixed-size-vhd"></a>Conversión de un disco virtual en un VHD de tamaño fijo

Use uno de los métodos descritos en esta sección para convertir y cambiar el tamaño del disco virtual al formato requerido por Azure:

1. Cree una copia de seguridad de la máquina virtual antes de ejecutar el proceso de conversión del disco virtual o el proceso de cambio de tamaño.

1. Asegúrese de que el disco duro virtual de Windows funciona correctamente en el servidor local. Resuelva los errores dentro de la propia máquina virtual antes de intentar la conversión o la carga en Azure.

1. Convierta el disco virtual al tipo de tamaño fijo.

1. Cambie el tamaño del disco virtual para cumplir los requisitos de Azure:

   1. Los discos de Azure deben tener un tamaño virtual alineado con 1 MiB. Si el VHD es una fracción de 1 MiB, deberá cambiar el tamaño del disco a un múltiplo de 1 MiB. Los discos que son fracciones de 1 MiB provocan errores al crear imágenes a partir del disco duro virtual cargado. Para comprobarlo, puede usar el cmdlet [Get-VHD](/powershell/module/hyper-v/get-vhd) de PowerShell para mostrar el valor de "Size", que debe ser un múltiplo de 1 MiB en Azure y "FileSize", que será igual al valor de "Size" más 512 bytes para el pie de página del VHD.
   
   1. El tamaño máximo permitido del disco duro virtual del sistema operativo en una máquina virtual de generación 1 es de 2048 GiB (2 TiB). 
   1. El tamaño máximo de un disco de datos es de 32 767 GiB (32 TiB).

> [!NOTE]
> - Si va a preparar un disco del sistema operativo Windows después de convertirlo en un disco fijo y cambiar el tamaño si es necesario, cree una máquina virtual que use el disco. Inicie la máquina virtual, inicie sesión en ella y continúe con las secciones de este artículo para finalizar la preparación para la carga.  
> - Si va a preparar un disco de datos, puede dejar esta sección y continuar con la carga del disco.

### <a name="use-hyper-v-manager-to-convert-the-disk"></a>Uso de Administrador de Hyper-V para convertir el disco

1. Abra el Administrador de Hyper-V y seleccione el equipo local de la izquierda. En el menú situado sobre la lista de equipos, seleccione **Acción** > **Editar disco**.
1. En la página **Localizar disco duro virtual**, localice y seleccione el disco virtual.
1. En la página **Elegir acción**, seleccione **Convertir** > **Siguiente**.
1. Para realizar la conversión desde VHDX, seleccione **VHD** > **Siguiente**.
1. Para realizar la conversión desde un disco de expansión dinámica, seleccione **Tamaño fijo** > **Siguiente**.
1. Localice y seleccione una ruta de acceso para guardar el nuevo archivo VHD.
1. Seleccione **Finalizar**.

### <a name="use-powershell-to-convert-the-disk"></a>Uso de PowerShell para convertir el disco

Puede convertir un disco virtual mediante el cmdlet [Convert-VHD](/powershell/module/hyper-v/convert-vhd) en PowerShell. Si necesita información sobre la instalación de este cmdlet, haga clic [aquí](/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server).

Con el siguiente ejemplo, el disco se convierte de VHDX a VHD. También se convierte un disco de expansión dinámica en un disco de tamaño fijo.

```powershell
Convert-VHD -Path C:\test\MyVM.vhdx -DestinationPath C:\test\MyNewVM.vhd -VHDType Fixed
```

En este ejemplo, reemplace el valor **Path** por la ruta de acceso del disco duro virtual que quiera convertir. Reemplace el valor de **DestinationPath** por la nueva ruta de acceso y el nombre del disco convertido.

### <a name="convert-from-vmware-vmdk-disk-format"></a>Conversión del formato de disco VMDK de VMware

Si tiene una imagen de máquina virtual de Windows en [formato de archivo VMDK](https://en.wikipedia.org/wiki/VMDK), use [Microsoft Virtual Machine Converter](https://www.microsoft.com/download/details.aspx?id=42497) para convertirla en un disco duro virtual. Para más información, vea [Cómo convertir un VMDK de VMWare a VHD de Hyper-V](/archive/blogs/timomta/how-to-convert-a-vmware-vmdk-to-hyper-v-vhd).

### <a name="use-hyper-v-manager-to-resize-the-disk"></a>Uso del Administrador de Hyper-V para cambiar el tamaño del disco

1. Abra el Administrador de Hyper-V y seleccione el equipo local de la izquierda. En el menú situado sobre la lista de equipos, seleccione **Acción** > **Editar disco**.
1. En la página **Localizar disco duro virtual**, localice y seleccione el disco virtual.
1. En la página **Elegir acción**, seleccione **Expandir** > **Siguiente**.
1. En la página **Localizar disco duro virtual**, escriba el nuevo tamaño en GiB > **Siguiente**.
1. Seleccione **Finalizar**.

### <a name="use-powershell-to-resize-the-disk"></a>Uso de PowerShell para cambiar el tamaño del disco

Puede cambiar el tamaño de un disco virtual mediante el cmdlet [Resize-VHD](/powershell/module/hyper-v/resize-vhd) de PowerShell. Si necesita información sobre la instalación de este cmdlet, haga clic [aquí](/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server).

En el ejemplo siguiente se cambia el tamaño del disco de 100,5 MiB a 101 MiB para cumplir con el requisito de alineación de Azure.

```powershell
Resize-VHD -Path C:\test\MyNewVM.vhd -SizeBytes 105906176
```

En este ejemplo, reemplace el valor de **Path** por la ruta de acceso del disco duro virtual cuyo tamaño quiere cambiar. Reemplace el valor de **SizeBytes** por el nuevo tamaño en bytes del disco.

## <a name="system-file-checker"></a>Comprobador de archivos de sistema

### <a name="run-windows-system-file-checker-utility-before-generalization-of-os-image"></a>Ejecución de la utilidad Comprobador de archivos de sistema de Windows antes de generalizar la imagen del SO

Comprobador de archivos de sistema (SFC) se usa para comprobar y reemplazar los archivos del sistema de Windows.

> [!IMPORTANT]
> Use una sesión de PowerShell con privilegios elevados para ejecutar los ejemplos de este artículo.

Ejecute el comando SFC:

```powershell
sfc.exe /scannow
```

```Output
Beginning system scan.  This process will take some time.

Beginning verification phase of system scan.
Verification 100% complete.

Windows Resource Protection did not find any integrity violations.
```

Una vez completado el examen de SFC, instale las actualizaciones de Windows y reinicie el equipo.

## <a name="set-windows-configurations-for-azure"></a>Establecimiento de configuraciones de Windows para Azure

> [!NOTE]
> La plataforma Azure monta un archivo ISO en el DVD-ROM cuando se crea una máquina virtual Windows a partir de una imagen generalizada. Por este motivo, el DVD-ROM debe estar habilitado en el sistema operativo de la imagen generalizada. Si está deshabilitado, la máquina virtual Windows se bloqueará en la configuración rápida (OOBE).

1. Quite cualquier ruta estática persistente en la tabla de enrutamiento:

   - Para ver la tabla de enrutamiento, ejecute `route.exe print`.
   - Compruebe la sección **Persistence Routes** (Rutas de persistencia). Si hay una ruta persistente, use el comando `route.exe delete` para quitarla.

1. Quite al proxy WinHTTP:

   ```powershell
   netsh.exe winhttp reset proxy
   ```

    Si es necesario que la máquina virtual funcione con algún proxy concreto, agregue una excepción de proxy para la dirección IP de Azure ([168.63.129.16](../../virtual-network/what-is-ip-address-168-63-129-16.md)), de forma que la máquina virtual tenga conectividad con Azure:

    ```
    $proxyAddress='<your proxy server>'
    $proxyBypassList='<your list of bypasses>;168.63.129.16'
    netsh.exe winhttp set proxy $proxyAddress $proxyBypassList
    ```

1. Abra DiskPart:

   ```powershell
   diskpart.exe
   ```

   Establezca la directiva SAN del disco en [`Onlineall`](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/gg252636(v=ws.11)):

   ```DiskPart
   DISKPART> san policy=onlineall
   DISKPART> exit
   ```

1. Establezca la Hora universal coordinada (UTC) para Windows. Establezca también el tipo de inicio del servicio Hora de Windows **w32time** en **Automatic**:

   ```powershell
   Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Control\TimeZoneInformation -Name RealTimeIsUniversal -Value 1 -Type DWord -Force
   Set-Service -Name w32time -StartupType Automatic
   ```

1. Establezca el perfil de energía en alto rendimiento:

   ```powershell
   powercfg.exe /setactive SCHEME_MIN
   ```

1. Asegúrese de que las variables de entorno **TEMP** y **TMP** se establecen en sus valores predeterminados:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment' -Name TEMP -Value "%SystemRoot%\TEMP" -Type ExpandString -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment' -Name TMP -Value "%SystemRoot%\TEMP" -Type ExpandString -Force
   ```

## <a name="check-the-windows-services"></a>Comprobación de los servicios de Windows

Asegúrese de que cada uno de los siguientes servicios de Windows está configurado en el valor predeterminado de Windows. Estos servicios son la configuración mínima necesaria para garantizar la conectividad de las máquinas virtuales. Para establecer la configuración de inicio, ejecute el siguiente ejemplo:

```powershell
Get-Service -Name BFE, Dhcp, Dnscache, IKEEXT, iphlpsvc, nsi, mpssvc, RemoteRegistry |
  Where-Object StartType -ne Automatic |
    Set-Service -StartupType Automatic

Get-Service -Name Netlogon, Netman, TermService |
  Where-Object StartType -ne Manual |
    Set-Service -StartupType Manual
```

## <a name="update-remote-desktop-registry-settings"></a>Actualización de la configuración del Registro de Escritorio remoto

Asegúrese de que la siguiente configuración está establecida correctamente para la conexión remota:

> [!NOTE]
> Si recibe un mensaje de error al ejecutar `Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services -Name <string> -Value <object>`, puede pasarlo por alto sin ningún problema. Significa que el dominio no establece esa configuración mediante un objeto de directiva de grupo.

1. Protocolo de escritorio remoto (RDP) está habilitado:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server' -Name fDenyTSConnections -Value 0 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name fDenyTSConnections -Value 0 -Type DWord -Force
   ```

1. El puerto de RDP está configurado correctamente con el puerto predeterminado 3389:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name PortNumber -Value 3389 -Type DWord -Force
   ```

   Al implementar una máquina virtual, las reglas predeterminadas se crean para el puerto 3389. Cambie el número de puerto una vez implementada la máquina virtual en Azure.

1. El agente de escucha está activo en todas las interfaces de red:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name LanAdapter -Value 0 -Type DWord -Force
   ```

1. Configure el modo Autenticación a nivel de red (NLA) para las conexiones RDP:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name UserAuthentication -Value 1 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name SecurityLayer -Value 1 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name fAllowSecProtocolNegotiation -Value 1 -Type DWord -Force
   ```

1. Establezca el valor de conexión persistente:

   ```PowerShell
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name KeepAliveEnable -Value 1  -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name KeepAliveInterval -Value 1  -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name KeepAliveTimeout -Value 1 -Type DWord -Force
   ```

1. Establezca las opciones de reconexión:

   ```PowerShell
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name fDisableAutoReconnect -Value 0 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name fInheritReconnectSame -Value 1 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name fReconnectSame -Value 0 -Type DWord -Force
   ```

1. Limite el número de conexiones simultáneas:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name MaxInstanceCount -Value 4294967295 -Type DWord -Force
   ```

1. Quite cualquier certificado autofirmado que pueda haber enlazado al agente de escucha del Protocolo de escritorio remoto:

   ```powershell
   if ((Get-Item -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp').Property -contains 'SSLCertificateSHA1Hash')
   {
       Remove-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name SSLCertificateSHA1Hash -Force
   }
   ```

   Este código garantiza que puede se conectar al implementar la máquina virtual. También puede revisar esta configuración después de implementar la máquina virtual en Azure.

1. Si la máquina virtual forma parte de un dominio, compruebe las siguientes directivas para asegurarse de que la configuración anterior no se revierta.

    |                 Objetivo                  |                                                                            Directiva                                                                            |                           Value                            |
    | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
    | RDP está habilitado                        | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\ Componentes\Servicios de Escritorio remoto\Host de sesión de Escritorio remoto\Conexiones         | Permitir a los usuarios conectarse de forma remota desde el Escritorio remoto    |
    | Directiva de grupo de NLA                      | Configuración\Plantillas administrativas\Componentes\Servicios de Escritorio remoto\Host de sesión de Escritorio remoto\Seguridad                                                    | Requerir la autenticación del usuario para las conexiones remotas mediante Autenticación a nivel de red |
    | Configuración de configuración persistente                   | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\ Componentes Windows\Servicios de Escritorio remoto\Host de sesión de Escritorio remoto\Conexiones | Configurar el intervalo de conexión persistente                   |
    | Configuración de reconexión                    | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\ Componentes Windows\Servicios de Escritorio remoto\Host de sesión de Escritorio remoto\Conexiones | Reconectar automáticamente                                    |
    | Configuración de número limitado de conexiones | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\ Componentes Windows\Servicios de Escritorio remoto\Host de sesión de Escritorio remoto\Conexiones | Limitar el número de conexiones                                |

## <a name="configure-windows-firewall-rules"></a>Configuración de reglas de firewall de Windows

1. Active Firewall de Windows en los tres perfiles (dominio, estándar y público):

   ```powershell
   Set-NetFirewallProfile -Profile Domain, Public, Private -Enabled True
   ```

1. Ejecute el ejemplo siguiente para permitir WinRM mediante los tres perfiles de firewall (dominio, privado y público) y habilitar el servicio remoto de PowerShell:

   ```powershell
   Enable-PSRemoting -Force
   Set-NetFirewallRule -DisplayName 'Windows Remote Management (HTTP-In)' -Enabled True
   ```

1. Habilite las siguientes reglas de firewall para permitir el tráfico RDP:

   ```powershell
   Set-NetFirewallRule -DisplayGroup 'Remote Desktop' -Enabled True
   ```

1. Habilite la regla de uso compartido de archivos e impresoras para que la máquina virtual pueda responder a las solicitudes ping dentro de la red virtual:

   ```powershell
   Set-NetFirewallRule -DisplayName 'File and Printer Sharing (Echo Request - ICMPv4-In)' -Enabled True
   ```

1. Cree una regla para la red de la plataforma Azure:

   ```powershell
   New-NetFirewallRule -DisplayName AzurePlatform -Direction Inbound -RemoteAddress 168.63.129.16 -Profile Any -Action Allow -EdgeTraversalPolicy Allow
   New-NetFirewallRule -DisplayName AzurePlatform -Direction Outbound -RemoteAddress 168.63.129.16 -Profile Any -Action Allow
   ```

1. Si la máquina virtual forma parte de un dominio, compruebe las siguientes directivas de Azure AD para asegurarse de que la configuración anterior no se revierta.

    |                 Objetivo                 |                                                                         Directiva                                                                          |                  Value                  |
    | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
    | Habilitar los perfiles de Firewall de Windows | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\Red\Conexión de red\Firewall de Windows\Perfil de dominio\Firewall de Window   | Proteger todas las conexiones de red         |
    | Habilitar RDP                           | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\Red\Conexión de red\Firewall de Windows\Perfil de dominio\Firewall de Window   | Permitir excepciones de Escritorio remoto entrantes |
    |                                      | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\Red\Conexión de red\Firewall de Windows\Perfil estándar\Firewall de Windows | Permitir excepciones de Escritorio remoto entrantes |
    | Habilitar ICMP-V4                       | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\Red\Conexión de red\Firewall de Windows\Perfil de dominio\Firewall de Window   | Permitir excepciones de ICMP                   |
    |                                      | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\Red\Conexión de red\Firewall de Windows\Perfil estándar\Firewall de Windows | Permitir excepciones de ICMP                   |

## <a name="verify-the-vm"></a>Comprobación de la máquina virtual

Asegúrese de que la máquina virtual tiene un estado correcto, es segura y está accesible desde RDP:

1. Para garantizar que el disco es coherente y está en buen estado, compruebe el disco en el próximo reinicio de máquina virtual:

   ```powershell
   chkdsk.exe /f
   ```

   Asegúrese de que el informe muestra un disco limpio y en buen estado.

1. Establezca la configuración de datos de la configuración de arranque (BCD).

   ```powershell
   bcdedit.exe /set "{bootmgr}" integrityservices enable
   bcdedit.exe /set "{default}" device partition=C:
   bcdedit.exe /set "{default}" integrityservices enable
   bcdedit.exe /set "{default}" recoveryenabled Off
   bcdedit.exe /set "{default}" osdevice partition=C:
   bcdedit.exe /set "{default}" bootstatuspolicy IgnoreAllFailures

   #Enable Serial Console Feature
   bcdedit.exe /set "{bootmgr}" displaybootmenu yes
   bcdedit.exe /set "{bootmgr}" timeout 5
   bcdedit.exe /set "{bootmgr}" bootems yes
   bcdedit.exe /ems "{current}" ON
   bcdedit.exe /emssettings EMSPORT:1 EMSBAUDRATE:115200
   ```

1. El registro de volcado de memoria puede ser útil para solucionar problemas de bloqueo de Windows. Habilite la colección de registros de volcado de memoria:

   ```powershell
   # Set up the guest OS to collect a kernel dump on an OS crash event
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -Name CrashDumpEnabled -Type DWord -Force -Value 2
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -Name DumpFile -Type ExpandString -Force -Value "%SystemRoot%\MEMORY.DMP"
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -Name NMICrashDump -Type DWord -Force -Value 1

   # Set up the guest OS to collect user mode dumps on a service crash event
   $key = 'HKLM:\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps'
   if ((Test-Path -Path $key) -eq $false) {(New-Item -Path 'HKLM:\SOFTWARE\Microsoft\Windows\Windows Error Reporting' -Name LocalDumps)}
   New-ItemProperty -Path $key -Name DumpFolder -Type ExpandString -Force -Value 'C:\CrashDumps'
   New-ItemProperty -Path $key -Name CrashCount -Type DWord -Force -Value 10
   New-ItemProperty -Path $key -Name DumpType -Type DWord -Force -Value 2
   Set-Service -Name WerSvc -StartupType Manual
   ```

1. Compruebe que el repositorio de Instrumental de administración de Windows (WMI) es coherente:

   ```powershell
   winmgmt.exe /verifyrepository
   ```

   Si el repositorio está dañado, consulte [WMI: daños en el repositorio, o no](https://techcommunity.microsoft.com/t5/ask-the-performance-team/wmi-repository-corruption-or-not/ba-p/375484).

1. Asegúrese de que ninguna otra aplicación está usando el puerto 3389. Este puerto se usa para el servicio RDP en Azure. Para ver qué puertos se usan en la máquina virtual, ejecute `netstat.exe -anob`:

   ```powershell
   netstat.exe -anob
   ```

1. Para cargar un VHD de Windows que sea un controlador de dominio:

   - Siga [estos pasos adicionales](https://support.microsoft.com/kb/2904015) para preparar el disco.

   - Asegúrese de que conoce la contraseña de Modo de restauración de servicios de directorio (DSRM), por si alguna vez tiene que iniciar la máquina virtual en DSRM. Para más información, vea [Establecer una contraseña de DSRM](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc754363(v=ws.11)).

1. Asegúrese de que conoce la cuenta predefinida de administrador y la contraseña. Es posible que quiera restablecer la contraseña de administrador local actual y asegurarse de que puede usar esta cuenta para iniciar sesión en Windows mediante la conexión RDP. Este permiso de acceso se controla mediante el objeto de directiva de grupo "Permitir inicio de sesión a través de Servicios de Escritorio remoto". Puede ver este objeto en el Editor de directivas de grupo local:

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights Assignment`

1. Compruebe las siguientes directivas de Azure AD para asegurarse de que no estén bloqueando el acceso del RDP:

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights
      Assignment\Deny access to this computer from the network`

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights
      Assignment\Deny log on through Remote Desktop Services`

1. Compruebe la siguiente directiva de Azure AD para asegurarse de que no están quitando ninguna de las cuentas de acceso obligatorias:

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights Assignment\Access this computer from the network`

   En la directiva deben aparecer los siguientes grupos:

   - Administradores

   - Operadores de copias de seguridad

   - Todos

   - Usuarios

1. Reinicie la máquina virtual para asegurarse de que Windows sigue funcionando correctamente y puede conectarse a ella a través de RDP. Llegado este punto, plantéese la creación de una máquina virtual en el servidor de Hyper-V local para asegurarse de que la máquina virtual se inicia completamente. Tras ello, realice pruebas para asegurarse de que se puede acceder a la máquina virtual a través de RDP.

1. Quite los filtros extra de Interfaz de controlador de transporte (TDI). Por ejemplo, quite el software que analice paquetes TCP o firewalls adicionales. Revise esto más adelante una vez implementada la máquina virtual en Azure.

1. Desinstale cualquier otro software de terceros o controlador relacionado con componentes físicos, o cualquier otra tecnología de virtualización.

### <a name="install-windows-updates"></a>Instalación de actualizaciones de Windows

Lo ideal es mantener la máquina actualizada en el *nivel de revisión* pero, si no es posible, asegúrese de que las siguientes actualizaciones están instaladas. Para obtener las actualizaciones más recientes, consulte las páginas del historial de actualizaciones de Windows: [Windows 10 y Windows Server 2019](https://support.microsoft.com/help/4000825), [Windows 8.1 y Windows Server 2012 R2](https://support.microsoft.com/help/4009470), y [Windows 7 SP1 y Windows Server 2008 R2 SP1](https://support.microsoft.com/help/4009469).

<br />

|        Componente        |     Binary     | Windows 7 SP1, Windows Server 2008 R2 SP1 |       Windows 8, Windows Server 2012        | Windows 8.1, Windows Server 2012 R2 | Windows 10 v1607, Windows Server 2016 v1607 |      Windows 10 v1703      | Windows 10 v1709, Windows Server 2016 v1709 | Windows 10 v1803, Windows Server 2016 v1803 |
| ----------------------- | -------------- | ----------------------------------------- | ------------------------------------------- | ----------------------------------- | ------------------------------------------- | -------------------------- | ------------------------------------------- | ------------------------------------------- |
| Storage                 | disk.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17638 / 6.2.9200.21757 - KB3137061 | 6.3.9600.18203 - KB3137061          | -                                           | -                          | -                                           | -                                           |
|                         | storport.sys   | 6.1.7601.23403 - KB3125574                | 6.2.9200.17188 / 6.2.9200.21306 - KB3018489 | 6.3.9600.18573 - KB4022726          | 10.0.14393.1358 - KB4022715                 | 10.0.15063.332             | -                                           | -                                           |
|                         | ntfs.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17623 / 6.2.9200.21743 - KB3121255 | 6.3.9600.18654 - KB4022726          | 10.0.14393.1198 - KB4022715                 | 10.0.15063.447             | -                                           | -                                           |
|                         | Iologmsg.dll   | 6.1.7601.23403 - KB3125574                | 6.2.9200.16384 - KB2995387                  | -                                   | -                                           | -                          | -                                           | -                                           |
|                         | Classpnp.sys   | 6.1.7601.23403 - KB3125574                | 6.2.9200.17061 / 6.2.9200.21180 - KB2995387 | 6.3.9600.18334 - KB3172614          | 10.0.14393.953 - KB4022715                  | -                          | -                                           | -                                           |
|                         | Volsnap.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.17047 / 6.2.9200.21165 - KB2975331 | 6.3.9600.18265 - KB3145384          | -                                           | 10.0.15063.0               | -                                           | -                                           |
|                         | partmgr.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.16681 - KB2877114                  | 6.3.9600.17401 - KB3000850          | 10.0.14393.953 - KB4022715                  | 10.0.15063.0               | -                                           | -                                           |
|                         | volmgr.sys     |                                           |                                             |                                     |                                             | 10.0.15063.0               | -                                           | -                                           |
|                         | Volmgrx.sys    | 6.1.7601.23403 - KB3125574                | -                                           | -                                   | -                                           | 10.0.15063.0               | -                                           | -                                           |
|                         | Msiscsi.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.21006 - KB2955163                  | 6.3.9600.18624 - KB4022726          | 10.0.14393.1066 - KB4022715                 | 10.0.15063.447             | -                                           | -                                           |
|                         | Msdsm.sys      | 6.1.7601.23403 - KB3125574                | 6.2.9200.21474 - KB3046101                  | 6.3.9600.18592 - KB4022726          | -                                           | -                          | -                                           | -                                           |
|                         | Mpio.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.21190 - KB3046101                  | 6.3.9600.18616 - KB4022726          | 10.0.14393.1198 - KB4022715                 | -                          | -                                           | -                                           |
|                         | vmstorfl.sys   | 6.3.9600.18907 - KB4072650                | 6.3.9600.18080 - KB3063109                  | 6.3.9600.18907 - KB4072650          | 10.0.14393.2007 - KB4345418                 | 10.0.15063.850 - KB4345419 | 10.0.16299.371 - KB4345420                  | -                                           |
|                         | Fveapi.dll     | 6.1.7601.23311 - KB3125574                | 6.2.9200.20930 - KB2930244                  | 6.3.9600.18294 - KB3172614          | 10.0.14393.576 - KB4022715                  | -                          | -                                           | -                                           |
|                         | Fveapibase.dll | 6.1.7601.23403 - KB3125574                | 6.2.9200.20930 - KB2930244                  | 6.3.9600.17415 - KB3172614          | 10.0.14393.206 - KB4022715                  | -                          | -                                           | -                                           |
| Red                 | netvsc.sys     | -                                         | -                                           | -                                   | 10.0.14393.1198 - KB4022715                 | 10.0.15063.250 - KB4020001 | -                                           | -                                           |
|                         | mrxsmb10.sys   | 6.1.7601.23816 - KB4022722                | 6.2.9200.22108 - KB4022724                  | 6.3.9600.18603 - KB4022726          | 10.0.14393.479 - KB4022715                  | 10.0.15063.483             | -                                           | -                                           |
|                         | mrxsmb20.sys   | 6.1.7601.23816 - KB4022722                | 6.2.9200.21548 - KB4022724                  | 6.3.9600.18586 - KB4022726          | 10.0.14393.953 - KB4022715                  | 10.0.15063.483             | -                                           | -                                           |
|                         | mrxsmb.sys     | 6.1.7601.23816 - KB4022722                | 6.2.9200.22074 - KB4022724                  | 6.3.9600.18586 - KB4022726          | 10.0.14393.953 - KB4022715                  | 10.0.15063.0               | -                                           | -                                           |
|                         | tcpip.sys      | 6.1.7601.23761 - KB4022722                | 6.2.9200.22070 - KB4022724                  | 6.3.9600.18478 - KB4022726          | 10.0.14393.1358 - KB4022715                 | 10.0.15063.447             | -                                           | -                                           |
|                         | http.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17285 - KB3042553                  | 6.3.9600.18574 - KB4022726          | 10.0.14393.251 - KB4022715                  | 10.0.15063.483             | -                                           | -                                           |
|                         | vmswitch.sys   | 6.1.7601.23727 - KB4022719                | 6.2.9200.22117 - KB4022724                  | 6.3.9600.18654 - KB4022726          | 10.0.14393.1358 - KB4022715                 | 10.0.15063.138             | -                                           | -                                           |
| Core                    | ntoskrnl.exe   | 6.1.7601.23807 - KB4022719                | 6.2.9200.22170 - KB4022718                  | 6.3.9600.18696 - KB4022726          | 10.0.14393.1358 - KB4022715                 | 10.0.15063.483             | -                                           | -                                           |
| Servicios de Escritorio remoto | rdpcorets.dll  | 6.2.9200.21506 - KB4022719                | 6.2.9200.22104 - KB4022724                  | 6.3.9600.18619 - KB4022726          | 10.0.14393.1198 - KB4022715                 | 10.0.15063.0               | -                                           | -                                           |
|                         | termsrv.dll    | 6.1.7601.23403 - KB3125574                | 6.2.9200.17048 - KB2973501                  | 6.3.9600.17415 - KB3000850          | 10.0.14393.0 - KB4022715                    | 10.0.15063.0               | -                                           | -                                           |
|                         | termdd.sys     | 6.1.7601.23403 - KB3125574                | -                                           | -                                   | -                                           | -                          | -                                           | -                                           |
|                         | win32k.sys     | 6.1.7601.23807 - KB4022719                | 6.2.9200.22168 - KB4022718                  | 6.3.9600.18698 - KB4022726          | 10.0.14393.594 - KB4022715                  | -                          | -                                           | -                                           |
|                         | rdpdd.dll      | 6.1.7601.23403 - KB3125574                | -                                           | -                                   | -                                           | -                          | -                                           | -                                           |
|                         | rdpwd.sys      | 6.1.7601.23403 - KB3125574                | -                                           | -                                   | -                                           | -                          | -                                           | -                                           |
| Seguridad                | MS17-010       | KB4012212                                 | KB4012213                                   | KB4012213                           | KB4012606                                   | KB4012606                  | -                                           | -                                           |
|                         |                |                                           | KB4012216                                   |                                     | KB4013198                                   | KB4013198                  | -                                           | -                                           |
|                         |                | KB4012215                                 | KB4012214                                   | KB4012216                           | KB4013429                                   | KB4013429                  | -                                           | -                                           |
|                         |                |                                           | KB4012217                                   |                                     | KB4013429                                   | KB4013429                  | -                                           | -                                           |
|                         | CVE-2018-0886  | KB4103718                                 | KB4103730                                   | KB4103725                           | KB4103723                                   | KB4103731                  | KB4103727                                   | KB4103721                                   |
|                         |                | KB4103712                                 | KB4103726                                   | KB4103715                           |                                             |                            |                                             |                                             |

> [!NOTE]
> Para evitar un reinicio accidental durante el aprovisionamiento de la máquina virtual, es aconsejable asegurarse de que todas las instalaciones de Windows Update han finalizado y de que no hay actualizaciones pendientes. Una manera de hacerlo es instalar todas las actualizaciones de Windows posibles y reiniciar una vez antes de ejecutar el comando `sysprep.exe`.

### <a name="determine-when-to-use-sysprep"></a>Determinar cuándo usar Sysprep

Herramienta de preparación del sistema (`sysprep.exe`) es un proceso que se puede ejecutar para restablecer una instalación de Windows.
Sysprep proporciona una experiencia original, al eliminar todos los datos personales y restablecer varios componentes.

`sysprep.exe` se suele ejecutar para crear una plantilla desde la que implementar muchas otras máquinas virtuales que tengan una configuración específica. Esta plantilla se conoce como *imagen generalizada*.

Para crear una sola máquina virtual desde un disco, no tiene que usar Sysprep, sino crear la máquina virtual desde una *imagen especializada*. Para más información sobre cómo crear una máquina virtual desde un disco especializado, vea:

- [Creación de una máquina virtual a partir de un disco especializado](create-vm-specialized.md)
- [Create a VM from a specialized VHD disk](./create-vm-specialized-portal.md) (Creación de una máquina virtual a partir de un disco duro virtual especializado)

Para crear una imagen generalizada, debe ejecutar Sysprep. Para información, vea [Uso de Sysprep: Introducción](/previous-versions/windows/it-pro/windows-xp/bb457073(v=technet.10)).

No todos los roles o aplicaciones instalados en un equipo basado en Windows admiten estas imágenes generalizadas; así que, antes de usar este procedimiento, asegúrese de que Sysprep admite el rol del equipo. Para más información, consulte [Sysprep Support for Server Roles](/windows-hardware/manufacture/desktop/sysprep-support-for-server-roles) (Compatibilidad de Sysprep con roles de servidor).

Específicamente, Sysprep requiere que las unidades se descifren por completo antes de la ejecución. Si ha habilitado el cifrado en la máquina virtual, deshabilítelo antes de ejecutar Sysprep.


### <a name="generalize-a-vhd"></a>Generalización de un VHD

>[!NOTE]
> Después de ejecutar `sysprep.exe` en los pasos siguientes, desactive la máquina virtual. No vuelva a activarla hasta que cree una imagen de ella en Azure.

1. Inicie sesión en la máquina virtual Windows.
1. Ejecute una sesión de PowerShell como administrador.
1. Elimine el directorio Panther (C:\Windows\Panther).
1. Cambie el directorio a `%windir%\system32\sysprep`. A continuación, ejecute `sysprep.exe`.
1. En el cuadro de diálogo **Herramienta de preparación del sistema**, seleccione **Iniciar la Configuración rápida (OOBE) del sistema** y asegúrese de que la casilla **Generalizar** está activada.

    ![Herramienta de preparación del sistema](media/prepare-for-upload-vhd-image/syspre.png)
1. En **Opciones de apagado**, seleccione **Apagar**.
1. Seleccione **Aceptar**.
1. Cuando Sysprep finalice, apague la máquina virtual. No use **Reiniciar** para apagar la máquina virtual.

Ahora el disco duro virtual está listo para cargarse. Para más información sobre cómo crear una máquina virtual desde un disco generalizado, vea [Carga de un VHD generalizado en Azure para crear una máquina virtual nueva](sa-upload-generalized.md).

>[!NOTE]
> No se admite un archivo *unattend.xml* personalizado, No obstante, se admite la propiedad **additionalUnattendContent**, que proporciona solo compatibilidad limitada para agregar las opciones de [microsoft-windows-shell-setup](/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup) en el archivo *unattend.xml* que usa el agente de aprovisionamiento de Azure. Por ejemplo, puede usar [additionalUnattendContent](/dotnet/api/microsoft.azure.management.compute.models.additionalunattendcontent?view=azure-dotnet) para agregar FirstLogonCommands y LogonCommands. Para más información, vea el [ejemplo de FirstLogonCommands de additionalUnattendContent](https://github.com/Azure/azure-quickstart-templates/issues/1407).

## <a name="complete-the-recommended-configurations"></a>Completar las configuraciones recomendadas

Los siguientes valores de configuración no afectan a la carga de discos duros virtuales. Sin embargo, se recomienda firmemente que los configure.

- Instale el [agente de máquina virtual de Azure](https://go.microsoft.com/fwlink/?LinkID=394789). A continuación, puede habilitar las extensiones de máquina virtual. Las extensiones de máquina virtual implementan la mayor parte de la funcionalidad crítica que es posible que quiera usar con las máquinas virtuales. Necesitará estas extensiones, por ejemplo, para restablecer contraseñas o configurar RDP. Para obtener más información, consulte [Información general del agente de máquina virtual de Azure](../extensions/agent-windows.md).
- Después de crear la máquina virtual en Azure, recomendamos que ponga el archivo de paginación en el *volumen de unidad temporal* para mejorar el rendimiento. Puede configurar la ubicación de archivos del siguiente modo:

  ```powershell
  Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management' -Name PagingFiles -Value 'D:\pagefile.sys' -Type MultiString -Force
  ```

  Si hay un disco de datos conectado a la máquina virtual, la letra de unidad del volumen de unidad temporal suele ser *D*. Esta designación podría ser diferente, dependiendo de la configuración y del número de unidades disponibles.

  - Se recomienda deshabilitar los bloqueadores de script que pueda incluir el software antivirus, Podrían interferir y bloquear los scripts del agente de aprovisionamiento de Windows que se ejecutan al implementar una nueva máquina virtual desde la imagen.

## <a name="next-steps"></a>Pasos siguientes

- [Carga de una imagen de máquina virtual de Windows en Azure para implementaciones de Resource Manager](upload-generalized-managed.md)
- [Solución de problemas de activación de máquinas virtuales Windows de Azure](../troubleshooting/troubleshoot-activation-problems.md)
