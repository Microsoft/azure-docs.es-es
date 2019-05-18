---
title: Detección de anomalías en Azure Stream Analytics (versión preliminar)
description: En este artículo se describe cómo usar Azure Stream Analytics y Azure Machine Learning conjuntamente para detectar anomalías.
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 02/13/2019
ms.custom: seodec18
ms.openlocfilehash: 6f1b40e6224b9ed872700d526b954c605159d6bf
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65789622"
---
# <a name="anomaly-detection-in-azure-stream-analytics"></a>Detección de anomalías en Azure Stream Analytics

Disponible tanto en la nube como en Azure IoT Edge, Azure Stream Analytics ofrece funcionalidades de detección de anomalías integradas basadas en aprendizaje automático, que se pueden usar para supervisar las dos anomalías que se producen con más frecuencia: temporales y persistentes. Con las funciones **AnomalyDetection_SpikeAndDip** y **AnomalyDetection_ChangePoint**, puede realizar la detección de anomalías directamente en el trabajo de Stream Analytics.

Los modelos de aprendizaje automático asumen una serie temporal muestreada uniformemente. Si la serie temporal no es uniforme, puede insertar un paso de agregación con una ventana de saltos de tamaño constante antes de llamar a la detección de anomalías.

Las operaciones de aprendizaje automático no admiten las tendencias de estacionalidad ni las correlaciones múltiples variadas.

## <a name="model-accuracy-and-performance"></a>Rendimiento y precisión de modelos

Por lo general, la precisión del modelo mejora con más datos en la ventana deslizante. Los datos de la ventana deslizante especificada se tratan como parte de su rango normal de valores para ese período. El modelo solo tiene en cuenta el historial de eventos a través de la ventana deslizante para comprobar si el evento actual es anómalo. Cuando se mueve la ventana deslizante, los valores antiguos se expulsan del entrenamiento del modelo.

Las funciones operan estableciendo un valor normal determinado en función de lo han observado hasta ahora. Los valores atípicos se identifican mediante la comparación con el valor normal establecido, en el nivel de confianza. El tamaño de ventana debe basarse en los eventos mínimos necesarios para entrenar el modelo para el comportamiento normal, con el fin de que cuando se produzca alguna anomalía, pueda reconocerlos.

Tenga en cuenta que el tiempo de respuesta del modelo aumenta con el tamaño del historial porque se debe comparar con un mayor número de eventos anteriores. Se recomienda que incluya solo el número de eventos para mejorar el rendimiento necesario.

Las brechas en la serie temporal pueden producirse porque el modelo no recibe los eventos en determinados puntos en el tiempo. Stream Analytics controla esta situación mediante imputación. El tamaño del historial, así como una duración, en la misma ventana deslizante se usa para calcular la velocidad media a la que se esperan que lleguen los eventos.

## <a name="spike-and-dip"></a>Picos e interrupciones

A las anomalías temporales de un flujo de eventos de serie temporal se les conocen como picos e interrupciones. Los picos y las interrupciones pueden supervisarse mediante el operador basado en Machine Learning, [AnomalyDetection_SpikeAndDip](https://docs.microsoft.com/stream-analytics-query/anomalydetection-spikeanddip-azure-stream-analytics
).

![Ejemplo de anomalías de picos e interrupciones](./media/stream-analytics-machine-learning-anomaly-detection/anomaly-detection-spike-dip.png)

En la misma ventana deslizante, si un pico de segundo es menor que el primero, la puntuación calculada del pico más pequeño probablemente no es lo suficientemente significativo en comparación con la puntuación del primer pico en el nivel de confianza especificado. Puede intentar reducir la configuración del nivel de confianza del modelo para capturar estas anomalías. Sin embargo, si empieza a recibir demasiadas alertas, puede usar un intervalo de confianza superior.

En la siguiente consulta de ejemplo se da por supuesto una velocidad uniforme de entrada de un evento por segundo en una ventana deslizante de 2 minutos con un historial de 120 eventos. La instrucción SELECT final extrae y produce la puntuación y el estado de anomalía con un nivel de confianza del 95 %.

```SQL
WITH AnomalyDetectionStep AS
(
    SELECT
        EVENTENQUEUEDUTCTIME AS time,
        CAST(temperature AS float) AS temp,
        AnomalyDetection_SpikeAndDip(CAST(temperature AS float), 95, 120, 'spikesanddips')
            OVER(LIMIT DURATION(second, 120)) AS SpikeAndDipScores
    FROM input
)
SELECT
    time,
    temp,
    CAST(GetRecordPropertyValue(SpikeAndDipScores, 'Score') AS float) AS
    SpikeAndDipScore,
    CAST(GetRecordPropertyValue(SpikeAndDipScores, 'IsAnomaly') AS bigint) AS
    IsSpikeAndDipAnomaly
INTO output
FROM AnomalyDetectionStep
```

## <a name="change-point"></a>Cambio de puntos

Las anomalías persistentes en un flujo de eventos de serie temporal son los cambios en la distribución de valores del flujo de eventos, como los cambios de nivel y tendencias. En Stream Analytics, estas anomalías se detectan mediante el operador [AnomalyDetection_ChangePoint](https://docs.microsoft.com/stream-analytics-query/anomalydetection-changepoint-azure-stream-analytics) basado en Machine Learning.

Los cambios persistentes duran mucho más que los picos y las interrupciones, y podrían indicar eventos catastróficos. Los cambios persistentes normalmente no son visibles a simple vista, pero se pueden detectar con el operador **AnomalyDetection_ChangePoint**.

La imagen siguiente es un ejemplo de un cambio de nivel:

![Ejemplo de anomalía de cambio de nivel](./media/stream-analytics-machine-learning-anomaly-detection/anomaly-detection-level-change.png)

La imagen siguiente es un ejemplo de un cambio de tendencia:

![Ejemplo de anomalía de cambio de tendencia](./media/stream-analytics-machine-learning-anomaly-detection/anomaly-detection-trend-change.png)

En la siguiente consulta de ejemplo se da por supuesto una velocidad uniforme de entrada de un evento por segundo en una ventana deslizante de 20 minutos con un historial de 1200 eventos. La instrucción SELECT final extrae y produce la puntuación y el estado de anomalía con un nivel de confianza del 80 %.

```SQL
WITH AnomalyDetectionStep AS
(
    SELECT
        EVENTENQUEUEDUTCTIME AS time,
        CAST(temperature AS float) AS temp,
        AnomalyDetection_ChangePoint(CAST(temperature AS float), 80, 1200) 
        OVER(LIMIT DURATION(minute, 20)) AS ChangePointScores
    FROM input
)
SELECT
    time,
    temp,
    CAST(GetRecordPropertyValue(ChangePointScores, 'Score') AS float) AS
    ChangePointScore,
    CAST(GetRecordPropertyValue(ChangePointScores, 'IsAnomaly') AS bigint) AS
    IsChangePointAnomaly
INTO output
FROM AnomalyDetectionStep

```

## <a name="anomaly-detection-using-machine-learning-in-azure-stream-analytics"></a>Detección de anomalías con machine learning en Azure Stream Analytics

El vídeo siguiente muestra cómo detectar una anomalía en tiempo real mediante funciones de machine learning en Azure Stream Analytics. 

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Anomaly-detection-using-machine-learning-in-Azure-Stream-Analytics/player]

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalación de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn834998.ASpx)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)

