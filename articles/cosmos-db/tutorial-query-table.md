---
title: ¿Cómo consultar datos de tabla en Azure Cosmos DB?
description: Obtenga información sobre cómo consultar los datos almacenados en la cuenta de Table API de Azure Cosmos DB mediante filtros OData y consultas LINQ.
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: tutorial
ms.date: 06/05/2020
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 481f1353e16fecd0e413152db89a4ae54824a0f8
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019290"
---
# <a name="tutorial-query-azure-cosmos-db-by-using-the-table-api"></a>Tutorial: Consulta de Azure Cosmos DB mediante la Table API

[Table API](table-introduction.md) de Azure Cosmos DB admite consultas de OData y [LINQ](/rest/api/storageservices/fileservices/writing-linq-queries-against-the-table-service) en los datos de clave-valor (tabla).  

En este artículo se tratan las tareas siguientes:

> [!div class="checklist"]
> * Consulta de datos con Table API

En las consultas de este artículo se usa la tabla `People` de ejemplo siguiente:

| PartitionKey | RowKey | Email | PhoneNumber |
| --- | --- | --- | --- |
| Harp | Walter | Walter@contoso.com| 425-555-0101 |
| Smith | Ben | Ben@contoso.com| 425-555-0102 |
| Smith | Jeff | Jeff@contoso.com| 425-555-0104 |

Vea [Consultar tablas y entidades](/rest/api/storageservices/fileservices/querying-tables-and-entities) para detalles sobre cómo realizar consultas mediante Table API.

Para más información sobre las funcionalidades premium que ofrece Azure Cosmos DB, consulte el artículo [Azure Cosmos DB: Table API](table-introduction.md) y [Azure Cosmos DB: desarrollo con Table API en .NET](tutorial-develop-table-dotnet.md).

## <a name="prerequisites"></a>Requisitos previos

Para que estas consultas funcionen, debe tener una cuenta de Azure Cosmos DB, así como datos de la entidad en el contenedor. ¿No tiene nada de lo anterior? Efectúe el [inicio rápido en cinco minutos](create-table-dotnet.md) o siga el [tutorial de desarrolladores](tutorial-develop-table-dotnet.md) para crear una cuenta y rellenar la base de datos.

## <a name="query-on-partitionkey-and-rowkey"></a>Consultas en PartitionKey y RowKey

Dado que las propiedades PartitionKey y RowKey forman la clave principal de una entidad, puede usar la siguiente sintaxis especial para identificar dicha entidad:

**Consultar**

```
https://<mytableendpoint>/People(PartitionKey='Harp',RowKey='Walter')  
```

**Resultados**

| PartitionKey | RowKey | Email | PhoneNumber |
| --- | --- | --- | --- |
| Harp | Walter | Walter@contoso.com| 425-555-0104 |

Como alternativa, puede especificar estas propiedades como parte de la opción `$filter`, tal y como se muestra en la sección siguiente. Tenga en cuenta que los nombres de propiedad de clave y los valores constantes distinguen entre mayúsculas y minúsculas. Las propiedades PartitionKey y RowKey son de tipo String.

## <a name="query-by-using-an-odata-filter"></a>Consultas mediante un filtro de OData

Al construir una cadena de filtro, tenga en cuenta estas reglas:

* Utilice los operadores lógicos definidos por la especificación del protocolo OData para comparar una propiedad con un valor. Tenga en cuenta que no puede comparar una propiedad con un valor dinámico. Un lado de la expresión debe ser una constante.
* El nombre de la propiedad, el operador y el valor constante se deben separar por espacios con codificación URL. Para codificar un espacio con codificación URL se usa `%20`.
* Todas las partes de la cadena de filtro distinguen mayúsculas de minúsculas.
* Para que el filtro devuelva resultados válidos, el valor constante debe ser del mismo tipo de datos que la propiedad. Para obtener más información sobre los tipos de propiedades que se admiten, consulte [Introducción al modelo de datos del servicio Tabla](/rest/api/storageservices/understanding-the-table-service-data-model).

Esta es una consulta de ejemplo en la que se muestra cómo filtrar por propiedades PartitionKey e Email con un elemento `$filter` de OData.

**Consultar**

```
https://<mytableapi-endpoint>/People()?$filter=PartitionKey%20eq%20'Smith'%20and%20Email%20eq%20'Ben@contoso.com'
```

Para obtener más información sobre cómo construir expresiones de filtro para diferentes tipos de datos, consulte [Consulta de tablas y entidades](/rest/api/storageservices/querying-tables-and-entities).

**Resultados**

| PartitionKey | RowKey | Email | PhoneNumber |
| --- | --- | --- | --- |
| Smith |Ben | Ben@contoso.com| 425-555-0102 |

Las consultas de las propiedades datetime no devolverán datos si se ejecutan en Table API de Azure Cosmos DB. Aunque Azure Table Storage almacena valores de datos con una granularidad de tiempo de tics, Table API en Azure Cosmos DB usa la propiedad `_ts`. La propiedad `_ts` está en un segundo nivel de granularidad, el cual no es un filtro de OData. Por lo tanto, Azure Cosmos DB bloqueará las consultas a las propiedades timestamp. Como solución alternativa, puede definir una propiedad personalizada con un tipo de datos datetime o long y establecer el valor de fecha desde el cliente.

## <a name="query-by-using-linq"></a>Consultas con LINQ 
También puede realizar consultas con LINQ, que se traduce a las correspondientes expresiones de consulta de OData. Este es un ejemplo de cómo compilar consultas mediante el SDK de .NET:

```csharp
IQueryable<CustomerEntity> linqQuery = table.CreateQuery<CustomerEntity>()
            .Where(x => x.PartitionKey == "4")
            .Select(x => new CustomerEntity() { PartitionKey = x.PartitionKey, RowKey = x.RowKey, Email = x.Email });
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha hecho lo siguiente:

> [!div class="checklist"]
> * Ha aprendido a realizar consultas mediante Table API

Ahora puede continuar con el tutorial siguiente para obtener información sobre cómo distribuir sus datos globalmente.

> [!div class="nextstepaction"]
> [Distribución de datos global](tutorial-global-distribution-table.md)
