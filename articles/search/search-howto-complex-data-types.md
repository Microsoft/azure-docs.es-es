---
title: 'Modelado de tipos de datos complejos: Azure Search'
description: Estructuras de datos jerárquicas o anidadas se pueden modelar en un índice de Azure Search mediante ComplexType y colecciones de tipos de datos.
author: brjohnstmsft
manager: jlembicz
ms.author: brjohnst
tags: complex data types; compound data types; aggregate data types
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/13/2019
ms.custom: seodec2018
ms.openlocfilehash: 10400b0342fbe8667b22fea82c6446713d019e0d
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65597337"
---
# <a name="how-to-model-complex-data-types-in-azure-search"></a>Modelado de tipos de datos complejos en Azure Search

Conjuntos de datos externos usados para rellenar un índice de Azure Search a veces incluyen subestructuras jerárquicas o anidadas. Algunos ejemplos podrían incluir varias ubicaciones y números de teléfono para un solo cliente, varios colores y tamaños de un único SKU, varios autores de un único libro y así sucesivamente. En términos de modelado, es posible que vea estas estructuras que se conoce como *tipos de datos complejos*, *compuesta de los tipos de datos*, *tipos de datos compuestos*, o *agregado tipos de datos*. En la terminología de Azure Search, un tipo complejo es un campo que contiene a elementos secundarios (campos secundarios) que a su vez puede ser simple o complejo. Esto es similar a un tipo de datos estructurados en un lenguaje de programación. Pueden ser complejos campos campos únicos, que representan un único objeto en el documento, o bien una colección, que representa una matriz de objetos

Azure Search admite de forma nativa las colecciones y tipos complejos. Juntos, estos tipos le permiten modelar casi cualquier estructura anidada JSON en un índice de Azure Search. En versiones anteriores de las API de búsqueda de Azure, solo con formato no se podrían importar conjuntos de filas. En la versión más reciente, el índice puede ahora correspondan más fielmente con datos de origen. En otras palabras, si los datos de origen tienen tipos complejos, el índice puede tener tipos complejos también.

Para empezar, se recomienda la [conjunto de datos de hoteles](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/README.md), que puede cargar en el **importar datos** asistente en el portal de Azure. El asistente detecta los tipos complejos en el origen y sugiere un esquema de índice basado en las estructuras detectadas.

> [!Note]
> Compatibilidad con tipos complejos está disponible con carácter general en `api-version=2019-05-06`. 
>
> Si se basa en la solución de búsqueda alternativas anteriores de conjuntos de datos sin formato en una colección, debe cambiar el índice para incluir tipos complejos como compatible con la versión más reciente de la API. Para obtener más información acerca de cómo actualizar las versiones de API, consulte [actualizar a la versión más reciente de la API de REST](search-api-migration.md) o [actualizar a la versión más reciente del SDK de .NET](search-dotnet-sdk-migration-version-9.md).

## <a name="example-of-a-complex-structure"></a>Ejemplo de una estructura compleja

El siguiente documento JSON se compone de campos simples y complejos. Campos complejos, tales como `Address` y `Rooms`, tiene campos secundarios. `Address` tiene un único conjunto de valores para esos campos secundarios, puesto que es un único objeto en el documento. En cambio, `Rooms` tiene varios conjuntos de valores de sus subcampos, uno para cada objeto en la colección.

```json
{
  "HotelId": "1",
  "HotelName": "Secret Point Motel",
  "Description": "Ideally located on the main commercial artery of the city in the heart of New York.",
  "Address": {
    "StreetAddress": "677 5th Ave",
    "City": "New York",
    "StateProvince": "NY"
  },
  "Rooms": [
    {
      "Description": "Budget Room, 1 Queen Bed (Cityside)",
      "Type": "Budget Room",
      "BaseRate": 96.99,
    },
    {
      "Description": "Deluxe Room, 2 Double Beds (City View)",
      "Type": "Deluxe Room",
      "BaseRate": 150.99,
    },
  ]
}
```

## <a name="creating-complex-fields"></a>Creación de campos complejos

Igual que con cualquier definición de índice, puede usar el portal, [API de REST](https://docs.microsoft.com/rest/api/searchservice/create-index), o [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.index?view=azure-dotnet) para crear un esquema que incluye los tipos complejos. 

El ejemplo siguiente muestra un esquema de índice JSON con campos simples, colecciones y tipos complejos. Tenga en cuenta que dentro de un tipo complejo, cada campo secundario tiene un tipo y puede tener atributos, lo hacen los campos solo como nivel superior. El esquema se corresponde con los datos de ejemplo anteriores. `Address` es un campo complejo que no es una colección (un hotel tiene una dirección). `Rooms` es un campo de colección compleja (un hotel tiene algunas salas).

<!---
For indexes used in a [push-model data import](search-what-is-data-import.md) strategy, where you are pushing a JSON data set to an Azure Search index, you can only have the basic syntax shown here: single complex types like `Address`, or a `Collection(Edm.ComplexType)` like `Rooms`. You cannot have complex types nested inside other complex types in an index used for push-model data ingestion.

Indexers are a different story. When defining an indexer, in particular one used to build a knowledge store, your index can have nested complex types. An indexer is able to hold a chain of complex data structures in-memory, and when it includes a skillset, it can support highly complex data forms. For more information and an example, see [How to get started with knowledge store](knowledge-store-howto.md).
-->

```json
{
  "name": "hotels",
  "fields": [
    { "name": "HotelId", "type": "Edm.String", "key": true, "filterable": true  },
    { "name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false },
    { "name": "Description", "type": "Edm.String", "searchable": true, "analyzer": "en.lucene" },
    { "name": "Address", "type": "Edm.ComplexType",
      "fields": [
        { "name": "StreetAddress", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "searchable": true },
        { "name": "City", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
        { "name": "StateProvince", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true }
      ]
    },
    { "name": "Rooms", "type": "Collection(Edm.ComplexType)",
      "fields": [
        { "name": "Description", "type": "Edm.String", "searchable": true, "analyzer": "en.lucene" },
        { "name": "Type", "type": "Edm.String", "searchable": true },
        { "name": "BaseRate", "type": "Edm.Double", "filterable": true, "facetable": true }
      ]
    }
  ]
}
```

## <a name="updating-complex-fields"></a>Actualizar campos complejos

Todos los [volver a indexar reglas](search-howto-reindex.md) que se aplican a campos general todavía se aplican a campos complejos. Redefinición de algunas de las principales reglas aquí, agregar un campo no requiere una regeneración de índice, pero hacer la mayoría de las modificaciones.

### <a name="structural-updates-to-the-definition"></a>Actualizaciones estructurales de la definición

Puede agregar nuevos campos secundarios a un campo complejo en cualquier momento sin necesidad de una regeneración de índice. Por ejemplo, agregando "ZipCode" a `Address` o "Servicios" para `Rooms` está permitido, al igual que agregar un campo de nivel superior a un índice. Los documentos existentes tienen un valor null para los nuevos campos hasta que se rellenan explícitamente esos campos mediante la actualización de los datos.

Tenga en cuenta que dentro de un tipo complejo, cada campo secundario tiene un tipo y puede tener atributos, lo hacen los campos solo como nivel superior

### <a name="data-updates"></a>Actualizaciones de datos

Actualizar los documentos existentes en un índice con la acción de carga funciona del mismo modo para los campos complejos como simples, se reemplazan todos los campos. Sin embargo, combinar (o mergeOrUpload cuando se aplica a un documento existente) no funciona igual en todos los campos. En concreto, combinación no tiene la capacidad de combinar elementos dentro de una colección. Esto es cierto para las colecciones de tipos primitivos, como colecciones complejas. Para actualizar una colección, se necesita para recuperar el valor de la colección completa, realiza cambios y, a continuación, incluir la nueva colección en la solicitud de API de índice.


## <a name="searching-complex-fields"></a>Buscar campos complejos

Las expresiones de búsqueda de forma libre funcionan según lo esperado con tipos complejos. Si coincide con cualquier campo de búsqueda o el campo secundario en cualquier parte de un documento, el propio documento es una coincidencia. 

Get de las consultas más matizada cuando tiene varios términos y operadores, y algunos de los términos tienen nombres de campo especificados, como sea posible con el [sintaxis de Lucene](query-lucene-syntax.md). Por ejemplo, esta consulta se intenta hacer coincidir los dos términos, "Portland" y "OR", con dos campos secundarios del campo de dirección:

```json
search=Address/City:Portland AND Address/State:OR
```

Consultas como ésta no están correlacionadas para búsqueda de texto completo (a diferencia de los filtros, donde las consultas a través de campos secundarios de un conjunto complejo se pueden correlacionar usando cualquiera o de todo, al igual que una subconsulta correlacionada en SQL). Esto significa que la consulta de Lucene anterior devuelve documentos que contengan "Portland, Maine", así como "Portland, Oregón" y otras ciudades en Oregon. Esto es porque cada cláusula se evalúa con respecto a todos los valores del campo especificado en el documento completo, por lo que no hay ninguna noción de un "documento secundario actual". 

 

## <a name="selecting-complex-fields"></a>Seleccionar campos complejos

El `$select` parámetro se utiliza para elegir qué campos se devuelven en los resultados de búsqueda. Para utilizar este parámetro para seleccionar campos secundarios específicos de un campo complejo, incluya los campos de primarios y secundarios separados por una barra diagonal (`/`).

```json
$select=HotelName, Address/City, Rooms/BaseRate
```

Los campos deben marcarse como recuperable en el índice, si quiere que estén en los resultados de búsqueda. Solo los campos marcados como recuperable se pueden usar en un `$select` instrucción. 


## <a name="filter-facet-and-sort-complex-fields"></a>Filter, facetas y campos complejos de ordenación

El mismo [sintaxis de ruta de acceso de OData](query-odata-filter-orderby-syntax.md) usado para filtrar y responderán las búsquedas también se pueden usar para las facetas, ordenación y seleccionar los campos en una solicitud de búsqueda. Para tipos complejos, se aplican reglas que determinan qué campos secundarios se pueden marcar como que se puede ordenar o navegación por facetas. 

### <a name="faceting-sub-fields"></a>Uso de facetas subcampos. 

Cualquier subcampo puede marcarse como navegación por facetas a menos que sea de tipo `Edm.GeographyPoint` o `Collection(Edm.GeographyPoint)`. 

Cuando se devuelven los recuentos de documentos para la estructura de navegación por facetas, los recuentos son en relación con el documento principal (un hotel), no a documentos anidados dentro de un conjunto complejo (locales). Por ejemplo, suponga que un hotel tiene 20 salas de tipo "conjunto". Dado este parámetro de faceta `facet=Rooms/Type`, el recuento de faceta se uno para el documento principal (hoteles) y no intermedio documentos secundarios (locales). 

### <a name="sorting-complex-fields"></a>Ordenar campos complejos

Las operaciones de ordenación se aplican a documentos (hoteles) y no documentos secundarios (locales). Cuando haya una colección de tipo complejo, como las salas, es importante tener en cuenta que no se puede ordenar en salas en absoluto. De hecho, no se puede ordenar en cualquier colección. 

Las operaciones de ordenación funcionan correctamente cuando los campos valor único, ya sea como un campo sencillo o como un subcampo en un tipo complejo. Por ejemplo, `$orderby=Address/ZipCode` tipo complejo es que se puede ordenar porque hay solo un código postal por hotel. 

Redefinición de las reglas de ordenación, dentro de un campo de índice debe estar marcada como filtrable y ordenable para usarse en un `$orderby` instrucción. 

## <a name="next-steps"></a>Pasos siguientes

 Pruebe el [conjunto de datos de hoteles](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/README.md) en el **importar datos** asistente. Necesitará la información de conexión de Cosmos DB proporciona en el archivo Léame para obtener acceso a los datos. 
 
 Con esa información a mano, el primer paso del asistente es crear un nuevo origen de datos de Azure Cosmos DB. Aún más en el asistente, cuando llegue a la página de índice de destino, verá un índice con tipos complejos. Crear y cargar este índice y, a continuación, ejecutar consultas para entender la estructura nuevo.

> [!div class="nextstepaction"]
> [Inicio rápido: Asistente para la importación, indexación y las consultas del portal](search-get-started-portal.md)