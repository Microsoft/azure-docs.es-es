---
title: 'Inicio rápido: Llamada al punto de conexión de Bing Custom Search con Python | Microsoft Docs'
titleSuffix: Azure Cognitive Services
description: Use este documento de inicio rápido para comenzar a solicitar los resultados de búsqueda de la instancia de Bing Custom Search con Python.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: quickstart
ms.date: 05/08/2020
ms.author: aahi
ms.custom: devx-track-python
ms.openlocfilehash: b29d9921de0226cb8660b3f7e3c6d3e0d4a22754
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852521"
---
# <a name="quickstart-call-your-bing-custom-search-endpoint-using-python"></a>Inicio rápido: Llamada al punto de conexión de Bing Custom Search con Python

Use este inicio rápido para aprender cómo solicitar los resultados de búsqueda de la instancia de Bing Custom Search. Si bien esta aplicación está escrita en Python, Bing Custom Search API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. El código fuente del ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/python/Search/BingCustomSearchv7.py).

## <a name="prerequisites"></a>Prerrequisitos

- Una instancia de Bing Custom Search. Para más información, consulte [Inicio rápido: Creación de la primera instancia de Bing Custom Search](quick-start.md).
- [Python](https://www.python.org/) 2.x o 3.x.

[!INCLUDE [cognitive-services-bing-custom-search-prerequisites](../../../includes/cognitive-services-bing-custom-search-signup-requirements.md)]


## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

- Cree un archivo de Python en el IDE o editor que prefiera y agregue las siguientes instrucciones de importación. Cree variables para la clave de suscripción, el identificador de configuración personalizada y un término de búsqueda.

    ```python
    import json
    import requests
    
    subscriptionKey = "YOUR-SUBSCRIPTION-KEY"
    customConfigId = "YOUR-CUSTOM-CONFIG-ID"
    searchTerm = "microsoft"
    ```

## <a name="send-and-receive-a-search-request"></a>Envío y recepción de una solicitud de búsqueda 

1. Construya la dirección URL de solicitud anexando el término de búsqueda al parámetro de consulta `q=`, y el identificador de configuración personalizada de la instancia de la búsqueda al parámetro `customconfig=`. Separe los parámetros con un carácter de "y" comercial (`&`). Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```python
    url = 'https://api.cognitive.microsoft.com/bingcustomsearch/v7.0/search?' + 'q=' + searchTerm + '&' + 'customconfig=' + customConfigId
    ```

2. Envíe la solicitud a la instancia de Bing Custom Search e imprima los resultados de búsqueda devueltos.  

    ```python
    r = requests.get(url, headers={'Ocp-Apim-Subscription-Key': subscriptionKey})
    print(r.text)
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una página web de Custom Search](./tutorials/custom-search-web-page.md)
