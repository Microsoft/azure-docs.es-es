---
title: Asignación de un rol a un grupo en la nube en Azure Active Directory | Microsoft Docs
description: Asigne un rol de Azure AD a un grupo al que se puedan asignar roles en Azure Portal, PowerShell o Graph API.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: article
ms.date: 07/27/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: f45b5709369dfc2025b55bc3acec69e9328ce403
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513424"
---
# <a name="assign-a-role-to-a-cloud-group-in-azure-active-directory"></a>Asignación de un rol a un grupo en la nube en Azure Active Directory

En esta sección se describe cómo un administrador de TI puede asignar el rol de Azure Active Directory (Azure AD) a un grupo de Azure AD.

## <a name="using-azure-ad-admin-center"></a>Uso del centro de administración de Azure AD

La asignación de un grupo a un rol de Azure AD es similar a la asignación de usuarios y entidades de seguridad de servicio, salvo que solo se pueden usar los grupos que admiten la asignación de roles. En Azure Portal, solo se muestran los grupos a los que se pueden asignar roles.

1. Inicie sesión en el [centro de administración de Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) con permisos de administrador de roles con privilegios o de administrador global en la organización de Azure AD.

1. Seleccione los roles de **Azure Active Directory** > **y los administradores** y, a continuación, seleccione el rol que quiera asignar.

1. En la página del ***nombre de rol***, seleccione > **Agregar asignación**.

   ![Agregación de la nueva asignación de roles](./media/roles-groups-assign-role/add-assignment.png)

1. Seleccione el grupo. Solo se muestran los grupos a los que se pueden asignar roles de Azure AD.

    [![Solo se muestran los grupos con los que se puede realizar una nueva asignación de roles.](media/roles-groups-assign-role/eligible-groups.png "Solo se muestran los grupos con los que se puede realizar una nueva asignación de roles.")](media/roles-groups-assign-role/eligible-groups.png#lightbox)

1. Seleccione **Agregar**.

Para obtener más información sobre la asignación de permisos de roles, consulte [Asignar roles de administrador y no administrador a los usuarios](../fundamentals/active-directory-users-assign-role-azure-portal.md).

## <a name="using-powershell"></a>Usar PowerShell

### <a name="create-a-group-that-can-be-assigned-to-role"></a>Creación de un grupo al que se pueda asignar un rol

```powershell
$group = New-AzureADMSGroup -DisplayName "Contoso_Helpdesk_Administrators" -Description "This group is assigned to Helpdesk Administrator built-in role in Azure AD." -MailEnabled $true -SecurityEnabled $true -MailNickName "contosohelpdeskadministrators" -IsAssignableToRole $true 
```

### <a name="get-the-role-definition-for-the-role-you-want-to-assign"></a>Obtención de la definición de roles para el rol que quiera asignar

```powershell
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Helpdesk Administrator'" 
```

### <a name="create-a-role-assignment"></a>Crear una asignación de rol

```powershell
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope '/' -RoleDefinitionId $roleDefinition.Id -PrincipalId $group.Id 
```

## <a name="using-microsoft-graph-api"></a>Uso de Microsoft Graph API

### <a name="create-a-group-that-can-be-assigned-azure-ad-role"></a>Creación de un grupo al que se pueda asignar un rol de Azure AD

```
POST https://graph.microsoft.com/beta/groups
{
"description": "This group is assigned to Helpdesk Administrator built-in role of Azure AD.",
"displayName": "Contoso_Helpdesk_Administrators",
"groupTypes": [
"Unified"
],
"mailEnabled": true,
"securityEnabled": true
"mailNickname": "contosohelpdeskadministrators",
"isAssignableToRole": true,
}
```

### <a name="get-the-role-definition"></a>Obtención de la definición de rol

```
GET https://graph.microsoft.com/beta/roleManagement/directory/roleDefinitions?$filter = displayName eq ‘Helpdesk Administrator’
```

### <a name="create-the-role-assignment"></a>Creación de la asignación de roles

```
POST https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments
{
"principalId":"<Object Id of Group>",
"roleDefinitionId":"<ID of role definition>",
"directoryScopeId":"/"
}
```
## <a name="next-steps"></a>Pasos siguientes

- [Uso de grupos en la nube para administrar asignaciones de roles](roles-groups-concept.md)
- [Solución de problemas de roles asignados a grupos en la nube](roles-groups-faq-troubleshooting.md)
