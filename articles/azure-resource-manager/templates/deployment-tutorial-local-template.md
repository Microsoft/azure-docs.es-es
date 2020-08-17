---
title: 'Tutorial: Implementación de una plantilla de Azure Resource Manager local'
description: Aprenda a implementar una plantilla de Azure Resource Manager desde la máquina local.
ms.date: 05/20/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: ef52d20b20446e3b8f3b761949bf255b99d358cc
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499592"
---
# <a name="tutorial-deploy-a-local-azure-resource-manager-template"></a>Tutorial: Implementación de una plantilla de Azure Resource Manager local

Aprenda a implementar una plantilla de Azure Resource Manager desde la máquina local. Se tarda en realizar unos **8 minutos**.

Este tutorial es el primero de una serie. A medida que avanza por la serie, puede modularizar la plantilla mediante la creación de una plantilla vinculada, almacenar la plantilla vinculada en una cuenta de almacenamiento y proteger esta plantilla mediante el token de SAS, y obtener información sobre cómo crear una canalización DevOp para implementar plantillas. Esta serie se centra en la implementación de plantillas.  Si desea conocer el desarrollo de plantillas, consulte los [tutoriales para principiantes](./template-tutorial-create-first-template.md).

## <a name="get-tools"></a>Obtención de las herramientas

Comience por asegurarse de que tiene las herramientas que necesita para implementar plantillas.

### <a name="command-line-deployment"></a>Implementación desde la línea de comandos

Necesita Azure PowerShell o la CLI de Azure para implementar la plantilla. Para obtener instrucciones de instalación, consulte:

- [Azure PowerShell](/powershell/azure/install-az-ps)
- [Instalación de la CLI de Azure en Windows](/cli/azure/install-azure-cli-windows)
- [Instalación de la CLI de Azure en Linux](/cli/azure/install-azure-cli-linux)

Después de instalar Azure PowerShell o la CLI de Azure, asegúrese de iniciar sesión por primera vez. Para obtener ayuda, consulte [Inicio de sesión: PowerShell](/powershell/azure/install-az-ps#sign-in) o [ CLI de Azure](/cli/azure/get-started-with-azure-cli#sign-in).

### <a name="editor-optional"></a>Editor (opcional)

Las plantillas son archivos JSON. Para revisar o editar plantillas, necesita un buen editor de JSON. Se recomienda Visual Studio Code con la extensión Herramientas de Resource Manager. Si necesita instalar estas herramientas, consulte [Inicio rápido: Creación de plantillas de Azure Resource Manager con Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

## <a name="review-template"></a>Revisión de la plantilla

La plantilla implementa una cuenta de almacenamiento, un plan de App Service y una aplicación web. Si está interesado en crear la plantilla, puede seguir el [tutorial sobre las plantillas de inicio rápido](template-tutorial-quickstart-template.md). No obstante, no es necesario para completar este tutorial.

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/local-template/azuredeploy.json":::

> [!IMPORTANT]
> Los nombres de cuentas de almacenamiento deben tener entre 3 y 24 caracteres, y usar solo números y letras minúsculas. El nombre debe ser único. En la plantilla, el nombre de la cuenta de almacenamiento es el nombre del proyecto con "store" anexado y el nombre del proyecto debe tener entre 3 y 11 caracteres. Por lo tanto, el nombre del proyecto debe cumplir los requisitos de nombre para la cuenta de almacenamiento y tener menos de 11 caracteres.

Guarde una copia de la plantilla en la máquina local con la extensión .json, por ejemplo, azuredeploy.json. Esta plantilla se implementa más adelante en el tutorial.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Para empezar a trabajar con Azure PowerShell o la CLI de Azure para implementar una plantilla, inicie sesión con sus credenciales de Azure.

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
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource and resource group names"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroup `
  -Name $resourceGroupName `
  -Location "Central US"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
echo "Enter a project name that is used to generate resource and resource group names:"
read projectName
resourceGroupName="${projectName}rg"

az group create \
  --name $resourceGroupName \
  --location "Central US"
```

---

## <a name="deploy-template"></a>Implementar plantilla

Use una o ambas opciones de implementación para implementar la plantilla.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name"
$templateFile = Read-Host -Prompt "Enter the template file path and file name"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroupDeployment `
  -Name DeployLocalTemplate `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile $templateFile `
  -projectName $projectName `
  -verbose
```

Para más información sobre la implementación de plantillas con Azure PowerShell, consulte [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](./deploy-powershell.md).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
echo "Enter the same project name:"
read projectName
echo "Enter the template file path and file name:"
read templateFile
resourceGroupName="${projectName}rg"

az deployment group create \
  --name DeployLocalTemplate \
  --resource-group $resourceGroupName \
  --template-file $templateFile \
  --parameters projectName=$projectName \
  --verbose
```

Para más información sobre la implementación de plantillas con la CLI de Azure, consulte [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](./deploy-cli.md).

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha aprendido a implementar una plantilla local. En el siguiente tutorial, dividirá la plantilla en una plantilla principal y una plantilla vinculada, y aprenderá a almacenar y proteger la plantilla vinculada.

> [!div class="nextstepaction"]
> [Implementación de una plantilla vinculada](./deployment-tutorial-linked-template.md)
