---
title: 'Tutorial: Adición de funciones de plantilla'
description: Agregue funciones de plantilla a una plantilla de Azure Resource Manager para construir valores.
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: fe6185f0e7d6ee570c1491f3b21b6aebe4f090ae
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497569"
---
# <a name="tutorial-add-template-functions-to-your-arm-template"></a>Tutorial: Adición de funciones de plantilla a una plantilla de Resource Manager

En este tutorial, aprenderá a agregar [funciones de plantilla](template-functions.md) a la plantilla de Resource Manager. Las funciones se usan para construir valores de forma dinámica. Además de estas funciones de plantilla proporcionadas por el sistema, también puede crear [funciones definidas por el usuario](./template-user-defined-functions.md). Este tutorial se realiza en **7 minutos**.

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre parámetros](template-tutorial-add-parameters.md).

Debe tener Visual Studio Code con la extensión Herramientas de Resource Manager y Azure PowerShell o la CLI de Azure. Para más información, consulte las [herramientas de plantilla](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Revisión de la plantilla

Al final del tutorial anterior, la plantilla tenía el siguiente código JSON:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-sku/azuredeploy.json":::

La ubicación de la cuenta de almacenamiento está codificada de forma rígida en **Este de EE. UU.** . Sin embargo, puede que tenga que implementar la cuenta de almacenamiento en otras regiones. Una vez más, se enfrenta al problema de que la plantilla carece de flexibilidad. Podría agregar un parámetro para la ubicación, pero sería aún mejor si su valor predeterminado fuera algo más que un simple valor codificado de forma rígida.

## <a name="use-function"></a>Uso de una función

Si ha completado el tutorial anterior de esta serie, ya ha usado una función. Cuando agregó **"[parameters('storageName')]"** , usó la función [parameters](template-functions-deployment.md#parameters). Los corchetes indican que la sintaxis incluida entre ellos es una expresión [template](template-expressions.md). Resource Manager resuelve la sintaxis en lugar de tratarla como un valor literal.

Las funciones agregan flexibilidad a la plantilla mediante la obtención dinámica de valores durante la implementación. En este tutorial, usará una función para obtener la ubicación del grupo de recursos que está usando para la implementación.

En el ejemplo siguiente se resaltan los cambios para agregar un parámetro denominado **location**.  El valor predeterminado del parámetro llama a la función [resourceGroup](template-functions-resource.md#resourcegroup). Esta función devuelve un objeto con información sobre el grupo de recursos que se utiliza para la implementación. Una de las propiedades del objeto es una propiedad de ubicación. Cuando se usa el valor predeterminado, la cuenta de almacenamiento tiene la misma ubicación que el grupo de recursos. No es necesario que los recursos incluidos en un grupo de recursos compartan la misma ubicación. También puede proporcionar una ubicación diferente cuando sea necesario.

Copie el archivo completo y reemplace la plantilla por su contenido.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-location/azuredeploy.json" range="1-44" highlight="24-27,34":::

## <a name="deploy-template"></a>Implementar plantilla

En los tutoriales anteriores, creó una cuenta de almacenamiento en Este de EE. UU., pero el grupo de recursos se creó en Centro de EE. UU. En este tutorial, la cuenta de almacenamiento se crea en la misma región que el grupo de recursos. Use el valor predeterminado para la ubicación, con lo que no será necesario proporcionar el valor de ese parámetro. Debe proporcionar un nuevo nombre para la cuenta de almacenamiento porque está creando una cuenta de almacenamiento en una ubicación diferente. Por ejemplo, use **store2** como prefijo en lugar de **store1**.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](template-tutorial-create-first-template.md#create-resource-group). En el ejemplo se supone que ha establecido la variable **templateFile** en la ruta de acceso al archivo de plantilla, como se muestra en el [primer tutorial](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addlocationparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storageName "{new-unique-name}"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addlocationparameter \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storageName={new-unique-name}
```

---

> [!NOTE]
> Si se ha producido un error en la implementación, use el modificador **debug** con el comando de implementación para mostrar los registros de depuración.  También puede usar el modificador **verbose** para mostrar los registros de depuración completos.

## <a name="verify-deployment"></a>Comprobación de la implementación

Para comprobar la implementación, explore el grupo de recursos desde Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú izquierdo, seleccione **Grupos de recursos**.
1. Seleccione el grupo de recursos en el que ha realizado la implementación.
1. Verá que un recurso de la cuenta de almacenamiento se ha implementado y tiene la misma ubicación que el grupo de recursos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha usado una función al definir el valor predeterminado de un parámetro. En esta serie de tutoriales, seguirá usando funciones. Al final de la serie, agregará funciones a cada sección de la plantilla.

> [!div class="nextstepaction"]
> [Adición de variables](template-tutorial-add-variables.md)
