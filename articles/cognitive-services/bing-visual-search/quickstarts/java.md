---
title: 'Inicio rápido: Obtención de conclusiones de imágenes mediante la API REST y Java: Bing Visual Search'
titleSuffix: Azure Cognitive Services
description: Aprenda a cargar una imagen en Bing Visual Search API y obtener conclusiones sobre ella.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: quickstart
ms.date: 05/22/2020
ms.custom: devx-track-java
ms.author: scottwhi
ms.openlocfilehash: 6e0e0cc2513b1c2a5f89e61984331399ebae269a
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320449"
---
# <a name="quickstart-get-image-insights-using-the-bing-visual-search-rest-api-and-java"></a>Inicio rápido: Obtención de conclusiones de imágenes mediante la API de REST Bing Visual Search y Java

Use este inicio rápido para realizar la primera llamada a la API Bing Visual Search. Esta aplicación de Java carga una imagen en la API y muestra la información que se devuelve. Aunque esta aplicación está escrita en Java, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

## <a name="prerequisites"></a>Prerrequisitos

* [Kit de desarrollo de Java (JDK) 7 u 8](https://aka.ms/azure-jdks)
* La [biblioteca Java para Gson](https://github.com/google/gson)
* [Apache HttpComponents](https://hc.apache.org/downloads.cgi)

[!INCLUDE [cognitive-services-bing-visual-search-signup-requirements](../../../../includes/cognitive-services-bing-visual-search-signup-requirements.md)]

## <a name="create-and-initialize-a-project"></a>Creación e inicialización de un proyecto

1. Cree un proyecto de Java en su IDE o editor favoritos e importe las bibliotecas siguientes:

    ```java
    import java.util.*;
    import java.io.*;
    import com.google.gson.Gson;
    import com.google.gson.GsonBuilder;
    import com.google.gson.JsonObject;
    import com.google.gson.JsonParser;
    
    // HttpClient libraries
    
    import org.apache.http.HttpEntity;
    import org.apache.http.HttpResponse;
    import org.apache.http.client.methods.HttpPost;
    import org.apache.http.entity.ContentType;
    import org.apache.http.entity.mime.MultipartEntityBuilder;
    import org.apache.http.impl.client.CloseableHttpClient;
    import org.apache.http.impl.client.HttpClientBuilder;
    ```

2. Cree variables para el punto de conexión de API, la clave de suscripción y la ruta de acceso a la imagen. Para el valor `endpoint` puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```java
    static String endpoint = "https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch";
    static String subscriptionKey = "your-key-here";
    static String imagePath = "path-to-your-image";
    ```

    
3. Al cargar una imagen local, los datos del formulario deben incluir el encabezado `Content-Disposition`. Establezca su parámetro `name` en "image", y el parámetro `filename` en el nombre de archivo de la imagen. El contenido del formulario incluye los datos binarios de la imagen. El tamaño de imagen máximo que puede cargar es de 1 MB.
    
    ```
    --boundary_1234-abcd
    Content-Disposition: form-data; name="image"; filename="myimagefile.jpg"
    
    ÿØÿà JFIF ÖÆ68g-¤CWŸþ29ÌÄøÖ‘º«™æ±èuZiÀ)"óÓß°Î= ØJ9á+*G¦...
    
    --boundary_1234-abcd--
    ```

## <a name="create-the-json-parser"></a>Creación del analizador JSON

Cree un método para hacer que la respuesta JSON desde la API sea más legible mediante `JsonParser`.

```java
public static String prettify(String json_text) {
        JsonParser parser = new JsonParser();
        JsonObject json = parser.parse(json_text).getAsJsonObject();
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        return gson.toJson(json);
    }
```

## <a name="construct-the-search-request-and-query"></a>Construcción de la consulta y la solicitud de búsqueda

1. En el método main de la aplicación, cree un cliente HTTP con `HttpClientBuilder.create().build();`.

    ```java
    CloseableHttpClient httpClient = HttpClientBuilder.create().build();
    ```

2. Cree un objeto `HttpEntity` para cargar la imagen en la API.

    ```java
    HttpEntity entity = MultipartEntityBuilder
        .create()
        .addBinaryBody("image", new File(imagePath))
        .build();
    ```

3. Cree un objeto `httpPost` con el punto de conexión y establezca el encabezado para usar la clave de suscripción.

    ```java
    HttpPost httpPost = new HttpPost(endpoint);
    httpPost.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);
    httpPost.setEntity(entity);
    ```

## <a name="receive-and-process-the-json-response"></a>Recepción y procesamiento de la respuesta JSON

1. Use el método `HttpClient.execute()` para enviar una solicitud a la API y almacene la respuesta en un objeto `InputStream`.
    
    ```java
    HttpResponse response = httpClient.execute(httpPost);
    InputStream stream = response.getEntity().getContent();
    ```

2. Almacene la cadena JSON e imprima la respuesta.

    ```java
    String json = new Scanner(stream).useDelimiter("\\A").next();
    System.out.println("\nJSON Response:\n");
    System.out.println(prettify(json));
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación web de página única de Visual Search](../tutorial-bing-visual-search-single-page-app.md)
