---
title: 'Tutorial: Cortar una imagen con el SDK de Bing Visual Search'
description: Use el SDK de Bing Visual Search para obtener información sobre áreas específicas de una imagen.
services: cognitive-services
titleSuffix: Azure Cognitive Services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: article
ms.date: 04/03/2019
ms.author: rosh
ms.openlocfilehash: 7c3fc67dbeac71530d9c5ddedc609296023fe901
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796537"
---
# <a name="tutorial-crop-an-image-with-the-bing-visual-search-sdk-for-c"></a>Tutorial: Recortar una imagen con el SDK de Bing Visual Search para C#

El SDK de Bing Visual Search le permite recortar una imagen antes de encontrar imágenes en línea similares. Esta aplicación recorta a una sola persona de una imagen que contiene varias personas y, a continuación, devuelve los resultados de búsqueda que contienen imágenes similares en línea.

El código fuente completo para esta aplicación está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/Bing-Visual-Search/BingVisualSearchCropImage.cs) con anotaciones y control de errores adicionales.

En este tutorial se muestra cómo:

> [!div class="checklist"]
> * Enviar una solicitud mediante el SDK de Bing Visual Search
> * Recortar un área de imagen para buscar con Bing Visual Search
> * Recibir y gestionar la respuesta
> * Encontrar las direcciones URL de elementos de acción en la respuesta

## <a name="prerequisites"></a>Requisitos previos

* Cualquier edición de [Visual Studio 2017](https://www.visualstudio.com/downloads/).
* Si usa Linux/MacOS, esta aplicación puede ejecutarse con [Mono](https://www.mono-project.com/).
* El paquete [NuGet Custom Search](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.CustomSearch/1.2.0) instalado.
    - En el Explorador de soluciones en Visual Studio, haga doble clic en el proyecto y seleccione **administrar paquetes de NuGet** en el menú. Instale el paquete `Microsoft.Azure.CognitiveServices.Search.CustomSearch`. Al instalar el paquete NuGet Custom Search, también se instalarán los ensamblados siguientes:
        - Microsoft.Rest.ClientRuntime
        - Microsoft.Rest.ClientRuntime.Azure
        - Newtonsoft.Json

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-visual-search-signup-requirements.md)]

## <a name="specify-the-image-crop-area"></a>Especificación del área de recorte de la imagen

Esta aplicación recorta un área de esta imagen del equipo de liderazgo de Microsoft. Esta área de recorte se define utilizando las coordenadas superior izquierda e inferior derecha, representadas como un porcentaje de toda la imagen:  

![Equipo de responsables sénior de Microsoft](./media/MS_SrLeaders.jpg)

Esta imagen se recorta mediante la creación de un objeto `ImageInfo` del el área de recorte y la carga del objeto `ImageInfo` en una solicitud `VisualSearchRequest`. La `ImageInfo` objeto también incluye la dirección URL de la imagen:

```csharp
CropArea CropArea = new CropArea(top: (float)0.01, bottom: (float)0.30, left: (float)0.01, right: (float)0.20);
string imageURL = "https://docs.microsoft.com/azure/cognitive-services/bing-visual-search/media/ms_srleaders.jpg";
ImageInfo imageInfo = new ImageInfo(cropArea: CropArea, url: imageURL);

VisualSearchRequest visualSearchRequest = new VisualSearchRequest(imageInfo: imageInfo);
```

## <a name="search-for-images-similar-to-the-crop-area"></a>Búsqueda de imágenes similares al área de recorte

La variable `VisualSearchRequest` contiene información sobre el área de recorte de la imagen y su dirección URL. El `VisualSearchMethodAsync()` método obtiene los resultados:

```csharp
Console.WriteLine("\r\nSending visual search request with knowledgeRequest that contains URL and crop area");
var visualSearchResults = client.Images.VisualSearchMethodAsync(knowledgeRequest: visualSearchRequest).Result;

```

## <a name="get-the-url-data-from-imagemoduleaction"></a>Obtener los datos de la dirección URL de `ImageModuleAction`

Los resultados de Bing Visual Search son objetos `ImageTag`. Cada etiqueta contiene una lista de objetos `ImageAction`. Cada `ImageAction` contiene un `Data` campo, que es una lista de valores que dependen del tipo de acción.

Puede imprimir los distintos tipos con el código siguiente:

```csharp
Console.WriteLine("\r\n" + "ActionType: " + i.ActionType + " -> WebSearchUrl: " + i.WebSearchUrl);
```

La aplicación completa devuelve:

|ActionType  |Dirección URL  | |
|---------|---------|---------|
|PagesIncluding WebSearchURL     |         |
|MoreSizes WebSearchURL     |         |  
|VisualSearch WebSearchURL    |         |
|ImageById WebSearchURL     |         |  
|RelatedSearches WebSearchURL     |         |
|Entity -> WebSearchUrl     | https://www.bing.com/cr?IG=E40D0E1A13404994ACB073504BC937A4&CID=03DCF882D7386A442137F49BD6596BEF&rd=1&h=BvvDoRtmZ35Xc_UZE4lZx6_eg7FHgcCkigU1D98NHQo&v=1&r=https%3a%2f%2fwww.bing.com%2fsearch%3fq%3dSatya%2bNadella&p=DevEx,5380.1        |
|TopicResults -> WebSearchUrl    |  https://www.bing.com/cr?IG=E40D0E1A13404994ACB073504BC937A4&CID=03DCF882D7386A442137F49BD6596BEF&rd=1&h=3QGtxPb3W9LemuHRxAlW4CW7XN4sPkUYCUynxAqI9zQ&v=1&r=https%3a%2f%2fwww.bing.com%2fdiscover%2fnadella%2bsatya&p=DevEx,5382.1        |
|ImageResults -> WebSearchUrl    |  https://www.bing.com/cr?IG=E40D0E1A13404994ACB073504BC937A4&CID=03DCF882D7386A442137F49BD6596BEF&rd=1&h=l-WNHO89Kkw69AmIGe2MhlUp6MxR6YsJszgOuM5sVLs&v=1&r=https%3a%2f%2fwww.bing.com%2fimages%2fsearch%3fq%3dSatya%2bNadella&p=DevEx,5384.1        |

Tal y como se muestra en el texto anterior, el tipo de acción `Entity` contiene una consulta de Bing Search que devuelve información acerca de una persona, lugar o cosa reconocibles. Los tipos `TopicResults` y `ImageResults` contienen consultas de imágenes relacionadas. Las direcciones URL de la lista vinculan a los resultados de Bing Search.

## <a name="get-urls-for-pagesincluding-actiontype-images"></a>Obtener las direcciones URL para `PagesIncluding` `ActionType` imágenes

Para la obtención de las direcciones URL de imágenes reales se requiere una conversión que lea un elemento `ActionType` como `ImageModuleAction`, que contenga un elemento `Data` con una lista de valores. Cada valor es la dirección URL de una imagen. Las conversiones siguientes el `PagesIncluding` tipo de acción para `ImageModuleAction` y lee los valores:

```csharp
    if (i.ActionType == "PagesIncluding")
    {
        foreach(ImageObject o in (i as ImageModuleAction).Data.Value)
        {
            Console.WriteLine("ContentURL: " + o.ContentUrl);
        }
    }
```

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Creación de una aplicación web de página única de Visual Search](tutorial-bing-visual-search-single-page-app.md)

## <a name="see-also"></a>Vea también
> [¿Qué es la API de Bing Visual Search?](https://docs.microsoft.com/azure/cognitive-services/bing-visual-search/overview)
