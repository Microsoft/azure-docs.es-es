---
title: Comportamiento de exportación de métricas con valores NULL y cero
description: Explicación de valores NULL frente a valores cero al exportar métricas y un puntero a una lista de las métricas que no son exportables.
services: azure-monitor
ms.topic: reference
ms.date: 07/22/2020
ms.subservice: metrics
ms.openlocfilehash: ca6acb97e52123a0663d988b3f217d305bce2c4b
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131691"
---
# <a name="azure-monitor-platform-metrics-exportable-via-diagnostic-settings"></a>Métricas de plataforma de Azure Monitor que pueden exportarse con la configuración de diagnóstico

De forma predeterminada, Azure Monitor proporciona [métricas de plataforma](data-platform-metrics.md) sin ninguna configuración. Existen diferentes métodos para interactuar con las métricas de plataforma; por ejemplo, se pueden representar en gráficos en el portal, se puede acceder a ellas utilizando la API REST o se pueden consultar con PowerShell o la CLI. Consulte en [Métricas compatibles con Azure Monitor](metrics-supported.md) una lista completa de las métricas de plataforma que actualmente están disponibles con la canalización de métricas consolidada de Azure Monitor. Para consultar estas métricas y acceder a ellas, use la [versión de la API 01-01-2018](/rest/api/monitor/metricdefinitions). Otras métricas pueden estar disponibles en el portal o mediante las API heredadas.

## <a name="metrics-not-exportable-via-diagnostic-settings"></a>Métricas no exportables a través de la configuración de diagnóstico

El contenido que solía estar en esta ubicación se ha migrado a [Métricas compatibles con Azure Monitor](metrics-supported.md#exporting-platform-metrics-to-other-locations).

Existen limitaciones al exportar las métricas a través de la configuración de diagnóstico. Todas las métricas se pueden exportar mediante la API REST. 

## <a name="exported-zero-vs-null-values"></a>Valores cero frente a valores NULL exportados 

Las métricas tienen un comportamiento diferente cuando se trabaja con valores 0 o NULL.  Algunas métricas indican 0 cuando no se obtienen datos, por ejemplo, las métricas de errores http. Otras métricas almacenan valores NULL cuando no se obtienen datos, ya que esta situación puede indicar que el recurso está sin conexión. Puede ver la diferencia al representar estas métricas con valores NULL que se muestran como [líneas discontinuas](metrics-troubleshoot.md#chart-shows-dashed-line). 

Cuando las métricas de plataforma se pueden exportar a través de la configuración de diagnóstico, coinciden con el comportamiento de la métrica. Es decir, exportan valores NULL cuando el recurso no envía datos.  Exportan "0" solo cuando el recurso subyacente los ha emitido realmente. 

Si elimina un grupo de recursos o un recurso específico, los datos de métricas de los recursos afectados ya no se enviarán a los destinos de exportación de la configuración de diagnóstico. Es decir, ya no aparecerán en Event Hubs, cuentas de Azure Storage ni áreas de trabajo de Log Analytics.

### <a name="metrics-that-used-to-export-zero-for-null"></a>Métricas que se solían exportar cero para NULL

Antes del 1 de junio de 2020, las métricas siguientes **solían** emitir "0" cuando no había datos. Estos ceros se podían exportar después a los sistemas de nivel inferior, como Log Analytics y Azure Storage.  Este comportamiento causaba cierta confusión entre los "0" reales (emitidos por el recurso) y los "0" interpretados (valores NULL), por lo que se cambió el comportamiento para que coincidiera con el de la métrica subyacente, como se mencionó en la sección anterior. 

El cambio se produjo en todas las nubes públicas y privadas.

El cambio no afectó al comportamiento de ninguna de las experiencias siguientes: 
   - Registros de recursos de la plataforma exportados a través de la configuración de diagnóstico
   - Creación de gráficos de métricas en el Explorador de métricas
   - Alertas en las métricas de la plataforma
 
A continuación se muestra una lista de las métricas cuyo comportamiento ha cambiado. 

| ResourceType                    | Métrica               |  MetricDisplayName  | 
|---------------------------------|----------------------|---------------------|
| Microsoft.ApiManagement/service | UnauthorizedRequests |  Unauthorized Gateway Requests (Deprecated) [Solicitudes de puerta de enlace no autorizadas (en desuso)]  | 
| Microsoft.ApiManagement/service | TotalRequests |  Total Gateway Requests (Deprecated) [Solicitudes de puerta de enlace en total (en desuso)]  | 
| Microsoft.ApiManagement/service | SuccessfulRequests |  Successful Gateway Requests (Deprecated) [Solicitudes de puerta de enlace correctas (en desuso)]  | 
| Microsoft.ApiManagement/service | Requests |  Requests  | 
| Microsoft.ApiManagement/service | OtherRequests |  Other Gateway Requests (Deprecated) [Otras solicitudes de puerta de enlace (en desuso)]  | 
| Microsoft.ApiManagement/service | FailedRequests |  Failed Gateway Requests (Deprecated) [Solicitudes de puerta de enlace erróneas (en desuso)]  | 
| Microsoft.ApiManagement/service | EventHubTotalFailedEvents |  Eventos EventHub con errores  | 
| Microsoft.ApiManagement/service | EventHubTotalEvents |  Eventos totales de EventHub  | 
| Microsoft.ApiManagement/service | EventHubTotalBytesSent |  Tamaño de los eventos EventHub  | 
| Microsoft.ApiManagement/service | EventHubTimedoutEvents |  Eventos EventHub con tiempo de espera agotado  | 
| Microsoft.ApiManagement/service | EventHubThrottledEvents |  Eventos EventHub limitados  | 
| Microsoft.ApiManagement/service | EventHubSuccessfulEvents |  Eventos EventHub correctos  | 
| Microsoft.ApiManagement/service | EventHubRejectedEvents |  Eventos EventHub rechazados  | 
| Microsoft.ApiManagement/service | EventHubDroppedEvents |  Eventos EventHub quitados  | 
| Microsoft.ApiManagement/service | Duration |  Duración total de las solicitudes de puerta de enlace  | 
| Microsoft.ApiManagement/service | BackendDuration |  Duration of Backend Requests (Duración de las solicitudes de back-end)  | 
| Microsoft.DBforMariaDB/servers | storage_used |  Almacenamiento utilizado  | 
| Microsoft.DBforMariaDB/servers | storage_percent |  Porcentaje de almacenamiento  | 
| Microsoft.DBforMariaDB/servers | storage_limit |  Límite de almacenamiento  | 
| Microsoft.DBforMariaDB/servers | serverlog_storage_usage |  Almacenamiento del registro del servidor usado  | 
| Microsoft.DBforMariaDB/servers | serverlog_storage_percent |  Porcentaje de almacenamiento del registro del servidor  | 
| Microsoft.DBforMariaDB/servers | serverlog_storage_limit |  Límite de almacenamiento del registro del servidor  | 
| Microsoft.DBforMariaDB/servers | seconds_behind_master |  Intervalo de replicación en segundos  | 
| Microsoft.DBforMariaDB/servers | network_bytes_ingress |  Red interna  | 
| Microsoft.DBforMariaDB/servers | network_bytes_egress |  Red interna  | 
| Microsoft.DBforMariaDB/servers | memory_percent |  Porcentaje de memoria  | 
| Microsoft.DBforMariaDB/servers | io_consumption_percent |  Porcentaje de E/S  | 
| Microsoft.DBforMariaDB/servers | cpu_percent |  Porcentaje de CPU  | 
| Microsoft.DBforMariaDB/servers | connections_failed |  Conexiones con errores  | 
| Microsoft.DBforMariaDB/servers | backup_storage_used |  Almacenamiento de copia de seguridad utilizado  | 
| Microsoft.DBforMariaDB/servers | active_connections |  Conexiones activas  | 
| Microsoft.DBforMySQL/servers | storage_used |  Almacenamiento utilizado  | 
| Microsoft.DBforMySQL/servers | storage_percent |  Porcentaje de almacenamiento  | 
| Microsoft.DBforMySQL/servers | storage_limit |  Límite de almacenamiento  | 
| Microsoft.DBforMySQL/servers | serverlog_storage_usage |  Almacenamiento del registro del servidor usado  | 
| Microsoft.DBforMySQL/servers | serverlog_storage_percent |  Porcentaje de almacenamiento del registro del servidor  | 
| Microsoft.DBforMySQL/servers | serverlog_storage_limit |  Límite de almacenamiento del registro del servidor  | 
| Microsoft.DBforMySQL/servers | seconds_behind_master |  Intervalo de replicación en segundos  | 
| Microsoft.DBforMySQL/servers | network_bytes_ingress |  Red interna  | 
| Microsoft.DBforMySQL/servers | network_bytes_egress |  Red interna  | 
| Microsoft.DBforMySQL/servers | memory_percent |  Porcentaje de memoria  | 
| Microsoft.DBforMySQL/servers | io_consumption_percent |  Porcentaje de E/S  | 
| Microsoft.DBforMySQL/servers | cpu_percent |  Porcentaje de CPU  | 
| Microsoft.DBforMySQL/servers | connections_failed |  Conexiones con errores  | 
| Microsoft.DBforMySQL/servers | backup_storage_used |  Almacenamiento de copia de seguridad utilizado  | 
| Microsoft.DBforMySQL/servers | active_connections |  Conexiones activas  | 
| Microsoft.Devices/Account | digitaltwins.telemetry.nodes |  Digital Twins Node Telemetry Placeholder (Marcador de posición de telemetría de nodos de Digital Twins)  | 
| Microsoft.Devices/IotHubs | twinQueries.success |  Consultas gemelas correctas  | 
| Microsoft.Devices/IotHubs | twinQueries.resultSize |  Tamaño de resultado de consultas gemelas  | 
| Microsoft.Devices/IotHubs | twinQueries.failure |  Consultas gemelas con error  | 
| Microsoft.Devices/IotHubs | jobs.queryJobs.success |  Consultas de trabajo correctas  | 
| Microsoft.Devices/IotHubs | jobs.queryJobs.failure |  Consultas de trabajo con error  | 
| Microsoft.Devices/IotHubs | jobs.listJobs.success |  Llamadas correctas para enumerar trabajos  | 
| Microsoft.Devices/IotHubs | jobs.listJobs.failure |  Llamadas con error para enumerar trabajos  | 
| Microsoft.Devices/IotHubs | jobs.failed |  Trabajos con error  | 
| Microsoft.Devices/IotHubs | jobs.createTwinUpdateJob.success |  Creaciones correctas de trabajos de actualización gemelos  | 
| Microsoft.Devices/IotHubs | jobs.createTwinUpdateJob.failure |  Creaciones con error de trabajos de actualización gemelos  | 
| Microsoft.Devices/IotHubs | jobs.createDirectMethodJob.success |  Creaciones correctas de trabajos de invocación de método  | 
| Microsoft.Devices/IotHubs | jobs.createDirectMethodJob.failure |  Creaciones con error de trabajos de invocación de método  | 
| Microsoft.Devices/IotHubs | jobs.completed |  Trabajos completados  | 
| Microsoft.Devices/IotHubs | jobs.cancelJob.success |  Cancelaciones de trabajos correctas  | 
| Microsoft.Devices/IotHubs | jobs.cancelJob.failure |  Cancelaciones de trabajos con error  | 
| Microsoft.Devices/IotHubs | EventGridLatency |  Latencia de Event Grid (versión preliminar)  | 
| Microsoft.Devices/IotHubs | EventGridDeliveries |  Event Grid deliveries(preview) [Entregas de Event Grid (versión preliminar)]  | 
| Microsoft.Devices/IotHubs | devices.totalDevices |  Total de dispositivos (en desuso)  | 
| Microsoft.Devices/IotHubs | devices.connectedDevices.allProtocol |  Dispositivos conectados (en desuso)   | 
| Microsoft.Devices/IotHubs | deviceDataUsageV2 |  Uso total de datos del dispositivo (versión preliminar)  | 
| Microsoft.Devices/IotHubs | deviceDataUsage |  Uso total de datos del dispositivo  | 
| Microsoft.Devices/IotHubs | dailyMessageQuotaUsed |  Número total de mensajes usados  | 
| Microsoft.Devices/IotHubs | d2c.twin.update.success |  Actualizaciones gemelas correctas de los dispositivos  | 
| Microsoft.Devices/IotHubs | d2c.twin.update.size |  Tamaño de las actualizaciones gemelas de los dispositivos  | 
| Microsoft.Devices/IotHubs | d2c.twin.update.failure |  Actualizaciones gemelas con error de los dispositivos  | 
| Microsoft.Devices/IotHubs | d2c.twin.read.success |  Lecturas gemelas correctas de los dispositivos  | 
| Microsoft.Devices/IotHubs | d2c.twin.read.size |  Tamaño de la respuesta de las lecturas gemelas de dispositivos  | 
| Microsoft.Devices/IotHubs | d2c.twin.read.failure |  Lecturas gemelas con error de los dispositivos  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.ingress.success |  Mensajes de telemetría enviados  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.ingress.sendThrottle |  Número de errores de limitación  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.ingress.allProtocol |  Intentos de envío de mensajes de telemetría  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.success |  Enrutamiento: mensajes de telemetría entregados  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.orphaned |  Enrutamiento: mensajes de telemetría huérfanos   | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.invalid |  Enrutamiento: mensajes de telemetría incompatibles  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.fallback |  Enrutamiento: mensajes entregados a la reserva  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.dropped |  Enrutamiento: mensajes de telemetría quitados   | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.storage |  Enrutamiento: latencia de mensajes para almacenamiento  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.serviceBusTopics |  Enrutamiento: latencia de mensajes del tema de Service Bus  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.serviceBusQueues |  Enrutamiento: latencia de mensajes de la cola de Service Bus  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.eventHubs |  Enrutamiento: latencia de mensajes para el centro de eventos  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.builtIn.events |  Enrutamiento: latencia de mensajes para mensajes/eventos  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.storage.bytes |  Enrutamiento: datos entregados al almacenamiento  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.storage.blobs |  Enrutamiento: blobs entregados al almacenamiento  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.storage |  Enrutamiento: mensajes entregados al almacenamiento  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.serviceBusTopics |  Enrutamiento: mensajes entregados al tema de Service Bus  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.serviceBusQueues |  Enrutamiento: mensajes entregados a la cola de Service Bus  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.eventHubs |  Enrutamiento: mensajes entregados al centro de eventos  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.builtIn.events |  Enrutamiento: mensajes entregados a mensajes/eventos  | 
| Microsoft.Devices/IotHubs | configuraciones |  Métricas de configuración  | 
| Microsoft.Devices/IotHubs | C2DMessagesExpired |  C2D Messages Expired (preview) [Mensajes de C2D expirados (versión preliminar)]  | 
| Microsoft.Devices/IotHubs | c2d.twin.update.success |  Actualizaciones gemelas correctas del back-end  | 
| Microsoft.Devices/IotHubs | c2d.twin.update.size |  Tamaño de las actualizaciones gemelas del back-end  | 
| Microsoft.Devices/IotHubs | c2d.twin.update.failure |  Actualizaciones gemelas con error del back-end  | 
| Microsoft.Devices/IotHubs | c2d.twin.read.success |  Lecturas gemelas correctas del back-end  | 
| Microsoft.Devices/IotHubs | c2d.twin.read.size |  Tamaño de la respuesta de las lecturas gemelas del back-end  | 
| Microsoft.Devices/IotHubs | c2d.twin.read.failure |  Lecturas gemelas con error del back-end  | 
| Microsoft.Devices/IotHubs | c2d.methods.success |  Invocaciones correctas al método directo  | 
| Microsoft.Devices/IotHubs | c2d.methods.responseSize |  Tamaño de la respuesta de las invocaciones a métodos directos  | 
| Microsoft.Devices/IotHubs | c2d.methods.requestSize |  Tamaño de la solicitud de las invocaciones a métodos directos  | 
| Microsoft.Devices/IotHubs | c2d.methods.failure |  Invocaciones al método directo con error  | 
| Microsoft.Devices/IotHubs | c2d.commands.egress.reject.success |  C2D messages rejected (Mensajes de C2D rechazados)  | 
| Microsoft.Devices/IotHubs | c2d.commands.egress.complete.success |  C2D message deliveries completed (Entregas de mensajes de C2D completadas)  | 
| Microsoft.Devices/IotHubs | c2d.commands.egress.abandon.success |  C2D messages abandoned (Mensajes de C2D abandonados)  | 
| Microsoft.Devices/provisioningServices | RegistrationAttempts |  Intentos de registro  | 
| Microsoft.Devices/provisioningServices | DeviceAssignments |  Dispositivos asignados  | 
| Microsoft.Devices/provisioningServices | AttestationAttempts |  Intentos de atestación  | 
| Microsoft.DocumentDB/databaseAccounts | TotalRequestUnits |  Unidades de solicitud totales  | 
| Microsoft.DocumentDB/databaseAccounts | TotalRequests |  Total de solicitudes  | 
| Microsoft.DocumentDB/databaseAccounts | MongoRequests |  Solicitudes de Mongo  | 
| Microsoft.DocumentDB/databaseAccounts | MongoRequestCharge |  Cargo de la solicitud de Mongo  | 
| Microsoft.EventGrid/domains | PublishSuccessLatencyInMs |  Latencia de publicación correcta  | 
| Microsoft.EventGrid/domains | PublishSuccessCount |  Eventos publicados  | 
| Microsoft.EventGrid/domains | PublishFailCount |  Eventos no publicados  | 
| Microsoft.EventGrid/domains | MatchedEventCount |  Eventos coincidentes  | 
| Microsoft.EventGrid/domains | DroppedEventCount |  Eventos quitados  | 
| Microsoft.EventGrid/domains | DeliverySuccessCount |  Eventos entregados  | 
| Microsoft.EventGrid/domains | DeadLetteredCount |  Eventos en la cola de mensajes fallidos  | 
| Microsoft.EventGrid/eventSubscriptions | MatchedEventCount |  Eventos coincidentes  | 
| Microsoft.EventGrid/eventSubscriptions | DroppedEventCount |  Eventos quitados  | 
| Microsoft.EventGrid/eventSubscriptions | DeliverySuccessCount |  Eventos entregados  | 
| Microsoft.EventGrid/eventSubscriptions | DeadLetteredCount |  Eventos en la cola de mensajes fallidos  | 
| Microsoft.EventGrid/extensionTopics | UnmatchedEventCount |  Eventos no coincidentes  | 
| Microsoft.EventGrid/extensionTopics | PublishSuccessLatencyInMs |  Latencia de publicación correcta  | 
| Microsoft.EventGrid/extensionTopics | PublishSuccessCount |  Eventos publicados  | 
| Microsoft.EventGrid/extensionTopics | PublishFailCount |  Eventos no publicados  | 
| Microsoft.EventGrid/topics | UnmatchedEventCount |  Eventos no coincidentes  | 
| Microsoft.EventGrid/topics | PublishSuccessLatencyInMs |  Latencia de publicación correcta  | 
| Microsoft.EventGrid/topics | PublishSuccessCount |  Eventos publicados  | 
| Microsoft.EventGrid/topics | PublishFailCount |  Eventos no publicados  | 
| Microsoft.EventHub/clusters | OutgoingMessages |  Mensajes salientes  | 
| Microsoft.EventHub/clusters | OutgoingBytes |  Bytes salientes.  | 
| Microsoft.EventHub/clusters | IncomingRequests |  Solicitudes entrantes  | 
| Microsoft.EventHub/clusters | IncomingMessages |  Mensajes entrantes  | 
| Microsoft.EventHub/clusters | IncomingBytes |  Bytes entrantes.  | 
| Microsoft.EventHub/namespaces | SVRBSY |  Errores de servidor ocupado (en desuso)  | 
| Microsoft.EventHub/namespaces | SUCCREQ |  Solicitudes correctas (en desuso)  | 
| Microsoft.EventHub/namespaces | OUTMSGS |  Outgoing Messages (obsolete) (Deprecated) [Mensajes salientes (obsoletos) (en desuso)]  | 
| Microsoft.EventHub/namespaces | OutgoingMessages |  Mensajes salientes  | 
| Microsoft.EventHub/namespaces | OutgoingBytes |  Bytes salientes.  | 
| Microsoft.EventHub/namespaces | MISCERR |  Otros errores (en desuso)  | 
| Microsoft.EventHub/namespaces | INTERR |  Errores internos del servidor (en desuso)  | 
| Microsoft.EventHub/namespaces | INREQS |  Solicitudes entrantes (en desuso)  | 
| Microsoft.EventHub/namespaces | INMSGS |  Incoming Messages (obsolete) (Deprecated) [Mensajes entrantes (obsoletos) (en desuso)]  | 
| Microsoft.EventHub/namespaces | IncomingRequests |  Solicitudes entrantes  | 
| Microsoft.EventHub/namespaces | IncomingMessages |  Mensajes entrantes  | 
| Microsoft.EventHub/namespaces | IncomingBytes |  Bytes entrantes.  | 
| Microsoft.EventHub/namespaces | FAILREQ |  Solicitudes con error (en desuso)  | 
| Microsoft.EventHub/namespaces | EHOUTMSGS |  Mensajes salientes (en desuso)  | 
| Microsoft.EventHub/namespaces | EHOUTMBS |  Outgoing bytes (obsolete) (Deprecated) [Bytes salientes (obsoletos) (en desuso)]  | 
| Microsoft.EventHub/namespaces | EHOUTBYTES |  Bytes de salida (en desuso)  | 
| Microsoft.EventHub/namespaces | EHINMSGS |  Mensajes entrantes (en desuso)  | 
| Microsoft.EventHub/namespaces | EHINMBS |  Incoming bytes (obsolete) (Deprecated) [Bytes entrantes (obsoletos) (en desuso)]  | 
| Microsoft.EventHub/namespaces | EHINBYTES |  Bytes de entrada (en desuso)  | 
| Microsoft.EventHub/namespaces | EHAMSGS |  Mensajes de archivado (en desuso)  | 
| Microsoft.EventHub/namespaces | EHAMBS |  Rendimiento de mensajes de archivado (en desuso)  | 
| Microsoft.EventHub/namespaces | EHABL |  Mensajes de trabajo pendiente de archivado (en desuso)  | 
| Microsoft.HDInsight/clusters | NumActiveWorkers |  Número de trabajos activos  | 
| Microsoft.HDInsight/clusters | GatewayRequests |  Solicitudes de puerta de enlace  | 
| Microsoft.HDInsight/clusters | CategorizedGatewayRequests |  Solicitudes de puerta de enlace categorizadas  | 
| Microsoft.Insights/AutoscaleSettings | ScaleActionsInitiated |  Acciones de escalado iniciadas  | 
| Microsoft.Insights/Components | traces/count |  Traces  | 
| Microsoft.Insights/Components | performanceCounters/requestsPerSecond |  Tasa de solicitudes HTTP  | 
| Microsoft.Insights/Components | performanceCounters/requestsInQueue |  Solicitudes HTTP en la cola de la aplicación  | 
| Microsoft.Insights/Components | performanceCounters/exceptionsPerSecond |  Velocidad de excepciones  | 
| Microsoft.Insights/Components | pageViews/count |  Vistas de página  | 
| Microsoft.Insights/Components | exceptions/count |  Excepciones  | 
| Microsoft.Kusto/Clusters | StreamingIngestResults |  Streaming Ingest Result (Resultado de la ingesta de streaming)  | 
| Microsoft.Kusto/Clusters | StreamingIngestDuration |  Streaming Ingest Duration (Duración de la ingesta de streaming)  | 
| Microsoft.Kusto/Clusters | StreamingIngestDataRate |  Streaming Ingest Data Rate (Velocidad de datos de la ingesta de streaming)  | 
| Microsoft.Kusto/Clusters | SteamingIngestRequestRate |  Streaming Ingest Request Rate [Velocidad de solicitudes de ingesta de streaming]  | 
| Microsoft.Kusto/Clusters | QueryDuration |  Duración de la consulta  | 
| Microsoft.Kusto/Clusters | KeepAlive |  Mantener conexión  | 
| Microsoft.Kusto/Clusters | IngestionVolumeInMB |  Volumen de ingesta (en MB)  | 
| Microsoft.Kusto/Clusters | IngestionUtilization |  Uso de la ingesta  | 
| Microsoft.Kusto/Clusters | IngestionResult |  Resultado de la ingesta  | 
| Microsoft.Kusto/Clusters | IngestionLatencyInSeconds |  Ingestion latency (in seconds) [Latencia de la ingesta (en segundos)]  | 
| Microsoft.Kusto/Clusters | ExportUtilization |  Utilización de la exportación  | 
| Microsoft.Kusto/Clusters | EventsProcessedForEventHubs |  Events processed (for Event/IoT Hubs) [Eventos procesados (para instancias de Event Hubs o IoT Hub)]  | 
| Microsoft.Kusto/Clusters | CPU |  CPU  | 
| Microsoft.Kusto/Clusters | ContinuousExportResult |  Continuous Export Result (Resultado de la exportación continua)  | 
| Microsoft.Kusto/Clusters | ContinuousExportPendingCount |  Recuento pendiente de exportación continua  | 
| Microsoft.Kusto/Clusters | ContinuousExportNumOfRecordsExported |  Continuous export - num of exported records (Exportación continua: número de registros exportados)  | 
| Microsoft.Kusto/Clusters | ContinuousExportMaxLatenessMinutes |  Minutos de retraso máximos de exportación continua  | 
| Microsoft.Kusto/Clusters | CacheUtilization |  Uso de la caché  | 
| Microsoft.Logic/integrationServiceEnvironments | TriggerThrottledEvents |  Eventos limitados de desencadenadores  | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersSucceeded |  Desencadenadores correctos   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersStarted |  Desencadenadores iniciados   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersSkipped |  Desencadenadores omitidos  | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersFired |  Desencadenadores activados   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersFailed |  Desencadenadores con errores   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersCompleted |  Desencadenadores completados   | 
| Microsoft.Logic/integrationServiceEnvironments | RunThrottledEvents |  Ejecución de eventos limitados  | 
| Microsoft.Logic/integrationServiceEnvironments | RunStartThrottledEvents |  Ejecución de eventos limitados de inicio  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsSucceeded |  Ejecuciones correctas  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsStarted |  Ejecuciones iniciadas  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsFailed |  Ejecuciones con errores  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsCompleted |  Ejecuciones completadas  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsCancelled |  Ejecuciones canceladas  | 
| Microsoft.Logic/integrationServiceEnvironments | RunFailurePercentage |  Porcentaje de errores de ejecución  | 
| Microsoft.Logic/integrationServiceEnvironments | ActionThrottledEvents |  Eventos limitados de acciones  | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsSucceeded |  Acciones correctas   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsStarted |  Acciones iniciadas   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsSkipped |  Acciones omitidas   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsFailed |  Acciones con errores   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsCompleted |  Acciones completadas   | 
| Microsoft.Logic/workflows | TriggerThrottledEvents |  Eventos limitados de desencadenadores  | 
| Microsoft.Logic/workflows | TriggersSucceeded |  Desencadenadores correctos   | 
| Microsoft.Logic/workflows | TriggersStarted |  Desencadenadores iniciados   | 
| Microsoft.Logic/workflows | TriggersSkipped |  Desencadenadores omitidos  | 
| Microsoft.Logic/workflows | TriggersFired |  Desencadenadores activados   | 
| Microsoft.Logic/workflows | TriggersFailed |  Desencadenadores con errores   | 
| Microsoft.Logic/workflows | TriggersCompleted |  Desencadenadores completados   | 
| Microsoft.Logic/workflows | TotalBillableExecutions |  Total de ejecuciones facturables  | 
| Microsoft.Logic/workflows | RunThrottledEvents |  Ejecución de eventos limitados  | 
| Microsoft.Logic/workflows | RunStartThrottledEvents |  Ejecución de eventos limitados de inicio  | 
| Microsoft.Logic/workflows | RunsSucceeded |  Ejecuciones correctas  | 
| Microsoft.Logic/workflows | RunsStarted |  Ejecuciones iniciadas  | 
| Microsoft.Logic/workflows | RunsFailed |  Ejecuciones con errores  | 
| Microsoft.Logic/workflows | RunsCompleted |  Ejecuciones completadas  | 
| Microsoft.Logic/workflows | RunsCancelled |  Ejecuciones canceladas  | 
| Microsoft.Logic/workflows | RunFailurePercentage |  Porcentaje de errores de ejecución  | 
| Microsoft.Logic/workflows | BillingUsageStorageConsumption |  Uso de facturación para las ejecuciones de consumo de almacenamiento  | 
| Microsoft.Logic/workflows | BillingUsageStorageConsumption |  Uso de facturación para las ejecuciones de consumo de almacenamiento  | 
| Microsoft.Logic/workflows | BillingUsageStandardConnector |  Uso de facturación para las ejecuciones del conector estándar  | 
| Microsoft.Logic/workflows | BillingUsageStandardConnector |  Uso de facturación para las ejecuciones del conector estándar  | 
| Microsoft.Logic/workflows | BillingUsageNativeOperation |  Uso de facturación para las ejecuciones de la operación nativa  | 
| Microsoft.Logic/workflows | BillingUsageNativeOperation |  Uso de facturación para las ejecuciones de la operación nativa  | 
| Microsoft.Logic/workflows | BillableTriggerExecutions |  Ejecuciones del desencadenador facturable  | 
| Microsoft.Logic/workflows | BillableActionExecutions |  Ejecuciones de acciones facturables  | 
| Microsoft.Logic/workflows | ActionThrottledEvents |  Eventos limitados de acciones  | 
| Microsoft.Logic/workflows | ActionsSucceeded |  Acciones correctas   | 
| Microsoft.Logic/workflows | ActionsStarted |  Acciones iniciadas   | 
| Microsoft.Logic/workflows | ActionsSkipped |  Acciones omitidas   | 
| Microsoft.Logic/workflows | ActionsFailed |  Acciones con errores   | 
| Microsoft.Logic/workflows | ActionsCompleted |  Acciones completadas   | 
| Microsoft.Network/frontdoors | WebApplicationFirewallRequestCount |  Recuento de solicitudes del firewall de aplicaciones web  | 
| Microsoft.Network/frontdoors | TotalLatency |  Latencia total  | 
| Microsoft.Network/frontdoors | ResponseSize |  Tamaño de la respuesta  | 
| Microsoft.Network/frontdoors | RequestSize |  Tamaño de la solicitud  | 
| Microsoft.Network/frontdoors | RequestCount |  Recuento de solicitudes  | 
| Microsoft.Network/frontdoors | BillableResponseSize |  Billable Response Size (Tamaño de respuesta facturable)  | 
| Microsoft.Network/frontdoors | BackendRequestLatency |  Latencia de las solicitudes de back-end  | 
| Microsoft.Network/frontdoors | BackendRequestCount |  Recuento de solicitudes de back-end  | 
| Microsoft.Network/frontdoors | BackendHealthPercentage |  Porcentaje de estado del back-end  | 
| Microsoft.Network/trafficManagerProfiles | QpsByEndpoint |  Consultas por punto de conexión devueltas  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | scheduled.pending |  Notificaciones programadas pendientes  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.update |  Operaciones de actualización de registros  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.get |  Operaciones de lectura de registros  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.delete |  Operaciones de eliminación de registros  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.create |  Operaciones de creación de registros  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.all |  Operaciones de registro  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.wrongtoken |  Errores de autorización de WNS (token erróneo)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.tokenproviderunreachable |  Errores de autorización de WNS (inaccesible)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.throttled |  Notificaciones limitadas de WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.success |  Notificaciones de WNS correctas  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.pnserror |  Errores de WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidtoken |  Errores de autorización de WNS (token no válido)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidnotificationsize |  Error de tamaño de notificación no válido de WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidnotificationformat |  Formato de notificación no válido de WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidcredentials |  Errores de autorización de WNS (credenciales no válidas)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.expiredchannel |  Error de canal expirado de WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.dropped |  Notificaciones descartadas de WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.channelthrottled |  Canal de WNS limitado  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.channeldisconnected |  Canal de WNS desconectado  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.badchannel |  Error de canal incorrecto de WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.authenticationerror |  Errores de autenticación de WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.throttled |  Notificaciones limitadas de MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.success |  Notificaciones de MPNS correctas  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.pnserror |  Errores de MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.invalidnotificationformat |  Formato de notificación no válido de MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.invalidcredentials |  Credenciales no válidas de MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.dropped |  Notificaciones descartadas de MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.channeldisconnected |  Canal de MPNS desconectado  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.badchannel |  Error de canal incorrecto de MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.authenticationerror |  Errores de autenticación de MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.wrongchannel |  Error de canal incorrecto de GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.throttled |  Notificaciones limitadas de GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.success |  Notificaciones de GCM correctas  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.pnserror |  Errores de GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.invalidnotificationsize |  Error de tamaño de notificación no válido de GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.invalidnotificationformat |  Formato de notificación no válido de GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.invalidcredentials |  Errores de autorización de GCM (credenciales no válidas)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.expiredchannel |  Error de canal expirado de GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.badchannel |  Error de canal incorrecto de GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.authenticationerror |  Errores de autenticación de GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.success |  Notificaciones de APNS correctas  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.pnserror |  Errores de APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.invalidnotificationsize |  Error de tamaño de notificación no válido de APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.invalidcredentials |  Errores de autorización de APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.expiredchannel |  Error de canal expirado de APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.badchannel |  Error de canal incorrecto de APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.success |  Notificaciones correctas  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.pnserror |  Errores del sistema de notificación externo  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.invalidpayload |  Errores de carga útil  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.channelerror |  Errores de canal  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | notificationhub.pushes |  Todas las notificaciones salientes  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.upsert |  Crear o actualizar operaciones de instalación  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.patch |  Aplicar revisión a operaciones de instalación  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.get |  Obtener operaciones de instalación  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.delete |  Eliminar operaciones de instalación  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.all |  Operaciones de administración de instalaciones  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming.scheduled.cancel |  Notificaciones push programadas canceladas  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming.scheduled |  Notificaciones push programadas enviadas  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming.all.requests |  Todas las solicitudes entrantes  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming.all.failedrequests |  Todas las solicitudes entrantes con error  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming |  Mensajes entrantes  | 
| Microsoft.OperationalInsights/workspaces | Latido |  Latido  | 
| Microsoft.Relay/namespaces | BytesTransferred |  BytesTransferred  | 
| Microsoft.ServiceBus/namespaces | OutgoingMessages |  Mensajes salientes  | 
| Microsoft.ServiceBus/namespaces | IncomingRequests |  Solicitudes entrantes  | 
| Microsoft.ServiceBus/namespaces | IncomingMessages |  Mensajes entrantes  | 
| Microsoft.SignalRService/SignalR | UserErrors |  Errores de usuario  | 
| Microsoft.SignalRService/SignalR | SystemErrors |  Errores de sistema  | 
| Microsoft.SignalRService/SignalR | OutboundTraffic |  Tráfico saliente  | 
| Microsoft.SignalRService/SignalR | MessageCount |  Número de mensajes  | 
| Microsoft.SignalRService/SignalR | InboundTraffic |  Tráfico entrante  | 
| Microsoft.SignalRService/SignalR | ConnectionCount |  Número de conexiones  | 
| Microsoft.Sql/managedInstances | avg_cpu_percent |  Porcentaje de CPU medio  | 
| Microsoft.Sql/servers | dtu_used |  DTU utilizada  | 
| Microsoft.Sql/servers | dtu_consumption_percent |  Porcentaje de DTU  | 
| Microsoft.Sql/servers/databases | xtp_storage_percent |  Porcentaje de almacenamiento de OLTP en memoria  | 
| Microsoft.Sql/servers/databases | workers_percent |  Porcentaje de trabajos  | 
| Microsoft.Sql/servers/databases | sessions_percent |  Porcentaje de sesiones  | 
| Microsoft.Sql/servers/databases | physical_data_read_percent |  Porcentaje de E/S de datos  | 
| Microsoft.Sql/servers/databases | log_write_percent |  Porcentaje de E/S de registro  | 
| Microsoft.Sql/servers/databases | dwu_used |  DWU utilizada  | 
| Microsoft.Sql/servers/databases | dwu_consumption_percent |  Porcentaje de DWU  | 
| Microsoft.Sql/servers/databases | dtu_used |  DTU utilizada  | 
| Microsoft.Sql/servers/databases | dtu_consumption_percent |  Porcentaje de DTU  | 
| Microsoft.Sql/servers/databases | deadlock |  Interbloqueos  | 
| Microsoft.Sql/servers/databases | cpu_used |  CPU utilizada  | 
| Microsoft.Sql/servers/databases | cpu_percent |  Porcentaje de CPU  | 
| Microsoft.Sql/servers/databases | connection_successful |  Conexiones correctas  | 
| Microsoft.Sql/servers/databases | connection_failed |  Conexiones con errores  | 
| Microsoft.Sql/servers/databases | cache_hit_percent |  Porcentaje de aciertos de caché  | 
| Microsoft.Sql/servers/databases | blocked_by_firewall |  Bloqueado por el firewall  | 
| Microsoft.Sql/servers/elasticPools | xtp_storage_percent |  Porcentaje de almacenamiento de OLTP en memoria  | 
| Microsoft.Sql/servers/elasticPools | workers_percent |  Porcentaje de trabajos  | 
| Microsoft.Sql/servers/elasticPools | sessions_percent |  Porcentaje de sesiones  | 
| Microsoft.Sql/servers/elasticPools | physical_data_read_percent |  Porcentaje de E/S de datos  | 
| Microsoft.Sql/servers/elasticPools | log_write_percent |  Porcentaje de E/S de registro  | 
| Microsoft.Sql/servers/elasticPools | eDTU_used |  eDTU utilizada  | 
| Microsoft.Sql/servers/elasticPools | dtu_consumption_percent |  Porcentaje de DTU  | 
| Microsoft.Sql/servers/elasticPools | cpu_percent |  Porcentaje de CPU  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | TotalFrontEnds |  Número total de front-ends  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | SmallAppServicePlanInstances |  Trabajos pequeños del plan de App Service  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Requests |  Requests  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | MemoryPercentage |  Porcentaje de memoria  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | MediumAppServicePlanInstances |  Trabajos medianos del plan de App Service  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | LargeAppServicePlanInstances |  Trabajos grandes del plan de App Service  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | HttpQueueLength |  Longitud de la cola HTTP  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http5xx |  Errores de servidor HTTP  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http4xx |  Http 4xx  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http406 |  Http 406  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http404 |  Http 404  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http403 |  Http 403  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http401 |  Http 401  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http3xx |  Http 3xx  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http2xx |  Http 2xx  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http101 |  Http 101  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | DiskQueueLength |  Longitud de la cola de disco  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | CpuPercentage |  Porcentaje de CPU  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | BytesSent |  Salida de datos  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | BytesReceived |  Entrada de datos  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | AverageResponseTime |  Tiempo de respuesta promedio  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | ActiveRequests |  Solicitudes activas  | 
| Microsoft.Web/hostingEnvironments/workerPools | WorkersUsed |  Trabajos usados  | 
| Microsoft.Web/hostingEnvironments/workerPools | WorkersTotal |  Número total de trabajos  | 
| Microsoft.Web/hostingEnvironments/workerPools | WorkersAvailable |  Trabajos disponibles  | 
| Microsoft.Web/hostingEnvironments/workerPools | MemoryPercentage |  Porcentaje de memoria  | 
| Microsoft.Web/hostingEnvironments/workerPools | CpuPercentage |  Porcentaje de CPU  | 
| Microsoft.Web/serverfarms | TcpTimeWait |  TCP Time Wait  | 
| Microsoft.Web/serverfarms | TcpSynSent |  TCP Syn Sent  | 
| Microsoft.Web/serverfarms | TcpSynReceived |  TCP Syn Received  | 
| Microsoft.Web/serverfarms | TcpLastAck |  TCP Last Ack  | 
| Microsoft.Web/serverfarms | TcpFinWait2 |  TCP Fin Wait 2  | 
| Microsoft.Web/serverfarms | TcpFinWait1 |  TCP Fin Wait 1  | 
| Microsoft.Web/serverfarms | TcpEstablished |  TCP Established  | 
| Microsoft.Web/serverfarms | TcpClosing |  TCP Closing (Cierre de TCP)  | 
| Microsoft.Web/serverfarms | TcpCloseWait |  TCP Close Wait  | 
| Microsoft.Web/serverfarms | MemoryPercentage |  Porcentaje de memoria  | 
| Microsoft.Web/serverfarms | HttpQueueLength |  Longitud de la cola HTTP  | 
| Microsoft.Web/serverfarms | DiskQueueLength |  Longitud de la cola de disco  | 
| Microsoft.Web/serverfarms | CpuPercentage |  Porcentaje de CPU  | 
| Microsoft.Web/serverfarms | BytesSent |  Salida de datos  | 
| Microsoft.Web/serverfarms | BytesReceived |  Entrada de datos  | 
| Microsoft.Web/sites | TotalAppDomainsUnloaded |  Dominios de aplicación totales descargados  | 
| Microsoft.Web/sites | TotalAppDomains |  Dominios de aplicación totales  | 
| Microsoft.Web/sites | Subprocesos |  Número de subprocesos  | 
| Microsoft.Web/sites | RequestsInApplicationQueue |  Solicitudes en cola de la aplicación  | 
| Microsoft.Web/sites | Requests |  Requests  | 
| Microsoft.Web/sites | PrivateBytes |  Bytes privados  | 
| Microsoft.Web/sites | MemoryWorkingSet |  Espacio de trabajo de memoria  | 
| Microsoft.Web/sites | IoWriteOperationsPerSecond |  Operaciones de escritura de E/S por segundo  | 
| Microsoft.Web/sites | IoReadBytesPerSecond |  Bytes de escritura de E/S por segundo  | 
| Microsoft.Web/sites | IoReadOperationsPerSecond |  Operaciones de lectura de E/S por segundo  | 
| Microsoft.Web/sites | IoReadBytesPerSecond |  Bytes de lectura de E/S por segundo  | 
| Microsoft.Web/sites | IoOtherOperationsPerSecond |  Otras operaciones de E/S por segundo  | 
| Microsoft.Web/sites | IoOtherBytesPerSecond |  Otros bytes de E/S por segundo  | 
| Microsoft.Web/sites | HttpResponseTime |  Tiempo de respuesta  | 
| Microsoft.Web/sites | Http5xx |  Errores de servidor HTTP  | 
| Microsoft.Web/sites | Http4xx |  Http 4xx  | 
| Microsoft.Web/sites | Http406 |  Http 406  | 
| Microsoft.Web/sites | Http404 |  Http 404  | 
| Microsoft.Web/sites | Http403 |  Http 403  | 
| Microsoft.Web/sites | Http401 |  Http 401  | 
| Microsoft.Web/sites | Http3xx |  Http 3xx  | 
| Microsoft.Web/sites | Http2xx |  Http 2xx  | 
| Microsoft.Web/sites | Http101 |  Http 101  | 
| Microsoft.Web/sites | HealthCheckStatus |  Estado de comprobación de estado  | 
| Microsoft.Web/sites | Asas |  Recuento de identificadores  | 
| Microsoft.Web/sites | Gen2Collections |  Recolección de elementos no utilizados de Gen 2  | 
| Microsoft.Web/sites | Gen1Collections |  Recolección de elementos no utilizados de Gen 1  | 
| Microsoft.Web/sites | Gen0Collections |  Recolección de elementos no utilizados de Gen 0  | 
| Microsoft.Web/sites | FunctionExecutionUnits |  Unidades de ejecución de función  | 
| Microsoft.Web/sites | FunctionExecutionCount |  Recuento de ejecución de funciones  | 
| Microsoft.Web/sites | CurrentAssemblies |  Ensamblados actuales  | 
| Microsoft.Web/sites | CpuTime |  Tiempo de CPU  | 
| Microsoft.Web/sites | BytesSent |  Salida de datos  | 
| Microsoft.Web/sites | BytesReceived |  Entrada de datos  | 
| Microsoft.Web/sites | AverageResponseTime |  Tiempo de respuesta promedio  | 
| Microsoft.Web/sites | AverageMemoryWorkingSet |  Espacio de trabajo de memoria promedio  | 
| Microsoft.Web/sites | AppConnections |  Conexiones  | 
| Microsoft.Web/sites/slots | TotalAppDomainsUnloaded |  Dominios de aplicación totales descargados  | 
| Microsoft.Web/sites/slots | TotalAppDomains |  Dominios de aplicación totales  | 
| Microsoft.Web/sites/slots | Subprocesos |  Número de subprocesos  | 
| Microsoft.Web/sites/slots | RequestsInApplicationQueue |  Solicitudes en cola de la aplicación  | 
| Microsoft.Web/sites/slots | Requests |  Requests  | 
| Microsoft.Web/sites/slots | PrivateBytes |  Bytes privados  | 
| Microsoft.Web/sites/slots | MemoryWorkingSet |  Espacio de trabajo de memoria  | 
| Microsoft.Web/sites/slots | IoWriteOperationsPerSecond |  Operaciones de escritura de E/S por segundo  | 
| Microsoft.Web/sites/slots | IoReadBytesPerSecond |  Bytes de escritura de E/S por segundo  | 
| Microsoft.Web/sites/slots | IoReadOperationsPerSecond |  Operaciones de lectura de E/S por segundo  | 
| Microsoft.Web/sites/slots | IoReadBytesPerSecond |  Bytes de lectura de E/S por segundo  | 
| Microsoft.Web/sites/slots | IoOtherOperationsPerSecond |  Otras operaciones de E/S por segundo  | 
| Microsoft.Web/sites/slots | IoOtherBytesPerSecond |  Otros bytes de E/S por segundo  | 
| Microsoft.Web/sites/slots | HttpResponseTime |  Tiempo de respuesta  | 
| Microsoft.Web/sites/slots | Http5xx |  Errores de servidor HTTP  | 
| Microsoft.Web/sites/slots | Http4xx |  Http 4xx  | 
| Microsoft.Web/sites/slots | Http406 |  Http 406  | 
| Microsoft.Web/sites/slots | Http404 |  Http 404  | 
| Microsoft.Web/sites/slots | Http403 |  Http 403  | 
| Microsoft.Web/sites/slots | Http401 |  Http 401  | 
| Microsoft.Web/sites/slots | Http3xx |  Http 3xx  | 
| Microsoft.Web/sites/slots | Http2xx |  Http 2xx  | 
| Microsoft.Web/sites/slots | Http101 |  Http 101  | 
| Microsoft.Web/sites/slots | HealthCheckStatus |  Estado de comprobación de estado  | 
| Microsoft.Web/sites/slots | Asas |  Recuento de identificadores  | 
| Microsoft.Web/sites/slots | Gen2Collections |  Recolección de elementos no utilizados de Gen 2  | 
| Microsoft.Web/sites/slots | Gen1Collections |  Recolección de elementos no utilizados de Gen 1  | 
| Microsoft.Web/sites/slots | Gen0Collections |  Recolección de elementos no utilizados de Gen 0  | 
| Microsoft.Web/sites/slots | FunctionExecutionUnits |  Unidades de ejecución de función  | 
| Microsoft.Web/sites/slots | FunctionExecutionCount |  Recuento de ejecución de funciones  | 
| Microsoft.Web/sites/slots | CurrentAssemblies |  Ensamblados actuales  | 
| Microsoft.Web/sites/slots | CpuTime |  Tiempo de CPU  | 
| Microsoft.Web/sites/slots | BytesSent |  Salida de datos  | 
| Microsoft.Web/sites/slots | BytesReceived |  Entrada de datos  | 
| Microsoft.Web/sites/slots | AverageResponseTime |  Tiempo de respuesta promedio  | 
| Microsoft.Web/sites/slots | AverageMemoryWorkingSet |  Espacio de trabajo de memoria promedio  | 
| Microsoft.Web/sites/slots | AppConnections |  Conexiones  | 
| Microsoft.Sql/servers/databases | cpu_percent | Porcentaje de CPU | 
| Microsoft.Sql/servers/databases | physical_data_read_percent | Porcentaje de E/S de datos | 
| Microsoft.Sql/servers/databases | log_write_percent | Porcentaje de E/S de registro | 
| Microsoft.Sql/servers/databases | dtu_consumption_percent | Porcentaje de DTU | 
| Microsoft.Sql/servers/databases | connection_successful | Conexiones correctas | 
| Microsoft.Sql/servers/databases | connection_failed | Conexiones con errores | 
| Microsoft.Sql/servers/databases | blocked_by_firewall | Bloqueado por el firewall | 
| Microsoft.Sql/servers/databases | deadlock | Interbloqueos | 
| Microsoft.Sql/servers/databases | xtp_storage_percent | Porcentaje de almacenamiento de OLTP en memoria | 
| Microsoft.Sql/servers/databases | workers_percent | Porcentaje de trabajos | 
| Microsoft.Sql/servers/databases | sessions_percent | Porcentaje de sesiones | 
| Microsoft.Sql/servers/databases | dtu_used | DTU utilizada | 
| Microsoft.Sql/servers/databases | cpu_used | CPU utilizada | 
| Microsoft.Sql/servers/databases | dwu_consumption_percent | Porcentaje de DWU | 
| Microsoft.Sql/servers/databases | dwu_used | DWU utilizada | 
| Microsoft.Sql/servers/databases | cache_hit_percent | Porcentaje de aciertos de caché | 
| Microsoft.Sql/servers/databases | wlg_allocation_relative_to_system_percent | Asignación de grupos de cargas de trabajo por porcentaje del sistema | 
| Microsoft.Sql/servers/databases | wlg_allocation_relative_to_wlg_effective_cap_percent | Asignación de grupos de cargas de trabajo por porcentaje máximo de recursos | 
| Microsoft.Sql/servers/databases | wlg_active_queries | Consultas activas de grupo de cargas de trabajo | 
| Microsoft.Sql/servers/databases | wlg_queued_queries | Consultas en cola del grupo de cargas de trabajo | 
| Microsoft.Sql/servers/databases | active_queries | Consultas activas | 
| Microsoft.Sql/servers/databases | queued_queries | Consultas en cola | 
| Microsoft.Sql/servers/databases | wlg_active_queries_timeouts | Tiempos de espera de consultas de grupo de cargas de trabajo | 
| Microsoft.Sql/servers/databases | wlg_queued_queries_timeouts | Tiempos de espera de consultas en cola de grupo de cargas de trabajo | 
| Microsoft.Sql/servers/databases | wlg_effective_min_resource_percent | Porcentaje mínimo de recursos efectivo | 
| Microsoft.Sql/servers/databases | wlg_effective_cap_resource_percent | Porcentaje máximo de recursos efectivo | 
| Microsoft.Sql/servers/elasticPools | cpu_percent | Porcentaje de CPU | 
| Microsoft.Sql/servers/elasticPools | physical_data_read_percent | Porcentaje de E/S de datos | 
| Microsoft.Sql/servers/elasticPools | log_write_percent | Porcentaje de E/S de registro | 
| Microsoft.Sql/servers/elasticPools | dtu_consumption_percent | Porcentaje de DTU | 
| Microsoft.Sql/servers/elasticPools | workers_percent | Porcentaje de trabajos | 
| Microsoft.Sql/servers/elasticPools | sessions_percent | Porcentaje de sesiones | 
| Microsoft.Sql/servers/elasticPools | eDTU_used | eDTU utilizada | 
| Microsoft.Sql/servers/elasticPools | xtp_storage_percent | Porcentaje de almacenamiento de OLTP en memoria | 
| Microsoft.Sql/servers | dtu_consumption_percent | Porcentaje de DTU | 
| Microsoft.Sql/servers | dtu_used | DTU utilizada | 
| Microsoft.Sql/managedInstances | avg_cpu_percent | Porcentaje de CPU medio | 

