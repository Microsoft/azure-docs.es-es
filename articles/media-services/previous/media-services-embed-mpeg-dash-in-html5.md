---
title: Inserción de un vídeo de streaming adaptable MPEG-DASH en una aplicación HTML5 con DASH.js | Microsoft Docs
description: En este vídeo se muestra cómo incluir un vídeo de streaming adaptable MPEG-DASH en una aplicación HTML5 con DASH.js.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 5aa0e7b6-f5c3-4cc1-aa33-ed16ea4780c2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 55c22a58ea76c268c40894c0ea64d43312b1ba27
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269104"
---
# <a name="embedding-an-mpeg-dash-adaptive-streaming-video-in-an-html5-application-with-dashjs"></a>Incrustación de un vídeo de streaming adaptativo MPEG-DASH en una aplicación HTML5 con DASH.js

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

## <a name="overview"></a>Información general
MPEG-DASH es una norma ISO para el streaming adaptable de contenido de vídeo, que ofrece ventajas significativas para los desarrolladores que desean ofrecer salida de streaming de vídeo adaptable de alta calidad. Con MPEG-DASH, la secuencia de vídeo se ajusta automáticamente a una definición inferior cuando la red está saturada. Esto reduce la probabilidad de que el usuario vea un vídeo "pausado" mientras el reproductor descarga los siguientes segundos para reproducirlos (es decir, se reduce la probabilidad de almacenamiento en búfer). A medida que la congestión de la red se reduce, el reproductor de vídeo volverá a su vez a una secuencia de mayor calidad. Esta capacidad para adaptar el ancho de banda necesario también produce una hora de inicio más rápida para el vídeo. Esto significa que los primeros segundos se pueden reproducir en un segmento de calidad inferior rápido para descargar y luego pasan a una calidad superior una vez se ha almacenado en búfer el contenido suficiente.

Dash.js es un reproductor de vídeo MPEG-DASH de código abierto escrito en JavaScript. Su objetivo es proporcionar un reproductor sólido entre plataformas que se pueda reutilizar libremente en aplicaciones que requieren reproducción de vídeo. Ofrece reproducción MPEG-DASH en cualquier explorador que admita las extensiones de origen multimedia (MSE) W3C; en la actualidad es Chrome, Microsoft Edge e IE11 (otros exploradores han indicado su intención de ser compatibles con MSE). Para obtener más información sobre DASH.js, vea el repositorio de GitHub dash.js.

## <a name="creating-a-browser-based-streaming-video-player"></a>Creación de un reproductor de vídeo de streaming basado en explorador
Para crear una página web sencilla que muestre un reproductor de vídeo con los controles esperados como reproducir, pausa, rebobinar, etc., necesita:

1. Crear una página HTML
2. Agregar la etiqueta de vídeo
3. Agregar el reproductor dash.js
4. Inicializar el reproductor
5. Agregar algún estilo CSS
6. Ver los resultados en un explorador que implemente MSE

La inicialización del reproductor se puede completar con tan solo unas líneas de código JavaScript. Con dash.js, es realmente así de sencillo incrustar vídeo MPEG-DASH en sus aplicaciones basadas en explorador.

## <a name="creating-the-html-page"></a>Creación de la página HTML
El primer paso es crear una página HTML estándar con el elemento de **vídeo**, guardar este archivo como basicPlayer.html, como se muestra en el ejemplo siguiente:

```html
    <!DOCTYPE html>
    <html>
      <head><title>Adaptive Streaming in HTML5</title></head>
      <body>
        <h1>Adaptive Streaming with HTML5</h1>
        <video id="videoplayer" controls></video>
      </body>
    </html>
```

## <a name="adding-the-dashjs-player"></a>Adición del reproductor dash.js
Para agregar la implementación de referencia de dash.js a la aplicación, deberá obtener el archivo dash.all.js de la versión más reciente del proyecto dash.js. Este se debe guardar en la carpeta de JavaScript de la aplicación. Este archivo es un archivo de conveniencia que reúne todo el código de dash.js necesario en un solo archivo. Si echa un vistazo al repositorio dash.js, observará los archivos individuales, probará código y mucho más, pero si todo lo que quiere hacer es usar dash.js, el archivo dash.all.js es lo que necesita.

Para agregar el reproductor de dash.js a sus aplicaciones, agregue una etiqueta de script a la sección de encabezado del archivo basicPlayer.html:

```html
    <!-- DASH-AVC/265 reference implementation -->
    < script src="js/dash.all.js"></script>
```

A continuación, cree una función para inicializar el reproductor cuando se cargue la página. Agregue el siguiente script después de la línea en la que carga dash.all.js:

```html
    <script>
    // setup the video element and attach it to the Dash player
    function setupVideo() {
      var url = "http://wams.edgesuite.net/media/MPTExpressionData02/BigBuckBunny_1080p24_IYUV_2ch.ism/manifest(format=mpd-time-csf)";
      var context = new Dash.di.DashContext();
      var player = new MediaPlayer(context);
                      player.startup();
                      player.attachView(document.querySelector("#videoplayer"));
                      player.attachSource(url);
    }
    </script>
```

Esta función crea primero un elemento DashContext. Se usa para configurar la aplicación para un entorno de tiempo de ejecución específico. Desde un punto de vista técnico, define las clases que debería usar el marco de inserción de dependencias al construir la aplicación. En la mayoría de los casos, usará Dash.di.DashContext.

A continuación, cree una instancia de la clase principal del marco de dash.js, MediaPlayer. Esta clase contiene los métodos principales necesarios, como reproducción y pausa, administra la relación con el elemento de vídeo y también la interpretación del archivo de descripción de presentación multimedia (MPD) que describe el vídeo que se va a reproducir.

Se llama a la función startup() de la clase MediaPlayer para asegurarse de que el reproductor está listo para reproducir vídeo. Entre otras cosas, esta función garantiza que se han cargado todas las clases necesarias (según se definen por el contexto). Cuando el reproductor esté listo, puede asociar el elemento de vídeo mediante la función attachView(). La función de inicio permite que MediaPlayer inserte la secuencia de vídeo en el elemento y también controlar la reproducción según sea necesario.

Pase la dirección URL del archivo MPD a MediaPlayer para que conozca el vídeo que espera que reproducir. La función setupVideo() que acaba de crear se debe ejecutar una vez que la página se haya cargado completamente. Haga esto mediante el evento onload del elemento body. Cambie el elemento `<body>` a:

```html
    <body onload="setupVideo()">
```

Por último, establezca el tamaño del elemento de vídeo mediante CSS. En un entorno de streaming adaptable, esto es especialmente importante porque el tamaño del vídeo que se reproduce puede cambiar a medida que la reproducción se adapta a las condiciones de red cambiantes. En esta sencilla demo simplemente debe forzar el elemento de vídeo para que sea el 80 % de la ventana del explorador disponible agregando el CSS siguiente a la sección inicial de la página:

```html
    <style>
    video {
      width: 80%;
      height: 80%;
    }
    </style>
```

## <a name="playing-a-video"></a>Reproducción de un vídeo
Para reproducir un vídeo, dirija el explorador al archivo basicPlayback.html y haga clic en Reproducir en el reproductor de vídeo que se muestra.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Consulte también

[Repositorio dash.js de GitHub](https://github.com/Dash-Industry-Forum/dash.js) 

