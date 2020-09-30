---
title: Tipos y estructura de las respuestas de Bing Web Search API
titleSuffix: Azure Cognitive Services
description: Cuando envía una solicitud de búsqueda a Bing Web Search, devuelve un objeto `SearchResponse` en el cuerpo de respuesta.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 06/25/2019
ms.author: aahi
ms.custom: seodec2018
ms.openlocfilehash: a5e69fe855f0c1e99dc3672425d9aeea13d4e827
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297797"
---
# <a name="bing-web-search-api-response-structure-and-answer-types"></a>Tipos y estructura de las respuestas de Bing Web Search API  

Cuando envía una solicitud de búsqueda a Bing Web Search, devuelve un objeto [`SearchResponse`](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#searchresponse) en el cuerpo de respuesta. El objeto incluye un campo para cada respuesta que para Bing era apropiado para la consulta. En este ejemplo se muestra un objeto de respuesta si Bing ha devuelto todas las respuestas:

```json
{
    "_type": "SearchResponse",
    "queryContext": {...},
    "webPages": {...},
    "images": {...},
    "relatedSearches": {...},
    "videos": {...},
    "news": {...},
    "spellSuggestion": {...},
    "computation": {...},
    "timeZone": {...},
    "rankingResponse": {...}
}, ...
```

Normalmente, Bing Web Search devuelve un subconjunto de las respuestas. Por ejemplo, si el término de consulta era *sailing dinghies*, la respuesta puede incluir `webPages`, `images` y `rankingResponse`. A menos que haya utilizado [responseFilter](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#responsefilter) para filtrar las páginas web, la respuesta incluye siempre las respuestas `webpages` y `rankingResponse`.

[!INCLUDE [cognitive-services-bing-url-note](../../../includes/cognitive-services-bing-url-note.md)]

## <a name="webpages-answer"></a>Respuesta de páginas web

La respuesta de [páginas web](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#webanswer) contiene una lista de vínculos a páginas web que para Bing Web Search eran apropiados para la consulta. Cada [página web](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#webpage) de la lista incluye el nombre de la página, la dirección URL, la URL de visualización, una descripción breve del contenido y la fecha en que Bing encontró el contenido de la página.

```json
{
    "id": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0",
    "name": "Dinghy sailing",
    "url": "https:\/\/www.bing.com\/cr?IG=3A43CA5...",
    "displayUrl": "https:\/\/en.contoso.com\/wiki\/Dinghy_sailing",
    "snippet": "Dinghy sailing is the activity of sailing small boats...",
    "dateLastCrawled": "2017-04-05T16:25:00"
}, ...
```

Use `name` y `url` para crear un hipervínculo que lleve al usuario a la página web.

<!-- Remove until this can be replaced with a sanitized version.
The following shows an example of how you might display the webpage in a search results page.

![Rendered webpage example](./media/cognitive-services-bing-web-api/bing-rendered-webpage-example.PNG)
-->

## <a name="images-answer"></a>Respuesta de imágenes

La respuesta de [imágenes](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#images) contiene una lista de imágenes que para Bing eran apropiadas para la consulta. Cada [imagen](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#image) de la lista incluye la dirección URL de la imagen, su tamaño, sus dimensiones y su formato de codificación. El objeto de imagen también incluye la dirección URL de una vista en miniatura de la imagen y las dimensiones de la miniatura.

```json
{
    "name": "Rich Passage Sailing Dinghy",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=3A43CA5CA64...",
    "thumbnailUrl": "https:\/\/tse1.mm.bing.net\/th?id=OIP....",
    "datePublished": "2011-10-29T11:26:00",
    "contentUrl": "http:\/\/upload.contoso.com\/sailing\/...",
    "hostPageUrl": "http:\/\/www.bing.com\/cr?IG=3A43CA5CA6464....",
    "contentSize": "79239 B",
    "encodingFormat": "jpeg",
    "hostPageDisplayUrl": "http:\/\/en.contoso.com\/wiki\/File...",
    "width": 526,
    "height": 688,
    "thumbnail": {
        "width": 229,
        "height": 300
    },
    "insightsSourcesSummary": {
        "shoppingSourcesCount": 0,
        "recipeSourcesCount": 0
    }
}, ...
```

En función del dispositivo del usuario, normalmente se muestra un subconjunto de las miniaturas con una opción para que el usuario vea las imágenes restantes [en la página](paging-webpages.md).

<!-- Remove until this can be replaced with a sanitized version.
![List of thumbnail images](./media/cognitive-services-bing-web-api/bing-web-image-thumbnails.PNG)
-->

También puede expandir la miniatura cuando el usuario desplace el cursor sobre la misma. No olvide establecer el atributo de la imagen si se expande. Por ejemplo, extrayendo el host de `hostPageDisplayUrl` y mostrándolo debajo de la imagen. Para obtener información acerca de cómo cambiar el tamaño de la miniatura, consulte [Cambio de tamaño y recorte de miniaturas](./resize-and-crop-thumbnails.md).

<!-- Remove until this can be replaced with a sanitized version.
![Expanded view of thumbnail image](./media/cognitive-services-bing-web-api/bing-web-image-thumbnail-expansion.PNG)
-->

Si el usuario hace clic en la miniatura, use `webSearchUrl` para llevar al usuario a la página de resultados de la búsqueda de Bing de imágenes, que contiene un collage de las imágenes.

Para más información sobre la respuesta de la imagen y las imágenes, consulte [Image Search API](../bing-image-search/search-the-web.md).

## <a name="related-searches-answer"></a>Respuesta de búsquedas relacionadas

La respuesta [relatedSearches](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#searchresponse-relatedsearches) contiene una lista de las consultas relacionadas más populares realizadas por otros usuarios. Cada [consulta](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#query_obj) de la lista incluye una cadena de consulta (`text`), una cadena de consulta con caracteres de resaltado de aciertos (`displayText`) y una dirección URL (`webSearchUrl`) a la página de resultados de búsqueda de Bing para esa consulta.

```json
{
    "text": "dinghy racing teams",
    "displayText": "dinghy racing teams",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=96C4CF214A0..."
}, ...
```

Utilice la cadena de consulta `displayText` y la dirección URL `webSearchUrl` para crear un hipervínculo que lleve al usuario a la página de resultados de búsqueda de Bing para la consulta relacionada. También puede usar la cadena de consulta `text` en su propia consulta de Web Search API y mostrar los resultados por sí mismo.

Para más información sobre cómo controlar los marcadores de resaltado en `displayText`, consulte [Resaltado de aciertos](../bing-web-search/hit-highlighting.md).

A continuación se muestra un ejemplo del uso de consultas relacionadas en Bing.com.

![Ejemplo de búsquedas relacionadas en Bing](./media/cognitive-services-bing-web-api/bing-web-rendered-relatedsearches.GIF)

## <a name="videos-answer"></a>Respuesta de vídeos

La respuesta de [vídeos](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#videos) contiene una lista de vídeos que para Bing eran apropiados para la consulta. Cada [vídeo](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#video) de la lista incluye la dirección URL del vídeo, su duración, sus dimensiones y su formato de codificación. El objeto de vídeo también incluye la dirección URL de una miniatura del vídeo y sus dimensiones.

```json
{
    "name": "Sailing dinghy",
    "description": "Northwind Traders is a 12 foot gunter rigged...",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=1CAE739681D84...",
    "thumbnailUrl": "https:\/\/tse2.mm.bing.net\/th?id=OVP.wsKiL...",
    "datePublished": "2013-11-06T01:56:28",
    "publisher": [{
        "name": "Fabrikam"
    }],
    "contentUrl": "https:\/\/www.fabrikam.com\/watch?v=MrVBWZpJjX",
    "hostPageUrl": "https:\/\/www.bing.com\/cr?IG=1CAE739681D8400DB...",
    "encodingFormat": "mp4",
    "hostPageDisplayUrl": "https:\/\/www.fabrikam.com\/watch?v=MrBWZpJjXo",
    "width": 1280,
    "height": 720,
    "duration": "PT3M47S",
    "motionThumbnailUrl": "https:\/\/tse2.mm.bing.net\/th?id=OM.oa...",
    "embedHtml": "<iframe width=\"1280\" height=\"720\" src=\"http:\/\/www....><\/iframe>",
    "allowHttpsEmbed": true,
    "viewCount": 19089,
    "thumbnail": {
        "width": 300,
        "height": 168
    },
    "allowMobileEmbed": true,
    "isSuperfresh": false
}, ...
```

En función del dispositivo del usuario, mostraría normalmente un subconjunto de los vídeos con una opción para que el usuario vea los vídeos restantes. Se mostrará una miniatura del vídeo con la duración del vídeo, su descripción (nombre) y su atribución (publicador).

<!-- Remove until this can be replaced with a sanitized version.
![List of video thumbnails](./media/cognitive-services-bing-web-api/bing-web-video-thumbnails.PNG)
-->

Según vaya el usuario desplazando el mouse sobre la miniatura, puede usar `motionThumbnailUrl` para reproducir una versión en miniatura del vídeo. No olvide atribuir la miniatura del movimiento al mostrarlo.

<!-- Remove until this can be replaced with a sanitized version.
![Motion thumbnail of a video](./media/cognitive-services-bing-web-api/bing-web-video-motion-thumbnail.PNG)
-->

Si el usuario hace clic en la miniatura, estas son las opciones de visualización del vídeo:

- Use `hostPageUrl` para ver el vídeo en el sitio web del host (por ejemplo, YouTube).
- Utilice `webSearchUrl` para ver el vídeo en el explorador de vídeos de Bing.
- Use `embedHtml` para insertar el vídeo en su propia experiencia.

Para más información acerca de la respuesta de vídeo y los vídeos, consulte [Video Search API](../bing-video-search/search-the-web.md).

## <a name="news-answer"></a>Respuesta de noticias

La respuesta de [noticias](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#news) contiene una lista de nuevos artículos que para Bing eran apropiados para la consulta. Cada [artículo de noticias](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#newsarticle) de la lista incluye el nombre del artículo, su descripción y su dirección URL al artículo en el sitio web del host. Si el artículo contiene una imagen, el objeto incluye una miniatura de la imagen.

```json
{
    "name": "WC Sailing Qualifies for America Trophy with...",
    "url": "http:\/\/www.bing.com\/cr?IG=3445EEF15DAF4FFFBF7...",
    "image": {
        "contentUrl": "http:\/\/www.contoso.com\/sports\/sail...",
        "thumbnail": {
            "contentUrl": "https:\/\/www.bing.com\/th?id=ON.1...",
            "width": 400,
            "height": 272
        }
    },
    "description": "The WC sailing team qualified for a...",
    "provider": [{
        "_type": "Organization",
        "name": "contoso.com"
    }],
    "datePublished": "2017-04-16T21:56:00"
}, ...
```

En función del dispositivo del usuario, mostraría un subconjunto de los artículos de noticias con una opción para que el usuario vea los artículos restantes. Use `name` y `url` para crear un hipervínculo que lleve al usuario al artículo de noticias en el sitio del host. Si el artículo incluye una imagen, asegúrese de que se pueda hacer clic en ella mediante `url`. Asegúrese de utilizar `provider` para atribuir el artículo.

<!-- Remove until this can be replaced with a sanitized version.
The following shows an example of how you might display articles in a search results page.

![List of news articles](./media/cognitive-services-bing-web-api/bing-web-news-list.PNG)
-->

Para más información acerca de la respuesta de noticias y artículos de noticias, consulte [News Search API](../bing-news-search/search-the-web.md).

## <a name="computation-answer"></a>Respuesta de cálculo

Si el usuario escribe una expresión matemática o una consulta de conversión de unidad, la respuesta puede contener una respuesta de [cálculo](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#computation). La respuesta `computation` contiene la expresión normalizada y su resultado.

Una consulta de conversión de unidad es una consulta que convierte una unidad en otra. ¿Por ejemplo, *¿Cuántos pies hay en 10 metros?* o *¿Cuántos cucharadas hay en 1/4 de taza?*

A continuación se muestra la respuesta `computation` a *¿Cuántos pies hay en 10 metros?*

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
    "expression": "10 meters",
    "value": "32.808399 feet"
}, ...
```

A continuación se muestran ejemplos de consultas matemáticas y sus respuestas `computation` correspondientes.

```
Query: (5+3)(10/2)+8
Encoded query: %285%2B3%29%2810%2F2%29%2B8
```

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
    "expression": "((5+3)*(10\/2))+8",
    "value": "48"
}
```

```
Query: sqrt(4^2+8^2)
Encoded query: sqrt%284^2%2B8^2%29
```

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
    "expression": "sqrt((4^2)+(8^2))",
    "value": "8.94427191"
}
```

```
Query: 30 6/8 - 18 8/16
Encoded query: 30%206%2F8%20-%2018%208%2F16
```

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#WolframAlpha",
    "expression": "30 6\/8-18 8\/16",
    "value": "12.25"
}
```

```
Query: 8^2+11^2-2*8*11*cos(37)
Encoded query: 8^2%2B11^2-2*8*11*cos%2837%29
```

```json
"computation": {
        "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
        "expression": "(8^2)+(11^2)-(2*8*11*cos(37))",
        "value": "44.4401502"
}
```

Una expresión matemática puede contener los siguientes símbolos:

|Símbolo|Descripción|
|------------|-----------------|
|+|Suma|
|-|Resta|
|/|División|
|*|Multiplicación|
|^|Power|
|!|Factorial|
|.|Decimal|
|()|Agrupación de precedencia|
|[]|Función|

Una expresión matemática puede contener las siguientes constantes:

|Símbolo|Descripción|
|------------|-----------------|
|Pi|3.14159...|
|Grado|Grado|
|i|Número imaginario|
|e|e, 2.71828...|
|GoldenRatio|Proporción áurea, 1.61803...|

Una expresión matemática puede contener las siguientes funciones:

|Símbolo|Descripción|
|------------|-----------------|
|Sort|Raíz cuadrada|
|Sen[x], Cos[x], Tan[x]<br />Csc[x], Sec[x], Cot[x]|Funciones trigonométricas (con argumentos en radianes)|
|ArcSin[x], ArcCos[x], ArcTan[x]<br />ArcCsc[x], ArcSec[x], ArcCot[x]|Funciones trigonométricas inversas (que proporcionan los resultados en radianes)|
|Exp[x], E^x|Función exponencial|
|Log[x]|Logaritmo natural|
|Sinh[x], Cosh[x], Tanh[x]<br />Csch[x], Sech[x], Coth[x]|Funciones hiperbólicas|
|ArcSinh[x], ArcCosh[x], ArcTanh[x]<br />ArcCsch[x], ArcSech[x], ArcCoth[x]|Funciones hiperbólicas inversas|

No se admiten expresiones matemáticas que contienen variables (por ejemplo, 4x+6 = 18, donde x es la variable).

## <a name="timezone-answer"></a>Respuesta de zonas horarias

Si el usuario escribe una consulta de fecha u hora, la respuesta puede contener una respuesta de [zona horaria](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#timezone). Esta respuesta es compatible con las consultas implícitas o explícitas. Una consulta implícita, como *¿Qué hora es?* , devuelve la hora local en función de la ubicación del usuario. Una consulta explícita, como *¿Qué hora es en Seattle?* , devuelve la hora local de Seattle, WA.

La respuesta `timeZone` proporciona el nombre de la ubicación, la fecha y hora UTC actuales en la ubicación especificada y la diferencia horaria con UTC. Si el límite de la ubicación está en varias zonas horarias, la respuesta contiene la fecha y hora UTC actuales de todas las zonas horarias dentro del límite. Por ejemplo, como el estado de Florida se encuentra dentro de dos zonas horarias, la respuesta contiene la fecha y hora locales de ambas zonas.  

Si la consulta solicita la hora de un estado, o país o región, Bing determina la ciudad principal dentro del límite geográfico de la ubicación y la devuelve en el campo `primaryCityTime`. Si el límite contiene varias zonas horarias,las restantes zonas horarias se devuelven en el campo `otherCityTimes`.

A continuación se muestran consultas de ejemplo que devuelven la respuesta `timeZone`.

```
Query: What time is it?

"timeZone": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
    "primaryCityTime": {
        "location": "Redmond, Washington, United States",
        "time": "2015-10-27T08:38:12.1189231Z",
        "utcOffset": "UTC-7"
    }
}

Query: What time is it in the Pacific time zone?

"timeZone": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
    "primaryCityTime": {
        "location": "Pacific Time Zone",
        "time": "2015-10-23T12:33:19.0728146Z",
        "utcOffset": "UTC-7"
    }
}

Query: Time in Florida?

"timeZone": {
        "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
        "primaryCityTime": {
            "location": "Tallahassee, Florida, United States",
            "time": "2015-10-23T13:04:56.6774389Z",
            "utcOffset": "UTC-4"
        },
        "otherCityTimes": [{
            "location": "Pensacola",
            "time": "2015-10-23T12:04:56.6664294Z",
            "utcOffset": "UTC-5"
        }]
}

Query: What time is it in the U.S.

"timeZone": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
    "primaryCityTime": {
        "location": "Washington, D.C., United States",
        "time": "2015-10-23T15:27:59.8892745Z",
        "utcOffset": "UTC-4"
    },
    "otherCityTimes": [{
        "location": "Honolulu",
        "time": "2015-10-23T09:27:59.8892745Z",
        "utcOffset": "UTC-10"
    },
    {
        "location": "Anchorage",
        "time": "2015-10-23T11:27:59.8892745Z",
        "utcOffset": "UTC-8"
    },
    {
        "location": "Phoenix",
        "time": "2015-10-23T12:27:59.8892745Z",
        "utcOffset": "UTC-7"
    },
    {
        "location": "Los Angeles",
        "time": "2015-10-23T12:27:59.8942788Z",
        "utcOffset": "UTC-7"
    },
    {
        "location": "Denver",
        "time": "2015-10-23T13:27:59.8812681Z",
        "utcOffset": "UTC-6"
    },
    {
        "location": "Chicago",
        "time": "2015-10-23T14:27:59.8892745Z",
        "utcOffset": "UTC-5"
    }]
}
```

## <a name="spellsuggestion-answer"></a>Respuesta de sugerencias ortográficas

Si Bing determina que el usuario tenía la intención de buscar algo diferente, la respuesta incluye un objeto [SpellSuggestions](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#spellsuggestions). Por ejemplo, si el usuario busca *carlos pen*, Bing puede determinar que el usuario pretende probablemente en su lugar buscar Carlos Pena (en función de las búsquedas anteriores realizadas por otros usuarios de *carlos pen*). A continuación se muestra un ejemplo de respuesta de sugerencia ortográfica.

```json
"spellSuggestions": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#SpellSuggestions",
    "value": [{
        "text": "carlos pena",
        "displayText": "carlos pena"
    }]
}, ...
```

## <a name="response-headers"></a>Encabezados de respuesta

Las respuestas de Bing Web Search API pueden tener los encabezados siguientes:

| Encabezado | Descripción |
|-|-|
|`X-MSEdge-ClientID`|El identificador único que Bing ha asignado al usuario|
|`BingAPIs-Market`|El mercado que se usó para cumplir la solicitud|
|`BingAPIs-TraceId`|La entrada de registro en el servidor de Bing API para esta solicitud (para soporte técnico)|

Es especialmente importante conservar el identificador de cliente y devolverlo con las sucesivas solicitudes. Al hacerlo, la búsqueda usa el contexto pasado en los resultados de búsqueda de clasificación y también se proporciona una experiencia coherente al usuario.

Sin embargo, cuando se llama a Bing Web Search API desde JavaScript, las características de seguridad integradas de su explorador (CORS) podrían impedirle acceder a los valores de estos encabezados.

Para acceder a los encabezados, puede realizar la solicitud de Bing Web Search API mediante un proxy CORS. La respuesta de un proxy de este tipo tiene un encabezado `Access-Control-Expose-Headers` que filtra los encabezados de respuesta y hace que estén disponibles para JavaScript.

Es fácil instalar un proxy CORS para permitir que nuestra [aplicación de tutorial](tutorial-bing-web-search-single-page-app.md) obtenga acceso a los encabezados de cliente opcionales. En primer lugar, si aún no lo tiene, [instale Node.js](https://nodejs.org/en/download/). A continuación, escriba el siguiente comando en un símbolo del sistema.

```console
npm install -g cors-proxy-server
```

Luego, cambie el punto de conexión de Bing Web Search API del archivo HTML a:\
`http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/search`

Por último, inicie el proxy CORS con el siguiente comando:

```console
cors-proxy-server
```

Deje abierta la ventana de comandos mientras usa la aplicación del tutorial, ya que si la cierra, se detendrá el proxy. En la sección de encabezados HTTP expandibles situada bajo los resultados de la búsqueda, puede ver el encabezado `X-MSEdge-ClientID` (entre otras cosas) y comprobar que es el mismo en todas las solicitudes.

## <a name="response-headers-in-production"></a>Encabezados de respuesta en producción

El enfoque de proxy CORS que se describe en la respuesta anterior es adecuado para desarrollo, pruebas y aprendizaje.

Sin embargo, en un entorno de producción, debe hospedar un script del lado servidor en el mismo dominio que la página web que usa Bing Web Search API. Este script debe realizar llamadas API tras la solicitud del código de JavaScript de la página web y pasar nuevamente al cliente todos los resultados, incluidos los encabezados. Dado que los dos recursos (página y script) comparten un origen, no se usa CORS y JavaScript puede obtener acceso a los encabezados especiales en la página web.

Este enfoque también protege la clave de API de una exposición pública, ya que solo la necesita el script del lado servidor. El script puede usar otro método para asegurarse de que la solicitud está autorizada.

A continuación se muestra cómo utiliza Bing la sugerencia ortográfica.

![Ejemplo de sugerencia ortográfica de Bing](./media/cognitive-services-bing-web-api/bing-web-spellingsuggestion.GIF)  

## <a name="next-steps"></a>Pasos siguientes  

* Revisión de la documentación de [limitación de solicitudes](throttling-requests.md).  

## <a name="see-also"></a>Consulte también  

* [Referencia de Bing Web Search API](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference)
