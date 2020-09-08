---
title: Filtrado de los manifiestos mediante el empaquetador dinámico
titleSuffix: Azure Media Services
description: Aprenda a crear filtros con el empaquetador dinámico para filtrar y transmitir selectivamente los manifiestos.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: acb30c1659c4c29e0af83da5594bdd9a7e3465d8
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299038"
---
# <a name="filter-your-manifests-using-dynamic-packager"></a>Filtrado de los manifiestos mediante el empaquetador dinámico

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Cuando entrega contenido de streaming con velocidad de bits adaptable, en ocasiones es necesario publicar varias versiones de un manifiesto para acomodarse a funcionalidades específicas del dispositivo o al ancho de banda de red disponible. El [empaquetador dinámico](dynamic-packaging-overview.md) permite especificar los filtros que pueden filtrar códecs específicos, resoluciones, velocidades de bits y combinaciones de pistas de audio sobre la marcha. Este filtrado elimina la necesidad de crear varias copias. Solo necesita publicar una nueva dirección URL con un conjunto específico de filtros configurados para los dispositivos de destino (iOS, Android, SmartTV o exploradores) y las funcionalidades de red (escenarios de ancho de banda alto o bajo o móviles). En este caso, los clientes pueden manipular el streaming de contenido mediante la cadena de consulta (mediante la especificación de los [filtros de recursos o de cuenta](filters-concept.md) disponibles) y usar filtros para el streaming de secciones específicas de una secuencia.

Algunos escenarios de entrega requieren asegurarse de que un cliente no pueda acceder a pistas específicas. Por ejemplo, quizás no le interese publicar un manifiesto que contenga las pistas de HD en un nivel de suscriptor específico. O bien, es posible que quiera quitar pistas con velocidad de bits adaptable (ABR) para reducir el costo de entrega a un dispositivo específico que no se beneficiaría de las pistas adicionales. En este caso, puede asociar una lista de filtros creados previamente a su [localizador de streaming](streaming-locators-concept.md) durante la creación. Así, los clientes no pueden manipular cómo se transmite el contenido, ya que está definido en dicho **localizador de streaming**.

Puede combinar el filtrado mediante la especificación de [filtros en el localizador de streaming](filters-concept.md#associating-filters-with-streaming-locator) y filtros adicionales específicos del dispositivo que el cliente defina en la dirección URL. Esta combinación puede ser útil para restringir las pistas adicionales, como secuencias de metadatos o eventos, idiomas de audio o pistas de audio descriptivas.

Esta capacidad para especificar filtros diferentes en la secuencia proporciona una eficaz solución de manipulación del **manifiesto dinámico** que permite diversos escenarios de casos de uso para los dispositivos de destino. En este tema se explican conceptos relacionados con los **manifiestos dinámicos** y se proporcionan ejemplos de escenarios en los que puede usar esta característica.

> [!NOTE]
> Los manifiestos dinámicos no cambian el recurso ni el manifiesto predeterminado de ese recurso.

## <a name="overview-of-manifests"></a>Introducción a los manifiestos

Azure Media Services admite los protocolos HLS, MPEG DASH y Smooth Streaming. Como parte del [empaquetado dinámico](dynamic-packaging-overview.md), los manifiestos de cliente de streaming (HLS Master Playlist, DASH Media Presentation Description [MPD] y Smooth Streaming), se generan dinámicamente a partir del selector de formato de la dirección URL. Para más información, consulte los protocolos de entrega en el [flujo de trabajo común a petición](dynamic-packaging-overview.md#to-prepare-your-source-files-for-delivery).

### <a name="get-and-examine-manifest-files"></a>Obtención y examen de archivos de manifiesto

Se especifica una lista de condiciones de propiedad de pista de filtro en función de las pistas de la secuencia (vídeo en directo o a petición [VOD]) que deben incluirse en el manifiesto creado de forma dinámica. Para obtener y examinar las propiedades de las pistas, primero debe cargar el manifiesto de Smooth Streaming.

El tutorial [Carga, codificación y streaming con .NET](stream-files-tutorial-with-api.md#get-streaming-urls) muestra cómo crear las direcciones URL de transmisión con .NET. Si ejecuta la aplicación, una de las URLs apunta al manifiesto Smooth Streaming: `https://amsaccount-usw22.streaming.media.azure.net/00000000-0000-0000-0000-0000000000000/ignite.ism/manifest`.<br/> Copie y pegue la dirección URL en la barra de direcciones de un explorador. Se descargará el archivo. Puede abrirlo en cualquier editor de texto.

Para ver un ejemplo de REST, consulte [Carga, codificación y streaming con REST](stream-files-tutorial-with-rest.md#list-paths-and-build-streaming-urls).

### <a name="monitor-the-bitrate-of-a-video-stream"></a>Supervisión de la velocidad de bits de una secuencia de vídeo

Puede utilizar la [página de demostración de Azure Media Player](https://aka.ms/azuremediaplayer) para supervisar la velocidad de bits de una secuencia de vídeo. La página de demostración muestra información de diagnóstico en la pestaña **Diagnósticos**:

![Diagnósticos de Azure Media Player][amp_diagnostics]

### <a name="examples-urls-with-filters-in-query-string"></a>Ejemplos: Direcciones URL con filtros en la cadena de consulta

Puede aplicar filtros a protocolos de streaming ABR: HLS, MPEG-DASH y Smooth Streaming. En la tabla siguiente se muestran algunos ejemplos de direcciones URL con filtros:

|Protocolo|Ejemplo|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|Smooth Streaming|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="rendition-filtering"></a>Filtrado de representaciones

Puede elegir codificar el recurso en varios perfiles de codificación (H.264 Baseline, H.264 High, AACL, AACH, Dolby Digital Plus) y varias velocidades de bits de calidad. Sin embargo, no todos los dispositivos cliente serán compatibles con todos los perfiles y velocidades de bits de su activo. Por ejemplo, los dispositivos Android más antiguos solo admiten H.264 Baseline+AACL. El envío de velocidades de bits más altas a un dispositivo que no puede obtener los beneficios, desperdicia el ancho de banda y el cálculo del dispositivo. Dicho dispositivo debe descodificar toda la información ofrecida, solo para reducirla verticalmente para presentación.

Con el manifiesto dinámico, puede crear perfiles de dispositivo como móvil, consola, HD/SD, etc. e incluir las pistas y calidades que quiere que formen parte de cada perfil. Esto se denomina filtrado de representaciones. El diagrama siguiente muestra un ejemplo.

![Ejemplo de filtrado de representación con el manifiesto dinámico][renditions2]

En el ejemplo siguiente, se usó un codificador para codificar un recurso intermedio en siete representaciones de vídeo MP4 ISO (de 180p a 1080p). El recurso codificado puede [empaquetarse dinámicamente](dynamic-packaging-overview.md) en cualquiera de los siguientes protocolos de transmisión: HLS, MPEG DASH y Smooth.

En la parte superior del diagrama siguiente se muestra el manifiesto HLS del recurso sin filtros. (Contiene las siete copias).  En la parte inferior izquierda, el diagrama muestra el manifiesto HLS al que se aplicó un filtro denominado "ott". El filtro de "ott" especifica la eliminación de todas las velocidades de bits por debajo de 1 Mbps, lo que dio lugar a que se quitaran los dos niveles de calidad inferiores en la respuesta. En la parte inferior derecha, el diagrama muestra el manifiesto HLS al que se aplicó un filtro llamado "mobile". El filtro "mobile" especifica la eliminación de las representaciones donde la resolución es mayor que 720p, lo que hizo que se quitaran las dos representaciones de 1080p.

![Filtrado de representación con el manifiesto dinámico][renditions1]

## <a name="removing-language-tracks"></a>Quitar pistas de idioma
Los recursos pueden incluir varios idiomas de audio, como inglés, español, francés, etc. Normalmente, el SDK del Reproductor administra la selección de las pistas de audio predeterminadas y la selección de las pistas de audio disponibles por usuario.

Desarrollar estos SDK del Reproductor es complicado, ya que requiere diferentes implementaciones en plataformas de reproductores específicas del dispositivo. Además, en algunas plataformas, las API del Reproductor están limitadas y no incluyen la característica de selección de audio, así que los usuarios no pueden seleccionar ni cambiar la pista de audio predeterminada. Con los filtros de activo, puede controlar el comportamiento creando filtros que solo incluyen idiomas de audio deseados.

![Filtrado de pistas de idiomas con el manifiesto dinámico][language_filter]

## <a name="trimming-the-start-of-an-asset"></a>Recorte del inicio de un recurso

En la mayoría de los eventos de streaming en directo, los operadores ejecutan algunas pruebas antes del evento real. Por ejemplo, podrían incluir una pizarra como esta antes del inicio del evento: "El programa comenzará momentáneamente".

Si el programa se está archivando, los datos de pizarra y de prueba también se archivan y se incluyen en la presentación. Sin embargo, esta información no se debe mostrar a los clientes. Con el manifiesto dinámico, puede crear un filtro de tiempo de inicio y quitar los datos no deseados del manifiesto.

![Recorte del inicio de un recurso con el manifiesto dinámico][trim_filter]

## <a name="creating-subclips-views-from-a-live-archive"></a>Crear clips secundarios (vistas) desde un archivo dinámico

Muchos eventos en directo son de larga ejecución y el archivo en directo puede incluir varios eventos. Después de que termine el evento en directo, es posible que los emisores deseen dividir el archivo dinámico en secuencias de inicio y detención de programa lógicas.

Puede publicar por separado estos programas virtuales sin procesar posteriormente el archivo dinámico y sin crear recursos independientes (que no se benefician de los fragmentos en caché existentes en las CDN). Entre los ejemplos de estos programas virtuales se encuentran los cuatro cuartos de un partido de baloncesto o fútbol americano, las entradas en el béisbol o los eventos individuales de una tarde de un programa deportivo.

Con el manifiesto dinámico, puede crear filtros mediante las horas inicial y final y crear vistas virtuales encima de su archivo dinámico.

![Filtro de subclip con el manifiesto dinámico][subclip_filter]

Este es el recurso filtrado:

![Recurso filtrado con el manifiesto dinámico][skiing]

## <a name="adjusting-the-presentation-window-dvr"></a>Ajustar la ventana de presentación (DVR)

En la actualidad, Azure Media Services ofrecen un archivo circular donde la duración se puede configurar entre 1 minuto y 25 horas. El filtrado de manifiestos se puede usar para crear una ventana de DVR dinámica encima del archivo, sin eliminar medios. Hay muchos escenarios en los que los emisores quieren proporcionar una ventana de DVR limitada que se mueva con el borde dinámico y al mismo tiempo mantener una ventana de archivado más grande. Puede que un emisor quiera usar los datos que están fuera de la ventana de DVR para resaltar clips, o que desee proporcionar diferentes ventanas de DVR para dispositivos diferentes. Por ejemplo, la mayoría de los dispositivos móviles no administran grandes ventanas de DVR (puede tener una ventana de DVR de 2 minutos para dispositivos móviles y 1 hora para clientes de escritorio).

![Ventana de DVR con el manifiesto dinámico][dvr_filter]

## <a name="adjusting-livebackoff-live-position"></a>Ajustar LiveBackoff (posición en directo)

El filtrado de manifiestos puede usarse para quitar varios segundos del borde directo de un programa activo. El filtrado permite a los emisores ver la presentación en el punto de publicación de versión preliminar y crear puntos de inserción de anuncios antes de que los visores reciban la transmisión (normalmente con una interrupción de copia anterior a 30 segundos). Entonces, los emisores pueden insertar estos anuncios en sus marcos de cliente a tiempo para su recepción y procesar la información antes de la oportunidad de anuncio.

Además de la compatibilidad de anuncio, la opción de configuración LiveBackoff se puede usar para ajustar la posición de los visores para que cuando los clientes se desvíen y lleguen al borde directo puedan obtener todavía fragmentos del servidor. De este modo, los clientes no recibirán un error HTTP 404 o 412.

![Filtrado de retroceso en vivo con el manifiesto dinámico][livebackoff_filter]

## <a name="combining-multiple-rules-in-a-single-filter"></a>Combinación de varias reglas en un filtro único

Puede combinar varias reglas de filtrado en un filtro único. Por ejemplo, puede definir una "regla de intervalos" para quitar las caretas de un archivo dinámico y filtrar además las velocidades de bits disponibles. Al aplicar varias reglas de filtrado, el resultado final es la intersección de todas las reglas.

![Varias reglas de filtrado con el manifiesto dinámico][multiple-rules]

## <a name="combining-multiple-filters-filter-composition"></a>Combinar múltiples filtros (composición de filtros)

También puede combinar varios filtros en una sola dirección URL. En el escenario siguiente se muestra para qué puede resultar útil la combinación de filtros:

1. Necesita filtrar las calidades de vídeos para dispositivos móviles, como Android o iPad (con el fin de limitar las calidades de vídeo). Para quitar las calidades no deseadas, debe crear un filtro de cuenta adecuado para los perfiles de dispositivo. Puede usar filtros de cuenta para todos sus recursos en la misma cuenta de servicios multimedia sin ninguna asociación adicional.
1. También desea recortar el tiempo de inicio y finalización de un activo. Para ello, debe crear un filtro de recursos y establecer la hora de inicio y fin.
1. Quiere combinar ambos filtros. Sin combinación tendría que agregar el filtrado de calidad al filtro de recorte, lo cual dificultaría el uso del filtro.

Para combinar filtros, establezca los nombres de filtro en la dirección URL del manifiesto o la lista de reproducción separados por punto y coma. Supongamos que tiene un filtro llamado *MyMobileDevice* que filtra cualidades, y que tiene otro llamado *MyStartTime* para establecer una determinada hora de inicio. Puede combinar hasta tres filtros.

Para más información, vea [esta publicación del blog](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/).

## <a name="considerations-and-limitations"></a>Consideraciones y limitaciones

- Los valores de **forceEndTimestamp**, **presentationWindowDuration** y **liveBackoffDuration** se deben establecer para un filtro de VOD. Solo se utilizan para escenarios de filtrado en vivo.
- El manifiesto dinámico funciona en los límites GOP (fotogramas clave), por lo que el recorte tiene precisión GOP.
- Puede usar el mismo nombre de filtro para los filtros de cuentas y recursos. Los filtros de recurso tienen una mayor prioridad e invalidarán los filtros de cuenta.
- Si actualiza un filtro, se pueden tardar hasta 2 minutos en que el punto de conexión de streaming actualice las reglas. Si usó filtros para suministrar el contenido (y se almacenó en caché en servidores proxy y cachés CDN), la actualización de estos filtros puede generar errores del reproductor. Se recomienda borrar la memoria caché después de actualizar el filtro. Si esta opción no es posible, piense en usar un filtro diferente.
- Los clientes necesitan descargar manualmente el manifiesto y analizar la marca de hora de inicio exacta y la escala temporal.

    - Para determinar las propiedades de las pistas de un recurso, [obtenga y examine el archivo de manifiesto](#get-and-examine-manifest-files).
    - La fórmula para establecer las propiedades de la marca de tiempo del filtro de recursos es: <br/>startTimestamp = &lt;hora de inicio del manifiesto&gt; +  &lt;hora de inicio del filtro esperado en segundos&gt; * escala temporal

## <a name="next-steps"></a>Pasos siguientes

Los artículos siguientes muestran cómo crear filtros mediante programación:  

- [Creación de filtros con las API REST](filters-dynamic-manifest-rest-howto.md)
- [Creación de filtros con .NET](filters-dynamic-manifest-dotnet-howto.md)
- [Creación de filtros con la CLI](filters-dynamic-manifest-cli-howto.md)

[renditions1]: ./media/filters-dynamic-manifest-overview/media-services-rendition-filter.png
[renditions2]: ./media/filters-dynamic-manifest-overview/media-services-rendition-filter2.png

[rendered_subclip]: ./media/filters-dynamic-manifests/media-services-rendered-subclip.png
[timeline_trim_event]: ./media/filters-dynamic-manifests/media-services-timeline-trim-event.png
[timeline_trim_subclip]: ./media/filters-dynamic-manifests/media-services-timeline-trim-subclip.png

[multiple-rules]:./media/filters-dynamic-manifest-overview/media-services-multiple-rules-filters.png

[subclip_filter]: ./media/filters-dynamic-manifest-overview/media-services-subclips-filter.png
[trim_event]: ./media/filters-dynamic-manifests/media-services-timeline-trim-event.png
[trim_subclip]: ./media/filters-dynamic-manifests/media-services-timeline-trim-subclip.png
[trim_filter]: ./media/filters-dynamic-manifest-overview/media-services-trim-filter.png
[redered_subclip]: ./media/filters-dynamic-manifests/media-services-rendered-subclip.png
[livebackoff_filter]: ./media/filters-dynamic-manifest-overview/media-services-livebackoff-filter.png
[language_filter]: ./media/filters-dynamic-manifest-overview/media-services-language-filter.png
[dvr_filter]: ./media/filters-dynamic-manifest-overview/media-services-dvr-filter.png
[skiing]: ./media/filters-dynamic-manifest-overview/media-services-skiing.png
[amp_diagnostics]: ./media/filters-dynamic-manifest-overview/amp_diagnostics.png
