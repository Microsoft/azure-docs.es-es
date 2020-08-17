---
title: Escritura de complementos para Azure Media Player
description: Aprenda a escribir un complemento con Azure Media Player con JavaScript.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: how-to
ms.date: 04/20/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: da827b994685e95baf28e3ad34072642c24fa510
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422613"
---
# <a name="writing-plugins-for-azure-media-player"></a>Escritura de complementos para Azure Media Player #

Un complemento es un fragmento de JavaScript escrito para ampliar o mejorar el reproductor. Puede escribir complementos que cambien la apariencia de Azure Media Player, su funcionalidad, o incluso tener una interfaz con otros servicios. Puede hacerlo de dos sencillas maneras:

## <a name="step-1"></a>Paso 1 ##

Escriba el código JavaScript en una función como la siguiente:

```javascript

    (function () {
        amp.plugin('yourPluginName', function (options) {
        var myPlayer = this;
           myPlayer.addEventListener(amp.eventName.ready, function () {
        console.log("player is ready!");
            });
        });
    }).call(this);
```

Puede escribir el código directamente en la página HTML dentro de las etiquetas `<script>` o en un archivo externo de JavaScript. Si hace lo último, asegúrese de incluir el archivo JavaScript en el `<head>` de la página HTML *después* del script de Azure Media Player.

Ejemplo:

```javascript
    <!--*****START OF Azure Media Player Scripts*****-->
    <script src="//amp.azure.net/libs/amp/latest/azuremediaplayer.min.js"></script>
    <link href="//amp.azure.net/libs/amp/latest/skins/amp-default/azuremediaplayer.min.css" rel="stylesheet">
    <!--*****END OF Azure Media Player Scripts*****-->
    <!--Add Plugins-->
    <script src="yourPluginName.js"></script>
```

## <a name="step-2"></a>Paso 2 ##
Inicialice el complemento con JavaScript de una de estas dos maneras:

Método 1:

```javascript
    var myOptions = {
        autoplay: true,
        controls: true,
        width: "640",
        height: "400",
        poster: "",
        plugins: {
            yourPluginName: {
                [your plugin options]: [example options]
           }
        }
    };     
    var myPlayer = amp([videotag id], myOptions);
```

Método 2:

```javascript
    var video = amp([videotag id]);
    video.yourPluginName({[your plugins option]: [example option]});
```

No se necesitan opciones de complemento, solo se las incluye para permitir a los desarrolladores usar el complemento para configurar su comportamiento sin tener que cambiar el código fuente.

Para obtener inspiración y más ejemplos sobre cómo crear un complemento, eche un vistazo a nuestra [galería](azure-media-player-plugin-gallery.md).

>[!NOTE]
> El código del complemento cambia dinámicamente los elementos del DOM durante la duración de la experiencia del espectador con el reproductor, nunca realiza cambios permanentes en el código fuente del reproductor. Aquí es donde resulta útil comprender las herramientas para desarrolladores del explorador. Por ejemplo, si desea cambiar la apariencia de un elemento en el reproductor, puede encontrar su elemento HTML por su nombre de clase y, a continuación, agregar o cambiar los atributos desde allí. Este es un excelente recurso para [cambiar los atributos HTML](http://www.w3schools.com/js/js_htmldom_html.asp).

### <a name="integrated-plugins"></a>Complementos integrados ###

 Actualmente hay dos complementos preparados en Azure Media Player: la [sugerencia de tiempo](http://sr-test.azurewebsites.net/Tests/Plugin%20Gallery/plugins/timetip/example.html) y las [teclas de acceso rápido](http://sr-test.azurewebsites.net/Tests/Plugin%20Gallery/plugins/hotkeys/example.html). Estos complementos se desarrollaron originalmente para ser complementos modulares del reproductor, pero ahora se incluyen en el código fuente del mismo.

### <a name="plugin-gallery"></a>Galería de complementos ###

La [galería de complementos](https://aka.ms/ampplugins) tiene varios complementos con los que la comunidad ya ha contribuido para características como marcadores de escala de tiempo, zoom, análisis, etc. La página proporciona acceso a los complementos e instrucciones sobre cómo configurarlos, así como una demostración que muestra el complemento en acción. Si crea un complemento de acceso esporádico que cree que debe incluirse en nuestra galería, no dude en enviarlo para que podamos comprobarlo.

## <a name="next-steps"></a>Pasos siguientes ##

- [Inicio rápido de Azure Media Player](azure-media-player-quickstart.md)
