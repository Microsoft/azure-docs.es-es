---
title: Preguntas más frecuentes de Azure Media Services v3 | Microsoft Docs
description: En este artículo se ofrecen respuestas a preguntas frecuentes sobre Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: d34b5aaaa12a3d296f92e0d7be34ae76931d8506
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89268492"
---
# <a name="media-services-v3-frequently-asked-questions"></a>Preguntas más frecuentes sobre Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En este artículo se ofrecen respuestas a preguntas frecuentes sobre Azure Media Services v3.

## <a name="general"></a>General

### <a name="what-are-the-azure-portal-limitations-for-media-services-v3"></a>¿Qué son las limitaciones de Azure Portal para Media Services v3?

Puede usar [Azure Portal](https://portal.azure.com/) para administrar los eventos en directo de la versión 3, ver los recursos y trabajos de la versión 3, obtener información sobre el acceso a las API y cifrar contenido. <br/>Para las tareas de administración restantes (por ejemplo, administración de transformaciones y trabajos o análisis de contenido de la versión 3), use la [API REST](https://aka.ms/ams-v3-rest-ref), la [CLI](https://aka.ms/ams-v3-cli-ref), o uno de los [SDK](media-services-apis-overview.md#sdks) compatibles.

Si el vídeo se ha cargado previamente en la cuenta de Media Services con la API de Media Services v3 o si el contenido se ha generado a partir de una salida en directo, no verá los botones **Codificar**, **Analizar** o **Cifrar** en Azure Portal. Use las API de Media Services v3 para realizar estas tareas.  

### <a name="what-azure-roles-can-perform-actions-on-azure-media-services-resources"></a>¿Qué roles de Azure pueden realizar acciones en recursos de Azure Media Services? 

Vea [Control de acceso basado en roles (RBAC) para cuentas de Media Services](rbac-overview.md).

### <a name="how-do-i-stream-to-apple-ios-devices"></a>¿Cómo hago streaming a dispositivos de Apple iOS?

Asegúrese de que tiene **(format=m3u8-aapl)** al final de la ruta de acceso (después de la parte **/manifest** de la dirección URL) para indicar al servidor de origen de streaming que devuelva el contenido de HTTP Live Streaming (HLS) para su consumo en dispositivos Apple iOS nativos. Para más información, consulte [Entrega de contenido](dynamic-packaging-overview.md).

### <a name="how-do-i-configure-media-reserved-units"></a>¿Cómo se pueden configurar las unidades reservadas de multimedia?

En trabajos de análisis de audio y vídeo desencadenados por Media Services v3 o Video Indexer, se recomienda que aprovisione la cuenta con diez unidades reservadas de multimedia (MRU) S3. Si necesita más de diez unidades MRU S3, abra una incidencia de soporte técnico mediante [Azure Portal](https://portal.azure.com/).

Para más información, consulte [Escalado de procesamiento de elementos multimedia](media-reserved-units-cli-how-to.md).

### <a name="what-is-the-recommended-method-to-process-videos"></a>¿Cuál es el método recomendado para procesar vídeos?

Use [transformaciones](/rest/api/media/transforms) para configurar tareas comunes para codificar o analizar vídeos. Cada Transformación describe una receta, o un flujo de trabajo simple de tareas para procesar los archivos de vídeo o audio. Un [trabajo](/rest/api/media/jobs) es la solicitud real a Media Services de aplicar la transformación a un contenido de vídeo o audio de entrada. Después de crear la transformación, puede enviar trabajos mediante las API de Media Services o cualquiera de los SDK publicados. Para obtener más información, consulte [Transformaciones y trabajos](transforms-jobs-concept.md).

### <a name="i-uploaded-encoded-and-published-a-video-why-wont-the-video-play-when-i-try-to-stream-it"></a>He cargado, codificado y publicado un vídeo. ¿Por qué no se reproduce el vídeo cuando intento transmitirlo?

Uno de los motivos más habituales es que no tiene el punto de conexión de streaming desde el que está intentando reproducir en estado Ejecutando.

### <a name="how-does-pagination-work"></a>¿Cómo funciona la paginación?

Al usar la paginación, siempre debe usar el vínculo siguiente para enumerar la colección y no tener que depender de un tamaño de página determinado. Para obtener información detallada y ejemplos, consulte [Filtrado, ordenación y paginación](entities-overview.md).

### <a name="what-features-are-not-yet-available-in-azure-media-services-v3"></a>¿Qué características todavía no están disponibles en Azure Media Services v3?

Para obtener detalles, vea [Carencias de características con respecto a las API v2](media-services-v2-vs-v3.md#feature-gaps-with-respect-to-v2-apis).

### <a name="what-is-the-process-of-moving-a-media-services-account-between-subscriptions"></a>¿Cómo se traslada una cuenta de Media Services entre suscripciones?  

Para más información, consulte [Traslado de una cuenta de Media Services entre suscripciones](media-services-account-concept.md).

## <a name="live-streaming"></a>Streaming en directo 

### <a name="how-do-i-stop-the-live-stream-after-the-broadcast-is-done"></a>¿Cómo detengo el streaming en vivo una vez que termine la difusión?

Puede abordarlo desde el lado cliente o desde el lado servidor.

#### <a name="client-side"></a>En el cliente

La aplicación web debería preguntarle al usuario si quiere finalizar la difusión al cerrar el explorador. Se trata de un evento del explorador que la aplicación web puede controlar.

#### <a name="server-side"></a>En el servidor

Puede supervisar eventos en directo si se suscribe a eventos de Azure Event Grid. Para más información, consulte el [esquema de eventos de Event Grid](media-services-event-schemas.md#live-event-types).

Puede:

* [Suscribirse](reacting-to-media-services-events.md) a los eventos [Microsoft.Media.LiveEventEncoderDisconnected](media-services-event-schemas.md#liveeventencoderdisconnected) de nivel de flujo y supervisar que no se produzca ninguna reconexión durante un tiempo para detener y eliminar el evento en directo.
* [Suscribirse](reacting-to-media-services-events.md) a los eventos de [latido](media-services-event-schemas.md#liveeventingestheartbeat) de nivel de pista. Si todas las pistas tienen una velocidad de bits de entrada que cae a 0, o bien si la última marca de tiempo ya no aumenta, puede apagar el evento en directo de manera segura. Los eventos de latido ocurren cada veinte segundos por cada pista, por lo que podría ser un poco detallado.

###  <a name="how-do-i-insert-breaksvideos-and-image-slates-during-a-live-stream"></a>¿Cómo inserto pausas o vídeos y caretas de imagen durante una transmisión en directo?

La codificación en directo de Media Services v3 no admite aún la inserción de caretas de vídeo o de imagen durante una transmisión en directo. 

Puede usar un [codificador local en directo](recommended-on-premises-live-encoders.md) para cambiar el vídeo de origen. Muchas aplicaciones proporcionan la posibilidad de cambiar los orígenes como, por ejemplo, Telestream Wirecast, Switcher Studio (en iOS) y OBS Studio (aplicación gratuita).

## <a name="content-protection"></a>Protección de contenido

### <a name="should-i-use-aes-128-clear-key-encryption-or-a-drm-system"></a>¿Debo usar cifrado de clave sin cifrado AES-128 o un sistema DRM?

Los clientes suelen preguntarse si deben usar el cifrado de AES o un sistema de DRM. La diferencia principal entre los dos sistemas es que, con el cifrado de AES, la clave de contenido se transmite al cliente sobre TLS para que la clave se cifre en tránsito pero sin un cifrado adicional ("sin cifrado"). Como resultado, la clave que se usa para descifrar el contenido está accesible en el reproductor del cliente y se puede ver en un seguimiento de la red en el cliente en texto sin formato. La clave sin cifrado AES-128 es adecuada para los casos de uso en los que el destinatario es una entidad de confianza (p. ej., cifrado de vídeos corporativos distribuidos dentro de una empresa para su visualización por parte de los empleados).

Los sistemas DRM como PlayReady, Widevine y FairPlay proporcionan un nivel adicional de cifrado en la clave que se usa para descifrar el contenido en comparación con una clave sin cifrado AES-128. La clave de contenido se cifra en una clave protegida por el entorno de ejecución de DRM, además de cualquier cifrado de nivel de transporte proporcionado por TLS. Además, el descifrado se controla en un entorno seguro en el nivel de sistema operativo donde a un usuario malintencionado le resulta más difícil atacar. Se recomienda DRM para los casos de uso en los que es posible que el destinatario no sea una entidad de confianza y se requiera el nivel de seguridad más alto.

### <a name="how-do-i-show-a-video-to-only-users-who-have-a-specific-permission-without-using-azure-ad"></a>¿Cómo muestro un vídeo solo a los usuarios que tienen un permiso específico, sin usar Azure AD?

No tiene que usar ningún proveedor de token específico como Azure Active Directory (Azure AD). Puede crear su propio proveedor [JWT](https://jwt.io/) (denominado servicio de token de seguridad o STS) mediante el cifrado de claves asimétricas. En el STS personalizado, puede agregar notificaciones basadas en la lógica de negocios.

Asegúrese de que el emisor, el público y las notificaciones coincidan exactamente entre lo que hay en JWT y el valor de `ContentKeyPolicyRestriction` que se usa en `ContentKeyPolicy`.

Para más información, consulte [Protección del contenido mediante el cifrado dinámico de Media Services](content-protection-overview.md).

### <a name="how-and-where-did-i-get-a-jwt-token-before-using-it-to-request-a-license-or-key"></a>¿Cómo y dónde obtuve un token JWT antes de usarlo para solicitar una licencia o clave?

Para entornos de producción, deberá tener un servicio de token seguro (que es un servicio web), que emite un token JWT tras una solicitud HTTPS. Para la prueba, puede usar el código que se muestra en el método `GetTokenAsync` definido en [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs).

Una vez que se autentica un usuario, el reproductor solicita a STS dicho tipo de token y lo asigna como el valor del token. Puede usar la [API de Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/).

Para un ejemplo de cómo ejecutar STS, ya sea con un clave simétrica o asimétrica, consulte la [herramienta JWT](https://aka.ms/jwt). Para un ejemplo de un reproductor basado en Azure Media Player usando dicho token JWT, consulte la [herramienta de prueba multimedia de Azure](https://aka.ms/amtest). (Expanda el vínculo **player_settings** para ver la entrada de token).

### <a name="how-do-i-authorize-requests-to-stream-videos-with-aes-encryption"></a>¿Cómo autorizo las solicitudes para transmitir vídeos con cifrado AES?

El enfoque correcto consiste en usar el servicio de token seguro. Según el perfil de usuario, agregue en STS notificaciones distintas (por ejemplo, "Usuario premium", "Usuario básico", "Usuario de evaluación gratuita"). Con notificaciones distintas en un token JWT, el usuario puede ver diferentes contenidos. Para los distintos contenidos o activos, `ContentKeyPolicyRestriction` tendrá el valor de `RequiredClaims` correspondiente.

Use las API de Azure Media Services para configurar la entrega de claves o licencias y cifrar los recursos (como se muestra en [este ejemplo](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES/Program.cs)).

Para más información, consulte:

- [Introducción a la protección de contenido](content-protection-overview.md)
- [Diseño del sistema de protección de contenido con DRM múltiple con control de acceso](design-multi-drm-system-with-access-control.md)

### <a name="should-i-use-http-or-https"></a>¿Debo usar HTTP o HTTPS?
La aplicación de reproductor de ASP.NET MVC debe ser compatible con lo siguiente:

* Autenticación de usuarios mediante Azure AD, que está en HTTPS.
* Intercambio de tokens JWT entre el cliente y Azure AD, que debe estar en HTTPS.
* La adquisición de licencias DRM por el cliente, que debe estar en HTTPS si Media Services proporciona la entrega de licencias. El conjunto de productos de PlayReady no impone HTTPS para la entrega de licencias. Si el servidor de licencias de PlayReady está fuera de Media Services, se podría usar HTTP o HTTPS.

La aplicación de reproductor de ASP. NET utiliza HTTPS como procedimiento recomendado, por lo que Media Player se encuentra en una página en HTTPS. Sin embargo, HTTP es preferible para el streaming, por lo que se deben tener en cuenta estas cuestiones con contenido mixto:

* El explorador no permite contenido mixto. Sin embargo, los complementos como Silverlight y OSMF para Smooth y DASH sí lo hacen. El contenido mixto es un problema de seguridad debido a la amenaza de la posibilidad de insertar JavaScript malintencionado, que puede poner en riesgo los datos del cliente. Los exploradores lo bloquean de forma predeterminada. La única manera de solucionar esto es en el servidor (origen) al permitir todos los dominios (con independencia de si son HTTPS o HTTP). Probablemente esto tampoco sea una buena idea.
* Evite el contenido mixto. Tanto la aplicación de reproducción y Media Player deben utilizar HTTP o HTTPS. Al reproducir contenido mixto, SilverlightSS tech requiere que se borre una advertencia de contenido mixto. FlashSS tech controla el contenido mixto sin advertencia de contenido mixto.
* Si su punto de conexión de streaming se ha creado antes de agosto de 2014, no admitirá HTTPS. En este caso, cree y utilice un nuevo punto de conexión de streaming para HTTPS.

### <a name="what-about-live-streaming"></a>¿Y qué pasa con el streaming en vivo?

Puede usar exactamente el mismo diseño y la misma implementación para ayudar a proteger el streaming en vivo en Media Services; solo hay que tratar el recurso asociado a un programa como un recurso VoD. Para proporcionar una protección de varias DRM del contenido en directo, aplique la misma configuración o procesamiento al recurso como si fuera un recurso VoD antes de asociar el recurso con la salida en directo.

### <a name="what-about-license-servers-outside-media-services"></a>¿Y qué sucede con los servidores de licencias que están fuera de Media Services?

Con frecuencia, los clientes han invertido en granjas de servidores de licencias en su propio centro de datos o en uno hospedado por proveedores de servicios DRM. Con la protección de contenido de Media Services, puede funcionar en modo híbrido. El contenido puede alojarse y protegerse dinámicamente en Media Services, mientras que las licencias DRM las entregan servidores externos a Media Services. En este caso, tenga en cuenta los siguientes cambios:

* El servicio de token de seguridad debe emitir tokens que sean aceptables y que se puedan comprobar en la granja de servidores de licencias. Por ejemplo, los servidores de licencias de Widevine proporcionados por Axinom requieren un token JWT específico que contenga un mensaje de derechos. Debe tener un servicio de token de seguridad para emitir dicho token JWT. 
* Ya no necesitará configurar el servicio de entrega de licencias en Media Services. Debe proporcionar las direcciones URL de adquisición de licencias (para PlayReady, Widevine y FairPlay) cuando configura `ContentKeyPolicy`.

> [!NOTE]
> Widevine es un servicio que ofrece Google y que está sujeto a los términos del servicio y la directiva de privacidad de Google.

## <a name="media-services-v2-vs-v3"></a>Versiones 2 y 3 de Media Services 

### <a name="can-i-use-the-azure-portal-to-manage-v3-resources"></a>¿Puedo usar Azure Portal para administrar los recursos de v3?

Actualmente, se puede usar [Azure Portal](https://portal.azure.com/) para:

* Administrar los [eventos en directo](live-events-outputs-concept.md) de la versión 3 de Media Services. 
* Ver (no administrar) los [recursos](assets-concept.md) de la versión 3. 
* [Obtener información sobre el acceso a las API](./access-api-howto.md). 

Para las restantes tareas de administración (por ejemplo, [Transformaciones y trabajos](transforms-jobs-concept.md) y [Protección de contenido](content-protection-overview.md)), use la [API REST](/rest/api/media/), la [CLI de Azure](https://aka.ms/ams-v3-cli-ref), o uno de los [SDK](media-services-apis-overview.md#sdks) compatibles.

### <a name="is-there-an-assetfile-concept-in-v3"></a>¿Existe el concepto de AssetFile en la versión v3?

El concepto `AssetFile` se quitó de la API de Media Services para separar Media Services de la dependencia del SDK de Storage. Ahora Azure Storage y no Media Services mantiene la información que pertenece al SDK de Storage. 

Para más información, consulte [Migración a Media Services v3](media-services-v2-vs-v3.md).

### <a name="where-did-client-side-storage-encryption-go"></a>¿Qué hay del cifrado del almacenamiento del lado del cliente?

Ahora se recomienda utilizar el cifrado de almacenamiento en el lado de servidor (que está activo de forma predeterminada). Para más información, consulte [Cifrado del servicio Azure Storage para datos en reposo](../../storage/common/storage-service-encryption.md).

## <a name="offline-streaming"></a>Streaming sin conexión

### <a name="fairplay-streaming-for-ios"></a>Streaming de FairPlay para iOS

Las preguntas más frecuentes que aparecen a continuación proporcionan ayuda para solucionar los problemas con el streaming sin conexión de FairPlay para iOS.

#### <a name="why-does-only-audio-play-but-not-video-during-offline-mode"></a>¿Por qué solo se reproduce el audio pero no el vídeo en el modo sin conexión?

Este comportamiento parece deberse al diseño de la aplicación de ejemplo. Cuando hay una pista de audio alternativo presente (como es el caso de HLS), durante el modo sin conexión, tanto iOS 10 como iOS 11 usan de manera predeterminada la pista de audio alternativo. Para compensar este comportamiento para el modo sin conexión de FPS, quite la pista de audio alternativo de la secuencia. Para hacer esto en Media Services, agregue el filtro de manifiesto dinámico **audio-only=false**. En otras palabras, una dirección URL de HLS finaliza con **.ism/manifest(format=m3u8-aapl,audio-only=false)** . 

#### <a name="why-does-it-still-play-audio-only-without-video-during-offline-mode-after-i-add-audio-onlyfalse"></a>¿Por qué se sigue reproduciendo solo audio sin vídeo durante el modo sin conexión después de agregar audio-only=false?

Según cómo esté diseñada la clave de caché para la red de entrega de contenido, es posible que el contenido se almacene en la memoria caché. Debe purgar la caché.

#### <a name="is-fps-offline-mode-supported-on-ios-11-in-addition-to-ios-10"></a>¿El modo sin conexión de FPS es compatible con iOS 11 además de iOS 10?

Sí. El modo sin conexión de FPS es compatible tanto con iOS 10 como con iOS 11.

#### <a name="why-cant-i-find-the-document-offline-playback-with-fairplay-streaming-and-http-live-streaming-in-the-fps-server-sdk"></a>¿Por qué no encuentro el documento "Reproducción sin conexión con FairPlay Streaming y HTTP Live Streaming" en el SDK de FPS Server?

A partir de la versión 4 del SDK de FPS Server, este documento se combinó con el documento relativo a la guía de programación de streaming de FairPlay.

#### <a name="what-is-the-downloadedoffline-file-structure-on-ios-devices"></a>¿Cuál es la estructura del archivo descargado o sin conexión en dispositivos iOS?

La estructura del archivo descargado en un dispositivo iOS es similar a la captura de pantalla siguiente. La carpeta `_keys` almacena licencias FPS descargadas, con un archivo de almacén para cada host de servicio de licencia. La carpeta `.movpkg` almacena el contenido de audio y vídeo. 

La primera carpeta, cuyo nombre finaliza con un guion seguido de un número, contiene datos de vídeo. El valor numérico es el ancho de banda máximo de la reproducción de vídeo. La segunda carpeta, cuyo nombre finaliza con un guion seguido de un 0, contiene datos de audio. La tercera carpeta, denominada `Data`, contiene la lista de reproducción maestra del contenido FPS. Por último, boot.xml proporciona una descripción completa del contenido de la carpeta `.movpkg`. 

![Estructura del archivo sin conexión para la aplicación de ejemplo de iOS de FairPlay](media/offline-fairplay-for-ios/offline-fairplay-file-structure.png)

Este es archivo boot.xml de ejemplo:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<HLSMoviePackage xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://apple.com/IMG/Schemas/HLSMoviePackage" xsi:schemaLocation="http://apple.com/IMG/Schemas/HLSMoviePackage /System/Library/Schemas/HLSMoviePackage.xsd">
  <Version>1.0</Version>
  <HLSMoviePackageType>PersistedStore</HLSMoviePackageType>
  <Streams>
    <Stream ID="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" Path="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(127000)/Manifest(aac_eng_2_127,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
    <Stream ID="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" Path="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(161000)/Manifest(video,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
  </Streams>
  <MasterPlaylist>
    <NetworkURL>https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/manifest(format=m3u8-aapl,audio-only=false)</NetworkURL>
  </MasterPlaylist>
  <DataItems Directory="Data">
    <DataItem>
      <ID>CB50F631-8227-477A-BCEC-365BBF12BCC0</ID>
      <Category>Playlist</Category>
      <Name>master.m3u8</Name>
      <DataPath>Playlist-master.m3u8-CB50F631-8227-477A-BCEC-365BBF12BCC0.data</DataPath>
      <Role>Master</Role>
    </DataItem>
  </DataItems>
</HLSMoviePackage>
```

### <a name="widevine-streaming-for-android"></a>Streaming de Widevine para Android

#### <a name="how-can-i-deliver-persistent-licenses-offline-enabled-for-some-clientsusers-and-non-persistent-licenses-offline-disabled-for-others-do-i-have-to-duplicate-the-content-and-use-separate-content-keys"></a>¿Cómo se pueden entregar licencias persistentes (habilitadas para modo sin conexión) para algunos clientes/usuarios y licencias no persistentes (deshabilitadas para modo sin conexión) para otros? ¿Es necesario duplicar el contenido y usar las claves de contenido independiente?

Dado que Media Services v3 permite que un recurso tenga varias instancias de `StreamingLocator`, puede tener:

* Una instancia de `ContentKeyPolicy` con `license_type = "persistent"`, `ContentKeyPolicyRestriction` con notificaciones en `"persistent"` y su `StreamingLocator`.
* Otra instancia de `ContentKeyPolicy` con `license_type="nonpersistent"`, `ContentKeyPolicyRestriction` con notificaciones en `"nonpersistent` y su `StreamingLocator`.
* Dos instancias de `StreamingLocator` que tengan diferentes valores de `ContentKey`.

Dependiendo de la lógica de negocio de la STS personalizada, se emiten diferentes notificaciones en el token de JWT. Con el token, solo se puede obtener la licencia correspondiente y solo se puede reproducir la dirección URL correspondiente.

#### <a name="what-is-the-mapping-between-the-widevine-and-media-services-drm-security-levels"></a>¿Cuál es la asignación entre los niveles de seguridad de DRM de Widevine y Media Services?

En el documento "Información general de la arquitectura de DRM de Widevine" de Google se definen tres niveles de seguridad: Sin embargo, la [documentación de Azure Media Services en la plantilla de licencias de Widevine](widevine-license-template-overview.md) describe cinco niveles de seguridad (requisitos de solidez del cliente para la reproducción). En esta sección se explica cómo se asignan los niveles de seguridad.

Los dos conjuntos de niveles de seguridad se definen mediante Google Widevine. La diferencia radica en el nivel de uso: arquitectura o API. Los niveles de cinco seguridad se utilizan en la API de Widevine. El objeto `content_key_specs`, que contiene `security_level`, se deserializa y se pasa al servicio de entrega global de Widevine por parte del servicio de licencias de Widevine de Azure Media Services. En la tabla siguiente se muestra la asignación entre los dos conjuntos de niveles de seguridad.

| **Niveles de seguridad definidos en la arquitectura de Widevine** |**Niveles de seguridad utilizados en la API de Widevine**|
|---|---| 
| **Nivel de seguridad 1**: todo el procesamiento de contenido, la criptografía y el control se realizan en un entorno de ejecución de confianza (TEE). En algunos modelos de implementación, el procesamiento de seguridad podría realizarse en chips diferentes.|**security_level=5**: La criptografía, la descodificación y todo el tratamiento de los medios (comprimidos y descomprimidos) deben administrarse dentro de un TEE con respaldo de hardware.<br/><br/>**security_level=4**: La criptografía y la descodificación del contenido deben realizarse dentro de un TEE con respaldo de hardware.|
**Nivel de seguridad 2**: La criptografía (pero no el procesamiento de vídeo) se realiza dentro de TEE. Los búferes descifrados se devuelven al dominio de aplicación y se procesan a través de software o hardware de vídeo independiente. En el nivel 2, sin embargo, la información criptográfica se sigue procesando solo dentro de un TEE.| **security_level=3**: Las operaciones de criptografía y material clave deben realizarse en un TEE con respaldo de hardware. |
| **Nivel de seguridad 3**: No hay ningún TEE en el dispositivo. Se pueden adoptar las medidas adecuadas para proteger la información criptográfica y el contenido descifrado en el sistema operativo del host. Una implementación de nivel 3 también podría incluir un motor de cifrado de hardware, pero esto solo mejora el rendimiento, no la seguridad. | **security_level=2**: Se requiere criptografía de software y un descodificador de ofuscación.<br/><br/>**security_level=1**: Se requiere criptografía white-box basada en software.|

#### <a name="why-does-content-download-take-so-long"></a>¿Por qué la descarga de contenido tarda tanto tiempo?

Hay dos maneras de mejorar la velocidad de descarga:

* Habilite una red de entrega de contenido para que los usuarios tengan más probabilidades de utilizarla en lugar del punto de conexión de streaming u origen para la descarga de contenido. Si un usuario utiliza un punto de conexión de streaming, cada segmento HLS o fragmento DASH se empaqueta y cifra dinámicamente. Aunque esta latencia se encuentre en escala de milisegundos para cada segmento o fragmento, si tiene un vídeo de una hora de duración, la latencia acumulada puede ser grande y ocasionar una descarga más larga.
* Proporcione a los usuarios la opción de descargar de forma selectiva las capas de la calidad del vídeo y las pistas de audio en lugar de todo el contenido. En el caso del modo sin conexión, no tiene sentido descargar todas las capas de calidad. Hay dos formas de lograrlo:

  * Controlado por el cliente: la aplicación del reproductor selecciona automáticamente, o el usuario selecciona, la capa de calidad de vídeo y las pistas de audio para descargar.
  * Controlado por el servicio: puede utilizar la característica Manifiesto dinámico de Azure Media Services para crear un filtro (global), que limita la lista de reproducción HLS o el MPD de DASH a una sola capa de calidad del vídeo y las pistas de audio seleccionadas. Así pues, la URL de descarga que se presenta a los usuarios finales incluirá este filtro.

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Media Services v3](media-services-overview.md)
