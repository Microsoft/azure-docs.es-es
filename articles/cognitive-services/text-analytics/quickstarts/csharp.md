---
title: 'Inicio rápido: Uso de C# para llamar a Text Analytics API'
titleSuffix: Azure Cognitive Services
description: Obtenga información y ejemplos de código que le ayuden a empezar a usar rápidamente Text Analytics API.
services: cognitive-services
author: raymondl
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: quickstart
ms.date: 04/29/2019
ms.author: assafi
ms.openlocfilehash: c521be03f884227116a21c8c5396d47cdd1ae253
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466535"
---
# <a name="quickstart-using-c-to-call-the-text-analytics-cognitive-service"></a>Inicio rápido: Uso de C# para llamar a Text Analytics de Cognitive Services
<a name="HOLTop"></a>

Use esta guía de inicio rápido para empezar a analizar el lenguaje con el SDK de Text Analytics para C#. Aunque la API de REST [Text Analytics](//go.microsoft.com/fwlink/?LinkID=759711) es compatible con la mayoría de los lenguajes de programación, el SDK proporciona una forma sencilla de integrar el servicio en sus aplicaciones. El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/samples/TextAnalytics).

Consulte las [definiciones de API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c7) para obtener la documentación técnica de las API.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [cognitive-services-text-analytics-signup-requirements](../../../../includes/cognitive-services-text-analytics-signup-requirements.md)]

También debe tener la [clave de acceso y punto de conexión](../How-tos/text-analytics-how-to-access-key.md) que se generó automáticamente durante el registro.

> [!Tip]
>  Aunque puede llamar a los [puntos de conexión HTTP](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c9) directamente desde C#, el SDK de Microsoft.Azure.CognitiveServices.Language, hace que resulte más fácil llamar al servicio sin tener que preocuparse sobre la serialización y deserialización de JSON.
>
> Algunos vínculos útiles:
> - [Página del SDK de NuGet](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.TextAnalytics)
> - [Código del SDK](https://github.com/Azure/azure-sdk-for-net/tree/master/src/SDKs/CognitiveServices/dataPlane/Language/TextAnalytics)

## <a name="create-the-visual-studio-solution-and-install-the-sdk"></a>Creación de la solución de Visual Studio e instalación del SDK

1. Cree un proyecto de aplicación de consola (.NET Core) en [Visual Studio](https://visualstudio.microsoft.com/vs/).
1. Haga clic con el botón derecho en la solución y seleccione **Administrar paquetes NuGet para la solución**
1. Seleccione la pestaña **Examinar** y busque **Microsoft.Azure.CognitiveServices.Language.TextAnalytics**.

## <a name="authenticate-your-credentials"></a>Autenticación con sus credenciales

1. Agregue las siguientes instrucciones `using` al archivo de clase principal (`Program.cs` de forma predeterminada).

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Net.Http;
    using System.Threading;
    using System.Threading.Tasks;

    using Microsoft.Azure.CognitiveServices.Language.TextAnalytics;
    using Microsoft.Azure.CognitiveServices.Language.TextAnalytics.Models;
    using Microsoft.Rest;
    ```

2. Cree una clase `ApiKeyServiceClientCredentials` para almacenar las credenciales y agregarlas a cada solicitud.

    ```csharp
    /// <summary>
    /// Allows authentication to the API using a basic apiKey mechanism
    /// </summary>
    class ApiKeyServiceClientCredentials : ServiceClientCredentials
    {
        private readonly string subscriptionKey;

        /// <summary>
        /// Creates a new instance of the ApiKeyServiceClientCredentails class
        /// </summary>
        /// <param name="subscriptionKey">The subscription key to authenticate and authorize as</param>
        public ApiKeyServiceClientCredentials(string subscriptionKey)
        {
            this.subscriptionKey = subscriptionKey;
        }

        /// <summary>
        /// Add the Basic Authentication Header to each outgoing request
        /// </summary>
        /// <param name="request">The outgoing request</param>
        /// <param name="cancellationToken">A token to cancel the operation</param>
        public override Task ProcessHttpRequestAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            if (request == null)
            {
                throw new ArgumentNullException("request");
            }

            request.Headers.Add("Ocp-Apim-Subscription-Key", this.subscriptionKey);
            return base.ProcessHttpRequestAsync(request, cancellationToken);
        }
    }
    ```

3. Actualice la clase `Program` y agregue un miembro de constante para su clave de suscripción de Text Analytics y otro para el punto de conexión de servicio. Recuerde que debe usar la región de Azure correcta para su suscripción de Text Analytics.

    ```csharp
    private const string SubscriptionKey = "enter-your-key-here";

    private const string Endpoint = "enter-your-service-endpoint-here"; // For example: "https://westus.api.cognitive.microsoft.com";
    ```
> [!Tip]
> Para una implementación segura de secretos en sistemas de producción, se recomienda usar [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/quick-create-net).
>

## <a name="create-a-text-analytics-client"></a>Creación de un cliente de Text Analytics

En la función `Main` del proyecto, llame al método de ejemplo que le gustaría invocar y pase los parámetros `Endpoint` y `SubscriptionKey` definidos.

```csharp
    public static void Main(string[] args)
    {
        var credentials = new ApiKeyServiceClientCredentials(SubscriptionKey);
        var client = new TextAnalyticsClient(credentials)
        {
            //Replace 'westus' with the correct region for your Text Analytics subscription
            Endpoint = "https://westus.api.cognitive.microsoft.com"
        };

        // Change console encoding to display non-ASCII characters
        Console.OutputEncoding = System.Text.Encoding.UTF8;
        SentimentAnalysisExample(client).Wait();
        // DetectLanguageExample(client).Wait();
        // RecognizeEntitiesExample(client).Wait();
        // KeyPhraseExtractionExample(client).Wait();
        Console.ReadLine();
    }
```

En las siguientes secciones se describe cómo llamar a cada una de las características de API.

## <a name="sentiment-analysis"></a>análisis de opiniones

1. Cree una función llamada `SentimentAnalysisExample()` que adopte el cliente que creó anteriormente.
2. Genere una lista de objetos `MultiLanguageInput`, que contenga los documentos que desea analizar.

    ```csharp
    public static async Task SentimentAnalysisExample(TextAnalyticsClient client)
    {
        // The documents to be analyzed. Add the language of the document. The ID can be any value.
        var inputDocuments = new MultiLanguageBatchInput(
            new List<MultiLanguageInput>
            {
                new MultiLanguageInput("en", "1", "I had the best day of my life."),
                new MultiLanguageInput("en", "2", "This was a waste of my time. The speaker put me to sleep."),
                new MultiLanguageInput("es", "3", "No tengo dinero ni nada que dar..."),
                new MultiLanguageInput("it", "4", "L'hotel veneziano era meraviglioso. È un bellissimo pezzo di architettura."),
            });
        //...
    }
    ```

3. En la misma función, llame a `client.SentimentAsync()` y obtenga el resultado. A continuación, itere por los resultados e imprima el identificador de cada documento y la puntuación de opiniones. Una puntuación cercana a 0 indica una opinión negativa, mientras que una puntuación cercana a 1 indica una opinión positiva.

    ```csharp
    var result = await client.SentimentAsync(false, inputDocuments);

    // Printing sentiment results
    foreach (var document in result.Documents)
    {
        Console.WriteLine($"Document ID: {document.Id} , Sentiment Score: {document.Score:0.00}");
    }
    ```

### <a name="output"></a>Salida

```console
Document ID: 1 , Sentiment Score: 0.87
Document ID: 2 , Sentiment Score: 0.11
Document ID: 3 , Sentiment Score: 0.44
Document ID: 4 , Sentiment Score: 1.00
```

## <a name="language-detection"></a>Detección de idiomas

1. Cree una función llamada `DetectLanguageExample()` que adopte el cliente que creó anteriormente.
2. Genere una lista de objetos `LanguageInput`, que contenga los documentos.

    ```csharp
    public static async Task DetectLanguageExample(TextAnalyticsClient client)
    {

        // The documents to be submitted for language detection. The ID can be any value.
        var inputDocuments = new LanguageBatchInput(
                new List<LanguageInput>
                    {
                        new LanguageInput(id: "1", text: "This is a document written in English."),
                        new LanguageInput(id: "2", text: "Este es un document escrito en Español."),
                        new LanguageInput(id: "3", text: "这是一个用中文写的文件")
                    });
        //...
    }
    ```

3. En la misma función, llame a `client.DetectLanguageAsync()` y obtenga el resultado. A continuación, itere por los resultados e imprima el identificador de cada documento y el primer lenguaje devuelto.

    ```csharp
    var langResults = await client.DetectLanguageAsync(false, inputDocuments);

    // Printing detected languages
    foreach (var document in langResults.Documents)
    {
        Console.WriteLine($"Document ID: {document.Id} , Language: {document.DetectedLanguages[0].Name}");
    }
    ```

### <a name="output"></a>Salida

```console
===== LANGUAGE EXTRACTION ======
Document ID: 1 , Language: English
Document ID: 2 , Language: Spanish
Document ID: 3 , Language: Chinese_Simplified
```

## <a name="entity-recognition"></a>Reconocimiento de entidades

1. Cree una función llamada `RecognizeEntitiesExample()` que adopte el cliente que creó anteriormente.
2. Genere una lista de objetos `MultiLanguageBatchInput`, que contenga los documentos.

    ```csharp
    public static async Task RecognizeEntitiesExample(TextAnalyticsClient client)
    {

        // The documents to be submitted for entity recognition. The ID can be any value.
        var inputDocuments = new MultiLanguageBatchInput(
            new List<MultiLanguageInput>
            {
                new MultiLanguageInput("en", "1", "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, to develop and sell BASIC interpreters for the Altair 8800."),
                new MultiLanguageInput("es", "2", "La sede principal de Microsoft se encuentra en la ciudad de Redmond, a 21 kilómetros de Seattle.")
            });
        //...
    }
    ```

3. En la misma función, llame a `client.EntitiesAsync()` y obtenga el resultado. A continuación, itere por los resultados e imprima el identificador de cada documento. Para cada identidad detectada, imprima su nombre de Wikipedia, el tipo y los subtipos (si existen) y las ubicaciones del texto original.

    ```csharp
    var entitiesResult = await client.EntitiesAsync(false, inputDocuments);

    // Printing recognized entities
    foreach (var document in entitiesResult.Documents)
    {
        Console.WriteLine($"Document ID: {document.Id} ");

        Console.WriteLine("\t Entities:");
        foreach (var entity in document.Entities)
        {
            Console.WriteLine($"\t\tName: {entity.Name},\tType: {entity.Type ?? "N/A"},\tSub-Type: {entity.SubType ?? "N/A"}");
            foreach (var match in entity.Matches)
            {
                Console.WriteLine($"\t\t\tOffset: {match.Offset},\tLength: {match.Length},\tScore: {match.EntityTypeScore:F3}");
            }
        }
    }
    ```

### <a name="output"></a>Salida

```console
Document ID: 1
         Entities:
                Name: Microsoft,        Type: Organization,     Sub-Type: N/A
                        Offset: 0,      Length: 9,      Score: 1.000
                Name: Bill Gates,       Type: Person,   Sub-Type: N/A
                        Offset: 25,     Length: 10,     Score: 1.000
                Name: Paul Allen,       Type: Person,   Sub-Type: N/A
                        Offset: 40,     Length: 10,     Score: 0.999
                Name: April 4,  Type: Other,    Sub-Type: N/A
                        Offset: 54,     Length: 7,      Score: 0.800
                Name: April 4, 1975,    Type: DateTime, Sub-Type: Date
                        Offset: 54,     Length: 13,     Score: 0.800
                Name: BASIC,    Type: Other,    Sub-Type: N/A
                        Offset: 89,     Length: 5,      Score: 0.800
                Name: Altair 8800,      Type: Other,    Sub-Type: N/A
                        Offset: 116,    Length: 11,     Score: 0.800
Document ID: 2
         Entities:
                Name: Microsoft,        Type: Organization,     Sub-Type: N/A
                        Offset: 21,     Length: 9,      Score: 1.000
                Name: Redmond (Washington),     Type: Location, Sub-Type: N/A
                        Offset: 60,     Length: 7,      Score: 0.991
                Name: 21 kilómetros,    Type: Quantity, Sub-Type: Dimension
                        Offset: 71,     Length: 13,     Score: 0.800
                Name: Seattle,  Type: Location, Sub-Type: N/A
                        Offset: 88,     Length: 7,      Score: 1.000
```

## <a name="key-phrase-extraction"></a>Extracción de la frase clave

1. Cree una función llamada `KeyPhraseExtractionExample()` que adopte el cliente que creó anteriormente.
2. Genere una lista de objetos `MultiLanguageBatchInput`, que contenga los documentos.

    ```csharp
    public static async Task KeyPhraseExtractionExample(TextAnalyticsClient client)
    {
        var inputDocuments = new MultiLanguageBatchInput(
                    new List<MultiLanguageInput>
                    {
                        new MultiLanguageInput("ja", "1", "猫は幸せ"),
                        new MultiLanguageInput("de", "2", "Fahrt nach Stuttgart und dann zum Hotel zu Fu."),
                        new MultiLanguageInput("en", "3", "My cat might need to see a veterinarian."),
                        new MultiLanguageInput("es", "4", "A mi me encanta el fútbol!")
                    });
        //...
    }
    ```

3. En la misma función, llame a `client.KeyPhrasesAsync()` y obtenga el resultado. A continuación, itere por los resultados e imprima el identificador de cada documento y todas las frases clave detectadas.

    ```csharp
    var kpResults = await client.KeyPhrasesAsync(false, inputDocuments);

    // Printing keyphrases
    foreach (var document in kpResults.Documents)
    {
        Console.WriteLine($"Document ID: {document.Id} ");

        Console.WriteLine("\t Key phrases:");

        foreach (string keyphrase in document.KeyPhrases)
        {
            Console.WriteLine($"\t\t{keyphrase}");
        }
    }
    ```

### <a name="output"></a>Salida

```console
Document ID: 1
         Key phrases:
                幸せ
Document ID: 2
         Key phrases:
                Stuttgart
                Hotel
                Fahrt
                Fu
Document ID: 3
         Key phrases:
                cat
                veterinarian
Document ID: 4
         Key phrases:
                fútbol
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Text Analytics con Power BI](../tutorials/tutorial-power-bi-key-phrases.md)


* [Información general de Text Analytics](../overview.md)
* [Preguntas más frecuentes](../text-analytics-resource-faq.md)

