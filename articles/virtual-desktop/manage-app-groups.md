---
title: 'Administración de grupos de aplicaciones para la versión preliminar de Windows Virtual Desktop: Azure'
description: Se describe cómo configurar los inquilinos de la versión preliminar de Windows Virtual Desktop en Azure Active Directory.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: tutorial
ms.date: 03/21/2019
ms.author: helohr
ms.openlocfilehash: f0cdd28be8c6e7390aa26fdc2dfbf32ec5542c2d
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233908"
---
# <a name="tutorial-manage-app-groups-for-windows-virtual-desktop-preview"></a>Tutorial: Administración de grupos de aplicaciones para la versión preliminar de Windows Virtual Desktop

El grupo de aplicaciones predeterminado creado para un nuevo grupo host de la versión preliminar de Windows Virtual Desktop también publica el escritorio completo. Además, puede crear uno o varios grupos de aplicaciones de RemoteApp para el grupo host. Siga este tutorial para crear un grupo de aplicaciones de RemoteApp y publicar aplicaciones individuales de menú Inicio.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un grupo de RemoteApp
> * Conceder acceso a aplicaciones de RemoteApp

Antes de comenzar y, si aún no lo ha hecho, [descargue e importe el módulo de PowerShell para Windows Virtual Desktop](https://docs.microsoft.com/powershell/windows-virtual-desktop/overview) que se usará en la sesión de PowerShell.

## <a name="create-a-remoteapp-group"></a>Creación de un grupo de RemoteApp

1. Ejecute el siguiente cmdlet de PowerShell para crear un grupo de aplicaciones de RemoteApp vacío.

   ```powershell
   New-RdsAppGroup <tenantname> <hostpoolname> <appgroupname> -ResourceType "RemoteApp"
   ```

2. (Opcional) Para comprobar que se creó el grupo de aplicaciones, puede ejecutar el siguiente cmdlet para ver una lista de todos los grupos de aplicaciones del grupo host.

   ```powershell
   Get-RdsAppGroup <tenantname> <hostpoolname>
   ```

3. Ejecute el siguiente cmdlet para obtener una lista de aplicaciones de menú de inicio en la imagen de máquina virtual del grupo host. Anote los valores de **FilePath**, **IconPath**, **IconIndex** y otra información importante para la aplicación que quiere publicar.

   ```powershell
   Get-RdsStartMenuApp <tenantname> <hostpoolname> <appgroupname>
   ```
   
4. Ejecute el siguiente cmdlet para instalar la aplicación según el alias de aplicación. El alias de aplicación se vuelve visible cuando se ejecuta la salida del paso 3.

   ```powershell
   New-RdsRemoteApp <tenantname> <hostpoolname> <appgroupname> -Name <remoteappname> -AppAlias <appalias>
   ```

5. (Opcional) Ejecute el siguiente cmdlet para publicar una nueva aplicación de RemoteApp en el grupo de aplicaciones creado en el paso 1.

   ```powershell
   New-RdsRemoteApp <tenantname> <hostpoolname> <appgroupname> -Name <remoteappname> -Filepath <filepath>  -IconPath <iconpath> -IconIndex <iconindex>
   ```

6. Para comprobar que se publicó la aplicación, ejecute el siguiente cmdlet.

   ```powershell
   Get-RdsRemoteApp <tenantname> <hostpoolname> <appgroupname>
   ```

7. Repita los pasos del 1 al 5 para cada aplicación de este grupo de aplicaciones que quiera publicar.
8. Ejecute el siguiente cmdlet para conceder a los usuarios acceso a las aplicaciones de RemoteApp del grupo de aplicaciones.

   ```powershell
   Add-RdsAppGroupUser <tenantname> <hostpoolname> <appgroupname> -UserPrincipalName <userupn>
   ```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a crear grupos de aplicaciones, rellenarlos con aplicaciones RemoteApps y asignar usuarios al grupo de aplicaciones. Para más información sobre el inicio de sesión en Windows Virtual Desktop, continúe con los procedimientos de conexión a Windows Virtual Desktop.

- [Conexión al cliente de Escritorio remoto en Windows 7 y Windows 10](connect-windows-7-and-10.md)
- [Conexión al cliente web de la versión preliminar de Windows Virtual Desktop](connect-web.md)
