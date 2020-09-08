---
title: Notas de la versión y recursos de la API del conector de Apache Spark para Azure Cosmos DB para SQL
description: Obtenga toda la información posible sobre la API del conector de Apache Spark para Azure Cosmos DB para SQL, incluidas la fechas de lanzamiento y de retirada, y los cambios realizados entre versiones del SDK de Java asincrónico para Azure Cosmos DB para SQL.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 46ddbd18051ffa44232468704ce189d4171b50e7
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88590016"
---
# <a name="azure-cosmos-db-apache-spark-connector-for-core-sql-api-release-notes-and-resources"></a>API del conector de Apache Spark para Azure Cosmos DB para Core (SQL): notas de la versión y recursos
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

Acelere el análisis de macrodatos mediante el conector de Apache Spark para Azure Cosmos DB para Core (SQL). El conector de Spark permite ejecutar trabajos de [Spark](https://spark.apache.org/) en datos almacenados en Azure Cosmos DB. Se admite el procesamiento por lotes y por secuencias.

Este conector se puede usar con [Azure Databricks](https://azure.microsoft.com/services/databricks) o con [Azure HDInsight](https://azure.microsoft.com/services/hdinsight/), que proporcionan clústeres de Spark administrados en Azure. En la tabla siguiente se muestran las versiones de Spark admitidas.

| Componente | Versión |
|---------|-------|
| Spark de Apache | 2.4.x, 2.3.x, 2.2.x y 2.1.x |
| Scala | 2.11 |
| Versión del runtime de Azure Databricks | > 3.4 |

> [!WARNING]
> Este conector admite la API Core (SQL) de Azure Cosmos DB.
> Si va a usar Cosmos DB para la API de MongoDB, use el [conector Spark de MongoDB](https://docs.mongodb.com/spark-connector/master/).
> Si va a usar Cosmos DB con Cassandra API, use el [conector Spark de Cassandra](https://github.com/datastax/spark-cassandra-connector).
>

## <a name="helpful-content"></a>Contenido útil

| Contenido | Vínculo |
|---|---|
| **Descarga del SDK** | [Descarga desde Apache Spark](https://aka.ms/CosmosDB_OLTP_Spark_2.4_LKG) |
|**Documentación de la API** | [Referencia del conector de Spark]() |
|**Contribuya al SDK** | [Conector de Apache Spark para Azure Cosmos DB en GitHub](https://github.com/Azure/azure-cosmosdb-spark) | 
|**Introducción** | [Aceleración de análisis de macrodatos mediante el conector de Apache Spark a Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/spark-connector#bk_working_with_connector) <br> [Uso del flujo estructurado de Apache Spark con Apache Kafka y Azure Cosmos DB](https://docs.microsoft.com/azure/hdinsight/apache-kafka-spark-structured-streaming-cosmosdb?toc=/azure/cosmos-db/toc.json&bc=/azure/cosmos-db/breadcrumb/toc.json) | 

## <a name="release-history"></a>Historial de versiones

### <a name="311"></a>3.1.1
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Corrige un caso extremo de punto de control de streaming donde "id" contiene el carácter "|" con la configuración "ChangeFeedMaxPagesPerBatch" aplicada

### <a name="310"></a>3.1.0
#### <a name="new-features"></a>Nuevas características
* Agrega compatibilidad con las actualizaciones en masa cuando se usan claves de partición anidadas
* Agrega compatibilidad con los tipos de datos decimal y float durante las escrituras en Cosmos DB
* Agrega compatibilidad con los tipos de marca de tiempo cuando usan Long (Unix Epoch) como un valor
#### <a name="key-bug-fixes"></a>Correcciones de errores clave

### <a name="308"></a>3.0.8
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Corrige la excepción de conversión de tipo cuando se usa la configuración "WriteThroughputBudget".

### <a name="307"></a>3.0.7
#### <a name="new-features"></a>Nuevas características
* Agrega información de error sobre errores en masa a la excepción y el registro.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave

### <a name="306"></a>3.0.6
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Corrige problemas de punto de control de streaming.

### <a name="305"></a>3.0.5
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Corrige el nivel de registro de un mensaje dejado sin querer con el nivel ERROR para reducir el ruido

### <a name="304"></a>3.0.4
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Corrige un error en el flujo estructurado durante las divisiones de particiones que posiblemente de lugar a que falten algunos registros de fuente de cambios o a que se vean excepciones Null para las escrituras de punto de control

### <a name="303"></a>3.0.3
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Corrige un error donde se omite un esquema personalizado proporcionado para readStream

### <a name="302"></a>3.0.2
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Corrige la regresión (el archivo JAR no sombreado incluye todas las dependencias sombreadas) con un aumento del 50 % en el tiempo de compilación

### <a name="301"></a>3.0.1
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Corrige un problema de dependencia que provoca un error de transporte directo a través de TCP que indica RequestTimeoutException

### <a name="300"></a>3.0.0
#### <a name="new-features"></a>Nuevas características
* Mejora la administración y la agrupación de conexiones para reducir el número de llamadas de metadatos
#### <a name="key-bug-fixes"></a>Correcciones de errores clave

## <a name="faq"></a>Preguntas más frecuentes
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Cosmos DB, consulte la página del servicio [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Para obtener más información sobre Apache Spark, vea la [página principal](https://spark.apache.org/).