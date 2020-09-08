---
title: Entrega de contenido a los clientes
description: Este tema proporciona información general de lo que implica la entrega de contenido con Azure Media Services.
services: media-services
author: Juliako
manager: femila
ms.assetid: 89ede54a-6a9c-4814-9858-dcfbb5f4fed5
ms.service: media-services
ms.workload: media
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: b636d1cdf2e4b9bd137768e22240d8a47d724a97
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266041"
---
# <a name="deliver-content-to-customers"></a>Entrega de contenido a los clientes

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Al proporcionar contenido de vídeo bajo demanda o streaming a los clientes, el objetivo consiste en entregar vídeo de alta calidad a varios dispositivos en condiciones de red diferentes.

Para ello, puede realizar estas tareas:

* Codificar la secuencia en una secuencia de vídeo de velocidad de bits múltiple (velocidad de bits adaptable) (esto se encargará de las condiciones de calidad y red).
* Usar el [empaquetado dinámico](media-services-dynamic-packaging-overview.md) de Microsoft Azure Media Services para volver a empaquetar de forma dinámica la transmisión en distintos protocolos (esto se encargará del streaming en dispositivos diferentes). Media Services admite la entrega de las siguientes tecnologías de streaming con velocidad de bits adaptable: <br/>
    * **HTTP Live Streaming** (HLS): agregue la ruta de acceso "(format=m3u8-aapl)" a la parte "/Manifest" de la dirección URL para indicar al servidor de origen de streaming que devuelva el contenido HLS para consumo en dispositivos nativos **Apple iOS** (para más información, vea [localizadores](#locators) y [direcciones URL](#URLs)),
    * **MPEG-DASH**: agregue la ruta de acceso "(format=mpd-time-csf)" a la parte "/Manifiest" de la dirección URL para indicar al servidor de origen de streaming que devuelva MPEG-DASH (para más información, vea [localizadores](#locators) y [direcciones URL](#URLs)),
    * **Smooth Streaming**.

>[!NOTE]
>Cuando se crea la cuenta de AMS, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Stopped** (Detenido). Para iniciar la transmisión del contenido y aprovechar el empaquetado dinámico y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución). 

En este artículo se ofrece información general sobre importantes conceptos de entrega de contenido.

Para comprobar los problemas conocidos, consulte [esta sección](media-services-deliver-content-overview.md#known-issues).

## <a name="dynamic-packaging"></a>Empaquetado dinámico
Gracias al empaquetado dinámico de Media Services, podrá entregar contenido codificado MP4 de velocidad de bits adaptable o Smooth Streaming en formatos compatibles con Media Services (MPEG-DASH, HLS y Smooth Streaming) sin tener que volver a realizar el empaquetamiento en estos formatos de streaming. Recomendamos entregar el contenido con empaquetado dinámico.

Para aprovechar el empaquetado dinámico, tiene que codificar el archivo intermedio (origen) en un conjunto de archivos MP4 de velocidad de bits adaptable o de Smooth Streaming de velocidad de bits adaptable.

Gracias al empaquetado dinámico, puede almacenar y pagar los archivos en un solo formato de almacenamiento. Media Services creará y publicará la respuesta adecuada según las solicitudes.

Empaquetado dinámico está disponible para puntos de conexión de streaming estándar y premium. 

Para obtener más información, consulte [Empaquetado dinámico](media-services-dynamic-packaging-overview.md).

## <a name="filters-and-dynamic-manifests"></a>Filtros y manifiestos dinámicos
Media Services permite definir filtros para los recursos. Estos filtros son reglas del lado servidor que ayudan a los clientes elegir realizar acciones como reproducir solo una sección de un vídeo o especificar solo un subconjunto de las representaciones de audio y vídeo que el dispositivo de su cliente puede controlar (en lugar de todas las copias asociadas al recurso). Esta funcionalidad de filtro de recursos se realiza a través de los *manifiestos dinámicos* que se crean tras la solicitud del cliente para transmitir un vídeo en función de los filtros especificados.

Para obtener más información, consulte [Filtros y manifiestos dinámicos](media-services-dynamic-manifest-overview.md).

## <a name="locators"></a><a name="locators"></a>Localizadores
Para proporcionar al usuario una dirección URL que pueda utilizarse para transmitir o descargar su contenido, primero necesitará publicar su recurso mediante la creación de un localizador. Los localizadores proporcionan un punto de entrada para tener acceso a los archivos que se encuentran en un recurso. Media Services admite dos tipos de localizadores:

* Localizadores OnDemandOrigin. Se usan para transmitir archivos multimedia (por ejemplo, MPEG DASH, HLS o Smooth Streaming) o archivos de descarga progresiva.
* Localizadores de direcciones URL de firma de acceso compartido (SAS). Se usan para descargar archivos multimedia en el equipo local.

Se utiliza una *directiva de acceso* para definir los permisos (como lectura, escritura y lista) y la duración del acceso del cliente a un recurso determinado. Observe que el permiso de la lista (AccessPermissions.List) no se debe usar al crear un localizador OnDemandOrigin.

Los localizadores tienen fecha de expiración. El Portal de Azure establece una fecha de expiración de 100 años en el futuro para los localizadores.

> [!NOTE]
> Si utilizó el Portal de Azure para crear localizadores antes de marzo de 2015, estos se establecieron para caducar a los 2 años.
> 
> 

Para actualizar la fecha de caducidad de un localizador, use las API de [REST](/rest/api/media/operations/locator#update_a_locator) o de [.NET](https://go.microsoft.com/fwlink/?LinkID=533259). Tenga en cuenta que, cuando se actualiza la fecha de caducidad de un localizador de SAS, cambia la dirección URL.

Los localizadores no están diseñados para administrar el control de acceso por usuario. Puede conceder derechos de acceso diferentes a usuarios individuales con las soluciones de administración de derechos digitales (DRM). Para obtener más información, consulte [Protección de archivos multimedia](/previous-versions/azure/dn282272(v=azure.100)).

Al crear un localizador, puede haber un retraso de 30 segundos debido a los procesos de almacenamiento y propagación requeridos en Azure Storage.

## <a name="adaptive-streaming"></a>Streaming adaptativo
Las tecnologías de velocidad de bits adaptable permiten a las aplicaciones para reproductor de vídeo determinar las condiciones de red y seleccionar entre varias velocidades de bits. Cuando se degrada la comunicación de red, el cliente puede seleccionar una velocidad de bits más baja que permita al reproductor seguir reproduciendo el vídeo con una calidad inferior. A medida que mejoren las condiciones de red, el cliente puede cambiar a una velocidad de bits superior con calidad de vídeo mejorada. Azure Media Services admite la entrega de las siguientes tecnologías de velocidad de bits adaptable: HTTP Live Streaming (HLS), Smooth Streaming y MPEG-DASH.

Para proporcionar direcciones URL de streaming a los usuarios, primero debe crear un localizador OnDemandOrigin. Crear el localizador brinda la ruta de acceso de base al recurso que contiene el contenido que desea transmitir. Sin embargo, para poder transmitir este contenido, es necesario modificar aún más esta ruta de acceso. Para crear una dirección URL completa al archivo de manifiesto del streaming, debe concatenar el valor de la ruta de acceso del localizador y el nombre de archivo del manifiesto (filename.ism). Luego, anexe **/Manifiest** y un formato adecuado (si corresponde) a la ruta de acceso del localizador.

> [!NOTE]
> También puede transmitir el contenido por una conexión TLS. Para ello, asegúrese de que las URL de streaming comienzan por HTTPS. Tenga en cuenta que, actualmente, AMS no admite TLS con dominios personalizados.  
> 

Solo puede transmitir por TLS si se creó el punto de conexión de streaming desde el que se entrega el contenido a partir del 10 de septiembre de 2014. Si las direcciones URL de streaming se basan en los puntos de conexión de streaming creados después del 10 de septiembre de 2014, la dirección URL contendrá streaming.mediaservices.windows.net. Las direcciones URL de streaming que contengan "origin.mediaservices.windows.net" (el formato anterior) no son compatibles con TLS. Si la dirección URL tiene un formato antiguo y quiere transmitir a través de TLS, cree un punto de conexión de streaming nuevo. Use direcciones URL creadas en función del nuevo punto de conexión de streaming para transmitir el contenido a través de TLS.

## <a name="streaming-url-formats"></a><a name="URLs"></a>Formatos de la dirección URL de streaming

### <a name="mpeg-dash-format"></a>Formato MPEG-DASH
{nombre de extremo de streaming-nombre de cuenta de servicios multimedia}.streaming.mediaservices.windows.net/{Id. de localizador}/{nombre de archivo}.ism/Manifest(formato=mpd-time-csf)

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf)

### <a name="apple-http-live-streaming-hls-v4-format"></a>Formato Apple HTTP Live Streaming (HLS) V4
{nombre de extremo de streaming-nombre de cuenta de servicios multimedia}.streaming.mediaservices.windows.net/{Id. de localizador}/{nombre de archivo}.ism/Manifest(formato=m3u8-aapl)

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl)

### <a name="apple-http-live-streaming-hls-v3-format"></a>Formato Apple HTTP Live Streaming (HLS) V3
{nombre de extremo de streaming-nombre de cuenta de servicios multimedia}.streaming.mediaservices.windows.net/{Id. de localizador}/{nombre de archivo}.ism/Manifest(formato=m3u8-aapl-v3)

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3)

### <a name="apple-http-live-streaming-hls-format-with-audio-only-filter"></a>Formato Apple HTTP Live Streaming (HLS) con filtro solo de audio
De forma predeterminada, las pistas solo de audio se incluyen en el manifiesto HLS. Esto es necesario para la certificación de la Apple Store para redes celulares. En este caso, si un cliente no tiene suficiente ancho de banda o se conecta a través de una conexión de 2G, se cambia a reproducción solo de audio. De este modo, se ayuda a mantener un streaming sin almacenamiento en búfer, pero con el inconveniente de no disponer de vídeo. En algunos escenarios, es posible que se prefiera el almacenamiento en búfer del reproductor a solo audio. Si desea quitar la pista de solo audio, puede agregar **audio-only=false** a la dirección URL.

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3,audio-only=false)

Para más información, consulte [Dynamic Manifest Composition support and HLS output additional features](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/)(Compatibilidad con Dynamic Manifest Composition y características adicionales de salida HLS).

### <a name="smooth-streaming-format"></a>Formato Smooth Streaming
{nombre de extremo de streaming-nombre de cuenta de servicios multimedia}.streaming.mediaservices.windows.net/{Id. de localizador}/{filename}.ism/Manifest

Ejemplo:

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest

### <a name="smooth-streaming-20-manifest-legacy-manifest"></a><a id="fmp4_v20"></a>Manifiesto Smooth Streaming 2.0 (manifiesto heredado)
De forma predeterminada el formato de manifiesto Smooth Streaming contiene la etiqueta de repetición (r-tag). Sin embargo, algunos reproductores no son compatibles con r-tag. Los clientes con estos reproductores pueden utilizar un formato que deshabilite la etiqueta r-tag:

{nombre de extremo de streaming-nombre de cuenta de servicios multimedia}.streaming.mediaservices.windows.net/{Id. de localizador}/{nombre de archivo}.ism/Manifest(formato=fmp4-v20)

`http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=fmp4-v20)`

## <a name="progressive-download"></a>Descarga progresiva
La descarga progresiva le permite comenzar a reproducir archivos multimedia antes de haber descargado todo el archivo. No se puede descargar progresivamente archivos .ism * (ismv, isma, ismt e ismc).

Para la descarga progresiva de contenido, utilice el tipo OnDemandOrigin de localizador. En el ejemplo siguiente se muestra la dirección URL que se basa en el tipo OnDemandOrigin de localizador:

`http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4`

Debe descifrar los recursos cifrados en almacenamiento que quiera transmitir desde el servicio de origen para la descarga progresiva.

## <a name="download"></a>Descargar
Para descargar el contenido en un dispositivo cliente, debe crear un localizador de SAS. El localizador de SAS da acceso al contenedor de Azure Storage donde se encuentra el archivo. Para generar la dirección URL de descarga, tiene que insertar el nombre de archivo entre el host y la firma de SAS.

En el ejemplo siguiente se muestra la dirección URL que se basa en el localizador de SAS:

`https://test001.blob.core.windows.net/asset-ca7a4c3f-9eb5-4fd8-a898-459cb17761bd/BigBuckBunny.mp4?sv=2012-02-12&se=2014-05-03T01%3A23%3A50Z&sr=c&si=7c093e7c-7dab-45b4-beb4-2bfdff764bb5&sig=msEHP90c6JHXEOtTyIWqD7xio91GtVg0UIzjdpFscHk%3D`

Se aplican las siguientes consideraciones:

* Debe descifrar los recursos cifrados en almacenamiento que quiera transmitir desde el servicio de origen para la descarga progresiva.
* Se producirá un error en una descarga que no se haya completado en 12 horas.

## <a name="streaming-endpoints"></a>Puntos de conexión de streaming

Un punto de conexión de streaming representa un servicio de streaming que puede entregar contenido directamente a una aplicación de reproducción de cliente o a una red de entrega de contenido (CDN) para su posterior distribución. La secuencia de salida del servicio de punto de conexión de streaming puede ser streaming en vivo o un recurso de vídeo a petición en la cuenta de Media Services. Existen dos tipos de puntos de conexión de streaming: **estándar** y **premium**. Para obtener más información, consulte [Streaming endpoints overview](media-services-streaming-endpoints-overview.md) (Información general de puntos de conexión de streaming).

>[!NOTE]
>Cuando se crea la cuenta de AMS, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Stopped** (Detenido). Para iniciar la transmisión del contenido y aprovechar el empaquetado dinámico y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución). 

## <a name="known-issues"></a>Problemas conocidos
### <a name="changes-to-smooth-streaming-manifest-version"></a>Cambios en la versión de manifiesto de Smooth Streaming
Antes de la versión de servicio de julio de 2016, cuando los recursos producidos por el flujo de trabajo de Media Encoder Standard, Media Encoder Premium o Azure Media Encoder heredado se transmitieron mediante el empaquetado dinámico, el manifiesto de Smooth Streaming devuelto podría ajustarse a la versión 2.0. En la versión 2.0, las duraciones de fragmentos no utilizan las llamadas "etiquetas de repetición". Por ejemplo:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SmoothStreamingMedia MajorVersion="2" MinorVersion="0" Duration="8000" TimeScale="1000">
    <StreamIndex Chunks="4" Type="video" Url="QualityLevels({bitrate})/Fragments(video={start time})" QualityLevels="3" Subtype="" Name="video" TimeScale="1000">
        <QualityLevel Index="0" Bitrate="1000000" FourCC="AVC1" MaxWidth="640" MaxHeight="360" CodecPrivateData="00000001674D4029965201405FF2E02A100000030010000003032E0A000F42400040167F18E3050007A12000200B3F8C70ED0B16890000000168EB7352" />
        <c t="0" d="2000" n="0" />
        <c d="2000" />
        <c d="2000" />
        <c d="2000" />
    </StreamIndex>
</SmoothStreamingMedia>
```

En la versión de servicio de julio de 2016, el manifiesto de Smooth Streaming generado se ajusta a la versión 2.2, con duraciones de fragmentos que utilizan etiquetas de repetición. Por ejemplo:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SmoothStreamingMedia MajorVersion="2" MinorVersion="2" Duration="8000" TimeScale="1000">
    <StreamIndex Chunks="4" Type="video" Url="QualityLevels({bitrate})/Fragments(video={start time})" QualityLevels="3" Subtype="" Name="video" TimeScale="1000">
        <QualityLevel Index="0" Bitrate="1000000" FourCC="AVC1" MaxWidth="640" MaxHeight="360" CodecPrivateData="00000001674D4029965201405FF2E02A100000030010000003032E0A000F42400040167F18E3050007A12000200B3F8C70ED0B16890000000168EB7352" />
        <c t="0" d="2000" r="4" />
    </StreamIndex>
</SmoothStreamingMedia>
```

Es posible que algunos de los clientes heredados de Smooth Streaming no admitan las etiquetas de repetición y se produzcan errores al cargar el manifiesto. Para mitigar este problema, puede usar el parámetro de formato de manifiesto heredado **(format=fmp4-v20)** o actualizar el cliente a la versión más reciente que admita las etiquetas de repetición. Para más información, consulte [Smooth Streaming 2.0](media-services-deliver-content-overview.md#fmp4_v20).

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-topics"></a>Temas relacionados
[Actualización de los localizadores de Media Services después de revertir las claves de almacenamiento](media-services-roll-storage-access-keys.md)
