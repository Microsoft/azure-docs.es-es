---
title: Supervisión de Azure Cache for Redis | Microsoft Docs
description: Aprenda a supervisar el estado y el rendimiento de las instancias de Azure Redis Cache for Redis
services: cache
documentationcenter: ''
author: yegu-ms
manager: jhubbard
editor: ''
ms.assetid: 7e70b153-9c87-4290-85af-2228f31df118
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.author: yegu
ms.openlocfilehash: 2cfd5a99144af1120afbf06fe6222228a9332bb6
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787424"
---
# <a name="how-to-monitor-azure-cache-for-redis"></a>Supervisión de Azure Cache for Redis
Azure Cache for Redis usa [Azure Monitor](https://docs.microsoft.com/azure/monitoring-and-diagnostics/) para proporcionar varias opciones para la supervisión de instancias de caché. Puede ver las métricas, anclar los gráficos de métricas al panel de inicio, personalizar el intervalo de fecha y hora de los gráficos de supervisión, agregar y quitar métricas de los gráficos y establecer alertas cuando se cumplen determinadas condiciones. Estas herramientas permiten supervisar el estado de las instancias de Azure Cache for Redis y le ayudan a administrar sus aplicaciones de almacenamiento en caché.

Las métricas de las instancias de Azure Cache for Redis se recopilan mediante el comando [INFO](https://redis.io/commands/info) de Redis aproximadamente dos veces por minuto y se almacenan automáticamente durante 30 días [consulte [Export cache metrics](#export-cache-metrics) (Exportación de métricas de caché) para configurar una directiva de retención diferente] por lo que se pueden mostrar en los gráficos de métricas y las pueden evaluar las reglas de alerta. Para obtener más información sobre los distintos valores INFO que se utilizan para las métricas de caché, consulte [Métricas disponibles e intervalos de informes](#available-metrics-and-reporting-intervals).

<a name="view-cache-metrics"></a>

Para ver las métricas de caché, [busque](cache-configure.md#configure-azure-cache-for-redis-settings) la instancia de su memoria caché en [Azure Portal](https://portal.azure.com).  Azure Cache for Redis proporciona varios gráficos integrados en las hoja **Introducción** y **Métricas de Redis**. Se puede personalizar cada gráfico agregando o quitando métricas y cambiando el intervalo de informes.

![Métricas de Redis](./media/cache-how-to-monitor/redis-cache-redis-metrics-blade.png)

## <a name="view-pre-configured-metrics-charts"></a>Visualización de gráficos de métricas preconfigurados

La hoja **Introducción** tiene los siguientes gráficos de supervisión preconfigurados.

* [Gráficos de supervisión](#monitoring-charts)
* [Gráficos de uso](#usage-charts)

### <a name="monitoring-charts"></a>Gráficos de supervisión
La sección **Supervisión** de la hoja **Introducción** contiene los gráficos **Aciertos y errores**, **Obtenciones y configuraciones**, **Conexiones** y **Comandos totales**.

![Gráficos de supervisión](./media/cache-how-to-monitor/redis-cache-monitoring-part.png)

### <a name="usage-charts"></a>Gráficos de uso
La sección **Uso** de la hoja **Introducción** contiene los gráficos **Carga del servidor Redis**, **Uso de memoria**, **Ancho de banda de la red** y **Uso de CPU**, y muestra el **plan de tarifa** de la instancia de caché.

![Gráficos de uso](./media/cache-how-to-monitor/redis-cache-usage-part.png)

El **Nivel de precios** muestra el nivel de precios de caché y se puede utilizar para [escalar](cache-how-to-scale.md) la memoria caché a un nivel de precios diferente.

## <a name="view-metrics-with-azure-monitor"></a>Visualización de métricas con Azure Monitor
Para ver las métricas de Redis y crear gráficos personalizados mediante Azure Monitor, haga clic en **Métrica** en el **menú Recursos** y personalice el gráfico con las métricas deseadas, el intervalo de generación de informes, el tipo de gráfico, etc.

![Métricas de Redis](./media/cache-how-to-monitor/redis-cache-monitor.png)

Para más información acerca de cómo trabajar con métricas mediante Azure Monitor, consulte [Información general sobre las métricas en Microsoft Azure](../monitoring-and-diagnostics/monitoring-overview-metrics.md).

<a name="how-to-view-metrics-and-customize-chart"></a>
<a name="enable-cache-diagnostics"></a>
## <a name="export-cache-metrics"></a>Exportación de métricas de caché
De manera predeterminada, en Azure Monitor las métricas de caché se [almacenan 30 días](../azure-monitor/platform/data-platform-metrics.md) y, después, se eliminan. Para conservar las métricas de memoria caché durante más de 30 días, puede [designar una cuenta de almacenamiento](../azure-monitor/platform/archive-diagnostic-logs.md) y especifique un **retención (días)** directiva para las métricas de memoria caché. 

Para configurar una cuenta de almacenamiento para las métricas de caché:

1. Haga clic en **Diagnósticos** en el **menú Recursos** en la hoja **Azure Cache for Redis**.
2. Haga clic en **Activado**.
3. Seleccione **Archivar en una cuenta de almacenamiento**.
4. Seleccione la cuenta de almacenamiento en la que va a almacenar las métricas de caché.
5. Seleccione la casilla **1 minuto** y especifique una directiva **Retención (días)**. Si no desea aplicar una directiva de retención y desea conserva los datos de forma permanente, establezca **Retención (días)** en **0**.
6. Haga clic en **Save**(Guardar).

![Diagnósticos de Redis](./media/cache-how-to-monitor/redis-cache-diagnostics.png)

>[!NOTE]
>Además de archivar las métricas de caché en el almacenamiento, también puede [transmitirlos a Event hubs o enviarlos a los registros de Azure Monitor](../azure-monitor/platform/rest-api-walkthrough.md#retrieve-metric-values).
>
>

Para acceder a las métricas, puede verlas en Azure Portal como ya se ha descrito en este mismo artículo, pero también puede acceder a ellas mediante la [API de REST de métricas de Azure Monitor](../azure-monitor/platform/stream-monitoring-data-event-hubs.md).

> [!NOTE]
> Si cambia las cuentas de almacenamiento, los datos de la cuenta de almacenamiento configurada anteriormente siguen estando disponibles para su descarga, pero no se muestran en el Portal de Azure.  
> 
> 

## <a name="available-metrics-and-reporting-intervals"></a>Métricas disponibles e intervalos de informes
Los informes de las métricas de caché se generan utilizando diferentes intervalos, como **Última hora**, **Hoy**, **Semana anterior** y **Personalizado**. La hoja **Métrica** para cada gráfico de métricas muestra los valores promedio, mínimos y máximo para cada métrica del gráfico, y algunas métricas muestran un total para el intervalo de informes. 

Cada métrica incluye dos versiones. Una métrica mide el rendimiento de toda la memoria caché. En el caso de las memorias caché que utilizan la [agrupación en clústeres](cache-how-to-premium-clustering.md), se usa otra versión de la métrica que incluye `(Shard 0-9)` en el nombre para medir el rendimiento de una sola partición de la memoria caché. Por ejemplo, si una memoria caché tiene 4 particiones, `Cache Hits` es la cantidad total de resultados en toda la memoria caché, y `Cache Hits (Shard 3)` refleja simplemente los resultados para esa partición de la memoria caché.

> [!NOTE]
> Incluso cuando la memoria caché está inactiva sin ninguna aplicación de cliente activa conectada, puede que vea alguna actividad de caché, como clientes conectados, uso de memoria y operaciones que se están realizando. Esta actividad es normal durante la operación de una instancia de Azure Cache for Redis.
> 
> 

| Métrica | DESCRIPCIÓN |
| --- | --- |
| Aciertos de caché |El número de búsquedas de claves correctas durante el intervalo de informes. Este valor se asigna a `keyspace_hits` desde el comando [INFO](https://redis.io/commands/info) de Redis. |
| Latencia de caché (versión preliminar) | Latencia de la memoria caché calculada a partir de la latencia entre nodos de la memoria caché. Esta métrica se mide en microsegundos y tiene tres dimensiones: "Promedio", "Mín." y "Máx.", que representan la latencia promedio, mínima y máxima de la memoria caché, respectivamente, durante el intervalo de informes especificado. |
| Errores de caché |El número de búsquedas de claves incorrectas durante el intervalo de informes. Este valor se asigna a `keyspace_misses` desde el comando INFO de Redis. Los errores de caché no significan necesariamente que haya un problema con la memoria caché. Por ejemplo, cuando se utiliza el modelo de programación cache-aside, una aplicación busca un elemento en primer lugar en la memoria caché. Si el elemento no está allí (error de caché), se recupera de la base de datos y se agrega a la caché para la próxima vez. Los errores de caché son un comportamiento normal del modelo de programación cache-aside. Si el número de errores de caché es mayor de lo esperado, examine la lógica de aplicación que rellena y lee de la memoria caché. Si se expulsan los elementos de la memoria caché debido a la presión de memoria, puede haber algunos errores de caché; una métrica mejor para supervisar la presión de memoria sería `Used Memory` o `Evicted Keys`. |
| Lectura de caché |La cantidad de datos que se leen de la memoria caché en megabytes por segundo (MB/s) durante el intervalo de informes especificado. Este valor se deriva de las tarjetas de interfaz de red que admiten la máquina virtual que hospeda la caché y no es específica de Redis. **Este valor corresponde al ancho de banda de red que emplea esta caché. Si desea configurar alertas para los límites de ancho de banda de red del lado servidor, hágalo mediante este contador `Cache Read`. Consulte [esta tabla](cache-faq.md#cache-performance) para conocer los límites de ancho de banda de los diferentes tamaños y planes de tarifa de caché.** |
| Escritura de caché |La cantidad de datos que se escriben en la memoria caché en megabytes por segundo (MB/s) durante el intervalo de informes especificado. Este valor se deriva de las tarjetas de interfaz de red que admiten la máquina virtual que hospeda la caché y no es específica de Redis. Este valor corresponde al ancho de banda de red de los datos enviados a la memoria caché desde el cliente. |
| Clientes conectados |El número de conexiones de clientes a la caché durante el intervalo de informes especificado. Este valor se asigna a `connected_clients` desde el comando INFO de Redis. Cuando se alcanza el [límite de conexión](cache-configure.md#default-redis-server-configuration) , se producirá un error en los intentos de conexión posteriores a la caché. Tenga en cuenta que, incluso si no hay ninguna aplicación de cliente activa, puede haber algunas instancias de clientes conectadas debido a procesos y conexiones internos. |
| CPU |El uso de CPU del servidor de Azure Cache for Redis como porcentaje durante el intervalo de informes especificado. Este valor se asigna al contador de rendimiento `\Processor(_Total)\% Processor Time` del sistema operativo. |
| Errors | Errores específicos y problemas de rendimiento que puede experimentar la memoria caché durante un intervalo de notificación especificado. Esta métrica tiene ocho dimensiones que representan los diferentes tipos de errores, pero se podrían agregar más en el futuro. Los tipos de error que se representan ahora son los siguientes: <br/><ul><li>**Conmutación por error** : cuando conmuta por error una memoria caché (subordinado se promueve al maestro)</li><li>**Bloqueo**: la memoria caché se bloquea de forma inesperada en cualquiera de los nodos</li><li>**Pérdida de datos**: se produce una pérdida de datos en la memoria caché</li><li>**UnresponsiveClients**: los clientes no leen los datos del servidor lo suficientemente rápido</li><li>**AOF**: hay un problema relacionado con la persistencia de AOF</li><li>**RDB**: hay un problema relacionado con la persistencia de RDB</li><li>**Importación**: hay un problema relacionado con la importación de RDB</li><li>**Exportación**: hay un problema relacionado con la exportación de RDB</li></ul> |
| Claves expulsadas |El número de elementos que se elimina de la caché durante el intervalo de informes especificado debido al límite `maxmemory` . Este valor se asigna a `evicted_keys` desde el comando INFO de Redis. |
| Claves expiradas |El número de elementos expirados de la caché durante el intervalo de informes especificado. Este valor se asigna a `expired_keys` desde el comando INFO de Redis.|
| Obtenciones |El número de operaciones get de la caché durante el intervalo de informes especificado. Este valor es la suma de los siguientes valores de todos los comandos INFO de Redis: `cmdstat_get`, `cmdstat_hget`, `cmdstat_hgetall`, `cmdstat_hmget`, `cmdstat_mget`, `cmdstat_getbit` y `cmdstat_getrange`, y es equivalente a la suma de aciertos y errores de caché durante el intervalo de informes. |
| Operaciones por segundo | El número total de comandos procesados por segundo por el servidor de caché durante el intervalo de informes especificado.  Este valor se asigna a "instantaneous_ops_per_sec" desde el comando INFO de Redis. |
| Carga del servidor Redis |El porcentaje de ciclos en el que el servidor de Redis está ocupado procesando y no esperando a los mensajes inactivo. Si este contador llega a 100, significa que el servidor de Redis ha llegado a un límite de rendimiento y la CPU no puede procesar el trabajo más rápidamente. Si ve alta carga del servidor de Redis, verá las excepciones de tiempo de espera en el cliente. En este caso, debería considerar la posibilidad de un escalado vertical o el particionamiento de los datos en varias cachés. |
| Conjuntos |El número de operaciones set a la caché durante el intervalo de informes especificado. Este valor es la suma de los siguientes valores de todos los comandos INFO de Redis: `cmdstat_set`, `cmdstat_hset`, `cmdstat_hmset`, `cmdstat_hsetnx`, `cmdstat_lset`, `cmdstat_mset`, `cmdstat_msetnx`, `cmdstat_setbit`, `cmdstat_setex`, `cmdstat_setrange` y `cmdstat_setnx`. |
| Total de claves  | El número máximo de claves en la memoria caché durante el período de generación de informes anterior. Este valor se asigna a `keyspace` desde el comando INFO de Redis. Debido a una limitación del sistema de métricas subyacente, en el caso de las memorias caché con la agrupación en clústeres habilitada, Total de claves devuelve el número máximo de claves de la partición que tenía el número máximo de claves durante el intervalo de generación de informes.  |
| Operaciones totales |El número total de comandos procesados por el servidor de caché durante el intervalo de informes especificado. Este valor se asigna a `total_commands_processed` desde el comando INFO de Redis. Tenga en cuenta que cuando se utiliza Azure Cache for Redis simplemente para publicación y suscripción, no habrá ninguna métrica para `Cache Hits`, `Cache Misses`, `Gets` o `Sets`, pero habrá métricas `Total Operations` que reflejan el uso de la memoria caché para operaciones de publicación y suscripción. |
| Memoria usada |La cantidad de memoria caché usada para pares clave-valor en la memoria caché en MB durante el intervalo de informes especificado. Este valor se asigna a `used_memory` desde el comando INFO de Redis. Esto no incluye los metadatos o la fragmentación. |
| Porcentaje de memoria usado | % del total de memoria que se usa durante el intervalo de notificación especificado.  Esto hace referencia al valor de "used_memory" del comando INFO de Redis para calcular el porcentaje. |
| Memoria RSS usada |La cantidad de memoria caché usada en MB durante el intervalo de informes especificado, incluida la fragmentación y los metadatos. Este valor se asigna a `used_memory_rss` desde el comando INFO de Redis. |

<a name="operations-and-alerts"></a>
## <a name="alerts"></a>Alertas
Puede configurar la recepción de alertas en función de métricas y registros de actividad. Azure Monitor permite configurar una alerta que haga lo siguiente cuando se desencadena:

* Enviar una notificación por correo electrónico
* Llamar a un webhook
* Invocar una aplicación lógica de Azure

Para configurar reglas de alerta para la memoria caché, haga clic en **Reglas de alerta** en el **menú Recursos**.

![Supervisión](./media/cache-how-to-monitor/redis-cache-monitoring.png)

Para más información acerca de la configuración y uso de alertas, consulte [Información general de las alertas](../monitoring-and-diagnostics/insights-alerts-portal.md).

## <a name="activity-logs"></a>Registros de actividad
Los registros de actividad proporcionan información sobre las operaciones llevadas a cabo en las instancias de Azure Cache for Redis. Antes se los conocía como "registros de auditoría" o "registros operativos". Con los registros de actividades, se puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en las instancias de Azure Cache for Redis. 

> [!NOTE]
> Los registros de actividad no incluyen operaciones de lectura (GET).
>
>

Para ver los registros de actividad de la memoria caché, haga clic en **Registros de actividad** en el **menú Recursos**.

Para más información acerca de los registros de actividad, consulte [Información general sobre el registro de actividad de Azure](../azure-monitor/platform/activity-logs-overview.md).











