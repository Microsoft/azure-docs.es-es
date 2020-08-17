---
title: 'Acceso delegado en Windows Virtual Desktop: Azure'
description: Se describe cómo delegar funcionalidades administrativas en una implementación de Windows Virtual Desktop y se incluyen ejemplos.
author: Heidilohr
ms.topic: conceptual
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f2aa2c74704cf89c082d2837b39e82902efa0a62
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88010062"
---
# <a name="delegated-access-in-windows-virtual-desktop"></a>Acceso delegado en Windows Virtual Desktop

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop de Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/delegated-access-virtual-desktop-2019.md).

Windows Virtual Desktop presenta un modelo de acceso delegado que permite definir la cantidad de acceso que puede tener un usuario determinado mediante la asignación de un rol. Una asignación de roles consta de tres componentes: entidad de seguridad, definición de rol y ámbito. El modelo de acceso delegado de Windows Virtual Desktop se basa en el modelo de Azure RBAC. Para más información sobre asignaciones de roles específicas y sus componentes, consulte la [información general sobre el control de acceso basado en roles de Azure](../role-based-access-control/built-in-roles.md).

El acceso delegado de Windows Virtual Desktop admite los siguientes valores para cada elemento de la asignación de roles:

* Entidad de seguridad
    * Usuarios
    * Grupos de usuarios
    * Entidades de servicio
* Definición de roles
    * Roles integrados
    * Roles personalizados
* Ámbito
    * Grupos de host
    * Grupos de aplicaciones
    * Áreas de trabajo

## <a name="powershell-cmdlets-for-role-assignments"></a>Cmdlets de PowerShell para asignaciones de roles

Antes de empezar, asegúrese de seguir las instrucciones que se indican en [Configuración del módulo de PowerShell](powershell-module.md) para configurar el módulo de PowerShell de Windows Virtual Desktop, en caso de que no lo haya hecho aún.

Windows Virtual Desktop usa el control de acceso basado en rol de Azure (Azure RBAC) al publicar grupos de aplicaciones para usuarios o grupos de usuarios. El rol Usuario de la virtualización del escritorio se asigna al usuario o grupo de usuarios, y el ámbito es el grupo de aplicaciones. Este rol concede al usuario un acceso especial a los datos en el grupo de aplicaciones.

Ejecute el siguiente cmdlet para agregar usuarios de Azure Active Directory a un grupo de aplicaciones:

```powershell
New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <appgroupname> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

Ejecute el siguiente cmdlet para agregar grupos de usuarios de Azure Active Directory a un grupo de aplicaciones:

```powershell
New-AzRoleAssignment -ObjectId <usergroupobjectid> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <appgroupname> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

## <a name="next-steps"></a>Pasos siguientes

Para una lista más completa de cmdlets de PowerShell que puede usar cada rol, consulte la [referencia de PowerShell](/powershell/windows-virtual-desktop/overview).

Para obtener una lista completa de los roles que se admiten en Azure RBAC, consulte [Roles integrados de Azure](../role-based-access-control/built-in-roles.md).

Para tener una guía para configurar un entorno de Windows Virtual Desktop, consulte [Entorno de Windows Virtual Desktop](environment-setup.md).
