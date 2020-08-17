---
title: Restricción del acceso de los usuarios solo a las operaciones de datos con Azure Cosmos DB
description: Obtenga información sobre cómo restringir el acceso de los usuarios solo a las operaciones de datos con Azure Cosmos DB
author: voellm
ms.service: cosmos-db
ms.topic: how-to
ms.date: 12/9/2019
ms.author: tvoellm
ms.openlocfilehash: 16452337eeda86a9b019897954179bfe6db6e1b2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87031999"
---
# <a name="restrict-user-access-to-data-operations-only"></a>Restricción del acceso de los usuarios solo a las operaciones de datos

En Azure Cosmos DB, hay dos maneras de autenticar las interacciones con el servicio de la base de datos:
- puede usar su identidad de Azure Active Directory al interactuar con Azure Portal;
- puede usar las [claves](secure-access-to-data.md#master-keys) o [tokens de recursos](secure-access-to-data.md#resource-tokens) de Azure Cosmos DB al emitir llamadas desde las API y los SDK.

Cada método de autenticación proporciona acceso a diferentes conjuntos de operaciones, con algunas superposiciones:

:::image type="content" source="./media/how-to-restrict-user-data/operations.png" alt-text="División de operaciones por tipo de autenticación" border="false":::

En algunos escenarios, puede que quiera restringir a algunos usuarios de la organización para que solo realicen operaciones de datos (es decir, solicitudes y consultas CRUD). Este suele ser el caso de los desarrolladores que no necesitan crear o eliminar recursos, ni cambiar el rendimiento aprovisionado de los contenedores en los que están trabajando.

Puede restringir el acceso al aplicar los pasos siguientes:
1. Cree un rol de Azure Active Directory personalizado para los usuarios a los que quiere restringir el acceso. El rol de Active Directory personalizado debe tener un nivel de acceso específico a las operaciones con ayuda de las [acciones pormenorizadas](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb) de Azure Cosmos DB.
1. No permita la ejecución de operaciones que no son de datos con las claves. Para ello, restrinja estas operaciones a solo llamadas de Azure Resource Manager.

En las secciones siguientes de este artículo se muestra cómo realizar estos pasos.

> [!NOTE]
> Para ejecutar los comandos de las secciones siguientes, debe instalar el módulo 3.0.0 o posterior de Azure PowerShell, así como el [rol de propietario de Azure](../role-based-access-control/built-in-roles.md#owner) en la suscripción que está intentando modificar.

En los scripts de PowerShell de las secciones siguientes, sustituya los siguientes marcadores de posición por los valores específicos de su entorno:
- `$MySubscriptionId`: el Id. de la suscripción que contiene la cuenta de Azure Cosmos en la que quiere limitar los permisos. Por ejemplo: `e5c8766a-eeb0-40e8-af56-0eb142ebf78e`.
- `$MyResourceGroupName`: el grupo de recursos que contiene la cuenta de Azure Cosmos. Por ejemplo: `myresourcegroup`.
- `$MyAzureCosmosDBAccountName`: el nombre de la cuenta de Azure Cosmos. Por ejemplo: `mycosmosdbsaccount`.
- `$MyUserName`: el inicio de sesión (username@domain) del usuario para que el que quiere limitar el acceso. Por ejemplo: `cosmosdbuser@contoso.com`.

## <a name="select-your-azure-subscription"></a>Selección de su suscripción a Azure

Los comandos de Azure PowerShell requieren que inicie sesión y seleccione la suscripción para ejecutar los comandos:

```azurepowershell
Login-AzAccount
Select-AzSubscription $MySubscriptionId
```

## <a name="create-the-custom-azure-active-directory-role"></a>Creación del rol de Azure Active Directory personalizado

El siguiente script crea una asignación de roles de Azure Active Directory con el acceso de "solo clave" para las cuentas de Azure Cosmos. El rol se basa en los [roles personalizados de Azure](../role-based-access-control/custom-roles.md) y las [acciones pormenorizadas para Azure Cosmos DB](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb). Estos roles y acciones forman parte del espacio de nombres `Microsoft.DocumentDB` de Azure Active Directory.

1. En primer lugar, cree un documento JSON denominado `AzureCosmosKeyOnlyAccess.json` con el siguiente contenido:

    ```
    {
        "Name": "Azure Cosmos DB Key Only Access Custom Role",
        "Id": "00000000-0000-0000-0000-0000000000",
        "IsCustom": true,
        "Description": "This role restricts the user to read the account keys only.",
        "Actions":
        [
            "Microsoft.DocumentDB/databaseAccounts/listKeys/action"
        ],
        "NotActions": [],
        "DataActions": [],
        "NotDataActions": [],
        "AssignableScopes":
        [
            "/subscriptions/$MySubscriptionId"
        ]
    }
    ```

1. Ejecute los siguientes comandos para crear la asignación de roles y asignarla al usuario:

    ```azurepowershell
    New-AzRoleDefinition -InputFile "AzureCosmosKeyOnlyAccess.json"
    New-AzRoleAssignment -SignInName $MyUserName -RoleDefinitionName "Azure Cosmos DB Key Only Access Custom Role" -ResourceGroupName $MyResourceGroupName -ResourceName $MyAzureCosmosDBAccountName -ResourceType "Microsoft.DocumentDb/databaseAccounts"
    ```

## <a name="disallow-the-execution-of-non-data-operations"></a>No permitir la ejecución de operaciones que no son de datos

Los siguientes comandos quitan la capacidad de usar claves para:
- crear, modificar o eliminar recursos;
- actualizar la configuración del contenedor (incluidas las directivas de indexación, el rendimiento, etc.).

```azurepowershell
$cdba = Get-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName $MyResourceGroupName -ResourceName $MyAzureCosmosDBAccountName
$cdba.Properties.disableKeyBasedMetadataWriteAccess="True"
$cdba | Set-AzResource -Force
```

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [control de acceso basado en rol de Cosmos DB](role-based-access-control.md).
- Obtenga información general sobre el [acceso seguro a los datos en Cosmos DB](secure-access-to-data.md).
