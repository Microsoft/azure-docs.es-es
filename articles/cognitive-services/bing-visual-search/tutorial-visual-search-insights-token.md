---
title: Búsqueda de imágenes similares a partir de búsquedas anteriores mediante ImageInsightsToken y Bing Visual Search API
titleSuffix: Azure Cognitive Services
description: Use la biblioteca cliente de Bing Visual Search para obtener las direcciones URL de las imágenes de las búsquedas anteriores.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: tutorial
ms.date: 03/31/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: c03800696897dc6cbb4cd793879e734366829b7d
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88925120"
---
# <a name="tutorial-find-similar-images-from-previous-searches-using-an-image-insights-token"></a>Tutorial: Búsqueda de imágenes similares de búsquedas anteriores mediante un token de información de imágenes

La biblioteca cliente de Visual Search le permite buscar imágenes en línea de una búsqueda anterior que devuelve un token `ImageInsightsToken`. En esta aplicación se obtiene un token `ImageInsightsToken` que se usa en una búsqueda posterior. Después, la aplicación envía el token `ImageInsightsToken` a Bing y devuelve los resultados que incluyen direcciones URL de Bing Search y las direcciones URL de imágenes similares que se encuentran en línea.

El código fuente completo de este tutorial está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/Bing-Visual-Search/BingVisualSearchInsightsTokens.cs) con anotaciones y control de errores adicionales.

## <a name="prerequisites"></a>Prerrequisitos

* Cualquier edición de [Visual Studio 2019](https://www.visualstudio.com/downloads/).
* Si usa Linux o MacOS, puede ejecutar esta aplicación con [Mono](https://www.mono-project.com/).
* Los paquetes NuGet para Visual Search e Image Search.
    - En el Explorador de soluciones de Visual Studio, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet** en el menú. Instale los paquetes `Microsoft.Azure.CognitiveServices.Search.CustomSearch` y `Microsoft.Azure.CognitiveServices.Search.ImageSearch`. La instalación de los paquetes NuGet también instala lo siguiente:
        - Microsoft.Rest.ClientRuntime
        - Microsoft.Rest.ClientRuntime.Azure
        - Newtonsoft.Json


[!INCLUDE [cognitive-services-bing-visual-search-signup-requirements](../../../includes/cognitive-services-bing-visual-search-signup-requirements.md)]

## <a name="get-the-imageinsightstoken-from-the-bing-image-search-client-library"></a>Obtención de ImageInsightsToken a partir de la biblioteca cliente de Bing Image Search

Esta aplicación usa un token `ImageInsightsToken` obtenido a través de la [biblioteca cliente de Bing Image Search](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/image-search-sdk-quickstart). En una nueva aplicación de consola de C#, crea un cliente para llamar a la API mediante `ImageSearchClient()`. A continuación, use `SearchAsync()` con su consulta:

```csharp
var client = new ImageSearchClient(new Microsoft.Azure.CognitiveServices.Search.ImageSearch.ApiKeyServiceClientCredentials(subKey));
var imageResults = client.Images.SearchAsync(query: "canadian rockies").Result;
Console.WriteLine("Search images for query \"canadian rockies\"");
```

Almacene el primer resultado de la búsqueda mediante `imageResults.Value.First()` y, a continuación, almacene el token `ImageInsightsToken` de la información de imagen.

```csharp
String insightTok = "None";
if (imageResults.Value.Count > 0)
{
    var firstImageResult = imageResults.Value.First();
    insightTok = firstImageResult.ImageInsightsToken;
}
else
{
    insightTok = "None found";
    Console.WriteLine("Couldn't find image results!");
}
```

Dicho token `ImageInsightsToken` se envía a Bing Visual Search en una solicitud.

## <a name="add-the-imageinsightstoken-to-a-visual-search-request"></a>Adición del token ImageInsightsToken a una solicitud de Visual Search

Especifique el token `ImageInsightsToken` para una solicitud de Visual Search mediante la creación de un objeto `ImageInfo` desde el token `ImageInsightsToken` contenido en las respuestas de Bing Visual Search.

```csharp
ImageInfo ImageInfo = new ImageInfo(imageInsightsToken: insightsTok);
```

## <a name="use-bing-visual-search-to-find-images-from-an-imageinsightstoken"></a>Uso de Bing Visual Search para buscar imágenes desde un token ImageInsightsToken

El objeto `VisualSearchRequest` contiene información sobre la imagen en `ImageInfo` que se va a buscar. El método `VisualSearchMethodAsync()` obtiene los resultados. No es necesario que proporcione un archivo binario de imagen, ya que la imagen se representa mediante el token.

```csharp
VisualSearchRequest VisualSearchRequest = new VisualSearchRequest(ImageInfo);

var visualSearchResults = client.Images.VisualSearchMethodAsync(knowledgeRequest: VisualSearchRequest).Result;

```

## <a name="iterate-through-the-visual-search-results"></a>Iteración de los resultados de Visual Search

Los resultados de Visual Search son objetos `ImageTag`. Cada etiqueta contiene una lista de objetos `ImageAction`. Cada elemento `ImageAction` contiene un campo `Data`, que es una lista de valores que dependen del tipo de acción. Puede iterar por los objetos `ImageTag` en `visualSearchResults.Tags`, por ejemplo, y obtener la etiqueta `ImageAction` que se encuentra dentro. En el ejemplo siguiente se imprimen los detalles de las acciones `PagesIncluding`:

```csharp
if (visualSearchResults.Tags.Count > 0)
{
    // List of tags
    foreach (ImageTag t in visualSearchResults.Tags)
    {
        foreach (ImageAction i in t.Actions)
        {
            Console.WriteLine("\r\n" + "ActionType: " + i.ActionType + " WebSearchURL: " + i.WebSearchUrl);

            if (i.ActionType == "PagesIncluding")
            {
                foreach (ImageObject o in (i as ImageModuleAction).Data.Value)
                {
                    Console.WriteLine("ContentURL: " + o.ContentUrl);
                }
            }
        }
    }
}
```

### <a name="pagesincluding-actiontypes"></a>PagesIncluding ActionTypes

Para la obtención de las direcciones URL de imágenes reales a partir de los tipos de acción se requiere una conversión que lea un elemento `ActionType` como `ImageModuleAction`, que contiene un elemento `Data` con una lista de valores. Cada valor es la dirección URL de una imagen.  El siguiente código convierte el tipo de acción `PagesIncluding` en `ImageModuleAction` y lee los valores:

```csharp
    if (i.ActionType == "PagesIncluding")
    {
        foreach(ImageObject o in (i as ImageModuleAction).Data.Value)
        {
            Console.WriteLine("ContentURL: " + o.ContentUrl);
        }
    }
```

Para obtener más información acerca de estos tipos de datos, vea [Images - Visual Search](https://docs.microsoft.com/rest/api/cognitiveservices/bingvisualsearch/images/visualsearch) (Imágenes: búsqueda visual).

## <a name="returned-urls"></a>Direcciones URL devueltas

La aplicación completa devuelve las siguientes direcciones URL:

|ActionType  |URL  |
|---------|---------|
|MoreSizes -> WebSearchUrl     |         |
|VisualSearch -> WebSearchUrl     |         |
|ImageById -> WebSearchUrl    |         |
|RelatedSearches -> WebSearchUrl:    |         |
|DocumentLevelSuggestions -> WebSearchUrl:     |         |
|TopicResults -> WebSearchUrl    | https:\//www.bing.com/cr?IG=3E32CC6CA5934FBBA14ABC3B2E4651F9&CID=1BA795A21EAF6A63175699B71FC36B7C&rd=1&h=BcQifmzdKFyyBusjLxxgO42kzq1Geh7RucVVqvH-900&v=1&r=https%3a%2f%2f www.bing.com%2fdiscover%2fcanadian%2brocky&p=DevEx,5823.1       |
|ImageResults -> WebSearchUrl    |  https:\//www.bing.com/cr?IG=3E32CC6CA5934FBBA14ABC3B2E4651F9&CID=1BA795A21EAF6A63175699B71FC36B7C&rd=1&h=PV9GzMFOI0AHZp2gKeWJ8DcveSDRE3fP2jHDKMpJSU8&v=1&r=https%3a%2f%2f www.bing.com%2fimages%2fsearch%3fq%3doutdoor&p=DevEx,5831.1       |

Como se mostró anteriormente, los tipos `TopicResults` y `ImageResults` contienen consultas de imágenes relacionadas. Las direcciones URL se vinculan a los resultados de Bing Search.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación web de página única de Visual Search](tutorial-bing-visual-search-single-page-app.md)
