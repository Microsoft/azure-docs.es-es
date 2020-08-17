---
title: Visualización de los roles asignados a un grupo en Azure Active Directory | Microsoft Docs
description: Versión preliminar de los roles personalizados de Azure AD para delegar la administración de identidades. Administre roles de Azure en Azure Portal, PowerShell o Graph API.
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
ms.openlocfilehash: 3b7613fb70299a70e4389b97c2647a26cb7c3374
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475897"
---
# <a name="view-roles-assigned-to-a-group-in-azure-active-directory"></a>Visualización de los roles asignados a un grupo en Azure Active Directory

En esta sección se describe cómo se pueden ver los roles asignados a un grupo mediante el centro de administración de Azure AD. La visualización de los grupos y los roles asignados son los permisos de usuario predeterminados.

1. Inicie sesión en el [centro de administración de Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) con credenciales que no sean de administrador o que sí lo sean.

1. Seleccione el grupo que le interese.

1. Seleccione **Roles asignados**. Así podrá ver todos los roles de Azure AD asignados a este grupo.

   ![Visualización de todos los roles asignados a un grupo seleccionado](./media/roles-groups-view-assignments/view-assignments.png)

## <a name="using-powershell"></a>Usar PowerShell

### <a name="get-object-id-of-the-group"></a>Obtención del id. de objeto del grupo

```powershell
Get-AzureADMSGroup -SearchString “Contoso_Helpdesk_Administrators”
```

### <a name="view-role-assignment-to-a-group"></a>Visualización de la asignación de roles a un grupo

```powershell
Get-AzureADMSRoleAssignment -Filter "principalId eq '<object id of group>" 
```

## <a name="using-microsoft-graph-api"></a>Uso de Microsoft Graph API

### <a name="get-object-id-of-the-group"></a>Obtención del id. de objeto del grupo

```powershell
GET https://graph.microsoft.com/beta/groups?$filter displayName eq ‘Contoso_Helpdesk_Administrator’ 
```

### <a name="get-role-assignments-to-a-group"></a>Obtención de la asignación de roles a un grupo

```powershell
GET https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments?$filter=principalId eq
```

## <a name="next-steps"></a>Pasos siguientes

- [Uso de grupos en la nube para administrar asignaciones de roles](roles-groups-concept.md)
- [Solución de problemas de roles asignados a grupos en la nube](roles-groups-faq-troubleshooting.md)
