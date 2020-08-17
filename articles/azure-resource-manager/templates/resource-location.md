---
title: Ubicación del recurso de plantilla
description: En este artículo se describe cómo establecer la ubicación de los recursos de una plantilla de Azure Resource Manager.
ms.topic: conceptual
ms.date: 09/04/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: e60fa9727ef899c3192c751614736cd1dda5b382
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87504201"
---
# <a name="set-resource-location-in-arm-template"></a>Establecimiento de la ubicación del recurso en la plantilla de Resource Manager

Al implementar una plantilla de Azure Resource Manager (ARM), debe proporcionar una ubicación para cada recurso. No es necesario que la ubicación sea la misma que la ubicación del grupo de recursos.

## <a name="get-available-locations"></a>Obtención de las ubicaciones disponibles

Se admiten diferentes tipos de recursos en diferentes ubicaciones. Para obtener las ubicaciones admitidas para un tipo de recurso, use Azure PowerShell o la CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
((Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes `
  | Where-Object ResourceTypeName -eq batchAccounts).Locations
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az provider show \
  --namespace Microsoft.Batch \
  --query "resourceTypes[?resourceType=='batchAccounts'].locations | [0]" \
  --out table
```

---

## <a name="use-location-parameter"></a>Uso del parámetro de ubicación

Para permitir la flexibilidad al implementar la plantilla, use un parámetro para especificar la ubicación de los recursos. Establezca el valor predeterminado del parámetro en `resourceGroup().location`.

En el ejemplo siguiente se muestra una cuenta de almacenamiento que está implementada en una ubicación especificada como parámetro:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS",
        "Premium_LRS"
      ],
      "metadata": {
        "description": "Storage Account type"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "variables": {
    "storageAccountName": "[concat('storage', uniquestring(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2018-07-01",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      },
      "kind": "StorageV2",
      "properties": {}
    }
  ],
  "outputs": {
    "storageAccountName": {
      "type": "string",
      "value": "[variables('storageAccountName')]"
    }
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener la lista completa de las funciones de plantilla, consulte [Funciones de la plantilla del Administrador de recursos de Azure](template-functions.md).
* Para más información sobre los archivos de plantilla, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de ARM](template-syntax.md).
