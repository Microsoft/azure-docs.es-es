---
title: Unidades de streaming en Azure Stream Analytics
description: En este artículo se describe la configuración de unidades de streaming y otros factores que afectan al rendimiento en Azure Stream Analytics.
author: JSeb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/06/2020
ms.openlocfilehash: 5d16e7f81a439d622a418dbc8cdff2d66c2a814f
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87903568"
---
# <a name="understand-and-adjust-streaming-units"></a>Descripción y ajuste de las unidades de streaming

Las unidades de streaming (SU) representan los recursos informáticos que se asignan para ejecutar un trabajo de Stream Analytics. Cuanto mayor sea el número de unidades de streaming, más recursos de CPU y memoria se asignarán al trabajo. Esta función le permite centrarse en la lógica de consulta y abstrae la necesidad de administrar el hardware para ejecutar el trabajo de Stream Analytics de manera oportuna.

Para lograr un procesamiento de streaming de latencia baja, los trabajos de Azure Stream Analytics realizan todo el procesamiento en memoria. El trabajo de streaming presenta un error cuando se queda sin memoria. Como resultado, en el caso de un trabajo de producción, resulta importante supervisar el uso de recursos de un trabajo de streaming, y debe asegurarse de que hay recursos asignados suficientes para mantener los trabajos en ejecución de manera ininterrumpida.

La métrica de uso del % de SU, que oscila de 0 % al 100 %, describe el consumo de memoria de la carga de trabajo. Para un trabajo de streaming con una superficie mínima, esta métrica suele estar entre el 10 % y el 20 %. Si el porcentaje de uso de SU es bajo y los eventos de entrada quedan pendientes, puede que la carga de trabajo requiera más recursos informáticos, para lo que es necesario aumentar el número de SU. Se recomienda mantener la métrica de SU por debajo del 80 % para tener en cuenta las subidas ocasionales. Microsoft recomienda establecer una alerta en un 80 % de métrica de uso de SU para evitar el agotamiento de recursos. Para más información, consulte el [Tutorial: Configuración de alertas en Azure Stream Analytics](stream-analytics-set-up-alerts.md).

## <a name="configure-stream-analytics-streaming-units-sus"></a>Configuración de las unidades de streaming (SU) de Stream Analytics
1. Inicie sesión en [Azure Portal](https://portal.azure.com/)

2. En la lista de recursos, busque el trabajo de Stream Analytics que quiere escalar y ábralo. 

3. En la página del trabajo, en el encabezado **Configurar**, seleccione **Escalar**. El número predeterminado de SUs es 3 cuando se crea un trabajo.

    ![Configuración del trabajo de Stream Analytics en Azure Portal][img.stream.analytics.preview.portal.settings.scale]
    
4. Use el control deslizante para establecer las unidades de streaming del trabajo. Tenga en cuenta que está limitado a la configuración de unidades de streaming específica. 
5. Puede cambiar el número de SU asignados a su trabajo incluso cuando se está ejecutando. Esto no es posible si el trabajo usa una [salida sin particiones](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-parallelization#query-using-non-partitioned-output) o si tiene una [consulta de varios pasos con diferentes valores de PARTITION BY](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-parallelization#multi-step-query-with-different-partition-by-values). Quizás se vea restringido de elegir entre un conjunto de valores de SU cuando el trabajo se está ejecutando. 

## <a name="monitor-job-performance"></a>Supervisión del rendimiento del trabajo
Mediante Azure Portal, puede realizar el seguimiento de la capacidad de procesamiento de un trabajo:

![Trabajos de supervisión de Azure Stream Analytics][img.stream.analytics.monitor.job]

Calcule la capacidad de procesamiento esperada de la carga de trabajo. Si la capacidad de procesamiento es inferior a la esperada, ajuste la partición de entrada, ajuste la consulta y agregue unidades de streaming al trabajo.

## <a name="how-many-sus-are-required-for-a-job"></a>¿Cuántas SU son necesarias para un trabajo?

La elección del número de SU necesarias para un trabajo determinado depende de la configuración de particiones de las entradas y de la consulta definida dentro del trabajo. La página **Escala** le permite establecer el número correcto de SU. Es recomendable asignar más unidades de streaming de las necesarias. El motor de procesamiento de Stream Analytics se optimiza para la latencia y el rendimiento, a costa de asignar memoria adicional.

En general, el procedimiento recomendado es comenzar con 6 SU para consultas que no usen **PARTITION BY**. Luego, determine el punto favorable mediante un método de prueba y error en el que modifica el número de SU después de pasar las cantidades de datos representativas y examinar la métrica de porcentaje de uso de SU. El número máximo de unidades de streaming que se puede utilizar en un trabajo de Stream Analytics depende del número de pasos de la consulta definida para el trabajo y del número de particiones en cada paso. Puede obtener más información sobre los límites [aquí](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-parallelization#calculate-the-maximum-streaming-units-of-a-job).

Para más información sobre la elección del número correcto de unidades de streaming, consulte esta página: [Escalar los trabajos de Azure Stream Analytics para incrementar el rendimiento](stream-analytics-scale-jobs.md)

> [!Note]
> La elección del número de unidades de streaming que se necesitan para un trabajo en concreto depende de la configuración de particiones para las entradas y de la consulta definida para el trabajo. Puede seleccionar como máximo su cuota en unidades de streaming para un trabajo. De manera predeterminada, cada suscripción de Azure tiene una cuota máxima de 500 SU en todos los trabajos de análisis de una región específica. Para aumentar las unidades de streaming de sus suscripciones, contacte con [Soporte técnico de Microsoft](https://support.microsoft.com). Los valores válidos para unidades de streaming por trabajo son 1, 3, 6 y más en incrementos de 6.

## <a name="factors-that-increase-su-utilization"></a>Factores que aumentar el uso de porcentaje de SU 

Los elementos de consulta temporal (orientados al tiempo) son el conjunto básico de operadores con estado proporcionados por Stream Analytics. Stream Analytics administra el estado de estas operaciones internamente en nombre de usuario, mediante la administración del consumo de memoria, los puntos de comprobación para la resistencia y la recuperación del estado durante las actualizaciones de servicio. Aunque Stream Analytics administra totalmente los estados, hay una serie de prácticas recomendadas que los usuarios deberían tener en cuenta.

Tenga en cuenta que un trabajo con lógica de consulta compleja podría tener un alto porcentaje de utilización de unidades de streaming, incluso cuando no está recibiendo continuamente eventos de entrada. Esto puede suceder después de un repentino aumento de los eventos de entrada y salida. El trabajo puede continuar manteniendo el estado en memoria si la consulta es compleja.

El porcentaje de utilización de unidades de streaming podría caer repentinamente a 0 durante un breve período antes de volver a los niveles esperados. Esto sucede debido a errores transitorios o a actualizaciones del sistema que se hayan iniciado. Aumentar el número de unidades de streaming para un trabajo podría no reducir el porcentaje de utilización de SU si la consulta no es [totalmente paralela](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-parallelization).

Al comparar el uso durante un período de tiempo, utilice las [métricas de velocidad de eventos](stream-analytics-monitoring.md). Las métricas InputEvents y OutputEvents muestran el número de eventos que se leyeron y procesaron. También hay métricas que indican el número de eventos de error, como los errores de deserialización. Cuando aumenta el número de eventos por unidad de tiempo, el porcentaje de unidades de almacenamiento aumenta en la mayoría de los casos.

## <a name="stateful-query-logicin-temporal-elements"></a>Lógica de consulta con estado en elementos temporales
Una funcionalidad única de un trabajo de Azure Stream Analytics es realizar procesamiento con estado, como funciones de análisis temporal, combinaciones temporales y agregados en ventanas. Cada uno de estos operadores conserva información de estado. El tamaño máximo de la ventana para estos elementos de consulta es siete días. 

El concepto de ventana temporal aparece en varios elementos de consulta de Stream Analytics:
1. Agregados en ventanas: GROUP BY de ventanas de saltos de tamaño constante, de salto y deslizantes

2. Combinaciones temporales: Función JOIN con DATEDIFF

3. Funciones de análisis temporal: ISFIRST, LAST y LAG con LIMIT DURATION

Los siguientes factores influyen en la memoria utilizada (que forma parte de la métrica de unidades de streaming) por los trabajos de Stream Analytics:

## <a name="windowed-aggregates"></a>Agregados en ventanas
La memoria utilizada (tamaño de estado) para un agregado en ventanas no siempre es directamente proporcional al tamaño de la ventana. En su lugar, la memoria utilizada es proporcional a la cardinalidad de los datos o el número de grupos de cada ventana temporal.


Por ejemplo, en la consulta siguiente, el número asociado con `clusterid` es la cardinalidad de la consulta. 

   ```sql
   SELECT count(*)
   FROM input 
   GROUP BY  clusterid, tumblingwindow (minutes, 5)
   ```

Con el fin de mitigar los problemas causados por la cardinalidad alta en la consulta anterior, puede enviar eventos a Event Hub particionados por `clusterid` y escalar horizontalmente la consulta permitiendo que el sistema procese de manera independiente cada partición de entrada con **PARTITION BY**, como se muestra en el ejemplo siguiente:

   ```sql
   SELECT count(*) 
   FROM input PARTITION BY PartitionId
   GROUP BY PartitionId, clusterid, tumblingwindow (minutes, 5)
   ```

Una vez que la consulta está particionada, se extiende por varios nodos. Como resultado, el número de valores `clusterid` que entra en cada nodo se reduce, lo que disminuye la cardinalidad del grupo por operador. 

Las particiones de Event Hub se deben particionar según la clave de agrupación para evitar la necesidad de un paso de reducción. Para obtener más información, consulte [Información general sobre Event Hubs](../event-hubs/event-hubs-what-is-event-hubs.md). 

## <a name="temporal-joins"></a>Combinaciones temporales
La memoria consumida (tamaño del estado) de una unión temporal es proporcional al número de eventos en el espacio de ondulación temporal de la unión, que es la tasa de entrada del evento multiplicada por el tamaño del espacio de ondulación. En otras palabras, la memoria utilizada por las combinaciones es proporcional al intervalo de tiempo DateDiff multiplicado por la tasa media de los eventos.

La cantidad de eventos no coincidentes en la combinación afecta el uso de memoria de la consulta. La consulta siguiente busca las impresiones de anuncios que generan clics:

   ```sql
   SELECT clicks.id
   FROM clicks 
   INNER JOIN impressions ON impressions.id = clicks.id AND DATEDIFF(hour, impressions, clicks) between 0 AND 10.
   ```

En este ejemplo, es posible que se muestren muchos anuncios, pero que pocas personas hagan clic en ellos, por lo que es necesario mantener todos los eventos en la ventana de tiempo. La memoria consumida es proporcional al tamaño de ventana y la tasa de eventos. 

Para corregir esto, envíe eventos a Event Hub particionados por las claves de unión (en este caso, el identificador) y escale horizontalmente la consulta permitiendo que el sistema procese de manera independiente cada partición de entrada con **PARTITION BY**, como se muestra a continuación:

   ```sql
   SELECT clicks.id
   FROM clicks PARTITION BY PartitionId
   INNER JOIN impressions PARTITION BY PartitionId 
   ON impression.PartitionId = clicks.PartitionId AND impressions.id = clicks.id AND DATEDIFF(hour, impressions, clicks) between 0 AND 10 
   ```

Una vez que la consulta está particionada, se extiende por varios nodos. Como resultado, el número de eventos que entra en cada nodo se reduce, lo que disminuye el tamaño del estado que se conserva en la ventana de la combinación. 

## <a name="temporal-analytic-functions"></a>Funciones de análisis temporal
La memoria consumida (tamaño del estado) de una función de análisis temporal es proporcional a la tasa de eventos multiplicada por la duración. La memoria utilizada por las funciones analíticas no es proporcional al tamaño de la ventana, sino más bien a la cantidad de particiones de cada ventana temporal.

La solución es similar a la combinación temporal. Puede escalar horizontalmente la consulta con **PARTITION BY**. 

## <a name="out-of-order-buffer"></a>Búfer desordenado 
El usuario puede configurar el tamaño de búfer desordenado en el panel de configuración de la ordenación de eventos. El búfer se usa para contener entradas para la duración de la ventana y para reordenarlas. El tamaño del búfer es proporcional a la tasa de entrada de eventos multiplicada por el tamaño de la ventana desordenada. El tamaño predeterminado de la ventana es 0. 

Para corregir el desbordamiento del búfer desordenado, escale la consulta horizontalmente con **PARTITION BY**. Una vez que la consulta está particionada, se extiende por varios nodos. Como resultado, el número de eventos que entra en cada nodo se reduce, lo que disminuye el número de eventos en cada búfer de reordenación. 

## <a name="input-partition-count"></a>Número de particiones de entrada 
Cada partición de entrada de una entrada del trabajo tiene un búfer. Mientras más grande sea la cantidad de particiones de entrada, más recursos consume el trabajo. Para cada unidad de streaming, Azure Stream Analytics puede procesar aproximadamente 1 MB/s de entrada. Por lo tanto, es posible mejorarlo haciendo coincidir el número de unidades de streaming de Stream Analytics con el número de particiones de Event Hub. 

Por lo general, un trabajo configurado con una unidad de streaming es suficiente para un Event Hub con dos particiones (que es el valor mínimo para el Event Hub). Si Event Hub tiene más particiones, el trabajo de Stream Analytics consume más recursos, pero no consume necesariamente el rendimiento adicional que proporciona Event Hub. 

En el caso de un trabajo de 6 unidades de streaming, podría necesitar 4 u 8 particiones de Event Hub. Sin embargo, evite demasiadas particiones innecesarias, porque provoca un uso de recursos excesivo. Por ejemplo, un Event Hub con 16 particiones o más en un trabajo de Stream Analytics que tiene 1 unidad de streaming. 

## <a name="reference-data"></a>Datos de referencia 
Los datos de referencia en ASA se cargan en la memoria para poder realizar una búsqueda rápida. Con la implementación actual, cada operación de combinación con datos de referencia conserva una copia de estos datos en la memoria, incluso si se combina varias veces con los mismos datos de referencia. Para consultas con **PARTITION BY**, cada partición tiene una copia de los datos de referencia, por lo que las particiones están completamente desacopladas. Con el efecto multiplicador, el uso de la memoria puede aumentar muchísimo rápidamente si combina con datos de referencia varias veces con varias particiones.  

### <a name="use-of-udf-functions"></a>Uso de las funciones UDF
Cuando agrega una función UDF, Azure Stream Analytics carga el entorno de tiempo de ejecución de Java en la memoria. Esto afectará el porcentaje de SU.

## <a name="next-steps"></a>Pasos siguientes
* [Creación de consultas que se pueden paralelizar en Azure Stream Analytics](stream-analytics-parallelization.md)
* [Escalar los trabajos de Azure Stream Analytics para incrementar el rendimiento](stream-analytics-scale-jobs.md)

<!--Image references-->

[img.stream.analytics.monitor.job]: ./media/stream-analytics-scale-jobs/StreamAnalytics.job.monitor-NewPortal.png
[img.stream.analytics.configure.scale]: ./media/stream-analytics-scale-jobs/StreamAnalytics.configure.scale.png
[img.stream.analytics.perfgraph]: ./media/stream-analytics-scale-jobs/perf.png
[img.stream.analytics.streaming.units.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsStreamingUnitsExample.jpg
[img.stream.analytics.preview.portal.settings.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsPreviewPortalJobSettings-NewPortal.png   
