---
title: 'Azure Cosmos DB: ejemplos de .NET (Microsoft.Azure.Cosmos) para la SQL API'
description: Encuentre los ejemplos del SDK de .NET V3 para C# en GitHub para ver tareas comunes que usan la SQL API de Azure Cosmos DB.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 10/07/2019
ms.author: sngun
ms.custom: devx-track-dotnet
ms.openlocfilehash: cfa79569ea4f2ec37eb7086d4878d3592babf18d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89015193"
---
# <a name="azure-cosmos-dbnet-v3-sdk-microsoftazurecosmos-examples-for-the-sql-api"></a>SDK de .NET V3 de Azure Cosmos DB (Microsoft.Azure.Cosmos) para la SQL API

> [!div class="op_single_selector"]
> * [Ejemplos del SDK de .NET V2](sql-api-dotnet-samples.md)
> * [Ejemplos del SDK de .NET V3](sql-api-dotnet-v3sdk-samples.md)
> * [Ejemplos del SDK de Java V4](sql-api-java-sdk-samples.md)
> * [Ejemplos de Node.js](sql-api-nodejs-samples.md)
> * [Ejemplos de Python](sql-api-python-samples.md)
> * [Galería de ejemplos de código de Azure](https://azure.microsoft.com/resources/samples/?sort=0&service=cosmos-db)
>
>

El repositorio [azure-cosmos-dotnet-v3](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage) de GitHub incluye ejemplos de las soluciones .NET más recientes que realizan operaciones CRUD y otras operaciones comunes en recursos de Azure Cosmos DB. Si ya está familiarizado con la versión anterior del SDK de .NET, puede que esté acostumbrado a los términos "colección" y "documento". Dado que Azure Cosmos DB admite varios modelos de API, la versión 3.0 del SDK de .NET usa los términos genéricos "contenedor" y "elemento". Un contenedor puede ser una colección, un grafo o una tabla. Un elemento puede ser un documento, un vértice o borde, o una fila, y es el contenido que hay en un contenedor. Este artículo ofrece:

* Vínculos a las tareas de cada uno de los archivos de proyecto de ejemplo de C#.
* Vínculos al contenido de referencia de la API relacionada.

## <a name="prerequisites"></a>Prerrequisitos

Visual Studio 2019 con el flujo de trabajo de desarrollo de Azure instalado

- Puede descargar y usar [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/), que es una edición **gratuita**. Asegúrese de que habilita **Desarrollo de Azure** durante la instalación de Visual Studio.

   El [paquete NuGet Microsoft.Azure.Cosmos](https://www.nuget.org/packages/Microsoft.Azure.cosmos/).

Una suscripción a Azure o una cuenta de evaluación gratuita de Cosmos DB

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
  
- Puede [activar los beneficios de suscripción a Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio): su suscripción a Visual Studio le proporciona créditos todos los meses que puede usar para servicios de Azure de pago.
- [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]  

> [!NOTE]
> Los ejemplos son independientes y se configuran y limpian por sí solos después. Con cada repetición se factura una hora de uso en su suscripción en el nivel de rendimiento del contenedor.
> 

## <a name="database-examples"></a>Ejemplos de base de datos

El método [RunDatabaseDemo](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/DatabaseManagement/Program.cs#L65-L91) del ejemplo del proyecto *DatabaseManagement* muestra cómo realizar las tareas siguientes. Para más información sobre las bases de datos de Azure Cosmos antes de ejecutar los siguientes ejemplos, consulte [Uso de bases de datos, contenedores y elementos](databases-containers-items.md).

| Tarea | Referencia de API |
| --- | --- |
| [Creación de una base de datos](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/DatabaseManagement/Program.cs#L68) |[CosmosClient.CreateDatabaseIfNotExistsAsync](/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync?view=azure-dotnet) |
| [Lectura de una base de datos por identificador](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/DatabaseManagement/Program.cs#L80) |[Database.ReadAsync](/dotnet/api/microsoft.azure.cosmos.database.readasync?view=azure-dotnet) |
| [Lectura de todas las bases de datos de una cuenta](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/DatabaseManagement/Program.cs#L96) |[CosmosClient.GetDatabaseQueryIterator](/dotnet/api/microsoft.azure.cosmos.cosmosclient.getdatabasequeryiterator?view=azure-dotnet) |
| [Eliminación de una base de datos](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/DatabaseManagement/Program.cs#L106) |[Database.DeleteAsync](/dotnet/api/microsoft.azure.cosmos.database.deleteasync?view=azure-dotnet) |

## <a name="container-examples"></a>Ejemplos de contenedor

El método [RunContainerDemo](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L69-L89) del proyecto *ContainerManagement* de ejemplo muestra cómo realizar las tareas siguientes. Para más información sobre los contenedores de Azure Cosmos antes de ejecutar los siguientes ejemplos, consulte [Uso de bases de datos, contenedores y elementos](databases-containers-items.md).

| Tarea | Referencia de API |
| --- | --- |
| [Creación de un contenedor](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L97-L107) |[Database.CreateContainerIfNotExistsAsync](/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync?view=azure-dotnet) |
| [Creación de un contenedor con una directiva de índice personalizada](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L111-L127) |[Database.CreateContainerIfNotExistsAsync](/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync?view=azure-dotnet) |
| [Cambio del rendimiento configurado de un contenedor](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L149-L171) |[Container.ReplaceThroughputAsync](/dotnet/api/microsoft.azure.cosmos.container.replacethroughputasync?view=azure-dotnet) |
| [Obtención de un contenedor por el identificador](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L176-L185) |[Container.ReadContainerAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readcontainerasync?view=azure-dotnet) |
| [Lectura de todos los contenedores de una base de datos](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L193-L205) |[Database.GetContainerQueryIterator](/dotnet/api/microsoft.azure.cosmos.database.getcontainerqueryiterator?view=azure-dotnet) |
| [Eliminación de un contenedor](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L213-L2018) |[Container.DeleteContainerAsync](/dotnet/api/microsoft.azure.cosmos.container.deletecontainerasync?view=azure-dotnet) |

## <a name="item-examples"></a>Ejemplos de elementos

El método [RunItemsDemo](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L119-L130) del proyecto *ItemManagement* de ejemplo muestra cómo realizar las siguientes tareas. Para conocer más sobre los elementos de Azure Cosmos antes de ejecutar los siguientes ejemplos, consulte [Uso de bases de datos, contenedores y elementos](databases-containers-items.md).

| Tarea | Referencia de API |
| --- | --- |
| [Creación de un elemento](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L161-L200) |[Container.CreateItemAsync](/dotnet/api/microsoft.azure.cosmos.container.createitemasync?view=azure-dotnet) |
| [Lectura de un elemento por identificador](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L203-L241) |[container.ReadItemAsync](/dotnet/api/microsoft.azure.cosmos.container.readitemasync?view=azure-dotnet) |
| [Consulta de elementos](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L244-L320) |[container.GetItemQueryIterator](/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator?view=azure-dotnet) |
| [Reemplazo de un elemento](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L323-L363) |[container.ReplaceItemAsync](/dotnet/api/microsoft.azure.cosmos.container.replaceitemasync?view=azure-dotnet) |
| [Realización de una operación upsert en un elemento](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L366-L411) |[container.UpsertItemAsync](/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync?view=azure-dotnet) |
| [Eliminación de un elemento](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L414-L424) |[container.DeleteItemAsync](/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync?view=azure-dotnet) |
| [Reemplazo de un elemento por la comprobación de ETag condicional](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L585-L674) |[RequestOptions.IfMatchEtag](/dotnet/api/microsoft.azure.cosmos.requestoptions.ifmatchetag?view=azure-dotnet) |

## <a name="indexing-examples"></a>Ejemplos de indización

El método [RunIndexDemo](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/IndexManagement/Program.cs#L108-L122) del proyecto *IndexManagement* de ejemplo muestra cómo realizar las tareas siguientes. Para más información sobre la indexación en Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte las [directivas de indexación](index-policy.md), los [tipos de indexación](index-types.md) y las [rutas de acceso de indexación](index-paths.md). 

| Tarea | Referencia de API |
| --- | --- |
| [Exclusión de un elemento del índice](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/IndexManagement/Program.cs#L130-L186) |[IndexingDirective.Exclude](/dotnet/api/microsoft.azure.cosmos.indexingdirective?view=azure-dotnet) |
| [Uso de la indexación diferida](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/IndexManagement/Program.cs#L198-L220) |[IndexingPolicy.IndexingMode](/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode?view=azure-dotnet) |
| [Exclusión de las rutas de acceso a elementos especificadas del índice](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/IndexManagement/Program.cs#L230-L297) |[IndexingPolicy.ExcludedPaths](/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths?view=azure-dotnet) |

## <a name="query-examples"></a>Ejemplos de consultas

El método [RunDemoAsync](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L76-L96) del proyecto de *Queries* de ejemplo muestra cómo realizar las siguientes tareas con la gramática de consultas SQL, el proveedor LINQ con consulta y Lambda. Para más información sobre la referencia de consultas SQL en Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte los [ejemplos de consultas SQL para Azure Cosmos DB](how-to-sql-query.md).

| Tarea | Referencia de API |
| --- | --- |
| [Consulta de elementos de una única partición](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L154-L186) |[container.GetItemQueryIterator](/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator?view=azure-dotnet) |
| [Consulta de elementos de varias particiones](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L215-L275) |[container.GetItemQueryIterator](/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator?view=azure-dotnet) |
| [Consulta mediante una instrucción SQL](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L189-L212) |[container.GetItemQueryIterator](/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator?view=azure-dotnet) |

## <a name="change-feed-examples"></a>Ejemplos de cambio de fuentes

El método [RunBasicChangeFeed](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs#L91-L119) del proyecto *ChangeFeed* de ejemplo muestra cómo realizar las siguientes tareas. Para más información sobre la fuente de cambios de Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte [Lectura de la fuente de cambios de Azure Cosmos DB](read-change-feed.md) y [Procesador de fuente de cambios](change-feed-processor.md).

| Tarea | Referencia de API |
| --- | --- |
| [Funcionalidad básica de fuente de cambios](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs#L91-L119) |[Container.GetChangeFeedProcessorBuilder](/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder?view=azure-dotnet) |
| [Lectura de la fuente de cambios a partir de un momento específico](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs#L127-L162) |[Container.GetChangeFeedProcessorBuilder](/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder?view=azure-dotnet) |
| [Lectura de la fuente de cambios desde el principio](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs#L170-L198) |[ChangeFeedProcessorBuilder.WithStartTime(DateTime)](/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withstarttime?view=azure-dotnet) |
| [Migración del procesador de la fuente de cambios a la fuente de cambios en el SDK V3](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs#L256-L333) |[Container.GetChangeFeedProcessorBuilder](/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder?view=azure-dotnet) |

## <a name="server-side-programming-examples"></a>Ejemplos de programación en el servidor

El método [RunDemoAsync](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ServerSideScripts/Program.cs#L72-L102) del proyecto *ServerSideScripts* de ejemplo muestra cómo realizar las tareas siguientes. Para más información sobre la programación del lado servidor en Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte [Procedimientos almacenados, desencadenadores y funciones definidas por el usuario](stored-procedures-triggers-udfs.md).

| Tarea | Referencia de API |
| --- | --- |
| [Creación de un procedimiento almacenado](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ServerSideScripts/Program.cs#L116) |[Scripts.CreateStoredProcedureAsync](/dotnet/api/microsoft.azure.cosmos.scripts.scripts.createstoredprocedureasync?view=azure-dotnet) |
| [Ejecución de un procedimiento almacenado](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ServerSideScripts/Program.cs#L135) |[Scripts.ExecuteStoredProcedureAsync](/dotnet/api/microsoft.azure.cosmos.scripts.scripts.executestoredprocedureasync?view=azure-dotnet) |
| [Eliminación de un procedimiento almacenado](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ServerSideScripts/Program.cs#L351) |[Scripts.DeleteStoredProcedureAsync](/dotnet/api/microsoft.azure.cosmos.scripts.scripts.deletestoredprocedureasync?view=azure-dotnet) |
