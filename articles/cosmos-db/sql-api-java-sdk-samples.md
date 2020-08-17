---
title: 'API de SQL de Azure Cosmos DB: Ejemplos de SDK para Java v4'
description: Busque ejemplos de Java en GitHub para tareas comunes mediante la API de SQL de Azure Cosmos DB, incluidas las operaciones CRUD.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 05/20/2020
ms.custom: devx-track-java
ms.author: anfeldma
ms.openlocfilehash: 4f61dcfa37547fe46a582a4a7ebb28ac68fe6c74
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87308685"
---
# <a name="azure-cosmos-db-sql-api-java-sdk-v4-examples"></a>API de SQL de Azure Cosmos DB: Ejemplos de SDK para Java v4

> [!div class="op_single_selector"]
> * [Ejemplos del SDK de .NET V2](sql-api-dotnet-samples.md)
> * [Ejemplos del SDK de .NET V3](sql-api-dotnet-v3sdk-samples.md)
> * [Ejemplos del SDK de Java V4](sql-api-java-sdk-samples.md)
> * [Ejemplos de Node.js](sql-api-nodejs-samples.md)
> * [Ejemplos de Python](sql-api-python-samples.md)
> * [Galería de ejemplos de código de Azure](https://azure.microsoft.com/resources/samples/?sort=0&service=cosmos-db)
> 
> 

> [!IMPORTANT]  
> Para más información sobre el SDK para Java v4, consulte [Notas de la versión](sql-api-sdk-java-v4.md) del SDK de Azure Cosmos DB para Java v4, el [repositorio de Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos), las [sugerencias de rendimiento](performance-tips-java-sdk-v4-sql.md) del SDK de Azure Cosmos DB para Java v4 y la [guía de solución de problemas](troubleshoot-java-sdk-v4-sql.md) del SDK de Azure Cosmos DB para Java v4. Si en la actualidad usa una versión anterior a v4, vea la guía [Migración al SDK de Azure Cosmos DB para Java v4](migrate-java-v4-sdk.md) a fin de obtener ayuda para actualizar a v4.
>

> [!IMPORTANT]  
>[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
>  
>- Puede [activar los beneficios de suscripción a Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio): su suscripción a Visual Studio le proporciona créditos todos los meses que puede usar para servicios de Azure de pago.
>
>[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]
>

En el repositorio de GitHub [azure-cosmos-java-sql-api-samples](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples) se incluyen las aplicaciones de ejemplo más recientes que realizan operaciones CRUD y otras operaciones comunes en recursos de Azure Cosmos DB. Este artículo ofrece:

* Vínculos para las tareas de cada uno de los archivos de proyecto de ejemplo de Java. 
* Vínculos al contenido de referencia de la API relacionada.

**Requisitos previos**

Necesita lo siguiente para poder ejecutar esta aplicación de ejemplo:

* Java Development Kit 8
* SDK de Azure Cosmos DB para Java v4

También puede usar Maven para obtener los archivos binarios más recientes del SDK de Microsoft Azure Cosmos DB para Java v4 para usarlos en el proyecto. Maven se encarga de agregar automáticamente las dependencias necesarias. En caso contrario, puede descargar directamente las dependencias que se enumeran en el archivo pom.xml y agregarlas a la ruta de acceso de la compilación.

```bash
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-cosmos</artifactId>
    <version>LATEST</version>
</dependency>
```

**Ejecutar las aplicaciones de ejemplo**

Clone el repositorio de ejemplo:
```bash
$ git clone https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples.git

$ cd azure-cosmos-java-sql-api-samples
```

Puede ejecutar los ejemplos mediante un IDE (Eclipse, IntelliJ o VSCODE) o bien desde la línea de comandos mediante Maven.

Estas variables de entorno se deben establecer

```
ACCOUNT_HOST=your account hostname;ACCOUNT_KEY=your account master key
```

con el fin de dar a los ejemplos acceso de lectura y escritura a su cuenta.

Para ejecutar un ejemplo, especifique su clase principal. 

```
com.azure.cosmos.examples.sample.synchronicity.MainClass
```

donde *sample. Synchronicity.MainClass* puede ser
* crudquickstart.sync.SampleCRUDQuickstart
* crudquickstart.async.SampleCRUDQuickstartAsync
* indexmanagement.sync.SampleIndexManagement
* indexmanagement.async.SampleIndexManagementAsync
* storedprocedure.sync.SampleStoredProcedure
* storedprocedure.async.SampleStoredProcedureAsync
* changefeed.SampleChangeFeedProcessor *(Changefeed solo tiene un ejemplo asincrónico, ningún ejemplo sincrónico)* , etc.

> [!NOTE]
> Cada ejemplo es independiente, es decir, se configura a sí mismo y posteriormente se limpia solo. Los ejemplos emiten varias llamadas para crear un `CosmosContainer`. Cada vez que esto ocurre, se cobra a la suscripción una hora de uso de acuerdo con el nivel de rendimiento de la colección que se va a crear. 
> 
> 

## <a name="database-examples"></a>Ejemplos de base de datos
El archivo de [ejemplos de CRUD de base de datos](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/databasecrud/sync/DatabaseCRUDQuickstart.java) muestra cómo realizar las siguientes tareas: Para información sobre las bases de datos de Azure Cosmos antes de ejecutar los siguientes ejemplos, consulte el artículo conceptual [Uso de bases de datos, contenedores y elementos](databases-containers-items.md). 

| Tarea | Referencia de API |
| --- | --- |
| [Creación de una base de datos](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/databasecrud/sync/DatabaseCRUDQuickstart.java#L77-L85) | CosmosClient.createDatabaseIfNotExists |
| [Lectura de una base de datos por identificador](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/databasecrud/sync/DatabaseCRUDQuickstart.java#L88-L95) | CosmosClient.getDatabase |
| [Lectura de todas las bases de datos](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/databasecrud/sync/DatabaseCRUDQuickstart.java#L98-L112) | CosmosClient.readAllDatabases |
| [Eliminación de una base de datos](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/databasecrud/sync/DatabaseCRUDQuickstart.java#L115-L123) | CosmosDatabase.delete |

## <a name="collection-examples"></a>Ejemplos de colección
El archivo de [ejemplos de CRUD de colección](https://github.com/Azure/azure-documentdb-java/blob/master/documentdb-examples/src/test/java/com/microsoft/azure/documentdb/examples/CollectionCrudSamples.java) muestra cómo realizar las siguientes tareas. Para información sobre las colecciones de Azure Cosmos antes de ejecutar los siguientes ejemplos, consulte el artículo conceptual [Uso de bases de datos, contenedores y elementos](databases-containers-items.md).

| Tarea | Referencia de API |
| --- | --- |
| [Creación de una colección](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/containercrud/sync/ContainerCRUDQuickstart.java#L97-L112) | CosmosDatabase.createContainerIfNotExists |
| [Cambio de rendimiento configurado de una colección](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/containercrud/sync/ContainerCRUDQuickstart.java#L115-L123) | CosmosContainer.replaceProvisionedThroughput |
| [Obtención de una colección por identificador](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/containercrud/sync/ContainerCRUDQuickstart.java#L126-L133) | CosmosDatabase.getContainer |
| [Lectura de todas las colecciones de una base de datos](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/containercrud/sync/ContainerCRUDQuickstart.java#L136-L150) | CosmosDatabase.readAllContainers |
| [Eliminación de un colección](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/containercrud/sync/ContainerCRUDQuickstart.java#L153-L161) | CosmosContainer.delete |

## <a name="autoscale-collection-examples"></a>Ejemplos de la colección de escalado automático

Para obtener más información sobre el escalado automático antes de ejecutar estos ejemplos, eche un vistazo a estas instrucciones para habilitar el escalado automático en la [cuenta ](https://azure.microsoft.com/resources/templates/101-cosmosdb-sql-autoscale/) y en las [bases de datos y contenedores](https://docs.microsoft.com/azure/cosmos-db/provision-throughput-autoscale).

El archivo de [ejemplos de CRUD de base de datos de escalado automático](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/autoscaledatabasecrud/sync/AutoscaleDatabaseCRUDQuickstart.java) muestra cómo realizar las siguientes tareas:

| Tarea | Referencia de API |
| --- | --- |
| [Crear una base de datos con el rendimiento máximo de escalado automático especificado](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/autoscaledatabasecrud/sync/AutoscaleDatabaseCRUDQuickstart.java#L78-L89) | CosmosClient.createDatabase<br>ThroughputProperties.createAutoscaledThroughput |

El archivo de [ejemplos de CRUD de colección de autoescalado](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/autoscalecontainercrud/sync/AutoscaleContainerCRUDQuickstart.java) muestra cómo realizar las siguientes tareas: 

| Tarea | Referencia de API |
| --- | --- |
| [Crear una colección con el rendimiento máximo de escalado automático especificado](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/autoscalecontainercrud/sync/AutoscaleContainerCRUDQuickstart.java#L97-L110) | CosmosDatabase.createContainerIfNotExists |
| [Cambiar el rendimiento máximo de escalado automático configurado de una colección](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/autoscalecontainercrud/sync/AutoscaleContainerCRUDQuickstart.java#L113-L120) | CosmosContainer.replaceThroughput |
| [Lectura de la configuración del rendimiento de escalado automático de una colección](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/autoscalecontainercrud/sync/AutoscaleContainerCRUDQuickstart.java#L122-L133) | CosmosContainer.readThroughput |

## <a name="analytical-storage-collection-examples"></a>Ejemplos de colección de almacenamiento analítico

El archivo de [ejemplos de CRUD de colección de almacenamiento analítico ](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/analyticalcontainercrud/sync/AnalyticalContainerCRUDQuickstart.java) muestra cómo realizar las siguientes tareas. Para obtener información sobre las colecciones de Azure Cosmos antes de ejecutar los ejemplos siguientes, lea acerca de Synapse y Analytical Store de Azure Cosmos DB.

| Tarea | Referencia de API |
| --- | --- |
| [Creación de una colección](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/analyticalcontainercrud/sync/AnalyticalContainerCRUDQuickstart.java#L93-L108) | CosmosDatabase.createContainerIfNotExists |

## <a name="document-examples"></a>Ejemplos de documento
El archivo de [ejemplos de CRUD de documentos](https://github.com/Azure/azure-documentdb-java/blob/master/documentdb-examples/src/test/java/com/microsoft/azure/documentdb/examples/DocumentCrudSamples.java) muestra cómo realizar las siguientes tareas. Para información sobre los documentos de Azure Cosmos antes de ejecutar los ejemplos siguientes, consulte el artículo conceptual [Uso de bases de datos, contenedores y elementos](databases-containers-items.md).

| Tarea | Referencia de API |
| --- | --- |
| [Creación de un documento](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L133-L147) | CosmosContainer.createItem |
| [Leer un documento por identificador](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L179-L193) | CosmosContainer.readItem |
| [Consulta de documentos](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L162-L176) | CosmosContainer.queryItems |
| [Sustitución de un documento](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L195-L210) | CosmosContainer.replaceItem |
| [Aplicación de upsert a un documento](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L212-L2225) | CosmosContainer.upsertItem |
| [Eliminar un documento](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L303-L310) | CosmosContainer.deleteItem |
| [Reemplazo de un documento con la comprobación de ETag condicional](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L227-L264) | AccessCondition.setType<br>AccessCondition.setCondition |
| [Lectura de documento únicamente si ha cambiado](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L266-L300) | AccessCondition.setType<br>AccessCondition.setCondition |

## <a name="indexing-examples"></a>Ejemplos de indización
El archivo de [ejemplos de CRUD de colección](https://github.com/Azure/azure-documentdb-java/blob/master/documentdb-examples/src/test/java/com/microsoft/azure/documentdb/examples/CollectionCrudSamples.java) muestra cómo realizar las siguientes tareas. Para información sobre la indexación en Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte los artículos conceptuales sobre las [directivas de indexación](index-policy.md), los [tipos de indexación](index-types.md) y las [rutas de acceso de indexación](index-paths.md). 

| Tarea | Referencia de API |
| --- | --- |
| Exclusión de un documento del índice | ExcludedIndex<br>IndexingPolicy |
| Uso de la indexación diferida | IndexingPolicy.IndexingMode |
| [Inclusión de las rutas de acceso de documentos especificados en el índice](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/indexmanagement/sync/SampleIndexManagement.java#L145-L148) | IndexingPolicy.IncludedPaths |
| [Exclusión de las rutas de acceso de documentos especificados del índice](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/indexmanagement/sync/SampleIndexManagement.java#L150-L153) | IndexingPolicy.ExcludedPaths |
| [Creación de un índice compuesto](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/indexmanagement/sync/SampleIndexManagement.java#L171-L186) | IndexingPolicy.setCompositeIndexes<br>CompositePath |
| Forzar una operación de examen de intervalo en una ruta de acceso indexada de hash | FeedOptions.EnableScanInQuery |
| Uso de índices de intervalo en cadenas | IndexingPolicy.IncludedPaths<br>RangeIndex |
| Realización de una transformación de índices | - |
| [Creación de un índice geoespacial](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/indexmanagement/sync/SampleIndexManagement.java#L157-L166) | IndexingPolicy.setSpatialIndexes<br>SpatialSpec<br>SpatialType |

Para más información sobre la indexación, consulte [Directivas de indexación de Azure Cosmos DB](index-policy.md).

## <a name="query-examples"></a>Ejemplos de consultas
El archivo [Ejemplos de consultas](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java) muestra cómo realizar las siguientes tareas con la gramática de consultas SQL. Para más información sobre la referencia de consultas SQL en Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte los [ejemplos de consultas SQL para Azure Cosmos DB](how-to-sql-query.md). 

| Tarea | Referencia de API |
| --- | --- |
| [Consulta de todos los documentos](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L210-L214) | CosmosContainer.queryItems |
| [Consulta de igualdad mediante ==](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L291-L295) | CosmosContainer.queryItems |
| [Consulta de desigualdad mediante != y NOT](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L297-L305) | CosmosContainer.queryItems |
| [Consulta mediante operadores de intervalo como &gt;, &lt;, &gt;= o &lt;=](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L307-L312) | CosmosContainer.queryItems |
| [Consulta con operadores de intervalo en cadenas](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L314-L319) | CosmosContainer.queryItems |
| [Consulta con ORDER BY](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L321-L326) | CosmosContainer.queryItems |
| [Query with DISTINCT](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L328-L333) | CosmosContainer.queryItems |
| [Consulta con funciones de agregado](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L335-L343) | CosmosContainer.queryItems |
| [Uso de subdocumentos](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L345-L353) | CosmosContainer.queryItems |
| [Consulta con Joins dentro de documentos](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L355-L377) | CosmosContainer.queryItems |
| [Consulta con operadores de cadena, matemáticos y de matriz](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L379-L390) | CosmosContainer.queryItems |
| [Consulta con SQL con parámetros mediante SqlQuerySpec](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L392-L421) |CosmosContainer.queryItems |
| [Consulta con paginación explícita](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L216-L266) | CosmosContainer.queryItems |
| [Consulta de colecciones particionadas en paralelo](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L268-L289) | CosmosContainer.queryItems |
| Consulta con ORDER BY en colecciones particionadas | CosmosContainer.queryItems |

## <a name="change-feed-examples"></a>Ejemplos de cambio de fuentes 
En el archivo del [ejemplo para cambiar la fuente de cambios](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/changefeed/SampleChangeFeedProcessor.java) se muestra cómo realizar las siguientes tareas. Para más información sobre la fuente de cambios de Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte [Lectura de la fuente de cambios de Azure Cosmos DB](read-change-feed.md) y [Procesador de fuente de cambios](change-feed-processor.md).

| Tarea | Referencia de API |
| --- | --- |
| [Funcionalidad básica de fuente de cambios](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/changefeed/SampleChangeFeedProcessor.java#L124-L154) |ChangeFeedProcessor.changeFeedProcessorBuilder |
| Lectura de la fuente de cambios a partir de un momento específico | ChangeFeedProcessor.changeFeedProcessorBuilder |
| [Lectura de la fuente de cambios desde el principio](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/changefeed/SampleChangeFeedProcessor.java#L124-L154) | - |

## <a name="server-side-programming-examples"></a>Ejemplos de programación en el servidor

El archivo de [ejemplo de procedimiento almacenado](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/storedprocedure/sync/SampleStoredProcedure.java) muestra cómo realizar las siguientes tareas. Para más información sobre la programación del lado servidor en Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte [Procedimientos almacenados, desencadenadores y funciones definidas por el usuario](stored-procedures-triggers-udfs.md).

| Tarea | Referencia de API |
| --- | --- |
| [Creación de un procedimiento almacenado](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/storedprocedure/sync/SampleStoredProcedure.java#L132-L151) | CosmosScripts.createStoredProcedure |
| [Ejecución de un procedimiento almacenado](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/storedprocedure/sync/SampleStoredProcedure.java#L167-L181) | CosmosStoredProcedure.execute |
| [Eliminación de un procedimiento almacenado](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/storedprocedure/sync/SampleStoredProcedure.java#L183-L193) | CosmosStoredProcedure.delete |

## <a name="user-management-examples"></a>Ejemplos de administración de usuarios
En el archivo de ejemplo de administración de usuarios se muestra cómo realizar las siguientes tareas.

| Tarea | Referencia de API |
| --- | --- |
| Creación de un usuario | - |
| Establecimiento de permisos en una colección o en un documento | - |
| Obtención de una lista de permisos de un usuario |- |