---
title: Mejora del rendimiento de las aplicaciones de Azure con Advisor
description: Use recomendaciones sobre rendimiento de Azure Advisor para mejorar la velocidad y la capacidad de respuesta de las aplicaciones empresariales críticas.
ms.topic: article
ms.date: 01/29/2019
ms.openlocfilehash: 7ecd6a45dc255f4748ed5074a3adb3d948f4122e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87057565"
---
# <a name="improve-the-performance-of-azure-applications-by-using-azure-advisor"></a>Mejora del rendimiento de las aplicaciones de Azure con Azure Advisor

Las recomendaciones sobre rendimiento de Azure Advisor pueden ayudar a mejorar la velocidad y la capacidad de respuesta de las aplicaciones críticas para la empresa. Puede obtener las recomendaciones sobre rendimiento de Advisor en la pestaña **Rendimiento** del panel de Advisor.

## <a name="reduce-dns-time-to-live-on-your-traffic-manager-profile-to-fail-over-to-healthy-endpoints-faster"></a>Reducción del período de vida de DNS en el perfil de Traffic Manager para conmutar por error a puntos de conexión en buen estado con más rapidez

Puede usar la [configuración del período de vida (TTL)](../traffic-manager/traffic-manager-performance-considerations.md) del perfil de Traffic Manager para especificar la rapidez con la que cambian los puntos de conexión si un punto de conexión determinado deja de responder a las consultas. Si reduce los valores de TTL, se enrutará más rápido a los clientes a los puntos de conexión en funcionamiento.

Azure Advisor identifica los perfiles de Traffic Manager con un TTL más largo configurado. Recomienda configurar el TTL en 20 o 60 segundos, dependiendo de si el perfil está configurado para [Fast Failover](https://azure.microsoft.com/roadmap/fast-failover-and-tcp-probing-in-azure-traffic-manager/).

## <a name="improve-database-performance-by-using-sql-database-advisor"></a>Mejora del rendimiento de la base de datos mediante SQL Database Advisor

Azure Advisor proporciona una vista coherente y consolidada de recomendaciones para todos los recursos de Azure. Se integra con SQL Database Advisor para ofrecer recomendaciones y mejorar el rendimiento de las bases de datos. SQL Database Advisor evalúa el rendimiento de las bases de datos mediante el análisis del historial de utilización. Después, ofrece las recomendaciones más adecuadas para ejecutar la carga de trabajo habitual de la base de datos.

> [!NOTE]
> Para obtener recomendaciones, la base de datos debe estar en uso durante una semana y, dentro de esa semana, debe haber alguna actividad coherente. SQL Database Advisor puede optimizar los patrones de consultas coherentes con más facilidad que en el caso de ráfagas aleatorias de actividad.

Para más información, consulte [SQL Database Advisor](../azure-sql/database/database-advisor-implement-performance-recommendations.md).

## <a name="upgrade-your-storage-client-library-to-the-latest-version-for-better-reliability-and-performance"></a>Actualice la biblioteca de cliente de Storage a la versión más reciente para mejorar la confiabilidad y el rendimiento.

La versión más reciente de la biblioteca de cliente de Storage o el SDK contiene correcciones para los problemas notificados por los clientes y que se identifican en nuestro proceso de control de calidad. La versión más reciente también incluye la optimización del rendimiento y la confiabilidad, además de nuevas características que pueden mejorar la experiencia global con Azure Storage. Advisor proporciona recomendaciones y pasos para actualizar a la versión más reciente del SDK si está en una versión obsoleta. Las recomendaciones son para los lenguajes admitidos: C++ y .NET.

## <a name="improve-app-service-performance-and-reliability"></a>Mejora de la confiabilidad y el rendimiento de App Service

Azure Advisor integra las sugerencias para mejorar su experiencia de App Services y para descubrir las funciones de la plataforma adecuada. Ejemplos de recomendaciones de App Service:
* Detección de instancias en las que los tiempos de ejecución de las aplicaciones con opciones de mitigación agotan los recursos de la memoria o la CPU.
* Detección de instancias donde la colocación de recursos como aplicaciones web y bases de datos puede mejorar el rendimiento y reducir el costo.

Para más información, consulte [Procedimientos recomendados para Azure App Service](../app-service/app-service-best-practices.md).

## <a name="use-managed-disks-to-prevent-disk-io-throttling"></a>Uso de discos administrados para evitar limitaciones de E/S del disco

Advisor identifica las máquinas virtuales que pertenecen a una cuenta de almacenamiento que está llegando a su objetivo de escalabilidad. Esta condición hace que las máquinas virtuales sean susceptibles a las limitaciones de E/S. Advisor recomendará que estas máquinas virtuales utilicen discos administrados para evitar la degradación del rendimiento.

## <a name="improve-the-performance-and-reliability-of-virtual-machine-disks-by-using-premium-storage"></a>Mejora del rendimiento y la confiabilidad de los discos de máquina virtual mediante Premium Storage

Advisor identifica las máquinas virtuales con discos estándar que tienen un gran volumen de transacciones en la cuenta de almacenamiento. Recomienda actualizar a discos premium. 

Azure Premium Storage ofrece compatibilidad con discos de alto rendimiento y latencia baja para máquinas virtuales que ejecutan cargas de trabajo intensivas de E/S. Los discos de máquinas virtuales que usan cuentas de Premium Storage almacenan datos en unidades de estado sólido (SSD). Para lograr un rendimiento óptimo en la aplicación, se recomienda migrar todos los discos de máquinas virtuales que requieran E/S por segundo elevados a Premium Storage.

## <a name="remove-data-skew-on-your-sql-data-warehouse-tables-to-increase-query-performance"></a>Eliminación de la asimetría de datos en las tablas de SQL Data Warehouse para aumentar el rendimiento de las consultas

La asimetría de datos puede provocar movimientos de datos innecesarios o cuellos de botella en los recursos al ejecutar la carga de trabajo. Advisor detecta una asimetría de datos de distribución mayor del 15 %. Recomienda que redistribuya los datos y revise de las selecciones de clave de distribución de las tablas. Para más información sobre la identificación y eliminación de asimetrías, consulte [Solución de problemas de asimetría](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-distribute.md#how-to-tell-if-your-distribution-column-is-a-good-choice).

## <a name="create-or-update-outdated-table-statistics-in-your-sql-data-warehouse-tables-to-increase-query-performance"></a>Creación o actualización de las estadísticas de tabla obsoletas en las tablas de SQL Data Warehouse para aumentar el rendimiento de las consultas

Advisor identifica las tablas que no tienen [estadísticas de tabla](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-statistics.md) actualizadas y recomienda crear o actualizar las estadísticas de tabla. El optimizador de consultas de SQL Data Warehouse utiliza estas estadísticas actualizadas para estimar la cardinalidad o número de filas en el resultado de la consulta. Estas estimaciones permiten al optimizador de consultas crear un plan de consulta para proporcionar el rendimiento más rápido.

## <a name="improve-mysql-connection-management"></a>Mejora de la administración de conexiones de MySQL

El análisis de Advisor puede indicar que la aplicación que se conecta al servidor de MySQL puede no administrar las conexiones de forma eficaz. Esta condición puede dar lugar a un consumo innecesario de recursos y a una mayor latencia de la aplicación. Para mejorar la administración de conexiones, se recomienda reducir el número de conexiones de corta duración y eliminar las conexiones innecesarias. Puede llevar a cabo estas mejoras mediante la configuración de un agrupador de conexiones del lado servidor, como ProxySQL.


## <a name="scale-up-to-optimize-cache-utilization-on-your-sql-data-warehouse-tables-to-increase-query-performance"></a>Escalado vertical para optimizar el uso de la memoria caché en las tablas de SQL Data Warehouse para mejorar el rendimiento de las consultas

Azure Advisor detecta si las tablas de SQL Data Warehouse tienen un elevado porcentaje de memoria caché utilizada y un reducido porcentaje de aciertos. Esta condición indica una elevada expulsión de la memoria caché, lo que puede afectar al rendimiento de la instancia de SQL Data Warehouse. Advisor le sugiere que escale verticalmente la instancia de SQL Data Warehouse para asegurarse de que asigna suficiente capacidad de memoria caché a la carga de trabajo.

## <a name="convert-sql-data-warehouse-tables-to-replicated-tables-to-increase-query-performance"></a>Conversión de tablas de SQL Data Warehouse en tablas replicadas para aumentar el rendimiento de las consultas

Advisor identifica las tablas que no están replicadas, pero que se beneficiarían de la conversión. Recomienda que convierta estas tablas. Las recomendaciones se basan en:
- El tamaño de la tabla replicada. 
- El número de columnas. 
- El tipo de distribución de la tabla. 
- El número de particiones en la tabla de SQL Data Warehouse. 

Se puede proporcionar heurística adicional en la recomendación para el contexto. Para obtener más información acerca de cómo se determina esta recomendación, vea [Recomendaciones de SQL Data Warehouse](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-concept-recommendations.md#replicate-tables). 

## <a name="migrate-your-storage-account-to-azure-resource-manager-to-get-the-latest-azure-features"></a>Migración de la cuenta de almacenamiento a Azure Resource Manager para obtener las características más recientes de Azure

Migre el modelo de implementación de la cuenta de almacenamiento a Azure Resource Manager para aprovechar las ventajas siguientes:
- Implementaciones de plantilla.
- Opciones de seguridad adicionales.
- La posibilidad de actualizar a una cuenta de GPv2 para usar las características de Azure Storage más recientes. 

Advisor identifica las cuentas de almacenamiento independientes que utilizan el modelo de implementación clásico y recomienda migrar al modelo de implementación de Resource Manager.

> [!NOTE]
> Las alertas clásicas de Azure Monitor se han retirado en agosto de 2019. Le recomendamos que actualice la cuenta de almacenamiento clásica y empiece a utilizar Resource Manager para conservar la funcionalidad de alertas con la nueva plataforma. Para más información, consulte la [retirada de alertas clásicas](../azure-monitor/platform/monitoring-classic-retirement.md#retirement-of-classic-monitoring-and-alerting-platform).

## <a name="design-your-storage-accounts-to-prevent-reaching-the-maximum-subscription-limit"></a>Diseño de las cuentas de almacenamiento para evitar alcanzar el límite máximo de la suscripción

Una región de Azure admite un máximo de 250 cuentas de almacenamiento por suscripción. Una vez alcanzado el límite, no podrá crear cuentas de almacenamiento en esa combinación de región y suscripción. Advisor comprueba las suscripciones y ofrece recomendaciones para que pueda diseñar menos cuentas de almacenamiento para cualquiera que esté a punto de alcanzar el límite máximo.

## <a name="consider-increasing-the-size-of-your-vpn-gateway-sku-to-address-high-p2s-use"></a>Considere la posibilidad de aumentar el tamaño de la SKU de VPN Gateway para abordar el uso elevado de conexiones de punto a sitio.

Cada SKU de Azure VPN Gateway solo puede admitir un número especificado de conexiones de punto a sitio simultáneas. Si el número de conexiones está cerca del límite de la puerta de enlace, pueden producirse errores en los intentos de conexión adicionales. Si aumenta el tamaño de la puerta de enlace, podrá admitir más usuarios de conexión de punto a sitio simultáneos. Advisor proporciona recomendaciones e instrucciones para aumentar el tamaño de la puerta de enlace.

## <a name="consider-increasing-the-size-of-your-vpn-gateway-sku-to-address-high-cpu"></a>Consideración de la posibilidad de aumentar el tamaño de la SKU de VPN Gateway para abordar el uso elevado de CPU

En una carga de tráfico alta, la puerta de enlace de VPN podría quitar paquetes por un uso elevado de la CPU. Considere la posibilidad de actualizar la SKU de VPN Gateway. El aumento del tamaño de la puerta de enlace de VPN garantizará que las conexiones no se eliminen debido a un uso elevado de CPU. Advisor proporciona recomendaciones para solucionar este problema de forma proactiva. 

## <a name="increase-batch-size-when-loading-to-maximize-load-throughput-data-compression-and-query-performance"></a>Aumento del tamaño del lote al cargar para maximizar el rendimiento de carga, la compresión de datos y el rendimiento de consultas

Advisor detecta si puede aumentar el rendimiento de carga aumentando el tamaño del lote al cargar en la base de datos. Podría considerar la posibilidad de usar la instrucción COPY. Si no puede usar la instrucción COPY, considere la posibilidad de aumentar el tamaño del lote cuando use utilidades de carga como la API SQLBulkCopy o BCP. Una buena regla general es usar un tamaño de lote entre 100 000 y 1 millón de filas. El aumento del tamaño del lote aumentará el rendimiento de carga, la compresión de datos y el rendimiento de consultas.

## <a name="co-locate-the-storage-account-in-the-same-region-to-minimize-latency-when-loading"></a>Ubicación de la cuenta de almacenamiento en la misma región para minimizar la latencia en la carga

Advisor detecta si está realizando las cargas desde una región diferente a la del grupo de SQL. Debe considerar la posibilidad de cargar desde una cuenta de almacenamiento que esté en la misma región que el grupo de SQL para minimizar la latencia al cargar los datos. Este cambio ayudará a minimizar la latencia y a aumentar el rendimiento de carga.

## <a name="use-a-supported-kubernetes-version"></a>Uso de una versión compatible de Kubernetes

Advisor detecta versiones no admitidas de Kubernetes.

## <a name="optimize-the-performance-of-your-azure-database-for-mysql-azure-database-for-postgresql-and-azure-database-for-mariadb-servers"></a>Optimización del rendimiento de los servidores de Azure Database for MySQL, Azure Database for PostgreSQL y Azure Database for MariaDB

### <a name="fix-the-cpu-pressure-of-your-azure-database-for-mysql-azure-database-for-postgresql-and-azure-database-for-mariadb-servers-with-cpu-bottlenecks"></a>Corrección de la presión de CPU en los servidores de Azure Database for MySQL, Azure Database for PostgreSQL y Azure Database for MariaDB con cuellos de botella de CPU
Un uso elevado de la CPU durante un período prolongado puede provocar un rendimiento lento de las consultas para la carga de trabajo. Aumentar el tamaño de la CPU ayudará a optimizar el tiempo de ejecución de las consultas de base de datos y mejorar el rendimiento general. Advisor identifica los servidores con un uso elevado de CPU que probablemente estén ejecutando cargas de trabajo restringidas por la CPU y recomienda escalar el proceso.

### <a name="reduce-memory-constraints-on-your-azure-database-for-mysql-azure-database-for-postgresql-and-azure-database-for-mariadb-servers-or-move-to-a-memory-optimized-sku"></a>Reducción de las restricciones de memoria en los servidores de Azure Database for MySQL, Azure Database for PostgreSQL y Azure Database for MariaDB, o traslado a una SKU optimizada para memoria
Una frecuencia de aciertos de caché bajo puede dar lugar a un rendimiento de consulta más lento y un aumento de IOPS. Esta condición podría deberse a un plan de consulta incorrecto o a una carga de trabajo con uso intensivo de memoria. Corregir el plan de consulta o  [aumentar la memoria](../postgresql/concepts-pricing-tiers.md) del servidor de Azure Database for PostgreSQL, Azure Database for MySQL, or Azure Database for MariaDB lo ayudará a optimizar la ejecución de la carga de trabajo de la base de datos. Azure Advisor identifica los servidores afectados por esta renovación elevada del grupo de búferes. Le recomienda realizar una de estas acciones: 
- Corregir el plan de consulta
- Trasladarse a una SKU que tenga más memoria 
- Aumentar el tamaño del almacenamiento para obtener más IOPS

### <a name="use-an-azure-database-for-mysql-or-azure-database-for-postgresql-read-replica-to-scale-out-reads-for-read-intensive-workloads"></a>Uso de una réplica de lectura de Azure Database for MySQL o Azure Database for PostgreSQL para escalar horizontalmente las lecturas para las cargas de trabajo que hacen un uso intensivo de lecturas
Advisor usa la heurística basada en la carga de trabajo como la relación de lecturas y escrituras en el servidor durante los últimos siete días para identificar las cargas de trabajo que hacen un uso intensivo de lecturas. Un recurso de Azure Database for PostgreSQL o Azure Database for MySQL con una relación de lecturas y escrituras muy alta puede dar lugar a que la CPU o las contenciones de memoria ralenticen la velocidad de las consultas. Agregar una  [réplica](../postgresql/howto-read-replicas-portal.md) le ayudará a escalar horizontalmente las lecturas en el servidor de réplicas y evitar las restricciones de CPU o memoria en el servidor principal. Advisor identifica los servidores con tales cargas de trabajo con uso muy intensivo de lecturas y recomienda agregar una  [réplica de lectura](../postgresql/concepts-read-replicas.md)  para descargar parte de las cargas de trabajo de lectura.


### <a name="scale-your-azure-database-for-mysql-azure-database-for-postgresql-or-azure-database-for-mariadb-server-to-a-higher-sku-to-prevent-connection-constraints"></a>Escalado del servidor Azure Database for MySQL, Azure Database for PostgreSQL o Azure Database for MariaDB a una SKU superior para evitar las restricciones de conexión
Cada nueva conexión con el servidor de bases de datos ocupa memoria. El rendimiento del servidor de bases de datos se degrada si se producen errores en las conexiones con el servidor debido a un  [límite superior](../postgresql/concepts-limits.md) en la memoria. Azure Advisor identifica los servidores que se ejecutan con muchos errores de conexión. Recomienda actualizar los límites de conexión del servidor para proporcionar más memoria al servidor realizando una de estas acciones:
- Escalar verticalmente el proceso 
- Usar SKU con optimización para memoria, que tienen más proceso por núcleo

## <a name="scale-your-cache-to-a-different-size-or-sku-to-improve-cache-and-application-performance"></a>Escalado de la memoria caché a un tamaño o SKU diferente para mejorar dicha memoria y el rendimiento de la aplicación

Las instancias de caché funcionan mejor cuando no se ejecutan con niveles altos de presión de memoria, carga de servidor o ancho de banda de red. Estas condiciones pueden provocar que dejen de responder, experimenten pérdida de datos o dejen de estar disponibles. Advisor identifica las instancias de caché en estas condiciones. Le recomienda realizar una de estas acciones:
- Aplicar los procedimientos recomendados para reducir la presión de memoria, la carga del servidor o el ancho de banda de red
- Escalar a un tamaño o SKU diferente que tenga más capacidad

## <a name="add-regions-with-traffic-to-your-azure-cosmos-db-account"></a>Incorporación de regiones con tráfico a la cuenta de Azure Cosmos DB

Advisor detecta las cuentas de Azure Cosmos DB que tienen tráfico de una región que no está configurada actualmente. Recomienda agregar esa región. Esta acción mejorará la latencia de las solicitudes procedentes de esa región y asegura la disponibilidad en caso de interrupciones de la región. [Más información sobre la distribución de datos global con Azure Cosmos DB](https://aka.ms/cosmos/globaldistribution)

## <a name="configure-your-azure-cosmos-db-indexing-policy-by-using-custom-included-or-excluded-paths"></a>Configuración de la directiva de indexación de Azure Cosmos DB con rutas de acceso personalizadas incluidas o excluidas

Advisor identifica los contenedores de Azure Cosmos DB que usan la directiva de indexación predeterminada, pero podrían aprovechar una directiva de indexación personalizada. Esta determinación se basa en el patrón de carga de trabajo. La directiva de indexación predeterminada indexa todas las propiedades. Una directiva de indexación personalizada con uso de rutas de acceso incluidas o excluidas de forma explícita en los filtros de consulta puede reducir las RU y el almacenamiento utilizados para la indexación. [Más información sobre la modificación de las directivas de índice.](https://aka.ms/cosmosdb/modify-index-policy)

## <a name="set-your-azure-cosmos-db-query-page-size-maxitemcount-to--1"></a>Configuración del tamaño de página de consulta (MaxItemCount) de Azure Cosmos DB en -1 

Azure Advisor identifica contenedores de Azure Cosmos DB que usan un tamaño de página de consulta de 100. Recomienda usar un tamaño de página de -1 para realizar análisis más rápidos. [Más información acerca de MaxItemCount.](https://aka.ms/cosmosdb/sql-api-query-metrics-max-item-count)

## <a name="how-to-access-performance-recommendations-in-advisor"></a>Obtención de acceso a las recomendaciones sobre rendimiento en Advisor

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y después abra [Azure Advisor](https://aka.ms/azureadvisordashboard).

2.  En el panel de Advisor, haga clic en la pestaña **Rendimiento**.

## <a name="next-steps"></a>Pasos siguientes

Para aprender más sobre las recomendaciones de Advisor, consulte:

* [Introducción a Advisor](advisor-overview.md)
* [Introducción a Advisor](advisor-get-started.md)
* [Recomendaciones sobre el costo de Advisor](advisor-cost-recommendations.md)
* [Recomendaciones de confiabilidad de Advisor](advisor-high-availability-recommendations.md)
* [Recomendaciones sobre seguridad de Advisor](advisor-security-recommendations.md)
* [Recomendaciones de excelencia operativa de Advisor](advisor-operational-excellence-recommendations.md)
* [API REST de Advisor](/rest/api/advisor/)
