---
title: 'Administración de grupos de aplicaciones para Windows Virtual Desktop (clásico): Azure'
description: Se describe cómo configurar los inquilinos de Windows Virtual Desktop (clásico) en Azure Active Directory.
author: Heidilohr
ms.topic: tutorial
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 7ad8afc3d842a134ba09281005e35cf8cb04818a
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009076"
---
# <a name="tutorial-manage-app-groups-for-windows-virtual-desktop-classic"></a>Tutorial: Administración de grupos de aplicaciones para Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager, consulte [este artículo](../manage-app-groups.md).

El grupo de aplicaciones predeterminado creado para un nuevo grupo host de Windows Virtual Desktop también publica el escritorio completo. Además, puede crear uno o varios grupos de aplicaciones de RemoteApp para el grupo host. Siga este tutorial para crear un grupo de aplicaciones de RemoteApp y publicar aplicaciones individualesl de menú **Inicio**.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un grupo de RemoteApp
> * Conceder acceso a programas de RemoteApp.

Antes de empezar, y si aún no lo ha hecho, [descargue e importe el módulo de PowerShell para Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/) que se usará en la sesión de PowerShell. Después, ejecute el siguiente cmdlet para iniciar sesión en su cuenta:

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="create-a-remoteapp-group"></a>Creación de un grupo de RemoteApp

1. Ejecute el siguiente cmdlet de PowerShell para crear un grupo de aplicaciones de RemoteApp vacío.

   ```powershell
   New-RdsAppGroup <tenantname> <hostpoolname> <appgroupname> -ResourceType "RemoteApp"
   ```

2. (Opcional) Para comprobar que se creó el grupo de aplicaciones, puede ejecutar el siguiente cmdlet para ver una lista de todos los grupos de aplicaciones del grupo host.

   ```powershell
   Get-RdsAppGroup <tenantname> <hostpoolname>
   ```

3. Ejecute el siguiente cmdlet para obtener una lista de aplicaciones del menú **Inicio** en la imagen de máquina virtual del grupo host. Anote los valores de **FilePath**, **IconPath**, **IconIndex** y otra información importante para la aplicación que quiere publicar.

   ```powershell
   Get-RdsStartMenuApp <tenantname> <hostpoolname> <appgroupname>
   ```

4. Ejecute el siguiente cmdlet para instalar la aplicación según `AppAlias`. `AppAlias` se vuelve visible cuando se ejecuta la salida del paso 3.

   ```powershell
   New-RdsRemoteApp <tenantname> <hostpoolname> <appgroupname> -Name <remoteappname> -AppAlias <appalias>
   ```

5. (Opcional) Ejecute el siguiente cmdlet para publicar un nuevo programa de RemoteApp en el grupo de aplicaciones creado en el paso 1.

   ```powershell
   New-RdsRemoteApp <tenantname> <hostpoolname> <appgroupname> -Name <remoteappname> -Filepath <filepath>  -IconPath <iconpath> -IconIndex <iconindex>
   ```

6. Para comprobar que se publicó la aplicación, ejecute el siguiente cmdlet.

   ```powershell
   Get-RdsRemoteApp <tenantname> <hostpoolname> <appgroupname>
   ```

7. Repita los pasos del 1 al 5 para cada aplicación de este grupo de aplicaciones que quiera publicar.
8. Ejecute el siguiente cmdlet para conceder a los usuarios acceso a los programas de RemoteApp del grupo de aplicaciones.

   ```powershell
   Add-RdsAppGroupUser <tenantname> <hostpoolname> <appgroupname> -UserPrincipalName <userupn>
   ```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a crear un grupo de aplicaciones, rellenarlo con programas de RemoteApps y asignar usuarios al grupo de aplicaciones. Para información sobre cómo crear un grupo de hosts de validación, consulte el tutorial siguiente. Puede usar un grupo de hosts de validación para supervisar las actualizaciones del servicio antes de implementarlas en el entorno de producción.

> [!div class="nextstepaction"]
> [Creación de un grupo de hosts para validar las actualizaciones del servicio](create-validation-host-pool-2019.md)
