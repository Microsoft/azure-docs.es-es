---
title: 'Inicio rápido: Revisión ortográfica con la API REST y C#: Bing Spell Check'
titleSuffix: Azure Cognitive Services
description: Introducción al uso de la API REST de Bing Spell Check para la revisión ortográfica y gramatical.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-spell-check
ms.topic: quickstart
ms.date: 05/21/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: df77ea73338f37925c0b59a8838c31617ff22ad8
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919410"
---
# <a name="quickstart-check-spelling-with-the-bing-spell-check-rest-api-and-c"></a>Inicio rápido: Revisión ortográfica con la API REST de Bing Spell Check y C#

Use este inicio rápido para realizar la primera llamada a la API REST de Bing Spell Check. Esta sencilla aplicación de C# envía una solicitud a la API y devuelve una lista de correcciones sugeridas. 

Aunque esta aplicación está escrita en C#, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. El código fuente de esta aplicación está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/Search/BingAutosuggestv7.cs).

## <a name="prerequisites"></a>Requisitos previos

* Cualquier edición de [Visual Studio 2017 o versiones posteriores](https://www.visualstudio.com/downloads/).
* El paquete NuGet Newtonsoft.Json. 
     
   Para instalar este paquete en Visual Studio:

     1. En el **Explorador de soluciones**, haga clic con el botón derecho en el archivo Solution.
     1. Seleccione **Administrar paquetes NuGet para la solución**.
     1. Busque *Newtonsoft.json* e instale el paquete.

* Si usa Linux o MacOS, puede ejecutar esta aplicación con [Mono](https://www.mono-project.com/).

[!INCLUDE [cognitive-services-bing-spell-check-signup-requirements](../../../../includes/cognitive-services-bing-spell-check-signup-requirements.md)]

## <a name="create-and-initialize-a-project"></a>Creación e inicialización de un proyecto

1. Cree una solución de consola denominada SpellCheckSample en Visual Studio. Después, agregue los siguientes espacios de nombres en el archivo de código principal:
    
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using Newtonsoft.Json;
    ```

2. Cree variables para el punto de conexión de API, la clave de suscripción y el texto cuya ortografía se va a revisar. Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```csharp
    namespace SpellCheckSample
    {
        class Program
        {
            static string host = "https://api.cognitive.microsoft.com";
            static string path = "/bing/v7.0/spellcheck?";
            static string key = "<ENTER-KEY-HERE>";
            //text to be spell-checked
            static string text = "Hollo, wrld!";
        }
    }
    ```

3. Cree una cadena para los parámetros de búsqueda: 

   1. Asigne el código de mercado al parámetro `mkt` con el operador `=`. El código de mercado es el código del país desde el que se realiza la solicitud. 

   1. Agregue el parámetro `mode` con el operador `&` y, a continuación, asigne el modo de revisión ortográfica. El modo puede ser `proof` (detecta la mayoría de los errores ortográficos y gramaticales) o `spell` (detecta la mayoría de los errores ortográficos, pero no todos los gramaticales).
    
    ```csharp
    static string params_ = "mkt=en-US&mode=proof";
    ```

## <a name="create-and-send-a-spell-check-request"></a>Creación y envío de una solicitud de revisión ortográfica

1. Cree una función asincrónica llamada `SpellCheck()` para enviar una solicitud a la API. Cree un objeto `HttpClient` y agregue su clave de suscripción al encabezado `Ocp-Apim-Subscription-Key`. Dentro de la función, siga los pasos siguientes.

    ```csharp
    async static void SpellCheck()
    {
        var client = new HttpClient();
        client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

        HttpResponseMessage response = null;
        // add the rest of the code snippets here (except for main())...
    }
    ```

2. Cree el URI para la solicitud mediante la anexión de su host, ruta de acceso y parámetros.
    
    ```csharp
    string uri = host + path + params_;
    ```

3. Cree una lista con un objeto `KeyValuePair` que contenga el texto y úsela para crear un objeto `FormUrlEncodedContent`. Establezca la información de encabezado y use `PostAsync()` para enviar la solicitud.

    ```csharp
    var values = new Dictionary<string, string>();
    values.Add("text", text);
    var content = new FormUrlEncodedContent(values);
    content.Headers.ContentType = new MediaTypeHeaderValue("application/x-www-form-urlencoded");
    response = await client.PostAsync(uri, new FormUrlEncodedContent(values));
    ```

## <a name="get-and-print-the-api-response"></a>Obtención e impresión de la respuesta de la API

### <a name="get-the-client-id-header"></a>Obtención del encabezado de identificador de cliente

Si la respuesta contiene un encabezado `X-MSEdge-ClientID`, obtenga el valor e imprímalo.

``` csharp
string client_id;
if (response.Headers.TryGetValues("X-MSEdge-ClientID", out IEnumerable<string> header_values))
{
    client_id = header_values.First();
    Console.WriteLine("Client ID: " + client_id);
}
```

### <a name="get-the-response"></a>Obtención de la respuesta

Obtenga la respuesta de la API. Deserialice el objeto JSON e imprímalo en la consola.

```csharp
string contentString = await response.Content.ReadAsStringAsync();

dynamic jsonObj = JsonConvert.DeserializeObject(contentString);
Console.WriteLine(jsonObj);
```

## <a name="call-the-spell-check-function"></a>Llamada a la función de revisión ortográfica

En la función `Main()` del proyecto, llame a `SpellCheck()`.

```csharp
static void Main(string[] args)
{
    SpellCheck();
    Console.ReadLine();
}
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Compile y ejecute el proyecto. Si usa Visual Studio, pulse **F5** para depurar el archivo.

## <a name="example-json-response"></a>Ejemplo de respuesta JSON

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
   "_type": "SpellCheck",
   "flaggedTokens": [
      {
         "offset": 0,
         "token": "Hollo",
         "type": "UnknownToken",
         "suggestions": [
            {
               "suggestion": "Hello",
               "score": 0.9115257530801
            },
            {
               "suggestion": "Hollow",
               "score": 0.858039839213461
            },
            {
               "suggestion": "Hallo",
               "score": 0.597385084464481
            }
         ]
      },
      {
         "offset": 7,
         "token": "wrld",
         "type": "UnknownToken",
         "suggestions": [
            {
               "suggestion": "world",
               "score": 0.9115257530801
            }
         ]
      }
   ]
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación web de una sola página](../tutorials/spellcheck.md)

- [¿Qué es Bing Spell Check API?](../overview.md)
- [Referencia de la API Bing Spell Check v7](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference)
