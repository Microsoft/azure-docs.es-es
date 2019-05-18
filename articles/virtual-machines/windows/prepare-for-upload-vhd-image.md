---
title: Preparación de un disco duro virtual de Windows para cargarlo en Azure | Microsoft Docs
description: Preparación de un VHD o un VHDX de Windows antes de cargarlo en Azure
services: virtual-machines-windows
documentationcenter: ''
author: glimoli
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 7802489d-33ec-4302-82a4-91463d03887a
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/11/2019
ms.author: genli
ms.openlocfilehash: 5ae0e7855db6bec9f48d2b9511f0d0626d883111
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65561351"
---
# <a name="prepare-a-windows-vhd-or-vhdx-to-upload-to-azure"></a>Preparación de un VHD o un VHDX de Windows antes de cargarlo en Azure

Antes de cargar un Windows máquinas virtuales (VM) locales de Microsoft Azure, debe preparar el disco duro virtual (VHD o VHDX). Azure admite la generación 1 y máquinas virtuales de generación 2 en formato de archivo VHD y dispone de un disco de tamaño fijo. El tamaño máximo permitido para los discos duros virtuales es de 1023 GB. Puede convertir una máquina virtual de generación 1 del sistema de archivos VHDX a VHD y de un disco de expansión dinámica a uno de tamaño fijo. Sin embargo, no puede cambiar la generación de una máquina virtual. Para obtener más información, consulte [debo crear una generación 1 o 2 máquinas virtuales de Hyper-V](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v) y [las máquinas virtuales de generación 2 en Azure](generation-2.md).

Para obtener información sobre la directiva de soporte de software de servidor de Microsoft ejecutado en Azure, consulte [Soporte de software de servidor de Microsoft para las máquinas virtuales de Microsoft Azure](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines).

> [!Note]
> Las instrucciones que aparecen en este artículo se aplican a la versión de 64 bits de Windows Server 2008 R2 y un sistema operativo Windows Server. Para obtener información sobre cómo ejecutar la versión de 32 bits del sistema operativo en Azure, consulte [Soporte para sistemas operativos de 32 bits en máquinas virtuales de Azure](https://support.microsoft.com/help/4021388/support-for-32-bit-operating-systems-in-azure-virtual-machines).

## <a name="convert-the-virtual-disk-to-vhd-and-fixed-size-disk"></a>Conversión del disco virtual a VHD y disco de tamaño fijo 
Si necesita convertir el disco virtual al formato requerido para Azure, utilice uno de los métodos de esta sección. Haga una copia de seguridad de la máquina virtual antes de ejecutar el proceso de conversión de disco virtual y asegúrese de que el VHD de Windows funciona correctamente en el servidor local. Resuelva los errores dentro de la propia máquina virtual antes de intentar la conversión o la carga en Azure.

Después de convertir el disco, cree una máquina virtual que use el disco convertido. Comience e inicie sesión en la máquina virtual a fin de preparar la máquina virtual para la carga.

### <a name="convert-disk-using-hyper-v-manager"></a>Conversión del disco mediante el Administrador de Hyper-V
1. Abra el Administrador de Hyper-V y seleccione el equipo local de la izquierda. En el menú situado sobre la lista de equipos, haga clic en **Acción** > **Editar disco**.
2. En la pantalla **Locate Virtual Hard Disk** (Localizar disco duro virtual), localice y seleccione el disco virtual.
3. En la pantalla **Elegir acción**, seleccione **Convertir** y **Siguiente**.
4. Si necesita realizar la conversión desde VHDX, seleccione **VHD** y, después, haga clic en **Siguiente**.
5. Si necesita realizar la conversión desde un disco de expansión dinámica, seleccione **Tamaño fijo** y, después, haga clic en **Siguiente**.
6. Localice y seleccione una ruta de acceso para guardar el nuevo archivo VHD en ella.
7. Haga clic en **Finalizar**

>[!NOTE]
>Los comandos de este artículo se deben ejecutar en una sesión de PowerShell con privilegios elevados.

### <a name="convert-disk-by-using-powershell"></a>Conversión de disco con PowerShell
Puede convertir un disco virtual mediante el comando [Convert-VHD](https://technet.microsoft.com/library/hh848454.aspx) en Windows PowerShell. Seleccione **Ejecutar como administrador** al iniciar PowerShell. 

El comando de ejemplo siguiente convierte de VHDX a VHD y de un disco de expansión dinámica a uno de tamaño fijo:

```Powershell
Convert-VHD –Path c:\test\MY-VM.vhdx –DestinationPath c:\test\MY-NEW-VM.vhd -VHDType Fixed
```
En este comando, reemplace el valor de "-Path" por la ruta de acceso al disco duro virtual que desea convertir y el valor de "-DestinationPath" por la nueva ruta de acceso y el nombre del disco convertido.

### <a name="convert-from-vmware-vmdk-disk-format"></a>Conversión del formato de disco VMDK de VMware
Si tiene una imagen de máquina virtual Windows en el [formato de archivo VMDK](https://en.wikipedia.org/wiki/VMDK), conviértalo en un disco duro virtual con [Microsoft Virtual Machine Converter](https://www.microsoft.com/download/details.aspx?id=42497). Consulte el artículo [How to Convert a VMWare VMDK to Hyper-V VHD](https://blogs.msdn.com/b/timomta/archive/2015/06/11/how-to-convert-a-vmware-vmdk-to-hyper-v-vhd.aspx) (Cómo convertir un VMDK de VMWare a VHD de Hyper-V) del blog para obtener más información.

## <a name="set-windows-configurations-for-azure"></a>Establecimiento de configuraciones de Windows para Azure

En la máquina virtual que tiene previsto cargar en Azure, ejecute todos los comandos en los pasos siguientes desde una [ventana del símbolo del sistema con privilegios elevados](https://technet.microsoft.com/library/cc947813.aspx):

1. Quite cualquier ruta estática persistente de la tabla de enrutamiento:
   
   * Para ver la tabla de rutas, ejecute `route print` en el símbolo del sistema.
   * Compruebe las secciones **Persistence Routes** (Rutas de persistencia). Si hay una ruta persistente, use el comando **route delete** para quitarla.
2. Quite al proxy WinHTTP:
   
    ```PowerShell
    netsh winhttp reset proxy
    ```

    Si la máquina virtual necesita trabajar con algún proxy concreto, debe agregar una excepción de proxy a la dirección IP de Azure ([168.63.129.16](https://blogs.msdn.microsoft.com/mast/2015/05/18/what-is-the-ip-address-168-63-129-16/
)), con el fin de que la máquina virtual tenga conectividad con Azure:
    ```
    $proxyAddress="<your proxy server>"
    $proxyBypassList="<your list of bypasses>;168.63.129.16"

    netsh winhttp set proxy $proxyAddress $proxyBypassList
    ```

3. Configure la directiva SAN del disco como [Onlineall](https://technet.microsoft.com/library/gg252636.aspx):
   
    ```PowerShell
    diskpart 
    ```
    En la ventana del símbolo del sistema abierta, escriba los comandos siguientes:

     ```DISKPART
    san policy=onlineall
    exit   
    ```

4. Establezca la hora universal coordinada (UTC) para Windows y el tipo de inicio del Servicio de hora de Windows (w32time) en **Automáticamente**:
   
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\TimeZoneInformation' -name "RealTimeIsUniversal" -Value 1 -Type DWord -force

    Set-Service -Name w32time -StartupType Automatic
    ```
5. Establezca el perfil de energía en **Alto rendimiento**:

    ```PowerShell
    powercfg /setactive SCHEME_MIN
    ```
6. Asegúrese de que las variables de entorno **TEMP** y **TMP** se establecen en sus valores predeterminados:

    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment' -name "TEMP" -Value "%SystemRoot%\TEMP" -Type ExpandString -force

    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment' -name "TMP" -Value "%SystemRoot%\TEMP" -Type ExpandString -force
    ```

## <a name="check-the-windows-services"></a>Comprobación de los servicios de Windows
Asegúrese de que cada uno de los siguientes servicios de Windows esté establecido en los **valores predeterminados de Windows**. Esta es la cantidad de servicios mínima que debe configurarse para garantizar que la máquina virtual tenga conectividad. Para restablecer la configuración de inicio, ejecute los siguientes comandos:
   
```PowerShell
Set-Service -Name bfe -StartupType Automatic
Set-Service -Name dhcp -StartupType Automatic
Set-Service -Name dnscache -StartupType Automatic
Set-Service -Name IKEEXT -StartupType Automatic
Set-Service -Name iphlpsvc -StartupType Automatic
Set-Service -Name netlogon -StartupType Manual
Set-Service -Name netman -StartupType Manual
Set-Service -Name nsi -StartupType Automatic
Set-Service -Name termService -StartupType Manual
Set-Service -Name MpsSvc -StartupType Automatic
Set-Service -Name RemoteRegistry -StartupType Automatic
```

## <a name="update-remote-desktop-registry-settings"></a>Actualización de la configuración del Registro de Escritorio remoto
Asegúrese de que la siguiente configuración está establecida correctamente para la conexión a Escritorio remoto:

>[!Note] 
>Puede recibir un mensaje de error al ejecutar **Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services -name &lt;nombre de objeto&gt; -value &lt;valor&gt;** en estos pasos. El mensaje de error puede pasarse por alto de forma segura. Solo significa que el dominio no obliga a esa configuración a aceptar un objeto de directiva de grupo.
>
>

1. Protocolo de escritorio remoto (RDP) está habilitado:
   
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0 -Type DWord -force

    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -name "fDenyTSConnections" -Value 0 -Type DWord -force
    ```
   
2. El puerto de RDP está configurado correctamente (puerto 3389 predeterminado):
   
    ```PowerShell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "PortNumber" -Value 3389 -Type DWord -force
    ```
    Al implementar una máquina virtual, las reglas predeterminadas se crean en el puerto 3389. Si desea cambiar el número de puerto, hágalo una vez implementada la máquina virtual en Azure.

3. El agente de escucha está escuchando en todas las interfaces de red:
   
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "LanAdapter" -Value 0 -Type DWord -force
   ```
4. Configure el modo Autenticación a nivel de red para las conexiones RDP:
   
    ```PowerShell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1 -Type DWord -force

    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "SecurityLayer" -Value 1 -Type DWord -force

    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "fAllowSecProtocolNegotiation" -Value 1 -Type DWord -force
     ```

5. Establezca el valor de conexión persistente:
    
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -name "KeepAliveEnable" -Value 1  -Type DWord -force
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -name "KeepAliveInterval" -Value 1  -Type DWord -force
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "KeepAliveTimeout" -Value 1 -Type DWord -force
    ```
6. Vuelva a conectarse:
    
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -name "fDisableAutoReconnect" -Value 0 -Type DWord -force
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "fInheritReconnectSame" -Value 1 -Type DWord -force
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "fReconnectSame" -Value 0 -Type DWord -force
    ```
7. Limite el número de conexiones simultáneas:
    
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "MaxInstanceCount" -Value 4294967295 -Type DWord -force
    ```
8. Si hay algún certificado autofirmado enlazado al agente de escucha del Protocolo de escritorio remoto, quítelo:
    
    ```PowerShell
    Remove-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "SSLCertificateSHA1Hash" -force
    ```
    Esto es para asegurarse de que puede conectarse al principio al implementar la máquina virtual. También puede revisar esto más tarde una vez implementada la máquina virtual en Azure en caso necesario.

9. Si la máquina virtual va a formar parte de un dominio, compruebe toda la configuración siguiente para asegurarse de que la configuración anterior no se revierte. Las directivas que deben comprobarse son los siguientes:
    
    | Objetivo                                     | Directiva                                                                                                                                                       | Valor                                                                                    |
    |------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
    | RDP está habilitado                           | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\ Componentes\Servicios de Escritorio remoto\Host de sesión de Escritorio remoto\Conexiones         | Permitir a los usuarios conectarse de forma remota desde el Escritorio remoto                                  |
    | Directiva de grupo de NLA                         | Configuración\Plantillas administrativas\Componentes\Servicios de Escritorio remoto\Host de sesión de Escritorio remoto\Seguridad                                                    | Exigir la autenticación de usuarios para conexiones remotas mediante Autenticación a nivel de red |
    | Configuración de conexión persistente                      | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\ Componentes Windows\Servicios de Escritorio remoto\Host de sesión de Escritorio remoto\Conexiones | Configurar el intervalo de conexión persistente                                                 |
    | Configuración de reconexión                       | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\ Componentes Windows\Servicios de Escritorio remoto\Host de sesión de Escritorio remoto\Conexiones | Reconexión automática                                                                   |
    | Limitar el número de valores de configuración de las conexiones | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\ Componentes Windows\Servicios de Escritorio remoto\Host de sesión de Escritorio remoto\Conexiones | Limitar el número de conexiones                                                              |

## <a name="configure-windows-firewall-rules"></a>Configuración de reglas de firewall de Windows
1. Active Firewall de Windows en los tres perfiles (Dominio, Estándar y Público):

   ```PowerShell
    Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
   ```

2. Ejecute el comando siguiente en PowerShell para permitir WinRM mediante los tres perfiles de firewall (Dominio, Privado y Público) y habilitar el servicio remoto de PowerShell:
   
   ```PowerShell
    Enable-PSRemoting -force

    Set-NetFirewallRule -DisplayName "Windows Remote Management (HTTP-In)" -Enabled True
   ```
3. Habilite las siguientes reglas de firewall para permitir el tráfico RDP:

   ```PowerShell
    Set-NetFirewallRule -DisplayGroup "Remote Desktop" -Enabled True
   ```   
4. Habilite la regla Compartir archivos e impresoras para que la máquina virtual pueda responder a un comando ping dentro de la red virtual:

   ```PowerShell
   Set-NetFirewallRule -DisplayName "File and Printer Sharing (Echo Request - ICMPv4-In)" -Enabled True
   ``` 
5. Si la máquina virtual va a formar parte de un dominio, compruebe la configuración siguiente para asegurarse de que la configuración anterior no se revierte. Las directivas de AD que deben comprobarse son las siguientes:

    | Objetivo                                 | Directiva                                                                                                                                                  | Valor                                   |
    |--------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------|
    | Habilitar los perfiles de Firewall de Windows | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\Red\Conexión de red\Firewall de Windows\Perfil de dominio\Firewall de Window   | Proteger todas las conexiones de red         |
    | Habilitar RDP                           | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\Red\Conexión de red\Firewall de Windows\Perfil de dominio\Firewall de Window   | Permitir excepciones de Escritorio remoto entrantes |
    |                                      | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\Red\Conexión de red\Firewall de Windows\Perfil estándar\Firewall de Windows | Permitir excepciones de Escritorio remoto entrantes |
    | Habilitar ICMP-V4                       | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\Red\Conexión de red\Firewall de Windows\Perfil de dominio\Firewall de Window   | Permitir excepciones de ICMP                   |
    |                                      | Configuración del equipo\Directivas\Configuración de Windows\Plantillas administrativas\Red\Conexión de red\Firewall de Windows\Perfil estándar\Firewall de Windows | Permitir excepciones de ICMP                   |

## <a name="verify-vm-is-healthy-secure-and-accessible-with-rdp"></a>Comprobación de que la que máquina virtual es correcta, está segura y es accesible con RDP 
1. Para garantizar que el disco sea coherente y esté en buen estado, ejecute una operación de comprobación de disco en el próximo reinicio de máquina virtual:

    ```PowerShell
    Chkdsk /f
    ```
    Asegúrese de que el informe muestra un disco limpio y en buen estado.

2. Establezca la configuración de datos de la configuración de arranque (BCD). 

    > [!Note]
    > Asegúrese de que ejecuta estos comandos en una ventana de PowerShell con privilegios elevados.
   
   ```powershell
    cmd

    bcdedit /set {bootmgr} integrityservices enable
    bcdedit /set {default} device partition=C:
    bcdedit /set {default} integrityservices enable
    bcdedit /set {default} recoveryenabled Off
    bcdedit /set {default} osdevice partition=C:
    bcdedit /set {default} bootstatuspolicy IgnoreAllFailures

    #Enable Serial Console Feature
    bcdedit /set {bootmgr} displaybootmenu yes
    bcdedit /set {bootmgr} timeout 5
    bcdedit /set {bootmgr} bootems yes
    bcdedit /ems {current} ON
    bcdedit /emssettings EMSPORT:1 EMSBAUDRATE:115200

    exit
   ```
3. El registro de volcado de memoria puede ser útil para solucionar problemas de bloqueo de Windows. Habilite la colección de registros de volcado de memoria:

    ```powershell
    # Setup the Guest OS to collect a kernel dump on an OS crash event
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -name CrashDumpEnabled -Type DWord -force -Value 2
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -name DumpFile -Type ExpandString -force -Value "%SystemRoot%\MEMORY.DMP"
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -name NMICrashDump -Type DWord -force -Value 1

    #Setup the Guest OS to collect user mode dumps on a service crash event
    $key = 'HKLM:\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps'
    if ((Test-Path -Path $key) -eq $false) {(New-Item -Path 'HKLM:\SOFTWARE\Microsoft\Windows\Windows Error Reporting' -Name LocalDumps)}
    New-ItemProperty -Path $key -name DumpFolder -Type ExpandString -force -Value "c:\CrashDumps"
    New-ItemProperty -Path $key -name CrashCount -Type DWord -force -Value 10
    New-ItemProperty -Path $key -name DumpType -Type DWord -force -Value 2
    Set-Service -Name WerSvc -StartupType Manual
    ```
4. Compruebe que el repositorio de Instrumental de administración de Windows es coherente. Para ello, ejecute el siguiente comando:

    ```PowerShell
    winmgmt /verifyrepository
    ```
    Si el repositorio está dañado, consulte [WMI: Repository Corruption, or Not](https://blogs.technet.microsoft.com/askperf/2014/08/08/wmi-repository-corruption-or-not) (WMI: daños en el repositorio, o no).

5. Asegúrese de que ninguna otra aplicación está utilizando el puerto 3389. Este puerto se usa para el servicio RDP en Azure. Puede ejecutar **netstat -anob** para ver qué puertos se utilizan en la máquina virtual:

    ```PowerShell
    netstat -anob
    ```

6. Si el disco duro virtual de Windows que quiere cargar es un controlador de dominio, siga estos pasos:

    1. Siga [estos pasos adicionales](https://support.microsoft.com/kb/2904015) para preparar el disco.

    1. Asegúrese de que conoce la contraseña de DSRM en caso de tener que iniciar la máquina virtual en DSRM en algún momento. Puede que desee hacer referencia a este vínculo para establecer la [contraseña de DSRM](https://technet.microsoft.com/library/cc754363(v=ws.11).aspx).

7. Asegúrese de que conoce la contraseña y la cuenta predefinida de administrador. Es posible que quiera restablecer la contraseña de administrador local actual y asegurarse de que puede usar esta cuenta para iniciar sesión en Windows mediante la conexión RDP. Este permiso de acceso se controla mediante el objeto de directiva de grupo "Permitir inicio de sesión a través de Servicios de Escritorio remoto". Puede ver este objeto en el Editor de directivas de grupo local en:

    Configuración del equipo\Configuración de Windows\Configuración de seguridad\Directivas locales\Asignación de derechos de usuario

8. Compruebe las siguientes directivas de AD para asegurarse de que no bloquea su acceso de RDP a través de RDP ni desde la red:

    - Configuración del equipo\Configuración de Windows\Configuración de seguridad\Directivas locales\Asignación de derechos de usuario\Denegar el acceso a este equipo desde la red

    - Configuración del equipo\Configuración de Windows\Configuración de seguridad\Directivas locales\Asignación de derechos de usuario\Denegar inicio de sesión a través de Servicios de Escritorio remoto


9. Compruebe la siguiente directiva de AD para asegurarse de que no se quita cualquiera de las siguientes cuentas de acceso obligatorias:

   - Configuración del equipo\Configuración de Windows\Configuración de seguridad\Directivas locales\Asignación de derechos de usuario\Acceder a este equipo desde la red

     Los siguientes grupos deben aparecen en esta directiva:

   - Administradores
   - Operadores de copias de seguridad
   - Todos
   - Usuarios

10. Reinicie la máquina virtual para asegurarse de que Windows funciona aún correctamente y se puede conectar a él mediante RDP. En este momento, puede que desee crear una máquina virtual en el Hyper-V local para asegurarse de que esta se inicia completamente y, a continuación, probar si es accesible desde RDP.

11. Quite los filtros adicionales de la Interfaz de controlador de transporte, como el software que analiza los paquetes TCP o los firewalls adicionales. También puede revisar esto más tarde una vez implementada la máquina virtual en Azure en caso necesario.

12. Desinstale cualquier otro software de terceros y controlador relacionado con componentes físicos o cualquier otra tecnología de virtualización.

### <a name="install-windows-updates"></a>Instalación de actualizaciones de Windows
La configuración ideal es **tener el nivel de revisión de la máquina en la versión más reciente**. Si no es posible, asegúrese de que las siguientes actualizaciones están instaladas:

| Componente               | Binary         | Windows 7 SP1 y Windows Server 2008 R2 SP1 | Windows 8 y Windows Server 2012               | Windows 8.1 y Windows Server 2012 R2 | Windows Server 2016 versión 1607 de Windows 10 versión 1607 | Windows 10 Versión 1703    | Windows 10, versión 1709 y Windows Server 2016, versión 1709 | Windows 10 1803 Windows Server 2016 versión 1803 |
|-------------------------|----------------|-------------------------------------------|---------------------------------------------|------------------------------------|---------------------------------------------------------|----------------------------|-------------------------------------------------|-------------------------------------------------|
| Almacenamiento                 | disk.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17638 / 6.2.9200.21757 - KB3137061 | 6.3.9600.18203 - KB3137061         | -                                                       | -                          | -                                               | -                                               |
|                         | storport.sys   | 6.1.7601.23403 - KB3125574                | 6.2.9200.17188 / 6.2.9200.21306 - KB3018489 | 6.3.9600.18573 - KB4022726         | 10.0.14393.1358 - KB4022715                             | 10.0.15063.332             | -                                               | -                                               |
|                         | ntfs.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17623 / 6.2.9200.21743 - KB3121255 | 6.3.9600.18654 - KB4022726         | 10.0.14393.1198 - KB4022715                             | 10.0.15063.447             | -                                               | -                                               |
|                         | Iologmsg.dll   | 6.1.7601.23403 - KB3125574                | 6.2.9200.16384 - KB2995387                  | -                                  | -                                                       | -                          | -                                               | -                                               |
|                         | Classpnp.sys   | 6.1.7601.23403 - KB3125574                | 6.2.9200.17061 / 6.2.9200.21180 - KB2995387 | 6.3.9600.18334 - KB3172614         | 10.0.14393.953 - KB4022715                              | -                          | -                                               | -                                               |
|                         | Volsnap.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.17047 / 6.2.9200.21165 - KB2975331 | 6.3.9600.18265 - KB3145384         | -                                                       | 10.0.15063.0               | -                                               | -                                               |
|                         | partmgr.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.16681 - KB2877114                  | 6.3.9600.17401 - KB3000850         | 10.0.14393.953 - KB4022715                              | 10.0.15063.0               | -                                               | -                                               |
|                         | volmgr.sys     |                                           |                                             |                                    |                                                         | 10.0.15063.0               | -                                               | -                                               |
|                         | Volmgrx.sys    | 6.1.7601.23403 - KB3125574                | -                                           | -                                  | -                                                       | 10.0.15063.0               | -                                               | -                                               |
|                         | Msiscsi.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.21006 - KB2955163                  | 6.3.9600.18624 - KB4022726         | 10.0.14393.1066 - KB4022715                             | 10.0.15063.447             | -                                               | -                                               |
|                         | Msdsm.sys      | 6.1.7601.23403 - KB3125574                | 6.2.9200.21474 - KB3046101                  | 6.3.9600.18592 - KB4022726         | -                                                       | -                          | -                                               | -                                               |
|                         | Mpio.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.21190 - KB3046101                  | 6.3.9600.18616 - KB4022726         | 10.0.14393.1198 - KB4022715                             | -                          | -                                               | -                                               |
|                         | vmstorfl.sys   | 6.3.9600.18907 - KB4072650                | 6.3.9600.18080 - KB3063109                  | 6.3.9600.18907 - KB4072650         | 10.0.14393.2007 - KB4345418                             | 10.0.15063.850 - KB4345419 | 10.0.16299.371 - KB4345420                      | -                                               |
|                         | Fveapi.dll     | 6.1.7601.23311 - KB3125574                | 6.2.9200.20930 - KB2930244                  | 6.3.9600.18294 - KB3172614         | 10.0.14393.576 - KB4022715                              | -                          | -                                               | -                                               |
|                         | Fveapibase.dll | 6.1.7601.23403 - KB3125574                | 6.2.9200.20930 - KB2930244                  | 6.3.9600.17415 - KB3172614         | 10.0.14393.206 - KB4022715                              | -                          | -                                               | -                                               |
| Red                 | netvsc.sys     | -                                         | -                                           | -                                  | 10.0.14393.1198 - KB4022715                             | 10.0.15063.250 - KB4020001 | -                                               | -                                               |
|                         | mrxsmb10.sys   | 6.1.7601.23816 - KB4022722                | 6.2.9200.22108 - KB4022724                  | 6.3.9600.18603 - KB4022726         | 10.0.14393.479 - KB4022715                              | 10.0.15063.483             | -                                               | -                                               |
|                         | mrxsmb20.sys   | 6.1.7601.23816 - KB4022722                | 6.2.9200.21548 - KB4022724                  | 6.3.9600.18586 - KB4022726         | 10.0.14393.953 - KB4022715                              | 10.0.15063.483             | -                                               | -                                               |
|                         | mrxsmb.sys     | 6.1.7601.23816 - KB4022722                | 6.2.9200.22074 - KB4022724                  | 6.3.9600.18586 - KB4022726         | 10.0.14393.953 - KB4022715                              | 10.0.15063.0               | -                                               | -                                               |
|                         | tcpip.sys      | 6.1.7601.23761 - KB4022722                | 6.2.9200.22070 - KB4022724                  | 6.3.9600.18478 - KB4022726         | 10.0.14393.1358 - KB4022715                             | 10.0.15063.447             | -                                               | -                                               |
|                         | http.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17285 - KB3042553                  | 6.3.9600.18574 - KB4022726         | 10.0.14393.251 - KB4022715                              | 10.0.15063.483             | -                                               | -                                               |
|                         | vmswitch.sys   | 6.1.7601.23727 - KB4022719                | 6.2.9200.22117 - KB4022724                  | 6.3.9600.18654 - KB4022726         | 10.0.14393.1358 - KB4022715                             | 10.0.15063.138             | -                                               | -                                               |
| Núcleo                    | ntoskrnl.exe   | 6.1.7601.23807 - KB4022719                | 6.2.9200.22170 - KB4022718                  | 6.3.9600.18696 - KB4022726         | 10.0.14393.1358 - KB4022715                             | 10.0.15063.483             | -                                               | -                                               |
| Servicios de Escritorio remoto | rdpcorets.dll  | 6.2.9200.21506 - KB4022719                | 6.2.9200.22104 - KB4022724                  | 6.3.9600.18619 - KB4022726         | 10.0.14393.1198 - KB4022715                             | 10.0.15063.0               | -                                               | -                                               |
|                         | termsrv.dll    | 6.1.7601.23403 - KB3125574                | 6.2.9200.17048 - KB2973501                  | 6.3.9600.17415 - KB3000850         | 10.0.14393.0 - KB4022715                                | 10.0.15063.0               | -                                               | -                                               |
|                         | termdd.sys     | 6.1.7601.23403 - KB3125574                | -                                           | -                                  | -                                                       | -                          | -                                               | -                                               |
|                         | win32k.sys     | 6.1.7601.23807 - KB4022719                | 6.2.9200.22168 - KB4022718                  | 6.3.9600.18698 - KB4022726         | 10.0.14393.594 - KB4022715                              | -                          | -                                               | -                                               |
|                         | rdpdd.dll      | 6.1.7601.23403 - KB3125574                | -                                           | -                                  | -                                                       | -                          | -                                               | -                                               |
|                         | rdpwd.sys      | 6.1.7601.23403 - KB3125574                | -                                           | -                                  | -                                                       | -                          | -                                               | -                                               |
| Seguridad                | MS17-010       | KB4012212                                 | KB4012213                                   | KB4012213                          | KB4012606                                               | KB4012606                  | -                                               | -                                               |
|                         |                |                                           | KB4012216                                   |                                    | KB4013198                                               | KB4013198                  | -                                               | -                                               |
|                         |                | KB4012215                                 | KB4012214                                   | KB4012216                          | KB4013429                                               | KB4013429                  | -                                               | -                                               |
|                         |                |                                           | KB4012217                                   |                                    | KB4013429                                               | KB4013429                  | -                                               | -                                               |
|                         | CVE-2018-0886  | KB4103718               | KB4103730                | KB4103725       | KB4103723                                               | KB4103731                  | KB4103727                                       | KB4103721                                       |
|                         |                | KB4103712          | KB4103726          | KB4103715|                                                         |                            |                                                 |                                                 |
       
### Cuándo usar sysprep <a id="step23"></a>    

Sysprep es un proceso que podría ejecutar en una instalación de Windows que restablecerá la instalación del sistema y proporcionará una “configuración rápida” quitando todos los datos personales y restableciendo varios componentes. Suele hacer esto si desea crear una plantilla desde la que pueda implementar muchas otras máquinas virtuales que tengan una configuración específica. Esto se conoce como **imagen generalizada**.

Si, en su lugar, solo desea crear una máquina virtual desde un disco, no tiene que usar sysprep. En esta situación, puede limitarse a crear la máquina virtual desde lo que se conoce como **imagen especializada**.

Para obtener más información sobre cómo crear una máquina virtual desde un disco especializado, consulte:

- [Creación de una máquina virtual a partir de un disco especializado](create-vm-specialized.md)
- [Create a VM from a specialized VHD disk](https://docs.microsoft.com/azure/virtual-machines/windows/create-vm-specialized-portal?branch=master) (Creación de una máquina virtual a partir de un disco duro virtual especializado)

Si desea crear una imagen generalizada, debe ejecutar sysprep. Para más información acerca de Sysprep, consulte [Uso de Sysprep: Introducción](https://technet.microsoft.com/library/bb457073.aspx). 

No todos los roles o aplicaciones instalados en un equipo basado en Windows admiten esta generalización. Así pues, antes de ejecutar este procedimiento, consulte el siguiente artículo para asegurarse de que sysprep admite el rol de ese equipo. Para obtener más información, consulte [Sysprep Support for Server Roles](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles) (Compatibilidad de Sysprep con los roles de servidor).

### <a name="steps-to-generalize-a-vhd"></a>Pasos para generalizar un disco duro virtual

>[!NOTE]
> Después de ejecutar sysprep.exe tal como se especifica en los pasos siguientes, desactive la máquina virtual y no vuelva a activarla hasta que cree una imagen de ella en Azure.

1. Inicie sesión en la máquina virtual Windows.
2. Ejecute **Símbolo del sistema** como administrador. 
3. Cambie el directorio a: **%windir%\system32\sysprep** y, a continuación, ejecute **sysprep.exe**.
3. En **Herramienta de preparación del sistema**, seleccione **Iniciar la Configuración rápida (OOBE)** y asegúrese de que la casilla **Generalizar** está activada.

    ![Herramienta de preparación del sistema](media/prepare-for-upload-vhd-image/syspre.png)
4. En **Opciones de apagado**, seleccione **Apagar**.
5. Haga clic en **OK**.
6. Cuando finalice Sysprep, apague la máquina virtual. No use **Reiniciar** para apagar la máquina virtual.
7. Ahora el disco duro virtual está listo para cargarse. Para obtener más información sobre cómo crear una máquina virtual desde un disco generalizado, consulte [Carga de un VHD generalizado en Azure para crear una máquina virtual nueva](sa-upload-generalized.md).


>[!NOTE]
> No se admite un archivo unattend.xml personalizado. Aunque se admite la propiedad additionalUnattendContent, solo proporciona compatibilidad limitada para agregar las opciones de [microsoft-windows-shell-setup](https://docs.microsoft.com/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup) en el archivo unattend.xml que utiliza el agente de aprovisionamiento de Azure. Por ejemplo,  pueden usar [additionalUnattendContent](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.compute.models.additionalunattendcontent?view=azure-dotnet) para agregar FirstLogonCommands y LogonCommands. Consulte también el [ejemplo de additionalUnattendContent FirstLogonCommands](https://github.com/Azure/azure-quickstart-templates/issues/1407).


## <a name="complete-recommended-configurations"></a>Realice las configuraciones recomendadas
Los siguientes valores de configuración no afectan a la carga de discos duros virtuales. Sin embargo, se recomienda firmemente que los configure.

* Instale el [agente de máquina virtual de Azure](https://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). A continuación, puede habilitar las extensiones de máquina virtual. Las extensiones de máquina virtual implementan la mayor parte de la funcionalidad crítica que es posible que quiera usar con las máquinas virtuales, como el restablecimiento de contraseñas, la configuración de RDP, etc. Para obtener más información, consulte [información general del agente de máquina Virtual de Azure](../extensions/agent-windows.md).
* Después de crearse la máquina virtual en Azure, recomendamos que ponga el archivo de paginación en el volumen "Unidad temporal" para mejorar el rendimiento. Puede configurar esto como se muestra a continuación:

   ```PowerShell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management' -name "PagingFiles" -Value "D:\pagefile.sys" -Type MultiString -force
   ```
  Si hay algún disco de datos conectado a la máquina virtual, la letra de unidad del volumen Unidad temporal suele ser "D". Esta designación podría ser diferente, dependiendo del número de unidades disponibles y de la configuración creada.

## <a name="next-steps"></a>Pasos siguientes
* [Carga de una imagen de máquina virtual de Windows en Azure para implementaciones de Resource Manager](upload-generalized-managed.md)
* [Solución de problemas de activación de máquinas virtuales Windows de Azure](troubleshoot-activation-problems.md)

