---
title: 'Tutorial: Adición de etiquetas a los recursos de la plantilla'
description: Agregue etiquetas a los recursos que implemente en la plantilla de Azure Resource Manager. Las etiquetas permiten organizar los recursos de manera lógica.
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: ce25047ae93a332b62b7e785b23da5794031a98b
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497518"
---
# <a name="tutorial-add-tags-in-your-arm-template"></a>Tutorial: Incorporación de etiquetas en la plantilla de Resource Manager

En este tutorial, aprenderá a agregar etiquetas a los recursos de la plantilla de Resource Manager. Las [etiquetas](../management/tag-resources.md) le ayudan a organizar los recursos de forma lógica. Los valores de la etiqueta se muestran en los informes de costos. Este tutorial se realiza en **8 minutos**.

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre las plantillas de inicio rápido](template-tutorial-quickstart-template.md).

Debe tener Visual Studio Code con la extensión Herramientas de Resource Manager y Azure PowerShell o la CLI de Azure. Para más información, consulte las [herramientas de plantilla](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Revisión de la plantilla

La plantilla anterior implementó una cuenta de almacenamiento, un plan de App Service y una aplicación web.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/quickstart-template/azuredeploy.json":::

Después de implementar estos recursos, es posible que tenga que realizar un seguimiento de los costos y encontrar los recursos que pertenecen a una categoría. Para ayudar a resolver estos problemas, puede agregar etiquetas.

## <a name="add-tags"></a>Incorporación de etiquetas

Etiquete los recursos para agregar valores que le ayuden a identificar su uso. Por ejemplo, puede agregar etiquetas que muestren el entorno y el proyecto. Podría agregar etiquetas que identificaran un centro de costos o el equipo que posee el recurso. Agregue valores que tengan sentido para su organización.

En el ejemplo siguiente se resaltan los cambios en la plantilla. Copie el archivo completo y reemplace la plantilla por su contenido.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.json" range="1-118" highlight="46-52,64,78,102":::

## <a name="deploy-template"></a>Implementar plantilla

Es momento de implementar la plantilla y examinar los resultados.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](template-tutorial-create-first-template.md#create-resource-group). En el ejemplo se supone que ha establecido la variable **templateFile** en la ruta de acceso al archivo de plantilla, como se muestra en el [primer tutorial](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addtags `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS `
  -webAppName demoapp
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addtags \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS webAppName=demoapp
```

---

> [!NOTE]
> Si se ha producido un error en la implementación, use el modificador **debug** con el comando de implementación para mostrar los registros de depuración.  También puede usar el modificador **verbose** para mostrar los registros de depuración completos.

## <a name="verify-deployment"></a>Comprobación de la implementación

Para comprobar la implementación, explore el grupo de recursos desde Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú izquierdo, seleccione **Grupos de recursos**.
1. Seleccione el grupo de recursos en el que ha realizado la implementación.
1. Seleccione uno de los recursos, por ejemplo, el recurso de cuenta de almacenamiento. Verá que ahora tiene etiquetas.

   ![Mostrar etiquetas](./media/template-tutorial-add-tags/show-tags.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha agregado etiquetas a los recursos. En el siguiente tutorial, aprenderá a usar archivos de parámetros para simplificar el paso de valores a la plantilla.

> [!div class="nextstepaction"]
> [Uso de archivos de parámetros](template-tutorial-use-parameter-file.md)
