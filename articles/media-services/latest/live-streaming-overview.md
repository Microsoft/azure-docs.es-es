---
title: Información general de streaming en vivo con Azure Media Services v3 | Microsoft Docs
description: En este artículo se proporciona información general de streaming en vivo con Azure Media Services v3.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 05/11/2019
ms.author: juliako
ms.openlocfilehash: fa09185e68c8d3a70562fe50c583ff872bf91e48
ms.sourcegitcommit: 9e8dfa1169a55c3c8af93a6c5f4e0dace4de48b2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65556224"
---
# <a name="live-streaming-with-azure-media-services-v3"></a>Streaming en vivo con Azure Media Services v3

Azure Media Services permite entregar eventos en directo a sus clientes en la nube de Azure. Para transmitir los eventos en directo con Media Services, necesita lo siguiente:  

- Una cámara que se utilice para capturar el evento en directo.<br/>Para obtener ideas para la configuración, consulte [Simple and portable event video gear setup]( https://link.medium.com/KNTtiN6IeT) (Equipo de vídeo para eventos sencillo y portátil).

    Si no tiene acceso a una cámara, herramientas como [Telestream Wirecast](https://www.telestream.net/wirecast/overview.htm) se puede usar generar una fuente directa de un archivo de vídeo.
- Un codificador de vídeo en vivo que convierte las señales de una cámara (u otro dispositivo, como un portátil) en una fuente de contribución que se envía a Media Services. La fuente de contribución puede incluir señales relacionadas con la publicidad, como los marcadores SCTE-35.<br/>Para obtener una lista de los codificadores de streaming en vivo recomendados, consulte [Codificadores de streaming en vivo](recommended-on-premises-live-encoders.md). Vea también este blog: [Live streaming production with OBS](https://link.medium.com/ttuwHpaJeT) (Producción de streaming en vivo con OBS).
- Componentes de Media Services, que permiten ingerir, previsualizar, empaquetar, registrar, cifrar y difundir el evento en directo a los clientes o a una red CDN para una futura distribución.

En este artículo ofrece información general y Guía de streaming en vivo con Media Services y vínculos a artículos relevantes.

> [!NOTE]
> Actualmente, no puede usar Azure Portal para administrar recursos de v3. Use la [API REST](https://aka.ms/ams-v3-rest-ref), la [CLI](https://aka.ms/ams-v3-cli-ref) o uno de los [SDK](media-services-apis-overview.md#sdks) admitidos.

## <a name="dynamic-packaging"></a>Empaquetado dinámico

Con Media Services, puede sacar partido de [empaquetado dinámico](dynamic-packaging-overview.md), lo que permite obtener una vista previa y difundir sus transmisiones en vivo en [formatos MPEG DASH, HLS y Smooth Streaming](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming) desde la fuente de contribución que se envía al servicio. Los espectadores pueden reproducir el streaming en vivo con cualquier reproductor compatible con HLS, DASH o Smooth Streaming. Puede utilizar [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) en las aplicaciones web o móviles para entregar la transmisión en cualquiera de estos protocolos.

## <a name="dynamic-encryption"></a>Cifrado dinámico

Cifrado dinámico permite cifrar dinámicamente el contenido en vivo o bajo demanda con AES-128 o cualquiera de los tres sistemas DRM (administración) de derechos digitales principales: Microsoft PlayReady, Google Widevine y Apple FairPlay. Media Services también proporciona un servicio para entregar claves AES y licencias de DMR (PlayReady, Widevine y FairPlay) a los clientes autorizados. Para obtener más información, consulte [cifrado dinámico](content-protection-overview.md).

## <a name="dynamic-manifest"></a>Manifiesto dinámico

El filtro dinámico se utiliza para controlar el número de pistas, formatos, velocidades de bits y ventanas de tiempo de presentación que se envían a los jugadores. Para obtener más información, consulte [filtros y manifiestos dinámicos](filters-dynamic-manifest-overview.md).

## <a name="live-event-types"></a>Tipos de objetos LiveEvent

Un evento en directo puede ser uno de los dos tipos: codificación en directo y paso a través. Para obtener más información sobre el streaming en vivo en Media Services v3, vea [eventos en vivo y salidas de Live](live-events-outputs-concept.md).

### <a name="pass-through"></a>Paso a través

![paso a través](./media/live-streaming/pass-through.svg)

Cuando se utiliza el **objeto LiveEvent** de paso a través, se confía en el codificador en directo local para generar una secuencia de vídeo con múltiples velocidades de bits y enviarla como fuente de contribución al objeto LiveEvent (mediante el protocolo RTMP o MP4 fragmentado). El objeto LiveEvent lleva a cabo las secuencias de vídeo entrantes sin ningún otro procesamiento. Tales un acceso directo evento en directo está optimizado para los eventos en directo de ejecución prolongada o 24 x 365 lineal streaming en vivo. 

### <a name="live-encoding"></a>Live Encoding  

![codificación en directo](./media/live-streaming/live-encoding.svg)

Si utiliza la codificación en directo con Media Services, deberá configurar el codificador en directo local para que envíe un vídeo con una única velocidad de bits como fuente de contribución al objeto LiveEvent (mediante el protocolo RTMP o Mp4 fragmentado). El objeto LiveEvent codifica ese flujo entrante de una velocidad de bits única en un [flujo de vídeo con varias velocidades de bits](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming) y hace que esté disponible para que pueda reproducirse en dispositivos mediante protocolos como MPEG-DASH, HLS y Smooth Streaming. 

## <a name="live-streaming-workflow"></a>Flujo de trabajo de streaming en vivo

Para entender el flujo de trabajo de streaming en vivo en Media Services v3, tiene que primero revise y entender los conceptos siguientes: 

- [Puntos de conexión de API de transmisión por secuencias](streaming-endpoint-concept.md)
- [Eventos y API de salidas en vivo de Live](live-events-outputs-concept.md)
- [API de los localizadores de streaming](streaming-locators-concept.md)

### <a name="general-steps"></a>Pasos generales

1. En la cuenta de Media Services, asegúrese de que el **Streamingendpoint** (origen) se está ejecutando. 
2. Cree un [evento en directo](live-events-outputs-concept.md). <br/>Al crear el evento, puede especificar que se inicie automáticamente. De lo contrario, puede iniciar el evento cuando esté listo para iniciar el streaming.<br/> Cuando el inicio automático está establecido en true, el evento en directo se iniciará después de la creación. La facturación comienza en cuanto el objeto LiveEvent empieza a ejecutarse. Debe llamar explícitamente a Stop en el recurso del objeto LiveEvent para evitar que continúe la facturación. Para más información, consulte [Estados y facturación de LiveEvent](live-event-states-billing.md).
3. Obtenga las direcciones URL de ingesta y configure el codificador en el entorno local para usar la dirección URL para enviar la fuente de contribución.<br/>Consulte [Codificadores de streaming en vivo recomendados](recommended-on-premises-live-encoders.md).
4. Obtenga la dirección URL de versión preliminar y úsela para verificar que la entrada del codificador se está recibiendo realmente.
5. Cree un nuevo objeto de **recurso**.
6. Cree un objeto de **salida en directo** y use el nombre del recurso que ha creado.<br/>El objeto **LiveOutput** archivará la secuencia en el objeto **Asset**.
7. Cree un objeto **Streaming Locator** con los tipos del objeto **Streaming Policy** integrados.<br/>Si va a cifrar el contenido, consulte [Introducción a la protección de contenido](content-protection-overview.md).
8. Enumere las rutas de acceso en el **localizador de streaming** para recuperar las direcciones URL que se van a usar (estas son deterministas).
9. Obtener el nombre de host para el **Streamingendpoint** (origen) que desea transmitir desde.
10. Combine la dirección URL del paso 8 con el nombre de host del paso 9 para obtener la dirección URL completa.
11. Si desea que **LiveEvent** deje de estar visible, es preciso que deje de transmitir en secuencias el evento y que elimine el objeto **StreamingLocator**.

## <a name="other-important-articles"></a>Otros artículos importantes

- [Recommended live encoders](recommended-on-premises-live-encoders.md) (Codificadores en directo recomendados)
- [Uso de un DVR en la nube](live-event-cloud-dvr.md)
- [Comparación de tipos de objetos LiveEvent](live-event-types-comparison.md)
- [Estados y facturación](live-event-states-billing.md)
- [Latency](live-event-latency.md)

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial de Live Streaming](stream-live-tutorial-with-api.md)
* [Guía de migración para mover de Media Services v2 a v3](migrate-from-v2-to-v3.md)
