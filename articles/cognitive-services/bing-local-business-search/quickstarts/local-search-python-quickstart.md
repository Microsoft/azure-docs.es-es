---
title: 'Guía de inicio rápido: Envío de una consulta a Bing Local Business Search API en Python | Microsoft Docs'
titleSuffix: Azure Cognitive Services
description: Use este artículo para empezar a usar Bing Local Business Search API en Python.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: article
ms.date: 11/01/2018
ms.author: rosh, v-gedod
ms.openlocfilehash: e6626c1243f9f05a56effe3538870a5be9ac6b34
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796562"
---
# <a name="quickstart-send-a-query-to-the-bing-local-business-search-api-in-python"></a>Inicio rápido: Envío de una consulta a Bing Local Business Search API en Python

Use esta guía de inicio rápido para empezar a enviar solicitudes a Bing Local Business Search API, que es un servicio de Azure Cognitive Services. Si bien esta aplicación sencilla está escrita en Python, la API es un servicio web RESTful compatible con cualquier lenguaje de programación que pueda realizar solicitudes HTTP y analizar JSON.

Esta aplicación de ejemplo obtiene los datos de respuesta local de la API para la consulta de búsqueda `hotel in Bellevue`.

## <a name="prerequisites"></a>Requisitos previos

* [Python](https://www.python.org/) 2.x o 3.x
 
Debe tener una [cuenta de Cognitive Services API](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) con las API de Bing. La [cuenta de evaluación gratuita](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api) es suficiente para esta guía de inicio rápido. Utilice la clave de acceso proporcionada por la evaluación gratuita.  Consulte también [Precios de Cognitive Services - Bing Search API](https://azure.microsoft.com/pricing/details/cognitive-services/search-api/).

## <a name="run-the-complete-application"></a>Ejecución de la aplicación completa

El código siguiente obtiene resultados localizados. Se implementa en los pasos siguientes:
1. Declare las variables para especificar el punto de conexión por host y ruta de acceso.
2. Especifique el parámetro de consulta. 
3. Defina la función de búsqueda que crea la solicitud y agrega el encabezado Ocp-Apim-Subscription-Key.
4. Establezca el encabezado Ocp-Apim-Subscription-Key. 
5. Realice la conexión y envíe la solicitud.
6. Imprima los resultados JSON.

Este es el código completo de esta demostración:

```
import http.client, urllib.parse
import json

# Replace the subscriptionKey string value with your valid subscription key.
subscriptionKey = 'YOUR-SUBSCRIPTION-KEY'

host = 'api.cognitive.microsoft.com/bing'
path = '/v7.0/localbusinesses/search'

query = 'restaurant in Bellevue'

params = '?q=' + urllib.parse.quote (query) + '&mkt=en-us'

def get_local():
    headers = {'Ocp-Apim-Subscription-Key': subscriptionKey}
    conn = http.client.HTTPSConnection (host)
    conn.request ("GET", path + params, None, headers)
    response = conn.getresponse ()
    return response.read ()

result = get_local()
print (json.dumps(json.loads(result), indent=4))

```

## <a name="next-steps"></a>Pasos siguientes
- [Guía de inicio rápido de Local Business Search para Java](local-search-java-quickstart.md)
- [Guía de inicio rápido de Local Business Search para C#](local-quickstart.md)
- [Guía de inicio rápido de Local Business Search para Node](local-search-node-quickstart.md)
