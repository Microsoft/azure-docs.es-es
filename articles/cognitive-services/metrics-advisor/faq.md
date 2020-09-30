---
title: Preguntas más frecuentes sobre Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Preguntas más frecuentes sobre el servicio Metrics Advisor.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/10/2020
ms.author: aahi
ms.openlocfilehash: 0fde9a0f46073a2f3a24962ea58431581455f474
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932214"
---
# <a name="metrics-advisor-frequently-asked-questions"></a>Preguntas más frecuentes sobre Metrics Advisor

### <a name="what-is-the-cost-of-my-instance"></a>¿Cuál es el costo de mi instancia?

Actualmente, no hay ningún costo por usar su instancia durante la versión preliminar.

### <a name="why-is-the-demo-website-readonly"></a>¿Por qué es de solo lectura el sitio web de demostración?

El [sitio web de demostración](https://anomaly-detector.azurewebsites.net/) está disponible públicamente. Esta instancia se convierte en solo lectura para evitar la carga accidental de cualquier dato.

### <a name="why-cant-i-create-the-resource-the-pricing-tier-is-unavailable-and-it-says-you-have-already-created-1-s0-for-this-subscription"></a>¿Por qué no puedo crear el recurso? El "plan de tarifa" no está disponible y dice "Ya ha creado 1 S0 para esta suscripción".

:::image type="content" source="media/pricing.png" alt-text="Mensaje cuando ya existe un recurso F0":::

Durante la versión preliminar pública, solo se permite crear una instancia de Metrics Advisor en una suscripción, en una región.

Si ya tiene una instancia creada en la misma región con la misma suscripción, puede probar una región diferente o una suscripción diferente para crear una nueva instancia. También puede eliminar una instancia existente para crear una nueva.

Si ya ha eliminado la instancia existente pero sigue apareciendo el error, espere unos veinte minutos después de eliminar el recurso antes de crear una nueva instancia.

## <a name="basic-concepts"></a>Conceptos básicos

### <a name="what-is-multi-dimensional-time-series-data"></a>¿Qué son los datos de serie temporal multidimensional?

Consulte la definición de [métrica multidimensional](glossary.md#multi-dimensional-metric) en el glosario.

### <a name="how-much-data-is-needed-for-metrics-advisor-to-start-anomaly-detection"></a>¿Cuántos datos se necesitan para que Metrics Advisor inicie la detección de anomalías?

Como mínimo, un punto de datos puede desencadenar la detección de anomalías. Sin embargo, esto no aporta la mejor precisión. Durante la creación de la fuente de distribución de datos, el servicio asumirá una ventana de puntos de datos anteriores utilizando el valor especificado como regla de "espacio de relleno".

Se recomienda tener algunos datos antes de la marca de tiempo en la que desea realizar la detección.
Según la granularidad de los datos, la cantidad de datos recomendada varía como se indica a continuación.

| Granularidad | Importe de datos recomendado para la detección |
| ----------- | ------------------------------------- |
| Menos de 5 minutos | 4 días de datos |
| 5 minutos a 1 día | 28 días de datos |
| Más de 1 día, hasta 31 días | 4 años de datos |
| Superior a 31 días | 48 años de datos |

### <a name="why-metrics-advisor-doesnt-detect-anomalies-from-historical-data"></a>¿Por qué Metrics Advisor no detecta las anomalías de los datos históricos?

Metrics Advisor está diseñado para detectar datos de streaming en vivo. Existe una limitación de la duración máxima de los datos históricos que el servicio analizará y en la que ejecutará la detección de anomalías. Esto significa que solo los puntos de datos después de una marca de tiempo más temprana determinada tendrán resultados de detección de anomalías. La marca de tiempo más temprana depende de la granularidad de los datos.

En función de la granularidad de los datos, las duraciones de los datos históricos que tendrán resultados de detección de anomalías son las siguientes.

| Granularidad | Duración máxima de los datos históricos para la detección de anomalías |
| ----------- | ------------------------------------- |
| Menos de 5 minutos | Tiempo de incorporación: 13 horas |
| De 5 minutos a menos de 1 hora | Tiempo de incorporación: 4 días  |
| De 1 hora a menos de 1 día | Tiempo de incorporación: 14 días  |
| 1 día | Tiempo de incorporación: 28 días  |
| Mayor de 1 día, menos de 31 días | Tiempo de incorporación: 2 años  |
| Superior a 31 días | Tiempo de incorporación: 24 años   |

### <a name="more-concepts-and-technical-terms"></a>Más conceptos y términos técnicos

Vaya a [Glosario](glossary.md) para obtener más información.

## <a name="how-do-i-detect-such-kinds-of-anomalies"></a>¿Cómo detecto esos tipos de anomalías? 

### <a name="how-do-i-detect-spikes--dips-as-anomalies"></a>¿Cómo detecto picos y caídas como anomalías?

Si tiene umbrales estrictos predefinidos, realmente puede establecer de forma manual el "umbral estricto" en las [configuraciones de detección de anomalías](how-tos/configure-metrics.md#anomaly-detection-methods).
Si no hay umbrales, puede usar la "detección inteligente", que se basa en IA. Consulte la [optimización de la configuración de detección](how-tos/configure-metrics.md#tune-the-detecting-configuration) para obtener detalles.

### <a name="how-do-i-detect-inconformity-with-regular-seasonal-patterns-as-anomalies"></a>¿Cómo detecto el incumplimiento con patrones normales (estacionales) como anomalías?

La "detección inteligente" es capaz de aprender el patrón de los datos, incluidos los patrones estacionales. Después, detecta los puntos de datos que no se ajustan a los patrones regulares como anomalías. Consulte la [optimización de la configuración de detección](how-tos/configure-metrics.md#tune-the-detecting-configuration) para obtener detalles.

### <a name="how-do-i-detect-flat-lines-as-anomalies"></a>¿Cómo detecto líneas planas como anomalías?

Si los datos suelen ser bastante inestables y fluctúan mucho, y desea recibir una alerta cuando pasen a ser demasiado estables o incluso se conviertan en una línea plana, el "umbral de cambio" puede configurarse para detectar dichos puntos de datos cuando el cambio sea demasiado pequeño.
Consulte las [configuraciones de detección de anomalías](how-tos/configure-metrics.md#anomaly-detection-methods) para obtener detalles.

## <a name="next-steps"></a>Pasos a seguir
- [Introducción a Metrics Advisor](overview.md)
- [Prueba del sitio de demostración](quickstarts/explore-demo.md)
- [Uso del portal web](quickstarts/web-portal.md)