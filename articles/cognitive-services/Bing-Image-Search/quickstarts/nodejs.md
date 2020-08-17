---
title: 'Inicio rápido: Búsqueda de imágenes con la API de REST Bing Image Search y Node.js'
titleSuffix: Azure Cognitive Services
description: Use esta guía de inicio rápido para enviar solicitudes de búsqueda de imágenes a la API de REST Bing Image Search mediante JavaScript y respuestas JSON.
services: cognitive-services
documentationcenter: ''
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: quickstart
ms.date: 05/08/2020
ms.author: aahi
ms.custom: seodec2018, devx-track-javascript
ms.openlocfilehash: f9737eb42552ef102a9a970c5d5ee28a781a1fea
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406119"
---
# <a name="quickstart-search-for-images-using-the-bing-image-search-rest-api-and-nodejs"></a>Inicio rápido: Búsqueda de imágenes con la API de REST Bing Image Search y Node.js

Utilice este inicio rápido para aprender cómo enviar solicitudes de búsqueda a la API Bing Image Search. Esta aplicación de JavaScript envía una consulta de búsqueda a la API y muestra la dirección URL de la primera imagen de los resultados. Aunque esta aplicación está escrita en JavaScript, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/nodejs/Search/BingImageSearchv7Quickstart.js) con anotaciones y control de errores adicionales.

## <a name="prerequisites"></a>Requisitos previos

* La última versión de [Node.js](https://nodejs.org/en/download/).

* La [biblioteca de solicitudes de JavaScript](https://github.com/request/request).

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

Para más información, consulte [Precios de Cognitive Services: API Bing Search](https://azure.microsoft.com/pricing/details/cognitive-services/search-api/).

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

1. Cree un archivo JavaScript en el editor o el IDE que prefiera y establezca la rigurosidad y los requisitos HTTPS.

    ```javascript
    'use strict';
    let https = require('https');
    ```

2. Cree variables para el punto de conexión de la API, la ruta de acceso de búsqueda de API de la imagen, la clave de suscripción y el término de búsqueda. Para `host` puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```javascript
    let subscriptionKey = 'enter key here';
    let host = 'api.cognitive.microsoft.com';
    let path = '/bing/v7.0/images/search';
    let term = 'tropical ocean';
    ```

## <a name="construct-the-search-request-and-query"></a>Construya la consulta y la solicitud de búsqueda.

1. Utilice las variables del último paso para dar formato a una dirección URL de búsqueda para la solicitud de API. Codifique como URL el término de búsqueda antes de enviarlo a la API.

    ```javascript
    let request_params = {
        method : 'GET',
        hostname : host,
        path : path + '?q=' + encodeURIComponent(search),
        headers : {
        'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };
    ```

2. Utilice la biblioteca de solicitudes para enviar la consulta a la API. 
    ```javascript
    let req = https.request(request_params, response_handler);
    req.end();
    ```

## <a name="handle-and-parse-the-response"></a>Control y análisis de la respuesta

1. Defina una función denominada `response_handler` que toma una llamada HTTP, `response`, como un parámetro. 

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

4. Cuando se señala una marca `end`, obtenga el primer resultado de la respuesta JSON. Imprima la dirección URL de la primera imagen, junto con el número total de imágenes devueltas.

    ```javascript
    response.on('end', function () {
        let firstImageResult = imageResults.value[0];
        console.log(`Image result count: ${imageResults.value.length}`);
        console.log(`First image thumbnail url: ${firstImageResult.thumbnailUrl}`);
        console.log(`First image web search url: ${firstImageResult.webSearchUrl}`);
     });
    ```

## <a name="example-json-response"></a>Ejemplo de respuesta JSON

Las respuestas de Bing Image Search API se devuelven como JSON. Esta respuesta de ejemplo se ha truncado para mostrar un único resultado.

```json
{
"_type":"Images",
"instrumentation":{
    "_type":"ResponseInstrumentation"
},
"readLink":"images\/search?q=tropical ocean",
"webSearchUrl":"https:\/\/www.bing.com\/images\/search?q=tropical ocean&FORM=OIIARP",
"totalEstimatedMatches":842,
"nextOffset":47,
"value":[
    {
        "webSearchUrl":"https:\/\/www.bing.com\/images\/search?view=detailv2&FORM=OIIRPO&q=tropical+ocean&id=8607ACDACB243BDEA7E1EF78127DA931E680E3A5&simid=608027248313960152",
        "name":"My Life in the Ocean | The greatest WordPress.com site in ...",
        "thumbnailUrl":"https:\/\/tse3.mm.bing.net\/th?id=OIP.fmwSKKmKpmZtJiBDps1kLAHaEo&pid=Api",
        "datePublished":"2017-11-03T08:51:00.0000000Z",
        "contentUrl":"https:\/\/mylifeintheocean.files.wordpress.com\/2012\/11\/tropical-ocean-wallpaper-1920x12003.jpg",
        "hostPageUrl":"https:\/\/mylifeintheocean.wordpress.com\/",
        "contentSize":"897388 B",
        "encodingFormat":"jpeg",
        "hostPageDisplayUrl":"https:\/\/mylifeintheocean.wordpress.com",
        "width":1920,
        "height":1200,
        "thumbnail":{
        "width":474,
        "height":296
        },
        "imageInsightsToken":"ccid_fmwSKKmK*mid_8607ACDACB243BDEA7E1EF78127DA931E680E3A5*simid_608027248313960152*thid_OIP.fmwSKKmKpmZtJiBDps1kLAHaEo",
        "insightsMetadata":{
        "recipeSourcesCount":0,
        "bestRepresentativeQuery":{
            "text":"Tropical Beaches Desktop Wallpaper",
            "displayText":"Tropical Beaches Desktop Wallpaper",
            "webSearchUrl":"https:\/\/www.bing.com\/images\/search?q=Tropical+Beaches+Desktop+Wallpaper&id=8607ACDACB243BDEA7E1EF78127DA931E680E3A5&FORM=IDBQDM"
        },
        "pagesIncludingCount":115,
        "availableSizesCount":44
        },
        "imageId":"8607ACDACB243BDEA7E1EF78127DA931E680E3A5",
        "accentColor":"0050B2"
    }]
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación de una sola página](../tutorial-bing-image-search-single-page-app.md)

## <a name="see-also"></a>Consulte también

* [¿Qué es Bing Image Search API?](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/overview)  
* [Prueba de una demostración interactiva en línea](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/)
* [Detalles de precios de las Bing Search API](https://azure.microsoft.com/pricing/details/cognitive-services/search-api/) 
* [Documentación de Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services)
* [Referencia de Bing Image Search API](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference)
