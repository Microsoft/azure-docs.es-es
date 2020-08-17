---
title: Especificación de la introducción en vivo de MP4 fragmentado de Azure Media Services | Microsoft Docs
description: En esta especificación se describe el protocolo y el formato para ingesta de streaming en vivo basada en MP4 fragmentado para Azure Media Services. Este documento también describe las prácticas recomendadas para crear mecanismos de ingesta en vivo sólidos y de alta redundancia.
services: media-services
documentationcenter: ''
author: cenkdin
manager: femila
editor: ''
ms.assetid: 43fac263-a5ea-44af-8dd5-cc88e423b4de
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 9d0bfdf4719b4c3a92a0632a1edda63324d700e5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87072036"
---
# <a name="azure-media-services-fragmented-mp4-live-ingest-specification"></a>Especificación de la introducción en directo de MP4 fragmentado de Azure Media Services 

En esta especificación se describe el protocolo y el formato para ingesta de streaming en vivo basada en MP4 fragmentado para Azure Media Services. Azure Media Services proporciona un servicio de streaming en vivo que los clientes pueden utilizar para la transmisión de eventos en directo y la difusión de contenido en tiempo real mediante el uso de Azure como plataforma en la nube. Este documento también describe las prácticas recomendadas para crear mecanismos de ingesta en vivo sólidos y de alta redundancia.

## <a name="1-conformance-notation"></a>1. Notación de cumplimiento
Las palabras clave "DEBE", "NO DEBE", "SE REQUIERE", "TENDRÁ QUE", "NO DEBERÁ", "DEBERÍA", "NO DEBERÍA", "RECOMENDADO", "PUEDE" y "OPCIONAL" de este documento se deben interpretar como se describe en RFC 2119.

## <a name="2-service-diagram"></a>2. Diagrama del servicio
En el diagrama siguiente se muestra la arquitectura general del servicio de streaming en vivo de Media Services:

1. El codificador en directo inserta fuentes en canales que se crearon y aprovisionaron mediante el SDK de Azure Media Services.
1. Los canales, los programas y los puntos de conexión de streaming de Media Services controlan todas las funcionalidades de streaming en vivo incluidas la ingesta, el formato, DVR en la nube, la seguridad, la escalabilidad y la redundancia.
1. Opcionalmente, los clientes podrían optar por implementar una capa de Azure Content Delivery Network entre el punto de conexión de streaming y los puntos de conexión de cliente.
1. Los puntos de conexión de cliente hacen streaming desde el punto de conexión de streaming mediante protocolos de streaming adaptable sobre HTTP. Algunos ejemplos son Microsoft Smooth Streaming, Dynamic Adaptive Streaming sobre HTTP (DASH o MPEG-DASH) y Apple HTTP Live Streaming (HLS).

![flujo de ingestión][image1]

## <a name="3-bitstream-format--iso-14496-12-fragmented-mp4"></a>3. Formato de secuencia de bits: MP4 fragmentado según ISO 14496-12
El formato de ingesta de streaming en vivo que se describe en este documento se basa en [ISO-14496-12]. Consulte [[MS-SSTR]](https://msdn.microsoft.com/library/ff469518.aspx) para una explicación detallada del formato MP4 fragmentado y de las extensiones tanto para archivos de vídeo bajo demanda como para la ingesta de streaming en vivo.

### <a name="live-ingest-format-definitions"></a>Definiciones de formato de introducción en directo
A continuación se muestra una lista de definiciones de formato especial que se aplican a la ingestión en vivo en Azure Media Services:

1. Los cuadros **ftyp**, **Live Server Manifest Box** y **moov** DEBEN enviarse con cada solicitud (HTTP POST). Estos cuadros DEBEN enviarse al principio de la secuencia y cada vez que el codificador deba volver a conectarse para reanudar la ingestión de secuencias. Para más información, consulte la sección 6 en [1].
1. La sección 3.3.2 en [1] define un cuadro opcional denominado **StreamManifestBox** para la ingestión en vivo. Debido a la lógica de enrutamiento del equilibrador de carga de Azure, la utilización de este cuadro está en desuso. El cuadro de NO DEBERÍA estar presente para la ingestión en Media Services. Si el cuadro está presente, Media Services lo omite de modo silencioso.
1. El cuadro **TrackFragmentExtendedHeaderBox** definido en 3.2.3.2 en [1] DEBE estar presente en cada fragmento.
1. Se DEBERÍA usar la versión 2 del cuadro **TrackFragmentExtendedHeaderBox** para generar segmentos multimedia con direcciones URL idénticas en varios centros de datos. SE REQUIERE el campo de índice de fragmento para la conmutación por error entre centros de datos de formatos de streaming basados en índices como Apple HLS y MPEG-DASH basado en índices. Para habilitar la conmutación por error entre centros de datos, el índice de fragmentos DEBE sincronizarse entre varios codificadores y aumentar en 1 para cada fragmento multimedia sucesivo, incluso entre reinicios o errores del codificador.
1. En la sección 3.3.6 en [1] se define el cuadro denominado **MovieFragmentRandomAccessBox** (**mfra**) que se PUEDE enviar al final de la ingesta en directo para indicar EOS (final de secuencia) al canal. Debido a la lógica de ingestión de Media Services, el uso de EOS (final de la secuencia) está en desuso y el cuadro **mfra** NO DEBERÍA enviarse para la ingestión en vivo. Si se envía, Media Services lo omite de modo silencioso. Para restablecer el estado del punto de ingestión, se recomienda utilizar [Restablecer canales](/rest/api/media/operations/channel#reset_channels). También se recomienda que use [Detener programa](/rest/api/media/operations/program#stop_programs) para finalizar una presentación y la transmisión.
1. La duración del fragmento MP4 DEBERÍA ser constante para reducir el tamaño de los manifiestos de cliente. Una duración del fragmento MP4 constante también mejora la heurística de descarga de cliente mediante el uso de repetir las etiquetas. La duración PUEDE fluctuar para compensar velocidades de fotogramas no enteras.
1. La duración de fragmentos MP4 DEBERÍA estar comprendida aproximadamente entre 2 y 6 segundos.
1. Las marcas de tiempo y los índices de los fragmentos MP4 (**TrackFragmentExtendedHeaderBox**`fragment_ absolute_ time` y `fragment_index`) DEBERÍAN llegar en orden ascendente. Aunque Media Services es resistente a fragmentos duplicados, tiene una capacidad limitada para reordenar fragmentos de acuerdo con la escala de tiempo multimedia.

## <a name="4-protocol-format--http"></a>4. Formato de protocolo: HTTP
La ingestión en vivo basada en MP4 fragmentado de ISO para Media Services usa una solicitud HTTP POST de larga ejecución estándar para transmitir datos multimedia codificados empaquetados en formato MP4 fragmentado al servicio. Cada HTTP POST envía una secuencia de bits MP4 fragmentada completa ("secuencia"), desde el comienzo con cuadros de encabezado (cuadros **ftyp**, **Live Server Manifest Box** y **moov**) y continuando con una secuencia de fragmentos (cuadros **moof** y **mdat**). Para más información sobre la sintaxis URL de la solicitud HTTP POST, consulte la sección 9.2 en [1]. Un ejemplo de la dirección URL de POST es: 

`http://customer.channel.mediaservices.windows.net/ingest.isml/streams(720p)`

### <a name="requirements"></a>Requisitos
Estos son los requisitos detallados:

1. El codificador DEBERÍA empezar la difusión enviando una solicitud HTTP POST con un "cuerpo" vacío (longitud de contenido cero) con la misma URL de ingestión. Esto puede ayudar a detectar con rapidez si el punto de conexión de la ingesta en vivo es válido y si hay alguna autenticación u otras condiciones necesarias. Por protocolo HTTP, el servidor no podrá devolver una respuesta HTTP hasta que se reciba la solicitud completa incluido el cuerpo del POST. Dada la naturaleza de larga duración de un evento en directo, sin este paso,es posible que el codificador no sea capaz de detectar ningún error hasta que finalice el envío de todos los datos.
1. El codificador DEBE controlar los errores o los desafíos de autenticación como resultado de (1). Si (1) se realiza correctamente con una respuesta 200, continúe.
1. El codificador DEBE iniciar una nueva solicitud HTTP POST con la secuencia MP4 fragmentada. La carga DEBE empezar con los cuadros de encabezado seguidos por los fragmentos. Tenga en cuenta que los cuadros **ftyp**, **Live Server Manifest Box** y **moov** (en este orden) DEBEN enviarse con cada solicitud, incluso si el codificador debe volver a conectarse porque se canceló la solicitud anterior antes del final de la secuencia. 
1. El codificador DEBE usar una codificación de transferencia fragmentada para cargar, ya que es imposible predecir la longitud total del contenido del evento en directo.
1. Cuando se termina el evento, después de enviar el último fragmento, el codificador DEBE finalizar correctamente la secuencia de mensajes de la codificación de transferencia fragmentada (la mayoría de las pilas de cliente HTTP lo controlan automáticamente). El codificador DEBE esperar que el servicio devuelva el código de respuesta final y luego finalizar la conexión. 
1. El codificador NO DEBE usar el nombre `Events()` como se describe en 9.2 en [1] para la ingestión en vivo en Media Services.
1. Si la solicitud HTTP POST finaliza o se agota el tiempo con un error TCP antes del final de la secuencia, el codificador DEBE emitir una nueva solicitud POST con una conexión nueva y seguir los requisitos anteriores. Además, el codificador DEBE reenviar los dos fragmentos MP4 anteriores por cada pista en la secuencia y reanudar sin introducir una discontinuidad en la escala de tiempo multimedia. El reenvío de los dos últimos fragmentos de MP4 para cada pista garantiza que no hay ninguna pérdida de datos. En otras palabras, si una secuencia contiene tanto una pista de audio como una de vídeo y se produce un error en la solicitud POST actual, el codificador debe volver a conectarse y reenviar los últimos dos fragmentos de la pista de audio, que se enviaron correctamente anteriormente y los dos últimos fragmentos para la pista de vídeo, que se enviaron correctamente anteriormente, para asegurarse de que no hay ninguna pérdida de datos. El codificador DEBE mantener un búfer de “reenvío” de fragmentos multimedia, que vuelve a enviar al volver a conectarse.

## <a name="5-timescale"></a>5. Timescale
[[MS-SSTR]](https://msdn.microsoft.com/library/ff469518.aspx) describe el uso de la escala de tiempo para **SmoothStreamingMedia** (sección 2.2.2.1), **StreamElement** (sección 2.2.2.3), **StreamFragmentElement** (sección 2.2.2.6) y **LiveSMIL** (sección 2.2.7.3.1). Si el valor de la escala de tiempo no está presente, el valor predeterminado usado es 10.000.000 (10 MHz). Aunque la especificación de formato de Smooth Streaming no bloquea el uso de otros valores de escala de tiempo, la mayoría de las implementaciones del codificador usan este valor predeterminado (10 MHz) para generar ingestión de datos de Smooth Streaming. Debido a la característica de [empaquetado dinámico de Azure Media](./previous/media-services-dynamic-packaging-overview.md), se recomienda usar la escala de tiempo de 90 kHz para secuencias de vídeo y de 44,1 o 48,1 kHz para secuencias de audio. Si se usan valores de escala de tiempo diferentes para distintas secuencias, se DEBE enviar la escala de tiempo de nivel de secuencia. Para más información, consulte [[MS-SSTR]](https://msdn.microsoft.com/library/ff469518.aspx).     

## <a name="6-definition-of-stream"></a>6. Definición de "secuencia"
"Secuencia" es la unidad básica de operación en la ingesta en vivo para crear presentaciones en directo, controlar la conmutación por error de streaming y los escenarios de redundancia. "Secuencia" se define como una única secuencia de bits MP4 fragmentada que puede contener una sola pista o varias pistas. Una presentación en directo completa podría contener una o varias secuencias en función de la configuración de los codificadores en directo. En los ejemplos siguientes se ilustran varias opciones de uso de secuencias para crear una presentación en directo completa.

**Ejemplo**: 

El cliente desea crear una presentación de streaming en vivo que incluye las siguientes velocidades de bits de audio y vídeo:

Vídeo: 3000 kbps, 1500 kbps, 750 kbps

Audio: 128 kbps

### <a name="option-1-all-tracks-in-one-stream"></a>Opción 1: Todas las pistas de una secuencia
En esta opción, un único codificador genera todas las pistas de audio y vídeo y, a continuación, las agrupa en una secuencia de bits MP4 fragmentada. La secuencia de bits MP4 fragmentada, a continuación, se envía a través de una sola conexión HTTP POST. En este ejemplo, solo hay una secuencia para esta presentación en directo.

![Secuencias: una pista][image2]

### <a name="option-2-each-track-in-a-separate-stream"></a>Opción 2: Cada pista de una secuencia independiente
En esta opción, el codificador coloca una pista en cada secuencia de bits MP4 fragmentada y envía todas las secuencias a través de varias conexiones HTTP independientes. Esto se puede realizar con un codificador o con varios. Desde el punto de vista de la ingestión en vivo, esta presentación en directo se compone de cuatro secuencias.

![Secuencias: pistas independientes][image3]

### <a name="option-3-bundle-audio-track-with-the-lowest-bitrate-video-track-into-one-stream"></a>Opción 3: Agrupar pista de audio con la pista de vídeo de velocidad de bits más baja en una secuencia
En esta opción, el cliente elige agrupar la pista de audio con la pista de vídeo de velocidad de bits más baja en una secuencia de bits MP4 fragmentada y dejar las otras dos pistas de vídeo como secuencias independientes. 

![Secuencias: pistas de audio y vídeo][image4]

### <a name="summary"></a>Resumen
Esta no es una lista exhaustiva de todas las opciones de ingestión posibles para este ejemplo. De hecho, cualquier agrupación de pistas en secuencias es compatible con la ingesta en vivo. Los clientes y los proveedores de codificación pueden elegir sus propias implementaciones en función de la complejidad de ingeniería, de la capacidad del codificador, y de las consideraciones de redundancia y conmutación por error. Sin embargo, en la mayoría de los casos, hay solo una pista de audio para toda la presentación en directo. Por lo tanto, es importante garantizar el buen estado de la secuencia de ingestión que contiene la pista de audio. Esta consideración a menudo da como resultado la colocación de la pista de audio en su propia secuencia (como en la opción 2) o su agrupación con la pista de vídeo de velocidad de bits más baja (como en la opción 3). Además, para mejorar la redundancia y la tolerancia a errores, es muy recomendable enviar la misma pista de audio en dos secuencias diferentes (la opción 2 con pistas de audio redundantes) o agrupar la pista de audio con al menos dos de las pistas de vídeo de velocidad de bits más baja (opción 3 con audio agrupado en al menos dos transmisiones de vídeo) para la ingestión en vivo en Media Services.

## <a name="7-service-failover"></a>7. Conmutación por error del servicio
Dada la naturaleza del streaming en vivo, resulta fundamental una buena compatibilidad de conmutación por error para garantizar la disponibilidad del servicio. Media Services se ha diseñado para controlar los diversos tipos de errores, incluidos los errores de red, los errores de servidor y los problemas de almacenamiento. Cuando se usa junto con la lógica de conmutación por error adecuada desde el lado del codificador en directo, el cliente puede lograr un servicio de streaming en directo altamente confiable desde la nube.

En esta sección, trataremos los escenarios de conmutación por error del servicio. En este caso, el error se produce en algún lugar dentro del servicio y se manifiesta como un error de red. Estas son algunas recomendaciones para la implementación del codificador para controlar la conmutación por error del servicio:

1. Use un tiempo de espera de 10 segundos para establecer la conexión TCP. Si un intento de establecer la conexión tarda más de 10 segundos, anule la operación e inténtelo de nuevo. 
1. Use un tiempo de espera breve para enviar la solicitud HTTP de fragmentos de mensajes. Si la duración del fragmento MP4 de destino es N segundos, use un tiempo de espera de envío entre N y 2 N segundos; por ejemplo, use un tiempo de espera de 6 a 12 segundos si la duración del fragmento MP4 es de 6 segundos. Si se produce un tiempo de espera, restablezca la conexión, abra una nueva conexión y reanude la introducción de secuencias en la nueva conexión. 
1. Mantenga un búfer gradual con los dos últimos fragmentos para cada pista que se enviaron correctamente y por completo al servicio.  Si la solicitud HTTP POST de una secuencia finaliza o se agota el tiempo de espera antes del final de la secuencia, abra una nueva conexión y comience otra solicitud HTTP POST, reenvíe los encabezados de secuencia, reenvíe los últimos dos fragmentos para cada pista y reanude la secuencia sin introducir una discontinuidad en la escala de tiempo multimedia. Esto reduce la posibilidad de pérdida de datos.
1. Se recomienda que el codificador NO limite el número de reintentos para establecer una conexión ni reanude el streaming cuando se produce un error TCP.
1. Después de un error TCP:
  
    a. Se DEBE cerrar la conexión actual y se DEBE crear una nueva conexión para una nueva solicitud HTTP POST.

    b. La nueva URL HTTP POST DEBE ser la misma que la URL de POST inicial.
  
    c. El nuevo HTTP POST DEBE incluir encabezados de secuencia (cuadros **ftyp**, **Live Server Manifest Box** y **moov**) idénticos a los encabezados de secuencia del POST inicial.
  
    d. Se deben reenviar los dos últimos fragmentos enviados para cada pista y se debe reanudar el streaming sin introducir una discontinuidad en la escala de tiempo multimedia. Las marcas de tiempo de fragmento MP4 deben aumentar continuamente, incluso a través de solicitudes HTTP POST.
1. El codificador DEBE finalizar la solicitud HTTP POST si no se envíen datos a una velocidad acorde con la duración de fragmento MP4.  Una solicitud HTTP POST que no envía datos puede impedir que Media Services se desconecte rápidamente del codificador en el caso de una actualización del servicio. Por este motivo, el HTTP POST para pistas dispersas (señal de anuncio) DEBERÍA ser de actividad breve, finalizando tan pronto como se envíe el fragmento disperso.

## <a name="8-encoder-failover"></a>8. Conmutación por error del codificador
La conmutación por error del codificador es el segundo tipo de escenario de conmutación por error que se debe tratar para la entrega de streaming en directo integral. En este escenario, la condición de error se produce en el lado del codificador. 

![conmutación por error del codificador][image5]

A continuación, se muestran las expectativas del punto de conexión de ingestión en vivo cuando se produce la conmutación por error del codificador:

1. Se DEBERÍA crear una nueva instancia del codificador para continuar con el streaming, como se muestra en el diagrama (secuencia para vídeo de 3000 k con línea discontinua).
1. El nuevo codificador DEBE usar la misma URL para las solicitudes HTTP POST que la instancia con errores.
1. La solicitud POST del nuevo codificador DEBE incluir los mismos cuadros de encabezado MP4 fragmentados que la instancia con errores.
1. El codificador nuevo DEBE estar sincronizado correctamente con todos los demás codificadores en ejecución para la misma presentación en directo para generar muestras de audio y vídeo sincronizadas con los límites de fragmentos alineados.
1. La nueva secuencia DEBE ser semánticamente equivalente a la secuencia anterior e intercambiable en el nivel de encabezado y fragmento.
1. El nuevo codificador DEBERÍA intentar minimizar la pérdida de datos. Los valores de `fragment_absolute_time` y `fragment_index` de los fragmentos multimedia DEBERÍAN aumentar desde el punto en el que se detuvo por última vez el codificador. `fragment_absolute_time` y `fragment_index` DEBERÍAN aumentar de forma continua, pero se puede introducir una discontinuidad en caso necesario. Media Services omite los fragmentos que ya ha recibido y procesado, por lo que es mejor equivocarse en el lado del reenvío de los fragmentos en lugar de introducir discontinuidades en la escala de tiempo mutimedia. 

## <a name="9-encoder-redundancy"></a>9. Redundancia del codificador
Para determinados eventos en directo críticos que demandan una disponibilidad incluso mayor y una calidad de experiencia, se recomienda emplear codificadores redundantes activo-activo para lograr una conmutación por error perfecta sin pérdida de datos.

![redundancia del codificador][image6]

Como se ilustra en el diagrama, hay dos grupos de codificadores que insertan dos copias de cada secuencia de manera simultánea en el servicio en directo. Esta configuración se admite porque Media Services puede filtrar fragmentos duplicados según el identificador de secuencia y la marca de tiempo del fragmento. La secuencia en directo resultante y el archivo es una copia única de todas las secuencias que es la mejor agregación posible de los dos orígenes. Por ejemplo, en un caso extremo hipotético, siempre y cuando haya un codificador (no tiene que ser el mismo) en ejecución en un momento dado en el tiempo para cada secuencia, la secuencia en directo resultante desde el servicio será continua sin pérdida de datos. 

Los requisitos para este escenario son prácticamente los mismos que los requisitos en el caso de "conmutación por error del codificador" con la excepción de que el segundo conjunto de codificadores se ejecutan al mismo tiempo que los codificadores principales.

## <a name="10-service-redundancy"></a>10. Redundancia del servicio
Para distribución global altamente redundante, a veces es necesario tener copia de seguridad en varias regiones para controlar los desastres regionales. Expandiendo la topología de la "Redundancia del codificador", los clientes pueden elegir tener una implementación de servicio redundante en una región distinta conectada con el segundo conjunto de codificadores. Los clientes también podrían trabajar con un proveedor de Content Delivery Network para implementar un administrador de Traffic Manager delante de las dos implementaciones del servicio con el fin de enrutar el tráfico de cliente sin problemas. Los requisitos para los codificadores son los mismos que en el caso de "Redundancia del codificador". La única excepción es que el segundo conjunto de codificadores debe apuntar a un punto de conexión de ingestión en vivo diferente. El siguiente diagrama muestra esta configuración:

![redundancia del servicio][image7]

## <a name="11-special-types-of-ingestion-formats"></a>11. Tipos especiales de formatos de ingesta
En esta sección se describen los tipos especiales de formatos de ingesta en vivo que están diseñados para administrar escenarios concretos.

### <a name="sparse-track"></a>Pista dispersa
Cuando se entrega una presentación de streaming en vivo con la experiencia de cliente enriquecida, a menudo es necesario transmitir eventos sincronizados de tiempo o señales en banda con los datos multimedia principales. Un ejemplo de esto es la inserción de anuncios en directo dinámicos. Este tipo de señalización de eventos es diferente del streaming de audio y vídeo regular debido a su naturaleza dispersa. Es decir, los datos de señalización no se suelen producir de manera continua y el intervalo puede ser difícil de predecir. El concepto de pista dispersa se diseñó para la ingestión y difusión de datos de señalización en banda.

Los pasos siguientes reflejan una implementación recomendada para la ingestión de pistas dispersas:

1. Cree una secuencia de bits MP4 fragmentada independiente que solo contenga pistas dispersas, sin pistas de audio y vídeo.
1. En el cuadro **Live Server Manifest Box**, según se define en la sección 6 en [1], use el parámetro *parentTrackName* para especificar el nombre de la pista principal. Para más información, consulte la sección 4.2.1.2.1.2 en [1].
1. En el cuadro **Live Server Manifest Box**, el valor de **manifestOutput** DEBE establecerse en **true**.
1. Dada la naturaleza dispersa del evento de señalización, se recomienda lo siguiente:
   
    a. Al principio del evento en directo, el codificador envía los cuadros de encabezados iniciales al servicio, lo que permite que el servicio registre la pista dispersa en el manifiesto del cliente.
   
    b. El codificador DEBERÍA finalizar la solicitud HTTP POST cuando no se están enviando datos. Una solicitud HTTP POST de larga duración que no envía datos puede impedir que Media Services se desconecte rápidamente del codificador en el caso de una actualización del servicio o reinicio del servidor. En estos casos, el servidor multimedia está bloqueado temporalmente en una operación de recepción en el socket.
   
    c. Durante el tiempo en el que no estén disponibles los datos de señalización, el codificador DEBERÍA cerrar la solicitud HTTP POST. Mientras la solicitud POST está activa, el codificador DEBERÍA enviar datos.

    d. Al enviar fragmentos dispersos, el codificador puede establecer un encabezado content-length explícito si está disponible.

    e. Al enviar fragmentos dispersos con una nueva conexión, el codificador DEBERÍA empezar a enviar desde los cuadros de encabezado, seguidos de los fragmentos nuevos. Se utiliza en casos donde la conmutación por error se produce entre tanto y se está estableciendo la nueva conexión dispersa a un nuevo servidor que no ha visto antes la pista dispersa.

    f. El fragmento de pista dispersa está disponible para el cliente cuando el fragmento de pista principal correspondiente que tenga un valor de marca de tiempo igual o posterior esté disponible para el cliente. Por ejemplo, si el fragmento disperso tiene una marca de tiempo de t=1000, se espera que después de que el cliente vea la marca de tiempo del fragmento "video" 1000 o superior (suponiendo que el nombre de la pista principal es "video"), puede descargar el fragmento disperso t=1000. Tenga en cuenta que la señal real podría usarse para una posición diferente en la escala de tiempo de la presentación para su propósito designado. En este ejemplo, es posible que el fragmento disperso de t=1000 tenga una carga XML para insertar un anuncio en una posición que es unos segundos posterior.

    g. La carga de fragmentos de pista dispersos puede estar en diferentes formatos (por ejemplo, XML, texto o binario) según el escenario.

### <a name="redundant-audio-track"></a>Pista de audio redundante
En un escenario de streaming adaptable HTTP típico (por ejemplo, Smooth Streaming o DASH), a menudo solo hay una pista de audio en toda la presentación. A diferencia de las pistas de vídeo, que tienen varios niveles de calidad para que el cliente elija a partir de las condiciones de error, la pista de audio puede ser un punto de error único si se interrumpe la ingestión de la secuencia que contiene la pista de audio. 

Para solucionar este problema, Media Services admite la ingesta en vivo de las pistas de audio redundantes. La idea es que la misma pista de audio se puede enviar varias veces en secuencias diferentes. Aunque el servicio solo registra la pista de audio una vez en el manifiesto del cliente, puede usar pistas de audio redundantes como copias de seguridad para recuperar fragmentos de audio si la pista de audio principal tiene problemas. Para ingerir pistas de audio redundantes, el codificador debe:

1. Crear la misma pista de audio en varios fragmentos de secuencias de bits MP4. Las pistas de audio redundantes DEBEN ser semánticamente equivalentes, con el mismo número de marcas de tiempo de fragmento e intercambiables en el nivel de encabezado y fragmento.
1. Asegúrese de que la entrada de "audio" en el Live Server Manifest (sección 6 en [1]) es la misma para todas las pistas de audio redundantes.

A continuación se muestra una implementación recomendada para pistas de audio redundantes:

1. Envíe cada pista de audio única en una secuencia de manera individual. Envíe además una secuencia redundante para cada una de estas secuencias de pista de audio, donde la segunda secuencia sea diferente de la primera únicamente en el identificador de la URL de HTTP POST: {protocolo}://{dirección de servidor}/{ruta de acceso a punto de publicación}/Streams({identificador}).
1. Use secuencias independientes para enviar las dos velocidades de bits de vídeos más bajas. Cada una de estas secuencias también DEBERÍA contener una copia de cada pista de audio única. Por ejemplo, cuando se admiten varios idiomas, estas secuencias DEBERÍAN contener pistas de audio para cada idioma.
1. Use instancias de servidor independientes (de codificador) y envíe las secuencias redundantes mencionadas en (1) y (2). 

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

[image1]: ./media/media-services-fmp4-live-ingest-overview/media-services-image1.png
[image2]: ./media/media-services-fmp4-live-ingest-overview/media-services-image2.png
[image3]: ./media/media-services-fmp4-live-ingest-overview/media-services-image3.png
[image4]: ./media/media-services-fmp4-live-ingest-overview/media-services-image4.png
[image5]: ./media/media-services-fmp4-live-ingest-overview/media-services-image5.png
[image6]: ./media/media-services-fmp4-live-ingest-overview/media-services-image6.png
[image7]: ./media/media-services-fmp4-live-ingest-overview/media-services-image7.png
