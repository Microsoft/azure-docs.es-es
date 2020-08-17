---
title: 'Reproducción de vídeo: Azure'
description: Marcador de posición
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 17b1f93c18dfb013916c4c0d3756fb97a73e2675
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87042975"
---
# <a name="video-playback"></a>Reproducción de vídeo 

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura 

* [Introducción a Live Video Analytics on IoT Edge](overview.md)
* [Terminología de Live Video Analytics on IoT Edge](terminology.md)
* [Concepto de grafo multimedia](media-graph-concept.md)

## <a name="overview"></a>Información general  

Puede usar [grafos multimedia](media-graph-concept.md) para grabar vídeo en un [recurso](terminology.md#asset) de Azure Media Services. En este documento, conocerá los pasos que debe seguir para reproducir un recurso con las funcionalidades de streaming existentes de Azure Media Services.

## <a name="streaming-endpoint"></a>extremo de streaming 

Puede usar Azure Media Services para [transmitir por secuencias](terminology.md#streaming) el recurso a reproductores de vídeo mediante protocolos de streaming multimedia estándar del sector basados en HTTP, como HTTP Live Streaming (HLS) y MPEG-DASH. Esta conversión de contenido multimedia grabado a formatos de streaming se administra mediante un [punto de conexión de streaming](../latest/streaming-endpoint-concept.md), que es un recurso que se debe aprovisionar en la cuenta de Azure Media Services.

## <a name="streaming-policy"></a>Directiva de streaming 

Azure Media Services ofrece distintos métodos para proteger las secuencias de vídeo, como se describe en el artículo [Protección del contenido mediante el cifrado dinámico de Media Services](../latest/content-protection-overview.md). En general, las opciones de protección del contenido son:

* **In-the-clear streaming** (Streaming sin cifrado): donde no se aplica ningún cifrado durante el streaming.
* **Use Estándar de cifrado avanzado (AES-128)** (Usar Estándar de cifrado avanzado [AES-128]): para implementar un método de entrega de las claves para descifrar el vídeo solo para espectadores autenticados.
* **Use Digital Rights Management (DRM) systems** (Usar sistemas de Administración de derechos digitales [DRM]): para controlar el uso, la modificación y la entrega de vídeo en dispositivos que aplican estas directivas.

Para lograr la protección del contenido, puede definir y crear una [directiva de streaming](../latest/streaming-policy-concept.md) en su cuenta de Media Services y usarla para transmitir por secuencias todos los recursos (suponiendo que todas las secuencias tengan los mismos requisitos de seguridad). También puede usar cualquiera de las directivas predefinidas (como Predefined_ClearStreamingOnly).

## <a name="streaming-locator"></a>Localizador de streaming  

Cuando haya iniciado un punto de conexión de streaming en su cuenta de Media Services y se haya definido la directiva de streaming, puede pasar a la transmisión del contenido multimedia grabado desde un recurso mediante los protocolos HLS o DASH. Los reproductores web y las aplicaciones móviles necesitan una dirección URL que apunte a esa secuencia HLS o DASH. Puede compilar esta dirección URL mediante el [localizador de streaming](../latest/streaming-locators-concept.md). Tal y como se describe en ese artículo y se muestra en el ejemplo [Creación de un localizador de streaming y compilación de direcciones URL](../latest/create-streaming-locator-build-url.md). la dirección URL de streaming se compone del punto de conexión de streaming, la directiva de streaming y el localizador de streaming.

## <a name="content-recorded-using-file-sink"></a>Contenido grabado mediante el receptor de archivos  

Tal y como se describe en [Receptor de archivos de grafo multimedia](media-graph-concept.md#file-sink), puede usar grafos multimedia para grabar vídeos en el sistema de archivos local del dispositivo perimetral mediante un receptor de archivos en el grafo multimedia. El receptor de archivos genera archivos [MP4](https://developer.mozilla.org/docs/Web/Media/Formats/Containers#MP4), y puede usar el elemento [&lt;video&gt;](https://developer.mozilla.org/docs/Web/HTML/Element/video) HTML5 para reproducir dicho contenido. 

## <a name="next-steps"></a>Pasos siguientes

[Azure IoT Edge](../../iot-edge/index.yml)
<!--
## Next steps

[Playback recording](playback-recording-how-to.md)
-->
