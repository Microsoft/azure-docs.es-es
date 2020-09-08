---
title: Elevación de los privilegios de acceso para administrar todas las suscripciones y los grupos de administración de Azure
description: Describe cómo elevar los privilegios de acceso de un administrador global para administrar todas las suscripciones y los grupos de administración en Azure Active Directory mediante Azure Portal o la API REST.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 06/09/2020
ms.author: rolyon
ms.openlocfilehash: 343f6b7a78ca98615d512d31d7ac1c10d9de8f10
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799339"
---
# <a name="elevate-access-to-manage-all-azure-subscriptions-and-management-groups"></a>Elevación de los privilegios de acceso para administrar todas las suscripciones y los grupos de administración de Azure

En tanto que administrador global de Azure Active Directory (Azure AD), es posible que no tenga acceso a todas las suscripciones y los grupos de administración de su directorio. En este artículo se describen los medios para elevar sus derechos de acceso a todas las suscripciones y grupos de administración.

[!INCLUDE [gdpr-dsr-and-stp-note](../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="why-would-you-need-to-elevate-your-access"></a>¿Por qué necesita elevar los derechos de acceso?

Si es un administrador global, es posible que haya momentos en los que quiera realizar las siguientes acciones:

- Recuperar el acceso a una suscripción o grupo de administración de Azure cuando un usuario ha perdido el acceso
- Conceder acceso a otro usuario o usted mismo a una suscripción o grupo de administración de Azure
- Ver todas las suscripciones o grupos de administración de Azure de una organización
- Permitir que una aplicación de automatización (por ejemplo, una aplicación de facturación o auditoría) tenga acceso a todas las suscripciones o grupos de administración de Azure

## <a name="how-does-elevated-access-work"></a>¿Cómo funciona el acceso con privilegios elevados?

Azure AD y los recursos de Azure están protegidos de forma independiente entre sí. Es decir, las asignaciones de roles de Azure AD no otorgan acceso a recursos de Azure, y las asignaciones de roles de Azure no conceden acceso a Azure AD. Sin embargo, si es [administrador global](../active-directory/users-groups-roles/directory-assign-admin-roles.md#company-administrator-permissions) de Azure AD, puede asignarse a sí mismo los privilegios de acceso para todas las suscripciones a Azure y los grupos de administración de su directorio. Use esta funcionalidad si no tiene acceso a recursos de suscripción a Azure, como máquinas virtuales o cuentas de almacenamiento, y quiere usar su privilegio de administrador global para obtener acceso a esos recursos.

Al elevar los privilegios de acceso, se le asignará el rol [Administrador de acceso de usuario](built-in-roles.md#user-access-administrator) en Azure en el ámbito raíz (`/`). Esto le permite ver todos los recursos y asignar acceso en cualquier suscripción o grupo de administración en el directorio. Se pueden quitar las asignaciones de roles de administrador de acceso de usuario mediante Azure PowerShell, CLI de Azure o la API de REST.

Debe quitar este acceso con privilegios elevados una vez que haya hecho los cambios necesarios en el ámbito raíz.

![Elevación de los privilegios de acceso](./media/elevate-access-global-admin/elevate-access.png)

## <a name="azure-portal"></a>Azure portal

### <a name="elevate-access-for-a-global-administrator"></a>Elevación de los privilegios de acceso de un administrador global

Siga estos pasos para elevar los privilegios de acceso de un administrador global mediante Azure Portal.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com) o en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com) como administrador global.

    Si usa Azure AD Privileged Identity Management, [active la asignación de roles de administrador global](../active-directory/privileged-identity-management/pim-how-to-activate-role.md).

1. Abra **Azure Active Directory**.

1. En **Administrar**, seleccione **Propiedades**.

   ![Seleccionar propiedades para las propiedades de Azure Active Directory: captura de pantalla](./media/elevate-access-global-admin/azure-active-directory-properties.png)

1. En **Administración del acceso para los recursos de Azure**, establezca el botón de alternancia en **Sí**.

   ![Captura de pantalla de administración de acceso a recursos de Azure](./media/elevate-access-global-admin/aad-properties-global-admin-setting.png)

   Al establecer el botón de alternancia en **Sí**, se le asigna el rol de administrador de accesos de usuario en Azure RBAC en el ámbito raíz (/). Esto le concede permiso para asignar roles en todas las suscripciones de Azure y los grupos de administración asociados a este directorio de Azure AD. Este botón de alternancia solo está disponible para los usuarios que tienen asignado el rol de administrador global de Azure AD.

   Al establecer el botón de alternancia en **No**, se quita el rol de administrador de accesos de usuario en Azure RBAC de su cuenta de usuario. Ya no puede asignar roles en todas las suscripciones de Azure y los grupos de administración asociados a este directorio de Azure AD. Puede ver y administrar solo las suscripciones de Azure y los grupos de administración a los que se le ha concedido acceso.

    > [!NOTE]
    > Si usa [Privileged Identity Management](../active-directory/privileged-identity-management/pim-configure.md), la desactivación de la asignación de roles no cambia la opción **Administración de acceso para recursos de Azure** a **No**. Para mantener el acceso con privilegios mínimos, se recomienda establecer esta opción en **No** antes de desactivar la asignación de roles.
    
1. Haga clic en **Guardar** para guardar la configuración.

   Esta configuración no es una propiedad global y se aplica solo al usuario que tiene la sesión iniciada. No puede elevar los privilegios de acceso para todos los miembros del rol de administrador global.

1. Cierre la sesión e inicie sesión de nuevo para actualizar el acceso.

    Ahora debería tener acceso a todas las suscripciones y los grupos de administración de Azure de este directorio. Observará que se le ha asignado el rol de administrador de accesos de usuario en el ámbito raíz al ver el panel de control de acceso (IAM).

   ![Asignaciones de roles de la suscripción con el ámbito raíz: captura de pantalla](./media/elevate-access-global-admin/iam-root.png)

1. Haga los cambios que tenga que hacer con privilegios de acceso elevados.

    Para más información sobre la asignación de roles, consulte [Incorporación o eliminación de asignaciones de roles mediante Azure Portal](role-assignments-portal.md). Si usa Privileged Identity Management, consulte [Detección de recursos de Azure para administrar](../active-directory/privileged-identity-management/pim-resource-roles-discover-resources.md) o [Asignación de roles de recursos de Azure](../active-directory/privileged-identity-management/pim-resource-roles-assign-roles.md).

1. Siga los pasos de la sección siguiente para quitar el acceso con privilegios elevados.

### <a name="remove-elevated-access"></a>Eliminación de privilegios de acceso elevados

Para quitar la asignación del rol de administrador de accesos de usuario en el ámbito raíz (`/`), siga estos pasos.

1. Inicie sesión como el mismo usuario que se usó para elevar el acceso.

1. En la lista de navegación, haga clic en **Azure Active Directory** y, luego, haga clic en **Propiedades**.

1. Establezca el botón de alternancia **Administración del acceso para los recursos de Azure** de nuevo en **No**. Puesto que se trata de una configuración que se realiza a nivel de usuario, debe haber iniciado sesión con el mismo usuario que el utilizado para elevar los privilegios de acceso.

    Si intenta quitar la asignación de rol de administración de identidad y acceso en el panel de control de acceso (IAM), verá el siguiente mensaje. Para quitar la asignación de roles, debe establecer el botón de alternancia en **No** o usar Azure PowerShell, la CLI de Azure o la API REST.

    ![Quitar las asignaciones de roles en el ámbito raíz](./media/elevate-access-global-admin/iam-root-remove.png)

1. Cerrar sesión como un administrador global.

    Si usa Privileged Identity Management, desactive la asignación de roles de administrador global.

    > [!NOTE]
    > Si usa [Privileged Identity Management](../active-directory/privileged-identity-management/pim-configure.md), la desactivación de la asignación de roles no cambia la opción **Administración de acceso para recursos de Azure** a **No**. Para mantener el acceso con privilegios mínimos, se recomienda establecer esta opción en **No** antes de desactivar la asignación de roles.

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

### <a name="list-role-assignment-at-root-scope-"></a>Mostrar la asignación de roles en el ámbito raíz (/)

Para mostrar la asignación del rol de administrador de accesos de usuario de un usuario en el ámbito raíz (`/`), use el comando [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment).

```azurepowershell
Get-AzRoleAssignment | where {$_.RoleDefinitionName -eq "User Access Administrator" `
  -and $_.SignInName -eq "<username@example.com>" -and $_.Scope -eq "/"}
```

```Example
RoleAssignmentId   : /providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111
Scope              : /
DisplayName        : username
SignInName         : username@example.com
RoleDefinitionName : User Access Administrator
RoleDefinitionId   : 18d7d88d-d35e-4fb5-a5c3-7773c20a72d9
ObjectId           : 22222222-2222-2222-2222-222222222222
ObjectType         : User
CanDelegate        : False
```

### <a name="remove-elevated-access"></a>Eliminación de privilegios de acceso elevados

Para quitarse a usted o a otro usuario la asignación del rol de administrador de accesos de usuario en el ámbito raíz (`/`), siga estos pasos.

1. Inicie sesión como un usuario que puede quitar los privilegios de acceso elevado. Puede ser el mismo usuario que se usó para elevar los privilegios de acceso u otro administrador global con privilegios de acceso elevados en el ámbito raíz.

1. Use el comando [Remove-AzRoleAssignment](/powershell/module/az.resources/remove-azroleassignment) para quitar la asignación del rol de administrador de accesos de usuario.

    ```azurepowershell
    Remove-AzRoleAssignment -SignInName <username@example.com> `
      -RoleDefinitionName "User Access Administrator" -Scope "/"
    ```

## <a name="azure-cli"></a>Azure CLI

### <a name="elevate-access-for-a-global-administrator"></a>Elevación de los privilegios de acceso de un administrador global

Utilice los siguientes pasos básicos para elevar los privilegios de acceso de un administrador global mediante la CLI de Azure.

1. Use el comando [az rest](/cli/azure/reference-index?view=azure-cli-latest#az-rest) para llamar al punto de conexión `elevateAccess`, que le concede el rol de administrador de accesos de usuario en el ámbito raíz (`/`).

    ```azurecli
    az rest --method post --url "/providers/Microsoft.Authorization/elevateAccess?api-version=2016-07-01"
    ```

1. Haga los cambios que tenga que hacer con privilegios de acceso elevados.

    Para más información sobre la asignación de roles, consulte [Incorporación o eliminación de asignaciones de roles mediante la CLI de Azure](role-assignments-cli.md).

1. Siga los pasos de una sección posterior para quitar el acceso con privilegios elevados.

### <a name="list-role-assignment-at-root-scope-"></a>Mostrar la asignación de roles en el ámbito raíz (/)

Para mostrar la asignación del rol de administrador de accesos de usuario para un usuario en el ámbito raíz (`/`), use el comando [az role assignment list](/cli/azure/role/assignment#az-role-assignment-list).

```azurecli
az role assignment list --role "User Access Administrator" --scope "/"
```

```Example
[
  {
    "canDelegate": null,
    "id": "/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111",
    "name": "11111111-1111-1111-1111-111111111111",
    "principalId": "22222222-2222-2222-2222-222222222222",
    "principalName": "username@example.com",
    "principalType": "User",
    "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
    "roleDefinitionName": "User Access Administrator",
    "scope": "/",
    "type": "Microsoft.Authorization/roleAssignments"
  }
]

```

### <a name="remove-elevated-access"></a>Eliminación de privilegios de acceso elevados

Para quitarse a usted o a otro usuario la asignación del rol de administrador de accesos de usuario en el ámbito raíz (`/`), siga estos pasos.

1. Inicie sesión como un usuario que puede quitar los privilegios de acceso elevado. Puede ser el mismo usuario que se usó para elevar los privilegios de acceso u otro administrador global con privilegios de acceso elevados en el ámbito raíz.

1. Use el comando [az role assignment delete](/cli/azure/role/assignment#az-role-assignment-delete) para quitar la asignación del rol de administrador de accesos de usuario.

    ```azurecli
    az role assignment delete --assignee username@example.com --role "User Access Administrator" --scope "/"
    ```

## <a name="rest-api"></a>API DE REST

### <a name="elevate-access-for-a-global-administrator"></a>Elevación de los privilegios de acceso de un administrador global

Utilice los siguientes pasos básicos para elevar los privilegios de acceso de un administrador global mediante la API de REST.

1. Mediante REST, llame a `elevateAccess`. Entonces, se le concede el rol de administrador de accesos de usuario en el ámbito raíz (`/`).

   ```http
   POST https://management.azure.com/providers/Microsoft.Authorization/elevateAccess?api-version=2016-07-01
   ```

1. Haga los cambios que tenga que hacer con privilegios de acceso elevados.

    Para más información sobre la asignación de roles, consulte [Incorporación o eliminación de asignaciones de roles de Azure mediante la API REST](role-assignments-rest.md).

1. Siga los pasos de una sección posterior para quitar el acceso con privilegios elevados.

### <a name="list-role-assignments-at-root-scope-"></a>Enumeración de las asignaciones de roles en el ámbito raíz (/)

Puede enumerar todas las asignaciones de roles para un usuario en el ámbito raíz (`/`).

- Llamar a [GET roleAssignments](/rest/api/authorization/roleassignments/listforscope) donde `{objectIdOfUser}` es el id. de objeto del usuario cuyas asignaciones de roles quiere recuperar.

   ```http
   GET https://management.azure.com/providers/Microsoft.Authorization/roleAssignments?api-version=2015-07-01&$filter=principalId+eq+'{objectIdOfUser}'
   ```

### <a name="list-deny-assignments-at-root-scope-"></a>Enumeración de las asignaciones de denegación en el ámbito raíz (/)

Puede enumerar todas las asignaciones de denegación de un usuario en el ámbito raíz (`/`).

- Llame a GET denyAssignments, donde `{objectIdOfUser}` es el id. de objeto del usuario cuyas asignaciones de denegación desea recuperar.

   ```http
   GET https://management.azure.com/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview&$filter=gdprExportPrincipalId+eq+'{objectIdOfUser}'
   ```

### <a name="remove-elevated-access"></a>Eliminación de privilegios de acceso elevados

Cuando llama a `elevateAccess`, se crea una asignación de roles para usted, por lo que, para revocar esos privilegios, debe quitarse la asignación de rol de administrador de accesos de usuario en el ámbito raíz (`/`).

1. Llame al comando [GET-roleDefinitions](/rest/api/authorization/roledefinitions/get), donde `roleName` es el administrador de accesos de usuario, para determinar el id. del nombre del rol de administrador de accesos de usuario.

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter=roleName+eq+'User Access Administrator'
    ```

    ```json
    {
      "value": [
        {
          "properties": {
      "roleName": "User Access Administrator",
      "type": "BuiltInRole",
      "description": "Lets you manage user access to Azure resources.",
      "assignableScopes": [
        "/"
      ],
      "permissions": [
        {
          "actions": [
            "*/read",
            "Microsoft.Authorization/*",
            "Microsoft.Support/*"
          ],
          "notActions": []
        }
      ],
      "createdOn": "0001-01-01T08:00:00.0000000Z",
      "updatedOn": "2016-05-31T23:14:04.6964687Z",
      "createdBy": null,
      "updatedBy": null
          },
          "id": "/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
          "type": "Microsoft.Authorization/roleDefinitions",
          "name": "18d7d88d-d35e-4fb5-a5c3-7773c20a72d9"
        }
      ],
      "nextLink": null
    }
    ```

    Guarde el id. del parámetro `name`; en este caso: `18d7d88d-d35e-4fb5-a5c3-7773c20a72d9`.

1. También tiene que mostrar la asignación de roles del administrador de directorios en el ámbito del directorio. Muestre todas las asignaciones en el ámbito del directorio para `principalId` del administrador de directorios que hizo la llamada de elevación de privilegios de acceso. Esto mostrará todas las asignaciones en el directorio para objectid.

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/roleAssignments?api-version=2015-07-01&$filter=principalId+eq+'{objectid}'
    ```
        
    >[!NOTE] 
    >Un administrador de directorios no debe tener muchas asignaciones; si la consulta anterior devuelve demasiadas asignaciones, puede consultar también todas las asignaciones en el nivel de ámbito de directorio y, después, filtrar los resultados: `GET https://management.azure.com/providers/Microsoft.Authorization/roleAssignments?api-version=2015-07-01&$filter=atScope()`
            
1. Las llamadas anteriores devuelven una lista de asignaciones de roles. Busque la asignación de roles en la que el ámbito sea `"/"`, `roleDefinitionId` termine con el id. del nombre de rol que se encuentra en el paso 1, y `principalId` coincida con el valor de ObjectId del administrador de directorios. 
    
    Ejemplo de asignación de rol:
    
    ```json
    {
      "value": [
        {
          "properties": {
            "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
            "principalId": "{objectID}",
            "scope": "/",
            "createdOn": "2016-08-17T19:21:16.3422480Z",
            "updatedOn": "2016-08-17T19:21:16.3422480Z",
            "createdBy": "22222222-2222-2222-2222-222222222222",
            "updatedBy": "22222222-2222-2222-2222-222222222222"
          },
          "id": "/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111",
          "type": "Microsoft.Authorization/roleAssignments",
          "name": "11111111-1111-1111-1111-111111111111"
        }
      ],
      "nextLink": null
    }
    ```
    
    Vuelva a guardar el Id. del parámetro `name`; en este caso, 11111111-1111-1111-1111-111111111111.

1. Por último, utilice el id. de asignación de roles para quitar la asignación agregada por `elevateAccess`:

    ```http
    DELETE https://management.azure.com/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111?api-version=2015-07-01
    ```

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de los distintos roles](rbac-and-directory-admin-roles.md)
- [Incorporación o eliminación de asignaciones de roles de Azure mediante la API REST](role-assignments-rest.md)
