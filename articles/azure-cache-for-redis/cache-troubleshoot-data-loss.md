---
title: Solución de problemas de pérdida de datos en Azure Cache for Redis
description: Obtenga información sobre cómo resolver problemas de pérdida de datos con Azure Cache for Redis, como la pérdida parcial de claves, la expiración de claves o la pérdida completa de claves.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 10/17/2019
ms.openlocfilehash: 29492ee6b7bce50c4807a36d0c252e18e6aadf87
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008957"
---
# <a name="troubleshoot-data-loss-in-azure-cache-for-redis"></a>Solución de problemas de pérdida de datos en Azure Cache for Redis

En este artículo se describe cómo diagnosticar pérdidas de datos reales o percibidas que se pueden producir en Redis Cache.

> [!NOTE]
> Algunos de los pasos de solución de problemas de esta guía incluyen instrucciones para ejecutar comandos de Redis y supervisar diversas métricas de rendimiento. Para más información e instrucciones, consulte los artículos de la sección [Información adicional](#additional-information) .
>

## <a name="partial-loss-of-keys"></a>Pérdida parcial de claves

Azure Cache for Redis no elimina aleatoriamente las claves una vez que se han almacenado en la memoria. Sin embargo, sí quita las claves en respuesta a las directivas de expiración o expulsión, así como a los comandos de eliminación de claves explícita. Es posible que las claves que se hayan escrito en el nodo principal en una instancia Premium o Estándar de Azure Cache for Redis tampoco estén disponibles inmediatamente en una réplica. Los datos se replican desde el principal a la réplica de forma asincrónica y sin bloqueo.

Si observa que las claves han desaparecido de la caché, compruebe las siguientes causas posibles:

| Causa | Descripción |
|---|---|
| [Expiración de la clave](#key-expiration) | Las claves se quitan debido a los tiempos de espera establecidos. |
| [Expulsión de la clave](#key-eviction) | Las claves se quitan en la presión de memoria. |
| [Eliminación de la clave](#key-deletion) | Las claves se quitan mediante comandos de eliminación explícita. |
| [Replicación asincrónica](#async-replication) | Las claves no están disponibles en una réplica debido a retrasos de replicación de datos. |

### <a name="key-expiration"></a>Expiración de la clave

Azure Cache for Redis quita automáticamente una clave si se le asigna un tiempo de espera y se ha superado ese período. Para obtener más información sobre la expiración de las claves de Redis, consulte la documentación del comando [EXPIRE](https://redis.io/commands/expire). Los valores de tiempo de expiración también se pueden establecer mediante los comandos [SET](https://redis.io/commands/set), [SETEX](https://redis.io/commands/setex), [GETSET](https://redis.io/commands/getset) y otros comandos **\*STORE**.

Para obtener estadísticas sobre cuántas claves han expirado, use el comando [INFO](https://redis.io/commands/info). La sección `Stats` muestra el número total de claves expiradas. La sección `Keyspace` proporciona más información sobre el número de claves con tiempos de expiración y el valor de tiempo de expiración promedio.

```
# Stats

expired_keys:46583

# Keyspace

db0:keys=3450,expires=2,avg_ttl=91861015336
```

También puede consultar las métricas de diagnóstico de la memoria caché para ver si hay una correlación entre el momento en que falta la clave y un pico en las claves expiradas. Consulte el Anexo de [Depuración de errores de espacios de claves](https://gist.github.com/JonCole/4a249477142be839b904f7426ccccf82#appendix) para obtener información sobre el uso de notificaciones de espacio de claves o **MONITOR** para depurar este tipo de problemas.

### <a name="key-eviction"></a>Expulsión de la clave

Azure Cache for Redis requiere espacio en memoria para almacenar datos. Purga las claves para liberar memoria disponible cuando sea necesario. Cuando los valores **used_memory** o **used_memory_rss** del comando [INFO](https://redis.io/commands/info) se aproximan al valor **maxmemory** configurado, Azure Cache for Redis comienza a expulsar las claves de la memoria en función de la [directiva de caché](https://redis.io/topics/lru-cache).

Puede supervisar el número de claves expulsadas con el comando [INFO](https://redis.io/commands/info):

```
# Stats

evicted_keys:13224
```

También puede consultar las métricas de diagnóstico de la memoria caché para ver si hay una correlación entre el momento en que falta la clave y un pico en las claves expulsadas. Consulte el Anexo de [Depuración de errores de espacios de claves](https://gist.github.com/JonCole/4a249477142be839b904f7426ccccf82#appendix) para obtener información sobre el uso de notificaciones de espacio de claves o **MONITOR** para depurar este tipo de problemas.

### <a name="key-deletion"></a>Eliminación de la clave

Los clientes de Redis pueden emitir el comando [DEL](https://redis.io/commands/del) o [HDEL](https://redis.io/commands/hdel) para quitar claves de Azure Cache for Redis explícitamente. Puede realizar un seguimiento del número de operaciones de eliminación mediante el comando [INFO](https://redis.io/commands/info). Si se ha llamado a los comandos **DEL** o **HDEL**, aparecerán en la sección `Commandstats`.

```
# Commandstats

cmdstat_del:calls=2,usec=90,usec_per_call=45.00

cmdstat_hdel:calls=1,usec=47,usec_per_call=47.00
```

### <a name="async-replication"></a>Replicación asincrónica

Cualquier instancia de Azure Cache for Redis en el nivel Estándar o Premium se configura con un nodo principal y al menos una réplica. Los datos se copian desde el principal a una réplica de forma asincrónica mediante un proceso en segundo plano. En el sitio web [redis.io](https://redis.io/topics/replication) se describe cómo funciona la replicación de datos de Redis en general. En los escenarios en los que los clientes escriben en Redis con frecuencia, se puede producir una pérdida parcial de datos porque no se garantiza que esta replicación será instantánea. Por ejemplo, si el principal deja de estar disponible *después* de que un cliente escriba una clave en él pero *antes* de que el proceso en segundo plano tenga ocasión de enviar esta clave a la réplica, la clave se perderá cuando la réplica tome el control como nuevo principal.

## <a name="major-or-complete-loss-of-keys"></a>Pérdida principal o completa de claves

Si todas las claves o la mayoría de estas han desaparecido de la caché, compruebe las siguientes causas posibles:

| Causa | Descripción |
|---|---|
| [Vaciado de la clave](#key-flushing) | Las claves se han purgado manualmente. |
| [Selección incorrecta de la base de datos](#incorrect-database-selection) | Azure Cache for Redis está configurado para usar una base de datos no predeterminada. |
| [Error de instancia de Redis](#redis-instance-failure) | El servidor de Redis no está disponible. |

### <a name="key-flushing"></a>Vaciado de la clave

Los clientes pueden llamar al comando [FLUSHDB](https://redis.io/commands/flushdb) para quitar todas las claves de una *sola* base de datos o [FLUSHALL](https://redis.io/commands/flushall) para quitar todas las claves de *todas* las bases de datos de una caché en Redis. Para averiguar si las claves se han vaciado, use el comando [INFO](https://redis.io/commands/info). En la sección `Commandstats` se muestra si se ha llamado al comando **FLUSH**:

```
# Commandstats

cmdstat_flushall:calls=2,usec=112,usec_per_call=56.00

cmdstat_flushdb:calls=1,usec=110,usec_per_call=52.00
```

### <a name="incorrect-database-selection"></a>Selección incorrecta de la base de datos

Redis Cache utiliza la base de datos **db0** de manera predeterminada. Si cambia a otra base de datos (por ejemplo, **db1**) e intenta leer sus claves, Azure Cache for Redis no las encontrará allí. Cada base de datos es una unidad independiente lógicamente y contiene un conjunto de datos diferente. Use el comando [SELECT](https://redis.io/commands/select) para usar otras bases de datos disponibles y buscar claves en cada una de ellas.

### <a name="redis-instance-failure"></a>Error de instancia de Redis

Redis es un almacén de datos en memoria. Los datos se mantienen en las máquinas físicas o virtuales que hospedan la caché de Redis. Una instancia de Redis Cache en el nivel Básico se ejecuta en una sola máquina virtual (VM). Si esa máquina virtual deja de estar disponible, se pierden todos los datos que ha almacenado en la caché. 

Las memorias caché de los niveles Estándar y Premium ofrecen una resistencia mucho mayor frente a la pérdida de datos mediante el uso de dos máquinas virtuales en una configuración replicada. Cuando se produce un error en el nodo principal de dicha caché, el nodo de réplica se encarga de servir los datos automáticamente. Estas máquinas virtuales se encuentran en dominios de errores y actualizaciones independientes para minimizar la posibilidad de que ambas no estén disponibles al mismo tiempo. Aunque, si se produce una interrupción importante del centro de datos, las máquinas virtuales pueden dejar de estar disponibles a la vez. Los datos se perderán en estos casos excepcionales.

Considere la posibilidad de utilizar [Permanencia de datos de Redis](https://redis.io/topics/persistence) y [replicación geográfica](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-geo-replication) para mejorar la protección de sus datos contra estos fallos de infraestructura.

## <a name="additional-information"></a>Información adicional

- [Solución de problemas del lado servidor de Redis Cache](cache-troubleshoot-server.md)
- [Elección del nivel correcto](cache-overview.md#choosing-the-right-tier)
- [Supervisión de Azure Cache for Redis](cache-how-to-monitor.md)
- [¿Cómo puedo ejecutar comandos de Redis?](cache-development-faq.md#how-can-i-run-redis-commands)
