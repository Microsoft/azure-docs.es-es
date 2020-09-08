---
title: Creación de directivas mediante programación
description: Este artículo le guiará a través de la creación y administración de directivas para Azure Policy mediante programación con la CLI de Azure, Azure PowerShell y API REST.
ms.date: 08/17/2020
ms.topic: how-to
ms.openlocfilehash: 9b0c2e50536a847555dfa5cc6b9c823cfc1a4cfa
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89047063"
---
# <a name="programmatically-create-policies"></a>Creación de directivas mediante programación

Este artículo le guiará a través de la creación y administración de directivas mediante programación. Las definiciones de Azure Policy aplican reglas y efectos diferentes sobre los recursos. Su aplicación garantiza que los recursos cumplan los estándares corporativos y los acuerdos de nivel de servicio.

Para información sobre el cumplimiento, consulte cómo [obtener datos de cumplimiento](get-compliance-data.md).

## <a name="prerequisites"></a>Requisitos previos

Asegúrese de que se cumplen los siguientes requisitos previos antes de empezar:

1. Si aún no lo ha hecho, instale [ARMClient](https://github.com/projectkudu/ARMClient). Esta es una herramienta que envía solicitudes HTTP a las API basadas en Azure Resource Manager.

1. Actualice el módulo de Azure PowerShell a la versión más reciente. Consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps) para más información. Para más información acerca de la versión más reciente, consulte [Azure PowerShell](https://github.com/Azure/azure-powershell/releases).

1. Registre el proveedor de recursos de Azure Policy Insights con Azure PowerShell para validar que la suscripción funciona con el proveedor de recursos. Para registrar un proveedor de recursos, debe tener permiso para ejecutar la operación de registro de una acción para él. Esta operación está incluida en los roles Colaborador y Propietario. Para registrar el proveedor de recursos, ejecute el siguiente comando:

   ```azurepowershell-interactive
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.PolicyInsights'
   ```

   Para más información sobre el registro y la visualización de los proveedores de recursos, consulte [Tipos y proveedores de recursos](../../../azure-resource-manager/management/resource-providers-and-types.md).

1. Si aún no lo ha hecho, instale la CLI de Azure. Puede obtener la versión más reciente en [Instalación de la CLI de Azure en Windows](/cli/azure/install-azure-cli-windows).

## <a name="create-and-assign-a-policy-definition"></a>Creación y asignación de una definición de directiva

El primer paso hacia una mejor visibilidad de los recursos es crear y asignar directivas sobre los recursos. El siguiente paso es aprender a crear y asignar una directiva mediante programación. La directiva de ejemplo audita cuentas de almacenamiento abiertas a todas las redes públicas con PowerShell, la CLI de Azure y solicitudes HTTP.

### <a name="create-and-assign-a-policy-definition-with-powershell"></a>Creación y asignación de una definición de directiva con PowerShell

1. Utilice el siguiente fragmento JSON para crear un archivo JSON con el nombre AuditStorageAccounts.json.

   ```json
   {
       "if": {
           "allOf": [{
                   "field": "type",
                   "equals": "Microsoft.Storage/storageAccounts"
               },
               {
                   "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                   "equals": "Allow"
               }
           ]
       },
       "then": {
           "effect": "audit"
       }
   }
   ```

   Para más información acerca de la creación de una definición de directiva, consulte [Estructura de definición de Azure Policy](../concepts/definition-structure.md).

1. Ejecute el siguiente comando para crear una definición de directiva mediante el archivo AuditStorageAccounts.json.

   ```azurepowershell-interactive
   New-AzPolicyDefinition -Name 'AuditStorageAccounts' -DisplayName 'Audit Storage Accounts Open to Public Networks' -Policy 'AuditStorageAccounts.json'
   ```

   El comando crea una definición de directiva denominada _Audit Storage Accounts Open to Public Networks_ (Auditoría de cuentas de almacenamiento abiertas a las redes públicas).
   Para más información acerca de otros parámetros que puede utilizar, consulte [New-AzPolicyDefinition](/powershell/module/az.resources/new-azpolicydefinition).

   Cuando se llama sin parámetros de ubicación, `New-AzPolicyDefinition` elige de forma predeterminada guardar la definición de directiva en la suscripción seleccionada del contexto de sesiones. Para guardar la definición en una ubicación diferente, use los siguientes parámetros:

   - **SubscriptionId**: se guarda en una suscripción diferente. Requiere un valor de _GUID_.
   - **ManagementGroupName**: se guarda en un grupo de administración. Requiere un valor de _cadena_.

1. Después de crear la definición de directiva, puede crear una asignación de directiva mediante la ejecución de los siguientes comandos:

   ```azurepowershell-interactive
   $rg = Get-AzResourceGroup -Name 'ContosoRG'
   $Policy = Get-AzPolicyDefinition -Name 'AuditStorageAccounts'
   New-AzPolicyAssignment -Name 'AuditStorageAccounts' -PolicyDefinition $Policy -Scope $rg.ResourceId
   ```

   Reemplace _ContosoRG_ por el nombre del grupo de recursos que desee.

   El parámetro **Scope** de `New-AzPolicyAssignment` funciona con un grupo de administración, una suscripción, un grupo de recursos o un único recurso. El parámetro utiliza una ruta de acceso de recurso completo, que devuelve la propiedad **ResourceId** en `Get-AzResourceGroup`. El patrón de **Scope** para cada contenedor es como sigue. Reemplace `{rName}`, `{rgName}`, `{subId}` y `{mgName}` por el nombre del recurso, el nombre del grupo de recursos, el identificador de suscripción y el nombre del grupo de administración, respectivamente.
   `{rType}` se reemplazaría por el **tipo de recurso** del recurso como, por ejemplo, `Microsoft.Compute/virtualMachines` para una máquina virtual.

   - Recurso `/subscriptions/{subID}/resourceGroups/{rgName}/providers/{rType}/{rName}`
   - Grupo de recursos `/subscriptions/{subId}/resourceGroups/{rgName}`
   - Suscripción `/subscriptions/{subId}/`
   - Grupo de administración `/providers/Microsoft.Management/managementGroups/{mgName}`

Para más información acerca de cómo administrar las directivas de recursos mediante el módulo PowerShell de Resource Manager, consulte [Az.Resources](/powershell/module/az.resources/#policies).

### <a name="create-and-assign-a-policy-definition-using-armclient"></a>Creación y asignación de una definición de directiva con ARMClient

Use el procedimiento siguiente para crear una definición de directiva.

1. Copie el siguiente fragmento de código JSON para crear un archivo JSON. En el paso siguiente llamará al archivo.

   ```json
   "properties": {
       "displayName": "Audit Storage Accounts Open to Public Networks",
       "policyType": "Custom",
       "mode": "Indexed",
       "description": "This policy ensures that storage accounts with exposure to Public Networks are audited.",
       "parameters": {},
       "policyRule": {
           "if": {
               "allOf": [{
                       "field": "type",
                       "equals": "Microsoft.Storage/storageAccounts"
                   },
                   {
                       "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                       "equals": "Allow"
                   }
               ]
           },
           "then": {
               "effect": "audit"
           }
       }
   }
   ```

1. Cree la definición de directiva con una de las llamadas siguientes:

   ```console
   # For defining a policy in a subscription
   armclient PUT "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyDefinitions/AuditStorageAccounts?api-version=2019-09-01" @<path to policy definition JSON file>

   # For defining a policy in a management group
   armclient PUT "/providers/Microsoft.Management/managementgroups/{managementGroupId}/providers/Microsoft.Authorization/policyDefinitions/AuditStorageAccounts?api-version=2019-09-01" @<path to policy definition JSON file>
   ```

   Sustituya el valor de {subscriptionId} anterior por el identificador de su suscripción o {managementGroupId} por el identificador de su [grupo de administración](../../management-groups/overview.md).

   Para más información acerca de la estructura de la consulta, consulte [Azure Policy Definitions – Create or Update](/rest/api/resources/policydefinitions/createorupdate) (Definiciones de Azure Policy: creación o actualización) y [Policy Definitions – Create or Update At Management Group](/rest/api/resources/policydefinitions/createorupdateatmanagementgroup) (Definiciones de directiva: creación o actualización en el grupo de administración).

Utilice el procedimiento siguiente para crear una asignación de directiva y asignar la definición de directiva en el nivel de grupo de recursos.

1. Copie el siguiente fragmento de código JSON para crear un archivo de asignación de directiva JSON. Reemplace la información de ejemplo de los símbolos &lt;&gt; por sus propios valores.

   ```json
   {
       "properties": {
           "description": "This policy assignment makes sure that storage accounts with exposure to Public Networks are audited.",
           "displayName": "Audit Storage Accounts Open to Public Networks Assignment",
           "parameters": {},
           "policyDefinitionId": "/subscriptions/<subscriptionId>/providers/Microsoft.Authorization/policyDefinitions/Audit Storage Accounts Open to Public Networks",
           "scope": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>"
       }
   }
   ```

1. Cree la asignación de directiva con la llamada siguiente:

   ```console
   armclient PUT "/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.Authorization/policyAssignments/Audit Storage Accounts Open to Public Networks?api-version=2019-09-01" @<path to Assignment JSON file>
   ```

   Reemplace la información de ejemplo de los símbolos &lt;&gt; por sus propios valores.

   Para más información acerca de cómo realizar llamadas HTTP a la API de REST, consulte [Recursos de la API de REST de Azure](/rest/api/resources/).

### <a name="create-and-assign-a-policy-definition-with-azure-cli"></a>Creación y asignación de una definición de directiva con la CLI de Azure

Use el procedimiento siguiente para crear una definición de directiva:

1. Copie el siguiente fragmento de código JSON para crear un archivo de asignación de directiva JSON.

   ```json
   {
       "if": {
           "allOf": [{
                   "field": "type",
                   "equals": "Microsoft.Storage/storageAccounts"
               },
               {
                   "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                   "equals": "Allow"
               }
           ]
       },
       "then": {
           "effect": "audit"
       }
   }
   ```

   Para más información acerca de la creación de una definición de directiva, consulte [Estructura de definición de Azure Policy](../concepts/definition-structure.md).

1. Para crear una definición de directiva, ejecute el siguiente comando:

   ```azurecli-interactive
   az policy definition create --name 'audit-storage-accounts-open-to-public-networks' --display-name 'Audit Storage Accounts Open to Public Networks' --description 'This policy ensures that storage accounts with exposures to public networks are audited.' --rules '<path to json file>' --mode All
   ```

   El comando crea una definición de directiva denominada _Audit Storage Accounts Open to Public Networks_ (Auditoría de cuentas de almacenamiento abiertas a las redes públicas).
   Para más información acerca de otros parámetros que puede utilizar, consulte [creación de la definición de directivas az](/cli/azure/policy/definition#az-policy-definition-create).

   Cuando se llama sin parámetros de ubicación, `az policy definition creation` elige de forma predeterminada guardar la definición de directiva en la suscripción seleccionada del contexto de sesiones. Para guardar la definición en una ubicación diferente, use los siguientes parámetros:

   - **subscription**: se guarda en otra suscripción. Requiere un valor de _GUID_ como identificador de suscripción o un valor de _cadena_ como nombre.
   - **management-group**: se guarda en un grupo de administración. Requiere un valor de _cadena_.

1. Use el siguiente comando para crear una asignación de directiva. Reemplace la información de ejemplo de los símbolos &lt;&gt; por sus propios valores.

   ```azurecli-interactive
   az policy assignment create --name '<name>' --scope '<scope>' --policy '<policy definition ID>'
   ```

   El parámetro **scope** de `az policy assignment create` funciona con un grupo de administración, una suscripción, un grupo de recursos o un único recurso. El parámetro utiliza una ruta de acceso de recurso completa. El patrón de **scope** para cada contenedor es el siguiente. Reemplace `{rName}`, `{rgName}`, `{subId}` y `{mgName}` por el nombre del recurso, el nombre del grupo de recursos, el identificador de suscripción y el nombre del grupo de administración, respectivamente. `{rType}` se reemplazaría por el **tipo de recurso** del recurso como, por ejemplo, `Microsoft.Compute/virtualMachines` para una máquina virtual.

   - Recurso `/subscriptions/{subID}/resourceGroups/{rgName}/providers/{rType}/{rName}`
   - Grupo de recursos `/subscriptions/{subID}/resourceGroups/{rgName}`
   - Suscripción `/subscriptions/{subID}`
   - Grupo de administración `/providers/Microsoft.Management/managementGroups/{mgName}`

Puede obtener el id. de definición de Azure Policy si usa PowerShell con el comando siguiente:

```azurecli-interactive
az policy definition show --name 'Audit Storage Accounts with Open Public Networks'
```

El identificador de definición de directiva para la definición de directiva que ha creado debe parecerse al ejemplo siguiente:

```output
"/subscription/<subscriptionId>/providers/Microsoft.Authorization/policyDefinitions/Audit Storage Accounts Open to Public Networks"
```

Para más información acerca de cómo administrar las directivas de recursos con la CLI de Azure, consulte las [directivas de recursos de la CLI de Azure](/cli/azure/policy).

## <a name="next-steps"></a>Pasos siguientes

Revise los artículos siguientes para más información sobre los comandos y las consultas en este artículo.

- [Recursos de la API de REST de Azure](/rest/api/resources/)
- [Módulos de Azure PowerShell](/powershell/module/az.resources/#policies)
- [Comandos de directiva de la CLI de Azure](/cli/azure/policy)
- [Referencia de API de REST de proveedor de recursos de Azure Policy Insights](/rest/api/policy-insights)
- [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md).