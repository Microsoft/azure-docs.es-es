---
title: Mensajes de error específicos de RDP para máquinas virtuales de Azure | Microsoft Docs
description: Comprenda los mensajes de error específicos que pueden aparecer al tratar de usar la conexión Escritorio remoto a una máquina virtual Windows de Azure.
keywords: Error de escritorio remoto, error de conexión del escritorio remoto, no se puede conectar a la máquina virtual, solución de problemas con el escritorio remoto
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.assetid: 5feb1d64-ee6f-4907-949a-a7cffcbc6153
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: f3e4c411223a3e8fc0602602cd941a00f7a19cca
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87028250"
---
# <a name="troubleshooting-specific-rdp-error-messages-to-a-windows-vm-in-azure"></a>Solución de problemas de mensajes de error específicos de RDP en una máquina virtual Windows en Azure
Puede recibir un mensaje de error específico al usar la conexión Escritorio remoto a una máquina virtual (VM) Windows en Azure. Este artículo describe algunos de los mensajes de error más comunes que se pueden obtener, junto con los pasos de solución de problemas para resolverlos. Si tiene problemas para conectarse a la máquina virtual mediante RDP, pero no ve un mensaje de error específico, consulte la [Guía de solución de problemas de Escritorio remoto](troubleshoot-rdp-connection.md).

Para obtener información sobre mensajes de error específicos, vea los siguientes recursos:

* [Se desconectó la sesión remota porque no hay servidores de licencias de Escritorio remoto disponibles para proporcionar una licencia](#rdplicense).
* [Escritorio remoto no encuentra el equipo "nombre"](#rdpname).
* [Error de autenticación. No se puede conectar con la autoridad de seguridad local](#rdpauth).
* [Error de Seguridad de Windows: Las credenciales no funcionaron](#wincred).
* [Este equipo no se puede conectar al equipo remoto](#rdpconnect).

<a id="rdplicense"></a>

## <a name="the-remote-session-was-disconnected-because-there-are-no-remote-desktop-license-servers-available-to-provide-a-license"></a>Se desconectó la sesión remota porque no hay servidores de licencias de Escritorio remoto disponibles para proporcionar una licencia.
Causa: el período de gracia de licencias de 120 días para el rol de servidor de Escritorio remoto expiró y necesita instalar licencias.

Como solución alternativa, guarde una copia local del archivo RDP desde el portal y ejecute este comando en un símbolo del sistema de PowerShell para conectarse. Con este paso se deshabilita la obtención de licencias solo de esa conexión:

```powershell
mstsc <File name>.RDP /admin
```

Si no necesita más de dos conexiones simultáneas de Escritorio remoto a la máquina virtual, puede usar el administrador del servidor para quitar el rol de servidor de Escritorio remoto.

Para obtener más información, consulte la entrada de blog [Azure VM fails with "No Remote Desktop License Servers available"](/archive/blogs/mast/rdp-to-azure-vm-fails-with-no-remote-desktop-license-servers-available)(Error de la máquina virtual de Azure: "No hay servidores de licencias de Escritorio remoto disponibles").

<a id="rdpname"></a>

## <a name="remote-desktop-cant-find-the-computer-name"></a>Remote Desktop can't find the computer "name" (Escritorio remoto no puede encontrar el equipo "nombre").
Causa: el cliente de Escritorio remoto del equipo no puede resolver el nombre del equipo en la configuración del archivo RDP.

Posibles soluciones:

* Si se encuentra en la intranet de una organización, asegúrese de que el equipo tenga acceso al servidor proxy y puede enviarle tráfico HTTPS.
* Si usa un archivo RDP almacenado localmente, intente usar uno generado por el portal. Con este paso se garantiza que tiene el nombre DNS correcto para la máquina virtual o el servicio en la nube y el puerto de punto de conexión de la máquina virtual. A continuación le facilitamos un archivo RDP de ejemplo generado por el portal:

    ```output
    full address:s:tailspin-azdatatier.cloudapp.net:55919
    prompt for credentials:i:1
    ```

La parte de la dirección de este archivo RDP tiene:

* El nombre de dominio completo del servicio en la nube que contiene la máquina virtual (“tailspin-azdatatier.cloudapp.net” en este ejemplo).
* El puerto TCP externo del punto de conexión para tráfico de Escritorio remoto (55919).

<a id="rdpauth"></a>

## <a name="an-authentication-error-has-occurred-the-local-security-authority-cannot-be-contacted"></a>Error de autenticación. No se puede conectar con la autoridad de seguridad Local.
Causa: la VM de destino no encuentra la autoridad de seguridad en la parte del nombre de usuario de las credenciales.

Si el nombre de usuario tiene la forma *autoridadDeSeguridad*\\*nombreDeUsuario* (ejemplo: CORP\User1), la parte *autoridadDeSeguridad* es el nombre del equipo de la máquina virtual (para la autoridad de seguridad local) o un nombre de dominio de Active Directory.

Posibles soluciones:

* Si su cuenta está en la propia VM, asegúrese de que el nombre de la VM está escrito correctamente.
* Si la cuenta está en un dominio de Active Directory, compruebe la ortografía del nombre de dominio.
* Si es una cuenta de dominio de Active Directory y el nombre de dominio está escrito correctamente, compruebe que hay algún controlador de dominio disponible en dicho dominio. Un problema común de las redes virtuales de Azure que contienen controladores de dominio es que un controlador de dominio no está disponible porque no se ha iniciado. Como solución alternativa, puede usar una cuenta de administrador local, en lugar de una cuenta de dominio.

<a id="wincred"></a>

## <a name="windows-security-error-your-credentials-did-not-work"></a>Error de Seguridad de Windows: Las credenciales no funcionaron.
Causa: la VM de destino no pudo validar el nombre y la contraseña de la cuenta.

Un equipo con Windows puede validar las credenciales de una cuenta local o de una cuenta de dominio.

* Para las cuentas locales, use la sintaxis *nombreDeEquipo*\\*nombreDeUsuario* (ejemplo: SQL1\Admin4798).
* Para las cuentas de dominio, use la sintaxis *nombreDeDominio*\\*nombreDeUsuario* (ejemplo: CONTOSO\peterodman).

Si promovió su máquina virtual a un controlador de dominio de un nuevo bosque de Active Directory, la cuenta de administrador local con la que inició sesión se convierte en una cuenta equivalente con la misma contraseña en el nuevo bosque y dominio. A continuación, se elimina la cuenta local.

Por ejemplo, si inició sesión con la cuenta local DC1\DCAdmin y después promovió la máquina virtual como controlador de dominio en un bosque nuevo para el dominio corp.contoso.com, la cuenta local DC1\DCAdmin se elimina y se crea una nueva cuenta de dominio CORP\DCAdmin con la misma contraseña.

Asegúrese de que el nombre de la cuenta es un nombre que la máquina virtual pueda comprobar como una cuenta válida y que la contraseña sea correcta.

Si necesita cambiar la contraseña de la cuenta de administrador local, consulte [Cómo restablecer una contraseña o el servicio de Escritorio remoto para máquinas virtuales de Windows](reset-rdp.md).

<a id="rdpconnect"></a>

## <a name="this-computer-cant-connect-to-the-remote-computer"></a>This computer can't connect to the remote computer (Este equipo no se puede conectar al equipo remoto).
Causa: la cuenta usada para conectarse no tiene derechos de inicio de sesión de Escritorio remoto.

Todos los equipos de Windows tienen un grupo local Usuarios de escritorio remoto que contiene las cuentas y los grupos que pueden iniciar sesión en él de forma remota. Los miembros del grupo de administradores locales también tienen acceso, aunque esas cuentas no se enumeran en el grupo local Usuarios de escritorio remoto. Para los equipos unidos a un dominio, el grupo de administradores locales también contiene los administradores de dominio correspondientes al dominio.

Asegúrese de que la cuenta que usa para conectarse tiene derechos de inicio de sesión de Escritorio remoto. Como alternativa, use una cuenta de administrador local o de dominio para conectarse a través de Escritorio remoto. Use el complemento Microsoft Management Console (**Herramientas del sistema > Usuarios y grupos locales > Grupos > Usuarios de escritorio remoto**) para agregar la cuenta que quiera al grupo local de usuarios de escritorio remoto.

## <a name="next-steps"></a>Pasos siguientes
Si no se produjo ninguno de estos errores y tiene un problema desconocido al conectarse mediante RDP, consulte la [Guía de solución de problemas de Escritorio remoto](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

* Para ver los pasos sobre cómo solucionar problemas de acceso a las aplicaciones, consulte [Solucionar problemas de acceso a una aplicación que se ejecuta en una máquina virtual de Azure](./troubleshoot-app-connection.md?toc=/azure/virtual-machines/linux/toc.json).
* Si tiene problemas con Secure Shell (SSH) para conectarse a una máquina virtual Linux en Azure, consulte el artículo sobre cómo [solucionar problemas con las conexiones SSH en una máquina virtual Linux en Azure](./troubleshoot-ssh-connection.md?toc=/azure/virtual-machines/linux/toc.json).
