---
title: 'Inicio rápido: Envío de una solicitud de búsqueda a la API REST mediante Node.js: Bing Entity Search'
titleSuffix: Azure Cognitive Services
description: Use esta guía de inicio rápido para enviar una solicitud a la API de REST Bing Entity Search mediante C# y reciba una respuesta JSON.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: quickstart
ms.date: 05/08/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: 82bdd8f3890f1685aa442463287fe72bde08d518
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87405966"
---
# <a name="quickstart-send-a-search-request-to-the-bing-entity-search-rest-api-using-nodejs"></a>Inicio rápido: Envío de una solicitud de búsqueda a Bing Entity Search REST API con Node.js

Use este inicio rápido para realizar la primera llamada a Bing Entity Search API y ver la respuesta JSON. Esta sencilla aplicación de JavaScript envía una consulta de búsqueda de noticias a la API y muestra la respuesta. El código fuente del ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/nodejs/Search/BingEntitySearchv7.js).

Aunque esta aplicación está escrita en JavaScript, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

## <a name="prerequisites"></a>Prerrequisitos

* La última versión de [Node.js](https://nodejs.org/en/download/).

* La [biblioteca de solicitudes de JavaScript](https://github.com/request/request).

[!INCLUDE [cognitive-services-bing-news-search-signup-requirements](../../../../includes/cognitive-services-bing-entity-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

1. Cree un archivo JavaScript en el editor o el IDE que prefiera y establezca la rigurosidad y los requisitos HTTPS.

    ```javaScript
    'use strict';
    let https = require ('https');
    ```

2. Cree variables para el punto de conexión de la API, la clave de suscripción y la consulta de búsqueda. Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```javascript
    let subscriptionKey = 'ENTER YOUR KEY HERE';
    let host = 'api.cognitive.microsoft.com';
    let path = '/bing/v7.0/entities';
    
    let mkt = 'en-US';
    let q = 'italian restaurant near me';
    ```

3. Anexe los parámetros market y query a una cadena denominada `query`. Asegúrese de codificar como URL la consulta con `encodeURI()`.
    ```javascript 
    let query = '?mkt=' + mkt + '&q=' + encodeURI(q);
    ```

## <a name="handle-and-parse-the-response"></a>Control y análisis de la respuesta

1. Defina una función denominada `response_handler()` que toma una llamada HTTP, `response`, como un parámetro. 

2. Dentro de esta función defina una variable para que contenga el cuerpo de la respuesta JSON.  
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

4. Cuando se señale una marca `end`, analice la respuesta JSON e imprímala.

    ```javascript
    response.on ('end', function () {
    let json = JSON.stringify(JSON.parse(body), null, '  ');
    console.log (json);
    });
        ```

## Send a request

1. Create a function called `Search()` to send a search request. In it, perform the following steps:

2. Within this function, create a JSON object containing your request parameters. Use `Get` for the method, and add your host and path information. Add your subscription key to the `Ocp-Apim-Subscription-Key` header. 

3. Use `https.request()` to send the request with the response handler created previously, and your search parameters.
    
   ```javascript
   let Search = function () {
    let request_params = {
        method : 'GET',
        hostname : host,
        path : path + query,
        headers : {
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };
    
    let req = https.request (request_params, response_handler);
    req.end ();
   }
      ```

2. Llame a la función `Search()`.

## <a name="example-json-response"></a>Ejemplo de respuesta JSON

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "_type": "SearchResponse",
  "queryContext": {
    "originalQuery": "italian restaurant near me",
    "askUserForLocation": true
  },
  "places": {
    "value": [
      {
        "_type": "LocalBusiness",
        "webSearchUrl": "https://www.bing.com/search?q=sinful+bakery&filters=local...",
        "name": "Liberty's Delightful Sinful Bakery & Cafe",
        "url": "https://www.contoso.com/",
        "entityPresentationInfo": {
          "entityScenario": "ListItem",
          "entityTypeHints": [
            "Place",
            "LocalBusiness"
          ]
        },
        "address": {
          "addressLocality": "Seattle",
          "addressRegion": "WA",
          "postalCode": "98112",
          "addressCountry": "US",
          "neighborhood": "Madison Park"
        },
        "telephone": "(800) 555-1212"
      },

      . . .
      {
        "_type": "Restaurant",
        "webSearchUrl": "https://www.bing.com/search?q=Pickles+and+Preserves...",
        "name": "Munson's Pickles and Preserves Farm",
        "url": "https://www.princi.com/",
        "entityPresentationInfo": {
          "entityScenario": "ListItem",
          "entityTypeHints": [
            "Place",
            "LocalBusiness",
            "Restaurant"
          ]
        },
        "address": {
          "addressLocality": "Seattle",
          "addressRegion": "WA",
          "postalCode": "98101",
          "addressCountry": "US",
          "neighborhood": "Capitol Hill"
        },
        "telephone": "(800) 555-1212"
      },
      
      . . .
    ]
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Compilar una aplicación web de una sola página](../tutorial-bing-entities-search-single-page-app.md)

* [¿Qué es Bing Entity Search API?](../overview.md )
* [Referencia de la API Bing Entity Search](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference).
