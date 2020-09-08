---
title: Notas de la versión y recursos de Spring Data Azure Cosmos DB v3 para API SQL
description: Obtenga toda la información sobre Spring Data Azure Cosmos DB v3 para SQL API, incluidas las fechas de lanzamiento y de retirada, y los cambios realizados entre las versiones del SDK de Java asincrónico para SQL de Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 08/18/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 04a50d37606754ff4540d1056e378d46388e2592
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88590533"
---
# <a name="spring-data-azure-cosmos-db-v3-for-core-sql-api-release-notes-and-resources"></a>Spring Data Azure Cosmos DB v3 para Core (SQL) API: notas de la versión y recursos
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

Spring Data Azure Cosmos DB v3 para Core (SQL) permite que los desarrolladores utilicen Azure Cosmos DB en aplicaciones de Spring. Spring Data Azure Cosmos DB expone la interfaz de Spring Data para manipular las bases de datos y las colecciones, trabajar con documentos y emitir consultas. Las API sincrónicas y las API asincrónicas (reactivas) se admiten en el mismo artefacto Maven. 

Spring Data Azure Cosmos DB toma una dependencia en la plataforma de Spring Data. El equipo del SDK de Azure Cosmos DB publica los artefactos de Maven para Spring Data v2.2 y v2.3.

[Spring Framework](https://spring.io/projects/spring-framework) es un modelo de programación y configuración que simplifica el desarrollo de aplicaciones Java. Para citar el sitio web de la organización, Spring simplifica los "recursos" de las aplicaciones mediante la inserción de dependencias. Son muchos los desarrolladores que prefieren Spring, porque la compilación y prueba de las aplicaciones resultan más sencillas. [Spring Boot](https://spring.io/projects/spring-boot) amplía esta idea de administrar los recursos pensando en la implementación de microservicios y aplicaciones web. [Spring Data](https://spring.io/projects/spring-data) es un modelo y una plataforma de programación para acceder a almacenes de datos como Azure Cosmos DB desde el contexto de una aplicación de Spring o Spring Boot. 

Puede usar Spring Data Azure Cosmos DB en las aplicaciones de [Azure Spring Cloud](https://azure.microsoft.com/services/spring-cloud/).

> [!IMPORTANT]  
> Estas notas de la versión son para v3 de Spring Data Azure Cosmos DB. Puede encontrar las notas de la versión v2 [aquí](sql-api-sdk-java-spring-v2.md). 
>
> Spring Data Azure Cosmos DB solo admite SQL API.
>
> Las guías siguientes admiten Spring Data en otras API de Azure Cosmos DB:
> * [Spring Data para Apache Cassandra con Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [Spring Data MongoDB con Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
> * [Spring Data Gremlin con Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-gremlin-java-app-with-cosmos-db)
>

## <a name="start-here"></a>Comience aquí

# <a name="explore"></a>[Explorar](#tab/explore)

<img src="media/sql-api-sdk-java-spring-v3/up-arrow.png" alt="explore the tabs above" width="80"/>

### <a name="navigate-the-tabs-above-for-basic-spring-data-azure-cosmos-db-samples"></a>Desplácese por las pestañas anteriores para obtener ejemplos básicos de Spring Data Azure Cosmos DB.

# <a name="pomxml"></a>[pom.xml](#tab/pom)

### <a name="configure-dependencies"></a>Configuración de dependencias

Hay dos artefactos de Maven de Spring Data Azure Cosmos DB v3 disponibles.

Artefacto que depende de la plataforma de Spring Data v2.2:
```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-spring-data-2-2-cosmos</artifactId>
    <version>latest</version>
</dependency>
```

Artefacto que depende de la plataforma de Spring Data v2.3:
```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-spring-data-2-3-cosmos</artifactId>
    <version>latest</version>
</dependency>
```

# <a name="connect"></a>[Conexión](#tab/connect)

### <a name="connect"></a>Conectar

Especifique los detalles de una cuenta y un contenedor de Azure Cosmos DB. Spring Data Azure Cosmos DB crea automáticamente el cliente y se conecta al contenedor.

[application.properties](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/blob/main/azure-spring-data-2-2-cosmos-java-getting-started/src/main/resources/application.properties):
```
cosmos.uri=${ACCOUNT_HOST}
cosmos.key=${ACCOUNT_KEY}
cosmos.secondaryKey=${SECONDARY_ACCOUNT_KEY}

dynamic.collection.name=spel-property-collection
# Populate query metrics
cosmos.queryMetricsEnabled=true
```

# <a name="doc-ops"></a>[Operaciones de documento](#tab/docs)

### <a name="document-operations"></a>Operaciones de documento

[Crear](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/blob/main/azure-spring-data-2-3-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java): [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-2-3-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Create)]

[Eliminar](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/blob/main/azure-spring-data-2-3-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java): [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-2-3-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Delete)]

# <a name="query"></a>[Consultar](#tab/queries)

### <a name="query"></a>Consultar

[Consultar](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/blob/main/azure-spring-data-2-3-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java): [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-2-3-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Query)]

---

## <a name="helpful-content"></a>Contenido útil

| Contenido | Plataforma de Spring Data v2.2 | Plataforma de Spring Data v2.3 |
|---|---|
| **Descarga del SDK** | [Maven](https://mvnrepository.com/artifact/com.azure/azure-spring-data-2-2-cosmos) | [Maven](https://mvnrepository.com/artifact/com.azure/azure-spring-data-2-3-cosmos) |
|**Contribuya al SDK** | [Repositorio de Spring Data Azure Cosmos DB en GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-spring-data-2-2-cosmos) | [Repositorio de Spring Data Azure Cosmos DB en GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-spring-data-2-3-cosmos) | 
|**Tutorial**| [Tutorial de Spring Data Azure Cosmos DB en GitHub](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/tree/main/azure-spring-data-2-2-cosmos-java-getting-started) | [Tutorial de Spring Data Azure Cosmos DB en GitHub](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/tree/main/azure-spring-data-2-3-cosmos-java-getting-started) |

## <a name="release-history"></a>Historial de versiones

### <a name="300-beta1-2020-08-17"></a>3.0.0-beta.1 (2020-08-17)
#### <a name="new-features"></a>Nuevas características
* Identificador de grupo actualizado a `com.azure`.
* Identificador de artefacto actualizado a `azure-spring-data-2-3-cosmos`.
* Dependencia del SDK de azure-cosmos actualizada a `4.3.2-beta.2`.
* Compatibilidad con entidades de auditoría: administración automática de los campos anotados de createdBy, createdDate, lastModifiedBy y lastModifiedDate.
* Compatibilidad con anotaciones `@GeneratedValue` para la generación de identificadores automáticos de los campos de identificador de tipo `String`.
* Compatibilidad con la configuración de varias bases de datos para una sola cuenta de cosmos con varias bases de datos y varias cuentas de cosmos con varias bases de datos.
* Compatibilidad con anotación `@Version` en cualquier campo de cadena.
* Las API de sincronización actualizadas devuelven tipos a tipos `Iterable` en lugar de `List`.
* `CosmosClientBuilder` expuesto del SDK de Cosmos como bean de Spring a la clase `@Configuration`.
* `CosmosConfig` actualizado para contener métricas de consulta y la implementación del procesador de diagnóstico de respuesta.
* Compatibilidad para devolver tipo de datos `Optional` a las consultas de un solo resultado.
#### <a name="renames"></a>Cambios de nombre
* De `CosmosDbFactory` a `CosmosFactory`.
* De `CosmosDBConfig` a `CosmosConfig`.
* De `CosmosDBAccessException` a `CosmosAccessException`.
* De anotación `Document` a anotación `Container`.
* De anotación `DocumentIndexingPolicy` a anotación `CosmosIndexingPolicy`.
* De `DocumentQuery` a `CosmosQuery`.
* Marca application.properties de `populateQueryMetrics` a `queryMetricsEnabled`.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Programación de la tarea de registro de diagnósticos a subprocesos `Parallel` para evitar el bloqueo de subprocesos de E/S de Netty.
* Se ha corregido el bloqueo optimista en la operación de eliminación.
* Se ha corregido un problema relacionado con las consultas de escape para la cláusula `IN`.
* Se ha corregido un problema al permitir el tipo de datos `long` para `@Id`.
* Se ha corregido un problema al permitir `boolean`, `long`, `int`, `double` como tipos de datos para la anotación `@PartitionKey`.
* Se han corregido las palabras clave `IgnoreCase` & `AllIgnoreCase` para las consultas que no distinguen entre mayúsculas y minúsculas.
* Se ha quitado el valor de unidad de solicitud predeterminado de 4000 cuando se crean contenedores automáticamente.

## <a name="faq"></a>Preguntas más frecuentes
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre Cosmos DB, consulte la página del servicio [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Para más información sobre Spring Framework, consulte la [página principal del proyecto](https://spring.io/projects/spring-framework).

Para más información sobre Spring Boot, consulte la [página principal del proyecto](https://spring.io/projects/spring-boot).

Para más información sobre Spring Data, consulte la [página principal del proyecto](https://spring.io/projects/spring-data).