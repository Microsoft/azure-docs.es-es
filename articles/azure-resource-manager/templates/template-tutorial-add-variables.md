---
title: 'Tutorial: Adición de una variable a una plantilla'
description: Agregue variables a la plantilla de Azure Resource Manager para simplificar la sintaxis.
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9607ddd4a44af6a515080d4fd6f0c475268470b9
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497501"
---
# <a name="tutorial-add-variables-to-your-arm-template"></a>Tutorial: Incorporación de variables a la plantilla de Resource Manager

En este tutorial, aprenderá a agregar una variable a la plantilla de Resource Manager. Las variables simplifican las plantillas al permitirle escribir una expresión una vez y reutilizarla en toda la plantilla. Este tutorial se realiza en **7 minutos**.

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre las funciones](template-tutorial-add-functions.md).

Debe tener Visual Studio Code con la extensión Herramientas de Resource Manager y Azure PowerShell o la CLI de Azure. Para más información, consulte las [herramientas de plantilla](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Revisión de la plantilla

Al final del tutorial anterior, la plantilla tenía el siguiente código JSON:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-location/azuredeploy.json":::

El parámetro del nombre de la cuenta de almacenamiento es difícil de usar porque tiene que proporcionar un nombre único. Si ha realizado los tutoriales anteriores de esta serie, probablemente esté cansado de pensar en un nombre único. Para solucionar este problema, agregue una variable que construya un nombre único para la cuenta de almacenamiento.

## <a name="use-variable"></a>Uso de la variable

En el ejemplo siguiente se resaltan los cambios para agregar una variable a la plantilla que crea un nombre de cuenta de almacenamiento único. Copie el archivo completo y reemplace la plantilla por su contenido.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-variable/azuredeploy.json" range="1-47" highlight="5-9,29-31,36":::

Observe que se incluye una variable llamada **uniqueStorageName**. Esta variable usa cuatro funciones para construir un valor de cadena.

Ya está familiarizado con la función [parameters](template-functions-deployment.md#parameters), por lo que no la examinaremos.

También está familiarizado con la función [resourceGroup](template-functions-resource.md#resourcegroup). En este caso, obtendrá la propiedad **id** en lugar de la propiedad **location**, tal como se muestra en el tutorial anterior. La propiedad **id** devuelve el identificador completo del grupo de recursos, incluido el identificador de la suscripción y el nombre del grupo de recursos.

La función [uniqueString](template-functions-string.md#uniquestring) crea un valor hash de 13 caracteres. El valor devuelto viene determinado por los parámetros que se pasan. En este tutorial, usará el identificador del grupo de recursos como entrada del valor hash. Esto significa que podría implementar esta plantilla en distintos grupos de recursos y obtener un valor de cadena único diferente. Sin embargo, se obtiene el mismo valor si la implementa en el mismo grupo de recursos.

La función [concat](template-functions-string.md#concat) toma valores y los combina. En el caso de esta variable, toma la cadena del parámetro y la cadena de la función uniqueString y las combina en una cadena.

El parámetro **storagePrefix** le permite pasar un prefijo que le ayuda a identificar las cuentas de almacenamiento. Puede crear su propia convención de nomenclatura que facilite la identificación de las cuentas de almacenamiento entre una larga lista de recursos después de la implementación.

Por último, observe que el nombre de almacenamiento ahora está establecido en la variable en vez de en un parámetro.

## <a name="deploy-template"></a>Implementar plantilla

Vamos a implementar la plantilla. Implementar esta plantilla es más fácil que las anteriores porque solo se proporciona el prefijo del nombre de almacenamiento.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](template-tutorial-create-first-template.md#create-resource-group). En el ejemplo se supone que ha establecido la variable **templateFile** en la ruta de acceso al archivo de plantilla, como se muestra en el [primer tutorial](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addnamevariable `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addnamevariable \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS
```

---

> [!NOTE]
> Si se ha producido un error en la implementación, use el modificador **debug** con el comando de implementación para mostrar los registros de depuración.  También puede usar el modificador **verbose** para mostrar los registros de depuración completos.

## <a name="verify-deployment"></a>Comprobación de la implementación

Para comprobar la implementación, explore el grupo de recursos desde Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú izquierdo, seleccione **Grupos de recursos**.
1. Seleccione el grupo de recursos en el que ha realizado la implementación.
1. Verá que se ha implementado un recurso de cuenta de almacenamiento. El nombre de la cuenta de almacenamiento es **store** más una cadena de caracteres aleatorios.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha agregado una variable que crea un nombre único para una cuenta de almacenamiento. En el siguiente tutorial, devolverá un valor de la cuenta de almacenamiento implementada.

> [!div class="nextstepaction"]
> [Adición de salidas](template-tutorial-add-outputs.md)
