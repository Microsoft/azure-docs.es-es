---
title: Uso del reproductor Video.js con Azure Media Services
description: En este artículo se explica cómo usar el objeto de vídeo HTML y JavaScript con Azure Media Services
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-javascript
ms.openlocfilehash: 2730c6ce523e618110cd29b13ba2f37115e2cbd0
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267690"
---
# <a name="how-to-use-the-videojs-player-with-azure-media-services"></a>Uso del reproductor Video.js con Azure Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

## <a name="overview"></a>Información general

Video.js es un reproductor de vídeo web creado para un entorno de HTML5. Reproduce formatos de elementos multimedia adaptables (como DASH y HLS) en un explorador, sin usar complementos ni Flash. En su lugar, Video.js usa los estándares de web abierta MediaSource Extensions y Encrypted Media Extensions. Además, admite la reproducción de vídeo en equipos de escritorio y dispositivos móviles.

La documentación oficial se puede encontrar en [https://docs.videojs.com/](https://docs.videojs.com/).

## <a name="sample-code"></a>Código de ejemplo
El código de ejemplo de este artículo está disponible en [Azure-Samples/media-services-3rdparty-player-samples](https://github.com/Azure-Samples/media-services-3rdparty-player-samples).

## <a name="implement-the-player"></a>Implementación del reproductor

1. Cree un archivo `index.html` en el que hospedará el reproductor. Agregue las siguientes líneas de código (puede reemplazar las versiones por otras más recientes si es necesario):

    ```html
    <html>
      <head>
        <link href="https://vjs.zencdn.net/7.8.2/video-js.css" rel="stylesheet" />
      </head>
      <body>
        <video id="video" class="video-js vjs-default-skin vjs-16-9" controls data-setup="{}">
        </video>

        <script src="https://vjs.zencdn.net/7.8.2/video.js"></script>
        <script src="https://cdn.jsdelivr.net/npm/videojs-contrib-eme@3.7.0/dist/videojs-contrib-eme.min.js"></script>
        <script type="module" src="index.js"></script>
      </body>
    <html>
    ```

2. Agregue un archivo `index.js` con el código siguiente:

    ```javascript
    var videoJS = videojs("video");
    videoJS.src({
      src: "manifestUrl",
      type: "protocolType",
    });
    ```

3. Reemplace `manifestUrl` por la dirección URL de HLS o DASH del localizador de streaming del recurso que se encuentra en la página del localizador de streaming de Azure Portal.
    ![Direcciones URL del localizador de streaming](media/how-to-shaka-player/streaming-urls.png)

4. Reemplace `protocolType` por las siguientes opciones:

    - "application/x-mpegURL" para protocolos HLS
    - "application/dash+xml" para protocolos DASH

### <a name="set-up-captions"></a>Configuración de subtítulos

Ejecute el método `addRemoteTextTrack` y reemplace:

- `subtitleKind` por `"captions"`, `"subtitles"`,`"descriptions"` o `"metadata"`  
- `caption` por la ruta de acceso del archivo .vtt (este archivo debe estar en el mismo host para evitar el error de CORS)
- `subtitleLang` por el código BCP 47 para el idioma, por ejemplo, `"eng"` para inglés o `"es"` para español
- `subtitleLabel` por el nombre para mostrar que desee para el subtítulo

```javascript
videojs.players.video.addRemoteTextTrack({
  kind: subtitleKind,
  src: caption,
  srclang: subtitleLang,
  label: subtitleLabel
});
```

### <a name="set-up-token-authentication"></a>Configuración de la autenticación de tokens

El token se debe establecer en el campo de autorización del encabezado de solicitud. Para evitar problemas con CORS, este token solo debe establecerse en las solicitudes con `'keydeliver'` en su dirección URL. Las siguientes líneas de código deberían funcionar:

```javascript
setupTokenForDecrypt (options) {
  if (options.uri.includes('keydeliver')) {
    options.headers = options.headers || {}
    options.headers.Authorization = 'Bearer=' + this.getInputToken()
  }

  return options
}
```

A continuación, la función anterior se debe asociar al evento `videojs.Hls.xhr.beforeRequest`.

```javascript
videojs.Hls.xhr.beforeRequest = setupTokenForDecrypt;
```

### <a name="set-up-aes-128-encryption"></a>Configuración del cifrado AES-128

Video.js admite el cifrado AES-128 sin ninguna configuración adicional. 

> [!NOTE] 
> Actualmente hay un [problema](https://github.com/videojs/video.js/issues/6717) con el cifrado y el contenido de CMAF de HLS o de DASH, contenidos que no se pueden reproducir.

### <a name="set-up-drm-protection"></a>Configuración de la protección DRM

Para admitir la protección DRM, debe agregar la extensión oficial [videojs-contrib-eme](https://github.com/videojs/videojs-contrib-eme). También funciona una versión de CDN.

1. En el archivo `index.js` descrito anteriormente, debe inicializar la extensión EME agregando `videoJS.eme();` *antes* de agregar el origen del vídeo:

   ```javascript
    videoJS.eme();
   ```

2. Ahora puede definir las direcciones URL de los servicios DRM y las direcciones URL de las licencias correspondientes como se indica a continuación:

   ```javascript
   videoJS.src({
       keySystems: {
           "com.microsoft.playready": "YOUR PLAYREADY LICENSE URL",
           "com.widevine.alpha": "YOUR WIDEVINE LICENSE URL",
           "com.apple.fps.1_0": {
            certificateUri: "YOUR FAIRPLAY CERTIFICATE URL",
            licenseUri: "YOUR FAIRPLAY LICENSE URL"
           }
         }
       })

   ```

#### <a name="acquiring-the-license-url"></a>Adquisición de la dirección URL de la licencia

Para adquirir la dirección URL de la licencia, puede:

- Consultar la configuración del proveedor de DRM
- o bien, si está usando el ejemplo, consultar el documento `output.json` que se genera al ejecutar el script *setup-vod.ps1* de PowerShell para vídeos bajo demanda o el script *start-live.ps1* para streaming en vivo. También encontrará los KID en este archivo.

#### <a name="using-tokenized-drm"></a>Uso de DRM tokenizada

Con el fin de admitir la protección DRM tokenizada, tiene que agregar la siguiente línea a la propiedad `src` del reproductor:

```javascript
videoJS.src({
src: ...,
emeHeaders: {'Authorization': "Bearer=" + "YOUR TOKEN"},
keySystems: {...
```

## <a name="next-steps"></a>Pasos siguientes

- [Uso de Azure Media Player](../azure-media-player/azure-media-player-overview.md)  
- [Inicio rápido: Cifrado del contenido](encrypt-content-quickstart.md)
