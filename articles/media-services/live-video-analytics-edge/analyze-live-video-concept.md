---
title: Análisis del vídeo en directo sin grabación - Azure
description: Un grafo multimedia se puede usar para extraer análisis de una secuencia de vídeo en directo sin tener que grabarlo en el borde ni en la nube. En este artículo se describe este concepto.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 5dda18b68cb19d29623f2120fe07d7cc617f0c2f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90893020"
---
# <a name="analyzing-live-video-without-any-recording"></a>Análisis del vídeo en directo sin grabación

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura 

* [Concepto de grafo multimedia](media-graph-concept.md)
* [Grabación de vídeo basada en eventos](event-based-video-recording-concept.md)

## <a name="overview"></a>Información general  

Puede usar un grafo multimedia para analizar vídeo en directo, sin grabar partes del vídeo en un archivo ni un recurso. Los grafos multimedia que se muestran a continuación son similares a los del artículo sobre [grabación de vídeo basada en eventos](event-based-video-recording-concept.md), pero sin un nodo receptor de recursos o de archivos.

### <a name="motion-detection"></a>Detección de movimiento

El grafo multimedia que se muestra a continuación consta de un nodo de [origen RTSP](media-graph-concept.md#rtsp-source), un [procesador de detección de movimiento](media-graph-concept.md#motion-detection-processor) y un nodo [receptor de mensajes de IoT Hub](media-graph-concept.md#iot-hub-message-sink). La representación JSON de la topología de grafo de este tipo de grafo multimedia se puede encontrar [aquí](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/motion-detection/topology.json). Este grafo permite detectar movimiento en la secuencia de vídeo entrante en directo y retransmitir los eventos de movimiento a otras aplicaciones y servicios mediante el nodo receptor de mensajes de IoT Hub. Las aplicaciones o servicios externos pueden desencadenar una alerta o enviar una notificación al personal adecuado.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/motion-detection.svg" alt-text="Live Video Analytics basado en la detección de movimiento":::

### <a name="analyzing-video-using-a-custom-vision-model"></a>Analizar vídeo con un modelo de visión personalizado 

El grafo multimedia que se muestra a continuación le permite analizar una secuencia de vídeo en directo mediante un modelo de visión personalizado empaquetado en un módulo independiente. La representación JSON de la topología de grafo de este grafo multimedia se puede encontrar [aquí](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/httpExtension/topology.json). Puede ver algunos ejemplos [aquí](https://github.com/Azure/live-video-analytics/tree/master/utilities/video-analysis) sobre el ajuste de modelos en módulos de IoT Edge que se ejecutan como un servicio de inferencia.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/motion-detected-frames.svg" alt-text="Live Video Analytics basado en un módulo de inferencia externo":::

En este grafo multimedia, el nodo procesador de filtro de velocidad de fotogramas reduce la velocidad de fotogramas de la secuencia de vídeo entrante en directo antes de enviarla a un nodo de [procesador de extensiones HTTP](media-graph-concept.md#http-extension-processor) que envía fotogramas de imagen (en formatos JPEG, BMP o PNG) a un servicio de inferencia externo sobre REST. Los resultados del servicio de inferencia externa se recuperan mediante el nodo de extensión HTTP y se retransmiten al centro de IoT Edge mediante el nodo receptor de mensajes de IoT Hub. Este tipo de gráfico multimedia se puede usar para crear soluciones para una variedad de escenarios, como comprender la distribución de la serie temporal de vehículos en una intersección, comprender el patrón de tráfico del consumidor en una tienda, etc.

Una mejora de este ejemplo es usar un procesador de detección de movimiento antes del nodo del procesador de filtros de velocidad de fotogramas. Esto reducirá la carga en el servicio de inferencia, ya que solo se usa cuando hay actividad de movimiento en el vídeo.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/custom-model.svg" alt-text="Live Video Analytics basado en movimiento detectó fotogramas mediante un módulo de inferencia externo":::

## <a name="next-steps"></a>Pasos siguientes

[Grabación de vídeo continua](continuous-video-recording-concept.md)
