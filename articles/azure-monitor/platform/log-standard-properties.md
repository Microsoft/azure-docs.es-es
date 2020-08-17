---
title: Propiedades estándar de los registros de Azure Monitor | Microsoft Docs
description: Describe las propiedades comunes para varios tipos de datos de los registros de Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/01/2020
ms.openlocfilehash: 9c7e6733a9149008f6ed98ae7a7e0103f92f79f5
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325277"
---
# <a name="standard-properties-in-azure-monitor-logs"></a>Propiedades estándar de los registros de Azure Monitor
Los datos de los registros de Azure Monitor se [almacenan como un conjunto de registros en un área de trabajo de Log Analytics o una aplicación de Application Insights](../log-query/logs-structure.md), cada uno con un tipo de datos determinado que tiene un conjunto singular de propiedades. Muchos tipos de datos tendrán propiedades estándar que son comunes en varios tipos. En este artículo se describen estas propiedades y se proporcionan ejemplos de cómo puede usarlas en las consultas.

> [!IMPORTANT]
> Si usa APM 2,1, las aplicaciones de Application Insights se almacenan en un área de trabajo de Log Analytics con el resto de datos de registro. Se ha cambiado el nombre de las tablas y se han reestructurado, pero tienen la misma información que las tablas de la aplicación Application Insights. Estas nuevas tablas tienen las mismas propiedades estándar que otras tablas del área de trabajo de Log Analytics.

> [!NOTE]
> Algunas de las propiedades estándar no se mostrarán en la vista de esquema ni en IntelliSense en Log Analytics, y no aparecerán en los resultados de la consulta, a menos que especifique explícitamente la propiedad en la salida.

## <a name="timegenerated-and-timestamp"></a>TimeGenerated y timestamp
La propiedades **TimeGenerated** (área de trabajo de Log Analytics) y **timestamp** (aplicación de Application Insights) contienen la fecha y hora en que el origen de datos creó el registro. Consulte [Tiempo de la ingesta de datos de registro en Azure Monitor](data-ingestion-time.md) para más detalles.

**TimeGenerated** y **timestamp** proporcionan una propiedad común para usarla para filtrar o resumir por tiempo. Cuando se selecciona un intervalo de tiempo para una vista o panel en Azure Portal, se usa TimeGenerated o timestamp para filtrar los resultados. 

### <a name="examples"></a>Ejemplos

La consulta siguiente devuelve el número de eventos de error creados para cada día de la semana anterior.

```Kusto
Event
| where EventLevelName == "Error" 
| where TimeGenerated between(startofweek(ago(7days))..endofweek(ago(7days))) 
| summarize count() by bin(TimeGenerated, 1day) 
| sort by TimeGenerated asc 
```

La consulta siguiente devuelve el número de excepciones creados para cada día de la semana anterior.

```Kusto
exceptions
| where timestamp between(startofweek(ago(7days))..endofweek(ago(7days))) 
| summarize count() by bin(TimeGenerated, 1day) 
| sort by timestamp asc 
```

## <a name="_timereceived"></a>\_TimeReceived
La propiedad **\_TimeReceived** contiene la fecha y hora en que el punto de ingesta de Azure Monitor recibió el registro en la nube de Azure. Esto puede resultar útil para identificar problemas de latencia entre el origen de datos y la nube. Un ejemplo sería un error de red que genere un retraso con los datos que se envían desde un agente. Consulte [Tiempo de la ingesta de datos de registro en Azure Monitor](data-ingestion-time.md) para más detalles.

En la consulta siguiente se proporciona la latencia promedio por hora para los registros de eventos de un agente. Esto incluye el tiempo del agente a la nube y el tiempo total para que el registro esté disponible en las consultas de registro.

```Kusto
Event
| where TimeGenerated > ago(1d) 
| project TimeGenerated, TimeReceived = _TimeReceived, IngestionTime = ingestion_time() 
| extend AgentLatency = toreal(datetime_diff('Millisecond',TimeReceived,TimeGenerated)) / 1000
| extend TotalLatency = toreal(datetime_diff('Millisecond',IngestionTime,TimeGenerated)) / 1000
| summarize avg(AgentLatency), avg(TotalLatency) by bin(TimeGenerated,1hr)
``` 

## <a name="type-and-itemtype"></a>Type e itemType
Las propiedades **Type** (área de trabajo de Log Analytics) e **itemType** (aplicación de Application Insights) contienen el nombre de la tabla desde la que se recuperó el registro, que también se puede considerar como el tipo de registro. Esta propiedad es útil en las consultas que combinan registros de varias tablas, como las que utilizan el operador `search`, para distinguir entre registros de diferentes tipos. **$table** puede utilizarse en lugar de **Type** en algunos lugares.

### <a name="examples"></a>Ejemplos
La siguiente consulta devuelve el número de registros por tipo recopilados durante la última hora.

```Kusto
search * 
| where TimeGenerated > ago(1h)
| summarize count() by Type

```
## <a name="_itemid"></a>\_ItemId
La propiedad **\_ItemId** contiene un identificador único para el registro.


## <a name="_resourceid"></a>\_ResourceId
La propiedad **\_ResourceId** contiene un identificador único para el recurso con el que está asociado el registro. Esto le ofrece una propiedad estándar para usarla para definir el ámbito de la consulta a solo los registros de un recurso determinado, o para unir datos relacionados en varias tablas.

En el caso de los recursos de Azure, el valor de **_ResourceId** es la [URL de id. de recurso de Azure](../../azure-resource-manager/templates/template-functions-resource.md). La propiedad está actualmente limitada a los recursos de Azure, pero se extenderá a los recursos de fuera de Azure, por ejemplo, en equipos locales.

> [!NOTE]
> Algunos tipos de datos ya tienen campos que contienen el identificador de recurso de Azure o al menos partes de él como el identificador de suscripción. Aunque estos campos se guardan por compatibilidad con versiones anteriores, se recomienda utilizar _ResourceId para realizar la correlación cruzada, ya que será más coherente.

### <a name="examples"></a>Ejemplos
La consulta siguiente combina datos de rendimiento y de eventos para cada equipo. Muestra todos los eventos con el identificador _101_ y un uso del procesador superior al 50 %.

```Kusto
Perf 
| where CounterName == "% User Time" and CounterValue  > 50 and _ResourceId != "" 
| join kind=inner (     
    Event 
    | where EventID == 101 
) on _ResourceId
```

La consulta siguiente combina los registros _AzureActivity_ con los de _SecurityEvent_. Muestra todas las operaciones de actividad con los usuarios que iniciaron sesión en estas máquinas.

```Kusto
AzureActivity 
| where  
    OperationName in ("Restart Virtual Machine", "Create or Update Virtual Machine", "Delete Virtual Machine")  
    and ActivityStatus == "Succeeded"  
| join kind= leftouter (    
   SecurityEvent 
   | where EventID == 4624  
   | summarize LoggedOnAccounts = makeset(Account) by _ResourceId 
) on _ResourceId  
```

La siguiente consulta analiza **_ResourceId** y agrega volúmenes de datos facturados por suscripción de Azure.

```Kusto
union withsource = tt * 
| where _IsBillable == true 
| parse tolower(_ResourceId) with "/subscriptions/" subscriptionId "/resourcegroups/" 
    resourceGroup "/providers/" provider "/" resourceType "/" resourceName   
| summarize Bytes=sum(_BilledSize) by subscriptionId | sort by Bytes nulls last 
```

Use estas consultas `union withsource = tt *` con moderación, ya que la ejecución de exámenes entre tipos de datos es costosa.

## <a name="_isbillable"></a>\_IsBillable
La propiedad **\_IsBillable** especifica si los datos ingeridos son facturables. Los datos con **\_IsBillable** igual a `false` se recopilan de forma gratuita y no se facturan en su cuenta de Azure.

### <a name="examples"></a>Ejemplos
Para obtener una lista de equipos que envían los tipos de datos de facturación, use la siguiente consulta:

> [!NOTE]
> Use las consultas con `union withsource = tt *` con moderación, ya que la ejecución de exámenes entre tipos de datos es costosa. 

```Kusto
union withsource = tt * 
| where _IsBillable == true 
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize TotalVolumeBytes=sum(_BilledSize) by computerName
```

Dichas consultas pueden ampliarse para devolver el número de equipos por hora que están enviando tipos de datos facturados:

```Kusto
union withsource = tt * 
| where _IsBillable == true 
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize dcount(computerName) by bin(TimeGenerated, 1h) | sort by TimeGenerated asc
```

## <a name="_billedsize"></a>\_BilledSize
La propiedad **\_BilledSize** especifica el tamaño en bytes de los datos que se facturarán a su cuenta de Azure si **\_IsBillable** es true.


### <a name="examples"></a>Ejemplos
Para ver el tamaño de los eventos facturables ingeridos por equipo, use la propiedad `_BilledSize`, que proporciona el tamaño en bytes:

```Kusto
union withsource = tt * 
| where _IsBillable == true 
| summarize Bytes=sum(_BilledSize) by  Computer | sort by Bytes nulls last 
```

Para ver el tamaño de los eventos facturables ingeridos por suscripción, use la siguiente consulta:

```Kusto
union withsource=table * 
| where _IsBillable == true 
| parse _ResourceId with "/subscriptions/" SubscriptionId "/" *
| summarize Bytes=sum(_BilledSize) by  SubscriptionId | sort by Bytes nulls last 
```

Para ver el tamaño de los eventos facturables ingeridos por grupo de recursos, use la siguiente consulta:

```Kusto
union withsource=table * 
| where _IsBillable == true 
| parse _ResourceId with "/subscriptions/" SubscriptionId "/resourcegroups/" ResourceGroupName "/" *
| summarize Bytes=sum(_BilledSize) by  SubscriptionId, ResourceGroupName | sort by Bytes nulls last 

```


Para ver el recuento de eventos ingeridos por equipo, use la consulta siguiente:

```Kusto
union withsource = tt *
| summarize count() by Computer | sort by count_ nulls last
```

Para ver el recuento de eventos facturables ingeridos por equipo, use la consulta siguiente: 

```Kusto
union withsource = tt * 
| where _IsBillable == true 
| summarize count() by Computer  | sort by count_ nulls last
```

Para ver el recuento de tipos de datos facturables desde un equipo específico, use la siguiente consulta:

```Kusto
union withsource = tt *
| where Computer == "computer name"
| where _IsBillable == true 
| summarize count() by tt | sort by count_ nulls last 
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre cómo se almacenan los [datos del registro de Azure Monitor](../log-query/log-query-overview.md).
- Reciba una lección sobre [escritura de consultas de registro](../log-query/get-started-queries.md).
- Reciba una lección sobre [la unión de tablas en consultas de registro](../log-query/joins.md).

