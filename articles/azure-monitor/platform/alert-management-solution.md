---
title: Solución Alert Management en Log Analytics | Microsoft Azure
description: La solución Administración de alertas de Log Analytics le ayuda a analizar todas las alertas de su entorno.  Además de consolidar las alertas generadas dentro de Log Analytics, importa las alertas de los grupos de administración conectados de System Center Operations Manager en Log Analytics.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 01/19/2018
ms.openlocfilehash: bd30a840327eaf338aec89c12ff8eb5d87c60c56
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322404"
---
# <a name="alert-management-solution-in-azure-log-analytics"></a>Solución Alert Management en Azure Log Analytics

![Icono Administración de alertas](media/alert-management-solution/icon.png)

> [!NOTE]
>  Azure Monitor ahora es compatible con funcionalidades mejoradas para la [administración de alertas a escala](https://aka.ms/azure-alerts-overview), incluidas las generadas por las [herramientas de supervisión como System Center Operations Manager, Zabbix o Nagios](https://aka.ms/managing-alerts-other-monitoring-services).
>  


La solución Administración de alertas le ayuda a analizar todas las alertas del repositorio de Log Analytics.  Estas alertas pueden proceder de diversos orígenes, incluidos los [creados por Log Analytics ](./alerts-overview.md) o los [importados de Nagios o Zabbix](../learn/quick-collect-linux-computer.md). La solución también importa alertas desde cualquier [grupo de administración conectado de System Center Operations Manager](./om-agents.md).

## <a name="prerequisites"></a>Prerrequisitos
La solución funciona con todos los registros del repositorio de Log Analytics con un tipo de **Alerta**, por lo que debe realizar las configuraciones que sean necesarias para recopilar estos registros.

- Para las alertas de Log Analytics, [cree reglas de alerta](./alerts-overview.md) para crear registros de alerta directamente en el repositorio.
- Para las alertas de Nagios y Zabbix, [configure esos servidores](../learn/quick-collect-linux-computer.md) para que envíen alertas a Log Analytics.
- Para las alertas de System Center Operations Manager,[conecte el grupo de administración de Operations Manager al área de trabajo de Log Analytics](./om-agents.md).  Las alertas creadas en System Center Operations Manager se importan en Log Analytics.  

## <a name="configuration"></a>Configuración
Agregue la solución Alert Management al área de trabajo de Log Analytics mediante el proceso descrito en [Adición de soluciones](../insights/solutions.md). No es necesario realizar ninguna configuración más.

## <a name="management-packs"></a>Módulos de administración
Si el grupo de administración de System Center Operations Manager está conectado al área de trabajo de Log Analytics, se instalarán los siguientes módulos de administración en System Center Operations Manager al agregar esta solución.  No es necesario realizar tareas de configuración o mantenimiento de estos módulos de administración.

* Administración de alertas de Microsoft System Center Advisor (Microsoft.IntelligencePacks.AlertManagement)

Para obtener más información sobre cómo se actualizan los módulos de administración de soluciones, consulte [Conexión de Operations Manager con Log Analytics](./om-agents.md).

## <a name="data-collection"></a>datos, recopilación
### <a name="agents"></a>Agentes
En la tabla siguiente se describen los orígenes conectados que son compatibles con esta solución.

| Origen conectado | Soporte técnico | Descripción |
|:--- |:--- |:--- |
| [Agentes de Windows](agent-windows.md) | No |Los agentes directos de Windows no generan alertas.  Se pueden crear alertas de Log Analytics de eventos y datos de rendimiento recopilados desde agentes de Windows. |
| [Agentes de Linux](../learn/quick-collect-linux-computer.md) | No |Los agentes directos de Linux no generan alertas.  Se pueden crear alertas de Log Analytics de eventos y datos de rendimiento recopilados desde agentes de Linux.  Las alertas de Nagios y Zabbix se recopilan desde los servidores que requieren el agente de Linux. |
| [Grupo de administración de System Center Operations](./om-agents.md) |Sí |Las alertas que se generan en agentes de Operations Manager se entregan al grupo de administración y luego se reenvían a Log Analytics.<br><br>No se requiere ninguna conexión directa entre los agentes de Operations Manager y Log Analytics. Los datos de alerta se reenvían desde el grupo de administración al repositorio de Log Analytics. |


### <a name="collection-frequency"></a>Frecuencia de recopilación
- Todos los registros de alerta están disponibles para la solución en cuanto se almacenan en el repositorio.
- Los datos de alerta se envían desde el grupo de administración de Operations Manager a Log Analytics cada tres minutos.  

## <a name="using-the-solution"></a>Uso de la solución
Al agregar la solución Alert Management al área de trabajo de Log Analytics, se agrega el icono **Alert Management** al panel.  Este icono muestra un recuento y una representación gráfica del número de alertas actualmente activas que se generaron en las últimas 24 horas.  Este intervalo de tiempo no se puede cambiar.

![Icono Administración de alertas](media/alert-management-solution/tile.png)

Haga clic en el icono **Administración de alertas** para abrir el panel **Administración de alertas**.  El panel incluye las columnas de la tabla siguiente.  Cada columna muestra las diez principales alertas por recuento que coinciden con los criterios de esa columna para el ámbito e intervalo de tiempo especificados.  Puede ejecutar una búsqueda de registros que proporcione toda la lista haciendo clic en **Ver todo** en la parte inferior de la columna o haciendo clic en el encabezado de columna.

| Columna | Descripción |
|:--- |:--- |
| Alertas críticas |Todas las alertas con una gravedad crítica agrupadas por nombre de alerta.  Haga clic en un nombre de la alerta para ejecutar una búsqueda de registros que devuelva todos los registros de esa alerta. |
| Alertas de advertencia |Todas las alertas con una gravedad de advertencia agrupadas por nombre de alerta.  Haga clic en un nombre de la alerta para ejecutar una búsqueda de registros que devuelva todos los registros de esa alerta. |
| Alertas activas de System Center Operations Manager |Todas las alertas recopiladas desde Operations Manager con cualquier estado distinto de *Cerrado* agrupadas por el origen que generó la alerta. |
| Todas las alertas activas |Todas las alertas con cualquier gravedad agrupadas por nombre de alerta. Solo incluye las alertas de Operations Manager con cualquier estado distinto de *Cerrado*. |

Si se desplaza a la derecha, el panel mostrará varias consultas comunes en las que puede hacer clic para realizar una [búsqueda de registros](../log-query/log-query-overview.md) para datos de alertas.

![Panel Administración de alertas](media/alert-management-solution/dashboard.png)


## <a name="log-analytics-records"></a>Registros de Log Analytics
La solución Administración de alertas analiza todos los registros con un tipo de **Alerta**.  La solución no recopila directamente las alertas creadas por Log Analytics o recopiladas desde Nagios o Zabbix.

La solución importa alertas desde System Center Operations Manager y crea un registro correspondiente para cada una con un tipo de **Alerta** y un SourceSystem de **OpsManager**.  Estos registros tienen las propiedades de la tabla siguiente:  

| Propiedad | Descripción |
|:--- |:--- |
| `Type` |*Alerta* |
| `SourceSystem` |*OpsManager* |
| `AlertContext` |Detalles del elemento de datos que provocó la alerta generados en formato XML. |
| `AlertDescription` |Descripción detallada de la alerta. |
| `AlertId` |GUID de la alerta. |
| `AlertName` |Nombre de la alerta |
| `AlertPriority` |Nivel de prioridad de la alerta. |
| `AlertSeverity` |Nivel de gravedad de la alerta. |
| `AlertState` |Estado de resolución más reciente de la alerta. |
| `LastModifiedBy` |Nombre del usuario que modificó por última vez la alerta. |
| `ManagementGroupName` |Nombre del grupo de administración donde se generó la alerta. |
| `RepeatCount` |Número de veces que se generó la misma alerta para el mismo objeto supervisado desde que se resolvió. |
| `ResolvedBy` |Nombre del usuario que resolvió la alerta. Vacío si la alerta todavía no se ha resuelto. |
| `SourceDisplayName` |Nombre para mostrar del objeto de supervisión que generó la alerta. |
| `SourceFullName` |Nombre completo del objeto de supervisión que generó la alerta. |
| `TicketId` |Id. de vale para la alerta si el entorno de System Center Operations Manager está integrado con un proceso para asignar vales para las alertas.  Vacío si no se asigna ningún vale. |
| `TimeGenerated` |Fecha y hora en que se creó la alerta. |
| `TimeLastModified` |Fecha y hora en que se cambió la alerta. |
| `TimeRaised` |Fecha y hora en que se generó la alerta. |
| `TimeResolved` |Fecha y hora en que se resolvió la alerta. Vacío si la alerta todavía no se ha resuelto. |

## <a name="sample-log-searches"></a>Búsquedas de registros de ejemplo
La tabla siguiente proporciona búsquedas de registros de ejemplo para los registros de alerta recopilados por esta solución: 

| Consultar | Descripción |
|:---|:---|
| Alert &#124; where SourceSystem == "OpsManager" and AlertSeverity == "error" and TimeRaised > ago(24h) |Alertas críticas generadas durante las últimas 24 horas |
| Alert &#124; where AlertSeverity == "warning" and TimeRaised > ago(24h) |Alertas de advertencia generadas durante las últimas 24 horas |
| Alert &#124; where SourceSystem == "OpsManager" and AlertState != "Closed" and TimeRaised > ago(24h) &#124; summarize Count = count() by SourceDisplayName |Orígenes con alertas activas generadas durante las últimas 24 horas |
| Alert &#124; where SourceSystem == "OpsManager" and AlertSeverity == "error" and TimeRaised > ago(24h) and AlertState != "Closed" |Alertas críticas generadas durante las últimas 24 horas que todavía están activas |
| Alert &#124; where SourceSystem == "OpsManager" and TimeRaised > ago(24h) and AlertState == "Closed" |Alertas generadas durante las últimas 24 horas que ahora están cerradas |
| Alert &#124; where SourceSystem == "OpsManager" and TimeRaised > ago(1d) &#124; summarize Count = count() by AlertSeverity |Alertas generadas durante el último día agrupadas por su gravedad |
| Alert &#124; where SourceSystem == "OpsManager" and TimeRaised > ago(1d) &#124; sort by RepeatCount desc |Alertas generadas durante el último día ordenadas por su valor de número de repeticiones |



## <a name="next-steps"></a>Pasos siguientes
* Obtenga información sobre [alertas en Log Analytics](./alerts-overview.md) para más detalles sobre la generación de alertas desde Log Analytics.

