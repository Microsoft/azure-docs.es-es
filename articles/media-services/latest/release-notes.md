---
title: Notas de la versión de Azure Media Services v3 | Microsoft Docs
description: Para mantenerse al día con los últimos desarrollos, en este artículo se proporcionan las actualizaciones más reciente en Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: na
ms.topic: article
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 5a22bd9508feac1348bcd8042fa6ac791864c261
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89425643"
---
# <a name="azure-media-services-v3-release-notes"></a>Notas de la versión de Azure Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

>Reciba notificaciones para volver a visitar esta página y obtener actualizaciones; para ello, copie y pegue esta URL (`https://docs.microsoft.com/api/search/rss?search=%22Azure+Media+Services+v3+release+notes%22&locale=en-us`) en el lector de fuentes RSS.

Para mantenerse al día con los avances más recientes, este artículo proporciona información acerca de los elementos siguientes:

* Versiones más recientes
* Problemas conocidos
* Corrección de errores
* Funciones obsoletas

## <a name="known-issues"></a>Problemas conocidos

> [!NOTE]
> Puede usar [Azure Portal](https://portal.azure.com/) para administrar los [eventos en directo](live-events-outputs-concept.md) de la versión 3, ver los [recursos](assets-concept.md) y trabajos de la versión 3, obtener información sobre el acceso a las API y cifrar contenido. Para las tareas de administración restantes (por ejemplo, la administración de transformaciones y trabajos), use la [API REST](https://aka.ms/ams-v3-rest-ref), la [CLI](https://aka.ms/ams-v3-cli-ref), o uno de los [SDK](media-services-apis-overview.md#sdks) compatibles.
>
> Para más detalles, consulte [las limitaciones de Azure Portal para Media Services v3](frequently-asked-questions.md#what-are-the-azure-portal-limitations-for-media-services-v3).


## <a name="august-2020"></a>Agosto de 2020

### <a name="dynamic-encryption"></a>Cifrado dinámico
La compatibilidad con el cifrado Protected Interoperable File Format (PIFF 1.1) de PlayReady heredado ya está disponible en el empaquetador dinámico. Proporciona compatibilidad con los televisores inteligentes heredados de Samsung y LG que implementaron los borradores iniciales del estándar Common Encryption (CENC) publicado por Microsoft.  El formato PIFF 1.1 también se conoce como el formato de cifrado admitido anteriormente por la biblioteca cliente de Silverlight. En la actualidad, el único caso de uso para este formato de cifrado es la segmentación del mercado de los televisores inteligentes heredados, donde todavía existen una cantidad considerable de televisores inteligentes en algunas regiones que solo admiten Smooth Streaming con el cifrado PIFF 1.1. 

Para usar la compatibilidad con el nuevo cifrado PIFF 1.1, cambie el valor de cifrado a "piff" en la ruta de acceso de la dirección URL del localizador de streaming. Para obtener más información, vea [Introducción a Content Protection](content-protection-overview.md).
Por ejemplo: `https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(encryption=piff)`|

> [!NOTE]
> La compatibilidad con PIFF 1.1 se proporciona como una solución compatible con versiones anteriores de televisores inteligentes (Samsung y LG) que implementó la versión "Silverlight" anterior de Common Encryption. Se recomienda usar solo el formato PIFF cuando sea necesario para la compatibilidad con los televisores inteligentes Samsung o LG heredados vendidos entre 2009-2015 compatibles con la versión PIFF 1.1 del cifrado de PlayReady. 

## <a name="july-2020"></a>Julio de 2020

### <a name="live-transcriptions"></a>Transcripciones en vivo

Las transcripciones en vivo ahora admiten 19 idiomas y 8 regiones.

### <a name="protecting-your-content-with-media-services-and-azure-ad"></a>Protección del contenido con Media Services y Azure AD

Publicamos un tutorial denominado [Protección de contenido de un extremo a otro con Azure AD](./azure-ad-content-protection.md).

### <a name="high-availablity"></a>Alta disponibilidad

Publicamos una [introducción](./media-services-high-availability-encoding.md) y un [ejemplo](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/master/HighAvailabilityEncodingStreaming) de alta disponibilidad con Media Services y Vídeo bajo demanda (VoD).

## <a name="june-2020"></a>Junio de 2020

### <a name="live-video-analytics-on-iot-edge-preview-release"></a>Live Video Analytics on IoT Edge (versión preliminar)

Ya se publicó la versión preliminar de Live Video Analytics on IoT Edge. Para más información, consulte las [notas de la versión](../live-video-analytics-edge/release-notes.md).

Live Video Analytics on IoT Edge es una expansión de la familia de Media Services. Le permite analizar vídeos en directo con los modelos de inteligencia artificial que prefiera en sus propios dispositivos perimetrales y, opcionalmente, capturar y grabar ese vídeo. Ahora puede crear aplicaciones con análisis de vídeo en tiempo real en el perímetro sin preocuparse por la complejidad de crear y operar una canalización de vídeo en directo.

## <a name="may-2020"></a>Mayo de 2020

Azure Media Services ya está disponible con carácter general en las regiones siguientes: "Norte de Alemania", "Centro-oeste de Alemania", "Norte de Suiza" y "Oeste de Suiza". Los clientes pueden implementar Media Services en estas regiones mediante Azure Portal.

## <a name="april-2020"></a>Abril de 2020

### <a name="improvements-in-documentation"></a>Mejoras en la documentación

La documentación de Azure Media Player se migró a la [documentación de Azure](../azure-media-player/azure-media-player-overview.md).

## <a name="january-2020"></a>Enero de 2020

### <a name="improvements-in-media-processors"></a>Mejoras en los procesadores de multimedia

- Compatibilidad mejorada con orígenes entrelazados en el análisis de vídeo: se ha eliminado correctamente el entrelazado de esos contenidos antes de enviarse a los motores de inferencia.
- Ahora, al generar miniaturas con el modo "Mejor", el codificador busca durante más de 30 segundos para seleccionar un fotograma que no sea monocromático.

### <a name="azure-government-cloud-updates"></a>Actualizaciones de la nube de Azure Government

Media Services está disponible con carácter temporal en las siguientes regiones de Azure Government: *USGov Arizona* y *USGov Texas*.

## <a name="december-2019"></a>Diciembre de 2019

Se agregó compatibilidad de la red CDN con los encabezados *Origin-Assist Prefetch* para streaming a petición tanto en directo como en vídeo; está disponible para clientes que tienen un contrato directo con CDN de Akamai. La característica Origin-Assist CDN-Prefetch supone los siguientes intercambios de encabezados HTTP entre CDN de Akamai y el origen de Azure Media Services:

|Encabezado HTTP|Valores|Remitente|Receptor|Propósito|
| ---- | ---- | ---- | ---- | ----- |
|CDN-Origin-Assist-Prefetch-Enabled | 1 (valor predeterminado) o 0 |CDN|Origen|Para indicar que la red CDN está habilitada para la captura previa.|
|CDN-Origin-Assist-Prefetch-Path| Ejemplo: <br/>Fragments(video=1400000000,format=mpd-time-cmaf)|Origen|CDN|Para proporcionar la ruta de acceso de captura previa a la red CDN.|
|CDN-Origin-Assist-Prefetch-Request|1 (solicitud de captura previa) o 0 (solicitud normal)|CDN|Origen|Para indicar que la solicitud de CDN es una captura previa.|

Para ver en acción parte del intercambio de encabezados, puede probar los pasos siguientes:

1. Use Postman o Curl para emitir una solicitud de un segmento o fragmento de audio o vídeo al origen de Media Services. Asegúrese de agregar el encabezado CDN-Origin-Assist-Prefetch-Enabled: 1 en la solicitud.
2. En la respuesta, debería ver el encabezado CDN-Origin-Assist-Prefetch-Path con una ruta de acceso relativa como su valor.

## <a name="november-2019"></a>Noviembre de 2019

### <a name="live-transcription-preview"></a>Transcripción en directo (versión preliminar)

La transcripción en directo está ahora en versión preliminar pública y disponible para su uso en la región Oeste de EE. UU. 2.

Está diseñada para funcionar en combinación con eventos en directo como una funcionalidad complementaria.  Es compatible con eventos en directo de codificación estándar y premium de paso a través.  Cuando esta característica está habilitada, el servicio usa la característica [Voz a texto](../../cognitive-services/speech-service/speech-to-text.md) de Cognitive Services para transcribir el texto oral del audio entrante en texto escrito. A continuación, se pone a disposición este texto para su entrega junto con el vídeo y el audio en los protocolos MPEG-DASH y HLS. La facturación se basa en un nuevo medidor complementario que supone un costo adicional para el evento en directo cuando está en estado "en ejecución".  Para más información sobre la transcripción en directo y la facturación, consulte [Transcripción en directo](live-transcription.md).

> [!NOTE]
> Actualmente, la transcripción en directo solo está disponible como una característica en vista previa en la región Oeste de EE. UU. 2. En este momento, solo admite la transcripción de texto oral en inglés (en-US).

### <a name="content-protection"></a>Protección de contenido

La característica de *prevención de reproducción de tokens* publicada en septiembre en algunas regiones está ahora disponible en todas las regiones.
Los clientes de Media Services ahora pueden establecer un límite en el número de veces que se puede usar el mismo token para solicitar una clave o una licencia. Para obtener más información, consulte [Prevención de reproducción de tokens](content-protection-overview.md#token-replay-prevention).

### <a name="new-recommended-live-encoder-partners"></a>Nuevos asociados de codificador en directo recomendados

Se ha agregado compatibilidad con los siguientes nuevos asociados de codificador recomendados para el streaming en vivo RTMP:

- [Cambria Live 4.3](https://www.capellasystems.net/products/cambria-live/)
- [Cámaras de acción GoPro Hero7/8 y Max](https://gopro.com/help/articles/block/getting-started-with-live-streaming)
- [Restream.io](https://restream.io/)

### <a name="file-encoding-enhancements"></a>Mejoras en la codificación de archivos

- Ahora hay disponible un nuevo valor preestablecido de codificación compatible con el contenido. Genera un conjunto de archivos MP4 con alineación GOP mediante la codificación compatible con el contenido. Dado cualquier contenido de entrada, el servicio realiza un análisis ligero inicial del contenido de entrada. Utiliza esos resultados para determinar automáticamente el número óptimo de capas, la velocidad de bits adecuada y la configuración de resolución para la entrega a través del streaming adaptable. Este valor predefinido resulta particularmente eficaz en los vídeos de complejidad media y baja, donde los archivos de salida tendrán velocidades de bits más lentas, pero una calidad que seguirá ofreciendo una buena experiencia a los espectadores. La salida contendrá archivos MP4 con el vídeo y audio intercalados. Para obtener más información, consulte las [especificaciones de API abiertas](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01/Encoding.json).
- Rendimiento mejorado y subprocesos múltiples en el cambio de tamaño de Standard Encoder. En condiciones específicas, el cliente apreciará un aumento del rendimiento de la codificación VOD entre un 5 % y un 40 %. El contenido de poca complejidad codificado en varias velocidades de bits experimentará los mayores aumentos del rendimiento. 
- La codificación estándar ahora mantiene una cadencia normal del GOP para el contenido de velocidad de fotogramas variable (VFR) durante la codificación VOD cuando se usa el valor GOP basado en el tiempo.  Esto significa que, por ejemplo, el cliente que envía contenido de velocidad de fotogramas mixta que varía entre 15 y 30 fps observará ahora distancias de GOP normales calculadas en la salida a archivos MP4 de streaming con velocidad de bits adaptable. Este hecho mejorará la posibilidad de cambiar sin problemas entre pistas al realizar las entregas a través de HLS o DASH. 
-  Sincronización de AV mejorada en el contenido de origen de velocidad de fotogramas variable (VFR)

### <a name="video-indexer-video-analytics"></a>Video Indexer, análisis de vídeo

- Los fotogramas clave extraídos mediante el valor preestablecido de VideoAnalyzer ahora se encuentran en la resolución original del vídeo en lugar de cambiarse de tamaño. La extracción de fotogramas clave de alta resolución proporciona imágenes de calidad original y permite usar los modelos de inteligencia artificial basados en imágenes proporcionados por los servicios Microsoft Computer Vision y Custom Vision para obtener aún más información del vídeo.

## <a name="september-2019"></a>Septiembre de 2019

###  <a name="media-services-v3"></a>Media Services v3  

#### <a name="live-linear-encoding-of-live-events"></a>Codificación lineal de eventos en directo

Media Services V3 anuncia la versión preliminar de la codificación lineal de eventos en directo durante 24 horas al día los 365 días del año.

###  <a name="media-services-v2"></a>Media Services v2  

#### <a name="deprecation-of-media-processors"></a>Desuso de los procesadores de multimedia

Estamos anunciando el desuso de *Azure Media Indexer* y *Azure Media Indexer 2 Preview*. Para ver las fechas de retirada, consulte el tema sobre [componentes heredados](../previous/legacy-components.md). [Video Indexer de Azure Media Services ](../video-indexer/index.yml) reemplaza a estos procesadores multimedia heredados.

Para más información, consulte [Migración de Azure Media Indexer y Azure Media Indexer 2 a Video Indexer de Azure Media Services](../previous/migrate-indexer-v1-v2.md).

## <a name="august-2019"></a>Agosto de 2019

###  <a name="media-services-v3"></a>Media Services v3  

#### <a name="south-africa-regional-pair-is-open-for-media-services"></a>El par regional de Sudáfrica está abierto para Media Services 

Media Services ya está disponible en las regiones Norte de Sudáfrica y Oeste de Sudáfrica.

Si desea obtener más información, vea [Nubes y regiones donde existe Azure Media Services v3](azure-clouds-regions.md).

###  <a name="media-services-v2"></a>Media Services v2  

#### <a name="deprecation-of-media-processors"></a>Desuso de los procesadores de multimedia

Anunciamos el desuso de los procesadores de multimedia *Windows Azure Media Encoder* (WAME) y *Azure Media Encoder* (AME), que se van a retirar. Para ver las fechas de retirada, consulte el tema [componentes heredados](../previous/legacy-components.md).

Para más información, consulte [Migración de WAME a Media Encoder Standard](https://go.microsoft.com/fwlink/?LinkId=2101334) y [Migración de AME a Media Encoder Standard](https://go.microsoft.com/fwlink/?LinkId=2101335).
 
## <a name="july-2019"></a>Julio de 2019

### <a name="content-protection"></a>Protección de contenido

Cuando el contenido de streaming está protegido con restricción de token, los usuarios finales deben obtener un token que se envía como parte de la solicitud de entrega de claves. La característica de *prevención de reproducción de tokens* permite a los clientes de Media Services establecer un límite en el número de veces que se puede usar el mismo token para solicitar una clave o una licencia. Para obtener más información, consulte [Prevención de reproducción de tokens](content-protection-overview.md#token-replay-prevention).

Desde julio, la característica de vista previa solo estaba disponible en las regiones Centro de EE. UU. y Centro y oeste de EE. UU.

## <a name="june-2019"></a>Junio de 2019

### <a name="video-subclipping"></a>Creación de subclips de vídeo

Ahora puede recortar un vídeo, o crear un subclip de vídeo al codificarlo mediante la opción [Trabajo](/rest/api/media/jobs). 

Esta funcionalidad se puede usar con cualquier elemento [Transformación](/rest/api/media/transforms) compilado mediante los valores preestablecidos [BuiltInStandardEncoderPreset](/rest/api/media/transforms/createorupdate#builtinstandardencoderpreset) o [StandardEncoderPreset](/rest/api/media/transforms/createorupdate#standardencoderpreset). 

Ver ejemplos:

* [Creación de un subclip de vídeo con .NET](subclip-video-dotnet-howto.md)
* [Creación de un subclip de vídeo con REST](subclip-video-rest-howto.md)

## <a name="may-2019"></a>Mayo de 2019

### <a name="azure-monitor-support-for-media-services-diagnostic-logs-and-metrics"></a>Soporte técnico de Azure Monitor para las métricas y los registros de diagnóstico de Media Services

Ya se puede usar Azure Monitor para ver los datos de telemetría emitidos por Media Services.

* Use los registros de diagnóstico de Azure Monitor para supervisar las solicitudes enviadas por el punto de conexión de la entrega de claves de Media Services. 
* Supervise las métricas emitidas por los [punto de conexión de streaming](streaming-endpoint-concept.md) de Media Services.   

Para obtener más información, consulte [Supervisión de los registros de diagnóstico y las métricas de Media Services](media-services-metrics-diagnostic-logs.md).

### <a name="multi-audio-tracks-support-in-dynamic-packaging"></a>Compatibilidad con varias pistas de audio en el empaquetado dinámico 

Al realizar el streaming de recursos que tienen varias pistas de audio con varios códecs y lenguajes, el [empaquetado dinámico](dynamic-packaging-overview.md) ahora admite varias pistas de audio para la salida HLS (versión 4 o superior).

### <a name="korea-regional-pair-is-open-for-media-services"></a>El par regional de Corea está abierto para Media Services 

Media Services ya está disponible en las regiones Centro de Corea del Sur y Sur de Corea del Sur. 

Si desea obtener más información, vea [Nubes y regiones donde existe Azure Media Services v3](azure-clouds-regions.md).

### <a name="performance-improvements"></a>Mejoras en el rendimiento

Se han agregado actualizaciones que incluyen mejoras de rendimiento de Media Services.

* Se actualizó el tamaño de archivo máximo admitido para el procesamiento. Consulte [Cuotas y límites](limits-quotas-constraints.md).
* [Mejoras de velocidades de codificación](media-reserved-units-cli-how-to.md#choosing-between-different-reserved-unit-types).

## <a name="april-2019"></a>Abril de 2019

### <a name="new-presets"></a>Nuevos valores preestablecidos

* [FaceDetectorPreset](/rest/api/media/transforms/createorupdate#facedetectorpreset) se agregó a los valores preestablecidos del analizador integrado.
* [ContentAwareEncodingExperimental](/rest/api/media/transforms/createorupdate#encodernamedpreset) se agregó a los valores preestablecidos del codificador integrado. Para más información, consulte [Codificación que tiene en cuenta el contenido](content-aware-encoding.md). 

## <a name="march-2019"></a>Marzo de 2019

Ahora el empaquetado dinámico admite Dolby Atmos. Para más información, consulte [Códecs de audio compatibles con el empaquetado dinámico](dynamic-packaging-overview.md#audio-codecs-supported-by-dynamic-packaging).

Ahora puede especificar una lista de filtros de recursos o cuentas, que se aplicarían a su localizador de streaming. Para más información, consulte el tema sobre la [asociación de filtros al localizador de streaming](filters-concept.md#associating-filters-with-streaming-locator).

## <a name="february-2019"></a>Febrero de 2019

Media Services v3 ya se admite en las nubes nacionales de Azure. Aún no todas las características están disponibles en todas las nubes. Para obtener más detalles, consulte [Nubes y regiones donde existe Azure Media Services v3](azure-clouds-regions.md).

El evento [Microsoft.Media.JobOutputProgress](media-services-event-schemas.md#monitoring-job-output-progress) se ha agregado a los esquemas de Azure Event Grid para Media Services.

## <a name="january-2019"></a>Enero de 2019

### <a name="media-encoder-standard-and-mpi-files"></a>Media Encoder Standard y archivos MPI 

Al codificar con Media Encoder Standard para generar archivos MP4, se genera un archivo .mpi nuevo y se agrega a la salida de activos. Este archivo MPI está diseñado para mejorar el rendimiento de escenarios de streaming y [empaquetado dinámico](dynamic-packaging-overview.md).

No debe modificar ni quitar el archivo MPI, así como tampoco tener ninguna dependencia en el servicio en la existencia (o no) de este tipo de archivo.

## <a name="december-2018"></a>Diciembre de 2018

Las actualizaciones de la versión de disponibilidad general de la API de V3 incluyen:
       
* Las propiedades de **PresentationTimeRange** ya no son necesarias para los **filtros de recursos** ni para los **filtros de cuenta**. 
* Las opciones de la consultas $top y $skip para **Jobs** y **Transforms** se han quitado y se ha agregado $orderby. Como parte de agregar la nueva funcionalidad de ordenación, se ha detectado que las opciones $top y $skip se expusieron previamente de forma accidental aunque no se habían implementado.
* La extensibilidad de enumeración se había activado de nuevo. Esta característica estaba habilitada en las versiones preliminares del SDK y se deshabilitó accidentalmente en la versión de disponibilidad general.
* Se ha cambiado el nombre de dos directivas de streaming predefinidas. **SecureStreaming** es ahora **MultiDrmCencStreaming**. **SecureStreamingWithFairPlay** es ahora **Predefined_MultiDrmStreaming**.

## <a name="november-2018"></a>Noviembre de 2018

El módulo de la CLI 2.0 está ahora disponible para [Azure Media Services v3 con disponibilidad general](/cli/azure/ams?view=azure-cli-latest) – v 2.0.50.

### <a name="new-commands"></a>Nuevos comandos

- [az ams account](/cli/azure/ams/account?view=azure-cli-latest)
- [az ams account-filter](/cli/azure/ams/account-filter?view=azure-cli-latest)
- [az ams asset](/cli/azure/ams/asset?view=azure-cli-latest)
- [az ams asset-filter](/cli/azure/ams/asset-filter?view=azure-cli-latest)
- [az ams content-key-policy](/cli/azure/ams/content-key-policy?view=azure-cli-latest)
- [az ams job](/cli/azure/ams/job?view=azure-cli-latest)
- [az ams live-event](/cli/azure/ams/live-event?view=azure-cli-latest)
- [az ams live-output](/cli/azure/ams/live-output?view=azure-cli-latest)
- [az ams streaming-endpoint](/cli/azure/ams/streaming-endpoint?view=azure-cli-latest)
- [az ams streaming-locator](/cli/azure/ams/streaming-locator?view=azure-cli-latest)
- [az ams account mru](/cli/azure/ams/account/mru?view=azure-cli-latest): permite administrar unidades reservadas de multimedia. Para más información, consulte [Escalado de unidades reservadas de multimedia](media-reserved-units-cli-how-to.md).

### <a name="new-features-and-breaking-changes"></a>Nuevas características y cambios importantes

#### <a name="asset-commands"></a>Comandos de recursos

- Se han agregado los argumentos ```--storage-account``` y ```--container```.
- Se han agregado valores predeterminados para la hora de expiración (ahora +23 h) y los permisos (lectura) al comando ```az ams asset get-sas-url```.

#### <a name="job-commands"></a>Comandos de trabajos

- Se han agregado los argumentos ```--correlation-data``` y ```--label```.
- ```--output-asset-names``` cambia su nombre a ```--output-assets```. Ahora acepta una lista separada por espacios de recursos en formato "assetName=label". Se puede enviar un recurso sin etiqueta del siguiente modo: "assetName=".

#### <a name="streaming-locator-commands"></a>Comandos de localizador de streaming

- Se ha reemplazado el comando base ```az ams streaming locator``` por ```az ams streaming-locator```.
- Se han agregado los argumentos ```--streaming-locator-id``` y ```--alternative-media-id support```.
- Se ha actualizado el argumento ```--content-keys argument```.
- ```--content-policy-name``` cambia su nombre a ```--content-key-policy-name```.

#### <a name="streaming-policy-commands"></a>Comandos de directiva de streaming

- Se ha reemplazado el comando base ```az ams streaming policy``` por ```az ams streaming-policy```.
- Se ha agregado compatibilidad con los parámetros de cifrado en ```az ams streaming-policy create```.

#### <a name="transform-commands"></a>Comandos de transformación

- Se ha reemplazado el argumento ```--preset-names``` por ```--preset```. Ahora solo puede establecer una salida o valor preestablecido cada vez (para agregar más tendrá que ejecutar ```az ams transform output add```). Ademas, puede pasar la ruta de acceso al código JSON personalizado para establecer un StandardEncoderPreset personalizado.
- ```az ams transform output remove``` se puede realizar pasando el índice de salida a eliminar.
- Se han agregado argumentos ```--relative-priority, --on-error, --audio-language and --insights-to-extract``` en los comandos ```az ams transform create``` y ```az ams transform output add```.

## <a name="october-2018---ga"></a>Octubre de 2018: disponibilidad general

En esta sección se describen las actualizaciones de octubre de Azure Media Services (AMS).

### <a name="rest-v3-ga-release"></a>Versión de disponibilidad general de REST v3

La [versión de disponibilidad general de REST v3](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01) incluye otras API de Live, filtros de manifiesto de nivel de cuenta y recurso y compatibilidad con DRM.

#### <a name="azure-resource-management"></a>Administración de recursos de Azure 

La compatibilidad con la administración de recursos de Azure habilita la API de operaciones y administración unificada (ahora todo en un solo lugar).

A partir de esta versión, puede usar plantillas de Resource Manager para crear eventos en vivo.

#### <a name="improvement-of-asset-operations"></a>Mejora de las operaciones de recurso 

Se han introducido las siguientes mejoras:

- Ingesta de direcciones URL de HTTP o direcciones URL de SAS de Azure Blob Storage.
- Especifique sus propios nombres de contenedor para los recursos. 
- Compatibilidad con la salida más sencilla para crear flujos de trabajo personalizados con Azure Functions.

#### <a name="new-transform-object"></a>Nuevo objeto Transform

El nuevo objeto **Transform** simplifica el modelo de codificación. El nuevo objeto facilita la creación y el uso compartido de la codificación de plantillas y valores preestablecidos de Resource Manager. 

#### <a name="azure-active-directory-authentication-and-rbac"></a>Autenticación y RBAC de Azure Active Directory

El control de acceso basado en rol (RBAC) y la autenticación de Azure AD habilitan transformaciones seguras, objetos LiveEvent, directivas de clave de contenido o recursos por rol o usuarios en Azure AD.

#### <a name="client-sdks"></a>SDK de cliente  

Idiomas admitidos en Media Services v3: .NET Core, Java, Node.js, Ruby, Typescript, Python y Go.

#### <a name="live-encoding-updates"></a>Actualizaciones de Live Encoding

Se presentan las siguientes actualizaciones de Live Encoding:

- Nuevo modo en vivo de baja latencia (10 segundos de principio a fin).
- Compatibilidad mejorada con RTMP (mayor estabilidad y mejor compatibilidad con codificadores de origen).
- Ingesta segura de RTMPS.

    Cuando se crea un evento en directo, ahora obtiene 4 direcciones URL de ingesta. Las cuatro direcciones URL de ingesta son casi idénticas, tienen el mismo token de streaming (AppId) y solo se diferencian en componente de número de puerto. Dos de las direcciones URL son principal y de respaldo para RTMPS. 
- Soporte técnico de transcodificación las 24 horas. 
- Mejor compatibilidad con señalización de anuncios en RTMP a través de SCTE35.

#### <a name="improved-event-grid-support"></a>Soporte técnico mejorado de Event Grid

Puede ver las siguientes mejoras de soporte técnico de Event Grid:

- Integración de Azure Event Grid para facilitar el desarrollo con Logic Apps y Azure Functions. 
- Suscríbase a eventos sobre codificación, canales en vivo y mucho más.

### <a name="cmaf-support"></a>Compatibilidad con CMAF

Compatibilidad con el cifrado de CMAF y “cbcs” para reproductores Apple HLS (iOS 11+) y MPEG-DASH que admiten CMAF.

### <a name="video-indexer"></a>Video Indexer

La versión de disponibilidad general Video Indexer se anunció en agosto. Para información reciente sobre las características admitidas actualmente, vea [Novedades de Video Indexer](../video-indexer/video-indexer-overview.md?bc=/azure/media-services/video-indexer/breadcrumb/toc.json&toc=/azure/media-services/video-indexer/toc.json). 

### <a name="plans-for-changes"></a>Planes de cambios

#### <a name="azure-cli-20"></a>CLI de Azure 2.0
 
Próximamente estará disponible el módulo de la CLI de Azure 2.0 que incluye operaciones en todas las características (como Live, directivas de clave de contenido, filtros de cuenta/recurso, directivas de streaming). 

### <a name="known-issues"></a>Problemas conocidos

Solo los clientes que usan la API de versión preliminar con filtros de cuenta o recurso se ven afectados por el problema siguiente.

Si creó filtros de cuenta o recurso entre el 28/09 y el 12/10 con las API o la CLI de Media Services v3, debe quitar todos los filtros de cuenta y recurso y volver a crearlos debido a un conflicto de versiones. 

## <a name="may-2018---preview"></a>Mayo de 2018: versión preliminar

### <a name="net-sdk"></a>.NET SDK

Las características siguientes están disponibles en el SDK de .NET:

* **Transformaciones** y **trabajos** para codificar o analizar el contenido multimedia. Para ver ejemplos, consulte los artículos sobre [streaming de archivos](stream-files-tutorial-with-api.md) y [análisis](analyze-videos-tutorial-with-api.md).
* **Localizadores de streaming** para publicar y transmitir contenido a los dispositivos de usuarios finales.
* **Directivas de streaming** y **directivas de claves de contenido** para configurar la entrega de claves y la protección de contenido (DRM) al entregar el contenido.
* **Eventos en directo** y **salidas de eventos** para configurar la ingesta y el archivo de contenido de streaming en vivo.
* **Recursos** para almacenar y publicado contenido multimedia en Azure Storage. 
* **Puntos de conexión de streaming** para configurar y escalar el empaquetado dinámico, cifrado y streaming tanto de contenido multimedia en vivo como a petición.

### <a name="known-issues"></a>Problemas conocidos

* Al enviar un trabajo, puede especificar que se ingiera el vídeo de origen mediante direcciones URL HTTPS, URL SAS o rutas de acceso a archivos ubicados en Azure Blob Storage. Actualmente, Media Services v3 no admite la codificación de transferencia fragmentada a través de direcciones URL HTTPS.

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="see-also"></a>Consulte también

[Guía de migración para migrar de Media Services v2 a v3](migrate-from-v2-to-v3.md#known-issues).

## <a name="next-steps"></a>Pasos siguientes

- [Información general](media-services-overview.md)
- [Actualizaciones de la documentación de Media Services v3](docs-release-notes.md)
- [Notas de la versión de Media Services v2](../previous/media-services-release-notes.md)
