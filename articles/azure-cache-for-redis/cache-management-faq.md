---
title: Preguntas frecuentes sobre la administración de Azure Cache for Redis
description: Conozca las respuestas a preguntas comunes que le ayudarán a administrar Azure Cache for Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 08/06/2020
ms.openlocfilehash: 5c5c7a5adae9891f764f714d1700c6024376de02
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88205390"
---
# <a name="azure-cache-for-redis-management-faqs"></a>Preguntas frecuentes sobre la administración de Azure Cache for Redis
En este artículo se proporcionan respuestas a preguntas habituales sobre cómo administrar Azure Cache for Redis.

## <a name="common-questions-and-answers"></a>Preguntas y respuestas habituales
En esta sección se incluyen las siguientes preguntas frecuentes:

* [¿Cuándo se debe habilitar el puerto que no es TLS/SSL para la conexión a Redis?](#when-should-i-enable-the-non-tlsssl-port-for-connecting-to-redis)
* [¿Cuáles son algunas prácticas recomendadas de producción?](#what-are-some-production-best-practices)
* [¿Cuáles son las consideraciones que deben tenerse en cuenta al usar los comandos de Redis comunes?](#what-are-some-of-the-considerations-when-using-common-redis-commands)
* [¿Cómo se pueden realizar bancos de pruebas y probar el rendimiento del caché?](#how-can-i-benchmark-and-test-the-performance-of-my-cache)
* [Detalles importantes sobre el crecimiento del grupo de subprocesos](#important-details-about-threadpool-growth)
* [Habilitación de GC del servidor para mejorar el rendimiento del cliente cuando se usa StackExchange.Redis](#enable-server-gc-to-get-more-throughput-on-the-client-when-using-stackexchangeredis)
* [Consideraciones sobre rendimiento de las conexiones](#performance-considerations-around-connections)

### <a name="when-should-i-enable-the-non-tlsssl-port-for-connecting-to-redis"></a>¿Cuándo se debe habilitar el puerto que no es TLS/SSL para la conexión a Redis?
El servidor Redis no admite TLS de forma nativa, pero Azure Cache for Redis, sí. Si se conecta a Azure Cache for Redis y el cliente admite TLS, como StackExchange.Redis, deberá utilizar TLS.

>[!NOTE]
>El puerto no TLS está deshabilitado de forma predeterminada para instancias nuevas de Azure Cache for Redis. Si el cliente no admite TLS, debe habilitar el puerto no TLS siguiendo las instrucciones que se indican en la sección [Access ports](cache-configure.md#access-ports) (Puertos de acceso) del artículo [Configuración de Azure Cache for Redis](cache-configure.md).
>
>

Las herramientas de Redis, como `redis-cli`, no funcionan con el puerto TLS. Sin embargo, puede usar una utilidad (por ejemplo, `stunnel`) para conectar de forma segura las herramientas con el puerto TLS. Para ello, siga las instrucciones que se describen en la publicación del blog [Announcing ASP.NET Session State Provider for Redis Preview Release](https://devblogs.microsoft.com/aspnet/announcing-asp-net-session-state-provider-for-redis-preview-release/) (Comunicación del proveedor de estado de la sesión de ASP.NET en la versión preliminar de Redis).

Para obtener instrucciones acerca de cómo descargar las herramientas de Redis, consulte la sección [¿Cómo puedo ejecutar comandos de Redis?](cache-development-faq.md#how-can-i-run-redis-commands) .

### <a name="what-are-some-production-best-practices"></a>¿Cuáles son algunas prácticas recomendadas de producción?
* [Prácticas recomendadas de StackExchange.Redis](#stackexchangeredis-best-practices)
* [Configuración y conceptos](#configuration-and-concepts)
* [Pruebas de rendimiento](#performance-testing)

#### <a name="stackexchangeredis-best-practices"></a>Prácticas recomendadas de StackExchange.Redis
* Establecer `AbortConnect` en false y deje que el ConnectionMultiplexer se vuelva a conectar automáticamente. [Haga clic aquí para obtener información detallada](https://gist.github.com/JonCole/36ba6f60c274e89014dd#file-se-redis-setabortconnecttofalse-md).
* Reutilice el ConnectionMultiplexer; no cree uno nuevo para cada solicitud. Se recomienda el patrón `Lazy<ConnectionMultiplexer>`[mostrado aquí](cache-dotnet-how-to-use-azure-redis-cache.md#connect-to-the-cache).
* Redis funciona mejor con valores más pequeños, por lo que puede cortar los datos más grandes en varias claves. En [esta discusión de Redis](https://groups.google.com/forum/#!searchin/redis-db/size/redis-db/n7aa2A4DZDs/3OeEPHSQBAAJ), 100 kB se considera "grande". Lea [este artículo](https://gist.github.com/JonCole/db0e90bedeb3fc4823c2#large-requestresponse-size) para ver un problema de ejemplo que puede deberse a valores grandes.
* Configure [ThreadPool](#important-details-about-threadpool-growth) para evitar que se agoten los tiempos de espera.
* Utilice al menos el valor de connectTimeout predeterminado de 5 segundos. Este intervalo da a StackExchange.Redis tiempo suficiente para volver a establecer la conexión, en caso de una interrupción momentánea de la red.
* Tenga en cuenta los costos de rendimiento de las diferentes operaciones que se estén ejecutando. Por ejemplo, el comando `KEYS` es una operación O(n) y debe evitarse. El [sitio redis.io](https://redis.io/commands/) tiene información sobre la complejidad de tiempo de cada operación admitida. Haga clic en cada comando para ver la complejidad de cada operación.

#### <a name="configuration-and-concepts"></a>Configuración y conceptos
* Use el nivel Estándar o Premium en sistemas de producción. El nivel básico es un sistema de nodo único sin replicación de datos ni Acuerdo de Nivel de Servicio. Además, utilice al menos una caché de C1. Las cachés C0 se usan normalmente en escenarios de desarrollo o pruebas sencillos.
* Recuerde que Redis es un almacén de datos **en memoria** . Lea [este artículo](https://gist.github.com/JonCole/b6354d92a2d51c141490f10142884ea4#file-whathappenedtomydatainredis-md) para ser consciente de los escenarios donde puede producirse la pérdida de datos.
* Desarrolle el sistema para que pueda controlar interrupciones momentáneas de conexión [debido a la aplicación de revisiones y la conmutación por error](https://gist.github.com/JonCole/317fe03805d5802e31cfa37e646e419d#file-azureredis-patchingexplained-md).

#### <a name="performance-testing"></a>Pruebas de rendimiento
* Empiece utilizando `redis-benchmark.exe` para hacerse una idea del rendimiento posible antes de escribir sus propias pruebas de rendimiento. Dado que `redis-benchmark` no admite TLS, debe [habilitar el puerto no TLS mediante Azure Portal](cache-configure.md#access-ports) antes de ejecutar la prueba. Para ver ejemplos, consulte [¿Cómo se pueden realizar bancos de pruebas y probar el rendimiento del caché?](#how-can-i-benchmark-and-test-the-performance-of-my-cache)
* El cliente para pruebas de máquina virtual debe estar en la misma región que la instancia de Azure Cache for Redis.
* Se recomienda usar la serie de máquina virtual Dv2 para su cliente, ya que tiene mejor hardware y logrará los mejores resultados.
* Asegúrese de que la VM cliente que elija tenga al menos tantas características de computación y de ancho de banda como la memoria caché que se esté probando.
* Habilite VRSS en el equipo cliente si se encuentra en Windows. [Haga clic aquí para obtener información detallada](https://technet.microsoft.com/library/dn383582.aspx).
* Las instancias de Redis de nivel Premium tienen mejor latencia de red y rendimiento porque se ejecutan en un hardware mejor de CPU y red.

### <a name="what-are-some-of-the-considerations-when-using-common-redis-commands"></a>¿Cuáles son algunas de las consideraciones al usar los comandos de Redis comunes?

* Evite usar determinados comandos de Redis que tardan mucho tiempo en completarse a menos que conozca el efecto que pueden tener estos comandos. Por ejemplo, no ejecute el comando [KEYS](https://redis.io/commands/keys) en producción. Dependiendo del número de claves, puede tardar mucho tiempo en completarse. Redis es un servidor de un único subproceso y procesa los comandos uno a la vez. Si tiene otros comandos emitidos después de KEYS, no se procesarán hasta que Redis procese el comando KEYS. El [sitio redis.io](https://redis.io/commands/) tiene información sobre la complejidad de tiempo de cada operación admitida. Haga clic en cada comando para ver la complejidad de cada operación.
* Tamaños de clave: ¿debo usar claves/valores pequeños o claves/valores grandes? Depende del escenario. Si su escenario requiere claves de mayor tamaño, puede ajustar el valor de ConnectionTimeout y los valores de reintento y ajustar la lógica de reintento. Desde la perspectiva del servidor Redis, los valores menores ofrecen un mejor rendimiento.
* Estas consideraciones no significan que no pueda almacenar valores mayores en Redis, simplemente debe tenerlas en cuenta. Las latencias serán mayores. Si tiene un conjunto de datos mayor y otro más pequeño, puede usar varias instancias de ConnectionMultiplexer, cada una configurado con un conjunto diferente de valores de tiempo de expiración y reintentos, tal como se describe en la sección anterior [¿Qué hacen las opciones de configuración de StackExchange.Redis?](cache-development-faq.md#what-do-the-stackexchangeredis-configuration-options-do) .

### <a name="how-can-i-benchmark-and-test-the-performance-of-my-cache"></a>¿Cómo se pueden realizar bancos de pruebas y probar el rendimiento del caché?
* [Habilite los diagnósticos de la memoria caché](cache-how-to-monitor.md#enable-cache-diagnostics) para que pueda [supervisar](cache-how-to-monitor.md) su estado. Puede ver las métricas en Azure Portal y también [descargarlas y revisarlas](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring) mediante las herramientas que prefiera.
* Puede utilizar redis-benchmark.exe para la prueba de carga del servidor Redis.
* Asegúrese de que el cliente para la prueba de carga y la instancia de Azure Cache for Redis se encuentran la misma región.
* Use redis cli.exe y supervise la memoria caché mediante el comando INFO.
* Si la carga provoca una elevada fragmentación de memoria, debe escalar verticalmente a un tamaño mayor de caché.
* Para obtener instrucciones acerca de cómo descargar las herramientas de Redis, consulte la sección [¿Cómo puedo ejecutar comandos de Redis?](cache-development-faq.md#how-can-i-run-redis-commands) .

Los siguientes comandos proporcionan un ejemplo de uso de redis benchmark.exe. Para obtener resultados precisos, ejecute estos comandos desde una máquina virtual de la misma región que la caché.

* Pruebe solicitudes SET con canalización con una carga útil de 1000

  `redis-benchmark.exe -h **yourcache**.redis.cache.windows.net -a **yourAccesskey** -t SET -n 1000000 -d 1024 -P 50`
* Pruebe solicitudes GET con canalización con una carga útil de 1000.

>[!NOTE]
> Ejecute primero la prueba SET mostrada anteriormente para rellenar la caché
>

  `redis-benchmark.exe -h **yourcache**.redis.cache.windows.net -a **yourAccesskey** -t GET -n 1000000 -d 1024 -P 50`

### <a name="important-details-about-threadpool-growth"></a>Detalles importantes sobre el crecimiento del grupo de subprocesos
El grupo de subprocesos de CLR tiene dos tipos de subprocesos: subprocesos de "trabajo" y subprocesos de "puertos de terminación de E/S" (lo que se conoce como IOCP).

* Los subprocesos de trabajo se utilizan para aspectos como el procesamiento de los métodos `Task.Run(…)` o `ThreadPool.QueueUserWorkItem(…)`. Estos subprocesos también se utilizan en varios componentes del CLR cuando el trabajo se debe ejecutar en un subproceso en segundo plano.
* Los subprocesos IOCP se usan cuando se produce E/S asincrónica (por ejemplo, cuando se lee de la red).

El grupo de subprocesos proporciona nuevos subprocesos de trabajo o de terminación de E/S a petición (sin limitación) hasta que se llega a la configuración "mínima" de cada tipo de subproceso. De forma predeterminada, el número mínimo de subprocesos se establece en el número de procesadores en un sistema.

Cuando el número de subprocesos existentes (ocupado) alcanza el número "mínimo" de subprocesos, el grupo de subprocesos limitará la velocidad a la que inserta nuevos subprocesos a un subproceso por 500 milisegundos. Normalmente, si su sistema obtiene una ráfaga de trabajo que necesita un subproceso IOCP, ese trabajo se procesará muy rápidamente. Sin embargo, si la ráfaga de trabajo es mayor que la configuración "mínima", habrá cierto retraso en el procesamiento de parte del trabajo ya que el grupo de subprocesos espera a que pasen dos cosas:

* Un subproceso existente queda libre para procesar el trabajo.
* Ningún subproceso existente queda libre durante 500 ms, por lo que se crea un nuevo subproceso.

Básicamente, esto significa que cuando el número de subprocesos ocupados es mayor que los subprocesos mínimos, es probable que experimente un retraso de 500 ms antes de que la aplicación procese el tráfico de red. Además, es importante tener en cuenta que, cuando un subproceso existente permanece inactivo durante más de 15 segundos (según lo que yo recuerdo), se elimina, y este ciclo de crecimiento y merma se puede repetir.

Si examinamos un mensaje de error de ejemplo de StackExchange.Redis (compilación 1.0.450 o posterior), verá que ahora se imprimen estadísticas del grupo de subprocesos (consulte a continuación los detalles de trabajo e IOCP).

```
System.TimeoutException: Timeout performing GET MyKey, inst: 2, mgr: Inactive,
queue: 6, qu: 0, qs: 6, qc: 0, wr: 0, wq: 0, in: 0, ar: 0,
IOCP: (Busy=6,Free=994,Min=4,Max=1000),
WORKER: (Busy=3,Free=997,Min=4,Max=1000)
```

En el ejemplo anterior, puede ver que para el subproceso de IOCP hay seis subprocesos ocupados y el sistema está configurado para permitir cuatro subprocesos mínimos. En este caso, el cliente probablemente habría visto dos retrasos de 500 ms porque 6 > 4.

Tenga en cuenta que StackExchange.Redis puede alcanzar los tiempos de espera si se limita el crecimiento de los subprocesos de trabajo o de IOCP.

#### <a name="recommendation"></a>Recomendación

Dada esta información, se recomienda encarecidamente que los clientes establezcan el valor de configuración mínimo para los subprocesos IOPC y de trabajo en un valor algo mayor que el predeterminado. No podemos dar una orientación exacta sobre cuál debe ser este valor porque el que sea correcto para una aplicación puede ser demasiado alto o bajo para otra. Esta configuración también puede afectar al rendimiento de otras partes de aplicaciones complicadas, por lo que cada cliente debe ajustar este valor de acuerdo con sus necesidades específicas. Un buen punto de partida es 200 o 300, y luego probar y ajustar según sea necesario.

Cómo configurar este valor:

* Se recomienda cambiar esta configuración mediante programación con el método [ThreadPool.SetMinThreads (...)](/dotnet/api/system.threading.threadpool.setminthreads#System_Threading_ThreadPool_SetMinThreads_System_Int32_System_Int32_) de `global.asax.cs`. Por ejemplo:

    ```csharp
    private readonly int minThreads = 200;
    void Application_Start(object sender, EventArgs e)
    {
        // Code that runs on application startup
        AreaRegistration.RegisterAllAreas();
        RouteConfig.RegisterRoutes(RouteTable.Routes);
        BundleConfig.RegisterBundles(BundleTable.Bundles);
        ThreadPool.SetMinThreads(minThreads, minThreads);
    }
    ```

    > [!NOTE]
    > El valor especificado por este método es una configuración global, que afecta a todo AppDomain. Por ejemplo, si tiene una máquina de 4 núcleos y desea establecer *minWorkerThreads* y *minIOThreads* a 50 por CPU durante el tiempo de ejecución, usaría **ThreadPool.SetMinThreads (200, 200)** .

* También es posible especificar el valor mínimo de subprocesos mediante el conjunto de configuración [*minIoThreads* o *minWorkerThreads*](https://msdn.microsoft.com/library/vstudio/7w2sway1(v=vs.100).aspx) del elemento de configuración `<processModel>` de `Machine.config`, que normalmente está ubicado en `%SystemRoot%\Microsoft.NET\Framework\[versionNumber]\CONFIG\`. **Por lo general, no se recomienda establecer el número mínimo de subprocesos de esta forma, ya que es una configuración que afecta a todo el sistema.**

  > [!NOTE]
  > El valor especificado en este elemento de configuración es *por núcleo*. Por ejemplo, si tiene una máquina de 4 núcleos y quiere que el valor de *minIOThreads* sea 200 en tiempo de ejecución, use `<processModel minIoThreads="50"/>`.
  >

### <a name="enable-server-gc-to-get-more-throughput-on-the-client-when-using-stackexchangeredis"></a>Habilitación de GC del servidor para obtener más rendimiento en el cliente cuando se usa StackExchange.Redis
La habilitación de GC del servidor puede optimizar el cliente y mejorar el rendimiento y la capacidad cuando se usa StackExchange.Redis. Para más información sobre GC del servidor y cómo habilitarlo, consulte los siguientes artículos.

* [Para habilitar GC del servidor](/dotnet/framework/configure-apps/file-schema/runtime/gcserver-element)
* [Fundamentos de la recolección de elementos no utilizados](/dotnet/standard/garbage-collection/fundamentals)
* [Recolección de elementos no utilizados y rendimiento](/dotnet/standard/garbage-collection/performance)

### <a name="performance-considerations-around-connections"></a>Consideraciones sobre rendimiento de las conexiones

Cada plan de tarifa tiene distintos límites para las conexiones de cliente, memoria y ancho de banda. Si bien cada tamaño de caché permite *hasta* cierta cantidad de conexiones, cada conexión a Redis tiene asociada una sobrecarga. Un ejemplo de dicha sobrecarga podría ser el uso de memoria y CPU como resultado del cifrado TLS/SSL. El límite máximo de conexiones para un tamaño de caché determinado supone una caché con poca carga. Si la carga proveniente de la sobrecarga de conexiones *más* la carga proveniente de las operaciones de clientes supera la capacidad del sistema, la caché puede tener problemas de capacidad incluso si no ha excedido el límite de conexiones para el tamaño de la caché actual.

Para más información acerca de los diferentes límites de conexión para cada nivel, consulte [Precios de Azure Cache for Redis](https://azure.microsoft.com/pricing/details/cache/). Para más información sobre las conexiones y otras configuraciones predeterminadas, consulte [Configuración predeterminada del servidor Redis](cache-configure.md#default-redis-server-configuration).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre otras [preguntas frecuentes sobre Azure Cache for Redis](cache-faq.md).
