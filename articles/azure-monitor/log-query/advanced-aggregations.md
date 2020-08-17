---
title: Agregaciones avanzadas en consultas de registros de Azure Monitor | Microsoft Docs
description: Describe algunas de las opciones de agregación más avanzadas disponibles para las consultas de registros de Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/16/2018
ms.openlocfilehash: dba058dce09e958a2ae769d927a5569fb3e42113
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324563"
---
# <a name="advanced-aggregations-in-azure-monitor-log-queries"></a>Agregaciones avanzadas en consultas de registros de Azure Monitor

> [!NOTE]
> Debe completar las [Agregaciones en consultas de Azure Monitor](./aggregations.md) antes de completar esta lección.

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

En este artículo se describen algunas de las opciones de agregación más avanzadas disponibles para las consultas de Azure Monitor.

## <a name="generating-lists-and-sets"></a>Generación de listas y conjuntos
Puede usar el elemento `makelist` para dinamizar los datos por el orden de los valores de una columna en particular. Por ejemplo, puede explorar el orden más común en el que los eventos se producen en los equipos. Básicamente, puede dinamizar los datos por el orden de los objetos EventID de cada máquina. 

```Kusto
Event
| where TimeGenerated > ago(12h)
| order by TimeGenerated desc
| summarize makelist(EventID) by Computer
```

|Computer|list_EventID|
|---|---|
| computer1 | [704,701,1501,1500,1085,704,704,701] |
| computer2 | [326,105,302,301,300,102] |
| … | … |

El elemento `makelist` genera una lista en el orden en el que se le pasaron datos. Para ordenar los eventos del más antiguo al más reciente, use el elemento `asc` en la instrucción order en lugar del elemento `desc`. 

También es útil crear una lista de valores simplemente distintos. Esto se denomina un _conjunto_ y se puede generar con el elemento `makeset`:

```Kusto
Event
| where TimeGenerated > ago(12h)
| order by TimeGenerated desc
| summarize makeset(EventID) by Computer
```

|Computer|list_EventID|
|---|---|
| computer1 | [704,701,1501,1500,1085] |
| computer2 | [326,105,302,301,300,102] |
| … | … |

Igual que con el elemento `makelist`, el elemento `makeset` también funciona con datos ordenados y generará las matrices en función del orden de las filas que se le pasen.

## <a name="expanding-lists"></a>Expansión de listas
La operación inversa de `makelist` o `makeset` es `mvexpand`, que expande una lista de valores para separar las filas. Puede expandir a cualquier número de columnas dinámicas, JSON y de matriz. Por ejemplo, puede comprobar la tabla *Heartbeat* para ver las soluciones que envían datos desde los equipos que enviaron un latido durante la última hora:

```Kusto
Heartbeat
| where TimeGenerated > ago(1h)
| project Computer, Solutions
```

| Computer | Soluciones | 
|--------------|----------------------|
| computer1 | "security", "updates", "changeTracking" |
| computer2 | "security", "updates" |
| computer3 | "antiMalware", "changeTracking" |
| … | … |

Use `mvexpand` para mostrar cada valor en una fila separada, en lugar de una lista separada por comas:

```Kusto
Heartbeat
| where TimeGenerated > ago(1h)
| project Computer, split(Solutions, ",")
| mvexpand Solutions
```

| Computer | Soluciones | 
|--------------|----------------------|
| computer1 | "security" |
| computer1 | "updates" |
| computer1 | "changeTracking" |
| computer2 | "security" |
| computer2 | "updates" |
| computer3 | "antiMalware" |
| computer3 | "changeTracking" |
| … | … |


Luego puede usar `makelist` de nuevo para agrupar elementos y, esta vez, ver la lista de equipos por solución:

```Kusto
Heartbeat
| where TimeGenerated > ago(1h)
| project Computer, split(Solutions, ",")
| mvexpand Solutions
| summarize makelist(Computer) by tostring(Solutions) 
```

|Soluciones | list_Computer |
|--------------|----------------------|
| "security" | ["computer1", "computer2"] |
| "updates" | ["computer1", "computer2"] |
| "changeTracking" | ["computer1", "computer3"] |
| "antiMalware" | ["computer3"] |
| … | … |

## <a name="handling-missing-bins"></a>Control de la ausencia de intervalos
Una aplicación útil del elemento `mvexpand` es la necesidad de rellenar valores predeterminados para los intervalos que faltan. Por ejemplo, suponga que busca el tiempo de actividad de un equipo determinado explorando su latido. También quiere ver el origen del latido que se encuentra en la columna _categoría_. Normalmente, se usaría una instrucción summarize simple, de la forma siguiente:

```Kusto
Heartbeat
| where TimeGenerated > ago(12h)
| summarize count() by Category, bin(TimeGenerated, 1h)
```

| Category | TimeGenerated | count_ |
|--------------|----------------------|--------|
| Agente directo | 2017-06-06T17:00:00Z | 15 |
| Agente directo | 2017-06-06T18:00:00Z | 60 |
| Agente directo | 2017-06-06T20:00:00Z | 55 |
| Agente directo | 2017-06-06T21:00:00Z | 57 |
| Agente directo | 2017-06-06T22:00:00Z | 60 |
| … | … | … |

Aunque, en estos resultados, falta el cubo asociado con "2017-06-06T19:00:00Z" porque no hay ningún dato de latido para esa hora. Use la función `make-series` para asignar un valor predeterminado a los cubos vacíos. Esto generará una fila para cada categoría con dos columnas de matriz adicionales, una para los valores y otra para los cubos de tiempo coincidentes:

```Kusto
Heartbeat
| make-series count() default=0 on TimeGenerated in range(ago(1d), now(), 1h) by Category 
```

| Category | count_ | TimeGenerated |
|---|---|---|
| Agente directo | [15,60,0,55,60,57,60,...] | ["2017-06-06T17:00:00.0000000Z","2017-06-06T18:00:00.0000000Z","2017-06-06T19:00:00.0000000Z","2017-06-06T20:00:00.0000000Z","2017-06-06T21:00:00.0000000Z",...] |
| … | … | … |

El tercer elemento de la matriz *count_* es 0, según lo previsto, y hay una marca de tiempo coincidente de "2017-06-06T19:00:00.0000000Z" en la matriz _TimeGenerated_. Aunque, este formato de matriz es difícil de leer. Use el elemento `mvexpand` para expandir las matrices y producir la salida en el mismo formato que el que genera la instrucción `summarize`:

```Kusto
Heartbeat
| make-series count() default=0 on TimeGenerated in range(ago(1d), now(), 1h) by Category 
| mvexpand TimeGenerated, count_
| project Category, TimeGenerated, count_
```

| Category | TimeGenerated | count_ |
|--------------|----------------------|--------|
| Agente directo | 2017-06-06T17:00:00Z | 15 |
| Agente directo | 2017-06-06T18:00:00Z | 60 |
| Agente directo | 2017-06-06T19:00:00Z | 0 |
| Agente directo | 2017-06-06T20:00:00Z | 55 |
| Agente directo | 2017-06-06T21:00:00Z | 57 |
| Agente directo | 2017-06-06T22:00:00Z | 60 |
| … | … | … |



## <a name="narrowing-results-to-a-set-of-elements-let-makeset-toscalar-in"></a>Acotar los resultados a un conjunto de elementos: `let`, `makeset`, `toscalar`, `in`
Un escenario común consiste en seleccionar los nombres de algunas entidades específicas según un conjunto de criterios y, a continuación, filtrar un conjunto de datos diferente para ese conjunto de entidades. Por ejemplo podría buscar los equipos a los que se sabe que les faltan actualizaciones e identificar las direcciones IP que estos equipos llaman:


```Kusto
let ComputersNeedingUpdate = toscalar(
    Update
    | summarize makeset(Computer)
    | project set_Computer
);
WindowsFirewall
| where Computer in (ComputersNeedingUpdate)
```

## <a name="next-steps"></a>Pasos siguientes

Consulte otras lecciones para usar el [lenguaje de consulta de Kusto](/azure/kusto/query/) con datos de registro de Azure Monitor:

- [Operaciones de cadena](string-operations.md)
- [Operaciones de fecha y hora](datetime-operations.md)
- [Funciones de agregación](aggregations.md)
- [Agregaciones avanzadas]()
- [JSON y estructuras de datos](json-data-structures.md)
- [Combinaciones](joins.md)
- [Gráficos](charts.md)

