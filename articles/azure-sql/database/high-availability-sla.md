---
title: Alta disponibilidad
titleSuffix: Azure SQL Database and SQL Managed Instance
description: Obtenga información acerca de las funcionalidades y características de alta disponibilidad de Azure SQL Database e Instancia administrada de SQL
services: sql-database
ms.service: sql-db-mi
ms.subservice: high-availability
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: sashan
ms.author: sashan
ms.reviewer: carlrab, sashan
ms.date: 08/12/2020
ms.openlocfilehash: 62dfa3214b86139a8f836b3d9bd72585653b7fa2
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88189943"
---
# <a name="high-availability-for-azure-sql-database-and-sql-managed-instance"></a>Alta disponibilidad para Azure SQL Database e Instancia administrada de SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

El objetivo de la arquitectura de alta disponibilidad de Azure SQL Database e Instancia administrada de SQL es garantizar que la base de datos funciona como mínimo el 99,99 % de las veces (para obtener más información en relación con el SLA específico de los distintos niveles, consulte [Contrato de nivel de servicio para Azure SQL Database e Instancia administrada de SQL](https://azure.microsoft.com/support/legal/sla/sql-database/)), sin preocuparse del impacto de las operaciones de mantenimiento y las interrupciones. Azure controla automáticamente las tareas de mantenimiento críticas, como la aplicación de revisiones, copias de seguridad, actualizaciones de Windows y Azure SQL, así como los eventos no planeados, como los errores subyacentes de hardware, software o red.  Cuando se conmute por error la base de datos subyacente de Azure SQL Database o se le apliquen revisiones, no se notará el tiempo de inactividad si [usa la lógica de reintento](develop-overview.md#resiliency) de su aplicación. Azure SQL Database e Instancia administrada de SQL pueden recuperarse rápidamente, incluso en las circunstancias más críticas, lo que garantiza que los datos estén siempre disponibles.

La solución de alta disponibilidad está diseñada para garantizar que los datos confirmados nunca se pierden debido a errores, que las operaciones de mantenimiento no afectan a la carga de trabajo y que la base de datos no será un único punto de error en la arquitectura de software. No hay ventanas de mantenimiento ni tiempos de inactividad que puedan requerir que detenga la carga de trabajo mientras se actualiza o se mantiene la base de datos.

Hay dos modelos arquitectónicos de alta disponibilidad:

- El **modelo de disponibilidad estándar** que se basa en la separación del proceso y el almacenamiento.  Depende de la alta disponibilidad y la confiabilidad de la capa de almacenamiento remoto. Esta arquitectura va dirigida a las aplicaciones de negocio preocupadas por la economía que pueden permitirse una cierta degradación del rendimiento durante las actividades de mantenimiento.
- El **modelo de disponibilidad prémium** que se basa en un clúster de procesos del motor de base de datos. Depende del hecho de que siempre hay un quórum de nodos del motor de base de datos disponibles. Esta arquitectura va dirigida a aplicaciones críticas con alto rendimiento de E/S y elevada tasa de transacciones, y garantiza un impacto mínimo sobre el rendimiento de la carga de trabajo durante las actividades de mantenimiento.

Azure SQL Database e Instancia administrada de SQL se ejecutan en la versión estable más reciente del Motor de base de datos de SQL Server y del sistema operativo Windows, y la mayoría de los usuarios no advierten que las actualizaciones se realizan continuamente.

## <a name="basic-standard-and-general-purpose-service-tier-availability"></a>Disponibilidad de los niveles de servicio Básico, Estándar y Uso general

Los niveles de servicio Básico, Estándar y De uso general aprovechan la arquitectura de disponibilidad estándar para los procesos aprovisionados y sin servidor. En la siguiente ilustración se muestran cuatro nodos diferentes con las capas de proceso y almacenamiento separadas.

![Separación de proceso y almacenamiento](./media/high-availability-sla/general-purpose-service-tier.png)

El modelo de disponibilidad estándar incluye dos capas:

- Una capa de proceso sin estado que ejecuta el proceso `sqlservr.exe` y que solo contiene datos transitorios y almacenados en caché, como las bases de datos modelo y TempDB en la memoria SSD conectada, y la memoria caché de planes, el grupo de búferes y el grupo de almacén de columnas en la memoria. Este nodo sin estado lo opera Azure Service Fabric, que inicializa `sqlservr.exe`, controla el estado del nodo y realiza la conmutación por error a otro nodo si es necesario.
- Una capa de datos con estado con archivos de base de datos (.mdf o .ldf) que se almacenan en Azure Blob Storage. Azure Blob Storage presenta características integradas de redundancia y disponibilidad de los datos. Garantiza que todos los registros del archivo de registro o de la página del archivo de datos se conservarán aunque se bloquee el proceso de `sqlservr.exe`.

Siempre que se actualice el motor de base de datos o el sistema operativo, o se detecte un error, Azure Service Fabric moverá el proceso sin estado de `sqlservr.exe` a otro nodo de proceso sin estado con capacidad suficiente disponible. Los datos de Azure Blob Storage no se ven afectados por esta operación y los archivos de registro o de datos se asocian al proceso de `sqlservr.exe` recién inicializado. Aunque este proceso garantiza el 99,99 % de disponibilidad, una carga de trabajo pesada podría experimentar una degradación del rendimiento durante la transición dado que el nuevo proceso de `sqlservr.exe` se inicia con la caché inactiva.

## <a name="premium-and-business-critical-service-tier-availability"></a>Disponibilidad de los niveles de servicio Premium y Crítico para la empresa

Los niveles de servicio prémium y crítico para la empresa aprovechan el modelo de disponibilidad prémium, que integra recursos de proceso (proceso de `sqlservr.exe`) y almacenamiento (SSD conectado localmente) en un único nodo. Para conseguir alta disponibilidad se replica el proceso y el almacenamiento en nodos adicionales a fin de crear un clúster formado por tres o cuatro nodos.

![Clúster de nodos del motor de base de datos](./media/high-availability-sla/business-critical-service-tier.png)

Los archivos de base de datos subyacentes (.mdf o .ldf) se colocan en el almacenamiento SSD conectado para proporcionar una latencia muy baja de E/S para la carga de trabajo. Para implementar alta disponibilidad se usa una tecnología parecida a la de los [grupos de disponibilidad AlwaysOn](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) de SQL Server. El clúster incluye una única réplica principal que es accesible para las cargas de trabajo de cliente de lectura y escritura, y hasta tres réplicas secundarias (proceso y almacenamiento) que contienen copias de los datos. El nodo principal inserta constantemente los cambios en los nodos secundarios en orden y garantiza que los datos se sincronizan con al menos una réplica secundaria antes de confirmar cada transacción. Este proceso garantiza que si el nodo principal se bloquea por cualquier motivo, siempre hay un nodo totalmente sincronizado al que conmutar por error. La conmutación por error se inicia en Azure Service Fabric. Una vez que la réplica secundaria se convierte en el nuevo nodo principal, se crea otra réplica secundaria para garantizar que el clúster tiene suficientes nodos (conjunto de quórum). Cuando finaliza la conmutación por error, las conexiones de Azure SQL se redirigen automáticamente al nuevo nodo principal.

Como ventaja adicional, el modelo de disponibilidad prémium incluye la posibilidad de redirigir las conexiones de Azure SQL de solo lectura a una de las réplicas secundarias. Esta característica se denomina [escalado horizontal de lectura](read-scale-out.md) y proporciona el 100 % de capacidad de proceso adicional al mismo costo para descargar operaciones de solo lectura, como cargas de trabajo de análisis, desde la réplica principal.

## <a name="hyperscale-service-tier-availability"></a>Disponibilidad del nivel de servicio Hiperescala

La arquitectura de nivel de servicio de hiperescala se describe en [Arquitectura de funciones distribuidas](https://docs.microsoft.com/azure/sql-database/sql-database-service-tier-hyperscale#distributed-functions-architecture) y solo está disponible actualmente para SQL Database, no para Instancia administrada de SQL.

![Arquitectura funcional de Hiperescala](./media/high-availability-sla/hyperscale-architecture.png)

El modelo de disponibilidad de Hiperescala incluye cuatro capas:

- Una capa de proceso sin estado que ejecuta los procesos `sqlservr.exe` y que solo contiene datos transitorios y almacenados en caché, como la memoria caché RBPEX sin cobertura, TempDB, la base de datos modelo, etc. en la memoria SSD conectada, la memoria caché de planes, el grupo de búferes y el grupo de almacén de columnas en memoria. Esta capa sin estado incluye la réplica de proceso principal y, opcionalmente, un número de réplicas de proceso secundarias que pueden servir como destinos de conmutación por error.
- Una capa de almacenamiento sin estado formada por servidores de páginas. Esta capa es el motor de almacenamiento distribuido para los procesos `sqlservr.exe` que se ejecutan en las réplicas de proceso. Cada servidor de páginas solo contiene datos transitorios y almacenados en caché, como la memoria caché de RBPEX de cobertura en la SSD conectada y las páginas de datos almacenadas en memoria caché. Cada servidor de páginas tiene un servidor de páginas emparejadas en una configuración activa-activa para proporcionar equilibrio de carga, redundancia y una alta disponibilidad.
- Una capa de almacenamiento de registro de transacciones con estado formada por el nodo de proceso que ejecuta el proceso de servicio de registro, la zona de entrada del registro de transacciones y el almacenamiento a largo plazo del registro de transacciones. La zona de aterrizaje y el almacenamiento a largo plazo usan Azure Storage, que proporciona disponibilidad y [redundancia](https://docs.microsoft.com/azure/storage/common/storage-redundancy) para el registro de transacciones, lo que garantiza la durabilidad de los datos para las transacciones confirmadas.
- Una capa de almacenamiento de datos con estado con los archivos de base de datos (.mdf/.ndf) que se almacenan en Azure Storage y que los servidores de páginas actualizan. Esta capa utiliza las características de disponibilidad de datos y [redundancia](https://docs.microsoft.com/azure/storage/common/storage-redundancy) de Azure Storage. Garantiza que todas las páginas de un archivo de datos se conserven aunque se bloqueen los procesos de otras capas de la arquitectura de Hiperescala o si se produzca un error en los nodos de proceso.

Los nodos de proceso de todas las capas de Hiperescala se ejecutan en Azure Service Fabric, que controla el estado de cada nodo y realiza conmutaciones por error en los nodos correctos disponibles según sea necesario.

Para más información sobre la alta disponibilidad en Hiperescala, consulte [Alta disponibilidad de base de datos en Hiperescala](https://docs.microsoft.com/azure/sql-database/sql-database-service-tier-hyperscale#database-high-availability-in-hyperscale).

## <a name="zone-redundant-configuration"></a>Configuración de redundancia de zona

De forma predeterminada, el clúster de nodos del modelo de disponibilidad premium se crea en el mismo centro de datos. Con la incorporación de [Azure Availability Zones](../../availability-zones/az-overview.md), SQL Database puede colocar diferentes réplicas de la base de datos de nivel Crítico para la empresa en diferentes zonas de disponibilidad de la misma región. Para eliminar un punto de error único, también se duplica el anillo de control en varias zonas como tres anillos de puerta de enlace. El enrutamiento a un anillo de puerta de enlace específico se controla mediante [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md) (ATM). Como la configuración de redundancia de zona de los niveles de servicio premium y crítico para la empresa no crean redundancia adicional de base de datos, puede habilitarla sin costo adicional. Si selecciona una configuración de redundancia de zona, puede hacer que las bases de datos de los niveles premium o crítico para la empresa sean resistentes a un número mucho mayor de errores, como interrupciones catastróficas de los centros de datos, sin necesidad de cambiar la lógica de la aplicación. También puede aplicar la configuración de redundancia de zona a cualquier grupo o base de datos existente del nivel Premium o Crítico para la empresa.

Como las bases de datos con redundancia de zona tienen réplicas en distintos centros de datos situados a cierta distancia entre ellos, la mayor latencia de red puede aumentar el tiempo de confirmación y, por lo tanto, afectar al rendimiento de algunas cargas de trabajo OLTP. Siempre puede volver a la configuración de zona única; para ello, deshabilite la configuración de redundancia de zona. Este proceso es una operación en línea similar a la actualización normal del nivel de servicio. Al final del proceso, la base de datos o el grupo se migra desde un anillo con redundancia de zona a un anillo de zona única, o viceversa.

> [!IMPORTANT]
> Los grupos elásticos y las bases de datos con redundancia de zona solo se admiten actualmente en los niveles de servicio premium y crítico para la empresa de algunas regiones. Cuando se usa el nivel crítico para la empresa, la configuración de redundancia de zona solo está disponible cuando se selecciona el hardware de proceso Gen5. Para información actualizada sobre las regiones que admiten bases de datos con redundancia de zona, consulte [Soporte técnico de servicios por región](../../availability-zones/az-region.md).

> [!NOTE]
> Esta característica no está disponible en Instancia administrada de SQL.

En el diagrama siguiente se ilustra la versión con redundancia de zona de la arquitectura de alta disponibilidad:

![arquitectura de alta disponibilidad con redundancia de zona](./media/high-availability-sla/zone-redundant-business-critical-service-tier.png)

## <a name="accelerated-database-recovery-adr"></a>Recuperación de base de datos acelerada (ADR)

[Recuperación acelerada de la base de datos (ADR)](../accelerated-database-recovery.md) es una nueva característica del motor de base de datos que mejora considerablemente la disponibilidad de la base de datos, en especial en presencia de transacciones de larga duración. ADR está disponible actualmente para Azure SQL Database, Instancia administrada de Azure SQL y Azure SQL Data Warehouse.

## <a name="testing-application-fault-resiliency"></a>Prueba de la resistencia a errores de la aplicación

La alta disponibilidad es una parte fundamental de la plataforma SQL Database e Instancia administrada de SQL que funciona de modo transparente para la aplicación de base de datos. Sin embargo, podría ser conveniente probar el modo en que las operaciones de conmutación por error automáticas iniciadas durante los eventos planeados o no planeados afectarían a una aplicación antes de implementarla para producción. Puede desencadenar manualmente una conmutación por error mediante una llamada a una API especial para reiniciar una base de datos, un grupo elástico o una instancia administrada. En el caso de una base de datos con redundancia de zona o un grupo elástico, la llamada a la API daría lugar a la redirección de las conexiones de cliente al nuevo elemento principal en una zona de disponibilidad diferente a la zona principal anterior. Por lo tanto, además de probar cómo afecta la conmutación por error a las sesiones de base de datos existentes, también puede comprobar si cambia el rendimiento de un extremo a otro debido a los cambios en la latencia de la red. Dado que la operación de reinicio es intrusiva y un gran número de ellas podría agotar la plataforma, solo se permite una llamada de conmutación por error cada 30 minutos para cada base de datos, grupo elástico o instancia administrada.

Se puede iniciar una conmutación por error mediante PowerShell, la API REST o la CLI de Azure:

|Tipo de implementación|PowerShell|API DE REST| Azure CLI|
|:---|:---|:---|:---|
|Base de datos|[Invoke-AzSqlDatabaseFailover](https://docs.microsoft.com/powershell/module/az.sql/invoke-azsqldatabasefailover)|[Conmutación por error de la base de datos](/rest/api/sql/databases(failover)/failover/)|[az rest](https://docs.microsoft.com/cli/azure/reference-index#az-rest) se puede usar para invocar una llamada a la API REST desde la CLI de Azure.|
|Grupo elástico|[Invoke-AzSqlElasticPoolFailover](https://docs.microsoft.com/powershell/module/az.sql/invoke-azsqlelasticpoolfailover)|[Conmutación por error del grupo elástico](/rest/api/sql/elasticpools(failover)/failover/)|[az rest](https://docs.microsoft.com/cli/azure/reference-index#az-rest) se puede usar para invocar una llamada a la API REST desde la CLI de Azure.|
|de SQL DB|[Invoke-AzSqlInstanceFailover](/powershell/module/az.sql/Invoke-AzSqlInstanceFailover/)|[Instancias administradas: conmutación por error](https://docs.microsoft.com/rest/api/sql/managed%20instances%20-%20failover/failover)|[az sql mi failover](/cli/azure/sql/mi/#az-sql-mi-failover)|

> [!IMPORTANT]
> El comando de conmutación por error no está disponible para las réplicas secundarias legibles de las bases de datos de hiperescala.

## <a name="conclusion"></a>Conclusión

Azure SQL Database e Instancia administrada de SQL presentan una solución de alta disponibilidad incorporada, que está totalmente integrada con la plataforma de Azure. Depende de Service Fabric para la detección y la recuperación de errores, de Azure Blob Storage para la protección de datos y de Availability Zones para la mayor tolerancia a errores (como ya se ha mencionado en el documento, aún no es aplicable a Azure SQL Managed Instance). Al mismo tiempo, SQL Database e Instancia administrada de SQL aprovechan la tecnología de grupo de disponibilidad Always On de la instancia de SQL Server para la replicación y la conmutación por error. La combinación de estas tecnologías permite que las aplicaciones obtengan todos los beneficios de un modelo de almacenamiento mixto y puedan admitir los SLA más exigentes.

## <a name="next-steps"></a>Pasos siguientes

- Obtener más información acerca de las [zonas de disponibilidad de Azure](../../availability-zones/az-overview.md).
- Obtener más información sobre [Service Fabric](../../service-fabric/service-fabric-overview.md).
- Obtener más información acerca de [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md).
- Aprenda [cómo iniciar una conmutación por error manual en SQL Managed Instance](../managed-instance/user-initiated-failover.md).
- Para conocer más opciones de alta disponibilidad y recuperación ante desastres, consulte [Continuidad del negocio](business-continuity-high-availability-disaster-recover-hadr-overview.md).
