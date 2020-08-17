---
title: Inicio rápido de la biblioteca cliente para Java de Bing Web Search
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/05/2020
ms.custom: devx-track-java
ms.author: aahi
ms.openlocfilehash: 3723345b10524b2063b3ec8fe43608f49b5b279b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87375279"
---
La biblioteca cliente de Bing Web Search facilita la integración de Bing Web Search en la aplicación de Java. En esta guía de inicio rápido, aprenderá a enviar una solicitud, recibir una respuesta JSON, y filtrar y analizar los resultados.

¿Desea ver el código ahora mismo? En GitHub se encuentran disponibles ejemplos de [bibliotecas cliente de Bing Search para Java](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search).

## <a name="prerequisites"></a>Prerequisites

Estas son algunas cosas que necesitará antes de ejecutar esta guía de inicio rápido:

* [JDK 7 o 8](https://aka.ms/azure-jdks)
* [Apache Maven](https://maven.apache.org/download.cgi) o herramienta de automatización de compilación que prefiera
* Una clave de suscripción

[!INCLUDE [bing-web-search-quickstart-signup](~/includes/bing-web-search-quickstart-signup.md)]

## <a name="create-a-project-and-set-up-your-pom-file"></a>Creación de un proyecto y configuración de su archivo POM

Cree un proyecto de Java mediante Maven o la herramienta de automatización de compilación que prefiera. Suponiendo que utilice Maven, agregue las siguientes líneas al archivo de [modelo de objetos de proyectos (POM)](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html). Reemplace todas las instancias de `mainClass` por la aplicación.

```xml
<build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>1.4.0</version>
        <configuration>
          <!--Your comment
            Replace the mainClass with the path to your java application.
            It should begin with com and doesn't require the .java extension.
            For example: com.bingwebsearch.app.BingWebSearchSample. This maps to
            The following directory structure:
            src/main/java/com/bingwebsearch/app/BingWebSearchSample.java.
          -->
          <mainClass>com.path.to.your.app.APP_NAME</mainClass>
        </configuration>
      </plugin>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.0</version>
        <configuration>
          <source>1.7</source>
          <target>1.7</target>
        </configuration>
      </plugin>
      <plugin>
        <artifactId>maven-assembly-plugin</artifactId>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>attached</goal>
            </goals>
            <configuration>
              <descriptorRefs>
                <descriptorRef>jar-with-dependencies</descriptorRef>
              </descriptorRefs>
              <archive>
                <manifest>
                  <!--Your comment
                    Replace the mainClass with the path to your java application.
                    For example: com.bingwebsearch.app.BingWebSearchSample.java.
                    This maps to the following directory structure:
                    src/main/java/com/bingwebsearch/app/BingWebSearchSample.java.
                  -->
                  <mainClass>com.path.to.your.app.APP_NAME.java</mainClass>
                </manifest>
              </archive>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  <dependencies>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure</artifactId>
      <version>1.9.0</version>
    </dependency>
    <dependency>
      <groupId>commons-net</groupId>
      <artifactId>commons-net</artifactId>
      <version>3.3</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure.cognitiveservices</groupId>
      <artifactId>azure-cognitiveservices-websearch</artifactId>
      <version>1.0.1</version>
    </dependency>
  </dependencies>
```

## <a name="declare-dependencies"></a>Declaración de dependencias

Abra el proyecto en su IDE o editor favorito e importe estas dependencias:

```java
import com.microsoft.azure.cognitiveservices.search.websearch.BingWebSearchAPI;
import com.microsoft.azure.cognitiveservices.search.websearch.BingWebSearchManager;
import com.microsoft.azure.cognitiveservices.search.websearch.models.ImageObject;
import com.microsoft.azure.cognitiveservices.search.websearch.models.NewsArticle;
import com.microsoft.azure.cognitiveservices.search.websearch.models.SearchResponse;
import com.microsoft.azure.cognitiveservices.search.websearch.models.VideoObject;
import com.microsoft.azure.cognitiveservices.search.websearch.models.WebPage;
```

Si ha creado el proyecto con Maven, el paquete ya debe estar declarado. En caso contrario, declare el paquete ahora. Por ejemplo:

```java
package com.bingwebsearch.app
```

## <a name="declare-the-bingwebsearchsample-class"></a>Declaración de la clase BingWebSearchSample

Declare la clase `BingWebSearchSample`. Incluirá la mayor parte de nuestro código, incluido el método `main`.  

```java
public class BingWebSearchSample {

// The code in the following sections goes here.

}
```

## <a name="construct-a-request"></a>Creación de una solicitud

El método `runSample`, que se encuentra en la clase `BingWebSearchSample`, crea la solicitud. Copie este código en la aplicación:

```java
public static boolean runSample(BingWebSearchAPI client) {
    /*
     * This function performs the search.
     *
     * @param client instance of the Bing Web Search API client
     * @return true if sample runs successfully
     */
    try {
        /*
         * Performs a search based on the .withQuery and prints the name and
         * url for the first web pages, image, news, and video result
         * included in the response.
         */
        System.out.println("Searched Web for \"Xbox\"");
        // Construct the request.
        SearchResponse webData = client.bingWebs().search()
            .withQuery("Xbox")
            .withMarket("en-us")
            .withCount(10)
            .execute();

// Code continues in the next section...
```

## <a name="handle-the-response"></a>Control de la respuesta

A continuación, vamos a agregar un código para analizar la respuesta e imprimir los resultados. Se imprimen `name` y `url` para la primera página web, imagen, artículo de noticias y vídeo si están incluidos en el objeto de respuesta.

```java
/*
* WebPages
* If the search response has web pages, the first result's name
* and url are printed.
*/
if (webData != null && webData.webPages() != null && webData.webPages().value() != null &&
        webData.webPages().value().size() > 0) {
    // find the first web page
    WebPage firstWebPagesResult = webData.webPages().value().get(0);

    if (firstWebPagesResult != null) {
        System.out.println(String.format("Webpage Results#%d", webData.webPages().value().size()));
        System.out.println(String.format("First web page name: %s ", firstWebPagesResult.name()));
        System.out.println(String.format("First web page URL: %s ", firstWebPagesResult.url()));
    } else {
        System.out.println("Couldn't find the first web result!");
    }
} else {
    System.out.println("Didn't find any web pages...");
}
/*
 * Images
 * If the search response has images, the first result's name
 * and url are printed.
 */
if (webData != null && webData.images() != null && webData.images().value() != null &&
        webData.images().value().size() > 0) {
    // find the first image result
    ImageObject firstImageResult = webData.images().value().get(0);

    if (firstImageResult != null) {
        System.out.println(String.format("Image Results#%d", webData.images().value().size()));
        System.out.println(String.format("First Image result name: %s ", firstImageResult.name()));
        System.out.println(String.format("First Image result URL: %s ", firstImageResult.contentUrl()));
    } else {
        System.out.println("Couldn't find the first image result!");
    }
} else {
    System.out.println("Didn't find any images...");
}
/*
 * News
 * If the search response has news articles, the first result's name
 * and url are printed.
 */
if (webData != null && webData.news() != null && webData.news().value() != null &&
        webData.news().value().size() > 0) {
    // find the first news result
    NewsArticle firstNewsResult = webData.news().value().get(0);
    if (firstNewsResult != null) {
        System.out.println(String.format("News Results#%d", webData.news().value().size()));
        System.out.println(String.format("First news result name: %s ", firstNewsResult.name()));
        System.out.println(String.format("First news result URL: %s ", firstNewsResult.url()));
    } else {
        System.out.println("Couldn't find the first news result!");
    }
} else {
    System.out.println("Didn't find any news articles...");
}

/*
 * Videos
 * If the search response has videos, the first result's name
 * and url are printed.
 */
if (webData != null && webData.videos() != null && webData.videos().value() != null &&
        webData.videos().value().size() > 0) {
    // find the first video result
    VideoObject firstVideoResult = webData.videos().value().get(0);

    if (firstVideoResult != null) {
        System.out.println(String.format("Video Results#%s", webData.videos().value().size()));
        System.out.println(String.format("First Video result name: %s ", firstVideoResult.name()));
        System.out.println(String.format("First Video result URL: %s ", firstVideoResult.contentUrl()));
    } else {
        System.out.println("Couldn't find the first video result!");
    }
} else {
    System.out.println("Didn't find any videos...");
}
```

## <a name="declare-the-main-method"></a>Declaración del método main

En esta aplicación, el método main incluye código que crea instancias del cliente, valida `subscriptionKey` y llama a `runSample`. Asegúrese de escribir una clave de suscripción válida para la cuenta de Azure antes de continuar.

```java
public static void main(String[] args) {
    try {
        // Enter a valid subscription key for your account.
        final String subscriptionKey = "YOUR_SUBSCRIPTION_KEY";
        // Instantiate the client.
        BingWebSearchAPI client = BingWebSearchManager.authenticate(subscriptionKey);
        // Make a call to the Bing Web Search API.
        runSample(client);
    } catch (Exception e) {
        System.out.println(e.getMessage());
        e.printStackTrace();
    }
}
```

## <a name="run-the-program"></a>Ejecución del programa

¡El último paso es ejecutar el programa!

```console
mvn compile exec:java
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con este proyecto, asegúrese de quitar la clave de suscripción desde el código del programa.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Ejemplos del SDK de Cognitive Services para Java](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingWebSearch)

## <a name="see-also"></a>Consulte también

* [Referencia de SDK de Azure de Java](https://docs.microsoft.com/java/api/overview/azure/cognitiveservices/client/bingwebsearchapi?view=azure-java-stable)
