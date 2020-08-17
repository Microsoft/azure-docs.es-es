---
title: Solución de problemas de tiempos de expiración de Redis Cache
description: Aprenda a resolver problemas comunes de tiempo de espera con Azure Cache for Redis, como la aplicación de revisiones al servidor Redis y las excepciones de tiempo de espera de StackExchange.Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 10/18/2019
ms.openlocfilehash: efe175e4086d5273471c1b0451e4cfb28449c236
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008940"
---
# <a name="troubleshoot-azure-cache-for-redis-timeouts"></a>Solución de problemas de tiempos de expiración de Redis Cache

En esta sección se describe la solución de problemas de tiempo de expiración que se producen al conectarse a Redis Cache.

- [Aplicación de revisiones del servidor de Redis](#redis-server-patching)
- [Excepciones de tiempo de expiración de StackExchange.Redis](#stackexchangeredis-timeout-exceptions)

> [!NOTE]
> Algunos de los pasos de solución de problemas de esta guía incluyen instrucciones para ejecutar comandos de Redis y supervisar diversas métricas de rendimiento. Para más información e instrucciones, consulte los artículos de la sección [Información adicional](#additional-information) .
>

## <a name="redis-server-patching"></a>Aplicación de revisiones del servidor de Redis

Redis Cache actualiza periódicamente su software de servidor como parte de la funcionalidad de servicio administrado que ofrece. Esta actividad de [aplicación de revisiones](cache-failover.md) tiene lugar en gran medida en segundo plano. Durante las conmutaciones por error cuando se aplican revisiones a los nodos del servidor de Redis, es posible que los clientes de Redis conectados a estos nodos experimenten tiempos de expiración temporales a medida que se cambian las conexiones entre estos nodos. Vea [cómo afecta una conmutación por error a la aplicación cliente](cache-failover.md#how-does-a-failover-affect-my-client-application) para obtener más información sobre qué efectos secundarios puede tener la aplicación y cómo puede mejorar su control de los eventos de aplicación de revisiones.

## <a name="stackexchangeredis-timeout-exceptions"></a>Excepciones de tiempo de espera de StackExchange.Redis

StackExchange.Redis usa una opción de configuración llamada `synctimeout` para operaciones sincrónicas, con un valor predeterminado de 5000 ms. Si no se completa una llamada sincrónica en este tiempo, el cliente de StackExchange.Redis genera un error de tiempo de expiración similar al ejemplo siguiente:

```output
    System.TimeoutException: Timeout performing MGET 2728cc84-58ae-406b-8ec8-3f962419f641, inst: 1,mgr: Inactive, queue: 73, qu=6, qs=67, qc=0, wr=1/1, in=0/0 IOCP: (Busy=6, Free=999, Min=2,Max=1000), WORKER (Busy=7,Free=8184,Min=2,Max=8191)
```

Este mensaje de error contiene métricas que pueden indicarle la causa y la posible solución del problema. La tabla siguiente contiene detalles acerca de las métricas del mensaje de error.

| Métrica del mensaje de error | Detalles |
| --- | --- |
| inst |En el último período de tiempo: se han emitido 0 comandos. |
| mgr |El administrador del socket está realizando `socket.select`, lo que significa que está solicitando al sistema operativo que indique un socket que tenga algo que hacer. El lector no está leyendo activamente de la red porque cree que no hay nada que hacer. |
| cola |Hay un total de 73 operaciones en curso. |
| qu |6 de las operaciones en curso están en la cola de no enviados y aún no se han escrito en la red de salida. |
| qs |67 de las operaciones en curso se han enviado al servidor, pero aún no hay una respuesta disponible. La respuesta podría ser `Not yet sent by the server` o `sent by the server but not yet processed by the client.`. |
| qc |0 de las operaciones en curso han visto respuestas, pero aún no se han marcado como completadas porque están esperando en el bucle de finalización. |
| wr |Hay un escritor activo (lo que significa que 6 solicitudes sin enviar no se omiten) bytes/activewriters. |
| bucear |No hay ningún lector activo y hay cero bytes disponibles para su lectura en los bytes/activereaders de NIC. |

Puede usar los pasos siguientes para investigar posibles causas principales.

1. Como procedimiento recomendado, asegúrese de usar el patrón siguiente para conectarse al usar el cliente de StackExchange.Redis.

    ```csharp
    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
    {
        return ConnectionMultiplexer.Connect("cachename.redis.cache.windows.net,abortConnect=false,ssl=true,password=...");

    });

    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }
    ```

    Para más información, consulte [Uso de Caché en Redis de Azure](cache-dotnet-how-to-use-azure-redis-cache.md#connect-to-the-cache).

1. Asegúrese de que el servidor y la aplicación cliente están en la misma región de Azure. Por ejemplo, podría obtener tiempos de expiración cuando la caché está en Este de EE. UU., pero el cliente está en Oeste de EE. UU. y no se completa la solicitud en el intervalo `synctimeout`, o bien podría obtener tiempos de expiración cuando realiza la depuración en la máquina de desarrollo local. 

    Se recomienda encarecidamente que la memoria caché y el cliente estén en la misma región de Azure. Si tiene un escenario que incluye llamadas entre regiones, debe establecer el intervalo `synctimeout` a un valor mayor que el intervalo predeterminado de 5000 ms mediante la inclusión de una propiedad `synctimeout` en la cadena de conexión. En el ejemplo siguiente se muestra un fragmento de una cadena de conexión para StackExchange.Redis que proporciona Azure Redis Cache con un `synctimeout` de 2000 ms.

    ```output
    synctimeout=2000,cachename.redis.cache.windows.net,abortConnect=false,ssl=true,password=...
    ```

1. Asegúrese de usar la versión más reciente del [paquete StackExchange.Redis NuGet](https://www.nuget.org/packages/StackExchange.Redis/). Hay errores que se solucionan constantemente en el código que lo hacen más solido frente a tiempos de espera agotados, por lo que es importante tener la versión más reciente.
1. Si las solicitudes se enlazan con las limitaciones de ancho de banda en el servidor o el cliente, tardan más en completarse y podrían provocar tiempos de expiración. Para ver si el tiempo de expiración se debe al ancho de banda de red del servidor, consulte [Limitación de ancho de banda del lado servidor](cache-troubleshoot-server.md#server-side-bandwidth-limitation). Para ver si el tiempo de expiración se debe al ancho de banda de red del cliente, consulte [Limitación de ancho de banda del lado cliente](cache-troubleshoot-client.md#client-side-bandwidth-limitation).
1. ¿la CPU está limitada en el servidor o en el cliente?

   - Compruebe si la CPU le está limitando en el cliente. Una CPU elevada podría provocar que la solicitud no se procese en el intervalo `synctimeout` y causar una solicitud para agotarse el tiempo de espera. Cambiar a un tamaño de cliente mayor o distribuir la carga puede ayudar a controlar este problema.
   - Supervise la [métrica de rendimiento de caché](cache-how-to-monitor.md#available-metrics-and-reporting-intervals) de la CPU para comprobar si la CPU está enlazada al servidor. Las solicitudes entrantes mientras Redis está limitado por la CPU pueden provocar que dichas solicitudes agoten el tiempo de espera. Para solucionar este problema, puede distribuir la carga entre varias particiones en una memoria caché premium o actualizar a un tamaño o plan de tarifa mayor. Para obtener más información, consulte [Limitación de ancho de banda del lado servidor](cache-troubleshoot-server.md#server-side-bandwidth-limitation).
1. ¿Hay comandos que tardan mucho tiempo en procesarse en el servidor? Los comandos de larga duración que tardan mucho tiempo en procesarse en el servidor de Redis pueden provocar tiempos de expiración. Para obtener más información sobre los comandos de ejecución prolongada, consulte [Comandos de ejecución prolongada](cache-troubleshoot-server.md#long-running-commands). Puede conectarse a la instancia de Azure Redis Cache mediante el cliente redis-cli o la [Consola de Redis](cache-configure.md#redis-console). Luego, ejecute el comando [SLOWLOG](https://redis.io/commands/slowlog) para ver si hay solicitudes más lentas de lo esperado. El servidor de Redis y StackExchange.Redis están optimizados para muchas solicitudes pequeñas, en lugar de menos solicitudes de gran tamaño. Dividir los datos en fragmentos menores puede mejorar las cosas aquí.

    Para obtener información sobre cómo conectarse al punto de conexión TLS/SSL de la caché con redis-cli y stunnel, vea la entrada de blog [Anuncio de proveedor de estado de sesión ASP.NET para la versión preliminar de Redis](https://devblogs.microsoft.com/aspnet/announcing-asp-net-session-state-provider-for-redis-preview-release/).
1. Una carga alta del servidor de Redis puede causar tiempos de espera agotados. Puede supervisar la carga del servidor con la `Redis Server Load` [métrica de rendimiento de caché](cache-how-to-monitor.md#available-metrics-and-reporting-intervals). Una carga del servidor de 100 (valor máximo) significa que el servidor de redis ha estado ocupado procesando solicitudes, sin tiempo de inactividad. Para ver si ciertas solicitudes ocupan toda la funcionalidad del servidor, ejecute el comando SlowLog, como se describe en el párrafo anterior. Para más información, consulte Uso elevado de la CPU/carga de servidor.
1. ¿Ha habido cualquier otro evento en el lado cliente que puede haber causado una señalización visual de red? Los eventos comunes incluyen: escalado o reducción vertical del número de instancias de cliente, implementación de una nueva versión del cliente o escalado automático habilitado. En nuestra prueba hemos observado que el escalado automático o el escalado o reducción vertical pueden provocar que la conectividad de red de salida se pierda durante varios segundos. El código de StackExchange.Redis es resistente a dichos eventos y se vuelve a conectar. Mientras se produce la reconexión, las solicitudes en la cola pueden agotar el tiempo de espera.
1. ¿Ha habido una solicitud grande antes de varias solicitudes pequeñas en la caché que haya agotado el tiempo de espera? El parámetro `qs` en el mensaje de error indica cuántas solicitudes se enviaron del cliente al servidor pero que no han procesado una respuesta. Este valor puede seguir creciendo, ya que StackExchange.Redis usa una sola conexión de TCP y solo puede leer una respuesta cada vez. Aunque la primera operación ha agotado el tiempo de espera, esto no detiene el envío de más datos al servidor o desde este. Se bloquearán otras solicitudes hasta que la solicitud grande finalice y pueda provocar que se agote el tiempo de espera. Una solución es reducir la posibilidad de tiempos de espera agotados, garantizando que la memoria caché sea lo suficientemente grande para la carga de trabajo y dividiendo los valores grandes en fragmentos menores. Otra posible solución es utilizar un grupo de objetos `ConnectionMultiplexer` en el cliente y elegir el parámetro `ConnectionMultiplexer` con menos carga al enviar una solicitud nueva. La carga en varios objetos de conexión debería evitar que un único tiempo de expiración haga que otras solicitudes también agoten el tiempo de espera.
1. Si está utilizando `RedisSessionStateProvider`, asegúrese de haber configurado correctamente el tiempo de expiración de los reintentos. `retryTimeoutInMilliseconds` debe ser superior a `operationTimeoutInMilliseconds`; de lo contrario, no se producirá ningún reintento. En el siguiente ejemplo, `retryTimeoutInMilliseconds` se establece en 3000. Para más información, consulte [ASP.NET Session State Provider for Azure Cache for Redis](cache-aspnet-session-state-provider.md) (Proveedor de estado de sesión de ASP.NET para Azure Cache for Redis) y [How to use the configuration parameters of Session State Provider and Output Cache Provider](https://github.com/Azure/aspnet-redis-providers/wiki/Configuration) (Uso de los parámetros de configuración del proveedor de estado de sesión y el proveedor de caché de resultados).

    ```xml
    <add
      name="AFRedisCacheSessionStateProvider"
      type="Microsoft.Web.Redis.RedisSessionStateProvider"
      host="enbwcache.redis.cache.windows.net"
      port="6380"
      accessKey="…"
      ssl="true"
      databaseId="0"
      applicationName="AFRedisCacheSessionState"
      connectionTimeoutInMilliseconds = "5000"
      operationTimeoutInMilliseconds = "1000"
      retryTimeoutInMilliseconds="3000" />
    ```

1. Compruebe el uso de la memoria en el servidor de Azure Cache for Redis mediante la [supervisión](cache-how-to-monitor.md#available-metrics-and-reporting-intervals) de `Used Memory RSS` y `Used Memory`. Si hay una directiva de expulsión implementada, Redis empieza a expulsar las claves cuando `Used_Memory` alcanza el tamaño de la memoria caché. Idealmente, `Used Memory RSS` solo debe ser ligeramente mayor que `Used memory`. Una gran diferencia significa que hay fragmentación de memoria (interna o externa). Cuando `Used Memory RSS` es menor que `Used Memory`, significa que el sistema operativo ha intercambiado parte de la memoria caché. Si se produce este intercambio, puede esperar latencias importantes. Dado que Redis no tiene control sobre cómo se realizan sus asignaciones a las páginas, a menudo un parámetro `Used Memory RSS` elevado suele ser el resultado de un pico en la utilización de memoria. Cuando el servidor de Redis libera memoria, el asignador toma la memoria, pero puede devolver la memoria al sistema o no hacerlo. Puede haber una discrepancia entre el valor de `Used Memory` y el consumo de memoria comunicado por el sistema operativo. Es posible que Redis haya usado y liberado la memoria, pero no la haya devuelto al sistema. Puede realizar los pasos siguientes para ayudar a mitigar las incidencias de memoria:

   - Actualizar la caché a un tamaño mayor para que no se ejecute con limitaciones de memoria en el sistema.
   - Establecer tiempos de expiración de las claves para que los valores antiguos se expulsen de forma proactiva.
   - Supervisar la métrica de la memoria caché `used_memory_rss`. Cuando este valor se aproxima al tamaño de la caché, es probable que empiece a ver incidencias de rendimiento. Distribuir los datos entre varias particiones, si utiliza una caché premium, o actualizar a un tamaño de caché mayor.

   Para obtener más información, consulte [Presión de memoria en el servidor de Redis](cache-troubleshoot-server.md#memory-pressure-on-redis-server).

## <a name="additional-information"></a>Información adicional

- [Solución de problemas del lado cliente de Redis Cache](cache-troubleshoot-client.md)
- [Solución de problemas del lado servidor de Redis Cache](cache-troubleshoot-server.md)
- [¿Cómo se pueden realizar bancos de pruebas y probar el rendimiento del caché?](cache-management-faq.md#how-can-i-benchmark-and-test-the-performance-of-my-cache)
- [Supervisión de Azure Cache for Redis](cache-how-to-monitor.md)
