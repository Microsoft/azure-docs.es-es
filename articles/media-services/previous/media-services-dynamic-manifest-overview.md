---
title: Filtros y manifiestos dinámicos | Microsoft Docs
description: En este tema se describe cómo crear filtros para que su cliente pueda usarlos para el streaming de secciones específicas de una secuencia. Media Services crea manifiestos dinámicos par lograr este streaming selectivo.
services: media-services
documentationcenter: ''
author: cenkdin
manager: femila
editor: ''
ms.assetid: ff102765-8cee-4c08-a6da-b603db9e2054
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 2d83b114487f882b7ee38d3d71c84b6abec04a2b
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266925"
---
# <a name="filters-and-dynamic-manifests"></a>Filtros y manifiestos dinámicos

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector" title1="Seleccione la versión de Media Services que se está usando:"]
> * [Versión 2](media-services-dynamic-manifest-overview.md)
> * [Versión 3](../latest/filters-dynamic-manifest-overview.md)

A partir de la versión 2.17, Media Services le permite definir filtros para los recursos. Estos filtros son reglas del lado servidor que permitirán a los clientes elegir realizar acciones como: reproducir solo una sección de un vídeo (en lugar de reproducir el vídeo completo) o especificar solo un subconjunto de las representaciones de audio y vídeo que el dispositivo de su cliente puede controlar (en lugar de todas las representaciones asociadas al recurso). Este filtrado de sus activos se logra a través de los **manifiestos dinámicos**que se crean tras la solicitud del cliente para transmitir un vídeo en función de los filtros especificados.

En este tema, se describen escenarios comunes en los que el uso de filtros resultaría muy beneficioso para los clientes y se proporcionan vínculos a temas donde se explica cómo crear filtros mediante programación.

## <a name="overview"></a>Información general
Cuando entrega su contenido a los clientes (streaming de eventos en directo o vídeo bajo demanda), su objetivo es entregar un vídeo de alta calidad a varios dispositivos en condiciones de red diferentes. Para lograr este objetivo, haga lo siguiente:

* codifique la secuencia a secuencia de vídeo de velocidad de bits múltiple ([velocidad de bits adaptativa](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming)) (esto se encargará de las condiciones de calidad y red) y 
* use el [empaquetado dinámico](media-services-dynamic-packaging-overview.md) de Media Services dinámicamente para volver a empaquetar dinámicamente su secuencia en distintos protocolos (esto se encargará de la transmisión por secuencias en dispositivos diferentes). Media Services admite la entrega de las siguientes tecnologías de streaming con velocidad de bits adaptable: HTTP Live Streaming (HLS), Smooth Streaming y MPEG DASH. 

### <a name="manifest-files"></a>Archivos de manifiesto
Cuando codifique un activo para transmisión por secuencias de velocidad de bits adaptativa, se crea un archivo de **manifiesto** (lista de reproducción) (el archivo se basa en texto o XML). El archivo de **manifiesto** incluye metadatos de transmisión por secuencias como: el tipo de pista (audio, vídeo o texto), el nombre de la pista, la hora inicial y final, la velocidad de bits (calidades), los idiomas de pista, la ventana de presentación (ventana deslizante de duración fija), el códec de vídeo (FourCC). También indica al reproductor que recupere el siguiente fragmento ofreciendo información sobre los próximos fragmentos de vídeo reproducibles disponibles y su ubicación. Los fragmentos (o segmentos) son "fragmentos" reales de un contenido de vídeo.

Este es un ejemplo de un archivo de manifiesto: 

```xml
<?xml version="1.0" encoding="UTF-8"?>    
<SmoothStreamingMedia MajorVersion="2" MinorVersion="0" Duration="330187755" TimeScale="10000000">

<StreamIndex Chunks="17" Type="video" Url="QualityLevels({bitrate})/Fragments(video={start time})" QualityLevels="8">
<QualityLevel Index="0" Bitrate="5860941" FourCC="H264" MaxWidth="1920" MaxHeight="1080" CodecPrivateData="0000000167640028AC2CA501E0089F97015202020280000003008000001931300016E360000E4E1FF8C7076850A4580000000168E9093525" />
<QualityLevel Index="1" Bitrate="4602724" FourCC="H264" MaxWidth="1920" MaxHeight="1080" CodecPrivateData="0000000167640028AC2CA501E0089F97015202020280000003008000001931100011EDC00002CD29FF8C7076850A45800000000168E9093525" />
<QualityLevel Index="2" Bitrate="3319311" FourCC="H264" MaxWidth="1280" MaxHeight="720" CodecPrivateData="000000016764001FAC2CA5014016EC054808080A00000300020000030064C0800067C28000103667F8C7076850A4580000000168E9093525" />
<QualityLevel Index="3" Bitrate="2195119" FourCC="H264" MaxWidth="960" MaxHeight="540" CodecPrivateData="000000016764001FAC2CA503C045FBC054808080A000000300200000064C1000044AA0000ABA9FE31C1DA14291600000000168E9093525" />
<QualityLevel Index="4" Bitrate="1469881" FourCC="H264" MaxWidth="960" MaxHeight="540" CodecPrivateData="000000016764001FAC2CA503C045FBC054808080A000000300200000064C04000B71A0000E4E1FF8C7076850A4580000000168E9093525" />
<QualityLevel Index="5" Bitrate="978815" FourCC="H264" MaxWidth="640" MaxHeight="360" CodecPrivateData="000000016764001EAC2CA50280BFE5C0548303032000000300200000064C08001E8480004C4B7F8C7076850A45800000000168E9093525" />
<QualityLevel Index="6" Bitrate="638374" FourCC="H264" MaxWidth="640" MaxHeight="360" CodecPrivateData="000000016764001EAC2CA50280BFE5C0548303032000000300200000064C080013D60000C65DFE31C1DA1429160000000168E9093525" />
<QualityLevel Index="7" Bitrate="388851" FourCC="H264" MaxWidth="320" MaxHeight="180" CodecPrivateData="000000016764000DAC2CA505067E7C054830303200000300020000030064C040030D40003D093F8C7076850A45800000000168E9093525" />

<c t="0" d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="9600000"/>
</StreamIndex>


<StreamIndex Chunks="17" Type="audio" Url="QualityLevels({bitrate})/Fragments(AAC_und_ch2_128kbps={start time})" QualityLevels="1" Name="AAC_und_ch2_128kbps">
<QualityLevel AudioTag="255" Index="0" BitsPerSample="16" Bitrate="125658" FourCC="AACL" CodecPrivateData="1210" Channels="2" PacketSize="4" SamplingRate="44100" />

<c t="0" d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="6965987" /></StreamIndex>


<StreamIndex Chunks="17" Type="audio" Url="QualityLevels({bitrate})/Fragments(AAC_und_ch2_56kbps={start time})" QualityLevels="1" Name="AAC_und_ch2_56kbps">
<QualityLevel AudioTag="255" Index="0" BitsPerSample="16" Bitrate="53655" FourCC="AACL" CodecPrivateData="1210" Channels="2" PacketSize="4" SamplingRate="44100" />

<c t="0" d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="6965987" /></StreamIndex>

</SmoothStreamingMedia>
```

### <a name="dynamic-manifests"></a>Manifiestos dinámicos
Hay [escenarios](media-services-dynamic-manifest-overview.md#scenarios) cuando el cliente necesita mayor flexibilidad que lo que se describe en el archivo de manifiesto del activo predeterminado. Por ejemplo:

* Específico de dispositivo: entregue únicamente las representaciones y pistas de idioma especificadas que admite el dispositivo que se usa para la reproducción del contenido ("filtrado de representaciones"). 
* Reduzca el manifiesto para mostrar un clip secundario de un evento en directo ("filtrado de vídeos secundarios").
* Recorte el inicio de un vídeo ("recorte de un vídeo").
* Ajuste la ventana de presentación (DVR) para ofrecer una longitud limitada de la ventana de DVR en el reproductor ("ventana de presentación de ajuste").

Para lograr esta flexibilidad, los Media Services ofrecen **manifiestos dinámicos** basados en [filtros](media-services-dynamic-manifest-overview.md#filters)predefinidos.  Cuando defina los filtros, los clientes podrían usarlos para transmitir una representación específica o clips secundarios del vídeo. Especificarían filtros en la URL de streaming. Se podrían aplicar filtros a protocolos de streaming con velocidad de bits adaptable compatibles con [empaquetado dinámico](media-services-dynamic-packaging-overview.md): HLS, MPEG-DASH y Smooth Streaming. Por ejemplo:

URL de MPEG DASH con filtro

`http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf,filter=MyLocalFilter)`

URL de Smooth Streaming con filtro

`http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(filter=MyLocalFilter)`


Para obtener más información sobre cómo entregar el contenido y crear URL de streaming, vea [Información general de entregar contenido](media-services-deliver-content-overview.md).

> [!NOTE]
> Tenga en cuenta que los manifiestos dinámicos no cambian el activo y el manifiesto predeterminado para ese activo. Su cliente puede elegir solicitar una secuencia con o sin filtros. 
> 
> 

### <a name="filters"></a><a id="filters"></a>Filtros
Hay dos tipos de filtros de activos: 

* Filtros globales (se pueden aplicar a cualquier activo de la cuenta de Azure Media Services, tienen una duración de la cuenta) y 
* Filtros locales (solo se pueden aplicar a un activo con el que estaba asociado el filtro una vez creado, tienen una duración del activo). 

Los tipos de filtros globales y locales tienen exactamente las mismas propiedades. La diferencia principal entre los dos es para qué escenarios es más adecuado cada tipo de filtro. Los filtros globales suelen ser adecuados para los perfiles de dispositivos (filtrado de representaciones) donde los filtros locales podrían usarse para recortar un activo específico.

## <a name="common-scenarios"></a><a id="scenarios"></a>Escenarios comunes
Tal y como se mencionó anteriormente, cuando entrega contenido a los clientes (transmisión de eventos en directo o vídeo bajo demanda), su objetivo es proporcionar un vídeo de alta calidad a diferentes dispositivos en condiciones de red variables. Pueden darse también otros requisitos que obliguen a filtrar los recursos y a utilizar **manifiestos dinámicos**. En las secciones siguientes se ofrece una breve introducción a los diferentes escenarios de filtrado.

* Especifique solo un subconjunto de representaciones de audio y vídeo que pueden controlar determinados dispositivos (en lugar de todas las representaciones asociadas al activo). 
* Reproducir solo una sección de un vídeo (en lugar de reproducir todo el vídeo).
* Ajustar la ventana de presentación de DVR.

## <a name="rendition-filtering"></a>Filtrado de representaciones
Puede elegir codificar el activo en varios perfiles de codificación (H.264 Baseline, H.264 High, AACL, AACH, Dolby Digital Plus) y varias velocidades de bits de calidad. Sin embargo, no todos los dispositivos cliente serán compatibles con todos los perfiles y velocidades de bits de su activo. Por ejemplo, los dispositivos Android más antiguos solo admiten H.264 Baseline+AACL. El envío de velocidades de bits más altas a un dispositivo que no puede obtener los beneficios, desperdicia el ancho de banda y el cálculo del dispositivo. Dicho dispositivo debe descodificar toda la información ofrecida, solo para reducirla verticalmente para presentación.

Con el manifiesto dinámico, puede crear perfiles de dispositivo como móvil, consola, HD/SD, etc. e incluir las pistas y calidades que quiere que formen parte de cada perfil.

![Ejemplo de filtrado de representaciones][renditions2]

En el ejemplo siguiente, se usó un codificador para codificar un recurso intermedio en siete representaciones de vídeo MP4 ISO (de 180p a 1080p). El recurso codificado puede empaquetarse dinámicamente en cualquiera de los siguientes protocolos de transmisión: HLS, Smooth y MPEG DASH.  En la parte superior del diagrama, se muestra el manifiesto HLS para el activo sin filtros (contiene las siete representaciones).  En la parte inferior izquierda, se muestra el manifiesto HLS al que se aplicó un filtro denominado "ott". El filtro de "ott" especifica la eliminación de todas las velocidades de bits por debajo de 1 Mbps, lo que dio lugar a que se quitaran los dos niveles de calidad inferiores en la respuesta. En la parte inferior derecha se muestra el manifiesto HLS al que se aplicó un filtro denominado "móvil". El filtro "móvil" especifica la eliminación de las representaciones donde la resolución es mayor que 720p, lo que hizo que se quitaran las dos representaciones de 1080p.

![Filtrado de representaciones][renditions1]

## <a name="removing-language-tracks"></a>Quitar pistas de idioma
Los activos pueden incluir varios idiomas de audio como inglés, español, francés, etc. Normalmente, los administradores del SDK del reproductor toman como valor predeterminado la selección de pistas de audio y las pistas de audio disponibles por selección de usuario. Es un desafío desarrollar estos SDK del Reproductor; requiere diferentes implementaciones en marcos de reproductores específicos de dispositivos. Además, en algunas plataformas, las API del reproductor están limitadas y no incluyen la característica de selección de audio, donde los usuarios no pueden seleccionar o cambiar la pista de audio predeterminada. Con los filtros de activo, puede controlar el comportamiento creando filtros que solo incluyen idiomas de audio deseados.

![Filtrado de pistas de idioma][language_filter]

## <a name="trimming-start-of-an-asset"></a>Recorte del inicio de un activo
En la mayoría de los eventos de streaming en directo, los operadores ejecutan algunas pruebas antes del evento real. Por ejemplo, podrían incluir una pizarra como esta antes del inicio del evento: "El programa comenzará momentáneamente". Si el programa se está archivando, los datos de pizarra y de prueba también se archivan y se incluyen en la presentación. Sin embargo, esta información no se debe mostrar a los clientes. Con el manifiesto dinámico, puede crear un filtro de tiempo de inicio y quitar los datos no deseados del manifiesto.

![Inicio de recorte][trim_filter]

## <a name="creating-subclips-views-from-a-live-archive"></a>Crear clips secundarios (vistas) desde un archivo dinámico
Muchos eventos en directo son de larga ejecución y el archivo en directo puede incluir varios eventos. Después de que termine el evento en directo, es posible que los emisores deseen dividir el archivo dinámico en secuencias de inicio y detención de programa lógicas. Después, publique por separado estos programas virtuales sin procesar posteriormente el archivo dinámico y sin crear activos separados (que no se beneficiarán de los fragmentos en caché existentes en las CDN). Entre los ejemplos de estos programas virtuales se encuentran los cuatro cuartos de un partido de baloncesto o fútbol americano, las entradas en el béisbol o los eventos individuales de una tarde de un programa deportivo.

Con el manifiesto dinámico, puede crear filtros mediante las horas inicial y final y crear vistas virtuales encima de su archivo dinámico. 

![Filtro de clips secundarios][subclip_filter]

Activo filtrado:

![Esquí][skiing]

## <a name="adjusting-presentation-window-dvr"></a>Ajustar la ventana de presentación (DVR)
En la actualidad, Azure Media Services ofrecen un archivo circular donde la duración se puede configurar entre 5 minutos y 25 horas. El filtrado de manifiestos se puede usar para crear una ventana de DVR dinámica encima del archivo, sin eliminar medios. Hay muchos escenarios en los que los emisores quieren proporcionar una ventana de DVR limitada que se mueva con el borde dinámico y al mismo tiempo mantener una ventana de archivado más grande. Es posible que un emisor desee usar los datos que están fuera de la ventana de DVR para resaltar clips, o que desee proporcionar diferentes ventanas de DVR para dispositivos diferentes. Por ejemplo, la mayoría de los dispositivos móviles no administran grandes ventanas de DVR (puede tener una ventana de DVR de 2 minutos para dispositivos móviles y 1 hora para clientes de escritorio).

![Ventana de DVR][dvr_filter]

## <a name="adjusting-livebackoff-live-position"></a>Ajustar LiveBackoff (posición en directo)
El filtrado de manifiestos puede usarse para quitar varios segundos del borde directo de un programa activo. El filtrado permite a los emisores ver la presentación en el punto de publicación de vista previa y crear puntos de inserción de anuncios antes de que los visores reciban la transmisión (normalmente con una interrupción de copia anterior a 30 segundos). Los emisores, entonces, pueden insertar estos anuncios en sus marcos de cliente a tiempo para su recepción y procesar la información antes de la oportunidad de anuncio.

Además de la compatibilidad de anuncio, la opción de configuración LiveBackoff se puede usar para ajustar la posición de los visores para que cuando los clientes se desvíen y lleguen al borde directo puedan obtener todavía fragmentos del servidor en lugar de obtener un error HTTP 404 o 412.

![livebackoff_filter][livebackoff_filter]

## <a name="combining-multiple-rules-in-a-single-filter"></a>Combinación de varias reglas en un filtro único
Puede combinar varias reglas de filtrado en un filtro único. Por ejemplo, puede definir una "regla de intervalos" para quitar las pizarras de un archivo dinámico y filtrar además las velocidades de bits disponibles. Al aplicar varias reglas de filtrado, el resultado final es la intersección de todas las reglas.

![varias reglas][multiple-rules]

## <a name="create-filters-programmatically"></a>Crear filtros mediante programación
En el siguiente artículo se describen las entidades de Media Services que están relacionadas con los filtros. En el artículo también se muestra cómo crear filtros mediante programación.  

[Crear filtros con las API de REST](media-services-rest-dynamic-manifest.md).

## <a name="combining-multiple-filters-filter-composition"></a>Combinar múltiples filtros (composición de filtros)
También puede combinar varios filtros en una sola dirección URL. 

En el escenario siguiente se muestra para qué puede resultar útil la combinación de filtros:

1. Necesita filtrar sus calidades de vídeos para dispositivos móviles, como Android o iPAD (con el fin de limitar las calidades de vídeo). Para quitar las calidades no deseadas, debe crear un filtro global adecuado para los perfiles de dispositivo. Como se mencionó anteriormente en este artículo, los filtros globales pueden utilizarse para todos sus activos en la misma cuenta de servicios multimedia sin ninguna asociación adicional. 
2. También desea recortar el tiempo de inicio y finalización de un activo. Para hacerlo, debe crear un filtro local y establecer la hora de inicio y fin. 
3. Quiere combinar ambos filtros (sin combinación tendría que agregar el filtrado de calidad al filtro de recorte, lo cual dificultaría el uso del filtro).

Para combinar filtros, deberá establecer los nombres de filtro de la dirección URL del manifiesto o la lista de reproducción separados por punto y coma. Supongamos que tiene un filtro denominado *MyMobileDevice* que filtra cualidades y que tiene otro denominado *MyStartTime* para establecer una determinada hora de inicio. Puede combinarlos así:

`http://teststreaming.streaming.mediaservices.windows.net/3d56a4d-b71d-489b-854f-1d67c0596966/64ff1f89-b430-43f8-87dd-56c87b7bd9e2.ism/Manifest(filter=MyMobileDevice;MyStartTime)`

Puede combinar hasta tres filtros. 

Para más información, consulte [este blog](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/) .

## <a name="know-issues-and-limitations"></a>Problemas conocidos y limitaciones
* El manifiesto dinámico funciona en los límites GOP (fotogramas clave), por lo que el recorte tiene precisión GOP. 
* Puede usar el mismo nombre de filtro para los filtros globales y locales. Los filtros locales tienen una mayor prioridad e invalidarán los filtros globales.
* Si actualiza un filtro, se pueden tardar hasta 2 minutos en que el extremo de streaming actualice las reglas. Si el contenido se suministró con algunos filtros (y se almacenó en caché en servidores proxy y cachés CDN), la actualización de  estos filtros puede generar errores del reproductor. Se recomienda borrar la memoria caché después de actualizar el filtro. Si esta opción no es posible, piense en usar un filtro diferente.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Consulte también
[Información general de entrega de contenido a los clientes](media-services-deliver-content-overview.md)

[renditions1]: ./media/media-services-dynamic-manifest-overview/media-services-rendition-filter.png
[renditions2]: ./media/media-services-dynamic-manifest-overview/media-services-rendition-filter2.png

[rendered_subclip]: ./media/media-services-dynamic-manifests/media-services-rendered-subclip.png
[timeline_trim_event]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-event.png
[timeline_trim_subclip]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-subclip.png

[multiple-rules]:./media/media-services-dynamic-manifest-overview/media-services-multiple-rules-filters.png

[subclip_filter]: ./media/media-services-dynamic-manifest-overview/media-services-subclips-filter.png
[trim_event]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-event.png
[trim_subclip]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-subclip.png
[trim_filter]: ./media/media-services-dynamic-manifest-overview/media-services-trim-filter.png
[redered_subclip]: ./media/media-services-dynamic-manifests/media-services-rendered-subclip.png
[livebackoff_filter]: ./media/media-services-dynamic-manifest-overview/media-services-livebackoff-filter.png
[language_filter]: ./media/media-services-dynamic-manifest-overview/media-services-language-filter.png
[dvr_filter]: ./media/media-services-dynamic-manifest-overview/media-services-dvr-filter.png
[skiing]: ./media/media-services-dynamic-manifest-overview/media-services-skiing.png
