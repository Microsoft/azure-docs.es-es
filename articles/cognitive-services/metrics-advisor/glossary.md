---
title: Glosario de Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Ideas y conceptos clave para el servicio Metrics Advisor
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: aahi
ms.openlocfilehash: 25e5cb6e34624f3272e83e1e7c1cd847b865ae8e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932164"
---
# <a name="metrics-advisor-glossary-of-common-vocabulary-and-concepts"></a>Glosario de conceptos y vocabulario comunes de Metrics Advisor

En este documento se explican los términos técnicos que se usan en Metrics Advisor. Use este artículo para obtener información sobre los conceptos y objetos comunes que puede encontrar al usar el servicio.

## <a name="data-feed"></a>Fuente de distribución de datos

> [!NOTE]
> Varias métricas pueden compartir el mismo origen de datos, incluso la misma fuente de distribución de datos.

Una fuente de distribución de datos es lo que Metrics Advisor ingiere del origen de datos, como Cosmos DB o SQL Server. Una fuente de distribución de datos contiene filas de:
* marcas de tiempo
* ninguna o varias dimensiones
* una o varias medidas. 

## <a name="metric"></a>Métrica

Una métrica es una medida cuantificable que se usa para supervisar y evaluar el estado de un proceso empresarial específico. Puede ser una combinación de varios valores de serie temporal divididos en dimensiones. Por ejemplo, una métrica de *mantenimiento de la Web* podría contener dimensiones para *recuento de usuarios* y el *mercado es-es*.

## <a name="dimension"></a>Dimensión

Una dimensión es uno o más valores de categorías. La combinación de esos valores identifica una serie temporal determinada de una variable; por ejemplo: país, idioma e inquilino.

## <a name="multi-dimensional-metric"></a>Métrica multidimensional

¿Qué es una métrica multidimensional? Vamos a usar dos ejemplos. 

**Ingresos de una empresa**

Supongamos que tiene datos de los ingresos de su empresa. Los datos de la serie temporal podrían tener un aspecto similar al siguiente:

| Timestamp | Category | Mercado | Ingresos |
| ----------|----------|--------|----- |
| 2020-6-1 | Alimentos | US | 1000 |
| 2020-6-1 | Ropa | US | 2000 |
| 2020-6-2 | Alimentos | Reino Unido | 800 | 
| ...      | ...  |... | ... |

En este ejemplo, *Categoría* y *Mercado* son dimensiones. *Ingresos* es el indicador clave de rendimiento (KPI) que se puede segmentar en diferentes categorías o mercados, y también se podría agregar. Por ejemplo, los ingresos de *alimentos* para todos los mercados.
 
**Recuentos de errores para una aplicación compleja**

Supongamos que tiene datos para el número de errores registrados en una aplicación. Los datos de la serie temporal podrían tener un aspecto similar al siguiente:

| Timestamp | Componente de la aplicación | Region | Recuento de errores |
| ----------|----------|--------|----- |
| 2020-6-1 | Base de datos de empleados | OESTE DE EUROPA | 9000 |
| 2020-6-1 | Cola de mensajes | ESTE DE EE. UU. | 1000 |
| 2020-6-2 | Cola de mensajes | ESTE DE EE. UU. | 8000| 
| ...      | ...           | ...     |  ...|

En este ejemplo, *Componente de la aplicación* y *Región* son dimensiones. *Recuento de errores* es el KPI que se puede segmentar en diferentes categorías o mercados, y también se podría agregar. Por ejemplo, el recuento de errores de *Cola de mensajes* en todas las regiones.

## <a name="measure"></a>Measure

Una medida es un término fundamental o específico de unidad y un valor cuantificable de la métrica.

## <a name="time-series"></a>Serie temporal

Una serie temporal es una serie de puntos de datos indexados (o enumerados o representados mediante grafos) en orden cronológico. Normalmente, una serie temporal es una secuencia tomada en puntos sucesivos igualmente espaciados en el tiempo. Es una secuencia de datos en tiempo discreto.

En Metrics Advisor, los valores de una métrica en una combinación de dimensiones específica se denominan serie.

## <a name="granularity"></a>Granularidad

La granularidad indica la frecuencia con la que se generarán los puntos de datos en el origen de datos. Por ejemplo, diariamente o cada hora.

## <a name="start-time"></a>Hora de inicio

La hora de inicio es la hora a la que desea que Metrics Advisor empiece a ingerir datos del origen de datos. El origen de datos debe tener datos a la hora de inicio especificada.

## <a name="confidence-boundaries"></a>Límites de confianza

> [!NOTE]
> Los límites de confianza no son la única medida que se usa para buscar anomalías. Es posible que el modelo de detección marque como normal los puntos de datos que se encuentran fuera de este límite. 

En Metrics Advisor, los límites de confianza representan la sensibilidad del algoritmo utilizado y se usan para filtrar las anomalías demasiado sensibles. En el portal web, los límites de confianza aparecen como una banda azul transparente. Todos los puntos que se encuentran dentro de la banda se tratan como puntos normales.

Metrics Advisor proporciona herramientas para ajustar la sensibilidad de los algoritmos usados. Consulte [Cómo: Configuración de métricas y ajuste de la configuración de la detección](how-tos/configure-metrics.md) para más información.

![Límites de confianza](media/confidence-bounds.png)


## <a name="hook"></a>Enlace

Metrics Advisor le permite crear alertas en tiempo real y suscribirse a ellas. Estas alertas se envían a través de Internet [mediante un enlace](how-tos/alerts.md).

## <a name="anomaly-incident"></a>Incidente de anomalía

Una vez que se aplica una configuración de detección a las métricas, se generan incidentes siempre que alguna serie dentro de ella tenga una anomalía. En conjuntos de datos grandes, esto puede ser abrumador, por lo que Metrics Advisor agrupa series de anomalías dentro de una métrica en un incidente. El servicio también evaluará la gravedad y proporcionará herramientas para [diagnosticar el incidente](how-tos/diagnose-incident.md).

### <a name="incident-tree"></a>Árbol de incidentes

En Metrics Advisor, puede aplicar la detección de anomalías en las métricas y después Metrics Advisor supervisa automáticamente todas las series temporales de todas las combinaciones de dimensiones. Siempre que se detecta alguna anomalía, Metrics Advisor agrega anomalías a incidentes.
Después de que ocurra un incidente, Metrics Advisor proporcionará un árbol de incidentes con una jerarquía de anomalías contribuyentes e identificará los que tengan el mayor impacto. Cada incidente tiene una anomalía de causa principal, que es el nodo superior del árbol.

### <a name="anomaly-grouping"></a>Agrupación de anomalías

Metrics Advisor proporciona la capacidad de buscar series temporales relacionadas con patrones similares. También puede proporcionar conclusiones más detalladas sobre el impacto en otras dimensiones y correlacionar las anomalías.

### <a name="time-series-comparison"></a>Comparación de series temporales

Puede elegir varias series temporales para comparar tendencias en una sola visualización. Esto proporciona una manera clara y perspicaz de ver y comparar series relacionadas.

## <a name="detection-configuration"></a>Configuración de detección

>[!Note]
>Las configuraciones de detección solo se aplican dentro de una métrica individual.

En el portal web de Metrics Advisor, se muestra una configuración de detección (como la sensibilidad, la posposición automática y la dirección) en el panel izquierdo cuando se visualiza una métrica. Los parámetros se pueden ajustar y aplicar a todas las series dentro de esta métrica.

Se requiere una configuración de detección para cada serie temporal y determina si un punto de la serie temporal es una anomalía. Metrics Advisor establecerá una configuración predeterminada para toda la métrica al incorporar los datos por primera vez. 

Además, puede refinar la configuración aplicando parámetros de ajuste a un grupo de series o a una serie específica. Solo se aplicará una configuración a una serie temporal:
* Las configuraciones que se aplican a una serie específica sobrescribirán las configuraciones para un grupo
* Las configuraciones para un grupo sobrescribirán las configuraciones que se aplican a toda la métrica.

Metrics Advisor proporciona varios [métodos de detección](how-tos/configure-metrics.md#anomaly-detection-methods) y puede combinarlos mediante operadores lógicos.

### <a name="smart-detection"></a>Detección inteligente

Detección de anomalías mediante el uso de varios algoritmos de aprendizaje automático.

**Sensibilidad**: Valor numérico para ajustar la tolerancia de la detección de anomalías. Visualmente, cuanto mayor sea el valor, menor serán los límites superior e inferior de la serie temporal.

### <a name="hard-threshold"></a>Umbral saturado

Los valores que están fuera de los límites superior o inferior son anomalías.

**Mínimo**: El límite inferior.

**Máximo**: El límite superior.

### <a name="change-threshold"></a>Umbral de cambio

Use el valor del punto anterior para determinar si este punto es una anomalía.

**Porcentaje de cambio**: En comparación con el punto anterior, el punto actual es una anomalía si el porcentaje de cambio es mayor que este parámetro.

**Cambio entre puntos**: Número de puntos que se van a examinar.

### <a name="common-parameters"></a>Parámetros comunes

**Dirección**: Un punto es una anomalía solo cuando la desviación se produce en la dirección*arriba*, *abajo* o en *ambas*.

**Anomalía no válida hasta que**: Un punto de datos solo es una anomalía si un porcentaje especificado de puntos anteriores también son anomalías.

## <a name="alert-settings"></a>Configuración de alertas

La configuración de alertas determina qué anomalías deben desencadenar una alerta. Puede establecer varias alertas con distintas configuraciones. Por ejemplo, puede crear una alerta para las anomalías con un menor impacto empresarial y otra para las alertas de mayor importancia.

También puede crear una alerta a través de las métricas. Por ejemplo, una alerta que solo se desencadena si dos métricas especificadas tienen anomalías.

### <a name="alert-scope"></a>Ámbito de alerta

El ámbito de alerta hace referencia al ámbito al que se aplica la alerta. Hay cuatro opciones:

**Anomalías de todas las series**: Las alertas se desencadenarán para las anomalías de todas las series dentro de la métrica.

**Anomalías del grupo de series**: Solo se desencadenarán alertas para las anomalías de dimensiones específicas del grupo de series. El número de dimensiones especificadas debe ser menor que el número total de dimensiones.

**Anomalías de series favoritas**: Solo se desencadenarán alertas para las anomalías que se agreguen como favoritos. Puede elegir un grupo de series como favorito para cada configuración de detección.

**Anomalías en las N series principales de todas las series**: Las alertas solo se desencadenarán para las anomalías de las N series principales. Puede establecer parámetros para especificar el número de marcas de tiempo que se deben tener en cuenta y cuántas anomalías deben encontrarse en ellas para enviar la alerta.

### <a name="severity"></a>Gravedad

La gravedad es una calificación que Metrics Advisor utiliza para describir la gravedad del incidente; las opciones son *Alta*, *Media* y *Baja*.

Actualmente, Metrics Advisor usa los siguientes factores para medir la gravedad de alerta:
1. La proporción del valor y la proporción de la cantidad de anomalías de la métrica.
1. Confianza de anomalías.
1. La configuración favorita también contribuye a la gravedad.

### <a name="auto-snooze"></a>Posponer automáticamente

Algunas anomalías son problemas transitorios, especialmente para métricas de granularidad pequeña. Puede *posponer* una alerta para un número específico de puntos de tiempo. Si se encuentran anomalías en el número especificado de puntos, no se desencadenará ninguna alerta. El comportamiento de la posposición automática se puede establecer en el nivel de métrica o de serie.

El comportamiento de la posposición se puede establecer en el nivel de métrica o de serie.

## <a name="data-feed-settings"></a>Configuración de la fuente de distribución de datos

### <a name="ingestion-time-offset"></a>Desfase en el tiempo de ingesta

De forma predeterminada, los datos se ingieren según la granularidad (por ejemplo, *diariamente*). Mediante el uso de un entero positivo, puede retrasar la ingesta de los datos según el valor especificado. Con un número negativo, puede adelantar la ingesta según el valor especificado.

### <a name="max-ingestion-per-minute"></a>Ingesta máxima por minuto

Establezca este parámetro si el origen de datos admite la simultaneidad limitada. De lo contrario, deje la configuración predeterminada.

### <a name="stop-retry-after"></a>Detener reintento después de

Si se ha producido un error en la ingesta de datos, Metrics Advisor volverá a intentarlo automáticamente transcurrido un período de tiempo. El comienzo del período es la hora a la que se produjo la primera ingesta de datos. La duración del período de reintento se define según la granularidad. Si usa el valor predeterminado (`-1`), el período de reintento se determinará en función de la granularidad:

| Granularidad       | Detener reintento después de           |
| :------------ | :--------------- |
| Diariamente, Personalizado (>= 1 día), Semanalmente, Mensualmente y Anualmente     | 7 días          |
| Cada hora y Personalizado (< 1 día)       | 72 horas |

### <a name="min-retry-interval"></a>Intervalo de reintento mínimo

Puede especificar el intervalo mínimo al reintentar la extracción de datos del origen. Si usa el valor predeterminado (`-1`), el intervalo de reintento se determinará en función de la granularidad:

| Granularidad       | Intervalo mínimo de reintento           |
| :------------ | :--------------- |
| Diariamente, Personalizado (>=1 día), Semanalmente y Mensualmente     | 30 minutos          |
| Cada hora y Personalizado (< 1 día)      | 10 minutos |
| Anual | 1 día          |

### <a name="grace-period"></a>Período de gracia

> [!Note]
> El período de gracia comienza en el tiempo de ingesta normal, más el desplazamiento de tiempo de ingesta especificado.
    
Un período de gracia es un período de tiempo en el que Metrics Advisor seguirá capturando datos del origen de datos, pero no activará ninguna alerta. Si no se ingirió ningún dato después del período de gracia, se desencadenará la alerta *Fuente de distribución de datos no disponible*.

### <a name="snooze-alerts-in"></a>Posponer alertas en

Cuando esta opción se establece en cero, cada marca de tiempo con *No disponible* desencadenará una alerta. Cuando se establece en un valor distinto de cero, el número especificado de alertas de tipo *No disponible* se pospondrá si no se ha capturado ningún dato.

## <a name="data-feed-permissions"></a>Permisos de fuente de distribución de datos

Existen dos roles para administrar los permisos de fuente de distribución de datos: *Administrador* y *Espectador*. 

* Un *administrador* tiene control total sobre la fuente de distribución de datos y las métricas que contiene. Pueden activar, pausar y eliminar la fuente de distribución de datos, así como efectuar actualizaciones en fuentes y configuraciones. Normalmente, un *administrador* es el propietario de las métricas.

* Un *espectador* puede ver la fuente de distribución de datos o las métricas, pero no puede realizar cambios. 

## <a name="next-steps"></a>Pasos siguientes
- [Introducción a Metrics Advisor](overview.md)
- [Prueba del sitio de demostración](quickstarts/explore-demo.md)
- [Uso del portal web](quickstarts/web-portal.md)