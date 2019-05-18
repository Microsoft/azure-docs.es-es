---
title: 'Conceptos de Azure Media Services: eventos en vivo y salidas en vivo | Microsoft Docs'
description: En este artículo se proporciona información general de conceptos de streaming en vivo con Azure Media Services v3.
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
ms.openlocfilehash: c025a4c6e2a5a06e12e25ce226a327b099b95306
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65550961"
---
# <a name="live-events-and-live-outputs"></a>Eventos en directo y salidas en vivo

Azure Media Services permite entregar eventos en directo a sus clientes en la nube de Azure. Para configurar los eventos de streaming en vivo en Media Services v3, deberá comprender los conceptos tratados en este artículo.

> [!TIP]
> Para los clientes que migran desde las API de Media Services v2, la **evento en directo** entidad reemplaza **canal** en v2 y **Live salida** reemplaza **programa**.


## <a name="live-events"></a>Eventos en directo

Los objetos [LiveEvents](https://docs.microsoft.com/rest/api/media/liveevents) son responsables de la ingesta y el procesamiento de las fuentes de vídeo en directo. Al crear un objeto LiveEvent, se crea un punto de conexión de entrada que puede utilizar para enviar una señal en directo desde un codificador remoto. El codificador en directo remoto envía la fuente de contribución a ese punto de conexión de entrada mediante el protocolo [RTMP](https://www.adobe.com/devnet/rtmp.html) o [Smooth Streaming](https://msdn.microsoft.com/library/ff469518.aspx) (MP4 fragmentado). Para el protocolo de introducción Smooth Streaming, los esquemas de dirección URL admitidos son `http://` o `https://`. Para el protocolo de introducción RTMP, los esquemas de dirección URL admitidos son `rtmp://` o `rtmps://`. 

## <a name="live-event-types"></a>Tipos de objetos LiveEvent

Un [objeto LiveEvent](https://docs.microsoft.com/rest/api/media/liveevents) puede ser de uno de estos dos tipos: paso a través y codificación en directo. 

### <a name="pass-through"></a>Paso a través

![paso a través](./media/live-streaming/pass-through.svg)

Cuando se utiliza el **objeto LiveEvent** de paso a través, se confía en el codificador en directo local para generar una secuencia de vídeo con múltiples velocidades de bits y enviarla como fuente de contribución al objeto LiveEvent (mediante el protocolo RTMP o MP4 fragmentado). El objeto LiveEvent lleva a cabo las secuencias de vídeo entrantes sin ningún otro procesamiento. Este tipo de objeto LiveEvent está optimizado para eventos en vivo de larga duración o para el streaming en vivo ininterrumpidamente. Al crear este tipo de objeto LiveEvent, especifique None (LiveEventEncodingType.None).

Puede enviar la fuente de contribución a resoluciones de hasta 4K y a una velocidad de fotogramas de 60 fotogramas/segundo, con códecs de vídeo H.264/AVC o H.265/HEVC y códecs de audio AAC (AAC-LC, HE-AACv1 o HE-AACv2).  Consulte el artículo [Comparación de tipos de objetos LiveEvent](live-event-types-comparison.md) para obtener más detalles.

> [!NOTE]
> El uso de un método de paso a través es la forma más económica de streaming en vivo cuando está realizando varios eventos en un largo período y ya ha invertido en codificadores locales. Consulte los detalles en [Precios de Servicios multimedia](https://azure.microsoft.com/pricing/details/media-services/) .
> 

Consulte un ejemplo de código .NET en [MediaV3LiveApp](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/Live/MediaV3LiveApp/Program.cs#L126).

### <a name="live-encoding"></a>Live Encoding  

![codificación en directo](./media/live-streaming/live-encoding.svg)

Si utiliza la codificación en directo con Media Services, deberá configurar el codificador en directo local para que envíe un vídeo con una única velocidad de bits como fuente de contribución al objeto LiveEvent (mediante el protocolo RTMP o Mp4 fragmentado). El objeto LiveEvent codifica ese flujo entrante de una velocidad de bits única en un [flujo de vídeo con varias velocidades de bits](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming) y hace que esté disponible para que pueda reproducirse en dispositivos mediante protocolos como MPEG-DASH, HLS y Smooth Streaming. Al crear este tipo de objeto LiveEvent, especifique el tipo de codificación como **Estándar** (LiveEventEncodingType.Standard).

Puede enviar la fuente de contribución a una resolución de hasta 1080p a una velocidad de fotogramas de 30 fotogramas/segundo, con códec de vídeo H.264/AVC y códec de audio AAC (AAC-LC, HE-AACv1 o HE-AACv2). Consulte el artículo [Comparación de tipos de objetos LiveEvent](live-event-types-comparison.md) para obtener más detalles.

Cuando se usa la codificación en directo (el evento LiveEvent se establece en **Standard**), la codificación preestablecida define cómo se codifica la transmisión entrante en varias velocidades de bits o capas. Para más información, consulte [Valores preestablecidos del sistema](live-event-types-comparison.md#system-presets).

> [!NOTE]
> Actualmente, el único valor preestablecido permitido para el tipo de evento en directo estándar es *Default720p*. Si necesita usar un valor preestablecido personalizado para la codificación en directo, póngase en contacto con amshelp@microsoft.com. Debe especificar la tabla de resoluciones y velocidades de bits deseadas. Compruebe que hay solo una capa a 720p y, como máximo, 6 capas.

## <a name="live-event-creation-options"></a>Opciones de creación de objetos LiveEvent

Al crear un objeto LiveEvent, puede especificar las siguientes opciones:

* El protocolo de streaming para el objeto LiveEvent (actualmente, se admiten los protocolos RTMP y Smooth Streaming).<br/>No se puede cambiar la opción de protocolo mientras estén en ejecución el objeto LiveEvent o sus objetos LiveOutput asociados. Si necesita diferentes protocolos, debe crear un objeto LiveEvent independiente para cada protocolo de streaming.  
* Restricciones de IP en la ingesta y vista previa. Puede definir las direcciones IP a las que se permite ingerir un vídeo en este objeto LiveEvent. Las direcciones IP permitidas se pueden especificar como una dirección IP única (por ejemplo, 10.0.0.1), un intervalo IP que usa una dirección IP y una máscara de subred CIDR (por ejemplo, 10.0.0.1/22) o un intervalo de IP que usa una máscara de subred decimal con puntos; por ejemplo, 10.0.0.1(255.255.252.0).<br/>Si no se especifica ninguna dirección IP y no hay ninguna definición de regla, no se permitirá ninguna dirección IP. Para permitir las direcciones IP, cree una regla y establezca 0.0.0.0/0.<br/>Las direcciones IP deben estar en uno de los siguientes formatos: dirección IpV4 con 4 números, intervalo de direcciones CIDR.
* Al crear el evento, puede especificar que se inicie automáticamente. <br/>Cuando el inicio automático está establecido en true, el evento en directo se inicia después de la creación. La facturación comienza en cuanto el objeto LiveEvent empieza a ejecutarse. Debe llamar explícitamente a Stop en el recurso del objeto LiveEvent para evitar que continúe la facturación. De lo contrario, puede iniciar el evento cuando esté listo para iniciar el streaming. 

    Para más información, consulte [Estados y facturación de LiveEvent](live-event-states-billing.md).

## <a name="live-event-ingest-urls"></a>Direcciones URL de ingesta de objetos LiveEvent

Una vez que se crea el objeto LiveEvent, puede ingerir las direcciones URL que proporcionará al codificador en directo. El codificador en directo usa estas direcciones URL para introducir una secuencia en vivo. Para más información, consulte [Recommended live streaming encoders](recommended-on-premises-live-encoders.md) (Codificadores de streaming en vivo recomendados). 

Puede usar direcciones URL que sean mnemónicas o no mnemónicas. 

* Dirección URL no mnemónica

    Este es el modo predeterminado en AMS v3. Posiblemente, puede obtener el objeto LiveEvent rápidamente, pero solo conocerá la dirección URL de ingesta cuando se haya iniciado el evento en directo. La dirección URL cambiará si detiene o inicia el objeto LiveEvent. <br/>El modo no mnemónico es útil en aquellos casos en los que un usuario final quiere hacer streaming mediante una aplicación y esta quiere obtener un objeto LiveEvent lo antes posible, y en los que tener una dirección URL de ingesta dinámica no es un problema.
* Dirección URL mnemónica

    El modo mnemónico lo prefieren los grandes operadores de difusión multimedia que usan codificadores de difusión y no quieren volver a configurarlos cuando empiezan un objeto LiveEvent. Prefieren una dirección URL de ingesta predictiva que no cambie con el tiempo.

> [!NOTE] 
> Para que una dirección URL de ingesta sea predictiva, deberá usar el modo "mnemónico" y pasar su propio token de acceso (para evitar un token aleatorio en la dirección URL).

### <a name="live-ingest-url-naming-rules"></a>Reglas de nomenclatura de direcciones URL de ingesta en directo

La cadena *random* que aparece a continuación es un número hexadecimal de 128 bits (que se compone de 32 caracteres del 0 al 9 y de la "a" a la "f").<br/>
El siguiente *token de acceso* es el que debe especificar para una dirección URL fija. También es un número hexadecimal de 128 bits.

#### <a name="non-vanity-url"></a>Dirección URL no mnemónica

##### <a name="rtmp"></a>RTMP

`rtmp://<random 128bit hex string>.channel.media.azure.net:1935/<access token>`
`rtmp://<random 128bit hex string>.channel.media.azure.net:1936/<access token>`
`rtmps://<random 128bit hex string>.channel.media.azure.net:2935/<access token>`
`rtmps://<random 128bit hex string>.channel.media.azure.net:2936/<access token>`

##### <a name="smooth-streaming"></a>Smooth Streaming

`http://<random 128bit hex string>.channel.media.azure.net/<access token>/ingest.isml`
`https://<random 128bit hex string>.channel.media.azure.net/<access token>/ingest.isml`

#### <a name="vanity-url"></a>Dirección URL mnemónica

##### <a name="rtmp"></a>RTMP

`rtmp://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:1935/<access token>`
`rtmp://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:1936/<access token>`
`rtmps://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:2935/<access token>`
`rtmps://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:2936/<access token>`

##### <a name="smooth-streaming"></a>Smooth Streaming

`http://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net/<access token>/ingest.isml`
`https://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net/<access token>/ingest.isml`

## <a name="live-event-preview-url"></a>Dirección URL de vista previa de objeto LiveEvent

Cuando el **objeto LiveEvent** comienza a recibir la fuente de contribución, puede utilizar el punto de conexión de vista previa para obtener una vista previa y validar que está recibiendo el streaming en vivo antes de seguir publicándola. Una vez que haya comprobado que la secuencia de vista previa es buena, puede utilizar el objeto LiveEvent para que el streaming en vivo esté disponible para su entrega mediante uno o más **puntos de conexión de streaming** (creados previamente). Para ello, cree un nuevo [objeto LiveOutput](https://docs.microsoft.com/rest/api/media/liveoutputs) en el **objeto LiveEvent**. 

> [!IMPORTANT]
> Asegúrese de que el vídeo fluye a la dirección URL de vista previa antes de continuar.

## <a name="live-event-long-running-operations"></a>Operaciones de larga ejecución eventos en directo

Para obtener más información, consulte [operaciones de larga ejecución](media-services-apis-overview.md#long-running-operations)

## <a name="live-outputs"></a>Salidas en vivo

Una vez que la secuencia fluye en el objeto LiveEvent, puede comenzar el evento de streaming mediante la creación de un [recurso](https://docs.microsoft.com/rest/api/media/assets), un [objeto LiveOutput](https://docs.microsoft.com/rest/api/media/liveoutputs) y un [objeto StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators). El objeto LiveOutput archivará la secuencia y la pondrá a disposición de los usuarios a través del [punto de conexión de streaming](https://docs.microsoft.com/rest/api/media/streamingendpoints).  

> [!NOTE]
> Los objetos LiveOutput comienzan al crearlos y se detienen cuando se eliminan. Cuando se elimina el objeto LiveOutput, no se elimina el recurso subyacente ni su contenido. 

La relación entre un objeto **LiveEvent** y su objeto **LiveOutput** es similar a la difusión de televisión tradicional, en la que un canal (objeto **LiveEvent**) representa un flujo constante de vídeo y una grabación (objeto **LiveOutput**) tiene un ámbito que está limitado a un segmento de tiempo específico (por ejemplo, las noticias de la tarde, que se emiten de 18:30 a 19:00). Puede grabar un programa de televisión utilizando una grabadora de vídeo digital (DVR). La característica equivalente de los eventos en directo se administra con la propiedad **ArchiveWindowLength**. Se trata de una duración de timespan ISO 8601 (por ejemplo, PTHH:MM:SS), que especifica la capacidad de la DVR y se puede establecer desde un mínimo de 3 minutos hasta un máximo de 25 horas.

El objeto **LiveOutput** es como una grabadora que capta y graba el streaming en vivo en un recurso de su cuenta de Media Services. El contenido grabado persistirá en la cuenta de Azure Storage adjunta a su cuenta, en el contenedor definido por el recurso. El objeto **LiveOuput** también le permite controlar las propiedades de la transmisión saliente en vivo, como qué parte de la transmisión se conserva en la grabación del archivo (por ejemplo, la capacidad de la DVR en la nube) y si los usuarios pueden empezar a ver la transmisión en vivo o no. El archivo en disco es una "ventana" circular de archivo que solo contiene la cantidad de contenido que se especifica en la propiedad **archiveWindowLength** del objeto **LiveOutput**. El contenido que está fuera de esta ventana se descarta automáticamente del contenedor de almacenamiento y no se puede recuperar. Puede crear varios objetos **LiveOutput** (hasta un máximo de tres) en un objeto **LiveEvent** con diferentes longitudes y configuraciones de archivo.  

Si ha publicado el **recurso** del objeto **LiveOutput** mediante un objeto **StreamingLocator**, el objeto **LiveEvent** (hasta la longitud de la ventana de DVR) seguirá estando visible hasta la expiración o eliminación del objeto StreamingLocator, lo que ocurra primero.

Para más información, consulte [Uso de una DVR en la nube](live-event-cloud-dvr.md).

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

[Tutorial de Live Streaming](stream-live-tutorial-with-api.md)
