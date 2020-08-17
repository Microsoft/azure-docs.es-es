---
title: 'Inicio rápido: Sugerencia de consultas de búsqueda con la API REST Bing Autosuggest y Python'
titleSuffix: Azure Cognitive Services
description: Aprenda a empezar rápidamente a sugerir términos de búsqueda en tiempo real con la API Bing Autosuggest.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-autosuggest
ms.topic: quickstart
ms.date: 05/06/2020
ms.author: aahi
ms.custom: devx-track-python
ms.openlocfilehash: 2106b36d1b769f0bb207d77eb2395623c14a4a17
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87848067"
---
# <a name="quickstart-suggest-search-queries-with-the-bing-autosuggest-rest-api-and-python"></a>Inicio rápido: Sugerencia de consultas de búsqueda con la API REST Bing Autosuggest y Python

Siga este inicio rápido para aprender a realizar llamadas a la API Bing Autosuggest y a leer la respuesta JSON. Esta sencilla aplicación de Python envía una consulta de búsqueda parcial a la API y devuelve sugerencias en las búsquedas. Si bien esta aplicación está escrita en Python, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/python/Search/BingAutosuggestv7.py).

## <a name="prerequisites"></a>Requisitos previos

* [Python 3.x](https://www.python.org/downloads/) 

[!INCLUDE [cognitive-services-bing-news-search-signup-requirements](../../../../includes/cognitive-services-bing-autosuggest-signup-requirements.md)]

## <a name="create-a-new-application"></a>Creación de una aplicación

1. Cree un archivo de Python en su IDE o editor favoritos. Agregue las importaciones siguientes:

    ```python
    import http.client, urllib.parse, json
    ```

2. Cree variables para el host y la ruta de acceso de la API, el [código de mercado](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-autosuggest-api-v7-reference#market-codes) y una consulta de búsqueda parcial. Use el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```python
    subscriptionKey = 'enter key here'
    host = 'api.cognitive.microsoft.com'
    path = '/bing/v7.0/Suggestions'
    mkt = 'en-US'
    query = 'sail'
    ```

3. Cree una cadena de parámetros; para ello, anexe al código de mercado el parámetro `mkt=` y anexe a la consulta el parámetro `q=`.

    ```python
    params = '?mkt=' + mkt + '&q=' + query
    ```

## <a name="create-and-send-an-api-request"></a>Creación y envío de una solicitud de API

1. Agregue la clave de suscripción a un encabezado `Ocp-Apim-Subscription-Key`.
    
    ```python
    headers = {'Ocp-Apim-Subscription-Key': subscriptionKey}
    ```

2. Conéctese a la API mediante `HTTPSConnection()` y envíe la solicitud `GET` que contiene los parámetros de solicitud.
    
    ```python
    conn = http.client.HTTPSConnection(host)
    conn.request ("GET", path + params, None, headers)
    response = conn.getresponse ()
    return response.read ()
    ```

3. Obtenga e imprima la respuesta JSON.

    ```python
    result = get_suggestions ()
    print (json.dumps(json.loads(result), indent=4))
    ```

## <a name="example-json-response"></a>Ejemplo de respuesta JSON

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "_type": "Suggestions",
  "queryContext": {
    "originalQuery": "sail"
  },
  "suggestionGroups": [
    {
      "name": "Web",
      "searchSuggestions": [
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dgvtP9TS9NwhajSapY2Se6y1eCbP2fq_GiP2n-cxi6OY\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailrite%26FORM%3dUSBAPI\u0026p\u003dDevEx,5003.1",
          "displayText": "sailrite",
          "query": "sailrite",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dBTS0G6AakxntIl9rmbDXtk1n6rQpsZZ99aQ7ClE7dTY\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsail%2bsand%2bpoint%26FORM%3dUSBAPI\u0026p\u003dDevEx,5004.1",
          "displayText": "sail sand point",
          "query": "sail sand point",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dc0QOA_j6swCZJy9FxqOwke2KslJE7ZRmMooGClAuCpY\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailboats%2bfor%2bsale%26FORM%3dUSBAPI\u0026p\u003dDevEx,5005.1",
          "displayText": "sailboats for sale",
          "query": "sailboats for sale",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dmnMdREUH20SepmHQH1zlh9Hy_w7jpOlZFm3KG2R_BoA\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailing%2banarchy%26FORM%3dUSBAPI\u0026p\u003dDevEx,5006.1",
          "displayText": "sailing anarchy",
          "query": "sailing anarchy",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dWLFO-B1GG5qtBGnoU1Bizz02YKkg5fgAQtHwhXn4z8I\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailpoint%26FORM%3dUSBAPI\u0026p\u003dDevEx,5007.1",
          "displayText": "sailpoint",
          "query": "sailpoint",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dquBMwmKlGwqC5wAU0K7n416plhWcR8zQCi7r-Fw9Y0w\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailflow%26FORM%3dUSBAPI\u0026p\u003dDevEx,5008.1",
          "displayText": "sailflow",
          "query": "sailflow",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003d0udadFl0gCTKCp0QmzQTXS3_y08iO8FpwsoKPHPS6kw\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailboatdata%26FORM%3dUSBAPI\u0026p\u003dDevEx,5009.1",
          "displayText": "sailboatdata",
          "query": "sailboatdata",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003deSSt0MRSbl2V0RFPSuVd-gC7fGOT4717pz55EBUgPec\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailor%2b2025%26FORM%3dUSBAPI\u0026p\u003dDevEx,5010.1",
          "displayText": "sailor 2025",
          "query": "sailor 2025",
          "searchKind": "WebSearch"
        }
      ]
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación web de una sola página](../tutorials/autosuggest.md)

## <a name="see-also"></a>Consulte también

- [¿Qué es Bing Autosuggest?](../get-suggested-search-terms.md)
- [Referencia de Bing Autosuggest API v7](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-autosuggest-api-v7-reference)
