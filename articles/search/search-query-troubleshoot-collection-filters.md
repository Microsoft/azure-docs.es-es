---
title: Solución de problemas de filtros de colección de OData
titleSuffix: Azure Cognitive Search
description: Conozca los enfoques para solucionar errores de filtros de colección de OData en consultas de Azure Cognitive Search.
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
ms.openlocfilehash: 3050f701c11773207aa6054d4d08d908d87b2ce7
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88932073"
---
# <a name="troubleshooting-odata-collection-filters-in-azure-cognitive-search"></a>Solución de problemas de filtros de colección de OData en Azure Cognitive Search

Para [filtrar](query-odata-filter-orderby-syntax.md) por campos de colección en Azure Cognitive Search, puede usar los operadores [`any` y `all`](search-query-odata-collection-operators.md) junto con **expresiones lambda**. Una expresión lambda es un subfiltro que se aplica a cada elemento de una colección.

No todas las características de las expresiones de filtro están disponibles dentro de una expresión lambda. Las características disponibles difieren en función del tipo de datos del campo de colección que se quiere filtrar. Esto puede producir un error si se intenta usar una característica en una expresión lambda que no se admite en ese contexto. Si estos errores se producen al intentar escribir un filtro complejo sobre campos de colección, este artículo le ayudará a solucionar el problema.

## <a name="common-collection-filter-errors"></a>Errores de filtro de colección comunes

En la tabla siguiente se enumeran los errores que pueden surgir al intentar ejecutar un filtro de colección. Estos errores se producen cuando se usa una característica de las expresiones de filtro que no se admite dentro de una expresión lambda. Cada error proporciona instrucciones sobre cómo puede volver a escribir el filtro para evitar el error. En la tabla también se incluye un vínculo a la sección correspondiente de este artículo en la que se proporciona más información sobre cómo evitar este error.

| Mensaje de error | Situación | Para obtener más información, vea |
| --- | --- | --- |
| La función "ismatch" no tiene ningún parámetro enlazado a la variable de rango "s". Dentro de las expresiones lambda solo se admiten referencias de campo enlazadas ("any" o "all"). Cambie el filtro para que la función "ismatch" esté fuera de la expresión lambda y vuelva a intentarlo. | Uso de `search.ismatch` o `search.ismatchscoring` dentro de una expresión lambda | [Reglas de filtrado de colecciones complejas](#bkmk_complex) |
| Expresión lambda no válida. Se ha encontrado una prueba de igualdad o desigualdad cuando se esperaba lo contrario en una expresión lambda que recorre en iteración un campo de tipo Collection(Edm.String). Para "any", use expresiones con el formato "x eq y" o "search.in(...)". Para "all", use expresiones con el formato "x ne y", "not (x eq y)" o "not search.in(...)". | Filtrado por un campo de tipo `Collection(Edm.String)` | [Reglas de filtrado de colecciones de cadenas](#bkmk_strings) |
| Expresión lambda no válida. Se ha encontrado un formato de expresión booleana compleja no admitido. Para "any", use expresiones que sean "OR de AND", también conocidas como forma normal disyuntiva. Por ejemplo: "(a and b) or (c y d)", donde a, b, c y d son las subexpresiones de comparación o igualdad. Para "all", use expresiones que sean "AND de OR", también conocidas como forma normal conjuntiva. Por ejemplo: "(a or b) and (c or d)", donde a, b, c y d son subexpresiones de comparación o desigualdad. Ejemplos de expresiones de comparación: "x gt 5", "x le 2". Ejemplo de una expresión de igualdad: "x eq 5". Ejemplo de una expresión de desigualdad: "x ne 5". | Filtrado por campos de tipo `Collection(Edm.DateTimeOffset)`, `Collection(Edm.Double)`, `Collection(Edm.Int32)` o `Collection(Edm.Int64)` | [Reglas de filtrado de colecciones comparables](#bkmk_comparables) |
| Expresión lambda no válida. Se ha encontrado un uso no admitido de geo.distance() o geo.intersects() en una expresión lambda que recorre en iteración un campo de tipo Collection(Edm.GeographyPoint). Para "any", asegúrese de comparar geo.distance() con los operadores "lt" o "le", y de que el uso de geo.intersects() no sea negativo. Para "all", asegúrese de comparar geo.distance() con los operadores "gt" o "ge", y de que el uso de geo.intersects() sea negativo. | Filtrado por un campo de tipo `Collection(Edm.GeographyPoint)` | [Reglas de filtrado para colecciones GeographyPoint](#bkmk_geopoints) |
| Expresión lambda no válida. No se admiten expresiones booleanas complejas en expresiones lambda que recorren en iteración campos de tipo Collection(Edm.GeographyPoint). Para "any", combine las subexpresiones con "or"; no se admite "and". Para "all", combine las subexpresiones con "and"; no se admite "or". | Filtrado por campos de tipo `Collection(Edm.String)` o `Collection(Edm.GeographyPoint)` | [Reglas de filtrado de colecciones de cadenas](#bkmk_strings) <br/><br/> [Reglas de filtrado para colecciones GeographyPoint](#bkmk_geopoints) |
| Expresión lambda no válida. Se ha encontrado un operador de comparación (uno de "lt", "le", "gt" o "ge"). Solo se permiten operadores de igualdad en expresiones lambda que recorren en iteración los campos de tipo Collection(Edm.String). Para "any", use expresiones con el formato "x eq y". Para "all", use expresiones con el formato "x ne y" o "not (x eq y)". | Filtrado por un campo de tipo `Collection(Edm.String)` | [Reglas de filtrado de colecciones de cadenas](#bkmk_strings) |

<a name="bkmk_examples"></a>

## <a name="how-to-write-valid-collection-filters"></a>Procedimientos para escribir filtros de colección válidos

Las reglas para escribir filtros de colección válidos son diferentes para cada tipo de datos. En las secciones siguientes se describen las reglas mediante ejemplos de las características de filtro que se admiten y las que no:

- [Reglas de filtrado de colecciones de cadenas](#bkmk_strings)
- [Reglas de filtrado de colecciones booleanas](#bkmk_bools)
- [Reglas de filtrado para colecciones GeographyPoint](#bkmk_geopoints)
- [Reglas de filtrado de colecciones comparables](#bkmk_comparables)
- [Reglas de filtrado de colecciones complejas](#bkmk_complex)

<a name="bkmk_strings"></a>

## <a name="rules-for-filtering-string-collections"></a>Reglas de filtrado de colecciones de cadenas

Dentro de las expresiones lambda para colecciones de cadenas, los únicos operadores de comparación que se pueden usar son `eq` y `ne`.

> [!NOTE]
> Azure Cognitive Search no admite los operadores `lt`/`le`/`gt`/`ge` para las cadenas, ya sea dentro o fuera de una expresión lambda.

El cuerpo de una expresión `any` solo se puede probar para igualdad, mientras el cuerpo de una expresión `all` solo se puede probar para la desigualdad.

También se pueden combinar varias expresiones a través de `or` en el cuerpo de `any`, y a través de `and` en el cuerpo de `all`. Como la función `search.in` equivale a combinar comprobaciones de igualdad con `or`, también se permite en el cuerpo de una expresión `any`. Por el contrario, `not search.in` se permite en el cuerpo de una expresión `all`.

Por ejemplo, se permiten estas expresiones:

- `tags/any(t: t eq 'books')`
- `tags/any(t: search.in(t, 'books, games, toys'))`
- `tags/all(t: t ne 'books')`
- `tags/all(t: not (t eq 'books'))`
- `tags/all(t: not search.in(t, 'books, games, toys'))`
- `tags/any(t: t eq 'books' or t eq 'games')`
- `tags/all(t: t ne 'books' and not (t eq 'games'))`

mientras que estas expresiones no se permiten:

- `tags/any(t: t ne 'books')`
- `tags/any(t: not search.in(t, 'books, games, toys'))`
- `tags/all(t: t eq 'books')`
- `tags/all(t: search.in(t, 'books, games, toys'))`
- `tags/any(t: t eq 'books' and t ne 'games')`
- `tags/all(t: t ne 'books' or not (t eq 'games'))`

<a name="bkmk_bools"></a>

## <a name="rules-for-filtering-boolean-collections"></a>Reglas de filtrado de colecciones booleanas

El tipo `Edm.Boolean` solo admite los operadores `eq` y `ne`. Por tanto, no tiene mucho sentido permitir la combinación de estas cláusulas que comprueban la misma variable de rango con `and`/`or`, ya que eso siempre produciría tautologías o contradicciones.

Estos son algunos ejemplos de filtros permitidos en colecciones booleanas:

- `flags/any(f: f)`
- `flags/all(f: f)`
- `flags/any(f: f eq true)`
- `flags/any(f: f ne true)`
- `flags/all(f: not f)`
- `flags/all(f: not (f eq true))`

A diferencia de las colecciones de cadenas, las booleanas no tienen ningún límite con respecto a qué operador se puede usar en cada tipo de expresión lambda. Tanto `eq` como `ne` se pueden usar en el cuerpo de `any` o `all`.

Para las colecciones booleanas no se permiten expresiones como las siguientes:

- `flags/any(f: f or not f)`
- `flags/any(f: f or f)`
- `flags/all(f: f and not f)`
- `flags/all(f: f and f eq true)`

<a name="bkmk_geopoints"></a>

## <a name="rules-for-filtering-geographypoint-collections"></a>Reglas de filtrado para colecciones GeographyPoint

Los valores de tipo `Edm.GeographyPoint` en una colección no se pueden comparar directamente entre sí. En su lugar, se deben usar como parámetros de las funciones `geo.distance` y `geo.intersects`. A su vez, la función `geo.distance` se debe comparar con un valor de distancia mediante uno de los operadores de comparación `lt`, `le`, `gt` o `ge`. Estas reglas también se aplican a los campos Edm.GeographyPoint que no sean de colección.

Al igual que las colecciones de cadenas, las colecciones `Edm.GeographyPoint` tienen algunas reglas sobre cómo se pueden usar y combinar las funciones geoespaciales en los diferentes tipos de expresiones lambda:

- Qué operadores de comparación se pueden usar con la función `geo.distance` depende del tipo de expresión lambda. Para `any`, solo puede usar `lt` o `le`. Para `all`, solo puede usar `gt` o `ge`. Puede negar las expresiones que contengan `geo.distance`, pero tendrá que cambiar el operador de comparación (`geo.distance(...) lt x` se convierte en `not (geo.distance(...) ge x)` y `geo.distance(...) le x` en `not (geo.distance(...) gt x)`).
- En el cuerpo de una expresión `all`, la función `geo.intersects` debe ser negativa. Por el contrario, en el cuerpo de una expresión `any`, la función `geo.intersects` no debe ser negativa.
- En el cuerpo de `any`, las expresiones geoespaciales se pueden combinar mediante `or`. En el cuerpo de `all`, ese tipo de expresiones se pueden combinar mediante `and`.

Las limitaciones anteriores existen por motivos similares a la limitación de igualdad y desigualdad en las colecciones de cadenas. Consulte [Descripción de los filtros de colección de OData en Azure Cognitive Search](search-query-understand-collection-filters.md) para obtener una visión más profunda de estos motivos.

Estos son algunos ejemplos de filtros permitidos en colecciones `Edm.GeographyPoint`:

- `locations/any(l: geo.distance(l, geography'POINT(-122 49)') lt 10)`
- `locations/any(l: not (geo.distance(l, geography'POINT(-122 49)') ge 10) or geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`
- `locations/all(l: geo.distance(l, geography'POINT(-122 49)') ge 10 and not geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`

Para las colecciones `Edm.GeographyPoint` no se permiten expresiones como las siguientes:

- `locations/any(l: l eq geography'POINT(-122 49)')`
- `locations/any(l: not geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`
- `locations/all(l: geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`
- `locations/any(l: geo.distance(l, geography'POINT(-122 49)') gt 10)`
- `locations/all(l: geo.distance(l, geography'POINT(-122 49)') lt 10)`
- `locations/any(l: geo.distance(l, geography'POINT(-122 49)') lt 10 and geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`
- `locations/all(l: geo.distance(l, geography'POINT(-122 49)') le 10 or not geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`

<a name="bkmk_comparables"></a>

## <a name="rules-for-filtering-comparable-collections"></a>Reglas de filtrado de colecciones comparables

Esta sección se aplica a todos los tipos de datos siguientes:

- `Collection(Edm.DateTimeOffset)`
- `Collection(Edm.Double)`
- `Collection(Edm.Int32)`
- `Collection(Edm.Int64)`

Los tipos como `Edm.Int32` y `Edm.DateTimeOffset` admiten los seis operadores de comparación: `eq`, `ne`, `lt`, `le`, `gt` y `ge`. Las expresiones lambda de colecciones de estos tipos pueden incluir expresiones simples en las que se use cualquiera de estos operadores. Esto se aplica a `any` y `all`. Por ejemplo, se permiten estos filtros:

- `ratings/any(r: r ne 5)`
- `dates/any(d: d gt 2017-08-24T00:00:00Z)`
- `not margins/all(m: m eq 3.5)`

Pero existen limitaciones con respecto a cómo se pueden combinar estas expresiones de comparación en expresiones más complejas dentro de una expresión lambda:

- Reglas para `any`:
  - Las expresiones de desigualdad simples no se pueden combinar de forma útil con otras expresiones. Por ejemplo, se permite esta expresión:
    - `ratings/any(r: r ne 5)`

    pero no esta expresión:
    - `ratings/any(r: r ne 5 and r gt 2)`

    y aunque se permite esta expresión, no es útil porque las condiciones se superponen:
    - `ratings/any(r: r ne 5 or r gt 7)`
  - Las expresiones de comparación simples que contengan `eq`, `lt`, `le`, `gt` o `ge` se pueden combinar con `and`/`or`. Por ejemplo:
    - `ratings/any(r: r gt 2 and r le 5)`
    - `ratings/any(r: r le 5 or r gt 7)`
  - Las expresiones de comparación combinadas con `and` (conjunciones) se pueden combinar aún más mediante `or`. En lógica booleana, este formato se conoce como "[forma normal disyuntiva](https://en.wikipedia.org/wiki/Disjunctive_normal_form)" (DNF). Por ejemplo:
    - `ratings/any(r: (r gt 2 and r le 5) or (r gt 7 and r lt 10))`
- Reglas para `all`:
  - Las expresiones de igualdad simples no se pueden combinar de forma útil con ninguna otra expresión. Por ejemplo, se permite esta expresión:
    - `ratings/all(r: r eq 5)`

    pero no esta expresión:
    - `ratings/all(r: r eq 5 or r le 2)`

    y aunque se permite esta expresión, no es útil porque las condiciones se superponen:
    - `ratings/all(r: r eq 5 and r le 7)`
  - Las expresiones de comparación simples que contengan `ne`, `lt`, `le`, `gt` o `ge` se pueden combinar con `and`/`or`. Por ejemplo:
    - `ratings/all(r: r gt 2 and r le 5)`
    - `ratings/all(r: r le 5 or r gt 7)`
  - Las expresiones de comparación combinadas con `or` (disyunciones) se pueden combinar aún más mediante `and`. En lógica booleana, este formato se conoce como "[forma normal conjuntiva](https://en.wikipedia.org/wiki/Conjunctive_normal_form)" (CNF). Por ejemplo:
    - `ratings/all(r: (r le 2 or gt 5) and (r lt 7 or r ge 10))`

<a name="bkmk_complex"></a>

## <a name="rules-for-filtering-complex-collections"></a>Reglas de filtrado de colecciones complejas

Las expresiones lambda sobre colecciones complejas admiten una sintaxis mucho más flexible que las expresiones lambda sobre colecciones de tipos primitivos. Dentro de una expresión lambda puede usar cualquier construcción de filtro que se pueda usar fuera de ella, con solo dos excepciones.

En primer lugar, las funciones `search.ismatch` y `search.ismatchscoring` no se admiten dentro de las expresiones lambda. Para más información, consulte [Descripción de los filtros de colección de OData en Azure Cognitive Search](search-query-understand-collection-filters.md).

En segundo lugar, los campos de referencia que no están *enlazados* a la variable de rango (denominados *variables independientes*) no están permitidos. Por ejemplo, fíjese en las dos siguientes expresiones de filtro de OData equivalentes:

1. `stores/any(s: s/amenities/any(a: a eq 'parking')) and details/margin gt 0.5`
1. `stores/any(s: s/amenities/any(a: a eq 'parking' and details/margin gt 0.5))`

La primera expresión se permitirá, mientras que la segunda se rechazará porque `details/margin` no está enlazado a la variable de rango `s`.

Esta regla también se extiende a las expresiones que tienen variables enlazadas en un ámbito externo. Estas variables son independientes con respecto al ámbito en el que aparecen. Por ejemplo, se permite la primera expresión, mientras que la segunda expresión equivalente no, porque `s/name` es independiente con respecto al ámbito de la variable de rango `a`:

1. `stores/any(s: s/amenities/any(a: a eq 'parking') and s/name ne 'Flagship')`
1. `stores/any(s: s/amenities/any(a: a eq 'parking' and s/name ne 'Flagship'))`

Esta limitación no debería ser un problema en la práctica, ya que siempre se pueden construir filtros de forma que las expresiones lambda solo contengan variables enlazadas.

## <a name="cheat-sheet-for-collection-filter-rules"></a>Hoja de referencia rápida de reglas de filtro de colección

En la tabla siguiente se resumen las reglas para construir filtros válidos para cada tipo de datos de la colección.

[!INCLUDE [Limitations on OData lambda expressions in Azure Cognitive Search](../../includes/search-query-odata-lambda-limitations.md)]

Para obtener ejemplos de cómo crear filtros válidos para cada caso, vea [Procedimientos para escribir filtros de colección válidos](#bkmk_examples).

Si escribe filtros con frecuencia, y entender las reglas de los primeros principios le ayudaría a algo más que memorizarlas simplemente, consulte [Descripción de los filtros de colección de OData en Azure Cognitive Search](search-query-understand-collection-filters.md).

## <a name="next-steps"></a>Pasos siguientes  

- [Descripción de los filtros de colección de OData en Azure Cognitive Search](search-query-understand-collection-filters.md)
- [Filtros de Azure Cognitive Search](search-filters.md)
- [Información general sobre el lenguaje de expresiones OData para Azure Cognitive Search](query-odata-filter-orderby-syntax.md)
- [Referencia de sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md)
- [Búsqueda de documentos &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/Search-Documents)