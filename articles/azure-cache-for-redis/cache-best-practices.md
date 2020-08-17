---
title: Procedimientos recomendados para Azure Cache for Redis
description: Obtenga información sobre cómo usar de forma eficaz Azure Cache for Redis mediante los siguientes procedimientos recomendados.
author: joncole
ms.service: cache
ms.topic: conceptual
ms.date: 01/06/2020
ms.author: joncole
ms.openlocfilehash: 0ed0009bce18e2b0970b425c31d2f38cef387187
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87008326"
---
# <a name="best-practices-for-azure-cache-for-redis"></a>Procedimientos recomendados para Azure Cache for Redis 
Si sigue estos procedimientos recomendados, puede maximizar el rendimiento y rentabilizar el uso de la instancia de Azure Cache for Redis.

## <a name="configuration-and-concepts"></a>Configuración y conceptos
 * **Use el nivel Estándar o Premium en sistemas de producción.**  El nivel básico es un sistema de nodo único sin replicación de datos ni Acuerdo de Nivel de Servicio. Además, utilice al menos una caché de C1.  Las cachés C0 están diseñadas para escenarios de desarrollo y prueba simples, ya que tienen un núcleo de CPU compartido, poca memoria y son propensas a tener problemas de "vecinos ruidosos".

 * **Recuerde que Redis es un almacén de datos en memoria.**  [En este artículo](cache-troubleshoot-data-loss.md) se describen algunos escenarios donde pueden perderse datos.

 * **Desarrolle su sistema para que pueda controlar los errores de conexión** [debido a la revisión y la conmutación por error](cache-failover.md).

 * **Configure la opción [maxmemory-reserved](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) para mejorar la capacidad de respuesta del sistema** en situaciones de presión de memoria.  Una configuración de reserva suficiente es especialmente importante para las cargas de trabajo con muchas operaciones de escritura o si almacena valores más grandes (100 KB o más) en Redis. Debe empezar con el 10 % del tamaño de la memoria caché y aumentar este porcentaje si tiene cargas con mucha actividad de escritura.

 * **Redis funciona mejor con valores más pequeños**, por lo que puede cortar los datos más grandes en varias claves.  En [este artículo de Redis](https://stackoverflow.com/questions/55517224/what-is-the-ideal-value-size-range-for-redis-is-100kb-too-large/) se enumeran algunas opciones que debe considerar cuidadosamente.  Lea [este artículo](cache-troubleshoot-client.md#large-request-or-response-size) para ver un problema de ejemplo que puede deberse a valores grandes.

 * **Localice su instancia de la caché y su aplicación en la misma región.**  Si se conecta a una caché de una región diferente puede aumentar significativamente la latencia y reducir la confiabilidad.  Si bien puede conectarse desde fuera de Azure, no es recomendable, *especialmente cuando se usa Redis como caché*.  Si está usando Redis solo como un almacén de clave/valor, es posible que la latencia no sea su principal preocupación. 

 * **Reutilizar las conexiones.**  La creación de nuevas conexiones es costosa y aumenta la latencia, por lo que puede reutilizar las conexiones lo máximo posible. Si prefiere crear nuevas conexiones, asegúrese de cerrar las conexiones anteriores antes de liberarlas (incluso en lenguajes de memoria administrados como .NET o Java).

 * **Configure su biblioteca cliente para usar un *tiempo de espera de la conexión* de al menos 15 segundos**, lo que le dará al sistema tiempo para conectarse incluso en condiciones donde la CPU se use más.  Tener un pequeño valor de tiempo de espera de conexión no garantiza que la conexión se establezca en ese plazo de tiempo.  Si algo sale mal (CPU de cliente alta, CPU de servidor alta, etc.), un valor de tiempo de espera de conexión pequeño hará que el intento de conexión falle. Este comportamiento suele empeorar una situación que ya es mala de por si.  En lugar de mejorar el problema, los tiempos de espera más cortos lo agravan, ya que obligan al sistema a reiniciar el proceso de conexión, lo que puede llevar a un bucle *conectar -> error -> reintentar*. En general, le recomendamos que deje el tiempo de espera de conexión en 15 segundos o más. Es mejor dejar que su intento de conexión tenga éxito después de 15 o 20 segundos que hacer que falle rápidamente para volver a intentarlo. Un bucle de reintentos de este tipo puede hacer que la interrupción dure más que si otorgara inicialmente más tiempo al sistema.  
     > [!NOTE]
     > En esta guía se detallan los *intentos de conexión* y no está relacionada con el tiempo que está dispuesto a esperar para que se complete una *operación* como GET o SET.
 
 * **Evite las operaciones caras**: algunas operaciones de Redis, como el comando [KEYS](https://redis.io/commands/keys), son *muy* caras y deben evitarse.  Para más información, consulte algunas opciones acerca de los [comandos de larga duración](cache-troubleshoot-server.md#long-running-commands).

 * **Use el cifrado TLS**: Redis Cache requiere comunicaciones cifradas TLS de forma predeterminada.  Actualmente se admiten las versiones de TLS 1.0, 1.1 y 1.2.  Sin embargo, TLS 1.0 y 1.1 están en proceso de desuso en todo el sector, por lo que se recomienda TLS 1.2 si es posible.  Si la biblioteca o la herramienta cliente no admiten TLS, se pueden habilitar las conexiones no cifradas [mediante Azure Portal](cache-configure.md#access-ports) o las [API de administración](https://docs.microsoft.com/rest/api/redis/redis/update).  En casos en los que no es posible establecer conexiones cifradas, se recomienda colocar la caché y la aplicación cliente en una red virtual.  Para más información sobre los puertos que se usan en el escenario de caché de la red virtual, consulte esta [tabla](cache-how-to-premium-vnet.md#outbound-port-requirements).
 
 * **Tiempo de espera de inactividad**: actualmente, Azure Redis tiene un tiempo de espera de inactividad de 10 minutos para las conexiones, por lo que debe establecerse en menos de 10 minutos.
 
## <a name="memory-management"></a>Administración de memoria
Es posible que quiera tener en cuenta varias cosas relacionadas con el uso de la memoria dentro de la instancia del servidor de Redis.  Estas son algunas:

 * **Elija una [directiva de expulsión](https://redis.io/topics/lru-cache) que funcione para su aplicación.**  La directiva predeterminada de Azure Redis es *volatile-lru*, lo que significa que solo las claves que tienen un valor de TTL establecido se podrán usar en la expulsión.  Si ninguna de las claves tiene un valor de TTL, el sistema no expulsará ninguna clave.  Si quiere que el sistema permita que se expulse cualquier clave si la memoria está bajo presión, puede usar la directiva *allkeys-lru*.

 * **Establezca un valor de expiración en sus claves.**  Una expiración quitará las claves de forma proactiva en lugar de esperar hasta que haya presión de memoria.  Cuando la expulsión se activa debido a la presión en la memoria, esto puede causar una carga adicional en el servidor.  Para más información, consulte la documentación de los comandos [Expire](https://redis.io/commands/expire) y [EXPIREAT](https://redis.io/commands/expireat).
 
## <a name="client-library-specific-guidance"></a>Guía específica de la biblioteca del cliente
 * [StackExchange.Redis (.NET)](https://gist.github.com/JonCole/925630df72be1351b21440625ff2671f#file-redis-bestpractices-stackexchange-redis-md)
 * [Java: ¿Qué cliente debo usar?](https://gist.github.com/warrenzhu25/1beb02a09b6afd41dff2c27c53918ce7#file-azure-redis-java-best-practices-md)
 * [Lettuce (Java)](https://gist.github.com/warrenzhu25/181ccac7fa70411f7eb72aff23aa8a6a#file-azure-redis-lettuce-best-practices-md)
 * [Jedis (Java)](https://gist.github.com/JonCole/925630df72be1351b21440625ff2671f#file-redis-bestpractices-java-jedis-md)
 * [Node.js](https://gist.github.com/JonCole/925630df72be1351b21440625ff2671f#file-redis-bestpractices-node-js-md)
 * [PHP](https://gist.github.com/JonCole/925630df72be1351b21440625ff2671f#file-redis-bestpractices-php-md)
 * [Proveedor de estado de sesión de ASP.NET](https://gist.github.com/JonCole/925630df72be1351b21440625ff2671f#file-redis-bestpractices-session-state-provider-md)


## <a name="when-is-it-safe-to-retry"></a>¿Cuándo es seguro volver a intentarlo?
Lamentablemente, no hay ninguna respuesta fácil.  Cada aplicación debe decidir qué operaciones se pueden volver a intentar y cuáles no.  Cada operación tiene diferentes requisitos y dependencias entre claves.  Estos son algunos aspectos que hay que tener en cuenta:

 * Puede obtener errores del lado cliente aunque Redis ejecute con éxito el comando que le pidió que ejecutara.  Por ejemplo:
     - Los tiempos de espera son un concepto del lado cliente.  Si la operación llegó al servidor, este ejecutará el comando incluso si el cliente deja de esperar.  
     - Cuando se produce un error en la conexión de socket, no es posible saber si la operación realmente se ejecutó en el servidor.  Por ejemplo, el error de conexión puede producirse después de que el servidor haya procesado la solicitud, pero antes de que el cliente reciba la respuesta.
 *  ¿Cómo reacciona mi aplicación si accidentalmente ejecuto la misma operación dos veces?  Por ejemplo, ¿qué ocurre si se incrementa un entero dos veces en lugar de una vez?  ¿Mi aplicación está escribiendo en la misma clave desde varios lugares?  ¿Qué sucede si mi lógica de reintentos sobrescribe un valor que estableció alguna otra parte de mi aplicación?

Si desea probar cómo funciona el código en condiciones de error, considere la posibilidad de usar la [Característica de reinicio](cache-administration.md#reboot). El reinicio permite ver cómo afectan los señales de conexión a la aplicación.

## <a name="performance-testing"></a>Pruebas de rendimiento
 * **Use `redis-benchmark.exe`** para hacerse una idea del rendimiento/latencia posibles antes de escribir sus propias pruebas de rendimiento.  La documentación del banco de pruebas de Redis se puede encontrar [aquí](https://redis.io/topics/benchmarks).  Tenga en cuenta que el banco de pruebas de Redis no admite TLS, por lo que tendrá que [habilitar un puerto que no sea TLS a través de Azure Portal](cache-configure.md#access-ports) antes de ejecutar la prueba.  [Aquí se puede encontrar una versión compatible con Windows de redis-benchmark.exe](https://github.com/MSOpenTech/redis/releases)
 * El cliente para pruebas de VM debe estar en la **misma región** que la instancia de caché de Redis.
 * **Se recomienda usar la serie de VM Dv2** para su cliente, ya que tiene mejor hardware y logrará los mejores resultados.
 * Asegúrese de que la VM del cliente que use tenga **al menos tantos procesos y ancho de banda* como la memoria caché que se está probando. 
 * **Habilite VRSS** en el equipo cliente si usa Windows.  [Haga clic aquí para obtener información detallada](https://technet.microsoft.com/library/dn383582(v=ws.11).aspx).  Script de PowerShell de ejemplo:
     >PowerShell -ExecutionPolicy Unrestricted Enable-NetAdapterRSS -Name (    Get-NetAdapter).Name 
     
 * **Use instancias de Redis de nivel Premium**.  Los tamaños de la caché tendrán mejor latencia de red y rendimiento porque se ejecutan en un hardware mejor de CPU y red.
 
     > [!NOTE]
     > Los resultados de rendimiento observados se [publicarán aquí](cache-faq.md#azure-cache-for-redis-performance) para que los tenga como referencia.   Además, tenga en cuenta que SSL/TLS agrega cierta sobrecarga, por lo que puede obtener diferentes latencias o rendimiento si está usando el cifrado de transporte.
 
### <a name="redis-benchmark-examples"></a>Ejemplos del banco de pruebas de Redis
**El programa de configuración de la prueba previa**: Prepare la instancia de caché con los datos necesarios para los comandos de prueba de latencia y rendimiento que se enumeran a continuación.
> redis-benchmark -h yourcache.redis.cache.windows.net -a yourAccesskey -t SET -n 10 -d 1024 

**Para probar la latencia**: Pruebe las solicitudes GET con una carga de 1 k.
> redis-benchmark -h yourcache.redis.cache.windows.net -a yourAccesskey -t GET -d 1024 -P 50 -c 4

**Para probar el rendimiento:** Solicitudes GET canalizadas con carga de 1 K.
> redis-benchmark -h yourcache.redis.cache.windows.net -a yourAccesskey -t  GET -n 1000000 -d 1024 -P 50  -c 50
