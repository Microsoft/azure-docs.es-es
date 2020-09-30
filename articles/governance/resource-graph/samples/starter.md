---
title: Ejemplos de consultas de inicio
description: Use Azure Resource Graph para ejecutar consultas de inicio, como el recuento de recursos, el pedido de recursos o las consultas por una etiqueta específica.
ms.date: 07/14/2020
ms.topic: sample
ms.openlocfilehash: 778e8bb7715f1eda1587ff79c4c9a99b69ab4e70
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89425269"
---
# <a name="starter-resource-graph-query-samples"></a>Ejemplos de consultas de inicio de Resource Graph

El primer paso para entender las consultas con Azure Resource Graph es el reconocimiento básico del [lenguaje de consultas](../concepts/query-language.md). Si aún no está familiarizado con el lenguaje de consulta [Kusto (KQL)](/azure/kusto/query/index), se recomienda repasar el [tutorial de KQL](/azure/kusto/query/tutorial) para entender cómo componer solicitudes para los recursos que está buscando.

Le guiaremos por las siguientes consultas de inicio:

- [Count Azure resources](#count-resources)
- [Recuento de recursos de Key Vault](#count-keyvaults)
- [List resources sorted by name](#list-resources)
- [Show all virtual machines ordered by name in descending order](#show-vms)
- [Show first five virtual machines by name and their OS type](#show-sorted)
- [Count virtual machines by OS type](#count-os)
- [Show resources that contain storage](#show-storage)
- [List all public IP addresses](#list-publicip)
- [Count resources that have IP addresses configured by subscription](#count-resources-by-ip)
- [List resources with a specific tag value](#list-tag)
- [List all storage accounts with specific tag value](#list-specific-tag)
- [Show aliases for a virtual machine resource](#show-aliases)
- [Show distinct values for a specific alias](#distinct-alias-values)
- [Mostrar grupos de seguridad de red no asociados](#unassociated-nsgs)
- [Obtención de un resumen de ahorro de costos de Azure Advisor](#advisor-savings)
- [Recuento de máquinas en el ámbito de las directivas de configuración de invitado](#count-gcmachines)

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

## <a name="language-support"></a>Compatibilidad con idiomas

La CLI de Azure (mediante una extensión) y Azure PowerShell (mediante un módulo) admiten Azure Resource Graph. Antes de ejecutar cualquiera de las siguientes consultas, compruebe que el entorno está listo. Consulte la [CLI de Azure](../first-query-azurecli.md#add-the-resource-graph-extension) y [Azure PowerShell](../first-query-powershell.md#add-the-resource-graph-module) para conocer los pasos para instalar y validar el entorno de shell que prefiera.

## <a name="count-azure-resources"></a><a name="count-resources"></a>Count Azure resources

Esta consulta devuelve el número de recursos de Azure que existen en las suscripciones a las que tiene acceso. También es una buena consulta para validar que el shell elegido tiene los componentes apropiados de Azure Resource Graph instalados y en funcionamiento.

```kusto
Resources
| summarize count()
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | summarize count()"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | summarize count()"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20summarize%20count%28%29" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20summarize%20count%28%29" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20summarize%20count%28%29" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="count-key-vault-resources"></a><a name="count-keyvaults"></a>Recuento de recursos de Key Vault

Esta consulta utiliza `count` en lugar de `summarize` para contar el número de registros devueltos. Solo se incluyen los almacenes de claves en el recuento.

```kusto
Resources
| where type =~ 'microsoft.keyvault/vaults'
| count
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.keyvault/vaults' | count"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.keyvault/vaults' | count"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.keyvault%2Fvaults%27%0D%0A%7C%20count" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.keyvault%2Fvaults%27%0D%0A%7C%20count" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.keyvault%2Fvaults%27%0D%0A%7C%20count" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="list-resources-sorted-by-name"></a><a name="list-resources"></a>List resources sorted by name

Esta consulta devuelve cualquier tipo de recurso, pero solo las propiedades **name** (nombre), **type** (tipo) y **location** (ubicación). Usa `order by` para ordenar las propiedades por la propiedad **name** en orden ascendente (`asc`).

```kusto
Resources
| project name, type, location
| order by name asc
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | project name, type, location | order by name asc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project name, type, location | order by name asc"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20name%2C%20type%2C%20location%0D%0A%7C%20order%20by%20name%20asc" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20name%2C%20type%2C%20location%0D%0A%7C%20order%20by%20name%20asc" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20name%2C%20type%2C%20location%0D%0A%7C%20order%20by%20name%20asc" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="show-all-virtual-machines-ordered-by-name-in-descending-order"></a><a name="show-vms"></a>Show all virtual machines ordered by name in descending order

Para enumerar solo las máquinas virtuales (que son de tipo `Microsoft.Compute/virtualMachines`), podemos hacer coincidir la propiedad **type** en los resultados. De forma similar a la consulta anterior, `desc` cambia el `order by` a descendente. `=~` en el tipo de coincidencia indica a Resource Graph que distinga mayúsculas de minúsculas.

```kusto
Resources
| project name, location, type
| where type =~ 'Microsoft.Compute/virtualMachines'
| order by name desc
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | project name, location, type| where type =~ 'Microsoft.Compute/virtualMachines' | order by name desc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project name, location, type| where type =~ 'Microsoft.Compute/virtualMachines' | order by name desc"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20name%2C%20location%2C%20type%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20order%20by%20name%20desc" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20name%2C%20location%2C%20type%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20order%20by%20name%20desc" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20name%2C%20location%2C%20type%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20order%20by%20name%20desc" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="show-first-five-virtual-machines-by-name-and-their-os-type"></a><a name="show-sorted"></a>Show first five virtual machines by name and their OS type

Esta consulta utilizará `top` para recuperar únicamente cinco registros coincidentes que se ordenan por nombre. El tipo de recurso de Azure es `Microsoft.Compute/virtualMachines`. `project` indica a Azure Resource Graph qué propiedades se incluirán.

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| project name, properties.storageProfile.osDisk.osType
| top 5 by name desc
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | project name, properties.storageProfile.osDisk.osType | top 5 by name desc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | project name, properties.storageProfile.osDisk.osType | top 5 by name desc"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20project%20name%2C%20properties.storageProfile.osDisk.osType%0D%0A%7C%20top%205%20by%20name%20desc" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20project%20name%2C%20properties.storageProfile.osDisk.osType%0D%0A%7C%20top%205%20by%20name%20desc" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20project%20name%2C%20properties.storageProfile.osDisk.osType%0D%0A%7C%20top%205%20by%20name%20desc" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="count-virtual-machines-by-os-type"></a><a name="count-os"></a>Count virtual machines by OS type

Basado en la consulta anterior, todavía se limita por los recursos de Azure de tipo `Microsoft.Compute/virtualMachines`, pero ya no se limita el número de registros devueltos.
En su lugar, hemos usado `summarize` y `count()` para definir cómo agrupar y agregar los valores por propiedad, que en este ejemplo es `properties.storageProfile.osDisk.osType`. Para un ejemplo del aspecto de esta cadena en el objeto completo, vea [explorar recursos: detección de máquinas virtuales](../concepts/explore-resources.md#virtual-machine-discovery).

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| summarize count() by tostring(properties.storageProfile.osDisk.osType)
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | summarize count() by tostring(properties.storageProfile.osDisk.osType)"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | summarize count() by tostring(properties.storageProfile.osDisk.osType)"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20summarize%20count%28%29%20by%20tostring%28properties.storageProfile.osDisk.osType%29" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20summarize%20count%28%29%20by%20tostring%28properties.storageProfile.osDisk.osType%29" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20summarize%20count%28%29%20by%20tostring%28properties.storageProfile.osDisk.osType%29" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

Otra forma de escribir la misma consulta es `extend` una propiedad y asignarle un nombre temporal para su uso dentro de la consulta, en este caso **os**. A continuación, `summarize` y `count()` usan **os** como en el ejemplo anterior.

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| extend os = properties.storageProfile.osDisk.osType
| summarize count() by tostring(os)
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | extend os = properties.storageProfile.osDisk.osType | summarize count() by tostring(os)"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | extend os = properties.storageProfile.osDisk.osType | summarize count() by tostring(os)"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20extend%20os%20%3D%20properties.storageProfile.osDisk.osType%0D%0A%7C%20summarize%20count%28%29%20by%20tostring%28os%29" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20extend%20os%20%3D%20properties.storageProfile.osDisk.osType%0D%0A%7C%20summarize%20count%28%29%20by%20tostring%28os%29" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20extend%20os%20%3D%20properties.storageProfile.osDisk.osType%0D%0A%7C%20summarize%20count%28%29%20by%20tostring%28os%29" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

> [!NOTE]
> Tenga en cuenta que mientras `=~` permite el uso de búsqueda de coincidencias sin distinguir mayúsculas de minúsculas, el uso de las propiedades (como **properties.storageProfile.osDisk.osType**) en la consulta distingue mayúsculas de minúsculas. Si la propiedad es el caso incorrecto, se devuelve un valor Null o incorrecto y la agrupación o el resumen serían incorrectos.

## <a name="show-resources-that-contain-storage"></a><a name="show-storage"></a>Show resources that contain storage

En lugar de definir explícitamente el tipo de coincidencia, esta consulta de ejemplo encontrará cualquier recurso de Azure que `contains` la palabra **storage**.

```kusto
Resources
| where type contains 'storage' | distinct type
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type contains 'storage' | distinct type"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type contains 'storage' | distinct type"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20contains%20%27storage%27%20%7C%20distinct%20type" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20contains%20%27storage%27%20%7C%20distinct%20type" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20contains%20%27storage%27%20%7C%20distinct%20type" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="list-all-public-ip-addresses"></a><a name="list-publicip"></a>List all public IP addresses

De forma similar a la consulta anterior, encontrará todos los tipos que contienen la palabra **publicIPAddresses**.
Esta consulta se expande en ese patrón para incluir solo los resultados donde **properties.ipAddress**
`isnotempty` es nulo para devolver solo **properties.ipAddress** y para definir `limit` para limitar los resultados a los primeros.
100. Es posible que sea necesario escapar las comillas según su shell elegido.

```kusto
Resources
| where type contains 'publicIPAddresses' and isnotempty(properties.ipAddress)
| project properties.ipAddress
| limit 100
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type contains 'publicIPAddresses' and isnotempty(properties.ipAddress) | project properties.ipAddress | limit 100"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type contains 'publicIPAddresses' and isnotempty(properties.ipAddress) | project properties.ipAddress | limit 100"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20contains%20%27publicIPAddresses%27%20and%20isnotempty%28properties.ipAddress%29%0D%0A%7C%20project%20properties.ipAddress%0D%0A%7C%20limit%20100" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20contains%20%27publicIPAddresses%27%20and%20isnotempty%28properties.ipAddress%29%0D%0A%7C%20project%20properties.ipAddress%0D%0A%7C%20limit%20100" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20contains%20%27publicIPAddresses%27%20and%20isnotempty%28properties.ipAddress%29%0D%0A%7C%20project%20properties.ipAddress%0D%0A%7C%20limit%20100" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="count-resources-that-have-ip-addresses-configured-by-subscription"></a><a name="count-resources-by-ip"></a>Count resources that have IP addresses configured by subscription

Con la consulta de ejemplo anterior y agregando `summarize` y `count()`, podemos obtener una lista por suscripción de recursos con direcciones IP configuradas.

```kusto
Resources
| where type contains 'publicIPAddresses' and isnotempty(properties.ipAddress)
| summarize count () by subscriptionId
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type contains 'publicIPAddresses' and isnotempty(properties.ipAddress) | summarize count () by subscriptionId"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type contains 'publicIPAddresses' and isnotempty(properties.ipAddress) | summarize count () by subscriptionId"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20contains%20%27publicIPAddresses%27%20and%20isnotempty%28properties.ipAddress%29%0D%0A%7C%20summarize%20count%20%28%29%20by%20subscriptionId" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20contains%20%27publicIPAddresses%27%20and%20isnotempty%28properties.ipAddress%29%0D%0A%7C%20summarize%20count%20%28%29%20by%20subscriptionId" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20contains%20%27publicIPAddresses%27%20and%20isnotempty%28properties.ipAddress%29%0D%0A%7C%20summarize%20count%20%28%29%20by%20subscriptionId" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="list-resources-with-a-specific-tag-value"></a><a name="list-tag"></a>List resources with a specific tag value

Podemos limitar los resultados por propiedades que no sean del tipo de recurso de Azure, como una etiqueta. En este ejemplo, se filtran los recursos de Azure con el nombre de etiqueta **Environment** que tienen un valor de **Internal**.

```kusto
Resources
| where tags.environment=~'internal'
| project name
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where tags.environment=~'internal' | project name"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where tags.environment=~'internal' | project name"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20tags.environment%3D~%27internal%27%0D%0A%7C%20project%20name" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20tags.environment%3D~%27internal%27%0D%0A%7C%20project%20name" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20tags.environment%3D~%27internal%27%0D%0A%7C%20project%20name" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

Para proporcionar también las etiquetas del recurso y sus valores, agregue la propiedad **tags** a la palabra clave `project`.

```kusto
Resources
| where tags.environment=~'internal'
| project name, tags
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where tags.environment=~'internal' | project name, tags"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where tags.environment=~'internal' | project name, tags"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20tags.environment%3D~%27internal%27%0D%0A%7C%20project%20name%2C%20tags" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20tags.environment%3D~%27internal%27%0D%0A%7C%20project%20name%2C%20tags" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20tags.environment%3D~%27internal%27%0D%0A%7C%20project%20name%2C%20tags" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="list-all-storage-accounts-with-specific-tag-value"></a><a name="list-specific-tag"></a>List all storage accounts with specific tag value

Combina la funcionalidad de filtro del ejemplo anterior y filtra el tipo de recurso de Azure por la propiedad **type**. Esta consulta también limita la búsqueda para determinados tipos de recursos de Azure con un valor y nombre de etiqueta específicos.

```kusto
Resources
| where type =~ 'Microsoft.Storage/storageAccounts'
| where tags['tag with a space']=='Custom value'
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Storage/storageAccounts' | where tags['tag with a space']=='Custom value'"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Storage/storageAccounts' | where tags['tag with a space']=='Custom value'"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Storage%2FstorageAccounts%27%0D%0A%7C%20where%20tags%5B%27tag%20with%20a%20space%27%5D%3D%3D%27Custom%20value%27" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Storage%2FstorageAccounts%27%0D%0A%7C%20where%20tags%5B%27tag%20with%20a%20space%27%5D%3D%3D%27Custom%20value%27" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Storage%2FstorageAccounts%27%0D%0A%7C%20where%20tags%5B%27tag%20with%20a%20space%27%5D%3D%3D%27Custom%20value%27" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

> [!NOTE]
> Este ejemplo utiliza `==` para buscar coincidencias en lugar del condicional `=~`. `==` es una coincidencia que distingue mayúsculas de minúsculas.

## <a name="show-aliases-for-a-virtual-machine-resource"></a><a name="show-aliases"></a>Visualización de los alias de un recurso de máquina virtual

Los [alias de Azure Policy](../../policy/concepts/definition-structure.md#aliases) los usa Azure Policy para administrar el cumplimiento de los recursos. Azure Resource Graph puede devolver los _alias_ de un tipo de recurso. Estos valores son útiles para comparar el valor actual de los alias al crear una definición de directiva personalizada. La matriz de _alias_ no se proporciona de forma predeterminada en los resultados de una consulta. Use `project aliases` para agregarla explícitamente a los resultados.

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| limit 1
| project aliases
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | limit 1 | project aliases"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | limit 1 | project aliases" | ConvertTo-Json
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20limit%201%0D%0A%7C%20project%20aliases" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20limit%201%0D%0A%7C%20project%20aliases" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20limit%201%0D%0A%7C%20project%20aliases" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="show-distinct-values-for-a-specific-alias"></a><a name="distinct-alias-values"></a>Visualización de los distintos valores de un alias específico

Ver el valor de los alias en un único recurso es útil, pero no muestra el verdadero valor del uso de Azure Resource Graph para consultar entre las suscripciones. Este ejemplo busca todos los valores de un alias específico y devuelve los valores distintos.

```kusto
Resources
| where type=~'Microsoft.Compute/virtualMachines'
| extend alias = aliases['Microsoft.Compute/virtualMachines/storageProfile.osDisk.managedDisk.storageAccountType']
| distinct tostring(alias)
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type=~'Microsoft.Compute/virtualMachines' | extend alias = aliases['Microsoft.Compute/virtualMachines/storageProfile.osDisk.managedDisk.storageAccountType'] | distinct tostring(alias)"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type=~'Microsoft.Compute/virtualMachines' | extend alias = aliases['Microsoft.Compute/virtualMachines/storageProfile.osDisk.managedDisk.storageAccountType'] | distinct tostring(alias)"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%3D~%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20extend%20alias%20%3D%20aliases%5B%27Microsoft.Compute%2FvirtualMachines%2FstorageProfile.osDisk.managedDisk.storageAccountType%27%5D%0D%0A%7C%20distinct%20tostring%28alias%29" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%3D~%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20extend%20alias%20%3D%20aliases%5B%27Microsoft.Compute%2FvirtualMachines%2FstorageProfile.osDisk.managedDisk.storageAccountType%27%5D%0D%0A%7C%20distinct%20tostring%28alias%29" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%3D~%27Microsoft.Compute%2FvirtualMachines%27%0D%0A%7C%20extend%20alias%20%3D%20aliases%5B%27Microsoft.Compute%2FvirtualMachines%2FstorageProfile.osDisk.managedDisk.storageAccountType%27%5D%0D%0A%7C%20distinct%20tostring%28alias%29" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="show-unassociated-network-security-groups"></a><a name="unassociated-nsgs"></a>Visualización de los grupos de seguridad de red sin asociar

Esta consulta devuelve los grupos de seguridad de red que no están asociados a una interfaz de red o subred.

```kusto
Resources
| where type =~ "microsoft.network/networksecuritygroups" and isnull(properties.networkInterfaces) and isnull(properties.subnets)
| project name, resourceGroup
| sort by name asc
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.network/networksecuritygroups' and isnull(properties.networkInterfaces) and isnull(properties.subnets) | project name, resourceGroup | sort by name asc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.network/networksecuritygroups' and isnull(properties.networkInterfaces) and isnull(properties.subnets) | project name, resourceGroup | sort by name asc"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%22microsoft.network%2Fnetworksecuritygroups%22%20and%20isnull%28properties.networkInterfaces%29%20and%20isnull%28properties.subnets%29%0D%0A%7C%20project%20name%2C%20resourceGroup%0D%0A%7C%20sort%20by%20name%20asc" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%22microsoft.network%2Fnetworksecuritygroups%22%20and%20isnull%28properties.networkInterfaces%29%20and%20isnull%28properties.subnets%29%0D%0A%7C%20project%20name%2C%20resourceGroup%0D%0A%7C%20sort%20by%20name%20asc" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%22microsoft.network%2Fnetworksecuritygroups%22%20and%20isnull%28properties.networkInterfaces%29%20and%20isnull%28properties.subnets%29%0D%0A%7C%20project%20name%2C%20resourceGroup%0D%0A%7C%20sort%20by%20name%20asc" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="get-cost-savings-summary-from-azure-advisor"></a><a name="advisor-savings"></a>Obtención de un resumen de ahorro de costos de Azure Advisor

Esta consulta resume el ahorro de costos de cada recomendación de [Azure Advisor](../../../advisor/advisor-overview.md).

```kusto
advisorresources
| where type == 'microsoft.advisor/recommendations'
| where properties.category == 'Cost'
| extend
    resources = tostring(properties.resourceMetadata.resourceId),
    savings = todouble(properties.extendedProperties.savingsAmount),
    solution = tostring(properties.shortDescription.solution),
    currency = tostring(properties.extendedProperties.savingsCurrency)
| summarize
    dcount(resources), 
    bin(sum(savings), 0.01)
    by solution, currency
| project solution, dcount_resources, sum_savings, currency
| order by sum_savings desc
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "advisorresources | where type == 'microsoft.advisor/recommendations' | where properties.category == 'Cost' | extend resources = tostring(properties.resourceMetadata.resourceId), savings = todouble(properties.extendedProperties.savingsAmount), solution = tostring(properties.shortDescription.solution), currency = tostring(properties.extendedProperties.savingsCurrency) | summarize dcount(resources), bin(sum(savings), 0.01) by solution, currency | project solution, dcount_resources, sum_savings, currency | order by sum_savings desc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "advisorresources | where type == 'microsoft.advisor/recommendations' | where properties.category == 'Cost' | extend resources = tostring(properties.resourceMetadata.resourceId), savings = todouble(properties.extendedProperties.savingsAmount), solution = tostring(properties.shortDescription.solution), currency = tostring(properties.extendedProperties.savingsCurrency) | summarize dcount(resources), bin(sum(savings), 0.01) by solution, currency | project solution, dcount_resources, sum_savings, currency | order by sum_savings desc"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/advisorresources%0D%0A%7C%20where%20type%20%3D%3D%20%27microsoft.advisor%2Frecommendations%27%0D%0A%7C%20where%20properties.category%20%3D%3D%20%27Cost%27%0D%0A%7C%20extend%0D%0A%20%20%20%20resources%20%3D%20tostring%28properties.resourceMetadata.resourceId%29%2C%0D%0A%20%20%20%20savings%20%3D%20todouble%28properties.extendedProperties.savingsAmount%29%2C%0D%0A%20%20%20%20solution%20%3D%20tostring%28properties.shortDescription.solution%29%2C%0D%0A%20%20%20%20currency%20%3D%20tostring%28properties.extendedProperties.savingsCurrency%29%0D%0A%7C%20summarize%0D%0A%20%20%20%20dcount%28resources%29%2C%20%0D%0A%20%20%20%20bin%28sum%28savings%29%2C%200.01%29%0D%0A%20%20%20%20by%20solution%2C%20currency%0D%0A%7C%20project%20solution%2C%20dcount_resources%2C%20sum_savings%2C%20currency%0D%0A%7C%20order%20by%20sum_savings%20desc" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure Government: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/advisorresources%0D%0A%7C%20where%20type%20%3D%3D%20%27microsoft.advisor%2Frecommendations%27%0D%0A%7C%20where%20properties.category%20%3D%3D%20%27Cost%27%0D%0A%7C%20extend%0D%0A%20%20%20%20resources%20%3D%20tostring%28properties.resourceMetadata.resourceId%29%2C%0D%0A%20%20%20%20savings%20%3D%20todouble%28properties.extendedProperties.savingsAmount%29%2C%0D%0A%20%20%20%20solution%20%3D%20tostring%28properties.shortDescription.solution%29%2C%0D%0A%20%20%20%20currency%20%3D%20tostring%28properties.extendedProperties.savingsCurrency%29%0D%0A%7C%20summarize%0D%0A%20%20%20%20dcount%28resources%29%2C%20%0D%0A%20%20%20%20bin%28sum%28savings%29%2C%200.01%29%0D%0A%20%20%20%20by%20solution%2C%20currency%0D%0A%7C%20project%20solution%2C%20dcount_resources%2C%20sum_savings%2C%20currency%0D%0A%7C%20order%20by%20sum_savings%20desc" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Portal de Azure China 21Vianet: <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/advisorresources%0D%0A%7C%20where%20type%20%3D%3D%20%27microsoft.advisor%2Frecommendations%27%0D%0A%7C%20where%20properties.category%20%3D%3D%20%27Cost%27%0D%0A%7C%20extend%0D%0A%20%20%20%20resources%20%3D%20tostring%28properties.resourceMetadata.resourceId%29%2C%0D%0A%20%20%20%20savings%20%3D%20todouble%28properties.extendedProperties.savingsAmount%29%2C%0D%0A%20%20%20%20solution%20%3D%20tostring%28properties.shortDescription.solution%29%2C%0D%0A%20%20%20%20currency%20%3D%20tostring%28properties.extendedProperties.savingsCurrency%29%0D%0A%7C%20summarize%0D%0A%20%20%20%20dcount%28resources%29%2C%20%0D%0A%20%20%20%20bin%28sum%28savings%29%2C%200.01%29%0D%0A%20%20%20%20by%20solution%2C%20currency%0D%0A%7C%20project%20solution%2C%20dcount_resources%2C%20sum_savings%2C%20currency%0D%0A%7C%20order%20by%20sum_savings%20desc" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="count-machines-in-scope-of-guest-configuration-policies"></a><a name="count-gcmachines"></a>Recuento de máquinas en el ámbito de las directivas de configuración de invitado

Se muestra el recuento de máquinas virtuales de Azure y los servidores conectados de Arc en el ámbito para las asignaciones de [configuración de invitado de Azure Policy](../../policy/concepts/guest-configuration.md).

```kusto
GuestConfigurationResources
| extend vmid = split(properties.targetResourceId,'/')
| mvexpand properties.latestAssignmentReport.resources
| where properties_latestAssignmentReport_resources.resourceId != 'Invalid assignment package.'
| project machine = tostring(vmid[(-1)]),
    type = tostring(vmid[(-3)])
| distinct machine, type
| summarize count() by type
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az graph query -q "GuestConfigurationResources | extend vmid = split(properties.targetResourceId,'/') | mvexpand properties.latestAssignmentReport.resources | where properties_latestAssignmentReport_resources.resourceId != 'Invalid assignment package.' | project machine = tostring(vmid[(-1)]), type = tostring(vmid[(-3)]) | distinct machine, type | summarize count() by type"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "GuestConfigurationResources | extend vmid = split(properties.targetResourceId,'/') | mvexpand properties.latestAssignmentReport.resources | where properties_latestAssignmentReport_resources.resourceId != 'Invalid assignment package.' | project machine = tostring(vmid[(-1)]), type = tostring(vmid[(-3)]) | distinct machine, type | summarize count() by type"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::Pruebe a ejecutar esta consulta en el Explorador de Azure Resource Graph:

- Azure Portal: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/GuestConfigurationResources%20%7C%20extend%20vmid%20%3D%20split(properties.targetResourceId%2C%22%2F%22)%20%7C%20mvexpand%20properties.latestAssignmentReport.resources%20%7C%20where%20properties_latestAssignmentReport_resources.resourceId%20!%3D%20'Invalid%20assignment%20package.'%20%7C%20project%20machine%20%3D%20tostring(vmid%5B(-1)%5D)%2C%20type%20%3D%20tostring(vmid%5B(-3)%5D)%20%7C%20distinct%20machine%2C%20type%20%7C%20summarize%20count()%20by%20type" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre el [lenguaje de consulta](../concepts/query-language.md).
- Obtenga más información sobre cómo [explorar recursos](../concepts/explore-resources.md).
- Consulte ejemplos de [consultas avanzadas](advanced.md).