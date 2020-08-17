---
title: 'Inicio rápido: Biblioteca cliente v3 de Text Analytics para Java | Microsoft Docs'
description: Introducción a la biblioteca cliente v3 de Text Analytics para Java.
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 07/27/2020
ms.custom: devx-track-java
ms.author: aahi
ms.reviewer: tasharm, assafi, sumeh
ms.openlocfilehash: e0edd688fa34033209798603ef15352b42cd9497
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87451144"
---
<a name="HOLTop"></a>

[Documentación de referencia](https://aka.ms/azsdk-java-textanalytics-ref-docs) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/textanalytics/azure-ai-textanalytics) | [Paquete](https://mvnrepository.com/artifact/com.azure/azure-ai-textanalytics/5.0.0) | [Ejemplos](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics/src/samples/java/com/azure/ai/textanalytics)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* [Kit de desarrollo de Java](https://www.oracle.com/technetwork/java/javase/downloads/index.html) (JDK), versión 8 o posterior
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="cree un recurso de Text Analytics"  target="_blank">cree un recurso de Text Analytics <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión.  Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a API Text Analytics. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="add-the-client-library"></a>Incorporación de la biblioteca cliente

Cree un proyecto de Maven en el entorno de desarrollo o IDE que prefiera. Luego, agregue la siguiente dependencia al archivo *pom.xml* del proyecto. La sintaxis de implementación [de otras herramientas de compilación](https://mvnrepository.com/artifact/com.azure/azure-ai-textanalytics/5.0.0) se puede encontrar en línea.

```xml
<dependencies>
     <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-ai-textanalytics</artifactId>
        <version>5.0.0</version>
    </dependency>
</dependencies>
```

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo [en GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/TextAnalytics/TextAnalyticsSamples.java), que contiene los ejemplos de código de este inicio rápido. 

Cree un archivo de Java llamado `TextAnalyticsSamples.java`. Abra el archivo y agregue las siguientes instrucciones `import`:

```java
import com.azure.core.credential.AzureKeyCredential;
import com.azure.ai.textanalytics.models.*;
import com.azure.ai.textanalytics.TextAnalyticsClientBuilder;
import com.azure.ai.textanalytics.TextAnalyticsClient;
```

En el archivo de Java, agregue una nueva clase y agregue el punto de conexión y la clave del recurso de Azure, como se muestra a continuación.

[!INCLUDE [text-analytics-find-resource-information](../find-azure-resource-info.md)]

```java
public class TextAnalyticsSamples {
    private static String KEY = "<replace-with-your-text-analytics-key-here>";
    private static String ENDPOINT = "<replace-with-your-text-analytics-endpoint-here>";
}
```

Agregue el siguiente método principal a la clase. Más adelante definirá los métodos a los que llama aquí.

```java
public static void main(String[] args) {
    //You will create these methods later in the quickstart.
    TextAnalyticsClient client = authenticateClient(KEY, ENDPOINT);

    sentimentAnalysisExample(client);
    detectLanguageExample(client);
    recognizeEntitiesExample(client);
    recognizeLinkedEntitiesExample(client);
    extractKeyPhrasesExample(client);
}
```

## <a name="object-model"></a>Modelo de objetos

El cliente de Text Analytics es un objeto `TextAnalyticsClient` que se autentica en Azure mediante su clave y que proporciona funciones para aceptar texto como cadenas individuales o como un lote. Puede enviar texto a la API de forma sincrónica o asincrónica. El objeto de respuesta contendrá la información del análisis de todos los documentos que envíe. 

## <a name="code-examples"></a>Ejemplos de código

* [Autenticar el cliente](#authenticate-the-client)
* [Análisis de sentimiento](#sentiment-analysis) 
* [Detección de idioma](#language-detection)
* [Reconocimiento de entidades con nombre](#named-entity-recognition-ner) 
* [Vinculación de entidad](#entity-linking)
* [Extracción de frases clave](#key-phrase-extraction)

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree un método para crear una instancia del objeto `TextAnalyticsClient` con la clave y el punto de conexión del recurso Text Analytics.

```java
static TextAnalyticsClient authenticateClient(String key, String endpoint) {
    return new TextAnalyticsClientBuilder()
        .credential(new AzureKeyCredential(key))
        .endpoint(endpoint)
        .buildClient();
}
```

En el método de `main()` del programa, llame al método de autenticación para crear una instancia del cliente.

## <a name="sentiment-analysis"></a>análisis de opiniones

Cree una función denominada `sentimentAnalysisExample()` que tome el cliente que creó anteriormente y llame a su función `analyzeSentiment()`. El objeto `AnalyzeSentimentResult` devuelto contendrá `documentSentiment` y `sentenceSentiments` si la operación se realiza correctamente, o `errorMessage` si no. 

```java
static void sentimentAnalysisExample(TextAnalyticsClient client)
{
    // The text that need be analyzed.
    String text = "I had the best day of my life. I wish you were there with me.";

    DocumentSentiment documentSentiment = client.analyzeSentiment(text);
    System.out.printf(
        "Recognized document sentiment: %s, positive score: %s, neutral score: %s, negative score: %s.%n",
        documentSentiment.getSentiment(),
        documentSentiment.getConfidenceScores().getPositive(),
        documentSentiment.getConfidenceScores().getNeutral(),
        documentSentiment.getConfidenceScores().getNegative());

    for (SentenceSentiment sentenceSentiment : documentSentiment.getSentences()) {
        System.out.printf(
            "Recognized sentence sentiment: %s, positive score: %s, neutral score: %s, negative score: %s.%n",
            sentenceSentiment.getSentiment(),
            sentenceSentiment.getConfidenceScores().getPositive(),
            sentenceSentiment.getConfidenceScores().getNeutral(),
            sentenceSentiment.getConfidenceScores().getNegative());
        }
    }
```

### <a name="output"></a>Output

```console
Recognized document sentiment: positive, positive score: 1.0, neutral score: 0.0, negative score: 0.0.
Recognized sentence sentiment: positive, positive score: 1.0, neutral score: 0.0, negative score: 0.0.
Recognized sentence sentiment: neutral, positive score: 0.21, neutral score: 0.77, negative score: 0.02.
```

## <a name="language-detection"></a>Detección de idiomas

Cree una función denominada `detectLanguageExample()` que tome el cliente que creó anteriormente y llame a su función `detectLanguage()`. El objeto `DetectLanguageResult` devuelto contendrá un idioma principal detectado, una lista de otros idiomas detectados si la operación se realiza correctamente o `errorMessage` si no.

> [!Tip]
> En algunos casos, puede ser difícil eliminar la ambigüedad de los idiomas en función de la entrada. Puede usar el parámetro `countryHint` para especificar un código de país de dos letras. De forma predeterminada, la API usa "US" como valor predeterminado de countryHint; para eliminar este comportamiento, puede restablecer este parámetro y configurarlo como una cadena `countryHint = ""` vacía. Para establecer un valor predeterminado diferente, establezca la propiedad `TextAnalyticsClientOptions.DefaultCountryHint` y pásela durante la inicialización del cliente.

```java
static void detectLanguageExample(TextAnalyticsClient client)
{
    // The text that need be analyzed.
    String text = "Ce document est rédigé en Français.";

    DetectedLanguage detectedLanguage = client.detectLanguage(text);
    System.out.printf("Detected primary language: %s, ISO 6391 name: %s, score: %.2f.%n",
        detectedLanguage.getName(),
        detectedLanguage.getIso6391Name(),
        detectedLanguage.getConfidenceScore());
}
```

### <a name="output"></a>Output

```console
Detected primary language: French, ISO 6391 name: fr, score: 1.00.
```
## <a name="named-entity-recognition-ner"></a>Reconocimiento de entidades con nombre (NER)

> [!NOTE]
> En la versión `3.0`:
> * NER incluye métodos independientes para detectar información personal. 
> * La vinculación de entidad es una solicitud independiente distinta a la de NER.

Cree una función denominada `recognizeEntitiesExample()` que tome el cliente que creó anteriormente y llame a su función `recognizeEntities()`. El objeto devuelto `RecognizeEntitiesResult` contendrá una lista de `NamedEntity` si la operación se realiza correctamente o `errorMessage` si no.

```java
static void recognizeEntitiesExample(TextAnalyticsClient client)
{
    // The text that need be analyzed.
    String text = "I had a wonderful trip to Seattle last week.";

    for (CategorizedEntity entity : client.recognizeEntities(text)) {
        System.out.printf(
            "Recognized entity: %s, entity category: %s, entity sub-category: %s, score: %s.%n",
            entity.getText(),
            entity.getCategory(),
            entity.getSubcategory(),
            entity.getConfidenceScore());
    }
}
```

### <a name="output"></a>Output

```console
Recognized entity: trip, entity category: Event, entity sub-category: null, score: 0.61.
Recognized entity: Seattle, entity category: Location, entity sub-category: GPE, score: 0.82.
Recognized entity: last week, entity category: DateTime, entity sub-category: DateRange, score: 0.8.
```

## <a name="entity-linking"></a>Vinculación de entidad

Cree una función denominada `recognizeLinkedEntitiesExample()` que tome el cliente que creó anteriormente y llame a su función `recognizeLinkedEntities()`. El objeto devuelto `RecognizeLinkedEntitiesResult` contendrá una lista de `LinkedEntity` si la operación se realiza correctamente o `errorMessage` si no. Puesto que las entidades vinculadas se identifican de forma única, las apariciones de la misma entidad se agrupan bajo un objeto `LinkedEntity` como una lista de objetos `LinkedEntityMatch`.

```java
static void recognizeLinkedEntitiesExample(TextAnalyticsClient client)
{
    // The text that need be analyzed.
    String text = "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, " +
        "to develop and sell BASIC interpreters for the Altair 8800. " +
        "During his career at Microsoft, Gates held the positions of chairman, " +
        "chief executive officer, president and chief software architect, " +
        "while also being the largest individual shareholder until May 2014.";

    System.out.printf("Linked Entities:%n");
    for (LinkedEntity linkedEntity : client.recognizeLinkedEntities(text)) {
        System.out.printf("Name: %s, ID: %s, URL: %s, Data Source: %s.%n",
            linkedEntity.getName(),
            linkedEntity.getDataSourceEntityId(),
            linkedEntity.getUrl(),
            linkedEntity.getDataSource());
        System.out.printf("Matches:%n");
        for (LinkedEntityMatch linkedEntityMatch : linkedEntity.getMatches()) {
            System.out.printf("Text: %s, Score: %.2f%n",
            linkedEntityMatch.getText(),
            linkedEntityMatch.getConfidenceScore());
        }
    }
}
```

### <a name="output"></a>Output

```console
Linked Entities:
Name: Altair 8800, ID: Altair 8800, URL: https://en.wikipedia.org/wiki/Altair_8800, Data Source: Wikipedia.
Matches:
Text: Altair 8800, Score: 0.88
Name: Bill Gates, ID: Bill Gates, URL: https://en.wikipedia.org/wiki/Bill_Gates, Data Source: Wikipedia.
Matches:
Text: Bill Gates, Score: 0.63
Text: Gates, Score: 0.63
Name: Paul Allen, ID: Paul Allen, URL: https://en.wikipedia.org/wiki/Paul_Allen, Data Source: Wikipedia.
Matches:
Text: Paul Allen, Score: 0.60
Name: Microsoft, ID: Microsoft, URL: https://en.wikipedia.org/wiki/Microsoft, Data Source: Wikipedia.
Matches:
Text: Microsoft, Score: 0.55
Text: Microsoft, Score: 0.55
Name: April 4, ID: April 4, URL: https://en.wikipedia.org/wiki/April_4, Data Source: Wikipedia.
Matches:
Text: April 4, Score: 0.32
Name: BASIC, ID: BASIC, URL: https://en.wikipedia.org/wiki/BASIC, Data Source: Wikipedia.
Matches:
Text: BASIC, Score: 0.33
```
## <a name="key-phrase-extraction"></a>Extracción de la frase clave

Cree una función denominada `extractKeyPhrasesExample()` que tome el cliente que creó anteriormente y llame a su función `extractKeyPhrases()`. El objeto `ExtractKeyPhraseResult` devuelto contendrá una lista de frases clave si la operación se realiza correctamente o `errorMessage` si no.

```java
static void extractKeyPhrasesExample(TextAnalyticsClient client)
{
    // The text that need be analyzed.
    String text = "My cat might need to see a veterinarian.";

    System.out.printf("Recognized phrases: %n");
    for (String keyPhrase : client.extractKeyPhrases(text)) {
        System.out.printf("%s%n", keyPhrase);
    }
}
```

### <a name="output"></a>Output

```console
Recognized phrases: 
cat
veterinarian
```
