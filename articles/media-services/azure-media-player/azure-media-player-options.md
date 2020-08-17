---
title: Opciones de Azure Media Player
description: El código insertado de Azure Media Player es simplemente una etiqueta de vídeo HTML5 por lo que, para muchas de las opciones, puede utilizar los atributos de etiqueta estándar para establecer estas.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: reference
ms.date: 04/20/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 2b7acb1ad6fbe0beb4d79ee2a833561f0829664c
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423038"
---
# <a name="options"></a>Opciones #

## <a name="setting-options"></a>Configuración de opciones ##

El código insertado de Azure Media Player es simplemente una etiqueta de vídeo HTML5 por lo que, para muchas de las opciones, puede utilizar los atributos de etiqueta estándar para establecer estas.

`<video controls autoplay ...>`

Alternativamente, también puede usar el atributo de configuración de datos para proporcionar opciones en formato [JSON](http://json.org/example.html). Esta también sería la manera de configurar aquellas opciones que no son estándar en la etiqueta de vídeo.

`<video data-setup='{ "controls": true, "autoplay": false }'...>`

Por último, si no va a utilizar el atributo de configuración de datos para desencadenar la configuración del reproductor, puede pasar un objeto con las opciones del reproductor como el segundo argumento de la función de configuración de JavaScript.

`amp("vid1", { "controls": true, "autoplay": false });`

> [!NOTE]
> Las opciones del constructor solo se establecen en la primera inicialización antes de establecer el origen.  Si desea modificar las opciones en el mismo elemento inicializado de Azure Media Player, debe actualizarlas antes de cambiar el origen. Puede actualizar las opciones en JavaScript mediante `myPlayer.options({/*updated options*/});`. Tenga en cuenta que solo las opciones modificadas se verán afectadas, todas las demás opciones establecidas anteriormente seguirán igual.

## <a name="individual-options"></a>Opciones individuales ##

> [!NOTE]
>Los atributos de la etiqueta de vídeo solo pueden ser true o false (booleanos). Solo tiene que incluir el atributo (sin el signo igual) para activarlo, o excluirlo para desactivarlo. Por ejemplo, para activar los controles: INCORRECTO: `<video controls="true" ...>` CORRECTO: `<video controls ...>` El problema con el que se encuentran los usuarios es intentar establecer estos valores en false usando "false" como valor (por ejemplo, controls="false") lo que realmente hace justo lo contrario ya que esto establece el valor en true porque se está incluyendo el atributo.

### <a name="controls"></a>controls ###

La opción de controles establece si el reproductor tiene controles o no con los que el usuario pueda interactuar. Sin controles, la única manera de iniciar la reproducción del vídeo es con el atributo autoplay (reproducción automática) o mediante la API.

`<video controls ...>` o `{ "controls": true }`

### <a name="autoplay"></a>autoplay ###

Si autoplay es true, el vídeo empezará la reproducción tan pronto como se cargue la página (sin ninguna interacción por parte del usuario).

> [!NOTE]
> Esta opción no la admiten dispositivos móviles como Windows Phone, Apple iOS y Android. En los dispositivos móviles la funcionalidad de reproducción automática está bloqueada para impedir un uso excesivo de los planes de datos mensuales del consumidor (a menudo, caros). En estos casos, se necesita un toque o un clic por parte del usuario para iniciar el vídeo.

`<video autoplay ...>`o `{ "autoplay": true }`

### <a name="poster"></a>poster ###
El atributo poster permite establecer la imagen que aparece antes de que el vídeo empiece a reproducirse. A menudo, se trata de un fotograma del vídeo o una pantalla de título personalizada. En cuanto el usuario hace clic en reproducir, la imagen desaparece.

`<video poster="myPoster.jpg" ...>` o `{ "poster": "myPoster.jpg" }`

### <a name="width"></a>width ###

El atributo width establece el ancho de pantalla del vídeo.

`<video width="640" ...>` o `{ "width": 640 }`

### <a name="height"></a>height ###

El atributo height establece el alto de pantalla del vídeo.

`<video height="480" ...>` o `{ "height": 480 }`

### <a name="plugins"></a>plugins ###

El JSON plugins determina qué complementos se cargarán con esa instancia de Azure Media Player y le permitirá configurar las opciones que ese complemento pueda tener.

   `<video... data-setup='{plugins: { "contentTitle": {"name": "Azure Medi Services Overview"}}}'...>`

Para más información sobre el desarrollo y el uso de complementos, consulte información sobre la [escritura de complementos](azure-media-player-writing-plugins.md).

### <a name="other-options"></a>Otras opciones ###

Se pueden establecer otras opciones en la etiqueta `<video>` mediante el parámetro `data-setup` que utiliza un JSON.
`<video ... data-setup='{"nativeControlsForTouch": false}'>`

#### <a name="nativecontrolsfortouch"></a>nativeControlsForTouch ####

Este se establece explícitamente en false. Esto permitirá que la máscara de Azure Media Player se reproduzca igual en todas las plataformas.  Es más, contrariamente a lo que puede parecer por el nombre, el toque estará habilitado.

### <a name="fluid"></a>fluid ###

Si establece esta opción en true, el elemento de vídeo ocupará el ancho de pantalla completo del contenedor principal y la altura se adaptará a un vídeo con relación de aspecto 16:9 estándar.

`<video ... data-setup='{"fluid": true}'>`

La opción `fluid` reemplaza a los valores explícitos `width` y `height`. Esta opción solo está disponible en la versión `2.0.0` y posteriores de Azure Media Player.

### <a name="playbackspeed"></a>playbackSpeed ###

La opción `playbackSpeed` permite establecer el valor de velocidad de la reproducción que está disponible para el usuario. `playbackSpeed` toma un objeto. Para habilitar el control de velocidad de reproducción en la barra de control, la propiedad `enabled` del objeto se debe establecer en true. Un ejemplo de cómo habilitar la velocidad de reproducción en el marcado:

`<video ... data-setup='{"playbackSpeed": {"enabled": true}}'>`


Otras opciones del valor `playbackSpeed` se proporcionan mediante el objeto [PlaybackSpeedOptions](/javascript/api/azuremediaplayer/amp.player.playbackspeedoptions).

Ejemplo de cómo configurar las opciones de velocidad de reproducción en JavaScript:

```javascript
    var myPlayer = amp('vid1', {
        fluid: true,
        playbackSpeed: {
            enabled: true,
            initialSpeed: 1.0,
            speedLevels: [
                { name: "x4.0", value: 4.0 },
                { name: "x3.0", value: 3.0 },
                { name: "x2.0", value: 2.0 },
                { name: "x1.75", value: 1.75 },
                { name: "x1.5", value: 1.5 },
                { name: "x1.25", value: 1.25 },
                { name: "normal", value: 1.0 },
                { name: "x0.75", value: 0.75 },
                { name: "x0.5", value: 0.5 },
            ]
        }
    });
```

Esta opción solo está disponible en la versión 2.0.0 y posteriores de Azure Media Player.

### <a name="staledatatimelimitinsec"></a>staleDataTimeLimitInSec ###

La opción `staleDataTimeLimitInSec` es una optimización que le permite configurar cuántos segundos de datos antiguos le gustaría conservar en los búferes de mediaSource. Esta opción está deshabilitada de manera predeterminada.

### <a name="cea708captionssettings"></a>cea708CaptionsSettings ###

El establecimiento de este valor en true le permite mostrar subtitulado CEA en directo en el streaming en vivo y los archivos en directo. No se necesita el atributo de etiqueta y, si no se incluye, el reproductor utilizará una etiqueta predeterminada.

```javascript
     cea708CaptionsSettings: {
                enabled: true,
                srclang: 'en',
                label: 'Live CC'
            }
```

Esta opción solo está disponible en la versión 2.1.1 y posteriores de Azure Media Player.

## <a name="next-steps"></a>Pasos siguientes ##

- [Inicio rápido de Azure Media Player](azure-media-player-quickstart.md)
