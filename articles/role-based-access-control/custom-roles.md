---
title: 'Roles personalizados de Azure: RBAC de Azure'
description: Aprenda a crear roles personalizados de Azure con el control de acceso basado en roles de Azure (RBAC de Azure) para la administración de acceso específico de recursos de Azure.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: e4206ea9-52c3-47ee-af29-f6eef7566fa5
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/13/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: fd737a22a37d6edc47c2769a470af00537d720eb
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87124160"
---
# <a name="azure-custom-roles"></a>Roles personalizados de Azure

> [!IMPORTANT]
> La adición de un grupo de administración a `AssignableScopes` está actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si los [roles integrados de Azure](built-in-roles.md) no cumplen las necesidades específicas de su organización, puede crear sus propios roles personalizados. Al igual que en el caso de los roles integrados, se pueden asignar roles personalizados a usuarios, grupos y entidades de servicio en los ámbitos del grupo de administración, de la suscripción y del grupo de recursos.

Los roles personalizados se pueden compartir entre suscripciones que confían en el mismo directorio de Azure AD. Hay un límite de **5000** roles personalizados por directorio. (En Azure Alemania y Azure China 21Vianet, el límite es 2000 roles personalizados). Se pueden crear roles personalizados con Azure Portal, Azure PowerShell, la CLI de Azure o la API REST.

## <a name="custom-role-example"></a>Ejemplo de rol personalizado

A continuación se muestra el aspecto de un rol personalizado en formato JSON a través de Azure PowerShell. Este rol personalizado puede usarse para supervisar y reiniciar máquinas virtuales.

```json
{
  "Name": "Virtual Machine Operator",
  "Id": "88888888-8888-8888-8888-888888888888",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.ResourceHealth/availabilityStatuses/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Insights/diagnosticSettings/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId1}",
    "/subscriptions/{subscriptionId2}",
    "/providers/Microsoft.Management/managementGroups/{groupId1}"
  ]
}
```

A continuación se muestra el mismo rol personalizado mediante la CLI de Azure.

```json
[
  {
    "assignableScopes": [
      "/subscriptions/{subscriptionId1}",
      "/subscriptions/{subscriptionId2}",
      "/providers/Microsoft.Management/managementGroups/{groupId1}"
    ],
    "description": "Can monitor and restart virtual machines.",
    "id": "/subscriptions/{subscriptionId1}/providers/Microsoft.Authorization/roleDefinitions/88888888-8888-8888-8888-888888888888",
    "name": "88888888-8888-8888-8888-888888888888",
    "permissions": [
      {
        "actions": [
          "Microsoft.Storage/*/read",
          "Microsoft.Network/*/read",
          "Microsoft.Compute/*/read",
          "Microsoft.Compute/virtualMachines/start/action",
          "Microsoft.Compute/virtualMachines/restart/action",
          "Microsoft.Authorization/*/read",
          "Microsoft.ResourceHealth/availabilityStatuses/read",
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Insights/alertRules/*",
          "Microsoft.Insights/diagnosticSettings/*",
          "Microsoft.Support/*"
        ],
        "dataActions": [],
        "notActions": [],
        "notDataActions": []
      }
    ],
    "roleName": "Virtual Machine Operator",
    "roleType": "CustomRole",
    "type": "Microsoft.Authorization/roleDefinitions"
  }
]
```

Después de crear un rol personalizado, aparece en Azure Portal con un icono de recurso naranja.

![Icono de rol personalizado](./media/custom-roles/roles-custom-role-icon.png)

## <a name="custom-role-properties"></a>Propiedades del rol personalizado

En la tabla siguiente se describe el significado de las propiedades de roles personalizadas.

| Propiedad | Obligatorio | Tipo | Descripción |
| --- | --- | --- | --- |
| `Name`</br>`roleName` | Sí | String | Nombre para mostrar del rol personalizado. Aunque una definición de roles es un recurso de nivel de grupo de administración o de suscripción, una definición de roles se puede usar en varias suscripciones que compartan el mismo directorio de Azure AD. Este nombre para mostrar debe ser único en el ámbito del directorio de Azure AD. Puede incluir letras, números, espacios y caracteres especiales. El número máximo de caracteres es 128. |
| `Id`</br>`name` | Sí | String | El identificador exclusivo del rol personalizado. Para Azure PowerShell y la CLI de Azure, este identificador se genera automáticamente cuando se crea un rol. |
| `IsCustom`</br>`roleType` | Sí | String | Indica si es un rol personalizado. Se establece en `true` o `CustomRole` para los roles personalizados. Se establece en `false` o `BuiltInRole` para los roles integrados. |
| `Description`</br>`description` | Sí | String | Descripción del rol personalizado. Puede incluir letras, números, espacios y caracteres especiales. El número máximo de caracteres es 1024. |
| `Actions`</br>`actions` | Sí | String[] | Matriz de cadenas que especifica las operaciones de administración que el rol permite realizar. Para obtener más información, consulte [Actions](role-definitions.md#actions). |
| `NotActions`</br>`notActions` | No | String[] | Matriz de cadenas que especifica las operaciones de administración que se excluyen de las `Actions` permitidas. Para obtener más información, consulte [notActions](role-definitions.md#notactions). |
| `DataActions`</br>`dataActions` | No | String[] | Matriz de cadenas que especifica las operaciones de datos que el rol permite realizar en los datos dentro de ese objeto. Si crea un rol personalizado con `DataActions`, ese rol no se puede asignar en el ámbito del grupo de administración. Para más información, vea [DataActions](role-definitions.md#dataactions). |
| `NotDataActions`</br>`notDataActions` | No | String[] | Matriz de cadenas que especifica las operaciones de datos que se excluyen de las `DataActions` permitidas. Para más información, vea [NotDataActions](role-definitions.md#notdataactions). |
| `AssignableScopes`</br>`assignableScopes` | Sí | String[] | Matriz de cadenas que especifica los ámbitos en los que el rol personalizado está disponible para la asignación. Solo se puede definir un grupo de administración en `AssignableScopes` de un rol personalizado. La adición de un grupo de administración a `AssignableScopes` está actualmente en versión preliminar. Para obtener más información, consulte [assignableScopes](role-definitions.md#assignablescopes). |

## <a name="wildcard-permissions"></a>Permisos con caracteres comodín

`Actions`, `NotActions`, `DataActions`y `NotDataActions` admiten caracteres comodín (`*`) para definir los permisos. Los caracteres comodín (`*`) amplían los permisos a todo aquello que coincida con la cadena de acción que se especifique. Por ejemplo, supongamos que desea agregar todos los permisos relacionados con Azure Cost Management y las exportaciones. Puede agregar todas estas cadenas de acción:

```
Microsoft.CostManagement/exports/action
Microsoft.CostManagement/exports/read
Microsoft.CostManagement/exports/write
Microsoft.CostManagement/exports/delete
Microsoft.CostManagement/exports/run/action
```

En lugar de agregar todas estas cadenas, podría agregar únicamente una cadena comodín. Por ejemplo, la siguiente cadena comodín es equivalente a las cinco cadenas anteriores. De esta forma también se incluirían los futuros permisos de exportación que se pudieran agregar.

```
Microsoft.CostManagement/exports/*
```

Una sola cadena puede tener varios caracteres comodín. Por ejemplo, la siguiente cadena representa todos los permisos de consulta de Cost Management.

```
Microsoft.CostManagement/*/query/*
```

## <a name="steps-to-create-a-custom-role"></a>Pasos para crear un rol personalizado

Para crear un rol personalizado, estos son los pasos básicos que debe seguir.

1. Decidir cómo se quiere crear el rol personalizado.

    Puede crear roles personalizados con Azure Portal, Azure PowerShell, la CLI de Azure o la API de REST.

1. Determinar los permisos que necesita.

    Cuando crea un rol personalizado, debe conocer las operaciones que están disponibles para definir los permisos. Para ver la lista de operaciones, vea [Operaciones del proveedor de recursos de Azure Resource Manager](resource-provider-operations.md). Agregará las operaciones a las propiedades `Actions` o `NotActions` de la [definición de roles](role-definitions.md). Si tiene operaciones de datos, las agregará a las propiedades `DataActions` o `NotDataActions`.

1. Crear el rol personalizado.

    Por lo general, empiece con un rol integrado existente y, a continuación, modifíquelo según sus necesidades. La manera más fácil es usar Azure Portal. Para conocer los pasos para crear un rol personalizado mediante Azure Portal, consulte [Creación o actualización de roles personalizados de Azure mediante Azure Portal](custom-roles-portal.md).

1. Probar el rol personalizado.

    Una vez que tenga el rol personalizado, tiene que probarlo para comprobar que funciona según lo esperado. Si tiene que realizar ajustes más adelante, puede actualizar el rol personalizado.

## <a name="who-can-create-delete-update-or-view-a-custom-role"></a>Quién puede crear, eliminar, actualizar o ver un rol personalizado

Al igual que los roles integrados, la propiedad `AssignableScopes` especifica los ámbitos en los que el rol está disponible para la asignación. La propiedad `AssignableScopes` de un rol personalizado también controla quién puede crear, eliminar, actualizar o ver dicho rol.

| Tarea | Operación | Descripción |
| --- | --- | --- |
| Creación o eliminación de un rol personalizado | `Microsoft.Authorization/ roleDefinitions/write` | Los usuarios que tienen acceso a esta operación en todos los ámbitos `AssignableScopes` del rol personalizado pueden crear (o eliminar) roles personalizados para usar en esos ámbitos. Por ejemplo, los [propietarios](built-in-roles.md#owner) y [administradores del acceso de los usuarios](built-in-roles.md#user-access-administrator) de los grupos de administración, las suscripciones y los grupos de recursos. |
| Actualización de un rol personalizado | `Microsoft.Authorization/ roleDefinitions/write` | Los usuarios que tienen acceso a esta operación en todos los ámbitos `AssignableScopes` del rol personalizado pueden actualizar roles personalizados en esos ámbitos. Por ejemplo, los [propietarios](built-in-roles.md#owner) y [administradores del acceso de los usuarios](built-in-roles.md#user-access-administrator) de los grupos de administración, las suscripciones y los grupos de recursos. |
| Visualización de un rol personalizado | `Microsoft.Authorization/ roleDefinitions/read` | Los usuarios que tienen acceso a esta operación en un ámbito pueden ver los roles personalizados que están disponibles para su asignación en ese ámbito. Todos los roles integrados permiten que los roles personalizados estén disponibles para la asignación. |

## <a name="custom-role-limits"></a>Límites de los roles personalizados

En la siguiente lista se describen los límites de los roles personalizados.

- Cada directorio puede tener hasta **5000** roles personalizados.
- Azure Alemania y Azure China 21Vianet pueden tener hasta 2000 roles personalizados por cada directorio.
- `AssignableScopes` no se puede establecer en el ámbito raíz (`"/"`).
- Solo se puede definir un grupo de administración en `AssignableScopes` de un rol personalizado. La adición de un grupo de administración a `AssignableScopes` está actualmente en versión preliminar.
- No se pueden asignar roles personalizados con `DataActions` en el ámbito del grupo de administración.
- Azure Resource Manager no valida la existencia del grupo de administración en el ámbito asignable de la definición de roles.

Para más información sobre los roles personalizados y los grupos de administración, vea [Organización de los recursos con grupos de administración de Azure](../governance/management-groups/overview.md#azure-custom-role-definition-and-assignment).

## <a name="input-and-output-formats"></a>Formatos de entrada y salida

Para crear un rol personalizado mediante la línea de comandos, normalmente se usa JSON para especificar las propiedades deseadas para el rol personalizado. En función de las herramientas que use, los formatos de entrada y salida tendrán un aspecto ligeramente diferente. En esta sección se enumeran los formatos de entrada y salida en función de la herramienta.

### <a name="azure-powershell"></a>Azure PowerShell

Para crear un rol personalizado mediante Azure PowerShell, debe proporcionar la siguiente entrada.

```json
{
  "Name": "",
  "Description": "",
  "Actions": [],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": []
}
```

Para actualizar un rol personalizado mediante Azure PowerShell, debe proporcionar la siguiente entrada. Observe que se ha agregado la propiedad `Id`. 

```json
{
  "Name": "",
  "Id": "",
  "Description": "",
  "Actions": [],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": []
}
```

A continuación se muestra un ejemplo de la salida al enumerar un rol personalizado mediante Azure PowerShell y el comando [ConvertTo-JSON](/powershell/module/microsoft.powershell.utility/convertto-json). 

```json
{
  "Name": "",
  "Id": "",
  "IsCustom": true,
  "Description": "",
  "Actions": [],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": []
}
```

### <a name="azure-cli"></a>Azure CLI

Para crear o actualizar un rol personalizado mediante la CLI de Azure, debe proporcionar la siguiente entrada. Este es el mismo formato que el usado para crear un rol personalizado mediante Azure PowerShell.

```json
{
  "Name": "",
  "Description": "",
  "Actions": [],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": []
}
```

A continuación se muestra un ejemplo de la salida al enumerar un rol personalizado mediante la CLI de Azure.

```json
[
  {
    "assignableScopes": [],
    "description": "",
    "id": "",
    "name": "",
    "permissions": [
      {
        "actions": [],
        "dataActions": [],
        "notActions": [],
        "notDataActions": []
      }
    ],
    "roleName": "",
    "roleType": "CustomRole",
    "type": "Microsoft.Authorization/roleDefinitions"
  }
]
```

### <a name="rest-api"></a>API DE REST

Para crear o actualizar un rol personalizado mediante la API de REST, debe proporcionar la siguiente entrada. Este es el mismo formato que se genera al crear un rol personalizado mediante Azure Portal.

```json
{
  "properties": {
    "roleName": "",
    "description": "",
    "assignableScopes": [],
    "permissions": [
      {
        "actions": [],
        "notActions": [],
        "dataActions": [],
        "notDataActions": []
      }
    ]
  }
}
```

A continuación se muestra un ejemplo de la salida al enumerar un rol personalizado mediante la API de REST.

```json
{
    "properties": {
        "roleName": "",
        "type": "CustomRole",
        "description": "",
        "assignableScopes": [],
        "permissions": [
            {
                "actions": [],
                "notActions": [],
                "dataActions": [],
                "notDataActions": []
            }
        ],
        "createdOn": "",
        "updatedOn": "",
        "createdBy": "",
        "updatedBy": ""
    },
    "id": "",
    "type": "Microsoft.Authorization/roleDefinitions",
    "name": ""
}
```

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Creación de un rol personalizado de Azure mediante Azure PowerShell](tutorial-custom-role-powershell.md)
- [Tutorial: Creación de un rol personalizado para los recursos de Azure con la CLI de Azure](tutorial-custom-role-cli.md)
- [Descripción de las definiciones de roles de Azure](role-definitions.md)
- [Solución de problemas de Azure RBAC](troubleshooting.md)
