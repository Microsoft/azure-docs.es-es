---
title: 'Tutorial: Exportación de una plantilla desde Azure Portal'
description: Aprenda a usar una plantilla exportada para realizar el desarrollo mediante plantillas.
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4042ed29b143ab160883ca46ecb1cc17d2e0c761
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497161"
---
# <a name="tutorial-use-exported-template-from-the-azure-portal"></a>Tutorial: Uso de una plantilla desde Azure Portal

En esta serie de tutoriales, ha creado una plantilla para implementar una cuenta de almacenamiento de Azure. En los dos tutoriales siguientes, agregará un *plan de App Service* y un *sitio web*. En lugar de crear plantillas desde cero, aprenderá a exportarlas desde Azure Portal y cómo usar las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/) como ejemplo. Se puede personalizar el uso de esas plantillas. Este tutorial se centra en la exportación de plantillas y en la personalización del resultado de la plantilla. Se tarda en realizar unos **14 minutos**.

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre las salidas](template-tutorial-add-outputs.md).

Debe tener Visual Studio Code con la extensión Herramientas de Resource Manager y Azure PowerShell o la CLI de Azure. Para más información, consulte las [herramientas de plantilla](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Revisión de la plantilla

Al final del tutorial anterior, la plantilla tenía el siguiente código JSON:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-outputs/azuredeploy.json":::

Esta plantilla es adecuada para implementar cuentas de almacenamiento, pero puede agregarle más recursos. Puede exportar una plantilla desde un recurso existente para obtener rápidamente el código JSON de ese recurso.

## <a name="create-app-service-plan"></a>Creación de un plan de App Service

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Crear un recurso**.
1. En **Buscar en Marketplace**, escriba **Plan de App Service** y, luego, seleccione **Plan de App Service**.  No seleccione **Plan de App Service (clásico)** .
1. Seleccione **Crear**.
1. Especifique:

    - **Suscripción**: seleccione su suscripción de Azure.
    - **Grupo de recursos**: Seleccione **Crear nuevo** y, luego, especifique un nombre. Proporcione un nombre de grupo de recursos diferente al que ha usado en esta serie de tutoriales.
    - **Nombre**: escriba un nombre para el plan de App Service.
    - **Sistema operativo**: seleccione **Linux**.
    - **Región**: seleccione una ubicación de Azure. Por ejemplo, **Centro de EE. UU**.
    - **Plan de tarifa**: para ahorrar costos, cambie la SKU a **Básico B1** (en desarrollo y pruebas).

    ![Plantilla de Resource Manager: exportación de plantillas en el portal](./media/template-tutorial-export-template/resource-manager-template-export.png)
1. Seleccione **Revisar y crear**.
1. Seleccione **Crear**. La creación del recurso tarda algunos minutos.

## <a name="export-template"></a>Exportación de la plantilla

1. Haga clic en **Go to resource** (Ir al recurso).

    ![Ir al recurso](./media/template-tutorial-export-template/resource-manager-template-export-go-to-resource.png)

1. Seleccione **Exportar plantilla**.

    ![Plantilla de Resource Manager: exportación de plantillas](./media/template-tutorial-export-template/resource-manager-template-export-template.png)

   La característica de exportación de plantillas toma el estado actual de un recurso y genera una plantilla para implementarlo. Exportar una plantilla puede ser una forma útil de obtener rápidamente el código JSON necesario para implementar un recurso.

1. Copie la definición **Microsoft.Web/serverfarms** y la definición del parámetro en la plantilla.

    ![Plantilla de Resource Manager: plantillas exportadas con la exportación de plantillas](./media/template-tutorial-export-template/resource-manager-template-exported-template.png)

> [!IMPORTANT]
> Normalmente, la plantilla exportada es más detallada que cuando se crea una plantilla. Por ejemplo, el objeto SKU de la plantilla exportada tiene cinco propiedades. Esta plantilla funciona, pero podría usar simplemente la propiedad **name**. Puede comenzar con la plantilla exportada y, luego, modificarla para adaptarla a sus necesidades.

## <a name="revise-existing-template"></a>Revisión de la plantilla existente

La plantilla exportada le proporciona la mayor parte del código JSON que necesita, pero debe personalizarlo para su plantilla. Preste especial atención a las diferencias de parámetros y variables que existen entre la plantilla y la plantilla exportada. Obviamente, el proceso de exportación no conoce los parámetros y las variables que ya ha definido en la plantilla.

En el ejemplo siguiente se resaltan los elementos agregados a la plantilla. Contiene el código exportado y algunos cambios. En primer lugar, cambia el nombre del parámetro para que coincida con su convención de nomenclatura. En segundo lugar, usa el parámetro de ubicación para la ubicación del plan de App Service. En tercer lugar, quita el parámetro **name** del objeto **properties** porque este valor es redundante con la propiedad **name** en el nivel de recurso.

Copie el archivo completo y reemplace la plantilla por su contenido.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/export-template/azuredeploy.json" range="1-77" highlight="28-31,50-69":::

## <a name="deploy-template"></a>Implementar plantilla

Use la CLI de Azure o Azure PowerShell para implementar una plantilla.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](template-tutorial-create-first-template.md#create-resource-group). En el ejemplo se supone que ha establecido la variable **templateFile** en la ruta de acceso al archivo de plantilla, como se muestra en el [primer tutorial](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addappserviceplan `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addappserviceplan \
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
1. El grupo de recursos contiene una cuenta de almacenamiento y un plan de App Service.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha aprendido a exportar una plantilla desde Azure Portal y cómo usar la plantilla exportada para el desarrollo con plantillas. También puede usar las plantillas de inicio rápido de Azure para simplificar el desarrollo con plantillas.

> [!div class="nextstepaction"]
> [Uso de las plantillas de inicio rápido de Azure](template-tutorial-quickstart-template.md)
