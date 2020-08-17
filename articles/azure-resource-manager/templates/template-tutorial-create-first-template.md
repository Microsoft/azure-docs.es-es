---
title: 'Tutorial: Creación e implementación de una plantilla'
description: Creación de la primera plantilla de Azure Resource Manager En este tutorial, aprenderá sobre la sintaxis del archivo de plantilla y cómo implementar una cuenta de almacenamiento.
author: mumian
ms.date: 06/10/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: 140eb73cdf6d5bb701c08b0dd8d5f85b6f137df6
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497484"
---
# <a name="tutorial-create-and-deploy-your-first-arm-template"></a>Tutorial: Creación e implementación de su primera plantilla de Resource Manager

Este tutorial es una introducción a las plantillas de Azure Resource Manager. Muestra cómo crear una plantilla de inicio e implementarla en Azure. Obtendrá información sobre la estructura de la plantilla y las herramientas que necesitará para trabajar con plantillas. Se tarda unos **12 minutos** en realizar este tutorial, pero el tiempo real variará en función del número de herramientas que necesite instalar.

Este tutorial es el primero de una serie. A medida que avance por la serie, modificará la plantilla inicial paso a paso hasta que haya explorado todas las partes principales de una plantilla de Resource Manager. Estos elementos son los bloques de creación de plantillas mucho más complejas. Esperamos que el final de la serie tenga seguridad para crear sus propias plantillas y esté listo para automatizar las implementaciones con plantillas.

Si desea obtener información sobre las ventajas de usar plantillas y por qué debería automatizar la implementación con plantillas, consulte [Plantillas de Azure Resource Manager](overview.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="get-tools"></a>Obtención de las herramientas

Comience por asegurarse de que tiene las herramientas que necesita para crear e implementar plantillas.

### <a name="editor"></a>Editor

Las plantillas son archivos JSON. Para crear plantillas, necesita un buen editor de JSON. Se recomienda Visual Studio Code con la extensión Herramientas de Resource Manager. Si necesita instalar estas herramientas, consulte [Inicio rápido: Creación de plantillas de Azure Resource Manager con Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

### <a name="command-line-deployment"></a>Implementación desde la línea de comandos

También necesita Azure PowerShell o la CLI de Azure para implementar la plantilla. Si usa la CLI de Azure, debe tener la versión más reciente. Para obtener instrucciones de instalación, consulte:

- [Azure PowerShell](/powershell/azure/install-az-ps)
- [Instalación de la CLI de Azure en Windows](/cli/azure/install-azure-cli-windows)
- [Instalación de la CLI de Azure en Linux](/cli/azure/install-azure-cli-linux)
- [Instalación de la CLI de Azure en macOS](/cli/azure/install-azure-cli-macos)

Después de instalar Azure PowerShell o la CLI de Azure, asegúrese de iniciar sesión por primera vez. Para obtener ayuda, consulte [Inicio de sesión: PowerShell](/powershell/azure/install-az-ps#sign-in) o [ CLI de Azure](/cli/azure/get-started-with-azure-cli#sign-in).

> [!IMPORTANT]
> Si utiliza la CLI de Azure, asegúrese de tener la versión 2.6 o posterior. Los comandos mostrados en este tutorial no funcionarán si utiliza versiones anteriores. Para comprobar la versión instalada, utilice `az --version`.

Bien, está listo para empezar a obtener información sobre las plantillas.

## <a name="create-your-first-template"></a>Creación de la primera plantilla

1. Abra Visual Studio Code con la extensión Herramientas de Resource Manager instalada.
1. En el menú **Archivo**, seleccione **Nuevo archivo** para crear un archivo nuevo.
1. En el menú **Archivo**, seleccione **Guardar como**.
1. Asigne al archivo el nombre **azuredeploy** y seleccione la extensión de archivo **JSON**. El nombre completo del archivo es **azuredeploy.json**.
1. Guarde el archivo en la estación de trabajo. Seleccione una ruta de acceso que sea fácil de recordar porque tendrá que proporcionar esa ruta de acceso más adelante al implementar la plantilla.
1. Copie y pegue el siguiente código JSON en el archivo:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "resources": []
    }
    ```

    Este es el aspecto del entorno de VS Code:

    ![Primera plantilla de Resource Manager con Visual Studio Code](./media/template-tutorial-create-first-template/resource-manager-visual-studio-code-first-template.png)

    Esta plantilla no implementa ningún recurso. Vamos a empezar con una plantilla en blanco para que pueda familiarizarse con los pasos necesarios para implementar una plantilla y minimizar la posibilidad de que algo vaya mal.

    El archivo JSON tiene estos elementos:

    - **$schema**: Especifica la ubicación del archivo de esquema JSON. El archivo de esquema describe las propiedades que están disponibles dentro de una plantilla. Por ejemplo, el esquema define **resources** (recursos) como una de las propiedades válidas para una plantilla. No se preocupe de que la fecha del esquema sea 2019-04-01. Esta versión del esquema está actualizada e incluye todas las características más recientes. La fecha del esquema no ha cambiado porque no se han producido cambios importantes desde su introducción.
    - **contentVersion**: Especifica la versión de la plantilla (por ejemplo, 1.0.0.0). Puede especificar cualquier valor para este elemento. Use este valor para documentar los cambios importantes de la plantilla. Al implementar los recursos con la plantilla, este valor se puede usar para asegurarse de que se está usando la plantilla correcta.
    - **resources**: Contiene los recursos que desea implementar o actualizar. Actualmente está vacía, pero agregará recursos más adelante.

1. Guarde el archivo.

Enhorabuena, ha creado su primera plantilla.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Para empezar a trabajar con Azure PowerShell o la CLI de Azure, inicie sesión con sus credenciales de Azure.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Connect-AzAccount
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az login
```

---

Si tiene varias suscripciones de Azure, seleccione la suscripción que desee usar:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Set-AzContext [SubscriptionID/SubscriptionName]
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az account set --subscription [SubscriptionID/SubscriptionName]
```

---


## <a name="create-resource-group"></a>Creación de un grupo de recursos

Al implementar una plantilla, se especifica un grupo de recursos que contendrá los recursos. Antes de ejecutar el comando de implementación, cree el grupo de recursos con la CLI de Azure o Azure PowerShell. Seleccione las pestañas en la siguiente sección de código para elegir entre Azure PowerShell y la CLI de Azure. Los ejemplos de la CLI de este artículo están escritos para el shell de Bash.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroup `
  -Name myResourceGroup `
  -Location "Central US"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az group create \
  --name myResourceGroup \
  --location "Central US"
```

---

## <a name="deploy-template"></a>Implementar plantilla

Use la CLI de Azure o Azure PowerShell para implementar la plantilla. Use el grupo de recursos que creó. Asigne un nombre a la implementación para que pueda identificarla fácilmente en el historial de implementaciones. Para mayor comodidad, cree también una variable que almacene la ruta de acceso al archivo de plantilla. Esta variable facilita la ejecución de los comandos de implementación, ya que no es necesario volver a escribir la ruta de acceso cada vez que se implementa.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$templateFile = "{provide-the-path-to-the-template-file}"
New-AzResourceGroupDeployment `
  -Name blanktemplate `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
templateFile="{provide-the-path-to-the-template-file}"
az deployment group create \
  --name blanktemplate \
  --resource-group myResourceGroup \
  --template-file $templateFile
```

---

El comando de implementación devuelve los resultados. Busque `ProvisioningState` para ver si la implementación se realizó correctamente.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

![Estado de aprovisionamiento de la implementación con PowerShell](./media/template-tutorial-create-first-template/resource-manager-deployment-provisioningstate.png)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

![Estado de aprovisionamiento de la implementación con la CLI de Azure](./media/template-tutorial-create-first-template/azure-cli-provisioning-state.png)

---

> [!NOTE]
> Si se ha producido un error en la implementación, use el modificador **debug** con el comando de implementación para mostrar los registros de depuración.  También puede usar el modificador **verbose** para mostrar los registros de depuración completos.

## <a name="verify-deployment"></a>Comprobación de la implementación

Para comprobar la implementación, explore el grupo de recursos desde Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En el menú izquierdo, seleccione **Grupos de recursos**.

1. Seleccione el grupo de recursos que implementó en el último procedimiento. El nombre predeterminado es **myResourceGroup**. No debería ver ningún recurso implementado en el grupo de recursos.

1. Observe que en la esquina superior derecha de la información general se muestra el estado de la implementación. Seleccione **1 Correcta**.

   ![Ver el estado de implementación](./media/template-tutorial-create-first-template/deployment-status.png)

1. Verá un historial de implementación del grupo de recursos. Seleccione **blanktemplate**.

   ![Selección de la implementación](./media/template-tutorial-create-first-template/select-from-deployment-history.png)

1. Verá un resumen de la implementación. En este caso, no hay mucho que ver porque no se ha implementado ningún recurso. Más adelante en esta serie puede resultarle útil revisar el resumen en el historial de implementaciones. Observe que a la izquierda puede ver las entradas, las salidas y la plantilla que se usó durante la implementación.

   ![Ver el resumen de la implementación](./media/template-tutorial-create-first-template/view-deployment-summary.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha creado una plantilla sencilla para implementarla en Azure. En el siguiente tutorial, agregará una cuenta de almacenamiento a la plantilla y la implementará en el grupo de recursos.

> [!div class="nextstepaction"]
> [Agregar recurso](template-tutorial-add-resource.md)
