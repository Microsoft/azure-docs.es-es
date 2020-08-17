---
title: Implementación de recursos en varios ámbitos
description: Muestra cómo captar más de un ámbito durante una implementación. El ámbito puede ser un inquilino, grupos de administración, suscripciones y grupos de recursos.
ms.topic: conceptual
ms.date: 07/28/2020
ms.openlocfilehash: 6161401ac039551a814b595715f56df1ac62dd6c
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87374479"
---
# <a name="deploy-azure-resources-across-scopes"></a>Implementación de recursos de Azure en varios ámbitos

Con las plantillas de Azure Resource Manager (plantillas de ARM), puede implementar en más de un ámbito en una sola implementación. Los ámbitos disponibles son un inquilino, grupos de administración, suscripciones y grupos de recursos. Por ejemplo, puede implementar recursos en un grupo de recursos y en la misma plantilla implementar recursos en otro grupo de recursos. También puede implementar recursos en un grupo de administración, así como implementar recursos en un grupo de recursos dentro de ese grupo de administración.

Las [plantillas anidadas o vinculadas](linked-templates.md) se usan para especificar ámbitos que son diferentes del ámbito principal de la operación de implementación.

## <a name="available-scopes"></a>Ámbitos disponibles

El ámbito que se usa para la operación de implementación determina qué otros ámbitos están disponibles. Puede realizar la implementación en el [inquilino](deploy-to-tenant.md), el [grupo de administración](deploy-to-management-group.md), la [suscripción](deploy-to-subscription.md) o el [grupo de recursos](deploy-powershell.md). En el nivel de implementación principal, no se pueden subir niveles en la jerarquía. Por ejemplo, si implementa en una suscripción, no puede subir un nivel para implementar recursos en un grupo de administración. Sin embargo, puede implementar en el grupo de administración y bajar dos niveles para implementar en una suscripción o un grupo de recursos.

Para cada ámbito, el usuario que implementa la plantilla debe tener permisos para crear recursos.

## <a name="cross-resource-groups"></a>Grupos de recursos cruzados

Para establecer como destino un grupo de recursos diferente al de la plantilla primaria, use una [plantilla anidada o vinculada](linked-templates.md). En el tipo de recurso de implementación, especifique los valores para el identificador de suscripción y el grupo de recursos en que quiere realizar la implementación de la plantilla anidada. Los grupos de recursos pueden existir en suscripciones distintas.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/crosssubscription.json" range="38-43" highlight="5-6":::

Si no se especifica el grupo de recursos o el identificador de la suscripción, se utilizan los de la plantilla primaria. Todos los grupos de recursos deben existir antes de que se ejecute la implementación.

> [!NOTE]
> Puede implementar hasta **800 grupos de recursos** en una única implementación. Normalmente, esta limitación significa que puede implementar en un grupo de recursos especificado para la plantilla primaria y hasta en 799 grupos de recursos en implementaciones anidadas o vinculadas. Sin embargo, si la plantilla principal contiene solo plantillas anidadas o vinculadas y no implementa por sí misma ningún recurso, puede incluir hasta 800 grupos de recursos en las implementaciones anidadas o vinculadas.

En el ejemplo siguiente se implementan dos cuentas de almacenamiento. La primera cuenta de almacenamiento se implementa en el grupo de recursos especificado en la operación de implementación. La segunda cuenta de almacenamiento se implementa en el grupo de recursos especificado en los parámetros `secondResourceGroup` y `secondSubscriptionID`:

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/crosssubscription.json":::

Si establece `resourceGroup` en el nombre de un grupo de recursos que no existe, la implementación generará un error.

Para probar la plantilla anterior y ver los resultados, use PowerShell o la CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para implementar dos cuentas de almacenamiento en dos grupos de recursos de la **misma suscripción**, use:

```azurepowershell-interactive
$firstRG = "primarygroup"
$secondRG = "secondarygroup"

New-AzResourceGroup -Name $firstRG -Location southcentralus
New-AzResourceGroup -Name $secondRG -Location eastus

New-AzResourceGroupDeployment `
  -ResourceGroupName $firstRG `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json `
  -storagePrefix storage `
  -secondResourceGroup $secondRG `
  -secondStorageLocation eastus
```

Para implementar dos cuentas de almacenamiento en **dos suscripciones**, use:

```azurepowershell-interactive
$firstRG = "primarygroup"
$secondRG = "secondarygroup"

$firstSub = "<first-subscription-id>"
$secondSub = "<second-subscription-id>"

Select-AzSubscription -Subscription $secondSub
New-AzResourceGroup -Name $secondRG -Location eastus

Select-AzSubscription -Subscription $firstSub
New-AzResourceGroup -Name $firstRG -Location southcentralus

New-AzResourceGroupDeployment `
  -ResourceGroupName $firstRG `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json `
  -storagePrefix storage `
  -secondResourceGroup $secondRG `
  -secondStorageLocation eastus `
  -secondSubscriptionID $secondSub
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para implementar dos cuentas de almacenamiento en dos grupos de recursos de la **misma suscripción**, use:

```azurecli-interactive
firstRG="primarygroup"
secondRG="secondarygroup"

az group create --name $firstRG --location southcentralus
az group create --name $secondRG --location eastus
az deployment group create \
  --name ExampleDeployment \
  --resource-group $firstRG \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json \
  --parameters storagePrefix=tfstorage secondResourceGroup=$secondRG secondStorageLocation=eastus
```

Para implementar dos cuentas de almacenamiento en **dos suscripciones**, use:

```azurecli-interactive
firstRG="primarygroup"
secondRG="secondarygroup"

firstSub="<first-subscription-id>"
secondSub="<second-subscription-id>"

az account set --subscription $secondSub
az group create --name $secondRG --location eastus

az account set --subscription $firstSub
az group create --name $firstRG --location southcentralus

az deployment group create \
  --name ExampleDeployment \
  --resource-group $firstRG \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json \
  --parameters storagePrefix=storage secondResourceGroup=$secondRG secondStorageLocation=eastus secondSubscriptionID=$secondSub
```

---

## <a name="cross-subscription-management-group-and-tenant"></a>Suscripciones, grupos de administración e inquilinos cruzados

Al especificar ámbitos diferentes para las implementaciones de nivel de suscripción, grupo de administración e inquilino, se usan implementaciones anidadas como en el ejemplo de grupos de recursos. Las propiedades que se usan para especificar el ámbito pueden diferir. Estos escenarios se describen en los artículos sobre los niveles de implementaciones. Para más información, consulte:

* [Creación de grupos de recursos y otros recursos en el nivel de suscripción](deploy-to-subscription.md)
* [Creación de recursos a nivel de grupo de administración](deploy-to-management-group.md)
* [Creación de recursos en el nivel de inquilino](deploy-to-tenant.md)

## <a name="how-functions-resolve-in-scopes"></a>Cómo se resuelven las funciones en ámbitos

Al implementar en más de un ámbito, las funciones [resourceGroup()](template-functions-resource.md#resourcegroup) y [subscription()](template-functions-resource.md#subscription) se resuelven de manera diferente en función de cómo se especifique la plantilla. Cuando se vincula a una plantilla externa, las funciones siempre se resuelven en el ámbito de esa plantilla. Al anidar una plantilla dentro de una plantilla primaria, use la propiedad `expressionEvaluationOptions` para especificar si las funciones se resuelven en el grupo de recursos y en la suscripción de la plantilla primaria o en la plantilla anidada. Establezca la propiedad en `inner` para resolver el ámbito de la plantilla anidada. Establezca la propiedad en `outer` para resolver el ámbito de la plantilla primaria.

En la tabla siguiente se muestra si las funciones se resuelven en el grupo de recursos y la suscripción primarios o insertados.

| Tipo de plantilla | Ámbito | Solución |
| ------------- | ----- | ---------- |
| anidada        | exterior (valor predeterminado) | Grupo de recursos primario |
| anidada        | interna | Grupo de subrecursos |
| Vinculado        | N/D   | Grupo de subrecursos |

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/crossresourcegroupproperties.json) se muestra:

* plantilla anidada con ámbito predeterminado (exterior)
* plantilla anidada con ámbito interno
* plantilla vinculada

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/crossresourcegroupproperties.json":::

Para probar la plantilla anterior y ver los resultados, use PowerShell o la CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name parentGroup -Location southcentralus
New-AzResourceGroup -Name inlineGroup -Location southcentralus
New-AzResourceGroup -Name linkedGroup -Location southcentralus

New-AzResourceGroupDeployment `
  -ResourceGroupName parentGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crossresourcegroupproperties.json
```

El resultado del ejemplo anterior es:

```output
 Name             Type                       Value
 ===============  =========================  ==========
 parentRG         String                     Parent resource group is parentGroup
 defaultScopeRG   String                     Default scope resource group is parentGroup
 innerScopeRG     String                     Inner scope resource group is inlineGroup
 linkedRG         String                     Linked resource group is linkedgroup
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az group create --name parentGroup --location southcentralus
az group create --name inlineGroup --location southcentralus
az group create --name linkedGroup --location southcentralus

az deployment group create \
  --name ExampleDeployment \
  --resource-group parentGroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crossresourcegroupproperties.json
```

El resultado del ejemplo anterior es:

```output
"outputs": {
  "defaultScopeRG": {
    "type": "String",
    "value": "Default scope resource group is parentGroup"
  },
  "innerScopeRG": {
    "type": "String",
    "value": "Inner scope resource group is inlineGroup"
  },
  "linkedRG": {
    "type": "String",
    "value": "Linked resource group is linkedGroup"
  },
  "parentRG": {
    "type": "String",
    "value": "Parent resource group is parentGroup"
  }
},
```

---



## <a name="next-steps"></a>Pasos siguientes

* Para entender cómo definir parámetros en la plantilla, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](template-syntax.md).
* Para obtener sugerencias para resolver los errores de implementación más comunes, consulte [Solución de errores comunes de implementación de Azure con Azure Resource Manager](common-deployment-errors.md).
* Para más información sobre la implementación de una plantilla que requiere un token de SAS, vea [Implementación de una plantilla privada con el token de SAS](secure-template-with-sas-token.md).
