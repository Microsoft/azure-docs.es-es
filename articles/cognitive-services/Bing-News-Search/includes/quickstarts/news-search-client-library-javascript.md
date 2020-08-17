---
title: Inicio rápido de la biblioteca cliente para JavaScript de Bing News Search
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/12/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: cc96233ea6e2d02f3c3a2036466e3934aa234f5b
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407987"
---
Este inicio rápido para empezar a buscar noticias con la biblioteca cliente de Bing News Search para JavaScript. Aunque Bing News Search tiene una API REST compatible con la mayoría de los lenguajes de programación, la biblioteca cliente proporciona una forma sencilla de integrar el servicio en sus aplicaciones. El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples/blob/master/Samples/newsSearch.js).

## <a name="prerequisites"></a>Prerrequisitos

* [Node.js](https://nodejs.org/en/)

Para configurar una aplicación de consola mediante la biblioteca cliente de Bing News Search:
1. Ejecute `npm install ms-rest-azure` en el entorno de desarrollo.
2. Ejecute `npm install azure-cognitiveservices-newssearch` en el entorno de desarrollo.


[!INCLUDE [cognitive-services-bing-news-search-signup-requirements](~/includes/cognitive-services-bing-news-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

1. Creación de una instancia de `CognitiveServicesCredentials`. Cree variables para el término de búsqueda y la clave de suscripción.

    ```javascript
    const CognitiveServicesCredentials = require('ms-rest-azure').CognitiveServicesCredentials;
    let credentials = new CognitiveServicesCredentials('YOUR-ACCESS-KEY');
    let search_term = 'Winter Olympics'
    ```

2. Cree una instancia del cliente:
    
    ```javascript
    const NewsSearchAPIClient = require('azure-cognitiveservices-newssearch');
    let client = new NewsSearchAPIClient(credentials);
    ```

## <a name="send-a-search-query"></a>Envío de una consulta de búsqueda

1. Utilice el cliente para buscar un término de consulta, en este caso "Juegos olímpicos de invierno":
    
    ```javascript
    client.newsOperations.search(search_term).then((result) => {
        console.log(result.value);
    }).catch((err) => {
        throw err;
    });
    ```

El código imprime `result.value` elementos en la consola sin analizar ningún texto. Los resultados, si hubiera por categoría, incluyen:

- `_type: 'NewsArticle'`
- `_type: 'WebPage'`
- `_type: 'VideoObject'`
- `_type: 'ImageObject'`

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación web de una sola página](../../tutorial-bing-news-search-single-page-app.md)
