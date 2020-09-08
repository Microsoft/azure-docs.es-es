---
title: Bloqueo de recursos para impedir cambios
description: Impida que los usuarios actualicen o eliminen recursos de Azure esenciales aplicando un bloqueo para todos los usuarios y roles.
ms.topic: conceptual
ms.date: 06/17/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: e76287c4524831a84a22fb23ddf8a5fdee8bc12b
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87827289"
---
# <a name="lock-resources-to-prevent-unexpected-changes"></a>Bloqueo de recursos para impedir cambios inesperados

Como administrador, puede que tenga que bloquear una suscripción, un grupo de recursos o un recurso para impedir que otros usuarios de su organización eliminen o modifiquen accidentalmente recursos esenciales. Puede establecer el bloqueo de nivel en **CanNotDelete** o **ReadOnly**. En el portal, los bloqueos se denominan **Delete** y **Read-only** respectivamente.

* **CanNotDelete** significa que los usuarios autorizados pueden leer y modificar recursos, pero no eliminarlos.
* **ReadOnly** significa que los usuarios autorizados solo pueden leer recursos, pero no actualizarlos ni eliminarlos. Aplicar este bloqueo es similar a restringir todos los usuarios autorizados a los permisos concedidos por el rol **Lector**.

## <a name="how-locks-are-applied"></a>Cómo se aplican los bloqueos

Cuando se aplica un bloqueo en un ámbito primario, todos los recursos heredan el mismo bloqueo. Incluso los recursos que agregue posteriormente heredan el bloqueo del elemento primario. El bloqueo más restrictivo de toda la herencia tiene prioridad.

Al diferencia del control de acceso basado en rol, los bloqueos de administración se usan para aplicar una restricción a todos los usuarios y roles. Para información sobre cómo establecer permisos para usuarios y roles, consulte [Control de acceso basado en rol de Azure (RBAC de Azure)](../../role-based-access-control/role-assignments-portal.md).

Los bloqueos de Resource Manager solo se aplican a las operaciones que se producen en el plano de la administración, que consta de las operaciones enviadas a `https://management.azure.com`. Los bloqueos no restringen cómo los recursos realizan sus propias funciones. Los cambios de recursos están restringidos, pero no así las operaciones de recursos. Por ejemplo, un bloqueo ReadOnly en una instancia de SQL Database impide que elimine o modifique la base de datos. Pero no le impide crear, actualizar o eliminar datos de la base de datos. Se permiten las transacciones de datos porque esas operaciones no se envían a `https://management.azure.com`.

## <a name="considerations-before-applying-locks"></a>Consideraciones antes de aplicar bloqueos

Aplicar bloqueos puede provocar resultados inesperados, ya que algunas operaciones que no parecen modificar el recurso realmente requieren acciones que el bloqueo ha bloqueado. Algunos ejemplos comunes de las operaciones los bloqueos bloquean son:

* Un bloqueo de solo lectura en una **cuenta de almacenamiento** impide que todos los usuarios muestren las claves. La operación de visualización claves se administra mediante solicitudes POST debido a que las claves devueltas están disponibles para las operaciones de escritura.

* Un bloqueo de solo lectura en un recurso de **App Service** evita que el Explorador de servidores de Visual Studio muestre los archivos del recurso, ya que esa interacción requiere acceso de escritura.

* Un bloqueo de solo lectura en un **grupo de recursos** que contiene una **máquina virtual impide** que todos los usuarios inicien o reinicien la máquina virtual. Estas operaciones requieren una solicitud POST.

* Un bloqueo que no se puede eliminar en un **grupo de recursos** impide que Azure Resource Manager [elimine de forma automática las implementaciones](../templates/deployment-history-deletions.md) en el historial. Si alcanza 800 implementaciones en el historial, se producirá un error en las implementaciones.

* Un bloqueo de no se puede eliminar en el **grupo de recursos** creado por el **Servicio Azure Backup** genera un error en las copias de seguridad. El servicio admite un máximo de 18 puntos de restauración. Cuando está bloqueado, el servicio de copia de seguridad no puede limpiar los puntos de restauración. Para más información, consulte [Preguntas más frecuentes sobre la copia de seguridad de máquinas virtuales de Azure](../../backup/backup-azure-vm-backup-faq.md).

* Un bloqueo de solo lectura en una **suscripción** impide que **Azure Advisor** funcione correctamente. Advisor no puede almacenar los resultados de sus consultas.

## <a name="who-can-create-or-delete-locks"></a>Quién puede crear o eliminar bloqueos

Para crear o eliminar bloqueos de administración, debe tener acceso a las acciones `Microsoft.Authorization/*` o `Microsoft.Authorization/locks/*`. Entre los roles integrados, solamente se conceden esas acciones al **propietario** y al **administrador de acceso de usuarios**.

## <a name="managed-applications-and-locks"></a>Bloqueos y aplicaciones administradas

Algunos servicios de Azure, como Azure Databricks, utilizan [aplicaciones administradas](../managed-applications/overview.md) para implementar el servicio. En ese caso, el servicio crea dos grupos de recursos. Un grupo de recursos contiene información general sobre el servicio y no está bloqueado. El otro grupo de recursos contiene la infraestructura para el servicio y está bloqueado.

Si intenta eliminar el grupo de recursos de infraestructura, obtendrá un error que indica que el grupo de recursos está bloqueado. Si intenta eliminar el bloqueo para el grupo de recursos de infraestructura, obtendrá un error que indica que el bloqueo no puede eliminarse porque pertenece a una aplicación del sistema.

En su lugar, elimine el servicio, lo que también elimina el grupo de recursos de infraestructura.

Para las aplicaciones administradas, seleccione el servicio implementado.

![Seleccionar servicio](./media/lock-resources/select-service.png)

Tenga en cuenta que el servicio incluye un vínculo para un **Grupo de recursos administrado**. Ese grupo de recursos contiene la infraestructura y está bloqueado. No se puede eliminar directamente.

![Mostrar grupo administrado](./media/lock-resources/show-managed-group.png)

Para eliminar todo el contenido para el servicio, incluido el grupo de recursos de infraestructura bloqueado, seleccione **Eliminar** para el servicio.

![Eliminar servicio](./media/lock-resources/delete-service.png)

## <a name="portal"></a>Portal

[!INCLUDE [resource-manager-lock-resources](../../../includes/resource-manager-lock-resources.md)]

## <a name="template"></a>Plantilla

Al usar una plantilla de Resource Manager para implementar un bloqueo, se usan valores diferentes para el nombre y el tipo, según el ámbito del bloqueo.

Al aplicar un bloqueo a un **recurso**, use los siguientes formatos:

* nombre: `{resourceName}/Microsoft.Authorization/{lockName}`
* tipo: `{resourceProviderNamespace}/{resourceType}/providers/locks`

Al aplicar un bloqueo a un **grupo de recursos** o **suscripción**, use los siguientes formatos:

* nombre: `{lockName}`
* tipo: `Microsoft.Authorization/locks`

En el ejemplo siguiente se muestra una plantilla que crea un plan de App Service, un sitio web y un bloqueo en el sitio web. El tipo de recurso del bloqueo es el tipo de recurso del recurso que se bloqueará y **/providers/locks**. El nombre del bloqueo se crea mediante la concatenación del nombre del recurso con **/Microsoft.Authorization/** y el propio nombre del bloqueo.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "hostingPlanName": {
      "type": "string"
    }
  },
  "variables": {
    "siteName": "[concat('ExampleSite', uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2019-08-01",
      "name": "[parameters('hostingPlanName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "tier": "Free",
        "name": "f1",
        "capacity": 0
      },
      "properties": {
        "targetWorkerCount": 1
      }
    },
    {
      "type": "Microsoft.Web/sites",
      "apiVersion": "2019-08-01",
      "name": "[variables('siteName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', parameters('hostingPlanName'))]"
      ],
      "properties": {
        "serverFarmId": "[parameters('hostingPlanName')]"
      }
    },
    {
      "type": "Microsoft.Web/sites/providers/locks",
      "apiVersion": "2016-09-01",
      "name": "[concat(variables('siteName'), '/Microsoft.Authorization/siteLock')]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', variables('siteName'))]"
      ],
      "properties": {
        "level": "CanNotDelete",
        "notes": "Site should not be deleted."
      }
    }
  ]
}
```

Para obtener un ejemplo sobre cómo establecer un bloqueo en un grupo de recursos, consulte [Crear un grupo de recursos y bloquearlo](https://github.com/Azure/azure-quickstart-templates/tree/master/subscription-deployments/create-rg-lock-role-assignment).

## <a name="powershell"></a>PowerShell

Bloquee recursos implementados con Azure PowerShell mediante el comando [New-AzResourceLock](/powershell/module/az.resources/new-azresourcelock).

Para bloquear un recurso, proporcione el nombre del recurso, su tipo y el nombre del grupo de recursos.

```azurepowershell-interactive
New-AzResourceLock -LockLevel CanNotDelete -LockName LockSite -ResourceName examplesite -ResourceType Microsoft.Web/sites -ResourceGroupName exampleresourcegroup
```

Para bloquear un grupo de recursos, proporcione el nombre del grupo de recursos.

```azurepowershell-interactive
New-AzResourceLock -LockName LockGroup -LockLevel CanNotDelete -ResourceGroupName exampleresourcegroup
```

Para obtener información sobre un bloqueo, use [Get-AzResourceLock](/powershell/module/az.resources/get-azresourcelock). Para obtener todos los bloqueos en su suscripción, use:

```azurepowershell-interactive
Get-AzResourceLock
```

Para obtener todos los bloqueos para un recurso, use:

```azurepowershell-interactive
Get-AzResourceLock -ResourceName examplesite -ResourceType Microsoft.Web/sites -ResourceGroupName exampleresourcegroup
```

Para obtener todos los bloqueos para un grupo de recursos, use:

```azurepowershell-interactive
Get-AzResourceLock -ResourceGroupName exampleresourcegroup
```

Para eliminar un bloqueo, use:

```azurepowershell-interactive
$lockId = (Get-AzResourceLock -ResourceGroupName exampleresourcegroup -ResourceName examplesite -ResourceType Microsoft.Web/sites).LockId
Remove-AzResourceLock -LockId $lockId
```

## <a name="azure-cli"></a>Azure CLI

Bloquee recursos implementados con la CLI de Azure mediante el comando [az lock create](/cli/azure/lock#az-lock-create).

Para bloquear un recurso, proporcione el nombre del recurso, su tipo y el nombre del grupo de recursos.

```azurecli
az lock create --name LockSite --lock-type CanNotDelete --resource-group exampleresourcegroup --resource-name examplesite --resource-type Microsoft.Web/sites
```

Para bloquear un grupo de recursos, proporcione el nombre del grupo de recursos.

```azurecli
az lock create --name LockGroup --lock-type CanNotDelete --resource-group exampleresourcegroup
```

Para obtener información sobre un bloqueo, use [az lock list](/cli/azure/lock#az-lock-list). Para obtener todos los bloqueos en su suscripción, use:

```azurecli
az lock list
```

Para obtener todos los bloqueos para un recurso, use:

```azurecli
az lock list --resource-group exampleresourcegroup --resource-name examplesite --namespace Microsoft.Web --resource-type sites --parent ""
```

Para obtener todos los bloqueos para un grupo de recursos, use:

```azurecli
az lock list --resource-group exampleresourcegroup
```

Para eliminar un bloqueo, use:

```azurecli
lockid=$(az lock show --name LockSite --resource-group exampleresourcegroup --resource-type Microsoft.Web/sites --resource-name examplesite --output tsv --query id)
az lock delete --ids $lockid
```

## <a name="rest-api"></a>API DE REST

Puede bloquear los recursos implementados con la [API de REST para bloqueos de administración](/rest/api/resources/managementlocks). La API de REST le permite crear y eliminar bloqueos, y recuperar información acerca de los bloqueos existentes.

Para crear un bloqueo, ejecute:

```http
PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/locks/{lock-name}?api-version={api-version}
```

El ámbito puede ser una suscripción, un grupo de recursos o un recurso. El nombre del bloqueo es el nombre con el que desee llamar al bloqueo. Como versión de la API, use **2016-09-01**.

En la solicitud, incluya un objeto JSON que especifique las propiedades para el bloqueo.

```json
{
  "properties": {
  "level": "CanNotDelete",
  "notes": "Optional text notes."
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

* Para aprender a organizar de manera lógica los recursos, consulte [Uso de etiquetas para organizar sus recursos](tag-resources.md).
* Puede aplicar restricciones y convenciones a través de su suscripción con directivas personalizadas. Para obtener más información, consulte [¿Qué es Azure Policy?](../../governance/policy/overview.md)
* Para obtener instrucciones sobre cómo las empresas pueden utilizar Resource Manager para administrar eficazmente las suscripciones, vea [Scaffold empresarial de Azure: Gobernanza de suscripción prescriptiva](/azure/architecture/cloud-adoption-guide/subscription-governance).
