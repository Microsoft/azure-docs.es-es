---
title: 'PowerShell para roles de Azure AD en PIM: Azure AD | Microsoft Docs'
description: Administre los roles de Azure AD con cmdlets de PowerShell en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: pim
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/11/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6792fdc405d539a662c8dc20c04b2891fd036704
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421916"
---
# <a name="powershell-for-azure-ad-roles-in-privileged-identity-management"></a>PowerShell para roles de Azure AD en Privileged Identity Management

En este artículo se incluyen instrucciones para usar los cmdlets de PowerShell de Azure Active Directory (Azure<AD) para administrar los roles de Azure AD en Privileged Identity Management (PIM). También se indica cómo configurar el módulo de Azure AD PowerShell.

> [!Note]
> PowerShell oficial solo es compatible si se usa la versión nueva de Azure AD Privileged Identity Management. Vaya a Privileged Identity Management y asegúrese de que aparece el banner siguiente en la hoja de inicio rápido.
> [![Compruebe la versión de Privileged Identity Management que tiene](media/pim-how-to-add-role-to-user/pim-new-version.png "Seleccionar Azure AD > Privileged Identity Management")](media/pim-how-to-add-role-to-user/pim-new-version.png#lightbox) Si no aparece este banner, tendrá que esperar porque estamos en proceso de implementar esta experiencia actualizada en las próximas semanas.
> Los cmdlets de PowerShell de Privileged Identity Management se admiten a través del módulo de la versión preliminar de Azure AD. Si ha usado otro módulo y ese módulo ahora devuelve un mensaje de error, empiece a usar este módulo nuevo. Si tiene algún sistema de producción basado en un módulo distinto, póngase en contacto con pim_preview@microsoft.com.

## <a name="installation-and-setup"></a>Instalación y configuración

1. Instale el módulo de versión preliminar de Azure AD.

    ```powershell
    Install-module AzureADPreview
    ```

1. Asegúrese de tener los permisos de rol necesarios antes de continuar. Si intenta llevar a cabo tareas de administración, como una asignación de roles o la actualización de la configuración de roles, asegúrese de que tiene el rol Administrador global o Administrador de roles con privilegios. Si solo intenta activar su propia asignación, no se requiere ningún permiso más allá de los permisos de usuario predeterminados.

1. Conéctese a Azure AD.

    ```powershell
    $AzureAdCred = Get-Credential  
    Connect-AzureAD -Credential $AzureAdCred
    ```

1. Busque el id. de inquilino de la organización de Azure AD; para ello, vaya a **Azure Active Directory** > **Propiedades** > **Identificador de directorio**. En la sección de cmdlets, use este identificador siempre que necesite proporcionar el parámetro resourceId.

    ![Busque el id. de organización en las propiedades de la organización de Azure AD](./media/powershell-for-azure-ad-roles/tenant-id-for-Azure-ad-org.png)

> [!Note]
> Las secciones siguientes son ejemplos sencillos que pueden ayudarlo a ponerse en marcha. Puede encontrar documentación más detallada sobre los cmdlets siguientes en https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0-preview#privileged_role_management. Sin embargo, tendrá que reemplazar "azureResources" en el parámetro providerID por "aadRoles". También debe recordar usar el id. de la organización de Azure AD como parámetro resourceId.

## <a name="retrieving-role-definitions"></a>Recuperación de las definiciones de roles

Use el cmdlet siguiente para obtener todos los roles de Azure AD integrados y personalizados en su organización de Azure AD. Este paso importante le proporciona la asignación entre el nombre de rol y roleDefinitionId. El parámetro roleDefinitionId se usa en estos cmdlets para hacer referencia a un rol específico.

El parámetro roleDefinitionId es específico de su organización de Azure AD y es distinto del parámetro roleDefinitionId devuelto por la API de administración de roles.

```powershell
Get-AzureADMSPrivilegedRoleDefinition -ProviderId aadRoles -ResourceId 926d99e7-117c-4a6a-8031-0cc481e9da26
```

Resultado:

![Obtener todos los roles de la organización de Azure AD](./media/powershell-for-azure-ad-roles/get-all-roles-result.png)

## <a name="retrieving-role-assignments"></a>Eliminación de asignaciones de roles

Use el cmdlet siguiente para recuperar todas las asignaciones de roles de su organización de Azure AD.

```powershell
Get-AzureADMSPrivilegedRoleAssignment -ProviderId "aadRoles" -ResourceId "926d99e7-117c-4a6a-8031-0cc481e9da26"
```

Use el cmdlet siguiente para recuperar todas las asignaciones de roles para un usuario determinado. Esta lista también se conoce como "Mis roles" en el portal de Azure AD. La única diferencia es que se agregó un filtro para el identificador del firmante. El identificador del firmante en este contexto es el identificador de usuario o el identificador de grupo.

```powershell
Get-AzureADMSPrivilegedRoleAssignment -ProviderId "aadRoles" -ResourceId "926d99e7-117c-4a6a-8031-0cc481e9da26" -Filter "subjectId eq 'f7d1887c-7777-4ba3-ba3d-974488524a9d'" 
```

Use el cmdlet siguiente para recuperar todas las asignaciones de roles para un rol determinado. El parámetro roleDefinitionId es el identificador que devuelve el cmdlet anterior.

```powershell
Get-AzureADMSPrivilegedRoleAssignment -ProviderId "aadRoles" -ResourceId "926d99e7-117c-4a6a-8031-0cc481e9da26" -Filter "roleDefinitionId eq '0bb54a22-a3df-4592-9dc7-9e1418f0f61c'"
```

Los cmdlets dan como resultado una lista de objetos de asignación de roles que se muestran a continuación. El identificador del firmante es el identificador de usuario del usuario al que se le asigna el rol. El estado de la asignación puede ser activo o elegible. Si el usuario está activo y hay un identificador en el campo LinkedEligibleRoleAssignmentId, significa que el rol actualmente está activado.

Resultado:

![Recuperar todas las asignaciones de roles para la organización de Azure AD](./media/powershell-for-azure-ad-roles/get-all-role-assignments-result.png)

## <a name="assign-a-role"></a>Asignar un rol

Use el cmdlet siguiente para crear una asignación elegible.

```powershell
Open-AzureADMSPrivilegedRoleAssignmentRequest -ProviderId 'aadRoles' -ResourceId '926d99e7-117c-4a6a-8031-0cc481e9da26' -RoleDefinitionId 'ff690580-d1c6-42b1-8272-c029ded94dec' -SubjectId 'f7d1887c-7777-4ba3-ba3d-974488524a9d' -Type 'adminAdd' -AssignmentState 'Eligible' -schedule $schedule -reason "dsasdsas" 
```

La programación, que define la hora de inicio y de finalización de la asignación, es un objeto que se puede crear como en el ejemplo siguiente:

```powershell
$schedule = New-Object Microsoft.Open.MSGraph.Model.AzureADMSPrivilegedSchedule
$schedule.Type = "Once"
$schedule.StartDateTime = (Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ss.fffZ")
$schedule.endDateTime = "2020-07-25T20:49:11.770Z"
```
> [!Note]
> Si el valor de endDateTime se establece en null, indica una asignación permanente.

## <a name="activate-a-role-assignment"></a>Activación de una asignación de roles

Use el cmdlet siguiente para activar una asignación elegible.

```powershell
Open-AzureADMSPrivilegedRoleAssignmentRequest -ProviderId 'aadRoles' -ResourceId '926d99e7-117c-4a6a-8031-0cc481e9da26' -RoleDefinitionId 'f55a9a68-f424-41b7-8bee-cee6a442d418' -SubjectId 'f7d1887c-7777-4ba3-ba3d-974488524a9d' -Type 'UserAdd' -AssignmentState 'Active' -schedule $schedule -reason "dsasdsas"
``` 

Este cmdlet es casi idéntico al cmdlet para crear una asignación de roles. La diferencia clave entre los cmdlets es que, para el parámetro –Type, la activación es "userAdd" en lugar de "adminAdd". La otra diferencia es que el parámetro –AssignmentState es "Active" (Activo) en lugar de "Eligible" (Elegible).

> [!Note]
> Hay dos escenarios limitantes para la activación de roles a través de PowerShell.
> 1. Si necesita un sistema de vales o un número de vale en la configuración del rol, no hay forma de proporcionarlos como parámetro. Por lo tanto, no sería posible activar el rol más allá de Azure Portal. Esta característica se implementará en PowerShell en los próximos meses.
> 1. Si necesita la autenticación multifactor para la activación de roles, actualmente no hay ninguna manera de que PowerShell desafíe al usuario cuando este activa su rol. En lugar de eso, los usuarios deberán desencadenar el desafío de MFA cuando se conectan a Azure AD según las instrucciones que aparecen en [esta entrada de blog](http://www.anujchaudhary.com/2020/02/connect-to-azure-ad-powershell-with-mfa.html) de uno de nuestros ingenieros. Si desarrolla una aplicación para PIM, una implementación posible es desafiar a los usuarios y volver a conectarlos con el módulo una vez que reciban un error "MfaRule".

## <a name="retrieving-and-updating-role-settings"></a>Recuperación y actualización de la configuración de roles

Use el cmdlet siguiente para obtener la configuración de todos los roles de la organización de Azure AD.

```powershell
Get-AzureADMSPrivilegedRoleSetting -ProviderId 'aadRoles' -Filter "ResourceId eq '926d99e7-117c-4a6a-8031-0cc481e9da26'" 
```

Hay cuatro objetos principales en la configuración. PIM solo usa tres de estos objetos. UserMemberSettings es la configuración de la activación, AdminEligibleSettings es la configuración de asignación de las asignaciones elegibles y AdminmemberSettings es la configuración de asignación de las asignaciones activas.

[![Obtención y actualización de la configuración de roles.](media/powershell-for-azure-ad-roles/get-update-role-settings-result.png)](media/powershell-for-azure-ad-roles/get-update-role-settings-result.png#lightbox)

Para actualizar la configuración de rol, debe obtener el objeto de configuración existente para un rol determinado y realizar cambios en él:

```powershell
$setting = Get-AzureADMSPrivilegedRoleSetting -ProviderId 'aadRoles' -Filter "roleDefinitionId eq 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'"
$setting.UserMemberSetting.justificationRule = '{"required":false}'
```

Luego, puede continuar y aplicar la configuración a uno de los objetos de un rol determinado, como se muestra a continuación. El identificador aquí es el identificador de la configuración de rol que se puede recuperar del resultado del cmdlet de configuración de roles de lista.

```powershell
Set-AzureADMSPrivilegedRoleSetting -ProviderId 'aadRoles' -Id 'ff518d09-47f5-45a9-bb32-71916d9aeadf' -ResourceId '3f5887ed-dd6e-4821-8bde-c813ec508cf9' -RoleDefinitionId '2387ced3-4e95-4c36-a915-73d803f93702' -UserMemberSettings $setting 
```

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de un rol personalizado de Azure AD](azure-ad-custom-roles-assign.md)
- [Eliminación o actualización de una asignación de roles personalizados de Azure AD](azure-ad-custom-roles-update-remove.md)
- [Configuración de una asignación de roles personalizados de Azure AD](azure-ad-custom-roles-configure.md)
- [Definiciones de roles en Azure AD](../users-groups-roles/directory-assign-admin-roles.md)
