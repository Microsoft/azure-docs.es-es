---
title: Usar clasificaciones para mostrar los resultados de búsqueda
titleSuffix: Azure Cognitive Services
description: Muestra cómo utilizar la respuesta de RankingResponse de Bing para mostrar los resultados de la búsqueda en orden de clasificación.
services: cognitive-services
author: aahill
manager: nitinme
ms.assetid: 2575A80C-FC74-4631-AE5D-8101CF2591D3
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: tutorial
ms.date: 06/24/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: 8bfd7b6e5c9a2a7e3d9ed750e544036f3874271f
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88933229"
---
# <a name="build-a-console-app-search-client-in-c"></a>Compilación de un cliente de búsqueda de aplicación de consola en C#

En este tutorial se muestra cómo compilar una aplicación de consola .NET Core sencilla que permite a los usuarios consultar Bing Web Search API y mostrar los resultados en orden de clasificación.

En este tutorial se muestra cómo realizar las siguientes acciones:

- Realizar una consulta simple en Bing Web Search API
- Mostrar resultados de la consulta en orden de clasificación

## <a name="prerequisites"></a>Requisitos previos

Para seguir el tutorial, necesitará:

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="cree un recurso de Bing Search"  target="_blank">crear un recurso de Bing Search<span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
* El [entorno de desarrollo integrado de Visual Studio](https://www.visualstudio.com/downloads/).

## <a name="create-a-new-console-app-project"></a>Creación de un proyecto de aplicación de consola

Cree un proyecto en Visual Studio con `Ctrl`+`Shift`+`N`.

En el cuadro de diálogo **Nuevo proyecto**, haga clic en **Visual C# > Escritorio clásico de Windows > Aplicación de consola (.NET Framework)** .

Asigne a la aplicación el nombre **MyConsoleSearchApp** y haga clic en **Aceptar**.

## <a name="add-the-jsonnet-nuget-package-to-the-project"></a>Adición del paquete NuGet JSON.net al proyecto

JSON.net permite trabajar con las respuestas JSON devueltas por la API. Agregue el paquete NuGet al proyecto:

- En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet…**
- En la pestaña **Examinar**, busque `Newtonsoft.Json`. Seleccione la versión más reciente y haga clic en **Instalar**.
- Haga clic en el botón **Aceptar** situado en la ventana **Revisar cambios**.
- Cierre la pestaña de Visual Studio denominada **NuGet: MyConsoleSearchApp**.

## <a name="add-a-reference-to-systemweb"></a>Adición de una referencia a System.Web

Este tutorial se basa en el ensamblado `System.Web`. Agregue al proyecto una referencia a este ensamblado:

- En el **Explorador de soluciones**, haga clic con el botón derecho en **Referencias** y seleccione **Agregar referencia…**
- Seleccione **Ensamblados > Marco**, a continuación, desplácese hacia abajo y marque **System.Web**
- Seleccione **Aceptar**.

## <a name="add-some-necessary-using-statements"></a>Adición de algunas instrucciones using necesarias

El código de este tutorial requiere tres instrucciones using adicionales. Agregue estas instrucciones a continuación de las instrucciones `using` existentes, arriba de **Program.cs**:

```csharp
using System.Web;
using System.Net.Http;
```

## <a name="ask-the-user-for-a-query"></a>Pedido de una consulta al usuario

En el **Explorador de soluciones**, abra **Program.cs**. Actualice el método `Main()`:

```csharp
static void Main()
{
    // Get the user's query
    Console.Write("Enter Bing query: ");
    string userQuery = Console.ReadLine();
    Console.WriteLine();

    // Run the query and display the results
    RunQueryAndDisplayResults(userQuery);

    // Prevent the console window from closing immediately
    Console.WriteLine("\nHit ENTER to exit...");
    Console.ReadLine();
}
```

Este método:

- Pide una consulta al usuario
- Llama a `RunQueryAndDisplayResults(userQuery)` para ejecutar la consulta y mostrar los resultados
- Espera la entrada del usuario para evitar que la ventana de la consola se cierre inmediatamente.

## <a name="search-for-query-results-using-the-bing-web-search-api"></a>Búsqueda de resultados de la consulta mediante Bing Web Search API

A continuación, agregue un método que consulte la API y muestra los resultados:

```csharp
static void RunQueryAndDisplayResults(string userQuery)
{
    try
    {
        // Create a query
        var client = new HttpClient();
        client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "<YOUR_SUBSCRIPTION_KEY_GOES_HERE>");
        var queryString = HttpUtility.ParseQueryString(string.Empty);
        queryString["q"] = userQuery;
        var query = "https://api.cognitive.microsoft.com/bing/v7.0/search?" + queryString;

        // Run the query
        HttpResponseMessage httpResponseMessage = client.GetAsync(query).Result;

        // Deserialize the response content
        var responseContentString = httpResponseMessage.Content.ReadAsStringAsync().Result;
        Newtonsoft.Json.Linq.JObject responseObjects = Newtonsoft.Json.Linq.JObject.Parse(responseContentString);

        // Handle success and error codes
        if (httpResponseMessage.IsSuccessStatusCode)
        {
            DisplayAllRankedResults(responseObjects);
        }
        else
        {
            Console.WriteLine($"HTTP error status code: {httpResponseMessage.StatusCode.ToString()}");
        }
    }
    catch (Exception e)
    {
        Console.WriteLine(e.Message);
    }
}
```

Este método:

- Crea un `HttpClient` para consultar Web Search API
- Establece el encabezado HTTP `Ocp-Apim-Subscription-Key`, que Bing usa para autenticar la solicitud
- Ejecuta la solicitud y usa JSON.net para deserializar los resultados
- Llama a `DisplayAllRankedResults(responseObjects)` para mostrar todos los resultados en orden de clasificación

Asegúrese de establecer el valor de `Ocp-Apim-Subscription-Key` en su clave de suscripción.

## <a name="display-ranked-results"></a>Visualización de resultados en orden de clasificación

Antes de mostrar cómo visualizar los resultados en orden de clasificación, eche un vistazo a una respuesta de búsqueda de web de ejemplo:

```json
{
    "_type" : "SearchResponse",
    "webPages" : {
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=70BE289346...",
        "totalEstimatedMatches" : 982000,
        "value" : [{
            "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0",
            "name" : "Contoso Sailing Club - Seattle",
            "url" : "https:\/\/www.bing.com\/cr?IG=70BE289346ED4594874FE...",
            "displayUrl" : "https:\/\/contososailingsea...",
            "snippet" : "Come sail with Contoso in Seattle...",
            "dateLastCrawled" : "2017-04-07T02:25:00"
        },
        {
            "id" : "https:\/\/api.cognitive.microsoft.com\/api\/7\/#WebPages.6",
            "name" : "Contoso Sailing Lessons - Official Site",
            "url" : "http:\/\/www.bing.com\/cr?IG=70BE289346ED4594874FE...",
            "displayUrl" : "https:\/\/www.constososailinglessonsseat...",
            "snippet" : "Contoso sailing lessons in Seattle...",
            "dateLastCrawled" : "2017-04-09T14:30:00"
        },

        ...

        ],
        "someResultsRemoved" : true
    },
    "relatedSearches" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/7\/#RelatedSearches",
        "value" : [{
            "text" : "sailing lessons",
            "displayText" : "sailing lessons",
            "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=70BE289346E..."
        }

        ...

        ]
    },
    "rankingResponse" : {
        "mainline" : {
            "items" : [{
                "answerType" : "WebPages",
                "resultIndex" : 0,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 1,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.1"
                }
            }

            ...

            ]
        },
        "sidebar" : {
            "items" : [{
                "answerType" : "RelatedSearches",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#RelatedSearches"
                }
            }]
        }
    }
}
```

El objeto JSON `rankingResponse` ([documentación](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#rankingresponse)) describe el orden de presentación adecuado de los resultados de la búsqueda. Incluye uno o varios de los grupos clasificados por orden de prioridad siguientes:

- `pole`: los resultados de la búsqueda para obtener el tratamiento más visible (por ejemplo, se muestran por encima de la línea principal y la barra lateral).
- `mainline`: Los resultados de la búsqueda se muestran en la línea principal.
- `sidebar`: los resultados de la búsqueda que se muestran en la barra lateral. Si no hay ninguna barra lateral, los resultados se muestran debajo de la línea principal.

El código JSON de respuesta de clasificación puede incluir uno o varios de los grupos.

En **Program.cs**, agregue el método siguiente para mostrar los resultados en el orden de clasificación correcto:

```csharp
static void DisplayAllRankedResults(Newtonsoft.Json.Linq.JObject responseObjects)
{
    string[] rankingGroups = new string[] { "pole", "mainline", "sidebar" };

    // Loop through the ranking groups in priority order
    foreach (string rankingName in rankingGroups)
    {
        Newtonsoft.Json.Linq.JToken rankingResponseItems = responseObjects.SelectToken($"rankingResponse.{rankingName}.items");
        if (rankingResponseItems != null)
        {
            foreach (Newtonsoft.Json.Linq.JObject rankingResponseItem in rankingResponseItems)
            {
                Newtonsoft.Json.Linq.JToken resultIndex;
                rankingResponseItem.TryGetValue("resultIndex", out resultIndex);
                var answerType = rankingResponseItem.Value<string>("answerType");
                switch (answerType)
                {
                    case "WebPages":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("webPages.value"), "WebPage", "name", "url", "displayUrl", "snippet");
                        break;
                    case "News":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("news.value"), "News", "name", "url", "description");
                        break;
                    case "Images":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("images.value"), "Image", "thumbnailUrl");
                        break;
                    case "Videos":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("videos.value"), "Video", "embedHtml");
                        break;
                    case "RelatedSearches":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("relatedSearches.value"), "RelatedSearch", "displayText", "webSearchUrl");
                        break;
                }
            }
        }
    }
}
```

Este método:

- Recorre los grupos `rankingResponse` que contiene la respuesta
- Muestra los elementos de cada grupo mediante una llamada a `DisplaySpecificResults(...)`

En **Program.cs**, agregue los dos métodos siguientes:

```csharp
static void DisplaySpecificResults(Newtonsoft.Json.Linq.JToken resultIndex, Newtonsoft.Json.Linq.JToken items, string title, params string[] fields)
{
    if (resultIndex == null)
    {
        foreach (Newtonsoft.Json.Linq.JToken item in items)
        {
            DisplayItem(item, title, fields);
        }
    }
    else
    {
        DisplayItem(items.ElementAt((int)resultIndex), title, fields);
    }
}

static void DisplayItem(Newtonsoft.Json.Linq.JToken item, string title, string[] fields)
{
    Console.WriteLine($"{title}: ");
    foreach( string field in fields )
    {
        Console.WriteLine($"- {field}: {item[field]}");
    }
    Console.WriteLine();
}
```

Estos métodos funcionan conjuntamente para generar los resultados de la búsqueda en la consola.

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación. La salida debe tener una apariencia similar a la siguiente:

```
Enter Bing query: sailing lessons seattle

WebPage:
- name: Contoso Sailing Club - Seattle
- url: https://www.bing.com/cr?IG=70BE289346ED4594874FE...
- displayUrl: https://contososailingsea....
- snippet: Come sail with Contoso in Seattle...

WebPage:
- name: Contoso Sailing Lessons Seattle - Official Site
- url: http://www.bing.com/cr?IG=70BE289346ED4594874FE...
- displayUrl: https://www.constososailinglessonsseat...
- snippet: Contoso sailing lessons in Seattle...

...
```

## <a name="next-steps"></a>Pasos siguientes

Leer más sobre el [uso de clasificaciones para mostrar resultados](rank-results.md).
