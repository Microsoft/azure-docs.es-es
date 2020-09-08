---
title: 'Eliminación de los límites para la creación de registros de aplicaciones: Azure AD | Microsoft Docs'
description: Asignación de un rol personalizado para conceder registros de aplicaciones sin restricciones en Azure Active Directory
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: quickstart
ms.date: 08/07/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro, devx-track-azurepowershell
ms.collection: M365-identity-device-management
ms.openlocfilehash: d7304ecd73ed73e3ec6ae3c060fef264e016fe38
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89067814"
---
# <a name="quickstart-grant-permission-to-create-unlimited-app-registrations"></a>Inicio rápido: Concesión de permisos para crear registros de aplicaciones ilimitados

En esta guía de inicio rápido, creará un rol personalizado con permiso para crear un número ilimitado de registros de aplicaciones y luego asignará ese rol a un usuario. Después, el usuario asignado puede usar el portal de Azure AD, Azure AD PowerShell o Microsoft Graph API para crear registros de aplicaciones. A diferencia del rol Desarrollador de aplicaciones integrado, este rol personalizado permite crear un número ilimitado de registros de aplicaciones. El rol Desarrollador de aplicaciones concede esa capacidad, pero el número total de objetos creados está limitado a 250 para evitar que se alcance [la cuota de objetos en todo el directorio](directory-service-limits-restrictions.md). El rol con menos privilegios necesario para crear y asignar roles personalizados de Azure AD es el administrador de roles con privilegios.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-a-custom-role-using-the-azure-ad-portal"></a>Creación de un rol personalizado mediante el portal de Azure AD

1. Inicie sesión en el  [centro de administración de Azure AD](https://aad.portal.azure.com) con permisos de administrador de roles con privilegios o de administrador global en la organización de Azure AD.
1. Seleccione**Azure Active Directory**, elija **Roles y administradores** y, luego, seleccione **Nuevo rol personalizado**.

    ![Creación o edición de roles en la página Roles y administradores](./media/roles-create-custom/new-custom-role.png)

1. En la pestaña **Conceptos básicos**, escriba "Creador del registro de aplicaciones" para el nombre del rol y "Puede crear un número ilimitado de registros de aplicaciones" para la descripción del rol. Por último, seleccione **Siguiente**.

    ![Proporcione un nombre y una descripción para un rol personalizado en la pestaña Aspectos básicos](./media/roles-quickstart-app-registration-limits/basics-tab.png)

1. En la pestaña **Permisos**, escriba "microsoft.directory/applications/create" en el cuadro de búsqueda y luego active las casillas situadas junto a los permisos deseados. Por último, **Siguiente**.

    ![Selección de los permisos de un rol personalizado en la pestaña Permisos](./media/roles-quickstart-app-registration-limits/permissions-tab.png)

1. En la pestaña **Revisar y crear**, revise los detalles y seleccione **Crear**.

### <a name="assign-the-role-in-the-azure-ad-portal"></a>Asignación del rol en el portal de Azure AD

1. Inicie sesión en el  [centro de administración de Azure AD](https://aad.portal.azure.com) con permisos de administrador de roles con privilegios o de administrador global en la organización de Azure AD.
1. Seleccione**Azure Active Directory** y elija **Roles y administradores**.
1. Seleccione el rol Creador del registro de aplicaciones y elija **Agregar asignación**.
1. Seleccione el usuario que desee y haga clic en **Seleccionar** para agregar el usuario al rol.

¡Listo! En este inicio rápido, ha creado un rol personalizado con permiso correctamente para crear un número ilimitado de registros de aplicaciones y luego asignar ese rol a un usuario.

> [!TIP]
> Para asignar el rol a una aplicación mediante el portal de Azure AD, escriba el nombre de la aplicación en el cuadro de búsqueda de la página asignación. De forma predeterminada, las aplicaciones no se muestran en la lista, pero se devuelven en los resultados de la búsqueda.

### <a name="app-registration-permissions"></a>Permisos del registro de aplicaciones

Hay dos permisos disponibles para conceder la capacidad de crear registros de aplicaciones, cada uno con un comportamiento diferente.

- microsoft.directory/applications/createAsOwner: Al asignar este permiso, el creador se agrega como el primer propietario del registro de aplicaciones creado y el registro de aplicaciones creado contará en la cuota de objetos creados de 250 del creador.
- microsoft.directory/applicationPolicies/create: Al asignar este permiso, el creador no se agrega como el primer propietario del registro de aplicaciones creado y el registro de aplicaciones creado no contará en la cuota de objetos creados de 250 del creador. Use este permiso con precaución, ya que no hay nada que impida que la persona asignada cree registros de aplicaciones hasta que se alcance la cuota de nivel de directorio. Si se asignan ambos permisos, el permiso de creación tiene prioridad.

## <a name="create-a-custom-role-in-azure-ad-powershell"></a>Creación de un rol personalizado en PowerShell de Azure AD

### <a name="prepare-powershell"></a>Preparación para PowerShell

En primer lugar, instale el módulo de PowerShell de Azure AD desde el [Galería de PowerShell](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.17). Después, importe el módulo de versión preliminar de Azure AD PowerShell con el comando siguiente:

```powershell
import-module azureadpreview
```

Para comprobar que el módulo está listo para su uso, haga coincidir la versión devuelta por el siguiente comando con la que aparece aquí:

```powershell
get-module azureadpreview
  ModuleType Version      Name                         ExportedCommands
  ---------- ---------    ----                         ----------------
  Binary     2.0.0.115    azureadpreview               {Add-AzureADAdministrati...}
```

### <a name="create-the-custom-role-in-azure-ad-powershell"></a>Creación del rol personalizado en PowerShell de Azure AD

Cree un nuevo rol mediante el siguiente script de PowerShell:

```powershell

# Basic role information
$displayName = "Application Registration Creator"
$description = "Can create an unlimited number of application registrations."
$templateId = (New-Guid).Guid

# Set of permissions to grant
$allowedResourceAction =
@(
    "microsoft.directory/applications/create"
    "microsoft.directory/applications/createAsOwner"
)
$rolePermissions = @{'allowedResourceActions'= $allowedResourceAction}

# Create new custom admin role
$customRole = New-AzureAdMSRoleDefinition -RolePermissions $rolePermissions -DisplayName $displayName -Description $description -TemplateId $templateId -IsEnabled $true
```

### <a name="assign-the-role-in-azure-ad-powershell"></a>Asignación del rol en PowerShell de Azure AD

Asigne el rol mediante el siguiente script de PowerShell:

```powershell
# Get the user and role definition you want to link
$user = Get-AzureADUser -Filter "userPrincipalName eq 'Adam@contoso.com'"
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Application Registration Creator'"

# Get resource scope for assignment
$resourceScope = '/'

# Create a scoped role assignment
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope $resourceScope -RoleDefinitionId $roleDefinition.Id -PrincipalId $user.objectId
```

## <a name="create-a-custom-role-in-the-microsoft-graph-api"></a>Creación de un rol personalizado en Microsoft Graph API

La solicitud HTTP para crear el rol personalizado.

POST

``` HTTP
https://graph.microsoft.com/beta/roleManagement/directory/roleDefinitions
```

Body

```HTTP
{
    "description":"Can create an unlimited number of application registrations.",
    "displayName":"Application Registration Creator",
    "isEnabled":true,
    "rolePermissions":
    [
        {
            "resourceActions":
            {
                "allowedResourceActions":
                [
                    "microsoft.directory/applications/create"
                    "microsoft.directory/applications/createAsOwner"
                ]
            },
            "condition":null
        }
    ],
    "templateId":"<PROVIDE NEW GUID HERE>",
    "version":"1"
}
```

### <a name="assign-the-role-in-the-microsoft-graph-api"></a>Asignación del rol en Microsoft Graph API

La asignación de roles combina un identificador de entidad de seguridad (que puede ser un usuario o una entidad de servicio), un identificador (rol) de definición de rol y un ámbito de recurso de Azure AD.

Solicitud HTTP para asignar un rol personalizado.

POST

``` HTTP
https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments
```

Body

``` HTTP
{
    "principalId":"<PROVIDE OBJECTID OF USER TO ASSIGN HERE>",
    "roleDefinitionId":"<PROVIDE OBJECTID OF ROLE DEFINITION HERE>",
    "resourceScopes":["/"]
}
```

## <a name="next-steps"></a>Pasos siguientes

- No dude en compartir con nosotros en el [foro de roles administrativos de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).
- Para obtener más información acerca de las asignaciones de roles de Azure AD, consulte [Asignación de roles de administrador](directory-assign-admin-roles.md).
- Para más información acerca de los permisos de usuario predeterminados, consulte la [comparación de los permisos predeterminados de los usuarios miembros e invitados](../fundamentals/users-default-permissions.md).
