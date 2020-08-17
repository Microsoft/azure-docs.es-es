---
title: Deshabilitar el firewall del sistema operativo invitado en la máquina virtual de Azure | Microsoft Docs
description: Obtenga información sobre un método de solución alternativo para las situaciones en las que el firewall de un sistema operativo invitado filtra de forma parcial o completa el tráfico a una máquina virtual.
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
ms.openlocfilehash: c0426c5359e4d82d0316613586b9298596d82605
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87009771"
---
# <a name="disable-the-guest-os-firewall-in-azure-vm"></a>Deshabilitar el firewall del sistema operativo invitado en Azure Virtual Machine

En este artículo se proporciona una referencia para las situaciones en que se sospecha que el firewall del sistema operativo invitado está filtrando el tráfico parcial o completo en una máquina virtual (VM). Esto puede ocurrir si deliberadamente se realizaron cambios en el firewall que han provocado errores en las conexiones RDP.

## <a name="solution"></a>Solución

El proceso que se describe en este artículo está pensado para usarse como solución alternativa para que pueda centrarse en solucionar el problema real, que es cómo configurar correctamente las reglas de firewall. Es un procedimiento recomendado de Microsoft habilitar el componente de firewall de Windows. El modo de configurar las reglas de firewall depende del nivel de acceso a la máquina virtual requerido.

### <a name="online-solutions"></a>Soluciones en línea 

Si la máquina virtual está en línea y se puede acceder desde otra máquina virtual en la misma red virtual, puede realizar estas mitigaciones mediante el uso de la otra máquina virtual.

#### <a name="mitigation-1-custom-script-extension-or-run-command-feature"></a>Mitigación 1: Característica Ejecutar comando o Extensión de script personalizado

Si tiene un agente de Azure en ejecución, puede usar la característica [Ejecutar comando](../windows/run-command.md) o [extensión de script personalizado](../extensions/custom-script-windows.md) (solo máquinas virtuales de Resource Manager) para ejecutar los siguientes scripts de forma remota.

> [!Note]
> * Si el firewall se establece localmente, ejecute el siguiente script:
>   ```
>   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\DomainProfile' -name "EnableFirewall" -Value 0
>   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\PublicProfile' -name "EnableFirewall" -Value 0
>   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\Standardprofile' -name "EnableFirewall" -Value 0 
>   Restart-Service -Name mpssvc
>   ```
> * Si el firewall se establece con una directiva de Active Directory, puede ejecutar el siguiente script para tener acceso temporal. 
>   ```
>   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\WindowsFirewall\DomainProfile' -name "EnableFirewall" -Value 0
>   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\WindowsFirewall\PublicProfile' -name "EnableFirewall" -Value 0
>   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\WindowsFirewall\StandardProfile' -name "EnableFirewall" -Value 0
>   Restart-Service -Name mpssvc
>   ```
>   Sin embargo, tan pronto como la directiva se aplique de nuevo, abandonará la sesión remota. La solución permanente para este problema consiste en modificar la directiva que se aplica en este equipo.

#### <a name="mitigation-2-remote-powershell"></a>Mitigación2: PowerShell remoto

1.  Conéctese a una máquina virtual que se encuentre en la misma red virtual que la máquina virtual a la que no se pueda acceder con una conexión RDP.

2.  Abra una ventana de la consola de PowerShell.

3.  Ejecute los comandos siguientes:

    ```powershell
    Enter-PSSession (New-PSSession -ComputerName "<HOSTNAME>" -Credential (Get-Credential) -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck)) 
    netsh advfirewall set allprofiles state off
    Restart-Service -Name mpssvc 
    exit
    ```

> [!Note]
> Si el firewall se establece con un objeto de directiva de grupo, puede que este método no funcione debido a que este comando cambia solo las entradas del registro local. Si hay una directiva en vigor, invalidará este cambio. 

#### <a name="mitigation-3-pstools-commands"></a>Mitigación 3: comandos de PSTools

1.  En la máquina virtual donde está solucionando el problema, descargue [PSTools](/sysinternals/downloads/pstools).

2.  Abra una instancia de CMD y, a continuación, acceda a la máquina virtual mediante su DIP.

3.  Ejecute los comandos siguientes:

    ```cmd
    psexec \\<DIP> -u <username> cmd
    netsh advfirewall set allprofiles state off
    psservice restart mpssvc
    ```

#### <a name="mitigation-4-remote-registry"></a>Mitigación4: Registro remoto 

Siga estos pasos para utilizar [Registro remoto](https://support.microsoft.com/help/314837/how-to-manage-remote-access-to-the-registry).

1.  En la máquina virtual donde solucionar el problema, inicie el editor del Registro y, a continuación, vaya a **Archivo** > **Conectar al Registro de red**.

2.  Abra la rama *MÁQUINA DE DESTINO*\SYSTEM y especifique los valores siguientes:

    ```
    <TARGET MACHINE>\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\DomainProfile\EnableFirewall           -->        0 
    <TARGET MACHINE>\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\PublicProfile\EnableFirewall           -->        0 
    <TARGET MACHINE>\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\StandardProfile\EnableFirewall         -->        0
    ```

3.  Reinicie el servicio. Dado que no puede hacerlo mediante el registro remoto, debe usar Remote Service Console (Quitar la consola de servicio).

4.  Abra una instancia de **Services.msc**.

5.  Haga clic en **Servicios (Local)** .

6.  Seleccione **Conectarse a otro equipo**.

7.  Escriba la **dirección IP privada (DIP)** de la máquina virtual con problemas.

8.  Reinicie la directiva de firewall local.

9.  Intente volver a conectar a la máquina virtual mediante RDP desde el equipo local.

### <a name="offline-solutions"></a>Soluciones sin conexión 

Si tiene una situación en la que no se puede llegar a la máquina virtual mediante ningún método, se producirá un error en la extensión de script personalizado y tendrá que trabajar en modo sin conexión directamente con el disco del sistema. Para ello, sigue estos pasos:

1.  [Conecte el disco del sistema a una máquina virtual de recuperación](troubleshoot-recovery-disks-portal-windows.md).

2.  Inicie una conexión mediante el Escritorio remoto a la máquina virtual de recuperación.

3.  Asegúrese de que el disco aparece marcado como En línea en la consola de Administración de discos. Anote la letra de unidad que se asigna al disco del sistema conectado.

4.  Antes de hacer cambios, cree una copia de la carpeta \Windows\system32\config, por si necesita una reversión de los cambios.

5.  En la máquina virtual donde solucionar el problema, inicie el editor del Registro (regedit.exe). 

6.  En procedimiento de solución de problemas, vamos a montar los subárboles como BROKENSYSTEM y BROKENSOFTWARE.

7.  Seleccione la clave HKEY_LOCAL_MACHINE y, a continuación, seleccione Archivo > Cargar subárbol en el menú.

8.  Busque el archivo \windows\system32\config\SYSTEM en el disco del sistema conectado.

9.  Abra una instancia de PowerShell con privilegios elevados y, luego, ejecute los comandos siguientes:

    ```cmd
    # Load the hives - If your attached disk is not F, replace the letter assignment here
    reg load HKLM\BROKENSYSTEM f:\windows\system32\config\SYSTEM
    reg load HKLM\BROKENSOFTWARE f:\windows\system32\config\SOFTWARE 
    # Disable the firewall on the local policy
    $ControlSet = (get-ItemProperty -Path 'HKLM:\BROKENSYSTEM\Select' -name "Current").Current
    $key = 'BROKENSYSTEM\ControlSet00'+$ControlSet+'\services\SharedAccess\Parameters\FirewallPolicy\DomainProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    $key = 'BROKENSYSTEM\ControlSet00'+$ControlSet+'\services\SharedAccess\Parameters\FirewallPolicy\PublicProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    $key = 'BROKENSYSTEM\ControlSet00'+$ControlSet+'\services\SharedAccess\Parameters\FirewallPolicy\StandardProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    # To ensure the firewall is not set through AD policy, check if the following registry entries exist and if they do, then check if the following entries exist:
    $key = 'HKLM:\BROKENSOFTWARE\Policies\Microsoft\WindowsFirewall\DomainProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    $key = 'HKLM:\BROKENSOFTWARE\Policies\Microsoft\WindowsFirewall\PublicProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    $key = 'HKLM:\BROKENSOFTWARE\Policies\Microsoft\WindowsFirewall\StandardProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    # Unload the hives
    reg unload HKLM\BROKENSYSTEM
    reg unload HKLM\BROKENSOFTWARE
    ```

10. [Desconecte el disco del sistema y vuelva a crear la máquina virtual](troubleshoot-recovery-disks-portal-windows.md).

11. Compruebe si se ha resuelto el problema.
