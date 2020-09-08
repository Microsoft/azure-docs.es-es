---
title: Referencia de las funciones de búsqueda de texto completo de OData
titleSuffix: Azure Cognitive Search
description: Funciones de búsqueda de texto completo de OData, search.ismatch y search.ismatchscoring, en consultas de Azure Cognitive Search.
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
ms.openlocfilehash: 78f9e4d8fa80fdf74bdb5cd79f4489d12696fcc2
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935796"
---
# <a name="odata-full-text-search-functions-in-azure-cognitive-search---searchismatch-and-searchismatchscoring"></a>Funciones de búsqueda de texto completo de OData en Azure Cognitive Search: `search.ismatch` y `search.ismatchscoring`

Azure Cognitive Search admite la búsqueda de texto completo en el contexto de [expresiones de filtro de OData](query-odata-filter-orderby-syntax.md) mediante las funciones `search.ismatch` y `search.ismatchscoring`. Estas funciones permiten combinar la búsqueda de texto completo con filtros booleanos estrictos de formas que no son posibles con el parámetro `search` de nivel superior de [Search API](/rest/api/searchservice/search-documents).

> [!NOTE]
> Las funciones `search.ismatch` y `search.ismatchscoring` solo se admiten en los filtros de [Search API](/rest/api/searchservice/search-documents). No se admiten en las API [Suggest](/rest/api/searchservice/suggestions) o [Autocomplete](/rest/api/searchservice/autocomplete).

## <a name="syntax"></a>Sintaxis

La siguiente EBNF ([notación de Backus-Naur extendida](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) define la gramática de las funciones `search.ismatch` y `search.ismatchscoring`:

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
search_is_match_call ::=
    'search.ismatch'('scoring')?'(' search_is_match_parameters ')'

search_is_match_parameters ::=
    string_literal(',' string_literal(',' query_type ',' search_mode)?)?

query_type ::= "'full'" | "'simple'"

search_mode ::= "'any'" | "'all'"
```

También está disponible un diagrama de sintaxis interactivo:

> [!div class="nextstepaction"]
> [Diagrama de la sintaxis de OData para Azure Cognitive Search](https://azuresearch.github.io/odata-syntax-diagram/#search_is_match_call)

> [!NOTE]
> Consulte [Referencia de la sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md) para obtener la EBNF completa.

### <a name="searchismatch"></a>search.ismatch

La función `search.ismatch` evalúa la consulta de búsqueda de texto completo como parte de una expresión de filtro. En el conjunto de resultados, se devolverán los documentos donde se encontraran coincidencias con la consulta de búsqueda. Están disponibles las siguientes sobrecargas de esta función:

- `search.ismatch(search)`
- `search.ismatch(search, searchFields)`
- `search.ismatch(search, searchFields, queryType, searchMode)`

Los parámetros se definen en la tabla siguiente:

| Nombre de parámetro | Tipo | Descripción |
| --- | --- | --- |
| `search` | `Edm.String` | La consulta de búsqueda (en la sintaxis de consulta Lucene [simple](query-simple-syntax.md) o [completa](query-lucene-syntax.md)). |
| `searchFields` | `Edm.String` | Lista separada por comas de los campos de búsqueda en los que se va buscar; el valor predeterminado son todos los campos de búsqueda del índice. Cuando se usa la [búsqueda clasificada por campos](query-lucene-syntax.md#bkmk_fields) en el parámetro `search`, los especificadores de campo de la consulta de Lucene invalidan los campos especificados en este parámetro. |
| `queryType` | `Edm.String` | `'simple'` o `'full'`; el valor predeterminado es `'simple'`. Especifica qué lenguaje de consulta se usó en el parámetro `search`. |
| `searchMode` | `Edm.String` | `'any'` o `'all'`; el valor predeterminado es `'any'`. Indica si alguno o todos los términos de búsqueda del parámetro `search` deben coincidir para que el documento cuente como una coincidencia. Cuando se usan los [operadores booleanos de Lucene](query-lucene-syntax.md#bkmk_boolean) en el parámetro `search`, tendrán prioridad sobre este parámetro. |

Todos los parámetros anteriores son equivalentes a los [parámetros de solicitud de búsqueda de Search API](/rest/api/searchservice/search-documents) correspondientes.

La función `search.ismatch` devuelve un valor de tipo `Edm.Boolean`, que permite redactarlo con otras subexpresiones de filtro mediante los [operadores lógicos](search-query-odata-logical-operators.md) booleanos.

> [!NOTE]
> Azure Cognitive Search no admite el uso de `search.ismatch` o `search.ismatchscoring` dentro de las expresiones lambda. Esto significa que no se pueden escribir filtros sobre colecciones de objetos que pueden poner en correlación las coincidencias de búsqueda de texto completo con coincidencias de filtro estrictas en el mismo objeto. Para más información sobre esta limitación y para ver ejemplos, consulte [Solución de problemas de los filtros de colección en Azure Cognitive Search](search-query-troubleshoot-collection-filters.md). Para más información sobre por qué existe esta limitación, consulte [Descripción de los filtros de colección en Azure Cognitive Search](search-query-understand-collection-filters.md).


### <a name="searchismatchscoring"></a>search.ismatchscoring

La función `search.ismatchscoring`, así como la función `search.ismatch`, devuelve `true` para los documentos que coinciden con la consulta de búsqueda de texto completo que se ha pasado como parámetro. La diferencia entre ellas es que la puntuación de relevancia de los documentos en los que se encontraron coincidencias con la consulta `search.ismatchscoring` contribuirá a la puntuación total de los documentos, mientras que en el caso de `search.ismatch`, la puntuación de los documentos no cambiará. Las siguientes sobrecargas de esta función están disponibles con parámetros idénticos a los de `search.ismatch`:

- `search.ismatchscoring(search)`
- `search.ismatchscoring(search, searchFields)`
- `search.ismatchscoring(search, searchFields, queryType, searchMode)`

Esto significa que las funciones `search.ismatch` y `search.ismatchscoring` se pueden usar en la misma expresión de filtro.

## <a name="examples"></a>Ejemplos

Buscar documentos con la palabra "waterfront". Esta consulta de filtro es idéntica a una [solicitud de búsqueda](/rest/api/searchservice/search-documents) con `search=waterfront`.

```odata-filter-expr
    search.ismatchscoring('waterfront')
```

Buscar documentos con la palabra "hostel" y una valoración superior o igual a cuatro, o documentos con la palabra "motel" y una valoración igual a cinco. Tenga en cuenta que esta solicitud no se puede expresar sin la función `search.ismatchscoring`.

```odata-filter-expr
    search.ismatchscoring('hostel') and Rating ge 4 or search.ismatchscoring('motel') and Rating eq 5
```

Buscar documentos sin la palabra "luxury".

```odata-filter-expr
    not search.ismatch('luxury')
```

Buscar documentos con la frase "ocean view" o con una valoración igual a cinco. La consulta `search.ismatchscoring` se ejecutará solo en los campos `HotelName` y `Rooms/Description`.

También se devolverán los documentos que cumplan la segunda cláusula de la disyunción (hoteles con `Rating` igual a 5). Para dejar claro que esos documentos no han coincidido con ninguno de los elementos puntuados de la expresión, se devolverán con una puntuación igual a cero.

```odata-filter-expr
    search.ismatchscoring('"ocean view"', 'Rooms/Description,HotelName') or Rating eq 5
```

Busque documentos donde los términos "hotel" y "airport" no estén separados por más de cinco palabras entre sí en la descripción del hotel y donde no se permita fumar en al menos algunas de las habitaciones. Esta consulta utiliza el [lenguaje de consulta completo de Lucene](query-lucene-syntax.md).

```odata-filter-expr
    search.ismatch('"hotel airport"~5', 'Description', 'full', 'any') and Rooms/any(room: not room/SmokingAllowed)
```

## <a name="next-steps"></a>Pasos siguientes  

- [Filtros de Azure Cognitive Search](search-filters.md)
- [Información general sobre el lenguaje de expresiones OData para Azure Cognitive Search](query-odata-filter-orderby-syntax.md)
- [Referencia de sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md)
- [Búsqueda de documentos &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/Search-Documents)