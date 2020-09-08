---
title: Auditoría de las consultas en las consultas de registro de Azure Monitor
description: Detalles de lo registros de auditoría de las consultas de registro, que proporcionan datos de telemetría sobre la ejecución de consultas de registro en Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/25/2020
ms.openlocfilehash: cb38dcba2f61a432decb56164b816688ad3192d8
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88893642"
---
# <a name="audit-queries-in-azure-monitor-logs-preview"></a>Auditoría de las consultas en los registros de Azure Monitor (versión preliminar)
Los registros de auditoría de las consultas de registro proporcionan datos de telemetría sobre la ejecución de consultas de registro en Azure Monitor. Esto incluye información como cuándo se ejecutó una consulta, quién la ejecutó, qué herramienta se usó, el texto de la consulta y las estadísticas de rendimiento que describen la ejecución de la consulta.

## <a name="current-limitations"></a>Limitaciones actuales
Durante la versión preliminar pública, se aplican las siguientes limitaciones:

- Solo se registrarán las consultas centradas en el área de trabajo. Las consultas que se ejecutan en modo centrado en recursos o se ejecutan en una instancia de Application Insights no configurada como basada en el área de trabajo no se registrarán.


## <a name="configure-query-auditing"></a>Configuración de la auditoría de las consultas
La auditoría de las consultas se habilita con una [configuración de diagnóstico](../platform/diagnostic-settings.md) en el área de trabajo Log Analytics. Esto le permite enviar datos de auditoría al área de trabajo actual o a cualquier otra área de trabajo de la suscripción, a Azure Event Hubs para enviar datos fuera de Azure o a Azure Storage para su archivo. 

### <a name="azure-portal"></a>Azure portal
Acceda a la configuración de diagnóstico de un área de trabajo de Log Analytics en Azure Portal en cualquiera de las siguientes ubicaciones:

- En el menú de **Azure Monitor**, seleccione **Configuración de diagnóstico** y, a continuación, busque y seleccione el área de trabajo.

    [![Configuración de diagnóstico de Azure Monitor](media/log-query-audit/diagnostic-setting-monitor.png) ](media/log-query-audit/diagnostic-setting-monitor.png#lightbox) 

- En el menú **Áreas de trabajo de Log Analytics**, seleccione el área de trabajo y, a continuación, seleccione **Configuración de diagnóstico**.

    [![Configuración de diagnóstico del área de trabajo de Log Analytics](media/log-query-audit/diagnostic-setting-workspace.png) ](media/log-query-audit/diagnostic-setting-workspace.png#lightbox) 

### <a name="resource-manager-template"></a>Plantilla de Resource Manager
Puede obtener un ejemplo de una plantilla de Resource Manager en [Configuración de diagnóstico del área de trabajo de Log Analytics](../samples/resource-manager-diagnostic-settings.md#diagnostic-setting-for-log-analytics-workspace).

## <a name="audit-data"></a>Datos de auditoría
Cada vez que se ejecuta una consulta, se crea un registro de auditoría. Si envía los datos a un área de trabajo de Log Analytics, se almacena en una tabla llamada *LAQueryLogs*. En la tabla siguiente se describen las propiedades de cada registro de los datos de auditoría.

| Campo | Descripción |
|:---|:---|
| TimeGenerated         | Hora UTC a la que se envió la consulta. |
| CorrelationId         | Identificador único para identificar la consulta. Se puede usar en escenarios de solución de problemas al ponerse en contacto con Microsoft para obtener ayuda. |
| AADObjectId           | Identificador de Azure Active Directory de la cuenta de usuario que ha iniciado la consulta.  |
| AADTenantId           | Identificador de inquilino de la cuenta de usuario que ha iniciado la consulta.  |
| AADEmail              | Correo electrónico del inquilino de la cuenta de usuario que ha iniciado la consulta.  |
| AADClientId           | Identificador y nombre resuelto de la aplicación utilizada para iniciar la consulta. |
| RequestClientApp      | Nombre resuelto de la aplicación utilizada para iniciar la consulta. |
| QueryTimeRangeStart   | Inicio del intervalo de tiempo seleccionado para la consulta. Este valor podría no rellenarse en determinados escenarios, como cuando la consulta se inicia desde Log Analytics y el intervalo de tiempo se especifica dentro de la consulta en lugar de en el selector de tiempo. |
| QueryTimeRangeEnd     | Fin del intervalo de tiempo seleccionado para la consulta. Este valor podría no rellenarse en determinados escenarios, como cuando la consulta se inicia desde Log Analytics y el intervalo de tiempo se especifica dentro de la consulta en lugar de en el selector de tiempo.  |
| QueryText             | Texto de la consulta que se ha ejecutado. |
| RequestTarget         | Dirección URL de la API que se usó para enviar la consulta.  |
| RequestContext        | Lista de recursos en los que se solicitó la ejecución de la consulta. Contiene hasta tres matrices de cadenas: áreas de trabajo, aplicaciones y recursos. Las consultas dirigidas a grupos de recursos o suscripciones se mostrarán como *recursos*. Incluye el destino implícito por RequestTarget. |
| RequestContextFilters | Conjunto de filtros especificado como parte de la invocación de la consulta. Incluye hasta tres matrices de cadenas posibles:<br>- ResourceTypes: tipo de recurso para limitar el ámbito de la consulta.<br>- Workspaces: lista de áreas de trabajo para limitar la consulta.<br>- WorkspaceRegions: lista de regiones de áreas de trabajo para limitar la consulta. |
| ResponseCode          | Código de respuesta HTTP que se devolvió cuando se envió la consulta. |
| ResponseDurationMs    | Tiempo que se tardó en devolver la respuesta.  |
| StatsCPUTimeMs       | Tiempo total de proceso usado para el cálculo, el análisis y la recuperación de los datos. Solo se rellena si la consulta devuelve el código de estado 200. |
| StatsDataProcessedKB | Cantidad de datos a los que se accedió para procesar la consulta. Se ve afectado por el tamaño de la tabla de destino, el intervalo de tiempo usado, los filtros aplicados y el número de columnas a que se hace referencia. Solo se rellena si la consulta devuelve el código de estado 200. |
| StatsDataProcessedStart | Hora de los datos más antiguos a los que se accedió para procesar la consulta. Se ve influenciado por el intervalo de tiempo explícito de la consulta y los filtros aplicados. Podría ser mayor que el intervalo de tiempo explícito debido a la creación de particiones de datos. Solo se rellena si la consulta devuelve el código de estado 200. |
| StatsDataProcessedEnd  |Hora de los datos más recientes a los que se accedió para procesar la consulta. Se ve influenciado por el intervalo de tiempo explícito de la consulta y los filtros aplicados. Podría ser mayor que el intervalo de tiempo explícito debido a la creación de particiones de datos. Solo se rellena si la consulta devuelve el código de estado 200. |
| StatsWorkspaceCount | Número de áreas de trabajo a las que tuvo acceso la consulta. Solo se rellena si la consulta devuelve el código de estado 200. |
| StatsRegionCount | Número de regiones a las que tuvo acceso la consulta. Solo se rellena si la consulta devuelve el código de estado 200. |



## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [configuración de diagnóstico](../platform/diagnostic-settings.md).
- Más información sobre la [optimización de consultas del registro](query-optimization.md).