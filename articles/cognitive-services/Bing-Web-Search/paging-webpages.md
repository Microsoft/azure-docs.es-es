---
title: Navegación por los resultados de la búsqueda de Bing Web Search API
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo navegar por los resultados de la búsqueda de Bing Web Search API.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.assetid: 26CA595B-0866-43E8-93A2-F2B5E09D1F3B
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 05/15/2019
ms.author: aahi
ms.openlocfilehash: fa02c0913329ec740e3066b05b2e44a36ad379ce
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797803"
---
# <a name="how-to-page-through-results-from-the-bing-web-search-api"></a>Cómo navegar por los resultados de Bing Web Search API

Cuando se llama a Web Search API, Bing devuelve una lista de resultados. La lista es un subconjunto del número total de resultados que pueden estar relacionados con la consulta. Para obtener el número total estimado de resultados disponibles, consulte el campo [totalEstimatedMatches](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference) del objeto de respuesta.  

En el siguiente ejemplo se muestra el campo `totalEstimatedMatches` que incluye una respuesta web.  

```json
{
    "_type" : "SearchResponse",
    "webPages" : {
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3A43CA...",
        "totalEstimatedMatches" : 262000,
        "value" : [...]
    }
}  
```

Para paginar mediante las páginas web disponibles, use los parámetros de consulta [count](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#count) y [offset](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#offset).  

El parámetro `count` especifica el número de resultados que se van a devolver en la respuesta. El número máximo de resultados que se pueden solicitar en la respuesta es 50. El valor predeterminado es 10. El número real entregado puede ser menor que el solicitado.

El parámetro `offset` especifica el número de resultados que se van a omitir. `offset` está basado en cero y debe ser menor que (`totalEstimatedMatches` - `count`).  

Si quiere mostrar 15 páginas web por página, debe establecer `count` en 15 y `offset` en 0 para obtener la primera página de resultados. Con todas las páginas posteriores, se incrementaría `offset` en 15 (por ejemplo, 15, 30).  

El ejemplo siguiente solicita 15 páginas web a partir del desplazamiento 45.  

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&count=15&offset=45&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
Host: api.cognitive.microsoft.com  
```

Si el valor predeterminado `count` es válido para su implementación, solo debe especificar el parámetro de consulta `offset`.  

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&offset=45&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
Host: api.cognitive.microsoft.com  
```

Web Search API devuelve resultados que incluyen páginas web y pueden incluir imágenes, vídeos y noticias. Al paginar los resultados de búsqueda, lo que hace es paginar la respuesta [WebAnswer](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#webanswer), y no las otras respuestas, como imágenes o noticias. Por ejemplo, si establece `count` en 50, obtendrá 50 resultados de página web, pero la respuesta puede incluir también resultados de las otras respuestas. Por ejemplo, la respuesta puede incluir 15 imágenes y 4 artículos de noticias. Los resultados pueden incluir también noticias en la primera página, pero no en la segunda página o viceversa.   

Si especifica el parámetro de consulta `responseFilter` y no incluye páginas web en la lista de filtros, no use los parámetros `count` y `offset`. 

> [!NOTE]
> El campo `TotalEstimatedAnswers` es una estimación del número total de resultados de la búsqueda que puede recuperar de la consulta actual.  Al establecer los parámetros `count` y `offset`, el número `TotalEstimatedAnswers` puede cambiar. 

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es la API de Bing Web Search](overview.md)?
