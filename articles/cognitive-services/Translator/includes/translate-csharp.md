---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 08/06/2019
ms.author: erhopf
ms.custom: devx-track-csharp
ms.openlocfilehash: 0c74c3709848e4296f07844278b076fdc7ffa9bc
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88921443"
---
[!INCLUDE [Prerequisites](prerequisites-csharp.md)]

[!INCLUDE [Set up and use environment variables](setup-env-variables.md)]

## <a name="create-a-net-core-project"></a>Creación de un proyecto de .NET Core

Abra un nuevo símbolo del sistema (o una sesión de terminal) y ejecute estos comandos:

```console
dotnet new console -o translate-sample
cd translate-sample
```

El primer comando hace dos cosas. Crea una nueva aplicación de consola .NET y crea un directorio denominado `translate-sample`. El segundo comando cambia al directorio del proyecto.

A continuación, deberá instalar Json.Net. En el directorio del proyecto, ejecute:

```console
dotnet add package Newtonsoft.Json --version 11.0.2
```

## <a name="select-the-c-language-version"></a>Selección de la versión del lenguaje C#

Esta guía de inicio rápido requiere C# 7.1 o posterior. Hay algunas maneras de cambiar la versión de C# del proyecto. En esta guía le mostraremos cómo ajustar el archivo `translate-sample.csproj`. Para todas las opciones disponibles, como el cambio de idioma en Visual Studio, consulte [Selección de la versión del lenguaje C#](https://docs.microsoft.com/dotnet/csharp/language-reference/configure-language-version).

Abra el proyecto y, después, abra `translate-sample.csproj`. Asegúrese de que `LangVersion` está establecido en 7.1 o posterior. Si no hay un grupo de propiedades para la versión del lenguaje, añada estas líneas:

```xml
<PropertyGroup>
   <LangVersion>7.1</LangVersion>
</PropertyGroup>
```

## <a name="add-required-namespaces-to-your-project"></a>Incorporación de los espacios de nombres necesarios al proyecto

El comando `dotnet new console` que ejecutó anteriormente creó un proyecto que incluía `Program.cs`. En este archivo es donde deberá colocar el código de la aplicación. Abra `Program.cs` y reemplace las instrucciones using existentes. Estas instrucciones garantizan que tiene acceso a todos los tipos necesarios para compilar y ejecutar la aplicación de ejemplo.

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
// Install Newtonsoft.Json with NuGet
using Newtonsoft.Json;
```

## <a name="create-classes-for-the-json-response"></a>Creación de clases para la respuesta JSON

A continuación, vamos a crear un conjunto de clases que se utilizan al deserializar la respuesta JSON devuelta por Traductor.

```csharp
/// <summary>
/// The C# classes that represents the JSON returned by the Translator.
/// </summary>
public class TranslationResult
{
    public DetectedLanguage DetectedLanguage { get; set; }
    public TextResult SourceText { get; set; }
    public Translation[] Translations { get; set; }
}

public class DetectedLanguage
{
    public string Language { get; set; }
    public float Score { get; set; }
}

public class TextResult
{
    public string Text { get; set; }
    public string Script { get; set; }
}

public class Translation
{
    public string Text { get; set; }
    public TextResult Transliteration { get; set; }
    public string To { get; set; }
    public Alignment Alignment { get; set; }
    public SentenceLength SentLen { get; set; }
}

public class Alignment
{
    public string Proj { get; set; }
}

public class SentenceLength
{
    public int[] SrcSentLen { get; set; }
    public int[] TransSentLen { get; set; }
}
```

## <a name="get-subscription-information-from-environment-variables"></a>Obtención de información de la suscripción a partir de las variables de entorno

Agregue las líneas siguientes a la clase `Program`. Estas líneas leen la clave de suscripción y el punto de conexión de las variables de entorno y devuelven un error si se produce algún problema.

```csharp
private const string key_var = "TRANSLATOR_TEXT_SUBSCRIPTION_KEY";
private static readonly string subscriptionKey = Environment.GetEnvironmentVariable(key_var);

private const string endpoint_var = "TRANSLATOR_TEXT_ENDPOINT";
private static readonly string endpoint = Environment.GetEnvironmentVariable(endpoint_var);

static Program()
{
    if (null == subscriptionKey)
    {
        throw new Exception("Please set/export the environment variable: " + key_var);
    }
    if (null == endpoint)
    {
        throw new Exception("Please set/export the environment variable: " + endpoint_var);
    }
}
// The code in the next section goes here.
```

## <a name="create-a-function-to-translate-text"></a>Creación de una función para traducir texto

En la clase `Program`, cree una función asincrónica denominada `TranslateTextRequest()`. Esta función toma cuatro argumentos: `subscriptionKey`, `host`, `route` y `inputText`.

```csharp
// This sample requires C# 7.1 or later for async/await.
// Async call to the Translator
static public async Task TranslateTextRequest(string subscriptionKey, string endpoint, string route, string inputText)
{
  /*
   * The code for your call to the translation service will be added to this
   * function in the next few sections.
   */
}
```

## <a name="serialize-the-translation-request"></a>Serialización de la solicitud de traducción

A continuación, se debe crear y serializar el objeto JSON que incluye el texto que desea traducir. Tenga en cuenta que puede pasar más de un objeto en `body`.

```csharp
object[] body = new object[] { new { Text = inputText } };
var requestBody = JsonConvert.SerializeObject(body);
```

## <a name="instantiate-the-client-and-make-a-request"></a>Creación de una instancia de cliente y realización de una solicitud

Estas líneas crean instancias de `HttpClient` y `HttpRequestMessage`:

```csharp
using (var client = new HttpClient())
using (var request = new HttpRequestMessage())
{
  // In the next few sections you'll add code to construct the request.
}
```

## <a name="construct-the-request-and-print-the-response"></a>Construcción de la solicitud e impresión de la respuesta

Dentro de `HttpRequestMessage`:

* Declarará el método HTTP
* Construirá el URI de solicitud
* Insertará el cuerpo de la solicitud (el objeto JSON serializado)
* Agregará los encabezados necesarios
* Realizará una solicitud asincrónica
* Imprimirá la respuesta mediante las clases que creó anteriormente.

Agregue este código a `HttpRequestMessage`:

```csharp
// Build the request.
// Set the method to Post.
request.Method = HttpMethod.Post;
// Construct the URI and add headers.
request.RequestUri = new Uri(endpoint + route);
request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
request.Headers.Add("Ocp-Apim-Subscription-Key", subscriptionKey);

// Send the request and get response.
HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
// Read response as a string.
string result = await response.Content.ReadAsStringAsync();
// Deserialize the response using the classes created earlier.
TranslationResult[] deserializedOutput = JsonConvert.DeserializeObject<TranslationResult[]>(result);
// Iterate over the deserialized results.
foreach (TranslationResult o in deserializedOutput)
{
    // Print the detected input language and confidence score.
    Console.WriteLine("Detected input language: {0}\nConfidence score: {1}\n", o.DetectedLanguage.Language, o.DetectedLanguage.Score);
    // Iterate over the results and print each translation.
    foreach (Translation t in o.Translations)
    {
        Console.WriteLine("Translated to {0}: {1}", t.To, t.Text);
    }
}
```

Si usa una suscripción a varios servicios de Cognitive Services, también debe incluir `Ocp-Apim-Subscription-Region` en los parámetros de la solicitud. [Más información sobre la autenticación con la suscripción a varios servicios](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication).

## <a name="put-it-all-together"></a>Colocación de todo junto

El último paso consiste en llamar a `TranslateTextRequest()` en la función `Main`. En este ejemplo, vamos a traducir al alemán (`de`), italiano (`it`), japonés (`ja`) y tailandés (`th`). Busque `static void Main(string[] args)` y reemplácelo por este código:

```csharp
static async Task Main(string[] args)
{
    // This is our main function.
    // Output languages are defined in the route.
    // For a complete list of options, see API reference.
    // https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-translate
    string route = "/translate?api-version=3.0&to=de&to=it&to=ja&to=th";
    // Prompts you for text to translate. If you'd prefer, you can
    // provide a string as textToTranslate.
    Console.Write("Type the phrase you'd like to translate? ");
    string textToTranslate = Console.ReadLine();
    await TranslateTextRequest(subscriptionKey, endpoint, route, textToTranslate);
    Console.WriteLine("Press any key to continue.");
    Console.ReadKey();
}
```

Observará que en `Main`, está declarando `subscriptionKey`, `endpoint` y `route`. Además, está pidiendo al usuario que indique `Console.Readline()` y asignando el valor a `textToTranslate`.

## <a name="run-the-sample-app"></a>Ejecutar la aplicación de ejemplo

Eso es todo, ya está listo para ejecutar la aplicación de ejemplo. Desde la línea de comandos (o sesión de terminal), vaya al directorio del proyecto y ejecute:

```console
dotnet run
```

## <a name="sample-response"></a>Respuesta de muestra

Después de ejecutar el ejemplo, debería ver lo siguiente impreso en el terminal:

```bash
Detected input language: en
Confidence score: 1

Translated to de: Hallo Welt!
Translated to it: Salve, mondo!
Translated to ja: ハローワールド！
Translated to th: หวัดดีชาวโลก!
```

Este mensaje se crea a partir del JSON sin formato, y tendrá un aspecto similar al siguiente:

```json
[
  {
    "detectedLanguage": {
      "language": "en",
      "score": 1.0
    },
    "translations": [
      {
        "text": "Hallo Welt!",
        "to": "de"
      },
      {
        "text": "Salve, mondo!",
        "to": "it"
      },
      {
        "text": "ハローワールド！",
        "to": "ja"
      },
      {
        "text": "หวัดดีชาวโลก!",
        "to": "th"
      }
    ]
  }
]
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Asegúrese de quitar cualquier información confidencial del código fuente de la aplicación de ejemplo como, por ejemplo, las claves de suscripción.

## <a name="next-steps"></a>Pasos siguientes

Eche un vistazo a la referencia de API para comprender todo lo que puede hacer con Traductor.

> [!div class="nextstepaction"]
> [Referencia de API](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference)
