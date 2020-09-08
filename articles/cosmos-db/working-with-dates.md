---
title: Trabajo con fechas en Azure Cosmos DB
description: Obtenga información acerca de cómo almacenar, indexar y consultar objetos DataTime en Azure Cosmos DB
ms.service: cosmos-db
author: SnehaGunda
ms.author: sngun
ms.topic: conceptual
ms.date: 04/03/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 7a524d34fcf8e629914d4fbc4669caada210866b
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019195"
---
# <a name="working-with-dates-in-azure-cosmos-db"></a>Trabajo con fechas en Azure Cosmos DB

Azure Cosmos DB proporciona flexibilidad de esquema e indexación completa mediante un modelo de datos [JSON](https://www.json.org) nativo. Todos los recursos de Azure Cosmos DB, incluidas las bases de datos, los contenedores, los documentos y los procedimientos almacenados, están modelados y almacenados como documentos JSON. Como requisito para ser portátil, JSON (y Azure Cosmos DB) admite solo un pequeño conjunto de tipos básicos: String, Number, Boolean, Array, Object y Null. Sin embargo, JSON es flexible y permite que los desarrolladores y marcos de trabajo representen tipos más complejos mediante estos primitivos y los compongan como objetos o matrices.

Además de los tipos básicos, muchas aplicaciones necesitan el tipo DateTime para representar fechas y marcas de tiempo. En este artículo se explica cómo los desarrolladores pueden almacenar, recuperar y consultar fechas en Azure Cosmos DB mediante el SDK de .NET.

## <a name="storing-datetimes"></a>Almacenamiento de valores DateTime

Azure Cosmos DB admite tipos JSON, como cadena, número, booleano, null, matriz y objeto. No admite directamente un tipo DateTime. Actualmente, Azure Cosmos DB no admite la localización de fechas. Por lo tanto, debe almacenar los valores de fecha y hora como cadenas. El formato recomendado para las cadenas DateTime en Azure Cosmos DB es `yyyy-MM-ddTHH:mm:ss.fffffffZ`, que sigue el estándar UTC de ISO 8601. Se recomienda almacenar todas las fechas en Azure Cosmos DB como UTC. La conversión de cadenas de fecha a este formato permite ordenar las fechas lexicográficamente. Si se almacenan fechas que no son UTC, la lógica debe administrarse en el lado cliente. Para convertir un valor de fecha y hora local en UTC, el desplazamiento se debe conocer o almacenar como propiedad en JSON y el cliente puede usar el desplazamiento para calcular el valor de fecha y hora UTC.

Las consultas por intervalos con cadenas de fecha y hora como filtros solo se admiten si las cadenas de fecha y hora están todas en UTC y tienen la misma longitud. En Azure Cosmos DB, la función del sistema [GetCurrentDateTime](sql-query-getcurrentdatetime.md) devolverá el valor de la cadena UTC de la fecha y hora ISO 8601 con el formato: `yyyy-MM-ddTHH:mm:ss.fffffffZ`.

La mayoría de las aplicaciones pueden la representación de cadena predeterminada para DateTime por los siguientes motivos:

* Las cadenas se pueden comparar, y se conserva el orden relativo de los valores DateTime cuando se transforman en cadenas.
* Este enfoque no requiere ninguna personalización del código o de los atributos para la conversión de JSON.
* Las fechas almacenadas en JSON son legibles para el usuario.
* Este enfoque puede aprovechar el índice de Azure Cosmos DB para aumentar el rendimiento de las consultas.

Por ejemplo, el fragmento de código siguiente almacena un objeto `Order` que contiene dos propiedades de DateTime: `ShipDate` y `OrderDate` como un documento mediante el SDK de .NET:

```csharp
    public class Order
    {
        [JsonProperty(PropertyName="id")]
        public string Id { get; set; }
        public DateTime OrderDate { get; set; }
        public DateTime ShipDate { get; set; }
        public double Total { get; set; }
    }

    await container.CreateItemAsync(
        new Order
        {
            Id = "09152014101",
            OrderDate = DateTime.UtcNow.AddDays(-30),
            ShipDate = DateTime.UtcNow.AddDays(-14),
            Total = 113.39
        });
```

Este documento se almacena en Azure Cosmos DB de la manera siguiente:

```json
    {
        "id": "09152014101",
        "OrderDate": "2014-09-15T23:14:25.7251173Z",
        "ShipDate": "2014-09-30T23:14:25.7251173Z",
        "Total": 113.39
    }
```  

Como alternativa, puede almacenar valores DateTime como marcas de tiempo de Unix, es decir, como un número que representa el número de segundos transcurridos desde el 1 de enero de 1970. La propiedad Timestamp (`_ts`) interna de Azure Cosmos DB sigue este enfoque. Puede usar la clase [UnixDateTimeConverter](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.unixdatetimeconverter.aspx) clase para serializar los valores DateTime como números.

## <a name="querying-datetimes-in-linq"></a>Consulta de valores DateTimes en LINQ

El SDK de .NET para SQL admite automáticamente la consulta de datos almacenados en Azure Cosmos DB mediante LINQ. Por ejemplo, el fragmento de código siguiente muestra una consulta LINQ que filtra los pedidos que se enviaron en los últimos tres días:

```csharp
    IQueryable<Order> orders = container.GetItemLinqQueryable<Order>(allowSynchronousQueryExecution: true).Where(o => o.ShipDate >= DateTime.UtcNow.AddDays(-3));
```

Traducido a la siguiente instrucción SQL y ejecutado en Azure Cosmos DB:

```sql
    SELECT * FROM root WHERE (root["ShipDate"] >= "2014-09-30T23:14:25.7251173Z")
```

Puede obtener más información sobre el lenguaje de consulta SQL de Azure Cosmos DB y el proveedor LINQ en [Consulta de Cosmos DB en LINQ](sql-query-linq-to-sql.md).

## <a name="indexing-datetimes-for-range-queries"></a>Indexación de valores DateTime para consultas de rango

Las consultas son comunes con valores DateTime. Para ejecutar estas consultas de forma eficaz, tiene que tener un índice definido en las propiedades del filtro de la consulta.

Puede aprender más sobre cómo configurar directivas de indexación en [¿Cómo funcionan los datos del índice de Azure Cosmos DB?](index-policy.md) 

## <a name="next-steps"></a>Pasos siguientes

* Descargue y ejecute los [ejemplos de código en GitHub](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/code-samples).
* Obtener más información acerca de las [consultas de SQL](sql-query-getting-started.md)
* Obtener más información sobre [Directivas de indexación de Azure Cosmos DB](index-policy.md)
