---
title: Modelado de datos relacionales de SQL para la importación y la indexación
titleSuffix: Azure Cognitive Search
description: Aprenda a modelar datos relacionales, desnormalizados en un conjunto de resultados planos, para la indexación y la búsqueda de texto completo en Azure Cognitive Search.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 6c70b42e7d0f647a3b2b60d29b5098a791e4975f
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88924527"
---
# <a name="how-to-model-relational-sql-data-for-import-and-indexing-in-azure-cognitive-search"></a>Modelado de datos relacionales de SQL para la importación y la indexación en Azure Cognitive Search

Azure Cognitive Search acepta un conjunto de filas plano como entrada para la [canalización de indexación](search-what-is-an-index.md). Si los datos de origen proceden de tablas combinadas en una base de datos relacional de SQL Server, en este artículo se explica cómo construir el conjunto de resultados y cómo modelar una relación de elementos primarios y secundarios en un índice de Azure Cognitive Search.

Como ejemplo, haremos referencia a una base de datos hipotética de hoteles basada en [datos de demostración](https://github.com/Azure-Samples/azure-search-sample-data/tree/master/hotels). Supongamos que la base de datos consta de una tabla Hotels$ con 50 hoteles y una tabla Rooms$ con 750 habitaciones de tipos, tarifas y servicios distintos. Hay una relación uno a varios entre las tablas. En nuestro enfoque, una vista proporcionará la consulta que devuelve 50 filas, una por hotel, con los detalles de la habitación asociada insertados en cada fila.

   ![Tablas y vista de la base de datos de hoteles](media/index-sql-relational-data/hotels-database-tables-view.png "Tablas y vista de la base de datos de hoteles")


## <a name="the-problem-of-denormalized-data"></a>El problema de los datos desnormalizados

Uno de los desafíos del uso de relaciones "uno a varios" es que las consultas estándar creadas en tablas combinadas devuelven datos desnormalizados, que no funcionan bien en un escenario de Azure Cognitive Search. Considere el ejemplo siguiente que combina hoteles y habitaciones.

```sql
SELECT * FROM Hotels$
INNER JOIN Rooms$
ON Rooms$.HotelID = Hotels$.HotelID
```
Los resultados de esta consulta devuelven todos los campos de hotel, seguidos de todos los campos de habitación, con la información de hotel preliminar repetida para cada valor de habitación.

   ![Datos desnormalizados, datos de hotel redundantes al agregar campos de habitación](media/index-sql-relational-data/denormalize-data-query.png "Datos desnormalizados, datos de hotel redundantes al agregar campos de habitación")


Aunque esta consulta se realiza correctamente en la superficie (proporciona todos los datos en un conjunto de filas plano), se produce un error al entregar la estructura de documento correcta para la experiencia de búsqueda esperada. Durante la indexación, Azure Cognitive Search creará un documento de búsqueda para cada fila ingerida. Si los documentos de búsqueda se parecían a los resultados anteriores, habrá observado duplicados (siete documentos independientes solo para el hotel Twin Dome). Una consulta sobre los "hoteles en Florida" devolvería siete resultados solo para el hotel Twin Dome, y empujaría a otros hoteles adecuados más abajo en la lista de resultados de búsqueda.

Para obtener la experiencia esperada de un documento por hotel, debe proporcionar un conjunto de filas con la granularidad adecuada y toda la información. Afortunadamente, puede hacerlo fácilmente mediante la adopción de las técnicas de este artículo.

## <a name="define-a-query-that-returns-embedded-json"></a>Definición de una consulta que devuelve JSON insertado

Para proporcionar la experiencia de búsqueda esperada, el conjunto de datos debe constar de una fila por cada documento de búsqueda de Azure Cognitive Search. En nuestro ejemplo, queremos una fila para cada hotel, pero también que los usuarios puedan buscar otros campos relacionados con la habitación que les interese, como la tarifa por noche, el tamaño y el número de camas, o vistas a la playa, elementos que forman parte de los detalles de la habitación.

La solución consiste en capturar los detalles de la habitación como JSON anidado e insertar la estructura JSON en un campo de una vista, como se muestra en el segundo paso. 

1. Supongamos que tenemos dos tablas combinadas, Hotels$ y Rooms$, con detalles de 50 hoteles y 750 habitaciones, y que se combinan en el campo HotelID. Individualmente, estas tablas contienen 50 hoteles y 750 habitaciones asociadas.

    ```sql
    CREATE TABLE [dbo].[Hotels$](
      [HotelID] [nchar](10) NOT NULL,
      [HotelName] [nvarchar](255) NULL,
      [Description] [nvarchar](max) NULL,
      [Description_fr] [nvarchar](max) NULL,
      [Category] [nvarchar](255) NULL,
      [Tags] [nvarchar](255) NULL,
      [ParkingIncluded] [float] NULL,
      [SmokingAllowed] [float] NULL,
      [LastRenovationDate] [smalldatetime] NULL,
      [Rating] [float] NULL,
      [StreetAddress] [nvarchar](255) NULL,
      [City] [nvarchar](255) NULL,
      [State] [nvarchar](255) NULL,
      [ZipCode] [nvarchar](255) NULL,
      [GeoCoordinates] [nvarchar](255) NULL
    ) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
    GO

    CREATE TABLE [dbo].[Rooms$](
      [HotelID] [nchar](10) NULL,
      [Description] [nvarchar](255) NULL,
      [Description_fr] [nvarchar](255) NULL,
      [Type] [nvarchar](255) NULL,
      [BaseRate] [float] NULL,
      [BedOptions] [nvarchar](255) NULL,
      [SleepsCount] [float] NULL,
      [SmokingAllowed] [float] NULL,
      [Tags] [nvarchar](255) NULL
    ) ON [PRIMARY]
    GO
    ```

2. Cree una vista compuesta por todos los campos de la tabla primaria (`SELECT * from dbo.Hotels$`), con la incorporación de un nuevo campo *Rooms* que contenga la salida de una consulta anidada. Una cláusula **FOR JSON AUTO** en `SELECT * from dbo.Rooms$` estructura la salida como JSON. 

     ```sql
   CREATE VIEW [dbo].[HotelRooms]
   AS
   SELECT *, (SELECT *
              FROM dbo.Rooms$
              WHERE dbo.Rooms$.HotelID = dbo.Hotels$.HotelID FOR JSON AUTO) AS Rooms
   FROM dbo.Hotels$
   GO
   ```

   En la captura de pantalla siguiente se muestra la vista resultante, con el campo nvarchar de *Rooms* en la parte inferior. El campo *Rooms* solo existe en la vista HotelRooms.

   ![Vista HotelRooms](media/index-sql-relational-data/hotelsrooms-view.png "Vista HoteRooms")

1. Ejecute `SELECT * FROM dbo.HotelRooms` para recuperar el conjunto de filas. Esta consulta devuelve 50 filas, una por hotel, con la información de la habitación asociada como colección JSON. 

   ![Conjunto de filas de la vista HotelRooms](media/index-sql-relational-data/hotelrooms-rowset.png "Conjunto de filas de la vista HotelRooms")

Este conjunto de filas ya está listo para la importación en Azure Cognitive Search.

> [!NOTE]
> Este enfoque presupone que el JSON insertado cumple los [límites de tamaño de columna máximos de SQL Server](/sql/sql-server/maximum-capacity-specifications-for-sql-server). 

 ## <a name="use-a-complex-collection-for-the-many-side-of-a-one-to-many-relationship"></a>Uso de una colección compleja para el lado "varios" de una relación "uno a varios"

En Azure Cognitive Search, cree un esquema de índice que modele la relación "uno a varios" mediante JSON anidado. El conjunto de resultados que creó en la sección anterior suele corresponderse con el esquema de índice que se proporciona a continuación (hemos acortado algunos campos por motivos de brevedad).

El ejemplo siguiente es similar al de [Modelado de tipos de datos complejos](search-howto-complex-data-types.md#creating-complex-fields). La estructura de *Rooms*, en la que se centra este artículo, se encuentra en la colección de campos de un índice denominado *hotels*. En este ejemplo también se muestra un tipo complejo para *Address*, diferente del de *Rooms*, ya que se compone de un conjunto fijo de elementos, en lugar del número arbitrario y múltiple de elementos que se permiten en una colección.

```json
{
  "name": "hotels",
  "fields": [
    { "name": "HotelId", "type": "Edm.String", "key": true, "filterable": true },
    { "name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false },
    { "name": "Description", "type": "Edm.String", "searchable": true, "analyzer": "en.lucene" },
    { "name": "Description_fr", "type": "Edm.String", "searchable": true, "analyzer": "fr.lucene" },
    { "name": "Category", "type": "Edm.String", "searchable": true, "filterable": false },
    { "name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": true, "facetable": true },
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
        { "name": "Description_fr", "type": "Edm.String", "searchable": true, "analyzer": "fr.lucene" },
        { "name": "Type", "type": "Edm.String", "searchable": true },
        { "name": "BaseRate", "type": "Edm.Double", "filterable": true, "facetable": true },
        { "name": "BedOptions", "type": "Edm.String", "searchable": true, "filterable": true, "facetable": true },
        { "name": "SleepsCount", "type": "Edm.Int32", "filterable": true, "facetable": true },
        { "name": "SmokingAllowed", "type": "Edm.Boolean", "filterable": true, "facetable": true },
        { "name": "Tags", "type": "Edm.Collection", "searchable": true }
      ]
    }
  ]
}
```

Con el conjunto de resultados y el esquema de índice anteriores tiene todos los componentes necesarios para que la operación de indexación sea correcta. El conjunto de datos aplanado cumple los requisitos de indexación al tiempo que conserva la información detallada. En el índice de Azure Cognitive Search, los resultados de la búsqueda se encontrarán fácilmente en las entidades basadas en hoteles y conservarán el contexto de las habitaciones individuales y sus atributos.

## <a name="next-steps"></a>Pasos siguientes

Con su propio conjunto de datos, puede usar el [Asistente para la importación de datos](search-import-data-portal.md) para crear y cargar el índice. El asistente detecta la colección JSON insertada, como la que se incluye en *Rooms*, e infiere un esquema de índice que incluye una colección de tipos complejos. 

  ![Índice inferido por el Asistente para la importación de datos](media/index-sql-relational-data/search-index-rooms-complex-collection.png "Índice inferido por el Asistente para la importación de datos")

Pruebe el inicio rápido siguiente para aprender los pasos básicos del Asistente para la importación de datos.

> [!div class="nextstepaction"]
> [Inicio rápido: Creación de un índice de búsqueda mediante Azure Portal](search-get-started-portal.md)