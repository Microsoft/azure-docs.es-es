---
title: Ejemplos de consultas de registro de Azure Monitor | Microsoft Docs
description: Ejemplos de consultas de registro en Azure Monitor mediante el lenguaje de consulta de Kusto.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/16/2020
ms.openlocfilehash: 736daa8a09a8f08721c7b7d9c20f012f274b384a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87045526"
---
# <a name="azure-monitor-log-query-examples"></a>Ejemplos de consultas de registro de Azure Monitor
En este artículo se incluyen varios ejemplos de [consultas](log-query-overview.md) que usan el [lenguaje de consulta de Kusto](/azure/kusto/query/) para recuperar distintos tipos de datos de registro de Azure Monitor. Se utilizan métodos diferentes para consolidar y analizar los datos, por lo que puede usar estos ejemplos para identificar las diferentes estrategias que puede utilizar para sus propios requisitos.  

Consulte la [referencia del lenguaje Kusto](/azure/kusto/query/) para más información sobre las diferentes palabras clave usadas en estos ejemplos. Puede consultar una [lección sobre cómo crear consultas](get-started-queries.md) si no está familiarizado con Azure Monitor.

## <a name="events"></a>Eventos

### <a name="search-application-level-events-described-as-cryptographic"></a>Búsqueda de eventos de nivel de aplicación descritos como "Criptográficos"
Este ejemplo busca registros en la tabla **Events** en los que **EventLog** es igual a _Application_ y **RenderedDescription** contiene _cryptographic_. Incluye registros de las últimas 24 horas.

```Kusto
Event
| where EventLog == "Application" 
| where TimeGenerated > ago(24h) 
| where RenderedDescription contains "cryptographic"
```

### <a name="search-events-related-to-unmarshaling"></a>Búsqueda de eventos relacionados con la deserialización
Búsqueda de registros en las tablas **Event** y **SecurityEvents** que mencionan _unmarshaling_.

```Kusto
search in (Event, SecurityEvent) "unmarshaling"
```

## <a name="heartbeat"></a>Latido

### <a name="chart-a-week-over-week-view-of-the-number-of-computers-sending-data"></a>Gráfico de una vista semana a semana del número de equipos que envían datos

El ejemplo siguiente crea un gráfico del número de equipos distintos que envían latidos, cada semana.

```Kusto
Heartbeat
| where TimeGenerated >= startofweek(ago(21d))
| summarize dcount(Computer) by endofweek(TimeGenerated) | render barchart kind=default
```

### <a name="find-stale-computers"></a>Búsqueda de equipos obsoletos

El ejemplo siguiente busca los equipos que estaban activos en el último día, pero no han enviado latidos en la última hora.

```Kusto
Heartbeat
| where TimeGenerated > ago(1d)
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| where isnotempty(Computer)
| where LastHeartbeat < ago(1h)
```

### <a name="get-the-latest-heartbeat-record-per-computer-ip"></a>Obtención del registro de latido más reciente por dirección IP del equipo

Este ejemplo devuelve el último registro de latido por cada dirección IP de equipo.
```Kusto
Heartbeat
| summarize arg_max(TimeGenerated, *) by ComputerIP
```

### <a name="match-protected-status-records-with-heartbeat-records"></a>Coincidencia de los registros de estado protegido con los registros de latidos

Este ejemplo busca registros de estado protegido y registros de latidos relacionados, coincidentes en equipo y hora.
Tenga en cuenta que el campo de hora se redondea al minuto más cercano. Hemos usado un cálculo binario en tiempo de ejecución para ello: `round_time=bin(TimeGenerated, 1m)`.

```Kusto
let protection_data = ProtectionStatus
    | project Computer, DetectionId, round_time=bin(TimeGenerated, 1m);
let heartbeat_data = Heartbeat
    | project Computer, Category, round_time=bin(TimeGenerated, 1m);
protection_data | join (heartbeat_data) on Computer, round_time
```

### <a name="server-availability-rate"></a>Tasa de disponibilidad del servidor

Calcula la tasa de disponibilidad del servidor en función de los registros de latidos. La disponibilidad se define como "al menos 1 latido por hora".
Por lo tanto, si un servidor ha estado disponible 98 horas de cada 100, la tasa de disponibilidad es un 98%.

```Kusto
let start_time=startofday(datetime("2018-03-01"));
let end_time=now();
Heartbeat
| where TimeGenerated > start_time and TimeGenerated < end_time
| summarize heartbeat_per_hour=count() by bin_at(TimeGenerated, 1h, start_time), Computer
| extend available_per_hour=iff(heartbeat_per_hour>0, true, false)
| summarize total_available_hours=countif(available_per_hour==true) by Computer 
| extend total_number_of_buckets=round((end_time-start_time)/1h)+1
| extend availability_rate=total_available_hours*100/total_number_of_buckets
```


## <a name="multiple-data-types"></a>Varios tipos de datos

### <a name="chart-the-record-count-per-table"></a>Gráfico del número de registros por cada tabla
El siguiente ejemplo recopila todos los registros de todas las tablas en las últimas cinco horas y realiza un recuento de cuántos registros hay en cada tabla. Los resultados se muestran en un gráfico de tiempo.

```Kusto
union withsource=sourceTable *
| where TimeGenerated > ago(5h) 
| summarize count() by bin(TimeGenerated,10m), sourceTable
| render timechart
```

### <a name="count-all-logs-collected-over-the-last-hour-by-type"></a>Recuento de todos los registros recopilados durante la última hora por tipo
El ejemplo siguiente busca todo lo que se ha notificado en la última hora y hace un recuento de los registros de cada tabla por **tipo**. Los resultados se muestran en un gráfico de barras.

```Kusto
search *
| where TimeGenerated > ago(1h) 
| summarize CountOfRecords = count() by Type
| render barchart
```

## <a name="azurediagnostics"></a>AzureDiagnostics

### <a name="count-azure-diagnostics-records-per-category"></a>Recuento de registros de diagnóstico de Azure por categoría
Este ejemplo hace un recuento de todos los registros de diagnóstico de Azure por cada categoría única.

```Kusto
AzureDiagnostics 
| where TimeGenerated > ago(1d)
| summarize count() by Category
```

### <a name="get-a-random-record-for-each-unique-category"></a>Obtención de un registro aleatorio de cada categoría única
En este ejemplo se obtiene un único registro de diagnóstico de Azure aleatorio de cada categoría única.

```Kusto
AzureDiagnostics
| where TimeGenerated > ago(1d) 
| summarize any(*) by Category
```

### <a name="get-the-latest-record-per-category"></a>Obtención del registro más reciente por categoría
En este ejemplo se obtiene el último registro de diagnóstico de Azure de cada categoría única.

```Kusto
AzureDiagnostics
| where TimeGenerated > ago(1d) 
| summarize arg_max(TimeGenerated, *) by Category
```

## <a name="network-monitoring"></a>Supervisión de redes

### <a name="computers-with-unhealthy-latency"></a>Equipos con latencia en estado incorrecto
En este ejemplo se crea una lista de equipos con latencia en estado incorrecto.

```Kusto
NetworkMonitoring 
| where LatencyHealthState <> "Healthy" 
| where Computer != "" 
| distinct Computer
```

## <a name="performance"></a>Rendimiento

### <a name="join-computer-perf-records-to-correlate-memory-and-cpu"></a>Unión de los registros de rendimiento de equipos para poner en correlación memoria y CPU
En este ejemplo se pone en correlación los registros de **rendimiento** de un equipo determinado y se crean dos gráficos de tiempo: el promedio de CPU y la memoria máxima.

```Kusto
let StartTime = now()-5d;
let EndTime = now()-4d;
Perf
| where CounterName == "% Processor Time"  
| where TimeGenerated > StartTime and TimeGenerated < EndTime
| project TimeGenerated, Computer, cpu=CounterValue 
| join kind= inner (
   Perf
    | where CounterName == "% Used Memory"  
    | where TimeGenerated > StartTime and TimeGenerated < EndTime
    | project TimeGenerated , Computer, mem=CounterValue 
) on TimeGenerated, Computer
| summarize avgCpu=avg(cpu), maxMem=max(mem) by TimeGenerated bin=30m  
| render timechart
```

### <a name="perf-cpu-utilization-graph-per-computer"></a>Gráfico de rendimiento del uso de CPU por equipo
En este ejemplo se calcula y se representa gráficamente el uso de CPU de los equipos que comienzan por _Contoso_.

```Kusto
Perf
| where TimeGenerated > ago(4h)
| where Computer startswith "Contoso" 
| where CounterName == @"% Processor Time"
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 15m) 
| render timechart
```

## <a name="protection-status"></a>Estado de protección

### <a name="computers-with-non-reporting-protection-status-duration"></a>Equipos con la duración del estado de protección sin informes
En este ejemplo se enumeran los equipos que tenían un estado de protección de _Not Reporting_ y la duración en la que se encontraban en este estado.

```Kusto
ProtectionStatus
| where ProtectionStatus == "Not Reporting"
| summarize count(), startNotReporting = min(TimeGenerated), endNotReporting = max(TimeGenerated) by Computer, ProtectionStatusDetails
| join ProtectionStatus on Computer
| summarize lastReporting = max(TimeGenerated), startNotReporting = any(startNotReporting), endNotReporting = any(endNotReporting) by Computer
| extend durationNotReporting = endNotReporting - startNotReporting
```

### <a name="match-protected-status-records-with-heartbeat-records"></a>Coincidencia de los registros de estado protegido con los registros de latidos
Este ejemplo busca registros de estado protegido y registros de latidos relacionados, coincidentes en equipo y hora.
El campo de hora se redondea al minuto más cercano con **bin**.

```Kusto
let protection_data = ProtectionStatus
    | project Computer, DetectionId, round_time=bin(TimeGenerated, 1m);
let heartbeat_data = Heartbeat
    | project Computer, Category, round_time=bin(TimeGenerated, 1m);
protection_data | join (heartbeat_data) on Computer, round_time
```


## <a name="security-records"></a>Registros de seguridad

### <a name="count-security-events-by-activity-id"></a>Número de eventos de seguridad por identificador de actividad


Este ejemplo se basa en la estructura fija de la columna **Activity**: \<ID\>-\<Name\>.
Analiza el valor de **Activity** en dos nuevas columnas y hace un recuento de las apariciones de cada **activityID**.

```Kusto
SecurityEvent
| where TimeGenerated > ago(30m) 
| project Activity 
| parse Activity with activityID " - " activityDesc
| summarize count() by activityID
```

### <a name="count-security-events-related-to-permissions"></a>Número de eventos de seguridad relacionados con los permisos
En este ejemplo se muestra el número de registros **securityEvent** en los que la columna **Activity** contiene el término completo _Permissions_. La consulta se aplica a los registros creados en los últimos 30 minutos.

```Kusto
SecurityEvent
| where TimeGenerated > ago(30m)
| summarize EventCount = countif(Activity has "Permissions")
```

### <a name="find-accounts-that-failed-to-log-in-from-computers-with-a-security-detection"></a>Búsqueda de cuentas en las que no se pudo iniciar sesión desde equipos con una detección de seguridad
Este ejemplo busca y hace un recuento de las cuentas en las que no se pudo iniciar sesión desde equipos en los que se ha identificado una detección de seguridad.

```Kusto
let detections = toscalar(SecurityDetection
| summarize makeset(Computer));
SecurityEvent
| where Computer in (detections) and EventID == 4624
| summarize count() by Account
```

### <a name="is-my-security-data-available"></a>¿Están disponibles mis datos de seguridad?
La exploración de los datos a menudo comienza por la comprobación de la disponibilidad de los datos. En este ejemplo se muestra el número de registros **SecurityEvent** en los últimos 30 minutos.

```Kusto
SecurityEvent 
| where TimeGenerated  > ago(30m) 
| count
```

### <a name="parse-activity-name-and-id"></a>Análisis del nombre y el identificador de la actividad
Los dos ejemplos siguientes se basan en la estructura fija de la columna **Activity**: \<ID\>-\<Name\>. El primer ejemplo utiliza el operador **parse** para asignar valores a dos nuevas columnas: **activityID** y **activityDesc**.

```Kusto
SecurityEvent
| take 100
| project Activity 
| parse Activity with activityID " - " activityDesc
```

Este ejemplo usa el operador **split** para crear una matriz de valores independientes
```Kusto
SecurityEvent
| take 100
| project Activity 
| extend activityArr=split(Activity, " - ") 
| project Activity , activityArr, activityId=activityArr[0]
```

### <a name="explicit-credentials-processes"></a>Procesos de credenciales explícitas
El ejemplo siguiente muestra un gráfico circular de procesos que han usado credenciales explícitas en la última semana

```Kusto
SecurityEvent
| where TimeGenerated > ago(7d)
// filter by id 4648 ("A logon was attempted using explicit credentials")
| where EventID == 4648
| summarize count() by Process
| render piechart 
```

### <a name="top-running-processes"></a>Procesos de mayor ejecución

El ejemplo siguiente muestra una línea de tiempo de actividad para los cinco procesos más comunes, durante los últimos tres días.

```Kusto
// Find all processes that started in the last three days. ID 4688: A new process has been created.
let RunProcesses = 
    SecurityEvent
    | where TimeGenerated > ago(3d)
    | where EventID == "4688";
// Find the 5 processes that were run the most
let Top5Processes =
RunProcesses
| summarize count() by Process
| top 5 by count_;
// Create a time chart of these 5 processes - hour by hour
RunProcesses 
| where Process in (Top5Processes) 
| summarize count() by bin (TimeGenerated, 1h), Process
| render timechart
```


### <a name="find-repeating-failed-login-attempts-by-the-same-account-from-different-ips"></a>Búsqueda de intentos repetidos de inicio de sesión fallidos con la misma cuenta desde direcciones IP diferentes

El ejemplo siguiente busca los intentos de inicio de sesión fallidos con la misma cuenta desde más de cinco direcciones IP diferentes en las últimas seis horas.

```Kusto
SecurityEvent 
| where AccountType == "User" and EventID == 4625 and TimeGenerated > ago(6h) 
| summarize IPCount = dcount(IpAddress), makeset(IpAddress)  by Account
| where IPCount > 5
| sort by IPCount desc
```

### <a name="find-user-accounts-that-failed-to-log-in"></a>Búsqueda de cuentas de usuario en las que no se pudo iniciar sesión 
El ejemplo siguiente identifica las cuentas de usuario en las que no se pudo iniciar sesión más de cinco veces en el último día y cuándo se intentó iniciar sesión por última vez.

```Kusto
let timeframe = 1d;
SecurityEvent
| where TimeGenerated > ago(1d)
| where AccountType == 'User' and EventID == 4625 // 4625 - failed log in
| summarize failed_login_attempts=count(), latest_failed_login=arg_max(TimeGenerated, Account) by Account 
| where failed_login_attempts > 5
| project-away Account1
```

Mediante el uso de las instrucciones **join** y **let**, podemos comprobar si las mismas cuentas sospechosas posteriormente pudieron iniciar sesión correctamente.

```Kusto
let timeframe = 1d;
let suspicious_users = 
    SecurityEvent
    | where TimeGenerated > ago(timeframe)
    | where AccountType == 'User' and EventID == 4625 // 4625 - failed login
    | summarize failed_login_attempts=count(), latest_failed_login=arg_max(TimeGenerated, Account) by Account 
    | where failed_login_attempts > 5
    | project-away Account1;
let suspicious_users_that_later_logged_in = 
    suspicious_users 
    | join kind=innerunique (
        SecurityEvent
        | where TimeGenerated > ago(timeframe)
        | where AccountType == 'User' and EventID == 4624 // 4624 - successful login,
        | summarize latest_successful_login=arg_max(TimeGenerated, Account) by Account
    ) on Account
    | extend was_login_after_failures = iif(latest_successful_login>latest_failed_login, 1, 0)
    | where was_login_after_failures == 1
;
suspicious_users_that_later_logged_in
```

## <a name="usage"></a>Uso

El tipo de datos `Usage` se puede usar para realizar un seguimiento del volumen de datos ingerido por solución o tipo de datos. Hay otras técnicas para estudiar los volúmenes de datos ingeridos por [equipo](../platform/manage-cost-storage.md#data-volume-by-computer) o por [suscripción, grupo de recursos o recurso de Azure](../platform/manage-cost-storage.md#data-volume-by-azure-resource-resource-group-or-subscription).

#### <a name="data-volume-by-solution"></a>Data volume by solution (Volumen de datos por solución)

La consulta que se usa para ver el volumen de datos facturable por solución en el último mes (excepto el último día parcial) es:

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), Solution | render barchart
```

Tenga en cuenta que la cláusula `where IsBillable = true` filtra los tipos de datos de determinadas soluciones para las que no hay ningún cargo de ingesta.  Además, la cláusula con `TimeGenerated` solo se utiliza para asegurarse de que la experiencia de consulta en Azure Portal examine más allá del periodo predeterminado de 24 horas. Al utilizar el tipo de datos de uso, `StartTime` y `EndTime` representan los periodos de tiempo de los que se presentan resultados. 

#### <a name="data-volume-by-type"></a>Volumen de datos por tipo

Puede profundizar más para ver las tendencias de datos por tipo de datos:

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), DataType | render barchart
```

O bien, para ver una tabla por solución y tipo durante el último mes,

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by Solution, DataType
| sort by Solution asc, DataType asc
```

> [!NOTE]
> Algunos de los campos del tipo de datos de uso, aunque siguen en el esquema, han quedado en desuso y ya no se rellenarán sus valores. Estos son **Computer**, además de los campos relacionados con la ingesta (**TotalBatches**, **BatchesWithinSla**, **BatchesOutsideSla**, **BatchesCapped** y **AverageProcessingTimeMs**.

## <a name="updates"></a>Actualizaciones

### <a name="computers-still-missing-updates"></a>Equipos con actualizaciones pendientes
En este ejemplo se muestra una lista de equipos a los que les faltaban una o más actualizaciones críticas hace unos días y todavía les faltan las actualizaciones.

```Kusto
let ComputersMissingUpdates3DaysAgo = Update
| where TimeGenerated between (ago(30d)..ago(1h))
| where Classification !has "Critical" and UpdateState =~ "Needed"
| summarize makeset(Computer);
Update
| where TimeGenerated > ago(1d)
| where Classification has "Critical" and UpdateState =~ "Needed"
| where Computer in (ComputersMissingUpdates3DaysAgo)
| summarize UniqueUpdatesCount = dcount(Product) by Computer, OSType
```


## <a name="next-steps"></a>Pasos siguientes

- Consulte la [referencia del lenguaje Kusto](/azure/kusto/query) para más información sobre el lenguaje.
- Consulte una [lección sobre la escritura de consultas de registro en Azure Monitor](get-started-queries.md).
