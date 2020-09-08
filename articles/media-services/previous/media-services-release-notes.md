---
title: Notas de la versión de Azure Media Services | Microsoft Docs
description: En este artículo se habla sobre las notas de la versión de Microsoft Azure Media Services v2.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: media
ms.devlang: dotnet
ms.topic: article
ms.date: 10/01/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 7764452d0e52a29204b276b4939c4a8a5c144ca4
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89268662"
---
# <a name="azure-media-services-release-notes"></a>Notas de la versión de Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Estas notas de la versión de Azure Media Services resumen los cambios realizados desde las versiones anteriores y los problemas conocidos.

> [!NOTE]
> No se van agregar características a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Queremos recibir opiniones de nuestros clientes para poder centrarnos en la solución de los problemas que les afectan. Para informar de un problema o formular una pregunta, envíe su consulta al [foro de MSDN de Azure Media Services]. 

## <a name="known-issues"></a><a name="issues"></a>Problemas conocidos
### <a name="media-services-general-issues"></a><a name="general_issues"></a>Problemas generales de Media Services

| Problema | Descripción |
| --- | --- |
| Varios encabezados HTTP comunes no se proporcionan en la API de REST. |Si desarrolla aplicaciones de Media Services mediante la API de REST, encontrará que algunos campos de encabezado HTTP comunes (como CLIENT-REQUEST-ID, REQUEST-ID y RETURN-CLIENT-REQUEST-ID) no se admiten. Los encabezados se agregarán en una futura actualización. |
| No se permite la codificación porcentual. |Media Services usa el valor de la propiedad IAssetFile.Name al generar direcciones URL para el contenido de streaming (por ejemplo, `http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters`). Por esta razón, no se permite la codificación porcentual. El valor de la propiedad Name no puede tener ninguno de los siguientes [caracteres reservados para la codificación porcentual](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters): !*'();:@&=+$,/?%#[]". Además, solo puede haber un "." para la extensión del nombre de archivo. |
| El método ListBlobs que es parte del SDK de Azure Storage, versión 3.x, no funciona correctamente. |Media Services genera direcciones URL de SAS basadas en la versión del [12-02-2012](/rest/api/storageservices/version-2012-02-12) . Si desea usar el SDK de Azure Storage para mostrar los blobs de un contenedor de blobs, utilice el método [CloudBlobContainer.ListBlobs](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobs) que es parte del SDK de Azure Storage, versión 2.x. |
| El mecanismo de limitación de Media Services restringe el uso de recursos en las aplicaciones que realizan un número excesivo de solicitudes al servicio. El servicio puede devolver el código de estado HTTP 503 "Servicio no disponible". |Para más información, consulte la descripción del código de estado HTTP 503 en el tema [Códigos de error de Media Services](media-services-encoding-error-codes.md). |
| Al consultar entidades, hay un límite de 1 000 entidades devueltas a la vez, porque la REST v2 pública limita los resultados de consulta a 1 000 resultados. |Debe usar Skip y Take (.NET) o top (REST) como se describe en [este ejemplo de .NET](media-services-dotnet-manage-entities.md#enumerating-through-large-collections-of-entities) y [este ejemplo de la API de REST](media-services-rest-manage-entities.md#enumerating-through-large-collections-of-entities). |
| Algunos clientes pueden experimentar un problema de etiqueta de repetición en el manifiesto de streaming con velocidad de transmisión adaptable. |Para más información, consulte [esta sección](media-services-deliver-content-overview.md#known-issues). |
| Los objetos del SDK de Media Services para .NET no se pueden serializar y, como resultado, no funcionan con Azure Cache for Redis. |Si intenta serializar el objeto AssetCollection del SDK para agregarlo a Azure Cache for Redis, se produce una excepción. |
|La API REST responde con el mensaje de error "The filter cannot be accessed by this version of REST Api" (Esta versión de la API REST no puede acceder al filtro) al intentar obtener un filtro de nivel de recurso o de cuenta.|El filtro se creó o modificó con una versión más reciente de la API que la que se usa para intentar obtener el filtro. Esto puede suceder si el código o las herramientas que usa el cliente utilizan dos versiones de la API.  Aquí, la mejor solución es actualizar el código o las herramientas para usar las dos versiones de la API o la más reciente.|

## <a name="rest-api-version-history"></a><a name="rest_version_history"></a>Historial de versiones de la API de REST
Para obtener información sobre el historial de versiones de la API de REST de Media Services, consulte la [Referencia de la API de REST de Azure Media Services].

## <a name="february-2020"></a>Febrero de 2020

Algunos procesadores de multimedia de análisis se retirarán. Para ver las fechas de retirada, consulte el tema sobre [componentes heredados](legacy-components.md).

## <a name="september-2019"></a>Septiembre de 2019

### <a name="deprecation-of-media-processors"></a>Desuso de los procesadores de multimedia

Estamos anunciando el desuso de *Azure Media Indexer* y *Azure Media Indexer 2 Preview*. [Video Indexer de Azure Media Services ](../video-indexer/index.yml) reemplaza a estos procesadores multimedia heredados.

Para ver las fechas de retirada, consulte el tema [componentes heredados](legacy-components.md).

Consulte también [Migración de Azure Media Indexer y Azure Media Indexer 2 a Video Indexer de Azure Media Services](migrate-indexer-v1-v2.md).

## <a name="august-2019"></a>Agosto de 2019

### <a name="deprecation-of-media-processors"></a>Desuso de los procesadores de multimedia

Anunciamos el desuso de los procesadores de multimedia *Windows Azure Media Encoder* (WAME) y *Azure Media Encoder* (AME). Para ver las fechas de retirada, consulte el tema [componentes heredados](legacy-components.md).

Para más información, consulte [Migración de WAME a Media Encoder Standard](https://go.microsoft.com/fwlink/?LinkId=2101334) y [Migración de AME a Media Encoder Standard](https://go.microsoft.com/fwlink/?LinkId=2101335).

## <a name="march-2019"></a>Marzo de 2019

La característica Media Hyperlapse Preview de Azure Media Services entró en desuso.

## <a name="december-2018"></a>Diciembre de 2018

La característica Media Hyperlapse Preview de Azure Media Services se retirará en breve. A partir del 19 de diciembre de 2018, Media Services ya no realizará cambios ni mejoras en la característica Media Hyperlapse. El 29 de marzo de 2019 se retirará y ya no estará disponible.

## <a name="october-2018"></a>Octubre de 2018

### <a name="cmaf-support"></a>Compatibilidad con CMAF

Compatibilidad con el cifrado de CMAF y “cbcs” para reproductores Apple HLS (iOS 11+) y MPEG-DASH que admiten CMAF.

### <a name="web-vtt-thumbnail-sprites"></a>Sprites de miniaturas WebVTT

Ahora puede usar Media Services para generar sprites de miniaturas VTTWeb mediante nuestras API v2. Para obtener más información, consulte [Generación de un sprite de miniaturas](generate-thumbnail-sprite.md).

## <a name="july-2018"></a>Julio de 2018

Con la versión más reciente del servicio, hay pequeños cambios en el formato de los mensajes de error que devuelve el servicio cuando se produce un error en un trabajo que tienen que ver con la forma en que se divide en dos, o más, líneas.

## <a name="may-2018"></a>Mayo de 2018 

A partir del 12 de mayo de 2018, los canales en directo ya no admitirán el protocolo de ingesta de secuencia de transporte RTP/MPEG-2. Migre de RTP/MPEG-2 a RTMP o protocolos de ingeesta de MP4 fragmentado (Smooth Streaming).

## <a name="october-2017-release"></a>Versión de octubre de 2017
> [!IMPORTANT] 
> Media Services está dejando el soporte técnico para las claves de autenticación de Azure Access Control Service. A partir del 22 de junio de 2018, ya no se podrá usar código para autenticarse en el back-end de Media Services con claves de Access Control Service. Debe actualizar el código para que use Azure Active Directory (Azure AD) por [autenticación basada en Azure AD](media-services-use-aad-auth-to-access-ams-api.md). Inspeccione las advertencias sobre este cambio en Azure Portal.

### <a name="updates-for-october-2017"></a>Actualizaciones de octubre de 2017
#### <a name="sdks"></a>SDK
* El SDK para .NET se ha actualizado para admitir la autenticación de Azure AD. Se ha eliminado la compatibilidad con la autenticación de Access Control Service del SDK para .NET más reciente en Nuget.org, con el fin de promover una migración más rápida a Azure AD. 
* El SDK para Java se ha actualizado para admitir la autenticación de Azure AD. Se ha agregado compatibilidad con la autenticación de Azure AD al SDK para Java. Para información acerca de cómo usar el SDK para Java con Media Services, consulte [Introducción al SDK de cliente de Java para Azure Media Services](media-services-java-how-to-use.md)

#### <a name="file-based-encoding"></a>Codificación basada en archivo
* Ahora puede usar el codificador Premium para codificar el contenido para el códec de vídeo H.265 de codificación de vídeo de alta eficacia (HEVC). No afecta a los precios si elige H.265 frente a otros códecs, por ejemplo, H.264. Para obtener información acerca de las licencias de la patente HEVC, consulte las [Condiciones de los servicios en línea](https://azure.microsoft.com/support/legal/).
* Para un vídeo de origen codificado con el códec de vídeo H.265(HEVC), como un vídeo capturado mediante iOS11 o GoPro Hero 6, ahora puede usar el codificador Premium o el Estándar para codificarlo. Para obtener información acerca de las licencias de la patente, consulte las [Condiciones de los servicios en línea](https://azure.microsoft.com/support/legal/).
* Para el contenido que contiene varias pistas de audio de idioma, los valores del idioma deben estar etiquetados correctamente según la especificación del formato de archivo correspondiente (por ejemplo, ISO MP4). A continuación, puede usar el codificador estándar para codificar el contenido para streaming. El localizador de streaming resultante muestra una lista de los idiomas de audio disponibles.
* El codificador estándar ahora admite dos nuevos valores preestablecidos del sistema de audio, "Audio AAC" y "Audio AAC de buena calidad". Ambos producen una salida de codificación de audio avanzada (AAC) en estéreo, a velocidades de bits de 128 kbps y 192 kbps, respectivamente.
* El codificador Premium ahora admite los formatos de archivo QuickTime y MOV como entrada. El códec de vídeo debe ser uno de los [tipos de ProRes de Apple que se enumeran en este artículo de GitHub](./media-services-media-encoder-standard-formats.md). El audio debe ser AAC o modulación de código de pulso (PCM). El codificador Premium no admite, por ejemplo, vídeo DVC o DVCPro encapsulado en archivos QuickTime o MOV como entrada. El codificador estándar admite estos códecs de vídeo.
* Se realizaron las correcciones de errores siguientes en los codificadores:

    * Ahora puede enviar trabajos mediante el uso de un recurso de entrada. Una vez finalizadas estas tareas, puede modificar el recurso (por ejemplo, agregar, eliminar o cambiar el nombre de archivos del recurso) y enviar trabajos adicionales.
    * Se ha mejorado la calidad de las miniaturas JPEG que genera el codificador estándar.
    * El codificador estándar controla mejor los metadatos de entrada y la generación de miniaturas en vídeos de muy corta duración.
    * Las mejoras en el descodificador H.264 que se usa en el codificador estándar eliminan ciertos artefactos poco frecuentes. 

#### <a name="media-analytics"></a>Media Analytics
Disponibilidad general de Azure Media Redactor: Este procesador de multimedia anonimiza poniendo borrosas las caras de las personas seleccionadas. Es ideal para usar en casos de seguridad pública y medios informativos. 

Para obtener información general sobre este nuevo procesador, consulte [esta publicación del blog](https://azure.microsoft.com/blog/azure-media-redactor/). Para obtener información sobre la documentación y configuración, consulte [Censura de rostros con Azure Media Analytics](media-services-face-redaction.md).



## <a name="june-2017-release"></a>Versión de junio de 2017

Media Services ahora admite la [autenticación basada en Azure AD](media-services-use-aad-auth-to-access-ams-api.md).

> [!IMPORTANT]
> Actualmente, Media Services es compatible con el modelo de autenticación de Access Control Service. La autorización de Access Control Service dejará de usarse el 1 de junio de 2018. Se recomienda migrar tan pronto como sea posible al modelo de autenticación de Azure AD.

## <a name="march-2017-release"></a>Versión de marzo de 2017

Ahora puede usar el codificador Estándar para [generar automáticamente una escalera de velocidad de bits](media-services-autogen-bitrate-ladder-with-mes.md) especificando la cadena preestablecida "Adaptive Streaming" (Streaming adaptable) al crear la tarea de codificación. Puede utilizar el valor preestablecido "Adaptive Streaming" para codificar un vídeo para streaming con Media Services. Para personalizar un valor preestablecido de codificación para su escenario específico, puede comenzar con [estos valores preestablecidos](media-services-mes-presets-overview.md).

Ahora puede usar Media Encoder Standard o Media Encoder Premium Workflow para [crear una tarea de codificación que genera fragmentos de fMP4](media-services-generate-fmp4-chunks.md). 

## <a name="february-2017-release"></a>Versión de febrero de 2017

A partir del 1 de abril de 2017, se eliminarán automáticamente los registros de trabajo de más de 90 días de su cuenta, junto con los registros de tarea asociados. La eliminación se produce aunque el número total de registros no llegue a la cuota máxima. Para archivar la información del trabajo o la tarea, puede usar el código descrito en [Administración de recursos y entidades relacionadas con el SDK de Media Services para .NET](media-services-dotnet-manage-entities.md).

## <a name="january-2017-release"></a>Versión de enero de 2017

En Media Services, un punto de conexión de streaming representa un servicio de streaming que puede entregar contenido directamente a una aplicación de reproducción de cliente o a una red de entrega de contenido (CDN) para la distribución posterior. Media Services también proporciona una integración perfecta con Azure Content Delivery Network. La secuencia de salida de un servicio de punto de conexión de streaming puede ser una secuencia en vivo, un vídeo bajo demanda o una descarga progresiva de su recurso en su cuenta de Media Services. Cada cuenta de Media Services incluye un punto de conexión de streaming predeterminado. Es posible crear puntos de conexión de streaming adicionales en la cuenta. 

Existen dos versiones de puntos de conexión de streaming, 1.0 y 2.0. A partir del 10 de enero de 2017 las nuevas cuentas de Media Services creadas incluirán la versión 2.0 del punto de conexión de streaming predeterminado. Los puntos de conexión de streaming adicionales que agregue a esta cuenta también se generarán en la versión 2.0. Este cambio no afecta a las cuentas existentes. Los puntos de conexión de streaming existentes son versión 1.0 y se pueden actualizar a la versión 2.0. Hay cambios de comportamiento, facturación y características con este cambio. Para obtener más información, consulte [Streaming endpoints overview](media-services-streaming-endpoints-overview.md) (Información general de puntos de conexión de streaming).

A partir de la versión 2.15, se agregaron en Media Services las siguientes propiedades a la entidad punto de conexión de streaming:

* CdnProvider 
* CdnProfile
* FreeTrialEndTime 
* Versión de punto de conexión de streaming 

Para más información acerca de estas propiedades, consulte [Punto de conexión de streaming](/rest/api/media/operations/streamingendpoint). 

## <a name="december-2016-release"></a>Versión de diciembre de 2016

 Ahora puede usar Media Services para acceder a los datos de telemetría y métricas de sus servicios. Puede usar la versión actual de Media Services para recopilar datos de telemetría de un canal en vivo, de un punto de conexión de streaming y de entidades de archivo. Para más información, consulte [Telemetría en Media Services](media-services-telemetry-overview.md).

## <a name="july-2016-release"></a><a name="july_changes16"></a>Versión de julio de 2016
### <a name="updates-to-the-manifest-file-ism-generated-by-encoding-tasks"></a>Actualizaciones al archivo de manifiesto (*.ism) generado por las tareas de codificación
Cuando se envía una tarea de codificación a Media Encoder Standard o Media Encoder Premium, la tarea de codificación genera un [archivo de manifiesto de streaming](media-services-deliver-content-overview.md) (*.ism) en el recurso de salida. Con la última versión del servicio se ha actualizado la sintaxis de este archivo de manifiesto de streaming.

> [!NOTE]
> La sintaxis del archivo de manifiesto de streaming (.ism) está reservada para uso interno. Está sujeto a cambios en versiones futuras. No modifique ni manipule el contenido de este archivo.
> 
> 

### <a name="a-new-client-manifest-ismc-file-is-generated-in-the-output-asset-when-an-encoding-task-outputs-one-or-more-mp4-files"></a>Se genera un nuevo archivo de manifiesto de cliente (*.ismc) en el recurso de salida cuando una tarea de codificación genera uno o varios archivos MP4
A partir de la versión del servicio más reciente, después de completar una tarea de codificación que genera uno más archivos MP4, el recurso de salida también contiene un archivo de manifiesto de cliente de streaming (*.ismc). El archivo .ismc ayuda a mejorar el rendimiento del streaming dinámico. 

> [!NOTE]
> La sintaxis del archivo de manifiesto de cliente (.ismc) está reservada para uso interno. Está sujeto a cambios en versiones futuras. No modifique ni manipule el contenido de este archivo.
> 
> 

Para más información, vea [este blog](/archive/blogs/randomnumber/encoder-changes-within-azure-media-services-now-create-ismc-file).

### <a name="known-issues"></a>Problemas conocidos
Algunos clientes pueden experimentar un problema de etiqueta de repetición en el manifiesto de streaming con velocidad de transmisión adaptable. Para más información, consulte [esta sección](media-services-deliver-content-overview.md#known-issues).

## <a name="april-2016-release"></a><a id="apr_changes16"></a>Versión de abril de 2016
### <a name="media-analytics"></a>Media Analytics
 Media Services presentó Media Analytics para proporcionar una inteligencia de vídeo eficaz. Para más información, consulte [Introducción a Analytics de Media Services](media-services-analytics-overview.md).

### <a name="apple-fairplay-preview"></a>Apple FairPlay (versión preliminar)
Ahora puede utilizar Media Services para cifrar dinámicamente el contenido HTTP Live Streaming (HLS) con Apple FairPlay. Ahora también puede utilizar el servicio de entrega de licencias de Media Services para entregar licencias de FairPlay a los clientes. Para más información, consulte "Uso de Azure Media Services para transmitir contenido HLS protegido con Apple FairPlay".

## <a name="february-2016-release"></a><a id="feb_changes16"></a>Versión de febrero de 2016
La última versión del SDK de Media Services para .NET (3.5.3) contiene una corrección de errores relacionada con Google Widevine. AssetDeliveryPolicy no se podía reutilizar para varios recursos cifrados con Widevine. Como parte de esta corrección de errores, se ha agregado la siguiente propiedad al SDK: WidevineBaseLicenseAcquisitionUrl.

```csharp
Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
    new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
{
    {AssetDeliveryPolicyConfigurationKey.WidevineBaseLicenseAcquisitionUrl,"http://testurl"},

};
```

## <a name="january-2016-release"></a><a id="jan_changes_16"></a>Versión de enero de 2016
Se cambia el nombre de las unidades reservadas de codificación para reducir la confusión con los nombres de codificador.

Los nombres de las unidades reservadas de codificación Básica, Estándar y Premium se cambian por unidades reservadas S1, S2 y S3, respectivamente. Los clientes que utilizan unidades reservadas de codificación Básica hoy en día ven S1 como etiqueta en Azure Portal (y en la factura). Los clientes que usen Estándar y Premium ven las etiquetas S2 y S3, respectivamente. 

## <a name="december-2015-release"></a><a id="dec_changes_15"></a>Versión de diciembre de 2015

### <a name="media-encoder-deprecation-announcement"></a>Anuncio de entrada en desuso de Media Encoder

 Media Encoder dejará de utilizarse unos 12 meses después del lanzamiento de Media Encoder Standard.

### <a name="azure-sdk-for-php"></a>SDK de Azure para PHP
El equipo del SDK de Azure publicó una nueva versión del paquete [SDK de Azure para PHP](https://github.com/Azure/azure-sdk-for-php) que contiene actualizaciones y nuevas características de Media Services. En concreto, el SDK de Media Services para PHP ahora admite las características más recientes de [protección de contenido](media-services-content-protection-overview.md). Estas características son el cifrado dinámico con AES y DRM (PlayReady y Widevine) con y sin restricciones de token. También admite el escalado de [unidades de codificación](media-services-dotnet-encoding-units.md).

Para más información, consulte:

* Los siguientes [ejemplos de código](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices) le ayudan a empezar a trabajar rápidamente:
  * **vodworkflow_aes.php**: Se trata de un archivo PHP que muestra cómo usar el cifrado dinámico AES-128 y el servicio de entrega de claves. Se basa en el ejemplo de .NET que se explica en [Uso del cifrado dinámico AES-128 y el servicio de entrega de claves](media-services-protect-with-aes128.md).
  * **vodworkflow_aes.php**: Se trata de un archivo PHP que muestra cómo usar el cifrado dinámico de PlayReady y el servicio de entrega de licencias. Se basa en el ejemplo de .NET que se explica en [Uso del cifrado dinámico común de PlayReady y Widevine](media-services-protect-with-playready-widevine.md).
  * **scale_encoding_units.php**: Se trata de un archivo PHP que muestra cómo escalar las unidades reservadas de codificación.

## <a name="november-2015-release"></a><a id="nov_changes_15"></a>Versión de noviembre de 2015
 Media Services ofrece ahora el servicio de entrega de licencias en la nube Widevine. Para más información, vea [este blog](https://azure.microsoft.com/blog/announcing-google-widevine-license-delivery-services-public-preview-in-azure-media-services/). Consulte también [este tutorial](media-services-protect-with-playready-widevine.md) y el [repositorio de GitHub](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-drm). 

Los servicios de entrega de licencias de Widevine proporcionados por Media Services se encuentran en versión preliminar. Para más información, vea [este blog](https://azure.microsoft.com/blog/announcing-google-widevine-license-delivery-services-public-preview-in-azure-media-services/).

## <a name="october-2015-release"></a><a id="oct_changes_15"></a>Versión de octubre de 2015
Media Services ahora está disponible en los centros de datos siguientes: Sur de Brasil, India occidental, Sur de la India y Centro de la India. Ahora se puede usar Azure Portal para [crear cuentas de Media Services](media-services-portal-create-account.md) y realizar varias tareas que se describen en la [página web de documentación de Media Services](https://azure.microsoft.com/documentation/services/media-services/). Live Encoding no está habilitado en estos centros de datos. Además, no todos los tipos de unidades reservadas de codificación están disponibles en estos centros de datos.

* Sur de Brasil:                                          Solo están disponibles las unidades reservadas de codificación Estándar y Básica.
* India occidental, Sur de la India y Centro de la India:             Solo están disponibles las unidades reservadas de codificación Básica.

## <a name="september-2015-release"></a><a id="september_changes_15"></a>Versión de septiembre de 2015
Media Services ofrece ahora la posibilidad de proteger tanto vídeo bajo demanda como secuencias en vivo con la tecnología DRM modular de Widevine. Puede usar los siguientes asociados de los servicios de entrega para ayudarle a entregar licencias de Widevine:
* [Axinom](https://www.axinom.com) 
* [EZDRM](https://ezdrm.com/) 
* [castLabs](https://castlabs.com/company/partners/azure/) 

Para más información, vea [este blog](https://azure.microsoft.com/blog/azure-media-services-adds-google-widevine-packaging-for-delivering-multi-drm-stream/).
  
Puede usar el [SDK de .NET de Media Services](https://www.nuget.org/packages/windowsazure.mediaservices/) (a partir de la versión 3.5.1) o la API REST para configurar AssetDeliveryConfiguration para usar Widevine. 
* Media Services agregó compatibilidad para vídeos ProRes de Apple. Ahora puede cargar sus archivos de vídeos de origen QuickTime que usan ProRes de Apple u otros códecs. Para más información, vea [este blog](https://azure.microsoft.com/blog/announcing-support-for-apple-prores-videos-in-azure-media-services/).
* Ahora puede usar Media Encoder Standard para realizar recortes secundarios y extracción de archivos en vivo. Para más información, vea [este blog](https://azure.microsoft.com/blog/sub-clipping-and-live-archive-extraction-with-media-encoder-standard/).
* Se realizaron las siguientes actualizaciones de filtrado: 
  
  * Ahora puede usar el formato Apple HLS con un filtro solo de audio. Esta actualización le permite quitar la pista solo de audio mediante la especificación de (audio-only=false) en la dirección URL.
  * Al definir filtros para los recursos, ahora puede combinar varios filtros (hasta tres) en una sola dirección URL.
    
    Para más información, vea [este blog](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/).
* Media Services admite ahora I-frames en HLS versión 4. La compatibilidad con I-frame optimiza las operaciones de avance rápido y rebobinado. De forma predeterminada, todas las salidas de HLS versión 4 incluyen la lista de reproducción de I-frame (EXT-X-I-FRAME-STREAM-INF).
Para más información, vea [este blog](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/).

## <a name="august-2015-release"></a><a id="august_changes_15"></a>Versión de agosto de 2015
* Ya están disponibles el SDK de Media Services para la versión de Java 0.8.0 y nuevos ejemplos. Para más información, consulte:
    
* Actualización de Azure Media Player con compatibilidad con secuencias de audio múltiples. Para más información, vea [esta publicación del blog](https://azure.microsoft.com/blog/2015/08/13/azure-media-player-update-with-multi-audio-stream-support/).

## <a name="july-2015-release"></a><a id="july_changes_15"></a>Versión de julio de 2015
* Se anuncia la disponibilidad general de Media Encoder Standard. Para más información, vea [esta publicación del blog](https://azure.microsoft.com/blog/2015/07/16/announcing-the-general-availability-of-media-encoder-standard/).
  
    Media Encoder Standard usa valores preestablecidos que se describen en [esta sección](https://go.microsoft.com/fwlink/?LinkId=618336). Cuando se usa un valor preestablecido para codificaciones de 4K, debe obtener el tipo de unidad reservada Premium. Para más información, consulte [Escalado de codificación](media-services-scale-media-processing-overview.md).
* Se utilizaron títulos en vivo en tiempo real con Media Services y Media Player. Para más información, vea [esta publicación del blog](https://azure.microsoft.com/blog/2015/07/08/live-real-time-captions-with-azure-media-services-and-player/).

### <a name="media-services-net-sdk-updates"></a>Actualizaciones del SDK de Media Services para .NET
La versión del SDK de Media Services para .NET ahora es la 3.4.0.0. Se realizaron las siguientes actualizaciones: 

* Se ha implementado la compatibilidad con los archivos activos. No se puede descargar un recurso que contenga un archivo activo.
* Se ha implementado compatibilidad con los filtros dinámicos.
* La funcionalidad se implementa para que los usuarios puedan mantener un contenedor de almacenamiento cuando eliminan un recurso.
* Se han realizado correcciones de errores relacionados con directivas de reintentos en canales.
* Se ha habilitado Media Encoder Premium Workflow.

## <a name="june-2015-release"></a><a id="june_changes_15"></a>Versión de junio de 2015
### <a name="media-services-net-sdk-updates"></a>Actualizaciones del SDK de Media Services para .NET
La versión del SDK de Media Services para .NET ahora es la 3.3.0.0. Se realizaron las siguientes actualizaciones: 

* Se agregó compatibilidad para la especificación de detección de OpenId Connect.
* Se agregó compatibilidad con el control de sustitución de claves en el lado del proveedor de identidades.

Si usa un proveedor de identidades que expone un documento de detección de OpenID Connect (algo que hacen Azure AD, Google y Salesforce), puede indicar a Media Services que obtenga las claves de firma para la validación de los token web JSON (JWT) de la especificación de detección de OpenID Connect. 

Para más información, consulte [Uso de claves web JSON de la especificación de detección OpenID Connect para trabajar con la autenticación JWT en Media Services](http://gtrifonov.com/2015/06/07/using-json-web-keys-from-openid-connect-discovery-spec-to-work-with-jwt-token-authentication-in-azure-media-services/).

## <a name="may-2015-release"></a><a id="may_changes_15"></a>Versión de mayo de 2015
Se anunciaron las nuevas características siguientes:

* [Una versión preliminar de Live Encoding con Media Services](media-services-manage-live-encoder-enabled-channels.md)
* [Manifiesto dinámico](media-services-dynamic-manifest-overview.md)

## <a name="april-2015-release"></a><a id="april_changes_15"></a>Versión de abril de 2015
### <a name="general-media-services-updates"></a>Actualizaciones generales de Media Services
* Se anunció [Media Player](https://azure.microsoft.com/blog/2015/04/15/announcing-azure-media-player/).
* A partir de Media Services REST 2.10, los canales configurados para ingerir un protocolo de mensajes en tiempo real (RTMP) se crean con direcciones URL de ingesta principal y secundaria. Para más información, consulte [Configuraciones de ingesta de canales](media-services-live-streaming-with-onprem-encoders.md#channel_input).
* Se actualizó Azure Media Indexer.
* Se agregó compatibilidad con el idioma español.
* Se agregó una nueva configuración para el formato XML.

Para más información, vea [este blog](https://azure.microsoft.com/blog/2015/04/13/azure-media-indexer-spanish-v1-2/).

### <a name="media-services-net-sdk-updates"></a>Actualizaciones del SDK de Media Services para .NET
La versión del SDK de Media Services para .NET ahora es la 3.2.0.0. Se realizaron las siguientes actualizaciones:

* Cambio importante: El tipo de TokenRestrictionTemplate.Issuer y de TokenRestrictionTemplate.Audience se ha cambiado a cadena.
* Se han realizado actualizaciones relacionadas con la creación de directivas de reintento personalizadas.
* Se han realizado correcciones de errores relacionados con la carga y descarga de archivos.
* La clase MediaServicesCredentials ahora acepta los puntos de conexión de control de acceso principal y secundario para la autenticación.

## <a name="march-2015-release"></a><a id="march_changes_15"></a>Versión de marzo de 2015
### <a name="general-media-services-updates"></a>Actualizaciones generales de Media Services
* Media Services ahora proporciona integración con Content Delivery Network. Para admitir la integración, se ha agregado la propiedad CdnEnabled a StreamingEndpoint. CdnEnabled puede utilizarse con las API de REST a partir de la versión 2.9. Para obtener más información, consulte [StreamingEndpoint](/rest/api/media/operations/streamingendpoint). CdnEnabled puede utilizarse con el SDK para .NET a partir de la versión 3.1.0.2. Para obtener más información, consulte [StreamingEndpoint](/archive/blogs/randomnumber/encoder-changes-within-azure-media-services-now-create-ismc-file).
* Se anunció Media Encoder Premium Workflow. Para más información, consulte [Introducción a la codificación Premium en Azure Media Services](https://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services/).

## <a name="february-2015-release"></a><a id="february_changes_15"></a>Versión de febrero de 2015
### <a name="general-media-services-updates"></a>Actualizaciones generales de Media Services
La versión de la API de REST de Media Services es ahora la 2.9. A partir de esta versión, puede habilitar la integración de Content Delivery Network con los puntos de conexión de streaming. Para obtener más información, consulte [StreamingEndpoint](/rest/api/media/operations/streamingendpoint).

## <a name="january-2015-release"></a><a id="january_changes_15"></a>Versión de enero de 2015
### <a name="general-media-services-updates"></a>Actualizaciones generales de Media Services
Anuncio de la disponibilidad general de la protección de contenido con cifrado dinámico. Para más información, consulte [Media Services mejora la seguridad en streaming con la disponibilidad general de la tecnología DRM](https://azure.microsoft.com/blog/2015/01/29/azure-media-services-enhances-streaming-security-with-general-availability-of-drm-technology/).

### <a name="media-services-net-sdk-updates"></a>Actualizaciones del SDK de Media Services para .NET
La versión del SDK de Media Services para .NET ahora es la 3.1.0.1.

Esta versión marcó el constructor Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization.TokenRestrictionTemplate como obsoleto. El nuevo constructor toma TokenType como argumento.

```csharp
TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);
```


## <a name="december-2014-release"></a><a id="december_changes_14"></a>Versión de diciembre de 2014
### <a name="general-media-services-updates"></a>Actualizaciones generales de Media Services
* Se agregaron algunas actualizaciones y nuevas características a Media Indexer. Para más información, consulte [Notas de la versión de Azure Media Indexer versión 1.1.6.7](https://azure.microsoft.com/blog/2014/12/03/azure-media-indexer-version-1-1-6-7-release-notes/).
* Se agregó una nueva API de REST que puede usar para actualizar las unidades reservadas de codificación. Para más información, consulte [EncodingReservedUnitType con REST](/rest/api/media/operations/encodingreservedunittype).
* Se agregó compatibilidad con CORS para el servicio de entrega de claves.
* Se realizaron mejoras en el rendimiento de la consulta de opciones de directivas de autorización.
* En el centro de datos de China, la [dirección URL de entrega de claves](/rest/api/media/operations/contentkey#get_delivery_service_url) es ahora por cliente (al igual que en otros centros de datos).
* Se agregó la duración de destino automático de HLS. Cuando se realiza el streaming en vivo, HLS siempre se empaqueta dinámicamente. De forma predeterminada, Media Services calcula automáticamente la proporción de empaquetado por segmento HLS (FragmentsPerSegment) según el intervalo de fotogramas clave (KeyFrameInterval). Este método también se conoce como un grupo de imágenes (GOP) que se recibe desde el codificador en directo. Para más información, consulte [Información general de streaming en vivo con Azure Media Services](/previous-versions/azure/dn783466(v=azure.100)).

### <a name="media-services-net-sdk-updates"></a>Actualizaciones del SDK de Media Services para .NET
La versión del [SDK de Media Services para .NET](https://www.nuget.org/packages/windowsazure.mediaservices/) ahora es la 3.1.0.0. Se realizaron las siguientes actualizaciones:

* Se ha actualizado la dependencia del SDK para .NET a .NET Framework 4.5.
* Se agregó una nueva API que puede usar para actualizar las unidades reservadas de codificación. Para más información, consulte [Actualización del tipo de unidades reservadas y aumento de las unidades reservadas de codificación mediante .NET](media-services-dotnet-encoding-units.md).
* Se agregó compatibilidad con la autenticación de token JWT. Para más información, consulte [Autenticación de token JWT en Media Services y cifrado dinámico](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/).
* Se agregaron desplazamientos relativos para BeginDate y ExpirationDate en la plantilla de licencia de PlayReady.

## <a name="november-2014-release"></a><a id="november_changes_14"></a>Versión de noviembre de 2014
* Ahora puede usar Media Services para ingerir contenido de Smooth Streaming (fMP4) en vivo sobre una conexión TLS. Para ingerir en TLS, asegúrese de actualizar la dirección URL de introducción a HTTPS. Actualmente, Media Services no admite TLS con dominios personalizados. Para más información sobre streaming en vivo, consulte [Información general de streaming en vivo con Azure Media Services](/previous-versions/azure/dn783466(v=azure.100)).
* En la actualidad no se puede ingerir una secuencia en vivo RTMP por una conexión TLS.
* Solo puede transmitir por TLS si el punto de conexión de streaming desde el que se entrega el contenido se creó después del 10 de septiembre de 2014. Si las direcciones URL de streaming se basan en puntos de conexión de streaming creados después del 10 de septiembre de 2014, la dirección URL contendrá "streaming.mediaservices.windows.net" (el formato nuevo). Las direcciones URL de streaming que contengan "origin.mediaservices.windows.net" (el formato anterior) no son compatibles con TLS. Si la dirección URL tiene el formato antiguo y desea transmitir a través de TLS, [cree un punto de conexión de streaming nuevo](media-services-portal-manage-streaming-endpoints.md). Use direcciones URL creadas en función del nuevo punto de conexión de streaming para transmitir el contenido a través de TLS.

### <a name="media-services-net-sdk"></a><a id="oct_sdk"></a>SDK .NET de Media Services
La versión de las extensiones del SDK de Media Services para .NET es ahora la 2.0.0.3.

La versión del SDK de Media Services para .NET es ahora la 3.0.0.8. Se realizaron las siguientes actualizaciones:

* Se ha implementado la refactorización en las clases de directivas de reintento.
* Se ha agregado una cadena de agente de usuario a los encabezados de solicitud HTTP.
* Se ha agregado un paso de compilación de restauración de NuGet.
* Se corrigieron las pruebas del escenario para utilizar el certificado x509 del repositorio.
* Se agregaron valores para la validación posterior a la actualización del canal y la transmisión.

### <a name="new-github-repository-to-host-media-services-samples"></a>Nuevo repositorio de GitHub para hospedar las muestras de Media Services
Hay ejemplos en el [repositorio de GitHub de ejemplos de Media Services](https://github.com/Azure/Azure-Media-Services-Samples).

## <a name="september-2014-release"></a><a id="september_changes_14"></a>Versión de septiembre de 2014
La versión de los metadatos de REST de Media Services es ahora la 2.7. Para más información sobre las últimas actualizaciones de REST, consulte [Referencia de la API de REST de Media Services](/rest/api/media/operations/azure-media-services-rest-api-reference).

La versión del SDK de Media Services para .NET es ahora la 3.0.0.7

### <a name="breaking-changes"></a><a id="sept_14_breaking_changes"></a>Cambios importantes
* Origin ha cambiado a [StreamingEndpoint].
* Se ha realizado un cambio en el comportamiento predeterminado al usar Azure Portal para codificar y después publicar archivos MP4.

### <a name="new-featuresscenarios-that-are-part-of-the-general-availability-release"></a><a id="sept_14_GA_changes"></a>Nuevas características y escenarios que forman parte de la versión de disponibilidad general
* Se anuncia el procesador de multimedia de Media Indexer. Para más información, consulte [Indexación de archivos multimedia con Media Indexer](/previous-versions/azure/dn783455(v=azure.100)).
* Puede usar la entidad [StreamingEndpoint] para agregar nombres de dominio personalizados (host).
  
    Para usar un nombre de dominio personalizado como nombre del punto de conexión de streaming de Media Services, debe agregar nombres de host personalizados al punto de conexión de streaming. Puede usar las API de REST o el SDK de Media Services para .NET para agregar nombres de host personalizados.
  
    Se aplican las siguientes consideraciones:
  
  * Debe tener la propiedad del nombre de dominio personalizado.
  * Media Services debe validar la propiedad del nombre de dominio. Para validar el dominio, cree un objeto CName que asigne el dominio primario MediaServicesAccountId para verificar el servidor DNS mediaservices-dns-zone.
  * Debe crear otro objeto CName que asigne el nombre de host personalizado (por ejemplo, sports.contoso.com) al nombre de host del punto de conexión de streaming de Media Services (por ejemplo, amstest.streaming.mediaservices.windows.net).

    Para más información, consulte la propiedad CustomHostNames en el artículo [Punto de conexión de streaming](/rest/api/media/operations/streamingendpoint).

### <a name="new-featuresscenarios-that-are-part-of-the-public-preview-release"></a><a id="sept_14_preview_changes"></a>Nuevas características/escenarios que forman parte de la versión preliminar pública
* Versión preliminar de streaming en vivo. Para más información, consulte [Información general de streaming en vivo con Azure Media Services](/previous-versions/azure/dn783466(v=azure.100)).
* Servicio de entrega de claves. Para más información, consulte [Uso del cifrado dinámico AES-128 y el servicio de entrega de claves](/previous-versions/azure/dn783457(v=azure.100)).
* Cifrado dinámico AES. Para más información, consulte [Uso del cifrado dinámico AES-128 y el servicio de entrega de claves](/previous-versions/azure/dn783457(v=azure.100)).
* Servicio de entrega de licencias PlayReady. 
* Cifrado dinámico de PlayReady. 
* Plantilla de licencias de PlayReady de Media Services. Para más información, consulte [Introducción a la plantilla de licencias de PlayReady de Media Services].
* Streaming de recursos de almacenamiento cifrado. Para más información, consulte [Streaming de contenido de almacenamiento cifrado](/previous-versions/azure/dn783451(v=azure.100)).

## <a name="august-2014-release"></a><a id="august_changes_14"></a>Versión de agosto de 2014
Cuando se codifica un recurso, se produce un recurso de salida tras la finalización del trabajo de codificación. Hasta esta versión, el codificador de Media Services producía metadatos sobre los recursos de salida. A partir de esta versión, el codificador también produce metadatos sobre los recursos de entrada. Para más información, consulte [Metadatos de entrada] y [Metadatos de salida].

## <a name="july-2014-release"></a><a id="july_changes_14"></a>Versión de julio de 2014
Se han corregido los siguientes errores para Azure Media Services Packager y Encryptor:

* Cuando un recurso de archivo dinámico se transmite a HLS, solo el audio se reproduce: Este problema se ha corregido y ahora pueden reproducirse el audio y el vídeo.
* Al empaquetar un recurso para el cifrado de sobre HLS y AES de 128 bits, las transmisiones empaquetadas no se reproducen en dispositivos Android: Este error se ha corregido y la transmisión empaquetada se reproduce en dispositivos Android compatibles con HLS.

## <a name="may-2014-release"></a><a id="may_changes_14"></a>Versión de mayo de 2014
### <a name="general-media-services-updates"></a><a id="may_14_changes"></a>Actualizaciones generales de Media Services
Ahora puede usar el [empaquetado dinámico] para transmitir HLS versión 3. Para transmitir HLS versión 3, agregue el siguiente formato a la ruta del localizador de origen: *.ism/manifest(format=m3u8-aapl-v3). Para más información, consulte esta [este foro](https://social.msdn.microsoft.com/Forums/en-US/13b8a776-9519-4145-b9ed-d2b632861fde/dynamic-packaging-to-hls-v3).

El empaquetado dinámico admite ahora también la entrega de HLS (versión 3 y versión 4) cifrado con PlayReady basado en Smooth Streaming cifrado estáticamente con PlayReady. Para obtener información sobre cómo cifrar Smooth Streaming con PlayReady, consulte [Protección de Smooth Streaming con PlayReady](/previous-versions/azure/dn189154(v=azure.100)).

### <a name="media-services-net-sdk-updates"></a><a name="may_14_donnet_changes"></a>Actualizaciones del SDK de Media Services para .NET
La versión del SDK de Media Services para .NET ahora es la 3.0.0.5. Se realizaron las siguientes actualizaciones:

* La velocidad y la resistencia son mejores al cargar y descargar recursos multimedia.
* Se han realizado mejoras en la gestión de excepciones transitorias y la lógica de reintento: 
  
  * La detección de errores transitorios y la lógica de reintento se han mejorado en las excepciones ocasionadas al realizar consultas, guardar cambios y cargar y descargar archivos. 
  * Al obtener excepciones web (por ejemplo, durante una solicitud de token de Access Control Service), los errores irrecuperables se producen más rápido ahora.

Para más información, consulte [Lógica de reintento en el SDK de Media Services para .NET].

## <a name="januaryfebruary-2014-releases"></a><a id="jan_feb_changes_14"></a>Versiones de enero/febrero de 2014
### <a name="media-services-net-sdk-3001-3002-and-3003"></a><a name="jan_fab_14_donnet_changes"></a>SDK de Media Services para .NET 3.0.0.1, 3.0.0.2 y 3.0.0.3
Los cambios en 3.0.0.1 y 3.0.0.2 incluyen los siguientes:

* Se han corregido los problemas relacionados con el uso de consultas LINQ con instrucciones OrderBy.
* Se han dividido las soluciones de prueba de [GitHub] en pruebas basadas en unidades y pruebas basadas en escenarios.

Para más información acerca de los cambios, consulte [SDK de Media Services para .NET 3.0.0.1 y 3.0.0.2](http://gtrifonov.com/2014/02/07/windows-azure-media-services-net-sdk-3-0-0-2-release/index.html).

Se han realizado los siguientes cambios en la versión 3.0.0.3:

* Se han actualizado las dependencias de Azure Storage para usar la versión 3.0.3.0.
* Se ha corregido un problema de compatibilidad con versiones anteriores en las versiones 3.0. *.* .

## <a name="december-2013-release"></a><a id="december_changes_13"></a>Versión de diciembre de 2013
### <a name="media-services-net-sdk-3000"></a><a name="dec_13_donnet_changes"></a>SDK de Media Services para .NET 3.0.0.0
> [!NOTE]
> Las versiones 3.0.x.x no son compatibles con las versiones 2.4.x.x.
> 
> 

La última versión del SDK de Media Services es ahora la 3.0.0.0. Puede descargar el último paquete de NuGet u obtener los bits de [GitHub].

A partir de la versión 3.0.0.0 del SDK de Media Services, puede reutilizar los tokens de [Azure AD Access Control Service](/previous-versions/azure/azure-services/hh147631(v=azure.100)). Para más información, consulte la sección "Reutilización de los tokens de Access Control Service" en [Conexión a Media Services con el SDK de Media Services para .NET](/previous-versions/azure/jj129571(v=azure.100)).

### <a name="media-services-net-sdk-extensions-2000"></a><a name="dec_13_donnet_ext_changes"></a>Extensiones del SDK de Media Services para .NET 2.0.0.0
 Las extensiones del SDK de Media Services para .NET son un conjunto de métodos de extensión y funciones auxiliares que simplificarán su código y facilitarán el desarrollo con Media Services. Puede obtener los bits más recientes en [Extensiones del SDK de Media Services para .NET](https://github.com/Azure/azure-sdk-for-media-services-extensions/tree/dev).

## <a name="november-2013-release"></a><a id="november_changes_13"></a>Versión de noviembre de 2013
### <a name="media-services-net-sdk-changes"></a><a name="nov_13_donnet_changes"></a>Cambios en el SDK de Media Services para .NET
A partir de esta versión, el SDK de Media Services para .NET administra los errores transitorios que se pueden producir al realizar llamadas a la capa de la API de REST de Media Services.

## <a name="august-2013-release"></a><a id="august_changes_13"></a>Versión de agosto de 2013
### <a name="media-services-powershell-cmdlets-included-in-azure-sdk-tools"></a><a name="aug_13_powershell_changes"></a>Cmdlets de PowerShell de Media Services incluidos en Herramientas de Azure SDK
Los siguientes cmdlets de PowerShell de Media Services se incluyen ahora en [Herramientas de Azure SDK](https://github.com/Azure/azure-sdk-tools):

* Get-AzureMediaServices 

    Por ejemplo: `Get-AzureMediaServicesAccount`
* New-AzureMediaServicesAccount 
  
    Por ejemplo: `New-AzureMediaServicesAccount -Name "MediaAccountName" -Location "Region" -StorageAccountName "StorageAccountName"`
* New-AzureMediaServicesKey 
  
    Por ejemplo: `New-AzureMediaServicesKey -Name "MediaAccountName" -KeyType Secondary -Force`
* Remove-AzureMediaServicesAccount 
  
    Por ejemplo: `Remove-AzureMediaServicesAccount -Name "MediaAccountName" -Force`

## <a name="june-2013-release"></a><a id="june_changes_13"></a>Versión de junio de 2013
### <a name="media-services-changes"></a><a name="june_13_general_changes"></a>Cambios en Media Services
Los cambios siguientes mencionados en esta sección son actualizaciones incluidas en las versiones de Media Services de junio de 2013:

* Posibilidad de vincular varias cuentas de almacenamiento a una cuenta de Servicios multimedia. 
    * StorageAccount
    * Asset.StorageAccountName and Asset.StorageAccount
* Posibilidad de actualizar Job.Priority. 
* Entidades y propiedades relacionadas con notificaciones: 
    * JobNotificationSubscription
    * NotificationEndPoint
    * Trabajo
* Asset.Uri 
* Locator.Name 

### <a name="media-services-net-sdk-changes"></a><a name="june_13_dotnet_changes"></a>Cambios en el SDK de Media Services para .NET
Los siguientes cambios se incluyen en las versiones del SDK de Media Services de junio de 2013. El último SDK de Media Services está disponible en GitHub.

* A partir de la versión 2.3.0.0, el SDK de Media Services admite la vinculación de varias cuentas de almacenamiento a una cuenta de Media Services. Las siguientes API admiten esta característica:
  
    * El tipo IStorageAccount
    * La propiedad Microsoft.WindowsAzure.MediaServices.Client.CloudMediaContext.StorageAccounts
    * La propiedad StorageAccount
    * La propiedad StorageAccountName
  
      Para más información, consulte [Administración de recursos de Media Services entre varias cuentas de almacenamiento](/previous-versions/azure/dn271889(v=azure.100)).
* API relacionadas con notificaciones. A partir de la versión 2.2.0.0, tiene la posibilidad de escuchar notificaciones de Azure Queue Storage. Para más información, consulte [Control de notificaciones de trabajos de Media Services](/previous-versions/azure/dn261241(v=azure.100)).
  
    * La propiedad Microsoft.WindowsAzure.MediaServices.Client.IJob.JobNotificationSubscriptions
    * El tipo Microsoft.WindowsAzure.MediaServices.Client.INotificationEndPoint
    * El tipo Microsoft.WindowsAzure.MediaServices.Client.IJobNotificationSubscription
    * El tipo Microsoft.WindowsAzure.MediaServices.Client.NotificationEndPointCollection
    * El tipo Microsoft.WindowsAzure.MediaServices.Client.NotificationEndPointType
* Dependencia del SDK del cliente de Storage 2.0 (Microsoft.WindowsAzure.StorageClient.dll)
* Dependencia de OData 5.5 (Microsoft.Data.OData.dll)

## <a name="december-2012-release"></a><a id="december_changes_12"></a>Versión de diciembre de 2012
### <a name="media-services-net-sdk-changes"></a><a name="dec_12_dotnet_changes"></a>Cambios en el SDK de Media Services para .NET
* IntelliSense: Se ha agregado documentación de IntelliSense que faltaba para muchos tipos.
* Microsoft.Practices.TransientFaultHandling.Core: Se ha corregido un problema en el que el SDK tenía aún una dependencia con una versión anterior de este ensamblado. El SDK hace referencia ahora a la versión 5.1.1209.1 de este ensamblado.

Correcciones de problemas encontrados en el SDK de noviembre de 2012:

* IAsset.Locators.Count: Este recuento se notifica ahora correctamente en las nuevas interfaces IAsset después de que todos los localizadores se hayan eliminado.
* IAssetFile.ContentFileSize: Este valor está ahora correctamente definido después de una carga por IAssetFile.Upload(filepath).
* IAssetFile.ContentFileSize: Esta propiedad se puede establecer ahora al crear un archivo de recurso. Anteriormente era de solo lectura.
* IAssetFile.Upload(filepath): Se ha corregido un problema en el que el método de carga sincrónico mostraba el siguiente error al cargar varios archivos en el recurso. El error era "El servidor no pudo autenticar la solicitud. Asegúrese de que el valor del encabezado de autenticación tenga el formato correcto, incluida la firma".
* IAssetFile.UploadAsync: Se corrigió un problema que limitaba la carga simultánea de archivos a cinco archivos.
* IAssetFile.UploadProgressChanged: Ahora el SDK proporciona este evento.
* IAssetFile.DownloadAsync(string, BlobTransferClient, ILocator, CancellationToken): Ahora se proporciona esta sobrecarga del método.
* IAssetFile.DownloadAsync: Se corrigió un problema que limitaba la descarga simultánea de archivos a cinco archivos.
* IAssetFile.Delete(): Se ha corregido un problema en el que al llamar al método delete se puede producir una excepción si no se ha cargado ningún archivo para IAssetFile.
* Jobs: Se ha corregido un problema en el que al encadenar una "tarea de MP4 a Smooth Streams" con una "tarea de protección de PlayReady" mediante una plantilla de trabajo, no se creaba ninguna tarea.
* EncryptionUtils.GetCertificateFromStore(): Este método ya no produce una excepción de referencia nula debido a un error de búsqueda del certificado basado en problemas de configuración del certificado.

## <a name="november-2012-release"></a><a id="november_changes_12"></a>Versión de noviembre de 2012
Los cambios mencionados en esta sección eran actualizaciones incluidas en el SDK de noviembre de 2012 (versión 2.0.0.0). Estos cambios pueden requerir la modificación o reescritura de cualquier código escrito para la versión preliminar del SDK de junio de 2012.

* Recursos
  
    * IAsset.Create(assetName) es la *única* función de creación de recursos. IAsset.Create ya no carga archivos como parte de la llamada al método. Use IAssetFile para cargar.
    * El método IAsset.Publish y el valor de enumeración AssetState.Publish se han eliminado del SDK de Media Services. Todo código que dependa de este valor se debe reescribir.
* FileInfo
  
    * Esta clase se ha eliminado y se ha sustituido por IAssetFile.
  
* IAssetFiles
  
    * IAssetFile sustituye a FileInfo y tiene un comportamiento diferente. Para usarla, cree una instancia del objeto IAssetFiles, seguida de una carga de archivos mediante el SDK de Media Services o el SDK de Storage. Se pueden usar las siguientes sobrecargas de IAssetFile.Upload:
  
        * IAssetFile.Upload(filePath): Este método sincrónico bloquea el subproceso y solo se recomienda al cargar un único archivo.
        * IAssetFile.UploadAsync(filePath, blobTransferClient, locator, cancellationToken): Este método asincrónico es el mecanismo de carga preferido. 
    
            Problema conocido: Si usa el token de cancelación, se cancela la carga. Las tareas pueden tener muchos estados de cancelación. Debe capturar y gestionar correctamente las excepciones.
* Localizadores
  
    * Las versiones específicas del origen se han eliminado. El comando context.Locators.CreateSasLocator(asset, accessPolicy) específico de SAS se marcará como obsoleto o se eliminará de la disponibilidad general. Consulte la sección "Localizadores" en "Nuevas funcionalidades" para información sobre el comportamiento actualizado.

## <a name="june-2012-preview-release"></a><a id="june_changes_12"></a>Versión preliminar de junio de 2012
La siguiente funcionalidad era nueva en la versión de noviembre del SDK:

* Eliminación de entidades
  
    * Los objetos IAsset, IAssetFile, ILocator, IAccessPolicy e IContentKey se han eliminado ahora en el nivel de objeto, es decir, IObject.Delete(), en lugar de requerir una eliminación en la colección, que es cloudMediaContext.ObjCollection.Delete(objInstance).
* Localizadores
  
    * Ahora los localizadores deben crearse mediante el método CreateLocator. Se deben usar los valores de enumeración LocatorType.SAS o LocatorType.OnDemandOrigin como argumento para el tipo específico de localizador que desea crear.
    * Se han agregado nuevas propiedades a los localizadores para que sea más fácil obtener identificadores URI utilizables para su contenido. Este rediseño de los localizadores proporciona una mayor flexibilidad para la futura extensibilidad de terceros y aumenta la facilidad de uso de las aplicaciones cliente multimedia.
* Compatibilidad con el método asincrónico
  
    * Se ha agregado compatibilidad asincrónica a todos los métodos.

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

<!-- Anchors. -->

<!-- Images. -->

<!--- URLs. --->
[Microsoft Q&A question page for Azure Media Services]: /answers/topics/azure-media-services.html
[Referencia de la API de REST de Azure Media Services]: /rest/api/media/operations/azure-media-services-rest-api-reference
[Media Services pricing details]: https://azure.microsoft.com/pricing/details/media-services/
[Metadatos de entrada]: ./media-services-input-metadata-schema.md
[Metadatos de salida]: ./media-services-output-metadata-schema.md
[Deliver content]: /previous-versions/azure/hh973618(v=azure.100)
[Index media files with the Azure Media Indexer]: /previous-versions/azure/dn783455(v=azure.100)
[StreamingEndpoint]: /rest/api/media/operations/streamingendpoint
[Work with Media Services live streaming]: /previous-versions/azure/dn783466(v=azure.100)
[Use AES-128 dynamic encryption and the key delivery service]: /previous-versions/azure/dn783457(v=azure.100)
[Use PlayReady dynamic encryption and the license delivery service]: /previous-versions/azure/dn783467(v=azure.100)
[Preview features]: https://azure.microsoft.com/services/preview/
[Introducción a la plantilla de licencias de PlayReady de Media Services]: /previous-versions/azure/dn783459(v=azure.100)
[Stream storage-encrypted content]: /previous-versions/azure/dn783451(v=azure.100)
[Azure portal]: https://portal.azure.com
[Empaquetado dinámico]: /previous-versions/azure/jj889436(v=azure.100)
[Nick Drouin's blog]: http://blog-ndrouin.azurewebsites.net/hls-v3-new-old-thing/
[Protect Smooth Streaming with PlayReady]: /previous-versions/azure/dn189154(v=azure.100)
[Lógica de reintento en el SDK de Media Services para .NET]: ./media-services-retry-logic-in-dotnet-sdk.md
[Grass Valley announces EDIUS 7 streaming through the cloud]: https://www.streamingmedia.com/Producer/Articles/ReadArticle.aspx?ArticleID=96351&utm_source=dlvr.it&utm_medium=twitter
[Control Media Services Encoder output file names]: /previous-versions/azure/dn303341(v=azure.100)
[Create overlays]: /previous-versions/azure/dn640496(v=azure.100)
[Stitch video segments]: /previous-versions/azure/dn640504(v=azure.100)
[Azure Media Services .NET SDK 3.0.0.1 and 3.0.0.2 releases]: http://www.gtrifonov.com/2014/02/07/windows-azure-media-services-.net-sdk-3.0.0.2-release/
[Azure AD Access Control Service]: /previous-versions/azure/azure-services/hh147631(v=azure.100)
[Connect to Media Services with the Media Services SDK for .NET]: /previous-versions/azure/jj129571(v=azure.100)
[Media Services .NET SDK extensions]: https://github.com/Azure/azure-sdk-for-media-services-extensions/tree/dev
[Azure SDK tools]: https://github.com/Azure/azure-sdk-tools
[GitHub]: https://github.com/Azure/azure-sdk-for-media-services
[Manage Media Services assets across multiple Storage accounts]: /previous-versions/azure/dn271889(v=azure.100)
[Handle Media Services job notifications]: /previous-versions/azure/dn261241(v=azure.100)