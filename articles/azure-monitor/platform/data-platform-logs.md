---
title: Registros en Azure Monitor | Microsoft Docs
description: Describe los registros de Azure Monitor que se usan para realizar análisis avanzados de los datos de supervisión.
documentationcenter: ''
author: bwren
manager: carmonm
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/26/2019
ms.author: bwren
ms.openlocfilehash: 413616034dfe7d1f13612ba12ba86014af62c704
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325634"
---
# <a name="logs-in-azure-monitor"></a>Registros en Azure Monitor

> [!NOTE]
> Todos los datos recopilados por Azure Monitor pueden clasificarse como uno de los dos tipos fundamentales: métricas y registros. En este artículo se describen los registros. Consulte [Métricas en Azure Monitor](data-platform-metrics.md) para obtener una descripción detallada de las métricas y [Monitoring data collected by Azure Monitor](data-platform.md) (Supervisión de los datos recopilados por Azure Monitor) para ver una comparación de ambos.

Los registros en Azure Monitor son especialmente útiles para llevar a cabo un análisis complejo de datos procedentes de orígenes diversos. En este artículo se describe cómo se estructuran los registros en Azure Monitor y qué se puede hacer con los datos, y se identifican los diferentes orígenes de datos que almacenan datos en los registros.

> [!NOTE]
> Es importante distinguir entre registros de Azure Monitor y orígenes de datos de registro de Azure. Por ejemplo, los eventos de nivel de suscripción de Azure se escriben en un [registro de actividad](platform-logs-overview.md) que puede verse en el menú de Azure Monitor. La mayoría de los recursos escriben información operativa en un [registro de recursos](platform-logs-overview.md) que se puede reenviar a distintas ubicaciones. Los registros de Azure Monitor son una plataforma de datos de registro que recopila los registros de actividad y los registros de recursos, junto con otros datos de supervisión, para proporcionar un análisis profundo de todo el conjunto de recursos.

## <a name="what-are-azure-monitor-logs"></a>¿Qué son los registros de Azure Monitor?

Los registros de Azure Monitor contienen distintos tipos de datos organizados en registros, donde cada tipo tiene diferentes conjuntos de propiedades. Los registros pueden contener valores numéricos, como métricas de Azure Monitor, pero normalmente contienen datos de texto con descripciones detalladas. Se diferencian aún más de los datos de las métricas en que varían en cuanto a su estructura y a menudo no se recopilan a intervalos regulares. Los datos de telemetría, como los eventos y los seguimientos, se almacenan como registros de Azure Monitor junto con los datos de rendimiento para poder analizarlos de forma combinada.

Un tipo común de entrada de registro es un evento. Los eventos se recopilan de forma esporádica. Una aplicación o servicio los crea, y suelen incluir suficiente información para proporcionar un contexto completo por sí solos. Por ejemplo, un evento puede indicar que se creó o modificó un recurso determinado, se inició un nuevo host en respuesta a un aumento de tráfico o se detectó un error en una aplicación.

 Dado que el formato de los datos puede variar, las aplicaciones pueden crear registros personalizados con la estructura que necesiten. Los datos de métricas se pueden almacenar incluso en registros para combinarlos con otros datos de supervisión para tendencias y otros análisis de datos.


## <a name="what-can-you-do-with-azure-monitor-logs"></a>¿Qué puede hacer con los registros de Azure Monitor?
En la tabla siguiente se enumeran las distintas formas en que se pueden usar los registros en Azure Monitor.


|  | Descripción |
|:---|:---|
| **Análisis** | Use [Log Analytics](../log-query/get-started-portal.md) en Azure Portal para escribir [consultas de registros](../log-query/log-query-overview.md) y analizar los datos de registro de forma interactiva mediante el potente motor de análisis de Data Explorer.<br>Use la [consola de análisis de Application Insights](../log-query/log-query-overview.md) en Azure Portal para escribir consultas de registros y analizar de forma interactiva los datos de registro desde Application Insights. |
| **Visualizar** | Ancle los resultados representados como tablas o gráficos a un [panel de Azure](../../azure-portal/azure-portal-dashboards.md).<br>Cree un [libro](./workbooks-overview.md) para combinar con varios conjuntos de datos en un informe interactivo. <br>Exportar los resultados de una consulta a [Power BI](powerbi.md) para usar diferentes visualizaciones y compartirlos con usuarios fuera de Azure.<br>Exporte los resultados de una consulta a [Grafana](grafana-plugin.md) para aprovechar sus paneles y combinar con otros orígenes de datos.|
| **Alerta** | Configurar un [regla de alerta de registro](alerts-log.md) que envíe una notificación o realice [una acción automatizada](action-groups.md) cuando los resultados de la consulta coincidan con un resultado concreto.<br>Configure una [regla de alerta de métrica](alerts-metric-logs.md) en determinados registros de datos de registro extraídos como métricas. |
| **Recuperar** | Obtenga acceso a los resultados de la consulta de registro desde una línea de comandos mediante la [CLI de Azure](/cli/azure/ext/log-analytics/monitor/log-analytics).<br>Obtenga acceso a los resultados de la consulta de registro desde una línea de comandos mediante [cmdlets de PowerShell](/powershell/module/az.operationalinsights).<br>Obtenga acceso a los resultados de la consulta de registro de una aplicación personalizada con la [API REST](https://dev.loganalytics.io/). |
| **Exportarar** | Cree un flujo de trabajo para recuperar datos de registro y cópielo en una ubicación externa mediante [Logic Apps](../../logic-apps/index.yml). |


## <a name="how-is-data-in-azure-monitor-logs-structured"></a>¿Cómo se estructuran los registros de Azure Monitor?
Los datos recopilados por los registros de Azure Monitor se almacenan en un [área de trabajo de Log Analytics](./design-logs-deployment.md). Cada área de trabajo contiene varias tablas que almacenan datos desde un origen determinado. Aunque todas las tablas comparten [algunas propiedades comunes](log-standard-properties.md), cada una tiene un único conjunto de propiedades que dependen del tipo de datos que almacena. Una nueva área de trabajo tendrá un conjunto estándar de tablas, y las diferentes soluciones de supervisión y otros servicios que escriben en el área de trabajo agregarán más tablas.

Los datos de registro de Application Insights utilizan el mismo motor de Log Analytics como áreas de trabajo, pero se almacenan por separado para cada aplicación supervisada. Cada aplicación tiene un conjunto estándar de tablas para almacenar datos, como solicitudes de aplicación, excepciones y vistas de página.

Las consultas de registros usarán datos desde un área de trabajo de Log Analytics o una aplicación de Application Insights. Puede usar una [consulta entre recursos](../log-query/cross-workspace-query.md) para analizar los datos de la aplicación junto con otros datos de registro o para crear consultas que incluyan varias áreas de trabajo o aplicaciones.

![Áreas de trabajo](media/data-platform-logs/workspaces.png)

## <a name="log-queries"></a>Consultas de registros
Los datos en los registros de Azure Monitor se recuperan mediante una [consulta de registro](../log-query/log-query-overview.md) escrita con el [lenguaje de consulta de Kusto](../log-query/get-started-queries.md), que le permite recuperar, consolidar y analizar rápidamente los datos recopilados. Use [Log Analytics](../log-query/log-query-overview.md) para escribir y probar consultas de registro en Azure Portal. Le permite trabajar con los resultados de forma interactiva o anclarlos a un panel para verlos con otras visualizaciones.

![Log Analytics](media/data-platform-logs/log-analytics.png)

Abra [Log Analytics desde Application Insights](../log-query/log-query-overview.md) para analizar datos de Application Insights.

![Application Insights Analytics](media/data-platform-logs/app-insights-analytics.png)

También puede recuperar datos de registro mediante el uso de la [API de Log Analytics](https://dev.loganalytics.io/documentation/overview) y la [API REST de Application Insights](https://dev.applicationinsights.io/documentation/overview).


## <a name="sources-of-azure-monitor-logs"></a>Orígenes de registros de Azure Monitor
Azure Monitor puede recopilar datos de registro de diversos orígenes, tanto en Azure como desde recursos locales. Las siguientes tablas enumeran los diferentes orígenes de datos disponibles de diferentes recursos que escriben datos en los registros de Azure Monitor. Cada una tiene un vínculo para obtener información detallada sobre cualquier configuración necesaria.

### <a name="azure-tenant-and-subscription"></a>Inquilino y suscripción de Azure

| data | Descripción |
|:---|:---|
| Registros de auditoría de Azure Active Directory | Se configuran a través de la configuración de diagnóstico para cada directorio. Consulte [Integración de registros de Azure AD con registros de Azure Monitor](../../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md).  |
| Registros de actividad | Se almacenan por separado de forma predeterminada y se pueden utilizar para alertas casi en tiempo real. Instale la solución Activity Log Analytics para escribir en el área de trabajo de Log Analytics. Consulte [Recopilar y analizar los registros de actividad de Azure en el área de trabajo de Log Analytics en Azure Monitor](./activity-log.md). |

### <a name="azure-resources"></a>Recursos de Azure

| data | Descripción |
|:---|:---|
| Diagnóstico de recursos | Configure los ajustes de diagnóstico para escribir en los datos de diagnóstico, incluidas las métricas en un área de trabajo de Log Analytics. Consulte [Transmisión de registros de recursos de Azure a Log Analytics](./resource-logs.md#send-to-log-analytics-workspace). |
| Soluciones de supervisión | Las soluciones de supervisión escriben los datos que recopilan en su área de trabajo de Log Analytics. Consulte [Detalles de la recopilación de inventario y datos para la supervisión de soluciones en Azure](../monitor-reference.md) para ver una lista de soluciones. Consulte [Soluciones de supervisión en Azure Monitor](../insights/solutions.md) para obtener más información sobre la instalación y el uso de soluciones. |
| Métricas | Envíe las métricas de la plataforma para los recursos de Azure Monitor a un área de trabajo de Log Analytics para conservar los datos de registro durante períodos más largos y realizar análisis complejos con otros tipos de datos utilizando el [lenguaje de consulta Kusto](/azure/kusto/query/). Consulte [Transmisión de registros de recursos de Azure a Log Analytics](./resource-logs.md#send-to-azure-storage). |
| Almacenamiento de tablas de Azure | Recopile datos de la instancia de Azure Storage donde algunos recursos de Azure escriben datos de supervisión. Consulte [Use Azure blob storage for IIS and Azure table storage for events with Log Analytics](diagnostics-extension-logs.md) (Uso de Azure Blob Storage para el almacenamiento de tablas de Azure e IIS de eventos con Log Analytics). |

### <a name="virtual-machines"></a>Virtual Machines

| data | Descripción |
|:---|:---|
|  Orígenes de datos de agentes | Los orígenes de datos recopilados de agentes de [Windows](agent-windows.md) y [Linux](../learn/quick-collect-linux-computer.md) incluyen eventos, datos de rendimiento y registros personalizados. Consulte [Orígenes de datos de agente en Azure Monitor](data-sources.md) para obtener una lista de orígenes de datos y los detalles de configuración. |
| Soluciones de supervisión | Las soluciones de supervisión escriben los datos que recopilan de los agentes en su área de trabajo de Log Analytics. Consulte [Detalles de la recopilación de inventario y datos para la supervisión de soluciones en Azure](../monitor-reference.md) para ver una lista de soluciones. Consulte [Soluciones de supervisión en Azure Monitor](../insights/solutions.md) para obtener más información sobre la instalación y el uso de soluciones. |
| System Center Operations Manager | Conecte el grupo de administración de Operations Manager a Azure Monitor para recopilar datos de eventos y rendimiento de los agentes locales en registros. Consulte [Connect Operations Manager to Log Analytics](om-agents.md) (Conexión de Operations Manager con Log Analytics ) para obtener más información sobre esta configuración. |


### <a name="applications"></a>APLICACIONES

| data | Descripción |
|:---|:---|
| Solicitudes y excepciones | Los datos detallados sobre las solicitudes y las excepciones de la aplicación se encuentran en las tablas _requests_, _pageViews_ y _exceptions_. Las llamadas a los [componentes externos](../app/asp-net-dependencies.md) están en la tabla _dependencies_. |
| Uso y rendimiento | El rendimiento de la aplicación está disponible en las tablas _requests_, _browserTimings_ y _performanceCounters_. Los datos de [métricas personalizadas](../app/api-custom-events-metrics.md#trackevent) están en la tabla _customMetrics_.|
| Datos de seguimiento | Los resultados del [seguimiento distribuido](../app/distributed-tracing.md) se almacenan en la tabla _traces_. |
| Pruebas de disponibilidad | Los datos de resumen de [pruebas de disponibilidad](../app/monitor-web-app-availability.md) se almacenan en la tabla _availabilityResults_. Los datos detallados de estas pruebas se encuentran en un almacenamiento independiente y son accesibles desde Application Insights en Azure Portal. |

### <a name="insights"></a>Información detallada

| data | Descripción |
|:---|:---|
| Azure Monitor para contenedores | Datos de inventario y de rendimiento recopilados por [Azure Monitor para contenedores](../insights/container-insights-overview.md). Consulte los [detalles de recopilación de datos del contenedor](../insights/container-insights-log-search.md#container-records) para obtener una lista de las tablas. |
| Azure Monitor para máquinas virtuales | Datos de inventario y asignación recopilados por [Azure Monitor para VM](../insights/vminsights-overview.md). Consulte [Cómo consultar registros de Azure Monitor para VM](../insights/vminsights-log-search.md) para obtener más información sobre las consultas de estos datos. |

### <a name="custom"></a>Personalizado 

| data | Descripción |
|:---|:---|
| API DE REST | Escriba datos en un área de trabajo de Log Analytics desde cualquier cliente de REST. Consulte [Envío de datos de registro a Azure Monitor con HTTP Data Collector API](data-collector-api.md) para obtener más información.
| Aplicación lógica | Escriba cualquier dato en un área de trabajo de Log Analytics desde un flujo de trabajo de Logic Apps con la acción **Azure Log Analytics Data Collector**. |

### <a name="security"></a>Seguridad

| data | Descripción |
|:---|:---|
| Azure Security Center | [Azure Security Center](../../security-center/index.yml) almacena los datos que recopila en un área de trabajo de Log Analytics donde se pueden analizar con otros datos del registro. Consulte [Recolección de datos en Azure Security Center](../../security-center/security-center-enable-data-collection.md) para obtener más información sobre la configuración del área de trabajo. |
| Azure Sentinel | [Azure Sentinel](../../sentinel/index.yml) almacena los datos de orígenes de datos en un área de trabajo de Log Analytics. Consulte [Conexión con orígenes de datos](../../sentinel/connect-data-sources.md).  |


## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [plataforma de datos de Azure Monitor](data-platform.md).
- Obtenga más información acerca de las [métricas en Azure Monitor](data-platform-metrics.md).
- Obtenga información sobre la [supervisión de datos disponible](data-sources.md) para diferentes recursos en Azure.

