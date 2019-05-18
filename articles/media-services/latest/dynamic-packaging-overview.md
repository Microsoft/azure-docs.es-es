---
title: Introducción al empaquetado dinámico de Azure Media Services | Microsoft Docs
description: El tema proporciona información general sobre el empaquetado dinámico en Media Services.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/27/2019
ms.author: juliako
ms.openlocfilehash: 78e3897ec653326bcd88a538a6ea7d33938659b9
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65761944"
---
# <a name="dynamic-packaging"></a>Empaquetado dinámico

Microsoft Azure Media Services puede usarse para proporcionar varios formatos de archivo de origen multimedia, formatos de streaming multimedia y formatos de protección de contenido a diversas tecnologías cliente (por ejemplo, iOS y XBOX). Estos clientes entienden distintos protocolos. Por ejemplo, iOS requiere un formato HTTP Live Streaming (HLS) y Xbox requiere Smooth Streaming. Si tiene un conjunto de velocidad de bits adaptable (velocidad de bits múltiple) MP4 archivos (ISO Base Media 14496-12) o un conjunto de archivos Smooth Streaming de velocidad de bits adaptable que desea prestar servicios a clientes que entienden HLS, MPEG DASH o Smooth Streaming, puede sacar partido de  **Empaquetado dinámico**. Se admiten SD y HD/UHD - 4K, el empaquetado es independiente de la resolución del vídeo.

En Media Services, un [Streamingendpoint](streaming-endpoint-concept.md) representa un dinámico (just-in-time) empaquetado y el origen de servicio que puede entregar el contenido en directo y a petición directamente a una aplicación de Reproductor de cliente, mediante uno de los comunes de streaming protocolos de multimedia (HLS o DASH). Empaquetado dinámico es una característica que viene incluida en todos los **extremos de Streaming** (Standard o Premium). 

Para aprovechar las ventajas de **empaquetado dinámico**, debe tener un **activos** con un conjunto de archivos MP4 de velocidad de bits adaptable y streaming de los archivos de configuración necesarios para el empaquetado dinámico de Media Services. Una manera de obtener los archivos consiste en codificar su archivo intermedio (origen) con Media Services. Para que los vídeos en el recurso codificado esté disponible para los clientes para la reproducción, tiene que crear un **localizador de Streaming** y generar direcciones URL de streaming. A continuación, según el formato especificado en el manifiesto del cliente de streaming (HLS, DASH o Smooth), reciba la secuencia en el protocolo elegido.

Como resultado, solo tendrá que almacenar y pagar los archivos en formato de almacenamiento único y Media Services creará y proporcionará la respuesta adecuada en función de las solicitudes de un cliente. 

En Media Services, el empaquetado dinámico se utiliza si la transmisión por secuencias en directo o a petición. 

## <a name="common-on-demand-workflow"></a>Flujo de trabajo comunes y a petición

El siguiente es un flujo de trabajo de streaming donde se utiliza el empaquetado dinámico común de Media Services.

1. Cargar un archivo de entrada (llamado archivo intermedio). Por ejemplo, MP4, MOV o MXF (para obtener la lista de formatos admitidos, consulte [formatos compatibles con Media Encoder Standard](media-encoder-standard-formats.md).
2. Codificar el archivo intermedio en conjuntos MP4 de velocidad de bits adaptable H.264.
3. Publicación del recurso que contiene el conjunto de MP4 de velocidad de bits adaptable. Publicar mediante la creación de un **localizador de Streaming**.
4. Generar direcciones URL que tienen como destino diferentes formatos (HLS, Dash y Smooth Streaming). El **Streamingendpoint** se ocupa de servir el manifiesto correcto y las solicitudes de todos estos formatos diferentes.

El siguiente diagrama muestra el streaming a petición con el flujo de trabajo de empaquetado dinámico.

![Empaquetado dinámico](./media/dynamic-packaging-overview/media-services-dynamic-packaging.png)

### <a name="encode-to-adaptive-bitrate-mp4s"></a>Codificación de velocidad de bits adaptativa MP4s

Para obtener información acerca de [cómo codificar un vídeo con Media Services](encoding-concept.md), vea los ejemplos siguientes:

* [Codificar desde una dirección URL de HTTPS con los valores preestablecidos integrados](job-input-from-http-how-to.md)
* [Codificar un archivo local con los valores preestablecidos integrados](job-input-from-local-file-how-to.md)
* [Crear un valor predeterminado para sus requisitos específicos de escenario o dispositivo de destino personalizado](customize-encoder-presets-how-to.md)

Para obtener una lista de códecs y formatos de Media Encoder Standard, consulte [códecs y formatos](media-encoder-standard-formats.md)

## <a name="common-live-streaming-workflow"></a>Flujo de trabajo de transmisión por secuencias en directo común

Estos son los pasos para un flujo de trabajo de streaming en vivo:

1. Cree un [evento en directo](live-events-outputs-concept.md).
1. Obtenga las direcciones URL de ingesta y configure el codificador en el entorno local para usar la dirección URL para enviar la fuente de contribución.
1. Obtenga la dirección URL de versión preliminar y úsela para verificar que la entrada del codificador se está recibiendo realmente.
1. Cree un nuevo **activos**.
1. Cree un objeto de **salida en directo** y use el nombre del recurso que ha creado.<br/>El objeto **LiveOutput** archivará la secuencia en el objeto **Asset**.
1. Cree un objeto **Streaming Locator** con los tipos del objeto **Streaming Policy** integrados.<br/>Si va a cifrar el contenido, consulte [Introducción a la protección de contenido](content-protection-overview.md).
1. Enumere las rutas de acceso en el **localizador de streaming** para recuperar las direcciones URL que se van a usar.
1. Obtenga el nombre de host para el **punto de conexión de streaming** desde el que quiere hacer el streaming.
1. Generar direcciones URL que tienen como destino diferentes formatos (HLS, Dash y Smooth Streaming). El **Streamingendpoint** se ocupa de servir el manifiesto correcto y las solicitudes de todos estos formatos diferentes.

Un evento en directo puede ser uno de los dos tipos: codificación en directo y paso a través. Para obtener más información sobre el streaming en vivo en Media Services v3, vea [información general de streaming en vivo](live-streaming-overview.md).

El siguiente diagrama muestra la transmisión por secuencias en directo con el flujo de trabajo de empaquetado dinámico.

![paso a través](./media/live-streaming/pass-through.svg)

## <a name="delivery-protocols"></a>Protocolos de entrega

|Protocol|Ejemplo|
|---|---|
|HLS V4 |`https://amsv3account-usw22.streaming.media.azure.net/21b17732-0112-4d76-b526-763dcd843449/ignite.ism/manifest(format=m3u8-aapl)`|
|HLS V3 |`https://amsv3account-usw22.streaming.media.azure.net/21b17732-0112-4d76-b526-763dcd843449/ignite.ism/manifest(format=m3u8-aapl-v3)`|
|HLS CMAF| `https://amsv3account-usw22.streaming.media.azure.net/21b17732-0112-4d76-b526-763dcd843449/ignite.ism/manifest(format=m3u8-cmaf)`|
|MPEG DASH CSF| `https://amsv3account-usw22.streaming.media.azure.net/21b17732-0112-4d76-b526-763dcd843449/ignite.ism/manifest(format=mpd-time-csf)` |
|MPEG DASH CMAF|`https://amsv3account-usw22.streaming.media.azure.net/21b17732-0112-4d76-b526-763dcd843449/ignite.ism/manifest(format=mpd-time-cmaf)` |
|Smooth Streaming| `https://amsv3account-usw22.streaming.media.azure.net/21b17732-0112-4d76-b526-763dcd843449/ignite.ism/manifest`|

## <a name="video-codecs-supported-by-dynamic-packaging"></a>Códecs de vídeo compatibles con el empaquetado dinámico

Empaquetado dinámico admite archivos MP4, que contienen vídeo codificado con [H.264](https://en.m.wikipedia.org/wiki/H.264/MPEG-4_AVC) (MPEG-4 AVC o AVC1), [H.265](https://en.m.wikipedia.org/wiki/High_Efficiency_Video_Coding) (HEVC, hev1 o hvc1).

## <a name="audio-codecs-supported-by-dynamic-packaging"></a>Códecs de audio compatibles con el empaquetado dinámico

Empaquetado dinámico admite archivos MP4 con audio codificado con [AAC](https://en.wikipedia.org/wiki/Advanced_Audio_Coding) (AAC-LC, HE-AAC v1, HE-AAC v2), [Dolby Digital Plus](https://en.wikipedia.org/wiki/Dolby_Digital_Plus)(Enhanced AC-3 o E-AC3), Dolby Atmos, o [DTS](https://en.wikipedia.org/wiki/DTS_%28sound_system%29) (DTS Express, LBR DTS, DTS HD, HD DTS sin pérdida de datos). Streaming de contenido Dolby Atmos se admite para estándares como MP4 fragmentado de protocolo de MPEG-DASH con formato de Streaming comunes (CSF) o formato de aplicación comunes de medios (CMAF) y, a través de HTTP Live Streaming (HLS) con CMAF.

> [!NOTE]
> El empaquetado dinámico no admite archivos que contienen audio [Dolby Digital](https://en.wikipedia.org/wiki/Dolby_Digital) (AC3) (es un códec heredado).

## <a name="dynamic-encryption"></a>Cifrado dinámico

**Cifrado dinámico** permite cifrar dinámicamente el contenido en vivo o bajo demanda con AES-128 o cualquiera de los sistemas de administración (DRM) de tres derechos digitales principales: Microsoft PlayReady, Google Widevine y Apple FairPlay. Media Services también proporciona un servicio para entregar claves AES y licencias de DMR (PlayReady, Widevine y FairPlay) a los clientes autorizados. Para obtener más información, consulte [cifrado dinámico](content-protection-overview.md).

## <a name="manifests"></a>Manifiestos 
 
Media Services admite HLS, MPEG DASH, protocolos de transmisión por secuencias suave. Como parte de **empaquetado dinámico**, los manifiestos de cliente de streaming (HLS Master Playlist, descripción guión de presentación multimedia (MPD) y Smooth Streaming) se generarán dinámicamente basándose en el selector de formato en la dirección URL. Consulte los protocolos de entrega en [en esta sección](#delivery-protocols). 

Un archivo de manifiesto incluye como metadatos de streaming: realizar un seguimiento de tipo (audio, vídeo o texto), realizar un seguimiento del nombre, hora de inicio y finalización, la velocidad de bits (calidades), idiomas de pista, la ventana de presentación (ventana deslizante de duración fija), el códec de vídeo (FourCC). También indica al reproductor que recupere el siguiente fragmento ofreciendo información sobre los próximos fragmentos de vídeo reproducibles disponibles y su ubicación. Los fragmentos (o segmentos) son "fragmentos" reales de un contenido de vídeo.

### <a name="hls-master-playlist"></a>Lista de reproducción HLS Master

Este es un ejemplo de un archivo de manifiesto HLS: 

```
#EXTM3U
#EXT-X-VERSION:4
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="audio",NAME="aac_eng_2_128041_2_1",LANGUAGE="eng",DEFAULT=YES,AUTOSELECT=YES,URI="QualityLevels(128041)/Manifest(aac_eng_2_128041_2_1,format=m3u8-aapl)"
#EXT-X-STREAM-INF:BANDWIDTH=536608,RESOLUTION=320x180,CODECS="avc1.64000d,mp4a.40.2",AUDIO="audio"
QualityLevels(381048)/Manifest(video,format=m3u8-aapl)
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=536608,RESOLUTION=320x180,CODECS="avc1.64000d",URI="QualityLevels(381048)/Manifest(video,format=m3u8-aapl,type=keyframes)"
#EXT-X-STREAM-INF:BANDWIDTH=884544,RESOLUTION=480x270,CODECS="avc1.640015,mp4a.40.2",AUDIO="audio"
QualityLevels(721495)/Manifest(video,format=m3u8-aapl)
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=884544,RESOLUTION=480x270,CODECS="avc1.640015",URI="QualityLevels(721495)/Manifest(video,format=m3u8-aapl,type=keyframes)"
#EXT-X-STREAM-INF:BANDWIDTH=1327398,RESOLUTION=640x360,CODECS="avc1.64001e,mp4a.40.2",AUDIO="audio"
QualityLevels(1154816)/Manifest(video,format=m3u8-aapl)
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=1327398,RESOLUTION=640x360,CODECS="avc1.64001e",URI="QualityLevels(1154816)/Manifest(video,format=m3u8-aapl,type=keyframes)"
#EXT-X-STREAM-INF:BANDWIDTH=2413312,RESOLUTION=960x540,CODECS="avc1.64001f,mp4a.40.2",AUDIO="audio"
QualityLevels(2217354)/Manifest(video,format=m3u8-aapl)
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=2413312,RESOLUTION=960x540,CODECS="avc1.64001f",URI="QualityLevels(2217354)/Manifest(video,format=m3u8-aapl,type=keyframes)"
#EXT-X-STREAM-INF:BANDWIDTH=3805760,RESOLUTION=1280x720,CODECS="avc1.640020,mp4a.40.2",AUDIO="audio"
QualityLevels(3579827)/Manifest(video,format=m3u8-aapl)
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=3805760,RESOLUTION=1280x720,CODECS="avc1.640020",URI="QualityLevels(3579827)/Manifest(video,format=m3u8-aapl,type=keyframes)"
#EXT-X-STREAM-INF:BANDWIDTH=139017,CODECS="mp4a.40.2",AUDIO="audio"
QualityLevels(128041)/Manifest(aac_eng_2_128041_2_1,format=m3u8-aapl)
```

### <a name="dash-media-presentation-description-mpd"></a>Descripción de presentación del medio DASH (MPD)

Este es un ejemplo de un manifiesto DASH:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<MPD xmlns="urn:mpeg:dash:schema:mpd:2011" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" profiles="urn:mpeg:dash:profile:isoff-live:2011" type="static" mediaPresentationDuration="PT1M10.315S" minBufferTime="PT7S">
   <Period>
      <AdaptationSet id="1" group="5" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="audio" mimeType="audio/mp4" codecs="mp4a.40.2" lang="en">
         <SegmentTemplate timescale="10000000" media="QualityLevels($Bandwidth$)/Fragments(aac_eng_2_128041_2_1=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(aac_eng_2_128041_2_1=i,format=mpd-time-csf)">
            <SegmentTimeline>
               <S d="60160000" r="10" />
               <S d="41386666" />
            </SegmentTimeline>
         </SegmentTemplate>
         <Representation id="5_A_aac_eng_2_128041_2_1_1" bandwidth="128041" audioSamplingRate="48000" />
      </AdaptationSet>
      <AdaptationSet id="2" group="1" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="video" mimeType="video/mp4" codecs="avc1.640020" maxWidth="1280" maxHeight="720" startWithSAP="1">
         <SegmentTemplate timescale="10000000" media="QualityLevels($Bandwidth$)/Fragments(video=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(video=i,format=mpd-time-csf)">
            <SegmentTimeline>
               <S d="60060000" r="10" />
               <S d="42375666" />
            </SegmentTimeline>
         </SegmentTemplate>
         <Representation id="1_V_video_1" bandwidth="3579827" width="1280" height="720" />
         <Representation id="1_V_video_2" bandwidth="2217354" codecs="avc1.64001F" width="960" height="540" />
         <Representation id="1_V_video_3" bandwidth="1154816" codecs="avc1.64001E" width="640" height="360" />
         <Representation id="1_V_video_4" bandwidth="721495" codecs="avc1.640015" width="480" height="270" />
         <Representation id="1_V_video_5" bandwidth="381048" codecs="avc1.64000D" width="320" height="180" />
      </AdaptationSet>
   </Period>
</MPD>
```
### <a name="smooth-streaming"></a>Smooth Streaming

Este es un ejemplo de un manifiesto de Smooth Streaming:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SmoothStreamingMedia MajorVersion="2" MinorVersion="2" Duration="703146666" TimeScale="10000000">
   <StreamIndex Chunks="12" Type="audio" Url="QualityLevels({bitrate})/Fragments(aac_eng_2_128041_2_1={start time})" QualityLevels="1" Language="eng" Name="aac_eng_2_128041_2_1">
      <QualityLevel AudioTag="255" Index="0" BitsPerSample="16" Bitrate="128041" FourCC="AACL" CodecPrivateData="1190" Channels="2" PacketSize="4" SamplingRate="48000" />
      <c t="0" d="60160000" r="11" />
      <c d="41386666" />
   </StreamIndex>
   <StreamIndex Chunks="12" Type="video" Url="QualityLevels({bitrate})/Fragments(video={start time})" QualityLevels="5">
      <QualityLevel Index="0" Bitrate="3579827" FourCC="H264" MaxWidth="1280" MaxHeight="720" CodecPrivateData="0000000167640020ACD9405005BB011000003E90000EA600F18319600000000168EBECB22C" />
      <QualityLevel Index="1" Bitrate="2217354" FourCC="H264" MaxWidth="960" MaxHeight="540" CodecPrivateData="000000016764001FACD940F0117EF01100000303E90000EA600F1831960000000168EBECB22C" />
      <QualityLevel Index="2" Bitrate="1154816" FourCC="H264" MaxWidth="640" MaxHeight="360" CodecPrivateData="000000016764001EACD940A02FF9701100000303E90000EA600F162D960000000168EBECB22C" />
      <QualityLevel Index="3" Bitrate="721495" FourCC="H264" MaxWidth="480" MaxHeight="270" CodecPrivateData="0000000167640015ACD941E08FEB011000003E90000EA600F162D9600000000168EBECB22C" />
      <QualityLevel Index="4" Bitrate="381048" FourCC="H264" MaxWidth="320" MaxHeight="180" CodecPrivateData="000000016764000DACD941419F9F011000003E90000EA600F14299600000000168EBECB22C" />
      <c t="0" d="60060000" r="11" />
      <c d="42375666" />
   </StreamIndex>
</SmoothStreamingMedia>
```

## <a name="dynamic-manifest"></a>Manifiesto dinámico

El filtro dinámico se utiliza para controlar el número de pistas, formatos, velocidades de bits y ventanas de tiempo de presentación que se envían a los jugadores. Para obtener más información, consulte [filtros y manifiestos dinámicos](filters-dynamic-manifest-overview.md).

> [!NOTE]
> Actualmente, no puede usar Azure Portal para administrar recursos de v3. Use la [API REST](https://aka.ms/ams-v3-rest-ref), la [CLI](https://aka.ms/ams-v3-cli-ref) o uno de los [SDK](media-services-apis-overview.md#sdks) admitidos.

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

[Carga, codificación, streaming de vídeos](stream-files-tutorial-with-api.md)

