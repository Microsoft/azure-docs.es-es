---
title: Uso de la biblioteca Bulk Executor en .NET de grafos con la API de Gremlin de Azure Cosmos DB
description: Aprenda a usar la biblioteca Bulk Executor para importar masivamente datos de grafos en un contenedor de Gremlin API de Azure Cosmos DB.
author: luisbosquez
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 05/28/2019
ms.author: lbosq
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 6fd81a844832fbe6ad7410ec786baa431ca9930c
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89004109"
---
# <a name="using-the-graph-bulk-executor-net-library-to-perform-bulk-operations-in-azure-cosmos-db-gremlin-api"></a>Uso de la biblioteca Bulk Executor en .NET de grafos para realizar operaciones en masa en Gremlin API de Azure Cosmos DB

En este tutorial se proporcionan instrucciones sobre el uso de la biblioteca Bulk Executor en .NET de Azure Cosmos DB para importar y actualizar objetos de grafos en un contenedor de Gremlin API de Azure Cosmos DB. Este proceso hace uso de la clase Graph de la [biblioteca Bulk Executor](https://docs.microsoft.com/azure/cosmos-db/bulk-executor-overview) para crear objetos de vértices y bordes mediante programación y, a continuación, insertar varios de ellos por solicitud de red. Este comportamiento es configurable mediante la biblioteca Bulk Executor, para hacer un uso óptimo de los recursos de base de datos y memoria local.

De forma contraria al envío de consultas de Gremlin a una base de datos, en las que los comandos se evalúan y después se ejecutan uno cada vez, el uso de la biblioteca Bulk Executor requiere crear y validar los objetos de forma local. Después de crear los objetos, la biblioteca permite enviar objetos de grafos al servicio de base de datos de forma secuencial. Con este método, la velocidad de ingesta de datos se puede aumentar hasta 100 veces, lo que lo convierte en un método ideal para las migraciones de datos iniciales o las operaciones periódicas de movimiento de datos. Para más información, visite la página de GitHub de la [aplicación de ejemplo de Bulk Executor de Azure Cosmos DB Graph](https://github.com/Azure-Samples/azure-cosmosdb-graph-bulkexecutor-dotnet-getting-started).

## <a name="bulk-operations-with-graph-data"></a>Operaciones masivas con datos de grafos

La [biblioteca Bulk Executor](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.graph?view=azure-dotnet) contiene un espacio de nombres `Microsoft.Azure.CosmosDB.BulkExecutor.Graph` que proporciona funcionalidad para crear e importar objetos de grafos. 

El siguiente proceso describe cómo se puede usar la migración de datos para un contenedor de la API de Gremlin:
1. Recupere los registros del origen de datos.
2. Construya objetos `GremlinVertex` y `GremlinEdge` a partir de los registros obtenidos y agréguelos a una estructura de datos `IEnumerable`. En esta parte de la aplicación se debería implementar la lógica para detectar y agregar relaciones, en caso de que el origen de datos no sea una base de datos de grafos.
3. Use el [método BulkImportAsync de Graph](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.graph.graphbulkexecutor.bulkimportasync?view=azure-dotnet) para insertar los objetos de grafos en la colección.

Este mecanismo mejorará la eficacia de la migración de datos en comparación con un cliente de Gremlin. Esta mejora se produce porque la inserción de datos con Gremlin requiere que la aplicación envíe una consulta cada vez, que deberá ser validada, evaluada y, finalmente, ejecutada para crear los datos. La biblioteca Bulk Executor controla la validación en la aplicación y envía varios objetos de grafos a la vez por cada solicitud de red.

### <a name="creating-vertices-and-edges"></a>Creación de vértices y bordes

`GraphBulkExecutor` proporciona el método `BulkImportAsync`, que requiere una lista `IEnumerable` de objetos `GremlinVertex` o `GremlinEdge`, ambos definidos en el espacio de nombres `Microsoft.Azure.CosmosDB.BulkExecutor.Graph.Element`. En el ejemplo se separan los bordes y los vértices en dos tareas de importación de BulkExecutor. Observe el ejemplo siguiente:

```csharp

IBulkExecutor graphbulkExecutor = new GraphBulkExecutor(documentClient, targetCollection);

BulkImportResponse vResponse = null;
BulkImportResponse eResponse = null;

try
{
    // Import a list of GremlinVertex objects
    vResponse = await graphbulkExecutor.BulkImportAsync(
            Utils.GenerateVertices(numberOfDocumentsToGenerate),
            enableUpsert: true,
            disableAutomaticIdGeneration: true,
            maxConcurrencyPerPartitionKeyRange: null,
            maxInMemorySortingBatchSize: null,
            cancellationToken: token);

    // Import a list of GremlinEdge objects
    eResponse = await graphbulkExecutor.BulkImportAsync(
            Utils.GenerateEdges(numberOfDocumentsToGenerate),
            enableUpsert: true,
            disableAutomaticIdGeneration: true,
            maxConcurrencyPerPartitionKeyRange: null,
            maxInMemorySortingBatchSize: null,
            cancellationToken: token);
}
catch (DocumentClientException de)
{
    Trace.TraceError("Document client exception: {0}", de);
}
catch (Exception e)
{
    Trace.TraceError("Exception: {0}", e);
}
```

Para más información sobre los parámetros de la biblioteca Bulk Executor, consulte el tema [Importación de datos en masa a Azure Cosmos DB](bulk-executor-dot-net.md#bulk-import-data-to-an-azure-cosmos-account).

La carga se debe realizar en instancias de objetos `GremlinVertex` y `GremlinEdge`. Le mostramos cómo se pueden crear estos objetos:

**Vértices**:
```csharp
// Creating a vertex
GremlinVertex v = new GremlinVertex(
    "vertexId",
    "vertexLabel");

// Adding custom properties to the vertex
v.AddProperty("customProperty", "value");

// Partitioning keys must be specified for all vertices
v.AddProperty("partitioningKey", "value");
```

**Bordes**:
```csharp
// Creating an edge
GremlinEdge e = new GremlinEdge(
    "edgeId",
    "edgeLabel",
    "targetVertexId",
    "sourceVertexId",
    "targetVertexLabel",
    "sourceVertexLabel",
    "targetVertexPartitioningKey",
    "sourceVertexPartitioningKey");

// Adding custom properties to the edge
e.AddProperty("customProperty", "value");
```

> [!NOTE]
> La utilidad Bulk Executor no comprueba automáticamente los vértices existentes antes de agregar bordes. Esto se debe validar en la aplicación antes de ejecutar las tareas de BulkImport.

## <a name="sample-application"></a>Aplicación de ejemplo

### <a name="prerequisites"></a>Requisitos previos
* Visual Studio 2019 con la carga de trabajo de desarrollo de Azure. Puede empezar a trabajar con [Visual Studio 2019 Community Edition](https://visualstudio.microsoft.com/downloads/) de forma gratuita.
* Suscripción a Azure. Puede crear [una cuenta de Azure gratuita aquí](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=cosmos-db). Como alternativa, puede crear una cuenta de base de datos de Cosmos en [Pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin una suscripción de Azure.
* Una base de datos de la API de Gremlin de Azure Cosmos DB con una **colección ilimitada**. Esta guía muestra cómo empezar a trabajar con la [API de Gremlin de Azure Cosmos DB en .NET](https://docs.microsoft.com/azure/cosmos-db/create-graph-dotnet).
* Git. Para más información consulte la [página de descargas de Git](https://git-scm.com/downloads).

### <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo
En este tutorial, seguiremos los pasos para comenzar a trabajar mediante el uso del [ejemplo de Bulk Executor de Azure Cosmos DB Graph](https://github.com/Azure-Samples/azure-cosmosdb-graph-bulkexecutor-dotnet-getting-started) hospedado en GitHub. Esta aplicación consta de una solución de .NET que genera aleatoriamente objetos vértice y borde y, a continuación, ejecuta las inserciones masivas en la cuenta de base de datos de grafos especificada. Para obtener la aplicación, ejecute el siguiente comando `git clone`:

```bash
git clone https://github.com/Azure-Samples/azure-cosmosdb-graph-bulkexecutor-dotnet-getting-started.git
```

Este repositorio contiene el ejemplo de GraphBulkExecutor con los siguientes archivos:

Archivo|Descripción
---|---
`App.config`|Aquí se especifican los parámetros específicos de la aplicación y de la base de datos. Este archivo debe modificarse, en primer lugar, para que se conecte a la base de datos y las colecciones de destino.
`Program.cs`| Este archivo contiene la lógica de la creación de la colección `DocumentClient`, el control de las limpiezas y el envío de las solicitudes de Bulk Executor.
`Util.cs`| Este archivo contiene una clase auxiliar que contiene la lógica para la generación de datos de prueba y la comprobación de la existencia de la base de datos y las colecciones.

En el archivo `App.config` se pueden proporcionar los siguientes valores de configuración:

Configuración|Descripción
---|---
`EndPointUrl`|**Punto de conexión del SDK de .NET**, se puede encontrar en la hoja Información general de la cuenta de base de datos de la API de Gremlin de Azure Cosmos DB. Tiene el formato `https://your-graph-database-account.documents.azure.com:443/`
`AuthorizationKey`|Clave principal o secundaria, aparece bajo la cuenta de Azure Cosmos DB. Más información en [Protección del acceso a los datos de Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/secure-access-to-data#master-keys)
`DatabaseName`, `CollectionName`|Son los **nombres de la base de datos y la colección de destino**. Cuando `ShouldCleanupOnStart` se establece en `true`, estos valores, junto con `CollectionThroughput`, se usarán para su eliminación y para crear una nueva base de datos y una colección. De forma similar, si `ShouldCleanupOnFinish` se establece en `true`, se usarán para eliminar la base de datos tan pronto como la ingesta finalice. Tenga en cuenta que la colección de destino debe ser **una colección ilimitada**.
`CollectionThroughput`|Se utiliza para crear una nueva colección cuando la opción `ShouldCleanupOnStart` se establece en `true`.
`ShouldCleanupOnStart`|Elimina la cuenta de base de datos y las colecciones antes de ejecutar el programa y, a continuación, crea otras nuevas con los valores de `DatabaseName`, `CollectionName` y `CollectionThroughput`.
`ShouldCleanupOnFinish`|Elimina la cuenta de base de datos y las colecciones con los valores especificados de `DatabaseName` y `CollectionName` después de la ejecución del programa.
`NumberOfDocumentsToImport`|Determina el número de vértices y bordes de prueba que se generan en el ejemplo. Este número se aplica tanto a vértices como bordes.
`NumberOfBatches`|Determina el número de vértices y bordes de prueba que se generan en el ejemplo. Este número se aplica tanto a vértices como bordes.
`CollectionPartitionKey`|Se usa para crear los vértices y bordes de prueba, donde esta propiedad será asignada automáticamente. También se usa al volver a crear la base de datos y las colecciones si la opción `ShouldCleanupOnStart` se establece en `true`.

### <a name="run-the-sample-application"></a>Ejecutar la aplicación de ejemplo

1. Agregue los parámetros de configuración específicos de la base de datos en `App.config`. Se usa para crear una instancia de DocumentClient. Si aún no se han creado la base de datos y el contenedor, se crearán automáticamente.
2. Ejecute la aplicación. Llama dos veces a `BulkImportAsync`, una para importar los vértices y otra para importar los bordes. Si cualquiera de los objetos genera un error en la inserción, se agregará a los archivos `.\BadVertices.txt` o `.\BadEdges.txt`.
3. Consulte la base de datos de grafos para evaluar los resultados. Si la opción `ShouldCleanupOnFinish` se establece en true, la base de datos se eliminará automáticamente.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre el paquete NuGet y las notas de la versión de la biblioteca Bulk Executor para .NET, vea los [detalles del SDK de Bulk Executor](sql-api-sdk-bulk-executor-dot-net.md). 
* Consulte los [Consejos de rendimiento](https://docs.microsoft.com/azure/cosmos-db/bulk-executor-dot-net#performance-tips) para optimizar aún más el uso de Bulk Executor.
* Revise el artículo [Referencia de BulkExecutor.Graph](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.graph?view=azure-dotnet) para más detalles sobre las clases y métodos definidos en este espacio de nombres.