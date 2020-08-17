---
title: Inicio rápido de la biblioteca cliente para JavaScript de Bing Image Search
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/04/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: 953648f5cf83d5ffd22683ba0ce02335a637f18a
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407238"
---
Use este inicio rápido para buscar su primera imagen mediante la biblioteca cliente de Bing Image Search, un contenedor de la API y que contiene las mismas características. Esta sencilla aplicación de JavaScript envía una consulta de búsqueda de imagen, analiza la respuesta de JSON y muestra la dirección URL de la primera imagen devuelta.

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples/blob/master/Samples/imageSearch.js) con anotaciones y control de errores adicionales.

## <a name="prerequisites"></a>Requisitos previos

* El [SDK de Image Search de Cognitive Services para Node.js](https://www.npmjs.com/package/@azure/cognitiveservices-imagesearch)
    * Instalación mediante `npm install @azure/cognitiveservices-imagesearch`
* El módulo [REST de Azure de Node.js](https://www.npmjs.com/package/ms-rest-azure)
    * Instalación mediante `npm install ms-rest-azure`

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](~/includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

1. Cree un nuevo archivo JavaScript en el editor o el IDE que prefiera y establezca la rigurosidad, los protocolos https y demás requisitos.

    ```javascript
    'use strict';
    const ImageSearchAPIClient = require('@azure/cognitiveservices-imagesearch');
    const CognitiveServicesCredentials = require('ms-rest-azure').CognitiveServicesCredentials;
    ```

2. En el método principal del proyecto, cree variables para la clave de suscripción válida, los resultados de imágenes que Bing devolverá y un término de búsqueda. Luego, use la clave para crear instancias del cliente de búsqueda de imagen.

    ```javascript
    //replace this value with your valid subscription key.
    let serviceKey = "ENTER YOUR KEY HERE";

    //the search term for the request
    let searchTerm = "canadian rockies";

    //instantiate the image search client
    let credentials = new CognitiveServicesCredentials(serviceKey);
    let imageSearchApiClient = new ImageSearchAPIClient(credentials);

    ```

## <a name="create-an-asynchronous-helper-function"></a>Creación de una función auxiliar asincrónica

1. Cree una función para llamar al cliente de manera asincrónica y devuelva la respuesta del servicio Bing Image Search.
    ```javascript
    //a helper function to perform an async call to the Bing Image Search API
    const sendQuery = async () => {
        return await imageSearchApiClient.imagesOperations.search(searchTerm);
    };
    ```
   ## <a name="send-a-query-and-handle-the-response"></a>Envío de una consulta y control de la respuesta

1. Llame a la función auxiliar y controle su `promise` para analizar los resultados de imágenes que se devuelven en la respuesta.

    Si la respuesta contiene resultados de la búsqueda, almacene el primer resultado e imprima sus detalles, como la dirección URL de una miniatura, la dirección URL original, además del número total de imágenes devueltas.
    ```javascript
    sendQuery().then(imageResults => {
        if (imageResults == null) {
        console.log("No image results were found.");
        }
        else {
            console.log(`Total number of images returned: ${imageResults.value.length}`);
            let firstImageResult = imageResults.value[0];
            //display the details for the first image result. After running the application,
            //you can copy the resulting URLs from the console into your browser to view the image.
            console.log(`Total number of images found: ${imageResults.value.length}`);
            console.log(`Copy these URLs to view the first image returned:`);
            console.log(`First image thumbnail url: ${firstImageResult.thumbnailUrl}`);
            console.log(`First image content url: ${firstImageResult.contentUrl}`);
        }
      })
      .catch(err => console.error(err))
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial de aplicación de una sola página de Bing Image Search](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/tutorial-bing-image-search-single-page-app)

## <a name="see-also"></a>Consulte también

* [¿Qué es Bing Image Search?](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/overview)
* [Prueba de una demostración interactiva en línea](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/)
* [Ejemplos de Node.js para el SDK de Azure Cognitive Services](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples)
* [Documentación de Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services)
* [Referencia de Bing Image Search API](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference)
