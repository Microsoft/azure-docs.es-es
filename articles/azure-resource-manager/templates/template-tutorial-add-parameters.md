---
title: 'Tutorial: Adición de parámetros a la plantilla'
description: Agregue parámetros a su plantilla de Azure Resource Manager para poder volver a utilizarla.
author: mumian
ms.date: 03/31/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: b1454106c4498f4519972633df8a871585d254f1
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497552"
---
# <a name="tutorial-add-parameters-to-your-arm-template"></a>Tutorial: Adición de parámetros a la plantilla de Resource Manager

En el [tutorial anterior](template-tutorial-add-resource.md), aprendió a agregar una cuenta de almacenamiento a la plantilla y a implementarla. En este tutorial, aprenderá a mejorar la plantilla de Resource Manager mediante la adición de parámetros. Este tutorial se realiza en unos **14 minutos**.

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre recursos](template-tutorial-add-resource.md).

Debe tener Visual Studio Code con la extensión Herramientas de Resource Manager y Azure PowerShell o la CLI de Azure. Para más información, consulte las [herramientas de plantilla](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Revisión de la plantilla

Al final del tutorial anterior, la plantilla tenía el siguiente código JSON:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-storage/azuredeploy.json":::

Es posible que haya observado que existe un problema con esta plantilla. El nombre de la cuenta de almacenamiento está codificado de forma rígida. Esta plantilla solo se puede usar para implementar la misma cuenta de almacenamiento cada vez. Para implementar una cuenta de almacenamiento con un nombre diferente, tendría que crear una nueva plantilla, lo que obviamente no es una forma práctica de automatizar las implementaciones.

## <a name="make-template-reusable"></a>Convertir la plantilla en reutilizable

Para que la plantilla se pueda volver a usar, vamos a agregar un parámetro que puede usar para pasar un nombre de cuenta de almacenamiento. El código JSON resaltado en el ejemplo siguiente muestra lo que ha cambiado en la plantilla. El parámetro **storageName** se identifica como una cadena. La longitud máxima se establece en 24 caracteres para evitar que los nombres sean demasiado largos.

Copie el archivo completo y reemplace la plantilla por su contenido.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-name/azuredeploy.json" range="1-26" highlight="4-10,15":::

## <a name="deploy-template"></a>Implementar plantilla

Vamos a implementar la plantilla. En el ejemplo siguiente se implementa la plantilla con la CLI de Azure o PowerShell. Observe que se proporciona el nombre de la cuenta de almacenamiento como uno de los valores en el comando de implementación. Para el nombre de la cuenta de almacenamiento, proporcione el mismo nombre que usó en el tutorial anterior.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](template-tutorial-create-first-template.md#create-resource-group). En el ejemplo se supone que ha establecido la variable **templateFile** en la ruta de acceso al archivo de plantilla, como se muestra en el [primer tutorial](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addnameparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storageName "{your-unique-name}"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addnameparameter \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storageName={your-unique-name}
```

---

## <a name="understand-resource-updates"></a>Descripción de actualizaciones de recursos

En la sección anterior, ha implementado una cuenta de almacenamiento con el mismo nombre que creó anteriormente. Es posible que se pregunte cómo afecta al recurso la reimplementación.

Si el recurso ya existe y no se detecta ningún cambio en las propiedades, no se realiza ninguna acción. Si el recurso ya existe y se produce un cambio en alguna propiedad, el recurso se actualiza. Si el recurso no existe, se crea.

Esta manera de controlar las actualizaciones implica que la plantilla puede incluir todos los recursos necesarios para una solución de Azure. Puede volver a implementar la plantilla de forma segura con la certeza de que los recursos se cambiarán o se crearán solo cuando sea necesario. Por ejemplo, si ha agregado archivos a su cuenta de almacenamiento, puede volver a implementar la cuenta de almacenamiento sin perder esos archivos.

## <a name="customize-by-environment"></a>Personalización por entorno

Los parámetros le permiten personalizar la implementación al proporcionar valores que son específicos para un entorno concreto. Por ejemplo, puede pasar diferentes valores en función de si está implementando en un entorno de desarrollo, de prueba o de producción.

La plantilla anterior siempre implementaba una cuenta de almacenamiento Standard_LRS. Es posible que desee disponer de flexibilidad para implementar diferentes SKU según el entorno. En el ejemplo siguiente se resaltan los cambios para agregar un parámetro para la SKU. Copie el archivo completo y péguelo en la plantilla.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-sku/azuredeploy.json" range="1-40" highlight="10-23,32":::

El parámetro **storageSKU** tiene un valor predeterminado. Este valor se usa cuando no se especifica ningún valor durante la implementación. También tiene una lista de valores permitidos. Estos valores coinciden con los valores necesarios para crear una cuenta de almacenamiento. Deseará evitar que los usuarios de su plantilla pasen SKU que no funcionan.

## <a name="redeploy-template"></a>Volver a implementar la plantilla

Está listo para volver a implementar. Dado que la SKU predeterminada está establecida en **Standard_LRS**, no es necesario proporcionar un valor para ese parámetro.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addskuparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storageName "{your-unique-name}"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az deployment group create \
  --name addskuparameter \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storageName={your-unique-name}
```

---

> [!NOTE]
> Si se ha producido un error en la implementación, use el modificador **debug** con el comando de implementación para mostrar los registros de depuración.  También puede usar el modificador **verbose** para mostrar los registros de depuración completos.

Para observar la flexibilidad de la plantilla, vamos a implementarla de nuevo. Esta vez, establezca el parámetro de SKU en **Standard_GRS**. Puede pasar un nuevo nombre para crear una cuenta de almacenamiento diferente, o bien usar el mismo nombre para actualizar la cuenta de almacenamiento existente. Ambas opciones funcionan.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name usenondefaultsku `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storageName "{your-unique-name}" `
  -storageSKU Standard_GRS
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az deployment group create \
  --name usenondefaultsku \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storageSKU=Standard_GRS storageName={your-unique-name}
```

---

Por último, vamos a realizar una prueba más y ver lo que sucede cuando se pasa una SKU que no es uno de los valores permitidos. En este caso, probaremos un escenario en el que un usuario de la plantilla piensa que **basic** es una de las SKU.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name testskuparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storageName "{your-unique-name}" `
  -storageSKU basic
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az deployment group create \
  --name testskuparameter \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storageSKU=basic storageName={your-unique-name}
```

---

Se produce un error en el comando inmediatamente con un mensaje de error que indica qué valores se permiten. Resource Manager identifica el error antes de que se inicie la implementación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha mejorado la plantilla creada en el [primer tutorial](template-tutorial-create-first-template.md) mediante la adición de parámetros. En el siguiente tutorial, obtendrá información sobre las funciones de plantilla.

> [!div class="nextstepaction"]
> [Adición de funciones de plantilla](template-tutorial-add-functions.md)
