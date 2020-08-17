---
title: 'Asignación y enumeración de roles con ámbito de unidad administrativa (versión preliminar): Azure Active Directory | Microsoft Docs'
description: Uso de unidades administrativas para restringir el ámbito de las asignaciones de roles en Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.subservice: users-groups-roles
ms.workload: identity
ms.date: 07/10/2020
ms.author: curtand
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: 918675b111b7b1b85669692b63fed683ea2831f8
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475641"
---
# <a name="assign-scoped-roles-to-an-administrative-unit"></a>Asignación de roles con ámbito a una unidad administrativa

En Azure Active Directory (Azure AD), puede asignar usuarios a un rol de Azure AD con un ámbito limitado a una o más unidades administrativas para lograr un control administrativo más específico.

Consulte [Introducción](roles-admin-units-manage.md#get-started) para conocer los pasos previos al uso de PowerShell y Microsoft Graph para la administración de unidades administrativas.

## <a name="roles-available"></a>Roles disponibles

Role  |  Descripción
----- |  -----------
Administrador de autenticación  |  Tiene acceso para ver, configurar y restablecer la información de los métodos de autenticación de cualquier usuario que no sea administrador solo en la unidad administrativa asignada.
Administrador de grupos  |  Puede administrar todos los aspectos de los grupos y las configuraciones de grupos, como las directivas de nomenclatura y expiración solo en la unidad administrativa asignada.
Administrador del departamento de soporte técnico  |  Puede restablecer contraseñas de usuarios que no son administradores y de administradores del departamento de soporte técnico solo en la unidad administrativa asignada.
Administrador de licencias  |  Puede asignar, quitar y actualizar las asignaciones de licencia solo dentro de la unidad administrativa.
Administrador de contraseñas  |  Puede restablecer contraseñas de usuarios que no son administradores y de administradores de contraseña solo en la unidad administrativa asignada.
Administrador de usuarios  |  Puede administrar todos los aspectos de usuarios y grupos, incluido el restablecimiento de contraseñas para administradores limitados solo en la unidad administrativa asignada.

## <a name="assign-a-scoped-role"></a>Asignación de un rol con ámbito

### <a name="azure-portal"></a>Azure Portal

Vaya a **Azure AD > Unidades administrativas** en el portal. Seleccione la unidad administrativa en la que quiere asignarle el rol a un usuario. En el panel izquierdo, seleccione Roles y administradores para enumerar todos los roles disponibles.

![Seleccione la unidad administrativa a la que cambiará el ámbito de rol.](./media/roles-admin-units-assign-roles/select-role-to-scope.png)

Seleccione el rol que se va a asignar y, a continuación, seleccione **Agregar asignaciones**. Se abrirá un panel a la derecha en el que podrá seleccionar uno o varios usuarios para asignarlos al rol.

![Seleccione el rol al que se agregará el ámbito y, a continuación, seleccione Agregar asignaciones.](./media/roles-admin-units-assign-roles/select-add-assignment.png)

### <a name="powershell"></a>PowerShell

```powershell
$AdminUser = Get-AzureADUser -ObjectId "Use the user's UPN, who would be an admin on this unit"
$Role = Get-AzureADDirectoryRole | Where-Object -Property DisplayName -EQ -Value "User Account Administrator"
$administrativeUnit = Get-AzureADAdministrativeUnit -Filter "displayname eq 'The display name of the unit'"
$RoleMember = New-Object -TypeName Microsoft.Open.AzureAD.Model.RoleMemberInfo
$RoleMember.ObjectId = $AdminUser.ObjectId
Add-AzureADScopedRoleMembership -ObjectId $administrativeUnit.ObjectId -RoleObjectId $Role.ObjectId -RoleMemberInfo $RoleMember
```

La sección resaltada se puede cambiar según sea necesario para el entorno específico.

### <a name="microsoft-graph"></a>Microsoft Graph

```http
Http request
POST /administrativeUnits/{id}/scopedRoleMembers
    
Request body
{
  "roleId": "roleId-value",
  "roleMemberInfo": {
    "id": "id-value"
  }
}
```

## <a name="list-the-scoped-admins-on-an-au"></a>Enumeración de los administradores con ámbito en una unidad administrativa

### <a name="azure-portal"></a>Azure Portal

Todas las asignaciones de roles realizadas con un ámbito de unidad administrativa se pueden ver en la sección [Unidades administrativas de Azure AD](https://ms.portal.azure.com/?microsoft_aad_iam_adminunitprivatepreview=true&microsoft_aad_iam_rbacv2=true#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AdminUnit). Vaya a **Azure AD > Unidades administrativas** en el portal. Seleccione la unidad de administración de las asignaciones de roles que quiere enumerar. Seleccione **Roles y administradores** y abra un rol para ver las asignaciones en la unidad de administración.

### <a name="powershell"></a>PowerShell

```powershell
$administrativeUnit = Get-AzureADAdministrativeUnit -Filter "displayname eq 'The display name of the unit'"
Get-AzureADScopedRoleMembership -ObjectId $administrativeUnit.ObjectId | fl *
```

La sección resaltada se puede cambiar según sea necesario para el entorno específico.

### <a name="microsoft-graph"></a>Microsoft Graph

```http
Http request
GET /administrativeUnits/{id}/scopedRoleMembers
Request body
{}
```

## <a name="next-steps"></a>Pasos siguientes

- [Uso de grupos en la nube para administrar asignaciones de roles](roles-groups-concept.md)
- [Solución de problemas de roles asignados a grupos en la nube](roles-groups-faq-troubleshooting.md)
