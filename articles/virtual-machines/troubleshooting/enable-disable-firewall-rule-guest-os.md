---
title: Habilitación o deshabilitación una regla del firewall en un SO invitado en una VM de Azure | Microsoft Docs
description: Obtenga información sobre cómo usar las herramientas remotas en línea o sin conexión, o la configuración del registro, para habilitar o deshabilitar las reglas de firewall del sistema operativo invitado en una máquina virtual de Azure remota.
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
ms.date: 11/22/2018
ms.author: delhan
ms.openlocfilehash: 17616a223292ec07186b0a3fba264400423977ac
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87058765"
---
# <a name="enable-or-disable-a-firewall-rule-on-an-azure-vm-guest-os"></a>Habilitación o deshabilitación de una regla de firewall en el SO invitado de una máquina virtual de Azure

En este artículo se proporciona una referencia para solucionar una situación en que se sospecha que el firewall del sistema operativo invitado está filtrando tráfico parcial en una máquina virtual (VM). Esto puede resultar útil por los siguientes motivos:

*   Si se realizó un cambio deliberado en el firewall que provocó el error de las conexiones de RDP, el uso de la función de la característica Extensión Custom Script puede solucionar el problema.

*   La deshabilitación de todos los perfiles de firewall es más probable que solucione el error que la configuración de una regla de firewall específica para RDP.

## <a name="solution"></a>Solución

El modo de configurar las reglas de firewall depende del nivel de acceso a la máquina virtual requerido. En los ejemplos siguientes se usan reglas de RDP. Sin embargo, pueden aplicarse los mismos métodos a cualquier otro tipo de tráfico que apunte a la clave del Registro correcta.

### <a name="online-troubleshooting"></a>Solución de problemas en línea 

#### <a name="mitigation-1-custom-script-extension"></a>Mitigación 1: Custom Script Extension

1.  Cree un script mediante la siguiente plantilla.

    *   Para habilitar una regla:
        ```cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=yes
        ```

    *   Para deshabilitar una regla:
        ```cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=no
        ```

2.  Cargue este script en Azure Portal mediante la característica [Custom Script Extension](../extensions/custom-script-windows.md). 

#### <a name="mitigation-2-remote-powershell"></a>Mitigación2: PowerShell remoto

Si la máquina virtual está en línea y se puede acceder desde otra máquina virtual en la misma red virtual, puede realizar estas mitigaciones mediante el uso de la otra máquina virtual.

1.  En la máquina virtual donde está solucionando el problema, abra una ventana de consola de PowerShell.

2.  Ejecute los comandos siguientes, según corresponda.

    *   Para habilitar una regla:
        ```powershell
        Enter-PSSession (New-PSSession -ComputerName "<HOSTNAME>" -Credential (Get-Credential) -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck)) 
        Enable-NetFirewallRule -DisplayName  "RemoteDesktop-UserMode-In-TCP"
        exit
        ```

    *   Para deshabilitar una regla:
        ```powershell
        Enter-PSSession (New-PSSession -ComputerName "<HOSTNAME>" -Credential (Get-Credential) -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck)) 
        Disable-NetFirewallRule -DisplayName  "RemoteDesktop-UserMode-In-TCP"
        exit
        ```

#### <a name="mitigation-3-pstools-commands"></a>Mitigación 3: comandos de PSTools

Si la máquina virtual está en línea y se puede acceder desde otra máquina virtual en la misma red virtual, puede realizar estas mitigaciones mediante el uso de la otra máquina virtual.

1.  En la máquina virtual donde está solucionando el problema, descargue [PSTools](/sysinternals/downloads/pstools).

2.  Abra una instancia de CMD y acceda a la máquina virtual a través de su dirección IP interna. 

    * Para habilitar una regla:
        ```cmd
        psexec \\<DIP> -u <username> cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=yes
        ```

    *   Para deshabilitar una regla:
        ```cmd
        psexec \\<DIP> -u <username> cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=no
        ```

#### <a name="mitigation-4-remote-registry"></a>Mitigación4: Registro remoto

Si la máquina virtual está en línea y se puede acceder desde otra máquina virtual de la misma red virtual, puede usar el [Registro remoto](https://support.microsoft.com/help/314837/how-to-manage-remote-access-to-the-registry) en la otra máquina virtual.

1.  En la máquina virtual donde está solucionando el problema, inicie el editor del Registro (regedit.exe) y, a continuación, vaya a **Archivo** > **Conectar al Registro de red**.

2.  Abra la rama *MÁQUINA DE DESTINO*\SYSTEM y especifique los valores siguientes:

    * Para habilitar una regla, abra el valor del Registro siguiente:
    
        *TARGET MACHINE*\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP
    
        A continuación, cambie **Active=FALSE** a **Active=TRUE** en la cadena:

        `v2.22|Action=Allow|Active=TRUE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`
    
    * Para deshabilitar una regla, abra el valor del Registro siguiente:
    
        *TARGET MACHINE*\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP

        A continuación, cambie **Active =TRUE** a **Active=FALSE**:
        
        `v2.22|Action=Allow|Active=FALSE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`

3.  Reinicie la máquina virtual para aplicar estos cambios.

### <a name="offline-troubleshooting"></a>Solución de problemas sin conexión 

Si no se puede acceder a la máquina virtual mediante ningún método, se producirá un error en el uso de Custom Script Extension y tendrá que trabajar en modo sin conexión directamente con el disco del sistema.

Antes de seguir estos pasos, tome una instantánea del disco del sistema de la máquina virtual afectada como copia de seguridad. Para obtener más información, consulte [Instantánea de un disco](../windows/snapshot-copy-managed-disk.md).

1.  [Conecte el disco del sistema a una máquina virtual de recuperación](troubleshoot-recovery-disks-portal-windows.md).

2.  Inicie una conexión mediante el Escritorio remoto a la máquina virtual de recuperación.

3.  Asegúrese de que el disco aparece marcado como **En línea** en la consola de Administración de discos. Anote la letra de unidad asignada al disco del sistema conectado.

4.  Antes de hacer cambios, cree una copia de la carpeta \Windows\system32\config, por si necesita una reversión de los cambios.

5.  En la máquina virtual donde está solucionando el problema, inicie el editor del Registro (regedit.exe).

6.  Seleccione la clave **HKEY_LOCAL_MACHINE** y **Archivo** > **Load Hive** (Cargar subárbol) en el menú.

    ![Regedit](./media/enable-or-disable-firewall-rule-guest-os/load-registry-hive.png)

7.  Busque y abra el archivo \windows\system32\config\SYSTEM. 

    > [!Note]
    > Se le pedirá un nombre. Escriba **BROKENSYSTEM**y expanda **HKEY_LOCAL_MACHINE**. Ahora verá una clave adicional que se denomina **BROKENSYSTEM**. Para esta solución de problemas, estamos montando estos subárboles de problema como **BROKENSYSTEM**.

8.  Realice los cambios siguientes en la rama BROKENSYSTEM:

    1.  Compruebe que la clave del Registro **ControlSet** es la VM desde donde se empieza. Verá su número de clave en HKLM\BROKENSYSTEM\Select\Current.

    2.  Para habilitar una regla, abra el valor del Registro siguiente:
    
        HKLM\BROKENSYSTEM\ControlSet00X\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP
        
        A continuación, cambie **Active=FALSE** a **Active=True**.
        
        `v2.22|Action=Allow|Active=TRUE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`

    3.  Para deshabilitar una regla, abra la clave del Registro siguiente:

        HKLM\BROKENSYSTEM\ControlSet00X\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP

        A continuación, cambie **Active=True** a **Active=FALSE**.
        
        `v2.22|Action=Allow|Active=FALSE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`

9.  Resalte **BROKENSYSTEM** y seleccione **Archivo** > **Unload Hive** (Descargar subárbol) en el menú.

10. [Desconecte el disco del sistema y vuelva a crear la máquina virtual](troubleshoot-recovery-disks-portal-windows.md).

11. Compruebe si se ha resuelto el problema.
