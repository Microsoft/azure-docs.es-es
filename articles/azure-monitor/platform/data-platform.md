---
title: Plataforma de datos de Azure Monitor | Microsoft Docs
description: Los datos de supervisión recopilados por Azure Monitor se dividen en métricas que son ligeras y capaces de admitir escenarios casi en tiempo real y registros que se usan para realizar análisis avanzados.
documentationcenter: ''
author: bwren
manager: carmonm
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/26/2019
ms.author: bwren
ms.openlocfilehash: e87ddd243aa248b896a26e6389ac1a219579a06d
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325583"
---
# <a name="azure-monitor-data-platform"></a>Plataforma de datos de Azure Monitor

Para permitir visibilidad entre los entornos informáticos complejos actuales que ejecutan aplicaciones distribuidas que se basan tanto en servicios en la nube como locales, se requiere la recopilación de datos operativos de todas las capas y todos los componentes del sistema distribuido. Es necesario poder extraer información detallada de estos datos y consolidarla en una única vista con distintas perspectivas para respaldar a la ingente cantidad de partes interesadas de su organización.

[Azure Monitor](../overview.md) recopila datos de diversos orígenes y los agrega a una plataforma de datos común que puede usarse para el análisis, la visualización y la alerta. Esta plataforma proporciona una experiencia coherente basada en los datos de varios orígenes, que le ofrece información detallada de todos los recursos supervisados e incluso con datos de otros servicios que almacenan sus datos en Azure Monitor.


![Introducción a Azure Monitor](media/data-platform/overview.png)

## <a name="observability-data-in-azure-monitor"></a>Datos de visibilidad en Azure Monitor
Las métricas, los registros y los seguimientos distribuidos se consideran comúnmente los tres pilares de la visibilidad. Estos son los diferentes tipos de datos que debe recopilar y analizar una herramienta de supervisión con el fin de proporcionar suficiente visibilidad sobre un sistema supervisado. Para conseguir visibilidad, se pueden correlacionar los datos de varios pilares y agregar los datos de todo el conjunto de recursos objeto de supervisión. Como Azure Monitor almacena juntos datos de varios orígenes, los datos se pueden correlacionar y analizar mediante un conjunto común de herramientas. También se correlacionan datos de varias suscripciones e inquilinos de Azure, además de hospedarse datos de otros servicios.

Los recursos de Azure generan una cantidad importante de datos de supervisión. Azure Monitor consolida estos datos junto con los datos de supervisión procedentes de otros orígenes en una plataforma de métricas o de registros. Cada una está optimizado para unos escenarios de supervisión determinados, y cada una admite diferentes características de Azure Monitor. Características como análisis de datos, visualizaciones o alertas requieren que comprenda las diferencias para que puedan implementar el escenario necesario de la manera más eficaz y rentable. Las conclusiones en Azure Monitor, como [Application Insights](../app/app-insights-overview.md) o [Azure Monitor para VM](../insights/vminsights-overview.md) disponen de herramientas de análisis que le permiten centrarse en el escenario de supervisión determinado sin necesidad de comprender las diferencias entre los dos tipos de datos. 


### <a name="metrics"></a>Métricas
Las [métricas](data-platform-metrics.md) son valores numéricos que describen algún aspecto de un sistema en un momento dado. Se recopilan a intervalos regulares y se identifican con una marca de tiempo, un nombre, un valor y una o más etiquetas de definición. Las métricas se pueden agregar con diversos algoritmos, en comparación con otras métricas, y analizarse para ver las tendencias con el paso del tiempo. 

Las métricas en Azure Monitor se almacenan en una base de datos de serie temporal que está optimizada para el análisis de los datos con marca de tiempo. Esta situación hace que las métricas sean especialmente adecuadas para la alerta y la detección rápida de problemas. Pueden indicar cómo funciona el sistema, pero normalmente deben combinarse con registros para identificar la causa raíz de los problemas.

Hay métricas disponibles para análisis interactivos en Azure Portal con el [Explorador de métricas](./metrics-getting-started.md). Se pueden agregar a un [panel de Azure](../learn/tutorial-app-dashboards.md) para su visualización en combinación con otros datos y usarse para la [alerta](alerts-metric.md) en tiempo real.

Conozca más sobre las métricas de Azure Monitor, incluidos sus orígenes de datos en [Métricas en Azure Monitor](data-platform-metrics.md).

### <a name="logs"></a>Registros
Los [registros](data-platform-logs.md) son eventos que se produjeron dentro del sistema. Pueden contener diferentes tipos de datos y pueden estar estructurados o tener texto de formato libre con una marca de tiempo. Se pueden crear esporádicamente como eventos en las entradas de registro de generación del entorno, y un sistema con mucha carga normalmente genera más volumen de registro.

Los registros de Azure Monitor se almacenan en un área de trabajo de Log Analytics basado en [Azure Data Explorer](/azure/data-explorer/) que proporciona un motor de análisis eficaz y un [lenguaje de consulta completo](/azure/kusto/query/). Los registros normalmente proporcionan información suficiente para ofrecer un contexto completo del problema identificado y son valiosos para identificar la causa de los problemas.

> [!NOTE]
> Es importante distinguir entre registros de Azure Monitor y orígenes de datos de registro de Azure. Por ejemplo, los eventos de nivel de suscripción de Azure se escriben en un [registro de actividad](platform-logs-overview.md) que puede verse en el menú de Azure Monitor. La mayoría de los recursos escriben información operativa en un [registro de recursos](platform-logs-overview.md) que se puede reenviar a distintas ubicaciones. Los registros de Azure Monitor son una plataforma de datos de registro que recopila los registros de actividad y los registros de recursos, junto con otros datos de supervisión, para proporcionar un análisis profundo de todo el conjunto de recursos.


 Puede trabajar con [consultas de registro](../log-query/log-query-overview.md) de manera interactiva con [Log Analytics](../log-query/log-query-overview.md) en Azure Portal o agregar los resultados a un [panel de Azure](../learn/tutorial-app-dashboards.md) para su visualización en combinación con otros datos. También puede crear [alertas de registro](alerts-log.md), que desencadenarán una alerta según los resultados de una consulta de programación.

Aprenda más sobre los registros de Azure Monitor, como sus orígenes de datos en [Registros en Azure Monitor](data-platform-logs.md).

### <a name="distributed-traces"></a>Seguimientos distribuidos
Los seguimientos son series de eventos relacionados que siguen la solicitud de un usuario por un sistema distribuido. Pueden usarse para determinar el comportamiento del código de aplicación y el rendimiento de diferentes transacciones. Aunque con frecuencia los registros se crearán mediante componentes individuales de un sistema distribuido, un seguimiento mide el funcionamiento y el rendimiento de la aplicación en todo el conjunto de componentes.

El seguimiento distribuido en Azure Monitor se habilita con el [SDK de Application Insights](../app/distributed-tracing.md), y los datos de seguimiento se almacenan con otros datos de registro de aplicaciones recopilados por Application Insights. De esta forma están disponibles para las mismas herramientas de análisis que otros datos de registro, como consultas de registros, paneles y alertas.

Aprenda más sobre el seguimiento distribuido en [What ' s seguimiento distribuido?](../app/distributed-tracing.md) (¿Qué es el seguimiento distribuido?).


## <a name="compare-azure-monitor-metrics-and-logs"></a>Comparación de las métricas y los registros de Azure Monitor

En la tabla siguiente se comparan las métricas y los registros de Azure Monitor.

| Atributo  | Métricas | Registros |
|:---|:---|:---|
| Ventajas | Ligero y capaz de escenarios casi en tiempo real, como la alerta. Ideal para la detección rápida de problemas. | Analizado con lenguaje de consulta enriquecido. Ideal para análisis profundo e identificación de la causa principal. |
| data | Solo valores numéricos | Datos numéricos o texto |
| Estructura | Conjunto estándar de propiedades, como el tiempo de muestra, el recurso supervisado, un valor numérico. Algunas métricas incluyen varias dimensiones para una mejor definición. | Conjunto único de propiedades que dependen del tipo de registro. |
| Colección | Se recopila a intervalos regulares. | Se pueden recopilar esporádicamente cuando los eventos desencadenan un registro que se va a crear. |
| Ver en Azure Portal | Explorador de métricas | Log Analytics |
| Inclusión de orígenes de datos | Métricas de la plataforma recopiladas de recursos de Azure.<br>Aplicaciones supervisadas por Application Insights.<br>Personalización definida por la aplicación o API. | Registros de aplicaciones y recursos.<br>Soluciones de supervisión.<br>Agentes y extensiones de máquina virtual.<br>Solicitudes y excepciones de aplicación.<br>Azure Security Center.<br>API de recopilador de datos. |

## <a name="collect-monitoring-data"></a>Recopilación de datos de supervisión
Diferentes [orígenes de datos de Azure Monitor](data-sources.md) escribirán en un área de trabajo de Log Analytics (registros), en la base de datos de métricas de Azure Monitor (métricas), o en ambos. Algunos orígenes escribirán directamente en estos almacenes de datos, mientras que otros pueden escribir en otra ubicación, como almacenamiento de Azure, y requieren algo de configuración para rellenar los registros o las métricas. 

Consulte [Métricas en Azure Monitor](data-platform-metrics.md) y [Registros en Azure Monitor](data-platform-logs.md) para obtener una lista de diferentes orígenes de datos que rellenan cada tipo.


## <a name="stream-data-to-external-systems"></a>Hacer streaming de datos a sistemas externos
Además de usar las herramientas de Azure para analizar datos de supervisión, es posible que necesite reenviarlos a una herramienta externa, como un producto de administración de eventos e información de seguridad (SIEM). Este reenvío normalmente se realiza directamente desde los recursos supervisados a través de [Azure Event Hubs](../../event-hubs/index.yml). Algunos orígenes pueden configurarse para enviar datos directamente a un centro de eventos, aunque se puede usar otro proceso, como una aplicación lógica, para recuperar los datos necesarios. Consulte [Flujo de datos de supervisión de Azure a un centro de eventos para que lo consuma una herramienta externa](stream-monitoring-data-event-hubs.md) para más información.



## <a name="next-steps"></a>Pasos siguientes

- Para más información, consulte [Métricas en Azure Monitor](data-platform-metrics.md).
- Para más información, consulte [Registros en Azure Monitor](data-platform-logs.md).
- Obtenga información sobre la [supervisión de datos disponible](data-sources.md) para diferentes recursos en Azure.

