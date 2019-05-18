---
title: 'Actualización a la versión más reciente de la API REST de Azure Search Service: Azure Search'
description: Revise las diferencias en las versiones de API y conozca qué acciones son necesarias para migrar código existente a la versión más reciente de la API REST de Azure Search Service.
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: brjohnst
ms.openlocfilehash: 85a8ddf4ce87d7ac8ce460c0aff56311a2ea4578
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540693"
---
# <a name="upgrade-to-the-latest-azure-search-service-rest-api-version"></a>Actualización a la versión más reciente de la API REST de Azure Search Service
Si usa una versión anterior de la [API de REST del servicio Azure Search](https://docs.microsoft.com/rest/api/searchservice/), este artículo le ayudará a actualizar la aplicación para usar la versión más reciente de la API disponible con carácter general, 2019-05-06.

Versión de 2019-05-06 de la API de REST contiene algunos cambios de versiones anteriores. Se trata principalmente de cambios compatibles con versiones anteriores, por lo que cambiar el código apenas debe exigir esfuerzo, según la versión que utilizaba antes. [Pasos para actualizar](#UpgradeSteps) se describen los cambios de código necesarios para usar nuevas características.

> [!NOTE]
> Una instancia del servicio Azure Search admite un versiones de API de REST de intervalo, incluidas las versiones anteriores. Aún puede usar esas versiones de API, pero se recomienda migrar el código a la versión más reciente, por lo que puede tener acceso a nuevas capacidades.

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-2019-05-06"></a>Novedades de la versión de 2019-05-06
Versión de 2019-05-06 es la versión más reciente disponible con carácter general de la API de REST del servicio Azure Search. Las características que se han pasado a estado disponible con carácter general en esta versión de API se incluyen:

* [Autocompletar](index-add-suggesters.md) es una característica de escritura anticipada que se completa una entrada de término parcial especificado.

* [Tipos complejos](search-howto-complex-data-types.md) proporciona compatibilidad nativa para los datos de objeto estructurado en un índice de Azure Search.

* [Modos de análisis de JsonLines](search-howto-index-json-blobs.md), parte de Azure Blob de indexación, crea un documento de búsqueda por entidad JSON que está separado por una línea nueva.

* [Búsqueda cognitiva](cognitive-search-concept-intro.md) ofrece indexación que aprovecha los motores de enriquecimiento de inteligencia artificial de Cognitive Services.

Varias versiones de la característica de vista previa coinciden con esta actualización disponible con carácter general. Para revisar la lista de nuevas características de versión preliminar, vea [versión de api de REST de búsqueda de 2019-05-06-Preview](search-api-preview.md).

## <a name="breaking-changes"></a>Cambios importantes

El código existente que contiene la siguiente funcionalidad se interrumpirá en api-version = 2019-05-06.

### <a name="indexer-for-azure-cosmos-db---datasource-is-now-type-cosmosdb"></a>Indizador de Azure Cosmos DB - datasource es ahora "de tipo": "cosmosdb"

Si usas un [indexador de Cosmos DB](search-howto-index-cosmosdb.md ), debe cambiar `"type": "documentdb"` a `"type": "cosmosdb"`.

### <a name="indexer-execution-result-errors-no-longer-have-status"></a>Errores de resultados de ejecución de indizador ya no tienen estado

La estructura de error de ejecución del indizador anteriormente tenía un `status` elemento. Este elemento se quitó porque no proporcionaba información útil.

### <a name="indexer-data-source-api-no-longer-returns-connection-strings"></a>Origen de datos API de indizador ya no devuelve las cadenas de conexión

Desde la API de versiones 2019-05-06 y 2019-05-06-Preview y versiones posteriores, el origen de datos API ya no devuelve las cadenas de conexión en la respuesta de cualquier operación de REST. En versiones anteriores de API, para orígenes de datos creados mediante POST, Azure Search devuelve **201** seguido de la respuesta de OData, que contiene la cadena de conexión en texto sin formato.

### <a name="named-entity-recognition-cognitive-skill-is-now-discontinued"></a>Denominado habilidades cognitivas ya no está disponible el reconocimiento de entidades

Si se llama a [reconocimiento de entidades de nombre](cognitive-search-skill-named-entity-recognition.md) aptitud en el código, la llamada se producirá un error. La funcionalidad de reemplazo es [reconocimiento de entidades](cognitive-search-skill-entity-recognition.md). Puede reemplazar la referencia de la aptitud con ningún otro cambio. La firma de la API es el mismo para ambas versiones. 

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Pasos para actualizar
Si va a actualizar desde una versión anterior de GA, 11-11-2017 o 2016-09-01, probablemente no tendrá que realizar los cambios en el código, distinto para cambiar el número de versión. Las únicas situaciones en las que puede que tenga que modificar el código ocurren si:

* Se produce un error en el código cuando se devuelven propiedades no reconocidas en una respuesta de la API. De forma predeterminada, la aplicación debe omitir propiedades que no comprende.

* El código conserva las solicitudes de API e intenta volver a enviarlas a la nueva versión de API. Por ejemplo, esto podría suceder si la aplicación conserva tokens de continuación devueltos por la API de búsqueda (para más información, busque `@search.nextPageParameters` en la [referencia de la API de búsqueda](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)).

Si alguna de estas situaciones se le presenta, debe cambiar el código en consecuencia. De lo contrario, ningún cambio debería ser necesario a menos que desee empezar a usar el [nuevas características](#WhatsNew) de versión de 2019-05-06.

Si va a actualizar desde una versión anterior de API, lo anterior también se aplica, pero también debe tener en cuenta que algunas características de vista previa no están disponibles en la versión de 2019-05-06:

* [Consultas "Más similar al siguiente"](search-more-like-this.md)
* [Indización de blobs CSV](search-howto-index-csv-blobs.md)
* [Compatibilidad con la API de MongoDB para los indizadores de Cosmos DB](search-howto-index-cosmosdb.md)

Si el código usa estas características, no podrá actualizar a la API versión 2019-05-06 sin quitar el uso de ellos.

> [!IMPORTANT]
> Las versiones preliminares de las API están pensadas para realizar pruebas y evaluaciones, y no deben usarse en entornos de producción.
> 

### <a name="upgrading-complex-types"></a>Actualizar tipos complejos

Si el código usa tipos complejos con las versiones anteriores de API de vista previa de 2017-11-11-Preview o 2016-09-01-Preview, hay algunos límites nuevas y modificadas en la versión de 2019-05-06, que debe tener en cuenta:

+ Se ha reducido los límites de la profundidad de campos secundarios y el número de colecciones complejas por índice. Si ha creado los índices que superar estos límites con las versiones de api de vista previa, cualquier intento de actualizar o volver a crearlos con la API versión 2019-05-06 se producirá un error. Si esto se aplica a usted, deberá volver a diseñar su esquema para ajustarse a los nuevos límites y, a continuación, volver a generar el índice.

+ Hay un límite nuevo en la versión de api de 2019-05-06, en el número de elementos de colecciones complejas por documento. Si ha creado los índices con documentos que superar estos límites con las versiones de api de vista previa, se producirá un error en cualquier intento de volver a indizar los datos mediante la api-version 2019-05-06. Si esto se aplica a usted, deberá reducir el número de elementos de colección compleja por documento antes de volver a indexar los datos.

Para obtener más información, consulte [límites de servicio de Azure Search](search-limits-quotas-capacity.md).

### <a name="how-to-upgrade-an-old-complex-type-structure"></a>Cómo actualizar una estructura de tipo complejo anterior

Si el código usa tipos complejos con una de las versiones anteriores de API de vista previa, puede que esté usando un formato de definición de índice que tiene este aspecto:

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

Se introdujo un nuevo formato de árbol para definir campos de índice en API versión 2017-11-11-Preview. En el nuevo formato, cada campo complejo tiene una colección de campos que se definen sus subcampos. En la API versión 2019-05-06, este nuevo formato se usa exclusivamente y se producirá un error al intentar crear o actualizar un índice con el formato antiguo. Si dispone de los índices creados mediante el formato antiguo, deberá usar la API versión 2017-11-11-Preview se actualizan al nuevo formato antes de que pueden administrarse mediante la API versión 2019-05-06.

Puede actualizar índices "planos" al nuevo formato con los pasos siguientes con la API versión 2017-11-11-Preview:

1. Realizar una solicitud GET para recuperar el índice. Si ya está en el nuevo formato, ya ha terminado.

2. Traducir el índice del formato de "plano" al nuevo formato. Tendrá que escribir código para esto, ya que no hay ningún código de ejemplo en el momento de redactar este artículo.

3. Realizar una solicitud PUT para actualizar el índice al nuevo formato. Asegúrese de que no se cambie cualquier otro detalle del índice, como la capacidad de búsqueda/filterability de campos, ya que esto no está permitido por la API de índice de actualización.

> [!NOTE]
> No es posible administrar los índices creados con el formato antiguo de "plano" desde el portal de Azure. Actualice los índices de la representación de "plana" a la representación de "árbol" lo antes posible.

## <a name="next-steps"></a>Pasos siguientes

Revise la documentación de referencia de API de REST del servicio Azure Search. Si tiene problemas, pedirnos ayuda en [StackOverflow](https://stackoverflow.com/) o [póngase en contacto con soporte técnico](https://azure.microsoft.com/support/community/?product=search).

> [!div class="nextstepaction"]
> [Referencia de API de REST del servicio de búsqueda](https://docs.microsoft.com/rest/api/searchservice/)

