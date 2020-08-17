---
title: Configuración completa de Azure Media Player
description: Aprenda a configurar Azure Media Player.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: how-to
ms.date: 04/20/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: d7e6d3c1554f70ea14e097ff2fe6df47987b5927
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423055"
---
# <a name="azure-media-player-full-setup"></a>Configuración completa de Azure Media Player #

Azure Media Player es fácil de configurar. Solo se tarda unos minutos en obtener la reproducción básica de contenido multimedia directamente desde la cuenta de Azure Media Services. También se proporcionan [ejemplos](https://github.com/Azure-Samples/azure-media-player-samples) en el directorio de ejemplos de la versión.


## <a name="step-1-include-the-javascript-and-css-files-in-the-head-of-your-page"></a>Paso 1: Incluir los archivos JavaScript y CSS en el encabezado de la página ##

Con Azure Media Player, puede acceder a los scripts desde la versión hospedada de la red CDN. A menudo se recomienda actualmente poner JavaScript antes de la etiqueta BODY final `<body>` en lugar de antes de `<head>`, pero Azure Media Player incluye una modificación de HTML5, que debe estar en el encabezado de las versiones más antiguas de IE para que respete la etiqueta de vídeo como un elemento válido.

> [!NOTE]
> Si ya usa una modificación de HTML5 como [Modernizr](https://modernizr.com/) puede incluir el JavaScript de Azure Media Player en cualquier lugar. Sin embargo, asegúrese de que la versión de Modernizr incluye shiv para vídeo.

### <a name="cdn-version"></a>Versión para CDN ###

```html
    <link href="//amp.azure.net/libs/amp/latest/skins/amp-default/azuremediaplayer.min.css" rel="stylesheet">
    <script src= "//amp.azure.net/libs/amp/latest/azuremediaplayer.min.js"></script>
```

> [!IMPORTANT]
> **NO** use la versión `latest` en producción, porque está sujeta a cambios a petición. Reemplace `latest` por una versión de Azure Media Player. Por ejemplo, reemplace `latest` por `2.1.1`. Las versiones de Azure Media Player pueden consultarse desde [aquí](azure-media-player-changelog.md).

> [!NOTE]
> Desde la versión `1.2.0`, ya no es necesario incluir la ubicación en las tecnologías de reserva (seleccionará automáticamente la ubicación de la ruta de acceso relativa del archivo azuremediaplayer.min.js). Para modificar la ubicación de las tecnologías de reserva, agregue el siguiente script en el `<head>` después de los scripts anteriores.

> [!NOTE]
> Debido a la naturaleza de los complementos de Flash y Silverlight, los archivos swf y xap deben estar hospedados en un dominio sin información confidencial. En la versión hospedada en Azure CDN, esto se hace automáticamente.

```javascript
    <script>
      amp.options.flashSS.swf = "//amp.azure.net/libs/amp/latest/techs/StrobeMediaPlayback.2.0.swf"
      amp.options.silverlightSS.xap = "//amp.azure.net/libs/amp/latest/techs/SmoothStreamingPlayer.xap"
    </script>
```

## <a name="step-2-add-an-html5-video-tag-to-your-page"></a>Paso 2: Agregar una etiqueta de vídeo HTML5 a la página ##

Con Azure Media Player, puede usar una etiqueta de vídeo HTML5 para insertar un vídeo. A continuación, Azure Media Player leerá la etiqueta y la hará funcionar en todos los exploradores, no solo en los que admiten vídeo HTML5. Además del lenguaje de marcado básico, Azure Media Player necesita algunos fragmentos adicionales.

1. El atributo `<data-setup>` en `<video>` indica a Azure Media Player que configure automáticamente el vídeo cuando la página esté lista y lea todo lo que haya (en formato JSON) en el atributo.
1. El atributo `id`: Se debe usar de forma exclusiva con cada vídeo de la misma página.
1. El atributo `class` contiene dos clases:
    - `azuremediaplayer` aplica los estilos necesarios para la funcionalidad de la interfaz de usuario de Azure Media Player.
    - `amp-default-skin` aplica la máscara predeterminada a los controles HTML5.
1. `<source>` incluye dos atributos necesarios.
    - El atributo `src` puede incluir un archivo * *.ism/manifest* de Azure Media Services, y Azure Media Player agrega automáticamente las direcciones URL para DASH, SMOOTH y HLS al reproductor.
    - El atributo `type` es el tipo MIME que requiere la secuencia. El tipo MIME asociado a *".ism/manifest"* es *"application/vnd.ms-sstr+xml"*
1. El atributo `<data-setup>` *opcional* en `<source>` indica a Azure Media Player si hay directivas de entrega únicas para la secuencia de Azure Media Services, incluyendo, entre otros, el tipo de cifrado (AES, PlayReady, Widevine o FairPlay) y el token.

Incluya o excluya los atributos, configuración, orígenes y pistas exactamente como lo haría para el vídeo HTML5.

```html
    <video id="vid1" class="azuremediaplayer amp-default-skin" autoplay controls width="640" height="400" poster="poster.jpg" data-setup='{"techOrder": ["azureHtml5JS", "flashSS", "html5FairPlayHLS","silverlightSS", "html5"], "nativeControlsForTouch": false}'>
        <source src="http://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest" type="application/vnd.ms-sstr+xml" />
        <p class="amp-no-js">
            To view this video please enable JavaScript, and consider upgrading to a web browser that supports HTML5 video
        </p>
    </video>
```

De forma predeterminada, el botón grande "Reproducir" se encuentra en la esquina superior izquierda, por lo que no tapa las partes interesantes del póster. Si prefiere centrar este botón, puede agregar una `class` `amp-big-play-centered` adicional al elemento `<video>`.

### <a name="alternative-setup-for-dynamically-loaded-html"></a>Configuración alternativa para HTML cargado dinámicamente ###

Si la página web o aplicación carga la etiqueta de vídeo dinámicamente (ajax, appendChild, etc.), de forma que puede que no exista cuando se cargue la página, puede que desee configurar manualmente el reproductor en lugar de confiar en el atributo de configuración de datos. Para ello, elimine primero el atributo de configuración de datos de la etiqueta para que no haya confusión sobre cuándo se inicializa el reproductor. A continuación, ejecute el siguiente código de JavaScript alguna vez después de que se haya cargado el JavaScript de Azure Media Player y después de que se haya cargado la etiqueta de vídeo en el DOM.

```javascript
    var myPlayer = amp('vid1', { /* Options */
            techOrder: ["azureHtml5JS", "flashSS", "html5FairPlayHLS","silverlightSS", "html5"],
            "nativeControlsForTouch": false,
            autoplay: false,
            controls: true,
            width: "640",
            height: "400",
            poster: ""
        }, function() {
              console.log('Good to go!');
               // add an event listener
              this.addEventListener('ended', function() {
                console.log('Finished!');
            }
          }
    );
    myPlayer.src([{
        src: "http://samplescdn.origin.mediaservices.windows.net/e0e820ec-f6a2-4ea2-afe3-1eed4e06ab2c/AzureMediaServices_Overview.ism/manifest",
        type: "application/vnd.ms-sstr+xml"
    }]);
```

El primer argumento de la función `amp` es el identificador de la etiqueta de vídeo. Reemplácelo por el suyo propio.

El segundo argumento es un objeto de opciones. Permite establecer opciones adicionales igual que con el atributo de configuración de datos.

El tercer argumento es una devolución de llamada con estado "ready". Una vez que se ha inicializado Azure Media Player, se llamará a esta función. En la devolución de llamada con estado "ready", "este" objeto hace referencia a la instancia del reproductor.

En lugar de usar un identificador de elemento, también puede pasar una referencia al elemento en sí mismo.

```javascript

    amp(document.getElementById('example_video_1'), {/*Options*/}, function() {
        //This is functionally the same as the previous example.
    });
    myPlayer.src([{ src: "//example/path/to/myVideo.ism/manifest", type: "application/vnd.ms-sstr+xml"]);
```

## <a name="next-steps"></a>Pasos siguientes ##

- [Inicio rápido de Azure Media Player](azure-media-player-quickstart.md)