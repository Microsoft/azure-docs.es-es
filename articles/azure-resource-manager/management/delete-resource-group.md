---
title: Eliminación del grupo de recursos y los recursos
description: Se describe cómo eliminar grupos de recursos y recursos. También se describe cómo Azure Resource Manager ordena la eliminación de recursos al eliminar un grupo de recursos. Describe los códigos de respuesta y cómo Resource Manager los controla para determinar si la eliminación se realizó correctamente.
ms.topic: conceptual
ms.date: 09/03/2019
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 2720a9ae8c0a67d8e6f9c1105b33c0a3c343cf47
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498504"
---
# <a name="azure-resource-manager-resource-group-and-resource-deletion"></a>Eliminación de grupos de recursos y recursos en Azure Resource Manager

En este artículo se muestra cómo eliminar grupos de recursos y recursos. Se describe cómo Azure Resource Manager ordena la eliminación de recursos al eliminar un grupo de recursos.

## <a name="how-order-of-deletion-is-determined"></a>Cómo se determina el orden de eliminación

Cuando se elimina un grupo de recursos, Resource Manager determina el orden para eliminar los recursos. Usa el orden siguiente:

1. Se eliminan todos los recursos secundarios (anidados).

2. Los recursos que administran otros recursos se eliminan a continuación. Un recurso puede tener establecida la propiedad `managedBy` para indicar que un recurso diferente lo administra. Cuando se establece esta propiedad, se elimina el recurso que administra el otro recurso antes que los demás.

3. El resto de los recursos se elimina después de las dos categorías anteriores.

Después de determinar el orden, el Administrador de recursos emite una operación de eliminación para cada recurso. Espera a que todas las dependencias finalicen antes de continuar.

En las operaciones sincrónicas, los códigos de respuesta correcta esperados son:

* 200
* 204
* 404

En las operaciones asincrónicas, la respuesta correcta esperada es 202. Resource Manager realiza el seguimiento del encabezado de ubicación o del encabezado de la operación asincrónica de Azure para determinar el estado de la operación de eliminación asincrónica.
  
### <a name="deletion-errors"></a>Errores de eliminación

Cuando una operación de eliminación devuelve un error, Resource Manager vuelve a intentar la llamada a DELETE. Los reintentos se producen para los códigos de estado 5xx, 429 y 408. De forma predeterminada, el período de tiempo de reintento es de 15 minutos.

## <a name="after-deletion"></a>Después de la eliminación

Resource Manager emite una llamada GET en cada recurso que ha intentado eliminar. Se espera que la respuesta de la llamada sea 404. Cuando Resource Manager obtiene un error 404, considera que la eliminación se ha completado correctamente. Resource Manager quita el recurso de su memoria caché.

Sin embargo, si la llamada a GET en el recurso devuelve una respuesta 200 o 201, Resource Manager vuelve a crear el recurso.

Si la operación GET devuelve un error, Resource Manager vuelve a intentar la operación GET para el código de error siguiente:

* Menor que 100
* 408
* 429
* Mayor que 500

Con otros códigos de error, Resource Manager no puede eliminar el recurso.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible.

## <a name="delete-resource-group"></a>Eliminación de un grupo de recursos

Use uno de los métodos siguientes para eliminar el grupo de recursos.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Remove-AzResourceGroup -Name ExampleResourceGroup
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az group delete --name ExampleResourceGroup
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En el [portal](https://portal.azure.com), seleccione el grupo de recursos que quiere eliminar.

1. Seleccione **Eliminar grupo de recursos**.

   ![Eliminación de un grupo de recursos](./media/delete-resource-group/delete-group.png)

1. Para confirmar la eliminación, escriba el nombre del grupo de recursos.

---

## <a name="delete-resource"></a>Eliminación de un recurso

Use uno de los métodos siguientes para eliminar un recurso.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Remove-AzResource `
  -ResourceGroupName ExampleResourceGroup `
  -ResourceName ExampleVM `
  -ResourceType Microsoft.Compute/virtualMachines
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az resource delete \
  --resource-group ExampleResourceGroup \
  --name ExampleVM \
  --resource-type "Microsoft.Compute/virtualMachines"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En el [portal](https://portal.azure.com), seleccione el recurso que quiere eliminar.

1. Seleccione **Eliminar**. La captura de pantalla siguiente muestra las opciones de administración de una máquina virtual.

   ![Eliminación de un recurso](./media/delete-resource-group/delete-resource.png)

1. Cuando se le solicite, confirme la eliminación.

---


## <a name="next-steps"></a>Pasos siguientes

* Para comprender los conceptos de Resource Manager, consulte [Información general sobre Azure Resource Manager](overview.md).
* Para los comandos de eliminación, consulte [PowerShell](/powershell/module/az.resources/Remove-AzResourceGroup), [CLI de Azure](/cli/azure/group?view=azure-cli-latest#az-group-delete), y [API de REST](/rest/api/resources/resourcegroups/delete).
