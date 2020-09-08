---
title: Referencia de select de OData
titleSuffix: Azure Cognitive Search
description: Referencia de sintaxis y lenguaje para la selección explícita de campos que se van a devolver en los resultados de búsqueda de las consultas de Azure Cognitive Search.
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
ms.openlocfilehash: 54b6ae227fc4b3b951717799660543c02874dda0
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919665"
---
# <a name="odata-select-syntax-in-azure-cognitive-search"></a>Sintaxis de $select de OData en Azure Cognitive Search

 Puede usar el [parámetro **$select** de OData](query-odata-filter-orderby-syntax.md) para elegir qué campos quiere incluir en los resultados de la búsqueda de Azure Cognitive Search. En este artículo se describe con detalle la sintaxis de **$select**. Para obtener información general sobre cómo usar **$select** al presentar los resultados de la búsqueda, consulte [Uso de los resultados de la búsqueda en Azure Cognitive Search](search-pagination-page-layout.md).

## <a name="syntax"></a>Sintaxis

El parámetro **$select** determina qué campos de cada documento se devuelven en el conjunto de resultados de la consulta. La siguiente EBNF ([notación de Backus-Naur extendida](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) define la gramática del parámetro **$select**:

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
select_expression ::= '*' | field_path(',' field_path)*

field_path ::= identifier('/'identifier)*
```

También está disponible un diagrama de sintaxis interactivo:

> [!div class="nextstepaction"]
> [Diagrama de la sintaxis de OData para Azure Cognitive Search](https://azuresearch.github.io/odata-syntax-diagram/#select_expression)

> [!NOTE]
> Consulte [Referencia de la sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md) para obtener la EBNF completa.

El parámetro **$select** tiene dos formatos:

1. Una sola estrella (`*`), que indica que se deben devolver todos los campos recuperables, o bien
1. Una lista separada por comas de las rutas de acceso de campo, para identificar qué campos se deben devolver.

Al usar el segundo formato, en la lista solo se pueden especificar campos recuperables.

Si se indica un campo complejo sin especificar sus subcampos de forma explícita, todos los campos secundarios recuperables se incluirán en el conjunto de resultados de la consulta. Por ejemplo, imagine que el índice tiene un campo `Address` con los subcampos `Street`, `City` y `Country`, todos ellos recuperables. Si se especifica `Address` en **$select**, en los resultados de la consulta se incluirán los tres subcampos.

## <a name="examples"></a>Ejemplos

En los resultados se incluyen los campos de nivel superior `HotelId`, `HotelName` y `Rating`, así como el subcampo `City` de `Address`:

```odata-filter-expr
    $select=HotelId, HotelName, Rating, Address/City
```

Un resultado de ejemplo podría tener este aspecto:

```json
{
  "HotelId": "1",
  "HotelName": "Secret Point Motel",
  "Rating": 4,
  "Address": {
    "City": "New York"
  }
}
```

En los resultados se incluye el campo de nivel superior `HotelName`, así como todos los subcampos de `Address` y los subcampos `Type` y `BaseRate` de todos los objetos de la colección `Rooms`:

```odata-filter-expr
    $select=HotelName, Address, Rooms/Type, Rooms/BaseRate
```

Un resultado de ejemplo podría tener este aspecto:

```json
{
  "HotelName": "Secret Point Motel",
  "Rating": 4,
  "Address": {
    "StreetAddress": "677 5th Ave",
    "City": "New York",
    "StateProvince": "NY",
    "Country": "USA",
    "PostalCode": "10022"
  },
  "Rooms": [
    {
      "Type": "Budget Room",
      "BaseRate": 9.69
    },
    {
      "Type": "Budget Room",
      "BaseRate": 8.09
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes  

- [Procedimientos para trabajar con los resultados de búsqueda en Azure Cognitive Search](search-pagination-page-layout.md)
- [Información general sobre el lenguaje de expresiones OData para Azure Cognitive Search](query-odata-filter-orderby-syntax.md)
- [Referencia de sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md)
- [Búsqueda de documentos &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/Search-Documents)