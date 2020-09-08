---
title: Introducción al Explorador de métricas de Azure
description: Aprenda a crear su primer gráfico de métricas con el Explorador de métricas de Azure.
author: vgorbenko
services: azure-monitor
ms.topic: conceptual
ms.date: 02/25/2019
ms.author: vitalyg
ms.subservice: metrics
ms.openlocfilehash: 3e5514c4a54083bcc6ef8187f2cd97515d2dd680
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88590190"
---
# <a name="getting-started-with-azure-metrics-explorer"></a>Introducción al Explorador de métricas de Azure

## <a name="where-do-i-start"></a>¿Por dónde empiezo?
El explorador de métricas de Azure Monitor es un componente de Microsoft Azure Portal que permite trazar los gráficos, correlacionar visualmente las tendencias e investigar crestas y valles en valores de las métricas. Utilice el Explorador de métricas para investigar el estado y la utilización de recursos. Comience en el siguiente orden:

1. [Seleccione un recurso y una métrica](#create-your-first-metric-chart) y verá un gráfico básico. Después, [seleccione un intervalo de tiempo](#select-a-time-range) que sea pertinente en su investigación.

1. Pruebe a [aplicar filtros de dimensión y divisiones](#apply-dimension-filters-and-splitting). Los filtros y las divisiones permiten analizar qué segmentos de la métrica contribuyen al valor general de la métrica, así como a identificar posibles valores atípicos.

1. Use la [configuración avanzada](#advanced-chart-settings) para personalizar el gráfico antes de anclarlo a algún panel. [Configure alertas](alerts-metric-overview.md) para recibir notificaciones cuando el valor de la métrica esté por encima o por debajo del umbral establecido.

## <a name="create-your-first-metric-chart"></a>Crear el primer gráfico de métricas

Para crear un gráfico de métricas, abra la pestaña **Métricas** desde el recurso, el grupo de recursos, la suscripción o la vista de Azure Monitor y haga lo siguiente:

1. Con el selector de recursos, seleccione el recurso del que quiera ver las métricas. El recurso aparecerá ya seleccionado si abre **Métricas** en el contexto de un recurso concreto.

    > ![Selección de un recurso](./media/metrics-getting-started/resource-picker.png)

2. En algunos recursos, debe elegir un espacio de nombres. El espacio de nombres es una forma de organizar las métricas para que se puedan encontrar fácilmente. Por ejemplo, las cuentas de almacenamiento tienen espacios de nombres aparte para almacenar las métricas de File service, Table service, Blob service y Queue service. Muchos tipos de recursos solo tienen un espacio de nombres.

3. Seleccione una métrica de una lista de métricas que haya disponible.

    > ![Selección de una métrica](./media/metrics-getting-started/metric-picker.png)

4. Si lo desea, puede [cambiar la agregación de la métrica](metrics-charts.md#changing-aggregation). Por ejemplo, puede que quiera que el gráfico muestre los valores mínimos, máximos o promedio de la métrica.

> [!TIP]
> Use el botón **Agregar métrica** y repita estos pasos si quiere ver varias métricas trazadas en el mismo gráfico. Para ver varios gráficos en una vista, seleccione el botón **Agregar gráfico** en la parte superior.

## <a name="select-a-time-range"></a>Selección de un intervalo de tiempo

> [!WARNING]
> [La mayoría de las métricas en Azure se almacenan durante 93 días](data-platform-metrics.md#retention-of-metrics). Sin embargo, no puede consultar datos de más de 30 días en un gráfico individual. Esta limitación no se aplica a las [métricas basadas en registros](../app/pre-aggregated-metrics-log-metrics.md#log-based-metrics).

El gráfico muestra las últimas 24 horas de los datos de métricas de forma predeterminada. Use el panel del **selector de hora** para cambiar el intervalo de tiempo o acercar o alejar el gráfico. 

![Panel para cambiar el intervalo de tiempo](./media/metrics-getting-started/time-picker.png)

> [!TIP]
> Use la herramienta del **pincel de tiempo** para investigar un área interesante del gráfico (un repunte o una caída). Coloque el puntero al principio del área, haga clic y mantenga presionado el botón principal, arrastre al otro lado del área y entonces suelte el botón. El gráfico acercará el intervalo de tiempo. 

## <a name="apply-dimension-filters-and-splitting"></a>Aplicar filtros de dimensión y divisiones

El [filtrado](metrics-charts.md#apply-filters-to-charts) y las [divisiones](metrics-charts.md#apply-splitting-to-a-chart) son herramientas de diagnóstico tremendamente eficaces con las métricas que tienen dimensiones. Estas características ponen de manifiesto el modo en que varios segmentos de la métrica ("valores de dimensión") influyen en el valor total de la métrica, y le permiten identificar posibles valores atípicos.

- El **filtrado** permite elegir qué valores de dimensión se van a incluir en el gráfico. Por ejemplo, puede que le interese mostrar las solicitudes correctas al representar gráficamente la métrica de *Tiempo de respuesta del servidor*. En tal caso, habría que aplicar el filtro en la dimensión de *solicitud correcta*. 

- Las **divisiones** controlan si el gráfico va a mostrar líneas independientes por cada valor de una dimensión, o si por el contrario va a agregar los valores en una sola línea. Por ejemplo, puede ver una línea correspondiente a un tiempo de respuesta promedio de todas las instancias de servidor, o ver líneas independientes de cada servidor. Para ver líneas independientes, habría que aplicar la división en la dimensión de *instancia del servidor*.

Vea [ejemplos de gráficos](metric-chart-samples.md) que tienen filtrado y divisiones aplicados. En el artículo se explican los pasos que se realizaron para configurar los gráficos.

## <a name="advanced-chart-settings"></a>Configuración gráfico avanzada

Se puede personalizar el estilo y el título de un gráfico, así como modificar la configuración de gráfico avanzada. Cuando haya terminado de personalizarlo, puede anclarlo a un panel para guardar el trabajo. También se pueden configurar alertas de métricas. Consulte la [documentación del producto](metrics-charts.md) para obtener información sobre estas y otras características avanzadas del Explorador de métricas de Azure Monitor.

## <a name="next-steps"></a>Pasos siguientes

* [Obtenga información sobre las características avanzadas del Explorador de métricas](metrics-charts.md)
* [Solución de problemas del Explorador de métricas](metrics-troubleshoot.md)
* [Vea una lista de métricas disponibles para servicios de Azure](metrics-supported.md)
* [Vea ejemplos de gráficos configurados](metric-chart-samples.md)
