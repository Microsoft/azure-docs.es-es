---
title: Empiece a explorar la demo de Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Descubra la interfaz web de Metrics Advisor con la demo disponible.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: quickstart
ms.date: 09/14/2020
ms.author: aahi
ms.openlocfilehash: fbc73e20b8cc2baa1cc5c5a5b2f674fb1b2dde84
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945786"
---
# <a name="quickstart-explore-the-metrics-advisor-demo-with-example-data"></a>Inicio rápido: Exploración de la demo de Metrics Advisor con datos de ejemplo

> [!Note]
> La demo de Metrics Advisor es de solo lectura, con lo que no podrá incorporar sus propios datos. Para usar el servicio con sus datos, primero debe [crear un recurso de Metrics Advisor](web-portal.md).

Use este artículo para empezar a explorar rápidamente las características clave de Metrics Advisor. Proporcionamos un sitio de demostración con datos de ejemplo y configuraciones preestablecidas para familiarizarse con el portal web completo.

Haga clic en [este vínculo](https://aka.ms/MetricsAdvisor/Demo) para ir al sitio de demostración.

## <a name="view-the-available-sample-data"></a>Visualización de los datos de ejemplo disponibles

Después de iniciar sesión en el sitio de demostración, verá una página de **fuente de distribución de datos**. Una fuente de distribución de datos es un grupo lógico de datos de serie temporal, que se consulta desde el origen de datos. Para más información sobre los términos y conceptos que se usan en Metrics Advisor, consulte el [Glosario](../glossary.md). 

Se muestran varias fuentes de distribución de datos, que se ingieren desde distintos tipos de orígenes de datos, como Azure SQL Database o Azure Table. Cada una de ellas usa opciones de configuración ligeramente diferentes para conectarse a los almacenes de datos asociados.

:::image type="content" source="../media/sample-datafeeds.png" alt-text="Lista de datos de ejemplo" lightbox="../media/sample-datafeeds.png":::

## <a name="explore-the-data-feed-configurations"></a>Exploración de las configuraciones de fuentes de distribución de datos

Haga clic en la fuente de distribución de datos *Sample - Cost/Revenue - City/Category* (Ejemplo - Costo/Ingresos - Ciudad/Categoría). Verá varias secciones de detalles de la fuente:

* Nombre de la fuente de distribución de datos y estado de ingesta.
* Una lista de las métricas consultadas desde el origen de datos. Por ejemplo, *cost* (costo) y *revenue* (ingresos). 
* Historial de alertas cuando la fuente de distribución de datos deja de estar disponible. 
* Registros de cuándo se actualizó la fuente de distribución de datos.   
* Información y valores de la fuente de distribución de datos.

:::image type="content" source="../media/data-feed-view.png" alt-text="Vista de la fuente de distribución de datos" lightbox="../media/data-feed-view.png":::


## <a name="view-time-series-visualizations-and-configurations"></a>Visualización de visualizaciones y configuraciones de series temporales

Haga clic en la métrica *cost* (costo) de la fuente de distribución de datos *Sample - Cost/Revenue - City/Category* (Ejemplo - Costo/Ingresos - Ciudad/Categoría). Verá la serie temporal asociada segmentada por dimensiones, con visualizaciones en función de los datos de métrica históricos. La banda azul en torno a los datos de la métrica representa el intervalo de valores esperado de los modelos de Machine Learning de Metrics Advisor. Los puntos que se encuentran fuera de esta banda se marcarán en rojo, lo que indica que se trata de anomalías detectadas. 

:::image type="content" source="../media/series-visualization.png" alt-text="Visualización de la serie" lightbox="../media/series-visualization.png":::

La detección de anomalías se puede configurar ajustando las **configuraciones de detección** en el lado izquierdo de la página de detalles de la métrica. Hay varios métodos de detección de anomalías disponibles y pueden combinarse. Además, puede probar diferentes sensibilidades, direcciones de detección y otras configuraciones. El vínculo **Advanced configuration** (Configuración avanzada) en la parte inferior de las **configuraciones de detección** permite crear una configuración de detección más compleja y personalizada, que se puede usar en grupos o series individuales. 

También puede ajustar la detección de anomalías proporcionando información al algoritmo de detección. Haga clic en una anomalía y use el panel **Add feeback** (Agregar comentarios) para configurar el estado de la anomalía, la estacionalidad y el estado del punto de cambio. Estos comentarios se incorporarán en la detección de puntos futuros.  

En la parte inferior del panel **Add feeback** (Agregar comentarios), se encuentra el vínculo al **centro de incidentes**, que le dirigirá a la página de análisis del incidente y analizará la causa principal del incidente.  

:::image type="content" source="../media/incident-link.png" alt-text="Vínculo del incidente" lightbox="../media/incident-link.png":::

## <a name="explore-anomaly-detection-results-and-perform-root-cause-analysis"></a>Exploración de los resultados de la detección de anomalías y realización de análisis de la causa principal

Al hacer clic en el vínculo al **centro de incidentes** desde una anomalía, verá una página de análisis del incidente con información de diagnóstico sobre el incidente, como la gravedad, el número de anomalías implicadas y la hora de inicio y finalización. En la sección **Root cause** (Causa principal) se muestran consejos automatizados resultantes de analizar el árbol de incidentes, que tienen en cuenta la desviación, distribución y contribución a las anomalías principales que pueden ser la causa principal del incidente.

La sección **Diagnostics** (Diagnóstico) muestra un árbol del incidente, junto con varias pestañas para diagnosticar el incidente.

:::image type="content" source="../media/incident-diagnostic.png" alt-text="Diagnóstico de incidentes" lightbox="../media/incident-diagnostic.png":::

Al identificar la causa principal del incidente, puede tomar medidas y mitigar el problema antes de que la situación empeore. También hacer clic en las otras características de diagnóstico proporcionadas para obtener más información. 

## <a name="next-steps"></a>Pasos siguientes

- [Uso del portal web](web-portal.md)
- [Uso de la API de REST](rest-api.md)
- [Incorporación de las fuentes de distribución de datos](../how-tos/onboard-your-data.md)
    - [Administración de las fuentes de distribución de datos](../how-tos/manage-data-feeds.md)
    - [Configuraciones para distintos orígenes de datos](../data-feeds-from-different-sources.md)
