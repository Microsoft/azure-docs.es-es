---
title: Envío y uso de consultas y respuestas de Bing Local Business Search API | Microsoft Docs
titleSuffix: Azure Cognitive Services
description: Use este artículo para obtener información sobre cómo enviar y usar consultas con Bing Local Business Search API.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: article
ms.date: 11/01/2018
ms.author: rosh; v-gedod
ms.openlocfilehash: 45f7adab684b7b6097328a42abb60c8d58c1acc2
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796792"
---
# <a name="sending-and-using-bing-local-business-search-api-queries-and-responses"></a>Envío y uso de consultas y respuestas de Bing Local Business Search API

Puede obtener resultados locales de Bing Local Business Search API si envía una consulta de búsqueda a su punto de conexión e incluye el encabezado `Ocp-Apim-Subscription-Key`, que es obligatorio. Junto con los [encabezados](local-search-reference.md#headers) y los [parámetros](local-search-reference.md#query-parameters) disponibles, las búsquedas se pueden personalizar mediante la especificación de [límites geográficos](specify-geographic-search.md) para el área de búsqueda y las [categorías](local-search-query-response.md) de lugares devueltos.

## <a name="creating-a-request"></a>Creación de una solicitud

Para enviar una solicitud a Bing Local Business Search API, anexe un término de búsqueda al parámetro `q=` antes de que se agregue al punto de conexión de API e incluya el encabezado `Ocp-Apim-Subscription-Key`. Por ejemplo:

`https://api.cognitive.microsoft.com/bing/localbusinesses/v7.0/search?q=restaurant+in+Bellevue`

A continuación se muestra la sintaxis de la dirección URL de solicitud completa. Consulte las [guías de inicio rápido](quickstarts/local-quickstart.md) de Bing Local Business Search API y el contenido de referencia para [encabezados](local-search-reference.md#headers) y [parámetros](local-search-reference.md#query-parameters) para más información sobre el envío de solicitudes. 

Para obtener información sobre las categorías de búsqueda local, consulte [Categorías de búsqueda para Bing Local Business Search API](local-categories.md).

```
https://api.cognitive.microsoft.com/bing/v7.0/localbusinesses/search[?q][&localCategories][&cc][&mkt][&safesearch][&setlang][&count][&first][&localCircularView][&localMapView]
```

## <a name="using-responses"></a>Uso de respuestas

Las respuestas JSON de Bing Local Business Search API contienen un objeto `SearchResponse`. La API devolverá resultados de búsqueda pertinentes en el campo `places`. Si no se encuentra ningún resultado, el campo `places` no se incluirá en la respuesta.

```
{
   "_type": "SearchResponse",
   "queryContext": {
      "originalQuery": "restaurant in Bellevue"
   },
   "places": {
      "totalEstimatedMatches": 10,
. . . 
```

### <a name="search-result-attributes"></a>Atributos de resultados de búsqueda

Los resultados JSON devueltos por la API incluyen los siguientes atributos:

* _type
* dirección
* entityPresentationInfo
* geoárea
* id
* Nombre
* routeablePoint
* telephone
* url

Para obtener información general sobre encabezados, parámetros, códigos de mercado, objetos de respuesta, errores etc., consulte la referencia [Bing Local Search API v7](local-search-reference.md).

> [!NOTE]
> Tenga en cuenta que ni usted, ni un tercero en su nombre, pueden usar, retener, guardar, almacenar en caché, compartir o distribuir datos de Local Search API con el fin de probar, desarrollar, entrenar, distribuir o poner a disposición cualquier servicio o característica que no sea de Microsoft. 


## <a name="example-json-response"></a>Ejemplo de respuesta JSON

La siguiente respuesta JSON incluye los resultados de búsqueda especificados por la consulta `?q=restaurant+in+Bellevue`.

```json
Vary: Accept-Encoding
BingAPIs-TraceId: 5376FFEB65294E24BB9F91AD70545826
BingAPIs-SessionId: 06ED7CEC80F746AA892EDAAC97CB0CB4
X-MSEdge-ClientID: 112C391E72C0624204153594738C63DE
X-MSAPI-UserState: aeab
BingAPIs-Market: en-US
X-Search-ResponseInfo: InternalResponseTime=659,MSDatacenter=CO4
X-MSEdge-Ref: Ref A: 5376FFEB65294E24BB9F91AD70545826 Ref B: BY3EDGE0306 Ref C: 2018-10-16T16:26:15Z
apim-request-id: fe54f585-7c54-4bf5-8b92-b9bede2b710a
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
x-content-type-options: nosniff
Cache-Control: max-age=0, private
Date: Tue, 16 Oct 2018 16:26:15 GMT
P3P: CP="NON UNI COM NAV STA LOC CURa DEVa PSAa PSDa OUR IND"
Content-Length: 978
Content-Type: application/json; charset=utf-8
Expires: Tue, 16 Oct 2018 16:25:15 GMT

{
  "_type": "SearchResponse",
  "queryContext": {
    "originalQuery": "restaurant Bellevue"
  },
  "places": {
    "totalEstimatedMatches": 50,
    "value": [{
      "_type": "LocalBusiness",
      "id": "https:\/\/cognitivegblppe.azure-api.net\/api\/v7\/#Places.0",
      "name": "Facing East Taiwanese Restaurant",
      "url": "http:\/\/litadesign.wix.com\/facingeastrestaurant",
      "entityPresentationInfo": {
        "entityScenario": "ListItem",
        "entityTypeHints": ["Place", "LocalBusiness", "Restaurant"]
      },
      "geo": {
        "latitude": 47.6199188232422,
        "longitude": -122.202796936035
      },
      "routablePoint": {
        "latitude": 47.6199188232422,
        "longitude": -122.201713562012
      },
      "address": {
        "streetAddress": "1075 Bellevue Way NE Ste B2",
        "addressLocality": "Bellevue",
        "addressRegion": "WA",
        "postalCode": "98004",
        "addressCountry": "US",
        "neighborhood": "Bellevue",
        "text": "1075 Bellevue Way NE Ste B2, Bellevue, WA 98004"
      },
      "telephone": "(425) 688-2986"
    }],
    "searchAction": {
      "location": [{
        "name": "Bellevue, Washington"
      }],
      "query": "restaurant"
    }
  }
}
 
```

## <a name="throttling-requests"></a>Solicitudes de limitación

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../includes/cognitive-services-bing-throttling-requests.md)]


## <a name="next-steps"></a>Pasos siguientes
- [Guía de inicio rápido de Local Business Search](quickstarts/local-quickstart.md)
- [Guía de inicio rápido de Local Business Search para Java](quickstarts/local-search-java-quickstart.md)
- [Guía de inicio rápido de Local Business Search para Node](quickstarts/local-search-node-quickstart.md)
- [Guía de inicio rápido de Local Business Search para Python](quickstarts/local-search-python-quickstart.md)
