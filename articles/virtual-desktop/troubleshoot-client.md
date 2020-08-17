---
title: 'Solución de problemas del cliente de Escritorio remoto de Windows Virtual Desktop: Azure'
description: Cómo resolver problemas al configurar conexiones de cliente en un entorno de inquilinos de Windows Virtual Desktop.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: troubleshooting
ms.date: 03/31/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f91e68ec2bd4b0b5400ee3e8e380d91ea6f31f36
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87291330"
---
# <a name="troubleshoot-the-remote-desktop-client"></a>Solución de problemas del cliente de Escritorio remoto

En este artículo se describen los problemas comunes relacionados con el cliente de Escritorio remoto y cómo corregirlos.

## <a name="remote-desktop-client-for-windows-7-or-windows-10-stops-responding-or-cannot-be-opened"></a>El cliente de Escritorio remoto para Windows 7 o Windows 10 deja de responder o no se puede abrir

A partir de la versión 1.2.790, puede restablecer los datos de usuario desde la página Acerca de o mediante un comando.

Use el siguiente comando para quitar los datos de usuario, restaurar la configuración predeterminada y cancelar la suscripción a todas las áreas de trabajo.

```cmd
msrdcw.exe /reset [/f]
```

Si usa una versión anterior del cliente de Escritorio remoto, se recomienda desinstalar y volver a instalar el cliente.

## <a name="web-client-wont-open"></a>El cliente web no se abre

En primer lugar, pruebe la conexión a Internet. Para ello, abra otro sitio web en el explorador; por ejemplo, [www.bing.com](https://www.bing.com).

Use **nslookup** para confirmar que DNS puede resolver el FQDN:

```cmd
nslookup rdweb.wvd.microsoft.com
```

Intente conectarse con otro cliente, como el cliente de Escritorio remoto para Windows 7 o Windows 10, y compruebe si puede abrir el cliente web.

### <a name="opening-another-site-fails"></a>error al abrir otro sitio

Esto se debe normalmente a problemas de conexión de red o a una interrupción de la red. Se recomienda que se ponga en contacto con el soporte técnico de red.

### <a name="nslookup-cannot-resolve-the-name"></a>Nslookup no puede resolver el nombre

Esto se debe normalmente a problemas de conexión de red o a una interrupción de la red. Se recomienda que se ponga en contacto con el soporte técnico de red.

### <a name="your-client-cant-connect-but-other-clients-on-your-network-can-connect"></a>El cliente no se puede conectar, pero otros clientes de la red sí pueden

Si el explorador comienza a dar problemas o deja de funcionar mientras usa el cliente web, siga estas instrucciones para solucionar el problema:

1. Reinicie el explorador.
2. Borre las cookies del explorador. Consulte [Cómo eliminar archivos de cookies en Internet Explorer](https://support.microsoft.com/help/278835/how-to-delete-cookie-files-in-internet-explorer).
3. Borre la memoria caché del explorador Consulte [Borrar la memoria caché del explorador](https://binged.it/2RKyfdU).
4. Abra el explorador en modo privado.

## <a name="web-client-does-not-show-my-resources"></a>El cliente web no muestra mis recursos

Primero compruebe la cuenta de Azure Active Directory que está usando. Si ya ha iniciado sesión con una cuenta de Azure Active Directory diferente a la que quiere usar para Windows Virtual Desktop, debe cerrar la sesión o usar una ventana privada del explorador.

Si usa Windows Virtual Desktop (clásico), use el vínculo del cliente web de [este artículo](./virtual-desktop-fall-2019/connect-web-2019.md) para conectarse a los recursos.

## <a name="web-client-stops-responding-or-disconnects"></a>El cliente web deja de responder o se desconecta

Intente conectarse con otro explorador o cliente.

### <a name="other-browsers-and-clients-also-malfunction-or-fail-to-open"></a>Otros exploradores y clientes también funcionan incorrectamente o no se abren

Si los problemas continúan incluso después de haber cambiado de explorador, es posible que el problema no se deba al explorador, sino a la red. Se recomienda que se ponga en contacto con el soporte técnico de red.

## <a name="web-client-keeps-prompting-for-credentials"></a>El cliente web sigue solicitando credenciales

Si el cliente web sigue solicitando las credenciales, siga estas instrucciones:

1. Confirme que la dirección URL del cliente web es correcta.
2. Confirme que las credenciales que está usando son para el entorno de Windows Virtual Desktop asociado a la dirección URL.
3. Borre las cookies del explorador. Para obtener más detalles, consulte [Cómo eliminar archivos de cookies en Internet Explorer](https://support.microsoft.com/help/278835/how-to-delete-cookie-files-in-internet-explorer).
4. Borre la memoria caché del explorador Para obtener más detalles, consulte [Borrar la memoria caché del explorador](https://binged.it/2RKyfdU).
5. Abra el explorador en modo privado.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre cómo solucionar problemas de Windows Virtual Desktop y las pistas de escalación, consulte [Introducción, comentarios y soporte técnico para solucionar problemas](troubleshoot-set-up-overview.md).
- Para solucionar problemas durante la creación de un entorno de Windows Virtual Desktop y de un grupo de hosts de este, consulte [Creación de entorno y grupo de hosts](troubleshoot-set-up-issues.md).
- Para solucionar problemas al configurar una máquina virtual (VM) en Windows Virtual Desktop, consulte [Configuración de la máquina virtual del host de sesión](troubleshoot-vm-configuration.md).
- Para solucionar problemas al usar PowerShell con Windows Virtual Desktop, consulte [PowerShell para Windows Virtual Desktop](troubleshoot-powershell.md).
- Para realizar un tutorial de solución de problemas, consulte [Tutorial: Solución de problemas de las implementaciones de plantillas de Resource Manager](../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
