---
title: Configuración de Azure Cache for Redis
description: Descripción de la configuración predeterminada de Redis para Azure Cache for Redis y más información sobre cómo configurar las instancias de Azure Cache for Redis
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 08/22/2017
ms.author: yegu
ms.openlocfilehash: 7459d674cde123bc45544322347bc4c1fe89e820
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009620"
---
# <a name="how-to-configure-azure-cache-for-redis"></a>Configuración de Azure Cache for Redis
En este tema se describe la configuración disponible para las instancias de Azure Cache for Redis. En este tema también se describe la configuración predeterminada del servidor Redis para las instancias de Azure Cache for Redis.

> [!NOTE]
> Para más información sobre cómo configurar y usar las características de caché premium, consulte [Cómo configurar la persistencia](cache-how-to-premium-persistence.md), [Cómo configurar la agrupación en clústeres un clúster](cache-how-to-premium-clustering.md) y [Cómo configurar la compatibilidad con Virtual Network](cache-how-to-premium-vnet.md).
>
>

## <a name="configure-azure-cache-for-redis-settings"></a>Configuración de las opciones de Azure Cache for Redis
[!INCLUDE [redis-cache-create](../../includes/redis-cache-browse.md)]

La configuración de Azure Redis Cache se ve y ajusta en la hoja **Azure Cache for Redis** mediante el **menú Recursos**.

![Configuración de Azure Cache for Redis](./media/cache-configure/redis-cache-settings.png)

Puede ver y configurar las siguientes opciones con el **menú Recursos**.

* [Información general](#overview)
* [Registro de actividad](#activity-log)
* [Control de acceso (IAM)](#access-control-iam)
* [Etiquetas](#tags)
* [Diagnóstico y solución de problemas](#diagnose-and-solve-problems)
* [Configuración](#settings)
    * [Claves de acceso](#access-keys)
    * [Configuración avanzada](#advanced-settings)
    * [Azure Cache for Redis Advisor](#azure-cache-for-redis-advisor)
    * [Escala](#scale)
    * [Tamaño del clúster](#cluster-size)
    * [Persistencia de los datos](#redis-data-persistence)
    * [Programación de actualizaciones](#schedule-updates)
    * [Replicación geográfica](#geo-replication)
    * [Virtual Network](#virtual-network)
    * [Firewall](#firewall)
    * [Propiedades](#properties)
    * [Bloqueos](#locks)
    * [Script de Automation](#automation-script)
* Administración
    * [Importar datos](#importexport)
    * [Exportación de datos](#importexport)
    * [Reboot](#reboot)
* [Supervisión](#monitoring)
    * [Métricas de Redis](#redis-metrics)
    * [Reglas de alertas](#alert-rules)
    * [Diagnóstico](#diagnostics)
* Configuración de soporte técnico y solución de problemas
    * [Estado de los recursos](#resource-health)
    * [Nueva solicitud de soporte técnico](#new-support-request)


## <a name="overview"></a>Información general

**Información general** proporciona también con información básica sobre la memoria caché, como el nombre, puertos, plan de tarifa y las métricas de la caché seleccionada.

### <a name="activity-log"></a>Registro de actividades

Haga clic en **Registro de auditoría** para ver las acciones realizadas en la memoria caché. También puede usar el filtrado para expandir esta vista e incluir otros recursos. Para obtener más información sobre cómo trabajar con los registros de auditoría, consulte [Operaciones de auditoría con Resource Manager](../azure-resource-manager/management/view-activity-logs.md). Para más información sobre la supervisión de eventos de Azure Cache for Redis, consulte [Operaciones y alertas](cache-how-to-monitor.md#operations-and-alerts).

### <a name="access-control-iam"></a>Control de acceso (IAM)

En la sección **Control de acceso (IAM)** se proporciona soporte para el control de acceso basado en rol (RBAC) en Azure Portal. Esta configuración ayuda a las organizaciones a cumplir los requisitos de administración de acceso de forma sencilla y precisa. Para más información, consulte [Uso de asignaciones de roles para administrar el acceso a los recursos de la suscripción de Azure](../role-based-access-control/role-assignments-portal.md).

### <a name="tags"></a>Etiquetas

La sección **Etiquetas** le ayuda a organizar sus recursos. Para obtener más información, vea [Uso de etiquetas para organizar los recursos de Azure](../azure-resource-manager/management/tag-resources.md).


### <a name="diagnose-and-solve-problems"></a>Diagnosticar y solucionar problemas

Haga clic en **Diagnosticar y solucionar problemas** para ver problemas comunes y estrategias para resolverlos.



## <a name="settings"></a>Configuración
La sección **Configuración** permite acceder a los siguientes ajustes de la memoria caché y configurarlos.

* [Claves de acceso](#access-keys)
* [Configuración avanzada](#advanced-settings)
* [Azure Cache for Redis Advisor](#azure-cache-for-redis-advisor)
* [Escala](#scale)
* [Tamaño del clúster](#cluster-size)
* [Persistencia de los datos](#redis-data-persistence)
* [Programación de actualizaciones](#schedule-updates)
* [Replicación geográfica](#geo-replication)
* [Virtual Network](#virtual-network)
* [Firewall](#firewall)
* [Propiedades](#properties)
* [Bloqueos](#locks)
* [Script de Automation](#automation-script)



### <a name="access-keys"></a>Claves de acceso
Haga clic en **Claves de acceso** para ver o volver a generar las claves de acceso de la memoria caché. Estas claves las usan los clientes que se conectan a la caché.

![Claves de acceso de Azure Cache for Redis](./media/cache-configure/redis-cache-manage-keys.png)

### <a name="advanced-settings"></a>Configuración avanzada
Los siguientes ajustes se configuran en la hoja **Configuración avanzada**.

* [Puertos de acceso](#access-ports)
* [Directivas de memoria](#memory-policies)
* [Notificaciones de espacio de claves (configuración avanzada)](#keyspace-notifications-advanced-settings)

#### <a name="access-ports"></a>Puertos de acceso
El acceso no TLS/SSL está deshabilitado de forma predeterminada para las nuevas memorias caché. Para habilitar el puerto no TLS, haga clic en **No** en la opción **Permitir el acceso solo mediante SSL** de la hoja **Configuración avanzada** y seleccione **Guardar**.

> [!NOTE]
> El acceso TLS a Azure Cache for Redis admite TLS 1.0, 1.1 y 1.2 actualmente, pero las versiones 1.0 y 1.1 se retirarán pronto.  Lea la [página Quitar TLS 1.0 y 1.1](cache-remove-tls-10-11.md) para obtener más detalles.

![Puertos de acceso de Azure Cache for Redis](./media/cache-configure/redis-cache-access-ports.png)

<a name="maxmemory-policy-and-maxmemory-reserved"></a>
#### <a name="memory-policies"></a>Directivas de memoria
Las opciones **Directiva de memoria máxima** **maxmemory-reserved** y **maxfragmentationmemory-reserved** de la hoja **Configuración avanzada** establecen las directivas de memoria de la memoria caché.

![Directiva de memoria máxima de Azure Cache for Redis](./media/cache-configure/redis-cache-maxmemory-policy.png)

**Directiva de memoria máxima** configura la directiva de expulsión para la memoria caché y permite elegir entre las siguientes directivas de expulsión:

* `volatile-lru`: esta es la directiva de expulsión predeterminada.
* `allkeys-lru`
* `volatile-random`
* `allkeys-random`
* `volatile-ttl`
* `noeviction`

Para más información sobre las directivas `maxmemory`, consulte [Eviction policies](https://redis.io/topics/lru-cache#eviction-policies) (Directivas de expulsión).

La opción **maxmemory-reserved** configura la cantidad de memoria en MB que se reserva para las operaciones ajenas a la memoria caché como, por ejemplo, la replicación durante la conmutación por error. Esta opción le permite tener una experiencia más coherente de servidor Redis cuando varía la carga. Este valor debe establecerse más alto para cargas de trabajo con muchas operaciones de escritura. Cuando se reserva memoria para dichas operaciones, no está disponible para el almacenamiento de datos en caché.

La opción **maxfragmentationmemory-reserved** configura la cantidad de memoria en MB que se reserva para adaptarse a la fragmentación de memoria. Establecer este valor permite tener una experiencia más coherente con el servidor Redis cuando la caché está llena o prácticamente llena, y la proporción de fragmentación es elevada. Cuando se reserva memoria para dichas operaciones, no está disponible para el almacenamiento de datos en caché.

Al elegir un nuevo valor de reserva de memoria (**maxmemory-reserved** o **maxfragmentationmemory-reserved**) hay que tener en cuenta cómo podría afectar este cambio a una memoria caché que ya se está ejecutando con grandes cantidades de datos en ella. Por ejemplo, si tiene una memoria caché de 53 GB con 49 GB de datos, al cambiar el valor de reserva a 8 GB, esta modificación reducirá la memoria máxima disponible para el sistema a 45 GB. Si los valores actuales de `used_memory` o `used_memory_rss` son mayores que el nuevo límite de 45 GB, entonces el sistema tendrá que expulsar datos hasta que `used_memory` y `used_memory_rss` estén por debajo de 45 GB. La expulsión puede aumentar la carga del servidor y la fragmentación de memoria. Para más información sobre las métricas de caché como `used_memory` y `used_memory_rss`, vea [Métricas disponibles e intervalos de informes](cache-how-to-monitor.md#available-metrics-and-reporting-intervals).

> [!IMPORTANT]
> Las opciones **maxmemory-reserved** y **maxfragmentationmemory-reserved** solo están disponibles para las memorias caché de nivel Standard y Premium.
>
>

#### <a name="keyspace-notifications-advanced-settings"></a>Notificaciones de espacio de claves (configuración avanzada)
Las notificaciones de espacio de claves de Redis se configuran en la hoja **Configuración avanzada** . Las notificaciones de espacio de claves permiten que los clientes reciban notificaciones cuando se producen determinados eventos.

![Configuración avanzada de Azure Cache for Redis](./media/cache-configure/redis-cache-advanced-settings.png)

> [!IMPORTANT]
> Las notificaciones de espacio de claves y el valor **notify-keyspace-event** solo están disponibles para las memorias caché de nivel Standard y Premium.
>
>

Para obtener más información, vea [Notificaciones de espacio de claves de Redis](https://redis.io/topics/notifications). Para obtener el ejemplo de código, consulte el archivo [KeySpaceNotifications.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/KeySpaceNotifications.cs) del ejemplo [Hello world](https://github.com/rustd/RedisSamples/tree/master/HelloWorld).


<a name="recommendations"></a>
## <a name="azure-cache-for-redis-advisor"></a>Azure Cache for Redis Advisor
La hoja **Azure Cache for Redis Advisor** muestra recomendaciones para la memoria caché. Durante las operaciones normales, no se muestra ninguna recomendación.

![Recomendaciones](./media/cache-configure/redis-cache-no-recommendations.png)

Si se produce cualquier problema durante las operaciones de la memoria caché, como un uso elevado de la memoria, el ancho de banda de red o la carga del servidor, se muestra una alerta en la hoja **Azure Cache for Redis**.

![Recomendaciones](./media/cache-configure/redis-cache-recommendations-alert.png)

Puede encontrar más información en la hoja **Recomendaciones** .

![Recomendaciones](./media/cache-configure/redis-cache-recommendations.png)

Estas métricas se pueden supervisar en las secciones [Monitoring charts](cache-how-to-monitor.md#monitoring-charts) (Gráficos de supervisión) y [Usage charts](cache-how-to-monitor.md#usage-charts) (Gráficos de uso) de la hoja **Azure Cache for Redis**.

Cada plan de tarifa tiene distintos límites para las conexiones de cliente, memoria y ancho de banda. Si la caché se aproxima a la capacidad máxima para estas métricas durante un período prolongado, se crea una recomendación. Para más información sobre las métricas y los límites que se revisan mediante la herramienta **Recomendaciones**, consulte la tabla siguiente:

| Métrica de Azure Cache for Redis | Más información |
| --- | --- |
| Uso de ancho de banda de red |[Rendimiento de la caché: ancho de banda disponible](cache-planning-faq.md#azure-cache-for-redis-performance) |
| Clientes conectados |[Configuración predeterminada del servidor Redis: maxclients](#maxclients) |
| Carga de servidor |[Gráficos de uso: carga del servidor Redis](cache-how-to-monitor.md#usage-charts) |
| Uso de la memoria |[Rendimiento y tamaño de la memoria caché](cache-planning-faq.md#azure-cache-for-redis-performance) |

Para actualizar la memoria caché, haga clic en **Actualizar ahora** para cambiar el plan de tarifa y [escalar](#scale) la memoria caché. Para más información sobre cómo elegir un plan de tarifa, consulte [Elección del plan de tarifa adecuado](cache-overview.md#choosing-the-right-tier).


### <a name="scale"></a>Escala
Haga clic en **Escalar** para ver o cambiar el plan de tarifa para su memoria caché. Para más información sobre el escalado, consulte [How to Scale Azure Cache for Redis](cache-how-to-scale.md) (Escalado de Azure Cache for Redis).

![Plan de tarifa de Azure Cache for Redis](./media/cache-configure/pricing-tier.png)

<a name="cluster-size"></a>

### <a name="redis-cluster-size"></a>Tamaño del Clúster en Redis
Haga clic en **Tamaño del clúster** para cambiar el tamaño del clúster para una caché premium en ejecución con la agrupación de clústeres habilitada.

![Tamaño del clúster](./media/cache-configure/redis-cache-redis-cluster-size.png)

Para modificar el tamaño del clúster, utilice el control deslizante o especifique un número comprendido entre 1 y 10 en el cuadro de texto **Número de particiones**. Después, haga clic en **Aceptar** para guardar.

> [!IMPORTANT]
> La agrupación en clústeres de Redis solo está disponible para las memorias cachés premium. Para más información, consulte [How to configure clustering for a Premium Azure Cache for Redis](cache-how-to-premium-clustering.md) (Configuración de la agrupación en clústeres para una instancia de Azure Cache for Redis Prémium).
>
>


### <a name="redis-data-persistence"></a>Persistencia de datos de Redis
Haga clic en **Persistencia de los datos** para habilitar, deshabilitar o configurar la persistencia de los datos para la caché premium. Azure Cache for Redis ofrece persistencia de Redis mediante [persistencia de RDB](cache-how-to-premium-persistence.md#configure-rdb-persistence) o [persistencia de AOF](cache-how-to-premium-persistence.md#configure-aof-persistence).

Para más información, vea [How to configure persistence for a Premium Azure Cache for Redis](cache-how-to-premium-persistence.md) (Configuración de la persistencia para una instancia de Azure Cache for Redis Prémium).


> [!IMPORTANT]
> La persistencia de los datos en Redis solo está disponible para las memorias cachés premium.
>
>

### <a name="schedule-updates"></a>Programar actualizaciones
La hoja **Programar actualizaciones** permite designar una ventana de mantenimiento para las actualizaciones del servidor de Redis para la caché.

> [!IMPORTANT]
> El período de mantenimiento solo se aplica a las actualizaciones del servidor de Redis y no a las actualizaciones de Azure o del sistema operativo de las máquinas virtuales que hospedan la caché.
>
>

![Programar actualizaciones](./media/cache-configure/redis-schedule-updates.png)

Para especificar una ventana de mantenimiento, compruebe los días deseados, especifique la hora de inicio de la ventana de mantenimiento para cada día y haga clic en **Aceptar**. La hora del período de mantenimiento está en formato UTC.

> [!IMPORTANT]
> La funcionalidad **Programar actualizaciones** solo está disponible para cachés del nivel Premium. Para más información e instrucciones, consulte [Azure Cache for Redis administration - Schedule updates](cache-administration.md#schedule-updates) (Administración de Azure Cache for Redis: Programación de actualizaciones).
>
>

### <a name="geo-replication"></a>Replicación geográfica

En la hoja **Replicación geográfica** se proporciona un mecanismo para vincular dos instancias de Azure Cache for Redis de nivel Premium. Una memoria caché se designa como la caché vinculada principal y la otra, como la caché vinculada secundaria. La caché vinculada secundaria pasa a ser de solo lectura, por lo que los datos escritos en la caché principal se replican en la caché vinculada secundaria. Esta funcionalidad se puede usar para replicar una caché en varias regiones de Azure.

> [!IMPORTANT]
> La **replicación geográfica** solo está disponible para las cachés de nivel Premium. Para más información e instrucciones, consulte [How to configure Geo-replication for Azure Cache for Redis](cache-how-to-geo-replication.md) (Configuración de la replicación geográfica para Azure Cache for Redis).
>
>

### <a name="virtual-network"></a>Virtual Network
La sección **Virtual Network** le permite configurar las opciones de red virtual de la caché. Para más información sobre cómo crear una caché premium con compatibilidad para red virtual y cómo actualizar su configuración, consulte [How to configure Virtual Network Support for a Premium Azure Cache for Redis](cache-how-to-premium-vnet.md) (Configuración de la compatibilidad de red virtual para una instancia de Azure Cache for Redis Premium).

> [!IMPORTANT]
> La configuración de red virtual solo está disponible para las memorias cachés premium que se configuraron con la compatibilidad de la red virtual durante la creación de la memoria caché.
>
>

### <a name="firewall"></a>Firewall

La configuración de las reglas de firewall está disponible para todos los niveles de Azure Cache for Redis.

Haga clic en **Firewall** para ver y configurar las reglas de firewall de la memoria caché.

![Firewall](./media/cache-configure/redis-firewall-rules.png)

Puede especificar las reglas de firewall con un intervalo de direcciones IP de inicio y finalización. Cuando se configuran las reglas de firewall, solo las conexiones de cliente de los intervalos de direcciones IP especificados pueden conectarse a la memoria caché. Cuando se guarda una regla de firewall, hay un breve retraso antes de que la regla se aplique. Normalmente, este retraso es inferior a un minuto.

> [!IMPORTANT]
> Siempre se permiten las conexiones de los sistemas de supervisión de Azure Cache for Redis, incluso si se configuran reglas de firewall.
>
>

### <a name="properties"></a>Propiedades
Haga clic en **Propiedades** para ver información sobre la memoria caché, incluidos los puertos y el extremo de la caché.

![Propiedades de Azure Cache for Redis](./media/cache-configure/redis-cache-properties.png)

### <a name="locks"></a>Bloqueos
La sección **Bloqueos** permite bloquear una suscripción, un grupo de recursos o un recurso para evitar que otros usuarios de la organización eliminen o modifiquen por error recursos críticos. Para obtener más información, consulte [Bloqueo de recursos con el Administrador de recursos de Azure](../azure-resource-manager/management/lock-resources.md).

### <a name="automation-script"></a>Script de Automation

Haga clic en **Script de Automation** para generar y exportar una plantilla de los recursos implementados para futuras implementaciones. Para más información sobre cómo trabajar con plantillas, consulte [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="administration-settings"></a>Configuración de administración
La configuración de la sección **Administración** le permite llevar a cabo las siguientes tareas administrativas en la caché.

![Administración](./media/cache-configure/redis-cache-administration.png)

* [Importar datos](#importexport)
* [Exportación de datos](#importexport)
* [Reboot](#reboot)


### <a name="importexport"></a>Import/Export
Import/Export es una operación de administración de datos de Azure Cache for Redis que permite importar y exportar datos en la caché mediante la importación y exportación de una instantánea de base de datos de Azure Cache for Redis (RDB) desde una caché premium a un blob en páginas en una cuenta de Azure Storage. Import/Export permite migrar entre diferentes instancias de Azure Cache for Redis o rellenar la memoria caché de datos antes de su uso.

La importación se puede usar para traer los archivos RDB compatibles de Redis desde cualquier servidor de Redis que se ejecute en cualquier nube o entorno, incluidas las instancias de Redis que se ejecutan en Linux, Windows o cualquier proveedor de nube como, por ejemplo, Amazon Web Services. La importación de datos supone una manera fácil de crear una caché con datos rellenados previamente. Durante el proceso de importación, Azure Cache for Redis carga los archivos RDB desde Azure Storage en la memoria y, luego, inserta las claves en la memoria caché.

La exportación permite exportar los datos almacenados en Azure Cache for Redis a archivos RDB compatibles. Puede utilizar esta característica para mover datos desde una instancia de Azure Cache for Redis a otra o a otro servidor de Redis. Durante el proceso de exportación, se crea un archivo temporal en la máquina virtual que hospeda la instancia del servidor de Azure Cache for Redis y el archivo se carga en la cuenta de almacenamiento designada. Una vez completada la operación de exportación (de manera correcta o incorrecta), se elimina el archivo temporal.

> [!IMPORTANT]
> Importación/Exportación solo está disponible para las memorias caché de nivel premium. Para más información e instrucciones, consulte [Import and Export data in Azure Cache for Redis](cache-how-to-import-export-data.md) (Importación y exportación de datos en Azure Cache for Redis).
>
>

### <a name="reboot"></a>Reboot
La hoja **Reiniciar** permite reiniciar los nodos de la caché. Esta funcionalidad de reinicio le permite probar la resistencia de la aplicación en caso de que haya un error de un nodo de la caché.

![Reboot](./media/cache-configure/redis-cache-reboot.png)

Si tiene una caché premium con la agrupación en clústeres habilitada, puede seleccionar qué particiones de la memoria caché se reiniciarán.

![Reboot](./media/cache-configure/redis-cache-reboot-cluster.png)

Para reiniciar uno o varios nodos de la caché, seleccione los nodos que prefiera y haga clic en **Reboot**(Reiniciar). Si tiene una caché premium con la agrupación en clústeres habilitada, seleccione las particiones para reiniciar y haga clic en **Reboot**(Reiniciar). Después de unos minutos, los nodos seleccionados se reinician y vuelven a estar en línea poco tiempo después.

> [!IMPORTANT]
> El reinicio ahora está disponible para todos los planes de tarifas. Para más información e instrucciones, consulte [Azure Cache for Redis administration - Reboot](cache-administration.md#reboot) (Administración de Azure Cache for Redis: Reinicio).
>
>


## <a name="monitoring"></a>Supervisión

La sección **Supervisión** permite configurar el diagnóstico y supervisar Azure Cache for Redis.
Para más información sobre los diagnósticos y la supervisión de Azure Cache for Redis, consulte [How to monitor Azure Cache for Redis](cache-how-to-monitor.md) (Supervisión de Azure Cache for Redis).

![Diagnóstico](./media/cache-configure/redis-cache-diagnostics.png)

* [Métricas de Redis](#redis-metrics)
* [Reglas de alertas](#alert-rules)
* [Diagnóstico](#diagnostics)

### <a name="redis-metrics"></a>Caché en Redis
Haga clic en **Métricas de Redis** para [ver las métricas](cache-how-to-monitor.md#view-cache-metrics) de la memoria caché.

### <a name="alert-rules"></a>Las reglas de alertas

Haga clic en **Reglas de alerta** para configurar las alertas basadas en métricas de Azure Cache for Redis. Para más información, consulte [Alertas](cache-how-to-monitor.md#alerts).

### <a name="diagnostics"></a>Diagnóstico

De manera predeterminada, en Azure Monitor las métricas de caché se [almacenan durante 30 días](../azure-monitor/platform/data-platform-metrics.md) y después se eliminan. Para conservar las métricas de caché más de 30 días, haga clic en **Diagnóstico** para [configurar la cuenta de almacenamiento](cache-how-to-monitor.md#export-cache-metrics) que se usa para almacenar los diagnósticos de la memoria caché.

>[!NOTE]
>Además de archivar las métricas de caché en el almacenamiento, también puede [transmitirlas a un centro de eventos o enviarlas a registros de Azure Monitor](../azure-monitor/platform/stream-monitoring-data-event-hubs.md).
>
>

## <a name="support--troubleshooting-settings"></a>Configuración de soporte técnico y solución de problemas
La configuración de la sección **Soporte y solución de problemas** proporciona opciones para resolver problemas con la memoria caché.

![Soporte y solución de problemas](./media/cache-configure/redis-cache-support-troubleshooting.png)

* [Estado de los recursos](#resource-health)
* [Nueva solicitud de soporte técnico](#new-support-request)

### <a name="resource-health"></a>Estado de los recursos
**estado de los recursos** supervisa el recurso e indica si se ejecuta del modo previsto. Para obtener más información sobre el servicio Estado de los recursos de Azure, consulte [Información general sobre Estado de los recursos de Azure](../resource-health/resource-health-overview.md).

> [!NOTE]
> El servicio Estado de los recursos no puede actualmente informar sobre el estado de las instancias de Azure Cache for Redis hospedadas en una red virtual. Para más información, consulte [¿Funcionarán todas las características al alojar una caché en una red virtual?](cache-how-to-premium-vnet.md#do-all-cache-features-work-when-hosting-a-cache-in-a-vnet)
>
>

### <a name="new-support-request"></a>Nueva solicitud de soporte
Haga clic en **Nueva solicitud de soporte técnico** para abrir una solicitud de soporte técnico para la memoria caché.





## <a name="default-redis-server-configuration"></a>Configuración predeterminada del servidor Redis
Las nuevas instancias de Azure Cache for Redis se configuran con los siguientes valores de configuración de Redis predeterminados:

> [!NOTE]
> No se puede cambiar la configuración de esta sección con el método `StackExchange.Redis.IServer.ConfigSet`. Si se llama a este método con uno de los comandos de esta sección, se produce una excepción similar a la del siguiente ejemplo:  
>
> `StackExchange.Redis.RedisServerException: ERR unknown command 'CONFIG'`
>
> Es posible configurar aquellos valores que permitan esta opción, como **max-memory-policy**, a través de Azure Portal o las herramientas de administración de la línea de comandos como la CLI de Azure o PowerShell.
>
>

| Configuración | Valor predeterminado | Descripción |
| --- | --- | --- |
| `databases` |16 |El número predeterminado de bases de datos es 16, pero se puede configurar otro número en función del plan de tarifa.<sup>1</sup> La base de datos predeterminada es DB 0, pero se puede seleccionar otra por conexión mediante `connection.GetDatabase(dbid)`, donde `dbid` es un número entre `0` y `databases - 1`. |
| `maxclients` |Depende del plan de tarifa<sup>2</sup> |Este valor es el número máximo de clientes conectados que se permiten al mismo tiempo. Una vez alcanzado el límite, Redis cierra todas las conexiones nuevas y devuelve un error de "número máximo alcanzado de clientes". |
| `maxmemory-policy` |`volatile-lru` |La directiva maxmemory es la configuración de cómo Redis selecciona lo que se debe quitar cuando se alcanza `maxmemory` (el tamaño de la oferta de caché que seleccionó al crear la caché). Con Azure Cache for Redis, la configuración predeterminada es `volatile-lru`, que quita las claves con una fecha de expiración definida mediante un algoritmo LRU. Esta opción puede configurarse en el Portal de Azure. Para más información, vea [Directivas de memoria](#memory-policies). |
| `maxmemory-samples` |3 |Para ahorrar memoria, los algoritmos LRU y TTL mínimo son algoritmos aproximados en lugar de algoritmos precisos. De forma predeterminada, Redis comprueba tres claves y selecciona la que se ha usado menos recientemente. |
| `lua-time-limit` |5\.000 |Tiempo máximo de ejecución de un script Lua en milisegundos. Si se alcanza el tiempo máximo de ejecución, Redis registra que un script está aún en ejecución una vez transcurrido el tiempo máximo permitido y empieza a responder a las consultas con un error. |
| `lua-event-limit` |500 |Tamaño máximo de la cola de eventos de script. |
| `client-output-buffer-limit` `normalclient-output-buffer-limit` `pubsub` |0 0 032mb 8mb 60 |Los límites de búfer de salida de cliente pueden usarse para forzar la desconexión de clientes que, por algún motivo (un motivo habitual es que un cliente de Pub/Sub no puede consumir mensajes tan rápidamente como el publicador los crea), no leen datos del servidor con suficiente rapidez. Para más información, vea [https://redis.io/topics/clients](https://redis.io/topics/clients). |

<a name="databases"></a>
<sup>1</sup>El límite de `databases` es diferente en cada plan de tarifa de Azure Cache for Redis y puede establecerse al crear la memoria caché. Si no se especifica la configuración de `databases` al crear la memoria caché, el valor predeterminado es 16.

* Cachés Basic y Standard
  * Memoria caché C0 (250 MB): hasta 16 bases de datos
  * Memoria caché C1 (1 GB): hasta 16 bases de datos
  * Memoria caché C2 (2,5 GB): hasta 16 bases de datos
  * Memoria caché C3 (6 GB): hasta 16 bases de datos
  * Memoria caché C4 (13 GB): hasta 32 bases de datos
  * Memoria caché C5 (26 GB): hasta 48 bases de datos
  * Memoria caché C6 (53 GB): hasta 64 bases de datos
* Cachés Premium
  * P1 (6 GB - 60 GB): hasta 16 bases de datos
  * P2 (13 GB - 130 GB): hasta 32 bases de datos
  * P3 (26 GB - 260 GB): hasta 48 bases de datos
  * P4 (53 GB - 530 GB): hasta 64 bases de datos
  * Todas las memorias caché premium con clúster de Redis habilitado: el clúster de Redis solo admite el uso de la base de datos 0 con el fin de que el límite `databases` para cualquier caché premium con clúster de Redis habilitado sea 1; el comando [Select](https://redis.io/commands/select) no se admite. Para más información, consulte [Configuración de Caché en Redis de Azure](cache-how-to-premium-clustering.md#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)

Para obtener más información sobre las bases de datos, consulte el artículo [¿Cuáles son las bases de datos de Redis?](cache-development-faq.md#what-are-redis-databases)

> [!NOTE]
> Los ajustes `databases` solo se puede configurar al crear la memoria caché y solo mediante PowerShell, la CLI u otros clientes de administración. Para ver un ejemplo de configuración de `databases` al crear la memoria caché mediante PowerShell, consulte [New-AzRmRedisCache](cache-how-to-manage-redis-cache-powershell.md#databases).
>
>

<a name="maxclients"></a>
<sup>2</sup>`maxclients` es diferente en cada plan de tarifa de Azure Cache for Redis.

* Cachés Basic y Standard
  * Memoria caché C0  (250 MB): hasta 256 conexiones
  * Memoria caché C1 (1 GB): hasta 1000 conexiones
  * Memoria caché C2 (2,5 GB): hasta 2000 conexiones
  * Memoria caché C3 (6 GB): hasta 5000 conexiones
  * Memoria caché C4 (13 GB): hasta 10 000 conexiones
  * Memoria caché C5 (26 GB): hasta 15 000 conexiones
  * Memoria caché C6 (53 GB): hasta 20 000 conexiones
* Cachés Premium
  * P1 (6 GB - 60 GB): hasta 7.500 conexiones
  * P2 (13 GB - 130 GB): hasta 15.000 conexiones
  * P3 (26 GB - 260 GB): hasta 30.000 conexiones
  * P4 (53 GB - 530 GB): hasta 40.000 conexiones

> [!NOTE]
> Si bien cada tamaño de caché permite *hasta* cierta cantidad de conexiones, cada conexión a Redis tiene asociada una sobrecarga. Un ejemplo de dicha sobrecarga podría ser el uso de memoria y CPU como resultado del cifrado TLS/SSL. El límite máximo de conexiones para un tamaño de caché determinado supone una caché con poca carga. Si la carga proveniente de la sobrecarga de conexiones *más* la carga proveniente de las operaciones de clientes supera la capacidad del sistema, la caché puede tener problemas de capacidad incluso si no ha excedido el límite de conexiones para el tamaño de la caché actual.
>
>



## <a name="redis-commands-not-supported-in-azure-cache-for-redis"></a>Comandos de Redis que no se admiten en Azure Cache for Redis
> [!IMPORTANT]
> Dado que Microsoft administra la configuración y la administración de instancias de Azure Cache for Redis, se han deshabilitado los siguientes comandos. Si intenta invocarlos, recibirá un mensaje de error similar a `"(error) ERR unknown command"`.
>
> * BGREWRITEAOF
> * BGSAVE
> * CONFIG
> * DEBUG
> * MIGRATE
> * SAVE
> * SHUTDOWN
> * SLAVEOF
> * CLÚSTER: los comandos de escritura del clúster están deshabilitados, pero se permiten los comandos de solo lectura del clúster.
>
>

Para más información sobre los comandos de Redis, consulte [https://redis.io/commands](https://redis.io/commands).

## <a name="redis-console"></a>Consola de Redis
Puede emitir comandos de forma segura a sus instancias de Azure Cache for Redis con la **consola de Redis**, que está disponible en Azure Portal para todos los niveles de memoria caché.

> [!IMPORTANT]
> - La consola de Redis no funciona con [redes virtuales](cache-how-to-premium-vnet.md). Cuando la memoria caché forma parte de una red virtual, solo los clientes de la red virtual pueden tener acceso a la memoria caché. Como la consola de Redis se ejecuta en el explorador local, que está fuera de la red virtual, no se puede conectar a la caché.
> - No se admiten todos los comandos de Redis en Azure Cache for Redis. Para una lista de los comandos de Redis deshabilitados para Azure Cache for Redis, vea la sección anterior [Comandos de Redis que no se admiten en Azure Cache for Redis](#redis-commands-not-supported-in-azure-cache-for-redis). Para más información sobre los comandos de Redis, consulte [https://redis.io/commands](https://redis.io/commands).
>
>

Para acceder a la Consola de Redis, en la hoja **Azure Cache for Redis**, haga clic en **Consola**.

![Consola de Redis](./media/cache-configure/redis-console-menu.png)

Para emitir comandos en su instancia de la memoria caché, escriba el comando que quiera en la consola.

![Consola de Redis](./media/cache-configure/redis-console.png)


### <a name="using-the-redis-console-with-a-premium-clustered-cache"></a>Uso de la consola de Redis con una caché en clúster premium

Si usa la consola de Redis con una caché en clúster premium, puede emitir comandos a una sola partición de la memoria caché. Para emitir un comando a una partición específica, primero debe conectarse a la partición deseada haciendo clic en ella en el selector de partición.

![Consola de Redis](./media/cache-configure/redis-console-premium-cluster.png)

Si intenta acceder a una clave almacenada en una partición diferente de la partición conectada, recibirá un mensaje de error similar al siguiente mensaje:

```
shard1>get myKey
(error) MOVED 866 13.90.202.154:13000 (shard 0)
```

En el ejemplo anterior, la partición 1 es la partición seleccionada, pero `myKey` se encuentra en la partición 0, tal y como indica el fragmento `(shard 0)` del mensaje de error. En este ejemplo, para obtener acceso a `myKey`, debe seleccionar la partición 0 con el selector de partición y, luego, emitir el comando deseado.


## <a name="move-your-cache-to-a-new-subscription"></a>Traslado de la memoria caché a una nueva suscripción
Para mover la memoria caché a una nueva suscripción, haga clic en **Mover**.

![Mover Azure Cache for Redis](./media/cache-configure/redis-cache-move.png)

Para obtener información acerca de cómo mover recursos de un grupo de recursos a otro y de una suscripción a otra, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md).

## <a name="next-steps"></a>Pasos siguientes
* Para más información sobre cómo trabajar con los comandos de Redis, consulte [¿Cómo puedo ejecutar comandos de Redis?](cache-development-faq.md#how-can-i-run-redis-commands).
