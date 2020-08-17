---
title: 'Inicio rápido: Envío de una solicitud de búsqueda a la API REST mediante Java: Bing Entity Search'
titleSuffix: Azure Cognitive Services
description: Use este inicio rápido para enviar una solicitud a Bing Entity Search REST API mediante Java y reciba una respuesta JSON.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: quickstart
ms.date: 05/08/2020
ms.custom: devx-track-java
ms.author: aahi
ms.openlocfilehash: 3afc19e93e400894a66e659d79d834b8a83132a2
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319293"
---
# <a name="quickstart-send-a-search-request-to-the-bing-entity-search-rest-api-using-java"></a>Inicio rápido: Envío de una solicitud de búsqueda a Bing Entity Search REST API con Java

Use este inicio rápido para realizar la primera llamada a Bing Entity Search API y ver la respuesta JSON. Esta sencilla aplicación de Java envía una consulta de búsqueda de noticias a la API y muestra la respuesta.

Aunque esta aplicación está escrita en Java, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

## <a name="prerequisites"></a>Prerrequisitos

* El [kit de desarrollo de Java (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/).
* La [biblioteca Gson](https://github.com/google/gson).


[!INCLUDE [cognitive-services-bing-news-search-signup-requirements](../../../../includes/cognitive-services-bing-entity-search-signup-requirements.md)]

## <a name="create-and-initialize-a-project"></a>Creación e inicialización de un proyecto

1. Cree un proyecto de Java en su IDE o editor favoritos e importe las bibliotecas siguientes:

   ```java
   import java.io.*;
   import java.net.*;
   import java.util.*;
   import javax.net.ssl.HttpsURLConnection;
   import com.google.gson.Gson;
   import com.google.gson.GsonBuilder;
   import com.google.gson.JsonObject;
   import com.google.gson.JsonParser;
   import com.google.gson.Gson;
   import com.google.gson.GsonBuilder;
   import com.google.gson.JsonObject;
   import com.google.gson.JsonParser;
   ```

2. En una nueva clase, cree variables para el punto de conexión de la API, la clave de suscripción y el término de búsqueda. Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

   ```java
   public class EntitySearch {

      static String subscriptionKey = "ENTER KEY HERE";
    
        static String host = "https://api.cognitive.microsoft.com";
        static String path = "/bing/v7.0/entities";
    
        static String mkt = "en-US";
        static String query = "italian restaurant near me";
   //...
    
   ```

## <a name="construct-a-search-request-string"></a>Construcción de una cadena de solicitud de búsqueda

1. Creación de una función denominada `search()` que devuelve un valor JSON `String`. Codifique como URL la consulta de búsqueda y agréguela a una cadena de parámetros con `&q=`. Agregue el mercado a la cadena de parámetro con `?mkt=`.
 
2. Cree un objeto de dirección URL con las cadenas de host, ruta de acceso y parámetros.
    
    ```java
    //...
    public static String search () throws Exception {
        String encoded_query = URLEncoder.encode (query, "UTF-8");
        String params = "?mkt=" + mkt + "&q=" + encoded_query;
        URL url = new URL (host + path + params);
    //...
    ```
      
## <a name="send-a-search-request-and-receive-a-response"></a>Envío de una solicitud de búsqueda y recepción de la respuesta

1. En la función `search()` creada anteriormente, cree un nuevo objeto `HttpsURLConnection` con `url.openCOnnection()`. Establezca el método de solicitud en `GET` y agregue la clave de suscripción al encabezado `Ocp-Apim-Subscription-Key`.

    ```java
    //...
    HttpsURLConnection connection = (HttpsURLConnection) url.openConnection();
    connection.setRequestMethod("GET");
    connection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);
    connection.setDoOutput(true);
    //...
    ```

2. Cree un nuevo `StringBuilder`. Utilice un nuevo `InputStreamReader` como parámetro al crear la instancia `BufferedReader` para leer la respuesta de la API.  
    
    ```java
    //...
    StringBuilder response = new StringBuilder ();
    BufferedReader in = new BufferedReader(
        new InputStreamReader(connection.getInputStream()));
    //...
    ```

3. Cree un objeto `String` para almacenar la respuesta de la `BufferedReader`. Itere por él y anexe cada línea en la cadena. Después, cierre el lector y devuelva la respuesta. 
    
    ```java
    String line;
    
    while ((line = in.readLine()) != null) {
      response.append(line);
    }
    in.close();
    
    return response.toString();
    ```

## <a name="format-the-json-response"></a>Formato de la respuesta JSON

1. Cree una nueva función denominada `prettify` para dar formato a la respuesta JSON. Cree un nuevo `JsonParser`, llame a `parse()` en el texto JSON y almacénelo como un objeto JSON. 

2. Utilice la biblioteca Gson para crear un nuevo `GsonBuilder()` y use `setPrettyPrinting().create()` para dar formato al objeto JSON, y luego devuélvalo.    
  
   ```java
   //...
   public static String prettify (String json_text) {
    JsonParser parser = new JsonParser();
    JsonObject json = parser.parse(json_text).getAsJsonObject();
    Gson gson = new GsonBuilder().setPrettyPrinting().create();
    return gson.toJson(json);
   }
   //...
   ```

## <a name="call-the-search-function"></a>Llamada a la función de búsqueda

- Desde el método principal del proyecto, puede llamar a `search()` y usar `prettify()` para dar formato al texto.
    
    ```java
        public static void main(String[] args) {
            try {
                String response = search ();
                System.out.println (prettify (response));
            }
            catch (Exception e) {
                System.out.println (e);
            }
        }
    ```

## <a name="example-json-response"></a>Ejemplo de respuesta JSON

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "_type": "SearchResponse",
  "queryContext": {
    "originalQuery": "italian restaurant near me",
    "askUserForLocation": true
  },
  "places": {
    "value": [
      {
        "_type": "LocalBusiness",
        "webSearchUrl": "https://www.bing.com/search?q=sinful+bakery&filters=local...",
        "name": "Liberty's Delightful Sinful Bakery & Cafe",
        "url": "https://www.contoso.com/",
        "entityPresentationInfo": {
          "entityScenario": "ListItem",
          "entityTypeHints": [
            "Place",
            "LocalBusiness"
          ]
        },
        "address": {
          "addressLocality": "Seattle",
          "addressRegion": "WA",
          "postalCode": "98112",
          "addressCountry": "US",
          "neighborhood": "Madison Park"
        },
        "telephone": "(800) 555-1212"
      },

      . . .
      {
        "_type": "Restaurant",
        "webSearchUrl": "https://www.bing.com/search?q=Pickles+and+Preserves...",
        "name": "Munson's Pickles and Preserves Farm",
        "url": "https://www.princi.com/",
        "entityPresentationInfo": {
          "entityScenario": "ListItem",
          "entityTypeHints": [
            "Place",
            "LocalBusiness",
            "Restaurant"
          ]
        },
        "address": {
          "addressLocality": "Seattle",
          "addressRegion": "WA",
          "postalCode": "98101",
          "addressCountry": "US",
          "neighborhood": "Capitol Hill"
        },
        "telephone": "(800) 555-1212"
      },
      
      . . .
    ]
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Compilar una aplicación web de una sola página](../tutorial-bing-entities-search-single-page-app.md)

* [¿Qué es Bing Entity Search API?](../overview.md)
* [Referencia de la API Bing Entity Search](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference).
