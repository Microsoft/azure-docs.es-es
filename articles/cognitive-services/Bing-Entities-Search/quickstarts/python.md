---
title: 'Inicio rápido: Envío de una solicitud de búsqueda a la API REST mediante Python: Bing Entity Search'
titleSuffix: Azure Cognitive Services
description: Use esta guía de inicio rápido para enviar una solicitud a la API de REST de Bing Entity Search mediante Python y reciba una respuesta de JSON.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: quickstart
ms.date: 05/08/2020
ms.author: aahi
ms.custom: devx-track-python
ms.openlocfilehash: 53f9badbb15c788606390921cb8ef62495c19839
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852759"
---
# <a name="quickstart-send-a-search-request-to-the-bing-entity-search-rest-api-using-python"></a>Inicio rápido: Envío de una solicitud de búsqueda a Bing Entity Search REST API con Python

Use este inicio rápido para realizar la primera llamada a Bing Entity Search API y ver la respuesta JSON. Esta sencilla aplicación de Python envía una consulta de búsqueda de noticias a la API y muestra la respuesta. El código fuente del ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/python/Search/BingEntitySearchv7.py).

Aunque esta aplicación está escrita en Python, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

## <a name="prerequisites"></a>Prerrequisitos

* [Python](https://www.python.org/downloads/) 2.x o 3.x

[!INCLUDE [cognitive-services-bing-news-search-signup-requirements](../../../../includes/cognitive-services-bing-entity-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

1. Cree un archivo de Python en el IDE o editor que prefiera y agregue las siguientes importaciones. Cree variables para la clave de suscripción, un punto de conexión, un mercado y una consulta de búsqueda. Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```python
    import http.client, urllib.parse
    import json
    
    subscriptionKey = 'ENTER YOUR KEY HERE'
    host = 'api.cognitive.microsoft.com'
    path = '/bing/v7.0/entities'
    mkt = 'en-US'
    query = 'italian restaurants near me'
    ```

2. Cree una dirección URL de solicitud anexando la variable de mercado al parámetro `?mkt=`. Codifique como URL la consulta y anéxela al parámetro `&q=`. 
    
    ```python
    params = '?mkt=' + mkt + '&q=' + urllib.parse.quote (query)
    ```

## <a name="send-a-request-and-get-a-response"></a>Envío de una solicitud y obtención de una respuesta

1. Cree una función denominada `get_suggestions()`. 

2. En esta función, agregue la clave de suscripción a un diccionario con `Ocp-Apim-Subscription-Key` como clave.

3. Use `http.client.HTTPSConnection()` para crear un objeto de cliente HTTPS. Envíe una solicitud `GET` mediante `request()` con la información de ruta de acceso, parámetros y encabezado.

4. Almacene la respuesta con `getresponse()` y devuelva `response.read()`.

   ```python
   def get_suggestions ():
    headers = {'Ocp-Apim-Subscription-Key': subscriptionKey}
    conn = http.client.HTTPSConnection (host)
    conn.request ("GET", path + params, None, headers)
    response = conn.getresponse ()
    return response.read()
   ```

5. Llame a `get_suggestions()` e imprima la respuesta JSON.

    ```python
    result = get_suggestions ()
    print (json.dumps(json.loads(result), indent=4))
    ```

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

* [¿Qué es Bing Entity Search API?](../search-the-web.md)
* [Referencia de la API Bing Entity Search](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference).
