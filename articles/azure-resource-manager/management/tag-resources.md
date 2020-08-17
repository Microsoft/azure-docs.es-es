---
title: Etiquetado de recursos, grupos de recursos y suscripciones para una organización lógica
description: Muestra cómo aplicar etiquetas para organizar los recursos de Azure para la facturación y administración.
ms.topic: conceptual
ms.date: 07/27/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: daedb5dcd660ec2637557fe5af75db2939318495
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500000"
---
# <a name="use-tags-to-organize-your-azure-resources-and-management-hierarchy"></a>Uso de etiquetas para organizar los recursos de Azure y la jerarquía de administración

Puede aplicar etiquetas a los recursos, grupos de recursos y suscripciones de Azure con el fin de organizarlos de manera lógica en una taxonomía. Cada etiqueta consta de un nombre y un par de valores. Por ejemplo, puede aplicar el nombre "Environment" y el valor "Production" a todos los recursos en producción.

Para recomendaciones sobre cómo implementar una estrategia de etiquetado, consulte [Guía de decisiones de nomenclatura y etiquetado de recursos](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json).

> [!IMPORTANT]
> Las nombres de etiqueta no distinguen mayúsculas de minúsculas para las operaciones. Una etiqueta con un nombre de etiqueta, independientemente del uso de mayúsculas o minúsculas, se actualiza o se recupera. Sin embargo, el proveedor de recursos podría mantener el uso de mayúsculas y minúsculas para el nombre de la etiqueta. Puede ver dicho uso de mayúsculas y minúsculas en los informes de costos.
> 
> Los valores de etiqueta distinguen mayúsculas de minúsculas.

[!INCLUDE [Handle personal data](../../../includes/gdpr-intro-sentence.md)]

## <a name="required-access"></a>Acceso necesario

Para aplicar etiquetas a un recurso, debe tener acceso de escritura al tipo de recurso **Microsoft.Resources/tags**. El rol [Colaborador de etiquetas](../../role-based-access-control/built-in-roles.md#tag-contributor) le permite aplicar etiquetas a una entidad sin tener acceso a la entidad misma. Actualmente, el rol colaborador de etiquetas no puede aplicar etiquetas a recursos o grupos de recursos a través del portal. Puede aplicar etiquetas a las suscripciones a través del portal. Este admite todas las operaciones de etiqueta a través de PowerShell y la API REST.  

El rol [Colaborador](../../role-based-access-control/built-in-roles.md#contributor) también concede el acceso necesario para aplicar etiquetas a cualquier entidad. Para aplicar etiquetas a un solo tipo de recurso, use el rol de colaborador para ese recurso. Por ejemplo, para aplicar etiquetas a las máquinas virtuales, use [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).

## <a name="powershell"></a>PowerShell

### <a name="apply-tags"></a>Aplicación de etiquetas

Azure PowerShell ofrece dos comandos para aplicar etiquetas: [New-AzTag](/powershell/module/az.resources/new-aztag) y [Update-AzTag](/powershell/module/az.resources/update-aztag). Debe tener el módulo Az.Resources 1.12.0 o una versión posterior. Puede consultar su versión con `Get-Module Az.Resources`. Puede instalar ese módulo o [Azure PowerShell](/powershell/azure/install-az-ps) 3.6.1 o una versión posterior.

**New-AzTag** reemplaza todas las etiquetas en el recurso, el grupo de recursos o la suscripción. Al llamar al comando, pase el id. de recurso de la entidad que desea etiquetar.

En el ejemplo siguiente se aplica un conjunto de etiquetas a una cuenta de almacenamiento:

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
$resource = Get-AzResource -Name demoStorage -ResourceGroup demoGroup
New-AzTag -ResourceId $resource.id -Tag $tags
```

Cuando se complete el comando, observe que el recurso tiene dos etiquetas.

```output
Properties :
        Name    Value
        ======  =======
        Dept    Finance
        Status  Normal
```

Si vuelve a ejecutar el comando, pero esta vez con etiquetas diferentes, observe que se quitaron las etiquetas anteriores.

```azurepowershell-interactive
$tags = @{"Team"="Compliance"; "Environment"="Production"}
New-AzTag -ResourceId $resource.id -Tag $tags
```

```output
Properties :
        Name         Value
        ===========  ==========
        Environment  Production
        Team         Compliance
```

Para agregar etiquetas a un recurso que ya tiene etiquetas, use **Update-AzTag**. Establezca el parámetro **-Operation** en **Merge**.

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Merge
```

Observe que las dos etiquetas nuevas se agregaron a las dos etiquetas existentes.

```output
Properties :
        Name         Value
        ===========  ==========
        Status       Normal
        Dept         Finance
        Team         Compliance
        Environment  Production
```

Cada nombre de etiqueta solo puede tener un valor. Si proporciona un valor nuevo para una etiqueta, el valor anterior se reemplaza incluso si se usa la operación de combinación. En el ejemplo siguiente se cambia la etiqueta Status (Estado) de Normal a Green (Verde).

```azurepowershell-interactive
$tags = @{"Status"="Green"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Merge
```

```output
Properties :
        Name         Value
        ===========  ==========
        Status       Green
        Dept         Finance
        Team         Compliance
        Environment  Production
```

Al establecer el parámetro **-Operation** en **Replace**, el nuevo conjunto de etiquetas reemplaza a las etiquetas existentes.

```azurepowershell-interactive
$tags = @{"Project"="ECommerce"; "CostCenter"="00123"; "Team"="Web"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Replace
```

Solo las etiquetas nuevas siguen en el recurso.

```output
Properties :
        Name        Value
        ==========  =========
        CostCenter  00123
        Team        Web
        Project     ECommerce
```

Los mismos comandos también funcionan con grupos de recursos o suscripciones. Puede pasar el identificador del grupo de recursos o de la suscripción que quiere etiquetar.

Para agregar un nuevo conjunto de etiquetas a un grupo de recursos, use:

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
$resourceGroup = Get-AzResourceGroup -Name demoGroup
New-AzTag -ResourceId $resourceGroup.ResourceId -tag $tags
```

Para actualizar las etiquetas de un grupo de recursos, use:

```azurepowershell-interactive
$tags = @{"CostCenter"="00123"; "Environment"="Production"}
$resourceGroup = Get-AzResourceGroup -Name demoGroup
Update-AzTag -ResourceId $resourceGroup.ResourceId -Tag $tags -Operation Merge
```

Para agregar un nuevo conjunto de etiquetas a una suscripción, use:

```azurepowershell-interactive
$tags = @{"CostCenter"="00123"; "Environment"="Dev"}
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
New-AzTag -ResourceId "/subscriptions/$subscription" -Tag $tags
```

Para actualizar las etiquetas de una suscripción, use:

```azurepowershell-interactive
$tags = @{"Team"="Web Apps"}
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Update-AzTag -ResourceId "/subscriptions/$subscription" -Tag $tags -Operation Merge
```

Puede tener más de un recurso con el mismo nombre en un grupo de recursos. En ese caso, puede establecer cada recurso con los siguientes comandos:

```azurepowershell-interactive
$resource = Get-AzResource -ResourceName sqlDatabase1 -ResourceGroupName examplegroup
$resource | ForEach-Object { Update-AzTag -Tag @{ "Dept"="IT"; "Environment"="Test" } -ResourceId $_.ResourceId -Operation Merge }
```

### <a name="list-tags"></a>Lista de etiquetas

Para obtener las etiquetas de un recurso, un grupo de recursos o una suscripción, use el comando [Get-AzTag](/powershell/module/az.resources/get-aztag) y pase el id. de recurso de la entidad.

Para ver las etiquetas de un recurso, use:

```azurepowershell-interactive
$resource = Get-AzResource -Name demoStorage -ResourceGroup demoGroup
Get-AzTag -ResourceId $resource.id
```

Para ver las etiquetas de un grupo de recursos, use:

```azurepowershell-interactive
$resourceGroup = Get-AzResourceGroup -Name demoGroup
Get-AzTag -ResourceId $resourceGroup.ResourceId
```

Para ver las etiquetas de una suscripción, use:

```azurepowershell-interactive
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Get-AzTag -ResourceId "/subscriptions/$subscription"
```

### <a name="list-by-tag"></a>Enumerar por etiqueta

Para obtener recursos que tengan un nombre y valor de etiqueta específicos, use:

```azurepowershell-interactive
(Get-AzResource -Tag @{ "CostCenter"="00123"}).Name
```

Para obtener recursos que tengan un nombre de etiqueta específico y cualquier valor de etiqueta, use:

```azurepowershell-interactive
(Get-AzResource -TagName "Dept").Name
```

Para obtener grupos de recursos que tengan un nombre y valor de etiqueta específicos, use:

```azurepowershell-interactive
(Get-AzResourceGroup -Tag @{ "CostCenter"="00123" }).ResourceGroupName
```

### <a name="remove-tags"></a>Eliminación de etiquetas

Para quitar etiquetas específicas, use **Update-AzTag** y establezca **-Operation** en **Delete**. Pase las etiquetas que quiere eliminar.

```azurepowershell-interactive
$removeTags = @{"Project"="ECommerce"; "Team"="Web"}
Update-AzTag -ResourceId $resource.id -Tag $removeTags -Operation Delete
```

Se quitan las etiquetas especificadas.

```output
Properties :
        Name        Value
        ==========  =====
        CostCenter  00123
```

Para quitar todas las etiquetas, use el comando [Remove-AzTag](/powershell/module/az.resources/remove-aztag).

```azurepowershell-interactive
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Remove-AzTag -ResourceId "/subscriptions/$subscription"
```

## <a name="azure-cli"></a>Azure CLI

### <a name="apply-tags"></a>Aplicación de etiquetas

Al agregar etiquetas a un grupo de recursos o recurso, puede sobrescribir las etiquetas existentes o agregar nuevas etiquetas a las etiquetas existentes.

Para sobrescribir las etiquetas en un recurso, use:

```azurecli-interactive
az resource tag --tags 'Dept=IT' 'Environment=Test' -g examplegroup -n examplevnet --resource-type "Microsoft.Network/virtualNetworks"
```

Para anexar una etiqueta a las etiquetas existentes en un recurso, use:

```azurecli-interactive
az resource update --set tags.'Status'='Approved' -g examplegroup -n examplevnet --resource-type "Microsoft.Network/virtualNetworks"
```

Para sobrescribir las etiquetas existentes en un grupo de recursos, use:

```azurecli-interactive
az group update -n examplegroup --tags 'Environment=Test' 'Dept=IT'
```

Para anexar una etiqueta a las etiquetas existentes en un grupo de recursos, use:

```azurecli-interactive
az group update -n examplegroup --set tags.'Status'='Approved'
```

Actualmente, la CLI de Azure no tiene un comando para aplicar etiquetas a las suscripciones. Sin embargo, puede usar la CLI para implementar una plantilla de ARM que aplique las etiquetas a una suscripción. Consulte [Aplicación de etiquetas a grupos de recursos o suscripciones](#apply-tags-to-resource-groups-or-subscriptions).

### <a name="list-tags"></a>Lista de etiquetas

Para ver las etiquetas existentes de un recurso, use:

```azurecli-interactive
az resource show -n examplevnet -g examplegroup --resource-type "Microsoft.Network/virtualNetworks" --query tags
```

Para ver las etiquetas existentes de un grupo de recursos, use:

```azurecli-interactive
az group show -n examplegroup --query tags
```

Ese script devuelve el siguiente formato:

```json
{
  "Dept"        : "IT",
  "Environment" : "Test"
}
```

### <a name="list-by-tag"></a>Enumerar por etiqueta

Para obtener todos los recursos que tengan una etiqueta y un valor particulares, use `az resource list`:

```azurecli-interactive
az resource list --tag Dept=Finance
```

Para obtener grupos de recursos que tengan una etiqueta específica, use `az group list`:

```azurecli-interactive
az group list --tag Dept=IT
```

### <a name="handling-spaces"></a>Control de los espacios

Si los nombres o valores de etiqueta incluyen espacios, debe realizar un par de pasos adicionales. En el ejemplo siguiente se aplican todas las etiquetas de un grupo de recursos a sus recursos cuando las etiquetas pueden contener espacios.

```azurecli-interactive
jsontags=$(az group show --name examplegroup --query tags -o json)
tags=$(echo $jsontags | tr -d '{}"' | sed 's/: /=/g' | sed "s/\"/'/g" | sed 's/, /,/g' | sed 's/ *$//g' | sed 's/^ *//g')
origIFS=$IFS
IFS=','
read -a tagarr <<< "$tags"
resourceids=$(az resource list -g examplegroup --query [].id --output tsv)
for id in $resourceids
do
  az resource tag --tags "${tagarr[@]}" --id $id
done
IFS=$origIFS
```

## <a name="templates"></a>Plantillas

Puede etiquetar recursos, grupos de recursos y suscripciones durante la implementación con una plantilla de Resource Manager.

### <a name="apply-values"></a>Aplicación de valores

En el ejemplo siguiente se implementa una cuenta de almacenamiento con tres etiquetas. Dos de las etiquetas (`Dept` y `Environment`) se establecen en valores literales. Una etiqueta (`LastDeployed`) se establece en un parámetro que tiene como valor predeterminado la fecha actual.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "utcShort": {
            "type": "string",
            "defaultValue": "[utcNow('d')]"
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "Dept": "Finance",
                "Environment": "Production",
                "LastDeployed": "[parameters('utcShort')]"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-an-object"></a>Aplicación de un objeto

Puede definir un parámetro de objeto que almacene varias etiquetas y aplicar ese objeto al elemento de etiqueta. Este enfoque proporciona más flexibilidad que el ejemplo anterior, porque el objeto puede tener propiedades diferentes. Cada propiedad del objeto se convierte en una etiqueta independiente para el recurso. El siguiente ejemplo tiene un parámetro denominado `tagValues` que se aplica al elemento de etiqueta.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        },
        "tagValues": {
            "type": "object",
            "defaultValue": {
                "Dept": "Finance",
                "Environment": "Production"
            }
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": "[parameters('tagValues')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-a-json-string"></a>Aplicación de una cadena JSON

Para almacenar muchos valores en una única etiqueta, aplique una cadena JSON que represente los valores. Toda la cadena JSON se almacena como una etiqueta que no puede superar los 256 caracteres. En el ejemplo siguiente se muestra una etiqueta denominada `CostCenter` que contiene varios valores de una cadena JSON:  

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "CostCenter": "{\"Dept\":\"Finance\",\"Environment\":\"Production\"}"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-tags-from-resource-group"></a>Aplicación de etiquetas del grupo de recursos

Para aplicar etiquetas desde un grupo de recursos a un recurso, use la función [resourceGroup()](../templates/template-functions-resource.md#resourcegroup). Cuando obtenga el valor de la etiqueta, use la sintaxis `tags[tag-name]` en lugar de la sintaxis `tags.tag-name`, porque algunos caracteres no se analizan correctamente en la notación de puntos.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "Dept": "[resourceGroup().tags['Dept']]",
                "Environment": "[resourceGroup().tags['Environment']]"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-tags-to-resource-groups-or-subscriptions"></a>Aplicación de etiquetas a grupos de recursos o suscripciones

Puede agregar etiquetas a un grupo de recursos o una suscripción mediante la implementación del tipo de recurso **Microsoft.Resources/tags**. Las etiquetas se aplican al grupo de recursos o a la suscripción de destino correspondiente a la implementación. Cada vez que implemente la plantilla, reemplace cualquier etiqueta que haya aplicado anteriormente.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "tagName": {
            "type": "string",
            "defaultValue": "TeamName"
        },
        "tagValue": {
            "type": "string",
            "defaultValue": "AppTeam1"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/tags",
            "name": "default",
            "apiVersion": "2019-10-01",
            "dependsOn": [],
            "properties": {
                "tags": {
                    "[parameters('tagName')]": "[parameters('tagValue')]"
                }
            }
        }
    ]
}
```

Para aplicar las etiquetas a un grupo de recursos, use PowerShell o la CLI de Azure. Implemente en el grupo de recursos que quiere etiquetar.

```azurepowershell-interactive
New-AzResourceGroupDeployment -ResourceGroupName exampleGroup -TemplateFile https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

```azurecli-interactive
az deployment group create --resource-group exampleGroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

Para aplicar las etiquetas a una suscripción, use PowerShell o la CLI de Azure. Implemente en la suscripción que quiere etiquetar.

```azurepowershell-interactive
New-AzSubscriptionDeployment -name tagresourcegroup -Location westus2 -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

```azurecli-interactive
az deployment sub create --name tagresourcegroup --location westus2 --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

Para más información sobre las implementaciones de suscripciones, consulte [Creación de grupos de recursos y otros recursos en el nivel de suscripción](../templates/deploy-to-subscription.md).

La plantilla siguiente agrega las etiquetas de un objeto a un grupo de recursos o a una suscripción.

```json
"$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "tags": {
            "type": "object",
            "defaultValue": {
                "TeamName": "AppTeam1",
                "Dept": "Finance",
                "Environment": "Production"
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/tags",
            "name": "default",
            "apiVersion": "2019-10-01",
            "dependsOn": [],
            "properties": {
                "tags": "[parameters('tags')]"
            }
        }
    ]
}
```

## <a name="portal"></a>Portal

[!INCLUDE [resource-manager-tag-resource](../../../includes/resource-manager-tag-resources.md)]

## <a name="rest-api"></a>API DE REST

Para trabajar con etiquetas a través de la API REST de Azure, use:

* [Etiquetas: crear o actualizar en el ámbito](/rest/api/resources/tags/createorupdateatscope) (operación PUT)
* [Etiquetas: actualizar en el ámbito](/rest/api/resources/tags/updateatscope) (operación PATCH)
* [Etiquetas: obtener en el ámbito](/rest/api/resources/tags/getatscope) (operación GET)
* [Etiquetas: eliminar en el ámbito](/rest/api/resources/tags/deleteatscope) (operación DELETE)

## <a name="inherit-tags"></a>Herencia de etiquetas

Los recursos no heredan las etiquetas aplicadas al grupo de recursos ni a la suscripción. Para aplicar etiquetas de un grupo de recursos o una suscripción a los recursos, consulte [Azure Policies: etiquetas](tag-policies.md).

## <a name="tags-and-billing"></a>Etiquetas y facturación

Puede usar etiquetas a fin de agrupar los datos de facturación. Por ejemplo, si va a ejecutar varias máquinas virtuales para distintas organizaciones, use las etiquetas para agrupar el uso por centro de costo. También puede usar etiquetas para clasificar los costos por entorno de tiempo de ejecución; por ejemplo, el uso de facturación en máquinas virtuales que se ejecutan en el entorno de producción.

Puede recuperar información sobre las etiquetas a través de [Azure Resource Usage API y Rate Card API](../../cost-management-billing/manage/usage-rate-card-overview.md) o mediante el archivo de valores separados por comas (CSV). Puede descargar el archivo de uso del [Centro de cuentas de Azure](https://account.azure.com/Subscriptions) o de Azure Portal. Para más información, consulte [Procedimiento para descargar las datos de uso diario y de factura de Azure](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md). Al descargar el archivo de uso del Centro de cuentas de Azure, seleccione **Versión 2**. En los servicios que admiten etiquetas con facturación, las etiquetas aparecen en la columna **Etiquetas**.

Para las operaciones de API de REST, vea [Referencia de API de REST de facturación de Azure](/rest/api/billing/).

## <a name="limitations"></a>Limitaciones

Se aplican las siguientes limitaciones a las etiquetas:

* No todos los tipos de recursos admiten etiquetas. Para determinar si se puede aplicar una etiqueta a un tipo de recurso determinado, consulte [Tag support for Azure resources](tag-support.md) (Compatibilidad con etiquetas para los recursos de Azure).
* Cada recurso, grupo de recursos y suscripción puede tener un máximo de 50 pares nombre-valor de etiqueta. Si necesita aplicar más etiquetas que el número máximo permitido, use una cadena JSON para el valor de etiqueta. La cadena JSON puede contener muchos valores que se aplican a un sol nombre de etiqueta. Un grupo de recursos o una suscripción puede contener muchos recursos con 50 pares clave-valor de etiqueta cada uno.
* El nombre de etiqueta está limitado a 512 caracteres y el valor de la etiqueta, a 256. En las cuentas de almacenamiento, el nombre de etiqueta se limita a 128 caracteres y el valor de la etiqueta, a 256.
* No se pueden aplicar etiquetas a recursos clásicos como Cloud Services.
* Los nombres de etiqueta no pueden contener estos caracteres: `<`, `>`, `%`, `&`, `\`, `?`, `/`

   > [!NOTE]
   > Actualmente, las zonas de Azure DNS y los servicios de Traffic Manager tampoco permiten el uso de espacios en la etiqueta.
   >
   > Azure Front Door no admite el uso de `#` en el nombre de la etiqueta.
   >
   > Azure Automation y Azure CDN solo admiten 15 etiquetas en los recursos.

## <a name="next-steps"></a>Pasos siguientes

* No todos los tipos de recursos admiten etiquetas. Para determinar si se puede aplicar una etiqueta a un tipo de recurso determinado, consulte [Tag support for Azure resources](tag-support.md) (Compatibilidad con etiquetas para los recursos de Azure).
* Para recomendaciones sobre cómo implementar una estrategia de etiquetado, consulte [Guía de decisiones de nomenclatura y etiquetado de recursos](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json).
