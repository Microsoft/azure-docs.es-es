---
title: Referencia de filter de OData
titleSuffix: Azure Cognitive Search
description: Referencia del lenguaje OData y sintaxis completa que se usa para crear expresiones de filtro en consultas de Azure Cognitive Search.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
translation.priority.mt:
- de-de
- es-es
- fr-fr
- it-it
- ja-jp
- ko-kr
- pt-br
- ru-ru
- zh-cn
- zh-tw
ms.openlocfilehash: 0f33b5a28d7c83be7e546c3f61bc517047c51312
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934861"
---
# <a name="odata-filter-syntax-in-azure-cognitive-search"></a>Sintaxis de $filter de OData en Azure Cognitive Search

En Azure Cognitive Search se usan [expresiones de filtro de OData](query-odata-filter-orderby-syntax.md) para aplicar criterios adicionales a una consulta de búsqueda, además de los términos de búsqueda de texto completo. En este artículo se describe con detalle la sintaxis de los filtros. Para información general sobre qué son los filtros y cómo usarlos para desarrollar escenarios de consulta específicos, consulte [Filtros de Azure Cognitive Search](search-filters.md).

## <a name="syntax"></a>Sintaxis

En el lenguaje OData, un filtro es una expresión booleana, que a su vez puede ser uno de los distintos tipos de expresión, como se muestra en la siguiente EBNF ([forma de Backus-Naur extendida](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)):

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
boolean_expression ::=
    collection_filter_expression
    | logical_expression
    | comparison_expression
    | boolean_literal
    | boolean_function_call
    | '(' boolean_expression ')'
    | variable

/* This can be a range variable in the case of a lambda, or a field path. */
variable ::= identifier | field_path
```

También está disponible un diagrama de sintaxis interactivo:

> [!div class="nextstepaction"]
> [Diagrama de la sintaxis de OData para Azure Cognitive Search](https://azuresearch.github.io/odata-syntax-diagram/#boolean_expression)

> [!NOTE]
> Consulte [Referencia de la sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md) para obtener la EBNF completa.

Los tipos de expresiones booleanas incluyen los siguientes:

- Expresiones de filtro de colección con `any` o `all`. Aplican criterios de filtro a campos de colección. Para más información, consulte [Operadores de colección de OData en Azure Cognitive Search](search-query-odata-collection-operators.md).
- Expresiones lógicas que combinan otras expresiones booleanas mediante los operadores `and`, `or` y `not`. Para más información, consulte [Operadores lógicos de OData en Azure Cognitive Search](search-query-odata-logical-operators.md).
- Expresiones de comparación, que comparan campos o variables de rango con valores constantes mediante los operadores `eq`, `ne`, `gt`, `lt`, `ge` y `le`. Para más información, consulte [Operadores de comparación de OData en Azure Search](search-query-odata-comparison-operators.md). Las expresiones de comparación también se usan para comparar distancias entre coordenadas geoespaciales mediante la función `geo.distance`. Para más información, consulte [Funciones geoespaciales de OData en Azure Cognitive Search](search-query-odata-geo-spatial-functions.md).
- Los literales booleanos `true` y `false`. En ocasiones, estas constantes pueden ser útiles al generar filtros mediante programación, pero en la práctica no se suelen usar en otros casos.
- Llamadas a funciones booleanas, incluidas las siguientes:
  - `geo.intersects`, que comprueba si un punto especificado está dentro de un polígono determinado. Para más información, consulte [Funciones geoespaciales de OData en Azure Cognitive Search](search-query-odata-geo-spatial-functions.md).
  - `search.in`, que compara un campo o una variable de rango con cada uno de los valores de una lista de valores. Para más información, consulte [Función `search.in` de OData en Azure Cognitive Search](search-query-odata-search-in-function.md).
  - `search.ismatch` y `search.ismatchscoring`, que ejecutan operaciones de búsqueda de texto completo en un contexto de filtro. Para más información, consulte [Funciones de búsqueda de texto completo de OData en Azure Cognitive Search](search-query-odata-full-text-search-functions.md).
- Rutas de acceso de campo o variables de rango de tipo `Edm.Boolean`. Por ejemplo, si el índice tiene un campo booleano denominado `IsEnabled` y quiere devolver todos los documentos en los que este campo es `true`, la expresión de filtro puede ser simplemente el nombre `IsEnabled`.
- Expresiones booleanas entre paréntesis. El uso de paréntesis puede ayudar a determinar de forma explícita el orden de las operaciones en un filtro. Para más información sobre la prioridad predeterminada de los operadores de OData, vea la sección siguiente.

### <a name="operator-precedence-in-filters"></a>Precedencia de operadores en filtros

Si escribe una expresión de filtro sin paréntesis alrededor de sus subexpresiones, Azure Cognitive Search la evaluará según un conjunto de reglas de precedencia de operadores. Estas reglas se basan en qué operadores se usan para combinar las subexpresiones. En la tabla siguiente se enumeran los grupos de operadores en orden de mayor a menor prioridad:

| Grupo | Operadores |
| --- | --- |
| Operadores lógicos | `not` |
| Operadores de comparación | `eq`, `ne`, `gt`, `lt`, `ge`, `le` |
| Operadores lógicos | `and` |
| Operadores lógicos | `or` |

Un operador situado en una posición superior en la tabla anterior se "enlazará más estrechamente" a sus operandos que otros operadores. Por ejemplo, `and` tiene mayor prioridad que `or`, y los operadores de comparación tienen una prioridad más alta que cualquiera de ellos, por lo que las dos expresiones siguientes son equivalentes:

```odata-filter-expr
    Rating gt 0 and Rating lt 3 or Rating gt 7 and Rating lt 10
    ((Rating gt 0) and (Rating lt 3)) or ((Rating gt 7) and (Rating lt 10))
```

El operador `not` tiene la prioridad más alta de todos, incluso superior a la de los operadores de comparación. Por ese motivo, si se intenta escribir un filtro similar al siguiente:

```odata-filter-expr
    not Rating gt 5
```

Obtendrá este mensaje de error:

```text
    Invalid expression: A unary operator with an incompatible type was detected. Found operand type 'Edm.Int32' for operator kind 'Not'.
```

Este error se produce porque el operador solo está asociado con el campo `Rating`, que es de tipo `Edm.Int32`, y no con la expresión de comparación completa. La solución consiste en colocar el operando de `not` entre paréntesis:

```odata-filter-expr
    not (Rating gt 5)
```

<a name="bkmk_limits"></a>

### <a name="filter-size-limitations"></a>Limitaciones del tamaño del filtro

Hay límites de tamaño y complejidad para las expresiones de filtro que puede enviar a Azure Cognitive Search. A grandes rasgos, los límites se basan en el número de cláusulas en su expresión de filtro. Como regla general, si tiene cientos de cláusulas, corre el riesgo de superar el límite. Es recomendable diseñar la aplicación de manera que no genere filtros de tamaño sin enlazar.

> [!TIP]
> El uso de [la función `search.in`](search-query-odata-search-in-function.md) en lugar de disyunciones largas de comparaciones de igualdad puede ayudar a evitar el límite de la cláusula de filtro, ya que una llamada de función se considera una sola cláusula.

## <a name="examples"></a>Ejemplos

Buscar todos los hoteles con al menos una habitación con una tarifa base inferior a 200 USD que tengan una valoración de 4 o superior:

```odata-filter-expr
    $filter=Rooms/any(room: room/BaseRate lt 200.0) and Rating ge 4
```

Buscar todos los hoteles que no sean "Sea View Motel" que se hayan reformado desde 2010:

```odata-filter-expr
    $filter=HotelName ne 'Sea View Motel' and LastRenovationDate ge 2010-01-01T00:00:00Z
```

Buscar todos los hoteles que se hayan reformado en 2010 o después. El literal de fecha y hora incluye información de zona horaria para la Hora estándar del Pacífico:  

```odata-filter-expr
    $filter=LastRenovationDate ge 2010-01-01T00:00:00-08:00
```

Buscar todos los hoteles que tengan aparcamiento y no permitan fumar en ninguna habitación:

```odata-filter-expr
    $filter=ParkingIncluded and Rooms/all(room: not room/SmokingAllowed)
```

 \- O BIEN -  

```odata-filter-expr
    $filter=ParkingIncluded eq true and Rooms/all(room: room/SmokingAllowed eq false)
```

Buscar todos los hoteles que sean de lujo o que incluyan estacionamiento y tengan una valoración de cinco:  

```odata-filter-expr
    $filter=(Category eq 'Luxury' or ParkingIncluded eq true) and Rating eq 5
```

Buscar todos los hoteles con la etiqueta "wifi" en al menos una habitación (donde cada habitación tiene etiquetas almacenadas en un campo `Collection(Edm.String)`):  

```odata-filter-expr
    $filter=Rooms/any(room: room/Tags/any(tag: tag eq 'wifi'))
```

Buscar todos los hoteles con cualquier número de habitaciones:  

```odata-filter-expr
    $filter=Rooms/any()
```

Buscar todos los hoteles que no tengan habitaciones:

```odata-filter-expr
    $filter=not Rooms/any()
```

Buscar todos los hoteles a una distancia no superior a diez kilómetros de un punto de referencia determinado (donde `Location` es un campo de tipo `Edm.GeographyPoint`):

```odata-filter-expr
    $filter=geo.distance(Location, geography'POINT(-122.131577 47.678581)') le 10
```

Buscar todos los hoteles dentro de una ventanilla dada descrita como un polígono (donde `Location` es un campo de tipo Edm.GeographyPoint). El polígono debe estar cerrado, lo que significa que el primer y último conjunto de puntos deben ser los mismos. Además, [los puntos se deben enumerar en sentido contrario a las agujas del reloj](/rest/api/searchservice/supported-data-types#Anchor_1).

```odata-filter-expr
    $filter=geo.intersects(Location, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))')
```

Buscar todos los hoteles donde el campo "Descripción" sea NULL. El campo será NULL si no se ha establecido nunca, o bien si se ha establecido en NULL de forma explícita:  

```odata-filter-expr
    $filter=Description eq null
```

Buscar todos los hoteles cuyo nombre sea "Sea View motel" o "Budget hotel". Estas frases contienen espacios, y un espacio es un delimitador predeterminado. Puede especificar un delimitador alternativo entre comillas simples como tercer parámetro de cadena:  

```odata-filter-expr
    $filter=search.in(HotelName, 'Sea View motel,Budget hotel', ',')
```

Buscar todos los hoteles cuyo nombre sea "Sea View motel" o "Budget hotel" separados por "|"):  

```odata-filter-expr
    $filter=search.in(HotelName, 'Sea View motel|Budget hotel', '|')
```

Buscar todos los hoteles en los que todas las habitaciones tengan la etiqueta "wifi" o "tub":

```odata-filter-expr
    $filter=Rooms/any(room: room/Tags/any(tag: search.in(tag, 'wifi, tub'))
```

Buscar una coincidencia en las frases de una colección, como "heated towel racks" o "hairdryer included" en las etiquetas.

```odata-filter-expr
    $filter=Rooms/any(room: room/Tags/any(tag: search.in(tag, 'heated towel racks,hairdryer included', ','))
```

Buscar documentos con la palabra "waterfront". Esta consulta de filtro es idéntica a una [solicitud de búsqueda](/rest/api/searchservice/search-documents) con `search=waterfront`.

```odata-filter-expr
    $filter=search.ismatchscoring('waterfront')
```

Buscar documentos con la palabra "hostel" y una valoración superior o igual a cuatro, o documentos con la palabra "motel" y una valoración igual a cinco. Esta solicitud no se podría expresar sin la función `search.ismatchscoring` ya que combina la búsqueda de texto completo con las operaciones de filtro mediante `or`.

```odata-filter-expr
    $filter=search.ismatchscoring('hostel') and rating ge 4 or search.ismatchscoring('motel') and rating eq 5
```

Buscar documentos sin la palabra "luxury".

```odata-filter-expr
    $filter=not search.ismatch('luxury')
```

Buscar documentos con la frase "ocean view" o con una valoración igual a cinco. La consulta `search.ismatchscoring` se ejecutará solo en los campos `HotelName` y `Description`. También se devolverán los documentos que cumplan la segunda cláusula de la disyunción (hoteles con `Rating` igual a 5). Esos documentos se devolverán con una puntuación igual a cero para dejar claro que no han coincidido con ninguno de los elementos puntuados de la expresión.

```odata-filter-expr
    $filter=search.ismatchscoring('"ocean view"', 'Description,HotelName') or Rating eq 5
```

Buscar hoteles donde los términos "hotel" y "airport" no estén separados en la descripción por más de cinco palabras y donde todas las habitaciones sean para no fumadores. Esta consulta utiliza el [lenguaje de consulta completo de Lucene](query-lucene-syntax.md).

```odata-filter-expr
    $filter=search.ismatch('"hotel airport"~5', 'Description', 'full', 'any') and not Rooms/any(room: room/SmokingAllowed)
```

## <a name="next-steps"></a>Pasos siguientes  

- [Filtros de Azure Cognitive Search](search-filters.md)
- [Información general sobre el lenguaje de expresiones OData para Azure Cognitive Search](query-odata-filter-orderby-syntax.md)
- [Referencia de sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md)
- [Búsqueda de documentos &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/Search-Documents)