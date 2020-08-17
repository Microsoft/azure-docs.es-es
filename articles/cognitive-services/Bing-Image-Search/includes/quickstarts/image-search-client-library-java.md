---
title: Inicio rápido de la biblioteca cliente para Java de Bing Image Search
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/04/2020
ms.custom: devx-track-java
ms.author: aahi
ms.openlocfilehash: d7433da99fa1aa607b8c85ce6c8f44adcc6c6f89
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87375569"
---
Use este inicio rápido para buscar su primera imagen mediante la biblioteca cliente de Bing Image Search, un contenedor de la API y que contiene las mismas características. Esta sencilla aplicación de Java envía una consulta de búsqueda de imagen, analiza la respuesta de JSON y muestra la dirección URL de la primera imagen devuelta.

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingImageSearch/Quickstart) con anotaciones y control de errores adicionales.

## <a name="prerequisites"></a>Requisitos previos

La última versión del [Kit de desarrollo de Java (JDK)](https://aka.ms/azure-jdks)

Instale las dependencias de la biblioteca cliente de Bing Image Search con Maven, Gradle u otro sistema de administración de dependencias. El archivo POM de Maven requiere la declaración siguiente:

```xml
 <dependencies>
    <dependency>
      <groupId>com.microsoft.azure.cognitiveservices</groupId>
      <artifactId>azure-cognitiveservices-imagesearch</artifactId>
      <version>1.0.1</version>
    </dependency>
 </dependencies>
```

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](~/includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

1. Cree un proyecto de Java en su IDE o editor favorito y agregue las siguientes importaciones a la implementación de clases:

    ```java
    import com.microsoft.azure.cognitiveservices.search.imagesearch.BingImageSearchAPI;
    import com.microsoft.azure.cognitiveservices.search.imagesearch.BingImageSearchManager;
    import com.microsoft.azure.cognitiveservices.search.imagesearch.models.ImageObject;
    import com.microsoft.azure.cognitiveservices.search.imagesearch.models.ImagesModel;
    ```

2. En el método Main, cree variables para el término de búsqueda y la clave de suscripción. Luego, cree la instancia del cliente de Bing Image Search.

    ```java
    final String subscriptionKey = "COPY_YOUR_KEY_HERE";
    String searchTerm = "canadian rockies";
    //Image search client
    BingImageSearchAPI client = BingImageSearchManager.authenticate(subscriptionKey);
    ```

## <a name="send-a-search-request-to-the-api"></a>Envío de una solicitud de búsqueda a la API

1. Con el uso de `bingImages().search()`, envíe la solicitud HTTP que contiene la consulta de búsqueda. Guarde la respuesta como `ImagesModel`.

   ```java
    ImagesModel imageResults = client.bingImages().search()
                .withQuery(searchTerm)
                .withMarket("en-us")
                .execute();
    ```

## <a name="parse-and-view-the-result"></a>Análisis y visualización del resultado

Analice los resultados de la imagen devueltos en la respuesta.
Si la respuesta contiene resultados de la búsqueda, almacene el primer resultado e imprima sus detalles, como la dirección URL de una miniatura, la dirección URL original, además del número total de imágenes devueltas.  

```java
if (imageResults != null && imageResults.value().size() > 0) {
    // Image results
    ImageObject firstImageResult = imageResults.value().get(0);

    System.out.println(String.format("Total number of images found: %d", imageResults.value().size()));
    System.out.println(String.format("First image thumbnail url: %s", firstImageResult.thumbnailUrl()));
    System.out.println(String.format("First image content url: %s", firstImageResult.contentUrl()));
}
else {
        System.out.println("Couldn't find image results!");
     }

```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial de aplicación de una sola página de Bing Image Search](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/tutorial-bing-image-search-single-page-app)

## <a name="see-also"></a>Consulte también

* [¿Qué es Bing Image Search?](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/overview)  
* [Prueba de una demostración interactiva en línea](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/)  
* [Ejemplos de Java para el SDK de Azure Cognitive Services](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples)
* [Documentación de Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services)
* [Referencia de Bing Image Search API](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference)
