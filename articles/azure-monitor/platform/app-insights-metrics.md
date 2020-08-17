---
title: Métricas basadas en registros de Azure Application Insights | Microsoft Docs
description: En este artículo se enumeran las métricas de Azure Application Insights con las agregaciones y dimensiones compatibles. Los detalles sobre las métricas basadas en registros incluyen las instrucciones de consulta de Kusto subyacentes.
author: vgorbenko
services: azure-monitor
ms.topic: reference
ms.date: 07/03/2019
ms.author: vitalyg
ms.subservice: application-insights
ms.openlocfilehash: 9ea98df4b6cd8572412e7082b451feac3736919c
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327079"
---
# <a name="application-insights-log-based-metrics"></a>Métricas basadas en registros de Application Insights

Las métricas basadas en registros de Application Insights le permiten analizar el estado de las aplicaciones supervisadas, crear paneles eficaces y configurar alertas. Existen dos tipos de métricas:

* [Las métricas basadas en registros](../app/pre-aggregated-metrics-log-metrics.md#log-based-metrics) subyacentes se traducen en [consultas de Kusto](/azure/kusto/query/) de eventos almacenados.
* [Las métricas estándar](../app/pre-aggregated-metrics-log-metrics.md#pre-aggregated-metrics) se almacenan como series temporales previamente agregadas.

Puesto que las *métricas estándar* se agregan previamente durante la recopilación, tienen un mejor rendimiento en el momento de la consulta. Esto las convierte en una opción mejor para los paneles y las alertas en tiempo real. Las *métricas basadas en registros* tienen más dimensiones, lo que las convierte en la mejor opción para el análisis de datos y los diagnósticos ad hoc. Use el [selector del espacio de nombres](metrics-getting-started.md#create-your-first-metric-chart) para cambiar entre las métricas basadas en registros y las métricas estándar en el [explorador de métricas](metrics-getting-started.md).

## <a name="interpret-and-use-queries-from-this-article"></a>Interpretación y uso de consultas de este artículo

En este artículo se enumeran las métricas con las agregaciones y dimensiones compatibles. Los detalles sobre las métricas basadas en registros incluyen las instrucciones de consulta de Kusto subyacentes. Para mayor comodidad, cada consulta usa valores predeterminados para la granularidad del tiempo, el tipo de gráfico y, a veces, para dividir la dimensión, lo que simplifica el uso de la consulta en Log Analytics sin necesidad de modificarla.

Cuando se traza la misma métrica en el [explorador de métricas](metrics-getting-started.md), no hay valores predeterminados: la consulta se ajusta dinámicamente en función de la configuración del gráfico:

- El **Intervalo de tiempo** seleccionado se traduce en un una cláusula *where timestamp...* adicional para seleccionar solo los eventos del intervalo de tiempo seleccionado. Por ejemplo, en un gráfico que muestra los datos de las últimas 24 horas, la consulta incluye *| where timestamp > ago(24 h)* .

- La **Granularidad de tiempo** seleccionada se pone en la cláusula final.*summarize ... by bin(timestamp, [time grain])* .

- Todas las dimensiones de **Filtro** seleccionadas se traducen en cláusulas *where* adicionales.

- La dimensión **Split chart** (Dividir gráfico) se traduce en una propiedad de resumen adicional. Por ejemplo, si divide el gráfico por *ubicación* y realiza un trazado usando una granularidad de tiempo de cinco minutos, la cláusula *summarize* se resume como *... by bin(timestamp, 5 m), location*.

> [!NOTE]
> Si no está familiarizado con el lenguaje de consulta de Kusto, comience por copiar y pegar las instrucciones de Kusto en el panel de consulta de Log Analytics sin realizar ninguna modificación. Haga clic en **Ejecutar** para ver el gráfico básico. Cuando empiece a comprender la sintaxis del lenguaje de consulta, podrá comenzar a realizar pequeñas modificaciones y ver el impacto del cambio. La exploración de sus propios datos es una excelente manera de comenzar a obtener toda la eficacia de [Log Analytics](../log-query/get-started-portal.md) y [Azure Monitor](../overview.md).

## <a name="availability-metrics"></a>Métricas de disponibilidad

Las métricas de la categoría Disponibilidad le permiten ver el estado de la aplicación web, tal y como se observa en los puntos de todo el mundo. [Configure las pruebas de disponibilidad](../app/monitor-web-app-availability.md) para empezar a usar las métricas de esta categoría.

### <a name="availability-availabilityresultsavailabilitypercentage"></a>Disponibilidad (availabilityResults/availabilityPercentage)
La métrica *Disponibilidad* muestra el porcentaje de las series de pruebas web que no detectaron ningún problema. El valor mínimo posible es 0, que indica que se han producido errores en todas las series de pruebas web. El valor de 100 significa que todas las series de pruebas web superaron los criterios de validación.

|Unidad de medida|Agregaciones compatibles|Dimensiones compatibles|
|---|---|---|---|---|---|
|Porcentaje|Average|Ubicación de ejecución, nombre de prueba|

```Kusto
availabilityResults 
| summarize sum(todouble(success == 1) * 100) / count() by bin(timestamp, 5m), location
| render timechart
```

### <a name="availability-test-duration-availabilityresultsduration"></a>Duración de la prueba de disponibilidad (availabilityResults/duration)

La métrica *Duración de la prueba de disponibilidad* muestra cuánto tiempo tardó en ejecutarse la prueba web. En el caso de las [pruebas web de varios pasos](../app/availability-multistep.md), la métrica refleja el tiempo total de ejecución de todos los pasos.

|Unidad de medida|Agregaciones compatibles|Dimensiones compatibles|
|---|---|---|---|---|---|
|Milisegundos|Promedio, mín., máx.|Ubicación de ejecución, nombre de prueba, resultado de prueba

```Kusto
availabilityResults
| where notempty(duration)
| extend availabilityResult_duration = iif(itemType == 'availabilityResult', duration, todouble(''))
| summarize sum(availabilityResult_duration)/sum(itemCount) by bin(timestamp, 5m), location
| render timechart
```

### <a name="availability-tests-availabilityresultscount"></a>Pruebas de disponibilidad (availabilityResults/count)

La métrica de las *Pruebas de disponibilidad* refleja el recuento de las series de pruebas web de Azure Monitor.

|Unidad de medida|Agregaciones compatibles|Dimensiones compatibles|
|---|---|---|---|---|---|
|Count|Count|Ubicación de ejecución, nombre de prueba, resultado de prueba|

```Kusto
availabilityResults
| summarize sum(itemCount) by bin(timestamp, 5m)
| render timechart
```

## <a name="browser-metrics"></a>Métricas del explorador

El SDK de JavaScript de Application Insights recopila las métricas de explorador de los exploradores de los usuarios finales reales. Proporcionan una gran información sobre la experiencia de los usuarios con la aplicación web. Normalmente, no se muestrean las métricas del explorador, lo que significa que proporcionan una mayor precisión de los números de uso en comparación con las métricas del lado servidor, que podrían sesgarse con el muestreo.

> [!NOTE]
> Para recopilar las métricas del explorador, la aplicación debe estar instrumentada con el [SDK de JavaScript para Application Insights](../app/javascript.md).

### <a name="browser-page-load-time-browsertimingstotalduration"></a>Tiempo de carga de página del explorador (browserTimings/totalDuration)

|Unidad de medida|Agregaciones compatibles|Dimensiones previamente agregadas|
|---|---|---|
|Milisegundos|Promedio, mín., máx.|None|

```Kusto
browserTimings
| where notempty(totalDuration)
| extend _sum = totalDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="client-processing-time-browsertimingprocessingduration"></a>Tiempo de procesamiento del cliente (browserTiming/processingDuration)

|Unidad de medida|Agregaciones compatibles|Dimensiones previamente agregadas|
|---|---|---|
|Milisegundos|Promedio, mín., máx.|None|

```Kusto
browserTimings
| where notempty(processingDuration)
| extend _sum = processingDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum)/sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="page-load-network-connect-time-browsertimingsnetworkduration"></a>Tiempo de conexión de red de carga de página (browserTimings/networkDuration)

|Unidad de medida|Agregaciones compatibles|Dimensiones previamente agregadas|
|---|---|---|
|Milisegundos|Promedio, mín., máx.|None|

```Kusto
browserTimings
| where notempty(networkDuration)
| extend _sum = networkDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="receiving-response-time-browsertimingsreceiveduration"></a>Tiempo de respuesta de recepción (browserTimings/receiveDuration)

|Unidad de medida|Agregaciones compatibles|Dimensiones previamente agregadas|
|---|---|---|
|Milisegundos|Promedio, mín., máx.|None|

```Kusto
browserTimings
| where notempty(receiveDuration)
| extend _sum = receiveDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="send-request-time-browsertimingssendduration"></a>Hora de envío de la solicitud (browserTimings/sendDuration)

|Unidad de medida|Agregaciones compatibles|Dimensiones previamente agregadas|
|---|---|---|
|Milisegundos|Promedio, mín., máx.|None|

```Kusto
browserTimings
| where notempty(sendDuration)
| extend _sum = sendDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

## <a name="failure-metrics"></a>Métricas de errores

Las métricas de **Errores** muestran problemas con el procesamiento de solicitudes, las llamadas de dependencia y las excepciones producidas.

### <a name="browser-exceptions-exceptionsbrowser"></a>Excepciones de explorador (exceptions/browser)

Esta métrica refleja el número de excepciones producidas a partir del código de aplicación que se ejecuta en el explorador. Solo se incluyen en la métrica las excepciones de las que se realiza un seguimiento con una llamada API de Application Insights ```trackException()```.

|Unidad de medida|Agregaciones compatibles|Dimensiones previamente agregadas|Notas|
|---|---|---|---|
|Count|Count|None|La versión basada en registros usa la agregación **Sum**.|

```Kusto
exceptions
| where notempty(client_Browser)
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="dependency-call-failures-dependenciesfailed"></a>Errores de llamada de dependencia (dependencies/failed)

Cantidad de llamadas de dependencia con errores.

|Unidad de medida|Agregaciones compatibles|Dimensiones previamente agregadas|Notas|
|---|---|---|---|
|Count|Count|None|La versión basada en registros usa la agregación **Sum**.|

```Kusto
dependencies
| where success == 'False'
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="exceptions-exceptionscount"></a>Excepciones (exceptions/count)

Cada vez que registra una excepción en Application Insights, se produce una llamada al [método trackException()](../app/api-custom-events-metrics.md#trackexception) del SDK. La métrica de excepciones muestra la cantidad de excepciones registradas.

|Unidad de medida|Agregaciones compatibles|Dimensiones previamente agregadas|Notas|
|---|---|---|---|
|Count|Count|Nombre del rol en la nube, instancia de rol en la nube, tipo de dispositivo|La versión basada en registros usa la agregación **Sum**.|

```Kusto
exceptions
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="failed-requests-requestsfailed"></a>Solicitudes con error (requests/failed)

Recuento de solicitudes de servidor de las que se ha realizado un seguimiento y que se marcaron como *errores*. De forma predeterminada, el SDK de Application Insights marca automáticamente cada solicitud de servidor que devolvió el código de respuesta HTTP 5xx o 4xx como una solicitud con error. Para personalizar esta lógica, puede modificar la propiedad *success* del elemento de telemetría de la solicitud en un [inicializador de telemetría personalizado](../app/api-filtering-sampling.md#addmodify-properties-itelemetryinitializer).

|Unidad de medida|Agregaciones compatibles|Dimensiones previamente agregadas|Notas|
|---|---|---|---|
|Count|Count|Instancia de rol en la nube, nombre de rol en la nube, tráfico real o sintético, rendimiento de la solicitud, código de respuesta|La versión basada en registros usa la agregación **Sum**.|

```Kusto
requests
| where success == 'False'
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="server-exceptions-exceptionsserver"></a>Excepciones del servidor (exceptions/server)

Esta métrica muestra el número de excepciones de servidor.

|Unidad de medida|Agregaciones compatibles|Dimensiones previamente agregadas|Notas|
|---|---|---|---|
|Count|Count|Nombre del rol en la nube, instancia de rol en la nube|La versión basada en registros usa la agregación **Sum**.|

```Kusto
exceptions
| where isempty(client_Browser)
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

## <a name="performance-counters"></a>Contadores de rendimiento

Use las métricas de la categoría de **Contadores de rendimiento** para acceder a los [contadores de rendimiento del sistema que recopila Application Insights](../app/performance-counters.md).

### <a name="available-memory-performancecountersavailablememory"></a>Memoria disponible (performanceCounters/availableMemory)

```Kusto
performanceCounters
| where ((category == "Memory" and counter == "Available Bytes") or name == "availableMemory")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="exception-rate-performancecountersexceptionrate"></a>Tasa de excepciones (performanceCounters/exceptionRate)

```Kusto
performanceCounters
| where ((category == ".NET CLR Exceptions" and counter == "# of Exceps Thrown / sec") or name == "exceptionRate")
| extend performanceCounter_value = iif(itemType == 'performanceCounter',value,todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="http-request-execution-time-performancecountersrequestexecutiontime"></a>Tiempo de ejecución de la solicitud HTTP (performanceCounters/requestExecutionTime)

```Kusto
performanceCounters
| where ((category == "ASP.NET Applications" and counter == "Request Execution Time") or name == "requestExecutionTime")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="http-request-rate-performancecountersrequestspersecond"></a>Frecuencia de solicitudes HTTP (performanceCounters/requestsPerSecond)

```Kusto
performanceCounters
| where ((category == "ASP.NET Applications" and counter == "Requests/Sec") or name == "requestsPerSecond")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="http-requests-in-application-queue-performancecountersrequestsinqueue"></a>Solicitudes HTTP en la cola de la aplicación (performanceCounters/requestsInQueue)

```Kusto
performanceCounters
| where ((category == "ASP.NET Applications" and counter == "Requests In Application Queue") or name == "requestsInQueue")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="process-cpu-performancecountersprocesscpupercentage"></a>CPU de proceso (performanceCounters/processCpuPercentage)

La métrica muestra la cantidad de la capacidad total del procesador que usa el proceso que hospeda la aplicación supervisada.

|Unidad de medida|Agregaciones compatibles|Dimensiones compatibles|
|---|---|---|
|Porcentaje|Promedio, mín., máx.|Instancia de rol en la nube

```Kusto
performanceCounters
| where ((category == "Process" and counter == "% Processor Time Normalized") or name == "processCpuPercentage")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="process-io-rate-performancecountersprocessiobytespersecond"></a>Tasa de E/S del proceso (performanceCounters/processIOBytesPerSecond)

|Unidad de medida|Agregaciones compatibles|Dimensiones compatibles|
|---|---|---|
|Bytes por segundo|Promedio, mín., máx.|Instancia de rol en la nube

```Kusto
performanceCounters
| where ((category == "Process" and counter == "IO Data Bytes/sec") or name == "processIOBytesPerSecond")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="process-private-bytes-performancecountersprocessprivatebytes"></a>Bytes privados del proceso (performanceCounters/processPrivateBytes)

Cantidad de memoria no compartida que el proceso supervisado asignó para sus datos.

|Unidad de medida|Agregaciones compatibles|Dimensiones compatibles|
|---|---|---|
|Bytes|Promedio, mín., máx.|Instancia de rol en la nube

```Kusto
performanceCounters
| where ((category == "Process" and counter == "Private Bytes") or name == "processPrivateBytes")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="processor-time-performancecountersprocessorcpupercentage"></a>Tiempo de procesador (performanceCounters/processorCpuPercentage)

Consumo de CPU de *todos* los procesos que se ejecutan en la instancia del servidor supervisado.

|Unidad de medida|Agregaciones compatibles|Dimensiones compatibles|
|---|---|---|
|Porcentaje|Promedio, mín., máx.|Instancia de rol en la nube

>[!NOTE]
> La métrica de tiempo del procesador no está disponible para las aplicaciones hospedadas en Azure App Services. Use la métrica de la [CPU de procesos](#process-cpu-performancecountersprocesscpupercentage) para realizar un seguimiento del uso de la CPU de las aplicaciones web hospedadas en App Services.

```Kusto
performanceCounters
| where ((category == "Processor" and counter == "% Processor Time") or name == "processorCpuPercentage")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

## <a name="server-metrics"></a>Métricas del servidor

### <a name="dependency-calls-dependenciescount"></a>Llamadas de dependencia (dependencies/count)

Esta métrica está relacionada con el número de llamadas de dependencia.

```Kusto
dependencies
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="dependency-duration-dependenciesduration"></a>Duración de la dependencia (dependencies/duration)

Esta métrica hace referencia a la duración de las llamadas de dependencia.

```Kusto
dependencies
| where notempty(duration)
| extend dependency_duration = iif(itemType == 'dependency',duration,todouble(''))
| extend _sum = dependency_duration
| extend _count = itemCount
| extend _sum = _sum*_count
| summarize sum(_sum)/sum(_count) by bin(timestamp, 1m)
| render timechart
```

### <a name="server-requests-requestscount"></a>Solicitudes de servidor (requests/count)

Esta métrica refleja el número de solicitudes de servidor entrantes que recibió la aplicación web.

```Kusto
requests
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="server-response-time-requestsduration"></a>Tiempo de respuesta del servidor (requests/duration)

Esta métrica refleja el tiempo que tardaron los servidores en procesar las solicitudes entrantes.

```Kusto
requests
| where notempty(duration)
| extend request_duration = iif(itemType == 'request', duration, todouble(''))
| extend _sum = request_duration
| extend _count = itemCount
| extend _sum = _sum*_count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 1m)
| render timechart
```

## <a name="usage-metrics"></a>Métricas de uso

### <a name="page-view-load-time-pageviewsduration"></a>Tiempo de carga de la vista de página (pageViews/duration)

Esta métrica hace referencia a la cantidad de tiempo que tardaron en cargarse los eventos de PageView.

```Kusto
pageViews
| where notempty(duration)
| extend pageView_duration = iif(itemType == 'pageView', duration, todouble(''))
| extend _sum = pageView_duration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render barchart
```

### <a name="page-views-pageviewscount"></a>Vistas de página (pageViews/count)

Recuento de eventos de PageView registrados con la API de Application Insights TrackPageView().

```Kusto
pageViews
| summarize sum(itemCount) by bin(timestamp, 1h)
| render barchart
```

### <a name="sessions-sessionscount"></a>Sesiones (sessions/count)

Esta métrica hace referencia al recuento de distintos identificadores de sesión.

```Kusto
union traces, requests, pageViews, dependencies, customEvents, availabilityResults, exceptions, customMetrics, browserTimings
| where notempty(session_Id)
| summarize dcount(session_Id) by bin(timestamp, 1h)
| render barchart
```

### <a name="traces-tracescount"></a>Seguimientos (traces/count)

Recuento de instrucciones de seguimiento registradas con la llamada API de Application Insights TrackTrace().

```Kusto
traces
| summarize sum(itemCount) by bin(timestamp, 1h)
| render barchart
```

### <a name="users-userscount"></a>Usuarios (users/count)

Número de usuarios distintos que han tenido acceso a la aplicación. La precisión de esta métrica puede verse afectada significativamente mediante el uso del filtrado y el muestreo de telemetría.

```Kusto
union traces, requests, pageViews, dependencies, customEvents, availabilityResults, exceptions, customMetrics, browserTimings
| where notempty(user_Id)
| summarize dcount(user_Id) by bin(timestamp, 1h)
| render barchart
```

### <a name="users-authenticated-usersauthenticated"></a>Usuarios, autenticados (users/authenticated)

Número de usuarios distintos que se han autenticado en la aplicación.

```Kusto
union traces, requests, pageViews, dependencies, customEvents, availabilityResults, exceptions, customMetrics, browserTimings
| where notempty(user_AuthenticatedId)
| summarize dcount(user_AuthenticatedId) by bin(timestamp, 1h)
| render barchart
```

