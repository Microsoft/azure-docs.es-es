---
title: Referencia de la sintaxis de expresiones OData
titleSuffix: Azure Cognitive Search
description: Especificación de gramática formal y sintaxis para expresiones de OData en las consultas de Azure Cognitive Search.
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
ms.openlocfilehash: 83e9ae4aa68bc0c819d02a0cc6c39758549811cb
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88928860"
---
# <a name="odata-expression-syntax-reference-for-azure-cognitive-search"></a>Referencia de sintaxis de expresiones de OData para Azure Cognitive Search

Azure Cognitive Search usa [expresiones de OData](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html) como parámetros en la API. Normalmente, las expresiones de OData se usan para los parámetros `$orderby` y `$filter`. Estas expresiones pueden ser complejas y pueden contener varias cláusulas, funciones y operadores. Sin embargo, incluso las expresiones simples de OData, como las rutas de acceso de las propiedades, se usan en muchas partes de la API REST de Azure Cognitive Search. Por ejemplo, las expresiones de ruta de acceso se utilizan para hacer referencia a campos secundarios de campos complejos en toda la API, como cuando se enumeran los campos secundarios en un [proveedor de sugerencias](index-add-suggesters.md), una [función de puntuación](index-add-scoring-profiles.md), el parámetro `$select` o incluso en la [búsqueda por campos en las consultas de Lucene](query-lucene-syntax.md).

En este artículo se describen todas estas formas de expresiones de OData mediante una gramática formal. También hay un [diagrama interactivo](#syntax-diagram) que ayuda a explorar visualmente la gramática.

## <a name="formal-grammar"></a>Gramática formal

Podemos describir el subconjunto del lenguaje de OData admitido en Azure Cognitive Search mediante una gramática EBNF ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)). Las reglas se enumeran por orden "descendente", comenzando con las expresiones más complejas, que se dividen en expresiones más primitivas. En la parte superior están las reglas de gramática que se corresponden con parámetros específicos de la API REST de Azure Cognitive Search:

- [`$filter`](search-query-odata-filter.md), definido por la regla `filter_expression`.
- [`$orderby`](search-query-odata-orderby.md), definido por la regla `order_by_expression`.
- [`$select`](search-query-odata-select.md), definido por la regla `select_expression`.
- Rutas de acceso de campos, definidos por la regla `field_path`. Las rutas de acceso de campos se utilizan en toda la API. Pueden hacer referencia a campos de nivel superior o a un índice o a campos secundarios con uno o varios antecesores de [campos complejos](search-howto-complex-data-types.md).

Después de la gramática EBNF, hay un [diagrama de sintaxis](https://en.wikipedia.org/wiki/Syntax_diagram) que le permite explorar de forma interactiva la gramática y las relaciones entre sus reglas.

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
/* Top-level rules */

filter_expression ::= boolean_expression

order_by_expression ::= order_by_clause(',' order_by_clause)*

select_expression ::= '*' | field_path(',' field_path)*

field_path ::= identifier('/'identifier)*


/* Shared base rules */

identifier ::= [a-zA-Z_][a-zA-Z_0-9]*


/* Rules for $orderby */

order_by_clause ::= (field_path | sortable_function) ('asc' | 'desc')?

sortable_function ::= geo_distance_call | 'search.score()'


/* Rules for $filter */

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

collection_filter_expression ::=
    field_path'/all(' lambda_expression ')'
    | field_path'/any(' lambda_expression ')'
    | field_path'/any()'

lambda_expression ::= identifier ':' boolean_expression

logical_expression ::=
    boolean_expression ('and' | 'or') boolean_expression
    | 'not' boolean_expression

comparison_expression ::= 
    variable_or_function comparison_operator constant | 
    constant comparison_operator variable_or_function

variable_or_function ::= variable | function_call

comparison_operator ::= 'gt' | 'lt' | 'ge' | 'le' | 'eq' | 'ne'


/* Rules for constants and literals */

constant ::=
    string_literal
    | date_time_offset_literal
    | integer_literal
    | float_literal
    | boolean_literal
    | 'null'

string_literal ::= "'"([^'] | "''")*"'"

date_time_offset_literal ::= date_part'T'time_part time_zone

date_part ::= year'-'month'-'day

time_part ::= hour':'minute(':'second('.'fractional_seconds)?)?

zero_to_fifty_nine ::= [0-5]digit

digit ::= [0-9]

year ::= digit digit digit digit

month ::= '0'[1-9] | '1'[0-2]

day ::= '0'[1-9] | [1-2]digit | '3'[0-1]

hour ::= [0-1]digit | '2'[0-3]

minute ::= zero_to_fifty_nine

second ::= zero_to_fifty_nine

fractional_seconds ::= integer_literal

time_zone ::= 'Z' | sign hour':'minute

sign ::= '+' | '-'

/* In practice integer literals are limited in length to the precision of
the corresponding EDM data type. */
integer_literal ::= digit+

float_literal ::=
    sign? whole_part fractional_part? exponent?
    | 'NaN'
    | '-INF'
    | 'INF'

whole_part ::= integer_literal

fractional_part ::= '.'integer_literal

exponent ::= 'e' sign? integer_literal

boolean_literal ::= 'true' | 'false'


/* Rules for functions */

function_call ::=
    geo_distance_call |
    boolean_function_call

geo_distance_call ::=
    'geo.distance(' variable ',' geo_point ')'
    | 'geo.distance(' geo_point ',' variable ')'

geo_point ::= "geography'POINT(" lon_lat ")'"

lon_lat ::= float_literal ' ' float_literal

boolean_function_call ::=
    geo_intersects_call |
    search_in_call |
    search_is_match_call

geo_intersects_call ::=
    'geo.intersects(' variable ',' geo_polygon ')'

/* You need at least four points to form a polygon, where the first and
last points are the same. */
geo_polygon ::=
    "geography'POLYGON((" lon_lat ',' lon_lat ',' lon_lat ',' lon_lat_list "))'"

lon_lat_list ::= lon_lat(',' lon_lat)*

search_in_call ::=
    'search.in(' variable ',' string_literal(',' string_literal)? ')'

/* Note that it is illegal to call search.ismatch or search.ismatchscoring
from inside a lambda expression. */
search_is_match_call ::=
    'search.ismatch'('scoring')?'(' search_is_match_parameters ')'

search_is_match_parameters ::=
    string_literal(',' string_literal(',' query_type ',' search_mode)?)?

query_type ::= "'full'" | "'simple'"

search_mode ::= "'any'" | "'all'"
```

## <a name="syntax-diagram"></a>Diagrama de sintaxis

Para explorar visualmente la gramática del lenguaje de OData admitida por Azure Cognitive Search, pruebe el diagrama de sintaxis interactivo:

> [!div class="nextstepaction"]
> [Diagrama de la sintaxis de OData para Azure Cognitive Search](https://azuresearch.github.io/odata-syntax-diagram/)

## <a name="see-also"></a>Consulte también  

- [Filtros de Azure Cognitive Search](search-filters.md)
- [Búsqueda de documentos &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/Search-Documents)
- [Sintaxis de consulta de Lucene](query-lucene-syntax.md)
- [Sintaxis de consulta simple en Azure Cognitive Search](query-simple-syntax.md)