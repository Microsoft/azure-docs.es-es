---
title: Notas de la versión y recursos de la versión 4 del SDK de Java para la API SQL de Azure Cosmos DB
description: Obtenga toda la información sobre la versión 4 del SDK de Java de Azure Cosmos DB para el SDK y API SQL, incluidas la fechas de lanzamiento y de retirada, y los cambios realizados entre las versiones del SDK de Java asincrónico para SQL de Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 05/20/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 51582fd9aba8721b28f6fb18daec4d0009d0ac15
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500664"
---
# <a name="azure-cosmos-db-java-sdk-v4-for-core-sql-api-release-notes-and-resources"></a>Versión 4 del SDK de Java para la API SQL de Azure Cosmos DB: notas de la versión y recursos
> [!div class="op_single_selector"]
> * [.NET SDK v3](sql-api-sdk-dotnet-standard.md)
> * [SDK de .NET, versión 2](sql-api-sdk-dotnet.md)
> * [SDK de .NET Core, versión 2](sql-api-sdk-dotnet-core.md)
> * [SDK de fuente de cambios de .NET, versión 2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.js](sql-api-sdk-node.md)
> * [SDK de Java v4](sql-api-sdk-java-v4.md)
> * [Versión 2 del SDK de Java asincrónico](sql-api-sdk-async-java.md)
> * [SDK de Java v2 sincrónico](sql-api-sdk-java.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [Proveedor de recursos de REST](/rest/api/cosmos-db-resource-provider/)
> * [SQL](sql-api-query-reference.md)
> * [Bulk Executor: .NET, versión 2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Bulk Executor: Java](sql-api-sdk-bulk-executor-java.md)

La versión 4 del SDK de Java de Azure Cosmos DB para Core (SQL) combina una API asincrónica y una API de sincronización en un solo artefacto de Maven. El SDK v4 ofrece rendimiento mejorado, nuevas características de API y compatibilidad asincrónica basada en Project Reactor y la [biblioteca Netty](https://netty.io/). Los usuarios pueden esperar un rendimiento mejorado con la versión 4 del SDK de Java de Azure Cosmos DB en comparación con la [versión 2 del SDK de Java asincrónico de Azure Cosmos DB](sql-api-sdk-async-java.md) y la [versión 2 del SDK de Java sincrónico de Azure Cosmos DB](sql-api-sdk-java.md).

> [!IMPORTANT]  
> Estas son las notas de la versión solo para la versión 4 del SDK de Java para Azure Cosmos DB. Si en la actualidad usa una versión anterior a la v4, vea la guía [Migración a la versión 4 del SDK de Java de Azure Cosmos DB](migrate-java-v4-sdk.md) a fin de obtener ayuda para actualizar a v4.
>
> Estos son tres pasos para empezar a trabajar rápidamente.
> 1. Instale el [runtime de Java mínimo admitido, JDK 8](/java/azure/jdk/?view=azure-java-stable) para poder usar el SDK.
> 2. Consulte la [Guía de inicio rápido para la versión 4 del SDK de Java de Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/create-sql-api-java), en la que se proporciona acceso al artefacto de Maven y se describen solicitudes básicas de Azure Cosmos DB.
> 3. Lea las [sugerencias de rendimiento](performance-tips-java-sdk-v4-sql.md) y las guías de [solución de problemas](troubleshoot-java-sdk-v4-sql.md) de la versión 4 del SDK de Java de Azure Cosmos DB para optimizar el SDK de la aplicación.
>
> Los [talleres y laboratorios de Azure Cosmos DB](https://aka.ms/cosmosworkshop) son otro fantástico recurso para aprender a usar la versión 4 del SDK de Java de Azure Cosmos DB.
>

| |  |
|---|---|
| **Descarga del SDK** | [Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos) |
|**Documentación de la API** | [Documentación de referencia de API](https://docs.microsoft.com/java/api/overview/azure/cosmosdb/client?view=azure-java-stable) |
|**Contribuya al SDK** | [Repositorio central del SDK de Azure para Java en GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos) | 
|**Introducción** | [Inicio rápido: Creación de una aplicación Java para administrar los datos de la API SQL de Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/create-sql-api-java) · [Repositorio de GitHub con código de inicio rápido](https://github.com/Azure-Samples/azure-cosmos-java-getting-started) | 
|**Ejemplos de código básicos** | [Azure Cosmos DB: ejemplos de Java para la API SQL](sql-api-java-sdk-samples.md) · [Repositorio de GitHub con código de ejemplo](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples)|
|**Aplicación de consola con fuente de cambios**| [Fuente de cambios: ejemplo de la versión 4 del SDK de Java](create-sql-api-java-changefeed.md) · [Repositorio de GitHub con código de ejemplo](https://github.com/Azure-Samples/azure-cosmos-java-sql-app-example)| 
|**Ejemplo de aplicación web**| [Compilación de una aplicación web con la versión 4 del SDK de Java](sql-api-java-application.md) · [Repositorio de GitHub con código de ejemplo](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-todo-app)|
| **Consejos de rendimiento**| [Sugerencias de rendimiento para la versión 4 del SDK de Java](performance-tips-java-sdk-v4-sql.md)| 
| **Solución de problemas** | [Solución de problemas de la versión 4 del SDK de Java](troubleshoot-java-sdk-v4-sql.md) |
| **Migración a v4 desde un SDK anterior** | [Migración a la versión 4 del SDK de Java](migrate-java-v4-sdk.md) |
| **Tiempo de ejecución mínimo admitido**|[JDK 8](/java/azure/jdk/?view=azure-java-stable) | 
| **Talleres y laboratorios de Azure Cosmos DB** |[Página principal de talleres de Cosmos DB](https://aka.ms/cosmosworkshop)

## <a name="release-history"></a>Historial de versiones

### <a name="440-beta1-unreleased"></a>4.4.0-beta.1 (no publicada)

### <a name="430-2020-07-29"></a>4.3.0 (2020-07-29)
#### <a name="new-features"></a>Nuevas características
* Versión actualizada de la biblioteca reactor-core a `3.3.8.RELEASE` 
* Versión actualizada de la biblioteca reactor-netty a `0.9.10.RELEASE` 
* Versión actualizada de la biblioteca netty a `4.1.51.Final` 
* Nuevas API de sobrecarga agregadas para `upsertItem` con `partitionKey` 
* Compatibilidad agregada con el seguimiento de telemetría abierto 
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se corrigió un problema por el que se genera SSLException en caso de cancelación de solicitudes en modo PUERTA DE ENLACE.
* Se corrigió una directiva de reintentos con limitación de recursos en la ejecución de procedimientos almacenados.
* Se corrigió un problema en el que el SDK se bloquea en modo DEPURACIÓN a nivel de registro. 
* Se corrigieron picos periódicos en la latencia en modo Directo. 
* Se corrigió un problema de hora de inicialización de cliente elevado. 
* Se corrigió el error del proxy HTTP al personalizar el cliente con el modo de puerta de enlace y el modo directo. 
* Se corrigió un posible error NPE en opciones null al transferir usuarios. 
* Se agregó timeUnit a `requestLatency` en la cadena de diagnóstico.
* Se quitó la cadena de URI duplicada de la cadena de diagnóstico. 
* Se corrigió la cadena de diagnóstico en el formato JSON adecuado para las operaciones de punto.
* Se corrigió un problema con el operador `.single()` que causaba que la cadena de reactor se deshiciera en caso de excepción no encontrada. 

### <a name="420-2020-07-14"></a>4.2.0 (2020-07-14)
#### <a name="new-features"></a>Nuevas características
* Se ha agregado la API habilitada para el registro de scripts a `CosmosStoredProcedureRequestOptions`.
* Se actualizó `idleEndpointTimeout` predeterminada de `DirectConnectionConfig` a 1 h y `connectTimeout` predeterminado a 5 s.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se ha corregido un problema por el que `GatewayConnectionConfig` `idleConnectionTimeout` invalidaba `DirectConnectionConfig` `idleConnectionTimeout`.
* Se han corregido las API de obtención y establecimiento `responseContinuationTokenLimitInKb` en `CosmosQueryRequestOptions`.
* Se ha corregido un problema en la consulta y fuente de cambios al volver a crear la colección con el mismo nombre.
* Se ha corregido el problema con la consulta superior lanzando ClassCastException.
* Se ha corregido el problema con el orden mediante la consulta lanzando NullPointerException.
* Se ha corregido un problema en el control de las solicitudes canceladas en el modo directo que causaba la llamada del reactor `onErrorDropped`. 

### <a name="410-2020-06-25"></a>4.1.0 (25-06-2020)
#### <a name="new-features"></a>Nuevas características
* Se ha agregado compatibilidad con la consulta `GROUP BY`.
* Se ha aumentado a 130 el valor predeterminado de maxConnectionsPerEndpoint en DirectConnectionConfig.
* Se ha aumentado a 30 el valor predeterminado de maxRequestsPerConnection en DirectConnectionConfig.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se han corregido problemas con el orden por consulta, que devolvía resultados duplicados al reanudar mediante un token de continuación. 
* Se han corregido problemas con la consulta de valores que devolvía valores NULL para el objeto anidado.
* Se ha corregido la excepción del puntero nulo en el administrador de solicitudes en RntbdClientChannelPool.

### <a name="401-2020-06-10"></a>4.0.1 (10-06-2020)
#### <a name="new-features"></a>Nuevas características
* Se ha cambiado el nombre de `QueryRequestOptions` por `CosmosQueryRequestOptions`.
* Se ha actualizado `ChangeFeedProcessorBuilder` al patrón de compilador.
* Se ha actualizado `CosmosPermissionProperties` con el nuevo nombre del contenedor y las API de los recursos secundarios.
* Se han agregado más ejemplos y documentos enriquecidos a `CosmosClientBuilder`. 
* Se han actualizado las API `CosmosDatabase` y `CosmosContainer` con throughputProperties para la compatibilidad con la escalabilidad automática y el autopiloto. 
* Se ha cambiado el nombre de `CosmosClientException` por `CosmosException`. 
* Se ha reemplazado `AccessCondition` y `AccessConditionType` por las API `ifMatchETag()` y `ifNoneMatchETag()`. 
* Se han combinado todos los tipos de `Cosmos*AsyncResponse` y `CosmosResponse` en un tipo único `CosmosResponse`.
* Se ha cambiado el nombre de `CosmosResponseDiagnostics` por `CosmosDiagnostics`.  
* Se ha resumido `FeedResponseDiagnostics` en `CosmosDiagnostics`. 
* Se ha quitado la dependencia `jackson` de azure-cosmos y ahora depende de azure-core. 
* Se ha reemplazado `CosmosKeyCredential` por el tipo `AzureKeyCredential`. 
* Se han agregado las API `ProxyOptions` a `GatewayConnectionConfig`. 
* Se ha actualizado el SDK para que usara el tipo `Instant` en vez de `OffsetDateTime`. 
* Se ha agregado el nuevo tipo de enumeración `OperationKind`. 
* Se ha cambiado el nombre de `FeedOptions` por `QueryRequestOptions`. 
* Se han agregado las API `getETag()` y `getTimestamp()` a los tipos `Cosmos*Properties`. 
* Se ha agregado información de `userAgent` a `CosmosException` y `CosmosDiagnostics`. 
* Se ha actualizado el carácter de nueva línea en `Diagnostics` al carácter de nueva línea del sistema. 
* Se han quitado las API `readAll*` para usar la consulta para seleccionar todas las API en su lugar.
* Se ha agregado la API de retardo estimada `ChangeFeedProcessor`.   
* Se ha agregado compatibilidad del SDK con el aprovisionamiento de rendimiento de la escalabilidad automática y el autopiloto.  
* Se ha reemplazado `ConnectionPolicy` por nuevas configuraciones de conexión. Se han expuesto las API `DirectConnectionConfig` y `GatewayConnectionConfig` a través de `CosmosClientBuilder` para las configuraciones de conexión del modo directo o de puerta de enlace.
* Se han trasladado `JsonSerializable` y `Resource` al paquete de implementación. 
* Se ha agregado la API `contentResponseOnWriteEnabled` a CosmosClientBuilder, que deshabilita el contenido de la respuesta completa en las operaciones de escritura.
* Se han expuesto las API `getETag()` en tipos de respuesta.
* Se ha movido `CosmosAuthorizationTokenResolver` a la implementación. 
* Se ha cambiado el nombre de las API `preferredLocations` y `multipleWriteLocations` por `preferredRegions` y `multipleWriteRegions`. 
* Se han actualizado las versiones `reactor-core` a 3.3.5.RELEASE, `reactor-netty` a 0.9.7.RELEASE y `netty` a 4.1.49.Final. 
* Se ha agregado compatibilidad con `analyticalStoreTimeToLive` en el SDK.     
* `CosmosClientException` amplía `AzureException`. 
* Se han quitado las API `maxItemCount` y `requestContinuationToken` de `FeedOptions` en vez de usar las API `byPage()` de `CosmosPagedFlux` y `CosmosPagedIterable`.
* Se ha presentado `CosmosPermissionProperties` en la superficie pública para las API `Permission`.
* Se ha quitado el tipo `SqlParameterList` y se ha reemplazado por `List`.
* Se han corregido varias fugas de memoria el cliente TCP directo. 
* Se ha agregado compatibilidad con las consultas `DISTINCT`. 
* Se han eliminado las dependencias externas en `fasterxml.uuid, guava, commons-io, commons-collection4, commons-text`.  
* Se ha trasladado `CosmosPagedFlux` y `CosmosPagedIterable` al paquete `utils`. 
* Se ha actualizado Netty a la versión 4.1.45.Final y Project Reactor a la versión 3.3.3.
* Se han actualizado los contratos Rest públicos a las clases `Final`.
* Se ha agregado compatibilidad con diagnósticos avanzados para operaciones de punto.
* Se ha actualizado el paquete a `com.azure.cosmos`
* Se ha agregado el paquete `models` a los contratos modelo y de REST.
* Se ha agregado el paquete `utils` a los tipos `CosmosPagedFlux` y `CosmosPagedIterable`. 
* Se han actualizado las API públicas para usar `Duration` en el SDK.
* Se han agregado todos los contratos de REST al paquete `models`.
* `RetryOptions` cambia su nombre a `ThrottlingRetryOptions`.
* Se han agregado los tipos de paginación `CosmosPagedFlux` & `CosmosPagedIterable` a las API de consulta. 
* Se ha agregado compatibilidad para compartir TransportClient entre varias instancias de CosmosClients con una nueva API en `CosmosClientBuilder#connectionSharingAcrossClientsEnabled(true)`.
* Se han optimizado las consultas quitando la doble serialización o deserialización. 
* Se han optimizado los encabezados de respuesta quitando copias innecesarias hacia adelante y hacia atrás. 
* Se ha optimizado la serialización y deserialización de `ByteBuffer` quitando las creaciones de instancias de cadena intermedias.

#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se ha corregido la excepción del puntero nulo ConnectionPolicy `toString()`.
* Se ha corregido un problema con el análisis de los resultados de la consulta al ordenar los valores por consulta. 
* Se han corregido problemas de pérdida de sockets con el cliente TCP directo.
* Se ha corregido `orderByQuery` con un error continuado en el token.
* Se ha corregido el error `ChangeFeedProcessor` que se producía al controlar las divisiones de particiones y al no encontrar la partición.
* Se ha corregido el error `ChangeFeedProcessor` que se producía al sincronizar las actualizaciones de concesión entre distintos subprocesos.
* Se ha corregido la condición de carrera que provocaba la excepción `ArrayIndexOutOfBound` en StoreReader.

## <a name="faq"></a>Preguntas más frecuentes
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Consulte también
Para más información sobre Cosmos DB, consulte la página del servicio [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).