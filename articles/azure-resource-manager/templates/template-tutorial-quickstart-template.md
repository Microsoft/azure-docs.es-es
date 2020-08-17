---
title: 'Tutorial: Uso de plantillas de inicio rápido'
description: Aprenda a usar las plantillas de inicio rápido de Azure para completar el desarrollo de plantillas.
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: 90d5c2688d20b62a4a5fda78eb67ed01777bc28c
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496889"
---
# <a name="tutorial-use-azure-quickstart-templates"></a>Tutorial: Uso de las plantillas de inicio rápido de Azure

[Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/) es un repositorio de plantillas aportadas por la comunidad. Puede usar las plantillas de ejemplo en el desarrollo de plantillas. En este tutorial, encontrará una definición de recursos de un sitio web y la agregará a su propia plantilla. Su tiempo de realización es de unos **12 minutos**.

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre plantillas exportadas](template-tutorial-export-template.md).

Debe tener Visual Studio Code con la extensión Herramientas de Resource Manager y Azure PowerShell o la CLI de Azure. Para más información, consulte las [herramientas de plantilla](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Revisión de la plantilla

Al final del tutorial anterior, la plantilla tenía el siguiente código JSON:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/export-template/azuredeploy.json":::

Esta plantilla funciona para implementar cuentas de almacenamiento y planes de App Service, pero puede que desee agregarle un sitio web. Puede usar plantillas predefinidas para averiguar rápidamente el código JSON necesario para implementar un recurso.

## <a name="find-template"></a>Búsqueda de la plantilla

1. Abra [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/).
1. En **Buscar**, escriba **deploy linux web app** (implementar aplicación web Linux).
1. Seleccione la que tenga el título **Deploy a basic Linux web app** (Implementación de una aplicación web Linux básica). Si tiene problemas para encontrarla, este es el [vínculo directo](https://azure.microsoft.com/resources/templates/101-webapp-basic-linux/).
1. Seleccione **Explorar en GitHub**.
1. Seleccione **azuredeploy.json**.
1. Revise la plantilla. En particular, busque el recurso `Microsoft.Web/sites`.

    ![Sitio web de inicio rápido de la plantilla de Resource Manager](./media/template-tutorial-quickstart-template/resource-manager-template-quickstart-template-web-site.png)

## <a name="revise-existing-template"></a>Revisión de la plantilla existente

Combine la plantilla de inicio rápido con la plantilla existente:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/quickstart-template/azuredeploy.json" range="1-108" highlight="32-45,49,85-100":::

El nombre de la aplicación web debe ser único en Azure. Para evitar tener nombres duplicados, la variable **webAppPortalName** se ha actualizado de **"webAppPortalName": "[concat(parameters('webAppName'), '-webapp')]"** a **"webAppPortalName": "[concat(parameters('webAppName'), uniqueString(resourceGroup().id))]"** .

Agregue una coma al final de la definición de `Microsoft.Web/serverfarms` para separar la definición de recursos de la definición de `Microsoft.Web/sites`.

Hay un par de características importantes que se deben tener en cuenta en este nuevo recurso.

Observará que tiene un elemento llamado **dependsOn** que se establece en el plan de App Service. Esta configuración es necesaria porque el plan de App Service debe existir antes de que se cree la aplicación web. El elemento **dependsOn** indica a Resource Manager cómo ordenar los recursos para la implementación.

La propiedad **serverFarmId** usa la función [resourceId](template-functions-resource.md#resourceid). Esta función obtiene el identificador único de un recurso. En este caso, obtiene el identificador único del plan de App Service. La aplicación web está asociada a un plan de App Service específico.

## <a name="deploy-template"></a>Implementar plantilla

Use la CLI de Azure o Azure PowerShell para implementar una plantilla.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](template-tutorial-create-first-template.md#create-resource-group). En el ejemplo se supone que ha establecido la variable **templateFile** en la ruta de acceso al archivo de plantilla, como se muestra en el [primer tutorial](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addwebapp `
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
  --name addwebapp \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS webAppName=demoapp
```

---

> [!NOTE]
> Si se ha producido un error en la implementación, use el modificador **debug** con el comando de implementación para mostrar los registros de depuración.  También puede usar el modificador **verbose** para mostrar los registros de depuración completos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha aprendido a usar una plantilla de inicio rápido para el desarrollo de plantillas. En el siguiente tutorial, agregará etiquetas a los recursos.

> [!div class="nextstepaction"]
> [Adición de etiquetas](template-tutorial-add-tags.md)
