---
title: Preguntas más frecuentes sobre Análisis de vídeos en vivo en IoT Edge en Azure
description: En este tema se proporcionan respuestas a las preguntas más frecuentes sobre Análisis de vídeos en directo en IoT Edge.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 705c74b1ba6d3a28911185b82a558099e87dc18d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87011522"
---
# <a name="frequently-asked-questions-faqs"></a>Preguntas más frecuentes (P+F)

En este tema se proporcionan respuestas a las preguntas más frecuentes sobre Análisis de vídeos en directo en IoT Edge.

## <a name="general"></a>General

¿Qué variables del sistema se pueden usar en la definición de la topología de grafos?

|Variable   |Descripción|
|---|---|
|[System.DateTime](/dotnet/framework/data/adonet/sql/linq/system-datetime-methods)|Representa un instante de tiempo, normalmente expresado en forma de fecha y hora del día.|
|System.GraphTopologyName   |Representa una topología de grafos multimedia y contiene el plano técnico de un grafo.|
|System.GraphInstanceName|  Representa una instancia de grafo multimedia, contiene los valores de parámetro y hace referencia a la topología.|

## <a name="configuration-and-deployment"></a>Configuración e implementación

¿Se puede implementar el módulo multimedia de Edge en un dispositivo Windows 10?
    * Sí. Consulte el artículo sobre los [contenedores Linux en Windows 10](/virtualization/windowscontainers/deploy-containers/linux-containers).

## <a name="capture-from-ip-camera-and-rtsp-settings"></a>Captura con una cámara IP y configuración de RTSP

* ¿Es necesario usar un SDK especial en el dispositivo para enviarlo en una secuencia de vídeo?
    * No. Análisis de vídeos en vivo en IoT Edge permite capturar elementos multimedia mediante el protocolo de streaming de vídeo RTSP (que es compatible con la mayoría de las cámaras IP).
* ¿Se puede insertar contenido multimedia en Análisis de vídeos en vivo en IoT Edge mediante RTMP o Smooth (como un evento en directo de Media Services)?
    * No. LVA solo admite RTSP para capturar vídeo con las cámaras IP.
    * Cualquier cámara que admita el streaming con RTSP a través de TCP/HTTP debería funcionar. 
* ¿Se puede restablecer o actualizar la dirección URL de origen RTSP en una instancia de grafo?
    * Sí, cuando la instancia del grafo está en estado inactivo.  
* ¿Hay algún simulador RTSP disponible para usarlo durante las pruebas y el desarrollo?
    * Sí. Hay un módulo perimetral del [simulador RTSP](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) disponible para su uso en los tutoriales y artículos de inicio rápido para respaldar el proceso de aprendizaje. Este módulo se proporciona siempre que sea posible y puede que no siempre esté disponible. Se recomienda no usar esto durante más de unas horas. Debe invertir en las pruebas con el origen RTSP real antes de planear una implementación en producción.
* ¿Admite la detección de ONVIF de cámaras IP en el perímetro?
    * No, no se admite la detección de ONVIF de dispositivos en el perímetro.

## <a name="streaming-and-playback"></a>Streaming y reproducción

* ¿Pueden reproducirse los recursos registrados en AMS desde el perímetro con las tecnologías de streaming de Media Services, como HLS o DASH?
    * Sí. Los recursos grabados se pueden transmitir como cualquier otro recurso de Azure Media Services. Para transmitir el contenido, debe existir un punto de conexión de streaming en ejecución. El uso del proceso de creación del localizador de streaming estándar dará acceso a un manifiesto de HLS o DASH para el streaming a cualquier marco de trabajo para reproductores compatible. Para obtener información detallada sobre la creación de manifiestos de publicación de HLS o DASH, vea [Empaquetado dinámico](../latest/dynamic-packaging-overview.md).
* ¿Se pueden usar las características de DRM y protección de contenido estándar de Media Services en un recurso archivado?
    * Sí. Todas las características de DRM y protección de contenido de cifrado dinámico están disponibles para usarlas en los recursos grabados desde un grafo multimedia.
* ¿Qué reproductores se pueden usar para ver el contenido de los recursos grabados?
   * Se admiten todos los reproductores estándar compatibles con Apple HTTP Live Streaming (HLS) versión 3 o versión 4. Además, también se admite cualquier reproductor que sea compatible con la reproducción de MPEG-DASH.
    Los reproductores recomendados para las pruebas incluyen:

    * [Azure Media Player](../latest/use-azure-media-player.md)
    * [HLS.js](https://hls-js.netlify.app/demo/)
    * [Video.js](https://videojs.com/)
    * [Dash.js](https://github.com/Dash-Industry-Forum/dash.js/wiki)
    * [Shaka Player](https://github.com/google/shaka-player)
    * [ExoPlayer](https://github.com/google/ExoPlayer)
    * [HTTP Live Streaming nativo de Apple](https://developer.apple.com/streaming/)
    * Reproductor de vídeo HTML5 integrado en Edge, Chrome o Safari
    * Reproductores comerciales compatibles con la reproducción HLS o DASH
* ¿Cuáles son los límites del streaming de un recurso de grafo multimedia?
    * El streaming de un recurso grabado o en directo desde un grafo multimedia usa la misma infraestructura de gran escala y el punto de conexión de streaming que Media Services admite para el streaming a petición y en directo para clientes de medios y entretenimiento, OTT y difusión. Esto significa que se puede habilitar rápida y fácilmente Azure CDN, Verizon o Akamai para ofrecer el contenido a una audiencia tan pequeña como algunos espectadores, o hasta millones, en función de su escenario.

    El contenido se puede entregar mediante Apple HTTP Live Streaming (HLS) o MPEG-DASH.

## <a name="monitoring-and-metrics"></a>Supervisión y métricas

* ¿Se puede supervisar el grafo multimedia en el perímetro mediante Event Grid?
    * No. Actualmente no se admite Event Grid.
* ¿Se puede usar Azure Monitor para ver el estado, las métricas y el rendimiento de los grafos multimedia en la nube o en el perímetro?
    * No.
* ¿Hay alguna herramienta que facilite la supervisión del módulo de IoT Edge de Media Services?
    * Visual Studio Code admite la extensión "Azure IoT Tools" que permite supervisar fácilmente los puntos de conexión del módulo LVAEdge. Puede usar esta herramienta para empezar a supervisar rápidamente el punto de conexión integrado de IoT Hub para "eventos" y ver los mensajes de inferencia que se enrutan desde el dispositivo perimetral hasta la nube. 

    Además, puede usar esta extensión para editar el módulo gemelo para el módulo LVAEdge con el fin de modificar la configuración del grafo multimedia.

Para más información, vea el artículo [Supervisión y registro](monitoring-logging.md).

## <a name="billing-and-availability"></a>Facturación y disponibilidad

* ¿Cómo se factura Análisis de vídeos en vivo en IoT Edge?
    * Vea la [página de precios](https://azure.microsoft.com/pricing/details/media-services/) para obtener más detalles.

## <a name="next-steps"></a>Pasos siguientes

[Inicio rápido: Introducción: Live Video Analytics on IoT Edge](get-started-detect-motion-emit-events-quickstart.md)
