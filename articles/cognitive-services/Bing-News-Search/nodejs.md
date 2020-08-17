---
title: 'Inicio rápido: Realizar una búsqueda de noticias con Node.js: API de REST de Bing News Search'
titleSuffix: Azure Cognitive Services
description: Use esta guía de inicio rápido para enviar una solicitud a la API de REST Bing News Search mediante Node.js y reciba una respuesta JSON.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: quickstart
ms.date: 05/22/2020
ms.author: aahi
ms.custom: seodec2018, devx-track-javascript
ms.openlocfilehash: d0ed69aecf03e5c8e3a65e3ac532c02c89b65ba8
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406612"
---
# <a name="quickstart-perform-a-news-search-using-nodejs-and-the-bing-news-search-rest-api"></a>Inicio rápido: Realizar una búsqueda de noticias mediante Node.js y la API de REST Bing News Search

Use este inicio rápido para realizar la primera llamada a la API Bing News Search. Esta sencilla aplicación de JavaScript envía una consulta de búsqueda a la API y muestra la respuesta JSON.

Aunque esta aplicación está escrita en JavaScript y se ejecuta en Node.js, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

El código fuente del ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/nodejs/Search/BingNewsSearchv7.js).

## <a name="prerequisites"></a>Requisitos previos

* La última versión de [Node.js](https://nodejs.org/en/download/).
* La [biblioteca de solicitudes de JavaScript](https://github.com/request/request).

[!INCLUDE [cognitive-services-bing-news-search-signup-requirements](../../../includes/cognitive-services-bing-news-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

1. Cree un archivo JavaScript en el editor o el IDE que prefiera y establezca la rigurosidad y los requisitos HTTPS.

    ```javascript
    'use strict';
    let https = require('https');
    ```

2. Cree variables para el punto de conexión de la API, la ruta de acceso de búsqueda de API de noticias, la clave de suscripción y el término de búsqueda. Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso. 

    ```javascript
    let subscriptionKey = 'enter key here';
    let host = 'api.cognitive.microsoft.com';
    let path = '/bing/v7.0/news/search';
    let term = 'Microsoft';
    ```

## <a name="handle-and-parse-the-response"></a>Control y análisis de la respuesta

1. Defina una función denominada `response_handler` que toma una llamada HTTP, `response`, como un parámetro. 

   Agregue código a esta función en los pasos siguientes.

2. Defina una variable para que contenga el cuerpo de la respuesta JSON.  

    ```javascript
    let response_handler = function (response) {
        let body = '';
    };
    ```

3. Almacene el cuerpo de la respuesta cuando se llame a la marca `data`.

    ```javascript
    response.on('data', function (d) {
        body += d;
    });
    ```

3. Cuando se señala una marca `end`, se puede ver el JSON y los encabezados.

    ```javascript
    response.on('end', function () {
        console.log('\nRelevant Headers:\n');
        for (var header in response.headers)
            // header keys are lower-cased by Node.js
            if (header.startsWith("bingapis-") || header.startsWith("x-msedge-"))
                 console.log(header + ": " + response.headers[header]);
        body = JSON.stringify(JSON.parse(body), null, '  ');
        console.log('\nJSON Response:\n');
        console.log(body);
     });
    ```

## <a name="example-json-response"></a>Ejemplo de respuesta JSON

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
   "_type": "News",
   "readLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/news\/search?q=Microsoft",
   "totalEstimatedMatches": 36,
   "sort": [
      {
         "name": "Best match",
         "id": "relevance",
         "isSelected": true,
         "url": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/news\/search?q=Microsoft"
      },
      {
         "name": "Most recent",
         "id": "date",
         "isSelected": false,
         "url": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/news\/search?q=Microsoft&sortby=date"
      }
   ],
   "value": [
      {
         "name": "Microsoft to open flagship London brick-and-mortar retail store",
         "url": "http:\/\/www.contoso.com\/article\/microsoft-to-open-flagshi...",
         "image": {
            "thumbnail": {
               "contentUrl": "https:\/\/www.bing.com\/th?id=ON.F9E4A49EC010417...",
               "width": 220,
               "height": 146
            }
         },
         "description": "After years of rumors about Microsoft opening a brick-and-mortar...", 
         "about": [
           {
             "readLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/entiti...", 
             "name": "Microsoft"
           }, 
           {
             "readLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/entit...", 
             "name": "London"
           }
         ], 
         "provider": [
           {
             "_type": "Organization", 
             "name": "Contoso"
           }
         ], 
          "datePublished": "2017-09-21T21:16:00.0000000Z", 
          "category": "ScienceAndTechnology"
      }, 

      . . .
      
      {
         "name": "Microsoft adds Availability Zones to its Azure cloud platform",
         "url": "https:\/\/contoso.com\/2017\/09\/21\/microsoft-adds-availability...",
         "image": {
            "thumbnail": {
               "contentUrl": "https:\/\/www.bing.com\/th?id=ON.0AE7595B9720...",
               "width": 700,
               "height": 466
            }
         },
         "description": "Microsoft has begun adding Availability Zones to its...",
         "about": [
            {
               "readLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/entities\/a093e9b...",
               "name": "Microsoft"
            },
            {
               "readLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/entities\/cf3abf7d-e379-...",
               "name": "Windows Azure"
            },
            {
               "readLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/entities\/9cdd061c-1fae-d0...",
               "name": "Cloud"
            }
         ],
         "provider": [
            {
               "_type": "Organization",
               "name": "Contoso"
            }
         ],
         "datePublished": "2017-09-21T09:01:00.0000000Z",
         "category": "ScienceAndTechnology"
      }
   ]
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación web de una sola página](tutorial-bing-news-search-single-page-app.md)
