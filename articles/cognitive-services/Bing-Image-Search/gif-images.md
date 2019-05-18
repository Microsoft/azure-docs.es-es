---
title: Búsqueda de imágenes GIF con Bing Image Search API
titleSuffix: Azure Cognitive Services
description: Use Bing Image Search API para buscar imágenes .gif en Internet.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: article
ms.date: 04/24/2018
ms.author: aahi
ms.custom: seodec2018
ms.openlocfilehash: 4b396c3b99c143f20caa774ee3c4c6ee25f96150
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65777116"
---
# <a name="search-for-gif-images"></a>Búsqueda de imágenes GIF 

Bing Image Search API le permite buscar en toda la Red la imágenes .gif más relevantes.  Los desarrolladores pueden integrar gifs atractivos en varios casos de conversación. 

La siguiente dirección URL es una consulta para las imágenes animadas en .gif.
```
https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=interesting&imageType=AnimatedGif&mkt=en-us
```
El parámetro [q](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#query) especifica los términos de la búsqueda.  En la consulta anterior también se especifica `animatedGif`, mediante el parámetro de filtro [imageType](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imagetype).

Para ver ejemplos de los resultados, use la siguiente dirección URL para buscar en bing.com.
```
https://www.bing.com/images/search?q=interesting&qft=%20filterui%3Aphoto-animatedgif

```
## <a name="query-parameters"></a>Parámetros de consulta

Para obtener más información sobre los parámetros de consulta y las opciones, consulte la [referencia de Image Search API](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#query-parameters). También podrá ver un ejemplo bajo el encabezado de la sección[Ejemplo de búsqueda de gif animados mediante Java](#gifExample).

## <a name="tips-and-suggestions"></a>Recomendaciones y sugerencias

- Puede especificar los parámetros [maxFileSize](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#maxfilesize) y [minFileSize](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#minfilesize). Le recomendamos establecer maxFileSize=2000000, ya que la mayoría de los gifs en nuestro índice tienen menos de 2 MB.  Esta opción también le permitirá controlar el tamaño de los datos si el ancho de banda es un problema, como suele suceder con los dispositivos móviles.
- Con el fin de mejorar el rendimiento percibido, cargue la miniatura antes de cargar la dirección URL de origen.  
- Para realizar la primera ejecución o poder obtener acceso a la página de aterrizaje donde todavía no tiene una consulta de usuario, intente utilizar nuestras búsquedas de novedades en gif para obtener ayuda de la [API de novedades en imágenes](trending-images.md).
- Hay tres configuraciones para el parámetro [safeSearch](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#safesearch).  La opción `strict` bloquea el contenido para adultos.
- Consulte [mkt](supported-countries-markets.md) para obtener la lista completa de idiomas y ubicaciones compatibles.
- *AnimatedGifHttps* solo devuelve imágenes gif animadas que provienen de una dirección HTTPS. Por seguridad, muchas aplicaciones deben conectarse a vínculos web externos a través del protocolo HTTPS. Por ejemplo, Apple App Store debe conectarse a servicios web a través de HTTPS, para poder encriptar los datos del usuario de forma segura durante el tránsito.

<a name="gifExample" />

## <a name="example-search-for-animated-gif-using-java"></a>Ejemplo de búsqueda de gif animados mediante Java

La siguiente dirección URL busca imágenes animadas en .gif: `q=interesting`
```
https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=interesting&imageType=AnimatedGif&mkt=en-us

```
Tal como se muestra en el siguiente ejemplo, la consulta de la dirección URL requiere el encabezado [Ocp-Apim-Subscription-Key](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#headers).

En el siguiente ejemplo de Java, se crea y se envía la solicitud.

```
package gifSearch;
import java.net.*;
import java.util.*;
import java.io.*;
import javax.net.ssl.HttpsURLConnection;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;

/*
 * Gson: https://github.com/google/gson
 * Maven info:
 *     groupId: com.google.code.gson
 *     artifactId: gson
 *     version: 2.8.1
 *
 * Once you have compiled or downloaded gson-2.8.1.jar, assuming you have placed it in the
 * same folder as this file (BingImageSearch.java), you can compile and run this program at
 * the command line as follows.
 *
 * javac GIFsearch.java -classpath .;gson-2.8.1.jar -encoding UTF-8
 * java -cp .;gson-2.8.1.jar GIFsearch
 */


public class GIFsearch {

    // Replace the subscriptionKey string value with your valid subscription key.
    static String subscriptionKey = "YOUR-ACCESS-KEY";

    static String host = "https://api.cognitive.microsoft.com";
    static String path = "/bing/v7.0/images/search";

    static String searchTerm = "interesting";

    public static SearchResults SearchImages (String searchQuery) throws Exception {
        // construct URL of search request (endpoint + query string)
        URL url = new URL(host + path + "?q=" +  URLEncoder.encode(searchQuery, "UTF-8") + "&imageType=AnimatedGif&mkt=en-us");
        HttpsURLConnection connection = (HttpsURLConnection)url.openConnection();
        connection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);

        // receive JSON body
        InputStream stream = connection.getInputStream();
        String response = new Scanner(stream).useDelimiter("\\A").next();

        // construct result object for return
        SearchResults results = new SearchResults(new HashMap<String, String>(), response);

        // extract Bing-related HTTP headers
        Map<String, List<String>> headers = connection.getHeaderFields();
        for (String header : headers.keySet()) {
            if (header == null) continue;      // may have null key
            if (header.startsWith("BingAPIs-") || header.startsWith("X-MSEdge-")) {
                results.relevantHeaders.put(header, headers.get(header).get(0));
            }
        }

        stream.close();
        return results;
    }

    // pretty-printer for JSON; uses GSON parser to parse and re-serialize
    public static String prettify(String json_text) {
        JsonParser parser = new JsonParser();
        JsonObject json = parser.parse(json_text).getAsJsonObject();
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        return gson.toJson(json);
    }

    public static void main (String[] args) {
        if (subscriptionKey.length() != 32) {
            System.out.println("Invalid Bing Search API subscription key!");
            System.out.println("Please paste yours into the source code.");
            System.exit(1);
        }

        try {
            System.out.println("Searching the Web for: " + searchTerm);

            SearchResults result = SearchImages(searchTerm);

            System.out.println("\nRelevant HTTP Headers:\n");
            for (String header : result.relevantHeaders.keySet())
                System.out.println(header + ": " + result.relevantHeaders.get(header));

            System.out.println("\nJSON Response:\n");
            System.out.println(prettify(result.jsonResponse));
        }
        catch (Exception e) {
            e.printStackTrace(System.out);
            System.exit(1);
        }
    }

}

//Container class for search results encapsulates relevant headers and JSON data
class SearchResults{
 HashMap<String, String> relevantHeaders;
 String jsonResponse;
 SearchResults(HashMap<String, String> headers, String json) {
     relevantHeaders = headers;
     jsonResponse = json;
 }
}

```

## <a name="results"></a>Resultados
El código obtiene los siguientes resultados como objetos JSON:

```json
    {
      "webSearchUrl": "https://www.bing.com/images/search?view\u003ddetai...",
      "name": "Very Interesting GIF - Thats Very Interesting - ...",
      "thumbnailUrl": "https://tse1.mm.bing.net/th?id\u003dOIP.yJX6Vz345JPK...",
      "datePublished": "2017-03-12T01:35:00.0000000Z",
      "contentUrl": "https://media.contoso.co/images/c895fa573df8e493ca8d0dec7d93b/raw",
      "hostPageUrl": "https://www.contoso.co/view/thats-very-interesting-christi...",
      "contentSize": "1295633 B",
      "encodingFormat": "animatedgif",
      "hostPageDisplayUrl": "https://www.contoso.co/view/thats-very-christian...",
      "width": 440,
      "height": 186,
      "thumbnail": {
        "width": 474,
        "height": 200
      },
      "imageInsightsToken": "ccid_yJX6Vz34*mid_9FF0FFA42AADA1357F042443D2103B40EA...",
      "insightsMetadata": {
        "recipeSourcesCount": 0,
        "bestRepresentativeQuery": {
          "text": "That\u0027s Very Interesting",
          "displayText": "That\u0027s Very Interesting",
          "webSearchUrl": "https://www.bing.com/images/search?q\u003dThat..."
        },
        "pagesIncludingCount": 19,
        "availableSizesCount": 2
      },
      "imageId": "9FF0FFA42AADA1357F042443D21030EAAA225F",
      "accentColor": "62452D"
    },

```

## <a name="next-steps"></a>Pasos siguientes
- [Inicio rápido de C#](quickstarts/csharp.md)
- [Tutorial de aplicación de una sola página de Image Search](tutorial-bing-image-search-single-page-app.md)
