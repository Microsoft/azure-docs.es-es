---
title: Creación de archivo de parámetros
description: Creación de un archivo de parámetros para pasar valores durante la implementación de una plantilla de Azure Resource Manager
ms.topic: conceptual
ms.date: 09/01/2020
ms.openlocfilehash: 2b6d942b21594fa608127bb8f403e72295671005
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89276667"
---
# <a name="create-resource-manager-parameter-file"></a>Creación de un archivo de parámetros de Resource Manager

En lugar de pasar parámetros como valores en línea en el script, quizá le resulte más fácil usar un archivo JSON que contiene los valores de parámetro. En este artículo se muestra cómo crear el archivo de parámetros.

## <a name="parameter-file"></a>Archivo de parámetros

El archivo de parámetros tiene el siguiente formato:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "<first-parameter-name>": {
      "value": "<first-value>"
    },
    "<second-parameter-name>": {
      "value": "<second-value>"
    }
  }
}
```

Tenga en cuenta que los valores de parámetro se almacenan como texto sin formato en el archivo de parámetros. Este enfoque funciona para los valores que no son confidenciales, como la especificación de la SKU de un recurso. No funciona con valores confidenciales, como contraseñas. Si necesita pasar un valor confidencial como parámetro, almacene el valor en una instancia de Key Vault y haga referencia a dicha instancia en el archivo de parámetros. El valor confidencial se recupera de forma segura durante la implementación.

El archivo de parámetros siguiente incluye un valor de texto sin formato y un valor que se almacena en una instancia de Key Vault.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "<first-parameter-name>": {
      "value": "<first-value>"
    },
    "<second-parameter-name>": {
      "reference": {
        "keyVault": {
          "id": "<resource-id-key-vault>"
        },
        "secretName": "<secret-name>"
      }
    }
  }
}
```

Para más información sobre el uso de valores de una instancia de Key Vault, consulte [Uso de Azure Key Vault para pasar el valor de parámetro seguro durante la implementación](key-vault-parameter.md).

## <a name="define-parameter-values"></a>Definición de los valores de parámetro

Para averiguar cómo definir los valores de parámetro, abra la plantilla que va a implementar. Examine la sección de parámetros de la plantilla. En el ejemplo siguiente se muestran los parámetros de una plantilla.

```json
"parameters": {
  "storagePrefix": {
    "type": "string",
    "maxLength": 11
  },
  "storageAccountType": {
    "type": "string",
    "defaultValue": "Standard_LRS",
    "allowedValues": [
    "Standard_LRS",
    "Standard_GRS",
    "Standard_ZRS",
    "Premium_LRS"
    ]
  }
}
```

El primer detalle que se debe tener en cuenta es el nombre de cada parámetro. Los valores del archivo de parámetros deben coincidir con los nombres.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
    },
    "storageAccountType": {
    }
  }
}
```

Tenga en cuenta el tipo del parámetro. Los valores del archivo de parámetros deben tener los mismos tipos. Para esta plantilla, puede proporcionar ambos parámetros como cadenas.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": ""
    },
    "storageAccountType": {
      "value": ""
    }
  }
}
```

A continuación, busque un valor predeterminado. Si un parámetro tiene un valor predeterminado, puede proporcionar un valor, pero no es necesario hacerlo.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": "" // This value must be provided.
    },
    "storageAccountType": {
      "value": "" // This value is optional. Template will use default value if not provided.
    }
  }
}
```

Por último, examine los valores permitidos y las restricciones, como la longitud máxima. Le indican el intervalo de valores que puede proporcionar para el parámetro.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": "storage"
    },
    "storageAccountType": {
      "value": "Standard_ZRS"
    }
  }
}
```

El archivo de parámetros solo puede contener valores para los parámetros que se definan en la plantilla. Si el archivo de parámetros contiene parámetros adicionales que no coinciden con los de la plantilla, recibirá un error.

## <a name="parameter-type-formats"></a>Formatos de tipo de parámetro

En el ejemplo siguiente se muestran los formatos de distintos tipos de parámetros.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "exampleString": {
      "value": "test string"
    },
    "exampleInt": {
      "value": 4
    },
    "exampleBool": {
      "value": true
    },
    "exampleArray": {
      "value": [
        "value 1",
        "value 2"
      ]
    },
    "exampleObject": {
      "value": {
        "property1": "value1",
        "property2": "value2"
      }
    }
   }
}
```

## <a name="deploy-template-with-parameter-file"></a>Implementación de la plantilla con el archivo de parámetros

Para pasar un archivo de parámetros local mediante la CLI de Azure, utilice @ y el nombre del archivo de parámetros.

```azurecli
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters @storage.parameters.json
```

Para más información, consulte [Implementación de recursos con plantillas de ARM y la CLI de Azure](./deploy-cli.md#parameters).

Para pasar un archivo de parámetros local mediante Azure PowerShell, utilice el parámetro `TemplateParameterFile`.

```azurepowershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -TemplateParameterFile c:\MyTemplates\storage.parameters.json
```

Para más información, consulte [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](./deploy-powershell.md#pass-parameter-values).

> [!NOTE]
> No es posible usar un archivo de parámetros con la hoja de plantilla personalizada en el portal.

## <a name="file-name"></a>Nombre de archivo

La convención general para asignar un nombre al archivo de parámetros consisten en agregar **.parameters** al nombre de la plantilla. Por ejemplo, si la plantilla se denomina **azuredeploy.json**, el archivo de parámetros se denomina **azuredeploy.parameters.json**. Esta convención de nomenclatura le ayuda a ver la conexión entre la plantilla y los parámetros.

Para realizar la implementación en entornos diferentes, cree más de un archivo de parámetros. Cuando asigne un nombre al archivo de parámetros, agregue una forma de identificar su uso. Por ejemplo, use **azuredeploy.parameters-dev.json** y **azuredeploy.parameters-prod.json**.

## <a name="parameter-precedence"></a>Prioridad de parámetros

Puede usar parámetros en línea y un archivo de parámetros local en la misma operación de implementación. Por ejemplo, puede especificar algunos valores en el archivo de parámetros local y agregar otros valores en línea durante la implementación. Si proporciona valores para un parámetro en el archivo de parámetros local y en línea, el valor en línea tiene prioridad.

Es posible usar un archivo de parámetros externo proporcionando el URI al archivo. Cuando se utiliza un archivo de parámetros externo, no se pueden pasar otros valores, tanto si se hace en línea como desde un archivo local. Se omiten todos los parámetros insertados. Proporcione todos los valores de parámetro en el archivo externo.

## <a name="parameter-name-conflicts"></a>Conflictos de nombres de parámetro

Si la plantilla incluye un parámetro con el mismo nombre que uno de los parámetros del comando de PowerShell, PowerShell presenta el parámetro de la plantilla con el postfijo **FromTemplate**. Por ejemplo, un parámetro denominado **ResourceGroupName** de su plantilla entra en conflicto con el parámetro **ResourceGroupName** del cmdlet [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) nuevo. Se le pedirá que proporcione un valor para **ResourceGroupNameFromTemplate**. Puede evitar esta confusión mediante el uso de nombres de parámetros que no se usan para los comandos de implementación.


## <a name="next-steps"></a>Pasos siguientes

- Para saber cómo definir parámetros en la plantilla, consulte [Parámetros en plantillas de Azure Resource Manager](template-parameters.md).
- Para más información sobre el uso de valores de una instancia de Key Vault, consulte [Uso de Azure Key Vault para pasar el valor de parámetro seguro durante la implementación](key-vault-parameter.md).
- Para más información sobre los parámetros, consulte [Parámetros en plantillas de Azure Resource Manager](template-parameters.md).
