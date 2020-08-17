---
title: Descripción del lenguaje de consultas
description: Describe las tablas de Resource Graph y los tipos de datos, los operadores y las funciones de Kusto disponibles que se pueden usar con Azure Resource Graph.
ms.date: 08/03/2020
ms.topic: conceptual
ms.openlocfilehash: b59811ecd877b9b2e22a43c00329ed7d02dfb97d
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87541828"
---
# <a name="understanding-the-azure-resource-graph-query-language"></a>Información del lenguaje de consulta de Azure Resource Graph

El lenguaje de consulta de Azure Resource Graph admite numerosos operadores y funciones. Cada uno funciona en función del [lenguaje de consulta de Kusto (KQL)](/azure/kusto/query/index). Para información sobre el lenguaje de consulta usado por Resource Graph, comience con el [tutorial para KQL](/azure/kusto/query/tutorial).

En este artículo se tratan los componentes de idioma admitidos por Resource Graph:

- [Tablas de Resource Graph](#resource-graph-tables)
- [Elementos del lenguaje personalizado de Resource Graph](#resource-graph-custom-language-elements)
- [Elementos del lenguaje KQL admitidos](#supported-kql-language-elements)
- [Ámbito de la consulta](#query-scope)
- [Caracteres de escape](#escape-characters)

## <a name="resource-graph-tables"></a>Tablas de Resource Graph

Resource Graph proporciona varias tablas para los datos que almacena sobre los tipos de recursos de Azure Resource Manager y sus propiedades. Estas tablas se pueden utilizar con los operadores `join` o `union` para obtener propiedades de tipos de recursos relacionados. Esta es la lista de tablas disponibles en Resource Graph:

|Tablas de Resource Graph |Descripción |
|---|---|
|Recursos |La tabla predeterminada, si no se ha definido ninguna en la consulta. La mayoría de los tipos de recursos y propiedades de Resource Manager están aquí. |
|ResourceContainers |Incluye los datos y los tipos de recursos de suscripción (en versión preliminar: `Microsoft.Resources/subscriptions`) y grupo de recursos (`Microsoft.Resources/subscriptions/resourcegroups`). |
|AdvisorResources |Incluye recursos _relacionados_ con `Microsoft.Advisor`. |
|AlertsManagementResources |Incluye recursos _relacionados_ con `Microsoft.AlertsManagement`. |
|HealthResources |Incluye recursos _relacionados_ con `Microsoft.ResourceHealth`. |
|MaintenanceResources |Incluye recursos _relacionados_ con `Microsoft.Maintenance`. |
|SecurityResources |Incluye recursos _relacionados_ con `Microsoft.Security`. |

Para obtener una lista completa, incluidos los tipos de recursos, consulte [Referencia: tablas y tipos de recursos admitidos](../reference/supported-tables-resources.md).

> [!NOTE]
> _Recursos_ es la tabla predeterminada. Al consultar la tabla _Recursos_, no es necesario proporcionar el nombre de la tabla a menos que se utilicen `join` o `union`. Sin embargo, el procedimiento recomendado es incluir siempre la tabla inicial en la consulta.

Use Azure Resource Graph en el portal para detectar qué tipos de recursos están disponibles en cada tabla. Como alternativa, utilice una consulta como `<tableName> | distinct type` para obtener una lista de los tipos de recursos que admite la tabla de Resource Graph que existe en su entorno.

En la consulta siguiente se muestra un uso sencillo de `join`. El resultado de la consulta combina las columnas y los nombres de columna duplicados de la tabla combinada, _ResourceContainers_ en este ejemplo, se anexan con **1**. Como la tabla _ResourceContainers_ tiene tipos para suscripciones y grupos de recursos, se puede usar cualquier tipo para unirse al recurso desde la tabla _recursos_.

```kusto
Resources
| join ResourceContainers on subscriptionId
| limit 1
```

En la consulta siguiente se muestra un uso más complejo de `join`. La consulta limita la tabla combinada a los recursos de suscripciones y con `project` para incluir solo el campo original _subscriptionId_ y el campo _name_ con el nombre cambiado a _SubName_. El cambio de nombre del campo evita `join` se agregue como _name1_, ya que el campo ya existe en _Resources_ (Recursos). La tabla original se filtra por `where` y el `project` siguiente incluye columnas de ambas tablas. El resultado de la consulta es un solo almacén de claves que muestra su tipo, su nombre y el nombre de la suscripción en que se encuentra.

```kusto
Resources
| where type == 'microsoft.keyvault/vaults'
| join (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId
| project type, name, SubName
| limit 1
```

> [!NOTE]
> Al limitar los resultados de `join` con `project`, la propiedad utilizada por `join` para relacionar las dos tablas, _subscriptionId_ en el ejemplo anterior, debe incluirse en `project`.

## <a name="resource-graph-custom-language-elements"></a>Elementos del lenguaje personalizado de Resource Graph

### <a name="shared-query-syntax-preview"></a><a name="shared-query-syntax"></a>Sintaxis de consulta compartida (versión preliminar)

Como característica en vista previa, se puede acceder directamente a una [consulta compartida](../tutorials/create-share-query.md) en una consulta de Resource Graph. Este escenario permite crear consultas estándar como consultas compartidas, así como reutilizarlas. Para llamar a una consulta compartida dentro de una consulta de Resource Graph, use la sintaxis `{{shared-query-uri}}`. El URI de la consulta compartida es el _identificador de recurso_ de la consulta compartida en la página **Configuración** de dicha consulta. En este ejemplo, el URI de la consulta compartida es `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS`.
Este URI apunta a la suscripción, al grupo de recursos y al nombre completo de la consulta compartida a la que se desea hacer referencia en otra consulta. Esta consulta es la misma que la creada en [Tutorial: Creación y uso compartido de una consulta](../tutorials/create-share-query.md).

> [!NOTE]
> No se puede guardar como consulta compartida una consulta que haga referencia a una consulta compartida.

Ejemplo 1: Solo de la consulta compartida únicamente

Los resultados de esta consulta de Resource Graph son los mismos que los de la consulta almacenada en la consulta compartida.

```kusto
{{/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS}}
```

Ejemplo 2: Inclusión de la consulta compartida como parte de una mayor

En primer lugar, esta consulta usa la consulta compartida y, a continuación, usa `limit` para restringir aún más los resultados.

```kusto
{{/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS}}
| where properties_storageProfile_osDisk_osType =~ 'Windows'
```

## <a name="supported-kql-language-elements"></a>Elementos del lenguaje KQL admitidos

Resource Graph admite todos los [tipos de datos](/azure/kusto/query/scalar-data-types/), [funciones escalares](/azure/kusto/query/scalarfunctions), [operadores escalares](/azure/kusto/query/binoperators) y [funciones de agregado](/azure/kusto/query/any-aggfunction) de KQL. Resource Graph admite [operadores tabulares](/azure/kusto/query/queries) específicos, algunos de los cuales tienen comportamientos distintos.

### <a name="supported-tabulartop-level-operators"></a>Operadores de nivel superior o tabulares compatibles

Esta es la lista de operadores tabulares de KQL admitidos por Resource Graph con ejemplos específicos:

|KQL |Consulta de ejemplo de Resource Graph |Notas |
|---|---|---|
|[count](/azure/kusto/query/countoperator) |[Contador de almacenes de claves](../samples/starter.md#count-keyvaults) | |
|[distinct](/azure/kusto/query/distinctoperator) |[Show distinct values for a specific alias](../samples/starter.md#distinct-alias-values) | |
|[extend](/azure/kusto/query/extendoperator) |[Count virtual machines by OS type](../samples/starter.md#count-os) | |
|[join](/azure/kusto/query/joinoperator) |[Almacén de claves con el nombre de la suscripción](../samples/advanced.md#join) |Tipos de combinación admitidos: [innerunique](/azure/kusto/query/joinoperator#default-join-flavor), [inner](/azure/kusto/query/joinoperator#inner-join), [leftouter](/azure/kusto/query/joinoperator#left-outer-join). Límite de 3 `join` en una sola consulta. No se permiten las estrategias de combinación personalizadas, como la combinación de difusión. Se puede usar en una sola tabla o entre las tablas _Resources_ y _ResourceContainers_. |
|[limit](/azure/kusto/query/limitoperator) |[List all public IP addresses](../samples/starter.md#list-publicip) |Sinónimo de `take` |
|[mvexpand](/azure/kusto/query/mvexpandoperator) | | Operador heredado, use `mv-expand` en su lugar. Máximo de _RowLimit_ de 400. El valor predeterminado es 128. |
|[mv-expand](/azure/kusto/query/mvexpandoperator) |[Enumeración de Cosmos DB con ubicaciones de escritura concretas](../samples/advanced.md#mvexpand-cosmosdb) |Máximo de _RowLimit_ de 400. El valor predeterminado es 128. |
|[order](/azure/kusto/query/orderoperator) |[List resources sorted by name](../samples/starter.md#list-resources) |Sinónimo de `sort` |
|[project](/azure/kusto/query/projectoperator) |[List resources sorted by name](../samples/starter.md#list-resources) | |
|[project-away](/azure/kusto/query/projectawayoperator) |[Eliminación de columnas de los resultados](../samples/advanced.md#remove-column) | |
|[sort](/azure/kusto/query/sortoperator) |[List resources sorted by name](../samples/starter.md#list-resources) |Sinónimo de `order` |
|[summarize](/azure/kusto/query/summarizeoperator) |[Count Azure resources](../samples/starter.md#count-resources) |Solo la primera página simplificada |
|[take](/azure/kusto/query/takeoperator) |[List all public IP addresses](../samples/starter.md#list-publicip) |Sinónimo de `limit` |
|[top](/azure/kusto/query/topoperator) |[Show first five virtual machines by name and their OS type](../samples/starter.md#show-sorted) | |
|[union](/azure/kusto/query/unionoperator) |[Combinación de los resultados de dos consultas para formar un solo resultado](../samples/advanced.md#unionresults) |Tabla única permitida: _T_ `| union` \[`kind=` `inner`\|`outer`\] \[`withsource=`_nombreDeColumna_\] _tabla_. Límite de 3 partes de `union` en una sola consulta. No se permite la resolución aproximada de tablas de segmento `union`. Se puede usar en una sola tabla o entre las tablas _Resources_ y _ResourceContainers_. |
|[where](/azure/kusto/query/whereoperator) |[Show resources that contain storage](../samples/starter.md#show-storage) | |

## <a name="query-scope"></a>Ámbito de la consulta

El ámbito de las suscripciones desde las que una consulta devuelve los recursos depende del método de acceso a Resource Graph. La CLI de Azure y Azure PowerShell rellenan la lista de suscripciones que se van a incluir en la solicitud, en función del contexto del usuario autorizado. Cada lista de suscripciones se puede definir manualmente con las **suscripciones** y los parámetros de **suscripción**, respectivamente.
En la API de REST y en todos los demás SDK, la lista de suscripciones en las que se incluyen los recursos debe definirse explícitamente como parte de la solicitud.

Como **versión preliminar**, la versión de API de REST `2020-04-01-preview` agrega una propiedad para limitar el ámbito de la consulta a un [grupo de administración](../../management-groups/overview.md). Esta API de versión preliminar también hace que la propiedad de suscripción sea opcional. Si no se define el grupo de administración o la lista de suscripciones, el ámbito de la consulta comprende todos los recursos a los que puede tener acceso el usuario autenticado. La nueva propiedad `managementGroupId` toma el id. del grupo de administración, que es diferente del nombre del grupo de administración.
Cuando se especifica `managementGroupId`, se incluyen los recursos de las primeras 5000 suscripciones de la jerarquía de grupos de administración especificada. `managementGroupId` no se puede usar al mismo tiempo que `subscriptions`.

Ejemplo: Consulte todos los recursos de la jerarquía del grupo de administración denominado "Mi grupo de administración" que cuenta con el id. "myMG".

- URI DE LA API REST

  ```http
  POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2020-04-01-preview
  ```

- Cuerpo de la solicitud

  ```json
  {
      "query": "Resources | summarize count()",
      "managementGroupId": "myMG"
  }
  ```

## <a name="escape-characters"></a>Carácter de escape

Algunos nombres de propiedad, como los que incluyen `.` o `$`, debe encapsularse o escapar en la consulta para que se interprete correctamente; de lo contrario, no proporcionan los resultados previstos.

- `.` -Encapsule el nombre de propiedad como en: `['propertyname.withaperiod']`
  
  Consulta de ejemplo que encapsula la propiedad _odata.type_:

  ```kusto
  where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.['odata.type']
  ```

- `$` -Incluya un carácter de escape en el nombre de propiedad. El carácter de escape utilizado depende el shell desde donde se ejecute Resource Graph.

  - **bash** - `\`

    Consulta de ejemplo con escape de la propiedad _\$type_ en bash:

    ```kusto
    where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.\$type
    ```

  - **cmd** -no escapa el carácter `$`.

  - **PowerShell** - ``` ` ```

    Consulta de ejemplo con escape de la propiedad _\$type_ en PowerShell:

    ```kusto
    where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.`$type
    ```

## <a name="next-steps"></a>Pasos siguientes

- Vea el lenguaje en uso en[Consultas básicas](../samples/starter.md).
- Vea los usos avanzados en [Consultas avanzadas](../samples/advanced.md).
- Obtenga más información sobre cómo [explorar recursos](explore-resources.md).