---
title: Métricas de consulta SQL para la API de SQL de Azure Cosmos DB
description: Aprenda cómo instrumentar y depurar el rendimiento de consultas SQL de las solicitudes de Azure Cosmos DB.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/23/2019
ms.author: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: ec98d194921cd9a7eced06ccee20a3375e8c8a82
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89008699"
---
# <a name="tuning-query-performance-with-azure-cosmos-db"></a>Optimización del rendimiento de consultas con Azure Cosmos DB

Azure Cosmos DB proporciona una [API de SQL para consultar datos](how-to-sql-query.md), sin necesidad de índices de esquema o secundarios. En este artículo se proporciona la siguiente información para los desarrolladores:

* Detalles de alto nivel sobre cómo funciona la ejecución de consultas SQL de Azure Cosmos DB
* Detalles sobre cómo consultar encabezados de solicitud y respuesta y opciones del SDK de cliente
* Sugerencias y procedimientos recomendados para el rendimiento de consultas
* Ejemplos de cómo usar estadísticas de ejecución de SQL para depurar el rendimiento de las consultas

## <a name="about-sql-query-execution"></a>Acerca de la ejecución de consultas SQL

En Azure Cosmos DB, se almacenan los datos en contenedores, que pueden crecer hasta cualquier [tamaño de almacenamiento o rendimiento de las solicitudes](partition-data.md). Azure Cosmos DB escala los datos de forma fluida entre particiones físicas en segundo plano para controlar el crecimiento de los datos o el aumento del rendimiento aprovisionado. Puede emitir consultas SQL a cualquier contenedor mediante la API de REST o uno de los [SDK de SQL](sql-api-sdk-dotnet.md) compatibles.

Una breve introducción a la creación de particiones: se define una clave de partición como "ciudad", que determina la forma en que los datos se dividen entre particiones físicas. Los datos que pertenecen a una única clave de partición (por ejemplo, "city" == "Seattle") se almacenan en una partición física, pero normalmente una única partición física tiene varias claves de partición. Cuando una partición alcanza su tamaño de almacenamiento, el servicio divide de forma fluida la partición en dos nuevas particiones y divide la clave de partición uniformemente entre estas particiones. Como las particiones son transitorias, las API usan una abstracción de un "intervalo de claves de partición", que indica los intervalos de valores hash de clave de partición. 

Cuando se emite una consulta a Azure Cosmos DB, el SDK realiza estos pasos lógicos:

* Analiza la consulta SQL para determinar el plan de ejecución de consultas. 
* Si la consulta incluye un filtro con la clave de partición, como `SELECT * FROM c WHERE c.city = "Seattle"`, se enruta a una única partición. Si la consulta no tiene un filtro por la clave de partición, se ejecuta entonces en todas las particiones y los resultados se combinan en el cliente.
* La consulta se ejecuta dentro de cada partición en serie o en paralelo, en función de la configuración del cliente. En cada partición, la consulta podría realizar uno o varios viajes en función de la complejidad de la consulta, el tamaño de página configurado y el rendimiento aprovisionado de la colección. Cada ejecución devuelve el número de [unidades de solicitud](request-units.md) que se consumen en la ejecución de consultas y, opcionalmente, las estadísticas de ejecución de consultas. 
* El SDK realiza un resumen de los resultados de la consulta entre particiones. Por ejemplo, si la consulta implica una cláusula ORDER BY entre particiones, los resultados de particiones individuales se ordenan mediante combinación para devolver resultados en orden clasificado globalmente. Si la consulta es una agregación como `COUNT`, los recuentos de particiones individuales se suman para generar el recuento total.

Los SDK ofrecen diversas opciones para la ejecución de consultas. Por ejemplo, en .NET estas opciones están disponibles en la clase `FeedOptions`. En la tabla siguiente se describen estas opciones y de qué manera pueden afectar el tiempo de ejecución de consulta. 

| Opción | Descripción |
| ------ | ----------- |
| `EnableCrossPartitionQuery` | Se debe establecer en true para cualquier consulta que se deba ejecutar entre más de una partición. Esta es una marca explícita que le permite realizar compensaciones conscientes de rendimiento durante el tiempo de desarrollo. |
| `EnableScanInQuery` | Se debe establecer en true si ha decidido no indexar, pero quiere ejecutar la consulta de todos modos mediante una exploración. Solo es aplicable si la indexación para la ruta de filtro solicitada está deshabilitada. | 
| `MaxItemCount` | El número máximo de elementos que se devolverán por recorrido de ida y vuelta al servidor. Si se establece en -1, puede dejar que el servidor administre el número de elementos. O bien, puede reducir este valor para recuperar únicamente un pequeño número de elementos por recorrido de ida y vuelta. 
| `MaxBufferedItemCount` | Esta es una opción de cliente y se usa para limitar el consumo de memoria al usar una cláusula ORDER BY entre particiones. Un valor más alto ayuda a reducir la latencia de ordenación entre particiones. |
| `MaxDegreeOfParallelism` | Obtiene o establece el número de operaciones simultáneas en el cliente durante la ejecución de consultas en paralelo en el servicio de base de datos de Azure Cosmos. Un valor de propiedad positivo limita el número de operaciones simultáneas al valor establecido. Si se establece en un valor menor que 0, el sistema decide automáticamente el número de operaciones simultáneas que se ejecutarán. |
| `PopulateQueryMetrics` | Permite el registro detallado de estadísticas del tiempo empleado en diversas fases de la ejecución de consultas, como tiempo de compilación, hora de ciclo del índice y tiempo de carga de documentos. Puede compartir la salida de las estadísticas de consulta con el soporte técnico de Azure con el fin de diagnosticar problemas de rendimiento de las consultas. |
| `RequestContinuation` | Puede reanudar la ejecución de consultas pasando el token de continuación opaco devuelto por cualquier consulta. El token de continuación encapsula todo el estado necesario para la ejecución de consultas. |
| `ResponseContinuationTokenLimitInKb` | Puede limitar el tamaño máximo del token de continuación devuelto por el servidor. Podría ser necesario establecer este valor si el host de la aplicación tiene límites en cuanto al tamaño del encabezado de respuesta. Si se establece, podría aumentar la duración total y las RU usadas para la consulta.  |

Por ejemplo, tomemos una consulta de ejemplo sobre una clave de partición solicitada en una colección con `/city` como clave de partición y aprovisionada con 100 000 RU/s de rendimiento. Esta consulta se solicita mediante `CreateDocumentQuery<T>` en .NET, como se muestra a continuación:

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        PopulateQueryMetrics = true, 
        MaxItemCount = -1, 
        MaxDegreeOfParallelism = -1, 
        EnableCrossPartitionQuery = true 
    }).AsDocumentQuery();

FeedResponse<dynamic> result = await query.ExecuteNextAsync();
```

El fragmento de código del SDK que se muestra arriba, corresponde a la siguiente solicitud de API de REST:

```
POST https://arramacquerymetrics-westus.documents.azure.com/dbs/db/colls/sample/docs HTTP/1.1
x-ms-continuation: 
x-ms-documentdb-isquery: True
x-ms-max-item-count: -1
x-ms-documentdb-query-enablecrosspartition: True
x-ms-documentdb-query-parallelizecrosspartitionquery: True
x-ms-documentdb-query-iscontinuationexpected: True
x-ms-documentdb-populatequerymetrics: True
x-ms-date: Tue, 27 Jun 2017 21:52:18 GMT
authorization: type%3dmaster%26ver%3d1.0%26sig%3drp1Hi83Y8aVV5V6LzZ6xhtQVXRAMz0WNMnUuvriUv%2b4%3d
x-ms-session-token: 7:8,6:2008,5:8,4:2008,3:8,2:2008,1:8,0:8,9:8,8:4008
Cache-Control: no-cache
x-ms-consistency-level: Session
User-Agent: documentdb-dotnet-sdk/1.14.1 Host/32-bit MicrosoftWindowsNT/6.2.9200.0
x-ms-version: 2017-02-22
Accept: application/json
Content-Type: application/query+json
Host: arramacquerymetrics-westus.documents.azure.com
Content-Length: 52
Expect: 100-continue

{"query":"SELECT * FROM c WHERE c.city = 'Seattle'"}
```

Cada página de ejecución de una consulta corresponde a una API de REST `POST` con el encabezado `Accept: application/query+json` y la consulta SQL en el cuerpo. Cada consulta realiza uno o varios recorridos de ida y vuelta al servidor, y el token `x-ms-continuation` se muestra entre el cliente y el servidor para reanudar la ejecución. Las opciones de configuración en FeedOptions se pasan al servidor en forma de encabezados de solicitud. Por ejemplo, `MaxItemCount` corresponde a `x-ms-max-item-count`. 

La solicitud devuelve la respuesta siguiente (truncada para que se entienda mejor):

```
HTTP/1.1 200 Ok
Cache-Control: no-store, no-cache
Pragma: no-cache
Transfer-Encoding: chunked
Content-Type: application/json
Server: Microsoft-HTTPAPI/2.0
Strict-Transport-Security: max-age=31536000
x-ms-last-state-change-utc: Tue, 27 Jun 2017 21:01:57.561 GMT
x-ms-resource-quota: documentSize=10240;documentsSize=10485760;documentsCount=-1;collectionSize=10485760;
x-ms-resource-usage: documentSize=1;documentsSize=884;documentsCount=2000;collectionSize=1408;
x-ms-item-count: 2000
x-ms-schemaversion: 1.3
x-ms-alt-content-path: dbs/db/colls/sample
x-ms-content-path: +9kEANVq0wA=
x-ms-xp-role: 1
x-ms-documentdb-query-metrics: totalExecutionTimeInMs=33.67;queryCompileTimeInMs=0.06;queryLogicalPlanBuildTimeInMs=0.02;queryPhysicalPlanBuildTimeInMs=0.10;queryOptimizationTimeInMs=0.00;VMExecutionTimeInMs=32.56;indexLookupTimeInMs=0.36;documentLoadTimeInMs=9.58;systemFunctionExecuteTimeInMs=0.00;userFunctionExecuteTimeInMs=0.00;retrievedDocumentCount=2000;retrievedDocumentSize=1125600;outputDocumentCount=2000;writeOutputTimeInMs=18.10;indexUtilizationRatio=1.00
x-ms-request-charge: 604.42
x-ms-serviceversion: version=1.14.34.4
x-ms-activity-id: 0df8b5f6-83b9-4493-abda-cce6d0f91486
x-ms-session-token: 2:2008
x-ms-gatewayversion: version=1.14.33.2
Date: Tue, 27 Jun 2017 21:59:49 GMT
```

Los encabezados de respuesta principales devueltos por la consulta incluyen los siguientes:

| Opción | Descripción |
| ------ | ----------- |
| `x-ms-item-count` | El número de elementos devueltos en la respuesta. Esto depende del valor de `x-ms-max-item-count` proporcionado, del número de elementos que se pueden incluir en el tamaño máximo de la carga de respuesta, del rendimiento aprovisionado y del tiempo de ejecución de consultas. |  
| `x-ms-continuation:` | El token de continuación para reanudar la ejecución de la consulta, si hay más resultados disponibles. | 
| `x-ms-documentdb-query-metrics` | Las estadísticas de consulta para la ejecución. Es una cadena delimitada que contiene las estadísticas del tiempo empleado en las distintas fases de ejecución de consultas. Se devuelve si `x-ms-documentdb-populatequerymetrics` está establecido en `True`. | 
| `x-ms-request-charge` | El número de [unidades de solicitud](request-units.md) usadas por la consulta. | 

Para más información sobre los encabezados de solicitud y las opciones de la API de REST, consulte [Querying resources using the REST API](/rest/api/cosmos-db/querying-cosmosdb-resources-using-the-rest-api) (Consulta de recursos mediante la API de REST).

## <a name="best-practices-for-query-performance"></a>Procedimientos recomendados para el rendimiento de consultas
A continuación se muestran los factores más comunes que afectan al rendimiento de consultas de Azure Cosmos DB. Cada uno de estos temas se tratará con mayor detalle en este artículo.

| Factor | Sugerencia | 
| ------ | -----| 
| Rendimiento aprovisionado | Mida la RU por consulta y asegúrese de que tiene el rendimiento aprovisionado necesario para sus consultas. | 
| Creación de particiones y claves de partición | Dé prioridad a las consultas con el valor de clave de partición de baja latencia en la cláusula de filtro. |
| SDK y opciones de consulta | Siga los procedimientos recomendados del SDK, como las opciones de conectividad directa y optimización de la ejecución de consultas de cliente. |
| Latencia de red | Justifique la sobrecarga de red en la medición, y use API de hospedaje múltiple para leer en la región más cercana. |
| Directiva de indexación | Asegúrese de que dispone de las rutas y directivas de indexación necesarias para la consulta. |
| Métricas de ejecución de consultas | Analice las métricas de ejecución de consultas para identificar posibles reescrituras de formas de datos y consultas.  |

### <a name="provisioned-throughput"></a>Rendimiento aprovisionado
En Cosmos DB, se crean contenedores de datos, cada uno con un rendimiento reservado expresado en unidades de solicitud (RU) por segundo. Una lectura de un documento de 1 KB es 1 RU, y cada operación (incluidas las consultas) se normaliza a un número fijo de RU según su complejidad. Por ejemplo, si tiene 1000 RU/s aprovisionadas en su contenedor y tiene una consulta como `SELECT * FROM c WHERE c.city = 'Seattle'` que consume 5 RU, puede realizar entonces (1000 RU/s)/(5 RU/consulta) = 200 consultas/s de estas por segundo. 

Si envía más de 200 consultas/s, el servicio comienza a limitar la tasa de solicitudes entrantes por encima de 200/s. Los SDK controlan automáticamente este caso mediante la realización de un retroceso/reintento; por lo tanto, podría advertir una latencia más alta con estas consultas. El aumento del rendimiento aprovisionado al valor necesario mejora la latencia de las consultas y el rendimiento. 

Para más información sobre las unidades de solicitud, consulte [Unidades de solicitud](request-units.md).

### <a name="partitioning-and-partition-keys"></a>Creación de particiones y claves de partición
Con Azure Cosmos DB, las consultas se realizan normalmente en el orden siguiente: desde la más rápida o eficiente a la más lenta o menos eficiente. 

* GET en una única clave de partición y una clave de elemento
* Consulta con una cláusula de filtro en una única clave de partición
* Consulta sin una cláusula de filtro de igualdad o intervalo en cualquier propiedad
* Sin filtros de consulta

Las consultas que necesitan consultar todas las particiones necesitan una mayor latencia y pueden consumir un mayor número de RU. Puesto que cada partición tiene indexación automática en todas las propiedades, la consulta se puede atender eficazmente desde el índice en este caso. Puede acelerar las consultas que abarcan particiones mediante las opciones de paralelismo.

Para aprender más sobre la creación de particiones y las claves de particiones, consulte [Creación de particiones en Azure Cosmos DB](partition-data.md).

### <a name="sdk-and-query-options"></a>SDK y opciones de consulta
Consulte [Sugerencias de rendimiento](performance-tips.md) y [Prueba de rendimiento](performance-testing.md) para saber cómo obtener el mejor rendimiento del cliente de Azure Cosmos DB. Esto incluye el uso de los SDK más recientes, la configuración de opciones específicas de la plataforma, como el número predeterminado de conexiones, la frecuencia de recolección de elementos no utilizados y el uso de opciones de conectividad ligera, como Direct/TCP. 


#### <a name="max-item-count"></a>Recuento máximo de elementos
Para las consultas, el valor de `MaxItemCount` puede tener un impacto significativo en el momento de realizar consultas globales. Cada recorrido de ida y vuelta al servidor devolverá un número de elementos inferior al especificado en `MaxItemCount` (con un valor predeterminado de 100 elementos). Si se establece en un valor más alto (-1 es el máximo y el número recomendado) mejorará la duración total de la consulta limitando el número de recorridos de ida y vuelta entre el servidor y cliente, especialmente en aquellas consultas con grandes conjuntos de resultados.

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        MaxItemCount = -1, 
    }).AsDocumentQuery();
```

#### <a name="max-degree-of-parallelism"></a>Grado máximo de paralelismo
Para las consultas, ajuste `MaxDegreeOfParallelism` para identificar las mejores configuraciones para su aplicación, en especial si realiza consultas entre particiones (sin un filtro en el valor de clave de partición). `MaxDegreeOfParallelism` controla el número máximo de tareas en paralelo, es decir, el número máximo de particiones que se van a visitar en paralelo. 

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        MaxDegreeOfParallelism = -1, 
        EnableCrossPartitionQuery = true 
    }).AsDocumentQuery();
```

Supongamos que
* D. = número máximo predeterminado de tareas en paralelo (= número total de procesadores en el equipo cliente)
* P = número máximo especificado por el usuario de tareas en paralelo
* N = número de particiones que se necesita visitar para responder a una consulta

A continuación se muestran las distintas implicaciones de cómo se comportarán las consultas en paralelo para valores diferentes de P.
* (P == 0) => modo serie
* (P == 1) => un máximo de una tarea
* (P > 1) => mínimo de tareas en paralelo (P, N) 
* (P < 1) => mínimo de tareas en paralelo (N, D)

Para acceder a notas de versión de SDK y conocer detalles sobre las clases y los métodos implementados, consulte [SDK de SQL](sql-api-sdk-dotnet.md)

### <a name="network-latency"></a>Latencia de red
Para saber cómo configurar la distribución global y conectarse a la región más cercana, consulte [Distribución global de Azure Cosmos DB](tutorial-global-distribution-sql-api.md). La latencia de red tiene un impacto significativo en el rendimiento de consultas cuando es necesario realizar varios recorridos de ida y vuelta o recuperar un gran conjunto de resultados de la consulta. 

La sección sobre métricas de ejecución de consultas explica cómo recuperar el tiempo de ejecución del servidor de consultas ( `totalExecutionTimeInMs`), de modo que pueda diferenciar entre el tiempo invertido en la ejecución de la consulta y el tiempo empleado en el tránsito de la red.

### <a name="indexing-policy"></a>Directiva de indexación
Para información sobre las rutas de indexación, las clases y los modos y cómo afectan a la ejecución de consultas, consulte [Configuración de la directiva de indexación](index-policy.md). De forma predeterminada, la directiva de indexación usa la indexación de valores hash para las cadenas, que resulta eficaz para las consultas de igualdad, aunque no así para las consultas de intervalo y las consultas ORDER BY. Si tiene consultas de intervalo para las cadenas, se recomienda especificar el tipo de índice de intervalo para todas las cadenas. 

De forma predeterminada, Azure Cosmos DB aplicará la indexación automática a todos los datos. Para escenarios de inserción de alto rendimiento, considere la posibilidad de excluir rutas de acceso ya que esto reducirá el costo de RU para cada operación de inserción. 

## <a name="query-execution-metrics"></a>Métricas de ejecución de consultas
Para obtener métrica detalladas sobre la ejecución de consultas, pase el encabezado opcional `x-ms-documentdb-populatequerymetrics` (`FeedOptions.PopulateQueryMetrics` en el SDK de .NET). El valor devuelto en `x-ms-documentdb-query-metrics` tiene los siguientes pares de clave-valor pensados para la solución avanzada de problemas de la ejecución de consultas. 

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        PopulateQueryMetrics = true, 
    }).AsDocumentQuery();

FeedResponse<dynamic> result = await query.ExecuteNextAsync();

// Returns metrics by partition key range Id
IReadOnlyDictionary<string, QueryMetrics> metrics = result.QueryMetrics;

```

| Métrica | Unidad | Descripción | 
| ------ | -----| ----------- |
| `totalExecutionTimeInMs` | milisegundos | Tiempo de ejecución de consulta | 
| `queryCompileTimeInMs` | milisegundos | Tiempo de compilación de consulta  | 
| `queryLogicalPlanBuildTimeInMs` | milisegundos | Tiempo en compilar el plan lógico de consultas | 
| `queryPhysicalPlanBuildTimeInMs` | milisegundos | Tiempo en compilar el plan físico de consulta | 
| `queryOptimizationTimeInMs` | milisegundos | Tiempo invertido en optimizar la consulta | 
| `VMExecutionTimeInMs` | milisegundos | Tiempo invertido en el tiempo de ejecución de consulta | 
| `indexLookupTimeInMs` | milisegundos | Tiempo invertido en la capa física de índice | 
| `documentLoadTimeInMs` | milisegundos | Tiempo invertido en la carga de documentos  | 
| `systemFunctionExecuteTimeInMs` | milisegundos | Tiempo total invertido en ejecutar funciones del sistema (integradas), en milisegundos  | 
| `userFunctionExecuteTimeInMs` | milisegundos | Tiempo total invertido en ejecutar funciones definidas por el usuario, en milisegundos | 
| `retrievedDocumentCount` | count | Número total de documentos recuperados  | 
| `retrievedDocumentSize` | bytes | Tamaño total de los documentos recuperados, en bytes  | 
| `outputDocumentCount` | count | Número de documentos de salida | 
| `writeOutputTimeInMs` | milisegundos | Tiempo de ejecución de consultas, en milisegundos | 
| `indexUtilizationRatio` | relación (<= 1) | Relación entre el número de documentos coincidentes mediante el filtro y el número de documentos cargados  | 

Los SDK del cliente pueden realizar internamente varias operaciones de consulta para atender la consulta dentro de cada partición. El cliente realiza más de una llamada por partición si los resultados totales superan `x-ms-max-item-count`, si la consulta supera el rendimiento aprovisionado de la partición, si la carga de consultas alcanza el tamaño máximo por página o si la consulta alcanza el límite de tiempo de espera asignado al sistema. Cada ejecución de consultas parcial devuelve un valor `x-ms-documentdb-query-metrics` para esa página. 

Estos son algunos ejemplos de consultas y cómo interpretar algunas de las métricas devueltas en la ejecución de consultas: 

| Consultar | Métrica de ejemplo | Descripción | 
| ------ | -----| ----------- |
| `SELECT TOP 100 * FROM c` | `"RetrievedDocumentCount": 101` | El número de documentos recuperados es 100 + 1 para coincidir con la cláusula TOP. El tiempo de la consulta se invierte mayormente en `WriteOutputTime` y `DocumentLoadTime` dado que es un examen. | 
| `SELECT TOP 500 * FROM c` | `"RetrievedDocumentCount": 501` | RetrievedDocumentCount es ahora mayor (500 + 1 para coincidir con la cláusula TOP). | 
| `SELECT * FROM c WHERE c.N = 55` | `"IndexLookupTime": "00:00:00.0009500"` | 0,9 ms se invierten en IndexLookupTime en una búsqueda de claves, porque es una búsqueda de índice en `/N/?`. | 
| `SELECT * FROM c WHERE c.N > 55` | `"IndexLookupTime": "00:00:00.0017700"` | Un poco más de tiempo (1,7 ms) se invierte en IndexLookupTime durante un examen de intervalo, porque es una búsqueda de índice en `/N/?`. | 
| `SELECT TOP 500 c.N FROM c` | `"IndexLookupTime": "00:00:00.0017700"` | El mismo tiempo se invierte en `DocumentLoadTime` que en las consultas anteriores, pero menos `WriteOutputTime` porque solo se proyecta una propiedad. | 
| `SELECT TOP 500 udf.toPercent(c.N) FROM c` | `"UserDefinedFunctionExecutionTime": "00:00:00.2136500"` | Unos 213 ms se invierten en `UserDefinedFunctionExecutionTime` al ejecutar el UDF en cada valor de `c.N`. |
| `SELECT TOP 500 c.Name FROM c WHERE STARTSWITH(c.Name, 'Den')` | `"IndexLookupTime": "00:00:00.0006400", "SystemFunctionExecutionTime": "00:00:00.0074100"` | Aproximadamente 0,6 ms se invierten en `IndexLookupTime` en `/Name/?`. La mayoría del tiempo de ejecución de consulta (~ 7 ms) se invierte en `SystemFunctionExecutionTime`. |
| `SELECT TOP 500 c.Name FROM c WHERE STARTSWITH(LOWER(c.Name), 'den')` | `"IndexLookupTime": "00:00:00", "RetrievedDocumentCount": 2491,  "OutputDocumentCount": 500` | La consulta se realiza como un examen porque se emplea `LOWER`, y se devuelven 500 de los 2491 documentos recuperados. |


## <a name="next-steps"></a>Pasos siguientes
* Para conocer los operadores de consulta y palabras clave SQL admitidos, consulte [Consultas SQL](sql-query-getting-started.md). 
* Para aprender más sobre las unidades de solicitud, consulte [Unidades de solicitud](request-units.md).
* Para aprender más sobre la directiva de indexación, consulte [Directiva de indexación](index-policy.md) 


