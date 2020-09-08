---
title: Referencia de la función search.in de OData
titleSuffix: Azure Cognitive Search
description: Documentación de referencia y sintaxis para usar la función search.in en las consultas de Azure Cognitive Search.
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
ms.openlocfilehash: 9ad6f89392846564631b70f0acfb5658a050be80
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88922827"
---
# <a name="odata-searchin-function-in-azure-cognitive-search"></a>Función `search.in` de OData en Azure Cognitive Search

Un escenario común en las [expresiones de filtro de OData](query-odata-filter-orderby-syntax.md) consiste en comprobar si un solo campo de cada documento es igual a uno de muchos valores posibles. Por ejemplo, para implementar el [recorte de seguridad](search-security-trimming-for-azure-search.md), algunas aplicaciones realizan una comparación entre un campo que contiene uno o varios identificadores de entidad de seguridad y una lista de identificadores de entidad de seguridad que representa al usuario que emite la consulta. Una manera de escribir una consulta como esta consiste en usar los operadores [`eq`](search-query-odata-comparison-operators.md) y [`or`](search-query-odata-logical-operators.md):

```odata-filter-expr
    group_ids/any(g: g eq '123' or g eq '456' or g eq '789')
```

También hay una forma más corta de escribirla mediante la función `search.in`:

```odata-filter-expr
    group_ids/any(g: search.in(g, '123, 456, 789'))
```

> [!IMPORTANT]
> Además de ser más corta y más fácil de leer, la función `search.in` también conlleva [ventajas de rendimiento](#bkmk_performance) y evita determinadas [limitaciones en el tamaño de los filtros](search-query-odata-filter.md#bkmk_limits) cuando hay que incluir en el filtro cientos o incluso miles de valores. Por este motivo, se recomienda encarecidamente usar `search.in` en lugar de una disyunción más compleja de expresiones de igualdad.

> [!NOTE]
> Recientemente se ha incluido en la versión 4.01 del estándar OData el [operador `in`](https://docs.oasis-open.org/odata/odata/v4.01/cs01/part2-url-conventions/odata-v4.01-cs01-part2-url-conventions.html#_Toc505773230), que tiene un comportamiento similar al de la función `search.in` en Azure Cognitive Search. Aun así, Azure Cognitive Search no admite este operador, por lo que debe usar en cambio la función `search.in`.

## <a name="syntax"></a>Sintaxis

La siguiente EBNF ([notación de Backus-Naur extendida](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) define la gramática de la función `search.in`:

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
search_in_call ::=
    'search.in(' variable ',' string_literal(',' string_literal)? ')'
```

También está disponible un diagrama de sintaxis interactivo:

> [!div class="nextstepaction"]
> [Diagrama de la sintaxis de OData para Azure Cognitive Search](https://azuresearch.github.io/odata-syntax-diagram/#search_in_call)

> [!NOTE]
> Consulte [Referencia de la sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md) para obtener la EBNF completa.

La función `search.in` prueba si un determinado campo de cadena o variable de rango es igual a uno de una lista de valores dada. La igualdad entre la variable y cada valor de la lista se determina distinguiendo entre mayúsculas y minúsculas, del mismo modo que para el operador `eq`. Por lo tanto, una expresión como `search.in(myfield, 'a, b, c')` es equivalente a `myfield eq 'a' or myfield eq 'b' or myfield eq 'c'`, salvo que `search.in` ofrecerá un rendimiento mucho mejor.

Hay dos sobrecargas de la función `search.in`:

- `search.in(variable, valueList)`
- `search.in(variable, valueList, delimiters)`

Los parámetros se definen en la tabla siguiente:

| Nombre de parámetro | Tipo | Descripción |
| --- | --- | --- |
| `variable` | `Edm.String` | Referencia de campo de cadena (o variable de rango en un campo de colección de cadenas en el caso de que `search.in` se use en una expresión `any` o `all`). |
| `valueList` | `Edm.String` | Cadena que contiene una lista delimitada de valores que deben coincidir con el parámetro `variable`. Si no se especifica el parámetro `delimiters`, los delimitadores predeterminados son espacio y coma. |
| `delimiters` | `Edm.String` | Cadena en la que cada carácter se trata como separador al analizar el parámetro `valueList`. El valor predeterminado de este parámetro es `' ,'`, lo que significa que todos los valores con espacios o comas entre ellos se separarán. Si tiene que usar separadores que no sean espacios y comas porque sus valores incluyen dichos caracteres, puede especificar delimitadores alternativos, como `'|'` en este parámetro. |

<a name="bkmk_performance"></a>

### <a name="performance-of-searchin"></a>Rendimiento de `search.in`

Si usa `search.in`, puede esperar un tiempo de respuesta de fracciones de segundo cuando el segundo parámetro contiene una lista de cientos o miles de valores. No hay ningún límite explícito en el número de elementos que puede pasar a `search.in`, aunque sigue estando limitado por el tamaño máximo de la solicitud. Sin embargo, la latencia aumentará a medida que crece el número de valores.

## <a name="examples"></a>Ejemplos

Busque todos los hoteles cuyo nombre sea "Sea View motel" o "Budget hotel". Las frases contienen espacios, que es un delimitador predeterminado. Puede especificar un delimitador alternativo entre comillas simples como tercer parámetro de cadena:  

```odata-filter-expr
    search.in(HotelName, 'Sea View motel,Budget hotel', ',')
```

Buscar todos los hoteles cuyo nombre sea "Sea View motel" o "Budget hotel" separados por "|"):

```odata-filter-expr
    search.in(HotelName, 'Sea View motel|Budget hotel', '|')
```

Busque todos los hoteles con habitaciones que tengan la etiqueta "wifi" o "tub":

```odata-filter-expr
    Rooms/any(room: room/Tags/any(tag: search.in(tag, 'wifi, tub')))
```

Buscar una coincidencia en las frases de una colección, como "heated towel racks" o "hairdryer included" en las etiquetas.

```odata-filter-expr
    Rooms/any(room: room/Tags/any(tag: search.in(tag, 'heated towel racks,hairdryer included', ','))
```

Busque todos los hoteles sin la etiqueta "motel" o "cabin":

```odata-filter-expr
    Tags/all(tag: not search.in(tag, 'motel, cabin'))
```

## <a name="next-steps"></a>Pasos siguientes  

- [Filtros de Azure Cognitive Search](search-filters.md)
- [Información general sobre el lenguaje de expresiones OData para Azure Cognitive Search](query-odata-filter-orderby-syntax.md)
- [Referencia de sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md)
- [Búsqueda de documentos &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/Search-Documents)