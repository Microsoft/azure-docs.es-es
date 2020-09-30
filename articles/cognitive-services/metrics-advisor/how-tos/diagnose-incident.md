---
title: Diagnóstico de incidentes mediante Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo diagnosticar incidentes mediante Metrics Advisor y obtener vistas detalladas de las anomalías en los datos.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 08/19/2020
ms.author: aahi
ms.openlocfilehash: 7acd895832307d68c259139704565962fe534d22
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932078"
---
# <a name="how-to-diagnose-an-incident-using-metrics-advisor"></a>Procedimiento: Diagnóstico de un incidente mediante Metrics Advisor

Metrics Advisor proporciona varias características de diagnóstico, una vista detallada de los incidentes detectados y análisis de la causa principal. Cuando se detecta un grupo de anomalías en una métrica, Metrics Advisor agrupará las anomalías en una jerarquía y las analizará.

> [!NOTE]
> Actualmente, Metrics Advisor admite el diagnóstico de incidentes en métricas con al menos una dimensión y una medida con el tipo *numeric*. La métrica debe tener un valor de dimensión agregado como SUM para cada dimensión, que se usa para crear la jerarquía de diagnósticos. Metrics Advisor ofrece la [**configuración de acumulación automática**](onboard-your-data.md#automatic-roll-up-settings) para ayudar a generar valores agregados. 

Haga clic en **Incident hub** (Centro de incidentes) en la ventana de navegación de la izquierda para ver todos los incidentes de una métrica determinada. En la parte superior de la página, puede seleccionar distintas métricas para ver las configuraciones de detección y sus resultados, así como cambiar el intervalo de tiempo.

> [!TIP]
> También puede acceder a **Incident hub** (Centro de incidentes) de las siguientes formas.
> * Haciendo clic en un punto de datos de la visualización de la métrica y usando los vínculos de la parte inferior de la ventana **Add feedback** (Agregar comentarios) que aparece.
> * Haciendo clic en una de las anomalías de la pestaña **Incidentes** (Incidentes) de la métrica. 

La sección **Overview** (Información general) contiene resultados de la detección, incluidos los recuentos de las anomalías y las alertas del intervalo de tiempo seleccionado.

:::image type="content" source="../media/diagnostics/incident-hub-overview.png" alt-text="Centro de incidentes" lightbox="../media/diagnostics/incident-hub-overview.png":::

Los incidentes detectados en la métrica y el intervalo de tiempo seleccionados se muestran en **Incident list** (Lista de incidentes). Existen opciones para filtrar y ordenar los incidentes. Por ejemplo, por gravedad. Haga clic en uno de los incidentes para ir a la página **Incident** (Incidente) y realizar más diagnósticos.

:::image type="content" source="../media/diagnostics/incident-list.png" alt-text="Lista de incidentes" lightbox="../media/diagnostics/incident-list.png":::

La sección **Diagnostic** (Diagnóstico) permite realizar un análisis exhaustivo de un incidente e incluye herramientas para identificar las causas principales.

:::image type="content" source="../media/diagnostics/diagnose-incident.png" alt-text="Diagnóstico de incidentes" lightbox="../media/diagnostics/diagnose-incident.png" :::

## <a name="root-cause-advice"></a>Aviso de causa principal

Cuando se detecta un grupo de anomalías en una métrica y se produce un incidente, Metrics Advisor intentará analizar la causa principal del incidente. **Root cause advice** (Aviso de causa principal) proporciona sugerencias automáticas para las causas probables de un incidente. Esta característica solo está disponible si hay un valor agregado dentro de la dimensión. Si la métrica no tiene ninguna dimensión, la causa principal será la misma. Las causas principales se muestran en el panel lateral derecho y puede haber varias razones. Si no hay ningún dato en la tabla, significa que la dimensión no cumple los requisitos para realizar el análisis.

:::image type="content" source="../media/diagnostics/root-cause-advice.png" lightbox="../media/diagnostics/root-cause-advice.png" alt-text="Aviso de causa principal":::


Cuando se proporciona la métrica de causa principal con dimensiones específicas, puede hacer clic en **go to metric** (ir a métrica) para ver más detalles de la métrica.

## <a name="incident-tree"></a>Árbol de incidentes

Junto con el análisis automatizado de posibles causas raíz, Metrics Advisor permite analizar la causa principal manual mediante **Incident Tree** (Árbol de incidentes). Hay dos tipos de árbol de incidentes en la página de incidentes: el **árbol de diagnóstico rápido** y el **árbol interactivo**.

El árbol de diagnóstico rápido sirve para diagnosticar un incidente actual, y el nodo raíz se limita al nodo raíz del incidente actual. Para expandir y contraer los nodos de árbol, haga clic en el árbol; su serie se mostrará junto con la serie de incidentes actual en el gráfico situado encima del árbol.

El árbol interactivo permite diagnosticar incidentes actuales, así como incidentes más antiguos y otros relacionados. Al usar el árbol interactivo, haga clic con el botón derecho en un nodo para abrir un menú de acciones, donde puede elegir una dimensión para explorar en profundidad los nodos raíz y otra para hacer lo mismo con cada nodo. Al hacer clic en el botón cancel (cancelar) de la lista de dimensiones en la parte superior, puede explorar en profundidad esta dimensión o ver sus datos resumidos. Haga clic en un nodo para seleccionarlo y mostrar sus series junto con la serie de incidentes actuales en el gráfico.

:::image type="content" source="../media/diagnostics/incident-tree.png" alt-text="Árbol de incidentes" lightbox="../media/diagnostics/incident-tree.png" :::

## <a name="anomaly-drill-down"></a>Exploración en profundidad de anomalías

Al ver la información de los incidentes, puede que necesite obtener información más detallada; por ejemplo, para las distintas dimensiones y marcas de tiempo. Si los datos tienen una o varias dimensiones, puede usar la función de exploración en profundidad para obtener una vista más detallada. 

Para usar la función de exploración en profundidad, haga clic en la pestaña **Metric drilling** (Exploración de métricas) de **Incident hub** (Centro de incidentes). 

:::image type="content" source="../media/diagnostics/metric-drilling.png" lightbox="../media/diagnostics/metric-drilling.png" alt-text="Exploración de métricas ":::

La configuración **Dimensions** (Dimensiones) es una lista de dimensiones para un incidente; puede seleccionar otros valores de dimensión disponibles para cada uno. Después, los valores de la dimensión se cambian. La configuración **Timestamp** (Marca de tiempo) permite ver el incidente actual en momentos diferentes.

### <a name="select-drilling-options-and-choose-a-dimension"></a>Selección de opciones de exploración en profundidad y elección de una dimensión

Hay dos tipos de opciones de exploración en profundidad. **Drill down** (Explorar en profundidad) and **Horizontal comparison** (Comparación horizontal).

> [!Note]
> 1. Para la primera, puede explorar los datos de distintos valores de dimensión, excepto las dimensiones seleccionadas actualmente. 
> 2. Para la comparación horizontal, puede explorar los datos de distintos valores de dimensión, excepto las dimensiones generales.

:::image type="content" source="../media/diagnostics/drill-down-dimension.png"  lightbox="../media/diagnostics/drill-down-dimension.png" alt-text="Exploración en profundidad":::

### <a name="value-comparison-for-different-dimension-values"></a>Comparación de valores para diferentes valores de dimensión

La segunda sección de la pestaña Drill down (Exploración en profundidad) es una tabla con comparaciones de distintos valores de dimensión. Incluye el valor, el valor de línea de base, el valor de diferencia, el valor delta y si se trata de una anomalía.
 
:::image type="content" source="../media/diagnostics/drill-down-comparison.png" alt-text="Comparación horizontal" lightbox="../media/diagnostics/drill-down-comparison.png":::


### <a name="value-and-expected-value-comparisons-for-different-dimension-value"></a>Comparaciones de valores y valores esperados para diferentes valores de dimensión

La tercera sección de la pestaña Drill down (Exploración en profundidad) es un histograma con los valores y los valores esperados para diferentes valores de dimensión. El histograma se ordena por la diferencia entre el valor y el valor esperado. Puede encontrar fácilmente el valor inesperado con el mayor impacto. Por ejemplo, en la imagen anterior, podemos descubrir que, salvo el valor all-up (general), **US7** es el valor que más contribuye en la anomalía.

:::image type="content" source="../media/diagnostics/drill-down-table.png" alt-text="Tabla de exploración en profundidad" lightbox="../media/diagnostics/drill-down-table.png":::

### <a name="raw-value-visualization"></a>Visualización de valores sin procesar
La última parte de la pestaña Drill down (Exploración en profundidad) es un gráfico de líneas de los valores sin procesar. Con este gráfico proporcionado, no es necesario ir a la página de la métrica para ver los detalles.

:::image type="content" source="../media/diagnostics/drill-down-line-chart.png" alt-text="Gráfico de líneas de exploración en profundidad" lightbox="../media/diagnostics/drill-down-line-chart.png":::

## <a name="view-similar-anomalies-using-time-series-clustering"></a>Visualización de anomalías similares mediante la agrupación en clústeres de series temporales

Al ver un incidente, puede usar la pestaña **Similar time-series-clustering** (Agrupación en clústeres de series temporales) para ver las distintas series asociadas a él. Las series de un grupo se resumen juntas. En la imagen anterior, podemos saber que hay al menos dos grupos de series. Esta característica solo está disponible si se cumplen los siguientes requisitos.

1. Las métricas deben tener una o varias dimensiones o valores de dimensión.
2. Las series dentro de una métrica deben tener una tendencia similar.

Las dimensiones disponibles se muestran en la parte superior de la pestaña, y puede hacer una selección para especificar las series.

:::image type="content" source="../media/diagnostics/series-group.png" lightbox="../media/diagnostics/series-group.png"alt-text="Grupo de series":::

## <a name="compare-time-series"></a>Comparación de series temporales

A veces, cuando se detecta una anomalía en una serie temporal específica, resulta útil compararla con varias series en una sola visualización. Haga clic en la pestaña **Compare tools** (Comparar herramientas) y, a continuación, haga clic en el botón azul **+ Add** (+ Agregar). 

:::image type="content" source="../media/diagnostics/add-series.png" alt-text="Adición de series para comparar" lightbox="../media/diagnostics/add-series.png":::

Seleccione una serie de la fuente de distribución de datos. Puede elegir la misma granularidad u otra. Seleccione las dimensiones de destino y cargue la tendencia de la serie y, a continuación, haga clic en **Ok** (Aceptar) para compararla con una serie anterior. La serie se colocará en una visualización. Puede seguir agregando más series para la comparación y obtener así más información. Haga clic en el menú desplegable de la parte superior de la pestaña **Compare tools** (Comparar herramientas) para hacer una comparación desplazada de los datos de series temporales durante un período.  

> [!Warning]
> Para habilitar una comparación desplazada, la granularidad de los datos debe admitirla. Por ejemplo, si los datos son semanales y usa la comparación **Day over day** (Día a día), no obtendrá ningún resultado. En este ejemplo, se usaría la comparación **Month over month** (Mes a mes).

Después de seleccionar una comparación desplazada, puede elegir si desea comparar los valores de los datos, los valores delta o la diferencia porcentual.

> [!Note]
> * El **valor de los datos** es el valor de los datos sin procesar.
> * El **valor delta** es la diferencia entre el valor sin procesar y el valor comparado.
> * El **valor diferencial porcentual** es la diferencia entre el valor sin procesar y el valor comparado dividido por el valor comparado.

## <a name="related-incidents-across-metrics"></a>Incidentes relacionados entre métricas

En ocasiones, es posible que tenga que comprobar los incidentes de distintas métricas al mismo tiempo o incidentes relacionados en otras métricas. Puede encontrar una lista de incidentes relacionados en la sección **Cross Metrics Analysis** (Análisis entre métricas). 

:::image type="content" source="../media/graph/metrics-graph-cross-metrics-analysis.png" alt-text="Incidentes relacionados entre métricas":::

Antes de que pueda ver incidentes relacionados con la métrica actual, debe agregar una relación entre las métricas. Haga clic en **Metrics Graph Settings** (Configuración de gráfico de métricas) para agregar una relación. Solo se pueden relacionar las métricas con los mismos nombres de dimensión. Utilice los siguientes parámetros.

- Current Data feed & Metric (Métrica y fuente de distribución de datos actuales): la fuente de distribución de datos y la métrica del incidente actual.
- Direction (Dirección): la dirección de la relación entre dos métricas. En este momento, no afecta a la lista de incidentes relacionados.
- Another Data feed & Metric (Otra métrica y fuente de distribución de datos): la fuente de distribución de datos y la métrica para conectarse con la métrica actual.


## <a name="next-steps"></a>Pasos siguientes 

- [Ajuste de la detección de anomalías mediante comentarios](anomaly-feedback.md)
- [Configuración de métricas y ajuste de la configuración de la detección](configure-metrics.md)