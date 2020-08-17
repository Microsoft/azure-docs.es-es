---
title: Sugerencias de rendimiento para el SDK asincrónico de Azure Cosmos DB para Java v2
description: Conozca las opciones de configuración de cliente para mejorar el rendimiento de las bases de datos de Azure Cosmos para el SDK asincrónico para Java v2.
author: anfeldma-ms
ms.service: cosmos-db
ms.devlang: java
ms.topic: how-to
ms.date: 05/11/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: d925c1387a408d38eb7974a01ebf3ce3386b7e58
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067617"
---
# <a name="performance-tips-for-azure-cosmos-db-async-java-sdk-v2"></a>Sugerencias de rendimiento para el SDK asincrónico de Azure Cosmos DB para Java v2

> [!div class="op_single_selector"]
> * [SDK para Java v4](performance-tips-java-sdk-v4-sql.md)
> * [Versión 2 del SDK de Java asincrónico](performance-tips-async-java.md)
> * [SDK de Java v2 sincrónico](performance-tips-java.md)
> * [.NET SDK v3](performance-tips-dotnet-sdk-v3-sql.md)
> * [SDK de .NET, versión 2](performance-tips.md)


> [!IMPORTANT]  
> *No* se trata de la versión de SDK de Java de Azure Cosmos DB más reciente. Debe actualizar el proyecto al [SDK de Azure Cosmos DB para Java v4](sql-api-sdk-java-v4.md) y luego leer la [guía de sugerencias de rendimiento](performance-tips-java-sdk-v4-sql.md) del SDK de Azure Cosmos DB para Java v4. Siga las instrucciones que se indican en la guía [Migración al SDK de Azure Cosmos DB para Java v4](migrate-java-v4-sdk.md) y la guía [Reactor frente a RxJava](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-rxjava-guide.md) para realizar la actualización. 
> 
> Las sugerencias de rendimiento de este artículo son solo para el SDK asincrónico de Azure Cosmos DB para Java v2. Consulte [Notas de la versión](sql-api-sdk-async-java.md) del SDK asincrónico de Azure Cosmos DB para Java v2, el [repositorio de Maven](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) y la [guía de solución de problemas](troubleshoot-java-async-sdk.md) del SDK de Azure Cosmos DB para Java v2 asincrónico para más información.
>

Azure Cosmos DB es una base de datos distribuida rápida y flexible que se escala sin problemas con una latencia y un rendimiento garantizados. No es necesario realizar cambios de arquitectura importantes ni escribir código complejo para escalar la base de datos con Azure Cosmos DB. Escalar y reducir verticalmente es tan sencillo como realizar una única llamada API o una llamada al método SDK. Sin embargo, como el acceso a Azure Cosmos DB se realiza mediante llamadas de red, puede realizar optimizaciones en el lado cliente para conseguir un rendimiento máximo al usar el [SDK asincrónico de Azure Cosmos DB para Java v2](sql-api-sdk-async-java.md).

Así que si se está preguntando "¿Cómo puedo mejorar el rendimiento de la base de datos?", tenga en cuenta las opciones siguientes:

## <a name="networking"></a>Redes

* **Modo de conexión: uso del modo directo**
    
  El modo en que un cliente se conecta a Azure Cosmos DB tiene implicaciones importantes sobre el rendimiento, especialmente en términos de la latencia en el lado cliente. *ConnectionMode* es una opción de configuración clave disponible para configurar el cliente *ConnectionPolicy*. En el caso del SDK asincrónico de Azure Cosmos DB para Java v2, los dos modos de conexión disponibles son:  
      
  * [Puerta de enlace (valor predeterminado)](/java/api/com.microsoft.azure.cosmosdb.connectionmode)  
  * [Direct](/java/api/com.microsoft.azure.cosmosdb.connectionmode)
  
  El modo de puerta de enlace se admite en todas las plataformas de SDK y es la opción predeterminada configurada. Si las aplicaciones se ejecutan dentro de una red corporativa con restricciones de firewall estrictas, el modo de puerta de enlace es la mejor opción, ya que utiliza el puerto HTTPS estándar y un único punto de conexión.   La desventaja para el rendimiento, sin embargo, es que el modo de puerta de enlace implica un salto de red adicional cada vez que se leen o escriben datos en Azure Cosmos DB. Por este motivo, el modo directo ofrece mejor rendimiento debido al número menor de saltos de red.
  
  *ConnectionMode* se configura durante la construcción de la instancia de *DocumentClient* con el parámetro *ConnectionPolicy*.

### <a name="async-java-sdk-v2-maven-commicrosoftazureazure-cosmosdb"></a><a id="asyncjava2-connectionpolicy"></a>SDK de Java v2 asincrónico (Maven com.microsoft.azure::azure-cosmosdb)

```java
    public ConnectionPolicy getConnectionPolicy() {
        ConnectionPolicy policy = new ConnectionPolicy();
        policy.setConnectionMode(ConnectionMode.Direct);
        policy.setMaxPoolSize(1000);
        return policy;
    }

    ConnectionPolicy connectionPolicy = new ConnectionPolicy();
    DocumentClient client = new DocumentClient(HOST, MASTER_KEY, connectionPolicy, null);
```

* **Colocación de los clientes en la misma región de Azure para aumentar el rendimiento**

  Cuando sea posible, coloque las aplicaciones que llaman a Azure Cosmos DB en la misma región que la base de datos de Azure Cosmos. Para obtener una comparación aproximada, las llamadas a Azure Cosmos DB en la misma región se realizan en menos de 1 o 2 ms, pero la latencia entre las costas este y oeste de Estados Unidos es mayor de 50 ms. Esta latencia podría variar de una solicitud a otra, según la ruta tomada por la solicitud cuando pasa del cliente al límite del centro de datos de Azure. Para conseguir la menor latencia posible, asegúrese de que la aplicación que llama se encuentra en la misma región de Azure que el punto de conexión de Azure Cosmos DB aprovisionado. Para obtener una lista de regiones disponibles, consulte [Regiones de Azure](https://azure.microsoft.com/regions/#services).

  :::image type="content" source="./media/performance-tips/same-region.png" alt-text="Ilustración de la directiva de conexión de Azure Cosmos DB" border="false":::

## <a name="sdk-usage"></a>Uso del SDK

* **Instalación del SDK más reciente**

  Los SDK de Azure Cosmos DB se mejoran constantemente para proporcionar el mejor rendimiento. Consulte las páginas de [Notas de la versión](sql-api-sdk-async-java.md) del SDK asincrónico de Azure Cosmos DB para Java v2 para determinar el SDK más reciente y revisar las mejoras.

* **Uso de un cliente de Azure Cosmos DB singleton para aumentar la duración de la aplicación**

  Cada instancia de AsyncDocumentClient está protegida frente a amenazas y realiza de manera eficiente la administración de las conexiones y el almacenamiento en caché de las direcciones. Para permitir la administración eficiente de las conexiones y un rendimiento mejor mediante AsyncDocumentClient, se recomienda usar una única instancia de AsyncDocumentClient por AppDomain durante la vigencia de la aplicación.

* **Optimización de ConnectionPolicy**

  De forma predeterminada, las solicitudes del modo directo de Cosmos DB se realizan sobre TCP cuando se usa el SDK asincrónico de Azure Cosmos DB para Java v2. Internamente, el SDK usa una arquitectura de modo directo especial para administrar dinámicamente los recursos de red y obtener el mejor rendimiento.

  En el SDK asincrónico de Azure Cosmos DB para Java v2, el modo directo es la mejor opción para mejorar el rendimiento de la base de datos con la mayoría de las cargas de trabajo. 

  * ***Introducción al modo directo***

  :::image type="content" source="./media/performance-tips-async-java/rntbdtransportclient.png" alt-text="Ilustración de la arquitectura del modo directo" border="false":::
  
  La arquitectura del lado cliente empleada en el modo directo permite un uso predecible de la red y el acceso multiplexado a las réplicas de Azure Cosmos DB. En el diagrama anterior se muestra cómo el modo directo enruta las solicitudes de cliente a las réplicas en el back-end de Cosmos DB. La arquitectura del modo directo asigna hasta 10 **canales** en el lado cliente por réplica de base de datos. Un canal es una conexión TCP precedida por un búfer de solicitudes, que tiene 30 solicitudes de profundidad. El **punto de conexión de servicio** de la réplica asigna dinámicamente según sea necesario los canales que pertenecen a una réplica. Cuando el usuario emite una solicitud en el modo directo, **TransportClient** enruta la solicitud al punto de conexión de servicio adecuado en función de la clave de partición. La **cola de solicitudes** almacena en búfer las solicitudes antes del punto de conexión de servicio.

  * ***Opciones de configuración de ConnectionPolicy para el modo directo***

    Como primer paso, utilice las opciones de configuración recomendadas a continuación. Póngase en contacto con el [equipo de Azure Cosmos DB](mailto:CosmosDBPerformanceSupport@service.microsoft.com) si tiene problemas con este tema específico.

    Si usa Azure Cosmos DB como una base de datos de referencia (es decir, la base de datos se utiliza para muchas operaciones de lectura de punto y pocas operaciones de escritura), puede ser aceptable establecer *idleEndpointTimeout* en 0 (es decir, sin tiempo de espera).


    | Opción de configuración       | Valor predeterminado    |
    | :------------------:       | :-----:    |
    | bufferPageSize             | 8192       |
    | connectionTimeout          | "PT1M"     |
    | idleChannelTimeout         | "PT0S"     |
    | idleEndpointTimeout        | "PT1M10S"  |
    | maxBufferCapacity          | 8388608    |
    | maxChannelsPerEndpoint     | 10         |
    | maxRequestsPerChannel      | 30         |
    | receiveHangDetectionTime   | "PT1M5S"   |
    | requestExpiryInterval      | "PT5S"     |
    | requestTimeout             | "PT1M"     |
    | requestTimerResolution     | "PT0.5S"   |
    | sendHangDetectionTime      | "PT10S"    |
    | shutdownTimeout            | "PT15S"    |

* ***Sugerencias de programación para el modo directo***

  Consulte el artículo [Solución de problemas](troubleshoot-java-async-sdk.md) del SDK asincrónico de Azure Cosmos DB para Java v2 como base de referencia para resolver cualquier problema del SDK.
  
  Algunas sugerencias de programación importantes al usar el modo directo:
  
  * **Usar multithreading en la aplicación para una transferencia de datos TCP eficaz**: después de realizar una solicitud, la aplicación debe suscribirse para recibir datos en otro subproceso. Si no lo hace, se fuerza una operación "dúplex medio" inesperada y las solicitudes subsiguientes se bloquean esperando la respuesta de la solicitud anterior.
  
  * **Llevar a cabo las cargas de trabajo de proceso intensivo en un subproceso dedicado**: por motivos similares a la sugerencia anterior, es mejor colocar las operaciones como el procesamiento de datos complejos en un subproceso independiente. Una solicitud que extrae datos de otro almacén de datos (por ejemplo, si el subproceso emplea Azure Cosmos DB y almacenes de datos de Spark simultáneamente) puede experimentar una mayor latencia y se recomienda generar un subproceso adicional que espere una respuesta del otro almacén de datos.
  
    * Netty administra la E/S de red subyacente en el SDK asincrónico de Azure Cosmos DB para Java v2; consulte [Sugerencias para evitar patrones de codificación que bloquean los subprocesos de E/S de Netty](troubleshoot-java-async-sdk.md#invalid-coding-pattern-blocking-netty-io-thread).
  
  * **Modelado de datos**: el Acuerdo de Nivel de Servicio de Azure Cosmos DB supone que el tamaño del documento es inferior a 1 KB. La optimización del modelo de datos y la programación para favorecer un tamaño de documento más pequeño normalmente dará lugar a una disminución de la latencia. Si va a necesitar almacenamiento y recuperación de documentos de más de 1 KB, el enfoque recomendado es vincular los documentos a datos de Azure Blob Storage.

* **Ajuste de consultas paralelas en colecciones particionadas**

  El SDK asincrónico de Azure Cosmos DB para Java v2 admite consultas en paralelo que permiten consultar una colección con particiones en paralelo. Para obtener más información, consulte [ejemplos de código](https://github.com/Azure/azure-cosmosdb-java/tree/master/examples/src/test/java/com/microsoft/azure/cosmosdb/rx/examples) relacionados para trabajar con los SDK. Las consultas paralelas están diseñadas para mejorar la latencia y el rendimiento de la consulta en todos sus homólogos seriales.

  * ***Ajuste de setMaxDegreeOfParallelism\:***
    
    Las consultas paralelas funcionan creando consultas en varias particiones en paralelo. Sin embargo, los datos de una recopilación con particiones individual se capturan en serie con respecto a la consulta. Por lo tanto, use el parámetro setMaxDegreeOfParallelism para establecer el número de particiones que tienen la máxima probabilidad de conseguir el mejor rendimiento de consulta, siempre y cuando el resto de las demás condiciones del sistema permanezcan invariables. Si no conoce el número de particiones, puede usar setMaxDegreeOfParallelism para establecer un número alto y el sistema elegirá el mínimo (número de particiones, entrada proporcionada por el usuario) como el grado máximo de paralelismo.

    Es importante tener en cuenta que las consultas en paralelo producen los mejores beneficios si los datos se distribuyen uniformemente entre todas las particiones con respecto a la consulta. Si la colección con particiones está dividida de tal forma que todos, o la mayoría de los datos, devueltos por una consulta se concentran en algunas particiones (una partición en el peor de los casos), entonces el rendimiento de la consulta se vería afectada por cuellos de botella debido a esas particiones.

  * ***Ajuste de setMaxBufferedItemCount\:***
    
    Las consultas en paralelo están diseñadas para capturar previamente los resultados mientras el cliente procesa el lote actual de resultados. La captura previa ayuda a mejorar la latencia general de una consulta. setMaxBufferedItemCount limita el número de resultados capturados previamente. Establece setMaxBufferedItemCount en el número esperado de resultados devueltos (o un número más alto) permite que la consulta reciba el máximo beneficio de la captura previa.

    La captura previa funciona de la misma manera con independencia de MaxDegreeOfParallelism, y hay un único búfer para los datos de todas las particiones.

* **Implementación del retroceso según intervalos de getRetryAfterInMilliseconds**

  Durante las pruebas de rendimiento, debe aumentar la carga hasta que se limite una tasa de solicitudes pequeña. Si se limita, la aplicación cliente debe retroceder de acuerdo con el intervalo de reintento que el servidor especificó. Respetar el retroceso garantiza que dedica una cantidad de tiempo mínima de espera entre reintentos.

* **Escalado horizontal de la carga de trabajo de cliente**

  Si va a realizar pruebas en niveles de alto rendimiento (>50 000 RU/s), la aplicación cliente puede volverse un cuello de botella debido a que la máquina limita el uso de CPU o de la red. Si llega a este punto, puede seguir insertando la cuenta de Azure Cosmos DB mediante la escala horizontal de las aplicaciones cliente en varios servidores.

* **Uso del direccionamiento basado en nombres**

  Use el direccionamiento basado en nombres, en el que los vínculos tienen el formato `dbs/MyDatabaseId/colls/MyCollectionId/docs/MyDocumentId`, en lugar de SelfLinks (\_self) que tiene el formato `dbs/<database_rid>/colls/<collection_rid>/docs/<document_rid>`, para evitar recuperar los valores de ResourceIds de todos los recursos empleados para la construcción del vínculo. Además, a medida que se recrean estos recursos (posiblemente con el mismo nombre), almacenarlos en la caché puede que no sea útil.

* **Ajuste del tamaño de página en consultas y fuentes de lectura para aumentar el rendimiento**

  Al realizar una lectura masiva de documentos mediante la funcionalidad de fuentes de lectura (por ejemplo, readDocuments) o al emitir una consulta SQL, los resultados se devuelven de forma segmentada si el conjunto de resultados es demasiado grande. De forma predeterminada, se devuelven resultados en fragmentos de 1 MB o de 100 artículos, el límite que se alcance primero.

  Para reducir el número de recorridos de ida y vuelta de red necesarios para recuperar todos los resultados aplicables, puede aumentar el tamaño de página con el encabezado de solicitud [x-ms-max-item-count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) hasta 1000. En aquellos casos en los que solo sea necesario mostrar unos cuantos resultados, por ejemplo, si la interfaz de usuario o la API de aplicación solo devuelven 10 resultados de una vez, también puede reducir el tamaño de página a 10 a fin de reducir el rendimiento consumido en las lecturas y consultas.

  También puede establecer el tamaño de página con el método setMaxItemCount.

* **Use el programador adecuado (evite el robo de subprocesos de E/S Eventloop de Netty)**

  El SDK asincrónico de Azure Cosmos DB para Java v2 usa [Netty](https://netty.io/) para la E/S sin bloqueo. El SDK usa un número fijo de subprocesos de E/S Eventloop de Netty (tantos como núcleos de CPU tenga su máquina) para ejecutar operaciones de E/S. El observable que devuelve la API emite el resultado en uno de los subprocesos de E/S Eventloop de Netty compartidos. Así que es importante no bloquear dichos subprocesos. Realizar trabajos que hacen un uso elevado de CPU o bloquear operaciones en el subproceso de E/S Eventloop de Netty puede provocar un interbloqueo o reducir el rendimiento del SDK de manera considerable.

  Por ejemplo, el código siguiente ejecuta un trabajo que hace un uso elevado de CPU en el subproceso de E/S Eventloop de Netty:

  **SDK de Java v2 asincrónico (Maven com.microsoft.azure::azure-cosmosdb)**

  ```java
    Observable<ResourceResponse<Document>> createDocObs = asyncDocumentClient.createDocument(
      collectionLink, document, null, true);

    createDocObs.subscribe(
      resourceResponse -> {
        //this is executed on eventloop IO netty thread.
        //the eventloop thread is shared and is meant to return back quickly.
        //
        // DON'T do this on eventloop IO netty thread.
        veryCpuIntensiveWork();
      });
  ```

  Tras recibir el resultado, si quiere realizar sobre él un trabajo que hace un uso elevado de CPU, debe evitar hacerlo sobre un subproceso de E/S Eventloop de Netty. En su lugar, proporcione su propio programador para suministrar su propio subproceso para ejecutar su trabajo.

  **SDK de Java v2 asincrónico (Maven com.microsoft.azure::azure-cosmosdb)**

  ```java
    import rx.schedulers;

    Observable<ResourceResponse<Document>> createDocObs = asyncDocumentClient.createDocument(
      collectionLink, document, null, true);

    createDocObs.subscribeOn(Schedulers.computation())
    subscribe(
      resourceResponse -> {
        // this is executed on threads provided by Scheduler.computation()
        // Schedulers.computation() should be used only when:
        //   1. The work is cpu intensive 
        //   2. You are not doing blocking IO, thread sleep, etc. in this thread against other resources.
        veryCpuIntensiveWork();
      });
  ```

  Según el tipo del trabajo, debe usar el programador de RxJava existente adecuado para su trabajo. Obtenga más información aquí [``Schedulers``](http://reactivex.io/RxJava/1.x/javadoc/rx/schedulers/Schedulers.html).

  Para más información, consulte la [página de GitHub](https://github.com/Azure/azure-cosmosdb-java) del SDK asincrónico de Azure Cosmos DB para Java v2.

* **Deshabilitación del registro de Netty**

    El registro de la biblioteca Netty es muy activo y debe desactivarse (puede que no baste con suprimir el inicio de sesión en la configuración) para evitar costos adicionales de CPU. Si no está en modo de depuración, deshabilite por completo el registro de Netty. De modo que, si va a usar log4j para eliminar los costos de CPU adicionales que se producen debido al uso de ``org.apache.log4j.Category.callAppenders()`` desde Netty, agregue la siguiente línea al código base:

    ```java
    org.apache.log4j.Logger.getLogger("io.netty").setLevel(org.apache.log4j.Level.OFF);
    ```

 * **Límite de recursos de archivos abiertos del sistema operativo**
 
    Algunos sistemas Linux (como Red Hat) tienen un límite superior sobre el número de archivos abiertos y, por tanto, sobre el número total de conexiones. Ejecute el siguiente código para ver los límites actuales:

    ```bash
    ulimit -a
    ```

    El número de archivos abiertos (nofile) debe ser lo bastante grande como para que haya espacio suficiente para el tamaño configurado del grupo de conexiones y otros archivos abiertos por el sistema operativo. Se puede modificar para permitir un tamaño mayor del grupo de conexiones.

    Abra el archivo limits.conf:

    ```bash
    vim /etc/security/limits.conf
    ```
    
    Agregue o modifique las siguientes líneas:

    ```
    * - nofile 100000
    ```

## <a name="indexing-policy"></a>Directiva de indexación
 
* **Exclusión de rutas de acceso sin utilizar de la indexación para acelerar las escrituras**

    La directiva de indexación de Azure Cosmos DB le permite especificar las rutas de acceso de documentos que se incluirán en la indexación o se excluirán de esta mediante el aprovechamiento de las rutas de acceso de indexación (setIncludedPaths y setExcludedPaths). El uso de rutas de acceso de indexación puede ofrecer un rendimiento de escritura mejorado y un almacenamiento de índices reducido en escenarios en los que los patrones de consulta se conocen de antemano, dado que los costos de indexación están directamente correlacionados con el número de rutas de acceso únicas indexadas. Por ejemplo, en el siguiente código se muestra cómo excluir una sección completa de los documentos (también conocido como subárbol) de la indexación mediante el comodín "*".

    ### <a name="async-java-sdk-v2-maven-commicrosoftazureazure-cosmosdb"></a><a id="asyncjava2-indexing"></a>SDK asincrónico para Java v2 (Maven com.microsoft.azure::azure-cosmosdb)

    ```Java
    Index numberIndex = Index.Range(DataType.Number);
    numberIndex.set("precision", -1);
    indexes.add(numberIndex);
    includedPath.setIndexes(indexes);
    includedPaths.add(includedPath);
    indexingPolicy.setIncludedPaths(includedPaths);
    collectionDefinition.setIndexingPolicy(indexingPolicy);
    ```

    Para más información, consulte [Directivas de indexación de Azure Cosmos DB](indexing-policies.md).

## <a name="throughput"></a><a id="measure-rus"></a>Capacidad de proceso

* **Medición y optimización del uso menor de unidades de solicitud por segundo**

    Azure Cosmos DB ofrece un amplio conjunto de operaciones de base de datos, incluidas consultas relacionales y jerárquicas con funciones definidas por el usuario, procedimientos almacenados y desencadenadores. Todo funciona con los documentos dentro de una colección de base de datos. El costo asociado a cada una de estas operaciones variará en función de la CPU, la E/S y la memoria necesarias para completar la operación. En lugar de administrar y pensar sobre los recursos de hardware, puede pensar en una unidad de solicitud (RU) como una medida única para los recursos necesarios para realizar varias operaciones de la base de datos y dar servicio a una solicitud de la aplicación.

    El rendimiento se aprovisiona en función del número de [unidades de solicitud](request-units.md) establecido para cada contenedor. El consumo de la unidad de solicitud se evalúa como frecuencia por segundo. Las aplicaciones que superan la frecuencia de unidad de solicitud aprovisionada para su contenedor están limitadas hasta que la frecuencia cae por debajo del nivel aprovisionado del contenedor. Si la aplicación requiere un mayor nivel de rendimiento, puede aumentar el rendimiento mediante el aprovisionamiento de unidades de solicitud adicionales.

    La complejidad de una consulta afecta a la cantidad de unidades de solicitud consumidas para una operación. El número de predicados, la naturaleza de los predicados, el número de UDF y el tamaño del conjunto de datos de origen influyen en el costo de operaciones de consulta.

    Para medir la sobrecarga de cualquier operación (crear, actualizar o eliminar), inspeccione el encabezado [x-ms-request-charge](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) para medir el número de unidades de solicitud usadas por estas operaciones. También puede examinar la propiedad RequestCharge equivalente en ResourceResponse\<T> o FeedResponse\<T>.

    ### <a name="async-java-sdk-v2-maven-commicrosoftazureazure-cosmosdb"></a><a id="asyncjava2-requestcharge"></a>SDK de Java v2 asincrónico (Maven com.microsoft.azure::azure-cosmosdb)

    ```Java
    ResourceResponse<Document> response = asyncClient.createDocument(collectionLink, documentDefinition, null,
                                                     false).toBlocking.single();
    response.getRequestCharge();
    ```

    El cargo de solicitud devuelto en este encabezado es una fracción de la capacidad de proceso aprovisionada. Por ejemplo, si tiene 2000 RU/segundo aprovisionadas, y si la consulta anterior devuelve 1000 documentos de 1 KB, el costo de la operación será 1000. Por lo tanto, al cabo de un segundo, el servidor atenderá solo dos de estas solicitudes antes de limitar la velocidad de las solicitudes posteriores. Para más información, consulte [Unidades de solicitud](request-units.md) y la [calculadora de unidades de solicitud](https://www.documentdb.com/capacityplanner).

* **Administración de la limitación de velocidad y la tasa de solicitudes demasiado grande**

    Cuando un cliente intenta superar la capacidad de proceso reservada para una cuenta, no habrá ninguna degradación del rendimiento en el servidor y no se utilizará ninguna capacidad de proceso más allá del nivel reservado. El servidor finalizará de forma preventiva la solicitud con RequestRateTooLarge (código de estado HTTP 429) y devolverá el encabezado [x-ms-retry-after-ms](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) para indicar la cantidad de tiempo, en milisegundos, que el usuario debe esperar antes de volver a intentar realizar la solicitud.

   ```xml
   HTTP Status 429,
   Status Line: RequestRateTooLarge
   x-ms-retry-after-ms :100
   ```

    Los SDK capturan implícitamente esta respuesta, respetan el encabezado retry-after especificado por el servidor y reintentan la solicitud. A menos que varios clientes obtengan acceso a la cuenta al mismo tiempo, el siguiente reintento se realizará correctamente.

    Si tiene más de un cliente que funciona acumulativamente de forma coherente por encima de la tasa de solicitudes, puede que el número de reintentos predeterminado establecido actualmente en 9 de manera interna por el cliente no sea suficiente; en este caso, el cliente producirá una excepción DocumentClientException con el código de estado 429 para la aplicación. El número de reintentos predeterminado se puede cambiar mediante setRetryOptions en la instancia ConnectionPolicy. De forma predeterminada, la excepción DocumentClientException con el código de estado 429 se devuelve tras un tiempo de espera acumulativo de 30 segundos si la solicitud sigue funcionando por encima de la tasa de solicitudes. Esto sucede incluso cuando el número de reintentos actual es inferior al número de reintentos máximo de 9, el valor predeterminado, o un valor definido por el usuario.

    Aunque el comportamiento de reintento automático ayuda a mejorar la resistencia y la usabilidad en la mayoría de las aplicaciones, podría no resultar ventajoso al realizar comparativas de rendimiento, en especial al medir la latencia. La latencia observada del cliente aumentará si el experimento llega a la limitación del servidor y hace que el SDK del cliente realice reintentos de forma silenciosa. Para evitar aumentos de latencia durante los experimentos de rendimiento, mida el gasto devuelto por cada operación y asegúrese de que las solicitudes funcionan por debajo de la tasa de solicitudes observada. Para más información, consulte [Unidades de solicitud](request-units.md).

* **Diseño de documentos más pequeños para un mayor rendimiento**

    El gasto de solicitud (es decir, el costo de procesamiento de solicitudes) de una operación dada está directamente correlacionado con el tamaño del documento. Las operaciones con documentos grandes cuestan más que las operaciones con documentos pequeños.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo diseñar la aplicación para escalarla y obtener un alto rendimiento, consulte [Partición y escalado en Azure Cosmos DB](partition-data.md).
