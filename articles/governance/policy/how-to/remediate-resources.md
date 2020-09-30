---
title: Corrección de recursos no compatibles
description: En esta guía se explica la corrección de los recursos que no son conformes con las directivas de Azure Policy.
ms.date: 08/27/2020
ms.topic: how-to
ms.openlocfilehash: 1274b049d7ce19601968697b22da38f0eb2cb5ff
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88958752"
---
# <a name="remediate-non-compliant-resources-with-azure-policy"></a>Corregir los recursos no conformes con Azure Policy

Los recursos que no son conformes con un directiva **deployIfNotExists** o **modify** se pueden colocar en un estado conforme a través de la **Corrección**. La corrección se consigue indicando a Azure Policy que ejecute el efecto **deployIfNotExists** o las **operaciones de modificación** de la directiva asignada en los recursos existentes, independientemente de que la asignación sea a un grupo de administración, una suscripción, un grupo de recursos o un recurso individual. En este artículo se muestran los pasos necesarios para comprender y realizar correcciones con Azure Policy.

## <a name="how-remediation-security-works"></a>Cómo funciona la seguridad de corrección

Cuando Azure Policy ejecuta la plantilla en la definición de directiva **deployIfNotExists**, lo hace mediante una [identidad administrada](../../../active-directory/managed-identities-azure-resources/overview.md).
Azure Policy crea una identidad administrada para cada asignación, pero debe proporcionar detalles sobre los roles que se conceden a la identidad administrada. Si faltan roles en la identidad administrada, este error se muestra durante la asignación de la directiva o una iniciativa. Al usar el portal, Azure Policy concede automáticamente a la identidad administrada los roles enumerados cuando se inicia la asignación. La propiedad _location_ de la identidad administrada no afecta a su funcionamiento con Azure Policy.

:::image type="content" source="../media/remediate-resources/missing-role.png" alt-text="Identidad administrada - función ausente" border="false":::

> [!IMPORTANT]
> Si un recurso modificado por **deployIfNotExists** o por **modify** está fuera del ámbito de la asignación de la directiva o si la plantilla accede a propiedades de recursos situados fuera del ámbito de la asignación de la directiva, debe [concederse manualmente acceso](#manually-configure-the-managed-identity) a la identidad administrada de la asignación o se producirá un error en la implementación de la corrección.

## <a name="configure-policy-definition"></a>Configurar la definición de directiva

El primer paso consiste en definir los roles que **deployIfNotExists** y **modify** necesitan en la definición de directiva para implementar correctamente el contenido de la plantilla incluida. En la propiedad **details**, agregue una propiedad **roleDefinitionIds**. Esta propiedad es una matriz de cadenas que coinciden con los roles de su entorno. Para consultar un ejemplo completo, vea el [ejemplo de deployIfNotExists](../concepts/effects.md#deployifnotexists-example) o los [ejemplos de modify](../concepts/effects.md#modify-examples).

```json
"details": {
    ...
    "roleDefinitionIds": [
        "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/{roleGUID}",
        "/providers/Microsoft.Authorization/roleDefinitions/{builtinroleGUID}"
    ]
}
```

La propiedad **roleDefinitionIds** utiliza el identificador de recurso completo y no toma el valor **roleName** corto del rol. Para obtener el identificador del rol “Contributor” en su entorno, use el siguiente código:

```azurecli-interactive
az role definition list --name 'Contributor'
```

## <a name="manually-configure-the-managed-identity"></a>Configurar manualmente la identidad administrada

Al crear un trabajo mediante el portal, Azure Policy genera la identidad administrada y le concede los roles definidos en **roleDefinitionIds**. En las siguientes condiciones, los pasos para crear la identidad administrada y asignarle permisos se deben realizar manualmente:

- Al usar el SDK (por ejemplo, Azure PowerShell)
- Cuando la plantilla modifica un recurso fuera del ámbito de asignación
- Cuando la plantilla lee un recurso fuera del ámbito de asignación

### <a name="create-managed-identity-with-powershell"></a>Creación de una identidad administrada con PowerShell

Para crear una identidad administrada durante la asignación de la directiva, debe definirse **Location** y debe usarse **AssignIdentity**. En el siguiente ejemplo se obtiene la definición de la directiva integrada **Implementar cifrado de datos transparente de SQL DB**, se establece el grupo de recursos de destino y después se crea la asignación.

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get the built-in "Deploy SQL DB transparent data encryption" policy definition
$policyDef = Get-AzPolicyDefinition -Id '/providers/Microsoft.Authorization/policyDefinitions/86a912f6-9a06-4e26-b447-11b16ba8659f'

# Get the reference to the resource group
$resourceGroup = Get-AzResourceGroup -Name 'MyResourceGroup'

# Create the assignment using the -Location and -AssignIdentity properties
$assignment = New-AzPolicyAssignment -Name 'sqlDbTDE' -DisplayName 'Deploy SQL DB transparent data encryption' -Scope $resourceGroup.ResourceId -PolicyDefinition $policyDef -Location 'westus' -AssignIdentity
```

La variable `$assignment` contiene ahora el identificador principal de la identidad administrada junto con los valores estándar devueltos cuando se crea una asignación de directiva. Se puede acceder a ella a través de `$assignment.Identity.PrincipalId`.

### <a name="grant-defined-roles-with-powershell"></a>Conceder roles definidos con PowerShell

La nueva identidad administrada debe completar la replicación a través de Azure Active Directory antes de se le puedan conceder los roles necesarios. Una vez se completa la replicación, el siguiente ejemplo itera la definición de directiva en `$policyDef` para **roleDefinitionIds** y usa [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) para conceder la nueva identidad administrada a los roles.

```azurepowershell-interactive
# Use the $policyDef to get to the roleDefinitionIds array
$roleDefinitionIds = $policyDef.Properties.policyRule.then.details.roleDefinitionIds

if ($roleDefinitionIds.Count -gt 0)
{
    $roleDefinitionIds | ForEach-Object {
        $roleDefId = $_.Split("/") | Select-Object -Last 1
        New-AzRoleAssignment -Scope $resourceGroup.ResourceId -ObjectId $assignment.Identity.PrincipalId -RoleDefinitionId $roleDefId
    }
}
```

### <a name="grant-defined-roles-through-portal"></a>Conceder roles definidos a través de Azure Portal

Hay dos maneras de conceder a la identidad administrada de una asignación los roles definidos mediante Azure Portal, mediante **Access control (IAM)** o editando la asignación de directiva o iniciativa y haciendo clic en **Guardar**.

Para agregar un rol a la identidad administrada de la asignación, siga estos pasos:

1. Inicie el servicio Azure Policy en Azure Portal. Para ello, haga clic en **Todos los servicios** y, a continuación, busque y seleccione **Directiva**.

1. Seleccione **Asignaciones** en el panel izquierdo de la página de Azure Policy.

1. Busque la asignación que tiene una identidad administrada y haga clic en el nombre.

1. Busque la propiedad **Id. de asignación** en la página Editar. El identificador de asignación será algo como:

   ```output
   /subscriptions/{subscriptionId}/resourceGroups/PolicyTarget/providers/Microsoft.Authorization/policyAssignments/2802056bfc094dfb95d4d7a5
   ```

   El nombre de la identidad administrada es la última parte del identificador del recurso de asignación, que es `2802056bfc094dfb95d4d7a5` en este ejemplo. Copie esta parte del identificador de recursos de asignación.

1. Navegue hasta el contenedor primario del recurso o recursos (grupo de recursos, suscripción, grupo de administración) al que se debe agregar manualmente la definición de función.

1. Haga clic en el vínculo **Control de acceso (IAM)** en la página de recursos y haga clic en **+ Agregar asignación de rol** en la parte superior de la página de control de acceso.

1. Seleccione el rol adecuado que coincide con un **roleDefinitionIds** de la definición de la directiva.
   Deje **Asignar acceso a** con el valor predeterminado de “Usuario de Azure AD, grupo o aplicación”. En el cuadro **Seleccionar**, pegue o escriba la parte del identificador del recurso de asignación que buscó anteriormente. Una vez finalizada la búsqueda, haga clic en el objeto con el mismo nombre que el identificador seleccionado y haga clic en **Guardar**.

## <a name="create-a-remediation-task"></a>Crear una tarea de corrección

### <a name="create-a-remediation-task-through-portal"></a>Creación de una tarea de corrección a través del portal

Durante la evaluación, la asignación de directiva con los efectos **deployIfNotExists** o **modify** determina si hay recursos no conformes. Cuando se encuentran los recursos no conformes, se proporcionan los detalles en la página **Corrección**. La opción para desencadenar una **tarea de corrección** está junto a la lista de directivas que tienen recursos no página. Esta opción crea una implementación a partir de la plantilla de **deployIfNotExists** o de las operaciones de **modify**.

Para crear un **tarea de corrección**, siga estos pasos:

1. Inicie el servicio Azure Policy en Azure Portal. Para ello, haga clic en **Todos los servicios** y, a continuación, busque y seleccione **Directiva**.

   :::image type="content" source="../media/remediate-resources/search-policy.png" alt-text="Búsqueda de la directiva en todos los servicios" border="false":::

1. Seleccione **Corrección** en el lado izquierdo de la página Azure Policy.

   :::image type="content" source="../media/remediate-resources/select-remediation.png" alt-text="Seleccione Corrección en la página Directiva" border="false":::

1. Todas las asignaciones de directiva de **deployIfNotExists** y **modify** con recursos no conformes se incluyen en la pestaña y la tabla de datos **Directivas que se van a corregir**. Haga clic en una directiva con recursos que no son conformes. Se abre la página **Nueva tarea de corrección**.

   > [!NOTE]
   > Una forma alternativa de abrir la página **Tareas de corrección** consiste en buscar y hacer clic en la directiva desde la página **Cumplimiento** y después hacer clic en el botón **Crear tarea de corrección**.

1. En la página **Nueva tarea de corrección**, filtre los recursos para corregir mediante la elipse **Ámbito** para seleccionar los recursos secundarios a partir de los cuales se asignó la directiva (incluidos los objetos de recursos individuales). Además, utilice la lista desplegable **Ubicaciones** para filtrar más los recursos. Solo los recursos enumerados en la tabla se corregirán.

   :::image type="content" source="../media/remediate-resources/select-resources.png" alt-text="Corrección: seleccione los recursos que quiere corregir" border="false":::

1. Inicie la tarea de corrección cuando se hayan filtrado los recursos, para ello, haga clic en **Corregir**. Se abre la página de cumplimiento de directivas en la pestaña **Tareas de corrección** para mostrar el estado del progreso de las tareas. Las implementaciones que ha creado la tarea de corrección se inician inmediatamente.

   :::image type="content" source="../media/remediate-resources/task-progress.png" alt-text="Corrección: progreso de las tareas de corrección" border="false":::

1. Haga clic en el **tarea de corrección** en la página de cumplimiento de directiva para obtener detalles sobre el progreso. El filtro usado en la tarea se muestra junto con una lista de los recursos que se van a corregir.

1. En la página **Tarea de corrección**, haga clic con el botón derecho en un recurso para ver el recurso o la implementación de la tarea de corrección. Al final de la fila, haga clic en **Eventos relacionados** para ver detalles, como un mensaje de error.

   :::image type="content" source="../media/remediate-resources/resource-task-context-menu.png" alt-text="Corrección: menú contextual de tarea de recurso" border="false":::

Los recursos implementados mediante una **tarea de corrección** se agregan a la pestaña **Recursos implementados** en la página de cumplimiento de la directiva.

### <a name="create-a-remediation-task-through-azure-cli"></a>Creación de una tarea de corrección a través de la CLI de Azure

Para crear una **tarea de corrección** con la CLI de Azure, use los comandos `az policy remediation`. Reemplace `{subscriptionId}` por el identificador de suscripción y `{myAssignmentId}` por el identificador de asignación de la directiva **deployIfNotExists** o **modify**.

```azurecli-interactive
# Login first with az login if not using Cloud Shell

# Create a remediation for a specific assignment
az policy remediation create --name myRemediation --policy-assignment '/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyAssignments/{myAssignmentId}'
```

Para obtener otros comandos de corrección y ejemplos, consulte los comandos [az policy remediation](/cli/azure/policy/remediation).

### <a name="create-a-remediation-task-through-azure-powershell"></a>Creación de una tarea de corrección a través de Azure PowerShell

Para crear una **tarea de corrección** con Azure PowerShell, use los comandos `Start-AzPolicyRemediation`. Reemplace `{subscriptionId}` por el identificador de suscripción y `{myAssignmentId}` por el identificador de asignación de la directiva **deployIfNotExists** o **modify**.

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Create a remediation for a specific assignment
Start-AzPolicyRemediation -Name 'myRemedation' -PolicyAssignmentId '/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyAssignments/{myAssignmentId}'
```

Para obtener otros cmdlets de corrección y ejemplos, consulte el módulo [Az.PolicyInsights](/powershell/module/az.policyinsights/#policy_insights).

### <a name="create-a-remediation-task-during-policy-assignment-in-the-azure-portal"></a>Creación de una tarea de corrección durante la asignación de la directiva en Azure Portal

Una forma optimizada de crear una tarea de corrección es hacerlo desde Azure Portal durante la asignación de la directiva. Si la definición de la directiva a asignar es **deployIfNotExists** o un efecto **Modify**, el asistente de la pestaña **Corrección** ofrece una opción _Crear una tarea de corrección_. Si se selecciona esta opción, se crea una tarea de corrección al mismo tiempo que la asignación de la directiva.

## <a name="next-steps"></a>Pasos siguientes

- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](get-compliance-data.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
