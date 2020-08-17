---
title: 'Tutorial: Creación de un rol personalizado de Azure con la CLI de Azure: Azure RBAC'
description: En este tutorial aprenderá a crear un rol personalizado de Azure mediante la CLI de Azure y el control de acceso basado en rol de Azure (Azure RBAC).
services: active-directory
documentationCenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 02/20/2019
ms.author: rolyon
ms.custom: devx-track-azurecli
ms.openlocfilehash: 319bca74c8e781e5dc5022e9fb901b2edca24a80
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87485650"
---
# <a name="tutorial-create-an-azure-custom-role-using-azure-cli"></a>Tutorial: Creación de un rol personalizado de Azure con la CLI de Azure

Si los [roles integrados de Azure](built-in-roles.md) no cumplen las necesidades específicas de su organización, puede crear los suyos propios. En este tutorial, creará un rol personalizado llamado Reader Support Tickets (Lector de incidencias de soporte) con la CLI de Azure. El rol personalizado permite al usuario ver todo lo relativo a la administración de una suscripción y también abrir incidencias de soporte técnico.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un rol personalizado
> * Lista de roles personalizados
> * Actualización de un rol personalizado
> * Eliminación de un rol personalizado

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para realizar este tutorial, necesita:

- Permisos para crear roles personalizados, como [Propietario](built-in-roles.md#owner) o [Administrador de acceso de usuarios](built-in-roles.md#user-access-administrator)
- [Azure Cloud Shell](../cloud-shell/overview.md) o [CLI de Azure](/cli/azure/install-azure-cli)

## <a name="sign-in-to-azure-cli"></a>Inicio de sesión en la CLI de Azure

Inicie sesión en la [CLI de Azure](/cli/azure/authenticate-azure-cli).

## <a name="create-a-custom-role"></a>Crear un rol personalizado

El modo más sencillo de crear un rol personalizado es tomar como partida una plantilla JSON, editarla y, a continuación, crear un nuevo rol.

1. Revise la lista de operaciones para el [proveedor de recursos Microsoft.Support](resource-provider-operations.md#microsoftsupport). Resulta útil conocer las operaciones que están disponibles para crear los permisos.

    | Operación | Descripción |
    | --- | --- |
    | Microsoft.Support/register/action | Registra para admitir el proveedor de recursos |
    | Microsoft.Support/supportTickets/read | Obtiene los detalles de las incidencias de soporte técnico (incluido el estado, gravedad, detalles de contacto y comunicaciones) u obtiene la lista de incidencias de soporte técnico de las diversas suscripciones. |
    | Microsoft.Support/supportTickets/write | Crea o actualiza una incidencia de soporte técnico. Puede crear una incidencia de soporte técnico para problemas relacionados con cuestiones técnicas, de facturación, cuotas o de administración de suscripciones. Puede actualizar la gravedad, los detalles de contacto y las comunicaciones de las incidencias de soporte técnico existentes. |
    
1. Cree un nuevo archivo llamado **ReaderSupportRole.json**.

1. Abra ReaderSupportRole.json en un editor y agregue el siguiente código JSON.

    Para información sobre las distintas propiedades, consulte [Roles personalizados de Azure](custom-roles.md).

    ```json
    {
      "Name": "",
      "IsCustom": true,
      "Description": "",
      "Actions": [],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/{subscriptionId1}"
      ]
    }
    ```
    
1. Agregue las siguientes operaciones a la propiedad `Actions`. Estas acciones permiten al usuario ver todo el contenido de la suscripción y crear incidencias de soporte técnico.

    ```
    "*/read",
    "Microsoft.Support/*"
    ```

1. Obtenga el identificador de la suscripción mediante el comando [az account list](/cli/azure/account#az-account-list).

    ```azurecli
    az account list --output table
    ```

1. En `AssignableScopes`, reemplace `{subscriptionId1}` por el identificador de la suscripción.

    Debe agregar identificadores de suscripción explícitos; en caso contrario, no podrá importar el rol en su suscripción.

1. Cambie el valor de las propiedades `Name` y `Description` por "Reader Support Tickets" (Lector de incidencias de soporte) y "Ver todo el contenido de la suscripción y también abrir incidencias de soporte técnico".

    El archivo JSON debe ser similar al siguiente:

    ```json
    {
      "Name": "Reader Support Tickets",
      "IsCustom": true,
      "Description": "View everything in the subscription and also open support tickets.",
      "Actions": [
        "*/read",
        "Microsoft.Support/*"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ]
    }
    ```
    
1. Para crear el nuevo rol personalizado, use el comando [az role definition create](/cli/azure/role/definition#az-role-definition-create) y especifique el archivo de definición de roles JSON.

    ```azurecli
    az role definition create --role-definition "~/CustomRoles/ReaderSupportRole.json"
    ```

    ```Output
    {
      "additionalProperties": {},
      "assignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ],
      "description": "View everything in the subscription and also open support tickets.",
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
      "name": "22222222-2222-2222-2222-222222222222",
      "permissions": [
        {
          "actions": [
            "*/read",
            "Microsoft.Support/*"
          ],
          "additionalProperties": {},
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": "Reader Support Tickets",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```

    El nuevo rol personalizado ya está disponible y se puede asignar a los usuarios, los grupos y las entidades de servicio igual que los roles integrados.

## <a name="list-custom-roles"></a>Lista de roles personalizados

- Para enumerar todos los roles personalizados, use el comando [az role definition list](/cli/azure/role/definition#az-role-definition-list) con el parámetro `--custom-role-only`.

    ```azurecli
    az role definition list --custom-role-only true
    ```
    
    ```Output
    [
      {
        "additionalProperties": {},
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000"
        ],
        "description": "View everything in the subscription and also open support tickets.",
        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
        "name": "22222222-2222-2222-2222-222222222222",
        "permissions": [
          {
            "actions": [
              "*/read",
              "Microsoft.Support/*",
              "Microsoft.Resources/deployments/*",
              "Microsoft.Insights/diagnosticSettings/*/read"
            ],
            "additionalProperties": {},
            "dataActions": [],
            "notActions": [],
            "notDataActions": []
          }
        ],
        "roleName": "Reader Support Tickets",
        "roleType": "CustomRole",
        "type": "Microsoft.Authorization/roleDefinitions"
      }
    ]
    ```
    
    También puede ver el rol personalizado en Azure Portal.

    ![Captura de pantalla de un rol importado en Azure Portal](./media/tutorial-custom-role-cli/custom-role-reader-support-tickets.png)

## <a name="update-a-custom-role"></a>Actualización de un rol personalizado

Para actualizar el rol personalizado, actualice el archivo JSON y, a continuación, actualice el rol personalizado.

1. Abra el archivo ReaderSupportRole.json.

1. En `Actions`, agregue la operación para crear y administrar implementaciones de grupos de recursos `"Microsoft.Resources/deployments/*"`. Asegúrese de incluir una coma después de la operación anterior.

    El archivo JSON actualizado debe ser similar al siguiente:

    ```json
    {
      "Name": "Reader Support Tickets",
      "IsCustom": true,
      "Description": "View everything in the subscription and also open support tickets.",
      "Actions": [
        "*/read",
        "Microsoft.Support/*",
        "Microsoft.Resources/deployments/*"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ]
    }
    ```
        
1. Para actualizar el rol personalizado, use el comando [az role definition update](/cli/azure/role/definition#az-role-definition-update) y especifique el archivo JSON actualizado.

    ```azurecli
    az role definition update --role-definition "~/CustomRoles/ReaderSupportRole.json"
    ```

    ```Output
    {
      "additionalProperties": {},
      "assignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ],
      "description": "View everything in the subscription and also open support tickets.",
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
      "name": "22222222-2222-2222-2222-222222222222",
      "permissions": [
        {
          "actions": [
            "*/read",
            "Microsoft.Support/*",
            "Microsoft.Resources/deployments/*"
          ],
          "additionalProperties": {},
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": "Reader Support Tickets",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```
    
## <a name="delete-a-custom-role"></a>Eliminación de un rol personalizado

- Use el comando [az role definition delete](/cli/azure/role/definition#az-role-definition-delete) y especifique el nombre del rol o el identificador del rol para eliminar el rol personalizado.

    ```azurecli
    az role definition delete --name "Reader Support Tickets"
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación o actualización de roles personalizados de Azure con la CLI de Azure](custom-roles-cli.md)
