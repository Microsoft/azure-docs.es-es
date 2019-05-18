---
title: 'Filtros para definir el ámbito de los resultados de búsqueda en un índice: Azure Search'
description: Puede filtrar por identidad de seguridad del usuario, idioma, geolocalización o valores numéricos para reducir los resultados de búsqueda para las consultas de Azure Search, un servicio de búsqueda en la nube hospedado de Microsoft Azure.
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/13/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 1871fee2734d347ff54d6aa70d90d1c28bd1f6f1
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65597287"
---
# <a name="filters-in-azure-search"></a>Filtros de Azure Search 

Un *filtro* proporciona criterios para seleccionar los documentos que se usan en una consulta de Azure Search. La búsqueda sin filtrar incluye todos los documentos del índice. Un filtro delimita el ámbito de una consulta de búsqueda a un subconjunto de documentos. Por ejemplo, un filtro puede restringir la búsqueda de texto completo a aquellos productos que tienen una marca o un color específicos y un precio de venta al público superior a un umbral determinado.

Algunas experiencias de búsqueda imponen los requisitos de filtro como parte de la implementación, pero puede usar filtros siempre que quiera restringir la búsqueda mediante criterios *basados en valores*; por ejemplo, puede delimitar la búsqueda al tipo de producto "libros" de la categoría "no ficción" publicados por "Simon & Schuster".

Si su objetivo es la búsqueda orientada a *estructuras* de datos específicos (limitar la búsqueda a un campo de opiniones de cliente, por ejemplo), existen métodos alternativos y se describen a continuación.

## <a name="when-to-use-a-filter"></a>Cuándo se debe usar un filtro

Los filtros son fundamentales para varias experiencias de búsqueda, como la búsqueda de servicios cercanos, la navegación por facetas y los filtros de seguridad que muestran únicamente los documentos que un usuario tiene permiso para ver. Si implementa alguna de estas experiencias, se necesita un filtro. El filtro asociado a la consulta de búsqueda proporciona coordenadas de geolocalización, la categoría de faceta que seleccionó el usuario o el id. de seguridad del solicitante.

A continuación se incluyen algunos escenarios de ejemplo:

1. Use un filtro para segmentar el índice según los valores de datos del índice. A partir de un esquema con valores de ciudad, tipo de alojamiento y servicios, puede crear un filtro para seleccionar explícitamente los documentos que cumplan los criterios (en Seattle, pisos, frente al mar). 

   Las búsquedas de texto completo con las mismas entradas a menudo generan resultados similares, pero un filtro es más preciso, ya que requiere a una coincidencia exacta del término del filtro en el contenido del índice. 

2. Use un filtro si la experiencia de búsqueda tiene un requisito de filtro:

   * La [navegación por facetas](search-faceted-navigation.md) usa un filtro para devolver la categoría de faceta que seleccionó el usuario.
   * La búsqueda georreferenciada usa un filtro para pasar las coordenadas de la ubicación actual en aplicaciones que buscan servicios cercanos. 
   * Los filtros de seguridad pasan los identificadores de seguridad como criterios de filtro, donde una coincidencia en el índice actúa como un proxy para los derechos de acceso al documento.

3. Use un filtro para aplicar criterios de búsqueda a un campo numérico. 

   Los campos numéricos se pueden recuperar en el documento y pueden aparecer en los resultados de búsqueda, pero no se pueden hacer búsquedas (de texto completo) en ellos individualmente. Si necesita criterios de selección basados en datos numéricos, use un filtro.

### <a name="alternative-methods-for-reducing-scope"></a>Métodos alternativos para reducir el ámbito

Si quiere un efecto de restricción en los resultados de búsqueda, los filtros no son la única opción. Estas alternativas podrían ser una solución mejor, en función de su objetivo:

 + El parámetro de consulta `searchFields` delimita la consulta a unos campos específicos. Por ejemplo, si el índice proporciona campos independientes para descripciones en inglés y español, puede usar searchFields para definir como destino los campos que quiera usar para la búsqueda de texto completo. 

+ El parámetro `$select` se usa para especificar qué campos se deben incluir en un conjunto de resultados, lo que recorta de forma eficaz la respuesta antes de enviarla a la aplicación que realiza la llamada. Este parámetro no refina la consulta ni reducir la colección de documentos, pero si una respuesta más pequeña es el objetivo, este parámetro es una opción a tener en cuenta. 

Para obtener más información acerca de estos parámetros, consulte [Search Documents > Request > Query parameters](https://docs.microsoft.com/rest/api/searchservice/search-documents#request) (Buscar en documentos > Solicitud > Parámetros de consulta).


## <a name="how-filters-are-executed"></a>Cómo se ejecutan los filtros

En el momento de la consulta, un analizador de filtro acepta criterios como entrada, convierte la expresión en expresiones booleanas atómicas representadas como un árbol y, a continuación, se evalúa como el árbol de filtros en campos filtrables de un índice.

El filtrado se produce en tándem con la búsqueda, define los documentos que va a incluir en el procesamiento de bajada para recuperación de documentos y la puntuación de importancia. Cuando se emparejan con una cadena de búsqueda, el filtro reduce de forma eficaz el conjunto de recuperación de la operación de búsqueda subsiguientes. Cuando se usa solo (por ejemplo, cuando la cadena de consulta está vacía en `search=*`), los criterios de filtro son la única entrada. 

## <a name="defining-filters"></a>Definición de filtros

Los filtros son expresiones de OData, articuladas mediante un [subconjunto de sintaxis de OData V4 admitido en Azure Search](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search). 

Puede especificar un filtro para cada **búsqueda** operación, pero el filtro en sí puede incluir varios campos, varios criterios, y si usa un **ismatch** (función), varias expresiones de búsqueda de texto completo. En una expresión de filtro de varias partes, puede especificar predicados en cualquier orden (de acuerdo con las reglas de precedencia de operadores). No hay ninguna mejora apreciable del rendimiento si intenta reorganizar los predicados en una secuencia determinada.

Uno de los límites de una expresión de filtro es el límite de tamaño máximo de la solicitud. La solicitud completa, incluido el filtro, puede tener un máximo de 16 MB para POST u 8 KB para GET. También hay un límite en el número de cláusulas de la expresión de filtro. Como regla general, si tiene cientos de cláusulas, está en riesgo de alcanzar el límite. Es recomendable diseñar la aplicación de manera que no genere filtros de tamaño ilimitado.

Los ejemplos siguientes representan definiciones de filtro prototípicas en varias API.

```http
# Option 1:  Use $filter for GET
GET https://[service name].search.windows.net/indexes/hotels/docs?search=*&$filter=baseRate lt 150&$select=hotelId,description&api-version=2019-05-06

# Option 2: Use filter for POST and pass it in the request body
POST https://[service name].search.windows.net/indexes/hotels/docs/search?api-version=2019-05-06
{
    "search": "*",
    "filter": "baseRate lt 150",
    "select": "hotelId,description"
}
```

```csharp
    parameters =
        new SearchParameters()
        {
            Filter = "baseRate lt 150",
            Select = new[] { "hotelId", "description" }
        };

    var results = searchIndexClient.Documents.Search("*", parameters);
```

## <a name="filter-usage-patterns"></a>Patrones de uso de filtro

Los ejemplos siguientes muestran varios patrones de uso para escenarios de filtro. Para obtener más ideas, consulte [OData expression syntax > Examples](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search#filter-examples) (Sintaxis de expresiones de OData > Ejemplos).

+ La expresión **$filter** independiente, sin una cadena de consulta, resulta útil cuando la expresión de filtro puede definir completamente los documentos de interés. Sin una cadena de consulta, no hay ningún análisis lingüístico ni léxico, ninguna puntuación y ninguna clasificación. Observe que la cadena de búsqueda es simplemente un asterisco, lo que significa "coincide con todos los documentos".

   ```
   search=*&$filter=(baseRate ge 60 and baseRate lt 300) and accommodation eq 'Hotel' and city eq 'Nogales'
   ```

+ Combinación de una cadena de consulta y **$filter**, donde el filtro crea el subconjunto y la cadena de consulta proporciona las entradas de términos para la búsqueda de texto completo en el subconjunto filtrado. Usar un filtro con una cadena de consulta es el patrón de uso más comunes.

   ```
   search=hotels ocean$filter=(baseRate ge 60 and baseRate lt 300) and city eq 'Los Angeles'
   ```

+ Consultas compuestas, separadas por "or", cada una con sus propios criterios de filtro (por ejemplo, 'beagles' en 'perro' o 'siamés' en 'gato'). Las expresiones se combinan con `or` se evalúan individualmente, con la unión de los documentos que coinciden con cada expresión enviado en la respuesta. Este patrón de uso se logra a través de la `search.ismatchscoring` función. También puede usar la versión sin puntuación, `search.ismatch`.

   ```
   # Match on hostels rated higher than 4 OR 5-star motels.
   $filter=search.ismatchscoring('hostel') and rating ge 4 or search.ismatchscoring('motel') and rating eq 5

   # Match on 'luxury' or 'high-end' in the description field OR on category exactly equal to 'Luxury'.
   $filter=search.ismatchscoring('luxury | high-end', 'description') or category eq 'Luxury'
   ```

  También es posible combinar la búsqueda de texto completo a través de `search.ismatchscoring` con filtros con `and` en lugar de `or`, pero esto es funcionalmente equivalente a usar el `search` y `$filter` parámetros en una solicitud de búsqueda. Por ejemplo, las dos consultas siguientes producen el mismo resultado:

  ```
  $filter=search.ismatchscoring('pool') and rating ge 4

  search=pool&$filter=rating ge 4
  ```

Consulte estos artículos para obtener una guía completa de casos de uso específicos:

+ [Filtros de faceta](search-filters-facets.md)
+ [Filtros de idioma](search-filters-language.md)
+ [Recorte de seguridad](search-security-trimming-for-azure-search.md) 

## <a name="field-requirements-for-filtering"></a>Requisitos de campo para filtrar

En la API de REST, filtrado está *en* campos simples de forma predeterminada. Los campos que se pueden filtrar aumentan el tamaño del índice. Asegúrese de definir `"filterable": false` para los campos que no tiene previsto usar en un filtro. Para obtener más información sobre la configuración de definiciones de campos, consulte [Create Index](https://docs.microsoft.com/rest/api/searchservice/create-index) (Creación de índices).

En el SDK de. NET, el filtrado está *desactivado* de forma predeterminada. Puede hacer que un campo filtrable estableciendo el [IsFilterable propiedad](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.field.isfilterable?view=azure-dotnet) de correspondiente [campo](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.field?view=azure-dotnet) objeto `true`. También puede hacerlo mediante declaración utilizando el [IsFilterable atributo](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.isfilterableattribute). En el ejemplo siguiente, el atributo se establece en el `BaseRate` propiedad de una clase de modelo que se asigna a la definición del índice.

```csharp
    [IsFilterable, IsSortable, IsFacetable]
    public double? BaseRate { get; set; }
```

### <a name="making-an-existing-field-filterable"></a>Hacer que un campo existente que se puede filtrar

No se puede modificar los campos existentes para que sean filtrables. En su lugar, deberá agregar un nuevo campo, o volver a generar el índice. Para obtener más información sobre cómo volver a generar un índice o volver a rellenar los campos, consulte [cómo volver a generar un índice de Azure Search](search-howto-reindex.md).

## <a name="text-filter-fundamentals"></a>Conceptos básicos de filtro de texto

Filtros de texto coincide con los campos de cadena en las cadenas literales que se proporcionan en el filtro. A diferencia de la búsqueda de texto completo, no hay ningún análisis léxico ni separación de palabras para los filtros de texto, por lo que las comparaciones son solo a coincidencias exactas. Por ejemplo, suponga un campo *f* contiene "día soleado", `$filter=f eq 'Sunny'` no coincide, pero `$filter=f eq 'sunny day'` le. 

Las cadenas de texto distinguen mayúsculas de minúsculas. No hay ningún inferior grafía de palabras en mayúsculas: `$filter=f eq 'Sunny day'` no encontrará "día soleado".

### <a name="approaches-for-filtering-on-text"></a>Enfoques para el filtrado en texto

| Enfoque | DESCRIPCIÓN | Cuándo se deben usar | 
|----------|-------------|-------------|
| [search.in](query-odata-filter-orderby-syntax.md) | Una función que coincide con un campo en una lista delimitada de cadenas. | Se recomienda para [filtros de seguridad](search-security-trimming-for-azure-search.md) y para los filtros donde se necesitan muchos valores de texto sin formato para que coincida con un campo de cadena. El **search.in** función está diseñado para acelerar el proceso y es mucho más rápido que compara explícitamente el campo en cada cadena con `eq` y `or`. | 
| [search.ismatch](query-odata-filter-orderby-syntax.md) | Una función que permite combinar las operaciones de búsqueda de texto completo con operaciones de filtro estrictamente booleano en la misma expresión de filtro. | Use **search.ismatch** (o su equivalente de puntuación, **search.ismatchscoring**) cuando desee varias combinaciones de filtro de búsqueda en una sola solicitud. También puede usarla para que un filtro *contains* filtre una cadena parcial en una cadena mayor. |
| [$filter=campo operador cadena](query-odata-filter-orderby-syntax.md) | Expresión definida por el usuario compuesta de campos, operadores y valores. | Utilícelo cuando desee buscar a coincidencias exactas entre un campo de cadena y un valor de cadena. |

## <a name="numeric-filter-fundamentals"></a>Conceptos básicos de filtro numérico

Los campos numéricos no se pueden definir como `searchable` en el contexto de búsqueda de texto completo. Solo las cadenas están sujetas a la búsqueda de texto completo. Por ejemplo, si escribe 99.99 como término de búsqueda, no obtendrá artículos con un precio de 99.99 USD. Lo que se devolverá son elementos con el número 99 en los campos de cadena del documento. Por lo tanto, si tiene datos numéricos, se asume que los va a usar para los filtros, como rangos, facetas, grupos, etc. 

Los documentos que contienen campos numéricos (precio, tamaño, SKU, id.) proporcionan esos valores en los resultados de la búsqueda si el campo está marcado como `retrievable`. En resumen, la búsqueda de texto completo no es aplicable a los tipos de campo numéricos.

## <a name="next-steps"></a>Pasos siguientes

En primer lugar, pruebe el **Explorador de búsqueda**  en el portal para enviar consultas con parámetros **$filter**. El [índice real-estate-sample](search-get-started-portal.md) proporciona resultados interesantes para las siguiente consultas filtradas al pegarlas en la barra de búsqueda:

```
# Geo-filter returning documents within 5 kilometers of Redmond, Washington state
# Use $count=true to get a number of hits returned by the query
# Use $select to trim results, showing values for named fields only
# Use search=* for an empty query string. The filter is the sole input

search=*&$count=true&$select=description,city,postCode&$filter=geo.distance(location,geography'POINT(-122.121513 47.673988)') le 5

# Numeric filters use comparison like greater than (gt), less than (lt), not equal (ne)
# Include "and" to filter on multiple fields (baths and bed)
# Full text search is on John Leclerc, matching on John or Leclerc

search=John Leclerc&$count=true&$select=source,city,postCode,baths,beds&$filter=baths gt 3 and beds gt 4

# Text filters can also use comparison operators
# Wrap text in single or double quotes and use the correct case
# Full text search is on John Leclerc, matching on John or Leclerc

search=John Leclerc&$count=true&$select=source,city,postCode,baths,beds&$filter=city gt 'Seattle'
```

Para trabajar con más ejemplos, consulte [OData Filter Expression Syntax > Examples](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search#filter-examples) (Sintaxis de expresión de filtro de OData > Ejemplos).

## <a name="see-also"></a>Vea también

+ [Cómo funciona la búsqueda de texto completo en Azure Search](search-lucene-query-architecture.md)
+ [API de REST de documentos de búsqueda](https://docs.microsoft.com/rest/api/searchservice/search-documents)
+ [Sintaxis de consulta simplificada](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)
+ [Sintaxis de consulta de Lucene](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)
+ [Supported data types](https://docs.microsoft.com/rest/api/searchservice/supported-data-types) (Tipos de datos admitidos)
