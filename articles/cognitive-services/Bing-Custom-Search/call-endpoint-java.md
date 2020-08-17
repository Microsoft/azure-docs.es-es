---
title: 'Inicio rápido: Llamada al punto de conexión de Bing Custom Search con Java | Microsoft Docs'
titleSuffix: Azure Cognitive Services
description: Use este documento de inicio rápido para comenzar a solicitar los resultados de búsqueda de la instancia de Bing Custom Search en Java.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: quickstart
ms.date: 05/08/2020
ms.custom: devx-track-java
ms.author: aahi
ms.openlocfilehash: 069f3b95d76d8321ce0b1bba4ae61f35d0f14a57
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326790"
---
# <a name="quickstart-call-your-bing-custom-search-endpoint-using-java"></a>Inicio rápido: Llamada al punto de conexión de Bing Custom Search con Java

Use este inicio rápido para aprender cómo solicitar los resultados de búsqueda de la instancia de Bing Custom Search. Aunque esta aplicación está escrita en Java, Bing Custom Search API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. El código fuente del ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/java/Search/BingCustomSearchv7.java).

## <a name="prerequisites"></a>Prerrequisitos

- Una instancia de Bing Custom Search. Para más información, consulte [Inicio rápido: Creación de la primera instancia de Bing Custom Search](quick-start.md).

- La versión más reciente del [Kit de desarrollo de Java](https://www.oracle.com/technetwork/java/javase/downloads/index.html).

- La [biblioteca Gson](https://github.com/google/gson).

[!INCLUDE [cognitive-services-bing-custom-search-prerequisites](../../../includes/cognitive-services-bing-custom-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

1. Cree un proyecto de Java en su IDE o editor favoritos e importe las bibliotecas siguientes:

    ```java
    import java.io.InputStream;
    import java.net.URL;
    import java.net.URLEncoder;
    import java.util.HashMap;
    import java.util.List;
    import java.util.Map;
    import java.util.Scanner;
    import javax.net.ssl.HttpsURLConnection;
    import com.google.gson.Gson;
    import com.google.gson.GsonBuilder;
    import com.google.gson.JsonObject;
    import com.google.gson.JsonParser;
    ```

2. Cree una clase llamada `CustomSrchJava` y luego cree variables para la clave de suscripción, el punto de conexión de búsqueda personalizada y el identificador de configuración personalizada de la instancia de búsqueda. Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.
    ```java
    public class CustomSrchJava {
        static String host = "https://api.cognitive.microsoft.com";
        static String path = "/bingcustomsearch/v7.0/search";
        static String subscriptionKey = "YOUR-SUBSCRIPTION-KEY"; 
        static String customConfigId = "YOUR-CUSTOM-CONFIG-ID";
        static String searchTerm = "Microsoft";  
    ...
    ```

3. Cree otra clase denominada `SearchResults` en la que se incluirá la respuesta de la instancia de Bing Custom Search.

    ```java
    class SearchResults {
        HashMap<String, String> relevantHeaders;
        String jsonResponse;
        SearchResults(HashMap<String, String> headers, String json) {
            relevantHeaders = headers;
            jsonResponse = json;
        }
    }
    ```

4. Cree una función denominada `prettify()` para dar formato a la respuesta JSON de Bing Custom Search API.

    ```java
        // pretty-printer for JSON; uses GSON parser to parse and re-serialize
        public static String prettify(String json_text) {
            JsonParser parser = new JsonParser();
            JsonObject json = parser.parse(json_text).getAsJsonObject();
            Gson gson = new GsonBuilder().setPrettyPrinting().create();
            return gson.toJson(json);
        }
    ```

## <a name="send-and-receive-a-search-request"></a>Envío y recepción de una solicitud de búsqueda 

1. Cree una función denominada `SearchWeb()` que envíe una solicitud y devuelva un objeto `SearchResults`. Cree la dirección URL de la solicitud combinando la información del identificador de configuración personalizada, la consulta y el punto de conexión. Agregue la clave de suscripción al encabezado `Ocp-Apim-Subscription-Key`.

    ```java
    public class CustomSrchJava {
    ...
        public static SearchResults SearchWeb (String searchQuery) throws Exception {
            // construct the URL for your search request (endpoint + query string)
            URL url = new URL(host + path + "?q=" +  URLEncoder.encode(searchTerm, "UTF-8") + "&CustomConfig=" + customConfigId);
            HttpsURLConnection connection = (HttpsURLConnection)url.openConnection();
            connection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);
    ...
    ```

2. Cree una secuencia y almacene la respuesta JSON en un objeto `SearchResults`.

    ```java
    public class CustomSrchJava {
    ...
        public static SearchResults SearchWeb (String searchQuery) throws Exception {
            ...
            // receive the JSON body
            InputStream stream = connection.getInputStream();
            String response = new Scanner(stream).useDelimiter("\\A").next();
        
            // construct result object for return
            SearchResults results = new SearchResults(new HashMap<String, String>(), response);
            
            stream.close();
            return results;
        }
    ```

3. Imprima la respuesta JSON.

    ```java
    System.out.println("\nJSON Response:\n");
    System.out.println(prettify(result.jsonResponse));
    ```

4. Ejecute el programa.
    
## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una página web de Custom Search](./tutorials/custom-search-web-page.md)
