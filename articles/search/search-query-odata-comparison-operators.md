---
title: Referencia de los operadores de comparación de OData
titleSuffix: Azure Cognitive Search
description: Documentación de referencia y sintaxis para usar los operadores de comparación de OData, eq, ne, gt, lt, ge y le, en consultas de Azure Cognitive Search.
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
ms.openlocfilehash: fc5803f96c30ea1df362676aa8c4104bb0b69db3
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934878"
---
# <a name="odata-comparison-operators-in-azure-cognitive-search---eq-ne-gt-lt-ge-and-le"></a>Operadores de comparación de OData en Azure Cognitive Search: `eq`, `ne`, `gt`, `lt`, `ge` y `le`

La operación más básica en una [expresión de filtro de OData](query-odata-filter-orderby-syntax.md) de Azure Cognitive Search consiste en comparar un campo con un valor determinado. Hay dos tipos de comparación posibles: comparación de igualdad y comparación de rango. Puede usar los operadores siguientes para comparar un campo con un valor constante:

Operadores de igualdad:

- `eq`: se comprueba si un campo es **igual a** un valor constante
- `ne`: se comprueba si campo **no es igual a** un valor constante

Operadores de rango:

- `gt`: se comprueba si un campo es **mayor que** un valor constante
- `lt`: se comprueba si un campo es **menor que** un valor constante
- `ge`: se comprueba si un campo es **mayor o igual que** un valor constante
- `le`: se comprueba si un campo es **menor o igual que** un valor constante

Puede usar los operadores de rango en combinación con los [operadores lógicos](search-query-odata-logical-operators.md) para comprobar si un campo está dentro de un intervalo de valores determinado. Vea los [ejemplos](#examples) más adelante en este artículo.

> [!NOTE]
> Si lo prefiere, puede colocar el valor constante en el lado izquierdo del operador y el nombre del campo en el lado derecho. Para los operadores de rango, se invierte el significado de la comparación. Por ejemplo, si es el valor constante está a la izquierda, `gt` podría probar si el valor constante es mayor que el campo. También puede usar los operadores de comparación para comparar el resultado de una función, como `geo.distance`, con un valor. Para las funciones booleanas como `search.ismatch`, la comparación del resultado con `true` o `false` es opcional.

## <a name="syntax"></a>Sintaxis

En la siguiente EBNF ([forma de Backus-Naur extendida](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) se define la gramática de una expresión de OData en la que se usan los operadores de comparación.

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
comparison_expression ::=
    variable_or_function comparison_operator constant |
    constant comparison_operator variable_or_function

variable_or_function ::= variable | function_call

comparison_operator ::= 'gt' | 'lt' | 'ge' | 'le' | 'eq' | 'ne'
```

También está disponible un diagrama de sintaxis interactivo:

> [!div class="nextstepaction"]
> [Diagrama de la sintaxis de OData para Azure Cognitive Search](https://azuresearch.github.io/odata-syntax-diagram/#comparison_expression)

> [!NOTE]
> Consulte [Referencia de la sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md) para obtener la EBNF completa.

Hay dos formas de expresiones de comparación. La única diferencia entre las dos es si la constante aparece en el lado izquierdo o derecho del operador. La expresión en el otro lado del operador debe ser una **variable** o una llamada de función. Una variable puede ser un nombre de campo o una variable de rango en el caso de una [expresión lambda](search-query-odata-collection-operators.md).

## <a name="data-types-for-comparisons"></a>Tipos de datos para las comparaciones

Los tipos de datos en ambos lados de un operador de comparación deben ser compatibles. Por ejemplo, si el lado izquierdo es un campo de tipo `Edm.DateTimeOffset`, el lado derecho debe ser una constante de fecha y hora. Los tipos de datos numéricos son más flexibles. Puede comparar variables y funciones de cualquier tipo numérico con constantes de cualquier otro tipo numérico, con algunas limitaciones, como se describe en la tabla siguiente.

| Tipo de variable o función | Tipo de valor constante | Limitaciones |
| --- | --- | --- |
| `Edm.Double` | `Edm.Double` | La comparación está sujeta a [reglas especiales para `NaN`](#special-case-nan) |
| `Edm.Double` | `Edm.Int64` | La constante se convierte a `Edm.Double`, lo que provoca una pérdida de precisión para los valores de gran magnitud |
| `Edm.Double` | `Edm.Int32` | N/D |
| `Edm.Int64` | `Edm.Double` | No se permiten las comparaciones con `NaN`, `-INF` o `INF` |
| `Edm.Int64` | `Edm.Int64` | N/D |
| `Edm.Int64` | `Edm.Int32` | La constante se convierte a `Edm.Int64` antes de la comparación |
| `Edm.Int32` | `Edm.Double` | No se permiten las comparaciones con `NaN`, `-INF` o `INF` |
| `Edm.Int32` | `Edm.Int64` | N/D |
| `Edm.Int32` | `Edm.Int32` | N/D |

Para las comparaciones que no están permitidas, como la de un campo de tipo `Edm.Int64` con `NaN`, la API REST de Azure Cognitive Search devolverá un error "HTTP 400: Solicitud incorrecta".

> [!IMPORTANT]
> Aunque las comparaciones de tipos numéricos son flexibles, se recomienda encarecidamente escribir comparaciones en filtros para que el valor constante sea del mismo tipo de datos que la variable o función con la que se compara. Esto es especialmente importante cuando se mezclan valores enteros y de punto flotante, en los que las conversiones implícitas pueden perder precisión.

<a name="special-case-nan"></a>

### <a name="special-cases-for-null-and-nan"></a>Casos especiales para `null` y `NaN`

Al usar los operadores de comparación, es importante recordar que en Azure Cognitive Search todos los campos que no son de colección pueden ser `null`. En la tabla siguiente se muestran todos los resultados posibles de una expresión de comparación en la que cualquiera de los lados puede ser `null`:

| Operator | Resultado cuando solo el campo o la variable es `null` | Resultado cuando solo la constante es `null` | Resultado cuando el campo o la variable y la constante son `null` |
| --- | --- | --- | --- |
| `gt` | `false` | HTTP 400: Error de solicitud incorrecta | HTTP 400: Error de solicitud incorrecta |
| `lt` | `false` | HTTP 400: Error de solicitud incorrecta | HTTP 400: Error de solicitud incorrecta |
| `ge` | `false` | HTTP 400: Error de solicitud incorrecta | HTTP 400: Error de solicitud incorrecta |
| `le` | `false` | HTTP 400: Error de solicitud incorrecta | HTTP 400: Error de solicitud incorrecta |
| `eq` | `false` | `false` | `true` |
| `ne` | `true` | `true` | `false` |

En resumen, `null` solo es igual a sí mismo, y no es menor o mayor que ningún otro valor.

Si el índice tiene campos de tipo `Edm.Double` y carga valores `NaN` a esos campos, tendrá que tenerlo en cuenta al escribir filtros. Azure Cognitive Search implementa el estándar IEEE 754 para el control de valores `NaN`, y las comparaciones con esos valores generan resultados que no son obvios, como se muestra en la tabla siguiente.

| Operator | Resultado cuando al menos un operando es `NaN` |
| --- | --- |
| `gt` | `false` |
| `lt` | `false` |
| `ge` | `false` |
| `le` | `false` |
| `eq` | `false` |
| `ne` | `true` |

En resumen, `NaN` no es igual a ningún valor, incluido él mismo.

### <a name="comparing-geo-spatial-data"></a>Comparación de datos geoespaciales

No se puede comparar directamente un campo de tipo `Edm.GeographyPoint` con un valor constante, pero se puede usar la función `geo.distance`. Esta función devuelve un valor de tipo `Edm.Double`, de modo que puede compararlo con un valor numérico constante para filtrar según la distancia con respecto a coordenadas geoespaciales constantes. Vea los [ejemplos](#examples) siguientes.

### <a name="comparing-string-data"></a>Comparación de datos de cadena

En los filtros, las cadenas se pueden comparar para obtener coincidencias exactas con los operadores `eq` y `ne`. Estas comparaciones distinguen mayúsculas de minúsculas.

## <a name="examples"></a>Ejemplos

Comparar documentos donde el campo `Rating` esté entre 3 y 5, inclusive:

```text
Rating ge 3 and Rating le 5
```

Comparar documentos donde el campo `Location` está a menos de dos kilómetros de la latitud y longitud especificadas:

```text
geo.distance(Location, geography'POINT(-122.031577 47.578581)') lt 2.0
```

Comparar documentos donde el campo `LastRenovationDate` es mayor o igual que el 1 de enero de 2015, medianoche (UTC):

```text
LastRenovationDate ge 2015-01-01T00:00:00.000Z
```

Comparar documentos donde el campo `Details/Sku` no es `null`:

```text
Details/Sku ne null
```

Comparar documentos para hoteles donde al menos una habitación tenga el tipo "Deluxe Room" (Habitación Deluxe), donde la cadena del campo `Rooms/Type` coincide exactamente con el filtro:

```text
Rooms/any(room: room/Type eq 'Deluxe Room')
```

## <a name="next-steps"></a>Pasos siguientes  

- [Filtros de Azure Cognitive Search](search-filters.md)
- [Información general sobre el lenguaje de expresiones OData para Azure Cognitive Search](query-odata-filter-orderby-syntax.md)
- [Referencia de sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md)
- [Búsqueda de documentos &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/Search-Documents)