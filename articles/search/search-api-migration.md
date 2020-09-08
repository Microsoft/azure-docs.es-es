---
title: Actualización de las versiones de API de REST
titleSuffix: Azure Cognitive Search
description: Revise las diferencias en las versiones de API y conozca qué acciones son necesarias para migrar código existente a la versión más reciente de la API de REST del servicio Búsqueda cognitiva de Azure.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/26/2020
ms.openlocfilehash: 0f1050bf58e0cd8d9a601d60a4c5dc22a5420483
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88949038"
---
# <a name="upgrade-to-the-latest-rest-api-in-azure-cognitive-search"></a>Actualización a la API de REST más reciente de Azure Cognitive Search

Si usa una versión anterior de la [**API de REST de Search**](/rest/api/searchservice/), este artículo le ayuda a actualizar la aplicación a la versión más reciente de la API disponible con carácter general, **2020-06-30**.

La versión 2020-06-30 incluye una característica nueva importante ([almacén de conocimiento](knowledge-store-concept-intro.md)) y presenta varios cambios de comportamiento secundarios. Como tal, esta versión es, en general, compatible con versiones anteriores, por lo que los cambios de código deben ser mínimos si va a actualizar desde la versión anterior (2019-05-06).

> [!NOTE]
> Un servicio de búsqueda admite una serie de versiones de la API de REST, incluidas versiones anteriores. Puede seguir usando esas versiones de API, pero se recomienda migrar el código a la versión más reciente, para poder acceder a las funciones nuevas. Con el tiempo, las versiones más obsoletas de la API de REST van a quedar en desuso y [ya no se van a admitir](search-api-versions.md#unsupported-versions).

<a name="UpgradeSteps"></a>

## <a name="how-to-upgrade"></a>Procedimiento de actualización

Al actualizar a una versión nueva, es probable que no tenga que hacer ningún cambio en el código, excepto cambiar el número de versión. Las únicas situaciones en las que puede que tenga que modificar el código ocurren si:

* Se produce un error en el código cuando se devuelven propiedades no reconocidas en una respuesta de la API. De forma predeterminada, la aplicación debe omitir propiedades que no comprende.

* El código conserva las solicitudes de API e intenta volver a enviarlas a la nueva versión de API. Por ejemplo, esto podría suceder si la aplicación conserva tokens de continuación devueltos por la API de búsqueda (para más información, busque `@search.nextPageParameters` en la [referencia de la API de búsqueda](/rest/api/searchservice/Search-Documents)).

* El código hace referencia a una versión de API anterior a la versión 2019-05-06 y que está sujeta a uno o varios de los cambios importantes de esa versión. La sección [Actualización a 2019-05-06](#upgrade-to-2019-05-06) proporciona más detalles. 

Si alguna de estas situaciones se le presenta, debe cambiar el código en consecuencia. Si no, no debería ser necesario ningún cambio, aunque puede que quiera empezar a usar características incorporadas en la versión nueva.

## <a name="upgrade-to-2020-06-30"></a>Actualización a 2020-06-30

La versión 2020-06-30 es la nueva versión disponible con carácter general de la API REST. No hay cambios importantes, pero sí algunas diferencias de comportamiento. 

Entre las características que ahora están disponibles con carácter general en esta versión de la API se incluyen:

* [Almacén de conocimiento](knowledge-store-concept-intro.md), almacenamiento persistente de contenido enriquecido creado a través de conjuntos de aptitudes, creado para el análisis y el procesamiento de nivel inferior a través de otras aplicaciones. Con esta capacidad, una canalización de enriquecimiento con IA basada controlada por indexadores puede rellenar un almacén de información además de un índice de búsqueda. Si usó la versión preliminar de esta característica, es equivalente a la versión disponible con carácter general. El único cambio de código necesario es modificar la versión de la API.

Estos cambios de cambio de comportamiento incluyen los siguientes:

* El [algoritmo de clasificación BM25](index-ranking-similarity.md) reemplaza el algoritmo de clasificación anterior por una tecnología más reciente. Los servicios nuevos usarán este algoritmo de manera automática. En el caso de los servicios existentes, debe establecer los parámetros para usar el algoritmo nuevo.

* Los resultados ordenados de los valores NULL han cambiado en esta versión y los valores NULL aparecen en primer lugar si la ordenación es `asc` y al último si la ordenación es `desc`. Si escribió código para controlar cómo se ordenan los valores NULL, tenga en cuenta este cambio.

## <a name="upgrade-to-2019-05-06"></a>Actualización a 2019-05-06

La versión 2019-05-06 es la versión disponible con carácter general anterior de la API REST. Entre las características que estuvieron disponibles con carácter general en esta versión de la API se incluyen:

* [Autocompletar](index-add-suggesters.md) es una característica de escritura anticipada que completa una entrada de término especificada de forma parcial.
* Los [tipos complejos](search-howto-complex-data-types.md) proporcionan compatibilidad nativa para los datos de objeto estructurado en un índice de búsqueda.
* Los [modos de análisis de JsonLines](search-howto-index-json-blobs.md), parte de la indexación de Azure Blob, crean un documento de búsqueda por entidad JSON que está separado por una línea nueva.
* El [enriquecimiento con inteligencia artificial](cognitive-search-concept-intro.md) ofrece indexación que aprovecha los motores de enriquecimiento de inteligencia artificial de Cognitive Services.

### <a name="breaking-changes"></a>Últimos cambios

El código existente escrito en versiones anteriores de la API se interrumpirá en api-version=2019-05-06 y posterior si contiene la siguiente funcionalidad:

#### <a name="indexer-for-azure-cosmos-db---datasource-is-now-type-cosmosdb"></a>Ahora el indizador del origen de datos de Azure Cosmos DB es "type": "cosmosdb"

Si usa un [indizador de Cosmos DB](search-howto-index-cosmosdb.md ), tendrá que cambiar `"type": "documentdb"` a `"type": "cosmosdb"`.

#### <a name="indexer-execution-result-errors-no-longer-have-status"></a>Los errores de resultados de ejecución del indizador ya no tienen estado

La estructura de error para la ejecución del indizador anteriormente tenía un elemento `status`. Este elemento se ha quitado porque no proporcionaba información útil.

#### <a name="indexer-data-source-api-no-longer-returns-connection-strings"></a>La API del origen de datos de indizador ya no devuelve cadenas de conexión

A partir de las versiones 2019-05-06 y 2019-05-06-Preview de la API, la API del origen de datos ya no devuelve cadenas de conexión en la respuesta de ninguna operación de REST. En versiones anteriores de la API, para los orígenes de datos creados mediante POST, Búsqueda cognitiva de Azure devolvía **201** seguido de la respuesta de OData, que contenía la cadena de conexión en texto sin formato.

#### <a name="named-entity-recognition-cognitive-skill-is-now-discontinued"></a>La aptitud cognitiva de reconocimiento de entidades se ha interrumpido

Si llamó a la aptitud [Reconocimiento de entidades de nombre](cognitive-search-skill-named-entity-recognition.md) en el código, se producirá un error en la llamada. La funcionalidad de reemplazo es [Reconocimiento de entidades](cognitive-search-skill-entity-recognition.md). Podrá reemplazar la referencia de la aptitud sin cambios adicionales. La firma de la API es la misma para las dos versiones. 

### <a name="upgrading-complex-types"></a>Actualización de tipos complejos

La versión de la API 2019-05-06 agrega compatibilidad formal con tipos complejos. Si el código implementó las recomendaciones anteriores para la equivalencia de tipo complejo en 2017-11-11-Preview o 2016-09-01-Preview, hay algunos límites nuevos y modificados a partir de la versión 2019-05-06 que tiene que tener en cuenta:

+ Se han reducido los límites de profundidad de campos secundarios y el número de colecciones complejas por índice. Si ha creado índices que superan estos límites mediante las versiones de la API en versión preliminar, cualquier intento de actualizarlos o volver a crearlos con la versión 2019-05-06 de la API producirá un error. Si esto se aplica a su caso, tendrá que volver a diseñar el esquema para ajustarlos a los nuevos límites y, después, volver a generar el índice.

+ Hay un límite nuevo a partir de la versión de la API 2019-05-06 para el número de elementos de colecciones complejas por documento. Si ha creado índices con documentos que superaban estos límites mediante las versiones de la API en versión preliminar, cualquier intento de indexar de nuevo esos datos con la versión 2019-05-06 de la API producirá un error. Si esto se aplica a su caso, tendrá que reducir el número de elementos de colección complejos por documento antes de volver a indexar los datos.

Para más información, vea [Límites de servicio en Búsqueda cognitiva de Azure](search-limits-quotas-capacity.md).

#### <a name="how-to-upgrade-an-old-complex-type-structure"></a>Procedimientos para actualizar una estructura de tipo complejo anterior

Si en el código se usan tipos complejos con una de las versiones anteriores de la API en versión preliminar, es posible que use un formato de definición de índice con este aspecto:

```json
{
  "name": "hotels",  
  "fields": [
    { "name": "HotelId", "type": "Edm.String", "key": true, "filterable": true },
    { "name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": true, "facetable": false },
    { "name": "Description", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "en.microsoft" },
    { "name": "Description_fr", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.microsoft" },
    { "name": "Category", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "sortable": false, "facetable": true, "analyzer": "tagsAnalyzer" },
    { "name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": true, "sortable": true, "facetable": true },
    { "name": "LastRenovationDate", "type": "Edm.DateTimeOffset", "filterable": true, "sortable": true, "facetable": true },
    { "name": "Rating", "type": "Edm.Double", "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address", "type": "Edm.ComplexType" },
    { "name": "Address/StreetAddress", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "searchable": true },
    { "name": "Address/City", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address/StateProvince", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address/PostalCode", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address/Country", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Location", "type": "Edm.GeographyPoint", "filterable": true, "sortable": true },
    { "name": "Rooms", "type": "Collection(Edm.ComplexType)" }, 
    { "name": "Rooms/Description", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "en.lucene" },
    { "name": "Rooms/Description_fr", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene" },
    { "name": "Rooms/Type", "type": "Edm.String", "searchable": true },
    { "name": "Rooms/BaseRate", "type": "Edm.Double", "filterable": true, "facetable": true },
    { "name": "Rooms/BedOptions", "type": "Edm.String", "searchable": true },
    { "name": "Rooms/SleepsCount", "type": "Edm.Int32", "filterable": true, "facetable": true },
    { "name": "Rooms/SmokingAllowed", "type": "Edm.Boolean", "filterable": true, "facetable": true },
    { "name": "Rooms/Tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "facetable": true, "analyzer": "tagsAnalyzer" }
  ]
}  
```

En la versión 2017-11-11-Preview de la API se ha introducido un nuevo formato de tipo árbol para definir campos de índice. En el nuevo formato, cada campo complejo tiene una colección de campos en la que se definen sus campos secundarios. En la versión 2019-05-06 de la API, este nuevo formato se usa de forma exclusiva y los intentos de crear o actualizar un índice con el formato antiguo producirán un error. Si tiene índices creados con el formato antiguo, tendrá que usar la versión 2017-11-11-Preview de la API para actualizarlos al formato nuevo antes de poder administrarlos con la versión 2019-05-06 de la API.

Puede actualizar índices "planos" al formato nuevo mediante los pasos siguientes con la versión 2017-11-11-Preview de la API:

1. Realice una solicitud GET para recuperar el índice. Si ya está en el formato nuevo, ya ha terminado.

2. Traduzca el índice del formato "plano" al formato nuevo. Tendrá que escribir código para esto, ya que en el momento de redactar este artículo no había ningún código de ejemplo.

3. Realice una solicitud PUT para actualizar el índice al formato nuevo. Asegúrese de que no se cambie ningún otro detalle del índice, como la capacidad de búsqueda y filtrado de los campos, ya que esto no está permitido por la API Update Index.

> [!NOTE]
> No se pueden administrar los índices creados con el formato "plano" antiguo desde Azure Portal. Actualice los índices desde la representación "plana" a la representación de "árbol" lo antes posible.

## <a name="next-steps"></a>Pasos siguientes

Revise la documentación de referencia de la API de REST del servicio Search. Si tiene algún problema, puede solicitar ayuda en [Stack Overflow](https://stackoverflow.com/) o [ponerse en contacto con el servicio de soporte técnico](https://azure.microsoft.com/support/community/?product=search).

> [!div class="nextstepaction"]
> [Referencia de la API de REST del servicio Search](/rest/api/searchservice/)