---
title: Inserción de widgets de Video Indexer en las aplicaciones
titlesuffix: Azure Media Services
description: Obtenga más información acerca de cómo insertar widgets de Video Indexer en su aplicación.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: juliako
ms.openlocfilehash: 5acd9b68368f56000a0a32d1ade310cf30143950
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799380"
---
# <a name="embed-video-indexer-widgets-into-your-applications"></a>Inserción de widgets de Video Indexer en las aplicaciones

En este vídeo se muestra cómo insertar widgets de Video Indexer en sus aplicaciones. Video Indexer admite la inserción de dos tipos de widgets en la aplicación: **Cognitive Insights** y **Player**. 

A partir de la versión 2, la dirección URL base del widget incluye la región de la cuenta. Por ejemplo, se genera una cuenta en la región Oeste de EE. UU.: `https://wus2.videoindexer.ai/embed/insights/...`.

## <a name="widget-types"></a>Tipos de widget

### <a name="cognitive-insights-widget"></a>Widget Cognitive Insights

El widget **Cognitive Insights** incluye toda la información detallada visual que se extrajo del proceso de indexación de vídeo. El widget de información detallada admite los siguientes parámetros de URL opcionales:

|NOMBRE|Definición|DESCRIPCIÓN|
|---|---|---|
|widgets|Cadenas separadas por coma|Le permite controlar la información detallada que desea representar. <br/>Ejemplo: `https://www.videoindexer.ai/embed/insights/<accountId>/<videoId>/?widgets=people,search` representará solo la información detallada de la interfaz de usuario de personas y marcas<br/>Opciones disponibles: personas, palabras clave, anotaciones, marcas, opiniones, transcripción y búsqueda.<br/>No se admite a través de la dirección URL en la versión=2<br/><br/>**Nota:** No se admite el parámetro de dirección URL de widgets en la versión 2. |

### <a name="player-widget"></a>Widget Player

El widget **Player** le permite transmitir el vídeo con velocidad de bits adaptable. El widget Player admite los siguientes parámetros de URL opcionales:

|NOMBRE|Definición|DESCRIPCIÓN|
|---|---|---|
|t|Segundos desde el inicio|Hace que el reproductor comience a reproducir desde un momento de tiempo determinado.<br/>Ejemplo: t=60|
|captions|Código de idioma|Recupera el título en el idioma dado durante la carga del widget para que esté disponible en el menú de títulos.<br/>Ejemplo: captions=en-US|
|showCaptions|Un valor booleano|Hace que el reproductor se cargue con los títulos ya habilitados.<br/>Ejemplo: showCaptions=true|
|Tipo||Activa una máscara del reproductor de audio (se quita la parte de vídeo).<br/>Ejemplo: type=audio|
|autoplay|Un valor booleano|Indica si el reproductor debe comenzar a reproducir el vídeo cuando esté cargado (el valor predeterminado es true).<br/>Ejemplo: autoplay=false|
|language|Código de idioma|Controla el idioma del reproductor (el valor predeterminado es en-US)<br/>Ejemplo: language=de-DE|

## <a name="embedding-public-content"></a>Inserción de contenido público

1. Vaya al sitio web de [Video Indexer](https://www.videoindexer.ai/) e inicie sesión.
2. Haga clic en el vídeo con el que quiere trabajar.
3. Haga clic en el botón "insertar" que aparece debajo del vídeo.

    ![Widget](./media/video-indexer-embed-widgets/video-indexer-widget01.png)

    Después de hacer clic en el botón, aparecerá un estado modal de inserción en la pantalla donde podrá elegir qué widget quiere insertar en la aplicación.
    Si se selecciona un widget (**Player** o **Cognitive Insights**), se genera el código insertado para pegarlo en la aplicación.
 
4. Elija el tipo de widget que desee (**Insights Cognitive** o **Player**).
5. Copie el código para insertar y agregar agréguelo a la aplicación. 

    ![Widget](./media/video-indexer-embed-widgets/video-indexer-widget02.png)

## <a name="embedding-private-content"></a>Inserción de contenido privado

Puede obtener códigos para insertar de elementos emergentes insertados (como se muestra en la sección anterior) solo para vídeos **públicos**. 

Si desea insertar un vídeo **privado**, tiene que pasar un token de acceso en el atributo **src** de **iframe**:

`https://www.videoindexer.ai/embed/[insights | player]/<accountId>/<videoId>/?accessToken=<accessToken>`
    
Utilice [**Get Insights Widget**](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Video-Insights-Widget?&pattern=widget) API para obtener el contenido del widget Cognitive Insights, o use [**Get Video Access Token**](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Video-Access-Token?) y agréguelo como un parámetro de consulta a la dirección URL, tal como se indicó anteriormente. Especifique esta dirección URL como el valor **src** de **iframe**.

Si desea proporcionar funcionalidades de edición de información detallada (como las que tenemos en nuestra aplicación web) en el widget insertado, tendrá que pasar un token de acceso con permisos de edición. Use [**Get Insights Widget**](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Video-Insights-Widget?&pattern=widget) o [**Get Video Access Token**](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Video-Access-Token?) con **&allowEdit=true**. 

## <a name="widgets-interaction"></a>Interacción de widgets

El widget **Cognitive Insights** puede interactuar con un vídeo en la aplicación. En esta sección se muestra cómo conseguir esta interacción.

![Widget](./media/video-indexer-embed-widgets/video-indexer-widget03.png)

### <a name="cross-origin-communications"></a>Comunicaciones entre orígenes

Para que los widgets de Video Indexer se comuniquen con otros componentes, el servicio Video Indexer hace lo siguiente:

- Usa el método HTML5 de la comunicación entre orígenes **postMessage** y 
- Valida el mensaje mediante el origen VideoIndexer.ai. 

Si decide implementar su propio código del reproductor y realizar la integración con los widgets **Cognitive Insights**, es su responsabilidad validar el origen del mensaje que proviene de VideoIndexer.ai.

### <a name="embed-both-types-of-widgets-in-your-application--blog-recommended"></a>Inserción de los dos tipos de widgets en la aplicación / blog (recomendado) 

En esta sección se muestra cómo lograr la interacción entre dos widgets de Video Indexer para que cuando un usuario haga clic en el control de la información detallada de la aplicación, el reproductor salte al momento pertinente.

`<script src="https://breakdown.blob.core.windows.net/public/vb.widgets.mediator.js"></script>`

1. Copie el código para insertar del widget **Player**.
2. Copie el código para insertar de **Cognitive Insights**.
3. Agregue el [**archivo mediador**](https://breakdown.blob.core.windows.net/public/vb.widgets.mediator.js) para controlar la comunicación entre los dos widgets:

`<script src="https://breakdown.blob.core.windows.net/public/vb.widgets.mediator.js"></script>`

Ahora cuando un usuario hace clic en el control de información en la aplicación, el reproductor salta al momento relevante.

Para más información, consulte [esta demostración](https://codepen.io/videoindexer/pen/NzJeOb).

### <a name="embed-the-cognitive-insights-widget-and-use-azure-media-player-to-play-the-content"></a>Inserte el widget Cognitive Insights y utilice Azure Media Player para reproducir el contenido.

En esta sección se muestra cómo lograr la interacción entre un widget **Cognitive Insights** y una instancia de Azure Media Player mediante el [complemento AMP](https://breakdown.blob.core.windows.net/public/amp-vb.plugin.js).
 
1. Agregue un complemento de Video Indexer al reproductor de AMP.<br/> `<script src="https://breakdown.blob.core.windows.net/public/amp-vb.plugin.js"></script>`
2. Cree una instancia de Azure Media Player con el complemento de Video Indexer.

        // Init Source
        function initSource() {
            var tracks = [{
            kind: 'captions',
            // Here is how to load vtt from VI, you can replace it with your vtt url.
            src: this.getSubtitlesUrl("c4c1ad4c9a", "English"),
            srclang: 'en',
            label: 'English'
            }];

            myPlayer.src([
            {
                "src": "//amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest",
                "type": "application/vnd.ms-sstr+xml"
            }
            ], tracks);
        }

        // Init your AMP instance
        var myPlayer = amp('vid1', { /* Options */
            "nativeControlsForTouch": false,
            autoplay: true,
            controls: true,
            width: "640",
            height: "400",
            poster: "",
            plugins: {
            videobreakedown: {}
            }
        }, function () {
            // Activate the plugin
            this.videobreakdown({
            videoId: "c4c1ad4c9a",
            syncTranscript: true,
            syncLanguage: true
            });

            // Set the source dynamically
            initSource.call(this);
        });

3. Copie el código para insertar de **Cognitive Insights**.

Ahora debería poder comunicarse con Azure Media Player.

Para más información, consulte [esta demostración](https://codepen.io/videoindexer/pen/rYONrO).

### <a name="embed-video-indexer-cognitive-insights-widget-and-use-your-own-player-could-be-any-player"></a>Inserción del widget Cognitive Insights de Video Indexer y uso de su propio reproductor (puede ser cualquier reproductor)

Si utiliza su propio reproductor, tiene que encargarse de manipularlo para lograr la comunicación. 

1. Inserte el reproductor de vídeo.

    Por ejemplo, un reproductor HTML5 estándar

        <video id="vid1" width="640" height="360" controls autoplay preload>
           <source src="//breakdown.blob.core.windows.net/public/Microsoft%20HoloLens-%20RoboRaid.mp4" type="video/mp4" /> 
           Your browser does not support the video tag.
        </video>    

2. Inserte el widget Cognitive Insights.
3. Implemente la comunicación para el reproductor mediante la escucha del evento "mensaje". Por ejemplo:

        <script>
    
            (function(){
            // Reference your player instance
            var playerInstance = document.getElementById('vid1');
        
            function jumpTo(evt) {
              var origin = evt.origin || evt.originalEvent.origin;
        
              // Validate that event comes from the videobreakdown domain.
              if ((origin === "https://www.videobreakdown.com") && evt.data.time !== undefined){
                
                // Here you need to call your player "jumpTo" implementation
                playerInstance.currentTime = evt.data.time;
               
                // Confirm arrival to us
                if ('postMessage' in window) {
                  evt.source.postMessage({confirm: true, time: evt.data.time}, origin);
                }
              }
            }
        
            // Listen to message event
            window.addEventListener("message", jumpTo, false);
          
            }())    
        
        </script>

Para más información, consulte [esta demostración](https://codepen.io/videoindexer/pen/YEyPLd).

## <a name="adding-subtitles"></a>Adición de subtítulos

Si inserta información detallada de Video Indexer con su propio reproductor AMP, puede utilizar el método **GetVttUrl** para obtener subtítulos. También puede llamar a un método de javascript desde el complemento AMP de Video Indexer **getSubtitlesUrl** (como se ha mostrado anteriormente). 

## <a name="customizing-embeddable-widgets"></a>Personalización de widgets que se pueden insertar

### <a name="cognitive-insights-widget"></a>Widget Cognitive Insights

Para elegir los tipos de conclusión que quiera, puede especificarlos como valor en el siguiente parámetro URL agregado al código insertado que obtiene (de la API o de la aplicación web): `&widgets=<list of wanted widgets>`.

Los valores posibles son: personas, palabras clave, opiniones, transcripción, búsqueda.

Por ejemplo, si desea insertar un widget que contiene solo personas y buscar conclusiones, la dirección URL para insertar de IFrame tendrá este aspecto:

`https://www.videoindexer.ai/embed/insights/<accountId>/<videoId>/?widgets=people,search`

También se puede personalizar el título de la ventana de IFrame proporcionando `&title=<YourTitle>` a la dirección URL de IFrame. (Personalizará el html \<title > valor).
    
Por ejemplo, si quiere asignar a la ventana de IFrame el título "MyInsights", la dirección URL tendrá este aspecto:

`https://www.videoindexer.ai/embed/insights/<accountId>/<videoId>/?title=MyInsights`

Tenga en cuenta que esta opción solo es relevante en aquellos casos en los que necesite abrir la información detallada en una nueva ventana.

### <a name="player-widget"></a>Widget Player

Si inserta el reproductor de Video Indexer, puede elegir el tamaño del reproductor al especificar el tamaño de IFrame.

Por ejemplo:

`<iframe width="640" height="360" src="https://www.videoindexer.ai/embed/player/<accountId>/<videoId>/" frameborder="0" allowfullscreen />`

De forma predeterminada, el reproductor de Video Indexer tendrá subtítulos generados automáticamente basados en la transcripción del vídeo que se extrajo del vídeo con el idioma de origen que se seleccionó cuando se cargó el vídeo.

Si quiere hacer una inserción con un idioma diferente, puede agregar `&captions=< Language | ”all” | “false” >` a la dirección URL incrustada del reproductor o poner "all" como valor si quiere tener todos los subtítulos de idiomas disponibles.
Si quiere que los subtítulos se muestren de forma predeterminada, puede pasar `&showCaptions=true`.

La dirección URL para insertar tendrá un aspecto similar al siguiente: 

`https://www.videoindexer.ai/embed/player/<accountId>/<videoId>/?captions=italian`

Si quiere deshabilitar los subtítulos, puede pasar "false" como valor en el parámetro de subtítulos.

Reproducción automática: de forma predeterminada, el reproductor iniciará la reproducción del vídeo. Puede elegir no pasar &autoplay=false para la dirección URL para insertar anterior.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo ver y editar la información detallada de Video Indexer, consulte [este](video-indexer-view-edit.md) artículo.

Asimismo, consulte [Video indexer CodePen](https://codepen.io/videoindexer/pen/eGxebZ).
