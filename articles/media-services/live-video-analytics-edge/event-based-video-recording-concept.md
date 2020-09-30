---
title: 'Grabación de vídeo basada en eventos: Azure'
description: La grabación de vídeo basada en eventos (EVR) hace referencia al proceso de grabación de vídeo desencadenado por un evento. El evento en cuestión podría generarse a partir del procesamiento de la propia señal de vídeo (por ejemplo, al detectar un objeto en movimiento) o desde un origen independiente (por ejemplo, la apertura de una puerta).  En este artículo se describen algunos casos de uso relacionados con la grabación de vídeo basada en eventos.
ms.topic: conceptual
ms.date: 05/27/2020
ms.openlocfilehash: f3efd2b9be41928ab4721d6db4aa84c0f1f57e2f
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89568525"
---
# <a name="event-based-video-recording"></a>Grabación de vídeo basada en eventos  
 
## <a name="suggested-pre-reading"></a>Sugerencias previas a la lectura  

* [Grabación de vídeo continua](continuous-video-recording-concept.md)
* [Reproducción de contenido grabado](video-playback-concept.md)
* [Concepto de grafo multimedia](media-graph-concept.md)

## <a name="overview"></a>Información general 

La grabación de vídeo basada en eventos (EVR) hace referencia al proceso de grabación de vídeo desencadenado por un evento. El evento en cuestión podría generarse a partir del procesamiento de la propia señal de vídeo (por ejemplo, al detectar un objeto en movimiento) o desde un origen independiente (por ejemplo, la apertura de una puerta). 

Puede grabar vídeo (desencadenado por un evento) desde una cámara de videovigilancia en un recurso de Media Services mediante un grafo multimedia compuesto por un nodo de [origen RTSP](media-graph-concept.md#rtsp-source), un nodo [receptor de recursos](media-graph-concept.md#asset-sink) y otros nodos, tal y como se describe en los siguientes casos de uso. Puede configurar el nodo [receptor de recursos](media-graph-concept.md#asset-sink) para generar nuevos recursos cada vez que se produzca un evento, de modo que el vídeo que corresponde con cada evento esté en su propio recurso. También puede optar por usar un recurso para contener el vídeo de todos los eventos. 

Como alternativa al nodo receptor de recursos, puede usar un nodo [receptor de archivos](media-graph-concept.md#file-sink) para capturar fragmentos cortos de vídeo (relacionados con un evento de interés) en una ubicación especificada del sistema de archivos local del dispositivo Edge. 

En este artículo se describen algunos casos de uso relacionados con la grabación de vídeo basada en eventos.

### <a name="video-recording-based-on-motion-detection"></a>Grabación de vídeo basada en la detección de movimiento  

En este caso de uso, puede grabar clips de vídeo solo cuando se detecten movimientos y recibir una alerta cada vez que se genere un clip de vídeo. Esto puede ser relevante en escenarios de seguridad comercial que impliquen la protección de infraestructura crítica. Al generar alertas y grabar clips de vídeo cuando se detectan movimientos inesperados, puede mejorar la eficacia del operador humano, ya que solo se necesita tomar medidas cuando se genera una alerta.

En el diagrama siguiente se muestra una representación gráfica de un grafo multimedia para este caso de uso. La representación JSON de la topología de grafo de este tipo de grafo multimedia se puede encontrar [aquí](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/evr-motion-assets/topology.json).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/motion-detection.svg" alt-text="Grabación de vídeo basada en la detección de movimiento":::

En el diagrama, el nodo de origen RTSP captura la fuente de vídeo en directo de la cámara y la entrega a un nodo del [procesador de detección de movimiento](media-graph-concept.md#motion-detection-processor). Después de detectar el movimiento en el vídeo en directo, el nodo del procesador de detección de movimiento genera un evento que va al nodo del [procesador de la puerta de señales](media-graph-concept.md#signal-gate-processor), así como al nodo receptor de mensajes de IoT Hub. El último nodo envía los eventos al centro de IoT Edge, desde donde se pueden enrutar a otros destinos para desencadenar alertas. 

Un evento del nodo detector de movimiento desencadenará el nodo del procesador de la puerta de señales y permitirá que la fuente de vídeo en directo del nodo de origen RTSP pase a través del nodo receptor de recursos. En ausencia de estos eventos posteriores de detección de movimiento, la puerta se cerrará después de un período de tiempo configurado. Esto determina la duración del vídeo grabado.

### <a name="video-recording-based-on-events-from-other-sources"></a>Grabación de vídeo basada en eventos desde otros orígenes  

En este caso de uso, se pueden usar las señales de otro sensor de IoT para desencadenar la grabación de vídeos. En el diagrama siguiente se muestra una representación gráfica de un grafo multimedia para este caso de uso. La representación JSON de la topología de grafo de este tipo de grafo multimedia se puede encontrar [aquí](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/evr-hubMessage-files/topology.json).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/other-sources.svg" alt-text="Grabación de vídeo basada en eventos desde otros orígenes":::

En el diagrama, el sensor externo envía eventos al centro de IoT Edge. Después, los eventos se enrutan al nodo del procesador de la puerta de señales a través del nodo de [origen de mensajes de IoT Hub](media-graph-concept.md#iot-hub-message-source). El comportamiento del nodo del procesador de la puerta de señales es el mismo que el del caso de uso anterior; se abrirá y permitirá que la fuente de vídeo en directo pase del nodo de origen RTSP al nodo receptor de archivos (o al nodo receptor de recursos) cuando el evento externo desencadene la grabación. 

Si usa un nodo receptor de archivos, el vídeo se grabará en el sistema de archivos local del dispositivo perimetral. Por el contrario, si usa un nodo receptor de recursos, el vídeo se grabará en un recurso.

### <a name="video-recording-based-on-an-external-inferencing-module"></a>Grabación de vídeo basada en un módulo de inferencia externo 

En este caso de uso, puede grabar clips de vídeo basados en una señal de un sistema lógico externo. Un ejemplo de este tipo de caso de uso podría ser la grabación de un clip de vídeo solo cuando se detecta un camión en la fuente de vídeo del tráfico en una autopista. En el diagrama siguiente se muestra una representación gráfica de un grafo multimedia para este caso de uso. La representación JSON de la topología de grafo de este tipo de grafo multimedia se puede encontrar [aquí](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/evr-hubMessage-assets/topology.json).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/external-inferencing-module.svg" alt-text="Grabación de vídeo basada en un módulo de inferencia externo":::

En el diagrama, el nodo de origen RTSP captura la fuente de vídeo en directo de la cámara y la entrega a dos ramas: una tiene un nodo del [procesador de la puerta de señales](media-graph-concept.md#signal-gate-processor) y la otra usa un nodo de [extensión HTTP](media-graph-concept.md) para enviar datos a un módulo lógico externo. El nodo de extensión HTTP permite que el grafo multimedia envíe fotogramas de imagen (en formato JPEG, BMP o PNG) a un servicio de inferencia externo a través de REST. Normalmente, esta ruta de señal solo admite velocidades de fotogramas bajas (<5 fps). Puede usar el nodo del [procesador de filtros de velocidad de fotogramas](media-graph-concept.md#frame-rate-filter-processor) para reducir la velocidad de fotogramas del vídeo que pasa al nodo de extensión HTTP.

Los resultados del servicio de inferencia externa se recuperan mediante el nodo de extensión HTTP y se retransmiten a centro de IoT Edge a través del nodo receptor de mensajes de IoT Hub, donde el módulo lógico externo puede procesarlos más. Por ejemplo, si el servicio de inferencia es capaz de detectar vehículos, el módulo lógico podría buscar un vehículo específico, como un autobús o un camión. Cuando el módulo lógico detecta el objeto de interés, puede desencadenar el nodo del procesador de la puerta de señales mediante el envío de un evento a través del centro de IoT Edge al nodo de origen de mensajes de IoT Hub en el grafo. La salida de la puerta de señales se muestra para ir a un nodo receptor de archivos o a un nodo receptor de recursos. En el primer caso, el vídeo se grabará en el sistema de archivos local del dispositivo perimetral. En el segundo caso, el vídeo se grabará en un recurso.

Una mejora de este ejemplo es usar un procesador de detección de movimiento antes del nodo del procesador de filtros de velocidad de fotogramas. Esto reducirá la carga en el servicio de inferencia, como por la noche, cuando hay períodos de tiempo prolongados en los que no pasan vehículos por la autopista. 

## <a name="next-steps"></a>Pasos siguientes

[Tutorial: grabación de vídeo basada en eventos](event-based-video-recording-tutorial.md)
