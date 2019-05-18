---
title: Examine la salida de vídeo de Azure Media Services Indexer producida por la API v2
titlesuffix: Azure Media Services
description: En este tema se examina la salida de Video Indexer producida por la API v2.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: juliako
ms.openlocfilehash: 205dc7d9e69788ea29a48ff342844a4b74e143bd
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799080"
---
# <a name="examine-the-video-indexer-output-produced-by-api"></a>Examine la salida de Video Indexer producida por la API

Cuando se llama a **Get Video Index** API y el estado de la respuesta es correcto, obtendrá una salida JSON detallada como contenido de la respuesta. El contenido JSON incluye información detallada del vídeo especificado. La información incluye dimensiones como: las transcripciones, aplica OCR, caras, temas, bloques, etcetera. Las dimensiones tienen instancias de intervalos de tiempo que muestran cuándo apareció cada dimensión en el vídeo.  

También puede examinar visualmente la información detallada del vídeo. Para ello, presione el botón **Reproducir** que hay encima del vídeo en el sitio web de [Video Indexer](https://www.videoindexer.ai/). Para más información, consulte [Visualización y edición de la información detallada de un vídeo](video-indexer-view-edit.md).

![Información detallada](./media/video-indexer-output-json/video-indexer-summarized-insights.png)

En este artículo se examina el contenido JSON devuelto por **Get Video Index** API. 

> [!NOTE]
> Todos los tokens de acceso en Video Indexer expiran en una hora.


## <a name="root-elements"></a>Elementos raíz

|Nombre|Descripción|
|---|---|
|accountId|Identificador de la cuenta de Video Indexer de la lista de reproducción.|
|id|Identificador de la lista de reproducción.|
|name|Nombre de la lista de reproducción.|
|description|Descripción de la lista de reproducción.|
|userName|Nombre del usuario que creó la lista de reproducción.|
|created|Hora de creación de la lista de reproducción.|
|privacyMode|Modo de privacidad de la lista de reproducción (privada/pública).|
|state|Estado de la lista de reproducción (cargada, en proceso, procesada, error, en cuarentena).|
|isOwned|Indica si la lista de reproducción la creó el usuario actual.|
|isEditable|Indica si el usuario actual está autorizado para editar la lista de reproducción.|
|isBase|Indica si la lista de reproducción es una lista de reproducción base (un vídeo) o una lista de reproducción formada por otros vídeos (derivada).|
|durationInSeconds|Duración total de la lista de reproducción.|
|summarizedInsights|Contiene la sección [summarizedInsights](#summarizedinsights) con información detallada.
|videos|Lista de [vídeos](#videos) que forman la lista de reproducción.<br/>Si esta lista de reproducción está compuesta por intervalos de tiempo de otros vídeos (derivados), los vídeos de esta lista contienen los datos únicamente de los intervalos de tiempo incluidos.|

```json
{
  "accountId": "bca61527-1221-bca6-1527-a90000002000",
  "id": "abc3454321",
  "name": "My first video",
  "description": "I am trying VI",
  "userName": "Some name",
  "created": "2018/2/2 18:00:00.000",
  "privacyMode": "Private",
  "state": "Processed",
  "isOwned": true,
  "isEditable": false,
  "isBase": false,
  "durationInSeconds": 120, 
  "summarizedInsights" : { . . . }
  "videos": [{ . . . }]
}
```

## <a name="summarizedinsights"></a>summarizedInsights

En esta sección se muestra el resumen de la información detallada.

|Atributo | Descripción|
|---|---|
|name|Nombre del vídeo. Por ejemplo, Azure Monitor.|
|id|Identificador del vídeo. Por ejemplo, 63c6d532ff.|
|privacyMode|El desglose puede tener uno de los siguientes modos: **Privado**, **Público**. **Público**: el vídeo es visible para todos los usuarios de la cuenta y cualquiera que tenga un vínculo al vídeo. **Privado**: el vídeo es visible para todos los usuarios de la cuenta.|
|duration|Duración que describe el momento del tiempo en el que se ha producido la información. La duración se mide en segundos.|
|thumbnailVideoId|Identificador del vídeo del que se tomó la miniatura.
|thumbnailId|Identificador de la miniatura del vídeo. Para obtener la miniatura real, llame a [Get miniatura](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Video-Thumbnail) y pasarle thumbnailVideoId y thumbnailId.|
|faces|Puede contener cero o más rostros. Para más información, consulte [faces](#faces).|
|keywords|Puede contener cero o más palabras clave. Para más información, consulte [keywords](#keywords).|
|sentiments|Puede contener cero o más opiniones. Para más información, consulte [sentiments](#sentiments).|
|audioEffects| Puede contener cero o más efectos de audio. Para más información, consulte [audioEffects](#audioEffects).|
|labels| Puede contener cero o más etiquetas. Para más información, consulte [labels](#labels).|
|brands| Puede contener cero o más marcas. Para más información, consulte [brands](#brands).|
|statistics | Para más información, consulte [statistics](#statistics).|
|emotions| Puede contener cero o más emociones. Para más información, consulte [emotions](#emotions).|
|topics|Puede contener cero o más temas. Dimensión [topics](#topics).|

## <a name="videos"></a>vídeos

|Nombre|Descripción|
|---|---|
|accountId|Identificador de la cuenta de Video Indexer del vídeo.|
|id|Identificador del vídeo.|
|name|Nombre del vídeo.
|state|Estado del vídeo (cargado, en proceso, procesado, error, en cuarentena).|
|processingProgress|Progreso del procesamiento (por ejemplo, 20 %).|
|failureCode|Código de error si no se pudo procesar (por ejemplo, "UnsupportedFileType").|
|failureMessage|Mensaje de error si no se pudo procesar.|
|externalId|Identificador externo del vídeo (si lo especifica el usuario).|
|externalUrl|URL externa del vídeo (si la especifica el usuario).|
|metadata|Metadatos externos del vídeo (si los especifica el usuario).|
|isAdult|Indica si el vídeo se ha revisado manualmente y se ha identificado como un vídeo para adultos.|
|insights|Objeto de información detallada. Para más información, consulte [insights](#insights).|
|thumbnailId|Identificador de la miniatura del vídeo. Para obtener la llamada real en miniatura [Get miniatura](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Video-Thumbnail) y pásele el identificador y thumbnailId vídeo.|
|publishedUrl|Dirección URL para transmitir el vídeo.|
|publishedUrlProxy|Dirección URL desde la que se transmitirá el vídeo (para dispositivos de Apple).|
|viewToken|Un token de visualización de corta duración para transmitir el vídeo.|
|sourceLanguage|Idioma de origen del vídeo.|
|language|Idioma real del vídeo (traducción).|
|indexingPreset|Valor predeterminado utilizado para indexar el vídeo.|
|streamingPreset|Valor predeterminado utilizado para publicar el vídeo.|
|linguisticModelId|Modelo CRIS utilizado para transcribir el vídeo.|
|statistics | Para más información, consulte [statistics](#statistics).|

```json
{
    "videos": [{
        "accountId": "2cbbed36-1972-4506-9bc7-55367912df2d",
        "id": "142a356aa6",
        "state": "Processed",
        "privacyMode": "Private",
        "processingProgress": "100%",
        "failureCode": "General",
        "failureMessage": "",
        "externalId": null,
        "externalUrl": null,
        "metadata": null,
        "insights": {. . . },
        "thumbnailId": "89d7192c-1dab-4377-9872-473eac723845",
        "publishedUrl": "https://videvmediaservices.streaming.mediaservices.windows.net:443/d88a652d-334b-4a66-a294-3826402100cd/Xamarine.ism/manifest",
        "publishedProxyUrl": null,
        "viewToken": "Bearer=<token>",
        "sourceLanguage": "En-US",
        "language": "En-US",
        "indexingPreset": "Default",
        "linguisticModelId": "00000000-0000-0000-0000-000000000000"
    }],
}
```
### <a name="insights"></a>insights

La información detallada es un conjunto de dimensiones (por ejemplo, líneas de transcripción, rostros, marcas, etc.), en el que cada dimensión es una lista de elementos únicos (por ejemplo, face1, face2, face3) y cada elemento tiene sus propios metadatos y una lista de sus instancias (que son intervalos de tiempo con metadatos opcionales adicionales).

Un rostro podría tener un identificador, un nombre, una miniatura, otros metadatos y una lista de sus instancias temporales (por ejemplo: 00:00:05: 00:00:10, 00:01:00: 00:02:30 y 00:41:21: 00:41:49). Cada instancia temporal puede tener metadatos adicionales. Por ejemplo, las coordenadas del rectángulo del rostro (20,230,60,60).

|Versión|Versión del código|
|---|---|
|sourceLanguage|Idioma de origen del vídeo (suponiendo que hay un idioma principal). Con el formato de cadena [BCP-47](https://tools.ietf.org/html/bcp47).|
|language|Idioma de la información detallada (traducida del idioma de origen). Con el formato de cadena [BCP-47](https://tools.ietf.org/html/bcp47).|
|transcript|Dimensión [transcript](#transcript).|
|ocr|El [OCR](#ocr) dimensión.|
|palabras clave|Dimensión [keywords](#keywords).|
|blocks|Puede contener uno o más dimensiones [blocks](#blocks).|
|faces|Dimensión [faces](#faces).|
|labels|Dimensión [labels](#labels).|
|shots|Dimensión [shots](#shots).|
|brands|Dimensión [brands](#brands).|
|audioEffects|Dimension [audioEffects](#audioEffects).|
|sentiments|Dimension [sentiments](#sentiments).|
|visualContentModeration|Dimensión [visualContentModeration](#visualcontentmoderation).|
|textualContentModeration|Dimensión [textualConentModeration](#textualcontentmoderation).|
|emotions| Dimensión [emotions](#emotions).|
|topics|Dimensión [topics](#topics).|

Ejemplo:

```json
{
  "version": "0.9.0.0",
  "sourceLanguage": "en-US",
  "language": "es-ES",
  "transcript": ...,
  "ocr": ...,
  "keywords": ...,
  "faces": ...,
  "labels": ...,
  "shots": ...,
  "brands": ...,
  "audioEffects": ...,
  "sentiments": ...,
  "visualContentModeration": ...,
  "textualContentModeration": ...
}
```

#### <a name="blocks"></a>blocks

Atributo | Descripción
---|---
id|Identificador del bloque|
instances|Lista de intervalos de tiempo de este bloque.|

#### <a name="transcript"></a>transcript

|Nombre|Descripción|
|---|---|
|id|Identificador de la línea.|
|text|La transcripción en sí.|
|language|Idioma de la transcripción. Diseñado para admitir la transcripción, donde cada línea puede tener un idioma distinto.|
|instances|Lista de los intervalos de tiempo donde apareció esta línea. Si la instancia es una transcripción, solo tendrá 1 instancia.|

Ejemplo:

```json
"transcript": [
{
    "id": 0,
    "text": "Hi I'm Doug from office.",
    "language": "en-US",
    "instances": [
    {
        "start": "00:00:00.5100000",
        "end": "00:00:02.7200000"
    }
    ]
},
{
    "id": 1,
    "text": "I have a guest. It's Michelle.",
    "language": "en-US",
    "instances": [
    {
        "start": "00:00:02.7200000",
        "end": "00:00:03.9600000"
    }
    ]
}
] 
```

#### <a name="ocr"></a>ocr

|Nombre|Descripción|
|---|---|
|id|Identificador de la línea OCR.|
|text|Texto OCR.|
|confidence|Confiabilidad del reconocimiento.|
|language|Idioma de OCR.|
|instances|Lista de los intervalos de tiempo donde apareció este OCR (el mismo OCR puede aparecer varias veces).|
|height|El alto del rectángulo de OCR|
|top|La ubicación superior en px|
|izquierda| La ubicación de la izquierda en px|
|width|El ancho del rectángulo de OCR|

```json
"ocr": [
    {
      "id": 0,
      "text": "LIVE FROM NEW YORK",
      "confidence": 675.971,
      "height": 35,
      "language": "en-US",
      "left": 31,
      "top": 97,
      "width": 400,      
      "instances": [
        {
          "start": "00:00:26",
          "end": "00:00:52"
        }
      ]
    }
  ],
```

#### <a name="keywords"></a>palabras clave

|Nombre|Descripción|
|---|---|
|id|Identificador de la palabra clave.|
|text|Texto de la palabra clave.|
|confidence|Confiabilidad del reconocimiento de la palabra clave.|
|language|Idioma de la palabra clave (cuando se traduce).|
|instances|Lista de los intervalos de tiempo donde apareció esta palabra clave (una palabra clave puede aparecer varias veces).|

```json
{
    id: 0,
    text: "technology",
    confidence: 1,
    language: "en-US",
    instances: [{
            adjustedStart: "0:05:15.782",
            adjustedEnd: "0:05:16.249",
            start: "0:05:15.782",
            end: "0:05:16.249"
    },
    {
            adjustedStart: "0:04:54.761",
            adjustedEnd: "0:04:55.228",
            start: "0:04:54.761",
            end: "0:04:55.228"
    }]
}
```

#### <a name="faces"></a>faces

|Nombre|Descripción|
|---|---|
|id|Identificador del rostro.|
|Nombre|Nombre del rostro. Puede ser "Unknown #0", una celebridad identificada o una persona capacitada del cliente.|
|confidence|Confidencialidad de la identificación facial.|
|description|Descripción de la celebridad. |
|thumbnailId|Identificador de la miniatura del rostro en cuestión.|
|knownPersonId|Si es alguien conocido, su identificador interno.|
|referenceId|Si es una celebridad de Bing, su identificador de Bing.|
|referenceType|Actualmente, solo Bing.|
|título|Si es una celebridad, su cargo (por ejemplo, "CEO de Microsoft").|
|imageUrl|Si es una celebridad, la dirección URL de su imagen.|
|instances|Instancias de cuándo apareció el rostro en el intervalo de tiempo determinado. Cada instancia también tiene su thumbnailsId. |

```json
"faces": [{
    "id": 2002,
    "name": "Xam 007",
    "confidence": 0.93844,
    "description": null,
    "thumbnailId": "00000000-aee4-4be2-a4d5-d01817c07955",
    "knownPersonId": "8340004b-5cf5-4611-9cc4-3b13cca10634",
    "referenceId": null,
    "title": null,
    "imageUrl": null,
    "instances": [{
        "thumbnailsIds": ["00000000-9f68-4bb2-ab27-3b4d9f2d998e",
        "cef03f24-b0c7-4145-94d4-a84f81bb588c"],
        "adjustedStart": "00:00:07.2400000",
        "adjustedEnd": "00:00:45.6780000",
        "start": "00:00:07.2400000",
        "end": "00:00:45.6780000"
    },
    {
        "thumbnailsIds": ["00000000-51e5-4260-91a5-890fa05c68b0"],
        "adjustedStart": "00:10:23.9570000",
        "adjustedEnd": "00:10:39.2390000",
        "start": "00:10:23.9570000",
        "end": "00:10:39.2390000"
    }]
}]
```

#### <a name="labels"></a>labels

|Nombre|Descripción|
|---|---|
|id|Identificador de la etiqueta.|
|name|Nombre de la etiqueta (por ejemplo, "Equipo", "TV").|
|language|Idioma del nombre de la etiqueta (cuando se traduce). BCP-47|
|instances|Lista de los intervalos de tiempo donde apareció esta etiqueta (una etiqueta puede aparecer varias veces). Cada instancia tiene un campo de confiabilidad. |


```json
"labels": [
    {
      "id": 0,
      "name": "person",
      "language": "en-US",
      "instances": [
        {
          "confidence": 1.0,
          "start": "00: 00: 00.0000000",
          "end": "00: 00: 25.6000000"
        },
        {
          "confidence": 1.0,
          "start": "00: 01: 33.8670000",
          "end": "00: 01: 39.2000000"
        }
      ]
    },
    {
      "name": "indoor",
      "language": "en-US",
      "id": 1,
      "instances": [
        {
          "confidence": 1.0,
          "start": "00: 00: 06.4000000",
          "end": "00: 00: 07.4670000"
        },
        {
          "confidence": 1.0,
          "start": "00: 00: 09.6000000",
          "end": "00: 00: 10.6670000"
        },
        {
          "confidence": 1.0,
          "start": "00: 00: 11.7330000",
          "end": "00: 00: 20.2670000"
        },
        {
          "confidence": 1.0,
          "start": "00: 00: 21.3330000",
          "end": "00: 00: 25.6000000"
        }
      ]
    }
  ] 
```

#### <a name="scenes"></a>scenes

|NOMBRE|Descripción|
|---|---|
|id|Identificador de la escena.|
|instancias|Una lista de intervalos de tiempo de esta escena (una escena puede tener solo 1 instancia).|

```json
"scenes":[  
    {  
      "id":0,
      "instances":[  
          {  
            "start":"0:00:00",
            "end":"0:00:06.34",
            "duration":"0:00:06.34"
          }
      ]
    },
    {  
      "id":1,
      "instances":[  
          {  
            "start":"0:00:06.34",
            "end":"0:00:47.047",
            "duration":"0:00:40.707"
          }
      ]
    },

]
```

#### <a name="shots"></a>shots

|Nombre|Descripción|
|---|---|
|id|Identificador de la toma.|
|keyFrames|Una lista de fotogramas clave dentro de la captura (cada uno tiene un identificador y una lista de intervalos de tiempo de las instancias). Cada instancia de fotograma clave tiene un campo thumbnailId, que contiene la miniatura del fotograma clave identificador.|
|instancias|Una lista de intervalos de tiempo de esta toma (una captura puede tener solo 1 instancia).|

```json
"shots":[  
    {  
      "id":0,
      "keyFrames":[  
          {  
            "id":0,
            "instances":[  
                {  
                  "thumbnailId":"00000000-0000-0000-0000-000000000000",
                  "start":"0:00:00.209",
                  "end":"0:00:00.251",
                  "duration":"0:00:00.042"
                }
            ]
          },
          {  
            "id":1,
            "instances":[  
                {  
                  "thumbnailId":"00000000-0000-0000-0000-000000000000",
                  "start":"0:00:04.755",
                  "end":"0:00:04.797",
                  "duration":"0:00:00.042"
                }
            ]
          }
      ],
      "instances":[  
          {  
            "start":"0:00:00",
            "end":"0:00:06.34",
            "duration":"0:00:06.34"
          }
      ]
    },

]
```

#### <a name="brands"></a>brands

Nombres de empresas y marcas de productos detectados en la transcripción de voz a texto o de OCR de vídeo. No incluye el reconocimiento visual de marcas ni la detección de logotipos.

|Nombre|Descripción|
|---|---|
|id|Identificador de marca.|
|name|Nombre de la marca.|
|referenceId | Sufijo de la dirección URL de la wikipedia de la marca. Por ejemplo, "Target_Corporation" es el sufijo de [https://en.wikipedia.org/wiki/Target_Corporation](https://en.wikipedia.org/wiki/Target_Corporation).
|referenceUrl | Dirección url de la wikipedia de la marca, si existe. Por ejemplo, [https://en.wikipedia.org/wiki/Target_Corporation](https://en.wikipedia.org/wiki/Target_Corporation).
|description|Descripción de la marca.|
|etiquetas|Lista de etiquetas predefinidas asociadas a esta marca.|
|confidence|Valor de la confianza del detector de marcas de Video Indexer (0-1).|
|instances|Lista de intervalos de tiempo de esta marca. Cada instancia tiene un atributo brandType, que indica si esta marca apareció en la transcripción o en el reconocimiento óptico de caracteres.|

```json
"brands": [
{
    "id": 0,
    "name": "MicrosoftExcel",
    "referenceId": "Microsoft_Excel",
    "referenceUrl": "http: //en.wikipedia.org/wiki/Microsoft_Excel",
    "referenceType": "Wiki",
    "description": "Microsoft Excel is a sprea..",
    "tags": [],
    "confidence": 0.975,
    "instances": [
    {
        "brandType": "Transcript",
        "start": "00: 00: 31.3000000",
        "end": "00: 00: 39.0600000"
    }
    ]
},
{
    "id": 1,
    "name": "Microsoft",
    "referenceId": "Microsoft",
    "referenceUrl": "http: //en.wikipedia.org/wiki/Microsoft",
    "description": "Microsoft Corporation is...",
    "tags": [
    "competitors",
    "technology"
    ],
    "confidence": 1.0,
    "instances": [
    {
        "brandType": "Transcript",
        "start": "00: 01: 44",
        "end": "00: 01: 45.3670000"
    },
    {
        "brandType": "Ocr",
        "start": "00: 01: 54",
        "end": "00: 02: 45.3670000"
    }
    ]
}
]
```

#### <a name="statistics"></a>statistics

|Nombre|Descripción|
|---|---|
|CorrespondenceCount|Número de correspondencias en el vídeo.|
|SpeakerWordCount|Número de palabras por orador.|
|SpeakerNumberOfFragments|Cantidad de fragmentos que el orador tiene en un vídeo.|
|SpeakerLongestMonolog|Monólogo más largo del orador. Si el orador tiene períodos de silencio dentro del monólogo, se incluyen. Los silencios al principio y al final del monólogo se eliminan.| 
|SpeakerTalkToListenRatio|El cálculo se basa en el tiempo invertido en el monólogo del orador (sin los silencios intermedios) dividido por el tiempo total del vídeo. El tiempo se redondea a tres decimales.|

#### <a name="a-idaudioeffectsaudioeffects"></a><a id="audioEffects"/>audioEffects

|NOMBRE|Descripción|
|---|---|
|id|Identificador del efecto de audio.|
|Tipo|Tipo de efecto de audio (por ejemplo, aplausos, voz, silencio).|
|instances|Lista de los intervalos de tiempo donde apareció este efecto de audio.|

```json
"audioEffects": [
{
    "id": 0,
    "type": "Clapping",
    "instances": [
    {
        "start": "00:00:00",
        "end": "00:00:03"
    },
    {
        "start": "00:01:13",
        "end": "00:01:21"
    }
    ]
}
]
```

#### <a name="sentiments"></a>sentiments

Las opiniones se agregan según su campo sentimentType (Positive/Neutral/Negative). Por ejemplo, 0-0,1, 0,1-0,2.

|Nombre|Descripción|
|---|---|
|id|Identificador de la opinión.|
|averageScore |Promedio de todas las puntuaciones de todas las instancias de ese tipo de opinión: Positive/Neutral/Negative.|
|instances|Lista de los intervalos de tiempo donde apareció esta opinión.|
|sentimentType |El tipo puede ser "Positive", "Neutral" o "Negative".|

```json
"sentiments": [
{
    "id": 0,
    "averageScore": 0.87,
    "sentimentType": "Positive",
    "instances": [
    {
        "start": "00:00:23",
        "end": "00:00:41"
    }
    ]
}, {
    "id": 1,
    "averageScore": 0.11,
    "sentimentType": "Positive",
    "instances": [
    {
        "start": "00:00:13",
        "end": "00:00:21"
    }
    ]
}
]
```

#### <a name="visualcontentmoderation"></a>visualContentModeration

El bloque visualContentModeration contiene intervalos de tiempo en los que Video Indexer ha detectado que podría haber contenido para adultos. Si visualContentModeration está vacío, no se ha identificado ningún contenido para adultos.

Los vídeos en los que se encuentre contenido para adultos o subido de tono podrían estar disponibles solo para visualización privada. Los usuarios tienen la opción de enviar una solicitud de revisión humana del contenido, en cuyo caso el atributo IsAdult contendrá el resultado de la revisión humana.

|Nombre|Descripción|
|---|---|
|id|Identificador de la moderación del contenido visual.|
|adultScore|Puntuación de contenido para adultos (del moderador de contenido).|
|racyScore|Puntuación de contenido subido de tono (del moderador de contenido).|
|instances|Lista de intervalos de tiempo donde apareció esta moderación de contenido visual.|

```json
"VisualContentModeration": [
{
    "id": 0,
    "adultScore": 0.00069,
    "racyScore": 0.91129,
    "instances": [
    {
        "start": "00:00:25.4840000",
        "end": "00:00:25.5260000"
    }
    ]
},
{
    "id": 1,
    "adultScore": 0.99231,
    "racyScore": 0.99912,
    "instances": [
    {
        "start": "00:00:35.5360000",
        "end": "00:00:35.5780000"
    }
    ]
}
] 
```

#### <a name="textualcontentmoderation"></a>textualContentModeration 

|NOMBRE|Descripción|
|---|---|
|id|Identificador de la moderación de contenido textual.|
|bannedWordsCount |Número de palabras no permitidas.|
|bannedWordsRatio |Proporción respecto al número total de palabras.|

#### <a name="emotions"></a>emotions

Video Indexer identifica emociones en función de las indicaciones para voz y audio. La emoción identificada podría ser: felicidad, tristeza, ira o miedo.

|NOMBRE|Descripción|
|---|---|
|id|Identificador de la emoción.|
|Tipo|Momento de la emoción que se identificó en función de las indicaciones para voz y audio. La emoción podría ser: felicidad, tristeza, ira o miedo.|
|instancias|Lista de los intervalos de tiempo donde apareció esta emoción.|

```json
"emotions": [{
    "id": 0,
    "type": "Fear",
    "instances": [{
      "adjustedStart": "0:00:39.47",
      "adjustedEnd": "0:00:45.56",
      "start": "0:00:39.47",
      "end": "0:00:45.56"
    },
    {
      "adjustedStart": "0:07:19.57",
      "adjustedEnd": "0:07:23.25",
      "start": "0:07:19.57",
      "end": "0:07:23.25"
    }]
  },
  {
    "id": 1,
    "type": "Anger",
    "instances": [{
      "adjustedStart": "0:03:55.99",
      "adjustedEnd": "0:04:05.06",
      "start": "0:03:55.99",
      "end": "0:04:05.06"
    },
    {
      "adjustedStart": "0:04:56.5",
      "adjustedEnd": "0:05:04.35",
      "start": "0:04:56.5",
      "end": "0:05:04.35"
    }]
  },
  {
    "id": 2,
    "type": "Joy",
    "instances": [{
      "adjustedStart": "0:12:23.68",
      "adjustedEnd": "0:12:34.76",
      "start": "0:12:23.68",
      "end": "0:12:34.76"
    },
    {
      "adjustedStart": "0:12:46.73",
      "adjustedEnd": "0:12:52.8",
      "start": "0:12:46.73",
      "end": "0:12:52.8"
    },
    {
      "adjustedStart": "0:30:11.29",
      "adjustedEnd": "0:30:16.43",
      "start": "0:30:11.29",
      "end": "0:30:16.43"
    },
    {
      "adjustedStart": "0:41:37.23",
      "adjustedEnd": "0:41:39.85",
      "start": "0:41:37.23",
      "end": "0:41:39.85"
    }]
  },
  {
    "id": 3,
    "type": "Sad",
    "instances": [{
      "adjustedStart": "0:13:38.67",
      "adjustedEnd": "0:13:41.3",
      "start": "0:13:38.67",
      "end": "0:13:41.3"
    },
    {
      "adjustedStart": "0:28:08.88",
      "adjustedEnd": "0:28:18.16",
      "start": "0:28:08.88",
      "end": "0:28:18.16"
    }]
  }
],
```

#### <a name="topics"></a>topics

Video Indexer saca conclusiones de los temas principales a partir de las transcripciones. Cuando es posible, se incluye la taxonomía [IPTC](https://iptc.org/standards/media-topics/) de primer nivel. 

|NOMBRE|Descripción|
|---|---|
|id|Identificador del tema.|
|Nombre|El nombre del tema, por ejemplo: "productos farmacéuticos".|
|referenceId|Rutas de navegación que reflejan la jerarquía de temas. Por ejemplo: "Salud y bienestar / Medicina y salud / Productos farmacéuticos".|
|confidence|Puntuación de confianza en el intervalo [0,1]. Cuanto mayor es, más segura es.|
|language|Idioma que se usa en el tema.|
|iptcName|Nombre del código multimedia IPTC, si se detecta.|
|instancias |Actualmente, Video Indexer no indexa ningún tema en intervalos de tiempo, por lo que se usa el vídeo completo como intervalo.|

```json
"topics": [{
    "id": 0,
    "name": "INTERNATIONAL RELATIONS",
    "referenceId": "POLITICS AND GOVERNMENT/FOREIGN POLICY/INTERNATIONAL RELATIONS",
    "referenceType": "VideoIndexer",
    "confidence": 1,
    "language": "en-US",
    "instances": [{
        "adjustedStart": "0:00:00",
        "adjustedEnd": "0:03:36.25",
        "start": "0:00:00",
        "end": "0:03:36.25"
    }]
}, {
    "id": 1,
    "name": "Politics and Government",
    "referenceType": "VideoIndexer",
    "iptcName": "Politics",
    "confidence": 0.9041,
    "language": "en-US",
    "instances": [{
        "adjustedStart": "0:00:00",
        "adjustedEnd": "0:03:36.25",
        "start": "0:00:00",
        "end": "0:03:36.25"
    }]
}]
. . .
```

## <a name="next-steps"></a>Pasos siguientes

[Portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai)

Para más información acerca de cómo insertar widgets en su aplicación, consulte [Embed Video Indexer widgets into your applications](video-indexer-embed-widgets.md) (Inserción de widgets de Video Indexer en sus aplicaciones). 

