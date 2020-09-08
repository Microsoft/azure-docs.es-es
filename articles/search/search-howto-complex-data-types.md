---
title: Modelado de tipos de datos complejos
titleSuffix: Azure Cognitive Search
description: Las estructuras de datos jerárquicas o anidadas se pueden modelar en un índice de Azure Cognitive Search mediante los tipos de datos ComplexType y Collections.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
tags: complex data types; compound data types; aggregate data types
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/12/2020
ms.openlocfilehash: 2b26a317f7338b3e87623b8312d9f7efd10dbed1
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88917863"
---
# <a name="how-to-model-complex-data-types-in-azure-cognitive-search"></a>Modelado de tipos de datos complejos en Azure Cognitive Search

Los conjuntos de datos externos usados para rellenar un índice de Azure Cognitive Search pueden tener muchas formas. A veces, incluyen subestructuras jerárquicas o anidadas. Algunos ejemplos son varias direcciones de un solo cliente, varios tamaños y colores de una única SKU, varios autores de un único libro, etc. En términos de modelado, puede que vea que se hace referencia a estas estructuras como tipos de datos *complejos*, *compuestos* o *agregados* *.* El término que Azure Cognitive Search usa para este concepto es **tipo complejo**. En Azure Cognitive Search, los tipos complejos se modelan mediante **campos complejos**. Un campo complejo es un campo que contiene a elementos secundarios (campos secundarios) que pueden ser de cualquier tipo de datos, incluidos otros tipos complejos. Esto funciona de forma similar a los tipos de datos estructurados de un lenguaje de programación.

Los campos complejos representan un único objeto en el documento, o bien una matriz de objetos, en función del tipo de datos. Los campos de tipo `Edm.ComplexType` representan objetos individuales, mientras que los campos de tipo `Collection(Edm.ComplexType)` representan matrices de objetos.

Azure Cognitive Search admite de forma nativa colecciones y tipos complejos. Estos tipos le permiten modelar casi cualquier estructura JSON en un índice de Azure Cognitive Search. En versiones anteriores de las API de Azure Cognitive Search, solo se podían importar conjuntos de filas planas. En la versión más reciente, ahora el índice puede corresponderse de forma más exacta con los datos de origen. En otras palabras, si los datos de origen tienen tipos complejos, el índice también puede tener tipos complejos.

Para empezar, recomendamos el [conjunto de datos Hotels](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/README.md), que puede cargarse en el asistente **Importar datos** de Azure Portal. El asistente detecta los tipos complejos en el origen y sugiere un esquema de índice basado en las estructuras detectadas.

> [!Note]
> La compatibilidad con tipos complejos está disponible con carácter general desde `api-version=2019-05-06`. 
>
> Si su solución de búsqueda se basa en soluciones alternativas anteriores de conjuntos de datos planos de una colección, debe cambiar su índice para incluir tipos complejos según se admite en la versión más nueva de la API. Para obtener más información acerca de cómo actualizar las versiones de la API, consulte [Actualización a la versión más reciente de la API REST](search-api-migration.md) o [Actualización a la versión más reciente del SDK de .NET](search-dotnet-sdk-migration-version-9.md).

## <a name="example-of-a-complex-structure"></a>Ejemplo de una estructura compleja

El siguiente documento JSON se compone de campos simples y complejos. Los campos complejos, tales como `Address` y `Rooms`, tienen campos secundarios. `Address` tiene un único conjunto de valores para esos campos secundarios, puesto que es un objeto único en el documento. En cambio, `Rooms` tiene varios conjuntos de valores para sus campos secundarios, uno para cada objeto de la colección.

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
      "BaseRate": 96.99
    },
    {
      "Description": "Deluxe Room, 2 Double Beds (City View)",
      "Type": "Deluxe Room",
      "BaseRate": 150.99
    },
  ]
}
```

## <a name="creating-complex-fields"></a>Creación de campos complejos

Al igual que con cualquier definición de índice, puede usar el portal, la [API REST](/rest/api/searchservice/create-index) o el [SDK de .NET](/dotnet/api/microsoft.azure.search.models.index?view=azure-dotnet) para crear un esquema que incluya tipos complejos. 

En el ejemplo siguiente se muestra un esquema de índice JSON con campos simples, colecciones y tipos complejos. Tenga en cuenta que, en un tipo complejo, cada campo secundario tiene un tipo y puede tener atributos, igual que sucede con los campos de nivel superior. El esquema se corresponde con los datos de ejemplo anteriores. `Address` es un campo complejo que no es una colección (un hotel tiene una dirección). `Rooms` es un campo de colección complejo (un hotel tiene varias salas).

```json
{
  "name": "hotels",
  "fields": [
    { "name": "HotelId", "type": "Edm.String", "key": true, "filterable": true },
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

## <a name="updating-complex-fields"></a>Actualización de los campos complejos

Todas las [reglas de nueva indexación](search-howto-reindex.md) que se aplican a los campos en general se aplican igualmente a los campos complejos. Si replanteamos algunas de las reglas principales aquí, la adición de un campo no requiere la recompilación de un índice, pero la mayoría de las modificaciones sí.

### <a name="structural-updates-to-the-definition"></a>Actualizaciones estructurales en la definición

Puede agregar nuevos campos secundarios a un campo complejo en cualquier momento sin necesidad de recompilar un índice. Por ejemplo, la adición de "ZipCode" a `Address` o "Amenities" a `Rooms` se permite, al igual que la adición de un campo de nivel superior a un índice. Los documentos existentes tienen un valor null para los nuevos campos hasta que rellena explícitamente dichos campos al actualizar sus datos.

Tenga en cuenta que, en un tipo complejo, cada campo secundario tiene un tipo y puede tener atributos, al igual que sucede con los campos de nivel superior.

### <a name="data-updates"></a>Actualizaciones de datos

La actualización de documentos existentes en un índice con la acción `upload` funciona del mismo modo tanto para los campos complejos como simples: se reemplazan todos los campos. Sin embargo, `merge` (o `mergeOrUpload` cuando se aplica a un documento existente) no funciona igual en todos los campos. En concreto, `merge` no admite la combinación de elementos dentro de una colección. Esta limitación existe para las colecciones de tipos primitivos y complejas. Para actualizar una colección, deberá recuperar el valor de la colección completa, realizar cambios y, a continuación, incluir la nueva colección en la solicitud de API de índice.

## <a name="searching-complex-fields"></a>Búsqueda de campos complejos

Las expresiones de búsqueda de forma libre funcionan según lo esperado con tipos complejos. Si cualquier campo o campo secundario de búsqueda de cualquier parte de un documento coincide, entonces el documento en sí es una coincidencia.

Las consultas adquieren más matices cuando tiene varios términos y operadores, y algunos términos tienen nombres de campos especificados, tal y como es posible con la [sintaxis de Lucene](query-lucene-syntax.md). Por ejemplo, esta consulta intenta hacer coincidir dos términos, "Portland" y "OR", con dos campos secundarios del campo Dirección:

> `search=Address/City:Portland AND Address/State:OR`

Las consultas de este tipo *no están correlacionadas*  para la búsqueda de texto completo, a diferencia de los filtros. En los filtros, las consultas a través de campos secundarios de una colección compleja se correlacionan mediante variables de rango en [`any` o `all`](search-query-odata-collection-operators.md). La consulta de Lucene anterior devuelve documentos que contienen "Portland, Maine" y "Portland, Oregon", junto con otras ciudades de Oregón. Esto sucede porque cada cláusula se aplica a todos los valores de su campo en todo el documento, por lo que no hay ningún concepto de un "documento secundario actual". Para obtener más información, consulte [Descripción de los filtros de colección de OData en Azure Cognitive Search](search-query-understand-collection-filters.md).

## <a name="selecting-complex-fields"></a>Selección de los campos complejos

El parámetro `$select` se utiliza para elegir qué campos se devuelven en los resultados de la búsqueda. Para utilizar este parámetro para seleccionar campos secundarios específicos de un campo complejo, incluya los campos primario y secundario separados por una barra diagonal (`/`).

> `$select=HotelName, Address/City, Rooms/BaseRate`

Los campos deben marcarse como Recuperables en el índice, si quiere que aparezcan en los resultados de la búsqueda. Solo los campos marcados como Recuperables se pueden usar en una instrucción `$select`.

## <a name="filter-facet-and-sort-complex-fields"></a>Filtros, facetas y orden de los campos complejos

La misma [sintaxis de ruta de acceso de OData ](query-odata-filter-orderby-syntax.md) utilizada para el filtrado y las búsquedas por campos también se puede usar para ordenar y seleccionar campos en una solicitud de búsqueda, así como para definirles facetas. En el caso de los tipos complejos, se aplican reglas que rigen los campos secundarios que se pueden marcar como definibles por facetas u ordenables. Para obtener más información sobre estas reglas, vea la [referencia de la API de creación de índices](/rest/api/searchservice/create-index).

### <a name="faceting-sub-fields"></a>Definición de facetas de los campos secundarios

Cualquier campo secundario puede marcarse como definible por facetas a menos que sea de tipo `Edm.GeographyPoint` o `Collection(Edm.GeographyPoint)`.

Los recuentos de documentos devueltos en los resultados de facetas se calculan para el documento principal (un hotel), y no para los documentos secundarios de una colección compleja (salas). Por ejemplo, supongamos que un hotel tiene 20 habitaciones de tipo "suite". Dado este parámetro de faceta `facet=Rooms/Type`, el recuento de facetas será uno para el hotel, y no 20 para las salas.

### <a name="sorting-complex-fields"></a>Ordenación de los campos complejos

Las operaciones de ordenación se aplican a documentos (hoteles) y no a documentos secundarios (salas). Cuando haya una colección de tipo complejo, como las salas, es importante tener en cuenta que se puede ordenar en salas de ningún modo. De hecho, no puede ordenar ninguna colección.

Las operaciones de ordenación funcionan cuando los campos tienen un único valor por documento, tanto si se trata de un campo sencillo o de un campo secundario en un tipo complejo. Por ejemplo, `Address/City` puede ordenarse porque solo hay una dirección por hotel, por lo que `$orderby=Address/City` ordenará los hoteles por ciudad.

### <a name="filtering-on-complex-fields"></a>Filtrado en campos complejos

Se puede hacer referencia a campos secundarios de un campo complejo en una expresión de filtro. Simplemente use la misma [sintaxis de ruta de acceso de OData](query-odata-filter-orderby-syntax.md) que se usa para ordenar y seleccionar campos, así como para definirles facetas. Por ejemplo, el filtro siguiente devolverá todos los hoteles de Canadá:

> `$filter=Address/Country eq 'Canada'`

Para filtrar según un campo de colección complejo, puede usar una **expresión lambda** con los [operadores `any` y `all`](search-query-odata-collection-operators.md). En ese caso, la **variable de rango** de la expresión lambda es un objeto con campos secundarios. Puede hacer referencia a esos campos secundarios con la sintaxis de ruta de acceso de OData estándar. Por ejemplo, el filtro siguiente devolverá los hoteles que tengan al menos una habitación deluxe y todas las habitaciones en las que no se permita fumar:

> `$filter=Rooms/any(room: room/Type eq 'Deluxe Room') and Rooms/all(room: not room/SmokingAllowed)`

Al igual que con los campos simples de nivel superior, los campos secundarios simples de campos complejos solo pueden incluirse en los filtros si tienen el atributo **filtrable** establecido en `true` en la definición del índice. Para obtener más información, vea la [referencia de la API de creación de índices](/rest/api/searchservice/create-index).

## <a name="next-steps"></a>Pasos siguientes

Pruebe el [conjunto de datos de hoteles](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/README.md) en el asistente **Importar datos**. Necesitará la información de conexión de Cosmos DB proporcionada en el archivo Léame para acceder a los datos.

Con esa información a mano, el primer paso del asistente es crear un nuevo origen de datos de Azure Cosmos DB. Más adelante en el asistente, cuando llegue a la página de índice de destino, verá un índice con tipos complejos. Cree y cargue este índice y, luego, ejecute las consultas para comprender la nueva estructura.

> [!div class="nextstepaction"]
> [Inicio rápido: asistente para la importación, la indexación y las consultas del portal](search-get-started-portal.md)