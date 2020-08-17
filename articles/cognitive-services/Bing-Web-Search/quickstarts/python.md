---
title: 'Inicio rápido: Realizar una búsqueda con Python: Bing Web Search API'
titleSuffix: Azure Cognitive Services
description: Use esta guía de inicio rápido para enviar solicitudes a la API de REST Bing Web Search mediante Python y reciba una respuesta JSON.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: quickstart
ms.date: 05/22/2020
ms.author: aahi
ms.custom: seodec2018, devx-track-python
ms.openlocfilehash: 1adb273cfedd2342a1429f0d21cd00e9a5e602a8
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87851178"
---
# <a name="quickstart-use-python-to-call-the-bing-web-search-api"></a>Inicio rápido: Uso de Python para llamar a Bing Web Search API  

Use este inicio rápido para realizar la primera llamada a la API Bing Web Search. Esta aplicación de Python envía una solicitud de búsqueda a la API y muestra la respuesta JSON. Aunque esta aplicación está escrita en Python, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

En este ejemplo se ejecuta como una instancia de Jupyter Notebook en [MyBinder](https://mybinder.org). Para ejecutarla seleccione el distintivo launch binder:

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=BingWebSearchAPI.ipynb)

## <a name="prerequisites"></a>Requisitos previos

* [Python 2.x o 3.x](https://www.python.org/)

[!INCLUDE [bing-web-search-quickstart-signup](../../../../includes/bing-web-search-quickstart-signup.md)]

## <a name="define-variables"></a>Definición de variables

1. Reemplace el valor `subscription_key` por una clave de suscripción válida de su cuenta de Azure.

   ```python
   subscription_key = "YOUR_ACCESS_KEY"
   assert subscription_key
   ```

2. Declare el punto de conexión de Bing Web Search API. Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

   ```python
   search_url = "https://api.cognitive.microsoft.com/bing/v7.0/search"
   ```

3. De forma opcional puede personalizar la consulta de búsqueda reemplazando el valor de `search_term`.

   ```python
   search_term = "Azure Cognitive Services"
   ```

## <a name="make-a-request"></a>Realización de una solicitud

Este código usa la biblioteca `requests` para llamar a la API Bing Web Search y devolver los resultados como objeto JSON. La clave de API se pasa en el diccionario `headers` y el término de búsqueda y los parámetros de consulta se pasan en el diccionario `params`. 

Para obtener una lista completa de opciones y parámetros consulte [Bing Web Search API v7](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference).

```python
import requests

headers = {"Ocp-Apim-Subscription-Key": subscription_key}
params = {"q": search_term, "textDecorations": True, "textFormat": "HTML"}
response = requests.get(search_url, headers=headers, params=params)
response.raise_for_status()
search_results = response.json()
```

## <a name="format-and-display-the-response"></a>Formato y presentación de la respuesta

El objeto `search_results` incluye los resultados de la búsqueda, y dichos metadatos como consultas y páginas relacionadas. Este código utiliza la biblioteca `IPython.display` para dar formato y mostrar la respuesta en el explorador.

```python
from IPython.display import HTML

rows = "\n".join(["""<tr>
                       <td><a href=\"{0}\">{1}</a></td>
                       <td>{2}</td>
                     </tr>""".format(v["url"], v["name"], v["snippet"])
                  for v in search_results["webPages"]["value"]])
HTML("<table>{0}</table>".format(rows))
```

## <a name="sample-code-on-github"></a>Código de ejemplo en GitHub

Para ejecutar este código localmente, consulte el [ejemplo completo que está disponible en GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/python/Search/BingWebSearchv7.py).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial de creación de una aplicación de una sola página con la API Bing Web Search](../tutorial-bing-web-search-single-page-app.md)

[!INCLUDE [bing-web-search-quickstart-see-also](../../../../includes/bing-web-search-quickstart-see-also.md)]
