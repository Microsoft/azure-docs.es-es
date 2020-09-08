---
title: Introducción a las especificaciones de plantilla
description: Describe cómo crear especificaciones de plantilla y compartirlas con otros usuarios de la organización.
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: tomfitz
author: tfitzmac
ms.openlocfilehash: fad7ca60e98dcaabc5f6fc106e0d2c1b77085d67
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89227889"
---
# <a name="azure-resource-manager-template-specs-preview"></a>Especificaciones de plantilla de Azure Resource Manager (versión preliminar)

Una especificación de plantilla es un nuevo tipo de recurso para almacenar una plantilla de Azure Resource Manager (plantilla de ARM) en Azure para su posterior implementación. Este tipo de recurso permite compartir plantillas de ARM con otros usuarios de la organización. Al igual que cualquier otro recurso de Azure, puede usar el control de acceso basado en rol (RBAC) para compartir la especificación de plantilla.

**Microsoft.Resources/templateSpecs** es el nuevo tipo de recurso para las especificaciones de plantilla. Consta de una plantilla principal y un número indefinido de plantillas vinculadas. Azure almacena de forma segura las especificaciones de plantilla en grupos de recursos. Las especificaciones de plantilla son compatibles con el [control de versiones](#versioning).

Para implementar la especificación de plantilla, use herramientas estándar de Azure como PowerShell, la CLI de Azure, Azure Portal, REST, y otros SDK y clientes compatibles. Utilice los mismos comandos que utilizaría para la plantilla.

> [!NOTE]
> Las especificaciones de plantilla se encuentran actualmente en versión preliminar. Para usarlas, debe [regístrese para la lista de espera](https://aka.ms/templateSpecOnboarding).

## <a name="why-use-template-specs"></a>¿Por qué usar especificaciones de plantilla?

Si actualmente tiene sus plantillas en un repositorio de GitHub o una cuenta de almacenamiento, se enfrenta a varios desafíos al intentar compartir y usar las plantillas. Para que un usuario la implemente, la plantilla debe ser local o su dirección URL debe ser accesible públicamente. Para superar esta limitación, puede compartir copias de la plantilla con los usuarios que necesiten implementarla o abrir el acceso al repositorio o a la cuenta de almacenamiento. Cuando los usuarios poseen copias locales de una plantilla, estas pueden diferir de la plantilla original. Cuando crea un repositorio o una cuenta de almacenamiento accesible públicamente, puede permitir que usuarios no deseados accedan a la plantilla.

La ventaja de utilizar especificaciones de plantilla es que puede crear plantillas canónicas y compartirlas con los equipos de la organización. Las especificaciones de plantilla son seguras porque están disponibles para Azure Resource Manager para la implementación, pero no para los usuarios sin permiso RBAC. Los usuarios solo necesitan acceso de lectura a la especificación de plantilla para implementar su plantilla, por lo que puede compartir la plantilla sin permitir que otros la modifiquen.

Los administradores de la organización deben comprobar las plantillas que se incluyen en una especificación de plantilla para seguir los requisitos y las instrucciones de la organización.

## <a name="create-template-spec"></a>Crear una especificación de plantilla

En el ejemplo siguiente se muestra una plantilla sencilla para crear una cuenta de almacenamiento en Azure.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS",
        "Premium_LRS"
      ]
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[concat('store', uniquestring(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "kind": "StorageV2",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      }
    }
  ]
}
```

Cuando se crea la especificación de plantilla, los comandos de PowerShell o la CLI se pasan el archivo de plantilla principal. Si la plantilla principal hace referencia a las plantillas vinculadas, los comandos las buscarán y empaquetarán para crear la especificación de plantilla. Para obtener más información, consulte [Creación de una especificación de plantilla con plantillas vinculadas](#create-a-template-spec-with-linked-templates).

Cree una especificación de plantilla mediante:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzTemplateSpec -Name storageSpec -Version 1.0 -ResourceGroupName templateSpecsRg -Location westus2 -TemplateJsonFile ./mainTemplate.json
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts create \
  --name storageSpec \
  --version "1.0" \
  --resource-group templateSpecRG \
  --location "westus2" \
  --template-file "./mainTemplate.json"
```

---

Puede ver todas las especificaciones de plantilla de su suscripción mediante:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzTemplateSpec
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts list
```

---

Puede ver los detalles de una especificación de plantilla, incluidas sus versiones con:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzTemplateSpec -ResourceGroupName templateSpecsRG -Name storageSpec
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts show \
    --name storageSpec \
    --resource-group templateSpecRG \
    --version "1.0"
```

---

## <a name="deploy-template-spec"></a>Implementación de la especificación de plantilla

Después de crear la especificación de plantilla, los usuarios con acceso de **lectura** a la especificación de plantilla pueden implementarla. Para obtener más información acerca de cómo conceder acceso, consulte [Tutorial: Concesión de acceso de grupo a recursos de Azure mediante Azure PowerShell](../../role-based-access-control/tutorial-role-assignments-group-powershell.md).

Las especificaciones de plantilla se pueden implementar a través del portal, de PowerShell, de la CLI de Azure o como una plantilla vinculada en una implementación de plantilla más grande. Los usuarios de una organización pueden implementar una especificación de plantilla en cualquier ámbito de Azure (grupo de recursos, suscripción, grupo de administración o inquilino).

En lugar de pasar una ruta de acceso o un URI para una plantilla, para implementar una especificación de plantilla puede proporcionar su identificador de recurso. El identificador de recurso tiene el formato siguiente:

**/subscriptions/{id-de-suscripción}/resourceGroups/{grupo-de-recursos}/providers/Microsoft.Resources/templateSpecs/{nombre-de-especificación-de-plantilla}/versions/{versión-de-especificación-de-plantilla}**

Observe que el identificador de recurso incluye un número de versión para la especificación de plantilla.

Por ejemplo, puede implementar una especificación de plantilla mediante el siguiente comando.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$id = "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/templateSpecsRG/providers/Microsoft.Resources/templateSpecs/storageSpec/versions/1.0"

New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
id = "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/templateSpecsRG/providers/Microsoft.Resources/templateSpecs/storageSpec/versions/1.0"

az deployment group create \
  --resource-group demoRG \
  --template-spec $id
```

---

En la práctica, normalmente ejecutará `Get-AzTemplateSpec` para obtener el identificador de la especificación de plantilla que quiere implementar.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$id = (Get-AzTemplateSpec -Name storageSpec -ResourceGroupName templateSpecsRg -Version 1.0).Version.Id

New-AzResourceGroupDeployment `
  -ResourceGroupName demoRG `
  -TemplateSpecId $id
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
id = $(az ts show --name storageSpec --resource-group templateSpecRG --version "1.0" --query "id")

az deployment group create \
  --resource-group demoRG \
  --template-spec $id
```

---

## <a name="parameters"></a>Parámetros

Pasar parámetros a la especificación de plantilla es exactamente igual que pasar parámetros a una plantilla de Resource Manager. Agregue los valores de parámetro insertados o en un archivo de parámetros.

Para pasar un parámetro insertado, use:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG `
  -StorageAccountType Standard_GRS
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --resource-group demoRG \
  --template-spec $id \
  --parameters storageAccountType='Standard_GRS'
```

---

Para crear un archivo de parámetros local, use:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "StorageAccountType": {
      "value": "Standard_GRS"
    }
  }
}
```

Y pase el archivo de parámetros con:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG `
  -TemplateParameterFile ./mainTemplate.parameters.json
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --resource-group demoRG \
  --template-spec $id \
  --parameters "./mainTemplate.parameters.json"
```

---

## <a name="create-a-template-spec-with-linked-templates"></a>Creación de una especificación de plantilla con plantillas vinculadas

Si la plantilla principal de la especificación de plantilla hace referencia a plantillas vinculadas, los comandos de PowerShell y de la CLI pueden buscar y empaquetar automáticamente las plantillas vinculadas desde la unidad local. No es necesario configurar manualmente las cuentas de almacenamiento ni los repositorios para hospedar las especificaciones de plantilla: todo es independiente en el recurso de especificación de plantilla.

El siguiente ejemplo consta de una plantilla principal con dos plantillas vinculadas. El ejemplo es solo un extracto de la plantilla. Observe que usa una propiedad denominada `relativePath` para la vinculación a las otras plantillas. Debe usar `apiVersion` `2020-06-01` o posterior para el recurso de implementaciones.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  ...
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "relativePath": "artifacts/webapp.json"
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "relativePath": "artifacts/database.json"
        }
      }
    }
  ],
  "outputs": {}
}
```

Cuando se ejecuta el comando de PowerShell o de la CLI para crear la especificación de plantilla del ejemplo anterior, el comando busca tres archivos: la plantilla principal, la plantilla de aplicación web (`webapp.json`) y la plantilla de base de datos (`database.json`), y las empaqueta en la especificación de plantilla.

Para más información, consulte el [Tutorial: Creación de una especificación de plantilla con plantillas vinculadas](template-specs-create-linked.md).

## <a name="deploy-template-spec-as-a-linked-template"></a>Implementación de una especificación de plantilla como una plantilla vinculada

Una vez que haya creado una especificación de plantilla, es fácil reutilizarla desde una plantilla de ARM u otra especificación de plantilla. Para la vinculación a una especificación de plantilla, agregue su identificador de recurso a la plantilla. La especificación de plantilla vinculada se implementa automáticamente al implementar la plantilla principal. Este comportamiento permite desarrollar especificaciones de plantilla modulares y reutilizarlas según sea necesario.

Por ejemplo, puede crear una especificación de plantilla que implemente recursos de red y otra especificación de plantilla que implemente recursos de almacenamiento. En las plantillas de ARM, debe vincular estas dos especificaciones de plantilla cada vez que necesite configurar redes o recursos de almacenamiento.

El ejemplo siguiente es similar al anterior, pero se usa la propiedad `id` para la vinculación a una especificación de plantilla en lugar de la propiedad `relativePath` para la vinculación a una plantilla local. Use `2020-06-01` para la versión de API del recurso de implementaciones. En el ejemplo, las especificaciones de plantilla se encuentran en un grupo de recursos denominado **templateSpecsRG**.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  ...
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      "name": "networkingDeployment",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "id": "[resourceId('templateSpecsRG', 'Microsoft.Resources/templateSpecs/versions', 'networkingSpec', '1.0')]"
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      "name": "storageDeployment",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "id": "[resourceId('templateSpecsRG', 'Microsoft.Resources/templateSpecs/versions', 'storageSpec', '1.0')]"
        }
      }
    }
  ],
  "outputs": {}
}
```

Para obtener más información acerca de la vinculación de especificaciones de plantilla, consulte [Tutorial: Implementación de una especificación de plantilla como una plantilla vinculada](template-specs-deploy-linked-template.md).

## <a name="versioning"></a>Control de versiones

Cuando se crea una especificación de plantilla, se debe proporciona un número de versión para esta. A medida que realiza la iteración en el código de plantilla, puede actualizar una versión existente (para revisiones) o publicar una nueva versión. La versión es una cadena de texto. Puede optar por seguir cualquier sistema de control de versiones, incluido el control de versiones semántico. Los usuarios de la especificación de plantilla pueden proporcionar el número de versión que quieren usar al implementarla.

## <a name="next-steps"></a>Pasos siguientes

* Para crear e implementar una especificación de plantilla, consulte [Inicio rápido: Creación e implementación de una especificación de plantilla](quickstart-create-template-specs.md).

* Para obtener más información sobre la vinculación de plantillas en las especificaciones de plantilla, consulte [Tutorial: Creación de una especificación de plantilla con plantillas vinculadas](template-specs-create-linked.md).

* Para obtener más información sobre la implementación de una especificación de plantilla como una plantilla vinculada, consulte [Tutorial: Implementación de una especificación de plantilla como una plantilla vinculada](template-specs-deploy-linked-template.md).
