---
title: Inicio rápido de la biblioteca cliente para C# de Bing Video Search
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/19/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: 6d50a8e2c9d0263616b25e25958be6a6f0fb7fe1
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88929283"
---
Use este artículo de inicio rápido para empezar a buscar vídeos con la biblioteca cliente de Bing Video Search para C#. Aunque Bing Video Search tiene una API REST compatible con la mayoría de los lenguajes de programación, la biblioteca cliente proporciona una forma sencilla de integrar el servicio en sus aplicaciones. El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7/BingVideoSearch) con anotaciones adicionales y características.

## <a name="prerequisites"></a>Prerrequisitos

* Cualquier edición de [Visual Studio 2017 o versiones posteriores](https://visualstudio.microsoft.com/downloads/).
* El marco Json.NET, disponible como [paquete NuGet](https://www.nuget.org/packages/Newtonsoft.Json/).

Para agregar la biblioteca cliente de Bing Video Search al proyecto, seleccione **Administrar paquetes NuGet** en el **Explorador de soluciones** en Visual Studio. Agregue el paquete `Microsoft.Azure.CognitiveServices.Search.VideoSearch`.

La instalación del [[paquete del SDK de NuGet Video Search]](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.VideoSearch/1.2.0) también instala las dependencias siguientes:

* Microsoft.Rest.ClientRuntime
* Microsoft.Rest.ClientRuntime.Azure
* Newtonsoft.Json

[!INCLUDE [cognitive-services-bing-video-search-signup-requirements](~/includes/cognitive-services-bing-video-search-signup-requirements.md)]


## <a name="create-and-initialize-a-project"></a>Creación e inicialización de un proyecto

1. Cree una solución de consola de C# en Visual Studio. Después, agregue lo siguiente en el archivo de código principal.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Azure.CognitiveServices.Search.VideoSearch;
    using Microsoft.Azure.CognitiveServices.Search.VideoSearch.Models;
    ```

2. Cree una instancia del cliente, para lo que debe crear un nuevo objeto `ApiKeyServiceClientCredentials` con su clave de suscripción y llamar al constructor.

    ```csharp
    var client = new VideoSearchAPI(new ApiKeyServiceClientCredentials("YOUR-ACCESS-KEY"));
    ```

## <a name="send-a-search-request-and-process-the-results"></a>Envío de una solicitud de búsqueda y procesamiento de los resultados

1. Use el cliente para enviar una solicitud de búsqueda. Use "SwiftKey" para la consulta de búsqueda.

    ```csharp
    var videoResults = client.Videos.SearchAsync(query: "SwiftKey").Result;
    ```

2. Si se devuelven resultados, obtenga el primera de ellos con `videoResults.Value[0]`. A continuación, imprima el identificador, el título y la dirección URL del vídeo.

    ```csharp
    if (videoResults.Value.Count > 0)
    {
        var firstVideoResult = videoResults.Value[0];

        Console.WriteLine($"\r\nVideo result count: {videoResults.Value.Count}");
        Console.WriteLine($"First video id: {firstVideoResult.VideoId}");
        Console.WriteLine($"First video name: {firstVideoResult.Name}");
        Console.WriteLine($"First video url: {firstVideoResult.ContentUrl}");
    }
    else
    {
        Console.WriteLine("Couldn't find video results!");
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación web de una sola página](../../tutorial-bing-video-search-single-page-app.md)

## <a name="see-also"></a>Consulte también 

* [¿Qué es Bing Video Search API?](../../overview.md)
* [Ejemplos del SDK de Cognitive Services para .NET](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7)
