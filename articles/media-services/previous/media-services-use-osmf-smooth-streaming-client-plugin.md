---
title: El complemento Smooth Streaming para Open Source Media Framework
description: Aprenda a usar el complemento Smooth Streaming de Azure Media Services para Adobe Open Source Media Framework.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 6068151f-b6b0-4507-9346-f03416d3d572
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 9b71a2064887cc2b6099ead2e59f9fa3113d2f0b
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269019"
---
# <a name="how-to-use-the-microsoft-smooth-streaming-plugin-for-the-adobe-open-source-media-framework"></a>Uso del complemento Smooth Streaming de Microsoft para Open Source Media Framework de Adobe

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Información general
El complemento Smooth Streaming de Microsoft para Open Source Media Framework 2.0 (SS para OSMF) amplía las capacidades predeterminadas de OSMF y agrega la reproducción del contenido Smooth Streaming de Microsoft a los reproductores con OSMF nuevos y existentes. El complemento también agrega las capacidades de reproducción de Smooth Streaming para Strobe Media Playback (SMP).

SS para OSMF incluye dos versiones del complemento:

* Complemento Smooth Streaming estático para OSMF (.swc)
* Complemento Smooth Streaming dinámico para OSMF (.swf)

En este documento se asume que el lector tiene un conocimiento general acerca del funcionamiento de los complementos OSMF y OSMF. Para obtener más información acerca de OSMF, consulte la documentación disponible en el [sitio oficial de OSMF (en inglés)](http://osmf.org/).

### <a name="smooth-streaming-plugin-for-osmf-20"></a>Complemento Smooth Streaming para OSMF 2.0
El complemento admite la carga y reproducción de contenido Smooth Streaming bajo demanda con las siguientes características:

* Reproducción de Smooth Streaming bajo demanda (Reproducir, Pausar, Buscar y Detener)
* Reproducción de Smooth Streaming en directo (Reproducir)
* Funciones de DVR en directo (Pausar, Buscar, Reproducción de DVR e Ir al directo)
* Compatibilidad con códecs de vídeo - H.264
* Compatibilidad con códecs de audio - AAC
* Conmutación de varios idiomas de audio con API integradas de OSMF
* Selección de calidad máxima de reproducción con API integradas de OSMF
* Títulos cerrados adicionales con el complemento de títulos de OSMF
* Adobe&reg; Flash&reg; Player 11.4 o superior.
* Esta versión solo admite OSMF 2.0.

## <a name="supported-features-and-known-issues"></a>Características admitidas y problemas conocidos
Para obtener una lista completa de características compatibles, características no compatibles y problemas conocidos, consulte [este documento](https://azure.microsoft.com/blog/microsoft-adaptive-streaming-plugin-for-osmf-update/).

## <a name="loading-the-plugin"></a>Carga del complemento
Los complementos de OSMF se pueden cargar estáticamente (en el momento de la compilación) o dinámicamente (en el tiempo de ejecución). La descarga del complemento Smooth Streaming para OSMF incluye las versiones dinámicas y estáticas.

* Carga estática: para esta carga, se necesita un archivo de biblioteca estática (SWC). Los complementos estáticos se agregan como una referencia a los proyectos y se combinan dentro del archivo de salida final en el momento de la compilación.
* Carga dinámica: para realizar esta carga, se necesita un archivo precompilado (SWF). Los complementos dinámicos se cargan durante el tiempo de ejecución y no se incluyen en el resultado del proyecto. (Resultado compilado) Los complementos dinámicos pueden cargarse con los protocolos HTTP y FILE.

Para obtener más información acerca de la carga estática y dinámica, consulte la [página oficial del complemento de OSMF (en inglés)](http://osmf.org/dev/osmf/OtherPDFs/osmf_plugin_dev_guide.pdf).

### <a name="ss-for-osmf-static-loading"></a>Carga estática de SS para OSMF
El fragmento de código siguiente muestra cómo cargar estáticamente el complemento SS para OSMF y reproducir vídeo básico con la utilización de la clase MediaFactory de OSMF. Antes de incluir el código SS para OSMF, asegúrese de que la referencia del proyecto incluye el complemento estático "MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swc".

```csharp
package 
{

    import com.microsoft.azure.media.AdaptiveStreamingPluginInfo;

    import flash.display.*;
    import org.osmf.media.*;
    import org.osmf.containers.MediaContainer;
    import org.osmf.events.MediaErrorEvent;
    import org.osmf.events.MediaFactoryEvent;
    import org.osmf.events.MediaPlayerStateChangeEvent;
    import org.osmf.layout.*;



    [SWF(width="1024", height="768", backgroundColor='#405050', frameRate="25")]
    public class TestPlayer extends Sprite
    {        
        public var _container:MediaContainer;
        public var _mediaFactory:DefaultMediaFactory;
        private var _mediaPlayerSprite:MediaPlayerSprite;


        public function TestPlayer( )
        {
            stage.quality = StageQuality.HIGH;

            initMediaPlayer();

        }

        private function initMediaPlayer():void
        {

            // Create the container (sprite) for managing display and layout
            _mediaPlayerSprite = new MediaPlayerSprite();    
            _mediaPlayerSprite.addEventListener(MediaErrorEvent.MEDIA_ERROR, onPlayerFailed);
            _mediaPlayerSprite.addEventListener(MediaPlayerStateChangeEvent.MEDIA_PLAYER_STATE_CHANGE, onPlayerStateChange);
            _mediaPlayerSprite.scaleMode = ScaleMode.NONE;
            _mediaPlayerSprite.width = stage.stageWidth;
            _mediaPlayerSprite.height = stage.stageHeight;
            //Adds the container to the stage
            addChild(_mediaPlayerSprite);

            // Create a mediafactory instance
            _mediaFactory = new DefaultMediaFactory();

            // Add the listeners for PLUGIN_LOADING
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD,onPluginLoaded);
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD_ERROR, onPluginLoadFailed );

            // Load the plugin class 
            loadAdaptiveStreamingPlugin( );  

        }

        private function loadAdaptiveStreamingPlugin( ):void
        {
            var pluginResource:MediaResourceBase;

            pluginResource = new PluginInfoResource(new AdaptiveStreamingPluginInfo( )); 
            _mediaFactory.loadPlugin( pluginResource ); 
        }

        private function onPluginLoaded( event:MediaFactoryEvent ):void
        {
            // The plugin is loaded successfully.
            // Your web server needs to host a valid crossdomain.xml file to allow plugin to download Smooth Streaming files.
        loadMediaSource("http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest")

        }

        private function onPluginLoadFailed( event:MediaFactoryEvent ):void
        {
            // The plugin is failed to load ...
        }


        private function onPlayerStateChange(event:MediaPlayerStateChangeEvent) : void
        {
            var state:String;

            state =  event.state;

            switch (state)
            {
                case MediaPlayerState.LOADING: 

                    // A new source is started to load.

                    break;

                case  MediaPlayerState.READY :   
                    // Add code to deal with Player Ready when it is hit the first load after a source is loaded. 

                    break;

                case MediaPlayerState.BUFFERING :

                    break;

                case  MediaPlayerState.PAUSED :
                    break;      
                // other states ...          
            }
        }

        private function onPlayerFailed(event:MediaErrorEvent) : void
        {
            // Media Player is failed .           
        }

        private function loadMediaSource(sourceURL : String):void 
        {
            // Take an URL of SmoothStreamingSource's manifest and add it to the page.

            var resource:URLResource= new URLResource( sourceURL );

            var element:MediaElement = _mediaFactory.createMediaElement( resource );
            _mediaPlayerSprite.scaleMode = ScaleMode.LETTERBOX;
            _mediaPlayerSprite.width = stage.stageWidth;
            _mediaPlayerSprite.height = stage.stageHeight;

            // Add the media element
            _mediaPlayerSprite.media = element;
        }     

    }
}
```


### <a name="ss-for-osmf-dynamic-loading"></a>Carga dinámica de SS para OSMF
El fragmento de código siguiente muestra cómo cargar dinámicamente el complemento SS para OSMF y reproducir vídeo básico con la utilización de la clase MediaFactory de OSMF. Antes de incluir el código SS para OSMF, copie el complemento "MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf" a la carpeta del proyecto si desea realizar la carga con el protocolo FILE, o bien cópielo en un servidor web para cargas HTTP. No es necesario incluir "MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swc" en las referencias del proyecto.

```csharp
package 
{

    import flash.display.*;
    import org.osmf.media.*;
    import org.osmf.containers.MediaContainer;
    import org.osmf.events.MediaErrorEvent;
    import org.osmf.events.MediaFactoryEvent;
    import org.osmf.events.MediaPlayerStateChangeEvent;
    import org.osmf.layout.*;
    import flash.events.Event;
    import flash.system.Capabilities;


    //Sets the size of the SWF

    [SWF(width="1024", height="768", backgroundColor='#405050', frameRate="25")]
    public class TestPlayer extends Sprite
    {        
        public var _container:MediaContainer;
        public var _mediaFactory:DefaultMediaFactory;
        private var _mediaPlayerSprite:MediaPlayerSprite;


        public function TestPlayer( )
        {
            stage.quality = StageQuality.HIGH;
            initMediaPlayer();
        }

        private function initMediaPlayer():void
        {

            // Create the container (sprite) for managing display and layout
            _mediaPlayerSprite = new MediaPlayerSprite();    
            _mediaPlayerSprite.addEventListener(MediaErrorEvent.MEDIA_ERROR, onPlayerFailed);
            _mediaPlayerSprite.addEventListener(MediaPlayerStateChangeEvent.MEDIA_PLAYER_STATE_CHANGE, onPlayerStateChange);

            //Adds the container to the stage
            addChild(_mediaPlayerSprite);

            // Create a mediafactory instance
            _mediaFactory = new DefaultMediaFactory();

            // Add the listeners for PLUGIN_LOADING
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD,onPluginLoaded);
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD_ERROR, onPluginLoadFailed );

            // Load the plugin class 
            loadAdaptiveStreamingPlugin( );  

        }

        private function loadAdaptiveStreamingPlugin( ):void
        {
            var pluginResource:MediaResourceBase;
            var adaptiveStreamingPluginUrl:String;

            // Your dynamic plugin web server needs to host a valid crossdomain.xml file to allow loading plugins.

            adaptiveStreamingPluginUrl = "http://yourdomain/MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf";
            pluginResource = new URLResource(adaptiveStreamingPluginUrl);
            _mediaFactory.loadPlugin( pluginResource ); 

        }

        private function onPluginLoaded( event:MediaFactoryEvent ):void
        {
            // The plugin is loaded successfully.

            // Your web server needs to host a valid crossdomain.xml file to allow plugin to download Smooth Streaming files.

    loadMediaSource("http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest")
        }

        private function onPluginLoadFailed( event:MediaFactoryEvent ):void
        {
            // The plugin is failed to load ...
        }


        private function onPlayerStateChange(event:MediaPlayerStateChangeEvent) : void
        {
            var state:String;

            state =  event.state;

            switch (state)
            {
                case MediaPlayerState.LOADING: 

                    // A new source is started to load.

                    break;

                case  MediaPlayerState.READY :   
                    // Add code to deal with Player Ready when it is hit the first load after a source is loaded. 

                    break;

                case MediaPlayerState.BUFFERING :

                    break;

                case  MediaPlayerState.PAUSED :
                    break;      
                // other states ...          
            }
        }

        private function onPlayerFailed(event:MediaErrorEvent) : void
        {
            // Media Player is failed .           
        }

        private function loadMediaSource(sourceURL : String):void 
        {
            // Take an URL of SmoothStreamingSource's manifest and add it to the page.

            var resource:URLResource= new URLResource( sourceURL );

            var element:MediaElement = _mediaFactory.createMediaElement( resource );
            _mediaPlayerSprite.scaleMode = ScaleMode.LETTERBOX;
            _mediaPlayerSprite.width = stage.stageWidth;
            _mediaPlayerSprite.height = stage.stageHeight;
            // Add the media element
            _mediaPlayerSprite.media = element;
        }     

    }
}
```

## <a name="strobe-media--playback-with-the-ss-odmf-dynamic-plugin"></a>Strobe Media Playback con el complemento dinámico SS para ODMF
El complemento dinámico Smooth Streaming para OSMF es compatible con [Strobe Media Playback (SMP)](https://sourceforge.net/adobe/smp/home/Strobe%20Media%20Playback/). Puede usar el complemento SS para OSMF para agregar la reproducción de contenido Smooth Streaming a SMP. Para ello, copie "MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf" en un servidor web para cargas HTTP mediante estos pasos:

1. Examine la [página de configuración de Strobe Media Playback (en inglés)](http://osmf.org/dev/2.0gm/setup.html). 
2. Defina src como un origen de Smooth Streaming (por ejemplo, http:\//devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest). 
3. Realice los cambios de configuración deseados y haga clic en Preview y Update.
   
   **Nota** : el servidor web de contenido necesita un archivo crossdomain.xml válido. 
4. Copie y pegue el código en una página HTML sencilla con su editor de texto favorito, como en el ejemplo siguiente:

    ```html
    <html>
    <body>
    <object width="920" height="640"> 
    <param name="movie" value="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf"></param>
    <param name="flashvars" value="src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest &autoPlay=true"></param>
    <param name="allowFullScreen" value="true"></param>
    <param name="allowscriptaccess" value="always"></param>
    <param name="wmode" value="direct"></param>
    <embed src="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf" 
        type="application/x-shockwave-flash" 
        allowscriptaccess="always" 
        allowfullscreen="true" 
        wmode="direct" 
        width="920" 
        height="640" 
        flashvars=" src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest&autoPlay=true">
    </embed>
    </object>
    </body>
    </html>
    ```


1. Agregue el complemento Smooth Streaming para OSMF al código embed y guárdelo.
   
    ```html
    <html>
    <object width="920" height="640"> 
    <param name="movie" value="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf"></param>
    <param name="flashvars" value="src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest&autoPlay=true&plugin_AdaptiveStreamingPlugin=http://yourdomain/MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf&AdaptiveStreamingPlugin_retryLive=true&AdaptiveStreamingPlugin_retryInterval=10"></param>
    <param name="allowFullScreen" value="true"></param>
    <param name="allowscriptaccess" value="always"></param>
    <param name="wmode" value="direct"></param>
    <embed src="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf" 
        type="application/x-shockwave-flash" 
        allowscriptaccess="always" 
        allowfullscreen="true" 
        wmode="direct" 
        width="920" 
        height="640" 
        flashvars="src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest&autoPlay=true&plugin_AdaptiveStreamingPlugin=http://yourdomain/MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf&AdaptiveStreamingPlugin_retryLive=true&AdaptiveStreamingPlugin_retryInterval=10">
    </embed>
    </object>
    </html>
    ```

2. Guarde la página HTML y publíquela en un servidor web. Vaya a la página web publicada utilizando el explorador de Internet que prefiera (Internet Explorer, Chrome y Firefox, entre otros) con Flash&reg; Player habilitado.
3. Disfrute del contenido de Smooth Streaming en Adobe&reg; Flash&reg; Player.

Para obtener más información acerca del desarrollo de OSMF general, consulte la [página oficial de desarrollo de OSMF (en inglés)](http://osmf.org/resources.html).

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Consulte también
[Actualización del complemento de streaming adaptable para OSMF](https://azure.microsoft.com/blog/2014/10/27/microsoft-adaptive-streaming-plugin-for-osmf-update/) 

