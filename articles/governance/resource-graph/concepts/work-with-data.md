---
title: Trabajo con grandes conjuntos de datos
description: Aprenda a obtener, paginar, omitir y aplicar formato a registros de grandes conjuntos de datos mientras trabaja con Azure Resource Graph.
ms.date: 08/10/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 5f3073986e424c641d884e1c2427d3d519658d37
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89005945"
---
# <a name="working-with-large-azure-resource-data-sets"></a>Trabajo con grandes conjuntos de datos de recursos de Azure

Azure Resource Graph está diseñado para trabajar con el entorno de Azure y obtener información sobre los recursos que contiene. Resource Graph permite obtener estos datos de manera más rápida, incluso cuando se consultan miles de registros. Resource Graph presenta varias opciones de funcionamiento con estos grandes conjuntos de datos.

Para obtener una guía sobre cómo trabajar con consultas con mucha frecuencia, vea [Guía para solicitudes limitadas](./guidance-for-throttled-requests.md).

## <a name="data-set-result-size"></a>Tamaño de resultados del conjunto de datos

De forma predeterminada, en Resource Graph hay un límite en el número de registros que se pueden devolver en una consulta, y son **100** registros. Este control protege tanto al usuario como al servicio de consultas no intencionadas que darían lugar a grandes conjuntos de datos. Este caso suele suceder cuando un cliente experimenta con consultas para buscar y filtrar los recursos de la manera en que se adapte a sus necesidades concretas. Este control es diferente que usar los operadores de lenguaje [top](/azure/kusto/query/topoperator) o [limit](/azure/kusto/query/limitoperator) de Azure Data Explorer para limitar los resultados.

> [!NOTE]
> Al usar la opción **First**, se recomienda ordenar los resultados con al menos una columna con `asc` o `desc`. Si los resultados no están ordenados, los resultados devueltos son aleatorios y no se pueden repetir.

El límite predeterminado se puede invalidar mediante todos los métodos de interacción con Resource Graph. En los ejemplos siguientes se muestra cómo cambiar el límite de tamaño del conjunto de datos a _200_:

```azurecli-interactive
az graph query -q "Resources | project name | order by name asc" --first 200 --output table
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project name | order by name asc" -First 200
```

En la [API REST](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources), el control es **$top** y forma parte de **QueryRequestOptions**.

El control que sea _más restrictivo_ ganará. Por ejemplo, si la consulta usa los operadores **top** o **limit** y el resultado va a ser un número de registros superior a **First**, el número máximo de registros devueltos será igual a **First**. Igualmente, si **top** o **limit** son más pequeños que **First**, el conjunto de registros devuelto sería el valor más pequeño que configure **top** o **limit**.

**First** actualmente tiene un valor máximo permitido de _5000_, que lo alcanza mediante la [paginación de resultados](#paging-results) de _1000_ registros a la vez.

> [!IMPORTANT]
> Cuando la opción **First** está configurada para ser mayor que _1000_ registros, la consulta debe **proyectar** el campo **id** para que funcione la paginación. Si no se encuentra en la consulta, la respuesta no se [paginará](#paging-results) y los resultados se limitan a _1000_ registros.

## <a name="skipping-records"></a>Omisión de registros

La siguiente opción para trabajar con grandes conjuntos de datos es el control **Skip**. Este control permite que la consulta salte u omita el número definido de registros antes de devolver los resultados. **Skip** es útil con consultas que ordenan los resultados de una manera significativa donde la intención es obtener los recursos que se encuentran hacia la mitad del conjunto de resultados. Si los resultados necesarios están al final del conjunto de datos devuelto, es mejor usar una configuración de ordenación diferente y recuperar los resultados del principio del conjunto de datos.

> [!NOTE]
> Al usar la opción **Skip**, se recomienda ordenar los resultados con al menos una columna con `asc` o `desc`. Si los resultados no están ordenados, los resultados devueltos son aleatorios y no se pueden repetir.

En los ejemplos siguientes se muestra cómo omitir los primeros _10_ registros que devolvería una consulta, en lugar de comenzar el conjunto de resultados devuelto con el registro número 11:

```azurecli-interactive
az graph query -q "Resources | project name | order by name asc" --skip 10 --output table
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project name | order by name asc" -Skip 10
```

En la [API REST](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources), el control es **$skip** y forma parte de **QueryRequestOptions**.

## <a name="paging-results"></a>Paginación de resultados

Cuando sea necesario dividir un conjunto de resultados en conjuntos de registros más pequeños para su procesamiento, o porque un conjunto de resultados superaría el valor máximo permitido de _1000_ registros devueltos, use la paginación. La [API REST](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources)
**QueryResponse** proporciona valores para indicar que un conjunto de resultados se ha dividido: **resultTruncated** y **$skipToken**. **resultTruncated** es un valor booleano que informa al consumidor si existen registros adicionales no devueltos en la respuesta. Esta condición también se puede identificar cuando la propiedad **count** es menor que la propiedad **totalRecords**. **totalRecords** define cuántos registros coinciden con la consulta.

 **resultTruncated** es **true** cuando la paginación está deshabilitada o no es posible debido a que no hay columna `id` o cuando hay menos recursos disponibles de los que una consulta está solicitando. Cuando **resultTruncated** es **true**, la propiedad **$skipToken** no se establece.

En los ejemplos siguientes se muestra cómo **omitir** los primeros 3000 registros y cómo devolver los **primeros** 1000 registros después de los registros omitidos con la CLI de Azure y Azure PowerShell:

```azurecli-interactive
az graph query -q "Resources | project id, name | order by id asc" --first 1000 --skip 3000
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project id, name | order by id asc" -First 1000 -Skip 3000
```

> [!IMPORTANT]
> La consulta debe **proyectar** el campo **id** para que la paginación funcione. Si no aparece en la consulta, la respuesta no incluirá **$skipToken**.

Para obtener un ejemplo, consulte [Consulta de página siguiente](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources#next-page-query) en la documentación de la API REST.

## <a name="formatting-results"></a>Resultados de formato

Los resultados de una consulta de Resource Graph se proporcionan en dos formatos, _Table_ y _ObjectArray_. El formato se configura con el parámetro **resultFormat** como parte de las opciones de solicitud. El formato _Table_ es el valor predeterminado para **resultFormat**.

De forma predeterminada, los resultados de la CLI de Azure se proporcionan en JSON. Los resultados en Azure PowerShell son un objeto **PSCustomObject** de forma predeterminada, pero se pueden convertir rápidamente a JSON mediante el cmdlet `ConvertTo-Json`. En el caso de otros SDK, los resultados de la consulta pueden configurarse para generar el formato _ObjectArray_.

### <a name="format---table"></a>Formato: Table

El formato predeterminado, _Table_, devuelve resultados en un formato JSON diseñado para resaltar el diseño de columna y los valores de fila de las propiedades que devuelve la consulta. Este formato se parece mucho a los datos definidos en una hoja de cálculo o tabla estructurada con las columnas identificadas primero y, después, cada fila que representa datos alineada con esas columnas.

Este es un ejemplo del resultado de una consulta con el formato _Table_:

```json
{
    "totalRecords": 47,
    "count": 1,
    "data": {
        "columns": [{
                "name": "name",
                "type": "string"
            },
            {
                "name": "type",
                "type": "string"
            },
            {
                "name": "location",
                "type": "string"
            },
            {
                "name": "subscriptionId",
                "type": "string"
            }
        ],
        "rows": [
            [
                "veryscaryvm2-nsg",
                "microsoft.network/networksecuritygroups",
                "eastus",
                "11111111-1111-1111-1111-111111111111"
            ]
        ]
    },
    "facets": [],
    "resultTruncated": "true"
}
```

### <a name="format---objectarray"></a>Formato: ObjectArray

El formato _ObjectArray_ también devuelve resultados en formato JSON, pero este diseño se alinea con la relación de par clave-valor común en JSON, donde los datos de columna y fila se relacionan en grupos de matrices.

Este es un ejemplo del resultado de una consulta con el formato _ObjectArray_:

```json
{
    "totalRecords": 47,
    "count": 1,
    "data": [{
        "name": "veryscaryvm2-nsg",
        "type": "microsoft.network/networksecuritygroups",
        "location": "eastus",
        "subscriptionId": "11111111-1111-1111-1111-111111111111"
    }],
    "facets": [],
    "resultTruncated": "true"
}
```

Aquí se muestran ejemplos al establecer **resultFormat** para usar el formato _ObjectArray_:

```csharp
var requestOptions = new QueryRequestOptions( resultFormat: ResultFormat.ObjectArray);
var request = new QueryRequest(subscriptions, "Resources | limit 1", options: requestOptions);
```

```python
request_options = QueryRequestOptions(
    result_format=ResultFormat.object_array
)
request = QueryRequest(query="Resources | limit 1", subscriptions=subs_list, options=request_options)
response = client.resources(request)
```

## <a name="next-steps"></a>Pasos siguientes

- Vea el lenguaje en uso en[Consultas básicas](../samples/starter.md).
- Vea los usos avanzados en [Consultas avanzadas](../samples/advanced.md).
- Obtenga más información sobre cómo [explorar recursos](explore-resources.md).