---
title: Implementación de una especificación de plantilla como una plantilla vinculada
description: Obtenga información sobre cómo implementar una especificación de plantilla existente en una implementación vinculada.
ms.topic: conceptual
ms.date: 07/20/2020
ms.openlocfilehash: 5d4824ea432d804418fda2cdc90d49154d496722
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095120"
---
# <a name="tutorial-deploy-a-template-spec-as-a-linked-template-preview"></a>Tutorial: Implementación de una especificación de plantilla como una plantilla vinculada (versión preliminar)

Obtenga información sobre cómo implementar una [especificación de plantilla](template-specs.md) existente mediante una [implementación vinculada](linked-templates.md#linked-template). Las especificaciones de plantilla se usan para compartir plantillas de ARM con otros usuarios de la organización. Después de crear una especificación de plantilla, puede implementarla mediante Azure PowerShell. También puede implementar la especificación de plantilla como parte de la solución mediante una plantilla vinculada.

## <a name="prerequisites"></a>Requisitos previos

Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

> [!NOTE]
> Las especificaciones de plantilla se encuentran actualmente en versión preliminar. Para usarlo, debe [registrarse en la versión preliminar](https://aka.ms/templateSpecOnboarding).

## <a name="create-a-template-spec"></a>Creación de una especificación de plantilla

Consulte [Inicio rápido: Creación e implementación de la especificación de plantilla](quickstart-create-template-specs.md) para crear una especificación de plantilla para implementar una cuenta de almacenamiento. Necesitará el nombre del grupo de recursos de la especificación de plantilla, el nombre de la especificación de plantilla y la versión de la especificación de plantilla en la sección siguiente.

## <a name="create-the-main-template"></a>Creación de la plantilla principal

Para crear una especificación de plantilla en una plantilla de ARM, agregue un [recurso de implementaciones](/azure/templates/microsoft.resources/deployments) a la plantilla principal. En la propiedad `templateLink`, especifique el identificador de recurso de una especificación de plantilla. Cree una plantilla con el siguiente código JSON denominado **azuredeploy.json**. En este tutorial se da por supuesto que se ha guardado en la ruta de acceso **c:\Templates\deployTS\azuredeploy.json**, pero puede usar cualquier ruta de acceso.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "tsResourceGroup":{
      "type": "string",
      "metadata": {
        "Description": "Specifies the resource group name of the template spec."
      }
    },
    "tsName": {
      "type": "string",
      "metadata": {
        "Description": "Specifies the name of the template spec."
      }
    },
    "tsVersion": {
      "type": "string",
      "defaultValue": "1.0.0.0",
      "metadata": {
        "Description": "Specifies the version the template spec."
      }
    },
    "storageAccountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "metadata": {
        "Description": "Specifies the storage account type required by the template spec."
      }
    }
  },
  "variables": {
    "appServicePlanName": "[concat('plan', uniquestring(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2016-09-01",
      "name": "[variables('appServicePlanName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "B1",
        "tier": "Basic",
        "size": "B1",
        "family": "B",
        "capacity": 1
      },
      "kind": "linux",
      "properties": {
        "perSiteScaling": false,
        "reserved": true,
        "targetWorkerCount": 0,
        "targetWorkerSizeId": 0
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      "name": "createStorage",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "id": "[resourceId(parameters('tsResourceGroup'), 'Microsoft.Resources/templateSpecs/versions', parameters('tsName'), parameters('tsVersion'))]"
        },
        "parameters": {
          "storageAccountType": {
            "value": "[parameters('storageAccountType')]"
          }
        }
      }
    }
  ],
  "outputs": {
    "templateSpecId": {
      "type": "string",
      "value": "[resourceId(parameters('tsResourceGroup'), 'Microsoft.Resources/templateSpecs/versions', parameters('tsName'), parameters('tsVersion'))]"
    }
  }
}
```

El identificador de especificación de plantilla se genera mediante la función [`resourceID()`](template-functions-resource.md#resourceid). El argumento del grupo de recursos de la función resourceID() es opcional si templateSpec está en el mismo grupo de recursos de la implementación actual.  También puede pasar directamente el identificador de recurso como un parámetro. Para obtener el identificador, use:

```azurepowershell-interactive
$id = (Get-AzTemplateSpec -ResourceGroupName $resourceGroupName -Name $templateSpecName -Version $templateSpecVersion).Version.Id
```

La sintaxis para pasar parámetros a la especificación de plantilla es la siguiente:

```json
"parameters": {
  "storageAccountType": {
    "value": "[parameters('storageAccountType')]"
  }
}
```

> [!NOTE]
> El valor de apiVersion de `Microsoft.Resources/deployments` debe ser 2020-06-01 o posterior.

## <a name="deploy-the-template"></a>Implementación de la plantilla

Al implementar la plantilla vinculada, se implementa tanto la aplicación web como la cuenta de almacenamiento. La implementación es la misma que la de otras plantillas de ARM.

```azurepowershell
New-AzResourceGroup `
  -Name webRG `
  -Location westus2

New-AzResourceGroupDeployment `
  -ResourceGroupName webRG `
  -TemplateFile "c:\Templates\deployTS\azuredeploy.json"
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo crear una especificación de plantilla que incluya plantillas vinculadas, consulte [Creación de una especificación de plantilla con plantillas vinculadas](template-specs-create-linked.md).
