---
title: Gráfico de métricas de Metrics Advisor
titleSuffix: Azure Cognitive Services
description: En este artículo se explica cómo configurar Metrics Graph y visualizar las anomalías relacionadas en los datos.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: aahi
ms.openlocfilehash: 09ea16f07973757b169f21c7c3f909a24651daa4
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932557"
---
# <a name="how-to-build-a-metrics-graph-to-analyze-related-metrics"></a>Procedimiento: Creación de un gráfico de métricas para analizar las métricas relacionadas

Cada métrica de Metrics Advisor se supervisa por separado en función de un modelo que aprende de datos históricos para predecir tendencias futuras. Cada métrica tiene un modelo independiente que se le aplica. En algunos casos, sin embargo, varias métricas pueden estar relacionadas entre sí y las anomalías se deben analizar en varias métricas. **Metrics Graph** ayuda con esta tarea. 

Por ejemplo, si tiene diferentes flujos de telemetría en métricas independientes, Metrics Advisor los supervisará por separado. Si las anomalías de una métrica causan anomalías en otras, encontrar esas relaciones y la causa principal de los datos puede ser útil al abordar incidentes. Metrics Graph permite crear un gráfico de topología visual de anomalías encontradas. 

## <a name="select-a-metric-to-put-the-first-node-to-the-graph"></a>Selección de una métrica para colocar el primer nodo en el gráfico

Haga clic en la pestaña **Metrics Graph** de la barra de navegación. El primer paso de la creación de un gráfico de métricas consiste en colocar un nodo en el gráfico. Seleccione una fuente de distribución de datos y una métrica en la parte superior de la página. Aparecerá un mensaje en la barra inferior. 

:::image type="content" source="../media/graph/metrics-graph-select.png" alt-text="Selección de la métrica":::

## <a name="add-a-noderelation-on-existing-node"></a>Adición de un nodo o una relación a un nodo

A continuación, debe agregar otro nodo y especificar una relación con uno o varios nodos. Seleccione un nodo y haga clic con el botón derecho en él. Aparecerá un menú contextual con varias opciones. 

Haga clic en **Add relation** (Agregar relación) para elegir otra métrica y especifique el tipo de relación entre los dos nodos. También puede aplicar filtros de dimensión específicos. 

:::image type="content" source="../media/graph/metrics-graph-node-action.png" alt-text="Adición de un nodo y una relación":::

Después de repetir los pasos anteriores, tendrá un gráfico de métricas que describe las relaciones entre todas las métricas relacionadas.
**Sugerencia sobre los colores de los nodos**
> [!TIP]
> - Al seleccionar una métrica y un filtro de dimensión, todos los nodos que tengan la misma métrica y el mismo filtro de dimensiones en el gráfico estarán coloreados de **<font color=blue>azul</font>** .
> - Los nodos no seleccionados que representan una métrica en el gráfico estarán coloreados de **<font color=green>verde</font>** .
> - Si se observa una anomalía en la métrica actual, el nodo se coloreará de **<font color=red>rojo</font>** .

## <a name="view-related-metrics-anomaly-status-in-incident-hub"></a>Visualización del estado de anomalías de las métricas relacionadas en Incident hub (Centro de incidentes)

Cuando se crea el gráfico de métricas, siempre que se detecta una anomalía en las métricas del gráfico, podrá ver los estados de anomalías relacionados y ver un resumen del incidente. 

Haga clic en un incidente dentro del gráfico y desplácese hacia abajo hasta **cross metrics analysis** (análisis entre métricas), debajo de la información de diagnóstico.

:::image type="content" source="../media/graph/metrics-graph-cross-metrics-analysis.png" alt-text="Visualización de anomalías y métricas relacionadas":::

## <a name="next-steps"></a>Pasos siguientes

- [Ajuste de la detección de anomalías mediante comentarios](anomaly-feedback.md)
- [Diagnóstico de incidentes](diagnose-incident.md)
- [Configuración de métricas y ajuste de la configuración de la detección](configure-metrics.md)
