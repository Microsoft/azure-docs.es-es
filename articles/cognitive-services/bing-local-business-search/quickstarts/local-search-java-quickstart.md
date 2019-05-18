---
title: 'Guía de inicio rápido: Envío de una consulta a Bing Local Business Search API mediante Java | Microsoft Docs'
titleSuffix: Azure Cognitive Services
description: Use este artículo para empezar a usar Bing Local Business Search API en Java.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: article
ms.date: 11/01/2018
ms.author: rosh, v-gedod
ms.openlocfilehash: b87fea5120ae09848eef9fcb29ef0912bf17fa0f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796915"
---
# <a name="quickstart-send-a-query-to-the-bing-local-business-search-api-using-java"></a>Inicio rápido: Envío de una consulta a Bing Local Business Search API mediante Java

Use esta guía de inicio rápido para empezar a enviar solicitudes a Bing Local Business Search API, que es un servicio de Azure Cognitive Services. Si bien esta aplicación sencilla está escrita en Java, la API es un servicio web RESTful compatible con cualquier lenguaje de programación que pueda realizar solicitudes HTTP y analizar JSON.

Esta aplicación de ejemplo obtiene los datos de respuesta local de la API para la consulta de búsqueda `hotel in Bellevue`.

## <a name="prerequisites"></a>Requisitos previos

* [Kit de desarrollo de Java (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)

Debe tener una [cuenta de Cognitive Services API](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) con Bing Search APIs. La [cuenta de evaluación gratuita](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api) es suficiente para esta guía de inicio rápido. Necesitará la clave de acceso proporcionada al activar la evaluación gratuita.  Consulte también [Precios de Cognitive Services - Bing Search API](https://azure.microsoft.com/pricing/details/cognitive-services/search-api/).

Esta aplicación de ejemplo obtiene los datos de respuesta local de la consulta de un *hotel en Bellevue*.

## <a name="create-the-request"></a>Creación de la solicitud 

El siguiente código crea una instancia de `WebRequest`, establece el encabezado de la clave de acceso y agrega una cadena de consulta para "hotel en Bellevue".  Después envía la solicitud y asigna la respuesta a una cadena que contiene el texto JSON.

```
    // construct URL of search request (endpoint + query string)
     URL url = new URL(host + path + "?q=" +  URLEncoder.encode(searchQuery, "UTF-8") + &mkt=en-us");
    HttpsURLConnection connection = (HttpsURLConnection)url.openConnection();
    connection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);

    // receive JSON body
    InputStream stream = connection.getInputStream();
    String response = new Scanner(stream).useDelimiter("\\A").next();

    // construct result object for return
    SearchResults results = new SearchResults(new HashMap<String, String>(), response);
```

## <a name="run-the-complete-application"></a>Ejecución de la aplicación completa

Bing Local Business Search API devuelve resultados del motor de búsqueda de Bing.
1. Descargue o instale la biblioteca gson.
2. Cree un proyecto de Java en su IDE o editor favorito.
3. Agregue el código que se le ha proporcionado a continuación.
4. Reemplace el valor subscriptionKey por una clave de acceso válida para la suscripción.
5. Ejecute el programa.

```
package localSearch;
import java.net.*;
import java.util.*;
import java.io.*;
import javax.net.ssl.HttpsURLConnection;

/*
 * Gson: https://github.com/google/gson
 * Maven info:
 *     groupId: com.google.code.gson
 *     artifactId: gson
 *     version: 2.8.1
 *
 * Once you have compiled or downloaded gson-2.8.1.jar, assuming you have placed it in the
 * same folder as this file (localSearch.java), you can compile and run this program at
 * the command line as follows.
 *
 * javac localSearch.java -classpath .;gson-2.8.1.jar -encoding UTF-8
 * java -cp .;gson-2.8.1.jar localSearch
 */
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;

public class LocalSearchCls {

    // ***********************************************
    // *** Update or verify the following values. ***
    // **********************************************

        // Replace the subscriptionKey string value with your valid subscription key.
        static String subscriptionKey = "YOUR-ACCESS-KEY";

        static String host = "https://api.cognitive.microsoft.com/bing";
        static String path = "/v7.0/localbusinesses/search";

        static String searchTerm = "Hotel in Bellevue";

        public static SearchResults SearchLocal (String searchQuery) throws Exception {
            // construct URL of search request (endpoint + query string)
            URL url = new URL(host + path + "?q=" +  URLEncoder.encode(searchQuery, "UTF-8") + "&mkt=en-us");
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
            try {
                System.out.println("Searching the Web for: " + searchTerm);

                SearchResults result = SearchLocal(searchTerm);

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

    // Container class for search results encapsulates relevant headers and JSON data
    class SearchResults{
        HashMap<String, String> relevantHeaders;
        String jsonResponse;
        SearchResults(HashMap<String, String> headers, String json) {
            relevantHeaders = headers;
            jsonResponse = json;
        }
    }

```

## <a name="next-steps"></a>Pasos siguientes
- [Guía de inicio rápido de Local Business Search](local-quickstart.md)
- [Guía de inicio rápido de Local Business Search con Node](local-search-node-quickstart.md)
- [Guía de inicio rápido de Local Business Search para Python](local-search-python-quickstart.md)
