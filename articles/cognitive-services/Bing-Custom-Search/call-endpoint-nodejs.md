---
title: 'Inicio rápido: Llamada al punto de conexión de Bing Custom Search con Node.js | Microsoft Docs'
titleSuffix: Azure Cognitive Services
description: Use este documento de inicio rápido para comenzar a solicitar los resultados de búsqueda de la instancia de Bing Custom Search con Node.js.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: quickstart
ms.date: 05/08/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: dcfb5ed0b7a7fe2ec004ef5ceebf9ee51112c67c
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406459"
---
# <a name="quickstart-call-your-bing-custom-search-endpoint-using-nodejs"></a>Inicio rápido: Llamada al punto de conexión de Bing Custom Search con Node.js

Use este inicio rápido para aprender cómo solicitar los resultados de búsqueda de la instancia de Bing Custom Search. Si bien esta aplicación está escrita en JavaScript, Bing Custom Search API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. El código fuente del ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/nodejs/Search/BingCustomSearchv7.js).

## <a name="prerequisites"></a>Prerrequisitos

- Una instancia de Bing Custom Search. Para más información, consulte [Inicio rápido: Creación de la primera instancia de Bing Custom Search](quick-start.md).

- [El entorno de tiempo de ejecución de JavaScript para Node.js](https://www.nodejs.org/).

- La [biblioteca de solicitudes de JavaScript](https://github.com/request/request).

[!INCLUDE [cognitive-services-bing-custom-search-prerequisites](../../../includes/cognitive-services-bing-custom-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

- Cree un archivo JavaScript en el editor o el IDE que prefiera y agregue una instrucción `require()` para la biblioteca de solicitudes. Cree variables para la clave de suscripción, el identificador de configuración personalizada y un término de búsqueda.

    ```javascript
    var request = require("request");
    
    var subscriptionKey = 'YOUR-SUBSCRIPTION-KEY';
    var customConfigId = 'YOUR-CUSTOM-CONFIG-ID';
    var searchTerm = 'microsoft';
    ```

## <a name="send-and-receive-a-search-request"></a>Envío y recepción de una solicitud de búsqueda 

1. Cree una variable para almacenar la información que se va a enviar en la solicitud. Construya la dirección URL de solicitud anexando el término de búsqueda al parámetro de consulta `q=`, y el identificador de configuración personalizada de la instancia de la búsqueda al parámetro `customconfig=`. Separe los parámetros con un carácter de "y" comercial (`&`). Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```javascript
    var info = {
        url: 'https://api.cognitive.microsoft.com/bingcustomsearch/v7.0/search?' + 
            'q=' + searchTerm + "&" +
            'customconfig=' + customConfigId,
        headers: {
            'Ocp-Apim-Subscription-Key' : subscriptionKey
        }
    }
    ```

1. Use la biblioteca de solicitudes de JavaScript para enviar una solicitud de búsqueda a la instancia de Bing Custom Search, e imprima la información de los resultados, incluidos el nombre, la dirección URL y la fecha en que se rastreó la página web por última vez.

    ```javascript
    request(info, function(error, response, body){
            var searchResponse = JSON.parse(body);
            for(var i = 0; i < searchResponse.webPages.value.length; ++i){
                var webPage = searchResponse.webPages.value[i];
                console.log('name: ' + webPage.name);
                console.log('url: ' + webPage.url);
                console.log('displayUrl: ' + webPage.displayUrl);
                console.log('snippet: ' + webPage.snippet);
                console.log('dateLastCrawled: ' + webPage.dateLastCrawled);
                console.log();
            }
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una página web de Custom Search](./tutorials/custom-search-web-page.md)
