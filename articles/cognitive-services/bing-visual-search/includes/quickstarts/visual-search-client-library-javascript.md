---
title: Inicio rápido de la biblioteca cliente para JavaScript de Bing Visual Search
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/26/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: 3a6c89c3932adb4f9465172ca64b9356db1f624a
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407017"
---
Use este artículo de inicio rápido para empezar a obtener información detallada de imágenes desde el servicio Bing Visual Search con la biblioteca cliente para JavaScript. Aunque Bing Visual Search tiene una API REST compatible con la mayoría de los lenguajes de programación, la biblioteca cliente proporciona una forma sencilla de integrar el servicio en sus aplicaciones. El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples/blob/master/Samples/visualSearch.js). 

[Documentación de referencia](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-visualsearch/?view=azure-node-latest) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-visualsearch) | [Paquete (NPM)](https://www.npmjs.com/package/@azure/cognitiveservices-visualsearch) | [Ejemplos](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples/)

## <a name="prerequisites"></a>Prerrequisitos
* [Node.js](https://www.nodejs.org/)
* La biblioteca cliente de Bing Visual Search para JavaScript
    * Para configurar una aplicación de consola mediante la biblioteca cliente de Bing Visual Search, ejecute los siguientes comandos:
        1. `npm install ms-rest-azure`
        2. `npm install azure-cognitiveservices-visualsearch`.


[!INCLUDE [cognitive-services-bing-visual-search-signup-requirements](~/includes/cognitive-services-bing-visual-search-signup-requirements.md)]

<a name="client"></a>

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

1. Cree un archivo JavaScript en el editor o el IDE que prefiera y agregue los siguientes requisitos. Después, cree variables para la clave de suscripción, el identificador de configuración personalizado y la ruta de acceso al archivo para imagen que quiere cargar. 

    ```javascript
    const os = require("os");
    const async = require('async');
    const fs = require('fs');
    const Search = require('azure-cognitiveservices-visualsearch');
    const CognitiveServicesCredentials = require('ms-rest-azure').CognitiveServicesCredentials;
    
    let keyVar = 'YOUR-VISUAL-SEARCH-ACCESS-KEY';
    let credentials = new CognitiveServicesCredentials(keyVar);
    let filePath = "../Data/image.jpg";
    ```

2. Cree una instancia del cliente.

    ```javascript
    let visualSearchClient = new Search.VisualSearchClient(credentials);
    ```

## <a name="search-for-images"></a>Búsqueda de imágenes

1. Use `fs.createReadStream()` para leer en el archivo de imagen y cree variables para la solicitud de búsqueda y los resultados. Luego, use el cliente para buscar imágenes.

    ```javascript
    let fileStream = fs.createReadStream(filePath);
    let visualSearchRequest = JSON.stringify({});
    let visualSearchResults;
    try {
        visualSearchResults = await visualSearchClient.images.visualSearch({
            image: fileStream,
            knowledgeRequest: visualSearchRequest
        });
        console.log("Search visual search request with binary of image");
    } catch (err) {
        console.log("Encountered exception. " + err.message);
    }
    ```

2. Analice los resultados de la consulta anterior:

    ```javascript
    // Visual Search results
    if (visualSearchResults.image.imageInsightsToken) {
        console.log(`Uploaded image insights token: ${visualSearchResults.image.imageInsightsToken}`);
    }
    else {
        console.log("Couldn't find image insights token!");
    }
    
    // List of tags
    if (visualSearchResults.tags.length > 0) {
        let firstTagResult = visualSearchResults.tags[0];
        console.log(`Visual search tag count: ${visualSearchResults.tags.length}`);
    
        // List of actions in first tag
        if (firstTagResult.actions.length > 0) {
            let firstActionResult = firstTagResult.actions[0];
            console.log(`First tag action count: ${firstTagResult.actions.length}`);
            console.log(`First tag action type: ${firstActionResult.actionType}`);
        }
        else {
            console.log("Couldn't find tag actions!");
        }
    
    }
    else {
        console.log("Couldn't find image tags!");
    }
    
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Compilar una aplicación web de una sola página](../../tutorial-bing-visual-search-single-page-app.md)
