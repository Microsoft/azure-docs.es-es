---
title: 'Inicio rápido: Biblioteca cliente de Text Analytics para Ruby | Microsoft Docs'
titleSuffix: Azure Cognitive Services
description: En esta guía de inicio rápido, detectará el idioma mediante la biblioteca cliente de Text Analytics para Ruby desde Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: quickstart
ms.date: 07/27/2020
ms.author: aahi
ms.openlocfilehash: 8afceb19af0d177415d0b68b5d38f19d18835af5
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87291758"
---
# <a name="quickstart-use-the-text-analytics-client-library-for-ruby"></a>Inicio rápido: Uso de la biblioteca cliente de Text Analytics para Ruby

Comience a trabajar con la biblioteca cliente de Text Analytics. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

La biblioteca cliente de Text Analytics se puede usar para realizar las siguientes operaciones:

* análisis de opiniones
* Detección de idiomas
* Reconocimiento de entidades
* Extracción de la frase clave

> [!NOTE]
> Este inicio rápido solo se aplica a Text Analytics 2.1. Actualmente, no hay disponible ninguna biblioteca de cliente v3 para Ruby.

[Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-ruby/tree/master/data/azure_cognitiveservices_textanalytics) | [Paquete (RubyGems)](https://rubygems.org/gems/azure_cognitiveservices_textanalytics) | [Ejemplos](https://github.com/Azure-Samples/cognitive-services-quickstart-code)

<a name="HOLTop"></a>

## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services).
* La versión actual de [Ruby](https://www.ruby-lang.org/)
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="cree un recurso de Text Analytics"  target="_blank">cree un recurso de Text Analytics <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. 
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a API Text Analytics. Lo hará más adelante en el inicio rápido.
    * Puede usar el plan de tarifa gratuito para probar el servicio y actualizarlo más adelante a un nivel de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-ruby-application"></a>Creación de una aplicación de Ruby

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. A continuación, cree un archivo denominado `GemFile` y un archivo de Ruby para el código.

```console
mkdir myapp && cd myapp
```

En `GemFile`, agregue las líneas siguientes para agregar la biblioteca cliente como dependencia.

```ruby
source 'https://rubygems.org'
gem 'azure_cognitiveservices_textanalytics', '~>0.17.3'
```

En el archivo de Ruby, importe los siguientes paquetes.

[!code-ruby[Import statements](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=includeStatement)]

Cree variables para el punto de conexión y la clave de Azure del recurso. 

[!INCLUDE [text-analytics-find-resource-information](../includes/find-azure-resource-info.md)]

```ruby
const subscription_key = '<paste-your-text-analytics-key-here>'
const endpoint = `<paste-your-text-analytics-endpoint-here>`
```

## <a name="object-model"></a>Modelo de objetos 

El cliente de Text Analytics se autentica en Azure mediante la clave. El cliente proporciona varios métodos para analizar texto, como una sola cadena o un lote. 

El texto se envía a la API como una lista de `documents`, que son objetos `dictionary` que contienen una combinación de atributos `id`, `text` y `language`, según el método utilizado. El atributo `text` almacena el texto que se va a analizar en el origen `language` y `id` puede ser cualquier valor. 

El objeto de respuesta es una lista que contiene la información de análisis para cada documento. 

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo hacer lo siguiente con la biblioteca cliente de Text Analytics para Ruby:

* [Autenticar el cliente](#authenticate-the-client)
* [Análisis de sentimiento](#sentiment-analysis)
* [Detección de idioma](#language-detection)
* [Reconocimiento de entidades](#entity-recognition)
* [Extracción de frases clave](#key-phrase-extraction)

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una clase denominada `TextAnalyticsClient`. 

```ruby
class TextAnalyticsClient
  @textAnalyticsClient
  #...
end
```

En esta clase, cree una función llamada `initialize` para autenticar el cliente mediante su clave y punto de conexión. 

[!code-ruby[initialize function for authentication](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=initialize)]

Fuera de la clase, utilice la función `new()` del cliente para crear una instancia de ella.

[!code-ruby[client creation](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=clientCreation)] 

<a name="SentimentAnalysis"></a>

## <a name="sentiment-analysis"></a>análisis de opiniones

En el objeto de cliente, cree una función llamada `AnalyzeSentiment()` que tome una lista de los documentos de entrada que se crearán más adelante. Llame a la función `sentiment()` del cliente y obtenga el resultado. A continuación, itere por los resultados e imprima el identificador de cada documento y la puntuación de opiniones. Una puntuación cercana a 0 indica una opinión negativa, mientras que una puntuación cercana a 1 indica una opinión positiva.

[!code-ruby[client method for sentiment analysis](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=analyzeSentiment)] 

Fuera de la función del cliente, cree una función llamada `SentimentAnalysisExample()` que tome el objeto `TextAnalyticsClient` que creó anteriormente. Cree una lista de objetos `MultiLanguageInput`, que contenga los documentos que desea analizar. Cada objeto contendrá un `id`, `Language` y un atributo `text`. El atributo `text` almacena el texto que se va a analizar, `language` es el idioma del documento y `id` puede ser cualquier valor. A continuación, llame a la función `AnalyzeSentiment()` del cliente.

[!code-ruby[sentiment analysis document creation and call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=sentimentCall)] 

Llame a la función `SentimentAnalysisExample()`.

```ruby
SentimentAnalysisExample(textAnalyticsClient)
```

### <a name="output"></a>Output

```console
===== SENTIMENT ANALYSIS =====
Document ID: 1 , Sentiment Score: 0.87
Document ID: 2 , Sentiment Score: 0.11
Document ID: 3 , Sentiment Score: 0.44
Document ID: 4 , Sentiment Score: 1.00
```

<a name="LanguageDetection"></a>

## <a name="language-detection"></a>Detección de idiomas

En el objeto de cliente, cree una función llamada `DetectLanguage()` que tome una lista de los documentos de entrada que se crearán más adelante. Llame a la función `detect_language()` del cliente y obtenga el resultado. A continuación, itere por los resultados e imprima el identificador de cada documento y el idioma detectado.

[!code-ruby[client method for language detection](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=detectLanguage)] 

Fuera de la función del cliente, cree una función llamada `DetectLanguageExample()` que tome el objeto `TextAnalyticsClient` que creó anteriormente. Cree una lista de objetos `LanguageInput`, que contenga los documentos que desea analizar. Cada objeto contendrá un atributo `id` y `text`. El atributo `text` almacena el texto que se va a analizar y el `id` puede ser cualquier valor. A continuación, llame a la función `DetectLanguage()` del cliente.

[!code-ruby[language detection document creation and call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=detectLanguageCall)] 

Llame a la función `DetectLanguageExample()`.

```ruby
DetectLanguageExample(textAnalyticsClient)
```

### <a name="output"></a>Output

```console
===== LANGUAGE EXTRACTION ======
Document ID: 1 , Language: English
Document ID: 2 , Language: Spanish
Document ID: 3 , Language: Chinese_Simplified
```

<a name="EntityRecognition"></a>

## <a name="entity-recognition"></a>Reconocimiento de entidades

En el objeto de cliente, cree una función llamada `RecognizeEntities()` que tome una lista de los documentos de entrada que se crearán más adelante. Llame a la función `entities()` del cliente y obtenga el resultado. A continuación, itere los resultados e imprima el identificador de cada documento y las entidades reconocidas.

[!code-ruby[client method for entity recognition](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=recognizeEntities)]

Fuera de la función del cliente, cree una función llamada `RecognizeEntitiesExample()` que tome el objeto `TextAnalyticsClient` que creó anteriormente. Cree una lista de objetos `MultiLanguageInput`, que contenga los documentos que desea analizar. Cada objeto contendrá un atributo `id`, `language` y `text`. El atributo `text` almacena el texto que se va a analizar, `language` es el idioma del texto, y `id` puede ser cualquier valor. A continuación, llame a la función `RecognizeEntities()` del cliente.

[!code-ruby[entity recognition documents and method call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=recognizeEntitiesCall)] 

Llame a la función `RecognizeEntitiesExample()`.

```ruby
RecognizeEntitiesExample(textAnalyticsClient)
```

### <a name="output"></a>Output

```console
===== ENTITY RECOGNITION =====
Document ID: 1
        Name: Microsoft,        Type: Organization,     Sub-Type: N/A
        Offset: 0, Length: 9,   Score: 1.0

        Name: Bill Gates,       Type: Person,   Sub-Type: N/A
        Offset: 25, Length: 10, Score: 0.999847412109375

        Name: Paul Allen,       Type: Person,   Sub-Type: N/A
        Offset: 40, Length: 10, Score: 0.9988409876823425

        Name: April 4,  Type: Other,    Sub-Type: N/A
        Offset: 54, Length: 7,  Score: 0.8

        Name: April 4, 1975,    Type: DateTime, Sub-Type: Date
        Offset: 54, Length: 13, Score: 0.8

        Name: BASIC,    Type: Other,    Sub-Type: N/A
        Offset: 89, Length: 5,  Score: 0.8

        Name: Altair 8800,      Type: Other,    Sub-Type: N/A
        Offset: 116, Length: 11,        Score: 0.8

Document ID: 2
        Name: Microsoft,        Type: Organization,     Sub-Type: N/A
        Offset: 21, Length: 9,  Score: 0.999755859375

        Name: Redmond (Washington),     Type: Location, Sub-Type: N/A
        Offset: 60, Length: 7,  Score: 0.9911284446716309

        Name: 21 kilómetros,    Type: Quantity, Sub-Type: Dimension
        Offset: 71, Length: 13, Score: 0.8

        Name: Seattle,  Type: Location, Sub-Type: N/A
        Offset: 88, Length: 7,  Score: 0.9998779296875
```

<a name="KeyPhraseExtraction"></a>

## <a name="key-phrase-extraction"></a>Extracción de la frase clave

En el objeto de cliente, cree una función llamada `ExtractKeyPhrases()` que tome una lista de los documentos de entrada que se crearán más adelante. Llame a la función `key_phrases()` del cliente y obtenga el resultado. A continuación, itere los resultados e imprima el identificador de cada documento y las frases clave extraídas.

[!code-ruby[key phrase extraction client method](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=extractKeyPhrases)] 

Fuera de la función del cliente, cree una función llamada `KeyPhraseExtractionExample()` que tome el objeto `TextAnalyticsClient` que creó anteriormente. Cree una lista de objetos `MultiLanguageInput`, que contenga los documentos que desea analizar. Cada objeto contendrá un atributo `id`, `language` y `text`. El atributo `text` almacena el texto que se va a analizar, `language` es el idioma del texto, y `id` puede ser cualquier valor. A continuación, llame a la función `ExtractKeyPhrases()` del cliente.

[!code-ruby[key phrase document creation and call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=keyPhrasesCall)]


Llame a la función `KeyPhraseExtractionExample()`.

```ruby
KeyPhraseExtractionExample(textAnalyticsClient)
```

### <a name="output"></a>Output

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
                rock
Document ID: 4
         Key phrases:
                fútbol
```
