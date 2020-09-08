---
title: Obtención de métricas de rendimiento y ejecución de consultas SQL
description: Obtenga información sobre cómo recuperar las métricas de ejecución de consultas SQL y generar perfiles de rendimiento de las consultas SQL de las solicitudes de Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/17/2019
ms.author: girobins
ms.custom: devx-track-csharp
ms.openlocfilehash: 71ebc90834083def5b82e16dc387a6e61943206d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021755"
---
# <a name="get-sql-query-execution-metrics-and-analyze-query-performance-using-net-sdk"></a>Obtención de métricas de ejecución de consultas SQL y análisis del rendimiento de las consultas mediante el SDK de .NET

En este artículo se presenta cómo generar perfiles de rendimiento de las consultas SQL en Azure Cosmos DB. Esta generación de perfiles puede realizarse mediante el objeto `QueryMetrics` recuperado del SDK de .NET y se detalla aquí. [QueryMetrics](https://msdn.microsoft.com/library/microsoft.azure.documents.querymetrics.aspx) es un objeto fuertemente tipado con información sobre la ejecución de consultas de back-end. Estas métricas se documentan con más detalle en el artículo [Ajuste el rendimiento de las consultas](https://docs.microsoft.com/azure/cosmos-db/documentdb-sql-query-metrics).

## <a name="set-the-feedoptions-parameter"></a>Establecer el parámetro FeedOptions

Todas las sobrecargas de [DocumentClient.CreateDocumentQuery](https://msdn.microsoft.com/library/microsoft.azure.documents.client.documentclient.createdocumentquery.aspx) toman un parámetro [FeedOptions](https://msdn.microsoft.com/library/microsoft.azure.documents.client.feedoptions.aspx) opcional. Esta opción es la que permite ajustar y parametrizar la ejecución de consultas. 

Para recopilar las métricas de ejecución de consultas SQL, debe establecer el parámetro [PopulateQueryMetrics](https://msdn.microsoft.com/library/microsoft.azure.documents.client.feedoptions.populatequerymetrics.aspx#P:Microsoft.Azure.Documents.Client.FeedOptions.PopulateQueryMetrics) de [FeedOptions](https://msdn.microsoft.com/library/microsoft.azure.documents.client.feedoptions.aspx) en `true`. Si `PopulateQueryMetrics` se establece en true, `FeedResponse` contendrá el objeto `QueryMetrics` pertinente. 

## <a name="get-query-metrics-with-asdocumentquery"></a>Obtención de métricas de consulta con AsDocumentQuery()
El ejemplo de código siguiente muestra cómo recuperar las métricas cuando se usa el método [AsDocumentQuery()](https://msdn.microsoft.com/library/microsoft.azure.documents.linq.documentqueryable.asdocumentquery.aspx):

```csharp
// Initialize this DocumentClient and Collection
DocumentClient documentClient = null;
DocumentCollection collection = null;

// Setting PopulateQueryMetrics to true in the FeedOptions
FeedOptions feedOptions = new FeedOptions
{
    PopulateQueryMetrics = true
};

string query = "SELECT TOP 5 * FROM c";
IDocumentQuery<dynamic> documentQuery = documentClient.CreateDocumentQuery(Collection.SelfLink, query, feedOptions).AsDocumentQuery();

while (documentQuery.HasMoreResults)
{
    // Execute one continuation of the query
    FeedResponse<dynamic> feedResponse = await documentQuery.ExecuteNextAsync();
    
    // This dictionary maps the partitionId to the QueryMetrics of that query
    IReadOnlyDictionary<string, QueryMetrics> partitionIdToQueryMetrics = feedResponse.QueryMetrics;
    
    // At this point you have QueryMetrics which you can serialize using .ToString()
    foreach (KeyValuePair<string, QueryMetrics> kvp in partitionIdToQueryMetrics)
    {
        string partitionId = kvp.Key;
        QueryMetrics queryMetrics = kvp.Value;
        
        // Do whatever logging you need
        DoSomeLoggingOfQueryMetrics(query, partitionId, queryMetrics);
    }
}
```
## <a name="aggregating-querymetrics"></a>Agregación de QueryMetrics

En la sección anterior, observe que había varias llamadas al método [ExecuteNextAsync](https://msdn.microsoft.com/library/azure/dn850294.aspx). Cada llamada devolvió un objeto `FeedResponse` que tiene un diccionario de `QueryMetrics`; uno para cada continuación de la consulta. En el ejemplo siguiente se muestra cómo agregar estos objetos `QueryMetrics` mediante LINQ:

```csharp
List<QueryMetrics> queryMetricsList = new List<QueryMetrics>();

while (documentQuery.HasMoreResults)
{
    // Execute one continuation of the query
    FeedResponse<dynamic> feedResponse = await documentQuery.ExecuteNextAsync();
    
    // This dictionary maps the partitionId to the QueryMetrics of that query
    IReadOnlyDictionary<string, QueryMetrics> partitionIdToQueryMetrics = feedResponse.QueryMetrics;
    queryMetricsList.AddRange(partitionIdToQueryMetrics.Values);
}

// Aggregate the QueryMetrics using the + operator overload of the QueryMetrics class.
QueryMetrics aggregatedQueryMetrics = queryMetricsList.Aggregate((curr, acc) => curr + acc);
Console.WriteLine(aggregatedQueryMetrics);
```

## <a name="grouping-query-metrics-by-partition-id"></a>Agrupación de métricas de consulta por el identificador de la partición

Puede agrupar el objeto `QueryMetrics` por el id. de la partición. La agrupación por el id. de la partición permite ver si una partición específica está causando problemas de rendimiento en comparación con otras. En el ejemplo siguiente se muestra cómo agrupar `QueryMetrics` con LINQ:

```csharp
List<KeyValuePair<string, QueryMetrics>> partitionedQueryMetrics = new List<KeyValuePair<string, QueryMetrics>>();
while (documentQuery.HasMoreResults)
{
    // Execute one continuation of the query
    FeedResponse<dynamic> feedResponse = await documentQuery.ExecuteNextAsync();
    
    // This dictionary is maps the partitionId to the QueryMetrics of that query
    IReadOnlyDictionary<string, QueryMetrics> partitionIdToQueryMetrics = feedResponse.QueryMetrics;
    partitionedQueryMetrics.AddRange(partitionIdToQueryMetrics.ToList());
}

// Now we are able to group the query metrics by partitionId
IEnumerable<IGrouping<string, KeyValuePair<string, QueryMetrics>>> groupedByQueryMetrics = partitionedQueryMetrics
    .GroupBy(kvp => kvp.Key);

// If we wanted to we could even aggregate the groupedby QueryMetrics
foreach(IGrouping<string, KeyValuePair<string, QueryMetrics>> grouping in groupedByQueryMetrics)
{
    string partitionId = grouping.Key;
    QueryMetrics aggregatedQueryMetricsForPartition = grouping
        .Select(kvp => kvp.Value)
        .Aggregate((curr, acc) => curr + acc);
    DoSomeLoggingOfQueryMetrics(query, partitionId, aggregatedQueryMetricsForPartition);
}
```

## <a name="linq-on-documentquery"></a>LINQ en DocumentQuery

También puede obtener el objeto `FeedResponse` de una consulta LINQ usando el método `AsDocumentQuery()`:

```csharp
IDocumentQuery<Document> linqQuery = client.CreateDocumentQuery(collection.SelfLink, feedOptions)
    .Take(1)
    .Where(document => document.Id == "42")
    .OrderBy(document => document.Timestamp)
    .AsDocumentQuery();
FeedResponse<Document> feedResponse = await linqQuery.ExecuteNextAsync<Document>();
IReadOnlyDictionary<string, QueryMetrics> queryMetrics = feedResponse.QueryMetrics;
```

## <a name="expensive-queries"></a>Consultas costosas

Puede capturar las unidades de solicitud consumidas por cada consulta para investigar consultas costosas o consultas que consumen un alto rendimiento. Puede obtener el cargo de la solicitud mediante la propiedad [RequestCharge](https://msdn.microsoft.com/library/azure/dn948712.aspx) de `FeedResponse`. Para obtener más información acerca de cómo obtener el cargo de la solicitud mediante Azure Portal y los diferentes SDK, consulte el artículo sobre la [búsqueda del cargo de la unidad de solicitud](find-request-unit-charge.md).

```csharp
string query = "SELECT * FROM c";
IDocumentQuery<dynamic> documentQuery = documentClient.CreateDocumentQuery(Collection.SelfLink, query, feedOptions).AsDocumentQuery();

while (documentQuery.HasMoreResults)
{
    // Execute one continuation of the query
    FeedResponse<dynamic> feedResponse = await documentQuery.ExecuteNextAsync();
    double requestCharge = feedResponse.RequestCharge
    
    // Log the RequestCharge how ever you want.
    DoSomeLogging(requestCharge);
}
```

## <a name="get-the-query-execution-time"></a>Obtener el tiempo de ejecución de consulta

Al calcular el tiempo necesario para ejecutar una consulta del cliente, asegúrese de incluir solo el tiempo para llamar al método `ExecuteNextAsync` y no otras partes del código base. Estas llamadas le ayudarán a calcular el tiempo que tardó la ejecución de la consulta, tal como se muestra en el ejemplo siguiente:

```csharp
string query = "SELECT * FROM c";
IDocumentQuery<dynamic> documentQuery = documentClient.CreateDocumentQuery(Collection.SelfLink, query, feedOptions).AsDocumentQuery();
Stopwatch queryExecutionTimeEndToEndTotal = new Stopwatch();
while (documentQuery.HasMoreResults)
{
    // Execute one continuation of the query
    queryExecutionTimeEndToEndTotal.Start();
    FeedResponse<dynamic> feedResponse = await documentQuery.ExecuteNextAsync();
    queryExecutionTimeEndToEndTotal.Stop();
}

// Log the elapsed time
DoSomeLogging(queryExecutionTimeEndToEndTotal.Elapsed);
```

## <a name="scan-queries-commonly-slow-and-expensive"></a>Consultas de examen (normalmente lento y costoso)

Una consulta de examen hace referencia a una consulta que no sirve el índice, motivo por el que muchos documentos se cargan antes de devolver el conjunto de resultados.

Aquí tiene un ejemplo de una consulta de examen:

```sql
SELECT VALUE c.description 
FROM   c 
WHERE UPPER(c.description) = "BABYFOOD, DESSERT, FRUIT DESSERT, WITHOUT ASCORBIC ACID, JUNIOR"
```

El filtro de esta consulta usa la función del sistema UPPER, que no se sirve desde el índice. La ejecución de esta consulta en una colección grande genera las siguientes métricas de consulta para la primera continuación:

```
QueryMetrics

Retrieved Document Count                 :          60,951
Retrieved Document Size                  :     399,998,938 bytes
Output Document Count                    :               7
Output Document Size                     :             510 bytes
Index Utilization                        :            0.00 %
Total Query Execution Time               :        4,500.34 milliseconds
  Query Preparation Times
    Query Compilation Time               :            0.09 milliseconds
    Logical Plan Build Time              :            0.05 milliseconds
    Physical Plan Build Time             :            0.04 milliseconds
    Query Optimization Time              :            0.01 milliseconds
  Index Lookup Time                      :            0.01 milliseconds
  Document Load Time                     :        4,177.66 milliseconds
  Runtime Execution Times
    Query Engine Times                   :          322.16 milliseconds
    System Function Execution Time       :           85.74 milliseconds
    User-defined Function Execution Time :            0.00 milliseconds
  Document Write Time                    :            0.01 milliseconds
Client Side Metrics
  Retry Count                            :               0
  Request Charge                         :        4,059.95 RUs
```

Observe los siguientes valores de la salida de las métricas de consulta:

```
Retrieved Document Count                 :          60,951
Retrieved Document Size                  :     399,998,938 bytes
```

Esta consulta cargó 60 951 documentos, que sumaban un total de 399 998 938 bytes. La carga de este número de bytes da lugar a un alto costo o cargo de unidad de solicitud. También se tarda mucho tiempo en ejecutar la consulta, lo que se ve claramente con la propiedad de tiempo total transcurrido:

```
Total Query Execution Time               :        4,500.34 milliseconds
```

Esto significa que la consulta tardó 4,5 segundos en ejecutarse (y solo era una continuación).

Para optimizar esta consulta de ejemplo, evite el uso de UPPER en el filtro. En su lugar, cuando se crean o actualizan los documentos, los valores `c.description` se deben insertar completamente en mayúsculas. La consulta se convierte en lo siguiente: 

```sql
SELECT VALUE c.description 
FROM   c 
WHERE c.description = "BABYFOOD, DESSERT, FRUIT DESSERT, WITHOUT ASCORBIC ACID, JUNIOR"
```

Ahora esta consulta puede servirse desde el índice.

Para más información acerca de cómo ajustar el rendimiento de las consultas, consulte el artículo [Ajuste del rendimiento de las consultas](https://docs.microsoft.com/azure/cosmos-db/documentdb-sql-query-metrics).

## <a name="references"></a><a id="References"></a>Referencias

- [Especificación de SQL de Azure Cosmos DB](https://go.microsoft.com/fwlink/p/?LinkID=510612)
- [SQL ANSI 2011](https://www.iso.org/iso/iso_catalogue/catalogue_tc/catalogue_detail.htm?csnumber=53681)
- [JSON](https://json.org/)
- [LINQ](/previous-versions/dotnet/articles/bb308959(v=msdn.10)) 

## <a name="next-steps"></a>Pasos siguientes

- [Ajuste del rendimiento de las consultas](sql-api-query-metrics.md)
- [Introducción a la indexación](index-overview.md)
- [Ejemplos de .NET de Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
