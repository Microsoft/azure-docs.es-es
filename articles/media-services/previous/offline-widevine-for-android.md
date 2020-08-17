---
title: Configuración de la cuenta para el streaming sin conexión de contenido protegido de Widevine - Azure
description: En este tema se muestra cómo configurar la cuenta de Azure Media Services para streaming sin conexión de contenido protegido de Widevine.
services: media-services
keywords: DASH, DRM, Modo sin conexión de Widevine, ExoPlayer, Android
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/16/2019
ms.author: willzhan
ms.reviewer: dwgeo
ms.openlocfilehash: b9a47ac9af1d96cbd65ed68dbafccefa3b95bb20
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87065490"
---
# <a name="offline-widevine-streaming-for-android"></a>Streaming de Widevine sin conexión para Android  

> [!div class="op_single_selector" title1="Seleccione la versión de Media Services que se está usando:"]
> * [Versión 3](../latest/offline-widevine-for-android.md)
> * [Versión 2](offline-widevine-for-android.md)

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Además de proteger el contenido del streaming en línea, la suscripción de contenido multimedia y los servicios de alquiler ofrecen contenido descargable que funciona cuando no hay conexión a Internet. Tendrá que descargar el contenido en su teléfono o tableta para reproducirlo en modo de avión al volar sin conexión a la red. Estos son otros escenarios en los que la descarga de contenido podría resultarle útil:

- Algunos proveedores de contenido pueden suspender la entrega de licencias de DRM al cruzar la frontera de un país o una región. Si un usuario desea ver contenido mientras viaja en el extranjero, se necesita la descarga sin conexión.
- En algunos países o regiones, la disponibilidad de Internet o de ancho de banda es limitada. Los usuarios pueden decidir descargar contenido para poder verlo en una resolución lo suficientemente alta que les permita disfrutar de una experiencia de visualización satisfactoria.

En este artículo se describe cómo implementar la reproducción en modo sin conexión para contenido de DASH protegido en dispositivos Widevine o Android. La opción de DRM sin conexión le ofrece modelos de suscripción, alquiler y compra para el contenido, permitiendo a los clientes de sus servicios acceder fácilmente a dicho contenido cuando no tienen acceso a Internet.

Para compilar aplicaciones para el reproductor de Android, se describen tres opciones:

> [!div class="checklist"]
> * Compilar un reproductor con la API de Java del SDK de ExoPlayer
> * Compilar un reproductor con el enlace de Xamarin del SDK de ExoPlayer
> * Crear un reproductor con Encrypted Media Extension (EME) y Media Source Extension (MSE) en la versión 62 o posterior del explorador móvil de Chrome

En el artículo también se responden algunas preguntas frecuentes relacionadas con el streaming sin conexión del contenido protegido de Widevine.

## <a name="requirements"></a>Requisitos 

Antes de implementar DRM para Widevine en dispositivos Android, debe hacer lo siguiente:

- Familiarizarse con los conceptos presentados para la protección de contenido en línea con Widevine DRM. Este tema se trata con detalle en los siguientes documentos y ejemplos:
    - [Uso de Azure Media Services para entregar licencias de DRM a claves de AES](media-services-deliver-keys-and-licenses.md)
    - [CENC with Multi-DRM and Access Control (CENC con DRM múltiple y Control de acceso): Un diseño y una implementación de referencia en Azure y Azure Media Services](media-services-cenc-with-multidrm-access-control.md)
    - [Using PlayReady and/or Widevine Dynamic Common Encryption with .NET](https://azure.microsoft.com/resources/samples/media-services-dotnet-dynamic-encryption-with-drm/) (Uso de cifrado dinámico común de PlayReady o Widevine con .NET)
    - [Use Azure Media Services to deliver PlayReady and/or Widevine licenses with .NET](https://azure.microsoft.com/resources/samples/media-services-dotnet-deliver-playready-widevine-licenses/) (Uso de Azure Media Services para ofrecer licencias de PlayReady o Widevine con .NET)
- Familiarícese con el SDK de ExoPlayer de Google para Android, un SDK de reproductor de vídeo de código abierto que admite la reproducción de Widevine DRM sin conexión. 
    - [ExoPlayer SDK](https://github.com/google/ExoPlayer) (SDK de ExoPlayer)
    - [ExoPlayer Developer Guide](https://google.github.io/ExoPlayer/guide.html) (Guía para desarrolladores de ExoPlayer)
    - [EoPlayer Developer Blog](https://medium.com/google-exoplayer) (Blog para desarrolladores de ExoPlayer)

## <a name="content-protection-configuration-in-azure-media-services"></a>Configuración de la protección de contenido en Azure Media Services

Al configurar la protección de Widevine de un recurso en Media Services, debe crear un valor ContentKeyAuthorizationPolicyOption que especifique las tres operaciones siguientes:

1. Sistema DRM (Widevine)
2. ContentKeyAuthorizationPolicyRestriction que especifique cómo se autoriza la entrega de la clave de contenido en el servicio de entrega de licencias (abierta o autorización por token)
3. Plantilla de licencia de DRM (Widevine)

Para habilitar el modo **sin conexión** para las licencias de Widevine, debe configurar la [plantilla de licencias de Widevine](media-services-widevine-license-template-overview.md). En el objeto **policy_overrides**, establezca la propiedad **can_persist** en **true** (el valor predeterminado es false). 

En el ejemplo de código siguiente se utiliza .NET para habilitar el modo **sin conexión** para licencias de Widevine. El código se basa en el ejemplo de [Using PlayReady and/or Widevine Dynamic Common Encryption with .NET](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-drm) (Uso de cifrado dinámico común de PlayReady o Widevine con .NET). 

```
private static string ConfigureWidevineLicenseTemplateOffline(Uri keyDeliveryUrl)
{
    var template = new WidevineMessage
    {
        allowed_track_types = AllowedTrackTypes.SD_HD,
        content_key_specs = new[]
        {
            new ContentKeySpecs
            {
                required_output_protection = new RequiredOutputProtection { hdcp = Hdcp.HDCP_NONE},
                security_level = 1,
                track_type = "SD"
            }
        },
        policy_overrides = new
        {
            can_play = true,
            can_persist = true,
            //can_renew = true,                                //if you set can_renew = false, you do not need renewal_server_url
            //renewal_server_url = keyDeliveryUrl.ToString(),    //not mandatory, renewal_server_url is needed only if license_duration_seconds is set
            can_renew = false,
            //rental_duration_seconds = 1209600,
            //playback_duration_seconds = 1209600,
            //license_duration_seconds = 1209600
        }
        };

    string configuration = Newtonsoft.Json.JsonConvert.SerializeObject(template);
    return configuration;
}
```

## <a name="configuring-the-android-player-for-offline-playback"></a>Configuración del reproductor Android para la reproducción sin conexión

La manera más fácil de desarrollar una aplicación de reproductor nativa para dispositivos Android consiste en usar el [SDK de ExoPlayer de Google](https://github.com/google/ExoPlayer), un SDK de reproductor de vídeo de código abierto. ExoPlayer admite características que no son actualmente compatibles con la API de MediaPlayer nativa de Android, como los protocolos de entrega MPEG-DASH y Smooth Streaming de Microsoft.

La versión 2.6 de ExoPlayer (y posteriores) incluyen muchas clases que admiten la reproducción de Widevine DRM sin conexión. En concreto, la clase OfflineLicenseHelper proporciona funciones de utilidad para facilitar el uso de DefaultDrmSessionManager para descargar, renovar y liberar las licencias sin conexión. Las clases proporcionadas en la carpeta del SDK "library/core/src/main/java/com/google/android/exoplayer2/offline/" admiten la descarga de contenido de vídeo sin conexión.

La siguiente lista de clases facilitan el modo sin conexión en el SDK de ExoPlayer para Android:

- library/core/src/main/java/com/google/android/exoplayer2/drm/OfflineLicenseHelper.java  
- library/core/src/main/java/com/google/android/exoplayer2/drm/DefaultDrmSession.java
- library/core/src/main/java/com/google/android/exoplayer2/drm/DefaultDrmSessionManager.java
- library/core/src/main/java/com/google/android/exoplayer2/drm/DrmSession.java
- library/core/src/main/java/com/google/android/exoplayer2/drm/ErrorStateDrmSession.java
- library/core/src/main/java/com/google/android/exoplayer2/drm/ExoMediaDrm.java
- library/core/src/main/java/com/google/android/exoplayer2/offline/SegmentDownloader.java
- library/core/src/main/java/com/google/android/exoplayer2/offline/DownloaderConstructorHelper.java 
- library/core/src/main/java/com/google/android/exoplayer2/offline/Downloader.java
- library/dash/src/main/java/com/google/android/exoplayer2/source/dash/offline/DashDownloader.java 

Los desarrolladores deben consultar [ExoPlayer Developer Guide](https://google.github.io/ExoPlayer/guide.html) (Guía para desarrolladores de ExoPlayer) y el correspondiente [Developer Blog](https://medium.com/google-exoplayer) (Blog de desarrolladores) durante el desarrollo de una aplicación. Google no ha publicado una implementación de referencia totalmente documentada ni código de ejemplo para la aplicación de ExoPlayer que admite Widevine sin conexión hasta este momento, así que la información se limita al contenido de la guía y el blog de los desarrolladores. 

### <a name="working-with-older-android-devices"></a>Trabajo con dispositivos Android más antiguos

En algunos dispositivos Android antiguos, debe configurar los valores de las siguientes propiedades de **policy_overrides** (que se definen en la [plantilla de la licencia de Widevine](media-services-widevine-license-template-overview.md): **rental_duration_seconds**, **playback_duration_seconds** y **license_duration_seconds**. También puede configurarlos en cero, lo que implica una duración infinita/ilimitada.  

Los valores se deben configurar para evitar un error de desbordamiento de enteros. Para más información sobre el problema, consulte https://github.com/google/ExoPlayer/issues/3150 y https://github.com/google/ExoPlayer/issues/3112. <br/>Si no se establecen explícitamente, se asignarán valores muy elevados para **PlaybackDurationRemaining** y **LicenseDurationRemaining** (por ejemplo, 9223372036854775807, que es el valor positivo máximo para un entero de 64 bits). Como resultado, la licencia de Widevine aparece caducada y, por tanto, no se realizará el descifrado. 

Este problema no se produce en Android 5.0 Lollipop ni en versiones posteriores, ya que Android 5.0 es la primera versión de Android que se ha diseñado para admitir completamente ARMv8 ([Advanced RISC Machine](https://en.wikipedia.org/wiki/ARM_architecture)) y las plataformas de 64 bits, mientras que Android 4.4 KitKat se diseñó originalmente para admitir ARMv7 y plataformas de 32 bits como en otras versiones anteriores de Android.

## <a name="using-xamarin-to-build-an-android-playback-app"></a>Uso de Xamarin para compilar una aplicación de reproducción de Android

Puede encontrar enlaces de Xamarin para ExoPlayer con los siguientes vínculos:

- [Xamarin bindings library for the Google ExoPlayer library](https://github.com/martijn00/ExoPlayerXamarin) (Biblioteca de enlaces de Xamarin para la biblioteca de ExoPlayer de Google)
- [Xamarin bindings for ExoPlayer NuGet](https://www.nuget.org/packages/Xam.Plugins.Android.ExoPlayer/) (Enlaces de Xamarin para NuGet de ExoPlayer)

Vea también el siguiente subproceso: [Enlace Xamarin](https://github.com/martijn00/ExoPlayerXamarin/pull/57). 

## <a name="chrome-player-apps-for-android"></a>Aplicaciones de reproductor de Chrome para Android

A partir de la versión [62 de Chrome para Android ](https://developers.google.com/web/updates/2017/09/chrome-62-media-updates), se admite la licencia persistente en EME. Ahora también se admite [Widevine L1](https://developers.google.com/web/updates/2017/09/chrome-62-media-updates#widevine_l1) en Chrome para Android. Esto le permite crear aplicaciones de reproducción sin conexión en Chrome si los usuarios finales tienen esta versión (o una superior) de Chrome. 

Además, Google ha generado un ejemplo de Progressive Web App (PWA) y lo ha publicado con código abierto: 

- [Código fuente](https://github.com/GoogleChromeLabs/sample-media-pwa)
- [Versión de Google hospedada](https://biograf-155113.appspot.com/ttt/episode-2/) (solo funciona en la versión 62 y posteriores de Chrome en dispositivos Android)

Si actualiza el explorador Chrome móvil a la versión 62 (o posterior) en un teléfono Android y prueba la aplicación de ejemplo hospedada anterior, verá que funciona tanto el streaming en línea como el streaming sin conexión.

La aplicación de PWA de código abierto anterior se creó en Node.js. Si desea hospedar su propia versión en un servidor de Ubuntu, tenga en cuenta los siguientes problemas habituales que pueden impedir la reproducción:

1. Problema de CORS: el vídeo de ejemplo de la aplicación de ejemplo se hospeda en https://storage.googleapis.com/biograf-video-files/videos/. Google ha configurado CORS para todos sus ejemplos de prueba hospedados en el cubo de Google Cloud Storage. Se proporcionan con encabezados CORS, especificando explícitamente la entrada CORS: `https://biograf-155113.appspot.com` (el dominio en el que Google hospeda su ejemplo) impidiendo el acceso de cualquier otro sitio. Si lo intenta, verá el siguiente error HTTP: `Failed to load https://storage.googleapis.com/biograf-video-files/videos/poly-sizzle-2015/mp4/dash.mpd: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'https:\//13.85.80.81:8080' is therefore not allowed access. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.`
2. Problema de certificado: a partir de la versión 58 de Chrome, EME para Widevine requiere HTTPS. Por lo tanto, debe hospedar la aplicación de ejemplo a través de HTTPS con un certificado X509. Un certificado de prueba habitual no funciona debido a los siguientes requisitos: Es necesario obtener un certificado que cumpla con los siguientes requisitos mínimos:
    - Chrome y Firefox requieren que existan los parámetros de SAN (nombre alternativo del firmante) en el certificado
    - El certificado debe tener una entidad de certificación de confianza y un certificado de desarrollo autofirmado no sirve para ello
    - El certificado debe tener un CN que coincida con el nombre de DNS del servidor web o de la puerta de enlace

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="question"></a>Pregunta

¿Cómo se pueden entregar licencias persistentes (habilitadas para modo sin conexión) para algunos clientes/usuarios y licencias no persistentes (deshabilitadas para modo sin conexión) para otros? ¿Es necesario duplicar el contenido y usar la clave de contenido independiente?

### <a name="answer"></a>Respuesta
No es necesario duplicar el contenido. Basta con utilizar una sola copia del contenido y un solo valor de ContentKeyAuthorizationPolicy, pero dos valores de ContentKeyAuthorizationPolicyOption independientes:

1. IContentKeyAuthorizationPolicyOption 1: usa una licencia persistente, y ContentKeyAuthorizationPolicyRestriction 1, que contiene una notificación como license_type = "Persistent"
2. IContentKeyAuthorizationPolicyOption 2: usa una licencia no persistente, y ContentKeyAuthorizationPolicyRestriction 2, que contiene una notificación como license_type = "Nonpersistent"

De este modo, cuando llega una solicitud de licencia desde la aplicación cliente, no hay ninguna diferencia en ella. Sin embargo, en el caso de dispositivo/usuario final diferente, el STS debe tener la lógica de negocios para emitir diferentes tokens JWT con diferentes notificaciones (uno de los dos anteriores valores de license_type). El valor de las notificaciones en el token JWT se usará para decidir en función del servicio de licencias para emitir el tipo de licencia: persistente o no persistente.

Esto significa que el servicio de token de seguridad (STS) debe tener la información sobre la lógica de negocios y el cliente/dispositivo para agregar el correspondiente valor de notificación en un token.

### <a name="question"></a>Pregunta

En el caso de niveles de seguridad de Widevine, en el documento [Widevine DRM Architecture Overview](https://storage.googleapis.com/wvdocs/Widevine_DRM_Architecture_Overview.pdf) (Información general de la arquitectura de Widevine DRM) de Google, se definen tres niveles de seguridad diferentes. Sin embargo, en la [documentación de Azure Media Services sobre la plantillas de licencias de Widevine](./media-services-widevine-license-template-overview.md), se describen cinco niveles de seguridad diferentes. ¿Cuál es la relación o la asignación entre los dos conjuntos diferentes de niveles de seguridad?

### <a name="answer"></a>Respuesta

En el documento [Widevine DRM Architecture Overview](https://storage.googleapis.com/wvdocs/Widevine_DRM_Architecture_Overview.pdf) (Información general de la arquitectura de Widevine DRM) de Google se definen los tres siguientes niveles de seguridad:

1.  Nivel de seguridad 1: todo el procesamiento de contenido, la criptografía y el control se realizan en un entorno de ejecución de confianza (TEE). En algunos modelos de implementación, el procesamiento de seguridad puede realizarse en chips diferentes.
2.  Nivel de seguridad 2: realiza la criptografía (pero no el procesamiento de vídeo) dentro de la TEE: los búferes descifrados se devuelven al dominio de aplicación y se procesan a través de software o hardware de vídeo independiente. En el nivel 2, sin embargo, la información criptográfica se sigue procesando solo dentro de TEE.
3.  Nivel de seguridad 3: no tiene un TEE en el dispositivo. Es posible que tengan que adoptarse las medidas adecuadas para proteger la información criptográfica y el contenido descifrado en el sistema operativo del host. Una implementación de nivel 3 también puede incluir un motor de cifrado de hardware, pero esto solo mejora el rendimiento, no la seguridad.

Al mismo tiempo, en la [documentación de Azure Media Services sobre la plantilla de licencia de Widevine](./media-services-widevine-license-template-overview.md), la propiedad security_level de content_key_specs puede tener los siguientes cinco valores diferentes (requisitos de solidez de cliente para la reproducción):

1.  Se requiere criptografía white-box basada en software.
2.  Se requiere criptografía de software y un descodificador de ofuscación.
3.  Las operaciones de criptografía y material clave deben realizarse en un TEE con respaldo de hardware.
4.  La criptografía y la descodificación del contenido deben realizarse dentro de un TEE con respaldo de hardware.
5.  La criptografía, la descodificación y todo el tratamiento de los medios (comprimidos y descomprimidos) deben administrarse dentro de TEE con respaldo de hardware.

Los dos niveles de seguridad se definen mediante Google Widevine. La diferencia radica en su nivel de uso: nivel de API o de arquitectura. Los niveles de cinco seguridad se utilizan en la API de Widevine. El objeto content_key_specs, que contiene security_level, se deserializa y se pasa al servicio de entrega global de Widevine por parte del servicio de licencias de Widevine de Azure Media Services. En la tabla siguiente se muestra la asignación entre los dos conjuntos de niveles de seguridad.

| **Niveles de seguridad definidos en la arquitectura de Widevine** |**Niveles de seguridad utilizados en la API de Widevine**|
|---|---| 
| **Nivel de seguridad 1**: todo el procesamiento de contenido, la criptografía y el control se realizan en un entorno de ejecución de confianza (TEE). En algunos modelos de implementación, el procesamiento de seguridad puede realizarse en chips diferentes.|**security_level=5**: La criptografía, la descodificación y todo el tratamiento de los medios (comprimidos y descomprimidos) deben administrarse dentro de TEE con respaldo de hardware.<br/><br/>**security_level=4**: La criptografía y la descodificación del contenido deben realizarse dentro de un TEE con respaldo de hardware.|
**Nivel de seguridad 2**: realiza la criptografía (pero no el procesamiento de vídeo) dentro de la TEE: los búferes descifrados se devuelven al dominio de aplicación y se procesan a través de software o hardware de vídeo independiente. En el nivel 2, sin embargo, la información criptográfica se sigue procesando solo dentro de TEE.| **security_level=3**: Las operaciones de criptografía y material clave deben realizarse en un TEE con respaldo de hardware. |
| **Nivel de seguridad 3**: no tiene un TEE en el dispositivo. Es posible que tengan que adoptarse las medidas adecuadas para proteger la información criptográfica y el contenido descifrado en el sistema operativo del host. Una implementación de nivel 3 también puede incluir un motor de cifrado de hardware, pero esto solo mejora el rendimiento, no la seguridad. | **security_level=2**: Se requiere criptografía de software y un descodificador de ofuscación.<br/><br/>**security_level=1**: Se requiere criptografía white-box basada en software.|

### <a name="question"></a>Pregunta

¿Por qué la descarga de contenido tarda tanto tiempo?

### <a name="answer"></a>Respuesta

Hay dos maneras de mejorar la velocidad de descarga:

1.  Habilite la red CDN para que los usuarios tengan más probabilidades de utilizarla en lugar del punto de conexión de streaming/origen para la descarga de contenido. Si el usuario utiliza el punto de conexión de streaming, cada segmento HLS o fragmento DASH se empaqueta y cifra dinámicamente. Aunque esta latencia se encuentre en escala de milisegundos para cada segmento o fragmento, si tiene un vídeo de una hora de duración, la latencia acumulada puede ser grande (ocasionando una descarga más larga).
2.  Proporcione a los usuarios finales la opción de descargar de forma selectiva las capas de la calidad del vídeo y las pistas de audio en lugar de todo el contenido. En el caso del modo sin conexión, no hay ningún punto para descargar todas las capas de calidad. Hay dos formas de lograrlo:
    1.  Controlado por el cliente: la aplicación del reproductor selecciona automáticamente o el usuario selecciona la capa de calidad de vídeo y las pistas de audio para descargar.
    2.  Controlado por el servicio: uno puede utilizar la característica Manifiesto dinámico de Azure Media Services para crear un filtro (global), que limita la lista de reproducción HLS o el MPD de DASH a una sola capa de calidad del vídeo y las pistas de audio seleccionadas. Así pues, la URL de descarga que se presenta a los usuarios finales incluirá este filtro.

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="summary"></a>Resumen

En este artículo se describió cómo implementar la reproducción en modo sin conexión para contenido de DASH protegido en dispositivos Widevine o Android.  También se respondieron algunas preguntas frecuentes relacionadas con el streaming sin conexión del contenido protegido de Widevine.
