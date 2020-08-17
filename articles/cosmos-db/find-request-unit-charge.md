---
title: Búsqueda del cargo de unidad de solicitud (RU) en Azure Cosmos DB
description: Sepa cómo encontrar el cargo de unidad de solicitud (RU) de cualquier operación que se ejecuta en un contenedor de Azure Cosmos.
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: how-to
ms.date: 09/01/2019
ms.author: thweiss
ms.custom: devx-track-javascript
ms.openlocfilehash: d692a884688d81fdf16261edff816fa8f0215877
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422324"
---
# <a name="find-the-request-unit-charge-in-azure-cosmos-db"></a>Búsqueda del cargo de unidad de solicitud en Azure Cosmos DB

En este artículo se presentan las distintas formas de encontrar el consumo de [unidades de solicitud](request-units.md) (RU) de cualquier operación que se ejecuta en un contenedor de Azure Cosmos DB. Actualmente, puede medir este consumo solo con Azure Portal o por medio de la inspección de la respuesta enviada desde Azure Cosmos DB mediante uno de los SDK.

## <a name="sql-core-api"></a>SQL API (Core)

Si usa la API de SQL, tiene varias opciones para buscar el consumo de RU para una operación en un contenedor de Azure Cosmos.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Actualmente, puede buscar el cargo de solicitud solo en Azure Portal para una consulta SQL.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. [Cree una cuenta de Azure Cosmos](create-sql-api-dotnet.md#create-account) y suminístrele datos, o seleccione una cuenta existente de Azure Cosmos que ya contenga datos.

1. Vaya al panel **Data Explorer** y seleccione el contenedor en el que quiere trabajar.

1. Seleccione **Nueva consulta SQL**.

1. Escriba una consulta válida y, luego, seleccione **Ejecutar consulta**.

1. Seleccione **Query Stats** (Estadísticas de consulta) para mostrar el cargo de solicitud real correspondiente a la solicitud que ha ejecutado.

:::image type="content" source="./media/find-request-unit-charge/portal-sql-query.png" alt-text="Captura de pantalla del cargo de solicitud de una consulta SQL en Azure Portal":::

### <a name="use-the-net-sdk"></a>Uso del SDK de .NET

# <a name="net-sdk-v2"></a>[SDK de .NET V2](#tab/dotnetv2)

Los objetos devueltos desde el [SDK de .NET v2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/) exponen una propiedad `RequestCharge`:

```csharp
ResourceResponse<Document> fetchDocumentResponse = await client.ReadDocumentAsync(
    UriFactory.CreateDocumentUri("database", "container", "itemId"),
    new RequestOptions
    {
        PartitionKey = new PartitionKey("partitionKey")
    });
var requestCharge = fetchDocumentResponse.RequestCharge;

StoredProcedureResponse<string> storedProcedureCallResponse = await client.ExecuteStoredProcedureAsync<string>(
    UriFactory.CreateStoredProcedureUri("database", "container", "storedProcedureId"),
    new RequestOptions
    {
        PartitionKey = new PartitionKey("partitionKey")
    });
requestCharge = storedProcedureCallResponse.RequestCharge;

IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri("database", "container"),
    "SELECT * FROM c",
    new FeedOptions
    {
        PartitionKey = new PartitionKey("partitionKey")
    }).AsDocumentQuery();
while (query.HasMoreResults)
{
    FeedResponse<dynamic> queryResponse = await query.ExecuteNextAsync<dynamic>();
    requestCharge = queryResponse.RequestCharge;
}
```

# <a name="net-sdk-v3"></a>[SDK de .NET V3](#tab/dotnetv3)

Los objetos que se devuelven desde el [SDK de .NET v3](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/) exponen una propiedad `RequestCharge`:

[!code-csharp[](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos/tests/Microsoft.Azure.Cosmos.Tests/SampleCodeForDocs/CustomDocsSampleCode.cs?name=GetRequestCharge)]

Para más información, consulte [Inicio rápido: Cree una aplicación web .NET con una cuenta de SQL API en Azure Cosmos DB](create-sql-api-dotnet.md).

---

### <a name="use-the-java-sdk"></a>Uso del SDK de Java

Los objetos devueltos desde el [SDK de Java](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) exponen un método `getRequestCharge()`:

```java
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPartitionKey(new PartitionKey("partitionKey"));

Observable<ResourceResponse<Document>> readDocumentResponse = client.readDocument(String.format("/dbs/%s/colls/%s/docs/%s", "database", "container", "itemId"), requestOptions);
readDocumentResponse.subscribe(result -> {
    double requestCharge = result.getRequestCharge();
});

Observable<StoredProcedureResponse> storedProcedureResponse = client.executeStoredProcedure(String.format("/dbs/%s/colls/%s/sprocs/%s", "database", "container", "storedProcedureId"), requestOptions, null);
storedProcedureResponse.subscribe(result -> {
    double requestCharge = result.getRequestCharge();
});

FeedOptions feedOptions = new FeedOptions();
feedOptions.setPartitionKey(new PartitionKey("partitionKey"));

Observable<FeedResponse<Document>> feedResponse = client
    .queryDocuments(String.format("/dbs/%s/colls/%s", "database", "container"), "SELECT * FROM c", feedOptions);
feedResponse.forEach(result -> {
    double requestCharge = result.getRequestCharge();
});
```

Para más información, consulte [Inicio rápido: Compilación de una aplicación Java mediante una cuenta de SQL API de Azure Cosmos DB](create-sql-api-java.md).

### <a name="use-the-nodejs-sdk"></a>Uso del SDK de Node.js

Los objetos devueltos por el [SDK de Node.js](https://www.npmjs.com/package/@azure/cosmos) exponen un subobjeto `headers` que asigna todos los encabezados devueltos por la API HTTP subyacente. El cargo de solicitud está disponible en la clave `x-ms-request-charge`:

```javascript
const item = await client
    .database('database')
    .container('container')
    .item('itemId', 'partitionKey')
    .read();
var requestCharge = item.headers['x-ms-request-charge'];

const storedProcedureResult = await client
    .database('database')
    .container('container')
    .storedProcedure('storedProcedureId')
    .execute({
        partitionKey: 'partitionKey'
    });
requestCharge = storedProcedureResult.headers['x-ms-request-charge'];

const query = client.database('database')
    .container('container')
    .items
    .query('SELECT * FROM c', {
        partitionKey: 'partitionKey'
    });
while (query.hasMoreResults()) {
    var result = await query.executeNext();
    requestCharge = result.headers['x-ms-request-charge'];
}
```

Para más información, consulte [Inicio rápido: Compilación de una aplicación Node.js con la cuenta de Azure Cosmos DB SQL API](create-sql-api-nodejs.md). 

### <a name="use-the-python-sdk"></a>Uso del SDK de Python

El objeto `CosmosClient` del [SDK de Python](https://pypi.org/project/azure-cosmos/) expone un diccionario `last_response_headers` que asigna todos los encabezados devueltos por la API HTTP subyacente de la última operación ejecutada. El cargo de solicitud está disponible en la clave `x-ms-request-charge`:

```python
response = client.ReadItem(
    'dbs/database/colls/container/docs/itemId', {'partitionKey': 'partitionKey'})
request_charge = client.last_response_headers['x-ms-request-charge']

response = client.ExecuteStoredProcedure(
    'dbs/database/colls/container/sprocs/storedProcedureId', None, {'partitionKey': 'partitionKey'})
request_charge = client.last_response_headers['x-ms-request-charge']
```

Para más información, consulte [Inicio rápido: Compilación de una aplicación Python con la cuenta de Azure Cosmos DB SQL API](create-sql-api-python.md). 

## <a name="azure-cosmos-db-api-for-mongodb"></a>Azure Cosmos DB API para MongoDB

El cargo de RU se expone mediante un [comando de base de datos](https://docs.mongodb.com/manual/reference/command/) personalizado llamado `getLastRequestStatistics`. El comando devuelve un documento que contiene el nombre de la última operación ejecutada, el cargo de la solicitud y su duración. Si usa la API de Azure Cosmos DB para MongoDB, tiene varias opciones para recuperar el cargo de RU.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Actualmente, puede buscar el cargo de solicitud solo en Azure Portal para una consulta.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. [Cree una cuenta de Azure Cosmos](create-mongodb-dotnet.md#create-a-database-account) y suminístrele datos, o seleccione una cuenta existente que ya contenga datos.

1. Vaya al panel **Data Explorer** y seleccione el contenedor en el que quiere trabajar.

1. Seleccione **Nueva consulta**.

1. Escriba una consulta válida y, luego, seleccione **Ejecutar consulta**.

1. Seleccione **Query Stats** (Estadísticas de consulta) para mostrar el cargo de solicitud real correspondiente a la solicitud que ha ejecutado.

:::image type="content" source="./media/find-request-unit-charge/portal-mongodb-query.png" alt-text="Captura de pantalla del cargo de solicitud de una consulta de MongoDB en Azure Portal":::

### <a name="use-the-mongodb-net-driver"></a>Uso del controlador de .NET de MongoDB

Al usar el [controlador oficial de .NET para MongoDB](https://docs.mongodb.com/ecosystem/drivers/csharp/), puede ejecutar comandos mediante una llamada al método `RunCommand` en un objeto `IMongoDatabase`. Este método requiere una implementación de la clase abstracta `Command<>`:

```csharp
class GetLastRequestStatisticsCommand : Command<Dictionary<string, object>>
{
    public override RenderedCommand<Dictionary<string, object>> Render(IBsonSerializerRegistry serializerRegistry)
    {
        return new RenderedCommand<Dictionary<string, object>>(new BsonDocument("getLastRequestStatistics", 1), serializerRegistry.GetSerializer<Dictionary<string, object>>());
    }
}

Dictionary<string, object> stats = database.RunCommand(new GetLastRequestStatisticsCommand());
double requestCharge = (double)stats["RequestCharge"];
```

Para más información, consulte [Inicio rápido: Compilación de una aplicación web .NET mediante la API de Azure Cosmos DB para MongoDB](create-mongodb-dotnet.md).

### <a name="use-the-mongodb-java-driver"></a>Uso del controlador de .NET de Java


Al usar el [controlador oficial de Java para MongoDB](https://mongodb.github.io/mongo-java-driver/), puede ejecutar comandos mediante una llamada al método `runCommand` en un objeto `MongoDatabase`:

```java
Document stats = database.runCommand(new Document("getLastRequestStatistics", 1));
Double requestCharge = stats.getDouble("RequestCharge");
```

Para más información, consulte [Inicio rápido: Compilación de una aplicación web mediante la API de Azure Cosmos DB para MongoDB y el SDK de Java](create-mongodb-java.md).

### <a name="use-the-mongodb-nodejs-driver"></a>Uso del controlador de Node.js de MongoDB

Al usar el [controlador oficial de Node.js para MongoDB](https://mongodb.github.io/node-mongodb-native/), puede ejecutar comandos mediante una llamada al método `command` en un objeto `db`:

```javascript
db.command({ getLastRequestStatistics: 1 }, function(err, result) {
    assert.equal(err, null);
    const requestCharge = result['RequestCharge'];
});
```

Para más información, consulte [Inicio rápido: Migración de una aplicación web actual de Node.js en MongoDB a Azure Cosmos DB](create-mongodb-nodejs.md).

## <a name="cassandra-api"></a>Cassandra API

Al realizar operaciones en Cassandra API de Azure Cosmos DB, el cargo de RU se devuelve en la carga de entrada como un campo llamado `RequestCharge`. Tiene varias opciones para recuperar el cargo de RU.

### <a name="use-the-net-sdk"></a>Uso del SDK de .NET

Al usar el [SDK de .NET](https://www.nuget.org/packages/CassandraCSharpDriver/), puede recuperar la carga de entrada en la propiedad `Info` de un objeto `RowSet`:

```csharp
RowSet rowSet = session.Execute("SELECT table_name FROM system_schema.tables;");
double requestCharge = BitConverter.ToDouble(rowSet.Info.IncomingPayload["RequestCharge"].Reverse().ToArray(), 0);
```

Para más información, consulte [Inicio rápido: Creación de una aplicación de Cassandra con el SDK de .NET y Azure Cosmos DB](create-cassandra-dotnet.md).

### <a name="use-the-java-sdk"></a>Uso del SDK de Java

Al usar el [SDK de Java](https://mvnrepository.com/artifact/com.datastax.cassandra/cassandra-driver-core), puede recuperar la carga de entrada mediante una llamada al método `getExecutionInfo()` en un objeto `ResultSet`:

```java
ResultSet resultSet = session.execute("SELECT table_name FROM system_schema.tables;");
Double requestCharge = resultSet.getExecutionInfo().getIncomingPayload().get("RequestCharge").getDouble();
```

Para más información, consulte [Inicio rápido: Creación de una aplicación de Cassandra con el SDK de Java y Azure Cosmos DB](create-cassandra-java.md).

## <a name="gremlin-api"></a>API de Gremlin

Si usa la API de Gremlin, tiene varias opciones para buscar el consumo de RU para una operación en un contenedor de Azure Cosmos. 

### <a name="use-drivers-and-sdk"></a>Uso de controladores y SDK

A los encabezados devueltos por la API de Gremlin se les asignan atributos de estado personalizados, que actualmente se exponen mediante el SDK de Java y .NET de Gremlin. El cargo de solicitud está disponible en la clave `x-ms-request-charge`.

### <a name="use-the-net-sdk"></a>Uso del SDK de .NET

Si usa el [SDK de .NET para Gremlin](https://www.nuget.org/packages/Gremlin.Net/), los atributos de estado están disponibles en la propiedad `StatusAttributes` del objeto `ResultSet<>`:

```csharp
ResultSet<dynamic> results = client.SubmitAsync<dynamic>("g.V().count()").Result;
double requestCharge = (double)results.StatusAttributes["x-ms-request-charge"];
```

Para más información, consulte [Inicio rápido: Compilación de una aplicación .NET Framework o Core mediante una cuenta de Gremlin API de Azure Cosmos DB](create-graph-dotnet.md).

### <a name="use-the-java-sdk"></a>Uso del SDK de Java

Al usar el [SDK de Java para Gremlin](https://mvnrepository.com/artifact/org.apache.tinkerpop/gremlin-driver), puede recuperar los atributos de estado mediante una llamada al método `statusAttributes()` en el objeto `ResultSet`:

```java
ResultSet results = client.submit("g.V().count()");
Double requestCharge = (Double)results.statusAttributes().get().get("x-ms-request-charge");
```

Para más información, consulte [Inicio rápido: Creación de una base de datos de grafos en Azure Cosmos DB mediante el SDK de Java](create-graph-java.md).

## <a name="table-api"></a>Table API

Actualmente, el único SDK que devuelve el cargo de RU para las operaciones de tabla es el [SDK de .NET Standard](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table). El objeto `TableResult` expone una propiedad `RequestCharge` que se rellena mediante el SDK cuando se usa con Table API de Azure Cosmos DB:

```csharp
CloudTable tableReference = client.GetTableReference("table");
TableResult tableResult = tableReference.Execute(TableOperation.Insert(new DynamicTableEntity("partitionKey", "rowKey")));
if (tableResult.RequestCharge.HasValue) // would be false when using Azure Storage Tables
{
    double requestCharge = tableResult.RequestCharge.Value;
}
```

Para más información, consulte [Inicio rápido: creación de una aplicación de Table API con el SDK de .NET y Azure Cosmos DB](create-table-dotnet.md).

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo optimizar el consumo de RU, vea estos artículos:

* [Rendimiento y unidades de solicitud en Azure Cosmos DB](request-units.md)
* [Optimización del costo de rendimiento aprovisionado en Azure Cosmos DB](optimize-cost-throughput.md)
* [Optimización de los costos de consulta de Azure Cosmos DB](optimize-cost-queries.md)
* [Escalado global del rendimiento aprovisionado](scaling-throughput.md)
* [Aprovisionamiento del rendimiento en contenedores y bases de datos](set-throughput.md)
* [Aprovisionamiento del rendimiento en un contenedor de Azure Cosmos](how-to-provision-container-throughput.md)
* [Supervisión y depuración con métricas de Azure Cosmos DB](use-metrics.md)
