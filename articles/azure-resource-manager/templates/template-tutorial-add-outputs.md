---
title: 'Tutorial: Adición de salidas a una plantilla'
description: Agregue salidas a la plantilla de Azure Resource Manager para simplificar la sintaxis.
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4c90f1ce4bc3f3f5ed91bee86bb1f6f9093876c7
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497790"
---
# <a name="tutorial-add-outputs-to-your-arm-template"></a>Tutorial: Incorporación de salidas a la plantilla de Resource Manager

En este tutorial, aprenderá a devolver un valor a partir de una plantilla de Resource Manager. Las salidas se usan cuando necesita un valor de un recurso implementado. Este tutorial se realiza en **7 minutos**.

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre variables](template-tutorial-add-variables.md).

Debe tener Visual Studio Code con la extensión Herramientas de Resource Manager y Azure PowerShell o la CLI de Azure. Para más información, consulte las [herramientas de plantilla](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Revisión de la plantilla

Al final del tutorial anterior, la plantilla tenía el siguiente código JSON:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-variable/azuredeploy.json":::

Este código implementa una cuenta de almacenamiento, pero no devuelve ninguna información sobre dicha cuenta. Es posible que necesite capturar propiedades de un nuevo recurso para que estén disponibles más adelante como referencia.

## <a name="add-outputs"></a>Adición de salidas

Puede usar salidas para devolver valores de la plantilla. Por ejemplo, puede resultar útil obtener los puntos de conexión de la nueva cuenta de almacenamiento.

En el ejemplo siguiente se resalta el cambio en la plantilla para agregar un valor de salida. Copie el archivo completo y reemplace la plantilla por su contenido.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-outputs/azuredeploy.json" range="1-53" highlight="47-52":::

Hay algunos elementos importantes que se deben tener en cuenta sobre el valor de salida que ha agregado.

El tipo de valor devuelto se establece en **object**, lo que significa que devuelve un objeto JSON.

Utiliza la función [reference](template-functions-resource.md#reference) para obtener el estado de tiempo de ejecución de la cuenta de almacenamiento. Para obtener el estado de tiempo de ejecución de un recurso, se pasa el nombre o el identificador del recurso. En este caso, usará la misma variable que utilizó para crear el nombre de la cuenta de almacenamiento.

Por último, devuelve la propiedad **primaryEndpoints** de la cuenta de almacenamiento.

## <a name="deploy-template"></a>Implementar plantilla

Ya está listo para implementar la plantilla y examinar el valor devuelto.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](template-tutorial-create-first-template.md#create-resource-group). En el ejemplo se supone que ha establecido la variable **templateFile** en la ruta de acceso al archivo de plantilla, como se muestra en el [primer tutorial](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addoutputs `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addoutputs \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS
```

---

En el resultado del comando de implementación, verá un objeto similar al siguiente ejemplo solo si el resultado está en formato JSON:

```json
{
    "dfs": "https://storeluktbfkpjjrkm.dfs.core.windows.net/",
    "web": "https://storeluktbfkpjjrkm.z19.web.core.windows.net/",
    "blob": "https://storeluktbfkpjjrkm.blob.core.windows.net/",
    "queue": "https://storeluktbfkpjjrkm.queue.core.windows.net/",
    "table": "https://storeluktbfkpjjrkm.table.core.windows.net/",
    "file": "https://storeluktbfkpjjrkm.file.core.windows.net/"
}
```

> [!NOTE]
> Si se ha producido un error en la implementación, use el modificador **debug** con el comando de implementación para mostrar los registros de depuración.  También puede usar el modificador **verbose** para mostrar los registros de depuración completos.

## <a name="review-your-work"></a>Revisión del trabajo

En los seis últimos tutoriales ha hecho muchas cosas. Dedique un momento a revisar lo que ha hecho. Ha creado una plantilla con parámetros fáciles de proporcionar. La plantilla se puede volver a usar en diferentes entornos porque se puede personalizar y crea dinámicamente valores necesarios. También devuelve información sobre la cuenta de almacenamiento que puede usar en el script.

Ahora, echemos un vistazo al grupo de recursos y al historial de implementaciones.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú izquierdo, seleccione **Grupos de recursos**.
1. Seleccione el grupo de recursos en el que ha realizado la implementación.
1. En función de los pasos que haya realizado, debe tener al menos una cuenta de almacenamiento en el grupo de recursos.
1. También debe tener varias implementaciones correctas enumeradas en el historial. Seleccione ese vínculo.

   ![Seleccionar implementaciones](./media/template-tutorial-add-outputs/select-deployments.png)

1. Verá todas las implementaciones del historial. Seleccione la implementación llamada **addoutputs**.

   ![Mostrar el historial de implementaciones](./media/template-tutorial-add-outputs/show-history.png)

1. Puede revisar las entradas.

   ![Mostrar entradas](./media/template-tutorial-add-outputs/show-inputs.png)

1. Puede revisar las salidas.

   ![Mostrar salidas](./media/template-tutorial-add-outputs/show-outputs.png)

1. Puede revisar la plantilla.

   ![Mostrar la plantilla](./media/template-tutorial-add-outputs/show-template.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha agregado un valor devuelto a la plantilla. En el siguiente tutorial, aprenderá a exportar una plantilla y a usar partes de la plantilla exportada en su plantilla.

> [!div class="nextstepaction"]
> [Uso de una plantilla exportada](template-tutorial-export-template.md)
