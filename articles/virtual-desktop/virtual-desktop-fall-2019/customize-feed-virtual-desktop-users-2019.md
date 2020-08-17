---
title: Personalización de fuente para usuarios de Windows Virtual Desktop (clásico) - Azure
description: Personalización de la fuente para los usuarios de Windows Virtual Desktop (clásico) con cmdlets de PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 3a8c1a5da4f4d46d98e01eeb0bf210567af19a95
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88007240"
---
# <a name="customize-feed-for-windows-virtual-desktop-classic-users"></a>Personalización de fuente para usuarios de Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager, consulte [este artículo](../customize-feed-for-virtual-desktop-users.md).

Puede personalizar la fuente para que RemoteApp y los recursos de escritorio remoto se muestren de manera reconocible para los usuarios.

En primer lugar y, si aún no lo ha hecho, [descargue e importe el módulo de PowerShell para Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/) que se usará en la sesión de PowerShell. Después, ejecute el siguiente cmdlet para iniciar sesión en su cuenta:

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="customize-the-display-name-for-a-remoteapp"></a>Personalizar el nombre para mostrar de RemoteApp

Para cambiar el nombre para mostrar de una instancia de RemoteApp publicada, defina un nombre descriptivo. De forma predeterminada, el nombre descriptivo es el mismo que el nombre del programa RemoteApp.

Para recuperar una lista de instancias de RemoteApp publicadas para un grupo de aplicaciones, ejecute el siguiente cmdlet de PowerShell:

```powershell
Get-RdsRemoteApp -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname>
```

> [!div class="mx-imgBorder"]
> ![Captura de pantalla del cmdlet de PowerShell Get-RDSRemoteApp con el nombre y el nombre descriptivo resaltado](../media/get-rdsremoteapp.png)

Para asignar un nombre descriptivo a RemoteApp, ejecute el siguiente cmdlet de PowerShell:

```powershell
Set-RdsRemoteApp -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname> -Name <existingappname> -FriendlyName <newfriendlyname>
```

> [!div class="mx-imgBorder"]
> ![Captura de pantalla del cmdlet de PowerShell Set-RDSRemoteApp con el nombre y el nuevo nombre descriptivo resaltado](../media/set-rdsremoteapp.png)

## <a name="customize-the-display-name-for-a-remote-desktop"></a>Personalizar el nombre para mostrar de Escritorio remoto

Para cambiar el nombre para mostrar de un escritorio remoto publicado, defina un nombre descriptivo. Si ha creado manualmente un grupo de hosts y un grupo de aplicaciones de escritorio mediante PowerShell, el nombre descriptivo predeterminado es "Session Desktop". Si ha creado un grupo de hosts y un grupo de aplicaciones de escritorio mediante la plantilla de Azure Resource Manager de GitHub o la oferta de Azure Marketplace, el nombre descriptivo predeterminado coincide con el nombre del grupo de hosts.

Para recuperar el recurso de escritorio remoto, ejecute el siguiente cmdlet de PowerShell:

```powershell
Get-RdsRemoteDesktop -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname>
```

> [!div class="mx-imgBorder"]
> ![Captura de pantalla del cmdlet de PowerShell Get-RDSRemoteApp con el nombre y el nombre descriptivo resaltado](../media/get-rdsremotedesktop.png)

Para asignar un nombre descriptivo al recurso de escritorio remoto, ejecute el siguiente cmdlet de PowerShell:

```powershell
Set-RdsRemoteDesktop -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname> -FriendlyName <newfriendlyname>
```

> [!div class="mx-imgBorder"]
> ![Captura de pantalla del cmdlet de PowerShell Set-RDSRemoteApp con el nombre y el nuevo nombre descriptivo resaltado](../media/set-rdsremotedesktop.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha personalizado la fuente para los usuarios, puede iniciar sesión en un cliente de Windows Virtual Desktop para probarlo. Para ello, continúe para conectarse a los procedimientos para conectarse a Windows Virtual Desktop:

 * [Conexión desde Windows 10 o Windows 7](connect-windows-7-10-2019.md)
 * [Conexión desde un explorador web](connect-web-2019.md)
