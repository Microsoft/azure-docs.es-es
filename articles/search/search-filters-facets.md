---
title: Filtros de faceta para la navegación de búsqueda en aplicaciones
titleSuffix: Azure Cognitive Search
description: Puede filtrar los criterios por identidad de seguridad del usuario, geolocalización o valores numéricos para reducir los resultados de búsqueda en las consultas en Azure Cognitive Search, un servicio de búsqueda en la nube hospedado de Microsoft Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 852d8f8f85536dc62dd792e5727dd7ec0571ba29
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084218"
---
# <a name="how-to-build-a-facet-filter-in-azure-cognitive-search"></a>Creación de un filtro de faceta en Azure Cognitive Search 

La navegación por facetas se usa para aplicar filtros autodirigidos en resultados de consulta en una aplicación de búsqueda, donde la aplicación ofrece controles de la interfaz de usuario para limitar la búsqueda a grupos de documentos (por ejemplo, categorías o marcas) y Azure Cognitive Search proporciona la estructura de datos necesaria para la experiencia. En este artículo, se revisan brevemente los pasos básicos para crear una estructura de navegación por facetas que respalde la experiencia de búsqueda que desea proporcionar. 

> [!div class="checklist"]
> * Elegir los campos para los filtros y las facetas
> * Definir atributos en el campo
> * Crear el índice y cargar los datos
> * Agregar filtros por faceta a una consulta
> * Administrar los resultados

Las facetas son dinámicas y se devuelven en una consulta. Las respuestas de búsqueda incluyen las categorías de faceta que se usan para navegar por los resultados. Si no está familiarizado con las facetas, el ejemplo siguiente es una ilustración de una estructura de navegación por facetas.

  ![Imagen que muestra un cuadro de diálogo de búsqueda con resultados de búsqueda filtrados agrupados por título de negocio. Una flecha indica que los resultados son facetas que se muestran en una estructura de navegación por facetas.](./media/search-filters-facets/facet-nav.png)

¿No conoce la navegación por facetas y quiere más detalles? Consulte [Procedimiento para implementar la navegación por facetas en Azure Cognitive Search](search-faceted-navigation.md).

## <a name="choose-fields"></a>Elegir los campos

Las facetas se pueden calcular tanto para campos de valor único como para colecciones. Los campos que funcionan mejor en la navegación por facetas tienen una cardinalidad baja: un pequeño número de valores distintos que se repiten a lo largo de los documentos del corpus de búsqueda (por ejemplo, una lista de colores, países o regiones, o nombres de marca). 

Las facetas se habilitan campo a campo al crear el índice mediante el establecimiento del atributo `facetable` en `true`. Por lo general, debe establecer también el atributo `filterable` en `true` para esos campos para que la aplicación de búsqueda pueda filtrar por esos campos en función de las facetas que seleccione el usuario final. 

Al crear un índice mediante la API de REST, cualquier [tipo de campo](https://docs.microsoft.com/rest/api/searchservice/supported-data-types) que pudiera usarse en la navegación por facetas se marca como `facetable` de forma predeterminada:

+ `Edm.String`
+ `Edm.DateTimeOffset`
+ `Edm.Boolean`
+ Tipos de campo numéricos: `Edm.Int32`, `Edm.Int64`, `Edm.Double`
+ Colecciones de los tipos anteriores (por ejemplo, `Collection(Edm.String)` o `Collection(Edm.Double)`)

No puede usar campos `Edm.GeographyPoint` ni `Collection(Edm.GeographyPoint)` en la navegación por facetas. Las facetas funcionan mejor en los campos con cardinalidad baja. Debido a la resolución de coordenadas geográficas, es poco frecuente que dos conjuntos de coordenadas sean iguales en un conjunto de datos determinado. Como tales, las facetas no se admiten para coordenadas geográficas. Para crear una faceta de ubicación, se necesita un campo de ciudad o región.

## <a name="set-attributes"></a>Definir atributos

Los atributos de índice que controlan cómo se usa un campo se agregan a las definiciones de campo individuales del índice. En el ejemplo siguiente, los campos con cardinalidad baja, útiles para la creación de facetas, son: `category` (hotel, motel, albergue), `tags` y `rating`. Estos campos tienen los atributos `filterable` y `facetable` establecidos de forma explícita en el ejemplo siguiente con fines ilustrativos. 

> [!Tip]
> Como procedimiento recomendado para optimizar el rendimiento y el almacenamiento, desactive las facetas para los campos que nunca se deban usar como facetas. En concreto, los campos de cadena de valores únicos, como un identificador o un nombre de producto, deben establecerse en `"facetable": false` para evitar su uso accidental (e ineficaz) en la navegación por facetas.


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
> Esta definición de índice se copia de [Creación de un índice de Azure Cognitive Search con la API REST](https://docs.microsoft.com/azure/search/search-create-index-rest-api). Es idéntico, salvo en diferencias superficiales en las definiciones de campo. Los atributos `filterable` y `facetable` se agregan explícitamente en los campos `category`, `tags`, `parkingIncluded`, `smokingAllowed` y `rating`. En la práctica, `filterable` y `facetable` se habilitarían en estos campos de forma predeterminada al usar la API de REST. Al usar el SDK de .NET, estos atributos deben habilitarse de forma explícita.

## <a name="build-and-load-an-index"></a>Creación y carga de un índice

Un paso intermedio (y, quizás, obvio) es que tiene que [crear y completar el índice](https://docs.microsoft.com/azure/search/search-get-started-dotnet#1---create-index) antes de formular una consulta. Hemos mencionado este paso aquí para mayor integridad. Una manera de determinar si el índice está disponible es comprobar la lista de índices en el [portal](https://portal.azure.com).

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

Cuando el usuario final hace clic en un valor de faceta, el controlador del evento de clic debe usar una expresión de filtro para materializar la intención del usuario. Dada una faceta `category`, un clic en la categoría "motel" se implementa con una expresión `$filter` que selecciona alojamientos de ese tipo. Cuando un usuario hace clic en "motel" para indicar que solo se deben mostrar los moteles, la consulta siguiente que la aplicación envía incluye `$filter=category eq 'motel'`.

El fragmento de código siguiente agrega la categoría al filtro si un usuario selecciona un valor de la faceta de categoría.

```csharp
if (!String.IsNullOrEmpty(categoryFacet))
    filter = $"category eq '{categoryFacet}'";
```

Si el usuario hace clic en un valor de faceta de un campo de la colección como `tags`, por ejemplo el valor "pool", la aplicación debe usar la sintaxis de filtro siguiente: `$filter=tags/any(t: t eq 'pool')`

## <a name="tips-and-workarounds"></a>Sugerencias y soluciones alternativas

### <a name="initialize-a-page-with-facets-in-place"></a>Inicializar una página con facetas definidas

Si quiere inicializar una página con facetas definidas, puede enviar una consulta como parte de la inicialización de la página para proveer a la página con una estructura de facetas inicial.

### <a name="preserve-a-facet-navigation-structure-asynchronously-of-filtered-results"></a>Conservar una estructura de navegación por facetas de forma asincrónica respecto a los resultados filtrados

Uno de los retos de la navegación por facetas de Azure Cognitive Search es que las facetas solo existen para los resultados actuales. En la práctica, se suele conservar un conjunto estático de facetas para que el usuario pueda navegar en orden inverso y reconstruir los pasos para explorar rutas alternativas a través del contenido de búsqueda. 

Aunque se trata de un caso de uso común, no es algo que la estructura de navegación por facetas proporcione de forma predeterminada actualmente. Para solucionar esta limitación, los desarrolladores que quieren facetas estáticas suelen emitir dos consultas filtradas: una centrada en los resultados y otra para crear una lista estática de facetas con fines de navegación.

## <a name="see-also"></a>Consulte también

+ [Filtros de Azure Cognitive Search](search-filters.md)
+ [Create Index REST API](https://docs.microsoft.com/rest/api/searchservice/create-index) (Creación de índices [API de REST])
+ [API de REST de documentos de búsqueda](https://docs.microsoft.com/rest/api/searchservice/search-documents)
