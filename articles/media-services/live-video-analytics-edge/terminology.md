---
title: 'Terminología de Live Video Analytics en IoT Edge: Azure'
description: En este artículo se proporciona información general sobre la terminología de Live Video Analytics on IoT Edge.
ms.topic: conceptual
ms.date: 05/30/2020
ms.openlocfilehash: e3a77b69adf2241a4af2652db4edb6673a63b4f0
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690619"
---
# <a name="terminology"></a>Terminología

En este artículo se proporciona información general sobre la terminología relacionada con [Live Video Analytics on IoT Edge](overview.md).

## <a name="azure-media-services"></a>Azure Media Services

Azure Media Services es una plataforma multimedia en la nube que permite compilar soluciones multimedia. Puede obtener más información al respecto en la documentación de [Azure Media Services](../latest/media-services-overview.md).

## <a name="asset"></a>Recurso

Un [recurso](../latest/assets-concept.md) es una entidad de Azure Media Services que se asigna a un contenedor de blobs en la cuenta de almacenamiento de Azure que está asociada a una cuenta de Media Services. Todos los archivos asociados a un recurso se almacenan como blobs en ese contenedor mientras Media Services contenga los metadatos (por ejemplo, un nombre, una descripción, un tiempo de creación) asociados al recurso.

Live Video Analytics on IoT Edge puede crear recursos o agregar datos a los recursos existentes. Esto permite los escenarios de grabación y reproducción de vídeo continuos y basados en eventos (con la captura de vídeo en el dispositivo perimetral, la grabación en Azure Media Services y la reproducción a través de las funcionalidades de streaming de Azure Media Services existentes).

## <a name="grpc"></a>gRPC

[gRPC](https://grpc.io/docs/guides/) es un marco de llamada a procedimiento remoto (RPC) de alto rendimiento e independiente del idioma. Usa esquemas estructurados basados en sesiones a través de [búferes de protocolo 3](https://developers.google.com/protocol-buffers/docs/proto3) como su formato de intercambio de mensajes subyacente para la comunicación.

## <a name="media-graph"></a>Grafo multimedia

Un [grafo multimedia](media-graph-concept.md) le permite definir dónde se debe capturar el elemento multimedia, cómo se debe procesar y dónde se deben entregar los resultados. Permite definir un grafo que consta de orígenes, procesadores y nodos receptores y, por tanto, proporciona la capacidad de crear aplicaciones de análisis de vídeo en vivo. Los grafos multimedia se describen en detalle en la página concepto de grafos multimedia.

## <a name="recording"></a>Grabación

En el contexto de un sistema de administración de vídeo para cámaras de seguridad, la grabación de vídeo es al proceso de capturar vídeo de las cámaras y almacenarlo en un archivo (o en varios) para verlo posteriormente mediante aplicaciones móviles y de explorador. La grabación de vídeo se puede clasificar en [grabación continua de vídeo](continuous-video-recording-concept.md) y [grabación de vídeo basada en eventos](event-based-video-recording-concept.md). Ambas categorías se explican con más detalle en la página sobre la [grabación de vídeo](video-recording-concept.md).

## <a name="rtsp"></a>RTSP

[RTSP](https://tools.ietf.org/html/rfc2326) hace referencia al Protocolo de streaming en tiempo real. Este es un protocolo de nivel de aplicación para controlar la entrega de datos con propiedades en tiempo real. RTSP proporciona un marco extensible para habilitar la entrega controlada a petición de datos en tiempo real, como audio y vídeo. 

## <a name="streaming"></a>Streaming

Si ha visto vídeo en un dispositivo móvil en servicios como Netflix o YouTube, entre otros, ha experimentado el streaming de vídeo. La reproducción comienza cuando se pulsa “reproducir” (si se dispone de ancho de banda suficiente) y se puede avanzar o retroceder en la escala de tiempo del vídeo. Con el streaming, la idea es ofrecer solo la parte del vídeo que se está viendo y dejar que el espectador comience a reproducirlo mientras los datos se están transfiriendo aún desde un servidor al cliente de reproducción. En el contexto de Azure Media Services, el [streaming](https://en.wikipedia.org/wiki/Streaming_media) es el proceso de entregar contenido multimedia de [Azure Media Services](../azure-media-player/azure-media-player-overview.md) a un cliente de streaming (por ejemplo, Azure Media Player). Puede usar Azure Media Services para transmitir por secuencias vídeo a los clientes mediante protocolos de streaming multimedia estándar del sector basados en HTTP, como [HTTP Live Streaming (HLS)](https://developer.apple.com/streaming/) y [MPEG-DASH](https://dashif.org/about/). HLS es compatible con Azure Media Player y reproductores web como [JW Player](https://www.jwplayer.com/), [hls.js](https://github.com/video-dev/hls.js/), [VideoJS](https://videojs.com/) y [Shaka Player, de Google](https://github.com/google/shaka-player), o bien se puede representar de forma nativa en aplicaciones móviles con [Exoplayer](https://github.com/google/ExoPlayer), de Android y [AV Foundation](https://developer.apple.com/av-foundation/), de iOS. MPEG-DASH también es compatible con Azure Media Player, [en esta página encontrará una lista de clientes](https://dashif.org/clients/). 

Mediante el uso de [grafos multimedia](#media-graph) para grabar vídeos en un recurso de Azure Media Services, puede hacer uso de la funcionalidad de streaming de Media Services streaming para ofrecer transmisiones de vídeo en HLS y DASH. Puede obtener más información al respecto en el artículo sobre la [reproducción de vídeo](video-playback-concept.md).

## <a name="vms"></a>VMS

[VMS](https://en.wikipedia.org/wiki/Video_management_system) hace referencia a un sistema de administración de vídeo. Estos sistemas se usan para configurar y controlar cámaras de vigilancia, así como para capturar y grabar vídeos de ellas. Estos sistemas también proporcionan aplicaciones cliente para reproducir el vídeo grabado

## <a name="next-steps"></a>Pasos siguientes

[Grafos multimedia](media-graph-concept.md)
