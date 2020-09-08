---
title: Uso de los filtros de colección de OData
titleSuffix: Azure Cognitive Search
description: Conozca la mecánica de cómo funcionan los filtros de recopilación de OData en consultas de Azure Cognitive Search, incluyendo las limitaciones y los comportamientos exclusivos de las colecciones.
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
ms.openlocfilehash: 6af0f2b5221a737687578e939c14cecf3be14509
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88932923"
---
# <a name="understanding-odata-collection-filters-in-azure-cognitive-search"></a>Uso de los filtros de colección de OData en Azure Cognitive Search

Para [filtrar](query-odata-filter-orderby-syntax.md) por campos de colección en Azure Cognitive Search, puede usar los operadores [`any` y `all`](search-query-odata-collection-operators.md) junto con **expresiones lambda**. Las expresiones lambda son expresiones booleanas que hacen referencia a una **variable de rango**. Los operadores `any` y `all` son similares a un bucle `for` en la mayoría de lenguajes de programación: la variable de rango adopta el rol de la variable de bucle y la expresión lambda es el cuerpo del bucle. La variable de rango toma el valor "actual" de la colección durante la iteración del bucle.

Al menos, así es cómo funciona conceptualmente. En realidad, Azure Cognitive Search implementa filtros de forma muy diferente a cómo funcionan los bucles `for`. Idealmente, esta diferencia tendría que ser invisible para usted, pero en ciertas situaciones no lo es. El resultado final es que hay reglas que se deben seguir al escribir las expresiones lambda.

En este artículo se explica por qué existen reglas para los filtros de colección, para lo cual se describe cómo se ejecutan estos filtros en Azure Cognitive Search. Si va a escribir filtros avanzados con expresiones lambda complejas, este artículo le puede ser útil para entender qué se puede hacer con los filtros y por qué.

Para información sobre qué son las reglas de los filtros de colección, con ejemplos incluidos, consulte [Solución de problemas de filtros de colección de OData en Azure Cognitive Search](search-query-troubleshoot-collection-filters.md).

## <a name="why-collection-filters-are-limited"></a>Por qué los filtros de colección son limitados

Hay tres razones subyacentes por las que no todas las características de filtro se admiten para todos los tipos de colecciones:

1. Solo se admiten determinados operadores para determinados tipos de datos. Por ejemplo, no tiene sentido comparar los valores booleanos `true` y `false` con `lt`, `gt` y así sucesivamente.
1. Azure Cognitive Search no admite la **búsqueda correlacionada** en campos de tipo `Collection(Edm.ComplexType)`.
1. Azure Cognitive Search usa índices invertidos para ejecutar los filtros en todos los tipos de datos, incluidas las colecciones.

El primer motivo es simplemente una consecuencia de cómo se definen el lenguaje OData y el sistema de tipos EDM. Los dos últimos se explican con más detalle en el resto de este artículo.

## <a name="correlated-versus-uncorrelated-search"></a>Diferencias entre búsqueda correlacionada y no correlacionada

Al aplicar varios criterios de filtro en una colección de objetos complejos, los criterios son **correlacionados**, ya que se aplican a *cada uno de los objetos de la colección*. Por ejemplo, el filtro siguiente devolverá los hoteles que tengan al menos una habitación deluxe con un precio inferior a 100:

```odata-filter-expr
    Rooms/any(room: room/Type eq 'Deluxe Room' and room/BaseRate lt 100)
```

Si el filtrado fuera *no correlacionado*, el filtro anterior podría devolver hoteles en los que una habitación es deluxe y otra diferente tiene una tarifa base inferior a 100. Eso no tendría sentido, ya que las dos cláusulas de la expresión lambda se aplican a la misma variable de rango, es decir, `room`. Por este motivo estos filtros se ponen en correlación.

Pero para la búsqueda de texto completo, no hay ninguna manera para hacer referencia a una variable de rango específica. Si usa la búsqueda clasificada por campos para emitir una [consulta completa de Lucene](query-lucene-syntax.md) como esta:

```odata-filter-expr
    Rooms/Type:deluxe AND Rooms/Description:"city view"
```

puede obtener hoteles donde una habitación sea deluxe, y en otra diferente se mencione "vistas a la ciudad" en la descripción. Por ejemplo, el documento siguiente con el `Id` de `1` coincidiría con la consulta:

```json
{
  "value": [
    {
      "Id": "1",
      "Rooms": [
        { "Type": "deluxe", "Description": "Large garden view suite" },
        { "Type": "standard", "Description": "Standard city view room" }
      ]
    },
    {
      "Id": "2",
      "Rooms": [
        { "Type": "deluxe", "Description": "Courtyard motel room" }
      ]
    }
  ]
}
```

El motivo es que `Rooms/Type` hace referencia a todos los términos analizados del campo `Rooms/Type` en todo el documento y lo mismo sucede con `Rooms/Description`, como se muestra en las tablas siguientes.

Cómo se almacena `Rooms/Type` para la búsqueda de texto completo:

| Término en `Rooms/Type` | Id. de documento |
| --- | --- |
| deluxe | 1, 2 |
| estándar | 1 |

Cómo se almacena `Rooms/Description` para la búsqueda de texto completo:

| Término en `Rooms/Description` | Id. de documento |
| --- | --- |
| courtyard | 2 |
| city | 1 |
| garden | 1 |
| large | 1 |
| motel | 2 |
| room | 1, 2 |
| estándar | 1 |
| suite | 1 |
| ver | 1 |

Por tanto, a diferencia del filtro anterior, que básicamente dice "comparar los documentos donde para una habitación `Type` es igual a "Deluxe Room" (habitación deluxe) y **esa misma habitación** tiene un valor `BaseRate` menor que 100", la consulta de búsqueda dice "comparar documentos donde `Rooms/Type` tiene el término "deluxe" y `Rooms/Description` tiene la frase "city view". En el último caso no hay ningún concepto de habitación individual cuyos campos se puedan poner en correlación.

> [!NOTE]
> Si quiere que se agregue compatibilidad con la búsqueda correlacionada a Azure Cognitive Search, vote por [este elemento en User Voice](https://feedback.azure.com/forums/263029-azure-search/suggestions/37735060-support-correlated-search-on-complex-collections).

## <a name="inverted-indexes-and-collections"></a>Colecciones e índices invertidos

Es posible que haya observado que hay muchas menos restricciones en las expresiones lambda sobre las colecciones complejas que para las colecciones simples como `Collection(Edm.Int32)`, `Collection(Edm.GeographyPoint)`, etc. El motivo es que en Azure Cognitive Search las colecciones complejas se almacenan como colecciones reales de subdocumentos, mientras que las colecciones simples no se almacenan en absoluto como colecciones.

Por ejemplo, considere un campo de la colección de cadena que se puede filtrar como `seasons` en un índice para un comerciante en línea. Algunos documentos cargados en este índice podrían tener este aspecto:

```json
{
  "value": [
    {
      "id": "1",
      "name": "Hiking boots",
      "seasons": ["spring", "summer", "fall"]
    },
    {
      "id": "2",
      "name": "Rain jacket",
      "seasons": ["spring", "fall", "winter"]
    },
    {
      "id": "3",
      "name": "Parka",
      "seasons": ["winter"]
    }
  ]
}
```

Los valores del campo `seasons` se almacenan en una estructura denominada **índice invertido**, similar a lo siguiente:

| Término | Id. de documento |
| --- | --- |
| spring | 1, 2 |
| summer | 1 |
| fall | 1, 2 |
| winter | 2, 3 |

Esta estructura de datos está diseñada para responder una pregunta a gran velocidad: ¿En qué documentos aparece un término determinado? La respuesta a esta pregunta funciona más como una sencilla comprobación de igualdad que un bucle a través de una colección. De hecho, este es el motivo de que Azure Cognitive Search solo permita `eq` como operador de comparación dentro de una expresión lambda `any` en las colecciones de cadenas.

Partiendo de la igualdad, a continuación se verá cómo se pueden combinar varias comprobaciones de igualdad en la misma variable de rango con `or`. Funciona gracias al álgebra y a [la propiedad distributiva de los cuantificadores](https://en.wikipedia.org/wiki/Existential_quantification#Negation). Esta expresión:

```odata-filter-expr
    seasons/any(s: s eq 'winter' or s eq 'fall')
```

equivale a:

```odata-filter-expr
    seasons/any(s: s eq 'winter') or seasons/any(s: s eq 'fall')
```

y cada una de las dos subexpresiones `any` se puede ejecutar de forma eficaz mediante el índice invertido. Además, gracias a [la ley de negación de cuantificadores](https://en.wikipedia.org/wiki/Existential_quantification#Negation), esta expresión:

```odata-filter-expr
    seasons/all(s: s ne 'winter' and s ne 'fall')
```

equivale a:

```odata-filter-expr
    not seasons/any(s: s eq 'winter' or s eq 'fall')
```

motivo por el que se puede usar `all` con `ne` y `and`.

> [!NOTE]
> Aunque los detalles están fuera del ámbito de este documento, estos mismos principios también se extienden a [las pruebas de distancia e intersección para colecciones de puntos geoespaciales](search-query-odata-geo-spatial-functions.md). Por este motivo, en `any`:
>
> - `geo.intersects` no se puede negar
> - `geo.distance` se debe comparar con `lt` o `le`
> - las expresiones se deben combinar con `or`, no con `and`
>
> Para `all` se aplican las reglas inversas.

Al filtrar por colecciones de tipos de datos que admiten los operadores `lt`, `gt`, `le` y `ge` (como por ejemplo `Collection(Edm.Int32)`) se admite una mayor variedad de expresiones. En concreto, en `any` se pueden usar `and` y `or`, siempre y cuando las expresiones de comparación subyacentes se combinen en **comparaciones de rango** mediante `and`, que después se vuelven a combinar con `or`. Esta estructura de expresiones booleanas se denomina [forma normal disyuntiva (DNF)](https://en.wikipedia.org/wiki/Disjunctive_normal_form), también conocida como "OR de AND". Por el contrario, las expresiones lambda para `all` para estos tipos de datos deben estar en [forma normal conjuntiva (CNF)](https://en.wikipedia.org/wiki/Conjunctive_normal_form), también conocida como "AND de OR". Azure Cognitive Search permite estas comparaciones de rango porque las puede ejecutar de forma eficaz mediante índices invertidos, al igual que puede realizar búsquedas de términos rápidas para las cadenas.

En resumen, estas son las reglas generales para lo que se permite en una expresión lambda:

- Dentro de `any`, las *comprobaciones positivas* siempre se permiten, como las comparaciones de rango de igualdad, `geo.intersects`, o bien `geo.distance` comparado con `lt` o `le` (imagine que la "proximidad" es similar a la igualdad cuando se comprueba la distancia).
- Dentro de `any`, `or` siempre se permite. Solo se puede usar `and` para los tipos de datos que pueden expresar comprobaciones de intervalo, y solo si se usan operadores OR de AND (DNF).
- Dentro de `all`, las reglas se invierten: solo se permiten *comprobaciones negativas*, siempre se puede usar `and` y `or` solo se puede usar para las comprobaciones de rango expresadas como operadores AND de OR (CNF).

De todos modos, en la práctica estos son los tipos de filtros que es más probable que use. Pero sigue siendo útil comprender los límites de lo que es posible.

Para obtener ejemplos específicos de qué tipos de filtros se permiten y cuáles no, vea [Procedimientos para escribir filtros de colección válidos](search-query-troubleshoot-collection-filters.md#bkmk_examples).

## <a name="next-steps"></a>Pasos siguientes  

- [Solución de problemas de filtros de colección de OData en Azure Cognitive Search](search-query-troubleshoot-collection-filters.md)
- [Filtros de Azure Cognitive Search](search-filters.md)
- [Información general sobre el lenguaje de expresiones OData para Azure Cognitive Search](query-odata-filter-orderby-syntax.md)
- [Referencia de sintaxis de expresiones OData para Azure Cognitive Search](search-query-odata-syntax-reference.md)
- [Búsqueda de documentos &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/Search-Documents)