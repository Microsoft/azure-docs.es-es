---
title: 'Inicio rápido: Búsqueda de vídeos con la API REST y Node.js: Bing Video Search'
titleSuffix: Azure Cognitive Services
description: Use este artículo de inicio rápido para enviar solicitudes de búsqueda de vídeo a Bing Video Search API mediante JavaScript.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-video-search
ms.topic: quickstart
ms.date: 05/22/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: 9658d488fac187e63d04ca435c9233629407ac07
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87408108"
---
# <a name="quickstart-search-for-videos-using-the-bing-video-search-rest-api-and-nodejs"></a>Inicio rápido: Búsqueda de vídeos con Bing Video Search API y Node.js

Use este inicio rápido para realizar la primera llamada a la API Bing Video Search. Esta sencilla aplicación de JavaScript envía una consulta de búsqueda de vídeo HTTP a la API y muestra la respuesta JSON. Aunque esta aplicación está escrita en JavaScript y usa Node.js, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. 

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/nodejs/Search/BingVideoSearchv7.js) con anotaciones de código y control de errores adicionales.

## <a name="prerequisites"></a>Prerrequisitos

* [Node.js](https://nodejs.org/en/download/).

* El módulo de solicitud para JavaScript. Instale este módulo mediante `npm install request`.

[!INCLUDE [cognitive-services-bing-video-search-signup-requirements](../../../../includes/cognitive-services-bing-video-search-signup-requirements.md)]

## <a name="initialize-the-application"></a>Inicialización de la aplicación

1. Cree un archivo de JavaScript en su IDE o editor favorito. Establezca la rigurosidad y agregue el siguiente requisito:

    ```javascript
    'use strict';
    let https = require('https');
    ```

2. Cree variables para el punto de conexión de API, la clave de suscripción y el término de búsqueda. Para el valor `host` puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```javascript
    let subscriptionKey = 'enter key here';
    let host = 'api.cognitive.microsoft.com';
    let path = '/bing/v7.0/videos/search';
    let term = 'kittens';
    ```

## <a name="create-a-response-handler"></a>Creación de un controlador de respuesta

1. Cree una función denominada `response_handler` para recibir una respuesta de la API. Cree una variable para el cuerpo de la respuesta. Anexe la respuesta cuando se reciba una marca `data`, mediante `response.on()`.

    ```javascript
    let response_handler = function (response) {
        let body = '';
        response.on('data', function (d) {
            body += d;
        });
    };
    ```
    
1. En esta función, utilice `response.on()` cuando `end` se señaliza para almacenar los encabezados relacionados con Bing (empezando por `bingapis` o `x-msedge-`). Analice el JSON mediante `JSON.parse()`, conviértalo en una cadena con `JSON.stringify()` e imprímalo.

    ```javascript
    response.on('end', function () {
        for (var header in response.headers)
            // header keys are lower-cased by Node.js
            if (header.startsWith("bingapis-") || header.startsWith("x-msedge-"))
                 console.log(header + ": " + response.headers[header]);
        body = JSON.stringify(JSON.parse(body), null, '  ');
        //JSON Response body
        console.log(body);
    });
    ```

## <a name="create-and-send-the-search-request"></a>Creación y envío de la solicitud de búsqueda

Cree una función denominada `bing_video_search()`. Agregue los parámetros de la solicitud, incluidos el nombre de host y los encabezados. Codifique el término de búsqueda y anéxelo al parámetro de ruta de acceso con el parámetro `?q=`. Después, envíe la solicitud con `req.end()`.

```javascript
let bing_video_search = function (search_term) {
  console.log('Searching videos for: ' + term);
let request_params = {
    method : 'GET',
    hostname : host,
    path : path + '?q=' + encodeURIComponent(search_term),
    headers : {
        'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };
    let req = https.request(request_params,
      response_handler);
    req.end();
}
```

## <a name="json-response"></a>Respuesta JSON

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
    "_type": "Videos",
    "instrumentation": {},
    "readLink": "https://api.cognitive.microsoft.com/api/v7/videos/search?q=kittens",
    "webSearchUrl": "https://www.bing.com/videos/search?q=kittens",
    "totalEstimatedMatches": 1000,
    "value": [
        {
            "webSearchUrl": "https://www.bing.com/videos/search?q=kittens&view=...",
            "name": "Top 10 cute kitten videos compilation",
            "description": "HELP HOMELESS ANIMALS AND WIN A PRIZE BY CHOOSING...",
            "thumbnailUrl": "https://tse4.mm.bing.net/th?id=OVP.n1aE_Oikl4MtzBb...",
            "datePublished": "2014-11-12T22:47:36.0000000",
            "publisher": [
                {
                    "name": "Fabrikam"
                }
            ],
            "creator": {
                "name": "Marcus Appel"
            },
            "isAccessibleForFree": true,
            "contentUrl": "https://www.fabrikam.com/watch?v=8HVWitAW-Qg",
            "hostPageUrl": "https://www.fabrikam.com/watch?v=8HVWitAW-Qg",
            "encodingFormat": "h264",
            "hostPageDisplayUrl": "https://www.fabrikam.com/watch?v=8HVWitAW-Qg",
            "width": 480,
            "height": 360,
            "duration": "PT3M52S",
            "motionThumbnailUrl": "https://tse4.mm.bing.net/th?id=OM.j4QyJAENJphdZQ_1501386166&pid=Api",
            "embedHtml": "<iframe width=\"1280\" height=\"720\" src=\"https://www.fabrikam.com/embed/8HVWitAW-Qg?autoplay=1\" frameborder=\"0\" allowfullscreen></iframe>",
            "allowHttpsEmbed": true,
            "viewCount": 7513633,
            "thumbnail": {
                "width": 300,
                "height": 168
            },
            "videoId": "655D98260D012432848F6558260D012432848F",
            "allowMobileEmbed": true,
            "isSuperfresh": false
        },
        . . .
    ],
    "nextOffset": 36,
    "queryExpansions": [
        {
            "text": "Kittens Meowing",
            "displayText": "Meowing",
            "webSearchUrl": "https://www.bing.com/videos/search?q=Kittens+Meowing...",
            "searchLink": "https://api.cognitive.microsoft.com/api/v7/videos/search...",
            "thumbnail": {
                "thumbnailUrl": "https://tse3.mm.bing.net/th?q=Kittens+Meowing&pid..."
            }
        },
        {
            "text": "Funny Kittens",
            "displayText": "Funny",
            "webSearchUrl": "https://www.bing.com/videos/search?q=Funny+Kittens...",
            "searchLink": "https://api.cognitive.microsoft.com/api/v7/videos/search...",
            "thumbnail": {
                "thumbnailUrl": "https://tse3.mm.bing.net/th?q=Funny+Kittens&..."
            }
        },
        . . .
    ],
    "pivotSuggestions": [
        {
            "pivot": "kittens",
            "suggestions": [
                {
                    "text": "Cat",
                    "displayText": "Cat",
                    "webSearchUrl": "https://www.bing.com/videos/search?q=Cat...",
                    "searchLink": "https://api.cognitive.microsoft.com/api/v7/videos/search?...",
                    "thumbnail": {
                        "thumbnailUrl": "https://tse3.mm.bing.net/th?q=Cat&pid=Api..."
                    }
                },
                {
                    "text": "Feral Cat",
                    "displayText": "Feral Cat",
                    "webSearchUrl": "https://www.bing.com/videos/search?q=Feral+Cat...",
                    "searchLink": "https://api.cognitive.microsoft.com/api/v7/videos/search...",
                    "thumbnail": {
                        "thumbnailUrl": "https://tse3.mm.bing.net/th?q=Feral+Cat&pid=Api&..."
                    }
                }
            ]
        }
    ],
    "relatedSearches": [
        {
            "text": "Kittens Being Born",
            "displayText": "Kittens Being Born",
            "webSearchUrl": "https://www.bing.com/videos/search?q=Kittens+Being+Born...",
            "searchLink": "https://api.cognitive.microsoft.com/api/v7/videos/search?...",
            "thumbnail": {
                "thumbnailUrl": "https://tse1.mm.bing.net/th?q=Kittens+Being+Born&pid=..."
            }
        },
        . . .
    ]
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Compilar una aplicación web de una sola página](../tutorial-bing-video-search-single-page-app.md)

## <a name="see-also"></a>Consulte también 

 [¿Qué es Bing Video Search API?](../overview.md)
