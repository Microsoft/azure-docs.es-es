---
title: 'Asignación de roles personalizados con Azure PowerShell: Azure AD | Microsoft Docs'
description: Administre los miembros de un rol personalizado de administrador de Azure con Azure PowerShell.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: how-to
ms.date: 04/29/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 243bc48f7e84951c78ecab3e7f5ee47a85a538bf
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798829"
---
# <a name="assign-custom-roles-with-resource-scope-using-powershell-in-azure-active-directory"></a>Asignación de roles personalizados con ámbito de recurso mediante PowerShell en Azure Active Directory

En este artículo se describe cómo crear una asignación de roles en un ámbito de toda la organización en Azure Active Directory (Azure AD). La asignación de un rol en un ámbito de toda la organización concede acceso a toda la organización de Azure AD. Para crear una asignación de roles con un ámbito de un solo recurso de Azure AD, consulte [cómo crear un rol personalizado y asignarlo en un ámbito de recurso](roles-create-custom.md).En este artículo se usa el módulo [Azure Active Directory PowerShell, versión 2](/powershell/module/azuread/?view=azureadps-2.0#directory_roles).

Para más información sobre los roles de administración de Azure AD, consulte  [Asignación de roles de administrador en Azure Active Directory](directory-assign-admin-roles.md).

## <a name="required-permissions"></a>Permisos necesarios

Conéctese a la organización de Azure AD con una cuenta de administrador global para asignar o eliminar roles.

## <a name="prepare-powershell"></a>Preparación para PowerShell

Instale el módulo de Azure AD PowerShell desde la [Galería de PowerShell](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.17). Después, importe el módulo de versión preliminar de Azure AD PowerShell con el comando siguiente:

``` PowerShell
import-module azureadpreview
```

Para comprobar que el módulo está listo para su uso, haga coincidir la versión devuelta por el siguiente comando con la que aparece aquí:

``` PowerShell
get-module azureadpreview
  ModuleType Version      Name                         ExportedCommands
  ---------- ---------    ----                         ----------------
  Binary     2.0.0.115    azureadpreview               {Add-AzureADMSAdministrati...}
```

Ahora puede empezar a usar los cmdlets del módulo. Para una descripción completa de los cmdlets del módulo de Azure AD, consulte la documentación de referencia en línea sobre el [módulo de versión preliminar de Azure AD](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.17).

## <a name="assign-a-role-to-a-user-or-service-principal-with-resource-scope"></a>Asignación de un rol a un usuario o entidad de servicio con ámbito de recurso

1. Abra el módulo de versión preliminar de Azure AD PowerShell.
1. Para iniciar sesión, ejecute el comando `Connect-AzureAD`.
1. Cree un nuevo rol mediante el siguiente script de PowerShell.

``` PowerShell
## Assign a role to a user or service principal with resource scope
# Get the user and role definition you want to link
$user = Get-AzureADUser -Filter "userPrincipalName eq 'cburl@f128.info'"
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Application Support Administrator'"

# Get app registration and construct resource scope for assignment.
$appRegistration = Get-AzureADApplication -Filter "displayName eq 'f/128 Filter Photos'"
$resourceScope = '/' + $appRegistration.objectId

# Create a scoped role assignment
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope $resourceScope -RoleDefinitionId $roleDefinition.Id -PrincipalId $user.objectId
```

Para asignar el rol a una entidad de servicio en lugar de a un usuario, use el [cmdlet Get-AzureADMSServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal?view=azureadps-2.0).

## <a name="operations-on-roledefinition"></a>Operaciones en RoleDefinition

Los objetos de definición de roles contienen la definición del rol integrado o personalizado, junto con los permisos que dicha asignación de roles concede. Este recurso muestra tanto las definiciones de roles personalizados como los roles directoryRoles integrados (que se muestran en el formulario equivalente de roleDefinition). En la actualidad, una organización de Azure AD puede tener un máximo de 30 RoleDefinitions personalizados únicos definidos.

### <a name="create-operations-on-roledefinition"></a>Operaciones de creación en RoleDefinition

``` PowerShell
# Basic information
$description = "Can manage credentials of application registrations"
$displayName = "Application Registration Credential Administrator"
$templateId = (New-Guid).Guid

# Set of actions to grant
$allowedResourceAction =
@(
    "microsoft.directory/applications/standard/read",
    "microsoft.directory/applications/credentials/update"
)
$rolePermissions = @{'allowedResourceActions'= $allowedResourceAction}

# Create new custom admin role
$customAdmin = New-AzureADMSRoleDefinition -RolePermissions $rolePermissions -DisplayName $displayName -Description $description -TemplateId $templateId -IsEnabled $true
```

### <a name="read-operations-on-roledefinition"></a>Operaciones de lectura en RoleDefinition

``` PowerShell
# Get all role definitions
Get-AzureADMSRoleDefinitions

# Get single role definition by objectId
Get-AzureADMSRoleDefinition -Id 86593cfc-114b-4a15-9954-97c3494ef49b

# Get single role definition by templateId
Get-AzureADMSRoleDefinition -Filter "templateId eq 'c4e39bd9-1100-46d3-8c65-fb160da0071f'"
```

### <a name="update-operations-on-roledefinition"></a>Operaciones de actualización en RoleDefinition

``` PowerShell
# Update role definition
# This works for any writable property on role definition. You can replace display name with other
# valid properties.
Set-AzureADMSRoleDefinition -Id c4e39bd9-1100-46d3-8c65-fb160da0071f -DisplayName "Updated DisplayName"
```

### <a name="delete-operations-on-roledefinition"></a>Operaciones de eliminación en RoleDefinition

``` PowerShell
# Delete role definition
Remove-AzureADMSRoleDefinitions -Id c4e39bd9-1100-46d3-8c65-fb160da0071f
```

## <a name="operations-on-roleassignment"></a>Operaciones en RoleAssignment

Las asignaciones de roles contienen información que vincula una entidad de seguridad determinada (un usuario o una entidad de servicio de aplicación) a una definición de roles. Si es necesario, puede agregar un ámbito de recurso de Azure AD único para los permisos asignados.  Es posible restringir el ámbito de permiso para los roles integrados y los personalizados.

### <a name="create-operations-on-roleassignment"></a>Operaciones de creación en RoleAssignment

``` PowerShell
# Get the user and role definition you want to link
$user = Get-AzureADUser -Filter "userPrincipalName eq 'cburl@f128.info'"
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Application Support Administrator'"

# Get app registration and construct resource scope for assignment.
$appRegistration = Get-AzureADApplication -Filter "displayName eq 'f/128 Filter Photos'"
$resourceScope = '/' + $appRegistration.objectId

# Create a scoped role assignment
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope $resourceScope -RoleDefinitionId $roleDefinition.Id -PrincipalId $user.objectId
```

### <a name="read-operations-on-roleassignment"></a>Operaciones de lectura en RoleAssignment

``` PowerShell
# Get role assignments for a given principal
Get-AzureADMSRoleAssignment -Filter "principalId eq '27c8ca78-ab1c-40ae-bd1b-eaeebd6f68ac'"

# Get role assignments for a given role definition 
Get-AzureADMSRoleAssignment -Filter "roleDefinitionId eq '355aed8a-864b-4e2b-b225-ea95482e7570'"
```

### <a name="delete-operations-on-roleassignment"></a>Operaciones de eliminación en RoleAssignment

``` PowerShell
# Delete role assignment
Remove-AzureADMSRoleAssignment -Id 'qiho4WOb9UKKgng_LbPV7tvKaKRCD61PkJeKMh7Y458-1'
```

## <a name="next-steps"></a>Pasos siguientes

- Comparta con nosotros en el [foro de roles administrativos de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).
- Para más información sobre los roles y las asignaciones de roles de administrador de Azure AD, consulte [Asignación de roles de administrador](directory-assign-admin-roles.md).
- Para conocer los permisos predeterminados de usuario, vea una [comparación de los permisos predeterminados de usuario miembro e invitado](../fundamentals/users-default-permissions.md).