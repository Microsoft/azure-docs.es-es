---
title: Streaming Widevine para Android sin conexión con Azure Media Services v3
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
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: willzhan
ms.custom: devx-track-csharp
ms.openlocfilehash: 8cfc426844180cabc6f42330135d8fc1ff8ebabd
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89289399"
---
# <a name="offline-widevine-streaming-for-android-with-media-services-v3"></a>Streaming de Widevine sin conexión para Android con Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

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

> [!NOTE]
> DRM sin conexión solo se factura para hacer una única solicitud de una licencia al descargar el contenido. Los errores no se facturan.

## <a name="prerequisites"></a>Prerrequisitos 

Antes de implementar DRM para Widevine en dispositivos Android, debe hacer lo siguiente:

- Familiarizarse con los conceptos presentados para la protección de contenido en línea con Widevine DRM. Este tema se trata con detalle en los siguientes documentos y ejemplos:
    - [Diseño del sistema de protección de contenido con DRM múltiple con control de acceso](design-multi-drm-system-with-access-control.md)
    - [Uso del cifrado dinámico de DRM y el servicio de entrega de licencias](protect-with-drm.md)
- Clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git.

    Necesitará modificar el código en [Codificación con DRM mediante .NET](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/EncryptWithDRM) para agregar configuraciones de Widevine.  
- Familiarícese con el SDK de ExoPlayer de Google para Android, un SDK de reproductor de vídeo de código abierto que admite la reproducción de Widevine DRM sin conexión. 
    - [ExoPlayer SDK](https://github.com/google/ExoPlayer) (SDK de ExoPlayer)
    - [ExoPlayer Developer Guide](https://google.github.io/ExoPlayer/guide.html) (Guía para desarrolladores de ExoPlayer)
    - [EoPlayer Developer Blog](https://medium.com/google-exoplayer) (Blog para desarrolladores de ExoPlayer)

## <a name="configure-content-protection-in-azure-media-services"></a>Configuración de la protección de contenido en Azure Media Services

En el método [GetOrCreateContentKeyPolicyAsync](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L189) están presentes los pasos siguientes necesarios:

1. Especifique cómo se autoriza la entrega de claves en el servicio de entrega de licencias de contenido: 

    ```csharp
    ContentKeyPolicySymmetricTokenKey primaryKey = new ContentKeyPolicySymmetricTokenKey(tokenSigningKey);
    List<ContentKeyPolicyTokenClaim> requiredClaims = new List<ContentKeyPolicyTokenClaim>()
    {
        ContentKeyPolicyTokenClaim.ContentKeyIdentifierClaim
    };
    List<ContentKeyPolicyRestrictionTokenKey> alternateKeys = null;
    ContentKeyPolicyTokenRestriction restriction 
        = new ContentKeyPolicyTokenRestriction(Issuer, Audience, primaryKey, ContentKeyPolicyRestrictionTokenType.Jwt, alternateKeys, requiredClaims);
    ```
2. Configure la plantilla de licencia de Widevine:  

    ```csharp
    ContentKeyPolicyWidevineConfiguration widevineConfig = ConfigureWidevineLicenseTempate();
    ```

3. Cree ContentKeyPolicyOptions:

    ```csharp
    options.Add(
        new ContentKeyPolicyOption()
        {
            Configuration = widevineConfig,
            Restriction = restriction
        });
    ```

## <a name="enable-offline-mode"></a>Habilitación del modo sin conexión

Para habilitar el modo **sin conexión** para las licencias de Widevine, debe configurar la [plantilla de licencias de Widevine](widevine-license-template-overview.md). En el objeto **policy_overrides**, establezca la propiedad **can_persist** en **true** (el valor predeterminado es false), tal como se muestra en [ConfigureWidevineLicenseTempate](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L563). 

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#ConfigureWidevineLicenseTempate)]

## <a name="configuring-the-android-player-for-offline-playback"></a>Configuración del reproductor Android para la reproducción sin conexión

La manera más fácil de desarrollar una aplicación de reproductor nativa para dispositivos Android consiste en usar el [SDK de ExoPlayer de Google](https://github.com/google/ExoPlayer), un SDK de reproductor de vídeo de código abierto. ExoPlayer admite características que no son actualmente compatibles con la API de MediaPlayer nativa de Android, como los protocolos de entrega MPEG-DASH y Smooth Streaming de Microsoft.

La versión 2.6 de ExoPlayer (y posteriores) incluyen muchas clases que admiten la reproducción de Widevine DRM sin conexión. En concreto, la clase OfflineLicenseHelper proporciona funciones de utilidad para facilitar el uso de DefaultDrmSessionManager para descargar, renovar y liberar las licencias sin conexión. Las clases proporcionadas en la carpeta del SDK "library/core/src/main/java/com/google/android/exoplayer2/offline/" admiten la descarga de contenido de vídeo sin conexión.

La siguiente lista de clases facilita el modo sin conexión en el SDK de ExoPlayer para Android:

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

En algunos dispositivos Android antiguos, debe configurar los valores de las siguientes propiedades de **policy_overrides** (que se definen en la [plantilla de la licencia de Widevine](widevine-license-template-overview.md): **rental_duration_seconds**, **playback_duration_seconds** y **license_duration_seconds**. También puede configurarlos en cero, lo que implica una duración infinita/ilimitada.  

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

## <a name="faqs"></a>Preguntas más frecuentes

Para obtener más información, vea [P+F de Widevine](frequently-asked-questions.md#widevine-streaming-for-android).

## <a name="additional-notes"></a>Notas adicionales

Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="summary"></a>Resumen

En este artículo se describió cómo implementar la reproducción en modo sin conexión para contenido de DASH protegido en dispositivos Widevine o Android.  También se respondieron algunas preguntas frecuentes relacionadas con el streaming sin conexión del contenido protegido de Widevine.
