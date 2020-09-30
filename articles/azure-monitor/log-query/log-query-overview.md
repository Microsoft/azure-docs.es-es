---
title: Introducción al registro de consultas en Azure Monitor | Microsoft Docs
description: Se responden preguntas comunes relacionadas con las consultas de registro y ayuda a comenzar a usarlas.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 06/19/2019
ms.openlocfilehash: 5a47de51a90d926b47f6b465d6c10111407da9c6
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90033173"
---
# <a name="overview-of-log-queries-in-azure-monitor"></a>Introducción a las consultas de registro en Azure Monitor
Las consultas de registro ayudan a aprovechar al máximo el valor de los datos recopilados en [registros de Azure Monitor](../platform/data-platform-logs.md). Un lenguaje de consulta eficaz permite combinar datos de varias tablas, agregar grandes conjuntos de datos y realizar operaciones complejas con una mínima cantidad de código. Se puede responder casi cualquier pregunta y realizar cualquier análisis, siempre y cuando se hayan recopilado los datos de respaldo y comprenda cómo construir la consulta adecuada.

Algunas características de Azure Monitor, como [conclusiones](../insights/insights-overview.md) y [soluciones](../monitor-reference.md) procesan datos de registro sin exponer las consultas subyacentes. Para aprovechar completamente las otras características de Azure Monitor, debe comprender cómo se construyen las consultas y cómo puede usar estas para analizar de forma interactiva los datos en registros de Azure Monitor.

Use este artículo como punto de partida para obtener más información sobre consultas de registros en Azure Monitor. Proporciona respuestas a preguntas comunes y vínculos a otra documentación que ofrece más detalles y lecciones.

## <a name="how-can-i-learn-how-to-write-queries"></a>¿Cómo puedo aprender cómo escribir consultas?
Si desea comenzar de inmediato, puede empezar con los siguientes tutoriales:

- [Introducción a Log Analytics en Azure Monitor](get-started-portal.md).
- [Introducción a consultas de registros en Azure Monitor](get-started-queries.md).

Cuando comprenda los conceptos básicos, realice varias lecciones con sus propios datos o datos de nuestro entorno de demostración, comenzando por la siguiente: 

- [Trabajo con cadenas en consultas de registro de Azure Monitor](string-operations.md)
 
## <a name="what-language-do-log-queries-use"></a>¿Qué lenguaje usan las consultas de registro?
Los registros de Azure Monitor se basan en [Azure Data Explorer](/azure/data-explorer) y las consultas de registros se escriben con el mismo lenguaje de consulta de Kusto (KQL). Esto es un lenguaje enriquecido diseñado para ser fácil de leer y crear, y debería poder empezar a usarlo con asistencia mínima.

Consulte la [documentación sobre KQL de Azure Data Explorer](/azure/kusto/query) para obtener documentación completa sobre KQL y referencia de las distintas funciones disponibles.<br>
Consulte [Introducción a las consultas de registro en Azure Monitor](get-started-queries.md) para ver un tutorial rápido del lenguaje con los datos de registros de Azure Monitor.
Consulte [Diferencias en el lenguaje de consulta de los registros de Azure Monitor](data-explorer-difference.md) para conocer las pequeñas diferencias en la versión de KQL que Azure Monitor usa.

## <a name="what-data-is-available-to-log-queries"></a>¿Qué datos están disponibles para registrar consultas?
Todos los datos recopilados en los registros de Azure Monitor están disponibles para recuperar y analizar en consultas de registros. Diferentes orígenes de datos escribirán sus datos en tablas diferentes, pero puede incluir varias tablas en una sola consulta para analizar datos entre varios orígenes. Cuando compila una consulta, primero debe determinar las tablas que tienen los datos que está buscando. Consulte [Estructura de los registros de Azure Monitor](logs-structure.md) para obtener una explicación de cómo se estructuran los datos.

## <a name="what-does-a-log-query-look-like"></a>¿Qué aspecto tiene una consulta de registro?
Una consulta puede ser tan simple como un nombre de tabla única para recuperar todos los registros de esa tabla:

```Kusto
Syslog
```

O bien, podría filtrar registros determinados, resumirlos y visualizar los resultados en un gráfico:

```
SecurityEvent
| where TimeGenerated > ago(7d)
| where EventID == 4625
| summarize count() by Computer, bin(TimeGenerated, 1h)
| render timechart 
```

Para un análisis más complejo, puede recuperar datos de varias tablas mediante una combinación para analizar los resultados entre sí.

```Kusto
app("ContosoRetailWeb").requests
| summarize count() by bin(timestamp,1hr)
| join kind= inner (Perf
    | summarize avg(CounterValue) 
      by bin(TimeGenerated,1hr))
on $left.timestamp == $right.TimeGenerated
```
Incluso si no está familiarizado con KQL, debería poder averiguar al menos la lógica básica que estas consultas usan. Comienzan por el nombre de una tabla y luego agregan varios comandos para filtrar y procesar los datos. Una consulta puede usar cualquier número de comandos y se pueden escribir consultas más complejas a medida que se familiarice con los distintos comandos de KQL disponibles.

Consulte [Introducción a las consultas de registro en Azure Monitor](get-started-queries.md) para obtener un tutorial sobre las consultas de registro en el que se introduce el lenguaje y las funciones comunes.<br>


## <a name="what-is-log-analytics"></a>¿Qué es Log Analytics?
Log Analytics es la herramienta principal en Azure Portal para escribir consultas de registros y analizar sus resultados de forma interactiva. Incluso si una consulta de registro se usa en otro lugar en Azure Monitor, normalmente deberá escribir y probar la consulta primero mediante Log Analytics.

Log Analytics se puede iniciar desde varios lugares en Azure Portal. El ámbito de los datos disponibles para Log Analytics se determina según el modo en que se inicia. Consulte [Ámbito de la consulta](scope.md) para obtener más detalles.

- Seleccione **Registros** desde el menú **Azure Monitor** o el menú **Áreas de trabajo de Log Analytics**.
- Seleccione **Registros** en la página **Introducción** de una aplicación de Application Insights.
- Seleccione **Registros** en el menú de un recurso de Azure.

![Log Analytics](media/log-query-overview/log-analytics.png)

Consulte [Introducción a Log Analytics en Azure Monitor](get-started-portal.md) para obtener un tutorial sobre Log Analytics en el que se introducen varias de sus características.

## <a name="where-else-are-log-queries-used"></a>¿Dónde se usan las consultas de registros?
Además de trabajar de manera interactiva con consultas de registro y sus resultados en Log Analytics, entre las áreas de Azure Monitor en las que se usan las consultas se incluyen las siguientes:

- **Reglas de alertas.** [Las reglas de alertas](../platform/alerts-overview.md) identifican de manera proactiva los problemas de datos del área de trabajo.  Cada regla de alertas se basa en una búsqueda de registros que se ejecuta automáticamente a intervalos regulares.  Los resultados se inspeccionan para determinar si se debe crear una alerta.
- **Paneles.** Puede anclar los resultados de cualquier consulta en un [panel de Azure](../learn/tutorial-logs-dashboards.md), que le permitirá visualizar los datos de registros y métricas en conjunto y, opcionalmente, compartirlos con otros usuarios de Azure.
- **Vistas.**  Puede crear visualizaciones de datos que se incluyan en los paneles de usuario con el [diseñador de vistas](../platform/view-designer.md).  Las consultas de registros proporcionan los datos utilizados por [iconos](../platform/view-designer-tiles.md) y [elementos de visualización](../platform/view-designer-parts.md) en cada vista.  
- **Exportación.**  Cuando importe datos de registro de Azure Monitor en Excel o [Power BI](../platform/powerbi.md), cree una consulta de registros para definir los datos que se van a exportar.
- **PowerShell.** Puede ejecutar un script de PowerShell desde una línea de comandos o un runbook de Azure Automation que use [Get-AzOperationalInsightsSearchResults](/powershell/module/az.operationalinsights/get-azoperationalinsightssearchresult) para recuperar los datos de registro de Azure Monitor.  Este cmdlet requiere una consulta para determinar los datos que se van a recuperar.
- **API de registros de Azure Monitor.**  La [API de registros de Azure Monitor](https://dev.loganalytics.io) permite que cualquier cliente de API REST recupere datos de registro del área de trabajo.  La solicitud de API incluye una consulta que se ejecuta en Azure Monitor para determinar los datos que se van a recuperar.


## <a name="next-steps"></a>Pasos siguientes
- Realice un [tutorial sobre el uso de Log Analytics en Azure Portal](get-started-portal.md).
- Realice un [tutorial sobre cómo escribir consultas](get-started-queries.md).
