---
title: Métricas compatibles de Azure Monitor por tipo de recurso
description: Lista de métricas disponibles para cada tipo de recurso con Azure Monitor.
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: reference
ms.date: 05/13/2019
ms.author: ancav
ms.subservice: metrics
ms.openlocfilehash: dae280f86abce47bfcc029f4d81e4ca3a7b696f4
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595436"
---
# <a name="supported-metrics-with-azure-monitor"></a>Métricas compatibles con Azure Monitor

Azure Monitor proporciona varias maneras de interactuar con las métricas, como la representación en gráficos en el portal, el acceso a ellas a través de la API de REST o consultarlas con PowerShell o la CLI. A continuación se muestra una lista completa de todas las métricas disponibles actualmente en la canalización de métricas de Azure Monitor. Otras métricas pueden estar disponibles en el portal o mediante las API heredadas. La siguiente lista incluye solo las métricas disponibles con la canalización de métricas consolidada de Azure Monitor. Para consultar estas métricas y acceder a ellas, use [2018-01-01 api-version](https://docs.microsoft.com/rest/api/monitor/metricdefinitions).

> [!NOTE]
> Actualmente no se admite el envío de métricas de varias dimensiones a través de la configuración de diagnóstico. Las métricas con dimensiones se exportan como métricas unidimensionales planas agregadas a través de los valores de dimensión.
>
> *Por ejemplo*: la métrica "Mensajes entrantes" de una instancia de Event Hub se puede explorar y representar gráficamente por colas. Sin embargo, cuando se exporta a través de la configuración de diagnóstico, la métrica se representará con todos los mensajes entrantes de todas las colas de Event Hub.
>
>

## <a name="microsoftanalysisservicesservers"></a>Microsoft.AnalysisServices/servers

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|qpu_metric|QPU|Count|Promedio|QPU. Intervalo de 0-100 para S1, 0-200 para S2 y 0-400 para S4|ServerResourceType|
|memory_metric|Memoria|Bytes|Promedio|Memoria. Intervalo de 0-25 GB para S1, 0-50 GB para S2 y 0-100 GB para S4|ServerResourceType|
|TotalConnectionRequests|Número total de solicitudes de conexión|Count|Promedio|Número total de solicitudes de conexión. Se trata de llegadas.|ServerResourceType|
|SuccessfullConnectionsPerSec|Conexiones correctas por segundo|CountPerSecond|Promedio|Tasa de finalizaciones de conexión correctas.|ServerResourceType|
|TotalConnectionFailures|Número total de errores de conexión|Count|Promedio|Número total de intentos de conexión con error.|ServerResourceType|
|CurrentUserSessions|Sesiones de usuario actuales|Count|Promedio|Número actual de sesiones de usuario establecidas.|ServerResourceType|
|QueryPoolBusyThreads|Subprocesos ocupados de grupo de consultas|Count|Promedio|Número de subprocesos ocupados del grupo de subprocesos de consulta.|ServerResourceType|
|CommandPoolJobQueueLength|Longitud de cola de trabajos de grupo de comandos|Count|Promedio|Número de trabajos en la cola del grupo de subprocesos de comandos.|ServerResourceType|
|ProcessingPoolJobQueueLength|Longitud de cola de trabajos de grupo de procesamiento|Count|Promedio|Número de trabajos que no son de E/S en la cola del grupo de subprocesos de procesamiento.|ServerResourceType|
|CurrentConnections|Conexión: Conexiones actuales|Count|Promedio|Número actual de conexiones de cliente establecidas.|ServerResourceType|
|CleanerCurrentPrice|Memoria: precio actual de limpieza|Count|Promedio|Precio actual de memoria, USD/byte/tiempo, normalizado a 1000.|ServerResourceType|
|CleanerMemoryShrinkable|Memoria: memoria de limpieza reducible|Bytes|Promedio|Cantidad de memoria, en bytes, que purgará el limpiador en segundo plano.|ServerResourceType|
|CleanerMemoryNonshrinkable|Memoria: memoria de limpieza no reducible|Bytes|Promedio|Cantidad de memoria, en bytes, que no purgará el limpiador en segundo plano.|ServerResourceType|
|MemoryUsage|Memoria: Uso de memoria|Bytes|Promedio|Uso de memoria del proceso de servidor tal como se usa para calcular el precio de la memoria del limpiador. Es igual al contador Proceso\Bytes privados más el tamaño de los datos asignados a la memoria; se ignora la memoria que ha asignado VertiPaq superior al límite de memoria de VertiPaq.|ServerResourceType|
|MemoryLimitHard|Memoria: límite de memoria física|Bytes|Promedio|Límite de memoria física del archivo de configuración.|ServerResourceType|
|MemoryLimitHigh|Memoria: límite alto de memoria|Bytes|Promedio|Límite alto de memoria del archivo de configuración.|ServerResourceType|
|MemoryLimitLow|Memoria: límite bajo de memoria|Bytes|Promedio|Límite bajo de memoria del archivo de configuración.|ServerResourceType|
|MemoryLimitVertiPaq|Memoria: VertiPaq de límite de memoria|Bytes|Promedio|Límite en memoria del archivo de configuración.|ServerResourceType|
|Cuota|Memoria: Cuota|Bytes|Promedio|Cuota de memoria actual, en bytes. La cuota de memoria también se denomina concesión de memoria o reserva de memoria.|ServerResourceType|
|QuotaBlocked|Memoria: cuota bloqueada|Count|Promedio|Número actual de solicitudes de cuota que están bloqueadas hasta que se liberen otras cuotas de memoria.|ServerResourceType|
|VertiPaqNonpaged|Memoria: VertiPaq no paginado|Bytes|Promedio|Bytes de memoria bloqueados en el espacio de trabajo para que los use el motor en memoria.|ServerResourceType|
|VertiPaqPaged|Memoria: VertiPaq paginado|Bytes|Promedio|Bytes de memoria paginada en uso para datos en memoria.|ServerResourceType|
|RowsReadPerSec|Procesamiento: filas leídas por segundo|CountPerSecond|Promedio|Velocidad de filas leídas de todas las bases de datos relacionales.|ServerResourceType|
|RowsConvertedPerSec|Procesamiento: filas convertidas por segundo|CountPerSecond|Promedio|Velocidad de filas convertidas durante el procesamiento.|ServerResourceType|
|RowsWrittenPerSec|Procesamiento: filas escritas por segundo|CountPerSecond|Promedio|Velocidad de filas escritas durante el procesamiento.|ServerResourceType|
|CommandPoolBusyThreads|Subprocesos: subprocesos ocupados del grupo de comandos|Count|Promedio|Número de subprocesos ocupados del grupo de subprocesos de comandos.|ServerResourceType|
|CommandPoolIdleThreads|Subprocesos: subprocesos inactivos del grupo de comandos|Count|Promedio|Número de subprocesos inactivos del grupo de subprocesos de comandos.|ServerResourceType|
|LongParsingBusyThreads|Subprocesos: subprocesos ocupados en análisis largos|Count|Promedio|Número de subprocesos ocupados del grupo de subprocesos de análisis largos.|ServerResourceType|
|LongParsingIdleThreads|Subprocesos: subprocesos inactivos en análisis largos|Count|Promedio|Número de subprocesos inactivos del grupo de subprocesos en análisis largos.|ServerResourceType|
|LongParsingJobQueueLength|Subprocesos: longitud de cola de trabajos en análisis largos|Count|Promedio|Número de trabajos en la cola del grupo de subprocesos en análisis largos.|ServerResourceType|
|ProcessingPoolBusyIOJobThreads|Subprocesos: subprocesos de trabajo de E/S ocupados del grupo de procesamiento|Count|Promedio|Número de subprocesos que ejecutan trabajos de E/S en el grupo de subprocesos de procesamiento.|ServerResourceType|
|ProcessingPoolBusyNonIOThreads|Subprocesos: subprocesos de trabajo ocupados que no son de E/S del grupo de procesamiento|Count|Promedio|Número de subprocesos que ejecutan trabajos que no son de E/S del grupo de subprocesos de procesamiento.|ServerResourceType|
|ProcessingPoolIOJobQueueLength|Subprocesos: Longitud de cola de trabajos de E/S de grupo de procesamiento|Count|Promedio|Número de trabajos de E/S en la cola del grupo de subprocesos de procesamiento.|ServerResourceType|
|ProcessingPoolIdleIOJobThreads|Subprocesos: subprocesos de trabajo de E/S inactivos del grupo de procesamiento|Count|Promedio|Número de subprocesos inactivos para trabajos de E/S en el grupo de subprocesos de procesamiento.|ServerResourceType|
|ProcessingPoolIdleNonIOThreads|Subprocesos: subprocesos de trabajo inactivos que no son de E/S del grupo de procesamiento|Count|Promedio|Número de subprocesos inactivos del grupo de subprocesos de procesamiento dedicado a trabajos que no son de E/S.|ServerResourceType|
|QueryPoolIdleThreads|Subprocesos: subprocesos inactivos del grupo de consultas|Count|Promedio|Número de subprocesos inactivos para trabajos de E/S en el grupo de subprocesos de procesamiento.|ServerResourceType|
|QueryPoolJobQueueLength|Subprocesos: longitud de cola de trabajos del grupo de consultas|Count|Promedio|Número de trabajos en la cola del grupo de subprocesos de consultas.|ServerResourceType|
|ShortParsingBusyThreads|Subprocesos: subprocesos ocupados en análisis cortos|Count|Promedio|Número de subprocesos ocupados del grupo de subprocesos de análisis cortos.|ServerResourceType|
|ShortParsingIdleThreads|Subprocesos: subprocesos inactivos en análisis cortos|Count|Promedio|Número de subprocesos inactivos del grupo de subprocesos de análisis cortos.|ServerResourceType|
|ShortParsingJobQueueLength|Subprocesos: longitud de cola de trabajos en análisis cortos|Count|Promedio|Número de trabajos en la cola del grupo de subprocesos en análisis cortos.|ServerResourceType|
|memory_thrashing_metric|Paginación excesiva de memoria|Porcentaje|Promedio|Paginación excesiva media de memoria.|ServerResourceType|
|mashup_engine_qpu_metric|QPU de motor M|Count|Promedio|Uso de QPU por los procesos del motor de mashup|ServerResourceType|
|mashup_engine_memory_metric|Memoria del motor M|Bytes|Promedio|Uso de memoria por los procesos del motor de mashup|ServerResourceType|

## <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|TotalRequests|Solicitudes de puerta de enlace en total|Count|Total|Número de solicitudes de puerta de enlace|Ubicación, Nombre de host|
|SuccessfulRequests|Solicitudes de puerta de enlace correctas|Count|Total|Número de solicitudes de puerta de enlace correctas|Ubicación, Nombre de host|
|UnauthorizedRequests|Solicitudes de puerta de enlace no autorizadas|Count|Total|Número de solicitudes de puerta de enlace no autorizadas|Ubicación, Nombre de host|
|FailedRequests|Solicitudes de puerta de enlace con error|Count|Total|Número de errores en solicitudes de puerta de enlace|Ubicación, Nombre de host|
|OtherRequests|Otras solicitudes de puerta de enlace|Count|Total|Número de otras solicitudes de puerta de enlace|Ubicación, Nombre de host|
|Duration|Duración total de las solicitudes de puerta de enlace|Milisegundos|Promedio|Duración total de las solicitudes de puerta de enlace en milisegundos|Ubicación, Nombre de host|
|Capacity|Capacity|Porcentaje|Promedio|Métrica de uso para el servicio ApiManagement|Ubicación|
|EventHubTotalEvents|Eventos totales de EventHub|Count|Total|Número de eventos enviados al centro de eventos|Ubicación|
|EventHubSuccessfulEvents|Eventos de centro de eventos correcta|Count|Total|Número de eventos correctos de EventHub|Ubicación|
|EventHubTotalFailedEvents|Eventos de centro de eventos con error|Count|Total|Número de eventos de centro de eventos con errores|Ubicación|
|EventHubRejectedEvents|Eventos de EventHub rechazadas|Count|Total|Número de eventos de EventHub rechazados (incorrecto de la configuración o Anexiones no autorizadas)|Ubicación|
|EventHubThrottledEvents|Eventos limitados de EventHub|Count|Total|Número de eventos limitados de EventHub|Ubicación|
|EventHubTimedoutEvents|Tiempo de espera de eventos de EventHub|Count|Total|Agotó el número de eventos de EventHub|Ubicación|
|EventHubDroppedEvents|Eventos de centro de eventos descartados|Count|Total|Número de eventos se han omitido debido a la cola alcanzado el límite de tamaño|Ubicación|
|EventHubTotalBytesSent|Tamaño de los eventos de centro de eventos|Bytes|Total|Tamaño total de eventos de EventHub en bytes|Ubicación|
|Solicitudes|Solicitudes|Count|Total|Solicitudes de puerta de enlace|Ubicación, BackendResponseCode, LastErrorReason, GatewayResponseCode|

## <a name="microsoftautomationautomationaccounts"></a>Microsoft.Automation/automationAccounts

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|TotalJob|Número total de trabajos|Count|Total|El número total de trabajos (jobs)|Runbook, Estado|
|TotalUpdateDeploymentRuns|Se ejecuta la implementación de actualización total|Count|Total|Se ejecuta la implementación de actualizaciones de software totales|SoftwareUpdateConfigurationName, Status|
|TotalUpdateDeploymentMachineRuns|Se ejecuta la máquina de implementación de actualización total|Count|Total|Máquina de implementación de actualización de software totales que se ejecuta en una implementación de actualizaciones de software que ejecute|SoftwareUpdateConfigurationName, Status, TargetComputer, SoftwareUpdateConfigurationRunId|

## <a name="microsoftbatchbatchaccounts"></a>Microsoft.Batch/batchAccounts

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|CoreCount|Recuento de núcleos dedicados|Count|Total|Número total de núcleos dedicados de la cuenta de Batch|Sin dimensiones|
|TotalNodeCount|Recuento de nodos dedicados|Count|Total|Número total de nodos dedicados de la cuenta de Batch|Sin dimensiones|
|LowPriorityCoreCount|Recuento de núcleos de baja prioridad|Count|Total|Número total de núcleos de baja prioridad de la cuenta de Batch|Sin dimensiones|
|TotalLowPriorityNodeCount|Recuento de nodos de baja prioridad|Count|Total|Número total de nodos de baja prioridad de la cuenta de Batch|Sin dimensiones|
|CreatingNodeCount|Recuento de nodos creados|Count|Total|Número de nodos que se van a crear|Sin dimensiones|
|StartingNodeCount|Recuento de nodos de inicio|Count|Total|Número de nodos que se van a iniciar|Sin dimensiones|
|WaitingForStartTaskNodeCount|Recuento de nodos esperando a la tarea de inicio|Count|Total|Número de nodos que esperan a que se complete la tarea de inicio|Sin dimensiones|
|StartTaskFailedNodeCount|Recuento de nodos con error de la tarea de inicio|Count|Total|Número de nodos con error en la tarea de inicio|Sin dimensiones|
|IdleNodeCount|Recuento de nodos inactivos|Count|Total|Número de nodos inactivos|Sin dimensiones|
|OfflineNodeCount|Recuento de nodos sin conexión|Count|Total|Número de nodos sin conexión|Sin dimensiones|
|RebootingNodeCount|Recuento de nodos de reinicio|Count|Total|Número de nodos de reinicio|Sin dimensiones|
|ReimagingNodeCount|Recuento de nodos de restablecimiento de imagen inicial|Count|Total|Número de nodos de restablecimiento de imagen inicial|Sin dimensiones|
|RunningNodeCount|Recuento de nodos en ejecución|Count|Total|Número de nodos en ejecución|Sin dimensiones|
|LeavingPoolNodeCount|Recuento de nodos que salen del grupo|Count|Total|Número de nodos que abandonan el grupo|Sin dimensiones|
|UnusableNodeCount|Recuento de nodos no utilizables|Count|Total|Número de nodos inutilizables|Sin dimensiones|
|PreemptedNodeCount|Recuento de nodos con prioridad|Count|Total|Número de nodos con prioridad|Sin dimensiones|
|TaskStartEvent|Eventos de inicio de tarea|Count|Total|Número total de tareas que se han iniciado|Sin dimensiones|
|TaskCompleteEvent|Eventos de tarea completada|Count|Total|Número total de tareas que se han completado|Sin dimensiones|
|TaskFailEvent|Eventos de error en tareas|Count|Total|Número total de tareas que se han completado con errores|Sin dimensiones|
|PoolCreateEvent|Eventos de creación de grupo|Count|Total|Número total de grupos que se han creado|Sin dimensiones|
|PoolResizeStartEvent|Eventos de inicio de cambio de tamaño de grupo|Count|Total|Número total de cambios de tamaño de grupo que se han iniciado|Sin dimensiones|
|PoolResizeCompleteEvent|Eventos de finalización de cambio de tamaño de grupo|Count|Total|Número total de cambios de tamaño de grupo completados|Sin dimensiones|
|PoolDeleteStartEvent|Eventos de inicio de eliminación de grupo|Count|Total|Número total de eliminaciones de grupo que se han iniciado|Sin dimensiones|
|PoolDeleteCompleteEvent|Eventos de finalización de eliminaciones de grupo|Count|Total|Número total de eliminaciones de grupo que se han completado|Sin dimensiones|
|JobDeleteCompleteEvent|Eventos de finalización de eliminaciones de trabajo|Count|Total|Número total de trabajos que se han eliminado correctamente.|Sin dimensiones|
|JobDeleteStartEvent|Eventos de inicio de eliminaciones de trabajo|Count|Total|Número total de trabajos cuya eliminación se ha solicitado.|Sin dimensiones|
|JobDisableCompleteEvent|Eventos de finalización de deshabilitación de trabajo|Count|Total|Número total de trabajos que se han deshabilitado correctamente.|Sin dimensiones|
|JobDisableStartEvent|Eventos de inicio de deshabilitación de trabajo|Count|Total|Número total de trabajos cuya deshabilitación se ha solicitado.|Sin dimensiones|
|JobStartEvent|Eventos de inicio de trabajo|Count|Total|Número total de trabajos que se han iniciado correctamente.|Sin dimensiones|
|JobTerminateCompleteEvent|Eventos de finalización de terminaciones de trabajo|Count|Total|Número total de trabajos que se han finalizado correctamente.|Sin dimensiones|
|JobTerminateStartEvent|Eventos de inicio de terminaciones de trabajo|Count|Total|Número total de trabajos cuya terminación se ha solicitado.|Sin dimensiones|

## <a name="microsoftcacheredis"></a>Microsoft.Cache/redis

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|connectedclients|Clientes conectados|Count|Máximo||ShardId|
|totalcommandsprocessed|Total de operaciones|Count|Total||ShardId|
|cachehits|Aciertos de caché|Count|Total||ShardId|
|cachemisses|Errores de caché|Count|Total||ShardId|
|getcommands|Obtenciones|Count|Total||ShardId|
|setcommands|Conjuntos|Count|Total||ShardId|
|operationsPerSecond|Operaciones por segundo|Count|Máximo||ShardId|
|evictedkeys|Claves expulsadas|Count|Total||ShardId|
|totalkeys|Total de claves|Count|Máximo||ShardId|
|expiredkeys|Claves expiradas|Count|Total||ShardId|
|usedmemory|Memoria usada|Bytes|Máximo||ShardId|
|usedmemorypercentage|Porcentaje de memoria usado|Porcentaje|Máximo||ShardId|
|usedmemoryRss|Memoria RSS usada|Bytes|Máximo||ShardId|
|serverLoad|Carga del servidor|Porcentaje|Máximo||ShardId|
|cacheWrite|Escritura de caché|BytesPerSecond|Máximo||ShardId|
|cacheRead|Lectura de caché|BytesPerSecond|Máximo||ShardId|
|percentProcessorTime|CPU|Porcentaje|Máximo||ShardId|
|cacheLatency|Microsegundos de latencia de la memoria caché (versión preliminar)|Count|Promedio||ShardId, SampleType|
|errores|Errors|Count|Máximo||ShardId, ErrorType|
|connectedclients0|Clientes conectados (partición 0)|Count|Máximo||Sin dimensiones|
|totalcommandsprocessed0|Total de operaciones (partición 0)|Count|Total||Sin dimensiones|
|cachehits0|Aciertos de caché (partición 0)|Count|Total||Sin dimensiones|
|cachemisses0|Errores de caché (partición 0)|Count|Total||Sin dimensiones|
|getcommands0|Obtenciones (partición 0)|Count|Total||Sin dimensiones|
|setcommands0|Definiciones (partición 0)|Count|Total||Sin dimensiones|
|operationsPerSecond0|Operaciones por segundo (partición 0)|Count|Máximo||Sin dimensiones|
|evictedkeys0|Claves expulsadas (partición 0)|Count|Total||Sin dimensiones|
|totalkeys0|Total de claves (partición 0)|Count|Máximo||Sin dimensiones|
|expiredkeys0|Claves expiradas (partición 0)|Count|Total||Sin dimensiones|
|usedmemory0|Memoria usada (partición 0)|Bytes|Máximo||Sin dimensiones|
|usedmemoryRss0|Memoria RSS usada (partición 0)|Bytes|Máximo||Sin dimensiones|
|serverLoad0|Carga de servidor (partición 0)|Porcentaje|Máximo||Sin dimensiones|
|cacheWrite0|Escritura de caché (partición 0)|BytesPerSecond|Máximo||Sin dimensiones|
|cacheRead0|Lectura de caché (partición 0)|BytesPerSecond|Máximo||Sin dimensiones|
|percentProcessorTime0|CPU (partición 0)|Porcentaje|Máximo||Sin dimensiones|
|connectedclients1|Clientes conectados (partición 1)|Count|Máximo||Sin dimensiones|
|totalcommandsprocessed1|Total de operaciones (partición 1)|Count|Total||Sin dimensiones|
|cachehits1|Aciertos de caché (partición 1)|Count|Total||Sin dimensiones|
|cachemisses1|Errores de caché (partición 1)|Count|Total||Sin dimensiones|
|getcommands1|Obtenciones (partición 1)|Count|Total||Sin dimensiones|
|setcommands1|Definiciones (partición 1)|Count|Total||Sin dimensiones|
|operationsPerSecond1|Operaciones por segundo (partición 1)|Count|Máximo||Sin dimensiones|
|evictedkeys1|Claves expulsadas (partición 1)|Count|Total||Sin dimensiones|
|totalkeys1|Total de claves (partición 1)|Count|Máximo||Sin dimensiones|
|expiredkeys1|Claves expiradas (partición 1)|Count|Total||Sin dimensiones|
|usedmemory1|Memoria usada (partición 1)|Bytes|Máximo||Sin dimensiones|
|usedmemoryRss1|Memoria RSS usada (partición 1)|Bytes|Máximo||Sin dimensiones|
|serverLoad1|Carga de servidor (partición 1)|Porcentaje|Máximo||Sin dimensiones|
|cacheWrite1|Escritura de caché (partición 1)|BytesPerSecond|Máximo||Sin dimensiones|
|cacheRead1|Lectura de caché (partición 1)|BytesPerSecond|Máximo||Sin dimensiones|
|percentProcessorTime1|CPU (partición 1)|Porcentaje|Máximo||Sin dimensiones|
|connectedclients2|Clientes conectados (partición 2)|Count|Máximo||Sin dimensiones|
|totalcommandsprocessed2|Total de operaciones (partición 2)|Count|Total||Sin dimensiones|
|cachehits2|Aciertos de caché (partición 2)|Count|Total||Sin dimensiones|
|cachemisses2|Errores de caché (partición 2)|Count|Total||Sin dimensiones|
|getcommands2|Obtenciones (partición 2)|Count|Total||Sin dimensiones|
|setcommands2|Definiciones (partición 2)|Count|Total||Sin dimensiones|
|operationsPerSecond2|Operaciones por segundo (partición 2)|Count|Máximo||Sin dimensiones|
|evictedkeys2|Claves expulsadas (partición 2)|Count|Total||Sin dimensiones|
|totalkeys2|Total de claves (partición 2)|Count|Máximo||Sin dimensiones|
|expiredkeys2|Claves expiradas (partición 2)|Count|Total||Sin dimensiones|
|usedmemory2|Memoria usada (partición 2)|Bytes|Máximo||Sin dimensiones|
|usedmemoryRss2|Memoria RSS usada (partición 2)|Bytes|Máximo||Sin dimensiones|
|serverLoad2|Carga de servidor (partición 2)|Porcentaje|Máximo||Sin dimensiones|
|cacheWrite2|Escritura de caché (partición 2)|BytesPerSecond|Máximo||Sin dimensiones|
|cacheRead2|Lectura de caché (partición 2)|BytesPerSecond|Máximo||Sin dimensiones|
|percentProcessorTime2|CPU (partición 2)|Porcentaje|Máximo||Sin dimensiones|
|connectedclients3|Clientes conectados (partición 3)|Count|Máximo||Sin dimensiones|
|totalcommandsprocessed3|Total de operaciones (partición 3)|Count|Total||Sin dimensiones|
|cachehits3|Aciertos de caché (partición 3)|Count|Total||Sin dimensiones|
|cachemisses3|Errores de caché (partición 3)|Count|Total||Sin dimensiones|
|getcommands3|Obtenciones (partición 3)|Count|Total||Sin dimensiones|
|setcommands3|Definiciones (partición 3)|Count|Total||Sin dimensiones|
|operationsPerSecond3|Operaciones por segundo (partición 3)|Count|Máximo||Sin dimensiones|
|evictedkeys3|Claves expulsadas (partición 3)|Count|Total||Sin dimensiones|
|totalkeys3|Total de claves (partición 3)|Count|Máximo||Sin dimensiones|
|expiredkeys3|Claves expiradas (partición 3)|Count|Total||Sin dimensiones|
|usedmemory3|Memoria usada (partición 3)|Bytes|Máximo||Sin dimensiones|
|usedmemoryRss3|Memoria RSS usada (partición 3)|Bytes|Máximo||Sin dimensiones|
|serverLoad3|Carga de servidor (partición 3)|Porcentaje|Máximo||Sin dimensiones|
|cacheWrite3|Escritura de caché (partición 3)|BytesPerSecond|Máximo||Sin dimensiones|
|cacheRead3|Lectura de caché (partición 3)|BytesPerSecond|Máximo||Sin dimensiones|
|percentProcessorTime3|CPU (partición 3)|Porcentaje|Máximo||Sin dimensiones|
|connectedclients4|Clientes conectados (partición 4)|Count|Máximo||Sin dimensiones|
|totalcommandsprocessed4|Total de operaciones (partición 4)|Count|Total||Sin dimensiones|
|cachehits4|Aciertos de caché (partición 4)|Count|Total||Sin dimensiones|
|cachemisses4|Errores de caché (partición 4)|Count|Total||Sin dimensiones|
|getcommands4|Obtenciones (partición 4)|Count|Total||Sin dimensiones|
|setcommands4|Definiciones (partición 4)|Count|Total||Sin dimensiones|
|operationsPerSecond4|Operaciones por segundo (partición 4)|Count|Máximo||Sin dimensiones|
|evictedkeys4|Claves expulsadas (partición 4)|Count|Total||Sin dimensiones|
|totalkeys4|Total de claves (partición 4)|Count|Máximo||Sin dimensiones|
|expiredkeys4|Claves expiradas (partición 4)|Count|Total||Sin dimensiones|
|usedmemory4|Memoria usada (partición 4)|Bytes|Máximo||Sin dimensiones|
|usedmemoryRss4|Memoria RSS usada (partición 4)|Bytes|Máximo||Sin dimensiones|
|serverLoad4|Carga de servidor (partición 4)|Porcentaje|Máximo||Sin dimensiones|
|cacheWrite4|Escritura de caché (partición 4)|BytesPerSecond|Máximo||Sin dimensiones|
|cacheRead4|Lectura de caché (partición 4)|BytesPerSecond|Máximo||Sin dimensiones|
|percentProcessorTime4|CPU (partición 4)|Porcentaje|Máximo||Sin dimensiones|
|connectedclients5|Clientes conectados (partición 5)|Count|Máximo||Sin dimensiones|
|totalcommandsprocessed5|Total de operaciones (partición 5)|Count|Total||Sin dimensiones|
|cachehits5|Aciertos de caché (partición 5)|Count|Total||Sin dimensiones|
|cachemisses5|Errores de caché (partición 5)|Count|Total||Sin dimensiones|
|getcommands5|Obtenciones (partición 5)|Count|Total||Sin dimensiones|
|setcommands5|Definiciones (partición 5)|Count|Total||Sin dimensiones|
|operationsPerSecond5|Operaciones por segundo (partición 5)|Count|Máximo||Sin dimensiones|
|evictedkeys5|Claves expulsadas (partición 5)|Count|Total||Sin dimensiones|
|totalkeys5|Total de claves (partición 5)|Count|Máximo||Sin dimensiones|
|expiredkeys5|Claves expiradas (partición 5)|Count|Total||Sin dimensiones|
|usedmemory5|Memoria usada (partición 5)|Bytes|Máximo||Sin dimensiones|
|usedmemoryRss5|Memoria RSS usada (partición 5)|Bytes|Máximo||Sin dimensiones|
|serverLoad5|Carga de servidor (partición 5)|Porcentaje|Máximo||Sin dimensiones|
|cacheWrite5|Escritura de caché (partición 5)|BytesPerSecond|Máximo||Sin dimensiones|
|cacheRead5|Lectura de caché (partición 5)|BytesPerSecond|Máximo||Sin dimensiones|
|percentProcessorTime5|CPU (partición 5)|Porcentaje|Máximo||Sin dimensiones|
|connectedclients6|Clientes conectados (partición 6)|Count|Máximo||Sin dimensiones|
|totalcommandsprocessed6|Total de operaciones (partición 6)|Count|Total||Sin dimensiones|
|cachehits6|Aciertos de caché (partición 6)|Count|Total||Sin dimensiones|
|cachemisses6|Errores de caché (partición 6)|Count|Total||Sin dimensiones|
|getcommands6|Obtenciones (partición 6)|Count|Total||Sin dimensiones|
|setcommands6|Definiciones (partición 6)|Count|Total||Sin dimensiones|
|operationsPerSecond6|Operaciones por segundo (partición 6)|Count|Máximo||Sin dimensiones|
|evictedkeys6|Claves expulsadas (partición 6)|Count|Total||Sin dimensiones|
|totalkeys6|Total de claves (partición 6)|Count|Máximo||Sin dimensiones|
|expiredkeys6|Claves expiradas (partición 6)|Count|Total||Sin dimensiones|
|usedmemory6|Memoria usada (partición 6)|Bytes|Máximo||Sin dimensiones|
|usedmemoryRss6|Memoria RSS usada (partición 6)|Bytes|Máximo||Sin dimensiones|
|serverLoad6|Carga de servidor (partición 6)|Porcentaje|Máximo||Sin dimensiones|
|cacheWrite6|Escritura de caché (partición 6)|BytesPerSecond|Máximo||Sin dimensiones|
|cacheRead6|Lectura de caché (partición 6)|BytesPerSecond|Máximo||Sin dimensiones|
|percentProcessorTime6|CPU (partición 6)|Porcentaje|Máximo||Sin dimensiones|
|connectedclients7|Clientes conectados (partición 7)|Count|Máximo||Sin dimensiones|
|totalcommandsprocessed7|Total de operaciones (partición 7)|Count|Total||Sin dimensiones|
|cachehits7|Aciertos de caché (partición 7)|Count|Total||Sin dimensiones|
|cachemisses7|Errores de caché (partición 7)|Count|Total||Sin dimensiones|
|getcommands7|Obtenciones (partición 7)|Count|Total||Sin dimensiones|
|setcommands7|Definiciones (partición 7)|Count|Total||Sin dimensiones|
|operationsPerSecond7|Operaciones por segundo (partición 7)|Count|Máximo||Sin dimensiones|
|evictedkeys7|Claves expulsadas (partición 7)|Count|Total||Sin dimensiones|
|totalkeys7|Total de claves (partición 7)|Count|Máximo||Sin dimensiones|
|expiredkeys7|Claves expiradas (partición 7)|Count|Total||Sin dimensiones|
|usedmemory7|Memoria usada (partición 7)|Bytes|Máximo||Sin dimensiones|
|usedmemoryRss7|Memoria RSS usada (partición 7)|Bytes|Máximo||Sin dimensiones|
|serverLoad7|Carga de servidor (partición 7)|Porcentaje|Máximo||Sin dimensiones|
|cacheWrite7|Escritura de caché (partición 7)|BytesPerSecond|Máximo||Sin dimensiones|
|cacheRead7|Lectura de caché (partición 7)|BytesPerSecond|Máximo||Sin dimensiones|
|percentProcessorTime7|CPU (partición 7)|Porcentaje|Máximo||Sin dimensiones|
|connectedclients8|Clientes conectados (partición 8)|Count|Máximo||Sin dimensiones|
|totalcommandsprocessed8|Total de operaciones (partición 8)|Count|Total||Sin dimensiones|
|cachehits8|Aciertos de caché (partición 8)|Count|Total||Sin dimensiones|
|cachemisses8|Errores de caché (partición 8)|Count|Total||Sin dimensiones|
|getcommands8|Obtenciones (partición 8)|Count|Total||Sin dimensiones|
|setcommands8|Definiciones (partición 8)|Count|Total||Sin dimensiones|
|operationsPerSecond8|Operaciones por segundo (partición 8)|Count|Máximo||Sin dimensiones|
|evictedkeys8|Claves expulsadas (partición 8)|Count|Total||Sin dimensiones|
|totalkeys8|Total de claves (partición 8)|Count|Máximo||Sin dimensiones|
|expiredkeys8|Claves expiradas (partición 8)|Count|Total||Sin dimensiones|
|usedmemory8|Memoria usada (partición 8)|Bytes|Máximo||Sin dimensiones|
|usedmemoryRss8|Memoria RSS usada (partición 8)|Bytes|Máximo||Sin dimensiones|
|serverLoad8|Carga de servidor (partición 8)|Porcentaje|Máximo||Sin dimensiones|
|cacheWrite8|Escritura de caché (partición 8)|BytesPerSecond|Máximo||Sin dimensiones|
|cacheRead8|Lectura de caché (partición 8)|BytesPerSecond|Máximo||Sin dimensiones|
|percentProcessorTime8|CPU (partición 8)|Porcentaje|Máximo||Sin dimensiones|
|connectedclients9|Clientes conectados (partición 9)|Count|Máximo||Sin dimensiones|
|totalcommandsprocessed9|Total de operaciones (partición 9)|Count|Total||Sin dimensiones|
|cachehits9|Aciertos de caché (partición 9)|Count|Total||Sin dimensiones|
|cachemisses9|Errores de caché (partición 9)|Count|Total||Sin dimensiones|
|getcommands9|Obtenciones (partición 9)|Count|Total||Sin dimensiones|
|setcommands9|Definiciones (partición 9)|Recuento|Total||Sin dimensiones|
|operationsPerSecond9|Operaciones por segundo (partición 9)|Count|Máximo||Sin dimensiones|
|evictedkeys9|Claves expulsadas (partición 9)|Count|Total||Sin dimensiones|
|totalkeys9|Total de claves (partición 9)|Count|Máximo||Sin dimensiones|
|expiredkeys9|Claves expiradas (partición 9)|Count|Total||Sin dimensiones|
|usedmemory9|Memoria usada (partición 9)|Bytes|Máximo||Sin dimensiones|
|usedmemoryRss9|Memoria RSS usada (partición 9)|Bytes|Máximo||Sin dimensiones|
|serverLoad9|Carga de servidor (partición 9)|Porcentaje|Máximo||Sin dimensiones|
|cacheWrite9|Escritura de caché (partición 9)|BytesPerSecond|Máximo||Sin dimensiones|
|cacheRead9|Lectura de caché (partición 9)|BytesPerSecond|Máximo||Sin dimensiones|
|percentProcessorTime9|CPU (partición 9)|Porcentaje|Máximo||Sin dimensiones|

## <a name="microsoftclassiccomputevirtualmachines"></a>Microsoft.ClassicCompute/virtualMachines

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|Porcentaje de CPU|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de unidades de proceso asignadas que las máquinas virtuales usan actualmente.|Sin dimensiones|
|Entrada de red|Red interna|Bytes|Total|Número de bytes que las máquinas virtuales recibieron en todas las interfaces de red (tráfico entrante).|Sin dimensiones|
|Salida de red|Red interna|Bytes|Total|Número de bytes que las máquinas virtuales enviaron en todas las interfaces de red (tráfico saliente).|Sin dimensiones|
|Bytes de lectura de disco/segundo|Lectura de disco|BytesPerSecond|Promedio|Promedio de bytes leídos del disco durante el período de supervisión.|Sin dimensiones|
|Bytes de escritura en disco/segundo|Escritura de disco|BytesPerSecond|Promedio|Promedio de bytes escritos en el disco durante el período de supervisión.|Sin dimensiones|
|Operaciones de lectura en disco por segundo|Operaciones de lectura de disco por segundo|CountPerSecond|Promedio|E/S por segundo de lectura en disco.|Sin dimensiones|
|Operaciones de escritura en disco por segundo|Operaciones de escritura por segundo en disco|CountPerSecond|Promedio|E/S por segundo de escritura en disco.|Sin dimensiones|

## <a name="microsoftclassiccomputedomainnamesslotsroles"></a>Microsoft.ClassicCompute/domainNames/slots/roles

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|Porcentaje de CPU|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de unidades de proceso asignadas que las máquinas virtuales usan actualmente.|RoleInstanceId|
|Entrada de red|Red interna|Bytes|Total|Número de bytes que las máquinas virtuales recibieron en todas las interfaces de red (tráfico entrante).|RoleInstanceId|
|Salida de red|Red interna|Bytes|Total|Número de bytes que las máquinas virtuales enviaron en todas las interfaces de red (tráfico saliente).|RoleInstanceId|
|Bytes de lectura de disco/segundo|Lectura de disco|BytesPerSecond|Promedio|Promedio de bytes leídos del disco durante el período de supervisión.|RoleInstanceId|
|Bytes de escritura en disco/segundo|Escritura de disco|BytesPerSecond|Promedio|Promedio de bytes escritos en el disco durante el período de supervisión.|RoleInstanceId|
|Operaciones de lectura en disco por segundo|Operaciones de lectura de disco por segundo|CountPerSecond|Promedio|E/S por segundo de lectura en disco.|RoleInstanceId|
|Operaciones de escritura en disco por segundo|Operaciones de escritura por segundo en disco|CountPerSecond|Promedio|E/S por segundo de escritura en disco.|RoleInstanceId|

## <a name="microsoftcognitiveservicesaccounts"></a>Microsoft.CognitiveServices/accounts

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|TotalCalls|Total de llamadas|Count|Total|Número total de llamadas.|ApiName, OperationName, región|
|SuccessfulCalls|Llamadas correctas|Count|Total|Número de llamadas correctas.|ApiName, OperationName, región|
|TotalErrors|Total de errores|Count|Total|Número total de llamadas con respuesta de error (código de respuesta HTTP 4xx o 5xx).|ApiName, OperationName, región|
|BlockedCalls|Llamadas bloqueadas|Count|Total|Número de llamadas que superaron el límite de tasa o de cuota.|ApiName, OperationName, región|
|ServerErrors|Errores del servidor|Count|Total|Número de llamadas con error interno del servicio (código de respuesta HTTP 5xx).|ApiName, OperationName, región|
|ClientErrors|Errores de cliente|Count|Total|Número de llamadas con error en el lado del cliente (código de respuesta HTTP 4xx).|ApiName, OperationName, región|
|DataIn|Datos entrantes|Bytes|Total|Tamaño de los datos entrantes, en bytes.|ApiName, OperationName, región|
|DataOut|Datos salientes|Bytes|Total|Tamaño de los datos salientes, en bytes.|ApiName, OperationName, región|
|Latencia|Latencia|MilliSeconds|Promedio|Latencia en milisegundos.|ApiName, OperationName, región|
|CharactersTranslated|Caracteres traducidos|Count|Total|Cantidad total de caracteres de la solicitud de texto entrante.|ApiName, OperationName, región|
|CharactersTrained|Caracteres entrenados|Count|Total|Número total de caracteres entrenado.|ApiName, OperationName, región|
|SpeechSessionDuration|Duración de la sesión de voz|Segundos|Total|Duración total de la sesión de voz en segundos.|ApiName, OperationName, región|
|TotalTransactions|Transacciones totales|Count|Total|Número total de transacciones.|Sin dimensiones|
|TotalTokenCalls|Llamadas de token totales|Count|Total|Número total de llamadas de token.|ApiName, OperationName, región|

## <a name="microsoftcomputevirtualmachines"></a>Microsoft.Compute/virtualMachines

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|Porcentaje de CPU|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de unidades de proceso asignadas que las máquinas virtuales está usando actualmente|Sin dimensiones|
|Entrada de red|Entrada de red facturable|Bytes|Total|Número de bytes facturables que las máquinas virtuales recibieron en todas las interfaces de red (tráfico entrante)|Sin dimensiones|
|Salida de red|Salida de red facturable|Bytes|Total|Número de bytes facturables que las máquinas virtuales enviaron en todas las interfaces de red (tráfico saliente)|Sin dimensiones|
|Bytes de lectura en disco|Bytes de lectura en disco|Bytes|Total|Bytes leídos del disco durante el período de supervisión|Sin dimensiones|
|Bytes de escritura en disco|Bytes de escritura de disco|Bytes|Total|Bytes escritos en el disco durante el período de supervisión|Sin dimensiones|
|Operaciones de lectura en disco por segundo|Operaciones de lectura de disco por segundo|CountPerSecond|Promedio|IOPS de lectura en disco|Sin dimensiones|
|Operaciones de escritura en disco por segundo|Operaciones de escritura por segundo en disco|CountPerSecond|Promedio|IOPS de escritura en disco|Sin dimensiones|
|Créditos de CPU restantes|Créditos de CPU restantes|Count|Promedio|Número total de créditos disponibles para la ráfaga|Sin dimensiones|
|Créditos de CPU consumidos|Créditos de CPU consumidos|Count|Promedio|Número total de créditos que consumió la máquina virtual|Sin dimensiones|
|Bytes de lectura de disco/s|Bytes de lectura de discos de datos/s (en desuso)|CountPerSecond|Promedio|Bytes/s leídos de un único disco durante el período de supervisión|RanuraID|
|Bytes de escritura en disco/s|Bytes de escritura de discos de datos/s (en desuso)|CountPerSecond|Promedio|Bytes/s escritos en un único disco durante el período de supervisión|RanuraID|
|Operaciones de lectura en disco por segundo|Operaciones de lectura de discos de datos/s (en desuso)|CountPerSecond|Promedio|IOPS de lectura de un único disco durante el período de supervisión|RanuraID|
|Operaciones de escritura en disco por segundo|Operaciones de escritura de discos de datos/s (en desuso)|CountPerSecond|Promedio|IOPS de escritura de un único disco durante el período de supervisión|RanuraID|
|Por disco QD|Disco de datos QD (en desuso)|Count|Promedio|Profundidad de la cola de disco de datos (o longitud de la cola)|SlotId|
|Sistema operativo por Bytes de lectura de disco/s|Bytes de lectura de discos del SO/s (en desuso)|CountPerSecond|Promedio|Bytes/s leídos de un único disco durante el período de supervisión del disco del SO|Sin dimensiones|
|Sistema operativo por disco Bytes escritos/s|Bytes de escritura de discos del SO/s (en desuso)|CountPerSecond|Promedio|Bytes/s escritos en un único disco durante el período de supervisión del disco del SO|Sin dimensiones|
|Sistema operativo por operaciones de lectura de disco/seg.|Operaciones de lectura de discos del SO/s (en desuso)|CountPerSecond|Promedio|IOPS de lectura de un único disco durante el período de supervisión para el disco del SO|Sin dimensiones|
|Sistema operativo por disco operaciones de escritura/s|Operaciones de escritura de discos del SO/s (en desuso)|CountPerSecond|Promedio|IOPS de escritura de un único disco durante el período de supervisión para el disco del SO|Sin dimensiones|
|Sistema operativo por disco QD|Disco del SO QD (en desuso)|Count|Promedio|Profundidad de la cola de disco de sistema operativo (o longitud de la cola)|Sin dimensiones|
|Bytes de lectura de discos/s|Bytes de lectura de discos de datos/segundo (versión preliminar)|CountPerSecond|Promedio|Bytes/s leídos de un único disco durante el período de supervisión|LUN|
|Bytes de escritura de discos de datos/s|Bytes de escritura de discos de datos/segundo (versión preliminar)|CountPerSecond|Promedio|Bytes/s escritos en un único disco durante el período de supervisión|LUN|
|Operaciones de lectura en disco por segundo|Operaciones de lectura de discos de datos/segundo (versión preliminar)|CountPerSecond|Promedio|IOPS de lectura de un único disco durante el período de supervisión|LUN|
|Operaciones de escritura en disco por segundo|Operaciones de escritura de discos de datos/segundo (versión preliminar)|CountPerSecond|Promedio|IOPS de escritura de un único disco durante el período de supervisión|LUN|
|Profundidad de cola del disco de datos|Profundidad de cola del disco de datos (versión preliminar)|Count|Promedio|Profundidad de la cola de disco de datos (o longitud de la cola)|LUN|
|Bytes de lectura de discos del SO/s|Bytes de lectura de discos de SO/segundo (versión preliminar)|CountPerSecond|Promedio|Bytes/s leídos de un único disco durante el período de supervisión del disco del SO|Sin dimensiones|
|Bytes de escritura de discos del SO/s|Bytes de escritura de discos de SO/segundo (versión preliminar)|CountPerSecond|Promedio|Bytes/s escritos en un único disco durante el período de supervisión del disco del SO|Sin dimensiones|
|Operaciones de lectura en disco por segundo|Operaciones de lectura de discos de SO/segundo (versión preliminar)|CountPerSecond|Promedio|IOPS de lectura de un único disco durante el período de supervisión para el disco del SO|Sin dimensiones|
|Operaciones de escritura en disco por segundo|Operaciones de escritura de discos de SO/segundo (versión preliminar)|CountPerSecond|Promedio|IOPS de escritura de un único disco durante el período de supervisión para el disco del SO|Sin dimensiones|
|Profundidad de la cola del disco del SO|Profundidad de la cola del disco del SO (versión preliminar)|Count|Promedio|Profundidad de la cola de disco de sistema operativo (o longitud de la cola)|Sin dimensiones|
|Flujos entrantes|Flujos entrantes (versión preliminar)|Count|Promedio|Los flujos entrantes son el número de flujos actuales en la dirección entrante (el tráfico dirigido a la VM).|Sin dimensiones|
|Flujos salientes|Flujos salientes (versión preliminar)|Count|Promedio|Los flujos salientes son los flujos actuales en dirección de salida (tráfico que sale de la VM).|Sin dimensiones|
|Velocidad máxima de creación de flujos entrantes|Velocidad máxima de creación de flujos entrantes (versión preliminar)|CountPerSecond|Promedio|Velocidad máxima de creación de flujos entrantes (tráfico dirigido a la VM)|Sin dimensiones|
|Velocidad máxima de creación de flujos salientes|Velocidad máxima de creación de flujos salientes (versión preliminar)|CountPerSecond|Promedio|Velocidad máxima de creación de flujos salientes (tráfico dirigido fuera de la VM)|Sin dimensiones|
|Aciertos de lectura de la caché de discos de datos prémium|Aciertos de lectura de la caché de discos de datos prémium (versión preliminar)|Porcentaje|Promedio|Aciertos de lectura de la caché de discos de datos prémium|LUN|
|Error de lectura de la caché de discos de datos prémium|Error de lectura de la caché de discos de datos prémium (versión preliminar)|Porcentaje|Promedio|Error de lectura de la caché de discos de datos prémium|LUN|
|Aciertos de lectura de la caché de discos del SO prémium|Aciertos de lectura de la caché de discos del SO prémium (versión preliminar)|Porcentaje|Promedio|Aciertos de lectura de la caché de discos del SO prémium|Sin dimensiones|
|Error de lectura de la caché de discos prémium del SO|Error de lectura de la caché de discos del SO prémium (versión preliminar)|Porcentaje|Promedio|Error de lectura de la caché de discos prémium del SO|Sin dimensiones|
|Entrada de red total|Entrada de red total|Bytes|Total|Número de bytes que las máquinas virtuales recibieron en todas las interfaces de red (tráfico entrante)|Sin dimensiones|
|Salida de red total|Salida de red total|Bytes|Total|Número de bytes que las máquinas virtuales enviaron en todas las interfaces de red (tráfico saliente)|Sin dimensiones|

## <a name="microsoftcomputevirtualmachinescalesets"></a>Microsoft.Compute/virtualMachineScaleSets

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|Porcentaje de CPU|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de unidades de proceso asignadas que las máquinas virtuales está usando actualmente|VMName|
|Entrada de red|Entrada de red facturable|Bytes|Total|Número de bytes facturables que las máquinas virtuales recibieron en todas las interfaces de red (tráfico entrante)|VMName|
|Salida de red|Salida de red facturable|Bytes|Total|Número de bytes facturables que las máquinas virtuales enviaron en todas las interfaces de red (tráfico saliente)|VMName|
|Bytes de lectura en disco|Bytes de lectura en disco|Bytes|Total|Bytes leídos del disco durante el período de supervisión|VMName|
|Bytes de escritura en disco|Bytes de escritura de disco|Bytes|Total|Bytes escritos en el disco durante el período de supervisión|VMName|
|Operaciones de lectura en disco por segundo|Operaciones de lectura de disco por segundo|CountPerSecond|Promedio|IOPS de lectura en disco|VMName|
|Operaciones de escritura en disco por segundo|Operaciones de escritura por segundo en disco|CountPerSecond|Promedio|IOPS de escritura en disco|VMName|
|Créditos de CPU restantes|Créditos de CPU restantes|Count|Promedio|Número total de créditos disponibles para la ráfaga|Sin dimensiones|
|Créditos de CPU consumidos|Créditos de CPU consumidos|Count|Promedio|Número total de créditos que consumió la máquina virtual|Sin dimensiones|
|Bytes de lectura de disco/s|Bytes de lectura de discos de datos/s (en desuso)|CountPerSecond|Promedio|Bytes/s leídos de un único disco durante el período de supervisión|RanuraID|
|Bytes de escritura en disco/s|Bytes de escritura de discos de datos/s (en desuso)|CountPerSecond|Promedio|Bytes/s escritos en un único disco durante el período de supervisión|RanuraID|
|Operaciones de lectura en disco por segundo|Operaciones de lectura de discos de datos/s (en desuso)|CountPerSecond|Promedio|IOPS de lectura de un único disco durante el período de supervisión|RanuraID|
|Operaciones de escritura en disco por segundo|Operaciones de escritura de discos de datos/s (en desuso)|CountPerSecond|Promedio|IOPS de escritura de un único disco durante el período de supervisión|RanuraID|
|Por disco QD|Disco de datos QD (en desuso)|Count|Promedio|Profundidad de la cola de disco de datos (o longitud de la cola)|SlotId|
|Sistema operativo por Bytes de lectura de disco/s|Bytes de lectura de discos del SO/s (en desuso)|CountPerSecond|Promedio|Bytes/s leídos de un único disco durante el período de supervisión del disco del SO|Sin dimensiones|
|Sistema operativo por disco Bytes escritos/s|Bytes de escritura de discos del SO/s (en desuso)|CountPerSecond|Promedio|Bytes/s escritos en un único disco durante el período de supervisión del disco del SO|Sin dimensiones|
|Sistema operativo por operaciones de lectura de disco/seg.|Operaciones de lectura de discos del SO/s (en desuso)|CountPerSecond|Promedio|IOPS de lectura de un único disco durante el período de supervisión para el disco del SO|Sin dimensiones|
|Sistema operativo por disco operaciones de escritura/s|Operaciones de escritura de discos del SO/s (en desuso)|CountPerSecond|Promedio|IOPS de escritura de un único disco durante el período de supervisión para el disco del SO|Sin dimensiones|
|Sistema operativo por disco QD|Disco del SO QD (en desuso)|Count|Promedio|Profundidad de la cola de disco de sistema operativo (o longitud de la cola)|Sin dimensiones|
|Bytes de lectura de discos/s|Bytes de lectura de discos de datos/segundo (versión preliminar)|CountPerSecond|Promedio|Bytes/s leídos de un único disco durante el período de supervisión|LUN, VMName|
|Bytes de escritura de discos de datos/s|Bytes de escritura de discos de datos/segundo (versión preliminar)|CountPerSecond|Promedio|Bytes/s escritos en un único disco durante el período de supervisión|LUN, VMName|
|Operaciones de lectura en disco por segundo|Operaciones de lectura de discos de datos/segundo (versión preliminar)|CountPerSecond|Promedio|IOPS de lectura de un único disco durante el período de supervisión|LUN, VMName|
|Operaciones de escritura en disco por segundo|Operaciones de escritura de discos de datos/segundo (versión preliminar)|CountPerSecond|Promedio|IOPS de escritura de un único disco durante el período de supervisión|LUN, VMName|
|Profundidad de cola del disco de datos|Profundidad de cola del disco de datos (versión preliminar)|Count|Promedio|Profundidad de la cola de disco de datos (o longitud de la cola)|LUN, VMName|
|Bytes de lectura de discos del SO/s|Bytes de lectura de discos de SO/segundo (versión preliminar)|CountPerSecond|Promedio|Bytes/s leídos de un único disco durante el período de supervisión del disco del SO|VMName|
|Bytes de escritura de discos del SO/s|Bytes de escritura de discos de SO/segundo (versión preliminar)|CountPerSecond|Promedio|Bytes/s escritos en un único disco durante el período de supervisión del disco del SO|VMName|
|Operaciones de lectura en disco por segundo|Operaciones de lectura de discos de SO/segundo (versión preliminar)|CountPerSecond|Promedio|IOPS de lectura de un único disco durante el período de supervisión para el disco del SO|VMName|
|Operaciones de escritura en disco por segundo|Operaciones de escritura de discos de SO/segundo (versión preliminar)|CountPerSecond|Promedio|IOPS de escritura de un único disco durante el período de supervisión para el disco del SO|VMName|
|Profundidad de la cola del disco del SO|Profundidad de la cola del disco del SO (versión preliminar)|Count|Promedio|Profundidad de la cola de disco de sistema operativo (o longitud de la cola)|VMName|
|Flujos entrantes|Flujos entrantes (versión preliminar)|Count|Promedio|Los flujos entrantes son el número de flujos actuales en la dirección entrante (el tráfico dirigido a la VM).|VMName|
|Flujos salientes|Flujos salientes (versión preliminar)|Count|Promedio|Los flujos salientes son los flujos actuales en dirección de salida (tráfico que sale de la VM).|VMName|
|Velocidad máxima de creación de flujos entrantes|Velocidad máxima de creación de flujos entrantes (versión preliminar)|CountPerSecond|Promedio|Velocidad máxima de creación de flujos entrantes (tráfico dirigido a la VM)|VMName|
|Velocidad máxima de creación de flujos salientes|Velocidad máxima de creación de flujos salientes (versión preliminar)|CountPerSecond|Promedio|Velocidad máxima de creación de flujos salientes (tráfico dirigido fuera de la VM)|VMName|
|Aciertos de lectura de la caché de discos de datos prémium|Aciertos de lectura de la caché de discos de datos prémium (versión preliminar)|Porcentaje|Promedio|Aciertos de lectura de la caché de discos de datos prémium|LUN, VMName|
|Error de lectura de la caché de discos de datos prémium|Error de lectura de la caché de discos de datos prémium (versión preliminar)|Porcentaje|Promedio|Error de lectura de la caché de discos de datos prémium|LUN, VMName|
|Aciertos de lectura de la caché de discos del SO prémium|Aciertos de lectura de la caché de discos del SO prémium (versión preliminar)|Porcentaje|Promedio|Aciertos de lectura de la caché de discos del SO prémium|VMName|
|Error de lectura de la caché de discos prémium del SO|Error de lectura de la caché de discos del SO prémium (versión preliminar)|Porcentaje|Promedio|Error de lectura de la caché de discos prémium del SO|VMName|
|Entrada de red total|Entrada de red total|Bytes|Total|Número de bytes que las máquinas virtuales recibieron en todas las interfaces de red (tráfico entrante)|VMName|
|Salida de red total|Salida de red total|Bytes|Total|Número de bytes que las máquinas virtuales enviaron en todas las interfaces de red (tráfico saliente)|VMName|

## <a name="microsoftcontainerinstancecontainergroups"></a>Microsoft.ContainerInstance/containerGroups

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|CpuUsage|Uso de CPU|Count|Promedio|Uso de la CPU en todos los núcleos en millares de núcleos.|containerName|
|MemoryUsage|Uso de memoria|Bytes|Promedio|Uso de memoria total en bytes.|containerName|
|NetworkBytesReceivedPerSecond|Bytes de red recibidos por segundo|Bytes|Promedio|Los bytes de red recibidos por segundo.|Sin dimensiones|
|NetworkBytesTransmittedPerSecond|Bytes de red transmitidos por segundo|Bytes|Promedio|Los bytes de red transmitidos por segundo.|Sin dimensiones|

## <a name="microsoftcontainerregistryregistries"></a>Microsoft.ContainerRegistry/registries

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|TotalPullCount|Recuento total de la incorporación de cambios|Count|Promedio|Número de la imagen se extrae en total|Sin dimensiones|
|SuccessfulPullCount|Recuento de extracción correcta|Count|Promedio|Número de extracciones de la imagen correcta|Sin dimensiones|
|TotalPushCount|Recuento total de inserción|Count|Promedio|Número de la imagen se inserta en total|Sin dimensiones|
|SuccessfulPushCount|Recuento de inserción correcta|Count|Promedio|Número de inserciones de la imagen correcta|Sin dimensiones|
|Valor de RunDuration|Duración de la ejecución|Milisegundos|Total|Duración de la ejecución en milisegundos|Sin dimensiones|

## <a name="microsoftcontainerservicemanagedclusters"></a>Microsoft.ContainerService/managedClusters

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|kube_node_status_allocatable_cpu_cores|Número total de núcleos de CPU disponibles en un clúster administrado|Count|Total|Número total de núcleos de CPU disponibles en un clúster administrado|Sin dimensiones|
|kube_node_status_allocatable_memory_bytes|Cantidad total de memoria disponible en un clúster administrado|Bytes|Total|Cantidad total de memoria disponible en un clúster administrado|Sin dimensiones|
|kube_pod_status_ready|Número de pods con estado Listo|Count|Total|Número de pods con estado Listo|espacio de nombres, pod|
|kube_node_status_condition|Estados de diversas condiciones de nodo|Count|Total|Estados de diversas condiciones de nodo|condición, estado, status2, nodo|
|kube_pod_status_phase|Número de pods por fase|Count|Total|Número de pods por fase|fase, espacio de nombres, pod|

## <a name="microsoftcustomerinsightshubs"></a>Microsoft.CustomerInsights/hubs

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|DCIApiCalls|Llamadas API Customer Insights|Count|Total||Sin dimensiones|
|DCIMappingImportOperationSuccessfulLines|Líneas correctas de la operación de importación de asignaciones|Count|Total||Sin dimensiones|
|DCIMappingImportOperationFailedLines|Líneas con error de la operación de importación de asignaciones|Count|Total||Sin dimensiones|
|DCIMappingImportOperationTotalLines|Líneas totales de la operación de importación de asignaciones|Count|Total||Sin dimensiones|
|DCIMappingImportOperationRuntimeInSeconds|Tiempo de ejecución en segundos de la operación de importación de asignaciones|Segundos|Total||Sin dimensiones|
|DCIOutboundProfileExportSucceeded|Exportación correcta del perfil saliente|Count|Total||Sin dimensiones|
|DCIOutboundProfileExportFailed|Error de exportación del perfil saliente|Count|Total||Sin dimensiones|
|DCIOutboundProfileExportDuration|Duración de la exportación del perfil saliente|Segundos|Total||Sin dimensiones|
|DCIOutboundKpiExportSucceeded|Exportación correcta de KPI saliente|Count|Total||Sin dimensiones|
|DCIOutboundKpiExportFailed|Error de exportación de KPI saliente|Count|Total||Sin dimensiones|
|DCIOutboundKpiExportDuration|Duración de exportación de KPI saliente|Segundos|Total||Sin dimensiones|
|DCIOutboundKpiExportStarted|Inicio de exportación de KPI saliente|Segundos|Total||Sin dimensiones|
|DCIOutboundKpiRecordCount|Recuento de registros de KPI saliente|Segundos|Total||Sin dimensiones|
|DCIOutboundProfileExportCount|Recuento de exportación de perfil saliente|Segundos|Total||Sin dimensiones|
|DCIOutboundInitialProfileExportFailed|Recuento de exportación de perfil inicial saliente|Segundos|Total||Sin dimensiones|
|DCIOutboundInitialProfileExportSucceeded|Exportación correcta de perfil inicial saliente|Segundos|Total||Sin dimensiones|
|DCIOutboundInitialKpiExportFailed|Error de exportación de KPI inicial saliente|Segundos|Total||Sin dimensiones|
|DCIOutboundInitialKpiExportSucceeded|Exportación correcta de KPI inicial saliente|Segundos|Total||Sin dimensiones|
|DCIOutboundInitialProfileExportDurationInSeconds|Duración en segundos de exportación de perfil inicial saliente|Segundos|Total||Sin dimensiones|
|AdlaJobForStandardKpiFailed|Error en segundos de trabajo de Adla para KPI estándar|Segundos|Total||Sin dimensiones|
|AdlaJobForStandardKpiTimeOut|Tiempo de espera en segundos del trabajo de Adla para KPI estándar|Segundos|Total||Sin dimensiones|
|AdlaJobForStandardKpiCompleted|Tiempo de finalización en segundos del trabajo de Adla para KPI estándar|Segundos|Total||Sin dimensiones|
|ImportASAValuesFailed|Error de recuento de valores ASA de importación|Count|Total||Sin dimensiones|
|ImportASAValuesSucceeded|Recuento correcto de valores ASA de importación|Count|Total||Sin dimensiones|
|DCIProfilesCount|Recuento de instancias del perfil|Count|Última||Sin dimensiones|
|DCIInteractionsPerMonthCount|Recuento de interacciones por mes|Count|Última||Sin dimensiones|
|DCIKpisCount|Recuento de KPI|Count|Última||Sin dimensiones|
|DCISegmentsCount|Número de segmentos|Count|Última||Sin dimensiones|
|DCIPredictiveMatchPoliciesCount|Recuento de coincidencias predictivo|Count|Última||Sin dimensiones|
|DCIPredictionsCount|Recuento de predicción|Count|Última||Sin dimensiones|

## <a name="microsoftdataboxedgedataboxedgedevices"></a>Microsoft.DataBoxEdge/dataBoxEdgeDevices

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|NICReadThroughput|Rendimiento de lectura (red)|BytesPerSecond|Promedio|El rendimiento de lectura de la interfaz de red en el dispositivo en el período de notificación para todos los volúmenes en la puerta de enlace.|InstanceName|
|NICWriteThroughput|Escribir el rendimiento (red)|BytesPerSecond|Promedio|El rendimiento de escritura de la interfaz de red en el dispositivo en el período de notificación para todos los volúmenes en la puerta de enlace.|InstanceName|
|CloudReadThroughputPerShare|Rendimiento de descarga en la nube (recurso compartido)|BytesPerSecond|Promedio|El rendimiento de descarga a Azure desde un recurso compartido durante el período de notificación.|Compartir|
|CloudUploadThroughputPerShare|Rendimiento de la carga en la nube (recurso compartido)|BytesPerSecond|Promedio|El rendimiento de carga en Azure desde un recurso compartido durante el período de notificación.|Compartir|
|BytesUploadedToCloudPerShare|En la nube de Bytes cargados (recurso compartido)|Bytes|Promedio|El número total de bytes que se carga en Azure desde un recurso compartido durante el período de notificación.|Compartir|
|TotalCapacity|Capacidad total|Bytes|Promedio|Capacidad total|Sin dimensiones|
|AvailableCapacity|Capacidad disponible|Bytes|Promedio|La capacidad disponible en bytes durante el período de notificación.|Sin dimensiones|
|CloudUploadThroughput|Rendimiento de la carga en la nube|BytesPerSecond|Promedio|El rendimiento de carga en la nube en Azure durante el período de notificación.|Sin dimensiones|
|CloudReadThroughput|Rendimiento de descarga en la nube|BytesPerSecond|Promedio|El rendimiento de descarga en la nube en Azure durante el período de notificación.|Sin dimensiones|
|BytesUploadedToCloud|En la nube de Bytes cargados (dispositivo)|Bytes|Promedio|El número total de bytes que se carga en Azure desde un dispositivo durante el período de notificación.|Sin dimensiones|
|HyperVVirtualProcessorUtilization|Calcular uso de CPU|Porcentaje|Promedio|Porcentaje de uso de CPU|InstanceName|
|HyperVMemoryUtilization|Calcular uso de memoria|Porcentaje|Promedio|Cantidad de memoria RAM en uso|InstanceName|

## <a name="microsoftdatafactorydatafactories"></a>Microsoft.DataFactory/datafactories

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|FailedRuns|Ejecuciones con errores|Count|Total||pipelineName, activityName|
|SuccessfulRuns|Ejecuciones correctas|Count|Total||pipelineName, activityName|

## <a name="microsoftdatafactoryfactories"></a>Microsoft.DataFactory/factories

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|PipelineFailedRuns|Métricas de ejecuciones de canalización erróneas|Count|Total||FailureType, Name|
|PipelineSucceededRuns|Las métricas de ejecuciones de canalización se realizaron correctamente|Count|Total||FailureType, Name|
|ActivityFailedRuns|Métricas de ejecuciones de actividad erróneas|Count|Total||ActivityType, PipelineName, FailureType, Name|
|ActivitySucceededRuns|Métricas de ejecución de actividad realizadas correctamente|Count|Total||ActivityType, PipelineName, FailureType, Name|
|TriggerFailedRuns|Métricas de ejecuciones de desencadenador erróneas|Count|Total||Name, FailureType|
|TriggerSucceededRuns|Métricas de ejecuciones de desencadenador realizadas correctamente|Count|Total||Name, FailureType|
|IntegrationRuntimeCpuPercentage|Uso de la CPU de entorno de ejecución de integración|Porcentaje|Promedio||IntegrationRuntimeName, NodeName|
|IntegrationRuntimeAvailableMemory|Memoria disponible de entorno de ejecución de integración|Bytes|Promedio||IntegrationRuntimeName, NodeName|
|MaxAllowedResourceCount|Número máximo de entidades permitidas|Count|Máximo||Sin dimensiones|
|MaxAllowedFactorySizeInGbUnits|Máximo permitido de tamaño de fábrica (unidad de GB)|Count|Máximo||Sin dimensiones|
|ResourceCount|Recuento total de entidades|Count|Máximo||Sin dimensiones|
|FactorySizeInGbUnits|Tamaño total de fábrica (unidad de GB)|Count|Máximo||Sin dimensiones|

## <a name="microsoftdatalakeanalyticsaccounts"></a>Microsoft.DataLakeAnalytics/accounts

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|JobEndedSuccess|Trabajos correctos|Count|Total|Recuento de trabajos realizados correctamente|Sin dimensiones|
|JobEndedFailure|Trabajos con error|Count|Total|Recuento de trabajos con error|Sin dimensiones|
|JobEndedCancelled|Trabajos cancelados|Count|Total|Recuento de trabajos cancelados.|Sin dimensiones|
|JobAUEndedSuccess|Tiempo AU correcto|Segundos|Total|Tiempo total AU para trabajos realizados correctamente|Sin dimensiones|
|JobAUEndedFailure|Error de tiempo AU|Segundos|Total|Tiempo total AU para trabajos con error|Sin dimensiones|
|JobAUEndedCancelled|Tiempo AU canceladas|Segundos|Total|Tiempo total AU para trabajos cancelados.|Sin dimensiones|

## <a name="microsoftdatalakestoreaccounts"></a>Microsoft.DataLakeStore/accounts

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|TotalStorage|Almacenamiento total|Bytes|Máximo|Cantidad total de datos almacenados en la cuenta.|Sin dimensiones|
|DataWritten|Datos escritos|Bytes|Total|Cantidad total de datos escritos en la cuenta.|Sin dimensiones|
|DataRead|Datos leídos|Bytes|Total|Cantidad total de datos leídos de la cuenta.|Sin dimensiones|
|WriteRequests|Solicitudes de escritura|Count|Total|Número de solicitudes de escritura de datos en la cuenta.|Sin dimensiones|
|ReadRequests|Solicitudes de lectura|Count|Total|Número de solicitudes de lectura de datos de la cuenta.|Sin dimensiones|

## <a name="microsoftdbformariadbservers"></a>Microsoft.DBforMariaDB/servers

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|cpu_percent|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de CPU|Sin dimensiones|
|memory_percent|Porcentaje de memoria|Porcentaje|Promedio|Porcentaje de memoria|Sin dimensiones|
|io_consumption_percent|Porcentaje de E/S|Porcentaje|Promedio|Porcentaje de E/S|Sin dimensiones|
|storage_percent|Porcentaje de almacenamiento|Porcentaje|Promedio|Porcentaje de almacenamiento|Sin dimensiones|
|storage_used|Almacenamiento usado|Bytes|Promedio|Almacenamiento usado|Sin dimensiones|
|storage_limit|Límite de almacenamiento|Bytes|Promedio|Límite de almacenamiento|Sin dimensiones|
|serverlog_storage_percent|Porcentaje de almacenamiento del registro del servidor|Porcentaje|Promedio|Porcentaje de almacenamiento del registro del servidor|Sin dimensiones|
|serverlog_storage_usage|Almacenamiento del registro del servidor usado|Bytes|Promedio|Almacenamiento del registro del servidor usado|Sin dimensiones|
|serverlog_storage_limit|Límite de almacenamiento del registro del servidor|Bytes|Promedio|Límite de almacenamiento del registro del servidor|Sin dimensiones|
|active_connections|Conexiones activas|Count|Promedio|Conexiones activas|Sin dimensiones|
|connections_failed|Conexiones con error|Count|Total|Conexiones con error|Sin dimensiones|
|seconds_behind_master|Intervalo de replicación en segundos|Count|Promedio|Intervalo de replicación en segundos|Sin dimensiones|
|backup_storage_used|Almacenamiento de copia de seguridad utilizado|Bytes|Promedio|Almacenamiento de copia de seguridad utilizado|Sin dimensiones|
|network_bytes_egress|Salida de red|Bytes|Total|Red externa a través de conexiones activas|Sin dimensiones|
|network_bytes_ingress|Entrada de red|Bytes|Total|Red interna a través de conexiones activas|Sin dimensiones|

## <a name="microsoftdbformysqlservers"></a>Microsoft.DBforMySQL/servers

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|cpu_percent|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de CPU|Sin dimensiones|
|memory_percent|Porcentaje de memoria|Porcentaje|Promedio|Porcentaje de memoria|Sin dimensiones|
|io_consumption_percent|Porcentaje de E/S|Porcentaje|Promedio|Porcentaje de E/S|Sin dimensiones|
|storage_percent|Porcentaje de almacenamiento|Porcentaje|Promedio|Porcentaje de almacenamiento|Sin dimensiones|
|storage_used|Almacenamiento usado|Bytes|Promedio|Almacenamiento usado|Sin dimensiones|
|storage_limit|Límite de almacenamiento|Bytes|Promedio|Límite de almacenamiento|Sin dimensiones|
|serverlog_storage_percent|Porcentaje de almacenamiento del registro del servidor|Porcentaje|Promedio|Porcentaje de almacenamiento del registro del servidor|Sin dimensiones|
|serverlog_storage_usage|Almacenamiento del registro del servidor usado|Bytes|Promedio|Almacenamiento del registro del servidor usado|Sin dimensiones|
|serverlog_storage_limit|Límite de almacenamiento del registro del servidor|Bytes|Promedio|Límite de almacenamiento del registro del servidor|Sin dimensiones|
|active_connections|Conexiones activas|Count|Promedio|Conexiones activas|Sin dimensiones|
|connections_failed|Conexiones con error|Count|Total|Conexiones con error|Sin dimensiones|
|seconds_behind_master|Intervalo de replicación en segundos|Count|Promedio|Intervalo de replicación en segundos|Sin dimensiones|
|backup_storage_used|Almacenamiento de copia de seguridad utilizado|Bytes|Promedio|Almacenamiento de copia de seguridad utilizado|Sin dimensiones|
|network_bytes_egress|Salida de red|Bytes|Total|Red externa a través de conexiones activas|Sin dimensiones|
|network_bytes_ingress|Entrada de red|Bytes|Total|Red interna a través de conexiones activas|Sin dimensiones|

## <a name="microsoftdbforpostgresqlservers"></a>Microsoft.DBforPostgreSQL/servers

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|cpu_percent|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de CPU|Sin dimensiones|
|memory_percent|Porcentaje de memoria|Porcentaje|Promedio|Porcentaje de memoria|Sin dimensiones|
|io_consumption_percent|Porcentaje de E/S|Porcentaje|Promedio|Porcentaje de E/S|Sin dimensiones|
|storage_percent|Porcentaje de almacenamiento|Porcentaje|Promedio|Porcentaje de almacenamiento|Sin dimensiones|
|storage_used|Almacenamiento usado|Bytes|Promedio|Almacenamiento usado|Sin dimensiones|
|storage_limit|Límite de almacenamiento|Bytes|Promedio|Límite de almacenamiento|Sin dimensiones|
|serverlog_storage_percent|Porcentaje de almacenamiento del registro del servidor|Porcentaje|Promedio|Porcentaje de almacenamiento del registro del servidor|Sin dimensiones|
|serverlog_storage_usage|Almacenamiento del registro del servidor usado|Bytes|Promedio|Almacenamiento del registro del servidor usado|Sin dimensiones|
|serverlog_storage_limit|Límite de almacenamiento del registro del servidor|Bytes|Promedio|Límite de almacenamiento del registro del servidor|Sin dimensiones|
|active_connections|Conexiones activas|Count|Promedio|Conexiones activas|Sin dimensiones|
|connections_failed|Conexiones con error|Count|Total|Conexiones con error|Sin dimensiones|
|backup_storage_used|Almacenamiento de copia de seguridad utilizado|Bytes|Promedio|Almacenamiento de copia de seguridad utilizado|Sin dimensiones|
|network_bytes_egress|Salida de red|Bytes|Total|Red externa a través de conexiones activas|Sin dimensiones|
|network_bytes_ingress|Entrada de red|Bytes|Total|Red interna a través de conexiones activas|Sin dimensiones|
|pg_replica_log_delay_in_seconds|Retraso entre réplicas|Segundos|Máximo|Réplica retardo en segundos|Sin dimensiones|
|pg_replica_log_delay_in_bytes|Retraso máximo entre réplicas|Bytes|Máximo|Retardo en bytes de la réplica de retraso al máximo|Sin dimensiones|

## <a name="microsoftdbforpostgresqlserversv2"></a>Microsoft.DBforPostgreSQL/serversv2

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|cpu_percent|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de CPU|Sin dimensiones|
|memory_percent|Porcentaje de memoria|Porcentaje|Promedio|Porcentaje de memoria|Sin dimensiones|
|e/s por segundo|E/S|Count|Promedio|Operaciones de E/S por segundo|Sin dimensiones|
|storage_percent|Porcentaje de almacenamiento|Porcentaje|Promedio|Porcentaje de almacenamiento|Sin dimensiones|
|storage_used|Almacenamiento usado|Bytes|Promedio|Almacenamiento usado|Sin dimensiones|
|active_connections|Conexiones activas|Count|Promedio|Conexiones activas|Sin dimensiones|
|network_bytes_egress|Salida de red|Bytes|Total|Red externa a través de conexiones activas|Sin dimensiones|
|network_bytes_ingress|Entrada de red|Bytes|Total|Red interna a través de conexiones activas|Sin dimensiones|

## <a name="microsoftdevicesiothubs"></a>Microsoft.Devices/IotHubs

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|d2c.telemetry.ingress.allProtocol|Intentos de envío de mensajes de telemetría|Count|Total|Número de mensajes de telemetría de dispositivo a la nube para enviar a IoT Hub|Sin dimensiones|
|d2c.telemetry.ingress.success|Mensajes de telemetría enviados|Count|Total|Número de mensajes de telemetría de dispositivo a la nube enviados correctamente a IoT Hub|Sin dimensiones|
|c2d.commands.egress.complete.success|Comandos completados|Count|Total|Número de comandos de la nube al dispositivo que el dispositivo ha completado correctamente|Sin dimensiones|
|c2d.commands.egress.abandon.success|Comandos abandonados|Count|Total|Número de comandos de la nube al dispositivo que el dispositivo ha abandonado|Sin dimensiones|
|c2d.commands.egress.reject.success|Comandos rechazados|Count|Total|Número de comandos de la nube al dispositivo que el dispositivo ha rechazado|Sin dimensiones|
|devices.totalDevices|Total de dispositivos (en desuso)|Count|Total|Número de dispositivos registrados en IoT Hub|Sin dimensiones|
|devices.connectedDevices.allProtocol|Dispositivos conectados (en desuso) |Count|Total|Número de dispositivos conectados a IoT Hub|Sin dimensiones|
|d2c.telemetry.egress.success|Enrutamiento: mensajes de telemetría entregados|Count|Total|Número de veces que los mensajes se han entregado correctamente a todos los puntos de conexión mediante el enrutamiento de IoT Hub. Si un mensaje se enruta a varios puntos de conexión, este valor aumenta en uno por cada entrega correcta. Si un mensaje se enruta al mismo punto de conexión varias veces, este valor aumenta en uno por cada entrega correcta.|Sin dimensiones|
|d2c.telemetry.egress.dropped|Enrutamiento: mensajes de telemetría quitados |Count|Total|Número de veces que se han quitado los mensajes con el enrutamiento de IoT Hub a causa de puntos de conexión inactivos. Este valor no cuenta los mensajes entregados a la ruta de reserva porque los mensajes quitados no se entregan allí.|Sin dimensiones|
|d2c.telemetry.egress.orphaned|Enrutamiento: mensajes de telemetría huérfanos |Count|Total|Número de veces que los mensajes eran huérfanos a causa del enrutamiento de IoT Hub porque no coincidían con las reglas de enrutamiento (incluida la regla de reserva). |Sin dimensiones|
|d2c.telemetry.egress.invalid|Enrutamiento: mensajes de telemetría incompatibles|Count|Total|Número de veces que el enrutamiento de IoT Hub no ha podido entregar mensajes debido a una incompatibilidad con el punto de conexión. Este valor no incluye los reintentos.|Sin dimensiones|
|d2c.telemetry.egress.fallback|Enrutamiento: mensajes entregados a la reserva|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes al punto de conexión asociado con la ruta de reserva.|Sin dimensiones|
|d2c.endpoints.egress.eventHubs|Enrutamiento: mensajes entregados al centro de eventos|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes correctamente a los puntos de conexión del centro de eventos.|Sin dimensiones|
|d2c.endpoints.latency.eventHubs|Enrutamiento: latencia de mensajes para el centro de eventos|Milisegundos|Promedio|Latencia media (milisegundos) entre la entrada de mensajes a IoT Hub y la entrada de mensajes a un punto de conexión del centro de eventos.|Sin dimensiones|
|d2c.endpoints.egress.serviceBusQueues|Enrutamiento: mensajes entregados a la cola de Service Bus|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes correctamente a los puntos de conexión de la cola de Service Bus.|Sin dimensiones|
|d2c.endpoints.latency.serviceBusQueues|Enrutamiento: latencia de mensajes de la cola de Service Bus|Milisegundos|Promedio|Latencia media (milisegundos) entre la entrada de mensajes a IoT Hub y la entrada de mensajes de telemetría a un punto de conexión de la cola de Service Bus.|Sin dimensiones|
|d2c.endpoints.egress.serviceBusTopics|Enrutamiento: mensajes entregados al tema de Service Bus|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes correctamente a los puntos de conexión del tema de Service Bus.|Sin dimensiones|
|d2c.endpoints.latency.serviceBusTopics|Enrutamiento: latencia de mensajes del tema de Service Bus|Milisegundos|Promedio|Latencia media (milisegundos) entre la entrada de mensajes a IoT Hub y la entrada de mensajes de telemetría a un punto de conexión del tema de Service Bus.|Sin dimensiones|
|d2c.endpoints.egress.builtIn.events|Enrutamiento: mensajes entregados a mensajes/eventos|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes correctamente al punto de conexión integrado (mensajes/eventos).|Sin dimensiones|
|d2c.endpoints.latency.builtIn.events|Enrutamiento: latencia de mensajes para mensajes/eventos|Milisegundos|Promedio|Latencia media (milisegundos) entre la entrada de mensajes a IoT Hub y la entrada de mensajes de telemetría al punto de conexión integrado (mensajes/eventos).|Sin dimensiones|
|d2c.endpoints.egress.storage|Enrutamiento: mensajes entregados al almacenamiento|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes correctamente a los puntos de conexión de almacenamiento.|Sin dimensiones|
|d2c.endpoints.latency.storage|Enrutamiento: latencia de mensajes para almacenamiento|Milisegundos|Promedio|Latencia media (milisegundos) entre la entrada de mensajes a IoT Hub y la entrada de mensajes de telemetría a un punto de conexión de almacenamiento.|Sin dimensiones|
|d2c.endpoints.egress.storage.bytes|Enrutamiento: datos entregados al almacenamiento|Bytes|Total|Cantidad de datos (bytes) que el enrutamiento de IoT Hub ha entregado a los punto de conexión de almacenamiento.|Sin dimensiones|
|d2c.endpoints.egress.storage.blobs|Enrutamiento: blobs entregados al almacenamiento|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado blobs a los puntos de conexión de almacenamiento.|Sin dimensiones|
|EventGridDeliveries|Entregas de cuadrícula de eventos (versión preliminar)|Count|Total|El número de solicitudes de IoT Hub emitir eventos a Event Grid. Este número incluye solicitudes correctas e incorrectas. Usar el resultado de la dimensión para el número de diferentes tipos de respuesta. Para ver el lugar en las solicitudes proceden de, use la dimensión EventType.|Resultado, EventType|
|EventGridLatency|Latencia de la cuadrícula de eventos (versión preliminar)|Milisegundos|Promedio|La latencia media (en milisegundos) entre la entrada de eventos a IoT Hub y entrada de eventos en Event Grid. Este número es un promedio entre todos los tipos de evento. Usar la dimensión EventType para ver la latencia de un tipo específico de evento.|Tipo de evento|
|d2c.twin.read.success|Lecturas gemelas correctas de los dispositivos|Count|Total|El recuento de todas las lecturas gemelas correctas iniciadas por el dispositivo.|Sin dimensiones|
|d2c.twin.read.failure|Lecturas gemelas con error de los dispositivos|Count|Total|El recuento de todas las lecturas gemelas con error iniciadas por el dispositivo.|Sin dimensiones|
|d2c.twin.read.size|Tamaño de la respuesta de las lecturas gemelas de dispositivos|Bytes|Promedio|El valor medio, mínimo y máximo de todas las lecturas gemelas correctas iniciadas por el dispositivo.|Sin dimensiones|
|d2c.twin.update.success|Actualizaciones gemelas correctas de los dispositivos|Count|Total|El recuento de todas las actualizaciones gemelas correctas iniciadas por el dispositivo.|Sin dimensiones|
|d2c.twin.update.failure|Actualizaciones gemelas con error de los dispositivos|Count|Total|El recuento de todas las actualizaciones gemelas con error iniciadas por el dispositivo.|Sin dimensiones|
|d2c.twin.update.size|Tamaño de las actualizaciones gemelas de los dispositivos|Bytes|Promedio|El tamaño medio, mínimo y máximo de todas las actualizaciones gemelas correctas iniciadas por el dispositivo.|Sin dimensiones|
|c2d.methods.success|Invocaciones correctas al método directo|Count|Total|El número de todas las llamadas correctas al método directo.|Sin dimensiones|
|c2d.methods.failure|Invocaciones al método directo con error|Count|Total|El número de todas las llamadas al método directo con error.|Sin dimensiones|
|c2d.methods.requestSize|Tamaño de la solicitud de las invocaciones a métodos directos|Bytes|Promedio|El valor medio, mínimo y máximo de todas las solicitudes correctas de método directo.|Sin dimensiones|
|c2d.methods.responseSize|Tamaño de la respuesta de las invocaciones a métodos directos|Bytes|Promedio|El valor medio, mínimo y máximo de todas las respuestas correctas de método directo.|Sin dimensiones|
|c2d.twin.read.success|Lecturas gemelas correctas del back-end|Count|Total|El recuento de todas las lecturas gemelas correctas iniciadas por el back-end.|Sin dimensiones|
|c2d.twin.read.failure|Lecturas gemelas con error del back-end|Count|Total|El recuento de todas las lecturas gemelas con error iniciadas por el back-end.|Sin dimensiones|
|c2d.twin.read.size|Tamaño de la respuesta de las lecturas gemelas del back-end|Bytes|Promedio|El valor medio, mínimo y máximo de todas las lecturas gemelas correctas iniciadas por el back-end.|Sin dimensiones|
|c2d.twin.update.success|Actualizaciones gemelas correctas del back-end|Count|Total|El recuento de todas las actualizaciones gemelas correctas iniciadas por el back-end.|Sin dimensiones|
|c2d.twin.update.failure|Actualizaciones gemelas con error del back-end|Count|Total|El recuento de todas las actualizaciones gemelas con error iniciadas por el back-end.|Sin dimensiones|
|c2d.twin.update.size|Tamaño de las actualizaciones gemelas del back-end|Bytes|Promedio|El tamaño medio, mínimo y máximo de todas las actualizaciones gemelas correctas iniciadas por el back-end.|Sin dimensiones|
|twinQueries.success|Consultas gemelas correctas|Count|Total|El recuento de todas las consultas gemelas correctas.|Sin dimensiones|
|twinQueries.failure|Consultas gemelas con error|Count|Total|El recuento de todas las consultas gemelas con error.|Sin dimensiones|
|twinQueries.resultSize|Tamaño de resultado de consultas gemelas|Bytes|Promedio|El valor medio, mínimo y máximo del tamaño del resultado de todas las consultas gemelas correctas.|Sin dimensiones|
|jobs.createTwinUpdateJob.success|Creaciones correctas de trabajos de actualización gemelos|Count|Total|El recuento de todas las creaciones correctas de trabajos de actualización gemelos.|Sin dimensiones|
|jobs.createTwinUpdateJob.failure|Creaciones con error de trabajos de actualización gemelos|Count|Total|El recuento de todas las creaciones con error de trabajos de actualización gemelos.|Sin dimensiones|
|jobs.createDirectMethodJob.success|Creaciones correctas de trabajos de invocación de método|Count|Total|El recuento de todas las creaciones correctas de los trabajos de invocación de método directos.|Sin dimensiones|
|jobs.createDirectMethodJob.failure|Creaciones con error de trabajos de invocación de método|Count|Total|El recuento de todas las creaciones con error de los trabajos de invocación de método directos.|Sin dimensiones|
|jobs.listJobs.success|Llamadas correctas para enumerar trabajos|Count|Total|El recuento de todas las llamadas correctas para enumerar trabajos.|Sin dimensiones|
|jobs.listJobs.failure|Llamadas con error para enumerar trabajos|Count|Total|El recuento de todas las llamadas con error para enumerar trabajos.|Sin dimensiones|
|jobs.cancelJob.success|Cancelaciones de trabajos correctas|Count|Total|El recuento de todas las llamadas correctas para cancelar un trabajo.|Sin dimensiones|
|jobs.cancelJob.failure|Cancelaciones de trabajos con error|Count|Total|El recuento de todas las llamadas con error para cancelar un trabajo.|Sin dimensiones|
|jobs.queryJobs.success|Consultas de trabajo correctas|Count|Total|El recuento de todas las llamadas correctas para consultar trabajos.|Sin dimensiones|
|jobs.queryJobs.failure|Consultas de trabajo con error|Count|Total|El recuento de todas las llamadas con error para consultar trabajos.|Sin dimensiones|
|jobs.completed|Trabajos completados|Count|Total|El recuento de todos los trabajos completados.|Sin dimensiones|
|jobs.failed|Trabajos con error|Count|Total|El recuento de todos los trabajos con error.|Sin dimensiones|
|d2c.telemetry.ingress.sendThrottle|Número de errores de limitación|Count|Total|Número de errores de limitación debido a las limitaciones de rendimiento del dispositivo|Sin dimensiones|
|dailyMessageQuotaUsed|Número total de mensajes usados|Count|Promedio|Número de mensajes totales usados hoy. Se trata de un valor acumulado que se restablece en cero a las 00:00 UTC cada día.|Sin dimensiones|
|deviceDataUsage|Uso de datos total del dispositivo|Bytes|Total|Bytes transferidos hacia y desde cualquier dispositivo conectado al Centro de IOT|Sin dimensiones|
|totalDeviceCount|Total de dispositivos (versión preliminar)|Count|Promedio|Número de dispositivos registrados en IoT Hub|Sin dimensiones|
|connectedDeviceCount|Dispositivos conectados (versión preliminar)|Count|Promedio|Número de dispositivos conectados a IoT Hub|Sin dimensiones|
|configuraciones|Métricas de configuración|Count|Total|Métricas de las operaciones de configuración|Sin dimensiones|

## <a name="microsoftdevicesprovisioningservices"></a>Microsoft.Devices/provisioningServices

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|RegistrationAttempts|Intentos de registro|Count|Total|Número de intentos de registro de dispositivos|ProvisioningServiceName, IotHubName, Status|
|DeviceAssignments|Dispositivos asignados|Count|Total|Número de dispositivos asignados a un IoT Hub|ProvisioningServiceName, IotHubName|
|AttestationAttempts|Intentos de atestación|Count|Total|Número de intentos de atestación de dispositivos|ProvisioningServiceName, Status, Protocol|

## <a name="microsoftdocumentdbdatabaseaccounts"></a>Microsoft.DocumentDB/databaseAccounts

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|AvailableStorage|Almacenamiento disponible|Bytes|Total|Almacenamiento total disponible, notificado una granularidad de 5 minutos|CollectionName, DatabaseName, región|
|CassandraConnectionClosures|Cierres de conexión de Cassandra|Count|Total|Número de conexiones de Cassandra que se han cerrado, notificado en una granularidad de 1 minuto|Región, ClosureReason|
|CassandraRequestCharges|Cargos de solicitud de Cassandra|Count|Total|Unidades de solicitud consumidas para las solicitudes de Cassandra realizadas|DatabaseName, CollectionName, Region, OperationType, ResourceType|
|CassandraRequests|Solicitudes de Cassandra|Count|Count|Número de solicitudes de Cassandra realizadas|DatabaseName, CollectionName, región, OperationType, ResourceType, código de error|
|DataUsage|Uso de datos|Bytes|Total|Uso de datos total notificado una granularidad de 5 minutos|CollectionName, DatabaseName, región|
|DocumentCount|Recuento de documentos|Count|Total|Número total de documentos que se notifica una granularidad de 5 minutos|CollectionName, DatabaseName, región|
|DocumentQuota|Cuota de documentos|Bytes|Total|Cuota de almacenamiento total notificado una granularidad de 5 minutos|CollectionName, DatabaseName, región|
|IndexUsage|Uso de índice|Bytes|Total|Uso de índice total notificado una granularidad de 5 minutos|CollectionName, DatabaseName, región|
|MetadataRequests|Solicitudes de metadatos|Count|Count|Recuento de las solicitudes de metadatos. Cosmos DB mantiene la colección de metadatos del sistema para cada cuenta, lo que le permite enumerar las colecciones, las bases de datos y sus configuraciones de forma gratuita.|DatabaseName, CollectionName, región, código de estado, |
|MongoRequestCharge|Cargo de la solicitud de Mongo|Count|Total|Unidades de la solicitud de Mongo consumidas|DatabaseName, CollectionName, Region, CommandName, ErrorCode|
|MongoRequests|Solicitudes de Mongo|Count|Count|Número de solicitudes de Mongo realizadas|DatabaseName, CollectionName, Region, CommandName, ErrorCode|
|ProvisionedThroughput|Rendimiento aprovisionado|Count|Máximo|Rendimiento aprovisionado|DatabaseName, CollectionName|
|ReplicationLatency|P99 Latencia de replicación|MilliSeconds|Promedio|Latencia de replicación P99 entre regiones de origen y destino para la cuenta habilitada geográficamente|SourceRegion, TargetRegion|
|ServiceAvailability|Disponibilidad del servicio|Porcentaje|Promedio|Disponibilidad de las solicitudes de cuenta una granularidad de una hora, día o mes|Sin dimensiones|
|TotalRequestUnits|Unidades de solicitud totales|Count|Total|Unidades de solicitud consumidas|DatabaseName, CollectionName, región, StatusCode, OperationType|
|TotalRequests|Total de solicitudes|Count|Count|Número de solicitudes realizadas|DatabaseName, CollectionName, región, StatusCode, OperationType|

## <a name="microsofteventgridtopics"></a>Microsoft.EventGrid/topics

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|PublishSuccessCount|Eventos publicados|Count|Total|Número total de eventos publicados en este tema|Sin dimensiones|
|PublishFailCount|Publicar los eventos con error|Count|Total|Número total de eventos que no se han podido publicar en este tema|ErrorType, Error|
|UnmatchedEventCount|Eventos no coincidentes|Count|Total|Número total de eventos que no coinciden con ninguna de las suscripciones de eventos de este tema|Sin dimensiones|
|PublishSuccessLatencyInMs|Publicar latencia correcta|Count|Total|Publicar latencia correcta en milisegundos|Sin dimensiones|

## <a name="microsofteventgrideventsubscriptions"></a>Microsoft.EventGrid/eventSubscriptions

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|MatchedEventCount|Eventos coincidentes|Count|Total|Número total de eventos que coinciden con esta suscripción de eventos|Sin dimensiones|
|DeliveryAttemptFailCount|Eventos de error en la entrega|Count|Total|Número total de eventos que no se han podido entregar a esta suscripción de eventos|Error, ErrorType|
|DeliverySuccessCount|Eventos entregados|Count|Total|Número total de eventos que se han entregado a esta suscripción de eventos|Sin dimensiones|
|DestinationProcessingDurationInMs|Duración del procesamiento de destino|Milisegundos|Promedio|Duración del procesamiento de destino en milisegundos|Sin dimensiones|
|DroppedEventCount|Eventos quitados|Count|Total|Número total de eventos quitados que coinciden con esta suscripción de eventos|DropReason|
|DeadLetteredCount|Eventos en la cola de mensajes fallidos|Count|Total|Número total de eventos en la cola de eventos fallidos que coinciden con esta suscripción de eventos|DeadLetterReason|

## <a name="microsofteventgridextensiontopics"></a>Microsoft.EventGrid/extensionTopics

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|PublishSuccessCount|Eventos publicados|Count|Total|Número total de eventos publicados en este tema|Sin dimensiones|
|PublishFailCount|Eventos erróneos|Count|Total|Número total de eventos que no se han podido publicar en este tema|ErrorType, Error|
|UnmatchedEventCount|Eventos no coincidentes|Count|Total|Número total de eventos que no coinciden con ninguna de las suscripciones de eventos de este tema|Sin dimensiones|
|PublishSuccessLatencyInMs|Publicar latencia correcta|Count|Total|Publicar latencia correcta en milisegundos|Sin dimensiones|

## <a name="microsofteventhubnamespaces"></a>Microsoft.EventHub/namespaces

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|SuccessfulRequests|Solicitudes correctas|Count|Total|Solicitudes correctas para Microsoft.EventHub.|EntityName, |
|ServerErrors|Errores del servidor.|Count|Total|Errores del servidor para Microsoft.EventHub.|EntityName, |
|UserErrors|Errores de usuario.|Count|Total|Errores de usuario para Microsoft.EventHub.|EntityName, |
|QuotaExceededErrors|Cuota de errores superada.|Count|Total|Cuota de errores superada para Microsoft.EventHub.|EntityName, |
|ThrottledRequests|Solicitudes limitadas.|Count|Total|Solicitudes limitadas para Microsoft.EventHub.|EntityName, |
|IncomingRequests|Solicitudes entrantes|Count|Total|Solicitudes entrantes para Microsoft.EventHub.|EntityName|
|IncomingMessages|Mensajes entrantes|Count|Total|Mensajes entrantes para Microsoft.EventHub.|EntityName|
|OutgoingMessages|Mensajes salientes|Count|Total|Mensajes salientes para Microsoft.EventHub.|EntityName|
|IncomingBytes|Bytes entrantes|Bytes|Total|Bytes entrantes para Microsoft.EventHub.|EntityName|
|OutgoingBytes|Bytes salientes|Bytes|Total|Bytes salientes para Microsoft.EventHub.|EntityName|
|ActiveConnections|ActiveConnections|Count|Promedio|Total de conexiones activas para Microsoft.EventHub.|Sin dimensiones|
|ConnectionsOpened|Conexiones abiertas.|Count|Promedio|Conexiones abiertas para Microsoft.EventHub.|EntityName|
|ConnectionsClosed|Conexiones cerradas.|Count|Promedio|Conexiones cerradas para Microsoft.EventHub.|EntityName|
|CaptureBacklog|Permite capturar el trabajo pendiente.|Count|Total|Permite capturar el trabajo pendiente para Microsoft.EventHub.|EntityName|
|CapturedMessages|Mensajes capturados.|Count|Total|Mensajes capturados para Microsoft.EventHub.|EntityName|
|CapturedBytes|Bytes capturados.|Bytes|Total|Bytes capturados para Microsoft.EventHub.|EntityName|
|Tamaño|Tamaño|Bytes|Promedio|Tamaño de un centro de eventos en bytes.|EntityName|
|INREQS|Solicitudes entrantes (en desuso)|Count|Total|Las solicitudes de un espacio de nombres (en desuso) de envío entrantes totales|Sin dimensiones|
|SUCCREQ|Solicitudes correctas (en desuso)|Count|Total|Solicitudes correctas totales de un espacio de nombres (en desuso)|Sin dimensiones|
|FAILREQ|Solicitudes con error (en desuso)|Count|Total|Número total de solicitudes con error para un espacio de nombres (en desuso)|Sin dimensiones|
|SVRBSY|Errores de servidor ocupado (en desuso)|Count|Total|Errores de servidor total ocupado para un espacio de nombres (en desuso)|Sin dimensiones|
|INTERR|Errores internos del servidor (en desuso)|Count|Total|Errores de servidor internos totales para un espacio de nombres (en desuso)|Sin dimensiones|
|MISCERR|Otros errores (en desuso)|Count|Total|Número total de solicitudes con error para un espacio de nombres (en desuso)|Sin dimensiones|
|INMSGS|Mensajes entrantes (en desuso) (desusados)|Count|Total|Total de mensajes entrantes para un espacio de nombres. Esta métrica está en desuso. En su lugar, use la métrica de mensajes entrantes (en desuso)|Sin dimensiones|
|EHINMSGS|Mensajes entrantes (en desuso)|Count|Total|Total de mensajes entrantes para un espacio de nombres (en desuso)|Sin dimensiones|
|OUTMSGS|Mensajes salientes (en desuso) (desusados)|Count|Total|Total de mensajes salientes para un espacio de nombres. Esta métrica está en desuso. En su lugar, use la métrica de los mensajes salientes (en desuso)|Sin dimensiones|
|EHOUTMSGS|Mensajes salientes (en desuso)|Count|Total|Mensajes salientes totales de un espacio de nombres (en desuso)|Sin dimensiones|
|EHINMBS|Bytes entrantes (en desuso) (en desuso)|Bytes|Total|Rendimiento de mensajes entrantes del centro de eventos para un espacio de nombres. Esta métrica está en desuso. En su lugar, use la métrica de bytes de entrada (en desuso)|Sin dimensiones|
|EHINBYTES|Bytes entrantes (en desuso)|Bytes|Total|Rendimiento Event Hub de mensajes entrantes para un espacio de nombres (en desuso)|Sin dimensiones|
|EHOUTMBS|Bytes salientes (en desuso) (en desuso)|Bytes|Total|Rendimiento de mensajes salientes del centro de eventos para un espacio de nombres. Esta métrica está en desuso. En su lugar use métrica de bytes salientes (en desuso)|Sin dimensiones|
|EHOUTBYTES|Bytes salientes (en desuso)|Bytes|Total|Rendimiento Event Hub de mensajes salientes para un espacio de nombres (en desuso)|Sin dimensiones|
|EHABL|Trabajo acumulado de archivar mensajes (en desuso)|Count|Total|Mensajes de archivo de centro de eventos en el trabajo pendiente para un espacio de nombres (en desuso)|Sin dimensiones|
|EHAMSGS|Archivar mensajes (en desuso)|Count|Total|Centro de eventos archivó los mensajes en un espacio de nombres (en desuso)|Sin dimensiones|
|EHAMBS|Archivar rendimiento de mensajes (en desuso)|Bytes|Total|Centro de eventos archivó el procesamiento de mensajes en un espacio de nombres (en desuso)|Sin dimensiones|

## <a name="microsofteventhubclusters"></a>Microsoft.EventHub/clusters

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|SuccessfulRequests|Solicitudes correctas (versión preliminar)|Count|Total|Solicitudes correctas para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|ServerErrors|Errores del servidor. (Versión preliminar)|Count|Total|Errores del servidor para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|UserErrors|Errores de usuario. (Versión preliminar)|Count|Total|Errores de usuario para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|QuotaExceededErrors|Cuota de errores superada. (Versión preliminar)|Count|Total|Cuota de errores superada para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|ThrottledRequests|Solicitudes limitadas. (Versión preliminar)|Count|Total|Solicitudes limitadas para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|IncomingRequests|Solicitudes entrantes (versión preliminar)|Count|Total|Solicitudes entrantes para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|IncomingMessages|Mensajes entrantes (versión preliminar)|Count|Total|Mensajes entrantes para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|OutgoingMessages|Mensajes salientes (versión preliminar)|Count|Total|Mensajes salientes para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|IncomingBytes|Bytes entrantes (Versión preliminar)|Bytes|Total|Bytes entrantes para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|OutgoingBytes|Bytes salientes (Versión preliminar)|Bytes|Total|Bytes salientes para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|ActiveConnections|ActiveConnections (versión preliminar)|Count|Promedio|Total de conexiones activas para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|ConnectionsOpened|Conexiones abiertas. (Versión preliminar)|Count|Promedio|Conexiones abiertas para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|ConnectionsClosed|Conexiones cerradas. (Versión preliminar)|Count|Promedio|Conexiones cerradas para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|CaptureBacklog|Permite capturar el trabajo pendiente. (Versión preliminar)|Count|Total|Permite capturar el trabajo pendiente para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|CapturedMessages|Mensajes capturados. (Versión preliminar)|Count|Total|Mensajes capturados para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|CapturedBytes|Bytes capturados. (Versión preliminar)|Bytes|Total|Bytes capturados para Microsoft.EventHub. (Versión preliminar)|Sin dimensiones|
|CPU|CPU (versión preliminar)|Porcentaje|Máximo|Uso de la CPU para el clúster del centro de eventos como porcentaje|Rol|
|AvailableMemory|Memoria disponible (versión preliminar)|Count|Máximo|Memoria disponible para el clúster del centro de eventos en bytes|Rol|

## <a name="microsofthdinsightclusters"></a>Microsoft.HDInsight/clusters

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|GatewayRequests|Solicitudes de puerta de enlace|Count|Total|Número de solicitudes de puerta de enlace|ClusterDnsName, HttpStatus|
|CategorizedGatewayRequests|Solicitudes de puerta de enlace categorizadas|Count|Total|Número de solicitudes de puerta de enlace por categorías (1xx/2xx/3xx/4xx/5xx)|ClusterDnsName, HttpStatus|
|Escalado automático|Métricas de escalado automático|Count|Máximo|Métricas de escalado automático|ClusterDnsName, MetricName|
|AllocatedMB|MB asignados|Count|Máximo|MB asignados|ClusterDnsName, MetricName|
|AvailableMB|MB disponibles|Count|Máximo|MB disponibles|ClusterDnsName, MetricName|
|AppsPending|Aplicaciones pendientes|Count|Máximo|Aplicaciones pendientes|ClusterDnsName, MetricName|
|AppsRunning|Aplicaciones que se ejecutan|Count|Máximo|Aplicaciones que se ejecutan|ClusterDnsName, MetricName|
|AppsSubmitted|Aplicaciones enviadas|Count|Máximo|Aplicaciones enviadas|ClusterDnsName, MetricName|
|NumActiveWorkers|Número de trabajadores activos|Count|Máximo|Número de trabajadores activos|ClusterDnsName, MetricName|

## <a name="microsoftinsightsautoscalesettings"></a>Microsoft.Insights/AutoscaleSettings

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|ObservedMetricValue|Valor de métrica observado|Count|Promedio|Valor que calculó la escalabilidad automática cuando se ejecutó.|MetricTriggerSource|
|MetricThreshold|Umbral de métrica|Count|Promedio|Umbral de escalabilidad automática que se configuró al ejecutar la escalabilidad automática.|MetricTriggerRule|
|ObservedCapacity|Capacidad observada|Count|Promedio|Capacidad notificada para la escalabilidad automática cuando se ejecutó.|Sin dimensiones|
|ScaleActionsInitiated|Acciones de escalado iniciadas|Count|Total|Dirección de la operación de escalado.|ScaleDirection|

## <a name="microsoftinsightscomponents"></a>Microsoft.Insights/Components

(Vista previa pública)

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|availabilityResults/availabilityPercentage|Disponibilidad|Porcentaje|Promedio|Porcentaje de pruebas de disponibilidad se completó correctamente|availabilityResult/name, availabilityResult/ubicación|
|availabilityResults/count|Pruebas de disponibilidad|Count|Count|Recuento de pruebas de disponibilidad|availabilityResult/name, availabilityResult/location, availabilityResult/success|
|availabilityResults/duration|Duración de la prueba de disponibilidad|MilliSeconds|Promedio|Duración de la prueba de disponibilidad|availabilityResult/name, availabilityResult/location, availabilityResult/success|
|browserTimings/networkDuration|Tiempo de conexión de red para carga de páginas|MilliSeconds|Promedio|Tiempo transcurrido entre la solicitud del usuario y la conexión de red. Incluye la búsqueda de DNS y la conexión de transporte.|Sin dimensiones|
|browserTimings/processingDuration|Tiempo de procesamiento del cliente|MilliSeconds|Promedio|Tiempo transcurrido entre la recepción del último byte de un documento hasta que se carga el DOM. Todavía se pueden procesar solicitudes asincrónicas.|Sin dimensiones|
|browserTimings/receiveDuration|Tiempo de recepción de respuesta|MilliSeconds|Promedio|Tiempo transcurrido entre el primer y el último byte, o hasta la desconexión.|Sin dimensiones|
|browserTimings/sendDuration|Tiempo de envío de solicitud|MilliSeconds|Promedio|Tiempo transcurrido entre la conexión de red y la recepción del primer byte.|Sin dimensiones|
|browserTimings/totalDuration|Tiempo de carga de página del explorador|MilliSeconds|Promedio|Tiempo transcurrido desde la solicitud del usuario hasta la carga del DOM, las hojas de estilos, los scripts y las imágenes.|Sin dimensiones|
|dependencies/count|Llamadas de dependencia|Count|Count|Número de llamadas realizadas por la aplicación a recursos externos.|dependency/type, dependency/performanceBucket, dependency/success, operation/synthetic, cloud/roleInstance, cloud/roleName|
|dependencies/duration|Duración de la dependencia|MilliSeconds|Promedio|Duración de las llamadas realizadas por la aplicación a recursos externos.|dependency/type, dependency/performanceBucket, dependency/success, operation/synthetic, cloud/roleInstance, cloud/roleName|
|dependencies/failed|Errores de llamada de dependencia|Count|Count|Número de llamadas de dependencia erróneas realizadas por la aplicación a recursos externos.|dependency/type, dependency/performanceBucket, operation/synthetic, cloud/roleInstance, cloud/roleName|
|pageViews/count|Vistas de página|Count|Count|Número de visitas a la página.|operation/synthetic|
|pageViews/duration|Tiempo de carga de la vista de página|MilliSeconds|Promedio|Tiempo de carga de la vista de página|operation/synthetic|
|performanceCounters/requestExecutionTime|Tiempo de ejecución de solicitud HTTP|MilliSeconds|Promedio|Tiempo de ejecución de la solicitud más reciente.|cloud/roleInstance|
|performanceCounters/requestsInQueue|Solicitudes HTTP en la cola de la aplicación|Count|Promedio|Longitud de la cola de solicitud de la aplicación.|cloud/roleInstance|
|performanceCounters/requestsPerSecond|Tasa de solicitudes HTTP|CountPerSecond|Promedio|Tasa de todas las solicitudes de la aplicación por segundo desde ASP.NET.|cloud/roleInstance|
|performanceCounters/exceptionsPerSecond|Tasa de excepciones|CountPerSecond|Promedio|Recuento de excepciones controladas y no controladas de las cuales se informó a Windows, incluidas las excepciones de .NET y las excepciones no administradas que se convierten en excepciones de .NET.|cloud/roleInstance|
|performanceCounters/processIOBytesPerSecond|Tasa de E/S del proceso|BytesPerSecond|Promedio|Bytes totales por segundo leídos y escritos en archivos, red o dispositivos.|cloud/roleInstance|
|performanceCounters/processCpuPercentage|CPU del proceso|Porcentaje|Promedio|Porcentaje de tiempo transcurrido que todos los subprocesos del proceso han usado el procesador para ejecutar instrucciones. Puede variar entre 0 y 100. Esta métrica indica el rendimiento solo del proceso w3wp.|cloud/roleInstance|
|performanceCounters/processorCpuPercentage|Tiempo del procesador|Porcentaje|Promedio|Porcentaje de tiempo que el procesador dedica a subprocesos no inactivos.|cloud/roleInstance|
|performanceCounters/memoryAvailableBytes|Memoria disponible|Bytes|Promedio|Memoria física disponible de inmediato para su asignación a un proceso o para uso del sistema.|cloud/roleInstance|
|performanceCounters/processPrivateBytes|Bytes privados del proceso|Bytes|Promedio|Memoria asignada exclusivamente a los procesos supervisados de la aplicación.|cloud/roleInstance|
|requests/duration|Tiempo de respuesta del servidor|MilliSeconds|Promedio|Tiempo transcurrido entre la recepción de la solicitud HTTP y la finalización del envío de la respuesta.|request/performanceBucket, request/resultCode, operation/synthetic, cloud/roleInstance, request/success, cloud/roleName|
|requests/count|Solicitudes de servidor|Count|Count|Número de solicitudes HTTP completadas.|request/performanceBucket, request/resultCode, operation/synthetic, cloud/roleInstance, request/success, cloud/roleName|
|requests/failed|Solicitudes con errores|Count|Count|Número de solicitudes HTTP marcadas como erróneas. En la mayoría de los casos, se trata de solicitudes con un código de respuesta > = 400 y no igual a 401.|request/performanceBucket, request/resultCode, operation/synthetic, cloud/roleInstance, cloud/roleName|
|tasa de solicitudes /|Tasa de solicitudes de servidor|CountPerSecond|Promedio|Tasa de solicitudes del servidor por segundo|request/performanceBucket, request/resultCode, operation/synthetic, cloud/roleInstance, request/success, cloud/roleName|
|exceptions/count|Excepciones|Count|Count|Recuento combinado de todas las excepciones no detectadas.|cloud/roleName, cloud/roleInstance, client/type|
|exceptions/browser|Excepciones del explorador|Count|Count|Número de excepciones no detectadas producidas en el explorador.|Sin dimensiones|
|exceptions/server|Excepciones de servidor|Count|Count|Número de excepciones no detectadas producidas en la aplicación de servidor.|cloud/roleName, cloud/roleInstance|
|traces/count|Seguimientos|Count|Count|Número de documentos de seguimiento|trace/severityLevel, operation/synthetic, cloud/roleName, cloud/roleInstance|

## <a name="microsoftkeyvaultvaults"></a>Microsoft.KeyVault/vaults

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|ServiceApiHit|Número total de visitas de la API de servicio|Count|Count|Corresponde al número total de visitas de la API de servicio.|ActivityType, ActivityName|
|ServiceApiLatency|Latencia general de la API de servicio|Milisegundos|Promedio|Corresponde a la latencia general de las solicitudes de la API de servicio.|ActivityType, ActivityName, StatusCode|
|ServiceApiResult|Número total de resultados de la API de servicio|Count|Count|Corresponde al número total de resultados de la API de servicio.|ActivityType, ActivityName, StatusCode|

## <a name="microsoftkustoclusters"></a>Microsoft.Kusto/Clusters

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|ClusterDataCapacityFactor|Uso de la caché|Porcentaje|Promedio|Nivel de uso en el ámbito de clúster|Sin dimensiones|
|QueryDuration|Duración de la consulta|Milisegundos|Promedio|Duración de las consultas en segundos|QueryStatus|
|IngestionsLoadFactor|Uso de la ingesta|Porcentaje|Promedio|Proporción de ranuras de ingesta usadas en el clúster|Sin dimensiones|
|IsEngineAnsweringQuery|Mantener conexión|Count|Promedio|La comprobación de integridad indica las respuestas del clúster a las consultas|Sin dimensiones|
|IngestCommandOriginalSizeInMb|Volumen de ingesta (en MB)|Count|Total|Volumen global de la ingesta de datos del clúster (en MB)|Sin dimensiones|
|IngestedEventAgeSeconds|Latencia de la ingesta (en segundos)|Segundos|Promedio|Tiempo de ingesta del origen (p. ej., el mensaje está en el centro de eventos) al clúster en segundos|Sin dimensiones|
|EventRecievedFromEventHub|Eventos procesados (para Event Hubs)|Count|Total|Número de eventos que ha procesado el clúster en la ingesta desde el centro de eventos|Sin dimensiones|
|IngestionResult|Resultado de la ingesta|Count|Count|Número de operaciones de ingesta|IngestionResultDetails|
|EngineCPU|CPU|Porcentaje|Promedio|Nivel de uso de CPU|Sin dimensiones|

## <a name="microsoftlocationbasedservicesaccounts"></a>Microsoft.LocationBasedServices/accounts

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|Uso|Uso|Count|Count|Recuento de llamadas a la API|ApiCategory, ApiName|

## <a name="microsoftlogicworkflows"></a>Microsoft.Logic/workflows

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|RunsStarted|Ejecuciones iniciadas|Count|Total|Número de ejecuciones de flujo de trabajo que se han iniciado.|Sin dimensiones|
|RunsCompleted|Ejecuciones completadas|Count|Total|Número de ejecuciones de flujo de trabajo que se han completado.|Sin dimensiones|
|RunsSucceeded|Ejecuciones correctas|Count|Total|Número de ejecuciones de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|RunsFailed|Ejecuciones con errores|Count|Total|Número de ejecuciones de flujo de trabajo en las que se ha producido un error.|Sin dimensiones|
|RunsCancelled|Ejecuciones canceladas|Count|Total|Número de flujo de trabajo se ejecuta cancelado.|Sin dimensiones|
|RunLatency|Latencia de ejecución|Segundos|Promedio|Latencia de las ejecuciones de flujo de trabajo que se han completado.|Sin dimensiones|
|RunSuccessLatency|Latencia de ejecuciones correctas|Segundos|Promedio|Latencia de las ejecuciones de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|RunThrottledEvents|Ejecución de eventos limitados|Count|Total|Número de eventos limitados del desencadenador o de la acción de flujo de trabajo.|Sin dimensiones|
|RunFailurePercentage|Porcentaje de errores de ejecución|Porcentaje|Total|Porcentaje de ejecuciones del flujo de trabajo erróneas.|Sin dimensiones|
|ActionsStarted|Acciones iniciadas |Count|Total|Número de acciones de flujo de trabajo que se han iniciado.|Sin dimensiones|
|ActionsCompleted|Acciones completadas |Count|Total|Número de acciones de flujo de trabajo que se han completado.|Sin dimensiones|
|ActionsSucceeded|Acciones correctas |Count|Total|Número de acciones de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|ActionsFailed|Acciones con errores|Count|Total|Número de acciones de flujo de trabajo en las que se ha producido un error.|Sin dimensiones|
|ActionsSkipped|Acciones omitidas |Count|Total|Número de acciones de flujo de trabajo que se han omitido.|Sin dimensiones|
|ActionLatency|Latencia de acciones |Segundos|Promedio|Latencia de las acciones de flujo de trabajo que se han completado.|Sin dimensiones|
|ActionSuccessLatency|Latencia de acciones correctas |Segundos|Promedio|Latencia de las acciones de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|ActionThrottledEvents|Eventos limitados de acciones|Count|Total|Número de eventos limitados de la acción de flujo de trabajo.|Sin dimensiones|
|TriggersStarted|Desencadenadores iniciados |Count|Total|Número de desencadenadores de flujo de trabajo que se han iniciado.|Sin dimensiones|
|TriggersCompleted|Desencadenadores completados |Count|Total|Número de desencadenadores de flujo de trabajo que se han completado.|Sin dimensiones|
|TriggersSucceeded|Desencadenadores correctos |Count|Total|Número de desencadenadores de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|TriggersFailed|Desencadenadores con errores |Count|Total|Número de desencadenadores de flujo de trabajo en los que se ha producido un error.|Sin dimensiones|
|TriggersSkipped|Desencadenadores omitidos|Count|Total|Número de desencadenadores de flujo de trabajo que se han omitido.|Sin dimensiones|
|TriggersFired|Desencadenadores activados |Count|Total|Número de desencadenadores de flujo de trabajo que se han activado.|Sin dimensiones|
|TriggerLatency|Latencia de desencadenador |Segundos|Promedio|Latencia de los desencadenadores de flujo de trabajo que se han completado.|Sin dimensiones|
|TriggerFireLatency|Latencia de desencadenador activado |Segundos|Promedio|Latencia de los desencadenadores de flujo de trabajo que se han activado.|Sin dimensiones|
|TriggerSuccessLatency|Latencia de desencadenadores correctos |Segundos|Promedio|Latencia de los desencadenadores de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|TriggerThrottledEvents|Eventos limitados de desencadenadores|Count|Total|Número de eventos limitados del desencadenador de flujo de trabajo.|Sin dimensiones|
|BillableActionExecutions|Ejecuciones de acciones facturables|Count|Total|Número de ejecuciones de acciones del flujo de trabajo que se están facturando.|Sin dimensiones|
|BillableTriggerExecutions|Ejecuciones del desencadenador facturable|Count|Total|Número de ejecuciones del desencadenador del flujo de trabajo que se están facturando.|Sin dimensiones|
|TotalBillableExecutions|Total de ejecuciones facturables|Count|Total|Número de ejecuciones del flujo de trabajo que se están facturando.|Sin dimensiones|
|BillingUsageNativeOperation|Uso de facturación para las ejecuciones de la operación nativa|Count|Total|Número de ejecuciones de la operación nativa que se facturan.|Sin dimensiones|
|BillingUsageStandardConnector|Uso de facturación para las ejecuciones del conector estándar|Count|Total|Número de ejecuciones del conector estándar que se facturan.|Sin dimensiones|
|BillingUsageStorageConsumption|Uso de facturación para las ejecuciones de consumo de almacenamiento|Count|Total|Número de ejecuciones del consumo de almacenamiento que se facturan.|Sin dimensiones|
|BillingUsageNativeOperation|Uso de facturación para las ejecuciones de la operación nativa|Count|Total|Número de ejecuciones de la operación nativa que se facturan.|Sin dimensiones|
|BillingUsageStandardConnector|Uso de facturación para las ejecuciones del conector estándar|Count|Total|Número de ejecuciones del conector estándar que se facturan.|Sin dimensiones|
|BillingUsageStorageConsumption|Uso de facturación para las ejecuciones de consumo de almacenamiento|Count|Total|Número de ejecuciones del consumo de almacenamiento que se facturan.|Sin dimensiones|

## <a name="microsoftlogicintegrationserviceenvironments"></a>Microsoft.Logic/integrationServiceEnvironments

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|RunsStarted|Ejecuciones iniciadas|Count|Total|Número de ejecuciones de flujo de trabajo que se han iniciado.|Sin dimensiones|
|RunsCompleted|Ejecuciones completadas|Count|Total|Número de ejecuciones de flujo de trabajo que se han completado.|Sin dimensiones|
|RunsSucceeded|Ejecuciones correctas|Count|Total|Número de ejecuciones de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|RunsFailed|Ejecuciones con errores|Count|Total|Número de ejecuciones de flujo de trabajo en las que se ha producido un error.|Sin dimensiones|
|RunsCancelled|Ejecuciones canceladas|Count|Total|Número de flujo de trabajo se ejecuta cancelado.|Sin dimensiones|
|RunLatency|Latencia de ejecución|Segundos|Promedio|Latencia de las ejecuciones de flujo de trabajo que se han completado.|Sin dimensiones|
|RunSuccessLatency|Latencia de ejecuciones correctas|Segundos|Promedio|Latencia de las ejecuciones de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|RunThrottledEvents|Ejecución de eventos limitados|Count|Total|Número de eventos limitados del desencadenador o de la acción de flujo de trabajo.|Sin dimensiones|
|RunStartThrottledEvents|Eventos limitados del inicio de ejecución|Count|Total|Número de eventos limitados del inicio de ejecución del flujo de trabajo.|Sin dimensiones|
|RunFailurePercentage|Porcentaje de errores de ejecución|Porcentaje|Total|Porcentaje de ejecuciones del flujo de trabajo erróneas.|Sin dimensiones|
|ActionsStarted|Acciones iniciadas |Count|Total|Número de acciones de flujo de trabajo que se han iniciado.|Sin dimensiones|
|ActionsCompleted|Acciones completadas |Count|Total|Número de acciones de flujo de trabajo que se han completado.|Sin dimensiones|
|ActionsSucceeded|Acciones correctas |Count|Total|Número de acciones de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|ActionsFailed|Acciones con errores |Count|Total|Número de acciones de flujo de trabajo en las que se ha producido un error.|Sin dimensiones|
|ActionsSkipped|Acciones omitidas |Count|Total|Número de acciones de flujo de trabajo que se han omitido.|Sin dimensiones|
|ActionLatency|Latencia de acciones |Segundos|Promedio|Latencia de las acciones de flujo de trabajo que se han completado.|Sin dimensiones|
|ActionSuccessLatency|Latencia de acciones correctas |Segundos|Promedio|Latencia de las acciones de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|ActionThrottledEvents|Eventos limitados de acciones|Count|Total|Número de eventos limitados de la acción de flujo de trabajo.|Sin dimensiones|
|TriggersStarted|Desencadenadores iniciados |Count|Total|Número de desencadenadores de flujo de trabajo que se han iniciado.|Sin dimensiones|
|TriggersCompleted|Desencadenadores completados |Count|Total|Número de desencadenadores de flujo de trabajo que se han completado.|Sin dimensiones|
|TriggersSucceeded|Desencadenadores correctos |Count|Total|Número de desencadenadores de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|TriggersFailed|Desencadenadores con errores |Count|Total|Número de desencadenadores de flujo de trabajo en los que se ha producido un error.|Sin dimensiones|
|TriggersSkipped|Desencadenadores omitidos|Count|Total|Número de desencadenadores de flujo de trabajo que se han omitido.|Sin dimensiones|
|TriggersFired|Desencadenadores activados |Count|Total|Número de desencadenadores de flujo de trabajo que se han activado.|Sin dimensiones|
|TriggerLatency|Latencia de desencadenador |Segundos|Promedio|Latencia de los desencadenadores de flujo de trabajo que se han completado.|Sin dimensiones|
|TriggerFireLatency|Latencia de desencadenador activado |Segundos|Promedio|Latencia de los desencadenadores de flujo de trabajo que se han activado.|Sin dimensiones|
|TriggerSuccessLatency|Latencia de desencadenadores correctos |Segundos|Promedio|Latencia de los desencadenadores de flujo de trabajo que se han realizado correctamente.|Sin dimensiones|
|TriggerThrottledEvents|Eventos limitados de desencadenadores|Count|Total|Número de eventos limitados del desencadenador de flujo de trabajo.|Sin dimensiones|
|IntegrationServiceEnvironmentWorkflowProcessorUsage|Uso de procesador del flujo de trabajo para el entorno del servicio de integración.|Porcentaje|Promedio|Uso de procesador del flujo de trabajo para el entorno del servicio de integración.|Sin dimensiones|
|IntegrationServiceEnvironmentWorkflowMemoryUsage|Uso de memoria del flujo de trabajo para el entorno del servicio de integración|Porcentaje|Promedio|Uso de memoria del flujo de trabajo para el entorno del servicio de integración.|Sin dimensiones|
|IntegrationServiceEnvironmentConnectorProcessorUsage|Uso de procesador del conector para el entorno del servicio de integración|Porcentaje|Promedio|Uso de procesador del conector para el entorno del servicio de integración.|Sin dimensiones|
|IntegrationServiceEnvironmentConnectorMemoryUsage|Uso de memoria del conector para el entorno del servicio de integración|Porcentaje|Promedio|Uso de memoria del conector para el entorno del servicio de integración.|Sin dimensiones|

## <a name="microsoftmachinelearningservicesworkspaces"></a>Microsoft.MachineLearningServices/workspaces

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|Ejecuciones completadas|Ejecuciones completadas|Count|Total|Número de ejecuciones completada correctamente para esta área de trabajo|Escenario|
|Ejecuciones de introducción|Ejecuciones de introducción|Count|Total|Número de ejecuciones de introducción para esta área de trabajo|Escenario|
|Ejecuciones con errores|Ejecuciones con errores|Count|Total|Número de ejecuciones con error para esta área de trabajo|Escenario|

## <a name="microsoftmapsaccounts"></a>Microsoft.Maps/accounts

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|Uso|Uso|Count|Count|Recuento de llamadas a la API|ApiCategory, ApiName, ResultType, ResponseCode|
|Disponibilidad|Disponibilidad|Porcentaje|Promedio|Disponibilidad de las API|ApiCategory, ApiName|

## <a name="microsoftnetappnetappaccountscapacitypoolsvolumes"></a>Microsoft.NetApp/netAppAccounts/capacityPools/Volumes

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|AverageOtherLatency|Otra latencia media|ms/op|Promedio|Otra latencia media (que no sea de lectura ni escritura) en milisegundos por operación|Sin dimensiones|
|AverageReadLatency|Latencia de lectura media|ms/op|Promedio|Latencia de lectura media en milisegundos por operación|Sin dimensiones|
|AverageTotalLatency|Latencia total media|ms/op|Promedio|Latencia total media en milisegundos por operación|Sin dimensiones|
|AverageWriteLatency|Latencia de escritura media|ms/op|Promedio|Latencia de escritura media en milisegundos por operación|Sin dimensiones|
|FilesystemOtherOps|Otras operaciones del sistema de archivos|ops|Promedio|Número de otras operaciones del sistema de archivos (que no sean de lectura ni escritura)|Sin dimensiones|
|FilesystemReadOps|Operaciones de lectura del sistema de archivos|ops|Promedio|Número de operaciones de lectura del sistema de archivos|Sin dimensiones|
|FilesystemTotalOps|Total de operaciones del sistema de archivos|ops|Promedio|Suma de todas las operaciones del sistema de archivos|Sin dimensiones|
|FilesystemWriteOps|Operaciones de escritura del sistema de archivos|ops|Promedio|Número de operaciones de escritura del sistema de archivos|Sin dimensiones|
|IoBytesPerOtherOps|Bytes de E/S por otras operaciones|bytes/op|Promedio|Número de bytes de entrada/salida para otras operaciones (que no sean de lectura ni escritura)|Sin dimensiones|
|IoBytesPerReadOps|Bytes de E/S por operación de lectura|bytes/op|Promedio|Número de bytes de entrada/salida por operación de lectura|Sin dimensiones|
|IoBytesPerTotalOps|Bytes de E/S por operación en todas las operaciones|bytes/op|Promedio|Suma de toda la operación de bytes de entrada/salida|Sin dimensiones|
|IoBytesPerWriteOps|Bytes de E/S por operación de escritura|bytes/op|Promedio|Número de bytes de entrada/salida por operación de lectura|Sin dimensiones|
|OtherIops|Otras IOPS|operations/second|Promedio|Otra operación de entrada/salida por segundo|Sin dimensiones|
|OtherThroughput|Otro rendimiento|MBps|Promedio|Otro rendimiento (que no sea de lectura ni escritura) en megabytes por segundo|Sin dimensiones|
|ReadIops|IOPS de lectura|operations/second|Promedio|Operaciones de entrada/salida de lectura por segundo|Sin dimensiones|
|ReadThroughput|Rendimiento de lectura|MBps|Promedio|Rendimiento de lectura en megabytes por segundo|Sin dimensiones|
|TotalIops|Total de IOPS|operations/second|Promedio|Suma de todas las operaciones de entrada/salida por segundo|Sin dimensiones|
|TotalThroughput|Rendimiento total|MBps|Promedio|Suma de todo el rendimiento en megabytes por segundo|Sin dimensiones|
|VolumeAllocatedSize|Tamaño asignado del volumen|bytes|Promedio|Tamaño asignado del volumen (no los bytes usados reales)|Sin dimensiones|
|VolumeLogicalSize|Tamaño lógico del volumen|bytes|Promedio|Tamaño lógico del volumen (bytes usados)|Sin dimensiones|
|VolumeSnapshotSize|Tamaño de instantánea de volumen|bytes|Promedio|Tamaño de todas las instantáneas del volumen|Sin dimensiones|
|WriteIops|IOPS de escritura|operations/second|Promedio|Operaciones de entrada/salida de escritura por segundo|Sin dimensiones|
|WriteThroughput|Rendimiento de escritura|MBps|Promedio|Rendimiento de escritura en megabytes por segundo|Sin dimensiones|

## <a name="microsoftnetappnetappaccountscapacitypools"></a>Microsoft.NetApp/netAppAccounts/capacityPools

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|VolumePoolAllocatedSize|Tamaño asignado del grupo de volumen|bytes|Promedio|Tamaño asignado del grupo (no los bytes usados reales)|Sin dimensiones|
|VolumePoolAllocatedUsed|Uso asignado del grupo de volumen|bytes|Promedio|Tamaño usado asignado del grupo|Sin dimensiones|
|VolumePoolTotalLogicalSize|Tamaño lógico total del grupo de volumen|bytes|Promedio|Suma del tamaño lógico de todos los volúmenes que pertenecen al grupo|Sin dimensiones|
|VolumePoolTotalSnapshotSize|Tamaño de instantánea total del grupo de volumen|bytes|Promedio|Suma de todas las instantáneas del grupo|Sin dimensiones|

## <a name="microsoftnetworknetworkinterfaces"></a>Microsoft.Network/networkInterfaces

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|BytesSentRate|Bytes enviados|Count|Total|Número de bytes de la interfaz de red enviados|Sin dimensiones|
|BytesReceivedRate|Bytes recibidos|Count|Total|Número de bytes de la interfaz de red recibidos|Sin dimensiones|
|PacketsSentRate|Paquetes enviados|Count|Total|Número de paquetes de la interfaz de red enviados|Sin dimensiones|
|PacketsReceivedRate|Paquetes recibidos|Count|Total|Número de paquetes de la interfaz de red recibidos|Sin dimensiones|

## <a name="microsoftnetworkloadbalancers"></a>Microsoft.Network/loadBalancers

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|VipAvailability|Disponibilidad de la ruta de acceso de datos|Count|Promedio|Disponibilidad media de la ruta de acceso de datos de Load Balancer por duración|FrontendIPAddress, FrontendPort|
|DipAvailability|Estado del sondeo de mantenimiento|Count|Promedio|Estado del sondeo de mantenimiento medio de Load Balancer por duración|ProtocolType, BackendPort, FrontendIPAddress, FrontendPort, BackendIPAddress|
|ByteCount|Recuento de bytes|Count|Total|Número total de bytes transmitidos en un período|FrontendIPAddress, FrontendPort, Direction|
|PacketCount|Recuento de paquetes|Count|Total|Número total de paquetes transmitidos en un período|FrontendIPAddress, FrontendPort, Direction|
|SYNCount|Recuento de SYN|Count|Total|Número total de paquetes SYN transmitidos en un período|FrontendIPAddress, FrontendPort, Direction|
|SnatConnectionCount|Recuento de conexiones SNAT|Count|Total|Número total de conexiones SNAT nuevas creadas en un período|FrontendIPAddress, BackendIPAddress, ConnectionState|
|AllocatedSnatPorts|Puertos SNAT asignados (versión preliminar)|Count|Total|Número total de puertos SNAT asignados dentro del período de tiempo|FrontendIPAddress, BackendIPAddress, ProtocolType|
|UsedSnatPorts|Puertos SNAT usados (versión preliminar)|Count|Total|Número total de puertos SNAT usados dentro del período de tiempo|FrontendIPAddress, BackendIPAddress, ProtocolType|

## <a name="microsoftnetworkdnszones"></a>Microsoft.Network/dnszones

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|QueryVolume|Volumen de consulta|Count|Total|Número de consultas que se sirven para una zona DNS|Sin dimensiones|
|RecordSetCount|Recuento de conjunto de registros|Count|Máximo|Número de conjuntos de registros de una zona DNS|Sin dimensiones|
|RecordSetCapacityUtilization|Uso de capacidad de conjunto de registros|Porcentaje|Máximo|Porcentaje de capacidad de conjunto de registros que usa una zona DNS|Sin dimensiones|

## <a name="microsoftnetworkpublicipaddresses"></a>Microsoft.Network/publicIPAddresses

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|PacketsInDDoS|Paquetes entrantes de DDoS|CountPerSecond|Máximo|Paquetes entrantes de DDoS|Sin dimensiones|
|PacketsDroppedDDoS|Paquetes entrantes de DDoS descartados|CountPerSecond|Máximo|Paquetes entrantes de DDoS descartados|Sin dimensiones|
|PacketsForwardedDDoS|Paquetes entrantes de DDoS reenviados|CountPerSecond|Máximo|Paquetes entrantes de DDoS reenviados|Sin dimensiones|
|TCPPacketsInDDoS|Paquetes TCP entrantes de DDoS|CountPerSecond|Máximo|Paquetes TCP entrantes de DDoS|Sin dimensiones|
|TCPPacketsDroppedDDoS|Paquetes TCP entrantes de DDoS descartados|CountPerSecond|Máximo|Paquetes TCP entrantes de DDoS descartados|Sin dimensiones|
|TCPPacketsForwardedDDoS|Paquetes TCP entrantes de DDoS reenviados|CountPerSecond|Máximo|Paquetes TCP entrantes de DDoS reenviados|Sin dimensiones|
|UDPPacketsInDDoS|Paquetes UDP entrantes de DDoS|CountPerSecond|Máximo|Paquetes UDP entrantes de DDoS|Sin dimensiones|
|UDPPacketsDroppedDDoS|Paquetes UDP entrantes de DDoS descartados|CountPerSecond|Máximo|Paquetes UDP entrantes de DDoS descartados|Sin dimensiones|
|UDPPacketsForwardedDDoS|Paquetes UDP entrantes de DDoS reenviados|CountPerSecond|Máximo|Paquetes UDP entrantes de DDoS reenviados|Sin dimensiones|
|BytesInDDoS|Bytes de entrada de DDoS|BytesPerSecond|Máximo|Bytes de entrada de DDoS|Sin dimensiones|
|BytesDroppedDDoS|Bytes de entrada de DDoS descartados|BytesPerSecond|Máximo|Bytes de entrada de DDoS descartados|Sin dimensiones|
|BytesForwardedDDoS|Bytes de entrada de DDoS reenviados|BytesPerSecond|Máximo|Bytes de entrada de DDoS reenviados|Sin dimensiones|
|TCPBytesInDDoS|Bytes de entrada TCP de DDoS|BytesPerSecond|Máximo|Bytes de entrada TCP de DDoS|Sin dimensiones|
|TCPBytesDroppedDDoS|Bytes de entrada TCP de DDoS descartados|BytesPerSecond|Máximo|Bytes de entrada TCP de DDoS descartados|Sin dimensiones|
|TCPBytesForwardedDDoS|Bytes de entrada TCP de DDoS reenviados|BytesPerSecond|Máximo|Bytes de entrada TCP de DDoS reenviados|Sin dimensiones|
|UDPBytesInDDoS|Bytes de entrada UDP de DDoS|BytesPerSecond|Máximo|Bytes de entrada UDP de DDoS|Sin dimensiones|
|UDPBytesDroppedDDoS|Bytes de entrada UDP de DDoS descartados|BytesPerSecond|Máximo|Bytes de entrada UDP de DDoS descartados|Sin dimensiones|
|UDPBytesDroppedDDoS|Bytes de entrada UDP de DDoS reenviados|BytesPerSecond|Máximo|Bytes de entrada UDP de DDoS reenviados|Sin dimensiones|
|IfUnderDDoSAttack|Bajo ataque de DDoS o no|Count|Máximo|Bajo ataque de DDoS o no|Sin dimensiones|
|DDoSTriggerTCPPackets|Paquetes entrantes TCP para desencadenar la mitigación de DDoS|CountPerSecond|Máximo|Paquetes entrantes TCP para desencadenar la mitigación de DDoS|Sin dimensiones|
|DDoSTriggerUDPPackets|Paquetes entrantes UDP para desencadenar la mitigación de DDoS|CountPerSecond|Máximo|Paquetes entrantes UDP para desencadenar la mitigación de DDoS|Sin dimensiones|
|DDoSTriggerSYNPackets|Paquetes entrantes SYN para desencadenar la mitigación de DDoS|CountPerSecond|Máximo|Paquetes entrantes SYN para desencadenar la mitigación de DDoS|Sin dimensiones|
|VipAvailability|Disponibilidad de la ruta de acceso de datos|Count|Promedio|Disponibilidad media de la dirección IP por duración|Port|
|ByteCount|Recuento de bytes|Count|Total|Número total de bytes transmitidos en un período|Port, Direction|
|PacketCount|Recuento de paquetes|Count|Total|Número total de paquetes transmitidos en un período|Port, Direction|
|SynCount|Recuento de SYN|Count|Total|Número total de paquetes SYN transmitidos en un período|Port, Direction|

## <a name="microsoftnetworkazurefirewalls"></a>Microsoft.Network/azurefirewalls

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|ApplicationRuleHit|Número de llamadas de las reglas de aplicación|Count|Total|Número de veces que se alcanzaron las reglas de aplicación|Protocolo de estado, motivo,|
|NetworkRuleHit|Número de llamadas de las reglas de red|Count|Total|Número de veces que se alcanzaron las reglas de red|Protocolo de estado, motivo,|

## <a name="microsoftnetworkapplicationgateways"></a>Microsoft.Network/applicationGateways

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|Throughput|Throughput|BytesPerSecond|Total|Número de bytes por segundo que ha ofrecido Application Gateway|Sin dimensiones|
|UnhealthyHostCount|Recuento de hosts con estado incorrecto|Count|Promedio|Número de hosts de back-end con estado incorrecto|BackendSettingsPool|
|HealthyHostCount|Recuento de hosts con estado correcto|Count|Promedio|Número de hosts de back-end con estado correcto|BackendSettingsPool|
|TotalRequests|Total de solicitudes|Count|Total|Recuento de solicitudes correctas que ha servido Application Gateway|BackendSettingsPool|
|FailedRequests|Solicitudes erróneas|Count|Total|Recuento de solicitudes con error que ha servido Application Gateway|BackendSettingsPool|
|ResponseStatus|Estado de respuesta|Count|Total|Estado de respuesta HTTP devuelto por Application Gateway|HttpStatusGroup|
|CurrentConnections|Conexiones actuales|Count|Total|Recuento de conexiones actuales establecidas con Application Gateway|Sin dimensiones|
|CapacityUnits|Unidades de capacidad actuales|Count|Promedio|Unidades de capacidad consumidas|Sin dimensiones|

## <a name="microsoftnetworkvirtualnetworkgateways"></a>Microsoft.Network/virtualNetworkGateways

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|AverageBandwidth|Ancho de banda S2S de puerta de enlace|BytesPerSecond|Promedio|Ancho de banda de sitio a sitio medio de una puerta de enlace en bytes por segundo|Sin dimensiones|
|P2SBandwidth|Ancho de banda P2S de puerta de enlace|BytesPerSecond|Promedio|Ancho de banda de punto a sitio medio de una puerta de enlace en bytes por segundo|Sin dimensiones|
|P2SConnectionCount|Recuento de conexiones P2S|Count|Máximo|Recuento de conexiones de punto a sitio de una puerta de enlace|Protocol|
|TunnelAverageBandwidth|Ancho de banda de túnel|BytesPerSecond|Promedio|Media de ancho de banda de un túnel en bytes por segundo|ConnectionName, RemoteIP|
|TunnelEgressBytes|Bytes de salida de túnel|Bytes|Total|Bytes de salida de un túnel|ConnectionName, RemoteIP|
|TunnelIngressBytes|Bytes de salida de túnel|Bytes|Total|Bytes de entrada de un túnel|ConnectionName, RemoteIP|
|TunnelEgressPackets|Paquetes de salida de túnel|Count|Total|Recuento de paquetes salientes de un túnel|ConnectionName, RemoteIP|
|TunnelIngressPackets|Paquetes de entrada de túnel|Count|Total|Recuento de paquetes entrantes de un túnel|ConnectionName, RemoteIP|
|TunnelEgressPacketDropTSMismatch|Colocación de paquetes con error de coincidencia del selector de tráfico de túnel de salida|Count|Total|Recuento de paquetes de salida con error de coincidencia del selector de tráfico de un túnel|ConnectionName, RemoteIP|
|TunnelIngressPacketDropTSMismatch|Colocación de paquetes con error de coincidencia del selector de tráfico de túnel de entrada|Count|Total|Recuento de paquetes entrantes con error de coincidencia del selector de tráfico de un túnel|ConnectionName, RemoteIP|

## <a name="microsoftnetworkexpressroutecircuits"></a>Microsoft.Network/expressRouteCircuits

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|BitsInPerSecond|BitsInPerSecond|CountPerSecond|Promedio|Bits que ingresan a Azure por segundo|PeeringType|
|BitsOutPerSecond|BitsOutPerSecond|CountPerSecond|Promedio|Bits que egresan de Azure por segundo|PeeringType|

## <a name="microsoftnetworkexpressroutecircuitspeerings"></a>Microsoft.Network/expressRouteCircuits/peerings

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|BitsInPerSecond|BitsInPerSecond|CountPerSecond|Promedio|Bits que ingresan a Azure por segundo|Sin dimensiones|
|BitsOutPerSecond|BitsOutPerSecond|CountPerSecond|Promedio|Bits que egresan de Azure por segundo|Sin dimensiones|

## <a name="microsoftnetworkconnections"></a>Microsoft.Network/connections

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|BitsInPerSecond|BitsInPerSecond|CountPerSecond|Promedio|Bits que ingresan a Azure por segundo|Sin dimensiones|
|BitsOutPerSecond|BitsOutPerSecond|CountPerSecond|Promedio|Bits que egresan de Azure por segundo|Sin dimensiones|

## <a name="microsoftnetworktrafficmanagerprofiles"></a>Microsoft.Network/trafficManagerProfiles

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|QpsByEndpoint|Consultas por punto de conexión devueltas|Count|Total|Número de veces que se devolvió un punto de conexión de Traffic Manager en el período dado|EndpointName|
|ProbeAgentCurrentEndpointStateByProfileResourceId|Estado de punto de conexión por punto de conexión|Count|Máximo|1 si el estado de sondeo de un punto de conexión es “Habilitado”, 0 en caso contrario.|EndpointName|

## <a name="microsoftnetworknetworkwatchersconnectionmonitors"></a>Microsoft.Network/networkWatchers/connectionMonitors

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|ProbesFailedPercent|% de sondeos con error|Porcentaje|Promedio|% de sondeos de supervisión de conectividad con error|Sin dimensiones|
|AverageRoundtripMs|Prom. Tiempo de ida y vuelta (ms)|MilliSeconds|Promedio|Promedio de tiempo de ida y vuelta de red (ms) para los sondeos de supervisión de conectividad que se envían entre el origen y el destino|Sin dimensiones|

## <a name="microsoftnetworkfrontdoors"></a>Microsoft.Network/frontdoors

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|RequestCount|Recuento de solicitudes|Count|Total|Número de solicitudes de cliente que atiende el proxy HTTP/S|HttpStatus, HttpStatusGroup, ClientRegion, ClientCountry|
|RequestSize|Tamaño de la solicitud|Bytes|Total|Número de bytes enviados como solicitudes de clientes al proxy HTTP/S|HttpStatus, HttpStatusGroup, ClientRegion, ClientCountry|
|ResponseSize|Tamaño de la respuesta|Bytes|Total|Número de bytes enviados como respuestas del proxy HTTP/S a los clientes|HttpStatus, HttpStatusGroup, ClientRegion, ClientCountry|
|BackendRequestCount|Recuento de solicitudes de back-end|Count|Total|Número de solicitudes enviadas del proxy HTTP/S a los back-end|HttpStatus, HttpStatusGroup, Backend|
|BackendRequestLatency|Latencia de las solicitudes de back-end|MilliSeconds|Promedio|Tiempo calculado desde que el proxy HTTP/S ha enviado la solicitud al back-end hasta que el proxy HTTP/S ha recibido el último byte de respuesta del back-end|Back-end|
|TotalLatency|Latencia total|MilliSeconds|Promedio|Tiempo calculado desde que el proxy HTTP/S ha recibido la solicitud de cliente hasta que el cliente ha reconocido el último byte de respuesta del proxy HTTP/S|HttpStatus, HttpStatusGroup, ClientRegion, ClientCountry|
|BackendHealthPercentage|Porcentaje de estado del back-end|Porcentaje|Promedio|El porcentaje de sondeos de estado correctos del proxy HTTP/S a los back-ends|Backend, BackendPool|
|WebApplicationFirewallRequestCount|Recuento de solicitudes del firewall de aplicaciones web|Count|Total|Número de solicitudes de cliente procesadas por el firewall de aplicaciones web|PolicyName, RuleName, Action|

## <a name="microsoftnotificationhubsnamespacesnotificationhubs"></a>Microsoft.NotificationHubs/Namespaces/NotificationHubs

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|registration.all|Operaciones de registro|Count|Total|El recuento de todas las operaciones de registro correctas (creaciones, actualizaciones, consultas y eliminaciones). |Sin dimensiones|
|registration.create|Operaciones de creación de registros|Count|Total|Número de creaciones de registro correctas.|Sin dimensiones|
|registration.update|Operaciones de actualización de registros|Count|Total|Número de actualizaciones de registro correctas.|Sin dimensiones|
|registration.get|Operaciones de lectura de registros|Count|Total|Número de consultas de registro correctas.|Sin dimensiones|
|registration.delete|Operaciones de eliminación de registros|Count|Total|Número de eliminaciones de registro correctas.|Sin dimensiones|
|incoming|Mensajes entrantes|Count|Total|El recuento de todas las llamadas a la API de envío. |Sin dimensiones|
|incoming.scheduled|Notificaciones push programadas enviadas|Count|Total|Notificaciones Push programadas canceladas|Sin dimensiones|
|incoming.scheduled.cancel|Notificaciones Push programadas canceladas|Count|Total|Notificaciones Push programadas canceladas|Sin dimensiones|
|scheduled.pending|Notificaciones programadas pendientes|Count|Total|Hay notificaciones programadas pendientes.|Sin dimensiones|
|installation.all|Operaciones de administración de instalaciones|Count|Total|Operaciones de administración de la instalación|Sin dimensiones|
|installation.get|Obtener operaciones de instalación|Count|Total|Obtener operaciones de instalación|Sin dimensiones|
|installation.upsert|Crear o actualizar operaciones de instalación|Count|Total|Crear o actualizar operaciones de instalación|Sin dimensiones|
|installation.patch|Operaciones de instalación de revisiones|Count|Total|Operaciones de instalación de revisiones|Sin dimensiones|
|installation.delete|Eliminar operaciones de instalación|Count|Total|Eliminar operaciones de instalación|Sin dimensiones|
|outgoing.allpns.success|Notificaciones correctas|Count|Total|Número de notificaciones correctas.|Sin dimensiones|
|outgoing.allpns.invalidpayload|Errores de carga útil|Count|Total|Número de inserciones con error debido a que PNS devolvió un error de carga incorrecta.|Sin dimensiones|
|outgoing.allpns.pnserror|Errores del sistema de notificación externo|Count|Total|Número de inserciones con error debido a que hubo un problema al establecer comunicación con PNS (excluye problemas de autenticación).|Sin dimensiones|
|outgoing.allpns.channelerror|Errores de canal|Count|Total|Número de inserciones con error debido a que el canal no era válido, no estaba asociado a la aplicación correcta, estaba limitado o expiró.|Sin dimensiones|
|outgoing.allpns.badorexpiredchannel|Errores de canal incorrecto o expirado|Count|Total|Número de inserciones con error debido a que el canal, el token o el elemento registrationId del registro expiró o no era válido.|Sin dimensiones|
|outgoing.wns.success|Notificaciones de WNS correctas|Count|Total|Número de notificaciones correctas.|Sin dimensiones|
|outgoing.wns.invalidcredentials|Errores de autorización de WNS (credenciales no válidas)|Count|Total|Número de inserciones con error debido a que PNS no aceptó las credenciales proporcionadas o las credenciales están bloqueadas. (Windows Live no reconoce las credenciales).|Sin dimensiones|
|outgoing.wns.badchannel|Error de canal incorrecto de WNS|Count|Total|El recuento de inserciones que provocaron un error porque no se reconoció ChannelURI en el registro (estado de WNS: 404 No encontrado).|Sin dimensiones|
|outgoing.wns.expiredchannel|Error de canal expirado de WNS|Count|Total|El recuento de inserciones que provocaron un error porque ChannelURI expiró (estado de WNS: 410 Ya no existe).|Sin dimensiones|
|outgoing.wns.throttled|Notificaciones limitadas de WNS|Count|Total|El recuento de inserciones que provocaron un error porque WNS limita esta aplicación (estado de WNS: 406 No aceptable).|Sin dimensiones|
|outgoing.wns.tokenproviderunreachable|Errores de autorización de WNS (inaccesible)|Count|Total|No se puede acceder a Windows Live.|Sin dimensiones|
|outgoing.wns.invalidtoken|Errores de autorización de WNS (token no válido)|Count|Total|El token que se proporciona a WNS no es válido (estado de WNS: 401 No autorizado).|Sin dimensiones|
|outgoing.wns.wrongtoken|Errores de autorización de WNS (token erróneo)|Count|Total|El token que se proporciona a WNS es válido, pero para otra aplicación (estado de WNS: 403 Prohibido). Esto puede ocurrir si la instancia de ChannelURI en el registro está asociada a otra aplicación. Compruebe que la aplicación cliente está asociada a la misma aplicación cuyas credenciales están en el centro de notificaciones.|Sin dimensiones|
|outgoing.wns.invalidnotificationformat|Formato de notificación no válido de WNS|Count|Total|El formato de la notificación no es válido (estado de WNS: 400). Tenga en cuenta que WNS no rechaza todas las cargas útiles no válidas.|Sin dimensiones|
|outgoing.wns.invalidnotificationsize|Error de tamaño de notificación no válido de WNS|Count|Total|La carga útil de la notificación es demasiado grande (estado de WNS: 413).|Sin dimensiones|
|outgoing.wns.channelthrottled|Canal de WNS limitado|Count|Total|La notificación se descartó porque la instancia de ChannelURI del registro está limitada (encabezado de respuesta WNS: X-WNS-NotificationStatus:channelThrottled).|Sin dimensiones|
|outgoing.wns.channeldisconnected|Canal de WNS desconectado|Count|Total|La notificación se descartó porque la instancia de ChannelURI del registro está limitada (encabezado de respuesta WNS: X-WNS-DeviceConnectionStatus: desconectado).|Sin dimensiones|
|outgoing.wns.dropped|Notificaciones descartadas de WNS|Count|Total|Se eliminó la notificación porque el elemento ChannelURI del registro está limitado (X-WNS-NotificationStatus: eliminado, pero no X-WNS-DeviceConnectionStatus: desconectado).|Sin dimensiones|
|outgoing.wns.pnserror|Errores de WNS|Count|Total|No se envió la notificación porque hubo errores de comunicación con WNS.|Sin dimensiones|
|outgoing.wns.authenticationerror|Errores de autenticación de WNS|Count|Total|No se envió la notificación porque hubo errores de comunicación con Windows Live por credenciales no válidas o por un token incorrecto.|Sin dimensiones|
|outgoing.apns.success|Notificaciones de APNS correctas|Count|Total|Número de notificaciones correctas.|Sin dimensiones|
|outgoing.apns.invalidcredentials|Errores de autorización de APNS|Count|Total|Número de inserciones con error debido a que PNS no aceptó las credenciales proporcionadas o las credenciales están bloqueadas.|Sin dimensiones|
|outgoing.apns.badchannel|Error de canal incorrecto de APNS|Count|Total|El recuento de inserciones que provocaron un error porque el token no es válido (código de estado de APNS: 8).|Sin dimensiones|
|outgoing.apns.expiredchannel|Error de canal expirado de APNS|Count|Total|Número de tokens que invalidó el canal de comentarios de APNs.|Sin dimensiones|
|outgoing.apns.invalidnotificationsize|Error de tamaño de notificación no válido de APNS|Count|Total|El recuento de inserciones que provocaron un error porque la carga útil era demasiado grande (código de estado de APNS: 7).|Sin dimensiones|
|outgoing.apns.pnserror|Errores de APNS|Count|Total|Número de inserciones con error debido a problemas de comunicación con APNs.|Sin dimensiones|
|outgoing.gcm.success|Notificaciones de GCM correctas|Count|Total|Número de notificaciones correctas.|Sin dimensiones|
|outgoing.gcm.invalidcredentials|Errores de autorización de GCM (credenciales no válidas)|Count|Total|Número de inserciones con error debido a que PNS no aceptó las credenciales proporcionadas o las credenciales están bloqueadas.|Sin dimensiones|
|outgoing.gcm.badchannel|Error de canal incorrecto de GCM|Count|Total|El recuento de inserciones que provocaron un error porque la instancia de registrationId del registro no se ha reconocido (resultado de GCM: registro no válido).|Sin dimensiones|
|outgoing.gcm.expiredchannel|Error de canal expirado de GCM|Count|Total|El recuento de inserciones que provocaron un error porque la instancia de registrationId del registro ha caducado (resultado de GCM: NotRegistered).|Sin dimensiones|
|outgoing.gcm.throttled|Notificaciones limitadas de GCM|Count|Total|El recuento de inserciones que provocaron un error porque GCM limitó esta aplicación (código de estado de GCM: 501-599 o resultado: no disponible).|Sin dimensiones|
|outgoing.gcm.invalidnotificationformat|Formato de notificación no válido de GCM|Count|Total|El recuento de inserciones que provocaron un error porque la carga útil no tenía el formato correcto (resultado de GCM: InvalidDataKey o InvalidTtl).|Sin dimensiones|
|outgoing.gcm.invalidnotificationsize|Error de tamaño de notificación no válido de GCM|Count|Total|El recuento de inserciones que provocaron un error porque la carga útil era demasiado grande (resultado de GCM: MessageTooBig).|Sin dimensiones|
|outgoing.gcm.wrongchannel|Error de canal incorrecto de GCM|Count|Total|El recuento de inserciones que provocaron un error porque la instancia de registrationId del registro no está asociado a la aplicación actual (resultado de GCM: InvalidPackageName).|Sin dimensiones|
|outgoing.gcm.pnserror|Errores de GCM|Count|Total|Número de inserciones con error debido a problemas de comunicación con GCM.|Sin dimensiones|
|outgoing.gcm.authenticationerror|Errores de autenticación de GCM|Count|Total|El recuento de inserciones que provocaron un error debido a que el PNS no aceptó las credenciales proporcionadas, las credenciales están bloqueadas o el Id. del remitente no está configurado correctamente en la aplicación (resultado de GCM: MismatchedSenderId).|Sin dimensiones|
|outgoing.mpns.success|Notificaciones de MPNS correctas|Count|Total|Número de notificaciones correctas.|Sin dimensiones|
|outgoing.mpns.invalidcredentials|Credenciales no válidas de MPNS|Count|Total|Número de inserciones con error debido a que PNS no aceptó las credenciales proporcionadas o las credenciales están bloqueadas.|Sin dimensiones|
|outgoing.mpns.badchannel|Error de canal incorrecto de MPNS|Count|Total|El recuento de inserciones que provocaron un error porque no se reconoció ChannelURI en el registro (estado de MPNS: 404 No encontrado).|Sin dimensiones|
|outgoing.mpns.throttled|Notificaciones limitadas de MPNS|Count|Total|El recuento de inserciones que provocaron un error porque MPNS limita esta aplicación (MPNS de WNS: 406 No aceptable).|Sin dimensiones|
|outgoing.mpns.invalidnotificationformat|Formato de notificación no válido de MPNS|Count|Total|Número de inserciones con error debido a que la carga de la notificación era demasiado grande.|Sin dimensiones|
|outgoing.mpns.channeldisconnected|Canal de MPNS desconectado|Count|Total|El recuento de inserciones que provocaron un error porque se desconectó la instancia ChannelURI en el registro (estado de MPNS: 412 No encontrado).|Sin dimensiones|
|outgoing.mpns.dropped|Notificaciones descartadas de MPNS|Count|Total|El recuento de inserciones que eliminó MPNS (encabezado de respuesta MPNS: X-NotificationStatus: QueueFull o Suprimido).|Sin dimensiones|
|outgoing.mpns.pnserror|Errores de MPNS|Count|Total|Número de inserciones con error debido a problemas de comunicación con MPNS.|Sin dimensiones|
|outgoing.mpns.authenticationerror|Errores de autenticación de MPNS|Count|Total|Número de inserciones con error debido a que PNS no aceptó las credenciales proporcionadas o las credenciales están bloqueadas.|Sin dimensiones|
|notificationhub.pushes|Todas las notificaciones salientes|Count|Total|Todas las notificaciones salientes del Centro de notificaciones|Sin dimensiones|
|incoming.all.requests|Todas las solicitudes entrantes|Count|Total|Total de solicitudes entrantes de un centro de notificaciones|Sin dimensiones|
|incoming.all.failedrequests|Todas las solicitudes entrantes con error|Count|Total|Total de solicitudes entrantes erróneas de un centro de notificaciones|Sin dimensiones|

## <a name="microsoftoperationalinsightsworkspaces"></a>Microsoft.OperationalInsights/workspaces

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|Average_% Free Inodes|% de Inodes libres|Count|Promedio|Average_% Free Inodes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Free Space|% de espacio libre|Count|Promedio|Average_% Free Space|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Used Inodes|% de Inodes usados|Count|Promedio|Average_% Used Inodes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Used Space|% espacio usado|Count|Promedio|Average_% Used Space|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Disk Read Bytes/sec|Bytes de lectura de disco/s|Count|Promedio|Average_Disk Read Bytes/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Disk Reads/sec|Lecturas de disco/s |Count|Promedio|Average_Disk Reads/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Disk Transfers/sec|Transferencias de disco/s|Count|Promedio|Average_Disk Transfers/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Disk Write Bytes/sec|Bytes de escritura en disco/s|Count|Promedio|Average_Disk Write Bytes/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Disk Writes/sec| Escrituras en disco/s|Count|Promedio|Average_Disk Writes/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Free Megabytes|Megabytes libres|Count|Promedio|Average_Free Megabytes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Logical Disk Bytes/sec|Bytes de disco lógico/s|Count|Promedio|Average_Logical Disk Bytes/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Available Memory|% de memoria disponible|Count|Promedio|Average_% Available Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Available Swap Space|% de espacio de intercambio disponible|Count|Promedio|Average_% Available Swap Space|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Used Memory|% de memoria usada|Count|Promedio|Average_% Used Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Used Swap Space|% de espacio de intercambio usado|Count|Promedio|Average_% Used Swap Space|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Available MBytes Memory|MB de memoria disponibles|Count|Promedio|Average_Available MBytes Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Available MBytes Swap|Intercambio de MB disponibles|Count|Promedio|Average_Available MBytes Swap|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Page Reads/sec|Lecturas de página/s|Count|Promedio|Average_Page Reads/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Page Writes/sec|Escrituras de página/s|Count|Promedio|Average_Page Writes/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Pages/sec|Páginas/s|Count|Promedio|Average_Pages/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Used MBytes Swap Space|Espacio de intercambio de MB usado|Count|Promedio|Average_Used MBytes Swap Space|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Used Memory MBytes|MB de memoria usados|Count|Promedio|Average_Used Memory MBytes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Bytes Transmitted|Número total de bytes transmitidos|Count|Promedio|Average_Total Bytes Transmitted|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Bytes Received|Número total de bytes recibidos|Count|Promedio|Average_Total Bytes Received|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Bytes|Total de bytes|Count|Promedio|Average_Total Bytes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Packets Transmitted|Número total de paquetes transmitidos|Count|Promedio|Average_Total Packets Transmitted|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Packets Received|Número total de paquetes recibidos|Count|Promedio|Average_Total Packets Received|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Rx Errors|Errores de Rx totales|Count|Promedio|Average_Total Rx Errors|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Tx Errors|Errores de Tx totales|Count|Promedio|Average_Total Tx Errors|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Collisions|Colisiones totales|Count|Promedio|Average_Total Collisions|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Avg. Segundos de disco/lecturas|Prom. Segundos de disco/lecturas|Count|Promedio|Average_Avg. Segundos de disco/lecturas|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Avg. Segundos de disco/transferencias|Prom. Segundos de disco/transferencias|Count|Promedio|Average_Avg. Segundos de disco/transferencias|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Avg. Segundos de disco/escrituras|Prom. Segundos de disco/escrituras|Count|Promedio|Average_Avg. Segundos de disco/escrituras|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Physical Disk Bytes/sec|Bytes de disco físico/s|Count|Promedio|Average_Physical Disk Bytes/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Pct Privileged Time|Porcentaje de tiempo con privilegios|Count|Promedio|Average_Pct Privileged Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Pct User Time|Porcentaje de tiempo de usuario|Count|Promedio|Average_Pct User Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Used Memory kBytes|Kilobytes de memoria usados|Count|Promedio|Average_Used Memory kBytes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Virtual Shared Memory|Memoria compartida virtual|Count|Promedio|Average_Virtual Shared Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% DPC Time|% de tiempo de DPC|Count|Promedio|Average_% DPC Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Idle Time|% de tiempo de inactividad|Count|Promedio|Average_% Idle Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Interrupt Time|% de tiempo de interrupción|Count|Promedio|Average_% Interrupt Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% IO Wait Time|% de tiempo de espera de E/S|Count|Promedio|Average_% IO Wait Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Nice Time|% de tiempo bueno|Count|Promedio|Average_% Nice Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Privileged Time|% de tiempo con privilegios|Count|Promedio|Average_% Privileged Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Processor Time|% de tiempo de procesador|Count|Promedio|Average_% Processor Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% User Time|% de tiempo de usuario|Count|Promedio|Average_% User Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Free Physical Memory|Memoria física libre|Count|Promedio|Average_Free Physical Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Free Space in Paging Files|Espacio libre en archivos de paginación|Count|Promedio|Average_Free Space in Paging Files|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Free Virtual Memory|Memoria virtual libre|Count|Promedio|Average_Free Virtual Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Processes|Procesos|Count|Promedio|Average_Processes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Size Stored In Paging Files|Tamaño almacenado en archivos de paginación|Count|Promedio|Average_Size Stored In Paging Files|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Uptime|Tiempo de actividad|Count|Promedio|Average_Uptime|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Users|Usuarios|Count|Promedio|Average_Users|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Avg. Segundos de disco/lecturas|Prom. Segundos de disco/lecturas|Count|Promedio|Average_Avg. Segundos de disco/lecturas|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Avg. Segundos de disco/escrituras|Prom. Segundos de disco/escrituras|Count|Promedio|Average_Avg. Segundos de disco/escrituras|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Current Disk Queue Length|Longitud actual de cola de disco|Count|Promedio|Average_Current Disk Queue Length|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Disk Reads/sec|Lecturas de disco/s |Count|Promedio|Average_Disk Reads/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Disk Transfers/sec|Transferencias de disco/s|Count|Promedio|Average_Disk Transfers/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Disk Writes/sec| Escrituras en disco/s|Count|Promedio|Average_Disk Writes/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Free Megabytes|Megabytes libres|Count|Promedio|Average_Free Megabytes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Free Space|% de espacio libre|Count|Promedio|Average_% Free Space|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Available MBytes|MB disponibles|Count|Promedio|Average_Available MBytes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Committed Bytes In Use|% de bytes confirmados en uso|Count|Promedio|Average_% Committed Bytes In Use|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Bytes Received/sec|Bytes recibidos/s|Count|Promedio|Average_Bytes Received/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Bytes Sent/sec|Bytes enviados/seg.|Count|Promedio|Average_Bytes Sent/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Bytes Total/sec|Bytes totales por segundo|Count|Promedio|Average_Bytes Total/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Processor Time|% de tiempo de procesador|Count|Promedio|Average_% Processor Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Processor Queue Length|Longitud de cola del procesador|Count|Promedio|Average_Processor Queue Length|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Latido|Latido|Count|Total|Latido|Computer, OSType, Version, SourceComputerId|
|Actualizar|Actualizar|Count|Promedio|Actualizar|Computer, Product, Classification, UpdateState, Optional, Approved|
|Evento|Evento|Count|Promedio|Evento|Source, EventLog, Computer, EventCategory, EventLevel, EventLevelName, EventID|

## <a name="microsoftpowerbidedicatedcapacities"></a>Microsoft.PowerBIDedicated/capacities

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|QueryDuration|Duración de la consulta|Milisegundos|Promedio|Duración de la consulta DAX en el último intervalo|Sin dimensiones|
|QueryPoolJobQueueLength|Subprocesos: longitud de cola de trabajos del grupo de consultas|Count|Promedio|Número de trabajos en la cola del grupo de subprocesos de consultas.|Sin dimensiones|
|qpu_high_utilization_metric|Uso intensivo de QPU|Count|Total|Uso intensivo de QPU en el último minuto: 1 para un uso intensivo de QPU y 0 en caso contrario.|Sin dimensiones|
|memory_metric|Memoria|Bytes|Promedio|Memoria. Intervalo de 0-3 GB para A1, 0-5 GB para A2, 0-10 GB para A3, 0-25 GB para A4, 0-50 GB para A5 y 0-100 GB para A6|Sin dimensiones|
|memory_thrashing_metric|Paginación excesiva de memoria|Porcentaje|Promedio|Paginación excesiva media de memoria.|Sin dimensiones|

## <a name="microsoftrelaynamespaces"></a>Microsoft.Relay/namespaces

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|ListenerConnections-Success|ListenerConnections-Success|Count|Total|Valor correcto de ListenerConnections para Microsoft.Relay.|EntityName|
|ListenerConnections-ClientError|ListenerConnections-ClientError|Count|Total|ClientError en ListenerConnections para Microsoft.Relay.|EntityName|
|ListenerConnections-ServerError|ListenerConnections-ServerError|Count|Total|ServerError en ListenerConnections para Microsoft.Relay.|EntityName|
|SenderConnections-Success|SenderConnections-Success|Count|Total|Valor correcto de SenderConnections para Microsoft.Relay.|EntityName|
|SenderConnections-ClientError|SenderConnections-ClientError|Count|Total|ClientError en SenderConnections para Microsoft.Relay.|EntityName|
|SenderConnections-ServerError|SenderConnections-ServerError|Count|Total|ServerError en SenderConnections para Microsoft.Relay.|EntityName|
|ListenerConnections-TotalRequests|ListenerConnections-TotalRequests|Count|Total|Valor total de ListenerConnections para Microsoft.Relay.|EntityName|
|SenderConnections-TotalRequests|SenderConnections-TotalRequests|Count|Total|Solicitudes totales de SenderConnections para Microsoft.Relay.|EntityName|
|ActiveConnections|ActiveConnections|Count|Total|Valor total de ActiveConnections para Microsoft.Relay.|EntityName|
|ActiveListeners|ActiveListeners|Count|Total|Valor total de ActiveListeners para Microsoft.Relay.|EntityName|
|BytesTransferred|BytesTransferred|Count|Total|Valor total de BytesTransferred para Microsoft.Relay.|EntityName|
|ListenerDisconnects|ListenerDisconnects|Count|Total|Valor total de ListenerDisconnects para Microsoft.Relay.|EntityName|
|SenderDisconnects|SenderDisconnects|Count|Total|Valor total de SenderDisconnects para Microsoft.Relay.|EntityName|

## <a name="microsoftsearchsearchservices"></a>Microsoft.Search/searchServices

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|SearchLatency|Latencia de búsqueda|Segundos|Promedio|Promedio de latencia de búsqueda para el servicio de búsqueda|Sin dimensiones|
|SearchQueriesPerSecond|Consultas de búsqueda por segundo|CountPerSecond|Promedio|Consultas de búsqueda por segundo para el servicio de búsqueda|Sin dimensiones|
|ThrottledSearchQueriesPercentage|Porcentaje de consultas de búsqueda limitadas|Porcentaje|Promedio|Porcentaje de consultas de búsqueda limitadas para el servicio de búsqueda|Sin dimensiones|

## <a name="microsoftservicebusnamespaces"></a>Microsoft.ServiceBus/namespaces

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|SuccessfulRequests|Solicitudes correctas (versión preliminar)|Count|Total|Número total de solicitudes correctas para un espacio de nombres (versión preliminar)|EntityName|
|ServerErrors|Errores del servidor. (Versión preliminar)|Count|Total|Errores del servidor para Microsoft.ServiceBus. (Versión preliminar)|EntityName|
|UserErrors|Errores de usuario. (Versión preliminar)|Count|Total|Errores de usuario para Microsoft.ServiceBus. (Versión preliminar)|EntityName|
|ThrottledRequests|Solicitudes limitadas. (Versión preliminar)|Count|Total|Solicitudes limitadas para Microsoft.ServiceBus. (Versión preliminar)|EntityName|
|IncomingRequests|Solicitudes entrantes (versión preliminar)|Count|Total|Solicitudes entrantes para Microsoft.ServiceBus. (Versión preliminar)|EntityName|
|IncomingMessages|Mensajes entrantes (versión preliminar)|Count|Total|Mensajes entrantes para Microsoft.ServiceBus. (Versión preliminar)|EntityName|
|OutgoingMessages|Mensajes salientes (versión preliminar)|Count|Total|Mensajes salientes para Microsoft.ServiceBus. (Versión preliminar)|EntityName|
|ActiveConnections|ActiveConnections (versión preliminar)|Count|Total|Total de conexiones activas para Microsoft.ServiceBus. (Versión preliminar)|Sin dimensiones|
|Tamaño|Tamaño (vista previa)|Bytes|Promedio|Tamaño de una cola o un tema en bytes. (Versión preliminar)|EntityName|
|Mensajes|Número de mensajes en un tema o una cola. (Versión preliminar)|Count|Promedio|Número de mensajes en un tema o una cola. (Versión preliminar)|EntityName|
|ActiveMessages|Número de mensajes activos en un tema o una cola. (Versión preliminar)|Count|Promedio|Número de mensajes activos en un tema o una cola. (Versión preliminar)|EntityName|
|DeadletteredMessages|Número de mensajes fallidos en una cola o un tema. (Versión preliminar)|Count|Promedio|Número de mensajes fallidos en una cola o un tema. (Versión preliminar)|EntityName|
|ScheduledMessages|Número de mensajes programados en una cola o un tema. (Versión preliminar)|Count|Promedio|Número de mensajes programados en una cola o un tema. (Versión preliminar)|EntityName|
|CPUXNS|Uso de CPU por espacio de nombres|Porcentaje|Máximo|Métrica del uso de CPU del espacio de nombres de Service Bus Premium|Sin dimensiones|
|WSXNS|Uso de tamaño de memoria por espacio de nombres|Porcentaje|Máximo|Métrica de uso de memoria del espacio de nombres de Service Bus Premium|Sin dimensiones|

## <a name="microsoftservicefabricmeshapplications"></a>Microsoft.ServiceFabricMesh/applications

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|AllocatedCpu|AllocatedCpu|Count|Promedio|Asignados a este contenedor en millares de núcleos de CPU|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|AllocatedMemory|AllocatedMemory|Bytes|Promedio|Memoria asignada a este contenedor en MB|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|ActualCpu|ActualCpu|Count|Promedio|Uso de CPU real en millares de núcleos|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|ActualMemory|ActualMemory|Bytes|Promedio|Uso de memoria real en MB|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|Gráfico|Gráfico|Porcentaje|Promedio|Uso de CPU para este contenedor como un porcentaje de AllocatedCpu|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|MemoryUtilization|MemoryUtilization|Porcentaje|Promedio|Uso de CPU para este contenedor como un porcentaje de AllocatedCpu|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|ApplicationStatus|ApplicationStatus|Count|Promedio|Estado de aplicación de Service Fabric de malla|ApplicationName, estado|
|ServiceStatus|ServiceStatus|Count|Promedio|Estado de mantenimiento de un servicio en la aplicación de Service Fabric de malla|ServiceName ApplicationName, estado,|
|ServiceReplicaStatus|ServiceReplicaStatus|Count|Promedio|Estado de mantenimiento de una réplica de servicio en la aplicación de Service Fabric de malla|ApplicationName, estado, ServiceName, ServiceReplicaName|
|ContainerStatus|ContainerStatus|Count|Promedio|Estado del contenedor de aplicación de Service Fabric de malla|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName, estado|
|RestartCount|RestartCount|Count|Promedio|Reiniciar recuento de un contenedor en la aplicación de Service Fabric de malla|ApplicationName, estado, ServiceName, ServiceReplicaName, CodePackageName|

## <a name="microsoftsignalrservicesignalr"></a>Microsoft.SignalRService/SignalR

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|ConnectionCount|Número de conexiones|Count|Máximo|Cantidad de conexiones de usuario.|Punto de conexión|
|MessageCount|Recuento de mensajes|Count|Total|Cantidad total de mensajes.|Sin dimensiones|
|InboundTraffic|Tráfico entrante|Bytes|Total|Tráfico entrante del servicio|Sin dimensiones|
|OutboundTraffic|Tráfico saliente|Bytes|Total|Tráfico saliente del servicio|Sin dimensiones|
|UserErrors|Errores de usuario|Porcentaje|Máximo|Porcentaje de errores de usuario|Sin dimensiones|
|SystemErrors|Errores de sistema|Porcentaje|Máximo|Porcentaje de errores del sistema|Sin dimensiones|

## <a name="microsoftsqlserversdatabases"></a>Microsoft.Sql/servers/databases

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|cpu_percent|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de CPU|Sin dimensiones|
|physical_data_read_percent|Media de porcentaje de E/S de datos|Porcentaje|Promedio|Media de porcentaje de E/S de datos|Sin dimensiones|
|log_write_percent|Porcentaje de E/S de registro|Porcentaje|Promedio|Porcentaje de E/S de registro|Sin dimensiones|
|dtu_consumption_percent|Porcentaje de DTU|Porcentaje|Promedio|Porcentaje de DTU|Sin dimensiones|
|almacenamiento|Espacio de datos usado|Bytes|Máximo|Tamaño total de base de datos|Sin dimensiones|
|connection_successful|Conexiones correctas|Count|Total|Conexiones correctas|Sin dimensiones|
|connection_failed|Conexiones con error|Count|Total|Conexiones con error|Sin dimensiones|
|blocked_by_firewall|Bloqueado por el firewall|Count|Total|Bloqueado por el firewall|Sin dimensiones|
|deadlock|Interbloqueos|Count|Total|Interbloqueos|Sin dimensiones|
|storage_percent|Porcentaje de espacio utilizado de datos|Porcentaje|Máximo|Porcentaje de tamaño de base de datos|Sin dimensiones|
|xtp_storage_percent|Porcentaje de almacenamiento de OLTP en memoria|Porcentaje|Promedio|Porcentaje de almacenamiento de OLTP en memoria|Sin dimensiones|
|workers_percent|Porcentaje de trabajos|Porcentaje|Promedio|Porcentaje de trabajos|Sin dimensiones|
|sessions_percent|Porcentaje de sesiones|Porcentaje|Promedio|Porcentaje de sesiones|Sin dimensiones|
|dtu_limit|Límite de DTU|Count|Promedio|Límite de DTU|Sin dimensiones|
|dtu_used|DTU utilizada|Count|Promedio|DTU utilizada|Sin dimensiones|
|cpu_limit|Límite de CPU|Count|Promedio|Límite de CPU|Sin dimensiones|
|cpu_used|Uso de CPU|Count|Promedio|Uso de CPU|Sin dimensiones|
|dwu_limit|Límite de DWU|Count|Máximo|Límite de DWU|Sin dimensiones|
|dwu_consumption_percent|Porcentaje de DWU|Porcentaje|Máximo|Porcentaje de DWU|Sin dimensiones|
|dwu_used|DWU utilizada|Count|Máximo|DWU utilizada|Sin dimensiones|
|dw_cpu_percent|Porcentaje de CPU de nivel de nodo de almacenamiento de datos|Porcentaje|Promedio|Porcentaje de CPU de nivel de nodo de almacenamiento de datos|DwLogicalNodeId|
|dw_physical_data_read_percent|Porcentaje de E/S de datos de nivel de nodo de almacenamiento de datos|Porcentaje|Promedio|Porcentaje de E/S de datos de nivel de nodo de almacenamiento de datos|DwLogicalNodeId|
    |cache_hit_percent|Porcentaje de aciertos de caché|Porcentaje|Máximo|Porcentaje de aciertos de caché|Sin dimensiones|
|cache_used_percent|Porcentaje de caché usada|Porcentaje|Máximo|Porcentaje de caché usada|Sin dimensiones|
|local_tempdb_usage_percent|Porcentaje de tempdb local|Porcentaje|Promedio|Porcentaje de tempdb local|Sin dimensiones|
|app_cpu_billed|CPU de aplicación facturada|Count|Total|CPU de aplicación facturada|Sin dimensiones|
|app_cpu_percent|Porcentaje de CPU de aplicación|Porcentaje|Promedio|Porcentaje de CPU de aplicación|Sin dimensiones|
|app_memory_percent|Porcentaje de memoria usada de aplicación|Porcentaje|Promedio|Porcentaje de memoria usada de aplicación|Sin dimensiones|
|allocated_data_storage|Espacio de datos asignado|Bytes|Promedio|Espacio de datos asignado|Sin dimensiones|

## <a name="microsoftsqlserverselasticpools"></a>Microsoft.Sql/servers/elasticPools

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|cpu_percent|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de CPU|Sin dimensiones|
|physical_data_read_percent|Media de porcentaje de E/S de datos|Porcentaje|Promedio|Media de porcentaje de E/S de datos|Sin dimensiones|
|log_write_percent|Porcentaje de E/S de registro|Porcentaje|Promedio|Porcentaje de E/S de registro|Sin dimensiones|
|dtu_consumption_percent|Porcentaje de DTU|Porcentaje|Promedio|Porcentaje de DTU|Sin dimensiones|
|storage_percent|Porcentaje de espacio utilizado de datos||Porcentaje|Promedio|Porcentaje de almacenamiento|Sin dimensiones|
|workers_percent|Porcentaje de trabajos|Porcentaje|Promedio|Porcentaje de trabajos|Sin dimensiones|
|sessions_percent|Porcentaje de sesiones|Porcentaje|Promedio|Porcentaje de sesiones|Sin dimensiones|
|eDTU_limit|Límite de eDTU|Count|Promedio|Límite de eDTU|Sin dimensiones|
|storage_limit|Tamaño máximo de datos|Bytes|Promedio|Límite de almacenamiento|Sin dimensiones|
|eDTU_used|eDTU utilizada|Count|Promedio|eDTU utilizada|Sin dimensiones|
|storage_used|Espacio de datos usado|Bytes|Promedio|Almacenamiento usado|Sin dimensiones|
|xtp_storage_percent|Porcentaje de almacenamiento de OLTP en memoria|Porcentaje|Promedio|Porcentaje de almacenamiento de OLTP en memoria|Sin dimensiones|
|cpu_limit|Límite de CPU|Count|Promedio|Límite de CPU|Sin dimensiones|
|cpu_used|Uso de CPU|Count|Promedio|Uso de CPU|Sin dimensiones|
|allocated_data_storage|Espacio de datos asignado|Bytes|Promedio|Espacio de datos asignado|Sin dimensiones|
|allocated_data_storage_percent|Porcentaje de espacio asignado de datos|Porcentaje|Máximo|Porcentaje de espacio asignado de datos|Sin dimensiones|

## <a name="microsoftsqlmanagedinstances"></a>Microsoft.Sql/managedInstances

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|virtual_core_count|Recuento de núcleos virtuales|Count|Promedio|Recuento de núcleos virtuales|Sin dimensiones|
|avg_cpu_percent|Porcentaje de CPU medio|Porcentaje|Promedio|Porcentaje de CPU medio|Sin dimensiones|
|reserved_storage_mb|Espacio de almacenamiento reservado|Count|Promedio|Espacio de almacenamiento reservado|Sin dimensiones|
|storage_space_used_mb|Espacio de almacenamiento usado|Count|Promedio|Espacio de almacenamiento usado|Sin dimensiones|
|io_requests|Recuento de solicitudes de E/S|Count|Promedio|Recuento de solicitudes de E/S|Sin dimensiones|
|io_bytes_read|Bytes de E/S leídos|Bytes|Promedio|Bytes de E/S leídos|Sin dimensiones|
|io_bytes_written|Bytes de E/S escritos|Bytes|Promedio|Bytes de E/S escritos|Sin dimensiones|

## <a name="microsoftstoragestorageaccounts"></a>Microsoft.Storage/storageAccounts

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|UsedCapacity|Capacidad usada|Bytes|Promedio|Capacidad de cuenta usada|Sin dimensiones|
|Transacciones|Transacciones|Count|Total|El número de solicitudes realizadas a un servicio de almacenamiento o la operación de API especificada. Este número incluye solicitudes correctas y con errores, así como las solicitudes que generaron errores. Use la dimensión ResponseType para el número de diferentes tipos de respuesta.|ResponseType, GeoType, ApiName, Authentication|
|Entrada|Entrada|Bytes|Total|La cantidad de datos de entrada, en bytes. Este número incluye la entradas desde un cliente externo en Azure Storage, así como la entrada dentro de Azure.|GeoType, ApiName, Authentication|
|Salida|Salida|Bytes|Total|La cantidad de datos de salida, en bytes. Este número incluye la salida de un cliente externo en Azure Storage, así como la salida dentro de Azure. En consecuencia, este número no refleja las salidas facturables.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Latencia del servidor correcta|Milisegundos|Promedio|La latencia media utilizada por Azure Storage para procesar una solicitud correcta, en milisegundos. Este valor no incluye la latencia de red especificada en AverageE2ELatency.|GeoType, ApiName, Authentication|
|SuccessE2ELatency|Latencia de E2E correcta|Milisegundos|Promedio|La latencia media de un extremo a otro de las solicitudes correctas realizadas a un servicio de almacenamiento o a la operación de API especificada, en milisegundos. Este valor incluye el tiempo de procesamiento requerido dentro de Azure Storage para leer la solicitud, enviar la respuesta y recibir la confirmación de la respuesta.|GeoType, ApiName, Authentication|
|Disponibilidad|Disponibilidad|Porcentaje|Promedio|El porcentaje de disponibilidad para el servicio de almacenamiento o la operación de API especificada. Para calcular la disponibilidad hay que tomar el valor deTotalBillableRequests y dividirlo por el número de solicitudes aplicables, incluidas aquellas que generaron errores inesperados. Todos los errores inesperados reducen la disponibilidad del servicio de almacenamiento o de la operación de API especificada.|GeoType, ApiName, Authentication|

## <a name="microsoftstoragestorageaccountsblobservices"></a>Microsoft.Storage/storageAccounts/blobServices

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|BlobCapacity|Capacidad de blob|Bytes|Promedio|La cantidad de almacenamiento que usa Blob service en la cuenta de Storage, en bytes.|BlobType, Tier|
|BlobCount|Recuento de blobs|Count|Total|El número de blob en Blob service en la cuenta de Storage.|BlobType|       |BlobCount|Recuento de blobs|Count|Promedio|El número de blob en Blob service en la cuenta de Storage.|BlobType, Tier|
|ContainerCount|Recuento de contenedores de blobs|Count|Promedio|Cantidad de contenedores en Blob service en la cuenta de Storage.|Sin dimensiones|
|IndexCapacity|Capacidad de índice|Bytes|Promedio|Cantidad de almacenamiento que usa el índice (jerárquico) de ADLS Gen2 en bytes.|Sin dimensiones|
|Transacciones|Transacciones|Count|Total|El número de solicitudes realizadas a un servicio de almacenamiento o la operación de API especificada. Este número incluye solicitudes correctas y con errores, así como las solicitudes que generaron errores. Use la dimensión ResponseType para el número de diferentes tipos de respuesta.|ResponseType, GeoType, ApiName, Authentication|
|Entrada|Entrada|Bytes|Total|La cantidad de datos de entrada, en bytes. Este número incluye la entradas desde un cliente externo en Azure Storage, así como la entrada dentro de Azure.|GeoType, ApiName, Authentication|
|Salida|Salida|Bytes|Total|La cantidad de datos de salida, en bytes. Este número incluye la salida de un cliente externo en Azure Storage, así como la salida dentro de Azure. En consecuencia, este número no refleja las salidas facturables.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Latencia del servidor correcta|Milisegundos|Promedio|La latencia media utilizada por Azure Storage para procesar una solicitud correcta, en milisegundos. Este valor no incluye la latencia de red especificada en AverageE2ELatency.|GeoType, ApiName, Authentication|
|SuccessE2ELatency|Latencia de E2E correcta|Milisegundos|Promedio|La latencia media de un extremo a otro de las solicitudes correctas realizadas a un servicio de almacenamiento o a la operación de API especificada, en milisegundos. Este valor incluye el tiempo de procesamiento requerido dentro de Azure Storage para leer la solicitud, enviar la respuesta y recibir la confirmación de la respuesta.|GeoType, ApiName, Authentication|
|Disponibilidad|Disponibilidad|Porcentaje|Promedio|El porcentaje de disponibilidad para el servicio de almacenamiento o la operación de API especificada. Para calcular la disponibilidad hay que tomar el valor deTotalBillableRequests y dividirlo por el número de solicitudes aplicables, incluidas aquellas que generaron errores inesperados. Todos los errores inesperados reducen la disponibilidad del servicio de almacenamiento o de la operación de API especificada.|GeoType, ApiName, Authentication|

## <a name="microsoftstoragestorageaccountsfileservices"></a>Microsoft.Storage/storageAccounts/fileServices

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|FileCapacity|Capacidad de archivo|Bytes|Promedio|La cantidad de almacenamiento que usa el servicio de archivos en la cuenta de Storage, en bytes.|Sin dimensiones|
|FileCount|Recuento de archivos|Count|Promedio|Número de archivo en el servicio de archivos de la cuenta de Storage.|Sin dimensiones|
|FileShareCount|Recuento de recursos compartidos de archivos|Count|Promedio|Cantidad de recursos compartidos de archivos del servicio de archivos de la cuenta de Storage.|Sin dimensiones|
|Transacciones|Transacciones|Count|Total|El número de solicitudes realizadas a un servicio de almacenamiento o la operación de API especificada. Este número incluye solicitudes correctas y con errores, así como las solicitudes que generaron errores. Use la dimensión ResponseType para el número de diferentes tipos de respuesta.|ResponseType, GeoType, ApiName, Authentication|
|Entrada|Entrada|Bytes|Total|La cantidad de datos de entrada, en bytes. Este número incluye la entradas desde un cliente externo en Azure Storage, así como la entrada dentro de Azure.|GeoType, ApiName, Authentication|
|Salida|Salida|Bytes|Total|La cantidad de datos de salida, en bytes. Este número incluye la salida de un cliente externo en Azure Storage, así como la salida dentro de Azure. En consecuencia, este número no refleja las salidas facturables.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Latencia del servidor correcta|Milisegundos|Promedio|La latencia media utilizada por Azure Storage para procesar una solicitud correcta, en milisegundos. Este valor no incluye la latencia de red especificada en AverageE2ELatency.|GeoType, ApiName, Authentication|
|SuccessE2ELatency|Latencia de E2E correcta|Milisegundos|Promedio|La latencia media de un extremo a otro de las solicitudes correctas realizadas a un servicio de almacenamiento o a la operación de API especificada, en milisegundos. Este valor incluye el tiempo de procesamiento requerido dentro de Azure Storage para leer la solicitud, enviar la respuesta y recibir la confirmación de la respuesta.|GeoType, ApiName, Authentication|
|Disponibilidad|Disponibilidad|Porcentaje|Promedio|El porcentaje de disponibilidad para el servicio de almacenamiento o la operación de API especificada. Para calcular la disponibilidad hay que tomar el valor deTotalBillableRequests y dividirlo por el número de solicitudes aplicables, incluidas aquellas que generaron errores inesperados. Todos los errores inesperados reducen la disponibilidad del servicio de almacenamiento o de la operación de API especificada.|GeoType, ApiName, Authentication|

## <a name="microsoftstoragestorageaccountsqueueservices"></a>Microsoft.Storage/storageAccounts/queueServices

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|QueueCapacity|Capacidad de la cola|Bytes|Promedio|La cantidad de almacenamiento que usa la instancia de Queue service en la cuenta de Storage, en bytes.|Sin dimensiones|
|QueueCount|Recuento de colas|Count|Promedio|Número de cola en la instancia de Queue service de la cuenta de Storage.|Sin dimensiones|
|QueueMessageCount|Recuento de mensajes en cola|Count|Promedio|Número aproximado de mensajes en cola en el servicio de colas de la cuenta de Storage.|Sin dimensiones|
|Transacciones|Transacciones|Count|Total|El número de solicitudes realizadas a un servicio de almacenamiento o la operación de API especificada. Este número incluye solicitudes correctas y con errores, así como las solicitudes que generaron errores. Use la dimensión ResponseType para el número de diferentes tipos de respuesta.|ResponseType, GeoType, ApiName, Authentication|
|Entrada|Entrada|Bytes|Total|La cantidad de datos de entrada, en bytes. Este número incluye la entradas desde un cliente externo en Azure Storage, así como la entrada dentro de Azure.|GeoType, ApiName, Authentication|
|Salida|Salida|Bytes|Total|La cantidad de datos de salida, en bytes. Este número incluye la salida de un cliente externo en Azure Storage, así como la salida dentro de Azure. En consecuencia, este número no refleja las salidas facturables.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Latencia del servidor correcta|Milisegundos|Promedio|La latencia media utilizada por Azure Storage para procesar una solicitud correcta, en milisegundos. Este valor no incluye la latencia de red especificada en AverageE2ELatency.|GeoType, ApiName, Authentication|
|SuccessE2ELatency|Latencia de E2E correcta|Milisegundos|Promedio|La latencia media de un extremo a otro de las solicitudes correctas realizadas a un servicio de almacenamiento o a la operación de API especificada, en milisegundos. Este valor incluye el tiempo de procesamiento requerido dentro de Azure Storage para leer la solicitud, enviar la respuesta y recibir la confirmación de la respuesta.|GeoType, ApiName, Authentication|
|Disponibilidad|Disponibilidad|Porcentaje|Promedio|El porcentaje de disponibilidad para el servicio de almacenamiento o la operación de API especificada. Para calcular la disponibilidad hay que tomar el valor deTotalBillableRequests y dividirlo por el número de solicitudes aplicables, incluidas aquellas que generaron errores inesperados. Todos los errores inesperados reducen la disponibilidad del servicio de almacenamiento o de la operación de API especificada.|GeoType, ApiName, Authentication|

## <a name="microsoftstoragestorageaccountstableservices"></a>Microsoft.Storage/storageAccounts/tableServices

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|TableCapacity|Capacidad de la tabla|Bytes|Promedio|Cantidad de almacenamiento que usa el servicio de tablas en la cuenta de Storage, en bytes.|Sin dimensiones|
|TableCount|Número de tablas|Count|Promedio|Número de tabla en la instancia de Table service de la cuenta de Storage.|Sin dimensiones|
|TableEntityCount|Recuento de entidades de tabla|Count|Promedio|Cantidad de entidades de tabla en el servicio de tablas de la cuenta de Storage.|Sin dimensiones|
|Transacciones|Transacciones|Count|Total|El número de solicitudes realizadas a un servicio de almacenamiento o la operación de API especificada. Este número incluye solicitudes correctas y con errores, así como las solicitudes que generaron errores. Use la dimensión ResponseType para el número de diferentes tipos de respuesta.|ResponseType, GeoType, ApiName, Authentication|
|Entrada|Entrada|Bytes|Total|La cantidad de datos de entrada, en bytes. Este número incluye la entradas desde un cliente externo en Azure Storage, así como la entrada dentro de Azure.|GeoType, ApiName, Authentication|
|Salida|Salida|Bytes|Total|La cantidad de datos de salida, en bytes. Este número incluye la salida de un cliente externo en Azure Storage, así como la salida dentro de Azure. En consecuencia, este número no refleja las salidas facturables.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Latencia del servidor correcta|Milisegundos|Promedio|La latencia media utilizada por Azure Storage para procesar una solicitud correcta, en milisegundos. Este valor no incluye la latencia de red especificada en AverageE2ELatency.|GeoType, ApiName, Authentication|
|SuccessE2ELatency|Latencia de E2E correcta|Milisegundos|Promedio|La latencia media de un extremo a otro de las solicitudes correctas realizadas a un servicio de almacenamiento o a la operación de API especificada, en milisegundos. Este valor incluye el tiempo de procesamiento requerido dentro de Azure Storage para leer la solicitud, enviar la respuesta y recibir la confirmación de la respuesta.|GeoType, ApiName, Authentication|
|Disponibilidad|Disponibilidad|Porcentaje|Promedio|El porcentaje de disponibilidad para el servicio de almacenamiento o la operación de API especificada. Para calcular la disponibilidad hay que tomar el valor deTotalBillableRequests y dividirlo por el número de solicitudes aplicables, incluidas aquellas que generaron errores inesperados. Todos los errores inesperados reducen la disponibilidad del servicio de almacenamiento o de la operación de API especificada.|GeoType, ApiName, Authentication|

## <a name="microsoftstoragesyncstoragesyncservices"></a>microsoft.storagesync/storageSyncServices

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|ServerSyncSessionResult|Resultado de la sesión de sincronización|Count|Promedio|Métrica de que los registros de un valor de 1 cada vez el punto de conexión de servidor correctamente completa una sesión de sincronización con el punto de conexión en la nube|SyncDirection SyncGroupName, ServerEndpointName,|
|StorageSyncSyncSessionAppliedFilesCount|Archivos sincronizados|Count|Total|Recuento de archivos sincronizados|SyncDirection SyncGroupName, ServerEndpointName,|
|StorageSyncSyncSessionPerItemErrorsCount|Archivos que no se están sincronizando|Count|Total|No se pudo sincronizar el recuento de archivos|SyncDirection SyncGroupName, ServerEndpointName,|
|StorageSyncBatchTransferredFileBytes|Bytes sincronizados|Bytes|Total|Tamaño total del archivo transferidos para las sesiones de sincronización|SyncDirection SyncGroupName, ServerEndpointName,|
|StorageSyncServerHeartbeat|Estado de conexión del servidor|Count|Máximo|Registros de métrica que los registros de un valor de 1 cada vez el servidor registrado correctamente un latido con el punto de conexión en la nube|nombreDeServidor|
|StorageSyncRecallIOTotalSizeBytes|Recuperación de niveles de la nube|Bytes|Total|Tamaño total de datos recuperados por el servidor|nombreDeServidor|

## <a name="microsoftstreamanalyticsstreamingjobs"></a>Microsoft.StreamAnalytics/streamingjobs

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|ResourceUtilization|SU % uso|Porcentaje|Máximo|% de utilización de la unidad de streaming|LogicalName, PartitionId|
|InputEvents|Eventos de entrada|Count|Total|Eventos de entrada|LogicalName, PartitionId|
|InputEventBytes|Bytes del evento de entrada|Bytes|Total|Bytes del evento de entrada|LogicalName, PartitionId|
|LateInputEvents|Eventos de entrada retrasada|Count|Total|Eventos de entrada retrasada|LogicalName, PartitionId|
|OutputEvents|Eventos de salida|Count|Total|Eventos de salida|LogicalName, PartitionId|
|ConversionErrors|Errores de conversión de datos|Count|Total|Errores de conversión de datos|LogicalName, PartitionId|
|Errors|Errores en runtime|Count|Total|Errores en runtime|LogicalName, PartitionId|
|DroppedOrAdjustedEvents|Eventos desordenados|Count|Total|Eventos desordenados|LogicalName, PartitionId|
|AMLCalloutRequests|Solicitudes de función|Count|Total|Solicitudes de funciones|LogicalName, PartitionId|
|AMLCalloutFailedRequests|Solicitudes de función con errores|Count|Total|Solicitudes de funciones con errores|LogicalName, PartitionId|
|AMLCalloutInputEvents|Eventos de función|Count|Total|Eventos de funciones|LogicalName, PartitionId|
|DeserializationError|Errores de deserialización de entrada|Count|Total|Errores de deserialización de entrada|LogicalName, PartitionId|
|EarlyInputEvents|Primeros eventos de entrada|Count|Total|Primeros eventos de entrada|LogicalName, PartitionId|
|OutputWatermarkDelaySeconds|Retraso de la marca de agua|Segundos|Máximo|Retraso de la marca de agua|LogicalName, PartitionId|
|InputEventsSourcesBacklogged|Eventos de entrada pendientes|Count|Máximo|Eventos de entrada pendientes|LogicalName, PartitionId|
|InputEventsSourcesPerSecond|Orígenes de entrada recibidos|Count|Total|Orígenes de entrada recibidos|LogicalName, PartitionId|

## <a name="microsofttimeseriesinsightsenvironments"></a>Microsoft.TimeSeriesInsights/environments

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|IngressReceivedMessages|Mensajes de entrada recibidos|Count|Total|Número de mensajes leídos desde todos los orígenes de eventos de Event Hubs o IoT Hub|Sin dimensiones|
|IngressReceivedInvalidMessages|Mensajes no válidos de entrada recibidos|Count|Total|Número de mensajes no válidos leídos desde todos los orígenes de eventos de Event Hubs o IoT Hub|Sin dimensiones|
|IngressReceivedBytes|Bytes de entrada recibidos|Bytes|Total|Número de bytes leídos desde todos los orígenes de eventos|Sin dimensiones|
|IngressStoredBytes|Bytes de entrada almacenados|Bytes|Total|Tamaño total de los eventos almacenados correctamente y disponibles para su consulta|Sin dimensiones|
|IngressStoredEvents|Eventos de entrada almacenados|Count|Total|Número de eventos acoplados que se procesaron correctamente y que están disponibles para su consulta|Sin dimensiones|
|IngressReceivedMessagesTimeLag|Tiempo de retardo de los mensajes de entrada recibidos|Segundos|Máximo|Diferencia entre el momento en el que el mensaje se pone en la cola en el origen del evento y el momento en que se procesa en la entrada|Sin dimensiones|
|IngressReceivedMessagesCountLag|Intervalo de recuento de los mensajes de entrada recibidos|Count|Promedio|Diferencia entre el número de secuencia del último mensaje en cola en la partición del origen del evento y el del mensaje que se está procesando en la entrada|Sin dimensiones|
|WarmStorageMaxProperties|Propiedades de máximo de almacenamiento en caliente|Count|Máximo|Número máximo permitido por el entorno de SKU S1/S2 y número máximo de propiedades permitido por Store en caliente de PAYG de SKU de propiedades que se usan|Sin dimensiones|
|WarmStorageUsedProperties|Propiedades utilizadas con almacenamiento en caliente |Count|Máximo|Número de propiedades utilizados por el entorno de SKU S1/S2 y número de propiedades utilizados por Store en caliente de PAYG de SKU|Sin dimensiones|

## <a name="microsofttimeseriesinsightsenvironmentseventsources"></a>Microsoft.TimeSeriesInsights/environments/eventsources

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|IngressReceivedMessages|Mensajes de entrada recibidos|Count|Total|Número de mensajes leídos desde el origen del evento|Sin dimensiones|
|IngressReceivedInvalidMessages|Mensajes no válidos de entrada recibidos|Count|Total|Número de mensajes leídos no válidos desde el origen del evento|Sin dimensiones|
|IngressReceivedBytes|Bytes de entrada recibidos|Bytes|Total|Número de bytes leídos desde el origen del evento|Sin dimensiones|
|IngressStoredBytes|Bytes de entrada almacenados|Bytes|Total|Tamaño total de los eventos almacenados correctamente y disponibles para su consulta|Sin dimensiones|
|IngressStoredEvents|Eventos de entrada almacenados|Count|Total|Número de eventos acoplados que se procesaron correctamente y que están disponibles para su consulta|Sin dimensiones|
|IngressReceivedMessagesTimeLag|Tiempo de retardo de los mensajes de entrada recibidos|Segundos|Máximo|Diferencia entre el momento en el que el mensaje se pone en la cola en el origen del evento y el momento en que se procesa en la entrada|Sin dimensiones|
|IngressReceivedMessagesCountLag|Intervalo de recuento de los mensajes de entrada recibidos|Count|Promedio|Diferencia entre el número de secuencia del último mensaje en cola en la partición del origen del evento y el del mensaje que se está procesando en la entrada|Sin dimensiones|
|WarmStorageMaxProperties|Propiedades de máximo de almacenamiento en caliente|Count|Máximo|Número máximo permitido por el entorno de SKU S1/S2 y número máximo de propiedades permitido por Store en caliente de PAYG de SKU de propiedades que se usan|Sin dimensiones|
|WarmStorageUsedProperties|Propiedades utilizadas con almacenamiento en caliente |Count|Máximo|Número de propiedades utilizados por el entorno de SKU S1/S2 y número de propiedades utilizados por Store en caliente de PAYG de SKU|Sin dimensiones|

## <a name="microsoftvmwarecloudsimplevirtualmachines"></a>Microsoft.VMwareCloudSimple/virtualMachines

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|DiskReadBytesPerSecond|Bytes de lectura de disco/segundo|BytesPerSecond|Promedio|Rendimiento de disco promedio debido a las operaciones de lectura durante el período de ejemplo.|Sin dimensiones|
|DiskWriteBytesPerSecond|Bytes de escritura en disco/segundo|BytesPerSecond|Promedio|Rendimiento de disco promedio debido a las operaciones de escritura durante el período de ejemplo.|Sin dimensiones|
|Bytes de lectura en disco|Bytes de lectura en disco|Bytes|Total|Rendimiento total del disco debido a las operaciones de lectura durante el período de ejemplo.|Sin dimensiones|
|Bytes de escritura en disco|Bytes de escritura de disco|Bytes|Total|Rendimiento total del disco debido a las operaciones de escritura durante el período de ejemplo.|Sin dimensiones|
|DiskReadOperations|Operaciones de lectura de disco|Count|Total|El número de E/S de las operaciones de lectura en el período de ejemplo anterior. Tenga en cuenta que estas operaciones pueden ser variable tamaño.|Sin dimensiones|
|DiskWriteOperations|Operaciones de escritura de disco|Count|Total|El número de E/S de las operaciones de escritura en el período de ejemplo anterior. Tenga en cuenta que estas operaciones pueden ser variable tamaño.|Sin dimensiones|
|Operaciones de lectura en disco por segundo|Operaciones de lectura de disco por segundo|CountPerSecond|Promedio|Las operaciones de lectura promedio de E/S en el período de ejemplo anterior. Tenga en cuenta que estas operaciones pueden ser variable tamaño.|Sin dimensiones|
|Operaciones de escritura en disco por segundo|Operaciones de escritura por segundo en disco|CountPerSecond|Promedio|El número promedio de E/S las operaciones de escritura en el período de ejemplo anterior. Tenga en cuenta que estas operaciones pueden ser variable tamaño.|Sin dimensiones|
|DiskReadLatency|Latencia de lectura de disco|Milisegundos|Promedio|Total de latencia de lectura. Latencias de lectura de la suma de los dispositivos y kernel.|Sin dimensiones|
|DiskWriteLatency|Latencia de escritura en disco|Milisegundos|Promedio|Latencia total de escritura. La suma del dispositivo y kernel latencias de escritura.|Sin dimensiones|
|NetworkInBytesPerSecond|Red en Bytes/seg.|BytesPerSecond|Promedio|Rendimiento medio de red para el tráfico recibido.|Sin dimensiones|
|NetworkOutBytesPerSecond|Salida de Bytes/seg. de red|BytesPerSecond|Promedio|Rendimiento de promedio de red para tráfico transmitido.|Sin dimensiones|
|Entrada de red|Red interna|Bytes|Total|Rendimiento de red total para el tráfico recibido.|Sin dimensiones|
|Salida de red|Red interna|Bytes|Total|Rendimiento de red total para tráfico transmitido.|Sin dimensiones|
|MemoryUsed|Memoria usada|Bytes|Promedio|La cantidad de memoria de la máquina que está usando la máquina virtual.|Sin dimensiones|
|MemoryGranted|Memoria concedida|Bytes|Promedio|La cantidad de memoria que le ha concedido a la máquina virtual del host. Memoria no se concede al host hasta que se toca una vez concedido memoria pueden intercambiar o y crecido rápidamente hasta alcanzar ausente si VMkernel necesita la memoria.|Sin dimensiones|
|MemoryActive|Memoria activo|Bytes|Promedio|La cantidad de memoria utilizada por la máquina virtual en la últimos pequeña ventana de tiempo. Este es el número de "true" de la cantidad de memoria la máquina virtual actualmente tiene necesidad de. Memoria adicional y sin usar se puede intercambiar o crecido rápidamente hasta alcanzar sin que ello afecte al rendimiento del invitado.|Sin dimensiones|
|Porcentaje de CPU|Porcentaje de CPU|Porcentaje|Promedio|El uso de CPU. Este valor se notifica con 100%, que representa todos los núcleos de procesador en el sistema. Por ejemplo, una máquina virtual de 2 vías con el 50% de un sistema de cuatro núcleos completamente usa dos núcleos.|Sin dimensiones|
|PercentageCpuReady|Uso de CPU de porcentaje|Milisegundos|Total|Ready time está el tiempo transcurrido esperando CPU esté disponible en el último intervalo de actualización.|Sin dimensiones|

## <a name="microsoftwebserverfarms"></a>Microsoft.Web/serverfarms

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|CpuPercentage|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de CPU|Instancia|
|MemoryPercentage|Porcentaje de memoria|Porcentaje|Promedio|Porcentaje de memoria|Instancia|
|DiskQueueLength|Longitud de la cola del disco|Count|Promedio|Longitud de la cola del disco|Instancia|
|HttpQueueLength|Longitud de la cola HTTP|Count|Promedio|Longitud de la cola HTTP|Instancia|
|BytesReceived|Entrada de datos|Bytes|Total|Datos entrantes|Instancia|
|BytesSent|Datos salientes|Bytes|Total|Datos salientes|Instancia|

## <a name="microsoftwebsites-excluding-functions"></a>Microsoft.Web/sites (se excluye Functions)

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|CpuTime|Tiempo de CPU|Segundos|Total|Tiempo de CPU|Instancia|
|Solicitudes|Solicitudes|Count|Total|Solicitudes|Instancia|
|BytesReceived|Entrada de datos|Bytes|Total|Datos entrantes|Instancia|
|BytesSent|Datos salientes|Bytes|Total|Datos salientes|Instancia|
|Http101|HTTP 101|Count|Total|HTTP 101|Instancia|
|Http2xx|Http 2xx|Count|Total|Http 2xx|Instancia|
|Http3xx|Http 3xx|Count|Total|Http 3xx|Instancia|
|Http401|Http 401|Count|Total|Http 401|Instancia|
|Http403|Http 403|Count|Total|Http 403|Instancia|
|Http404|Http 404|Count|Total|Http 404|Instancia|
|Http406|Http 406|Count|Total|Http 406|Instancia|
|Http4xx|Http 4xx|Count|Total|Http 4xx|Instancia|
|Http5xx|Errores del servidor Http|Count|Total|Errores del servidor Http|Instancia|
|MemoryWorkingSet|Espacio de trabajo de memoria|Bytes|Promedio|Espacio de trabajo de memoria|Instancia|
|AverageMemoryWorkingSet|Espacio medio de trabajo de memoria|Bytes|Promedio|Espacio medio de trabajo de memoria|Instancia|
|AverageResponseTime|Tiempo de respuesta promedio|Segundos|Promedio|Tiempo medio de respuesta|Instancia|
|AppConnections|Conexiones|Count|Promedio|Conexiones|Instancia|
|Indicadores|Recuento de identificadores|Count|Promedio|Recuento de identificadores|Instancia|
|Subprocesos|Número de subprocesos|Count|Promedio|Número de subprocesos|Instancia|
|PrivateBytes|Bytes privados|Bytes|Promedio|Bytes privados|Instancia|
|IoReadBytesPerSecond|Bytes de lectura de E/S por segundo|BytesPerSecond|Total|Bytes de lectura de E/S por segundo|Instancia|
|IoReadBytesPerSecond|Bytes de escritura de E/S por segundo|BytesPerSecond|Total|Bytes de escritura de E/S por segundo|Instancia|
|IoOtherBytesPerSecond|Otros bytes de E/S por segundo|BytesPerSecond|Total|Otros bytes de E/S por segundo|Instancia|
|IoReadOperationsPerSecond|Operaciones de lectura de E/S por segundo|BytesPerSecond|Total|Operaciones de lectura de E/S por segundo|Instancia|
|IoWriteOperationsPerSecond|Operaciones de escritura de E/S por segundo|BytesPerSecond|Total|Operaciones de escritura de E/S por segundo|Instancia|
|IoOtherOperationsPerSecond|Otras operaciones de E/S por segundo|BytesPerSecond|Total|Otras operaciones de E/S por segundo|Instancia|
|RequestsInApplicationQueue|Solicitudes en cola de la aplicación|Count|Promedio|Solicitudes en cola de la aplicación|Instancia|
|CurrentAssemblies|Ensamblados actuales|Count|Promedio|Ensamblados actuales|Instancia|
|TotalAppDomains|Dominios de aplicación totales|Count|Promedio|Dominios de aplicación totales|Instancia|
|TotalAppDomainsUnloaded|Dominios de aplicación totales descargados|Count|Promedio|Dominios de aplicación totales descargados|Instancia|
|Gen0Collections|Recolección de elementos no utilizados de gen. 0|Count|Total|Recolección de elementos no utilizados de gen. 0|Instancia|
|Gen1Collections|Recolección de elementos no utilizados de gen. 1|Count|Total|Recolección de elementos no utilizados de gen. 1|Instancia|
|Gen2Collections|Recolección de elementos no utilizados de gen. 2|Count|Total|Recolección de elementos no utilizados de gen. 2|Instancia|

## <a name="microsoftwebsites-functions"></a>Microsoft.Web/sites (Functions)

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|BytesReceived|Entrada de datos|Bytes|Total|Datos entrantes|Instancia|
|BytesSent|Datos salientes|Bytes|Total|Datos salientes|Instancia|
|Http5xx|Errores del servidor Http|Count|Total|Errores del servidor Http|Instancia|
|MemoryWorkingSet|Espacio de trabajo de memoria|Bytes|Promedio|Espacio de trabajo de memoria|Instancia|
|AverageMemoryWorkingSet|Espacio medio de trabajo de memoria|Bytes|Promedio|Espacio medio de trabajo de memoria|Instancia|
|FunctionExecutionUnits|Unidades de ejecución de la función|MB / milisegundos|Total|[Unidades de ejecución de función](https://github.com/Azure/Azure-Functions/wiki/Consumption-Plan-Cost-Billing-FAQ#how-can-i-view-graphs-of-execution-count-and-gb-seconds)|Instancia|
|FunctionExecutionCount|Recuento de la ejecución de la función|Count|Total|Recuento de la ejecución de la función|Instancia|
|PrivateBytes|Bytes privados|Bytes|Promedio|Bytes privados|Instancia|
|IoReadBytesPerSecond|Bytes de lectura de E/S por segundo|BytesPerSecond|Total|Bytes de lectura de E/S por segundo|Instancia|
|IoReadBytesPerSecond|Bytes de escritura de E/S por segundo|BytesPerSecond|Total|Bytes de escritura de E/S por segundo|Instancia|
|IoOtherBytesPerSecond|Otros bytes de E/S por segundo|BytesPerSecond|Total|Otros bytes de E/S por segundo|Instancia|
|IoReadOperationsPerSecond|Operaciones de lectura de E/S por segundo|BytesPerSecond|Total|Operaciones de lectura de E/S por segundo|Instancia|
|IoWriteOperationsPerSecond|Operaciones de escritura de E/S por segundo|BytesPerSecond|Total|Operaciones de escritura de E/S por segundo|Instancia|
|IoOtherOperationsPerSecond|Otras operaciones de E/S por segundo|BytesPerSecond|Total|Otras operaciones de E/S por segundo|Instancia|
|RequestsInApplicationQueue|Solicitudes en cola de la aplicación|Count|Promedio|Solicitudes en cola de la aplicación|Instancia|
|CurrentAssemblies|Ensamblados actuales|Count|Promedio|Ensamblados actuales|Instancia|
|TotalAppDomains|Dominios de aplicación totales|Count|Promedio|Dominios de aplicación totales|Instancia|
|TotalAppDomainsUnloaded|Dominios de aplicación totales descargados|Count|Promedio|Dominios de aplicación totales descargados|Instancia|
|Gen0Collections|Recolección de elementos no utilizados de gen. 0|Count|Total|Recolección de elementos no utilizados de gen. 0|Instancia|
|Gen1Collections|Recolección de elementos no utilizados de gen. 1|Count|Total|Recolección de elementos no utilizados de gen. 1|Instancia|
|Gen2Collections|Recolección de elementos no utilizados de gen. 2|Count|Total|Recolección de elementos no utilizados de gen. 2|Instancia|

## <a name="microsoftwebsitesslots"></a>Microsoft.Web/sites/slots

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|CpuTime|Tiempo de CPU|Segundos|Total|Tiempo de CPU|Instancia|
|Solicitudes|Solicitudes|Count|Total|Solicitudes|Instancia|
|BytesReceived|Entrada de datos|Bytes|Total|Datos entrantes|Instancia|
|BytesSent|Datos salientes|Bytes|Total|Datos salientes|Instancia|
|Http101|HTTP 101|Count|Total|HTTP 101|Instancia|
|Http2xx|Http 2xx|Count|Total|Http 2xx|Instancia|
|Http3xx|Http 3xx|Count|Total|Http 3xx|Instancia|
|Http401|Http 401|Count|Total|Http 401|Instancia|
|Http403|Http 403|Count|Total|Http 403|Instancia|
|Http404|Http 404|Count|Total|Http 404|Instancia|
|Http406|Http 406|Count|Total|Http 406|Instancia|
|Http4xx|Http 4xx|Count|Total|Http 4xx|Instancia|
|Http5xx|Errores del servidor Http|Count|Total|Errores del servidor Http|Instancia|
|MemoryWorkingSet|Espacio de trabajo de memoria|Bytes|Promedio|Espacio de trabajo de memoria|Instancia|
|AverageMemoryWorkingSet|Espacio medio de trabajo de memoria|Bytes|Promedio|Espacio medio de trabajo de memoria|Instancia|
|AverageResponseTime|Tiempo de respuesta promedio|Segundos|Promedio|Tiempo medio de respuesta|Instancia|
|FunctionExecutionUnits|Unidades de ejecución de la función|Count|Total|Unidades de ejecución de la función|Instancia|
|FunctionExecutionCount|Recuento de la ejecución de la función|Count|Total|Recuento de la ejecución de la función|Instancia|
|AppConnections|Conexiones|Count|Promedio|Conexiones|Instancia|
|Indicadores|Recuento de identificadores|Count|Promedio|Recuento de identificadores|Instancia|
|Subprocesos|Número de subprocesos|Count|Promedio|Número de subprocesos|Instancia|
|PrivateBytes|Bytes privados|Bytes|Promedio|Bytes privados|Instancia|
|IoReadBytesPerSecond|Bytes de lectura de E/S por segundo|BytesPerSecond|Total|Bytes de lectura de E/S por segundo|Instancia|
|IoReadBytesPerSecond|Bytes de escritura de E/S por segundo|BytesPerSecond|Total|Bytes de escritura de E/S por segundo|Instancia|
|IoOtherBytesPerSecond|Otros bytes de E/S por segundo|BytesPerSecond|Total|Otros bytes de E/S por segundo|Instancia|
|IoReadOperationsPerSecond|Operaciones de lectura de E/S por segundo|BytesPerSecond|Total|Operaciones de lectura de E/S por segundo|Instancia|
|IoWriteOperationsPerSecond|Operaciones de escritura de E/S por segundo|BytesPerSecond|Total|Operaciones de escritura de E/S por segundo|Instancia|
|IoOtherOperationsPerSecond|Otras operaciones de E/S por segundo|BytesPerSecond|Total|Otras operaciones de E/S por segundo|Instancia|
|RequestsInApplicationQueue|Solicitudes en cola de la aplicación|Count|Promedio|Solicitudes en cola de la aplicación|Instancia|
|CurrentAssemblies|Ensamblados actuales|Count|Promedio|Ensamblados actuales|Instancia|
|TotalAppDomains|Dominios de aplicación totales|Count|Promedio|Dominios de aplicación totales|Instancia|
|TotalAppDomainsUnloaded|Dominios de aplicación totales descargados|Count|Promedio|Dominios de aplicación totales descargados|Instancia|
|Gen0Collections|Recolección de elementos no utilizados de gen. 0|Count|Total|Recolección de elementos no utilizados de gen. 0|Instancia|
|Gen1Collections|Recolección de elementos no utilizados de gen. 1|Count|Total|Recolección de elementos no utilizados de gen. 1|Instancia|
|Gen2Collections|Recolección de elementos no utilizados de gen. 2|Count|Total|Recolección de elementos no utilizados de gen. 2|Instancia|

## <a name="microsoftwebhostingenvironmentsmultirolepools"></a>Microsoft.Web/hostingEnvironments/multiRolePools

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|Solicitudes|Solicitudes|Count|Total|Solicitudes|Instancia|
|BytesReceived|Entrada de datos|Bytes|Total|Datos entrantes|Instancia|
|BytesSent|Datos salientes|Bytes|Total|Datos salientes|Instancia|
|Http101|HTTP 101|Count|Total|HTTP 101|Instancia|
|Http2xx|Http 2xx|Count|Total|Http 2xx|Instancia|
|Http3xx|Http 3xx|Count|Total|Http 3xx|Instancia|
|Http401|Http 401|Count|Total|Http 401|Instancia|
|Http403|Http 403|Count|Total|Http 403|Instancia|
|Http404|Http 404|Count|Total|Http 404|Instancia|
|Http406|Http 406|Count|Total|Http 406|Instancia|
|Http4xx|Http 4xx|Count|Total|Http 4xx|Instancia|
|Http5xx|Errores del servidor Http|Count|Total|Errores del servidor Http|Instancia|
|AverageResponseTime|Tiempo de respuesta promedio|Segundos|Promedio|Tiempo medio de respuesta|Instancia|
|CpuPercentage|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de CPU|Instancia|
|MemoryPercentage|Porcentaje de memoria|Porcentaje|Promedio|Porcentaje de memoria|Instancia|
|DiskQueueLength|Longitud de la cola del disco|Count|Promedio|Longitud de la cola del disco|Instancia|
|HttpQueueLength|Longitud de la cola HTTP|Count|Promedio|Longitud de la cola HTTP|Instancia|
|ActiveRequests|Solicitudes activas|Count|Total|Solicitudes activas|Instancia|
|TotalFrontEnds|Servidores front-end totales|Count|Promedio|Servidores front-end totales|Sin dimensiones|
|SmallAppServicePlanInstances|Trabajos de planes de App Service pequeños|Count|Promedio|Trabajos de planes de App Service pequeños|Sin dimensiones|
|MediumAppServicePlanInstances|Trabajos medianos del plan de App Service|Count|Media|Trabajos de planes de App Service medianos|Sin dimensiones|
|LargeAppServicePlanInstances|Trabajos de planes de App Service grandes|Count|Promedio|Trabajos de planes de App Service grandes|Sin dimensiones|

## <a name="microsoftwebhostingenvironmentsworkerpools"></a>Microsoft.Web/hostingEnvironments/workerPools

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|WorkersTotal|Trabajos totales|Count|Promedio|Trabajos totales|Sin dimensiones|
|WorkersAvailable|Trabajos disponibles|Count|Promedio|Trabajos disponibles|Sin dimensiones|
|WorkersUsed|Trabajos usados|Count|Promedio|Trabajos usados|Sin dimensiones|
|CpuPercentage|Porcentaje de CPU|Porcentaje|Promedio|Porcentaje de CPU|Instancia|
|MemoryPercentage|Porcentaje de memoria|Porcentaje|Promedio|Porcentaje de memoria|Instancia|

## <a name="next-steps"></a>Pasos siguientes
* [Lea información sobre las métricas en Azure Monitor](data-platform.md)
* [Creación de alertas basadas en métricas](alerts-overview.md)
* [Exportación de métricas a cuentas de almacenamiento, Event Hubs o Log Analytics](diagnostic-logs-overview.md)
