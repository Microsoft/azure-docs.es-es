---
title: Envío de solicitudes a Bing Autosuggest API
titleSuffix: Azure Cognitive Services
description: Bing Autosuggest API devuelve una lista de consultas sugeridas según la cadena de consulta parcial del cuadro de búsqueda. Más información sobre el envío de solicitudes.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-autosuggest
ms.topic: conceptual
ms.date: 06/27/2019
ms.author: scottwhi
ms.openlocfilehash: 3aa8d691eb62b94c5784d8a6efc35b53a3762d7f
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89294125"
---
# <a name="sending-requests-to-the-bing-autosuggest-api"></a>Envío de solicitudes a Bing Autosuggest API

Si una aplicación envía consultas a cualquier Bing Search API, puede usar Bing Autosuggest API para mejorar su experiencia de búsqueda. Bing Autosuggest API devuelve una lista de consultas sugeridas según la cadena de consulta parcial del cuadro de búsqueda. A medida que se escriben caracteres en el cuadro de búsqueda, puede mostrar sugerencias en una lista desplegable. Use este artículo para aprender sobre el envío de solicitudes a esta API. 

## <a name="bing-autosuggest-api-endpoint"></a>Punto de conexión de Bing Autosuggest API

**Bing Autosuggest API** incluye un punto de conexión, que devuelve una lista de consultas sugeridas a partir de un término de búsqueda parcial.

Para obtener consultas sugeridas mediante la API de Bing, envíe una solicitud `GET` al punto de conexión siguiente. Use los encabezados y parámetros de dirección URL para definir especificaciones adicionales.

**Punto de conexión:** devuelve sugerencias de búsqueda como resultados JSON pertinentes para la entrada del usuario definida por `?q=""`.

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/Suggestions 
```

Para obtener más información sobre encabezados, parámetros, códigos de mercado, objetos de respuesta, errores etc., consulte la referencia [Bing Autosuggest API v7](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-autosuggest-api-v7-reference).

Las API de **Bing** admiten acciones de búsqueda que devuelven resultados según su tipo. Todos los puntos de conexión de búsqueda devuelven resultados como objetos de respuesta JSON.
Todos los puntos de conexión admiten consultas que devuelven un idioma o una ubicación en concreto por longitud, latitud y radio de búsqueda.

Para una información completa acerca de los parámetros admitidos por cada punto de conexión, consulte las páginas de referencia de cada tipo.
Para obtener ejemplos de solicitudes básicas mediante Autosuggest API, consulte los [inicios rápidos de Autosuggest](https://docs.microsoft.com/azure/cognitive-services/Bing-Autosuggest).

## <a name="bing-autosuggest-api-requests"></a>Bing Autosuggest API, Ruby

> [!NOTE]
> * Las solicitudes a Bing Autosuggest API deben usar el protocolo HTTPS.

Se recomienda que todas las solicitudes procedan de un servidor. Al distribuir la clave como parte de una aplicación cliente, existe una mayor probabilidad de que un tercero malintencionado acceda a ella. Además, cuando se realizan llamadas desde un servidor se proporciona un único punto de actualización para futuras versiones.

La solicitud debe especificar el parámetro de consulta [q](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#query), que contiene el término de búsqueda parcial del usuario. Aunque es opcional, la solicitud debe especificar también el parámetro de consulta [mkt](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#mkt), que identifica el mercado de donde desea que procedan los resultados. Para ver una lista de parámetros de consulta opcionales, consulte [Query Parameters](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#query-parameters) (Parámetros de consulta). Todos los valores de parámetro de consulta deben estar codificados con una dirección URL.

La solicitud debe especificar el encabezado [Ocp-Apim-Subscription-Key](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#subscriptionkey). Aunque es opcional, se recomienda especificar los encabezados siguientes:

- [User-Agent](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#useragent)
- [X-MSEdge-ClientID](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#clientid)
- [X-Search-ClientIP](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#clientip)
- [X-Search-Location](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#location)

Los encabezados de IP y ubicación del cliente son importantes para devolver contenido relativo a la ubicación.

Para ver una lista de todos los encabezados de solicitud y respuesta, consulte [Encabezados](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#headers).

> [!NOTE]
> Cuando se llama a Bing Autosuggest API desde JavaScript, las características de seguridad integradas de su explorador podrían impedirle acceder a los valores de estos encabezados.

Para resolverlo, puede realizar una solicitud de Bing Autosuggest API a través de un proxy CORS. La respuesta de un proxy de este tipo tiene un encabezado `Access-Control-Expose-Headers` que filtra los encabezados de respuesta y hace que estén disponibles para JavaScript.

Es fácil instalar un proxy CORS para permitir que nuestra [aplicación de tutorial](../tutorials/autosuggest.md) obtenga acceso a los encabezados de cliente opcionales. En primer lugar, si aún no lo tiene, [instale Node.js](https://nodejs.org/en/download/). A continuación, escriba el siguiente comando en un símbolo del sistema.

```console
npm install -g cors-proxy-server
```

Luego, cambie el punto de conexión de Bing Autosuggest API del archivo HTML a:

```http
http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/Suggestions
```

Por último, inicie el proxy CORS con el siguiente comando:

```console
cors-proxy-server
```

Deje abierta la ventana de comandos mientras usa la aplicación del tutorial, ya que si la cierra, se detendrá el proxy. En la sección de encabezados HTTP expandibles situada bajo los resultados de la búsqueda, puede ver el encabezado `X-MSEdge-ClientID` (entre otras cosas) y comprobar que es el mismo en todas las solicitudes.

Las solicitudes deben incluir todos los encabezados y parámetros de consulta sugeridos. 

En el ejemplo siguiente se muestra una solicitud que devuelve las cadenas de consulta sugeridas para *navegar*.

> ```http
> GET https://api.cognitive.microsoft.com/bing/v7.0/suggestions?q=sail&mkt=en-us HTTP/1.1
> Ocp-Apim-Subscription-Key: 123456789ABCDE
> X-MSEdge-ClientIP: 999.999.999.999
> X-Search-Location: lat:47.60357;long:-122.3295;re:100
> X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
> Host: api.cognitive.microsoft.com
> ```

Si es la primera vez que llama a cualquiera de las API de Bing, no incluya el encabezado de identificador de cliente. Solo debe incluir el encabezado del identificador de cliente si se ha llamado previamente a una API de Bing y Bing ha devuelto un identificador de cliente para esa combinación de usuario y dispositivo.

El siguiente grupo de sugerencias web es una respuesta a la solicitud anterior. El grupo contiene una lista de sugerencias de consulta de búsqueda, y cada sugerencia incluye un campo `displayText`, `query` y `url`.

El campo `displayText` contiene la consulta sugerida que usaría para rellenar la lista desplegable del cuadro de búsqueda. Debe mostrar todas las sugerencias que incluye la respuesta, y en el orden especificado.  

Si el usuario selecciona una consulta de la lista desplegable, puede usarla para llamar a una de las [API de Bing Search](https://docs.microsoft.com/azure/cognitive-services/bing-web-search/bing-api-comparison?toc=%2Fen-us%2Fazure%2Fcognitive-services%2Fbing-autosuggest%2Ftoc.json&bc=%2Fen-us%2Fazure%2Fbread%2Ftoc.json) y mostrar los resultados por su cuenta, o bien enviar al usuario a la página de resultados de Bing mediante el campo `url` devuelto.

[!INCLUDE [cognitive-services-bing-url-note](../../../../includes/cognitive-services-bing-url-note.md)]

```json
BingAPIs-TraceId: 76DD2C2549B94F9FB55B4BD6FEB6AC
X-MSEdge-ClientID: 1C3352B306E669780D58D607B96869
BingAPIs-Market: en-US

{
    "_type" : "Suggestions",
    "queryContext" : {
        "originalQuery" : "sail"
    },
    "suggestionGroups" : [{
        "name" : "Web",
        "searchSuggestions" : [{
            "url" : "https:\/\/www.bing.com\/search?q=sailing+lessons+seattle&FORM=USBAPI",
            "displayText" : "sailing lessons seattle",
            "query" : "sailing lessons seattle",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailor+moon+news&FORM=USBAPI",
            "displayText" : "sailor moon news",
            "query" : "sailor moon news",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailor+jack%27s+lincoln+city&FORM=USBAPI",
            "displayText" : "sailor jack's lincoln city",
            "query" : "sailor jack's lincoln city",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailing+anarchy&FORM=USBAPI",
            "displayText" : "sailing anarchy",
            "query" : "sailing anarchy",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailboats+for+sale&FORM=USBAPI",
            "displayText" : "sailboats for sale",
            "query" : "sailboats for sale",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailstn.mylabsplus.com&FORM=USBAPI",
            "displayText" : "sailstn.mylabsplus.com",
            "query" : "sailstn.mylabsplus.com",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailusfood&FORM=USBAPI",
            "displayText" : "sailusfood",
            "query" : "sailusfood",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailboats+for+sale+seattle&FORM=USBAPI",
            "displayText" : "sailboats for sale seattle",
            "query" : "sailboats for sale seattle",
            "searchKind" : "WebSearch"
        }]
    }]
}
```

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es Bing Autosuggest?](../get-suggested-search-terms.md)
- [Referencia de Bing Autosuggest API v7](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-autosuggest-api-v7-reference)
- [Obtención de términos de búsqueda sugeridos de Bing Autosuggest API](get-suggestions.md)
