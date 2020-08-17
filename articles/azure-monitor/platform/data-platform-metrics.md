---
title: Métricas en Azure Monitor | Microsoft Docs
description: Describe las métricas en Azure Monitor que son datos de supervisión ligeros capaces de admitir escenarios casi en tiempo real.
documentationcenter: ''
author: bwren
manager: carmonm
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/26/2019
ms.author: bwren
ms.openlocfilehash: b05007e2ea7815afbba2a7a71368686cf7c049fb
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325617"
---
# <a name="metrics-in-azure-monitor"></a>Métricas en Azure Monitor

> [!NOTE]
> La plataforma de datos de Azure Monitor se basa en dos tipos de datos fundamentales: métricas y registros. En este artículo se describen las métricas. Consulte [Registros en Azure Monitor](data-platform-logs.md) para obtener una descripción detallada de los registros y [Plataforma de datos de Azure Monitor](data-platform.md) para ver una comparación de ambos.

Las métricas en Azure Monitor son ligeras y capaces de admitir escenarios casi en tiempo real, lo que hace que sean especialmente útiles para las alertas y una detección rápida de problemas. En este artículo se describe cómo se estructuran las métricas, qué se puede hacer con ellas y se identifican diferentes orígenes de datos que almacenan datos en las métricas.

## <a name="what-are-metrics"></a>¿Qué son las métricas?
Las métricas son valores numéricos que describen algunos aspectos de un sistema en un momento dado. Las métricas se recopilan en intervalos regulares y son útiles para las alertas, dado que se pueden muestrear con frecuencia y se puede activar una alerta con rapidez con una lógica relativamente sencilla.

## <a name="what-can-you-do-with-azure-monitor-metrics"></a>¿Qué puede hacer con las métricas de Azure Monitor?
En la tabla siguiente se enumeran las distintas formas en que se pueden usar datos de métricas en Azure Monitor.

|  | Descripción |
|:---|:---|
| **Análisis** | Use el [Explorador de métricas](metrics-charts.md) para analizar las métricas recopiladas en un gráfico y compare las métricas de diferentes recursos. |
| **Visualizar** | Ancle un gráfico del Explorador de métricas en un [panel de Azure](../learn/tutorial-app-dashboards.md).<br>Cree un [libro](./workbooks-overview.md) para combinar con varios conjuntos de datos en un informe interactivo. Exporte los resultados de una consulta a [Grafana](grafana-plugin.md) para aprovechar sus paneles y combinarlos con otros orígenes de datos. |
| **Alerta** | Configure una [regla de alertas de métricas](alerts-metric.md) que envíe una notificación o realice una [acción automatizada](action-groups.md) cuando el valor de la métrica cruce un umbral. |
| **Automatización** |  Use la [escalabilidad automática](autoscale-overview.md) para aumentar o disminuir los recursos según un valor de métrica que cruce un umbral. |
| **Exportarar** | [Redirija las métricas a los registros](./resource-logs.md#send-to-azure-storage) para analizar los datos en las métricas en Microsoft Azure junto con los datos en los registros de Azure Monitor y para almacenar los valores de las métricas durante un período superior a 93 días.<br>Haga streaming de métricas a un [centro de eventos](stream-monitoring-data-event-hubs.md) para redirigirlas a sistemas externos. |
| **Recuperar** | Obtenga acceso a los valores de métricas desde una línea de comandos mediante [cmdlets de PowerShell](/powershell/module/az.applicationinsights).<br>Obtenga acceso a los valores de métricas de aplicaciones personalizadas con [API REST](rest-api-walkthrough.md).<br>Obtenga acceso a los valores de métricas desde una línea de comandos mediante la [CLI](/cli/azure/monitor/metrics). |
| **Archivar** | [Archivar](./platform-logs-overview.md) el historial de rendimiento o estado del recurso para fines de cumplimiento, auditoría o creación de informes sin conexión. |

## <a name="how-is-data-in-azure-monitor-metrics-structured"></a>¿Cómo se estructuran los datos de métricas de Azure Monitor?
Los datos recopilados por las métricas en Azure Monitor se almacenan en una base de datos de serie temporal que está optimizada para el análisis de los datos con marca de tiempo. Cada conjunto de valores de métricas es una serie temporal con las siguientes propiedades:

* Hora en que se recopiló el valor.
* Recurso al que el valor está asociado.
* Espacio de nombres que actúa como una categoría para la métrica.
* Nombre de la métrica.
* El propio valor.
* Algunas métricas pueden tener varias dimensiones, como se describe en la sección [Métricas multidimensionales](#multi-dimensional-metrics). Las métricas personalizadas pueden tener hasta 10 dimensiones.

## <a name="multi-dimensional-metrics"></a>Métricas multidimensionales
Uno de los desafíos de los datos de métricas es que a menudo tienen información limitada para proporcionar contexto para los valores recopilados. Azure Monitor aborda este desafío con métricas multidimensionales. Las dimensiones de una métrica son pares nombre-valor que transportan datos adicionales para describir el valor de la métrica. Por ejemplo, una métrica _Espacio disponible en disco_ puede tener una dimensión denominada _Unidad_ con los valores _C:_ o _D:_ , que permitiría ver cualquier espacio disponible en disco de todas las unidades o de cada unidad individual.

En el ejemplo siguiente se muestran dos conjuntos de datos para una métrica hipotética denominada _Rendimiento de la red_. El primer conjunto de datos no tiene ninguna dimensión. El segundo conjunto de datos muestra los valores con dos dimensiones, _Dirección IP_ y _Dirección_:

### <a name="network-throughput"></a>Rendimiento de la red

| Timestamp     | Valor de métrica |
| ------------- |:-------------|
| 9/8/2017 8:14 | 1331,8 Kbps |
| 9/8/2017 8:15 | 1141,4 Kbps |
| 9/8/2017 8:16 | 1110,2 Kbps |

Esta métrica no dimensional solo puede responder una pregunta básica, como “¿cuál era el rendimiento de la red en un momento dado?”.

### <a name="network-throughput--two-dimensions-ip-and-direction"></a>Rendimiento de la red + dos dimensiones (“IP” y “Dirección”)

| Timestamp     | Dimensión “IP”   | Dimensión “Dirección” | Valor de métrica|
| ------------- |:-----------------|:------------------- |:-----------|
| 9/8/2017 8:14 | IP="192.168.5.2" | Dirección="Envío"    | 646,5 Kbps |
| 9/8/2017 8:14 | IP="192.168.5.2" | Dirección="Recepción" | 420,1 Kbps |
| 9/8/2017 8:14 | IP="10.24.2.15"  | Dirección="Envío"    | 150,0 Kbps |
| 9/8/2017 8:14 | IP="10.24.2.15"  | Dirección="Recepción" | 115,2 Kbps |
| 9/8/2017 8:15 | IP="192.168.5.2" | Dirección="Envío"    | 515,2 Kbps |
| 9/8/2017 8:15 | IP="192.168.5.2" | Dirección="Recepción" | 371,1 Kbps |
| 9/8/2017 8:15 | IP="10.24.2.15"  | Dirección="Envío"    | 155,0 Kbps |
| 9/8/2017 8:15 | IP="10.24.2.15"  | Dirección="Recepción" | 100,1 Kbps |

Esta métrica puede responder a preguntas como “¿cuál era el rendimiento de la red para cada dirección IP?” y “¿cuántos datos se enviaron en comparación los datos que se recibieron?”. Las métricas multidimensionales llevan valores analíticos y de diagnóstico adicionales en comparación con las métricas no dimensionales.

## <a name="interacting-with-azure-monitor-metrics"></a>Interactuar con las métricas de Azure Monitor
Use el [Explorador de métricas](metrics-charts.md) para analizar de forma interactiva los datos en la base de datos de métricas y representar los valores de varias métricas con el tiempo. Puede anclar los gráficos a un panel para verlos con otras visualizaciones. También puede recuperar las métricas con la [API REST de supervisión de Azure](rest-api-walkthrough.md).

![Explorador de métricas](media/data-platform/metrics-explorer.png)

## <a name="sources-of-azure-monitor-metrics"></a>Orígenes de métricas de Azure Monitor
Las métricas que Azure Monitor recopila proceden de tres orígenes fundamentales. Una vez que estas métricas se recopilan en la base de datos de métricas de Azure Monitor, se pueden evaluar juntas independientemente de su origen.

Las **métricas de plataforma** se crean mediante los recursos de Azure y brindan visibilidad sobre su estado y rendimiento. Cada tipo de recurso crea un [conjunto distinto de métricas](metrics-supported.md) sin ninguna configuración necesaria. Las métricas de plataforma se recopilan de los recursos de Azure con una frecuencia de un minuto, a menos que se especifique lo contrario en la definición de la métrica. 

Las **métricas de SO invitado** se recopilan desde el sistema operativo invitado de una máquina virtual. Habilite las métricas de SO invitado para máquinas virtuales Windows con la [extensión de diagnóstico de Windows (WAD)](./diagnostics-extension-overview.md) y para las máquinas virtuales Linux con el [agente de InfluxData Telegraf](https://www.influxdata.com/time-series-platform/telegraf/).

Las **métricas de aplicación** se crean mediante Application Insights para sus aplicaciones supervisadas y le ayudan a detectar problemas de rendimiento y a realizar un seguimiento de las tendencias de uso de la aplicación. Incluye valores como _Tiempo de respuesta del servidor_ y _Excepciones del explorador_.

Las **métricas personalizadas** son métricas que se definen además de las métricas estándar que están disponibles automáticamente. También puede [definir métricas personalizadas en la aplicación](../app/api-custom-events-metrics.md) que se supervisa mediante Application Insights o crear métricas personalizadas para un servicio de Azure con la [API de métricas personalizadas](metrics-store-custom-rest-api.md).

## <a name="retention-of-metrics"></a>Retención de métricas
Para la mayoría de los recursos en Azure, las métricas se almacenan durante 93 días. Hay algunas excepciones:

**Métricas de SO invitado**
-   **Métricas clásicas de SO invitado**. Estos son los contadores de rendimiento que recopila la [extensión de diagnóstico de Windows (WAD)](./diagnostics-extension-overview.md) o la [extensión de diagnóstico de Linux (LAD)](../../virtual-machines/extensions/diagnostics-linux.md) y que se enrutan a una cuenta de almacenamiento de Azure. La retención de estas métricas es de 14 días.
-   **Métricas de SO invitado enviadas a las métricas de Azure Monitor**. Estos son los contadores de rendimiento recopilados por [Windows Diagnostic Extension (WAD)](diagnostics-extension-overview.md) que se envían al [receptor de datos de Azure Monitor](diagnostics-extension-overview.md#data-destinations) o que se envían mediante el [Agente de InfluxData Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) en las máquinas Linux. La retención de estas métricas es de 93 días.
-   **Métricas de SO invitado que recopila el agente de Log Analytics**. Estos son contadores de rendimiento que recopila el agente de Log Analytics y que se envían a un área de trabajo de Log Analytics. La retención de estas métricas es de 31 días y se puede ampliar a un máximo de 2 años.

**Métricas basadas en registros de Application Insights**. 
- Entre bastidores, las [métricas basadas en registros](../app/pre-aggregated-metrics-log-metrics.md) se traducen en consultas de registros. Su retención coincide con la retención de eventos en registros subyacentes. Para los recursos de Application Insights, los registros se almacenan durante 90 días.


> [!NOTE]
> También puede [enviar métricas de plataforma para recursos de Azure Monitor a un área de trabajo de Log Analytics](./resource-logs.md#send-to-azure-storage) para tendencias a largo plazo.





## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [plataforma de datos de Azure Monitor](data-platform.md).
- Más información sobre los [datos de registro en Azure Monitor](data-platform-logs.md).
- Obtenga información sobre la [supervisión de datos disponible](data-sources.md) para diferentes recursos en Azure.

