---
title: Inserción de anuncios en el lado cliente | Microsoft Docs
description: En este artículo se muestra cómo insertar anuncios en el medio del cliente.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 65c9c747-128e-497e-afe0-3f92d2bf7972
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 70db1ff63e3c2a7e7806e5ef3c55b4e4af4a5259
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89263593"
---
# <a name="inserting-ads-on-the-client-side"></a>Inserción de anuncios en el lado cliente

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Este tema contiene información sobre cómo insertar varios tipos de anuncios en el lado cliente.

Para obtener información acerca de la compatibilidad con anuncios y subtítulos en vídeos de streaming en vivo, consulte [Estándares de inserción de anuncios y subtítulos compatibles](media-services-live-streaming-with-onprem-encoders.md#cc_and_ads).

> [!NOTE]
> Actualmente, Azure Media Player no admite anuncios.
> 
> 

## <a name="inserting-ads-into-your-media"></a><a id="insert_ads_into_media"></a>Inserción de anuncios en contenido multimedia
Azure Media Services admite la inserción de anuncios mediante la plataforma de Windows Media: Player Frameworks. Player framework con compatibilidad con anuncios está disponible para dispositivos Windows 8, Silverlight, Windows Phone 8 e iOS. Cada marco de trabajo de reproductor contiene código de ejemplo que muestra cómo implementar una aplicación de reproductor. Hay tres tipos diferentes de anuncios que se pueden insertar en el archivo multimedia:

* **Lineales** : anuncios de fotograma completo que pausan el vídeo principal.
* **No lineales** : anuncios superpuestos que se muestran mientras se reproduce el vídeo principal, normalmente un logotipo u otra imagen estática colocada dentro del reproductor.
* **Complementarios** : anuncios que se muestran fuera del reproductor.

Los anuncios se pueden colocar en cualquier punto de la línea de tiempo del vídeo principal. Debe indicar al reproductor cuándo reproducir el anuncio y qué anuncios reproducir. Esto se realiza con un conjunto de archivos XML estándar: Video Ad Service Template (VAST, plantilla de servicio de anuncio de vídeo), Digital Video Multiple Ad Playlist (VMAP, lista de reproducción de varios anuncios de vídeo digital), Media Abstract Sequencing Template (MAST, plantilla de secuenciación abstracta multimedia) y Digital Video Player Ad Interface Definition (VPAID, definición de interfaz de anuncios del reproductor de vídeo digital). Los archivos VAST especifican qué anuncios mostrar. Los archivos VMAP especifican cuándo reproducir varios anuncios y contienen XML VAST. Los archivos MAST constituyen otra manera de secuenciar los anuncios que también pueden contener XML VAST. Los archivos VPAID definen una interfaz entre el reproductor de vídeo y el anuncio o el servidor de anuncios.

Cada Player Framework funciona de forma diferente y cada uno se explicará en su propio artículo. En este artículo se describen los mecanismos básicos usados para insertar anuncios. Las aplicaciones de reproductor de vídeo solicitan anuncios de un servidor de anuncios. El servidor de anuncios puede responder de varias maneras:

* Devolver un archivo VAST
* Devolver un archivo VMAP (con VAST insertado)
* Devolver un archivo MAST (con VAST insertado)
* Devolver un archivo VAST con anuncios VPAID

### <a name="using-a-video-ad-service-template-vast-file"></a>Mediante un archivo Video Ad Service Template (VAST)
Un archivo VAST especifica qué anuncios mostrar. El siguiente código XML es un ejemplo de un archivo VAST para un anuncio lineal:

```xml
    <VAST version="2.0" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
      <Ad id="115571748">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://www.myserver.com/tracking-resource]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:32</Duration>
                <TrackingEvents>
                  <Tracking event="start"><![CDATA[http://www.myserver.com/start-tracking-resource]]></Tracking>
                  <Tracking event="midpoint"><![CDATA[http://www.myserver.com/midpoint-tracking-resource]]></Tracking>
                  <Tracking event="complete"><![CDATA http://www.myserver.com/complete-tracking-resource]]></Tracking>
                  <Tracking event="expand"><![CDATA[http://www.myserver.com/expand-tracking-resource]]></Tracking>
                </TrackingEvents>
                <VideoClicks>
                  <ClickThrough id="Atlas Redirect"><![CDATA[http://www.myserver.com/click-resource]]></ClickThrough>
                  <ClickTracking id="Spare"></ClickTracking>
                </VideoClicks>
                <MediaFiles>
                  <MediaFile apiFramework="Windows Media" id="windows_progressive_200" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="200" width="400" height="300" type="video/x-ms-wmv">
                    <![CDATA[http://www.myserver.com/media/myad_200_4x3.wmv]]>
                  </MediaFile>
                  <MediaFile apiFramework="Windows Media" id="windows_progressive_300" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="300" width="400" height="300" type="video/x-ms-wmv">
                    <![CDATA[http://www.myserver.com/media/myad_300_4x3.wmv]]>
                  </MediaFile>
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
          <Extensions>
            <Extension type="Atlas">
            </Extension>
          </Extensions>
        </InLine>
      </Ad>
    </VAST>
```

El anuncio lineal se describe en el elemento <**Linear**>. Especifica la duración del anuncio, eventos de seguimiento, click-through, el seguimiento de clics y diversos elementos **MediaFile**. Los eventos de seguimiento se especifican dentro del elemento <**TrackingEvents**> y permiten que un servidor de anuncios realice el seguimiento de diversos eventos que se producen mientras se visualiza el anuncio. En este caso se realiza un seguimiento de los eventos start, midpoint, complete y expand. El evento start se produce cuando se muestra el anuncio. El evento midpoint se produce cuando se visualiza al menos el 50% de la escala de tiempo del anuncio. Cuando el anuncio llega al final, se produce el evento complete. El evento expand se produce cuando el usuario expande el reproductor de vídeo a pantalla completa. Los click-through se especifican con un elemento <**ClickThrough**> dentro de un elemento <**VideoClicks**>, que especifica el identificador URI de un recurso que se muestra cuando el usuario hace clic en el anuncio. ClickTracking se especifica en un elemento <**ClickTracking**>, también dentro del elemento <**VideoClicks**> y especifica un recurso de seguimiento para que el reproductor lo solicite cuando el usuario hace clic en el anuncio. Los elementos <**MediaFile**> especifican información sobre una codificación específica de un anuncio. Cuando haya más de un elemento <**MediaFile**>, el reproductor de vídeo puede elegir la mejor codificación para la plataforma.

Los anuncios lineales pueden mostrarse en un orden especificado. Para ello, agregue elementos `<Ad>` adicionales al archivo VAST y especifique el orden usando el atributo sequence. Esto se ilustra en el ejemplo siguiente:

```xml
    <VAST version="2.0" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
      <Ad id="1" sequence="0">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://myserver.com/Impression/Ad1trackingResource]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:32</Duration>
                <MediaFiles>
                  <!-- ... -->
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
        </InLine>
      </Ad>
      <Ad id="2" sequence="1">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://myserver.com/Impression/Ad2trackingResource]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:30</Duration>
                <MediaFiles>
                  <!-- ... -->
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
        </InLine>
      </Ad>
    </VAST>
```

Los anuncios no lineales se especifican también en un elemento `<Creative>`. El ejemplo siguiente muestra un elemento `<Creative>` que describe un anuncio no lineal.

```xml
    <Creative id="video" sequence="1" AdID="">
      <NonLinearAds>
        <NonLinear width="216" height="121" minSuggestedDuration="00:00:15">
          <StaticResource creativeType="image/png"><![CDATA[http://myserver/images/image.png]]></StaticResource>
          <StaticResource creativeType="image/jpg"><![CDATA[http://myserver/images/image.jpg]]></StaticResource>
        </NonLinear>
        <TrackingEvents>
             <Tracking event="acceptInvitation"><![CDATA[http://myserver/tracking/trackingID]></Tracking>
             <Tracking event="collapse"><![CDATA[http://myserver/tracking/trackingID2]]></Tracking>
         </TrackingEvents>
       </NonLinearAds>
    </Creative>
```

El elemento <**NonLinearAds**> puede contener uno o varios elementos <**NonLinear**>, y cada uno de ellos puede describir un anuncio no lineal. El elemento <**NonLinear**> especifica el recurso para el anuncio no lineal. El recurso puede ser <**StaticResource**>, <**IFrameResource**> o <**HTMLResource**>. \<**StaticResource**> describe un recurso no HTML y define un atributo creativeType que especifica cómo se muestra el recurso:

Image/gif, image/jpeg, image/png: el recurso se muestra en una etiqueta HTML <**img**>.

Application/x-javascript: el recurso se muestra en una etiqueta HTML <**script**>.

Application/x-shockwave-flash: el recurso se muestra en un reproductor Flash.

**IFrameResource** describe un recurso HTML que se puede mostrar en un elemento IFrame. **HTMLResource** describe un fragmento de código HTML que se puede insertar en una página web. **TrackingEvents** especifica los eventos de seguimiento y el identificador URI que se solicita cuando se produce el evento. En este ejemplo se realiza el seguimiento de los eventos acceptInvitation y collapse. Para más información sobre el elemento **NonLinearAds** y sus elementos secundarios, consulte IAB.NET/VAST. Tenga en cuenta que el elemento **TrackingEvents** se encuentra dentro del elemento **NonLinearAds**, en lugar del elemento **NonLinear**.

Los anuncios complementarios se definen dentro de un elemento `<CompanionAds>`. El elemento `<CompanionAds>` puede contener uno o varios elementos `<Companion>`. Cada elemento `<Companion>` describe un anuncio complementario y puede contener un `<StaticResource>`, una `<IFrameResource>` o un `<HTMLResource>`, que se especifican de la misma forma que un anuncio no lineal. Un archivo VAST puede contener varios anuncios complementarios y la aplicación de reproductor puede elegir el anuncio más apropiado para mostrar. Para obtener más información acerca de VAST, consulte [VAST 3.0](https://www.iab.net/media/file/VASTv3.0.pdf).

### <a name="using-a-digital-video-multiple-ad-playlist-vmap-file"></a>Uso de un archivo Digital Video Multiple Ad Playlist (VMAP)
Un archivo VMAP permite especificar cuándo se producen pausas comerciales, cuánto dura cada pausa, cuántos anuncios pueden mostrarse en una pausa y qué tipos de anuncios se pueden mostrar durante una pausa. El siguiente en un ejemplo de un archivo VMAP que define una única pausa comercial:

```xml
    <vmap:VMAP xmlns:vmap="http://www.iab.net/vmap-1.0" version="1.0">
      <vmap:AdBreak breakType="linear" breakId="mypre" timeOffset="start">
        <vmap:AdSource allowMultipleAds="true" followRedirects="true" id="1">
          <vmap:VASTData>
            <VAST version="2.0" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
              <Ad id="115571748">
                <InLine>
                  <AdSystem version="2.0 alpha">Atlas</AdSystem>
                  <AdTitle>Unknown</AdTitle>
                  <Description>Unknown</Description>
                  <Survey></Survey>
                  <Error></Error>
                  <Impression id="Atlas"><![CDATA[https://view.atdmt.com/000/sview/115571748/direct;ai.201582527;vt.2/01/634364885739970673]]></Impression>
                  <Creatives>
                    <Creative id="video" sequence="0" AdID="">
                      <Linear>
                        <Duration>00:00:32</Duration>
                        <MediaFiles>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_200" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="200" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_1_000_200_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_300" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="300" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_300_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_500" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="500" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_1_000_500_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_700" maintainAspectRatio="true" scalable="true" delivery="progressive" bitrate="700" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_700_4x3.wmv]]>
                          </MediaFile>
                        </MediaFiles>
                      </Linear>
                    </Creative>
                  </Creatives>
                </InLine>
              </Ad>
            </VAST>
          </vmap:VASTData>
        </vmap:AdSource>
        <vmap:TrackingEvents>
          <vmap:Tracking event="breakStart">
            http://MyServer.com/breakstart.gif
          </vmap:Tracking>
        </vmap:TrackingEvents>
      </vmap:AdBreak>
    </vmap:VMAP>
```

Un archivo VMAP comienza con un elemento `<VMAP>` que contiene uno o más elementos `<AdBreak>`, cada uno de los cuales define una pausa comercial. Cada pausa comercial especifica un tipo de pausa, el identificador de la pausa y la compensación de tiempo. El atributo breakType especifica el tipo de anuncio que se puede reproducir durante la pausa: linear, nonlinear o display. Los anuncios display se corresponden con los anuncios complementarios de VAST. Se puede especificar más de un tipo de anuncio en una lista separada por comas (sin espacios). breakID es el identificador opcional del anuncio. timeOffset especifica cuándo se debe mostrar el anuncio. Se puede especificar de una de las siguientes maneras:

1. Hora: en formato hh:mm:ss o hh:mm:ss.mmm, donde .mmm es milisegundos. El valor de este atributo especifica el tiempo que transcurre desde el principio de la escala de tiempo del vídeo al principio de la pausa comercial.
2. Porcentaje: en formato n%, donde n es el porcentaje de la escala de tiempo de vídeo que se reproducirá antes del anuncio.
3. Start/end: especifica que un anuncio debe mostrarse antes o después de que se muestre el vídeo.
4. Posición: especifica el orden de las pausas comerciales cuando no se conoce la temporización de las mismas, como en el caso del streaming en vivo. El orden de cada pausa publicitaria en el formato #n, donde n es un entero igual o mayor que 1. 1 significa que el anuncio debe reproducirse en la primera oportunidad; 2 significa el anuncio debe reproducirse en la segunda oportunidad, y así sucesivamente.

En el elemento `<AdBreak>`, puede haber un elemento <**AdSource**>. El elemento <**AdSource**> contiene los siguientes atributos:

1. Id: especifica el identificador del origen del anuncio.
2. allowMultipleAds: valor booleano que especifica si se pueden mostrar varios anuncios durante la pausa comercial.
3. followRedirects: un valor booleano opcional que especifica si el reproductor de vídeo debe respetar las redirecciones dentro de una respuesta de anuncio.

El elemento <**AdSource**> proporciona al reproductor una respuesta de anuncio en línea o una referencia a una respuesta de anuncio. Puede contener uno de los siguientes elementos:

* `<VASTAdData>`: indica que una respuesta de anuncio VAST se inserta dentro del archivo VMAP.
* `<AdTagURI>`: un URI que hace referencia a una respuesta de anuncio desde otro sistema.
* `<CustomAdData>`: cadena arbitraria que representa una respuesta distinta de VAST.

En este ejemplo se especifica una respuesta de anuncio en línea con un elemento `<VASTAdData>` que contiene una respuesta de anuncio VAST. Para obtener más información acerca de los demás elementos, consulte [VMAP](https://www.iab.net/guidelines/508676/digitalvideo/vsuite/vmap).

El elemento <**AdBreak**> también puede contener un elemento <**TrackingEvents**>. El elemento <**TrackingEvents**> permite realizar el seguimiento del inicio o fin de una pausa comercial o de si se produjo un error durante la pausa comercial. El elemento <**TrackingEvents**> contiene uno o más <**Tracking**>, cada uno de los cuales especifica un evento de seguimiento y un URI de seguimiento. Los posibles eventos de seguimiento son:

1. breakStart: realiza un seguimiento del principio de una pausa comercial
2. breakend: realiza un seguimiento de la finalización de una pausa comercial
3. error: realiza un seguimiento de un error producido durante la pausa comercial

En el ejemplo siguiente se muestra un archivo VMAP que especifica eventos de seguimiento.

```xml
    <vmap:VMAP xmlns:vmap="http://www.iab.net/vmap-1.0" version="1.0">
      <vmap:AdBreak breakType="linear" breakId="mypre" timeOffset="start">
        <vmap:AdSource allowMultipleAds="true" followRedirects="true" id="1">
          <vmap:VASTData>
            <!--Inline VAST -->
          </vmap:VASTData>
        </vmap:AdSource>
        <vmap:TrackingEvents>
          <vmap:Tracking event="breakStart">
            http://MyServer.com/breakstart.gif
          </vmap:Tracking>
          <vmap:Tracking event="breakend">
            http://MyServer.com/breakend.gif
          </vmap:Tracking>
          <vmap:Tracking event="error">
            http://MyServer.com/error.gif
          </vmap:Tracking>
        </vmap:TrackingEvents>
      </vmap:AdBreak>
    </vmap:VMAP>
```

Para información sobre el elemento <**TrackingEvents**> y sus elementos secundarios, consulte http://iab.net/VMAP.pdf

### <a name="using-a-media-abstract-sequencing-template-mast-file"></a>Uso de un archivo Media Abstract Sequencing Template (MAST)
Un archivo MAST permite especificar desencadenadores que definen cuándo se muestra un anuncio. El siguiente es un archivo MAST de ejemplo que contiene desencadenadores para un anuncio de cuña previa, de cuña intermedia y de cuña posterior.

```xml
    <MAST xsi:schemaLocation="http://openvideoplayer.sf.net/mast http://openvideoplayer.sf.net/mast/mast.xsd" xmlns="http://openvideoplayer.sf.net/mast" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
      <triggers>
        <trigger id="preroll" description="preroll every item"  >
          <startConditions>
            <condition type="event" name="OnItemStart" />
          </startConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>

        <trigger id="midroll" description="midroll at 15 sec."  >
          <startConditions>
            <condition type="property" name="Position" value="00:00:15.0" operator="GEQ" />
          </startConditions>
          <endConditions>
            <condition type="event" name="OnItemEnd"/>
            <!--This 'resets' the trigger for the next clip-->
          </endConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>

        <trigger id="postroll" description="postroll"  >
          <startConditions>
            <condition type="event" name="OnItemEnd"/>
          </startConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>
      </triggers>
    </MAST>
```


Un archivo MAST comienza con un elemento **MAST** que contiene un elemento **triggers**. El elemento `<triggers>` contiene uno o varios elementos **trigger** que definen cuándo se debe reproducir un anuncio.

El elemento **trigger** contiene un elemento **startConditions** que especifica cuándo debe comenzar a reproducirse un anuncio. El elemento **startConditions** contiene uno o varios elementos `<condition>`. Cuando cada `<condition>` se evalúa como true, se inicia o se revoca un desencadenador en función de si `<condition>` está dentro de un elemento **startConditions** o **endConditions**, respectivamente. Cuando hay varios elementos `<condition>` presentes, se tratan como un OR implícito y cualquier condición que se evalúe como true hará que el desencadenador se inicie. Los elementos `<condition>` pueden estar anidados. Cuando hay elementos `<condition>` secundarios predefinidos, se tratan como un AND implícito y todas las condiciones deben evaluarse como true para iniciar el desencadenador. El elemento `<condition>` contiene los siguientes atributos que definen la condición:

1. **type**: especifica el tipo de condición, evento o propiedad.
2. **name** : nombre de la propiedad o evento que se usará durante la evaluación.
3. **value** : valor de una propiedad con la que se evaluará.
4. **operator**: operación que se va a usar durante la evaluación: EQ (igual), NEQ (distinto), GTR (mayor que), GEQ (mayor o igual que), LT (menor que), LEQ (menor o igual que), MOD (módulo)

**endConditions** también contiene elementos `<condition>`. Cuando una condición se evalúa como true, el desencadenador se restablece. El elemento `<trigger>` también contiene un elemento `<sources>` que contiene uno o varios elementos `<source>`. Los elementos `<source>` definen el URI a la respuesta de anuncio y el tipo de respuesta de anuncio. En este ejemplo se proporciona un URI a una respuesta VAST.

```xml
    <trigger id="postroll" description="postroll"  >
      <startConditions>
        <condition/>
      </startConditions>
      <sources>
        <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
          <sources />
        </source>
      </sources>
    </trigger>
```

### <a name="using-video-player-ad-interface-definition-vpaid"></a>Uso de Video Player-Ad Interface Definition (VPAID)
VPAID es una API que permite a las unidades de anuncios ejecutables comunicarse con un reproductor de vídeo. Esto permite disfrutar de experiencias de anuncios altamente interactivas. El usuario puede interactuar con el anuncio y el anuncio puede responder a las acciones realizadas por el espectador. Por ejemplo, un anuncio puede mostrar botones que permiten al usuario ver más información o una versión más larga del anuncio. El reproductor de vídeo debe admitir la API VPAID y el anuncio ejecutable debe implementar la API. Cuando un reproductor solicita un anuncio de un servidor de anuncios, el servidor puede responder con una respuesta VAST que contiene un anuncio VPAID.

Se crea un anuncio ejecutable en el código que debe ejecutarse en un entorno en tiempo de ejecución, como Adobe Flash ™ o JavaScript, que se pueden ejecutar en un explorador web. Cuando un servidor de anuncios devuelve una respuesta VAST que contiene un anuncio VPAID, el valor del atributo apiFramework en el elemento `<MediaFile>` debe ser "VPAID". Este atributo especifica que el anuncio contenido es un ejecutable VPAID. El atributo type debe establecerse en el tipo MIME del ejecutable, por ejemplo, "application/x-shockwave-flash" o "application/x-javascript". El siguiente fragmento XML muestra el elemento `<MediaFile>` de una respuesta VAST que contiene un anuncio ejecutable VPAID.

```xml
    <MediaFiles>
       <MediaFile id="1" delivery="progressive" type="application/x-shockwaveflash"
                  width="640" height="480" apiFramework="VPAID">
           <!-- CDATA wrapped URI to executable ad -->
       </MediaFile>
    </MediaFiles>
```

Un anuncio ejecutable se puede inicializar usando el elemento `<AdParameters>` dentro de los elementos `<Linear>` o `<NonLinear>` en una respuesta VAST. Para obtener más información sobre el elemento `<AdParameters>`, vea [VAST 3.0](https://www.iab.net/media/file/VASTv3.0.pdf). Para obtener más información acerca de la API VPAID, consulte [VPAID 2.0](https://www.iab.net/media/file/VPAID_2.0_Final_04-10-2012.pdf).

## <a name="implementing-a-windows-or-windows-phone-8-player-with-ad-support"></a>Implementación de un reproductor de Windows o Windows Phone 8 con compatibilidad para anuncios
Microsoft Media Platform: Player Framework para Windows 8 y Windows Phone 8 contiene una colección de aplicaciones de ejemplo que muestran cómo implementar una aplicación de reproductor de vídeo usando el marco. Puede descargar Player Framework y los ejemplos de [Player Framework para Windows 8 y Windows Phone 8](https://playerframework.codeplex.com).

Cuando abra la solución Microsoft.PlayerFramework.Xaml.Samples, verá varias carpetas en el proyecto. La carpeta Advertising contiene el código de ejemplo correspondiente a la creación de un reproductor de vídeo con compatibilidad para anuncios. Dentro de la carpeta Advertising, hay varios archivos XAML/cs que muestran diferentes maneras de insertar anuncios. La lista siguiente describe cada una de ellas:

* AdPodPage.xaml: ilustra cómo mostrar un bloque de anuncios.
* AdSchedulingPage.xaml: muestra cómo programar anuncios.
* FreeWheelPage.xaml: muestra cómo usar el complemento FreeWheel para programar anuncios.
* MastPage.xaml: muestra cómo programar anuncios con un archivo MAST.
* ProgrammaticAdPage.xaml: muestra cómo programar anuncios en un vídeo mediante programación.
* ScheduleClipPage.xaml: muestra cómo programar un anuncio sin un archivo VAST.
* VastLinearCompanionPage.xaml: muestra cómo insertar un anuncio lineal y uno complementario.
* VastNonLinearPage.xaml: muestra cómo insertar un anuncio no lineal.
* VmapPage.xaml: muestra cómo especificar anuncios con un archivo VMAP.

Cada uno de estos ejemplos usa la clase MediaPlayer definida por el Player Framework. En la mayoría de los ejemplos se usan complementos que agregan compatibilidad para varios formatos de respuesta de anuncio. El ejemplo ProgrammaticAdPage interactúa mediante programación con una instancia de MediaPlayer.

### <a name="adpodpage-sample"></a>Ejemplo AdPodPage
Este ejemplo usa el AdSchedulerPlugin para definir cuándo se debe mostrar un anuncio. En este ejemplo, se programa un anuncio de cuña intermedia para que se reproduzca transcurridos cinco segundos. El bloque de anuncios (un grupo de anuncios que se muestran en orden) se especifica en un archivo VAST que se devuelve desde un servidor de anuncios. El URI del archivo VAST se especifica en el elemento `<RemoteAdSource>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">

        <mmppf:MediaPlayer.Plugins>
            <ads:AdSchedulerPlugin>
                <ads:AdSchedulerPlugin.Advertisements>

                    <ads:MidrollAdvertisement Time="00:00:05">
                        <ads:MidrollAdvertisement.Source>
                            <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_adpod.xml" Type="vast"/>
                        </ads:MidrollAdvertisement.Source>
                    </ads:MidrollAdvertisement>

                </ads:AdSchedulerPlugin.Advertisements>
            </ads:AdSchedulerPlugin>
            <ads:AdHandlerPlugin/>
        </mmppf:MediaPlayer.Plugins>
    </mmppf:MediaPlayer>
```

Para obtener más información acerca del AdSchedulerPlugin, consulte [Publicidad en Player Framework en Windows 8 y Windows Phone 8](https://playerframework.codeplex.com/wikipage?title=Advertising&referringTitle=Windows%208%20Player%20Documentation)

### <a name="adschedulingpage"></a>AdSchedulingPage
Este ejemplo también usa el AdSchedulerPlugin. Programa tres anuncios, un anuncio de cuña previa, uno de cuña intermedia y uno de cuña posterior. El URI del archivo VAST de cada anuncio se especifica en un elemento `<RemoteAdSource>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:PrerollAdvertisement>
                                <ads:PrerollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:PrerollAdvertisement.Source>
                            </ads:PrerollAdvertisement>

                            <ads:MidrollAdvertisement Time="00:00:15">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                            <ads:PostrollAdvertisement>
                                <ads:PostrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:PostrollAdvertisement.Source>
                            </ads:PostrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="freewheelpage"></a>FreeWheelPage
En este ejemplo se usa FreeWheelPlugin, que especifica un atributo Source que especifica un URI que apunta a un archivo SmartXML que especifica el contenido del anuncio así como la información de programación.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:FreeWheelPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/freewheel.xml"/>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="mastpage"></a>MastPage
Este ejemplo usa el MastSchedulerPlugin, que permite usar un archivo MAST. El atributo Source especifica la ubicación del archivo MAST.
```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:MastSchedulerPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/mast.xml" />
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="programmaticadpage"></a>ProgrammaticAdPage
Este ejemplo interactúa mediante programación con MediaPlayer. El archivo ProgrammaticAdPage.xaml crea una instancia de MediaPlayer:

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4"/>
```

El archivo ProgrammaticAdPage.xaml.cs crea un elemento AdHandlerPlugin, agrega un elemento TimelineMarker para especificar cuándo se debe mostrar un anuncio y, después, agrega un controlador para el evento MarkerReached que carga un elemento RemoteAdSource que especifica el URI de un archivo VAST y, después, reproduce el anuncio.

```csharp
    public sealed partial class ProgrammaticAdPage : Microsoft.PlayerFramework.Samples.Common.LayoutAwarePage
        {
            AdHandlerPlugin adHandler;

            public ProgrammaticAdPage()
            {
                this.InitializeComponent();
                adHandler = new AdHandlerPlugin();
                player.Plugins.Add(new AdHandlerPlugin());
                player.Markers.Add(new TimelineMarker() { Time = TimeSpan.FromSeconds(5), Type = "myAd" });
                player.MarkerReached += pf_MarkerReached;
            }

            async void pf_MarkerReached(object sender, TimelineMarkerRoutedEventArgs e)
            {
                if (e.Marker.Type == "myAd")
                {
                    var adSource = new RemoteAdSource() { Type = VastAdPayloadHandler.AdType, Uri = new Uri("http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml") };
                    //var adSource = new AdSource() { Type = DocumentAdPayloadHandler.AdType, Payload = SampleAdDocument };
                    var progress = new Progress<AdStatus>();
                    try
                    {
                        await player.PlayAd(adSource, progress, CancellationToken.None);
                    }
                    catch { /* ignore */ }
                }
            }
```

### <a name="scheduleclippage"></a>ScheduleClipPage
Este ejemplo usa el AdSchedulerPlugin para programar un anuncio de cuña intermedia especificando un archivo .wmv que contiene el anuncio.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.cloudapp.net/html5/media/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:AdSource Type="clip">
                                        <ads:AdSource.Payload>
                                            <ads:ClipAdPayload MediaSource="http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_700_4x3.wmv" MimeType="video/x-ms-wmv" />
                                        </ads:AdSource.Payload>
                                    </ads:AdSource>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vastlinearcompanionpage"></a>VastLinearCompanionPage
En este ejemplo se muestra cómo usar AdSchedulerPlugin para programar un anuncio lineal de cuña intermedia con un anuncio complementario. El elemento `<RemoteAdSource>` especifica la ubicación del archivo VAST.

```xml
    <mmppf:MediaPlayer Grid.Row="1"  x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear_companions.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vastlinearnonlinearpage"></a>VastLinearNonLinearPage
Este ejemplo usa el AdSchedulerPlugin para programar un anuncio lineal y uno no lineal. La ubicación del archivo VAST se especifica con el elemento `<RemoteAdSource>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear_nonlinear.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vmappage"></a>VMAPPage
Los ejemplos usan VmapSchedulerPlugin para programar anuncios con un archivo VMAP. El URI del archivo VMAP se especifica en el atributo Source del elemento `<VmapSchedulerPlugin>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:VmapSchedulerPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/vmap.xml"/>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

## <a name="implementing-an-ios-video-player-with-ad-support"></a>Implementación de un reproductor de vídeo de iOS con compatibilidad para anuncios
Microsoft Media Platform: Player Framework para iOS contiene una colección de aplicaciones de ejemplo que muestran cómo implementar una aplicación de reproductor de vídeo usando el marco. Puede descargar la plataforma del reproductor y los ejemplos de [Plataforma de Azure Media Player](https://github.com/CloudMetal/azure-media-player-framework). La página de GitHub incluye un vínculo a un wiki que contiene información adicional sobre el marco del reproductor y una introducción al ejemplo del reproductor: [Wiki de Azure Media Player](https://github.com/CloudMetal/azure-media-player-framework/wiki/How-to-use-Azure-media-player-framework).

### <a name="scheduling-ads-with-vmap"></a>Programación de anuncios con VMAP
El ejemplo siguiente muestra cómo programar anuncios usando un archivo VMAP.

```csharp
    // How to schedule an Ad using VMAP.
    //First download the VMAP manifest

    if (![framework.adResolver downloadManifest:&manifest withURL:[NSURL URLWithString:@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVMAP.xml"]])
            {
                [self logFrameworkError];
            }
            else
            {
                // Schedule a list of ads using the downloaded VMAP manifest
                if (![framework scheduleVMAPWithManifest:manifest])
                {
                    [self logFrameworkError];
                }
            }
```

### <a name="scheduling-ads-with-vast"></a>Programación de anuncios con VAST
El ejemplo siguiente muestra cómo programar un anuncio VAST de enlace en tiempo de ejecución.


```csharp
    //Example:3 How to schedule a late binding VAST ad.
    // set the start time for the ad
    adLinearTime.startTime = 13;
    adLinearTime.duration = 0;
    // Specify the URI of the VAST file
    NSString *vastAd1=@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVAST.xml";
    // Create an AdInfo object
     AdInfo *vastAdInfo1 = [[[AdInfo alloc] init] autorelease];
    // set URL to VAST file
    vastAdInfo1.clipURL = [NSURL URLWithString:vastAd1];
    // set running time of ad
    vastAdInfo1.mediaTime = [[[MediaTime alloc] init] autorelease];
    vastAdInfo1.mediaTime.clipBeginMediaTime = 0;
    vastAdInfo1.mediaTime.clipEndMediaTime = 10;
    vastAdInfo1.policy = @"policy for late binding VAST";
    // specify ad type
    vastAdInfo1.type = AdType_Midroll;
    vastAdInfo1.appendTo=-1;
    adIndex = 0;
    // schedule ad
    if (![framework scheduleClip:vastAdInfo1 atTime:adLinearTime forType:PlaylistEntryType_VAST andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

   El ejemplo siguiente muestra cómo programar un anuncio VAST de enlace anticipado.

```csharp
    //Example:4 Schedule an early binding VAST ad
    //Download the VAST file
    if (![framework.adResolver downloadManifest:&manifest withURL:[NSURL URLWithString:@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVAST.xml"]])
    {
        [self logFrameworkError];
    }
    else
    {
        adLinearTime.startTime = 7;
        adLinearTime.duration = 0;

        // Create AdInfo instance
        AdInfo *vastAdInfo2 = [[[AdInfo alloc] init] autorelease];
        vastAdInfo2.mediaTime = [[[MediaTime alloc] init] autorelease];
        vastAdInfo2.policy = @"policy for early binding VAST";
        // specify ad type
        vastAdInfo2.type = AdType_Midroll;
        vastAdInfo2.appendTo=-1;
        // schedule ad
        if (![framework scheduleVASTClip:vastAdInfo2 withManifest:manifest atTime:adLinearTime andGetClipId:&adIndex])
        {
            [self logFrameworkError];
        }
    }
```

El ejemplo siguiente muestra cómo insertar un anuncio usando RCE (edición tentativa)

```csharp
    //Example:1 How to use RCE.
    // specify manifest for ad content
    NSString *secondContent=@"http://wamsblureg001orig-hs.cloudapp.net/6651424c-a9d1-419b-895c-6993f0f48a26/The%20making%20of%20Microsoft%20Surface-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";

    // specify ad length
    mediaTime.currentPlaybackPosition = 0;
    mediaTime.clipBeginMediaTime = 0;
    mediaTime.clipEndMediaTime = 80;
    // append ad content
    if (![framework appendContentClip:[NSURL URLWithString:secondContent] withMediaTime:mediaTime andGetClipId:&clipId])
    {
        [self logFrameworkError];
    }
```

El ejemplo siguiente muestra cómo programar un bloque de anuncios.

```csharp
    //Example:5 Schedule an ad Pod.
    // Set start time for ad
    adLinearTime.startTime = 23;
    adLinearTime.duration = 0;

    // Specify URL to content
    NSString *adpodSt1=@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/asset-e47b43fd-05dc-4587-ac87-5916439ad07f/Windows%208_%20Cliffjumpers.mp4?st=2012-11-28T16%3A31%3A57Z&se=2014-11-28T16%3A31%3A57Z&sr=c&si=2a6dbb1e-f906-4187-a3d3-7e517192cbd0&sig=qrXYZBekqlbbYKqwovxzaVZNLv9cgyINgMazSCbdrfU%3D";
    // Create an AdInfo instance
    AdInfo *adpodInfo1 = [[[AdInfo alloc] init] autorelease];
    // set URI to ad content
    adpodInfo1.clipURL = [NSURL URLWithString:adpodSt1];
    // Set ad running time
    adpodInfo1.mediaTime = [[[MediaTime alloc] init] autorelease];
    adpodInfo1.mediaTime.clipBeginMediaTime = 0;
    adpodInfo1.mediaTime.clipEndMediaTime = 17;
    adpodInfo1.policy = @"policy for ad pod 1";
    // Set ad type
    adpodInfo1.type = AdType_Midroll;
    adpodInfo1.appendTo=-1;
    adIndex = 0;
    // Schedule ad
    if (![framework scheduleClip:adpodInfo1 atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

El ejemplo siguiente muestra cómo programar una cuña intermedia no estática. Un anuncio no estático solo se reproduce una vez independientemente de la búsqueda que realice el espectador.

```csharp
    //Example:6 Schedule a single non sticky mid roll Ad
    // specify URL to content
    NSString *oneTimeAd=@"http://wamsblureg001orig-hs.cloudapp.net/5389c0c5-340f-48d7-90bc-0aab664e5f02/Windows%208_%20You%20and%20Me%20Together-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";

    // create an AdInfo instance
    AdInfo *oneTimeInfo = [[[AdInfo alloc] init] autorelease];
    // set URL of ad
    oneTimeInfo.clipURL = [NSURL URLWithString:oneTimeAd];
    oneTimeInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    oneTimeInfo.mediaTime.clipBeginMediaTime = 0;
    oneTimeInfo.mediaTime.clipEndMediaTime = -1;
    oneTimeInfo.policy = @"policy for one-time ad";
    // set ad start time
    adLinearTime.startTime = 43;
    adLinearTime.duration = 0;
    // set ad type
    oneTimeInfo.type = AdType_Midroll;
    // non sticky ad
    oneTimeInfo.deleteAfterPlayed = YES;
    // schedule ad
    if (![framework scheduleClip:oneTimeInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

El ejemplo siguiente muestra cómo programar una cuña intermedia estática. Un anuncio estático se muestra cada vez que se alcanza el punto especificado en la escala de tiempo del vídeo.

```csharp
    //Example:7 Schedule a single sticky mid roll Ad
    NSString *stickyAd=@"http://wamsblureg001orig-hs.cloudapp.net/2e4e7d1f-b72a-4994-a406-810c796fc4fc/The%20Surface%20Movement-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    // create AdInfo instance
    AdInfo *stickyAdInfo = [[[AdInfo alloc] init] autorelease];
    // set URI to ad
    stickyAdInfo.clipURL = [NSURL URLWithString:stickyAd];
    stickyAdInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    stickyAdInfo.mediaTime.clipBeginMediaTime = 0;
    stickyAdInfo.mediaTime.clipEndMediaTime = 15;
    stickyAdInfo.policy = @"policy for sticky mid-roll ad";
    // set ad start time
    adLinearTime.startTime = 64;
    adLinearTime.duration = 0;
    // set ad type
    stickyAdInfo.type = AdType_Midroll;
    stickyAdInfo.deleteAfterPlayed = NO;
    // schedule ad
    if (![framework scheduleClip:stickyAdInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

El ejemplo siguiente muestra cómo programar un anuncio de cuña posterior.

```csharp
    //Example:8 Schedule Post Roll Ad
    NSString *postAdURLString=@"http://wamsblureg001orig-hs.cloudapp.net/aa152d7f-3c54-487b-ba07-a58e0e33280b/wp-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    // create AdInfo instance
    AdInfo *postAdInfo = [[[AdInfo alloc] init] autorelease];
    postAdInfo.clipURL = [NSURL URLWithString:postAdURLString];
    postAdInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    postAdInfo.mediaTime.clipBeginMediaTime = 0;
    // set ad length
    postAdInfo.mediaTime.clipEndMediaTime = 45;
    postAdInfo.policy = @"policy for post-roll ad";
    // set ad type
    postAdInfo.type = AdType_Postroll;
    adLinearTime.duration = 0;
    if (![framework scheduleClip:postAdInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

El ejemplo siguiente muestra cómo programar un anuncio de cuña anterior.

```csharp
    //Example:9 Schedule Pre Roll Ad
    NSString *adURLString = @"http://wamsblureg001orig-hs.cloudapp.net/2e4e7d1f-b72a-4994-a406-810c796fc4fc/The%20Surface%20Movement-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    AdInfo *adInfo = [[[AdInfo alloc] init] autorelease];
    adInfo.clipURL = [NSURL URLWithString:adURLString];
    adInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    adInfo.mediaTime.currentPlaybackPosition = 0;
    adInfo.mediaTime.clipBeginMediaTime = 40; //You could play a portion of an Ad. Yeh!
    adInfo.mediaTime.clipEndMediaTime = 59;
    adInfo.policy = @"policy for pre-roll ad";
    adInfo.appendTo = -1;
    adInfo.type = AdType_Preroll;
    adLinearTime.duration = 0;
    // schedule ad
    if (![framework scheduleClip:adInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

El ejemplo siguiente muestra cómo programar un anuncio superpuesto de cuña intermedia.

```csharp
    // Example10: Schedule a Mid Roll overlay Ad
    NSString *adURLString = @"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/asset-e47b43fd-05dc-4587-ac87-5916439ad07f/Windows%208_%20Cliffjumpers.mp4?st=2012-11-28T16%3A31%3A57Z&se=2014-11-28T16%3A31%3A57Z&sr=c&si=2a6dbb1e-f906-4187-a3d3-7e517192cbd0&sig=qrXYZBekqlbbYKqwovxzaVZNLv9cgyINgMazSCbdrfU%3D";
    //Create AdInfo instance
    AdInfo *adInfo = [[[AdInfo alloc] init] autorelease];
    adInfo.clipURL = [NSURL URLWithString:adURLString];
    adInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    adInfo.mediaTime.currentPlaybackPosition = 0;
    adInfo.mediaTime.clipBeginMediaTime = 0;
    // specify ad length
    adInfo.mediaTime.clipEndMediaTime = 20;
    adInfo.policy = @"policy for mid-roll overlay ad";
    adInfo.appendTo = -1;
    // specify ad type
    adInfo.type = AdType_Midroll;
    // specify ad start time & duration
    adLinearTime.startTime = 300;
    adLinearTime.duration = 20;
    // schedule ad            if (![framework scheduleClip:adInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```


## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
