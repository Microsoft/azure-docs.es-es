---
title: Eliminación de asignaciones de rol de un grupo en Azure Active Directory | Microsoft Docs
description: Versión preliminar de los roles personalizados de Azure AD para delegar la administración de identidades. Administre roles de Azure en Azure Portal, PowerShell o Graph API.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: article
ms.date: 11/05/2020
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 78ed23f563fce9760768a99e5bbf58f68500d665
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "103012027"
---
# <a name="remove-role-assignments-from-a-group-in-azure-active-directory"></a>Eliminación de asignaciones de rol de un grupo en Azure Active Directory

En este artículo se describe cómo un administrador de TI puede quitar roles de Azure AD asignados a grupos. En Azure Portal, ahora puede eliminar asignaciones de roles directas e indirectas a un usuario. Si un usuario tiene asignado un rol mediante la pertenencia a un grupo, elimine el usuario del grupo para quitar la asignación de roles.

## <a name="using-azure-admin-center"></a>Uso del centro de administración de Azure

1. Inicie sesión en el [centro de administración de Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) con permisos de administrador de roles con privilegios o de administrador global en la organización de Azure AD.

1. Seleccione **Roles y administradores** > **_nombre de rol_**.

1. Seleccione el grupo del que desea eliminar la asignación de rol y, a continuación, **Quitar asignación**.

   ![Quite una asignación de roles de un grupo seleccionado.](./media/groups-remove-assignment/remove-assignment.png)

1. Cuando se le solicite confirmación, seleccione **Sí**.

## <a name="using-powershell"></a>Usar PowerShell

### <a name="create-a-group-that-can-be-assigned-to-role"></a>Creación de un grupo al que se pueda asignar un rol

```powershell
$group = New-AzureADMSGroup -DisplayName "Contoso_Helpdesk_Administrators" -Description "This group is assigned to Helpdesk Administrator built-in role in Azure AD." -MailEnabled $true -SecurityEnabled $true -MailNickName "contosohelpdeskadministrators" -IsAssignableToRole $true
```

### <a name="get-the-role-definition-you-want-to-assign-the-group-to"></a>Obtención de la definición de roles a la que desea asignar el grupo

```powershell
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Helpdesk Administrator'"
```

### <a name="create-a-role-assignment"></a>Crear una asignación de rol

```powershell
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope '/' -RoleDefinitionId $roleDefinition.Id -PrincipalId $group.objectId
```

### <a name="remove-the-role-assignment"></a>Quitar la asignación de roles

```powershell
Remove-AzureAdMSRoleAssignment -Id $roleAssignment.Id 
```

## <a name="using-microsoft-graph-api"></a>Uso de Microsoft Graph API

### <a name="create-a-group-that-can-be-assigned-an-azure-ad-role"></a>Creación de un grupo al que asignar un rol de Azure AD

```http
POST https://graph.microsoft.com/beta/groups
{
"description": "This group is assigned to Helpdesk Administrator built-in role of Azure AD",
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

```http
GET https://graph.microsoft.com/beta/roleManagement/directory/roleDefinitions?$filter=displayName+eq+'Helpdesk Administrator'
```

### <a name="create-the-role-assignment"></a>Creación de la asignación de roles

```http
POST https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments
{
"principalId":"{object-id-of-group}",
"roleDefinitionId":"{role-definition-id}",
"directoryScopeId":"/"
}
```

### <a name="delete-role-assignment"></a>Eliminación de asignaciones de roles

```http
DELETE https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments/{role-assignment-id}
```

## <a name="next-steps"></a>Pasos siguientes

- [Uso de grupos en la nube para administrar asignaciones de roles](groups-concept.md)
- [Solución de problemas de roles asignados a grupos en la nube](groups-faq-troubleshooting.md)
