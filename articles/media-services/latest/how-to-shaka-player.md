---
title: Uso de Shaka Player con Azure Media Services
description: En este artículo se explica el uso de Shaka Player con Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/23/2020
ms.author: inhenkel
ms.custom: devx-track-javascript
ms.openlocfilehash: 5882b79232e858bbc8ad7e0da94ad4b04f5165ca
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422987"
---
# <a name="how-to-use-the-shaka-player-with-azure-media-services"></a>Uso de Shaka Player con Azure Media Services

## <a name="overview"></a>Información general

Shaka Player es una biblioteca JavaScript de código abierto para medios adaptables. Reproduce formatos de elementos multimedia adaptables (como DASH y HLS) en un explorador, sin usar complementos ni Flash. En su lugar, Shaka Player usa los estándares de web abierta MediaSource Extensions y Encrypted Media Extensions.

Se recomienda usar [Mux.js](https://github.com/videojs/mux.js/), ya que, sin él, Shaka Player admitiría el formato HLS CMAF, pero no HLS TS.

La información oficial se puede encontrar en la [documentación de Shaka Player](https://shaka-player-demo.appspot.com/docs/api/tutorial-welcome.html).

## <a name="sample-code"></a>Código de ejemplo
El código de ejemplo de este artículo está disponible en [Azure-Samples/media-services-3rdparty-player-samples](https://github.com/Azure-Samples/media-services-3rdparty-player-samples).

## <a name="implementing-the-player"></a>Implementación del reproductor

Siga estas instrucciones si necesita implementar su propia instancia del reproductor:

1. Cree un archivo `index.html` en el que hospedará el reproductor. Agregue las siguientes líneas de código (puede reemplazar las versiones por otras más recientes si es necesario):

    ```html
    <html>
      <head>
        <script src="//cdn.jsdelivr.net/npm/shaka-player@3.0.1/dist/shaka-player.compiled.js"></script>
        <script src="//cdn.jsdelivr.net/npm/mux.js@5.6.3/dist/mux.js"></script>
        <script type="module" src="index.js"></script>
      </head>
      <body>
        <video id="video" controls></video>
      </body>
    </html>
    ```

1. Agregue un archivo JavaScript con el código siguiente:

    ```javascript
    // myScript.js
    shaka.polyfill.installAll();

    var video = document.getElementById('video');
    var player = new shaka.Player(video);
    window.player = player;

    var manifestUrl = 'https://amsplayeraccount-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/sample-vod.ism/manifest(format=m3u8-aapl)';
    player.load(manifestUrl);
    ```

1. Reemplace `manifestUrl` por la dirección URL de HLS o DASH del localizador de streaming del recurso, que se encuentra en la página del localizador de streaming de Azure Portal.
    ![Direcciones URL del localizador de streaming](media/how-to-shaka-player/streaming-urls.png)

1. Ejecute un servidor (por ejemplo, con `npm http-server`) y el reproductor debería funcionar…

## <a name="set-up-captions"></a>Configuración de subtítulos

### <a name="set-up-vod-captions"></a>Configuración de subtítulos de VoD

Ejecute las siguientes líneas de código y reemplace `captionUrl` por el directorio .ctt (el archivo VTT debe estar en el mismo host para evitar errores de CORS), `lang` por el código de dos letras para el idioma, y `type` por `caption` o `subtitle`:

```javascript
player.configure('streaming.alwaysStreamText', true)
player.load(manifestUrl).then(function(){
        player.addTextTrack(captionUrl, lang, type, 'text/vtt');
        var tracks = player.getTextTracks();
        player.selectTextTrack(tracks[0]);
});
```

### <a name="set-up-live-stream-captions"></a>Configuración de subtítulos de streaming en vivo

La habilitación de los subtítulos en streaming en vivo se configura agregando la siguiente línea de código:

```javascript
player.setTextTrackVisibility(true)
```

## <a name="set-up-token-authentication"></a>Configuración de la autenticación de tokens

Ejecute las siguientes líneas de código y reemplace `token` por su cadena de token:

```javascript
player.getNetworkingEngine().registerRequestFilter(function (type, request) {
  if (type === shaka.net.NetworkingEngine.RequestType.LICENSE) {
    request.headers['Authorization'] = 'Bearer ' + token;
  }
});
```

## <a name="set-up-aes-128-encryption"></a>Configuración del cifrado AES-128

Actualmente, Shaka Player no admite el cifrado AES-128.

Un vínculo a un [problema](https://github.com/google/shaka-player/issues/850) en GitHub para seguir el estado de esta característica.

## <a name="set-up-drm-protection"></a>Configuración de la protección DRM

Shaka Player usa Encrypted Media Extensions (EME), que requiere que se use una dirección URL segura. Por lo tanto, para probar cualquier contenido protegido con DRM, es necesario usar HTTPS. Si el sitio usa HTTPS, el manifiesto y cada segmento tendrán que usar también HTTPS. Esto se debe a los requisitos de contenido mixto.

El orden de preferencia para la administración de Shaka de las direcciones URL de sus servidores de licencias:

1. La configuración de ClearKey, que se usa para la depuración, debe invalidar todo lo demás. (La aplicación todavía puede especificar un servidor de licencias de ClearKey).
2. Los servidores configurados por la aplicación, si los hay, deben invalidar todo elemento del manifiesto.
3. Los servidores de licencias proporcionados por el manifiesto solo se usan si no se especifica nada más.

Para especificar la dirección URL del servidor de licencias para Widevine o PlayReady, se puede usar el código siguiente:

```javascript
player.configure({
  drm: {
    servers: {
      "com.widevine.alpha": "YOUR WIDEVINE LICENSE URL",
      "com.microsoft.playready": "YOUR PLAYREADY LICENSE URL"
    }
  }
});

```

Todo el contenido de FairPlay requiere la configuración de un certificado de servidor. Se establece en la configuración del reproductor:

```javascript
const req = await fetch("YOUR FAIRPLAY CERTIFICATE URL");
const cert = await req.arrayBuffer();
player.configure('drm.advanced.com\\.apple\\.fps\\.1_0.serverCertificate', new Uint8Array(cert));
```

Para obtener más información, consulte la [documentación de protección DRM de Shaka Player](https://shaka-player-demo.appspot.com/docs/api/tutorial-drm-config.html).

## <a name="next-steps"></a>Pasos siguientes

* [Uso de Azure Media Player](../azure-media-player/azure-media-player-overview.md)
* [Inicio rápido: Cifrado del contenido](encrypt-content-quickstart.md)
