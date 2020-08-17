---
title: 'Supervisión y ajuste de Azure Database for PostgreSQL: servidor único'
description: 'En este artículo se describen las características de supervisión y ajuste de Azure Database for PostgreSQL: Single Server.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 07/10/2020
ms.openlocfilehash: f6a042dfee000dd6341368b46db32fe36060cc72
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87171585"
---
# <a name="monitor-and-tune-azure-database-for-postgresql---single-server"></a>Supervisión y ajuste en Azure Database for PostgreSQL: Single Server
La supervisión de los datos sobre los servidores le permite solucionar problemas y optimizar la carga de trabajo. Azure Database for PostgreSQL proporciona diversas opciones de supervisión para ofrecer información detallada sobre el comportamiento del servidor.

## <a name="metrics"></a>Métricas
Azure Database for PostgreSQL proporciona diversas métricas que proporcionan información sobre el comportamiento de los recursos que admiten el servidor PostgreSQL. Cada métrica se emite con una frecuencia de un minuto y tiene un [historial de hasta 93 días](https://docs.microsoft.com/azure/azure-monitor/platform/data-platform-metrics#retention-of-metrics). Puede configurar alertas en las métricas. Para obtener instrucciones paso a paso, consulte [How to set up alerts](howto-alert-on-metric.md) (Configuración de alertas). Otras tareas incluyen la configuración de acciones automatizadas, la realización de análisis avanzados y el archivo del historial. Para obtener más información, consulte [Información general sobre las métricas en Microsoft Azure](../monitoring-and-diagnostics/monitoring-overview-metrics.md).

### <a name="list-of-metrics"></a>Lista de métricas
Estas métricas están disponibles para Azure Database for PostgreSQL:

|Métrica|Nombre de métrica para mostrar|Unidad|Descripción|
|---|---|---|---|
|cpu_percent|Porcentaje de CPU|Percent|Porcentaje de CPU en uso.|
|memory_percent|Porcentaje de memoria|Percent|Porcentaje de memoria en uso.|
|io_consumption_percent|Porcentaje de E/S|Percent|Porcentaje de E/S en uso. (No se aplica a los servidores de nivel Básico).|
|storage_percent|Porcentaje de almacenamiento|Percent|Porcentaje de almacenamiento que se usa más allá del límite máximo del servidor.|
|storage_used|Almacenamiento utilizado|Bytes|Cantidad de almacenamiento en uso. El almacenamiento que usa el servicio puede incluir los archivos de base de datos, los registros de transacciones y los registros de servidor.|
|storage_limit|Límite de almacenamiento|Bytes|Almacenamiento máximo de este servidor.|
|serverlog_storage_percent|Porcentaje de almacenamiento del registro del servidor|Percent|El porcentaje usado del almacenamiento máximo de registro del servidor.|
|serverlog_storage_usage|Almacenamiento del registro del servidor usado|Bytes|La cantidad de almacenamiento de registro del servidor en uso.|
|serverlog_storage_limit|Límite de almacenamiento del registro del servidor|Bytes|El almacenamiento máximo de registro de este servidor.|
|active_connections|Conexiones activas|Count|Número de conexiones activas al servidor.|
|connections_failed|Conexiones con errores|Count|El número de conexiones establecidas con error.|
|network_bytes_egress|Red interna|Bytes|Red externa a través de conexiones activas.|
|network_bytes_ingress|Red interna|Bytes|Red interna a través de conexiones activas.|
|backup_storage_used|Almacenamiento de copia de seguridad utilizado|Bytes|Cantidad de almacenamiento de copia de seguridad utilizado. Esta métrica representa la suma del almacenamiento consumido por todas las copias de seguridad de base de datos completas, copias de seguridad diferenciales y copias de seguridad de registros, conservadas según el período de retención de copia de seguridad establecido para el servidor. La frecuencia de las copias de seguridad la administra el servicio, y se explicó en el [artículo sobre los conceptos](concepts-backup.md). En el caso del almacenamiento con redundancia geográfica, el uso del almacenamiento de copia de seguridad es dos veces el del almacenamiento con redundancia local.|
|pg_replica_log_delay_in_bytes|Retraso máximo entre réplicas|Bytes|El retardo en bytes entre el servidor maestro y la réplica con mayor retardo. Esta métrica está disponible únicamente en el servidor maestro.|
|pg_replica_log_delay_in_seconds|Retraso entre réplicas|Segundos|El tiempo transcurrido desde la última transacción reproducida. Esta métrica está disponible únicamente para los servidores de réplica.|

## <a name="server-logs"></a>Registros del servidor
Puede habilitar el registro en el servidor. Estos registros de recursos se pueden enviar a los [registros de Azure Monitor](../azure-monitor/log-query/log-query-overview.md), Event Hubs y una cuenta de Storage. Para más información sobre el registro, visite la página [Registros de servidor](concepts-server-logs.md).

## <a name="query-store"></a>Almacén de consultas
El [Almacén de consultas](concepts-query-store.md) realiza un seguimiento del rendimiento de las consultas a lo largo del tiempo, que incluye las estadísticas en tiempo de ejecución y los eventos de espera de consultas. La característica conserva información del rendimiento en tiempo de ejecución de las consultas en una base de datos del sistema denominada **azure_sys** bajo el esquema query_store. Puede controlar la recopilación y el almacenamiento de datos mediante diversos botones de configuración.

## <a name="query-performance-insight"></a>Información de rendimiento de consultas
[Información de rendimiento de consultas](concepts-query-performance-insight.md) funciona en combinación con el Almacén de consultas para proporcionar visualizaciones accesibles desde Azure Portal. Estos gráficos le permiten identificar las consultas clave que afectan al rendimiento. Se puede acceder a la Información de rendimiento de consultas desde la sección **Soporte técnico y solución de problemas** de la página del portal del servidor de Azure Database for PostgreSQL.

## <a name="performance-recommendations"></a>Recomendaciones de rendimiento
La característica [Recomendaciones de rendimiento](concepts-performance-recommendations.md) identifica oportunidades para mejorar el rendimiento de la carga de trabajo. Proporciona recomendaciones para crear índices que tienen el potencial de mejorar el rendimiento de las cargas de trabajo. Para generar recomendaciones de índices, la característica tiene en cuenta diversas funcionalidades de la base de datos, como su esquema y la carga de trabajo devueltos por el Almacén de consultas. Después de implementar cualquier recomendación de rendimiento, los clientes deben probar el rendimiento para evaluar el efecto de estos cambios. 

## <a name="planned-maintenance-notification"></a>Notificación de mantenimiento planeado

Las **notificaciones de mantenimiento planeado** le permiten recibir alertas de mantenimiento planeado futuro para su instancia de Azure Database for PostgreSQL - Un solo servidor. Estas notificaciones se integran con el mantenimiento planeado de [Service Health](../service-health/overview.md) y le permiten ver todo el mantenimiento programado para sus suscripciones en un mismo lugar. También ayuda a escalar la notificación a las audiencias adecuadas de distintos grupos de recursos, ya que puede tener distintos contactos responsables para los distintos recursos. Recibirá la notificación sobre el próximo mantenimiento 72 horas antes del evento.

> [!Note]
> Intentaremos por todos los medios proporcionar la **notificación de mantenimiento planeado** con una antelación de 72 horas para todos los eventos. Sin embargo, en los casos de revisiones críticas o de seguridad, es posible que las notificaciones se envíen más cerca del evento o se omitan.

### <a name="to-receive-planned-maintenance-notification"></a>Para recibir una notificación de mantenimiento planeado

1. En el [portal](https://portal.azure.com), seleccione **Estado del servicio**.
2. En la sección **Alertas**, seleccione **Alertas de estado**.
3. Seleccione **+ Añadir alerta de Service Health** y rellene los campos.
4. Rellene los campos obligatorios. 
5. Elija la opción de **Tipo de evento** y elija **Mantenimiento planeado** o **Seleccionar todo**.
6. En **Grupos de acciones** defina cómo quiere recibir la alerta (obtener un correo electrónico, desencadenar una aplicación lógica, etc.).  
7. Asegúrese de que Habilitar regla tras la creación esté establecido en Sí.
8. Seleccione **Crear regla de alertas** para completar la alerta.

Para conocer los pasos detallados sobre cómo crear **alertas de Service Health**, consulte [Creación de alertas del registro de actividad en notificaciones del servicio](../service-health/alerts-activity-log-service-notifications.md).

> [!IMPORTANT]
> Actualmente, las notificaciones de mantenimiento planeado están disponibles en versión preliminar para todas las regiones **menos** Centro-oeste de EE. UU.

## <a name="next-steps"></a>Pasos siguientes
- Consulte [Configuración de alertas](howto-alert-on-metric.md) para obtener instrucciones sobre cómo crear una alerta en una métrica.
- Para obtener más información sobre cómo acceder a las métricas y exportarlas con Azure Portal, la API de REST o la CLI, consulte [Información general sobre las métricas en Microsoft Azure](../monitoring-and-diagnostics/monitoring-overview-metrics.md).
- Lea nuestro blog sobre [procedimientos recomendados para supervisar el servidor](https://azure.microsoft.com/blog/best-practices-for-alerting-on-metrics-with-azure-database-for-postgresql-monitoring/).
