---
title: Agregaciones en consultas de registros de Azure Monitor | Microsoft Docs
description: Describe funciones de agregación en consultas de registro de Azure Monitor que ofrecen maneras útiles de analizar los datos.
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 08/16/2018
ms.author: bwren
ms.openlocfilehash: fd8e886a78d0689ca60d8ea7c4d16639c81d5733
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65602734"
---
# <a name="aggregations-in-azure-monitor-log-queries"></a>Agregaciones en consultas de registros de Azure Monitor

> [!NOTE]
> Debe completar la [Introducción al portal de Analytics](get-started-portal.md) y la [Introducción a las consultas en Log Analytics](get-started-queries.md) antes de completar esta lección.

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

En este artículo se describen las funciones de agregación en consultas de registro de Azure Monitor que ofrecen maneras útiles de analizar los datos. Estas funciones usan el operador `summarize` que genera una tabla con resultados agregados de la tabla de entrada.

## <a name="counts"></a>Recuentos

### <a name="count"></a>número
Cuente el número de filas del conjunto de resultados después de aplicar filtros. En el ejemplo siguiente se devuelve el número total de filas de la tabla _Perf_ de los últimos 30 minutos. El resultado se devuelve en una columna denominada *count_* a menos que se le asigne un nombre específico:


```Kusto
Perf
| where TimeGenerated > ago(30m) 
| summarize count()
```

```Kusto
Perf
| where TimeGenerated > ago(30m) 
| summarize num_of_records=count() 
```

La visualización de un gráfico de tiempo puede ser útil para ver la tendencia a lo largo del tiempo:

```Kusto
Perf 
| where TimeGenerated > ago(30m) 
| summarize count() by bin(TimeGenerated, 5m)
| render timechart
```

La salida de este ejemplo muestra la línea de tendencia del recuento de registros de perf en intervalos de 5 minutos:

![Tendencia del recuento](media/aggregations/count-trend.png)


### <a name="dcount-dcountif"></a>dcount, dcountif
Use `dcount` y `dcountif` para contar valores distintos en una columna específica. En la siguiente consulta se evalúa cuántos equipos distintos enviaron latidos en la última hora:

```Kusto
Heartbeat 
| where TimeGenerated > ago(1h) 
| summarize dcount(Computer)
```

Para contar solo los equipos Linux que enviaron latidos, use `dcountif`:

```Kusto
Heartbeat 
| where TimeGenerated > ago(1h) 
| summarize dcountif(Computer, OSType=="Linux")
```

### <a name="evaluating-subgroups"></a>Evaluación de subgrupos
Para realizar un recuento u otras agregaciones en subgrupos de los datos, use la palabra clave `by`. Por ejemplo, para contar el número de equipos Linux distintos que envían latidos en cada país o región:

```Kusto
Heartbeat 
| where TimeGenerated > ago(1h) 
| summarize distinct_computers=dcountif(Computer, OSType=="Linux") by RemoteIPCountry
```

|RemoteIPCountry  | distinct_computers  |
------------------|---------------------|
|Estados Unidos    | 19                  |
|Canadá           | 3                   |
|Irlanda          | 0                   |
|Reino Unido   | 0                   |
|Países bajos      | 2                   |


Para analizar subgrupos incluso más pequeños de los datos, agregue nombres de columna adicionales a la sección `by`. Por ejemplo, podría desea contar los distintos equipos de cada país o región por OSType:

```Kusto
Heartbeat 
| where TimeGenerated > ago(1h) 
| summarize distinct_computers=dcountif(Computer, OSType=="Linux") by RemoteIPCountry, OSType
```

## <a name="percentiles-and-variance"></a>Percentiles y varianza
Al evaluar valores numéricos, una práctica común es que calcular la media con `summarize avg(expression)`. Las medias se ven afectadas por valores extremos que caracterizan a solo unos pocos casos. Para solucionar ese problema, puede usar funciones menos sensibles, como `median` o `variance`.

### <a name="percentile"></a>Percentil
Para buscar el valor medio, utilice la función `percentile` con un valor para especificar el percentil:

```Kusto
Perf
| where TimeGenerated > ago(30m) 
| where CounterName == "% Processor Time" and InstanceName == "_Total" 
| summarize percentiles(CounterValue, 50) by Computer
```

También puede especificar percentiles diferentes para obtener un resultado agregado para cada uno:

```Kusto
Perf
| where TimeGenerated > ago(30m) 
| where CounterName == "% Processor Time" and InstanceName == "_Total" 
| summarize percentiles(CounterValue, 25, 50, 75, 90) by Computer
```

Esto podría mostrar que algunas CPU de equipos tienen valores medios similares, pero mientras algunas son constantes en torno al valor medio, otros equipos han informado de valores de CPU mucho menores y mayores, lo que significa que experimentaron picos.

### <a name="variance"></a>Variance
Para evaluar directamente la varianza de un valor, use los métodos estándar de desviación y varianza:

```Kusto
Perf
| where TimeGenerated > ago(30m) 
| where CounterName == "% Processor Time" and InstanceName == "_Total" 
| summarize stdev(CounterValue), variance(CounterValue) by Computer
```

Una buena forma de analizar la estabilidad del uso de CPU consiste en combinar stdev con el cálculo del valor medio:

```Kusto
Perf
| where TimeGenerated > ago(130m) 
| where CounterName == "% Processor Time" and InstanceName == "_Total" 
| summarize stdev(CounterValue), percentiles(CounterValue, 50) by Computer
```

Consulte otras lecciones para usar el [lenguaje de consulta de Kusto](/azure/kusto/query/) con datos de registro de Azure Monitor:

- [Operaciones de cadena](string-operations.md)
- [Operaciones de fecha y hora](datetime-operations.md)
- [Agregaciones avanzadas](advanced-aggregations.md)
- [JSON y estructuras de datos](json-data-structures.md)
- [Escritura de consultas avanzadas](advanced-query-writing.md)
- [Combinaciones](joins.md)
- [Gráficos](charts.md)
