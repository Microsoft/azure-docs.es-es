---
title: Referencia de los operadores de colección de OData
titleSuffix: Azure Cognitive Search
description: Al crear expresiones de filtro en las consultas de Azure Cognitive Search, use los operadores "any" y "all" en expresiones lambda cuando el filtro se encuentre en un campo de colección o un campo de colección complejo.
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
ms.openlocfilehash: 4c1243d5d9122539466e94b6bbfdd5ced588e69a
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934912"
---
# <a name="odata-collection-operators-in-azure-cognitive-search---any-and-all"></a>Operadores de colección de OData en Azure Cognitive Search: `any` y `all`

Al escribir una [expresión de filtro de OData](query-odata-filter-orderby-syntax.md) para usarla con Azure Cognitive Search, a menudo es útil filtrar por los campos de la colección. Esto se puede conseguir con los operadores `any` y `all`.

## <a name="syntax"></a>Sintaxis

En la siguiente EBNF ([forma de Backus-Naur extendida](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) se define la gramática de una expresión de OData en la que se usa `any` o `all`.

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
collection_filter_expression ::=
    field_path'/all(' lambda_expression ')'
    | field_path'/any(' lambda_expression ')'
    | field_path'/any()'

lambda_expression ::= identifier ':' boolean_expression
```

También está disponible un diagrama de sintaxis interactivo:

> [!div class="nextstepaction"]
> [Diagrama de la sintaxis de OData para Azure Cognitive Search](https://azuresearch.github.io/odata-syntax-diagram/#collection_filter_expression)

> [!NOTE]
> Consulte [Referencia de la sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md) para obtener la EBNF completa.

Hay tres formas de expresión que filtran las colecciones.

- Las dos primeras recorren en iteración un campo de colección y aplican un predicado con forma de expresión lambda a cada elemento de la colección.
  - Una expresión que use `all` devuelve `true` si el predicado es true para todos los elementos de la colección.
  - Una expresión que use `any` devuelve `true` si el predicado es true al menos para un elemento de la colección.
- En la tercera forma de filtro de colección se usa `any` sin una expresión lambda para comprobar si un campo de la colección está vacío. Si la colección tiene algún elemento, devuelve `true`. Si la colección está vacía, devuelve `false`.

Una **expresión lambda** en un filtro de colección es como el cuerpo de un bucle en un lenguaje de programación. Define una variable, denominada **variable de rango**, que contiene el elemento actual de la colección durante la iteración. También define otra expresión booleana que son los criterios de filtro que se van a aplicar a la variable de rango para cada elemento de la colección.

## <a name="examples"></a>Ejemplos

Comparar documentos cuyo campo `tags` contenga exactamente la cadena "wifi":

```text
tags/any(t: t eq 'wifi')
```

Comparar documentos donde todos los elementos del campo `ratings` estén comprendidos entre 3 y 5, inclusive:

```text
ratings/all(r: r ge 3 and r le 5)
```

Comparar documentos donde cualquiera de las geocoordenadas del campo `locations` esté dentro del polígono indicado:

```text
locations/any(loc: geo.intersects(loc, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))
```

Comparar documentos donde el campo `rooms` está vacío:

```text
not rooms/any()
```

Comparar documentos donde, para todas las habitaciones, el campo `rooms/amenities` contiene "tv" y `rooms/baseRate` es inferior a 100:

```text
rooms/all(room: room/amenities/any(a: a eq 'tv') and room/baseRate lt 100.0)
```

## <a name="limitations"></a>Limitaciones

No todas las características de las expresiones de filtro están disponibles dentro del cuerpo de una expresión lambda. Las limitaciones varían según el tipo de datos del campo de colección que se quiere filtrar. En la tabla siguiente se resumen las limitaciones.

[!INCLUDE [Limitations on OData lambda expressions in Azure Cognitive Search](../../includes/search-query-odata-lambda-limitations.md)]

Para más información sobre estas limitaciones y ver ejemplos, consulte [Solución de problemas de los filtros de colección en Azure Cognitive Search](search-query-troubleshoot-collection-filters.md). Para más información sobre por qué existen estas limitaciones, consulte [Descripción de los filtros de colección en Azure Cognitive Search](search-query-understand-collection-filters.md).

## <a name="next-steps"></a>Pasos siguientes  

- [Filtros de Azure Cognitive Search](search-filters.md)
- [Información general sobre el lenguaje de expresiones OData para Azure Cognitive Search](query-odata-filter-orderby-syntax.md)
- [Referencia de sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md)
- [Búsqueda de documentos &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/Search-Documents)