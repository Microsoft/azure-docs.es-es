---
title: Solucionar errores de autenticación cuando use RDP para conectarse a la máquina virtual de Azure | Microsoft Docs
description: Obtenga información sobre cómo solucionar los errores de autenticación que se producen al usar el protocolo de escritorio remoto (RDP) para conectarse a una máquina virtual (VM) de Azure.
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
ms.openlocfilehash: cc1ad3104596cc7ad4bb48f88e4c8312bc833371
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088570"
---
# <a name="troubleshoot-authentication-errors-when-you-use-rdp-to-connect-to-azure-vm"></a>Solucionar errores de autenticación cuando use RDP para conectarse a la máquina virtual de Azure

Con este artículo podrá solucionar los errores de autenticación que se producen al usar una conexión de protocolo de escritorio remoto (RDP) para conectarse a una máquina virtual (VM) de Azure.

## <a name="symptoms"></a>Síntomas

Hace una captura de pantalla de una máquina virtual de Azure que muestra la pantalla de bienvenida e indica que se está ejecutando el sistema operativo. Sin embargo, cuando intenta conectarse a la máquina virtual mediante la conexión a Escritorio remoto, recibe uno de los siguientes mensajes de error.

### <a name="error-message-1"></a>Mensaje de error 1

**Error de autenticación. No se puede conectar con la autoridad de seguridad local.**

### <a name="error-message-2"></a>Mensaje de error 2

**El equipo remoto al que está intentando conectarse requiere que realice la autenticación a nivel de red (NLA), pero no se puede establecer contacto con el controlador de dominio de Windows para llevar a cabo el proceso de NLA. Si es administrador en el equipo remoto, puede deshabilitar la NLA mediante las opciones de la ficha acceso remoto que se encuentra en el cuadro de diálogo Propiedades del sistema.**

### <a name="error-message-3-generic-connection-error"></a>Mensaje de error 3 (error de conexión genérica)

**Este equipo no se puede conectar al equipo remoto. Intente conectarse de nuevo y, si el problema continúa, póngase en contacto con el propietario del equipo remoto o con el administrador de red.**

## <a name="cause"></a>Causa

Hay varias razones por las que la NLA puede bloquear el acceso RDP a una máquina virtual.

### <a name="cause-1"></a>Causa 1

La máquina virtual no puede comunicarse con el controlador de dominio (DC). Este problema puede impedir que una sesión RDP obtenga acceso a una máquina virtual mediante las credenciales de dominio. Sin embargo, aún podrá iniciar sesión con las credenciales de administrador local. Este problema puede producirse en las situaciones siguientes:

1. El canal de seguridad de Active Directory entre esta máquina virtual y el controlador de dominio se interrumpe.

2. La máquina virtual tiene una copia antigua de la contraseña de cuenta y el controlador de dominio tiene una copia más reciente.

3. El controlador de dominio que se está conectando a la máquina virtual está en mal estado.

### <a name="cause-2"></a>Causa 2

El nivel de cifrado de la máquina virtual es mayor que el que usa el equipo cliente.

### <a name="cause-3"></a>Causa 3

Los protocolos TLS 1.0, 1.1 o 1.2 (servidor) están deshabilitados en la máquina virtual.

### <a name="cause-4"></a>Causa 4

La máquina virtual se configuró para deshabilitar el inicio de sesión mediante las credenciales de dominio, y la autoridad de seguridad local (LSA) se ha configurado correctamente.

### <a name="cause-5"></a>Causa 5

La máquina virtual se ha configurado para que solo acepte el Estándar federal de procesamiento de información (FIPS), el cual cumple con las conexiones de algoritmo. Normalmente esta opción se lleva a cabo mediante la directiva de Active Directory. Esta es una configuración poco frecuente, pero FIPS solo se puede aplicar en las conexiones de escritorio remoto.

## <a name="before-you-troubleshoot"></a>Antes de iniciar los pasos de la solución

### <a name="create-a-backup-snapshot"></a>Cree una instantánea de copia de seguridad

Para crear una instantánea de copia de seguridad, siga los pasos descritos en [Instantánea de un disco](../windows/snapshot-copy-managed-disk.md).

### <a name="connect-to-the-vm-remotely"></a>Conéctese a la máquina virtual de forma remota

Para conectarse a la máquina virtual de forma remota, use uno de los métodos que se detallan en [How to use remote tools to troubleshoot Azure VM issues](remote-tools-troubleshoot-azure-vm-issues.md) (Cómo usar herramientas remotas para solucionar problemas con la máquina virtual de Azure).

### <a name="group-policy-client-service"></a>Servicio de cliente de la directiva de grupo

Si se trata de una máquina virtual unida al dominio, primero debe detener el servicio de cliente de la directiva de grupo para evitar que cualquier directiva de Active Directory sobrescriba los cambios. Para ello, ejecute el siguiente comando:

```cmd
REM Disable the member server to retrieve the latest GPO from the domain upon start
REG add "HKLM\SYSTEM\CurrentControlSet\Services\gpsvc" /v Start /t REG_DWORD /d 4 /f
```

Una vez corregido el problema, restaure la capacidad de esta máquina virtual para ponerse en contacto con el dominio y así poder recuperar el GPO más reciente del dominio. Para ello, ejecute los siguientes comandos:

```cmd
sc config gpsvc start= auto
sc start gpsvc

gpupdate /force
```

Si el cambio se revierte, significa que una directiva de Active Directory está causando el problema. 

### <a name="workaround"></a>Solución alternativa

Para solucionar este problema, ejecute los siguientes comandos en la ventana de comandos para deshabilitar la NLA:

```cmd
REM Disable the Network Level Authentication
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD /d 0
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD /d 0
```

A continuación, reinicie la máquina virtual.

Para volver a habilitar la NLA, ejecute el siguiente comando y, a continuación, reinicie la máquina virtual:

```cmd
REG add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v disabledomaincreds /t REG_DWORD /d 0 /f

REG add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD /d 1 /f
REG add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 1 /f
REG add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD /d 1 /f
```

## <a name="troubleshooting"></a>Solución de problemas

### <a name="for-domain-joined-vms"></a>Para máquinas virtuales unidas al dominio

Para solucionar este problema, compruebe primero si la máquina virtual puede conectarse a un controlador de dominio, si el controlador de dominio tiene un estado "Correcto" y si puede controlar las solicitudes de la máquina virtual.

>[!Note] 
>Para probar el estado del controlador de dominio, puede usar otra máquina virtual en la misma red virtual y subred que compartan el mismo servidor de inicio de sesión.

Conéctese a la máquina virtual que tiene el problema mediante la consola serie, el CMD remoto o una instancia remota de PowerShell, según los pasos descritos en la sección "Conectarse a la máquina virtual de forma remota".

Para determinar a qué controlador de dominio se conectará la máquina virtual, ejecute el siguiente comando en la consola: 

```cmd
set | find /i "LOGONSERVER"
```

A continuación, compruebe el estado del canal seguro entre la máquina virtual y el controlador de dominio. Para ello, ejecute el comando siguiente en una instancia de PowerShell con privilegios elevados. Este comando devuelve una marca booleana que indica si el canal seguro está activo:

```powershell
Test-ComputerSecureChannel -verbose
```

Si se interrumpe el canal, ejecute el siguiente comando para reparar el error:

```powershell
Test-ComputerSecureChannel -repair
```

Asegúrese de que la contraseña de la cuenta de equipo en Active Directory se actualiza en la máquina virtual y el controlador de dominio:

```powershell
Reset-ComputerMachinePassword -Server "<COMPUTERNAME>" -Credential <DOMAIN CREDENTIAL WITH DOMAIN ADMIN LEVEL>
```

Si la comunicación entre el controlador de dominio y la máquina virtual es buena, pero el controlador de dominio no es correcto para abrir una sesión RDP, puede intentar reiniciar el controlador de dominio.

Si los comandos anteriores no solucionan el problema de comunicación al dominio, puede volver a unir esta máquina virtual al dominio. Para ello, siga estos pasos.

1. Cree un script que se denomine Unjoin.ps1 mediante el siguiente contenido y, a continuación, implemente el script como una extensión de script personalizado en Azure Portal:

    ```cmd
    cmd /c "netdom remove <<MachineName>> /domain:<<DomainName>> /userD:<<DomainAdminhere>> /passwordD:<<PasswordHere>> /reboot:10 /Force"
    ```
    
    Este script saca la máquina virtual del dominio a la fuerza y la reinicia 10 segundos más tarde. A continuación, tendrá que limpiar el objeto Computer en el lado del dominio.

2.  Después de realizar la limpieza, vuelva a unir la máquina virtual al dominio. Para ello, cree un script que se denomine JoinDomain.ps1 mediante el siguiente contenido y, a continuación, implemente el script como una extensión de script personalizado en Azure Portal: 

    ```cmd
    cmd /c "netdom join <<MachineName>> /domain:<<DomainName>> /userD:<<DomainAdminhere>> /passwordD:<<PasswordHere>> /reboot:10"
    ```

    >[!Note] 
    >Esta opción une la máquina virtual al dominio mediante las credenciales especificadas.

Si el canal de Active Directory está en buen estado, se actualiza la contraseña del equipo y el controlador de dominio funciona según lo esperado, realice los pasos siguientes.

Si el problema persiste, compruebe si la credencial de dominio está deshabilitada. Para ello, abra una ventana de símbolo del sistema con privilegios elevados y, a continuación, ejecute el siguiente comando para determinar si la máquina virtual está configurada para deshabilitar las cuentas de dominio que se usan para iniciar sesión en la máquina virtual:

```cmd
REG query "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v disabledomaincreds
```

Si la clave se establece en **1**, esto significa que el servidor se estableció para no permitir credenciales de dominio. Cambie la clave a **0**.

### <a name="for-standalone-vms"></a>Para máquinas virtuales independientes

#### <a name="check-minencryptionlevel"></a>Compruebe el elemento MinEncryptionLevel

En una instancia de CMD, ejecute el comando siguiente para consultar el valor del registro **MinEncryptionLevel**:

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MinEncryptionLevel
```

En función del valor del registro, siga estos pasos:

* 4 (FIPS): vaya a [Comprobar las conexiones de algoritmos que cumplan el estándar federal de procesamiento de información](#fips-compliant).

* 3 (cifrado de 128 bits): establezca la gravedad en **2**; para ello, ejecute el comando siguiente:

    ```cmd
    reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MinEncryptionLevel /t REG_DWORD /d 2 /f
    ```

* 2 (el cifrado más alto posible, tal como indicó el cliente): puede intentar establecer el cifrado en el valor mínimo de **1** mediante la ejecución del comando siguiente:

    ```cmd
    reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MinEncryptionLevel /t REG_DWORD /d 1 /f
    ```
    
Reinicie la máquina virtual para que los cambios en el Registro surtan efecto.

#### <a name="tls-version"></a>Versión TLS

Dependiendo del sistema, RDP usa el protocolo TLS 1.0, 1.1 o 1.2 (servidor). Para ver cómo se configuran estos protocolos en la máquina virtual, abra una instancia de CMD y, a continuación, ejecute los siguientes comandos:

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled
reg query "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled
reg query "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled
```

Si todos los valores devueltos no son **1**, esto significa que el protocolo está deshabilitado. Para habilitar estos protocolos, ejecute los siguientes comandos:

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f
```

Si usa otras versiones del protocolo, puede ejecutar los comandos siguientes:

<pre lang="bat">
reg query "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS <i>x.x</i>\Server" /v Enabled
reg query "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS <i>x.x</i>\Server" /v Enabled
</pre>

> [!Note]
> Obtenga la versión x.x de SSH/TLS de los registros de sistema operativo invitado en los errores de SCHANNEL.

#### <a name="check-fips-compliant-algorithms-connections"></a><a name="fips-compliant"></a> Compruebe las conexiones de algoritmos que cumplan el estándar federal de procesamiento de información.

Se puede configurar el Escritorio remoto para que use solo las conexiones de algoritmo que cumplan con el estándar FIPS. Esto se puede establecer mediante el uso de una clave del Registro. Para ello, abra una ventana de símbolo del sistema con privilegios elevados y, a continuación, consulte las siguientes claves:

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Lsa\FIPSAlgorithmPolicy" /v Enabled
```

Si el comando devuelve **1**, cambie el valor del Registro a **0**.

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Lsa\FIPSAlgorithmPolicy" /v Enabled /t REG_DWORD /d 0
```

Compruebe cuál es el valor de MinEncryptionLevel actual en la máquina virtual:

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MinEncryptionLevel
```

Si el comando devuelve **4**, cambie el valor del Registro a **2**.

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MinEncryptionLevel /t REG_DWORD /d 2
```

Reinicie la máquina virtual para que los cambios en el Registro surtan efecto.

## <a name="next-steps"></a>Pasos siguientes

[SetEncryptionLevel method of the Win32_TSGeneralSetting class](/windows/desktop/termserv/win32-tsgeneralsetting-setencryptionlevel) (Establecer el método SetEncryptionLevel de la clase Win32_TSGeneralSetting)

[Configure Server Authentication and Encryption Levels](/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc770833(v=ws.11)) (Configurar la autenticación de servidor y los niveles de cifrado)

[Win32_TSGeneralSetting class](/windows/desktop/termserv/win32-tsgeneralsetting) (Clase Win32_TSGeneralSetting)
