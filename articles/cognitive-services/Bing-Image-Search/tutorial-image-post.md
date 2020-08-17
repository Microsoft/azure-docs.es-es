---
title: 'Tutorial: Extracción de detalles de imágenes con la API REST y C#: Bing Image Search'
titleSuffix: Azure Cognitive Services
description: Use este inicio rápido para crear una aplicación de C# que extraiga los detalles de la imagen con Bing Image Search API.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: tutorial
ms.date: 12/06/2019
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: 2b51fb8dcc456d895d5c4032bd16e1502ee86f0f
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407088"
---
# <a name="tutorial-extract-image-details-using-the-bing-image-search-api-and-c"></a>Tutorial: Extracción de detalles de imágenes con Bing Image Search API y C#

Hay varios [puntos de conexión](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/image-search-endpoint) disponibles mediante Bing Image Search API. El punto de conexión `/details` acepta una solicitud POST con una imagen y puede devolver una variedad de detalles sobre la imagen. Esta aplicación de C# envía una imagen mediante esta API y muestra los detalles devueltos por Bing, que son objetos JSON, como los siguientes:

![[Resultados JSON]](media/cognitive-services-bing-images-api/jsonResult.jpg)

Este tutorial explica cómo realizar lo siguiente:

> [!div class="checklist"]
> * Utilizar el punto de conexión `/details` de Image Search en una solicitud `POST`.
> * Especificar encabezados para la solicitud.
> * Utilizar parámetros de dirección URL para especificar los resultados.
> * Cargar los datos de imagen y enviar la solicitud `POST`.
> * Imprimir los resultados JSON en la consola.

El código fuente del ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/BingGetSimilarImages.cs).

## <a name="prerequisites"></a>Prerrequisitos

* Cualquier edición de [Visual Studio 2017 o posterior](https://visualstudio.microsoft.com/downloads/).

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="construct-an-image-details-search-request"></a>Construcción de una solicitud de búsqueda de detalles de la imagen

A continuación, se muestra el punto de conexión `/details`, que acepta solicitudes POST con los datos de imagen en el cuerpo de la solicitud. Puede usar el punto de conexión global siguiente o el punto de conexión del [subdominio personalizado](../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.
```
https://api.cognitive.microsoft.com/bing/v7.0/images/details
```

Al construir la dirección URL de la solicitud de búsqueda, el parámetro `modules` sigue el punto de conexión anterior y especifica los tipos de detalles que contendrán los resultados:

* `modules=All`
* `modules=RecognizedEntities` (personas o lugares visibles en la imagen)

Especifique `modules=All` en la solicitud POST para obtener el texto JSON que incluye lo siguiente:

* `bestRepresentativeQuery` - una consulta de Bing que devuelve imágenes similares a la imagen cargada
* `detectedObjects` - los objetos encontrados en la imagen
* `image` - los metadatos de la imagen
* `imageInsightsToken` - un token para una solicitud GET posterior que obtiene `RecognizedEntities` (personas o lugares visibles en la imagen) de la imagen
* `imageTags` - etiquetas para la imagen
* `pagesIncluding` - páginas web que incluyen la imagen
* `relatedSearches` - búsquedas basadas en los detalles de la imagen
* `visuallySimilarImages` -imágenes similares en la Web

Especifique `modules=RecognizedEntities` en la solicitud POST para obtener solo `imageInsightsToken`, que puede utilizarse en una solicitud GET subsiguiente para identificar usuarios o lugares en la imagen.

## <a name="create-a-webclient-object-and-set-headers-for-the-api-request"></a>Cree un objeto WebClient y establezca los encabezados de la solicitud de API

Cree un objeto `WebClient` y establezca los encabezados. Todas las solicitudes de Bing Search API necesitan `Ocp-Apim-Subscription-Key`. Una solicitud `POST` para cargar una imagen también debe especificar `ContentType: multipart/form-data`.

```javascript
WebClient client = new WebClient();
client.Headers["Ocp-Apim-Subscription-Key"] = accessKey;
client.Headers["ContentType"] = "multipart/form-data";
```

## <a name="upload-the-image-and-display-the-results"></a>Cargar la imagen y mostrar los resultados

El método `UpLoadFile()` de la clase `WebClient` aplica formato a los datos de la solicitud `POST`, incluido el formato `RequestStream` y la llamada a `HttpWebRequest`.

Llame a `WebClient.UpLoadFile()` con el punto de conexión `/details` y el archivo de imagen para cargar. Use la respuesta JSON para inicializar una instancia de la estructura `SearchResult` y almacenar la respuesta.

```javascript        
const string uriBase = "https://api.cognitive.microsoft.com/bing/v7.0/images/details";
// The image to upload. Replace with your file and path.
const string imageFile = "your-image.jpg";
byte[] resp = client.UploadFile(uriBase + "?modules=All", imageFile);
var json = System.Text.Encoding.Default.GetString(resp);
// Create result object for return
var searchResult = new SearchResult()
{
    jsonResult = json,
    relevantHeaders = new Dictionary<String, String>()
};
```
Esta respuesta JSON después se puede imprimir en la consola.

## <a name="use-an-image-insights-token-in-a-request"></a>Uso de un token de información de la imagen en una solicitud

Para usar el elemento `ImageInsightsToken` devuelto con los resultados de `POST`, puede agregarlo a una solicitud `GET`. Por ejemplo:

```
https://api.cognitive.microsoft.com/bing/v7.0/images/details?InsightsToken="bcid_A2C4BB81AA2C9EF8E049C5933C546449*ccid_osS7gaos*mid_BF7CC4FC4A882A3C3D56E644685BFF7B8BACEAF2
```

Si hay personas o lugares identificables en la imagen, esta solicitud devolverá información sobre ellos.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Mostrar imágenes y opciones de búsqueda en una aplicación web de una sola página ](tutorial-bing-image-search-single-page-app.md)

## <a name="see-also"></a>Consulte también

* [Referencia de Bing Image Search API](//docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference)
