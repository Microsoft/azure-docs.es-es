---
title: 'Inicio rápido: Envío de una consulta a la API mediante Java: Bing Local Business Search'
titleSuffix: Azure Cognitive Services
description: Use esta guía de inicio rápido para empezar a enviar solicitudes a Bing Local Business Search API, que es un servicio de Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-local-business
ms.topic: quickstart
ms.date: 05/12/2020
ms.custom: devx-track-java
ms.author: aahi
ms.openlocfilehash: 8780caae20b779997e32750ed7829afd7d969a02
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320245"
---
# <a name="quickstart-send-a-query-to-the-bing-local-business-search-api-using-java"></a>Inicio rápido: Envío de una consulta a Bing Local Business Search API mediante Java

Use este inicio rápido para aprender a enviar solicitudes a la API Bing Local Business Search, que es un servicio de Azure Cognitive Services. Aunque esta aplicación sencilla está escrita en Java, la API es un servicio web RESTful compatible con cualquier lenguaje de programación que pueda realizar solicitudes HTTP y analizar JSON.

Esta aplicación de ejemplo obtiene los datos de respuesta local de la API para una consulta de búsqueda.

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* El [kit de desarrollo de Java (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="cree un recurso de Bing Search"  target="_blank">crear un recurso de Bing Search<span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.

## <a name="create-the-request"></a>Creación de la solicitud 

El siguiente código crea una instancia de `WebRequest`, establece el encabezado de la clave de acceso y agrega una cadena de consulta para *hotel in Bellevue*.  Después envía la solicitud y asigna la respuesta a una cadena que contiene el texto JSON.

```java
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

El código siguiente usa la API Bing Local Business Search para devolver los resultados de búsqueda del motor de búsqueda de Bing. Ejecute este código siguiendo estos pasos:
1. Descargue o instale la biblioteca gson.
2. Cree un nuevo proyecto de Java en su IDE o editor favorito.
3. Agregue el código que se proporciona a continuación.
4. Reemplace el valor `subscriptionKey` por una clave de acceso válida para la suscripción.
5. Ejecute el programa.

```java
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
- [Inicio rápido de Local Business Search para C#](local-quickstart.md)
- [Inicio rápido de Local Business Search para Node](local-search-node-quickstart.md)
- [Guía de inicio rápido de Local Business Search para Python](local-search-python-quickstart.md)
