---
title: Asignación de un rol a un grupo mediante Privileged Identity Management en Azure AD | Microsoft Docs
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
ms.openlocfilehash: 343da87048cf43c04a137376e9a7f24270ce729a
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475899"
---
# <a name="assign-a-role-to-a-group-using-privileged-identity-management"></a>Asignación de un rol a un grupo mediante Privileged Identity Management

En este artículo se describe cómo puede asignar un rol de Azure Active Directory (Azure AD) a un grupo mediante Azure AD Privileged Identity Management (PIM).

> [!NOTE]
> Debe usar la versión actualizada de Privileged Identity Management para poder asignar un grupo a un rol de Azure AD mediante PIM. Podría estar en una versión anterior de PIM si su organización de Azure AD aprovecha la API de Privileged Identity Management. Si es así, póngase en contacto con el alias pim_preview@microsoft.com para trasladar la organización y actualizar la API. Obtenga más información en el artículo sobre [roles y características de Azure AD en PIM](../privileged-identity-management/azure-ad-roles-features.md).

## <a name="using-azure-ad-admin-center"></a>Uso del Centro de administración de Azure AD

1. Inicie sesión en [Azure AD Privileged Identity Management](https://ms.portal.azure.com/?Microsoft_AAD_IAM_GroupRoles=true&Microsoft_AAD_IAM_userRolesV2=true&Microsoft_AAD_IAM_enablePimIntegration=true#blade/Microsoft_Azure_PIMCommon/CommonMenuBlade/quickStart) como administrador de roles con privilegios o administrador global de la organización.

1. Seleccione **Privileged Identity Management** > **Roles de Azure AD** > **Roles** > **Agregar asignaciones**

1. Seleccione un rol y, a continuación, seleccione un grupo. Solo se muestran los grupos que son aptos para la asignación de roles (grupos a los que se pueden asignar roles), no todos los grupos.

    ![selección del usuario al que le va a asignar el rol](./media/roles-groups-pim-eligible/select-member.png)

1. Seleccione la configuración de pertenencia deseada. En el caso de los roles que requieren activación, elija **elegible**. De forma predeterminada, la elegibilidad del usuario sería permanente, pero también podría establecer una hora de inicio y finalización de la misma. Cuando termine, pulse en Guardar y agregar para completar la asignación de roles.

    ![selección del usuario al que le va a asignar el rol](./media/roles-groups-pim-eligible/set-assignment-settings.png)

## <a name="using-powershell"></a>Usar PowerShell

### <a name="download-the-azure-ad-preview-powershell-module"></a>Descarga del módulo de versión preliminar de Azure AD PowerShell

Para instalar el módulo de Azure AD #PowerShell, use los siguientes cmdlets:

```powershell
install-module azureadpreview
import-module azureadpreview
```

A fin de comprobar que el módulo esté listo para utilizarse, use el siguiente cmdlet:

```powershell
get-module azureadpreview
```

### <a name="assign-a-group-as-an-eligible-member-of-a-role"></a>Asignación de un grupo como miembro apto de un rol

```powershell
$schedule = New-Object Microsoft.Open.MSGraph.Model.AzureADMSPrivilegedSchedule
$schedule.Type = "Once"
$schedule.StartDateTime = "2019-04-26T20:49:11.770Z"
$schedule.endDateTime = "2019-07-25T20:49:11.770Z"
Open-AzureADMSPrivilegedRoleAssignmentRequest -ProviderId aadRoles -Schedule $schedule -ResourceId "[YOUR TENANT ID]" -RoleDefinitionId "9f8c1837-f885-4dfd-9a75-990f9222b21d" -SubjectId "[YOUR GROUP ID]" -AssignmentState "Eligible" -Type "AdminAdd"
```

## <a name="using-microsoft-graph-api"></a>Uso de Microsoft Graph API

```powershell
POST
https://graph.microsoft.com/beta/privilegedAccess/aadroles/roleAssignmentRequests  

{

 "roleDefinitionId": {roleDefinitionId},

 "resourceId": {tenantId},

 "subjectId": {GroupId},

 "assignmentState": "Eligible",

 "type": "AdminAdd",

 "reason": "reason string",

 "schedule": {

   "startDateTime": {DateTime},

   "endDateTime": {DateTime},

   "type": "Once"

 }

}
```

## <a name="next-steps"></a>Pasos siguientes

- [Uso de grupos en la nube para administrar asignaciones de roles](roles-groups-concept.md)
- [Solución de problemas de roles asignados a grupos en la nube](roles-groups-faq-troubleshooting.md)
- [Configuración del rol de administrador de Azure AD en Privileged Identity Management](../privileged-identity-management/pim-how-to-change-default-settings.md)
- [Asignación de roles de recursos de Azure en Privileged Identity Management](../privileged-identity-management/pim-resource-roles-assign-roles.md)
