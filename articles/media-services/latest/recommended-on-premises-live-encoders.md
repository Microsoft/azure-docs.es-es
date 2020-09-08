---
title: Codificadores de streaming en vivo recomendados por Media Services en Azure | Microsoft Docs
description: Información sobre los codificadores locales de streaming en vivo recomendados por Media Services
services: media-services
keywords: encoding;encoders;media
author: johndeu
manager: johndeu
ms.author: johndeu
ms.date: 08/31/2020
ms.topic: conceptual
ms.service: media-services
ms.openlocfilehash: 3532032f8fd3ac6e673d3913fd13f7f83ae7759e
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89295366"
---
# <a name="verified-on-premises-live-streaming-encoders"></a>Codificadores de streaming en vivo locales comprobados

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En Azure Media Services, un [evento en directo](/rest/api/media/liveevents) (canal) representa una canalización para procesar contenido de streaming en vivo. El evento en directo recibe las secuencias de entrada en vivo de una de estas dos formas:

* Un codificador en directo local envía una secuencia RTMP o Smooth Streaming (MP4 fragmentado) de velocidad de bits múltiple al canal Evento en directo que no está habilitado para realizar la codificación en vivo con Media Services. Las secuencias recopiladas pasan a través de Eventos en directo sin más procesamiento. Este método se llama **paso a través**. Se recomienda que el codificador en directo envíe transmisiones de múltiples velocidades de bits en lugar de una transmisión de una sola velocidad de bits a un evento en directo de paso a través para permitir el streaming con velocidad de bits adaptable al cliente. 

    Si usa transmisiones de múltiples velocidades de bits para el evento en directo de paso a través, el tamaño del GOP de vídeo y los fragmentos de vídeo con diferentes velocidades de bits deben estar sincronizados para evitar un comportamiento inesperado en el lado de reproducción.

  > [!TIP]
  > El método de paso a través es la forma más económica de realizar un streaming en vivo.
 
* Un codificador en directo local envía una secuencia de velocidad de bits única al Evento en directo que está habilitado para realizar la codificación en vivo con Media Services, con uno de los siguientes formatos: RTMP o Smooth Streaming (MP4 fragmentado). Después, el Evento en directo codifica en vivo la secuencia entrante de velocidad de bits única en una secuencia de vídeo de velocidad de bits múltiple (adaptable).

En este artículo se abordan los codificadores de streaming en vivo locales que se han comprobado. La comprobación se realiza a través de la autoverificación del proveedor o la verificación del cliente. Microsoft Azure Media Services no realiza pruebas completas o rigurosas de cada codificador y no vuelve a comprobar continuamente las actualizaciones. Para obtener instrucciones sobre cómo comprobar su codificador en vivo local, vea [Comprobación del codificador local](become-on-premises-encoder-partner.md).

Para obtener información detallada sobre la codificación en vivo con Media Services, vea [Streaming en vivo con Azure Media Services v3](live-streaming-overview.md).

## <a name="encoder-requirements"></a>Requisitos del codificador

Los codificadores deben admitir TLS 1.2 al usar protocolos HTTPS o RTMPS.

## <a name="live-encoders-that-output-rtmp"></a>Codificadores en directo que generan una salida RTMP

Media Services recomienda usar uno de los siguientes codificadores en directo que tienen RTMP como salida. Los esquemas URL admitidos son `rtmp://` o `rtmps://`.

Al hacer el streaming mediante RTMP, compruebe la configuración del firewall o del proxy para confirmar que los puertos TCP salientes 1935 y 1936 están abiertos.<br/><br/>
Al hacer el streaming mediante RTMPS, compruebe la configuración del firewall o del proxy para confirmar que los puertos TCP salientes 2935 y 2936 están abiertos.

> [!NOTE]
> Los codificadores deben admitir TLS 1.2 al usar protocolos RTMPS.

- Adobe Flash Media Live Encoder 3.2
- [Blackmagic ATEM Mini y ATEM Mini PRO](https://www.blackmagicdesign.com/products/atemmini)
- [Cambria Live 4.3](https://www.capellasystems.net/products/cambria-live/)
- Elemental Live (versión 2.14.15 y posteriores)
- [Ffmpeg](https://www.ffmpeg.org)
- [GoPro](https://gopro.com/help/articles/block/getting-started-with-live-streaming) Hero 7 y Hero 8
- Haivision KB
- Haivision Makito X HEVC
- [Restream.io](https://restream.io/)
- OBS Studio
- [Streamlabs OBS](https://streamlabs.com/)
- [Switcher Studio (iOS)](https://www.switcherstudio.com/)
- Telestream Wirecast (versión 13.0.2 o posterior debido al requisito de TLS 1.2)
- Telestream Wirecast S (solo se admite RTMP. No se admite RTMPS debido a la falta de TLS 1.2 o versiones posteriores)
- Teradek Slice 756
- VMIX
- xStream

> [!WARNING]
> La lista anterior de codificadores es solo una lista de recomendaciones. Microsoft no prueba ni valida los codificadores de forma continua, y se pueden producir actualizaciones o cambios importantes en los proveedores de codificadores o en los proyectos de código abierto que podrían interrumpir la compatibilidad. 

## <a name="live-encoders-that-output-fragmented-mp4-smooth-streaming-ingest"></a>Codificadores en directo que generan una salida de MP4 fragmentado (ingesta de Smooth Streaming)

Media Services recomienda usar uno de los codificadores en directo siguientes que tienen Smooth Streaming de velocidad de bits múltiple (MP4 fragmentado) como salida. Los esquemas URL admitidos son `http://` o `https://`.

> [!NOTE]
> Los codificadores deben admitir TLS 1.2 al usar protocolos HTTPS.

- Ateme TITAN Live
- Cisco Digital Media Encoder 2200
- Elemental Live (versión 2.14.15 y posterior debido al requisito de TLS 1.2)
- Envivio 4Caster C4 Gen III 
- [Ffmpeg](https://www.ffmpeg.org)
- Imagine Communications Selenio MCP3
- Media Excel Hero Live y Hero 4K (UHD/HEVC)

> [!TIP]
>  Si está transmitiendo eventos en directo en varios idiomas (por ejemplo, una pista de audio en inglés y una en español), puede hacerlo con el codificador en directo de elementos multimedia de Excel configurado para enviar la fuente en directo a un evento en directo de paso a través.

> [!WARNING]
> La lista anterior de codificadores es solo una lista de recomendaciones. Microsoft no prueba ni valida los codificadores de forma continua, y se pueden producir problemas de soporte o errores en los proveedores de codificadores o en los proyectos de código abierto que podrían interrumpir la compatibilidad. 

## <a name="configuring-on-premises-live-encoder-settings"></a>Configuración de los valores del codificador en directo local

Para información acerca de qué valores son válidos para el tipo de evento en directo, consulte [Comparación de tipos de objetos LiveEvent](live-event-types-comparison.md).

### <a name="playback-requirements"></a>Requisitos de reproducción

Para volver a reproducir un contenido, debe haber una secuencia de audio y una secuencia de vídeo. No se admite la reproducción de la secuencia solo de vídeo.

### <a name="configuration-tips"></a>Sugerencias de configuración

- Siempre que sea posible, use una conexión a Internet por cable.
- Cuando tenga que determinar los requisitos de ancho de banda, duplique las velocidades de bits de streaming. Aunque no es obligatorio, esta sencilla regla ayuda a mitigar el impacto de la congestión de la red.
- Cuando se usen codificadores por software, cierre todos los programas innecesarios.
- Si se modifica la configuración del codificador una vez iniciada la inserción, se producirán efectos negativos en el evento. Los cambios de configuración pueden provocar que el evento se vuelva inestable. 
- Pruebe y valide siempre las versiones más recientes del software del codificador para mantener una compatibilidad continua con Azure Media Services. Microsoft no vuelve a validar los codificadores de esta lista y la mayoría de las validaciones las realizan los proveedores de software directamente como una "certificación automática".
- Asegúrese de dejar tiempo suficiente para configurar el evento. En el caso de los eventos a gran escala, se recomienda iniciar la configuración una hora antes del evento.
- Use el vídeo H.264 y la salida de códec de audio AAC-LC.
- Siga con las resoluciones y las velocidades de fotogramas admitidas para el tipo de evento en directo que se va a retransmitir (por ejemplo, 60fps se rechaza actualmente).
- Asegúrese de que haya una alineación temporal de fotograma clave o de GOP a lo largo de las calidades de vídeo.
- Asegúrese de que cada calidad de vídeo tiene un nombre de secuencia único.
- Use la codificación CBR estricta recomendada para obtener un rendimiento óptimo de velocidad de bits adaptable.

> [!IMPORTANT]
> Vigile la condición física de la máquina (CPU/memoria, etc.), ya que la carga de fragmentos en la nube conlleva operaciones de CPU y de E/S. Si cualquiera de las configuraciones del codificador cambia, asegúrese de restablecer el evento en directo/canales para que los cambios surten efecto.

## <a name="see-also"></a>Consulte también

[Streaming en vivo con Media Services v3](live-streaming-overview.md)

## <a name="next-steps"></a>Pasos siguientes

[Cómo comprobar su codificador](become-on-premises-encoder-partner.md)
