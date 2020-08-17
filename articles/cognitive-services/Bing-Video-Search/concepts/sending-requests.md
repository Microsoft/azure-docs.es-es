---
title: Envío de solicitudes de búsqueda a Bing Video Search API
titleSuffix: Azure Cognitive Services
description: En este artículo se describen los parámetros y atributos de las solicitudes enviadas a Bing Video Search API, así como el objeto de respuesta JSON que devuelve.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-video-search
ms.topic: conceptual
ms.date: 12/03/2019
ms.author: aahi
ms.openlocfilehash: b27aa1409d543c157069d2701c49ef54a097e552
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075224"
---
# <a name="sending-search-requests-to-the-bing-video-search-api"></a>Envío de solicitudes de búsqueda a Bing Video Search API

En este artículo se describen los parámetros y atributos de las solicitudes enviadas a Bing Video Search API, así como el objeto de respuesta JSON que devuelve. 

[!INCLUDE [cognitive-services-bing-video-search-signup-requirements](../../../../includes/cognitive-services-bing-video-search-signup-requirements.md)]

## <a name="suggest-search-terms-with-the-bing-autosuggest-api"></a>Sugerencia de términos de búsqueda con Bing Autosuggest API

Si proporciona un cuadro de búsqueda donde el usuario escribe su término de búsqueda, use [Bing Autosuggest API](../../bing-autosuggest/get-suggested-search-terms.md) para mejorar la experiencia. La API devuelve cadenas consulta sugeridas basadas en términos de búsqueda parciales a medida que el usuario escribe.

Una vez que el usuario escriba el término de búsqueda, codifíquelo como URL antes de establecer el parámetro de consulta [q](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#query). Por ejemplo, si el usuario escribe *sailing dinghies*, establezca `q` en `sailing+dinghies` o `sailing%20dinghies`.

## <a name="sending-a-request"></a>Envío de una solicitud

Para obtener resultados de la búsqueda de vídeo, debe enviar una solicitud GET al punto de conexión siguiente:  
  
```
https://api.cognitive.microsoft.com/bing/v7.0/videos/search
```
   
La solicitud debe usar el protocolo HTTPS.

Se recomienda que todas las solicitudes procedan de un servidor. Al distribuir la clave como parte de una aplicación cliente, existe una mayor probabilidad de que un tercero malintencionado acceda a ella. Además, la realización de llamadas desde un servidor proporciona un único punto de actualización para futuras versiones de la API.

  
La solicitud debe especificar el parámetro de consulta [q](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#query), que contiene el término de búsqueda del usuario. Aunque es opcional, la solicitud debe especificar también el parámetro de consulta [mkt](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#mkt), que identifica el mercado de donde desea que procedan los resultados. Para ver una lista de parámetros de consulta opcionales, como `pricing`, consulte [Query Parameters](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#query-parameters) (Parámetros de consulta). Todos los valores de parámetro de consulta deben estar codificados con una dirección URL.  
  
La solicitud debe especificar el encabezado [Ocp-Apim-Subscription-Key](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#subscriptionkey). Aunque es opcional, se recomienda especificar los encabezados siguientes:  
  
-   [User-Agent](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#useragent)  
-   [X-MSEdge-ClientID](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#clientid)  
-   [X-Search-ClientIP](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#clientip)  
-   [X-Search-Location](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#location)  

Los encabezados de IP y ubicación del cliente son importantes para devolver contenido relativo a la ubicación.  

Para ver una lista de todos los encabezados de solicitud y respuesta, consulte [Encabezados](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#headers).

## <a name="example-search-request"></a>Ejemplo de solicitud de búsqueda

A continuación se muestra una solicitud de búsqueda que incluye todos los encabezados y parámetros de consulta sugeridos. Si es la primera vez que llama a cualquiera de las API de Bing, no incluya el encabezado de identificador de cliente. Solo debe incluir el identificador de cliente si se ha llamado previamente a una API de Bing y Bing ha devuelto un identificador de cliente para esa combinación de usuario y dispositivo. 
  
```  
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/search?q=sailing+dinghies&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

## <a name="example-json-response"></a>Ejemplo de respuesta JSON

A continuación se muestra la respuesta a la solicitud anterior. En el ejemplo también se muestran los encabezados de respuesta específicos de Bing.

[!INCLUDE [cognitive-services-bing-url-note](../../../../includes/cognitive-services-bing-url-note.md)]

```json
BingAPIs-TraceId: 76DD2C2549B94F9FB55B4BD6FEB6AC
X-MSEdge-ClientID: 1C3352B306E669780D58D607B96869
BingAPIs-Market: en-US

{
    "_type" : "Videos",
    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7545D5694...",
    "totalEstimatedMatches" : 1000,
    "value" : [
        {
            "name" : "How to sail - What to Wear for Dinghy Sailing",
            "description" : "An informative video on what to wear when...",
            "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7545D56...",
            "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=OVP.DYWCvh...",
            "datePublished" : "2014-03-04T11:51:53",
            "publisher" : [
                {
                    "name" : "Fabrikam"
                }
            ],
            "creator" : {
                "name" : "Marcus Appel"
            },
            "contentUrl" : "https:\/\/www.fabrikam.com\/watch?v=vzmPjHZ--g",
            "hostPageUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7545D56944...",
            "encodingFormat" : "h264",
            "hostPageDisplayUrl" : "https:\/\/www.fabrikam.com\/watch?v=vzmPjBZ--g",
            "width" : 1280,
            "height" : 720,
            "duration" : "PT2M47S",
            "motionThumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?id=OM.Y6...",
            "embedHtml" : "<iframe width=\"1280\" height=\"720\" src=\"https:...><\/iframe>",
            "allowHttpsEmbed" : true,
            "viewCount" : 8743,
            "thumbnail" : {
                "width" : 300,
                "height" : 168
            },
            "videoId" : "6DB795E11A6E3CBAAD636DB795E11E3CBAAD63",
            "allowMobileEmbed" : true,
            "isSuperfresh" : false
        },
        . . .
    ],
    "nextOffset" : 0,
    "pivotSuggestions" : [
        {
            "pivot" : "sailing",
            "suggestions" : []
        },
        {
            "pivot" : "dinghies",
            "suggestions" : [
                {
                    "text" : "Sailing Cruising",
                    "displayText" : "Cruising",
                    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=81EF754...",
                    "searchLink" : "https:\/\/api.cognitive.microsoft.com...",
                    "thumbnail" : {
                        "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?q=Sailing..."
                    }
                },
                . . .
            ]
        }
    ]
}
```

## <a name="next-steps"></a>Pasos siguientes

Pruebe la API. Vaya a la [consola de pruebas de Video Search API](https://dev.cognitive.microsoft.com/docs/services/56b43f3ccf5ff8098cef3809/operations/58113fe5e31dac0a1ce6b0a8). 

Para más información sobre cómo consumir los objetos de respuesta, consulte [Búsqueda de vídeos en la web](../search-the-web.md).

Para obtener más información sobre cómo obtener información detallada de un vídeo (por ejemplo, búsquedas relacionadas) consulte [Información de vídeos](../video-insights.md).  
  
Para más información sobre los vídeos más populares en las redes sociales, consulte [Vídeos de tendencia](../trending-videos.md).  
