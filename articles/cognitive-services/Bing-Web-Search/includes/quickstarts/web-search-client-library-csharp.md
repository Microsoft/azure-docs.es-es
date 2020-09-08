---
title: Inicio rápido de la biblioteca cliente para C# de Bing Web Search
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/05/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: 3f8f7c41e4d18909ec4f483a0e37d83b4d05e79d
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88931101"
---
La biblioteca cliente de Bing Web Search facilita la integración de Bing Web Search en la aplicación de C#. En esta guía de inicio rápido, aprenderá a crear una instancia de un cliente, enviar una solicitud e imprimir la respuesta.

¿Desea ver el código ahora mismo? En GitHub se encuentran disponibles ejemplos de [bibliotecas cliente de Bing Search para .NET](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7).

## <a name="prerequisites"></a>Prerequisites
Estas son algunas cosas que necesitará antes de ejecutar esta guía de inicio rápido:

* [Visual Studio](https://visualstudio.microsoft.com/downloads/) o
* [Visual Studio Code 2017](https://code.visualstudio.com/download)
  * [C# para Visual Studio Code](https://visualstudio.microsoft.com/downloads/)
  * [Administrador de paquetes NuGet](https://github.com/jmrog/vscode-nuget-package-manager)
* [SDK de .NET Core](https://www.microsoft.com/net/download)

[!INCLUDE [bing-web-search-quickstart-signup](~/includes/bing-web-search-quickstart-signup.md)]

## <a name="create-a-project-and-install-dependencies"></a>Creación de un proyecto e instalación de dependencias

> [!TIP]
> Obtenga el código más reciente como solución de Visual Studio desde [GitHub](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/).

El primer paso es crear un proyecto de consola. Si necesita ayuda para configurar un proyecto de consola, consulte [Hola mundo, su primer programa (Guía de programación de C#)](https://docs.microsoft.com/dotnet/csharp/programming-guide/inside-a-program/hello-world-your-first-program). Para usar el SDK de Bing Web Search en la aplicación, necesita instalar `Microsoft.Azure.CognitiveServices.Search.WebSearch` con el Administrador de paquetes NuGet.

El [paquete del SDK de Web Search](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.WebSearch/1.2.0) también instala:

* Microsoft.Rest.ClientRuntime
* Microsoft.Rest.ClientRuntime.Azure
* Newtonsoft.Json

## <a name="declare-dependencies"></a>Declaración de dependencias

Abra el proyecto en Visual Studio o Visual Studio Code e importe estas dependencias:

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Azure.CognitiveServices.Search.WebSearch;
using Microsoft.Azure.CognitiveServices.Search.WebSearch.Models;
using System.Linq;
```

## <a name="create-project-scaffolding"></a>Creación del scaffolding del proyecto

Al crear el proyecto de consola, se han tenido que crear un espacio de nombres y una clase para la aplicación. El aspecto del programa debería ser similar a este ejemplo:

```csharp
namespace WebSearchSDK
{
    class YOUR_PROGRAM
    {

        // The code in the following sections goes here.

    }
}
```

En las siguientes secciones, vamos a crear una aplicación de ejemplo dentro de esta clase.

## <a name="construct-a-request"></a>Creación de una solicitud

Este código crea la consulta de búsqueda.

```csharp
public static async void WebResults(WebSearchClient client)
{
    try
    {
        var webData = await client.Web.SearchAsync(query: "Yosemite National Park");
        Console.WriteLine("Searching for \"Yosemite National Park\"");

        // Code for handling responses is provided in the next section...

    }
    catch (Exception ex)
    {
        Console.WriteLine("Encountered exception. " + ex.Message);
    }
}
```

## <a name="handle-the-response"></a>Control de la respuesta

A continuación, vamos a agregar un código para analizar la respuesta e imprimir los resultados. Se imprimen `Name` y `Url` para la primera página web, imagen, artículo de noticias y vídeo si están presentes en el objeto de respuesta.

```csharp
if (webData?.WebPages?.Value?.Count > 0)
{
    // find the first web page
    var firstWebPagesResult = webData.WebPages.Value.FirstOrDefault();

    if (firstWebPagesResult != null)
    {
        Console.WriteLine("Webpage Results # {0}", webData.WebPages.Value.Count);
        Console.WriteLine("First web page name: {0} ", firstWebPagesResult.Name);
        Console.WriteLine("First web page URL: {0} ", firstWebPagesResult.Url);
    }
    else
    {
        Console.WriteLine("Didn't find any web pages...");
    }
}
else
{
    Console.WriteLine("Didn't find any web pages...");
}

/*
 * Images
 * If the search response contains images, the first result's name
 * and url are printed.
 */
if (webData?.Images?.Value?.Count > 0)
{
    // find the first image result
    var firstImageResult = webData.Images.Value.FirstOrDefault();

    if (firstImageResult != null)
    {
        Console.WriteLine("Image Results # {0}", webData.Images.Value.Count);
        Console.WriteLine("First Image result name: {0} ", firstImageResult.Name);
        Console.WriteLine("First Image result URL: {0} ", firstImageResult.ContentUrl);
    }
    else
    {
        Console.WriteLine("Didn't find any images...");
    }
}
else
{
    Console.WriteLine("Didn't find any images...");
}

/*
 * News
 * If the search response contains news articles, the first result's name
 * and url are printed.
 */
if (webData?.News?.Value?.Count > 0)
{
    // find the first news result
    var firstNewsResult = webData.News.Value.FirstOrDefault();

    if (firstNewsResult != null)
    {
        Console.WriteLine("\r\nNews Results # {0}", webData.News.Value.Count);
        Console.WriteLine("First news result name: {0} ", firstNewsResult.Name);
        Console.WriteLine("First news result URL: {0} ", firstNewsResult.Url);
    }
    else
    {
        Console.WriteLine("Didn't find any news articles...");
    }
}
else
{
    Console.WriteLine("Didn't find any news articles...");
}

/*
 * Videos
 * If the search response contains videos, the first result's name
 * and url are printed.
 */
if (webData?.Videos?.Value?.Count > 0)
{
    // find the first video result
    var firstVideoResult = webData.Videos.Value.FirstOrDefault();

    if (firstVideoResult != null)
    {
        Console.WriteLine("\r\nVideo Results # {0}", webData.Videos.Value.Count);
        Console.WriteLine("First Video result name: {0} ", firstVideoResult.Name);
        Console.WriteLine("First Video result URL: {0} ", firstVideoResult.ContentUrl);
    }
    else
    {
        Console.WriteLine("Didn't find any videos...");
    }
}
else
{
    Console.WriteLine("Didn't find any videos...");
}
```

## <a name="declare-the-main-method"></a>Declaración del método main

En esta aplicación, el método main incluye código que crea instancias del cliente, valida `subscriptionKey` y llama a `WebResults`. Asegúrese de escribir una clave de suscripción válida para la cuenta de Azure antes de continuar.

```csharp
static void Main(string[] args)
{
    var client = new WebSearchClient(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

    WebResults(client);

    Console.WriteLine("Press any key to exit...");
    Console.ReadKey();
}
```

## <a name="run-the-application"></a>Ejecución de la aplicación

¡Vamos a ejecutar la aplicación!

```console
dotnet run
```

## <a name="define-functions-and-filter-results"></a>Definición de funciones y filtro de resultados

Ahora que ha realizado su primera llamada a Bing Web Search API, veamos algunas funciones que resaltan la funcionalidad del SDK para perfeccionar las consultas y filtrar los resultados. Cada función se puede agregar a la aplicación de C# creada en la sección anterior.

### <a name="limit-the-number-of-results-returned-by-bing"></a>Limitación del número de resultados devueltos por Bing

Este ejemplo utiliza los parámetros `count` y `offset` para limitar el número de resultados devueltos para "Best restaurants in Seattle" ("Los mejores restaurantes de Seattle"). `Name` y `Url` se imprimen para el primer resultado.

1. Agregue este código a su proyecto de consola:

    ```csharp
    public static async void WebResultsWithCountAndOffset(WebSearchClient client)
    {
        try
        {
            var webData = await client.Web.SearchAsync(query: "Best restaurants in Seattle", offset: 10, count: 20);
            Console.WriteLine("\r\nSearching for \" Best restaurants in Seattle \"");

            if (webData?.WebPages?.Value?.Count > 0)
            {
                var firstWebPagesResult = webData.WebPages.Value.FirstOrDefault();

                if (firstWebPagesResult != null)
                {
                    Console.WriteLine("Web Results #{0}", webData.WebPages.Value.Count);
                    Console.WriteLine("First web page name: {0} ", firstWebPagesResult.Name);
                    Console.WriteLine("First web page URL: {0} ", firstWebPagesResult.Url);
                }
                else
                {
                    Console.WriteLine("Couldn't find first web result!");
                }
            }
            else
            {
                Console.WriteLine("Didn't see any Web data..");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Encountered exception. " + ex.Message);
        }
    }
    ```

2. Agregue `WebResultsWithCountAndOffset` a `main`:

    ```csharp
    static void Main(string[] args)
    {
        var client = new WebSearchClient(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

        WebResults(client);
        // Search with count and offset...
        WebResultsWithCountAndOffset(client);  

        Console.WriteLine("Press any key to exit...");
        Console.ReadKey();
    }
    ```

3. Ejecute la aplicación.

### <a name="filter-for-news"></a>Filtro para noticias

Este ejemplo utiliza el parámetro `response_filter` para filtrar los resultados de búsqueda. Los resultados de búsqueda devueltos se limitan a los artículos de noticias para "Microsoft". `Name` y `Url` se imprimen para el primer resultado.

1. Agregue este código a su proyecto de consola:

    ```csharp
    public static async void WebSearchWithResponseFilter(WebSearchClient client)
    {
        try
        {
            IList<string> responseFilterstrings = new List<string>() { "news" };
            var webData = await client.Web.SearchAsync(query: "Microsoft", responseFilter: responseFilterstrings);
            Console.WriteLine("\r\nSearching for \" Microsoft \" with response filter \"news\"");

            if (webData?.News?.Value?.Count > 0)
            {
                var firstNewsResult = webData.News.Value.FirstOrDefault();

                if (firstNewsResult != null)
                {
                    Console.WriteLine("News Results #{0}", webData.News.Value.Count);
                    Console.WriteLine("First news result name: {0} ", firstNewsResult.Name);
                    Console.WriteLine("First news result URL: {0} ", firstNewsResult.Url);
                }
                else
                {
                    Console.WriteLine("Couldn't find first News results!");
                }
            }
            else
            {
                Console.WriteLine("Didn't see any News data..");
            }

        }
        catch (Exception ex)
        {
            Console.WriteLine("Encountered exception. " + ex.Message);
        }
    }
    ```

2. Agregue `WebResultsWithCountAndOffset` a `main`:

    ```csharp
    static void Main(string[] args)
    {
        var client = new WebSearchClient(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

        WebResults(client);
        // Search with count and offset...
        WebResultsWithCountAndOffset(client);  
        // Search with news filter...
        WebSearchWithResponseFilter(client);

        Console.WriteLine("Press any key to exit...");
        Console.ReadKey();
    }
    ```

3. Ejecute la aplicación.

### <a name="use-safe-search-answer-count-and-the-promote-filter"></a>Uso de búsqueda segura, recuento de respuestas y filtro de promoción

Este ejemplo utiliza los parámetros `answer_count`, `promote` y `safe_search`para filtrar los resultados de búsqueda para "Music Videos" ("Vídeos musicales"). `Name` y `ContentUrl` se muestran para el primer resultado.

1. Agregue este código a su proyecto de consola:

    ```csharp
    public static async void WebSearchWithAnswerCountPromoteAndSafeSearch(WebSearchClient client)
    {
        try
        {
            IList<string> promoteAnswertypeStrings = new List<string>() { "videos" };
            var webData = await client.Web.SearchAsync(query: "Music Videos", answerCount: 2, promote: promoteAnswertypeStrings, safeSearch: SafeSearch.Strict);
            Console.WriteLine("\r\nSearching for \"Music Videos\"");

            if (webData?.Videos?.Value?.Count > 0)
            {
                var firstVideosResult = webData.Videos.Value.FirstOrDefault();

                if (firstVideosResult != null)
                {
                    Console.WriteLine("Video Results # {0}", webData.Videos.Value.Count);
                    Console.WriteLine("First Video result name: {0} ", firstVideosResult.Name);
                    Console.WriteLine("First Video result URL: {0} ", firstVideosResult.ContentUrl);
                }
                else
                {
                    Console.WriteLine("Couldn't find videos results!");
                }
            }
            else
            {
                Console.WriteLine("Didn't see any data..");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Encountered exception. " + ex.Message);
        }
    }
    ```

2. Agregue `WebResultsWithCountAndOffset` a `main`:

    ```csharp
    static void Main(string[] args)
    {
        var client = new WebSearchClient(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

        WebResults(client);
        // Search with count and offset...
        WebResultsWithCountAndOffset(client);  
        // Search with news filter...
        WebSearchWithResponseFilter(client);
        // Search with answer count, promote, and safe search
        WebSearchWithAnswerCountPromoteAndSafeSearch(client);

        Console.WriteLine("Press any key to exit...");
        Console.ReadKey();
    }
    ```

3. Ejecute la aplicación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con este proyecto, asegúrese de quitar la clave de suscripción desde el código de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Ejemplos del SDK de Cognitive Services para Node.js](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/)
                                    