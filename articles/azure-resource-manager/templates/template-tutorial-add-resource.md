---
title: 'Tutorial: Adición de un recurso a una plantilla'
description: Describe los pasos para crear su primera plantilla de Azure Resource Manager. Obtendrá información sobre la sintaxis del archivo de plantilla y cómo implementar una cuenta de almacenamiento.
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: 0163fcc62ea6c813af8e9e5693ca35fa0e650f96
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497535"
---
# <a name="tutorial-add-a-resource-to-your-arm-template"></a>Tutorial: Incorporación de un recurso a la plantilla de Resource Manager

En el [tutorial anterior](template-tutorial-create-first-template.md), aprendió a crear una plantilla en blanco y a implementarla. Ya está listo para implementar un recurso. En este tutorial, agregará una cuenta de almacenamiento. Este tutorial se realiza en unos **9 minutos**.

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial de introducción a las plantillas](template-tutorial-create-first-template.md).

Debe tener Visual Studio Code con la extensión Herramientas de Resource Manager y Azure PowerShell o la CLI de Azure. Para más información, consulte las [herramientas de plantilla](template-tutorial-create-first-template.md#get-tools).

## <a name="add-resource"></a>Agregar recurso

Para agregar una definición de cuenta de almacenamiento a la plantilla existente, examine el código JSON resaltado en el ejemplo siguiente. En lugar de intentar copiar secciones de la plantilla, copie todo el archivo y reemplace su plantilla por su contenido.

Reemplace **{provide-unique-name}** (incluidos los corchetes) por un nombre único de cuenta de almacenamiento.

> [!IMPORTANT]
> El nombre de la cuenta de almacenamiento debe ser único en Azure. El nombre debe tener solo letras minúsculas o números. No debe superar los 24 caracteres. Para el nombre, puede probar a utilizar **store1** como prefijo y, a continuación, agregar sus iniciales y la fecha del día. Por ejemplo, podría usar el nombre **store1abc09092019**.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-storage/azuredeploy.json" range="1-19" highlight="5-17":::

No es fácil adivinar un nombre único para una cuenta de almacenamiento y no resulta eficaz para automatizar implementaciones de gran tamaño. Más adelante en esta serie de tutoriales, usará características de plantilla que facilitan la creación de un nombre único.

## <a name="resource-properties"></a>Propiedades del recurso

Es posible que se pregunte cómo buscar las propiedades que se usarán para cada tipo de recurso. Puede usar la [referencia de plantilla de Resource Manager](/azure/templates/) para buscar los tipos de recursos que desea implementar.

Cada recurso implementado tiene al menos las siguientes tres propiedades:

- **type**: Tipo de recurso. Este valor es una combinación del espacio de nombres del proveedor de recursos y el tipo de recurso (como Microsoft.Storage/storageAccounts).
- **apiVersion**: Versión de la API de REST que debe usar para crear el recurso. Cada proveedor de recursos publicó sus propias versiones de API, por lo que este valor es específico del tipo.
- **name**: Nombre del recurso.

La mayoría de los recursos también tienen una propiedad **location**, que establece la región donde se implementa el recurso.

Las demás propiedades varían según el tipo de recurso y la versión de API. Es importante comprender la relación entre la versión de API y las propiedades disponibles, así que lo analizaremos con más detalle.

En este tutorial, ha agregado una cuenta de almacenamiento a la plantilla. Puede ver la versión de API en [storageAccounts 2019-04-01](/azure/templates/microsoft.storage/2019-04-01/storageaccounts). Tenga en cuenta que no ha agregado todas las propiedades a la plantilla. Muchas de las propiedades son opcionales. El proveedor de recursos Microsoft.Storage podría publicar una nueva versión de API, pero la versión que está implementando no tiene que cambiar. Puede seguir usando esa versión con la seguridad de que los resultados de la implementación serán coherentes.

Si ve una versión de API anterior, como [storageAccounts 2016-05-01](/azure/templates/microsoft.storage/2016-05-01/storageaccounts), verá que hay disponible un conjunto más pequeño de propiedades.

Si decide cambiar la versión de API de un recurso, asegúrese de evaluar las propiedades de esa versión y ajustar la plantilla consecuentemente.

## <a name="deploy-template"></a>Implementar plantilla

Puede implementar la plantilla para crear la cuenta de almacenamiento. Asigne un nombre diferente a la implementación para poder encontrarla fácilmente en el historial.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](template-tutorial-create-first-template.md#create-resource-group). En el ejemplo se supone que ha establecido la variable **templateFile** en la ruta de acceso al archivo de plantilla, como se muestra en el [primer tutorial](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addstorage `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addstorage \
  --resource-group myResourceGroup \
  --template-file $templateFile
```

---

> [!NOTE]
> Si se ha producido un error en la implementación, use el modificador **debug** con el comando de implementación para mostrar los registros de depuración.  También puede usar el modificador **verbose** para mostrar los registros de depuración completos.

Dos posibles errores de implementación que puede encontrar:

- Error: Code=AccountNameInvalid; Message={provide-unique-name} no es un nombre de cuenta de almacenamiento válido. El nombre de la cuenta de almacenamiento debe tener entre 3 y 24 caracteres, y solo puede contener números y letras minúsculas.

    En la plantilla, reemplace **{provide-unique-name}** por un nombre único de cuenta de almacenamiento.  Consulte [Agregar recurso](#add-resource).

- Error: Code=StorageAccountAlreadyTaken; Message=El nombre de la cuenta de almacenamiento store1abc09092019 ya está en uso.

    En la plantilla, pruebe con un nombre de cuenta de almacenamiento diferente.

Esta implementación tarda más tiempo que la implementación de una plantilla en blanco porque se crea la cuenta de almacenamiento. Puede tardar aproximadamente un minuto, aunque suele ser más rápida.

## <a name="verify-deployment"></a>Comprobación de la implementación

Para comprobar la implementación, explore el grupo de recursos desde Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú izquierdo, seleccione **Grupos de recursos**.
1. Seleccione el grupo de recursos en el que ha realizado la implementación.
1. Verá que se ha implementado una cuenta de almacenamiento.
1. Observe que la etiqueta de implementación ahora indica: **Deployments: 2 Succeeded** (Implementaciones: 2 correctas).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha creado una plantilla sencilla para implementar una cuenta de almacenamiento de Azure.  En los tutoriales posteriores, aprenderá a agregar parámetros, variables, recursos y salidas a una plantilla. Estas características son los bloques de creación de plantillas mucho más complejas.

> [!div class="nextstepaction"]
> [Adición de parámetros](template-tutorial-add-parameters.md)
