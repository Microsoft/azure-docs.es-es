---
title: Solucionar problemas de conexión de RDP de máquinas virtuales de Azure en función de los identificadores de eventos | Microsoft Docs
description: Use los identificadores de evento para solucionar diversos problemas que impiden realizar una conexión del protocolo de Escritorio remoto (RDP) a una máquina virtual (VM) de Azure.
services: virtual-machines-windows
documentationcenter: ''
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: azurecli
ms.date: 11/01/2018
ms.author: delhan
ms.openlocfilehash: 507cd6cfe9f251dbc304b579d634ff986b001264
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088604"
---
# <a name="troubleshoot-azure-vm-rdp-connection-issues-by-event-id"></a>Solucionar problemas de conexión de RDP de máquinas virtuales de Azure en función de los identificadores de eventos 

En este artículo se explica cómo usar los identificadores de eventos para solucionar problemas que impiden realizar una conexión del protocolo de Escritorio remoto (RDP) a una máquina virtual de Azure (VM).

## <a name="symptoms"></a>Síntomas

Está intentando usar una sesión del protocolo de Escritorio remoto (RDP) para conectarse a una máquina virtual de Azure. Después de escribir las credenciales, la conexión falla y recibe el siguiente mensaje de error:

**Este equipo no se puede conectar al equipo remoto. Intente conectarse de nuevo y, si el problema continúa, póngase en contacto con el propietario del equipo remoto o con el administrador de red.**

Para solucionar este problema, revise los registros de eventos en la máquina virtual y consulte los siguientes escenarios que le indicamos.

## <a name="before-you-troubleshoot"></a>Antes de iniciar los pasos de la solución

### <a name="create-a-backup-snapshot"></a>Cree una instantánea de copia de seguridad

Para crear una instantánea de copia de seguridad, siga los pasos descritos en [Instantánea de un disco](../windows/snapshot-copy-managed-disk.md).

### <a name="connect-to-the-vm-remotely"></a>Conéctese a la máquina virtual de forma remota

Para conectarse a la máquina virtual de forma remota, use uno de los métodos que se detallan en [How to use remote tools to troubleshoot Azure VM issues](remote-tools-troubleshoot-azure-vm-issues.md) (Cómo usar herramientas remotas para solucionar problemas con la máquina virtual de Azure).

## <a name="scenario-1"></a>Escenario 1

### <a name="event-logs"></a>Registros de eventos

En una instancia de CMD, ejecute los siguientes comandos para comprobar si el evento 1058 o el evento 1057 se anotó en el registro del sistema en las últimas 24 horas:

```cmd
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-TerminalServices-RemoteConnectionManager'] and EventID=1058 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-TerminalServices-RemoteConnectionManager'] and EventID=1057 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
```

**Nombre de registro**:      Sistema <br />
**Origen**:        Microsoft-Windows-TerminalServices-RemoteConnectionManager <br />
**Fecha:**          *hora* <br />
**Id. de evento**:      1058 <br />
**Categoría de tarea**: None <br />
**Nivel**:         Error <br />
**Palabras clave:**      Clásico <br />
**Usuario**:          N/D <br />
**Equipo:**      *equipo* <br />
**Descripción:** El servidor host de sesión de Escritorio remoto no pudo reemplazar el certificado autofirmado que expiró y que se usó para autenticar el servidor host de sesión de Escritorio remoto en las conexiones TLS. El código de estado correspondiente era Access y fue denegado.

**Nombre de registro**:      Sistema <br />
**Origen**:        Microsoft-Windows-TerminalServices-RemoteConnectionManager <br />
**Fecha:**          *hora* <br />
**Id. de evento**:      1058 <br />
**Categoría de tarea**: None <br />
**Nivel**:         Error <br />
**Palabras clave:**      Clásico <br />
**Usuario**:          N/D <br />
**Equipo:**      *equipo* <br />
**Descripción:** el servidor host de sesión de Escritorio remoto no pudo crear el certificado autofirmado que se usará en la autenticación del servidor host de sesión de Escritorio remoto en las conexiones TLS. El código de estado correspondiente era un objeto que ya existe.

**Nombre de registro**:      Sistema <br />
**Origen**:        Microsoft-Windows-TerminalServices-RemoteConnectionManager <br />
**Fecha:**          *hora* <br />
**Id. de evento**:      1057 <br />
**Categoría de tarea**: None <br />
**Nivel**:         Error <br />
**Palabras clave:**      Clásico <br />
**Usuario**:          N/D <br />
**Equipo:**      *equipo* <br />
**Descripción:** el servidor host de sesión de Escritorio remoto no pudo crear el certificado autofirmado que se usará en la autenticación del servidor host de sesión de Escritorio remoto en las conexiones TLS. El código de estado correspondiente era Keyset y no existe.

También puede comprobar los eventos de error SCHANNEL 36872 y 36870 ejecutando los comandos siguientes:

```cmd
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Schannel'] and EventID=36870 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Schannel'] and EventID=36872 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
```

**Nombre de registro**:      Sistema <br />
**Origen**:        Schannel <br />
**Fecha**          — <br />
**Id. de evento**:      36870 <br />
**Categoría de tarea**: None <br />
**Nivel**:         Error <br />
**Palabras clave:**       <br />
**Usuario**:          SYSTEM <br />
**Equipo:**      *equipo* <br />
**Descripción:** se produjo un error grave al intentar acceder a la clave privada de las credenciales del servidor TLS. El código de error devuelto del módulo criptográfico es 0x8009030D.  <br />
El estado del error interno es 10001.

### <a name="cause"></a>Causa
Este problema se produce porque no se puede obtener acceso a las claves de cifrado RSA locales en la carpeta MachineKeys de la máquina virtual. Este problema puede ocurrir debido a uno de los siguientes motivos:

1. La configuración de los permisos en la carpeta Machinekeys o en los archivos RSA no es correcta.

2. La clave RSA está dañada o falta.

### <a name="resolution"></a>Solución

Para solucionar este problema, debe configurar los permisos correctos en el certificado RDP siguiendo estos pasos.

#### <a name="grant-permission-to-the-machinekeys-folder"></a>Otorgue permiso a la carpeta MachineKeys

1. Cree un script mediante el siguiente contenido:

   ```powershell
   remove-module psreadline 
   icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\BeforeScript_permissions.txt
   takeown /f "C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys" /a /r
   icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\System:(F)"
   icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\NETWORK SERVICE:(R)"
   icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "BUILTIN\Administrators:(F)"
   icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\AfterScript_permissions.txt
   Restart-Service TermService -Force
   ```

2.  Ejecute este script para restablecer los permisos de la carpeta MachineKey y para restablecer los archivos RSA a sus valores predeterminados.

3.  Vuelva a acceder a la máquina virtual.

Después de ejecutar el script, puede comprobar los siguientes archivos que están experimentando problemas de permisos:

* c:\temp\BeforeScript_permissions.txt
* c:\temp\AfterScript_permissions.txt

#### <a name="renew-rdp-self-signed-certificate"></a>Renueve el certificado RDP autofirmado

Si el problema persiste, ejecute el siguiente script para asegurarse de que el certificado RDP autofirmado se renueva:

```powershell
Import-Module PKI
Set-Location Cert:\LocalMachine
$RdpCertThumbprint = 'Cert:\LocalMachine\Remote Desktop\'+((Get-ChildItem -Path 'Cert:\LocalMachine\Remote Desktop\').thumbprint)
Remove-Item -Path $RdpCertThumbprint
Stop-Service -Name "SessionEnv"
Start-Service -Name "SessionEnv"
```

Si no puede renovar el certificado, siga estos pasos para intentar eliminarlo:

1. En otra máquina virtual que se encuentre en la misma red virtual, abra el cuadro **Ejecutar**, escriba **mmc** y seleccione **Aceptar**. 

2. En el menú **Archivo**, seleccione **Agregar o quitar complemento**.

3. En la lista **Complementos disponibles**, seleccione **Certificados** y, a continuación, seleccione **Agregar**.

4. Seleccione la **cuenta de equipo** y haga clic en **Siguiente**.

5. Seleccione **Otro equipo**y, a continuación, agregue la dirección IP de la máquina virtual que tiene problemas.
   >[!Note]
   >Use la red interna para evitar usar una dirección IP virtual.

6. Seleccione **Finalizar** y, a continuación, **Aceptar**.

   ![Seleccione el equipo](./media/event-id-troubleshoot-vm-rdp-connecton/select-computer.png)

7. Expanda los certificados, vaya a la carpeta Escritorio remoto/Certificados, haga clic con el botón derecho en el certificado y luego seleccione **Eliminar**.

8. Reinicie el servicio de configuración del Escritorio remoto:

   ```cmd
   net stop SessionEnv
   net start SessionEnv
   ```

   >[!Note]
   >En este punto, si actualiza el almacén desde mmc, el certificado vuelve a aparecer. 

Intente obtener acceso a la máquina virtual mediante RDP.

#### <a name="update-tlsssl-certificate"></a>Actualización del certificado TLS/SSL

Si configura la máquina virtual para usar un certificado TLS/SSL, ejecute el siguiente comando para obtener la huella digital. A continuación, compruebe si es la misma que la huella digital del certificado:

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SSLCertificateSHA1Hash
```

Si no es así, cambie la huella digital:

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SSLCertificateSHA1Hash /t REG_BINARY /d <CERTIFICATE THUMBPRINT>
```

También puede intentar eliminar la clave para que RDP use el certificado autofirmado de RDP:

```cmd
reg delete "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SSLCertificateSHA1Hash
```

## <a name="scenario-2"></a>Escenario 2

### <a name="event-log"></a>Registro de eventos

En una instancia de CMD, ejecute los siguientes comandos para comprobar si el evento 36871 del error SCHANNEL se anotó en el registro del sistema en las últimas 24 horas:

```cmd
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Schannel'] and EventID=36871 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
```

**Nombre de registro**:      Sistema <br />
**Origen**:        Schannel <br />
**Fecha**          — <br />
**Id. de evento**:      36871 <br />
**Categoría de tarea**: None <br />
**Nivel**:         Error <br />
**Palabras clave:**       <br />
**Usuario**:          SYSTEM <br />
**Equipo:**      *equipo* <br />
**Descripción:** Se produjo un error grave al crear las credenciales del servidor TLS. El estado del error interno es 10013.
 
### <a name="cause"></a>Causa

Este problema se debe a las directivas de seguridad. Cuando se deshabilitan las versiones anteriores de TLS (por ejemplo, 1.0), se produce un error de acceso de RDP.

### <a name="resolution"></a>Solución

RDP usa TLS 1.0 como protocolo predeterminado. Sin embargo, el protocolo puede cambiarse a TLS 1.1, que es el nuevo estándar.

Para solucionar este problema, consulte [Troubleshoot authentication errors when you use RDP to connect to Azure VM](troubleshoot-authentication-error-rdp-vm.md#tls-version) (Solucionar errores de autenticación cuando use RDP para conectarse a la máquina virtual de Azure).

## <a name="scenario-3"></a>Escenario 3

Si ha instalado el rol del **agente de conexión a Escritorio remoto** en la máquina virtual, compruebe si se creó el evento 2056 o 1296 en las últimas 24 horas. En una instancia de CMD, ejecute los siguientes comandos: 

```cmd
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name=' Microsoft-Windows-TerminalServices-SessionBroker '] and EventID=2056 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name=' Microsoft-Windows-TerminalServices-SessionBroker-Client '] and EventID=1296 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
```

**Nombre de registro**:      Microsoft-Windows-TerminalServices-SessionBroker/Operational <br />
**Origen**:        Microsoft-Windows-TerminalServices-SessionBroker <br />
**Fecha:**          *hora* <br />
**Id. de evento**:      2056 <br />
**Categoría de tarea**: (109) <br />
**Nivel**:         Error <br />
**Palabras clave:**       <br />
**Usuario**:          SERVICIO DE RED <br />
**Equipo:**      *equipo fqdn* <br />
**Descripción:** No se puede encontrar la descripción del id. de evento 2056 del origen Microsoft-Windows-TerminalServices-SessionBroker. El componente que provoca este evento no está instalado en el equipo local o la instalación está dañada. Puede instalar o reparar el componente en el equipo local. <br />
Si el evento se originó en otro equipo, la información que aparezca en pantalla se debe guardar con el evento. <br />
La siguiente información se incluyó con el evento: <br />
NULL <br />
NULL <br />
No se pudo iniciar sesión en la base de datos.

**Nombre de registro**:      Microsoft-Windows-TerminalServices-SessionBroker-Client/Operational <br />
**Origen**:        Microsoft-Windows-TerminalServices-SessionBroker-Client <br />
**Fecha:**          *hora* <br />
**Id. de evento**:      1296 <br />
**Categoría de tarea**: (104) <br />
**Nivel**:         Error <br />
**Palabras clave:**       <br />
**Usuario**:          SERVICIO DE RED <br />
**Equipo:**      *equipo fqdn* <br />
**Descripción:** No se puede encontrar la descripción del id. de evento 1296 del origen Microsoft-Windows-TerminalServices-SessionBroker-Client. El componente que provoca este evento no está instalado en el equipo local o la instalación está dañada. Puede instalar o reparar el componente en el equipo local.
Si el evento se originó en otro equipo, la información que aparezca en pantalla se debe guardar con el evento.
La siguiente información se incluyó con el evento:  <br />
*text* <br />
*text* <br />
El Agente de conexión a Escritorio remoto no está listo para comunicarse con RPC.

### <a name="cause"></a>Causa

Este problema se produce porque se cambia el nombre de host de servidor del Agente de conexión a Escritorio remoto, y este no es un cambio admitido. 

El nombre de host tiene entradas y dependencias en Windows Internal Database, las cuales son necesaria para que la granja del servicio de Escritorio remoto pueda funcionar correctamente. Si cambia el nombre de host después de que la granja ya esté integrada, provocará un sinfín de errores y puede hacer que el servidor del agente deje de funcionar.

### <a name="resolution"></a>Solución 

Para corregir este problema, debe reinstalar el rol de Agente de conexión a Escritorio remoto y Windows Internal Database.

## <a name="next-steps"></a>Pasos siguientes

[Eventos de Schannel](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn786445(v=ws.11))

[Información general técnica de SSP de Schannel](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn786429(v=ws.11))

[RDP Fails with Event ID 1058 & Event 36870 with Remote Desktop Session Host Certificate & SSL Communication](https://techcommunity.microsoft.com/t5/ask-the-performance-team/bg-p/AskPerf) (Se produce un error en RDP con el id. de evento 1058 y 36870 con el certificado del host de sesión de Escritorio remoto y la comunicación de SSL)

[Schannel 36872 or Schannel 36870 on a Domain Controller](/archive/blogs/instan/schannel-36872-or-schannel-36870-on-a-domain-controller) (Schannel 36872 o Schannel 36870 en un controlador de dominio)

[Event ID 1058 — Remote Desktop Services Authentication and Encryption](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee890862(v=ws.10)) (Id. de evento 1058: autenticación y cifrado de los servicios de Escritorio remoto)
