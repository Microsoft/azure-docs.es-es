---
title: Escritorio remoto se desconecta con frecuencia en la máquina virtual de Azure | Microsoft Docs
description: Aprenda a solucionar problemas de desconexiones frecuentes de Escritorio remoto en la máquina virtual de Azure.
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/24/2018
ms.author: genli
ms.openlocfilehash: 60be7c234a0166331c35eb6528eae11bfbbf518f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074319"
---
# <a name="remote-desktop-disconnects-frequently-in-azure-vm"></a>Escritorio remoto se desconecta con frecuencia en la máquina virtual de Azure

En este artículo se explica cómo solucionar problemas de desconexiones frecuentes en una máquina virtual de Azure mediante el protocolo de Escritorio remoto (RDP).


## <a name="symptom"></a>Síntoma

Se enfrenta a problemas de conectividad intermitentes de RDP durante las sesiones. Inicialmente puede conectarse a la máquina virtual, pero después se pierde la conexión.

## <a name="cause"></a>Causa

Este problema puede producirse si el agente de escucha de RDP está mal configurado. Normalmente, este problema se produce en una máquina virtual que usa una imagen personalizada.

## <a name="solution"></a>Solución

Antes de seguir estos pasos, [tome una instantánea del disco del sistema operativo](../windows/snapshot-copy-managed-disk.md) de la máquina virtual afectada como copia de seguridad. 

Para solucionar este problema, utilice el control serie o [repare la máquina virtual sin conexión](#repair-the-vm-offline) mediante la conexión del disco del sistema operativo de la máquina virtual a una máquina virtual de recuperación.

### <a name="serial-control"></a>Control serie

1. Conéctese a una [consola serie y abra la instancia de CMD](./serial-console-windows.md). A continuación, ejecute los siguientes comandos para restablecer la configuración de RDP. Si la consola serie no está habilitada en la máquina virtual, vaya al paso siguiente.
2. Reduzca el nivel de seguridad de RDP a 0. En esta configuración, las comunicaciones entre el cliente y servidor usan el cifrado RDP nativo.

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'SecurityLayer' /t REG_DWORD /d 0 /f`

3. Baje el nivel de cifrado al valor mínimo para permitir que los clientes heredados de RDP se conecten.

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'MinEncryptionLevel' /t REG_DWORD /d 1 /f`

4. Establezca el RDP para cargar la configuración de usuario del equipo cliente.

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fQueryUserConfigFromLocalMachine' /t REG_DWORD /d 1 /f`

5. Habilite el control Keep-Alive de RDP:

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'KeepAliveTimeout' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'KeepAliveEnable' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'KeepAliveInterval' /t REG_DWORD /d 1 /f`

6. Establezca el control de reconexión de RDP:

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritReconnectSame' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fReconnectSame' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'fDisableAutoReconnect' /t REG_DWORD /d 0 /f`

7. Establezca el control de tiempo de la sesión de RDP:

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxSessionTime' /t REG_DWORD /d 1 /f`

8. Establezca el control de tiempo de desconexión de RDP: 

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxDisconnectionTime' /t REG_DWORD /d 1 /f` `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxDisconnectionTime' /t REG_DWORD /d 0 /f`

9. Establezca el control de tiempo de conexión de RDP:

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxConnectionTime' /t REG_DWORD /d 0 /f`

10. Establezca el control de tiempo de inactividad de sesión de RDP:

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxIdleTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxIdleTime' /t REG_DWORD /d 0 /f`

11. Establezca el control para limitar el número máximo de conexiones simultáneas:

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxInstanceCount' /t REG_DWORD /d 4294967295 /f`

12. Reinicie la máquina virtual e inténtelo de nuevo para conectarse a ella mediante RDP.

### <a name="repair-the-vm-offline"></a>Reparación de la máquina virtual sin conexión

1. [Conecte el disco del sistema operativo a una máquina virtual de recuperación](./troubleshoot-recovery-disks-portal-windows.md).
2. Después de que el disco del sistema operativo esté conectado a una máquina virtual de recuperación, asegúrese de que el disco aparece marcado como **En línea** en la consola de administración de discos. Anote la letra de unidad que se asigna al disco del sistema operativo conectado.
3. En el disco del sistema operativo que ha asociado, vaya a la carpeta **\windows\system32\config**. Copie todos los archivos de esta carpeta como copia de seguridad, por si fuera necesaria una reversión.
4. Inicie el Editor del Registro (regedit.exe).
5. Seleccione la clave **HKEY_LOCAL_MACHINE**. En el menú, seleccione **Archivo** > **Cargar subárbol**:
6. Vaya a la carpeta **\windows\system32\config\SYSTEM** en el disco del sistema operativo que ha asociado. Como nombre de la sección, escriba **BROKENSYSTEM**. El nuevo subárbol del Registro se muestra bajo la clave **HKEY_LOCAL_MACHINE**. A continuación, cargue el subárbol de software **\windows\system32\config\SOFTWARE** bajo la clave **HKEY_LOCAL_MACHINE**. Como nombre del software del subárbol, escriba **BROKENSOFTWARE**. 
7. Abra una ventana de símbolo del sistema con privilegios elevados (**Ejecutar como administrador**) y ejecute comandos en los pasos restantes para restablecer las configuraciones de RDP. 
8. Baje el nivel de seguridad de RDP a 0 para que las comunicaciones entre el cliente y servidor usen el cifrado RDP nativo:

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'SecurityLayer' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'SecurityLayer' /t REG_DWORD /d 0 /f`

9. Baje el nivel de cifrado para la configuración mínima para permitir que se conecten los clientes heredados de RDP:

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MinEncryptionLevel' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MinEncryptionLevel' /t REG_DWORD /d 1 /f`

10. Establezca RDP para cargar la configuración de usuario de la máquina cliente.

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fQueryUserConfigFromLocalMachine' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fQueryUserConfigFromLocalMachine' /t REG_DWORD /d 1 /f`

11. Habilite el control Keep-Alive de RDP:

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'KeepAliveTimeout' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'KeepAliveTimeout' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'KeepAliveEnable' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'KeepAliveInterval' /t REG_DWORD /d 1 /f`

12. Establezca el control de reconexión de RDP:

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritReconnectSame' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fReconnectSame' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritReconnectSame' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fReconnectSame' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'fDisableAutoReconnect' /t REG_DWORD /d 0 /f`

13. Establezca el control de tiempo de la sesión de RDP:

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxSessionTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxSessionTime' /t REG_DWORD /d 1 /f`

14. Establezca el control de tiempo de desconexión de RDP:

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxDisconnectionTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxDisconnectionTime' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxDisconnectionTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxDisconnectionTime' /t REG_DWORD /d 0 /f`

15. Establezca el control de tiempo de conexión de RDP:

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxConnectionTime' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxConnectionTime' /t REG_DWORD /d 0 /f`

16. Establezca el control de tiempo de inactividad de sesión de RDP:

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxIdleTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v ' MaxIdleTime' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxIdleTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v ' MaxIdleTime' /t REG_DWORD /d 0 /f`

17. Establezca el control para limitar el número máximo de conexiones simultáneas:

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxInstanceCount' /t REG_DWORD /d ffffffff /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxInstanceCount' /t REG_DWORD /d ffffffff /f`

18. Reinicie la máquina virtual e inténtelo de nuevo para conectarse a ella mediante RDP.

## <a name="need-help"></a>¿Necesita ayuda? 
Póngase en contacto con el servicio de soporte técnico. Si sigue necesitando ayuda, [póngase en contacto con el servicio de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.
