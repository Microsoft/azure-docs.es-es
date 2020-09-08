---
title: 'Azure Cosmos DB: API, SDK y recursos de .NET para Bulk Executor'
description: Obtenga toda la información sobre la API y el SDK de .NET para BulkExecutor, incluidas la fechas de lanzamiento, fechas de retirada y cambios realizados entre las versiones del SDK de .NET para BulkExecutor de Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.openlocfilehash: 5944cbeea2e26d7209cde7c926f862683c7cab7e
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88590207"
---
# <a name="net-bulk-executor-library-download-information"></a>Biblioteca Bulk Executor para .NET: Información de descarga 

> [!div class="op_single_selector"]
> * [.NET SDK v3](sql-api-sdk-dotnet-standard.md)
> * [SDK de .NET v2](sql-api-sdk-dotnet.md)
> * [SDK de .NET Core v2](sql-api-sdk-dotnet-core.md)
> * [SDK de fuente de cambios de .NET v2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.js](sql-api-sdk-node.md)
> * [SDK de Java v4](sql-api-sdk-java-v4.md)
> * [Versión 2 del SDK de Java asincrónico](sql-api-sdk-async-java.md)
> * [SDK de Java v2 sincrónico](sql-api-sdk-java.md)
> * [Spring Data v2](sql-api-sdk-java-spring-v2.md)
> * [Spring Data v3](sql-api-sdk-java-spring-v3.md)
> * [Conector de Spark](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [Proveedor de recursos de REST](/rest/api/cosmos-db-resource-provider/)
> * [SQL](sql-api-query-reference.md)
> * [Bulk Executor: .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Bulk Executor: Java](sql-api-sdk-bulk-executor-java.md)

| |  |
|---|---|
| **Descripción**| La biblioteca Bulk Executor para .NET permite a las aplicaciones cliente realizar operaciones masivas en las cuentas de Azure Cosmos DB. Esta biblioteca proporciona los espacios de nombres BulkImport, BulkUpdate y BulkDelete. El módulo BulkImport puede ingerir documentos en masa de forma optimizada, de tal forma que la capacidad de proceso aprovisionada para una colección se consuma en el máximo nivel posible. El módulo BulkUpdate puede actualizar en masa los datos existentes en los contenedores de Azure Cosmos como revisiones. El módulo BulkDelete puede eliminar documentos en masa de forma optimizada, de tal forma que el rendimiento aprovisionado para una colección se consuma en el máximo nivel posible.|
|**Descarga del SDK**| [NuGet](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.BulkExecutor/) |
| **Biblioteca BulkExecutor en GitHub**| [GitHub](https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started)|
|**Documentación de la API**|[Documentación de referencia de API de .NET](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor?view=azure-dotnet)|
|**Introducción**|[Introducción al SDK de .NET para la biblioteca BulkExecutor](bulk-executor-dot-net.md)|
| **Plataforma admitida actualmente**| Microsoft .NET Framework 4.5.2, 4.6.1 y .NET Standard 2.0 |

> [!NOTE]
> Si usa Bulk Executor, consulte la versión 3.x más reciente del [SDK de .NET](tutorial-sql-api-dotnet-bulk-import.md), que tiene una instancia de Bulk Executor incorporada en el SDK. 

## <a name="release-notes"></a>Notas de la versión

### <a name="241-preview"></a><a name="2.4.1-preview"></a>2.4.1-preview

* Se corrigió TotalElapsedTime en la respuesta de BulkDelete para medir correctamente el tiempo total, incluidos los reintentos.

### <a name="240-preview"></a><a name="2.4.0-preview"></a>2.4.0-preview

* Se ha modificado la dependencia del SDK por la versión 2.5.1 o posteriores

### <a name="230-preview2"></a><a name="2.3.0-preview2"></a>2.3.0-preview2

* Se ha agregado compatibilidad para que el ejecutor en masa acepte TTL en vértices y bordes.

### <a name="220-preview2"></a><a name="2.2.0-preview2"></a>2.2.0-preview2

* Se ha corregido un problema que provocaba excepciones durante el escalado elástico de Azure Cosmos DB cuando se ejecutaba en modo de puerta de enlace. Esta corrección la hace funcionalmente equivalente a la versión 1.4.1.

### <a name="210-preview2"></a><a name="2.1.0-preview2"></a>2.1.0-preview2

* Se ha agregado compatibilidad con la operación BulkDelete para que las cuentas de la API de SQL acepten la clave de partición y las tuplas de identificación de documentos que se van a eliminar. Este cambio la hace funcionalmente equivalente a la versión 1.4.0.

### <a name="200-preview2"></a><a name="2.0.0-preview2"></a>2.0.0-preview2

* Incluye MongoBulkExecutor para la compatibilidad con .NET Standard 2.0. Esta característica tiene una funcionalidad similar a la de la versión 1.3.0, pero se ha agregado la compatibilidad con .NET Standard 2.0 como plataforma de destino.

### <a name="200-preview"></a><a name="2.0.0-preview"></a>2.0.0-preview

* Se ha agregado .NET Standard 2.0 como una de las plataformas de destino admitidas para que la biblioteca Bulk Executor funcione con las aplicaciones .NET Core.

### <a name="189"></a><a name="1.8.9"></a>1.8.9

* Se ha corregido un problema con BulkDeleteAsync cuando los valores con comillas de escape se pasaban como parámetros de entrada.

### <a name="188"></a><a name="1.8.8"></a>1.8.8

* Se ha corregido un problema en MongoBulkExecutor que incrementaba el tamaño del documento de forma inesperada agregando relleno y, en algunos casos, superando el tamaño máximo del documento.

### <a name="187"></a><a name="1.8.7"></a>1.8.7

* Se ha corregido un problema con BulkDeleteAsync cuando la colección tiene rutas de acceso de clave de partición anidadas.

### <a name="186"></a><a name="1.8.6"></a>1.8.6

* MongoBulkExecutor ahora implementa IDisposable y se espera que se elimine después de usarse.

### <a name="185"></a><a name="1.8.5"></a>1.8.5

* Se quitó el bloqueo de la versión del SDK. El paquete depende ahora del SDK > = 2.5.1.

### <a name="184"></a><a name="1.8.4"></a>1.8.4

* Se corrigió el control de los identificadores al llamar a BulkImport con una lista de objetos POCO con valores numéricos.

### <a name="183"></a><a name="1.8.3"></a>1.8.3

* Se corrigió TotalElapsedTime en la respuesta de BulkDelete para medir correctamente el tiempo total, incluidos los reintentos.

### <a name="182"></a><a name="1.8.2"></a>1.8.2

* Se ha corregido el consumo elevado de CPU en ciertos escenarios.
* El seguimiento ahora utiliza TraceSource. Los usuarios pueden definir clientes de escucha para el origen `BulkExecutorTrace`.
* Se ha corregido un escenario poco frecuente que podía provocar un bloqueo al enviar documentos de aproximadamente 2 MB de tamaño.

### <a name="160"></a><a name="1.6.0"></a>1.6.0

* Se ha actualizado Bulk Executor para usar la versión más reciente del SDK de .NET de Azure Cosmos DB (2.4.0).

### <a name="150"></a><a name="1.5.0"></a>1.5.0

* Se ha agregado compatibilidad para que el ejecutor en masa acepte TTL en vértices y bordes.

### <a name="141"></a><a name="1.4.1"></a>1.4.1

* Se ha corregido un problema que provocaba excepciones durante el escalado elástico de Azure Cosmos DB cuando se ejecutaba en modo de puerta de enlace.

### <a name="140"></a><a name="1.4.0"></a>1.4.0

* Se ha agregado compatibilidad con la operación BulkDelete para que las cuentas de la API de SQL acepten la clave de partición y las tuplas de identificación de documentos que se van a eliminar.

### <a name="130"></a><a name="1.3.0"></a>1.3.0

* Se ha corregido un problema que generaba un error de formato en el agente de usuario utilizado por Bulk Executor.

### <a name="120"></a><a name="1.2.0"></a>1.2.0

* Se han mejorado las API de importación y actualización de Bulk Executor para adaptarlas de forma transparente al escalado elástico del contenedor de Cosmos cuando el espacio de almacenamiento sobrepase la capacidad actual sin generar excepciones.

### <a name="112"></a><a name="1.1.2"></a>1.1.2

* Se ha incrementado la independencia de .NET SDK para DocumentDB con la versión 2.1.3.

### <a name="111"></a><a name="1.1.1"></a>1.1.1

* Se ha corregido un problema que hacía que Bulk Executor generara un error de JSRT al realizar la importación en colecciones fijas.

### <a name="110"></a><a name="1.1.0"></a>1.1.0

* Se agregó compatibilidad con la operación BulkDelete para las cuentas de la API de SQL de Azure Cosmos DB.
* Se agregó compatibilidad con la operación BulkImport para las cuentas con la API de Azure Cosmos DB para MongoDB.
* Se aumentó la dependencia del SDK de .NET de DocumentDB a la versión 2.0.0. 

### <a name="102"></a><a name="1.0.2"></a>1.0.2

* Se agregó compatibilidad con la operación BulkImport para las cuentas de la API de Gremlin de Azure Cosmos DB.

### <a name="101"></a><a name="1.0.1"></a>1.0.1

* Se corrigió un error leve de la operación BulkImport para las cuentas de la API de SQL de Azure Cosmos DB.

### <a name="100"></a><a name="1.0.0"></a>1.0.0

* Se agregó compatibilidad con las operaciones BulkImport y BulkUpdate de las cuentas de la API de SQL de Azure Cosmos DB.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de la biblioteca Bulk Executor de Java, consulte el artículo siguiente:

[SDK e información sobre la versión de la biblioteca Bulk Executor para Java](sql-api-sdk-bulk-executor-java.md)
