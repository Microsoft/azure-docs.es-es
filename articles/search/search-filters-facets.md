---
title: 'Filtros de faceta para la navegación de búsqueda en aplicaciones: Azure Search'
description: Puede filtrar los criterios por identidad de seguridad del usuario, geolocalización o valores numéricos para reducir los resultados de búsqueda en las consultas en Azure Search, un servicio de búsqueda en la nube hospedado de Microsoft Azure.
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 5/13/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 8dffc5b87aefe23953d3a74f1d96b5ee03e0315d
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65597392"
---
# <a name="how-to-build-a-facet-filter-in-azure-search"></a>Cómo crear un filtro de faceta en Azure Search 

La navegación por facetas se usa para aplicar filtros autodirigidos en resultados de consulta en una aplicación de búsqueda, donde la aplicación ofrece controles de UI para limitar la búsqueda a grupos de documentos (por ejemplo, categorías o marcas) y Azure Search proporciona la estructura de datos necesaria para la experiencia. En este artículo, se revisan brevemente los pasos básicos para crear una estructura de navegación por facetas que respalde la experiencia de búsqueda que desea proporcionar. 

> [!div class="checklist"]
> * Elegir los campos para los filtros y las facetas
> * Definir atributos en el campo
> * Crear el índice y cargar los datos
> * Agregar filtros por faceta a una consulta
> * Administrar los resultados

Las facetas son dinámicas y se devuelven en una consulta. Las respuestas de búsqueda incluyen las categorías de faceta que se usan para navegar por los resultados. Si no está familiarizado con las facetas, el ejemplo siguiente es una ilustración de una estructura de navegación por facetas.

  ![](./media/search-filters-facets/facet-nav.png)

¿No conoce la navegación por facetas y quiere más detalles? Consulte [Procedimiento para implementar la navegación por facetas en Azure Search](search-faceted-navigation.md).

## <a name="choose-fields"></a>Elegir los campos

Las facetas se pueden calcular tanto para campos de valor único como para colecciones. Los campos que funcionan mejor en la navegación por facetas tienen una cardinalidad baja: un número pequeño de valores distintos que se repiten a lo largo de los documentos en el corpus de búsqueda (por ejemplo, una lista de colores, países o regiones o nombres de marca). 

Las facetas se habilitan en el campo por campo al crear el índice estableciendo la `facetable` atributo `true`. Por lo general debe establecer también la `filterable` atributo `true` para tales campos para que la aplicación de búsqueda puede filtrar por esos campos en función de las facetas que selecciona el usuario final. 

Al crear un índice mediante la API de REST, cualquier [tipo de campo](https://docs.microsoft.com/rest/api/searchservice/supported-data-types) que podrían usarse en la navegación por facetas se marca como `facetable` de forma predeterminada:

+ `Edm.String`
+ `Edm.DateTimeOffset`
+ `Edm.Boolean`
+ Tipos de campo numéricos: `Edm.Int32`, `Edm.Int64`, `Edm.Double`
+ Las colecciones de los tipos anteriores (por ejemplo, `Collection(Edm.String)` o `Collection(Edm.Double)`)

No puede usar `Edm.GeographyPoint` o `Collection(Edm.GeographyPoint)` campos en la navegación por facetas. Las facetas funcionan mejor en los campos con cardinalidad baja. Debido a la resolución de coordenadas geográficas, es poco frecuente que los dos conjuntos de coordenadas será iguales en un conjunto de datos determinado. Como tales, las facetas no se admiten para coordenadas geográficas. Para crear una faceta de ubicación, se necesita un campo de ciudad o región.

## <a name="set-attributes"></a>Definir atributos

Los atributos de índice que controlan cómo se usa un campo se agregan a las definiciones de campo individuales del índice. En el ejemplo siguiente, los campos con cardinalidad baja, útiles para el uso de facetas, constan de: `category` (hotel, motel, albergue), `tags`, y `rating`. Estos campos tienen el `filterable` y `facetable` atributos establecer explícitamente en el ejemplo siguiente con fines ilustrativos. 

> [!Tip]
> Como procedimiento recomendado para optimizar el rendimiento y el almacenamiento, desactive las facetas para los campos que nunca se deban usar como facetas. En concreto, los campos de cadena para valores únicos, como un identificador o nombre del producto, se deben establecer en `"facetable": false` para impedir su uso accidental (e ineficaz) en la navegación por facetas.


```json
{
  "name": "hotels",  
  "fields": [
    { "name": "hotelId", "type": "Edm.String", "key": true, "searchable": false, "sortable": false, "facetable": false },
    { "name": "baseRate", "type": "Edm.Double" },
    { "name": "description", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false },
    { "name": "description_fr", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene" },
    { "name": "hotelName", "type": "Edm.String", "facetable": false },
    { "name": "category", "type": "Edm.String", "filterable": true, "facetable": true },
    { "name": "tags", "type": "Collection(Edm.String)", "filterable": true, "facetable": true },
    { "name": "parkingIncluded", "type": "Edm.Boolean",  "filterable": true, "facetable": true, "sortable": false },
    { "name": "smokingAllowed", "type": "Edm.Boolean", "filterable": true, "facetable": true, "sortable": false },
    { "name": "lastRenovationDate", "type": "Edm.DateTimeOffset" },
    { "name": "rating", "type": "Edm.Int32", "filterable": true, "facetable": true },
    { "name": "location", "type": "Edm.GeographyPoint" }
  ]
}
```

> [!Note]
> Esta definición de índice se copia de [Creación de un índice de Azure Search con la API de REST](https://docs.microsoft.com/azure/search/search-create-index-rest-api). Es idéntico, salvo en diferencias superficiales en las definiciones de campo. El `filterable` y `facetable` atributos se agregan explícitamente `category`, `tags`, `parkingIncluded`, `smokingAllowed`, y `rating` campos. En la práctica, `filterable` y `facetable` se habilitaría en estos campos de forma predeterminada cuando se usa la API de REST. Al usar el SDK. NET, estos atributos deben habilitarse explícitamente.

## <a name="build-and-load-an-index"></a>Creación y carga de un índice

Un paso intermedio (y, quizás, obvio) es que tiene que [crear y completar el índice](https://docs.microsoft.com/azure/search/search-create-index-dotnet#3---construct-index) antes de formular una consulta. Hemos mencionado este paso aquí para mayor integridad. Una manera de determinar si el índice está disponible es comprobar la lista de índices en el [portal](https://portal.azure.com).

## <a name="add-facet-filters-to-a-query"></a>Agregar filtros por faceta a una consulta

En el código de la aplicación, cree una consulta que especifique todas las partes de una consulta válida, incluidas las expresiones de búsqueda, las facetas, los filtros o los perfiles de puntuación: todo lo que se usa para formular una solicitud. En el ejemplo siguiente, se crea una solicitud que crea navegación por facetas a partir del tipo de alojamiento, clasificación y otros servicios.

```csharp
var sp = new SearchParameters()
{
    ...
    // Add facets
    Facets = new[] { "category", "rating", "parkingIncluded", "smokingAllowed" }.ToList()
};
```

### <a name="return-filtered-results-on-click-events"></a>Devolver resultados filtrados en eventos de clic

Cuando el usuario final hace clic en un valor de faceta, el controlador para el evento click debe usar una expresión de filtro para que tenga en cuenta la intención del usuario. Dado un `category` faceta, al hacer clic en la categoría "motel" se implementa con un `$filter` expresión que selecciona alojamientos de ese tipo. Cuando un usuario hace clic en "motel" para indicar que se deben mostrar sólo los moteles, la consulta siguiente, la aplicación envía incluye `$filter=category eq 'motel'`.

El fragmento de código siguiente agrega la categoría al filtro si un usuario selecciona un valor de la faceta de categoría.

```csharp
if (!String.IsNullOrEmpty(categoryFacet))
    filter = $"category eq '{categoryFacet}'";
```

Si el usuario hace clic en un valor de faceta para un campo de la colección como `tags`, por ejemplo el "grupo" de valor, la aplicación debe usar la sintaxis de filtro siguientes: `$filter=tags/any(t: t eq 'pool')`

## <a name="tips-and-workarounds"></a>Sugerencias y soluciones alternativas

### <a name="initialize-a-page-with-facets-in-place"></a>Inicializar una página con facetas definidas

Si quiere inicializar una página con facetas definidas, puede enviar una consulta como parte de la inicialización de la página para proveer a la página con una estructura de facetas inicial.

### <a name="preserve-a-facet-navigation-structure-asynchronously-of-filtered-results"></a>Conservar una estructura de navegación por facetas de forma asincrónica respecto a los resultados filtrados

Uno de los retos de la navegación por facetas de Azure Search es que las facetas solo existen para los resultados actuales. En la práctica, se suele conservar un conjunto estático de facetas para que el usuario pueda navegar en orden inverso y reconstruir los pasos para explorar rutas alternativas a través del contenido de búsqueda. 

Aunque se trata de un caso de uso común, no es algo que la estructura de navegación por facetas proporcione de forma predeterminada actualmente. Para solucionar esta limitación, los desarrolladores que quieren facetas estáticas suelen emitir dos consultas filtradas: una centrada en los resultados y otra para crear una lista estática de facetas con fines de navegación.

## <a name="see-also"></a>Vea también

+ [Filtros de Azure Search](search-filters.md)
+ [Create Index REST API](https://docs.microsoft.com/rest/api/searchservice/create-index) (Creación de índices [API de REST])
+ [API de REST de documentos de búsqueda](https://docs.microsoft.com/rest/api/searchservice/search-documents)
