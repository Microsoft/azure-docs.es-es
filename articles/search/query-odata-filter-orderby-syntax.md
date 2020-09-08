---
title: Introducción al lenguaje OData
titleSuffix: Azure Cognitive Search
description: Introducción a las expresiones filter, select y order by del lenguaje OData en consultas de Azure Cognitive Search.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/10/2020
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
ms.openlocfilehash: d04311fce81d147a0830918aee1d4a2a9c0808d4
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88923405"
---
# <a name="odata-language-overview-for-filter-orderby-and-select-in-azure-cognitive-search"></a>Introducción a `$filter`, `$orderby` y `$select` del lenguaje OData en Azure Cognitive Search

Azure Cognitive Search admite un subconjunto de la sintaxis de expresiones de OData **$filter**, **$orderby** y **$select**. Las expresiones de filtro se evalúan durante el análisis de la consulta, restringiendo la búsqueda a campos específicos o agregando criterios de coincidencia que se usan durante los exámenes de índice. Las expresiones Order by se aplican como un paso posterior al procesamiento sobre un conjunto de resultados para ordenar los documentos que se devuelven. Las expresiones select determinan qué campos de documento se incluyen en el conjunto de resultados. La sintaxis de estas expresiones es distinta de la sintaxis de consulta [simple](query-simple-syntax.md) o [completa](query-lucene-syntax.md) que se usa en el parámetro **search**, si bien existe cierto solapamiento en la sintaxis de los campos de referencia.

En este artículo se proporciona información general sobre el lenguaje de expresiones OData usado en las expresiones filter, order by y select. El lenguaje se presenta en sentido ascendente, se comienza con los elementos más básicos y se construye sobre ellos. La sintaxis de nivel superior de cada parámetro se describe en otro artículo:

- [Sintaxis de $filter](search-query-odata-filter.md)
- [Sintaxis de $orderby](search-query-odata-orderby.md)
- [Sintaxis de $select](search-query-odata-select.md)

Las expresiones de OData pueden ser sencillas o muy complejas, pero todas comparten elementos comunes. Las partes más básicas de una expresión de OData en Azure Cognitive Search son:

- **Rutas de acceso de campo**, que hacen referencia a campos específicos del índice.
- **Constantes**, que son valores literales de un tipo de datos determinado.

> [!NOTE]
> La terminología de Azure Cognitive Search se diferencia del [estándar de OData](https://www.odata.org/documentation/) de varias maneras. Lo que llamamos un **campo** en Azure Cognitive Search se denomina **propiedad** en OData y lo mismo para **ruta de acceso de campo** frente a **ruta de acceso de propiedad**. Un **índice** que contiene **documentos** en Azure Cognitive Search se conoce más generalmente en OData como un **conjunto de entidades** que contiene **entidades**. A lo largo de esta referencia se usará la terminología de Azure Cognitive Search.

## <a name="field-paths"></a>Rutas de acceso de campo

La siguiente EBNF ([notación de Backus-Naur extendida](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) define la gramática de las rutas de acceso de campo:

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
field_path ::= identifier('/'identifier)*

identifier ::= [a-zA-Z_][a-zA-Z_0-9]*
```

También está disponible un diagrama de sintaxis interactivo:

> [!div class="nextstepaction"]
> [Diagrama de la sintaxis de OData para Azure Cognitive Search](https://azuresearch.github.io/odata-syntax-diagram/#field_path)

> [!NOTE]
> Consulte [Referencia de la sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md) para obtener la EBNF completa.

Una ruta de acceso de campo se compone de uno o varios **identificadores** separados por barras diagonales. Cada identificador es una secuencia de caracteres que debe comenzar con una letra ASCII o un carácter de subrayado y contener solo letras ASCII, dígitos o caracteres de subrayado. Las letras pueden ser mayúsculas o minúsculas.

Un identificador puede hacer referencia al nombre de un campo o a una **variable de rango** en el contexto de una [expresión de colección](search-query-odata-collection-operators.md) (`any` o `all`) en un filtro. Una variable de rango es como una variable de bucle que representa el elemento actual de la colección. Si las colecciones son complejas, esa variable representa un objeto, que es el motivo de que pueda usar rutas de acceso de campo para hacer referencia a subcampos de la variable. Esta sintaxis es análoga a la notación de puntos de muchos lenguajes de programación.

En la tabla siguiente se muestran ejemplos de rutas de acceso de campo:

| Ruta de acceso de campo | Descripción |
| --- | --- |
| `HotelName` | Hace referencia a un campo de nivel superior del índice. |
| `Address/City` | Hace referencia al subcampo `City` de un campo complejo en el índice; `Address` es de tipo `Edm.ComplexType` en este ejemplo. |
| `Rooms/Type` | Hace referencia al subcampo `Type` de un campo de colección complejo en el índice; `Rooms` es de tipo `Collection(Edm.ComplexType)` en este ejemplo. |
| `Stores/Address/Country` | Hace referencia al subcampo `Country` del subcampo `Address` de un campo de colección complejo en el índice; `Stores` es de tipo `Collection(Edm.ComplexType)` y `Address` es de tipo `Edm.ComplexType` en este ejemplo. |
| `room/Type` | Hace referencia al subcampo `Type` de la variable de rango `room`, por ejemplo, en la expresión de filtro `Rooms/any(room: room/Type eq 'deluxe')`. |
| `store/Address/Country` | Hace referencia al subcampo `Country` del subcampo `Address` de la variable de rango `store`, por ejemplo, en la expresión de filtro `Stores/any(store: store/Address/Country eq 'Canada')`. |

El significado de una ruta de acceso de campo varía según el contexto. En los filtros, una ruta de acceso de campo hace referencia al valor de una *única instancia* de un campo en el documento actual. En otros contextos, como **$orderby**, **$select** o en la [búsqueda clasificada por campos de la sintaxis completa de Lucene](query-lucene-syntax.md#bkmk_fields), una ruta de acceso de campo hace referencia al campo propiamente dicho. Esta diferencia tiene algunas consecuencias en cómo se usan las rutas de acceso de campo en los filtros.

Considere la ruta de acceso de campo `Address/City`. En un filtro, esta ruta hacer referencia a una única ciudad del documento actual, como "San Francisco". En cambio, `Rooms/Type` hace referencia al subcampo `Type` para muchas habitaciones (por ejemplo, "standard" para la primera habitación, "deluxe" para la segunda, etc.). Puesto que `Rooms/Type` no hace referencia a una *única instancia* del subcampo `Type`, no se puede usar directamente en un filtro. Así que, para filtrar por el tipo de habitación, usaría una [expresión lambda](search-query-odata-collection-operators.md) con una variable de rango, como esta:

```odata
Rooms/any(room: room/Type eq 'deluxe')
```

En este ejemplo, la variable de rango `room` aparece en la ruta de acceso de campo `room/Type`. De este modo, `room/Type` hace referencia al tipo de habitación actual en el documento actual. Como es una única instancia del subcampo `Type`, se puede usar directamente en el filtro.

### <a name="using-field-paths"></a>Uso de rutas de acceso de campo

Las rutas de acceso de campo se usan en numerosos parámetros de las [API REST de Azure Cognitive Search](/rest/api/searchservice/). En la tabla siguiente se enumeran todos los lugares donde pueden usarse, además de las restricciones sobre su uso:

| API | Nombre de parámetro | Restricciones |
| --- | --- | --- |
| [Create](/rest/api/searchservice/create-index) or [Update](/rest/api/searchservice/update-index) Index | `suggesters/sourceFields` | None |
| [Create](/rest/api/searchservice/create-index) or [Update](/rest/api/searchservice/update-index) Index | `scoringProfiles/text/weights` | Solo puede hacer referencia a campos que **permiten realizar búsquedas**. |
| [Create](/rest/api/searchservice/create-index) or [Update](/rest/api/searchservice/update-index) Index | `scoringProfiles/functions/fieldName` | Solo puede hacer referencia a campos **filtrables**. |
| [Búsqueda](/rest/api/searchservice/search-documents) | `search` cuando `queryType` es `full` | Solo puede hacer referencia a campos que **permiten realizar búsquedas**. |
| [Búsqueda](/rest/api/searchservice/search-documents) | `facet` | Solo puede hacer referencia a campos **clasificables** |
| [Búsqueda](/rest/api/searchservice/search-documents) | `highlight` | Solo puede hacer referencia a campos que **permiten realizar búsquedas**. |
| [Búsqueda](/rest/api/searchservice/search-documents) | `searchFields` | Solo puede hacer referencia a campos que **permiten realizar búsquedas**. |
| [Suggest](/rest/api/searchservice/suggestions) y [Autocomplete](/rest/api/searchservice/autocomplete) | `searchFields` | Solo puede hacer referencia a campos que forman parte de un [proveedor de sugerencias](index-add-suggesters.md). |
| [Search](/rest/api/searchservice/search-documents), [Suggest](/rest/api/searchservice/suggestions) y [Autocomplete](/rest/api/searchservice/autocomplete) | `$filter` | Solo puede hacer referencia a campos **filtrables**. |
| [Search](/rest/api/searchservice/search-documents) y [Suggest](/rest/api/searchservice/suggestions) | `$orderby` | Solo puede hacer referencia a campos **ordenables**. |
| [Search](/rest/api/searchservice/search-documents), [Suggest](/rest/api/searchservice/suggestions) y [Lookup](/rest/api/searchservice/lookup-document) | `$select` | Solo puede hacer referencia a campos **recuperables** |

## <a name="constants"></a>Constantes

Las constantes en OData son valores literales de un determinado tipo de [Entity Data Model](/dotnet/framework/data/adonet/entity-data-model) (EDM). Consulte [Tipos de datos admitidos](/rest/api/searchservice/supported-data-types) para ver una lista de tipos admitidos en Azure Cognitive Search. No se admiten constantes de tipos de colección.

En la tabla siguiente se muestran ejemplos de constantes de cada uno de los tipos de datos admitidos por Azure Cognitive Search:

| Tipo de datos | Constantes de ejemplo |
| --- | --- |
| `Edm.Boolean` | `true`, `false` |
| `Edm.DateTimeOffset` | `2019-05-06T12:30:05.451Z` |
| `Edm.Double` | `3.14159`, `-1.2e7`, `NaN`, `INF`, `-INF` |
| `Edm.GeographyPoint` | `geography'POINT(-122.131577 47.678581)'` |
| `Edm.GeographyPolygon` | `geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'` |
| `Edm.Int32` | `123`, `-456` |
| `Edm.Int64` | `283032927235` |
| `Edm.String` | `'hello'` |

### <a name="escaping-special-characters-in-string-constants"></a>Caracteres especiales de escape en constantes de cadena

Las constantes de cadena en OData están delimitadas por comillas simples. Si tiene que crear una consulta con una constante de cadena que, a su vez, pueda contener comillas simples, puede escapar las comillas incrustadas si las duplica.

Por ejemplo, una frase con un apóstrofo sin formato, como "Alice's car", se representaría en OData como la constante de cadena `'Alice''s car'`.

> [!IMPORTANT]
> Al construir filtros mediante programación, es importante recordar escapar las constantes de cadena que proceden de los datos proporcionados por el usuario. Esto es para mitigar la posibilidad de [ataques por inyección de código](https://wikipedia.org/wiki/SQL_injection), en especial cuando se usan filtros para implementar el [recorte de seguridad](search-security-trimming-for-azure-search.md).

### <a name="constants-syntax"></a>Sintaxis de las constantes

El siguiente EBNF ([notación de Backus-Naur extendida](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) define la gramática de la mayoría de las constantes que se muestran en la tabla anterior. La gramática de los tipos geoespaciales se puede encontrar en [Funciones geoespaciales de Azure Cognitive Search](search-query-odata-geo-spatial-functions.md).

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
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
```

También está disponible un diagrama de sintaxis interactivo:

> [!div class="nextstepaction"]
> [Diagrama de la sintaxis de OData para Azure Cognitive Search](https://azuresearch.github.io/odata-syntax-diagram/#constant)

> [!NOTE]
> Consulte [Referencia de la sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md) para obtener la EBNF completa.

## <a name="building-expressions-from-field-paths-and-constants"></a>Generación de expresiones a partir de rutas de acceso de campo y constantes

Las rutas de acceso de campo y las constantes son la parte más básica de una expresión de OData, pero ya son expresiones completas por derecho propio. De hecho, el parámetro **$select** de Azure Cognitive Search no es más que una lista separada por comas de las rutas de acceso de campo y **$orderby** no es mucho más complicado que **$select**. Si resulta que tiene un campo de tipo `Edm.Boolean` en el índice, puede escribir incluso un filtro que no sea si no la ruta de acceso de ese campo. Las constantes `true` y `false` son igualmente filtros válidos.

Sin embargo, casi siempre necesitará expresiones más complejas que hagan referencia a más de un campo y más de una constante. Estas expresiones se compilan de diferentes maneras según el parámetro.

La siguiente EBNF ([notación de Backus-Naur extendida](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) define la gramática de los parámetros **$filter**, **$orderby** y **$select**. Estos parámetros se generan a partir de expresiones más sencillas que hacen referencia a las rutas de acceso de campo y a las constantes:

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
filter_expression ::= boolean_expression

order_by_expression ::= order_by_clause(',' order_by_clause)*

select_expression ::= '*' | field_path(',' field_path)*
```

También está disponible un diagrama de sintaxis interactivo:

> [!div class="nextstepaction"]
> [Diagrama de la sintaxis de OData para Azure Cognitive Search](https://azuresearch.github.io/odata-syntax-diagram/#filter_expression)

> [!NOTE]
> Consulte [Referencia de la sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md) para obtener la EBNF completa.

Los parámetros **$orderby** y **$select** son ambos listas separadas por comas de expresiones más sencillas. El parámetro **$filter** es una expresión booleana que se compone de subexpresiones más sencillas. Estas subexpresiones se combinan mediante operadores lógicos, como [`and`, `or`, y `not`](search-query-odata-logical-operators.md), operadores de comparación, como [`eq`, `lt`, `gt`, etc.](search-query-odata-comparison-operators.md) y operadores de colección, como [`any` y `all`](search-query-odata-collection-operators.md).

Los parámetros **$filter**, **$orderby**, y **$select** se exploran con más detalle en los siguientes artículos:

- [Sintaxis de $filter de OData en Azure Cognitive Search](search-query-odata-filter.md)
- [Sintaxis de $orderby de OData en Azure Cognitive Search](search-query-odata-orderby.md)
- [Sintaxis de $select de OData en Azure Cognitive Search](search-query-odata-select.md)

## <a name="see-also"></a>Consulte también  

- [Navegación por facetas en Azure Cognitive Search](search-faceted-navigation.md)
- [Filtros de Azure Cognitive Search](search-filters.md)
- [Búsqueda de documentos &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/Search-Documents)
- [Sintaxis de consulta de Lucene](query-lucene-syntax.md)
- [Sintaxis de consulta simple en Azure Cognitive Search](query-simple-syntax.md)