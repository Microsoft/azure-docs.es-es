---
title: 'Inicio rápido: Llamada al punto de conexión de Bing Custom Search con C# | Microsoft Docs'
titleSuffix: Azure Cognitive Services
description: Use este documento de inicio rápido para comenzar a solicitar los resultados de búsqueda de la instancia de Bing Custom Search con C#.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: quickstart
ms.date: 05/08/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: eae9565db5fd88a38343423422cfcc92a3fac33f
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936816"
---
# <a name="quickstart-call-your-bing-custom-search-endpoint-using-c"></a>Inicio rápido: Llamada al punto de conexión de Bing Custom Search con C# 

Use este inicio rápido para aprender cómo solicitar los resultados de búsqueda de la instancia de Bing Custom Search. Si bien esta aplicación está escrita en C#, Bing Custom Search API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. El código fuente del ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/Search/BingCustomSearchv7.cs).

## <a name="prerequisites"></a>Prerrequisitos

- Una instancia de Bing Custom Search. Para más información, consulte [Inicio rápido: Creación de la primera instancia de Bing Custom Search](quick-start.md).
- [Microsoft .NET Core](https://www.microsoft.com/net/download/core).
- Cualquier edición de [Visual Studio 2019 o versiones posteriores](https://www.visualstudio.com/downloads/).
- Si usa Linux o MacOS, esta aplicación puede ejecutarse con [Mono](https://www.mono-project.com/).
- El paquete [Bing Custom Search](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.CustomSearch/2.0.0) de NuGet. 

   Para instalar este paquete en Visual Studio: 
     1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet**. 
     2. Busque y seleccione *Microsoft.Azure.CognitiveServices.Search.CustomSearch* y, a continuación, instale el paquete.

   Al instalar el paquete NuGet de Bing Custom Search, Visual Studio también instala los siguientes paquetes:
     - **Microsoft.Rest.ClientRuntime**
     - **Microsoft.Rest.ClientRuntime.Azure**
     - **Newtonsoft.Json**


[!INCLUDE [cognitive-services-bing-custom-search-prerequisites](../../../includes/cognitive-services-bing-custom-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

1. Cree una aplicación de consola en C# mediante Visual Studio. Agregue los siguientes paquetes al proyecto:

    ```csharp
    using System;
    using System.Net.Http;
    using System.Web;
    using Newtonsoft.Json;
    ```

2. Cree las siguientes clases para almacenar los resultados de búsqueda devueltos por Bing Custom Search API:

    ```csharp
    public class BingCustomSearchResponse {        
        public string _type{ get; set; }            
        public WebPages webPages { get; set; }
    }

    public class WebPages {
        public string webSearchUrl { get; set; }
        public int totalEstimatedMatches { get; set; }
        public WebPage[] value { get; set; }        
    }

    public class WebPage {
        public string name { get; set; }
        public string url { get; set; }
        public string displayUrl { get; set; }
        public string snippet { get; set; }
        public DateTime dateLastCrawled { get; set; }
        public string cachedPageUrl { get; set; }
    }
    ```

3. En el método principal del proyecto, cree las siguientes variables para la clave de suscripción de Bing Custom Search API, el identificador de configuración personalizada de su instancia de búsqueda y un término de búsqueda:

    ```csharp
    var subscriptionKey = "YOUR-SUBSCRIPTION-KEY";
    var customConfigId = "YOUR-CUSTOM-CONFIG-ID";
    var searchTerm = args.Length > 0 ? args[0]:"microsoft";
    ```

4. Construya la dirección URL de solicitud anexando el término de búsqueda al parámetro de consulta `q=`, y el identificador de configuración personalizada de la instancia de la búsqueda al parámetro `customconfig=`. Separe los parámetros con un carácter de "y" comercial (`&`). Para el valor de variable `url` puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```csharp
    var url = "https://api.cognitive.microsoft.com/bingcustomsearch/v7.0/search?" +
                "q=" + searchTerm + "&" +
                "customconfig=" + customConfigId;
    ```

## <a name="send-and-receive-a-search-request"></a>Envío y recepción de una solicitud de búsqueda 

1. Cree una solicitud de cliente y agregue su clave de suscripción al encabezado `Ocp-Apim-Subscription-Key`.

    ```csharp
    var client = new HttpClient();
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
    ```

2. Realice la solicitud de búsqueda y obtenga la respuesta como una cadena JSON.

    ```csharp
    var httpResponseMessage = client.GetAsync(url).Result;
    var responseContent = httpResponseMessage.Content.ReadAsStringAsync().Result;
    BingCustomSearchResponse response = JsonConvert.DeserializeObject<BingCustomSearchResponse>(responseContent);
    ```
## <a name="process-and-view-the-results"></a>Procese y visualice los resultados

- Recorre en iteración el objeto de respuesta para mostrar información sobre cada resultado de búsqueda, incluido el nombre, la dirección URL y la fecha en que la página web se rastreó por última vez.

    ```csharp
    for(int i = 0; i < response.webPages.value.Length; i++) {                
        var webPage = response.webPages.value[i];
        
        Console.WriteLine("name: " + webPage.name);
        Console.WriteLine("url: " + webPage.url);                
        Console.WriteLine("displayUrl: " + webPage.displayUrl);
        Console.WriteLine("snippet: " + webPage.snippet);
        Console.WriteLine("dateLastCrawled: " + webPage.dateLastCrawled);
        Console.WriteLine();
    }
    Console.WriteLine("Press any key to exit...");
    Console.ReadKey();
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una página web de Custom Search](./tutorials/custom-search-web-page.md)
