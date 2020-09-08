---
title: Sugerencias de rendimiento para la versión 4 del SDK de Java de Azure Cosmos DB
description: Conozca las opciones de configuración de cliente para mejorar el rendimiento de las bases de datos de Azure Cosmos para la versión 4 del SDK de Java.
author: anfeldma-ms
ms.service: cosmos-db
ms.devlang: java
ms.topic: how-to
ms.date: 07/08/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: a014038996ae2846d059551b565feedd8de560a0
ms.sourcegitcommit: ef055468d1cb0de4433e1403d6617fede7f5d00e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/16/2020
ms.locfileid: "88258314"
---
# <a name="performance-tips-for-azure-cosmos-db-java-sdk-v4"></a>Sugerencias de rendimiento para la versión 4 del SDK de Java de Azure Cosmos DB

> [!div class="op_single_selector"]
> * [Versión 4 del SDK de Java](performance-tips-java-sdk-v4-sql.md)
> * [Versión 2 del SDK de Java asincrónico](performance-tips-async-java.md)
> * [SDK de Java v2 sincrónico](performance-tips-java.md)
> * [.NET SDK v3](performance-tips-dotnet-sdk-v3-sql.md)
> * [.NET SDK v2](performance-tips.md)
> 

> [!IMPORTANT]  
> Las sugerencias de rendimiento de este artículo son solo para la versión 4 del SDK de Java de Azure Cosmos DB. Consulte las [notas de la versión](sql-api-sdk-java-v4.md) de la versión 4 del SDK de Java de Azure Cosmos DB, el [repositorio de Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos) y la [guía de solución de problemas](troubleshoot-java-sdk-v4-sql.md) de la versión 4 del SDK de Java de Azure Cosmos DB para obtener más información. Si en la actualidad usa una versión anterior a la 4, vea la guía [Migración a la versión 4 del SDK de Java de Azure Cosmos DB](migrate-java-v4-sdk.md) a fin de obtener ayuda para actualizar a la versión 4.
>

Azure Cosmos DB es una base de datos distribuida rápida y flexible que se escala sin problemas con una latencia y un rendimiento garantizados. No es necesario realizar cambios de arquitectura importantes ni escribir código complejo para escalar la base de datos con Azure Cosmos DB. Escalar y reducir verticalmente es tan sencillo como realizar una única llamada API o una llamada al método SDK. Sin embargo, como el acceso a Azure Cosmos DB se realiza mediante llamadas de red, puede realizar optimizaciones en el lado cliente para conseguir un rendimiento máximo al usar la versión 4 del SDK de Java de Azure Cosmos DB.

Así que si se está preguntando "¿Cómo puedo mejorar el rendimiento de la base de datos?", tenga en cuenta las opciones siguientes:

## <a name="networking"></a>Redes

* **Modo de conexión: uso del modo directo**
<a id="direct-connection"></a>
    
    El modo en que un cliente se conecta a Azure Cosmos DB tiene implicaciones importantes sobre el rendimiento, especialmente en términos de la latencia en el lado cliente. El modo de conexión es una opción de configuración clave disponible para configurar el cliente. Para la versión 4 del SDK de Java de Azure Cosmos DB, los dos modos de conexión disponibles son:  

    * Modo directo (valor predeterminado)      
    * Modo de puerta de enlace

    En esencia, estos modos de conexión determinan la ruta que las solicitudes de plano de datos (lecturas y escrituras de documentos) toman de la máquina cliente a las particiones en el back-end de Azure Cosmos DB. Por lo general, el modo directo es la opción preferida para obtener el mejor rendimiento: permite al cliente abrir conexiones TCP directamente en las particiones en el back-end de Azure Cosmos DB y enviar solicitudes *directamente* sin intermediarios. Por el contrario, en el modo de puerta de enlace, las solicitudes realizadas por el cliente se enrutan a un servidor denominado "puerta de enlace" en el front-end de Azure Cosmos DB que, a su vez, deriva sus solicitudes a las particiones adecuadas en el back-end de Azure Cosmos DB. Si la aplicación se ejecuta dentro de una red corporativa con restricciones de firewall estrictas, el modo de puerta de enlace es la mejor opción, ya que utiliza el puerto HTTPS estándar y un único punto de conexión. La desventaja para el rendimiento, sin embargo, es que el modo de puerta de enlace implica un salto de red adicional (cliente a puerta de enlace más puerta de enlace a partición) cada vez que se leen o escriben datos en Azure Cosmos DB. Por este motivo, el modo directo ofrece mejor rendimiento debido al número menor de saltos de red.

    El modo de conexión de las solicitudes de plano de datos se configura en el generador de cliente de Azure Cosmos DB con los métodos *directMode()* o *gatewayMode()* , como se muestra a continuación. Para configurar cualquiera de los modos con los valores predeterminados, llame a cualquiera de los métodos sin argumentos. De lo contrario, pase una instancia de clase de configuración como el argumento (*DirectConnectionConfig* para *directMode()* , *GatewayConnectionConfig* para *gatewayMode()* .)
    
    ### <a name="java-v4-sdk"></a><a id="override-default-consistency-javav4"></a> SDK de Java V4

    # <a name="async"></a>[Asincrónico](#tab/api-async)

    API asincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceClientConnectionModeAsync)]

    # <a name="sync"></a>[Sincronizar](#tab/api-sync)

    API sincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceClientConnectionModeSync)]

    --- 

    El método *directMode()* tiene un reemplazo adicional por la siguiente razón. Las operaciones del plano de control, como CRUD de base de datos y contenedor, *siempre* usan el modo Puerta de enlace. Cuando el usuario tiene configurado el modo directo para las operaciones del plano de datos, las operaciones del plano de control usan la opción del modo Puerta de enlace predeterminada. Esto se adapta a la mayoría de los usuarios. Sin embargo, los usuarios que quieren usar el modo directo para las operaciones del plano de datos, así como la capacidad de ajuste de los parámetros del modo Puerta de enlace del plano de control, pueden usar el reemplazo *directMode()* siguiente:

    ### <a name="java-v4-sdk"></a><a id="override-default-consistency-javav4"></a> SDK de Java V4

    # <a name="async"></a>[Asincrónico](#tab/api-async)

    API asincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceClientDirectOverrideAsync)]

    # <a name="sync"></a>[Sincronizar](#tab/api-sync)

    API sincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceClientDirectOverrideSync)]

    --- 

<a name="collocate-clients"></a>
* **Colocación de los clientes en la misma región de Azure para aumentar el rendimiento** <a id="same-region"></a>

    Cuando sea posible, coloque las aplicaciones que llaman a Azure Cosmos DB en la misma región que la base de datos de Azure Cosmos. Para obtener una comparación aproximada, las llamadas a Azure Cosmos DB en la misma región se realizan en menos de 1 o 2 ms, pero la latencia entre las costas este y oeste de Estados Unidos es mayor de 50 ms. Esta latencia podría variar de una solicitud a otra, según la ruta tomada por la solicitud cuando pasa del cliente al límite del centro de datos de Azure. Para conseguir la menor latencia posible, asegúrese de que la aplicación que llama se encuentra en la misma región de Azure que el punto de conexión de Azure Cosmos DB aprovisionado. Para obtener una lista de regiones disponibles, consulte [Regiones de Azure](https://azure.microsoft.com/regions/#services).

    :::image type="content" source="./media/performance-tips/same-region.png" alt-text="Ilustración de la directiva de conexión de Azure Cosmos DB" border="false":::

    Una aplicación que interactúa con una cuenta de Azure Cosmos DB de varias regiones debe configurar [ubicaciones preferidas](tutorial-global-distribution-sql-api.md#preferred-locations) para asegurarse de que las solicitudes vayan a una región colocalizada.

* **Habilite las redes aceleradas en la máquina virtual de Azure para una menor latencia.**

Se recomienda seguir las instrucciones para habilitar redes aceleradas en su VM de Azure con [Windows (haga clic para obtener instrucciones)](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-powershell) o [Linux (haga clic en las instrucciones)](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli) para maximizar el rendimiento.

Sin las redes aceleradas, la E/S que pasa entre la máquina virtual de Azure y otros recursos de Azure puede enrutarse innecesariamente a través de un host y un conmutador virtual situado entre la máquina virtual y su tarjeta de red. Si el host y el conmutador virtual están alineados en la ruta de datos no solo se aumenta la latencia y la vibración en el canal de comunicación, sino que también se roban ciclos de la CPU de la máquina virtual. Con las redes aceleradas, la máquina virtual interactúa directamente con la NIC sin intermediarios; los detalles de la directiva de red que se administraban mediante el host y el conmutador virtual se administran ahora en hardware en la NIC; se omiten el host y el conmutador virtual. Por lo general, al habilitar las redes aceleradas puede esperar una menor latencia y un mayor rendimiento, así como una latencia más *uniforme* y una disminución del uso de la CPU.

Limitaciones: las redes aceleradas deben ser compatibles con el sistema operativo de la máquina virtual y solo se pueden habilitar cuando la máquina virtual se ha detenido y se ha desasignado. No se puede implementar la máquina virtual con Azure Resource Manager.

Consulte las instrucciones de [Windows](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-powershell) y [Linux](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli) para obtener más detalles.

## <a name="sdk-usage"></a>Uso del SDK
* **Instalación del SDK más reciente**

    Los SDK de Azure Cosmos DB se mejoran constantemente para proporcionar el mejor rendimiento. Consulte las páginas del [SDK de Azure Cosmos DB](sql-api-sdk-async-java.md) para determinar las mejoras de los SDK y las revisiones más recientes.

* **Uso de un cliente de Azure Cosmos DB singleton para aumentar la duración de la aplicación**

    Cada instancia de Azure Cosmos DB está protegida frente a amenazas y realiza de manera eficiente la administración de las conexiones y el almacenamiento en caché de las direcciones. Para permitir la administración eficiente de las conexiones y un rendimiento mejor mediante el cliente de Azure Cosmos DB, se recomienda usar una sola instancia del cliente de Azure Cosmos DB por AppDomain durante la vigencia de la aplicación.

   <a id="max-connection"></a>

* **Uso del nivel de coherencia más bajo necesario para la aplicación**

    Cuando se crea una instancia de *CosmosClient*, la coherencia predeterminada utilizada si no se establece explícitamente es *Sesión*. Si la lógica de la aplicación no requiere coherencia en el nivel *Sesión*, establezca el valor de *Coherencia* en *Ocasional*. Nota: Se recomienda usar al menos la coherencia en el nivel *Sesión* en las aplicaciones que emplean el procesador de fuente de cambios de Azure Cosmos DB.

* **Uso de la API asincrónica para maximizar el rendimiento aprovisionado**

    La versión 4 del SDK de Java de Azure Cosmos DB agrupa dos API: sincrónica y asincrónica. En términos generales, la API asincrónica implementa la funcionalidad del SDK, mientras que la API sincrónica es un contenedor fino que realiza llamadas de bloqueo a la API asincrónica. Esto se opone a la anterior versión 2 del SDK de Java asincrónico de Azure Cosmos DB y a la anterior versión 2 del SDK de Java sincrónico de Azure Cosmos DB, que era solo de sincronización y tenía una implementación completamente independiente. 
    
    La elección de la API se determina durante la inicialización del cliente; *CosmosAsyncClient* admite la API asincrónica, mientras que *CosmosClient* admite la API sincrónica. 
    
    La API asincrónica implementa E/S sin bloqueo y es la opción óptima si el objetivo es maximizar el rendimiento al emitir solicitudes para Azure Cosmos DB. 
    
    El uso de la API sincrónica puede ser la opción correcta si desea o necesita una API que bloquee la respuesta a cada solicitud, o si la operación sincrónica es el paradigma dominante en la aplicación. Por ejemplo, es posible que desee la API sincrónica al guardar los datos en Azure Cosmos DB en una aplicación de microservicios, siempre que el rendimiento no sea crítico.  
    
    Tenga en cuenta que el rendimiento de la API sincrónica se degrada con un tiempo de respuesta de solicitud creciente, mientras que la API asincrónica puede saturar todas las capacidades de ancho de banda del hardware. 
    
    La colocación geográfica puede brindarle un rendimiento más alto y más uniforme cuando se usa la API sincrónica (consulte [Colocación de los clientes en la misma región de Azure para aumentar el rendimiento](#collocate-clients)) pero aún no se espera que exceda el rendimiento que puede alcanzar la API asincrónica.

    Algunos usuarios también pueden no estar familiarizados con [Project Reactor](https://projectreactor.io/), el marco de Reactive Streams utilizado para implementar la API asincrónica de la versión 4 del SDK de Java de Azure Cosmos DB. Si esto supone un problema, le recomendamos que lea nuestra [guía de Reactor Pattern](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-pattern-guide.md) introductoria y, después, eche un vistazo a esta [introducción a la programación reactiva](https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Intro) con el fin de familiarizarse con estos conceptos. Si ya ha usado Azure Cosmos DB con una interfaz asincrónica y el SDK que usó era la versión 2 del SDK de Java asincrónico de Azure Cosmos DB, es posible que conozca [ReactiveX](http://reactivex.io/)/[RxJava](https://github.com/ReactiveX/RxJava) pero no esté al corriente de los cambios en Project Reactor. En ese caso, eche un vistazo a nuestra [guía comparativa de Reactor y RxJava](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-rxjava-guide.md) para familiarizarse con ello.

    En los fragmentos de código siguientes se muestra cómo inicializar el cliente de Azure Cosmos DB para la operación de API asincrónica o API sincrónica, respectivamente:

    ### <a name="java-v4-sdk"></a><a id="override-default-consistency-javav4"></a> SDK de Java V4

    # <a name="async"></a>[Asincrónico](#tab/api-async)

    API asincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceClientAsync)]

    # <a name="sync"></a>[Sincronizar](#tab/api-sync)

    API sincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceClientSync)]

    --- 

* **Optimización de ConnectionPolicy**

    De forma predeterminada, las solicitudes del modo directo de Cosmos DB se realizan sobre TCP cuando se usa la versión 4 del SDK de Java de Azure Cosmos DB. Internamente, el modo directo usa una arquitectura especial para administrar dinámicamente los recursos de red y obtener el mejor rendimiento.

    En la versión 4 del SDK de Java de Azure Cosmos DB, el modo directo es la mejor opción para mejorar el rendimiento de la base de datos con la mayoría de las cargas de trabajo. 

    * ***Introducción al modo directo***

        :::image type="content" source="./media/performance-tips-async-java/rntbdtransportclient.png" alt-text="Ilustración de la arquitectura del modo directo" border="false":::

        La arquitectura del lado cliente empleada en el modo directo permite un uso predecible de la red y el acceso multiplexado a las réplicas de Azure Cosmos DB. En el diagrama anterior se muestra cómo el modo directo enruta las solicitudes de cliente a las réplicas en el back-end de Cosmos DB. La arquitectura del modo directo asigna hasta 10 **canales** en el lado cliente por réplica de base de datos. Un canal es una conexión TCP precedida por un búfer de solicitudes, que tiene 30 solicitudes de profundidad. El **punto de conexión de servicio** de la réplica asigna dinámicamente según sea necesario los canales que pertenecen a una réplica. Cuando el usuario emite una solicitud en el modo directo, **TransportClient** enruta la solicitud al punto de conexión de servicio adecuado en función de la clave de partición. La **cola de solicitudes** almacena en búfer las solicitudes antes del punto de conexión de servicio.

    * ***Opciones de configuración para el modo directo***

        Si se desea un comportamiento del modo directo no predeterminado, cree una instancia de *DirectConnectionConfig* y personalice sus propiedades y, a continuación, pase la instancia de la propiedad personalizada al método *directMode()* en el generador de cliente de Azure Cosmos DB.

        Estos valores de configuración controlan el comportamiento de la arquitectura del modo directo subyacente que se describió anteriormente.

        Como primer paso, utilice las opciones de configuración recomendadas a continuación. Estas opciones de *DirectConnectionConfig* son opciones de configuración avanzada que pueden afectar al rendimiento del SDK de maneras inesperadas; se recomienda que los usuarios no las modifiquen a menos que comprendan perfectamente los inconvenientes y sea absolutamente necesario. Póngase en contacto con el [equipo de Azure Cosmos DB](mailto:CosmosDBPerformanceSupport@service.microsoft.com) si tiene problemas con este tema específico.

        | Opción de configuración       | Valor predeterminado    |
        | :------------------:       | :-----:    |
        | idleConnectionTimeout      | "PT1M"     |
        | maxConnectionsPerEndpoint  | "PT0S"     |
        | connectTimeout             | "PT1M10S"  |
        | idleEndpointTimeout        | 8388608    |
        | maxRequestsPerConnection   | 10         |

* **Ajuste de consultas paralelas en colecciones particionadas**

    La versión 4 del SDK de Java de Azure Cosmos DB admite consultas paralelas que permiten consultar una colección con particiones en paralelo. Para obtener más información, consulte [ejemplos de código](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples) relacionados para trabajar con la versión 4 del SDK de Java de Azure Cosmos DB. Las consultas paralelas están diseñadas para mejorar la latencia y el rendimiento de la consulta en todos sus homólogos seriales.

    * ***Ajuste de setMaxDegreeOfParallelism\:***
    
        Las consultas paralelas funcionan creando consultas en varias particiones en paralelo. Sin embargo, los datos de una recopilación con particiones individual se capturan en serie con respecto a la consulta. Por lo tanto, use el parámetro setMaxDegreeOfParallelism para establecer el número de particiones que tienen la máxima probabilidad de conseguir el mejor rendimiento de consulta, siempre y cuando el resto de las demás condiciones del sistema permanezcan invariables. Si no conoce el número de particiones, puede usar setMaxDegreeOfParallelism para establecer un número alto y el sistema elegirá el mínimo (número de particiones, entrada proporcionada por el usuario) como el grado máximo de paralelismo.

        Es importante tener en cuenta que las consultas en paralelo producen los mejores beneficios si los datos se distribuyen uniformemente entre todas las particiones con respecto a la consulta. Si la colección con particiones está dividida de tal forma que todos, o la mayoría de los datos, devueltos por una consulta se concentran en algunas particiones (una partición en el peor de los casos), entonces el rendimiento de la consulta se vería afectada por cuellos de botella debido a esas particiones.

    * ***Ajuste de setMaxBufferedItemCount\:***
    
        Las consultas en paralelo están diseñadas para capturar previamente los resultados mientras el cliente procesa el lote actual de resultados. La captura previa ayuda a mejorar la latencia general de una consulta. setMaxBufferedItemCount limita el número de resultados capturados previamente. Establece setMaxBufferedItemCount en el número esperado de resultados devueltos (o un número más alto) permite que la consulta reciba el máximo beneficio de la captura previa.

        La captura previa funciona de la misma manera con independencia de MaxDegreeOfParallelism, y hay un único búfer para los datos de todas las particiones.

* **Escalado horizontal de la carga de trabajo de cliente**

    Si va a realizar pruebas en niveles de alto rendimiento, la aplicación cliente puede volverse un cuello de botella debido a que la máquina limita el uso de CPU o de la red. Si llega a este punto, puede seguir insertando la cuenta de Azure Cosmos DB mediante la escala horizontal de las aplicaciones cliente en varios servidores.

    Una buena regla general es no superar el uso de la CPU al 50 % en cualquier servidor para mantener baja la latencia.

   <a id="tune-page-size"></a>

* **Ajuste del tamaño de página en consultas y fuentes de lectura para aumentar el rendimiento**

    Al realizar una lectura masiva de documentos mediante la funcionalidad de fuentes de lectura (por ejemplo, *readItems*) o al emitir una consulta SQL (*queryItems*), los resultados se devuelven de forma segmentada si el conjunto de resultados es demasiado grande. De forma predeterminada, se devuelven resultados en fragmentos de 1 MB o de 100 artículos, el límite que se alcance primero.

    Supongamos que la aplicación emite una consulta para Azure Cosmos DB y que la aplicación requiere el conjunto completo de resultados de la consulta para poder completar su tarea. Para reducir el número de recorridos de ida y vuelta de red necesarios para recuperar todos los resultados aplicables, puede aumentar el tamaño de página ajustando el campo del encabezado de la solicitud [x-ms-max-item-count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers). 

    * Para consultas de una sola partición, el ajuste del valor del campo [x-ms-max-item-count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) a -1 (sin límite en el tamaño de página) maximiza la latencia al minimizar el número de páginas de respuesta de consulta: el conjunto de resultados completo se devolverá en una sola página o, si la consulta tarda más que el intervalo de tiempo de espera, se devolverá el conjunto de resultados completo en el número mínimo de páginas posible. Esto ahorra en múltiplos el tiempo de ida y vuelta de la solicitud.
    
    * En el caso de las consultas entre particiones, si se establece el campo [x-ms-max-item-count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) en -1 y se quitan los riesgos del límite de tamaño de página, se satura el SDK con tamaños de página no administrables. En el caso de particiones cruzadas, se recomienda aumentar el límite de tamaño de página hasta un valor grande pero finito, quizás 1000, para reducir la latencia. 
    
    En algunas aplicaciones, es posible que no necesite el conjunto completo de resultados de la consulta. En aquellos casos en los que solo sea necesario mostrar unos cuantos resultados, por ejemplo, si la interfaz de usuario o la API de aplicación solo devuelven 10 resultados de una vez, también puede reducir el tamaño de página a 10 a fin de reducir el rendimiento consumido en las lecturas y consultas.

    También puede establecer el argumento de tamaño de página preferido del método *byPage*, en lugar de modificar el campo de encabezado REST directamente. Tenga en cuenta que [x-MS-Max-Item-Count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) o el argumento de tamaño de página preferido de *byPage* solo establece un límite superior en el tamaño de página, no un requisito absoluto; por lo tanto, por diversos motivos, puede ver que Azure Cosmos DB devuelve páginas que son menores que el tamaño de página preferido. 

* **Use el programador adecuado (evite el robo de subprocesos de E/S Eventloop de Netty)**

    La funcionalidad asincrónica del SDK de Java de Azure Cosmos DB se basa en la E/S sin bloqueo de [netty](https://netty.io/). El SDK usa un número fijo de subprocesos de E/S Eventloop de Netty (tantos como núcleos de CPU tenga su máquina) para ejecutar operaciones de E/S. El elemento Flux que devuelve la API emite el resultado en uno de los subprocesos de E/S Eventloop de Netty compartidos. Así que es importante no bloquear dichos subprocesos. Realizar trabajos que hacen un uso elevado de CPU o bloquear operaciones en el subproceso de E/S Eventloop de Netty puede provocar un interbloqueo o reducir el rendimiento del SDK de manera considerable.

    Por ejemplo, el código siguiente ejecuta un trabajo que hace un uso elevado de CPU en el subproceso de E/S Eventloop de Netty:
    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-noscheduler"></a>Versión 4 del SDK de Java (Maven com.azure::azure-cosmos) API asincrónica

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceNeedsSchedulerAsync)]

    Tras recibir el resultado, si quiere realizar sobre él un trabajo que hace un uso elevado de CPU, debe evitar hacerlo sobre un subproceso de E/S Eventloop de Netty. En su lugar, proporcione su propio programador a fin de suministrar su propio subproceso para ejecutar su trabajo, como se muestra a continuación (requiere `import reactor.core.scheduler.Schedulers`).

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-scheduler"></a>API asincrónica del SDK para Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceAddSchedulerAsync)]

    Según el tipo del trabajo, debe usar el programador de Reactor existente adecuado para su trabajo. Obtenga más información aquí [``Schedulers``](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Schedulers.html).

    Para obtener más información sobre la versión 4 del SDK de Java de Azure Cosmos DB, consulte el [repositorio único del directorio Cosmos DB del SDK de Azure para Java en GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-cosmos).

* **Optimización de la configuración de registro en la aplicación**

    Por diversos motivos, puede que desee o necesite agregar registro en un subproceso que genera un rendimiento de solicitud elevado. Si su objetivo es saturar completamente el rendimiento aprovisionado de un contenedor con las solicitudes generadas por este subproceso, las optimizaciones de registro pueden mejorar considerablemente el rendimiento.

    * ***Configuración de un registrador asincrónico***

        La latencia de un registrador sincrónico necesariamente tiene en cuenta el cálculo de la latencia general de su subproceso generador de solicitudes. Se recomienda un registrador asincrónico como [log4j2](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flogging.apache.org%2Flog4j%2Flog4j-2.3%2Fmanual%2Fasync.html&data=02%7C01%7CCosmosDBPerformanceInternal%40service.microsoft.com%7C36fd15dea8384bfe9b6b08d7c0cf2113%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637189868158267433&sdata=%2B9xfJ%2BWE%2F0CyKRPu9AmXkUrT3d3uNA9GdmwvalV3EOg%3D&reserved=0) para desacoplar la sobrecarga de registro de los subprocesos de la aplicación de alto rendimiento.

    * ***Deshabilitación del registro de Netty***

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

* **Especificación de la clave de partición en escrituras puntuales**

    Para mejorar el rendimiento de las escrituras puntuales, especifique la clave de partición del elemento en la llamada API de escritura puntual, como se muestra a continuación:

    # <a name="async"></a>[Asincrónico](#tab/api-async)

    API asincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceNoPKAsync)]

    # <a name="sync"></a>[Sincronizar](#tab/api-sync)

    API sincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceNoPKSync)]

    --- 

    en lugar de proporcionar solo la instancia del elemento, como se muestra a continuación:

    # <a name="async"></a>[Asincrónico](#tab/api-async)

    API asincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceAddPKAsync)]

    # <a name="sync"></a>[Sincronizar](#tab/api-sync)

    API sincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceAddPKSync)]

    --- 

    Esta última es compatible, pero agregará latencia a la aplicación; el SDK debe analizar el elemento y extraer la clave de partición.

## <a name="indexing-policy"></a>Directiva de indexación
 
* **Exclusión de rutas de acceso sin utilizar de la indexación para acelerar las escrituras**

    La directiva de indexación de Azure Cosmos DB le permite especificar las rutas de acceso de documentos que se incluirán en la indexación o se excluirán de esta mediante el aprovechamiento de las rutas de acceso de indexación (setIncludedPaths y setExcludedPaths). El uso de rutas de acceso de indexación puede ofrecer un rendimiento de escritura mejorado y un almacenamiento de índices reducido en escenarios en los que los patrones de consulta se conocen de antemano, dado que los costos de indexación están directamente correlacionados con el número de rutas de acceso únicas indexadas. Por ejemplo, en el código siguiente se muestra cómo incluir y excluir secciones completas de los documentos (que también se conocen como subárbol) de la indexación mediante el comodín "*".

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos"></a><a id="java4-indexing"></a>SDK de Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateIndexingAsync)]

    Para más información, consulte [Directivas de indexación de Azure Cosmos DB](indexing-policies.md).

## <a name="throughput"></a>Throughput
<a id="measure-rus"></a>

* **Medición y optimización del uso menor de unidades de solicitud por segundo**

    Azure Cosmos DB ofrece un amplio conjunto de operaciones de base de datos, incluidas consultas relacionales y jerárquicas con funciones definidas por el usuario, procedimientos almacenados y desencadenadores. Todo funciona con los documentos dentro de una colección de base de datos. El costo asociado a cada una de estas operaciones variará en función de la CPU, la E/S y la memoria necesarias para completar la operación. En lugar de administrar y pensar sobre los recursos de hardware, puede pensar en una unidad de solicitud (RU) como una medida única para los recursos necesarios para realizar varias operaciones de la base de datos y dar servicio a una solicitud de la aplicación.

    El rendimiento se aprovisiona en función del número de [unidades de solicitud](request-units.md) establecido para cada contenedor. El consumo de la unidad de solicitud se evalúa como frecuencia por segundo. Las aplicaciones que superan la frecuencia de unidad de solicitud aprovisionada para su contenedor están limitadas hasta que la frecuencia cae por debajo del nivel aprovisionado del contenedor. Si la aplicación requiere un mayor nivel de rendimiento, puede aumentar el rendimiento mediante el aprovisionamiento de unidades de solicitud adicionales.

    La complejidad de una consulta afecta a la cantidad de unidades de solicitud consumidas para una operación. El número de predicados, la naturaleza de los predicados, el número de UDF y el tamaño del conjunto de datos de origen influyen en el costo de operaciones de consulta.

    Para medir la sobrecarga de cualquier operación (crear, actualizar o eliminar), inspeccione el encabezado [x-ms-request-charge](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) para medir el número de unidades de solicitud usadas por estas operaciones. También puede examinar la propiedad RequestCharge equivalente en ResourceResponse\<T> o FeedResponse\<T>.

    # <a name="async"></a>[Asincrónico](#tab/api-async)

    API asincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceRequestChargeAsync)]

    # <a name="sync"></a>[Sincronizar](#tab/api-sync)

    API sincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceRequestChargeSync)]

    --- 

    El cargo de solicitud devuelto en este encabezado es una fracción de la capacidad de proceso aprovisionada. Por ejemplo, si tiene 2000 RU/segundo aprovisionadas, y si la consulta anterior devuelve 1000 documentos de 1 KB, el costo de la operación será 1000. Por lo tanto, al cabo de un segundo, el servidor atenderá solo dos de estas solicitudes antes de limitar la velocidad de las solicitudes posteriores. Para más información, consulte [Unidades de solicitud](request-units.md) y la [calculadora de unidades de solicitud](https://www.documentdb.com/capacityplanner).

<a id="429"></a>
* **Administración de la limitación de velocidad y la tasa de solicitudes demasiado grande**

    Cuando un cliente intenta superar la capacidad de proceso reservada para una cuenta, no habrá ninguna degradación del rendimiento en el servidor y no se utilizará ninguna capacidad de proceso más allá del nivel reservado. El servidor finalizará de forma preventiva la solicitud con RequestRateTooLarge (código de estado HTTP 429) y devolverá el encabezado [x-ms-retry-after-ms](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) para indicar la cantidad de tiempo, en milisegundos, que el usuario debe esperar antes de volver a intentar realizar la solicitud.

    ```xml
        HTTP Status 429,
        Status Line: RequestRateTooLarge
        x-ms-retry-after-ms :100
    ```

    Los SDK capturan implícitamente esta respuesta, respetan el encabezado retry-after especificado por el servidor y reintentan la solicitud. A menos que varios clientes obtengan acceso a la cuenta al mismo tiempo, el siguiente reintento se realizará correctamente.

    Si tiene más de un cliente que funciona acumulativamente de forma coherente por encima de la tasa de solicitudes, puede que el número de reintentos predeterminado establecido actualmente en 9 de manera interna por el cliente no sea suficiente; en este caso, el cliente producirá *CosmosClientException* con el código de estado 429 para la aplicación. El número de reintentos predeterminado se puede cambiar mediante setRetryOptions en la instancia ConnectionPolicy. De forma predeterminada, la excepción *CosmosClientException* con el código de estado 429 se devuelve tras un tiempo de espera acumulativo de 30 segundos si la solicitud sigue funcionando por encima de la tasa de solicitudes. Esto sucede incluso cuando el número de reintentos actual es inferior al número de reintentos máximo de 9, el valor predeterminado, o un valor definido por el usuario.

    Aunque el comportamiento de reintento automático ayuda a mejorar la resistencia y la usabilidad en la mayoría de las aplicaciones, podría no resultar ventajoso al realizar comparativas de rendimiento, en especial al medir la latencia. La latencia observada del cliente aumentará si el experimento llega a la limitación del servidor y hace que el SDK del cliente realice reintentos de forma silenciosa. Para evitar aumentos de latencia durante los experimentos de rendimiento, mida el gasto devuelto por cada operación y asegúrese de que las solicitudes funcionan por debajo de la tasa de solicitudes observada. Para más información, consulte [Unidades de solicitud](request-units.md).

* **Diseño de documentos más pequeños para un mayor rendimiento**

    El gasto de solicitud (es decir, el costo de procesamiento de solicitudes) de una operación dada está directamente correlacionado con el tamaño del documento. Las operaciones con documentos grandes cuestan más que las operaciones con documentos pequeños. Idealmente, diseñe la aplicación y los flujos de trabajo para que el tamaño del elemento sea ~1 KB, o un orden o una magnitud similares. En el caso de las aplicaciones dependientes de la latencia, deben evitarse los elementos de gran tamaño; los documentos de varios MB ralentizarán la aplicación.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo diseñar la aplicación para escalarla y obtener un alto rendimiento, consulte [Partición y escalado en Azure Cosmos DB](partition-data.md).
