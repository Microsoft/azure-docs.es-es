---
title: 'Solución de problemas con la conexión al servicio de Windows Virtual Desktop (clásico): Azure'
description: Cómo resolver problemas al configurar conexiones de cliente en un entorno de inquilinos de Windows Virtual Desktop (clásico).
author: Heidilohr
ms.topic: troubleshooting
ms.date: 05/20/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 8a034cb8371972c2c7908cdba4dd491c17d8cc9f
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008549"
---
# <a name="windows-virtual-desktop-classic-service-connections"></a>Conexiones al servicio de Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager, consulte [este artículo](../troubleshoot-service-connection.md).

Use este artículo para resolver problemas con conexiones de cliente de Windows Virtual Desktop.

## <a name="provide-feedback"></a>Envío de comentarios

Puede proporcionarnos comentarios y hablar sobre el servicio Windows Virtual Desktop con el equipo del producto y otros miembros activos de la comunidad en la [comunidad técnica de Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).

## <a name="user-connects-but-nothing-is-displayed-no-feed"></a>El usuario se conecta, pero no se muestra nada (ninguna fuente)

Un usuario puede iniciar clientes de Escritorio remoto y es capaz de realizar la autenticación, pero no ve ningún icono en la fuente de detección de web.

Confirme que el usuario que notificó los problemas se ha asignado a grupos de aplicaciones mediante el uso de esta línea de comandos:

```PowerShell
Get-RdsAppGroupUser <tenantname> <hostpoolname> <appgroupname>
```

Confirme que el usuario ha iniciado sesión con las credenciales correctas.

Si se usa el cliente web, confirme que no hay ningún problema de credenciales almacenadas en caché.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre cómo solucionar problemas de Windows Virtual Desktop y las pistas de escalación, consulte [Introducción, comentarios y soporte técnico para solucionar problemas](troubleshoot-set-up-overview-2019.md).
- Para solucionar problemas durante la creación de un grupo de inquilinos y de hosts en un entorno de Windows Virtual Desktop, consulte [Creación de los grupos de inquilinos y de host](troubleshoot-set-up-issues-2019.md).
- Para solucionar problemas al configurar una máquina virtual (VM) en Windows Virtual Desktop, consulte [Configuración de la máquina virtual del host de sesión](troubleshoot-vm-configuration-2019.md).
- Para solucionar problemas al usar PowerShell con Windows Virtual Desktop, consulte [PowerShell para Windows Virtual Desktop](troubleshoot-powershell-2019.md).
- Para realizar un tutorial de solución de problemas, consulte [Tutorial: Solución de problemas de las implementaciones de plantillas de Resource Manager](../../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
