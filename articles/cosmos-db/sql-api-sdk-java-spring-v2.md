---
title: Notas de la versión y recursos de Spring Data Azure Cosmos DB v2 para SQL API
description: Obtenga toda la información sobre Spring Data Azure Cosmos DB v2 para SQL API, incluidas la fechas de lanzamiento y de retirada, y los cambios realizados entre las versiones del SDK de Java asincrónico para SQL de Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 47d9a8ff884e29dc5692c97d5e7867a856d01063
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88590536"
---
# <a name="spring-data-azure-cosmos-db-v2-for-core-sql-api-release-notes-and-resources"></a>Spring Data Azure Cosmos DB v2 para Core (SQL) API: notas de la versión y recursos
> [!div class="op_single_selector"]
> * [.NET SDK v3](sql-api-sdk-dotnet-standard.md)
> * [SDK de .NET v2](sql-api-sdk-dotnet.md)
> * [SDK de .NET Core v2](sql-api-sdk-dotnet-core.md)
> * [SDK de fuente de cambios de .NET v2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.js](sql-api-sdk-node.md)
> * [SDK de Java v4](sql-api-sdk-java-v4.md)
> * [Versión 2 del SDK de Java asincrónico](sql-api-sdk-async-java.md)
> * [SDK de Java v2 sincrónico](sql-api-sdk-java.md)
> * [Spring Data v2](sql-api-sdk-java-spring-v2.md)
> * [Spring Data v3](sql-api-sdk-java-spring-v3.md)
> * [Conector de Spark](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [Proveedor de recursos de REST](/rest/api/cosmos-db-resource-provider/)
> * [SQL](sql-api-query-reference.md)
> * [Bulk Executor: .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Bulk Executor: Java](sql-api-sdk-bulk-executor-java.md)

Spring Data Azure Cosmos DB v2 para Core (SQL) permite que los desarrolladores utilicen Azure Cosmos DB en aplicaciones de Spring. Spring Data Azure Cosmos DB expone la interfaz de Spring Data para manipular las bases de datos y las colecciones, trabajar con documentos y emitir consultas. Las API sincrónicas y las API asincrónicas (reactivas) se admiten en el mismo artefacto Maven. 

[Spring Framework](https://spring.io/projects/spring-framework) es un modelo de programación y configuración que simplifica el desarrollo de aplicaciones Java. Para citar el sitio web de la organización, Spring simplifica los "recursos" de las aplicaciones mediante la inserción de dependencias. Son muchos los desarrolladores que prefieren Spring, porque la compilación y prueba de las aplicaciones resultan más sencillas. [Spring Boot](https://spring.io/projects/spring-boot) amplía esta idea de administrar los recursos pensando en la implementación de microservicios y aplicaciones web. [Spring Data](https://spring.io/projects/spring-data) es un modelo de programación para acceder a almacenes de datos como Azure Cosmos DB desde el contexto de una aplicación de Spring o Spring Boot. 

Puede usar Spring Data Azure Cosmos DB en las aplicaciones de [Azure Spring Cloud](https://azure.microsoft.com/services/spring-cloud/).

> [!IMPORTANT]  
> Estas notas de la versión son para v2 de Spring Data Azure Cosmos DB. Puede encontrar las notas de la versión v3 [aquí](sql-api-sdk-java-spring-v3.md). 
>
> Spring Data Azure Cosmos DB solo admite SQL API.
>
> Las guías siguientes admiten Spring Data en otras API de Azure Cosmos DB:
> * [Spring Data para Apache Cassandra con Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [Spring Data MongoDB con Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
> * [Spring Data Gremlin con Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-gremlin-java-app-with-cosmos-db)
>
> ¿Desea empezar a trabajar rápidamente?
> 1. Instale el [runtime de Java mínimo admitido, JDK 8](/java/azure/jdk/?view=azure-java-stable) para poder usar el SDK.
> 2. Cree una aplicación de Spring Data Azure Cosmos DB con la utilidad Starter, [es sencillo](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db).
> 3. Trabaje con la [Guía del desarrollador de Spring Data Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/how-to-guides-spring-data-cosmosdb) que lo guía a través de las solicitudes básicas de Azure Cosmos DB.
>
> Puede poner en marcha aplicaciones de Spring Boot Starter rápidamente con [Spring Initializr](https://start.spring.io/).
>

## <a name="helpful-content"></a>Contenido útil

| Contenido | Vínculo |
|---|---|
| **Descarga del SDK** | [Maven](https://mvnrepository.com/artifact/com.microsoft.azure/spring-data-cosmosdb) |
|**Documentación de la API** | [Documentación de referencia de Spring Data Azure Cosmos DB]() |
|**Contribuya al SDK** | [Repositorio de Spring Data Azure Cosmos DB en GitHub](https://github.com/microsoft/spring-data-cosmosdb) | 
|**Spring Boot Starter**| [Biblioteca cliente de Azure Cosmos DB Spring Boot Starter para Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-starter-cosmosdb) |
|**Ejemplo de aplicación TODO de Spring con Azure Cosmos DB**| [Experiencia completa en Java en App Service Linux (parte 2)](https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2) |
|**Guía para desarrolladores** | [Guía del desarrollador de Spring Data Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/how-to-guides-spring-data-cosmosdb) | 
|**Guía para usar la utilidad Starter** | [Uso de la utilidad Spring Boot Starter con SQL API de Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db) <br> [Repositorio de GitHub para Azure Spring Boot Starter Cosmos DB](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-starter-cosmosdb) |
|**Ejemplo con App Services** | [Cómo usar Spring y Cosmos DB con App Service en Linux](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-app-with-cosmos-db-on-app-service-linux) <br> [Ejemplo de aplicación TODO](https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2.git) |

## <a name="release-history"></a>Historial de versiones

### <a name="230-2020-05-21"></a>2.3.0 (2020-05-21)
#### <a name="new-features"></a>Nuevas características
* Versión de Spring Boot actualizada a 2.3.0 
#### <a name="key-bug-fixes"></a>Correcciones de errores clave

### <a name="225-2020-05-19"></a>2.2.5 (2020-05-19)
#### <a name="new-features"></a>Nuevas características
* Versión de Azure Cosmos DB actualizada a v3.7.3
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Contiene correcciones de fuga de memoria y actualizaciones de versiones de Netty desde la versión 3.7.3 del SDK de Cosmos 

### <a name="224-2020-04-06"></a>2.2.4 (2020-04-06)
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Marca allowTelemetry corregida para tener en cuenta desde CosmosDbConfig
* Se corrigió la propiedad TTL en el contenedor

### <a name="223-2020-02-25"></a>2.2.3 (2020-02-25)
#### <a name="new-features"></a>Nuevas características
* Se agregó un nuevo elemento findAll por API de clave de partición
* Versión de azure-cosmos actualizada a 3.7.0
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se corrigió collectionName -> containerName
* Se corrigió entityClass y domainClass -> domainType
* Se corrigió "Return entity collection saved by repository instead of input entity" ("Devolver colección de entidades guardada por el repositorio en lugar de la entidad de entrada")

### <a name="2110-2020-02-25"></a>2.1.10 (2020-02-25)
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Corrección para "Return entity collection saved by repository instead of input entity" ("Devolver colección de entidades guardada por el repositorio en lugar de la entidad de entrada")

### <a name="222-2020-01-15"></a>2.2.2 (2020-01-15)
#### <a name="new-features"></a>Nuevas características
* Versión de azure-cosmos actualizada a v3.6.0
#### <a name="key-bug-fixes"></a>Correcciones de errores clave

### <a name="221-2019-12-31"></a>2.2.1 (2019-12-31)
#### <a name="new-features"></a>Nuevas características
* Versión del SDK de Cosmos DB actualizada a 3.5.0
* Se agregó un campo de anotaciones para habilitar/deshabilitar una colección de creación automática
* Mejor control de excepciones, se expuso CosmosClientException a CosmosDBAccessException
* Se expuso requestCharge y activityId a ResponseDiagnostics
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* La actualización 3.5.0 del SDK corrige "Exception when Cosmos DB HTTP response header is larger than 8192 bytes" ("Excepción cuando el encabezado de respuesta HTTP de Cosmos DB es mayor que 8192 bytes"), "ConsistencyPolicy.defaultConsistencyLevel() fails on Bounded Staleness and Consistent Prefix" ("ConsistencyPolicy.defaultConsistencyLevel() produce un error en la obsolescencia limitada y prefijo coherente")
* Se corrigió el comportamiento de las API findById, devolver un valor vacío cuando no se encuentra, en lugar de generar una excepción
* Se corrigió un error en el que no se aplicaba la ordenación en la página siguiente al usar CosmosPageRequest

### <a name="219-2019-12-26"></a>2.1.9 (2019-12-26)
#### <a name="new-features"></a>Nuevas características
* Se agregó un campo de anotaciones para habilitar/deshabilitar una colección de creación automática
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se corrigió el comportamiento de las API findById, devolver un valor vacío cuando no se encuentra, en lugar de generar una excepción

### <a name="220-2019-10-21"></a>2.2.0 (2019-10-21)
#### <a name="new-features"></a>Nuevas características
* Compatibilidad completa con el repositorio reactivo de Cosmos
* Compatibilidad con métricas de consulta y cadena de diagnóstico de solicitud de Cosmos DB
* Versión del SDK de Cosmos DB actualizada a 3.3.1
* Versión de Spring Framework actualizada a 5.2.0.RELEASE
* Versión de Spring Data Commons actualizada a 2.2.0.RELEASE
* Se agregaron las API findByIdAndPartitionKey, deleteByIdAndPartitionKey
* Se quitó la dependencia de azure-doumentdb
* DocumentDb pasó a llamarse Cosmos
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se corrigió "Sorting throws exception when pageSize is less than total items in repository" ("La ordenación genera una excepción cuando pageSize es inferior a los elementos totales del repositorio")

### <a name="218-2019-10-18"></a>2.1.8 (2019-10-18)
#### <a name="new-features"></a>Nuevas características
* Desuso de las API de Document DB
* Se agregaron las API findByIdAndPartitionKey, deleteByIdAndPartitionKey.
* Se agregó el bloqueo optimista basado en _etag
* Se habilitó la expresión SPeL para el nombre de la colección de documentos
* Mejoras de ObjectMapper.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave

### <a name="217-2019-10-18"></a>2.1.7 (2019-10-18)
#### <a name="new-features"></a>Nuevas características
* Se agregó la dependencia de la versión 3 del SDK de Cosmos
* Se agregó el repositorio reactivo de Cosmos
* Se actualizó la implementación de DocumentDbTemplate para usar la versión 3 del SDK de Cosmos.
* Otros cambios de configuración para la compatibilidad con el repositorio reactivo de Cosmos.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave

### <a name="212-2019-03-19"></a>2.1.2 (2019-03-19)
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se quita la dependencia de applicationInsights para
    * Riesgo potencial de contaminación de dependencias
    * Incompatibilidad con Java 11
    * Evitar el posible impacto en el rendimiento de la CPU o la memoria.

### <a name="207-2019-03-20"></a>2.0.7 (2019-03-20)
#### <a name="new-features"></a>Nuevas características
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* La corrección quita la dependencia de applicationInsights para
    * Riesgo potencial de contaminación de dependencias
    * Incompatibilidad con Java 11
    * Evitar el posible impacto en el rendimiento de la CPU o la memoria.

### <a name="211-2019-03-07"></a>2.1.1 (2019-03-07)
#### <a name="new-features"></a>Nuevas características
* Actualizar la versión maestra a 2.1.1
#### <a name="key-bug-fixes"></a>Correcciones de errores clave

### <a name="206-2019-03-07"></a>2.0.6 (2019-03-07)
#### <a name="new-features"></a>Nuevas características
* Omitir todas las excepciones de la telemetría
#### <a name="key-bug-fixes"></a>Correcciones de errores clave

### <a name="210-2018-12-17"></a>2.1.0 (2018-12-17)
#### <a name="new-features"></a>Nuevas características
* Actualización de la versión a 2.1.0 para solucionar el problema
#### <a name="key-bug-fixes"></a>Correcciones de errores clave

### <a name="205-2018-09-13"></a>2.0.5 (2018-09-13)
#### <a name="new-features"></a>Nuevas características
* Agregar la palabra clave exists, startsWith
* Actualizar Léame
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Corrección de "Cant call self href directly for Entity" ("No se puede llamar a self hred directamente para la entidad")
* Correción de "findAll will fail if collection is not created" ("findAll generará error si no se crea la colección")

### <a name="204-pre-release-2018-08-23"></a>2.0.4 (versión preliminar) (2018-08-23)
#### <a name="new-features"></a>Nuevas características
* Cambio de nombre del paquete de documentdb a cosmosdb
* Se agrega una característica nueva de palabra clave del método de consulta, se admiten 16 palabras clave desde API SQL.
* Se agrega una característica nueva de consulta con paginación y ordenación.
* Se simplifica la configuración de spring-data-cosmosdb.
* Se agregan deleteCollection API y deleteAll API.

#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Corrección de errores y mejora de defectos.

## <a name="faq"></a>Preguntas más frecuentes
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre Cosmos DB, consulte la página del servicio [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Para más información sobre Spring Framework, consulte la [página principal del proyecto](https://spring.io/projects/spring-framework).

Para más información sobre Spring Boot, consulte la [página principal del proyecto](https://spring.io/projects/spring-boot).

Para más información sobre Spring Data, consulte la [página principal del proyecto](https://spring.io/projects/spring-data).