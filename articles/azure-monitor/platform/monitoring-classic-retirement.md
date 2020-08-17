---
title: Actualización de alertas y supervisión clásicas en Azure Monitor
description: Descripción de la retirada de la funcionalidad y los servicios de supervisión clásicos, que antes se mostraban en Azure Portal en Alertas (clásico).
author: yanivlavi
services: azure-monitor
ms.topic: conceptual
ms.date: 2/7/2019
ms.author: yalavi
ms.subservice: alerts
ms.openlocfilehash: 2f8f1cceb5cfee4bf910674ecd50c7fba1d5c021
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318375"
---
# <a name="unified-alerting--monitoring-in-azure-monitor-replaces-classic-alerting--monitoring"></a>Alertas y supervisión unificadas de Azure Monitor reemplaza a alertas y supervisión clásicas

Azure Monitor se ha convertido en un servicio de supervisión completo y unificado que ahora admite "One Metrics" (métricas unificadas) y "One Alerts" (alertas unificadas) en los recursos; para más información, consulte nuestra [entrada de blog acerca del nuevo servicio Azure Monitor](https://azure.microsoft.com/blog/new-full-stack-monitoring-capabilities-in-azure-monitor/). Las nuevas plataformas de supervisión y generación de alertas de Azure se han creado para que sean más rápidas, inteligentes y extensibles (con lo que se sigue el ritmo de la creciente expansión de la informática en la nube y está en línea con la filosofía de Microsoft Intelligent Cloud). 

Con la implantación de la nueva plataforma de supervisión y generación de alertas de Azure, se retirará la plataforma "clásica" de supervisión y alertas (hospedada en la sección *Ver alertas clásicas* de las alertas de Azure y **dejará de usarse en agosto de 2019 en las nubes públicas de Azure**). La [nube de Azure Government](../../azure-government/documentation-government-welcome.md) y [Azure China 21Vianet](https://docs.azure.cn/) no se verán afectadas.

> [!NOTE]
> Debido a un retraso en la implementación de la herramienta de migración, la fecha de retirada para la migración de alertas clásicas se [extendió hasta el 31 de agosto de 2019](https://azure.microsoft.com/updates/azure-monitor-classic-alerts-retirement-date-extended-to-august-31st-2019/), desde la fecha anunciada originalmente del 30 de junio de 2019.

 ![Alerta clásica en Azure Portal](media/monitoring-classic-retirement/monitor-alert-screen2.png) 

Le animamos a volver a crear las alertas en la nueva plataforma. Los clientes que tienen un gran número de alertas, estamos [implementando en fases](alerts-understand-migration.md#rollout-phases) una [herramienta de migración voluntaria](alerts-using-migration-tool.md) para mover las alertas clásicas existentes al nuevo sistema de alertas sin interrupciones ni costes adicionales.

> [!IMPORTANT]
> Las reglas de alerta clásicas creadas en el registro de actividad no caerán en desuso ni se migrarán. Puede acceder a todas las reglas de alerta clásicas creadas en el registro de actividad y usarlas tal cual, desde la nueva Azure Monitor: Alertas. Para más información, vea [Crear, ver y administrar las alertas del registro de actividad mediante Azure Monitor](./alerts-activity-log.md). De forma similar, puede acceder a alertas en Service Health y usarlas tal cual desde la nueva sección Service Health. Para más información, consulte las [alertas en las notificaciones de Service Health](../../service-health/alerts-activity-log-service-notifications-portal.md).

## <a name="unified-metrics-and-alerts-in-application-insights"></a>Métricas y alertas unificadas en Application Insights

La nueva plataforma de métricas de Azure Monitor alimentará la supervisión procedente de Application Insights. Este cambio significa que Application Insights se enlazará a Grupos de acciones, lo que aporta muchas más opciones que anteriores las llamadas de webhook y el correo electrónico. Ahora, las alertas pueden desencadenar llamadas de voz, Azure Functions, Logic Apps, SMS y las herramientas de ITSM, como ServiceNow y runbooks de Automation. Con la generación de alertas y la supervisión casi en tiempo real, la nueva plataforma permite a los usuarios de Application Insights sacar provecho de la misma tecnología que alimenta la supervisión en otros recursos de Azure y sustenta la supervisión de los productos de Microsoft.

Comprenden la nueva supervisión y generación de alertas unificadas de Application Insights englobará:

- **Métricas de la plataforma Application Insights**: que proporciona métricas populares creadas previamente del producto Application Insights. Para más información, consulte este artículo acerca del uso de [métricas de plataforma para Application Insights en el nuevo Azure Monitor](../app/pre-aggregated-metrics-log-metrics.md#pre-aggregated-metrics).
- **Disponibilidad de Application Insights y prueba web**: que proporciona la capacidad de evaluar la capacidad de respuesta y disponibilidad de una aplicación web o servidor. Para más información, consulte este artículo acerca del uso de [prueba de disponibilidad y alertas para Application Insights en el nuevo Azure Monitor](../app/monitor-web-app-availability.md).
- **Métricas personalizadas de Application Insights**: que permiten definir y emitir sus propias métricas de supervisión y alertas. Para más información, consulte este artículo acerca del uso de [métricas personalizadas para Application Insights en el nuevo Azure Monitor](../app/pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation).
- **Anomalías en los errores de Application Insights (parte de Detección inteligente)** : que le notifica automáticamente casi en tiempo real si una aplicación web sufre un aumento anómalo en la frecuencia de solicitudes HTTP o llamadas de dependencia erróneas. Para más información, consulte este artículo [Detección inteligente: anomalías de error](../app/proactive-failure-diagnostics.md).

## <a name="unified-metrics-and-alerts-for-other-azure-resources"></a>Métricas y alertas unificadas para otros recursos de Azure

Desde marzo de 2018, la próxima generación de alertas y supervisión multidimensional para recursos de Azure han estado en disponibilidad. Ahora la plataforma de métricas más reciente y la generación de alertas es más rápida con funcionalidades casi en tiempo real. Y lo que es más importante, las nuevas alertas de la plataforma de métricas proporcionan mayor granularidad, ya que la nueva plataforma incluye la opción de dimensiones, lo que le permiten segmentar y filtrar en una combinación de valores, condición u operación específicas. Al igual que todas las alertas del nuevo Azure Monitor, las nuevas alertas de métricas son más extensibles con el uso de ActionGroups (lo que permite que las notificaciones vayan más allá del correo electrónico o webhook a SMS, voz, Azure Functions, runbook de Automation, etc. Para más información, vea [Creación, visualización y administración de alertas de métricas mediante Azure Monitor](./alerts-metric.md).
Las métricas más recientes de los recursos de Azure están disponibles como:

- **Métricas de la plataforma estándar de Azure Monitor**: que proporciona métricas populares rellenadas previamente de varios productos y servicios de Azure. Para más información, consulte este artículo acerca de las [métricas compatibles con Azure Monitor](./alerts-metric-near-real-time.md#metrics-and-dimensions-supported) y [Compatibilidad con alertas de métricas en Azure Monitor](./alerts-metric-overview.md#supported-resource-types-for-metric-alerts).
- **Métricas personalizadas de Azure Monitor**: que proporciona métricas de orígenes controlados por el usuario, lo que incluye el agente de Azure Diagnostics. Para más información, consulte este artículo acerca de las [métricas personalizadas en Azure Monitor](./metrics-custom-overview.md). Con las métricas personalizadas también puede publicar las métricas recopiladas por el [agente de Windows Azure Diagnostics](./collect-custom-metrics-guestos-resource-manager-vm.md) y el [agente de InfluxData Telegraf](./collect-custom-metrics-linux-telegraf.md).

## <a name="retirement-of-classic-monitoring-and-alerting-platform"></a>Retirada de la plataforma de alertas y supervisión clásica

Como se indicó anteriormente, la plataforma de alertas y supervisión clásica que se puede usar actualmente desde la [sección Alertas (clásico)](./alerts-classic.overview.md) de Azure Portal se retirará el portal en los próximos meses, ya que se ha reemplazado por el nuevo sistema.
El 31 de agosto de 2019 se retirará la funcionalidad clásica de supervisión y alertas anterior, lo que incluye el cierre de las API relacionadas, la interfaz de Azure Portal y los servicios que contiene. En concreto, estas características quedarán en desuso:

- Métricas y alertas anteriores (clásicas) para recursos de Azure están actualmente disponibles en la [sección Alertas (clásico)](./alerts-classic.overview.md) de Azure Portal y se puede acceder a ellas como el recurso [microsoft.insights/alertrules](/rest/api/monitor/alertrules).
- Métricas personalizadas y de la plataforma anteriores (clásicas) para Application Insights, así como la generación de alertas sobre ellos, que actualmente está disponible en la [sección Alertas (clásico)](./alerts-classic.overview.md) de Azure Portal y se puede acceder a ellas como el recurso [microsoft.insights/alertrules](/rest/api/monitor/alertrules).
- Alerta de anomalías en los errores anteriores (clásicas), que actualmente están disponibles como [Detección inteligente en Application Insights](../app/proactive-diagnostics.md) en Azure Portal, con las alertas configuradas que se muestran en la sección [Alertas (clásico)](./alerts-classic.overview.md) de Azure Portal.

Todos los sistemas clásicos de supervisión y generación de alertas, incluidos la [API](/rest/api/monitor/alertrules) correspondiente, [PowerShell](./alerts-classic-portal.md), la [CLI](./alerts-classic-portal.md), la [página de Azure Portal](./alerts-classic-portal.md) y la [plantilla de recursos](./alerts-enable-template.md), se podrán usar hasta finales de agosto de 2019. 

Al finales de agosto de 2019, en Azure Monitor:

- El servicio de alertas y supervisión clásico se retirará y dejará de estar disponible para la creación de nuevas reglas de alerta.
- Todas las reglas de alertas que sigan en Alertas (clásico) después de agosto de 2019 seguirán ejecutándose y enviando notificaciones, pero no estarán disponibles para su modificación.
- A partir de septiembre de 2019, Microsoft trasladará las reglas de alertas en la supervisión y generación de alertas clásicas que se pueden migrar a su equivalente en la nueva plataforma de Azure Monitor, en las fases que abarcarán varias semanas. El proceso será ininterrumpido, no habrá tiempo de inactividad y garantizará que los clientes no pierden cobertura de supervisión.
- Las reglas de alertas migradas a la nueva plataforma de alertas proporcionarán cobertura de supervisión como antes, pero activarán notificaciones con las nuevas cargas útiles. Todas las direcciones de correo electrónico, puntos de conexión de webhook o vínculos a aplicaciones lógicas asociados con la regla de alertas clásica se traspasarán al realizar la migración, pero puede que su comportamiento no sea correcto, ya que la carga útil de la alerta será distinta en la nueva plataforma.
- Algunas [reglas de alertas clásicas que no se pueden migrar automáticamente](alerts-understand-migration.md#classic-alert-rules-that-will-not-be-migrated) y requieren una acción manual de los usuarios seguirán ejecutándose hasta junio de 2020.

> [!IMPORTANT]
> Microsoft Azure Monitor ha implantado en fases una [herramienta para migrar voluntariamente](alerts-using-migration-tool.md) sus reglas de alertas clásicas a la nueva plataforma. La ha ejecutado a la fuerza para todas las reglas de alertas clásicas que aún existen y se pueden migrar, a partir de septiembre de 2019. Los clientes deberán garantizar la adaptación de la automatización que consume la carga útil de reglas de alertas clásicas para poder controlar la nueva carga útil siguiendo los procedimientos descritos en [Métricas y alertas unificadas en Application Insights](#unified-metrics-and-alerts-in-application-insights) o [Métricas y alertas unificadas para otros recursos de Azure](#unified-metrics-and-alerts-for-other-azure-resources) después de la migración de las reglas de alertas clásicas. Para más información, consulte [Preparar sus aplicaciones lógicas y los runbooks para la migración de las reglas de alertas clásicas](alerts-prepare-migration.md).

Este artículo se actualizará constantemente con vínculos y detalles relativos a la nueva funcionalidad de supervisión y generación de alertas de Azure, así como con la disponibilidad de las herramientas que ayudan a los usuarios a adoptar la nueva plataforma de Azure Monitor.

## <a name="pricing-for-migrated-alert-rules"></a>Precios de las reglas de alerta migradas

Vamos a implementar una herramienta de migración para ayudarle a migrar las [alertas clásicas](./alerts-classic.overview.md) de Azure Monitor a la nueva experiencia de alertas. Las reglas de alertas migradas y los grupos de acciones migrados correspondientes (correo electrónico, webhook o LogicApp) seguirán siendo gratuitos. La funcionalidad que tenía con las alertas clásicas, incluida la capacidad de editar el umbral, el tipo de agregación y la granularidad de agregaciones, seguirá estando disponible de forma gratuita con la regla de alertas migrada. Sin embargo, si edita la regla de alertas migrada para usar cualquiera de las nuevas características, notificaciones o tipos de acción de la plataforma de alertas, se aplicará el cargo correspondiente. Para obtener más información sobre los precios de las reglas de alertas y las notificaciones, consulte los [precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

A continuación se muestran ejemplos de casos en los que se incurrirá en un cargo por la regla de alertas:

- Todas las nuevas reglas de alertas (no migradas) creadas después de las unidades gratuitas en la nueva plataforma de Azure Monitor
- Todos los datos ingeridos y conservados en unidades que no sean las gratuitas incluidas por Azure Monitor
- Todas las pruebas web que ejecuta Application Insights
- Todas las métricas personalizadas almacenadas en unidades que no sean las gratuitas que se incluyen en Azure Monitor
- Todas las reglas de alertas migradas que se editen para usar las características de alertas de métricas más recientes, como la frecuencia, varios recursos o dimensiones, los [umbrales dinámicos](alerts-dynamic-thresholds.md), el cambio de recurso o señal, etc.
- Todos los grupos de acciones migrados que se editen para usar notificaciones más recientes, o tipos de acción como SMS, llamada de voz o integración de ITSM.

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca del [nuevo Azure Monitor unificado](../overview.md).
* Más información acerca de las nuevas [alertas de Azure](./alerts-overview.md).

