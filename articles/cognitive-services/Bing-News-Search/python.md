---
title: 'Inicio rápido: Realizar una búsqueda de noticias mediante Python y la API de REST de Bing News Search'
titleSuffix: Azure Cognitive Services
description: Use esta guía de inicio rápido para enviar una solicitud a la API de REST de Bing News Search mediante Python y reciba una respuesta de JSON.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: quickstart
ms.date: 06/16/2020
ms.author: aahi
ms.custom: seodec2018, devx-track-python
ms.openlocfilehash: 115d0204b4e1a05e5d4c655efb52b3fc3123bebe
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87851841"
---
# <a name="quickstart-perform-a-news-search-using-python-and-the-bing-news-search-rest-api"></a>Inicio rápido: Realizar una búsqueda de noticias mediante Python y la API de REST de Bing News Search

Use este inicio rápido para realizar la primera llamada a la API Bing News Search. Esta sencilla aplicación de Python envía una consulta de búsqueda a la API y procesa el resultado JSON. 

Aunque esta aplicación está escrita en Python, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

Para ejecutar este ejemplo de código como un cuaderno de Jupyter Notebook en [MyBinder](https://mybinder.org), seleccione el distintivo **launch binder**: 

[![launch binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=BingNewsSearchAPI.ipynb)

El código fuente del ejemplo está también disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/python/Search/BingNewsSearchv7.py).

[!INCLUDE [cognitive-services-bing-news-search-signup-requirements](../../../includes/cognitive-services-bing-news-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

Cree un nuevo archivo de Python en el IDE o editor que prefiera e importe el módulo de solicitud. Cree variables para la clave de suscripción, un punto de conexión y un término de búsqueda. Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

```python
import requests

subscription_key = "your subscription key"
search_term = "Microsoft"
search_url = "https://api.cognitive.microsoft.com/bing/v7.0/news/search"
```

## <a name="create-parameters-for-the-request"></a>Creación de parámetros para la solicitud

Agregue la clave de suscripción a un nuevo diccionario con `Ocp-Apim-Subscription-Key` como clave. Haga lo mismo para los parámetros de búsqueda.

```python
headers = {"Ocp-Apim-Subscription-Key" : subscription_key}
params  = {"q": search_term, "textDecorations": True, "textFormat": "HTML"}
```

## <a name="send-a-request-and-get-a-response"></a>Envío de una solicitud y obtención de una respuesta

1. Utilice la biblioteca de solicitudes para llamar a la API Bing Visual Search con su clave de suscripción y los objetos de diccionario que se crearon en el paso anterior.

    ```python
    response = requests.get(search_url, headers=headers, params=params)
    response.raise_for_status()
    search_results = json.dumps(response.json())
    ```

2. Acceda a las descripciones de los artículos contenidos en la respuesta de la API, que se almacena en `search_results` como un objeto JSON. 
    
    ```python
    descriptions = [article["description"] for article in search_results["value"]]
    ```

## <a name="display-the-results"></a>Mostrar los resultados

A continuación, estas descripciones se pueden representar como una tabla con la palabra clave de búsqueda resaltada en negrita.

```python
from IPython.display import HTML
rows = "\n".join(["<tr><td>{0}</td></tr>".format(desc)
                  for desc in descriptions])
HTML("<table>"+rows+"</table>")
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación web de una sola página](tutorial-bing-news-search-single-page-app.md)
