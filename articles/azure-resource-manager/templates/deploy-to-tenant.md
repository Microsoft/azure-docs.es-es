---
title: Implementación de recursos en el inquilino
description: Se describe cómo implementar recursos en el ámbito de un inquilino en una plantilla de Azure Resource Manager.
ms.topic: conceptual
ms.date: 08/06/2020
ms.openlocfilehash: 2f5249eb54a62e4df082a18b22625bb93a0f09f8
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88002763"
---
# <a name="create-resources-at-the-tenant-level"></a>Creación de recursos en el nivel de inquilino

A medida que crece la organización, es posible que necesite definir y asignar [directivas](../../governance/policy/overview.md) o el [control de acceso basado en rol (RBAC de Azure)](../../role-based-access-control/overview.md) en el inquilino de Azure AD. Con las plantillas de nivel de inquilino, puede aplicar directivas y asignar roles a nivel global de forma declarativa.

## <a name="supported-resources"></a>Recursos compatibles

No todos los tipos de recursos se pueden implementar a nivel de inquilino. En esta sección se enumeran los tipos de recursos que se admiten.

Para las directivas de Azure, use:

* [policyAssignments](/azure/templates/microsoft.authorization/policyassignments)
* [policyDefinitions](/azure/templates/microsoft.authorization/policydefinitions)
* [policySetDefinitions](/azure/templates/microsoft.authorization/policysetdefinitions)

Para el control de acceso basado en rol, use:

* [roleAssignments](/azure/templates/microsoft.authorization/roleassignments)

Para plantillas anidadas que se implementan en grupos de administración, suscripciones o grupos de recursos, use:

* [deployments](/azure/templates/microsoft.resources/deployments)

Para crear grupos de administración, use:

* [managementGroups](/azure/templates/microsoft.management/managementgroups)

Para la administración de costos, use:

* [billingProfiles](/azure/templates/microsoft.billing/billingaccounts/billingprofiles)
* [instructions](/azure/templates/microsoft.billing/billingaccounts/billingprofiles/instructions)
* [invoiceSections](/azure/templates/microsoft.billing/billingaccounts/billingprofiles/invoicesections)

### <a name="schema"></a>Schema

El esquema que se usa para las implementaciones de nivel de inquilino es diferente del esquema de las implementaciones de grupo de recursos.

Para las plantillas, use:

```json
https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#
```

El esquema de un archivo de parámetros es el mismo para todos los ámbitos de implementación. Para los archivos de parámetros, use:

```json
https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#
```

## <a name="required-access"></a>Acceso necesario

La entidad de seguridad que implementa la plantilla debe tener permisos para crear recursos en el ámbito del inquilino. La entidad de seguridad debe tener permiso para ejecutar las acciones de implementación (`Microsoft.Resources/deployments/*`) y para crear los recursos definidos en la plantilla. Por ejemplo, para crear un grupo de administración, la entidad de seguridad debe tener permiso de colaborador en el ámbito del inquilino. Para crear asignaciones de roles, la entidad de seguridad debe tener permiso de propietario.

El administrador global de Azure Active Directory no tiene permiso para asignar roles de forma automática. Para habilitar las implementaciones de plantilla en el ámbito del inquilino, el administrador global debe realizar los siguientes pasos:

1. Eleve el acceso a la cuenta para que el administrador global pueda asignar roles. Para más información, consulte [Elevación de los privilegios de acceso para administrar todas las suscripciones y los grupos de administración de Azure](../../role-based-access-control/elevate-access-global-admin.md).

1. Asigne el rol de propietario o colaborador a la entidad de seguridad que necesita implementar las plantillas.

   ```azurepowershell-interactive
   New-AzRoleAssignment -SignInName "[userId]" -Scope "/" -RoleDefinitionName "Owner"
   ```

   ```azurecli-interactive
   az role assignment create --assignee "[userId]" --scope "/" --role "Owner"
   ```

La entidad de seguridad tiene ahora los permisos necesarios para implementar la plantilla.

## <a name="deployment-commands"></a>Comandos de implementación

Los comandos para las implementaciones de inquilino son diferentes de los comandos para las implementaciones de grupos de recursos.

Para la CLI de Azure, use [az deployment tenant create](/cli/azure/deployment/tenant?view=azure-cli-latest#az-deployment-tenant-create):

```azurecli-interactive
az deployment tenant create \
  --name demoTenantDeployment \
  --location WestUS \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/tenant-deployments/new-mg/azuredeploy.json"
```

En el caso de Azure PowerShell, use [New-AzTenantDeployment](/powershell/module/az.resources/new-aztenantdeployment).

```azurepowershell-interactive
New-AzTenantDeployment `
  -Name demoTenantDeployment `
  -Location "West US" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/tenant-deployments/new-mg/azuredeploy.json"
```

Para la API REST, use [Implementaciones: Crear o actualizar en el ámbito del inquilino](/rest/api/resources/deployments/createorupdateattenantscope).

## <a name="deployment-location-and-name"></a>Ubicación y nombre de la implementación

En el caso de las implementaciones de nivel de inquilino, debe proporcionar una ubicación para la implementación. La ubicación de la implementación es independiente de la ubicación de los recursos que se implementan. La ubicación de implementación especifica dónde se almacenarán los datos de la implementación.

Puede proporcionar un nombre para la implementación o usar el nombre de implementación predeterminado. El nombre predeterminado es el nombre del archivo de plantilla. Por ejemplo, al implementar una plantilla llamada **azuredeploy.json**, se crea un nombre de predeterminado **azuredeploy**.

Para cada nombre de implementación, la ubicación es inmutable. No se puede crear una implementación en una ubicación si ya existe una implementación con el mismo nombre en otra ubicación. Si recibe el código de error `InvalidDeploymentLocation`, use un nombre diferente o utilice la ubicación de la implementación anterior que tenía ese mismo nombre.

## <a name="deployment-scopes"></a>Ámbitos de implementación

Durante la implementación en un inquilino, puede tener como destino el inquilino o grupos de administración, suscripciones y grupos de recursos del inquilino. El usuario que implementa la plantilla debe tener acceso al ámbito especificado.

Los recursos definidos en la sección de recursos de la plantilla se aplican al inquilino.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        tenant-level-resources
    ],
    "outputs": {}
}
```

Para establecer como destino un grupo de administración dentro del inquilino, agregue una implementación anidada y especifique la propiedad `scope`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "mgName": {
            "type": "string"
        }
    },
    "variables": {
        "mgId": "[concat('Microsoft.Management/managementGroups/', parameters('mgName'))]"
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2020-06-01",
            "name": "nestedMG",
            "scope": "[variables('mgId')]",
            "location": "eastus",
            "properties": {
                "mode": "Incremental",
                "template": {
                    nested-template
                }
            }
        }
    ],
    "outputs": {}
}
```

## <a name="use-template-functions"></a>Usar funciones de plantillas

En las implementaciones de nivel de inquilino, hay algunas consideraciones importantes que deben tenerse en cuenta al usar las funciones de plantilla:

* La función [resourceGroup()](template-functions-resource.md#resourcegroup)**no** se admite.
* La función [subscription()](template-functions-resource.md#subscription)**no** se admite.
* Se admiten las funciones [reference()](template-functions-resource.md#reference) y [list()](template-functions-resource.md#list).
* Use la función [tenantResourceId()](template-functions-resource.md#tenantresourceid) para obtener el identificador de recurso de los recursos que se implementan en el nivel de inquilino.

  Por ejemplo, para obtener el identificador de recurso de una definición de directiva, utilice:

  ```json
  tenantResourceId('Microsoft.Authorization/policyDefinitions/', parameters('policyDefinition'))
  ```

  El identificador de recurso devuelto tiene el formato siguiente:

  ```json
  /providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
  ```

## <a name="create-management-group"></a>Creación de un grupo de administración

La [siguiente plantilla](https://github.com/Azure/azure-quickstart-templates/tree/master/tenant-deployments/new-mg) crea un grupo de administración.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "mgName": {
      "type": "string",
      "defaultValue": "[concat('mg-', uniqueString(newGuid()))]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Management/managementGroups",
      "apiVersion": "2019-11-01",
      "name": "[parameters('mgName')]",
      "properties": {
      }
    }
  ]
}
```

## <a name="assign-role"></a>Asignación de un rol

La [siguiente plantilla](https://github.com/Azure/azure-quickstart-templates/tree/master/tenant-deployments/tenant-role-assignment) asigna un rol en el ámbito del inquilino.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "type": "string",
      "metadata": {
        "description": "principalId if the user that will be given contributor access to the resourceGroup"
      }
    },
    "roleDefinitionId": {
      "type": "string",
      "defaultValue": "8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
      "metadata": {
        "description": "roleDefinition for the assignment - default is owner"
      }
    }
  },
  "variables": {
    // This creates an idempotent guid for the role assignment
    "roleAssignmentName": "[guid('/', parameters('principalId'), parameters('roleDefinitionId'))]"
  },
  "resources": [
    {
      "name": "[variables('roleAssignmentName')]",
      "type": "Microsoft.Authorization/roleAssignments",
      "apiVersion": "2019-04-01-preview",
      "properties": {
        "roleDefinitionId": "[tenantResourceId('Microsoft.Authorization/roleDefinitions', parameters('roleDefinitionId'))]",
        "principalId": "[parameters('principalId')]",
        "scope": "/"
      }
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la asignación de roles, consulte [Incorporación de asignaciones de roles mediante plantillas de Azure Resource Manager](../../role-based-access-control/role-assignments-template.md).
* También puede implementar plantillas en el [nivel de suscripción](deploy-to-subscription.md) o el [nivel de grupo de administración](deploy-to-management-group.md).
