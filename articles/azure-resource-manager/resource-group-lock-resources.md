---
title: Bloqueo de recursos de Azure para impedir cambios | Microsoft Docs
description: Impida que los usuarios actualicen o eliminen recursos de Azure esenciales aplicando un bloqueo para todos los usuarios y roles.
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 05/14/2019
ms.author: tomfitz
ms.openlocfilehash: a6c7983d22eed4a4232fbb2db490c1743684a04c
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65813400"
---
# <a name="lock-resources-to-prevent-unexpected-changes"></a>Bloqueo de recursos para impedir cambios inesperados 

Como administrador, puede que tenga que bloquear una suscripción, un grupo de recursos o un recurso para impedir que otros usuarios de su organización eliminen o modifiquen accidentalmente recursos esenciales. Puede establecer el bloqueo de nivel en **CanNotDelete** o **ReadOnly**. En el portal, los bloqueos se denominan **Delete** y **Read-only** respectivamente.

* **CanNotDelete** significa que los usuarios autorizados pueden leer y modificar recursos, pero no eliminarlos. 
* **ReadOnly** significa que los usuarios autorizados solo pueden leer recursos, pero no actualizarlos ni eliminarlos. Aplicar este bloqueo es similar a restringir todos los usuarios autorizados a los permisos concedidos por el rol **Lector**. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="how-locks-are-applied"></a>Cómo se aplican los bloqueos

Cuando se aplica un bloqueo en un ámbito primario, todos los recursos heredan el mismo bloqueo. Incluso los recursos que agregue posteriormente heredan el bloqueo del elemento primario. El bloqueo más restrictivo de toda la herencia tiene prioridad.

Al diferencia del control de acceso basado en rol, los bloqueos de administración se usan para aplicar una restricción a todos los usuarios y roles. Para obtener información sobre cómo establecer permisos para usuarios y roles, vea [Control de acceso basado en roles de Azure](../role-based-access-control/role-assignments-portal.md).

Los bloqueos de Resource Manager solo se aplican a las operaciones que se producen en el plano de la administración, que consta de las operaciones enviadas a `https://management.azure.com`. Los bloqueos no restringen cómo los recursos realizan sus propias funciones. Los cambios de recursos están restringidos, pero no así las operaciones de recursos. Por ejemplo, un bloqueo ReadOnly en una instancia de SQL Database impide que elimine o modifique la base de datos. Pero no le impide crear, actualizar o eliminar datos de la base de datos. Las transacciones de datos se permiten porque esas operaciones no se envían a `https://management.azure.com`.

Aplicar **ReadOnly** puede provocar resultados inesperados ya que algunas operaciones que no parecen modificar el recurso realmente requieren acciones que están bloqueadas por el bloqueo. El **ReadOnly** bloqueo puede aplicarse en el recurso o en el grupo de recursos que contiene el recurso. Algunos ejemplos comunes de las operaciones que están bloqueadas por un **ReadOnly** bloqueo son:

* Un **ReadOnly** bloqueo en una cuenta de almacenamiento impide que todos los usuarios muestren las claves. La operación de visualización claves se administra mediante solicitudes POST debido a que las claves devueltas están disponibles para las operaciones de escritura.

* Un bloqueo **ReadOnly** en un recurso de App Service evita que el Explorador de servidores de Visual Studio muestre los archivos del recurso, ya que esa interacción requiere acceso de escritura.

* Un **ReadOnly** bloqueo en un grupo de recursos que contiene una máquina virtual, impide que todos los usuarios de iniciar o reiniciar la máquina virtual. Estas operaciones requieren una solicitud POST.

## <a name="who-can-create-or-delete-locks"></a>Quién puede crear o eliminar bloqueos
Para crear o eliminar bloqueos de administración, debe tener acceso a las acciones `Microsoft.Authorization/*` o `Microsoft.Authorization/locks/*`. Entre los roles integrados, solamente se conceden esas acciones al **propietario** y al **administrador de acceso de usuarios**.

## <a name="managed-applications-and-locks"></a>Las aplicaciones y bloqueos

Algunos servicios de Azure, como Azure Databricks, utilizan [aplicaciones administradas](../managed-applications/overview.md) para implementar el servicio. En ese caso, el servicio crea dos grupos de recursos. Un grupo de recursos contiene información general sobre el servicio y no está bloqueado. Otro grupo de recursos contiene la infraestructura para el servicio y está bloqueado.

Si intenta eliminar el grupo de recursos de infraestructura, obtendrá un error que indica que el grupo de recursos está bloqueado. Si intenta eliminar el bloqueo para el grupo de recursos de infraestructura, obtendrá un error que indica que el bloqueo no puede eliminarse porque pertenece a una aplicación del sistema.

En su lugar, elimine el servicio, lo que también elimina el grupo de recursos de infraestructura.

Para las aplicaciones administradas, seleccione el servicio implementado.

![Seleccionar servicio](./media/resource-group-lock-resources/select-service.png)

Tenga en cuenta el servicio incluye un vínculo para un **el grupo de recursos administrado**. Ese grupo de recursos contiene la infraestructura y se bloquea. No se puede eliminar directamente.

![Mostrar el grupo administrado](./media/resource-group-lock-resources/show-managed-group.png)

Para eliminar todo el contenido para el servicio, incluido el grupo de recursos de infraestructura bloqueado, seleccione **eliminar** para el servicio.

![Eliminar el servicio](./media/resource-group-lock-resources/delete-service.png)

## <a name="portal"></a>Portal
[!INCLUDE [resource-manager-lock-resources](../../includes/resource-manager-lock-resources.md)]

## <a name="template"></a>Plantilla

Al usar una plantilla de Resource Manager para implementar un bloqueo, utilice valores diferentes para el nombre y tipo según el ámbito del bloqueo.

Al aplicar un bloqueo a una **recursos**, use los siguientes formatos:

* nombre: `{resourceName}/Microsoft.Authorization/{lockName}`
* type - `{resourceProviderNamespace}/{resourceType}/providers/locks`

Al aplicar un bloqueo a una **grupo de recursos** o **suscripción**, use los siguientes formatos:

* nombre: `{lockName}`
* type - `Microsoft.Authorization/locks`

En el ejemplo siguiente se muestra una plantilla que crea un plan de App Service, un sitio web y un bloqueo en el sitio web. El tipo de recurso del bloqueo es el tipo de recurso del recurso que se bloqueará y **/providers/locks**. El nombre del bloqueo se crea mediante la concatenación del nombre del recurso con **/Microsoft.Authorization/** y el propio nombre del bloqueo.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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
            "apiVersion": "2016-09-01",
            "type": "Microsoft.Web/serverfarms",
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
            "apiVersion": "2016-08-01",
            "name": "[variables('siteName')]",
            "type": "Microsoft.Web/sites",
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

Para obtener un ejemplo de establecer un bloqueo en un grupo de recursos, consulte [crear un grupo de recursos y bloquearla](https://github.com/Azure/azure-quickstart-templates/tree/master/subscription-level-deployments/create-rg-lock-role-assignment).

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

Para obtener información sobre un bloqueo, use [Get-AzureRmResourceLock](/powershell/module/az.resources/get-azresourcelock). Para obtener todos los bloqueos en su suscripción, use:

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
Puede bloquear los recursos implementados con la [API de REST para bloqueos de administración](https://docs.microsoft.com/rest/api/resources/managementlocks). La API de REST le permite crear y eliminar bloqueos, y recuperar información acerca de los bloqueos existentes.

Para crear un bloqueo, ejecute:

    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/locks/{lock-name}?api-version={api-version}

El ámbito puede ser una suscripción, un grupo de recursos o un recurso. El nombre del bloqueo es el nombre con el que desee llamar al bloqueo. Versión de api, use **2016-09-01**.

En la solicitud, incluya un objeto JSON que especifique las propiedades para el bloqueo.

    {
      "properties": {
        "level": "CanNotDelete",
        "notes": "Optional text notes."
      }
    } 

## <a name="next-steps"></a>Pasos siguientes
* Para aprender a organizar de manera lógica los recursos, vea [Uso de etiquetas para organizar sus recursos](resource-group-using-tags.md)
* Puede aplicar restricciones y convenciones a través de su suscripción con directivas personalizadas. Para obtener más información, consulte [¿Qué es Azure Policy?](../governance/policy/overview.md)
* Para obtener instrucciones sobre cómo las empresas pueden utilizar Resource Manager para administrar eficazmente las suscripciones, vea [Scaffold empresarial de Azure: Gobernanza de suscripción prescriptiva](/azure/architecture/cloud-adoption-guide/subscription-governance).

