---
title: 'Inicio rápido: Biblioteca cliente de Text Analytics para Go | Microsoft Docs'
titleSuffix: Azure Cognitive Services
description: En esta guía de inicio rápido, detectará el idioma mediante la biblioteca cliente de Text Analytics para Go desde Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: quickstart
ms.date: 07/27/2020
ms.author: aahi
ms.openlocfilehash: 6849046211ae4216f181dd8ef0ca391c2876363c
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87291782"
---
# <a name="quickstart-use-the-text-analytics-client-library-for-go"></a>Inicio rápido: Uso de la biblioteca cliente de Text Analytics para Go

- [Documentación de referencia](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/textanalytics)
- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-go/blob/090dc0ee4d8d2d60e2a9525774d967a4111a2b0c/services/cognitiveservices/v2.1/textanalytics/client.go)
- [Paquete (GitHub)](https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.1/textanalytics)
- [Muestras](https://github.com/Azure-Samples/cognitive-services-quickstart-code)

> [!NOTE]
> Este inicio rápido solo se aplica a Text Analytics 2.1. Actualmente, no hay disponible ninguna biblioteca de cliente v3 para Go.

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services).
* La versión más reciente de [Go](https://golang.org/dl/).
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="cree un recurso de Text Analytics"  target="_blank">cree un recurso de Text Analytics <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. 
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a API Text Analytics. Lo hará más adelante en el inicio rápido.
    * Puede usar el plan de tarifa gratuito para probar el servicio y actualizarlo más adelante a un nivel de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-go-project"></a>Creación de un nuevo proyecto de Go

En una ventana de consola (cmd, PowerShell, Terminal, Bash), cree una nueva área de trabajo para el proyecto de Go y vaya hasta ella. El área de trabajo contendrá tres carpetas: 

* **src**: este directorio contiene código fuente y paquetes. Todos los paquetes instalados con el comando `go get` residirán aquí.
* **pkg**: este directorio contiene objetos de paquete de Go compilados. Todos estos archivos tienen la extensión `.a`.
* **bin**: este directorio contiene los archivos ejecutables binarios que se crean al ejecutar `go install`.

> [!TIP]
> Obtenga más información sobre la estructura de [un área de trabajo de Go](https://golang.org/doc/code.html#Workspaces). En esta guía se incluye información para establecer `$GOPATH` y `$GOROOT`.

Cree un área de trabajo llamada `my-app` y los subdirectorios necesarios para `src`, `pkg` y `bin`:

```console
$ mkdir -p my-app/{src, bin, pkg}  
$ cd my-app
```

### <a name="install-the-text-analytics-client-library-for-go"></a>Instalación de la biblioteca cliente de Text Analytics para Go

Instale la biblioteca cliente para Go: 

```console
$ go get -u <https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.1/textanalytics>
```

O, si usa dep, dentro de su repositorio ejecute:

```console
$ dep ensure -add <https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.1/textanalytics>
```

### <a name="create-your-go-application"></a>Creación de la aplicación de Go

A continuación, cree un archivo llamado `src/quickstart.go`:

```bash
$ cd src
$ touch quickstart.go
```

Abra `quickstart.go` en el IDE o editor de texto que prefiera. A continuación, agregue el nombre del paquete e importe las siguientes bibliotecas:

[!code-go[Import statements](~/azure-sdk-for-go-samples/cognitiveservices/textanalytics.go?name=imports)]

## <a name="object-model"></a>Modelo de objetos 

El cliente de Text Analytics es un objeto [BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/textanalytics#New) que se autentica en Azure mediante la clave. El cliente proporciona varios métodos para analizar texto, como una sola cadena o un lote. 

El texto se envía a la API como una lista de `documents`, que son objetos `dictionary` que contienen una combinación de atributos `id`, `text` y `language`, según el método utilizado. El atributo `text` almacena el texto que se va a analizar en el origen `language` y `id` puede ser cualquier valor. 

El objeto de respuesta es una lista que contiene la información de análisis para cada documento. 

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo hacer lo siguiente con la biblioteca cliente de Text Analytics para Go:

* [Autenticar el cliente](#authenticate-the-client)
* [Análisis de sentimiento](#sentiment-analysis)
* [Detección de idioma](#language-detection)
* [Reconocimiento de entidades](#entity-recognition)
* [Extracción de frases clave](#key-phrase-extraction)

## <a name="authenticate-the-client"></a>Autenticar el cliente


En una nueva función, cree variables para la clave de suscripción y el punto de conexión de Azure del recurso.

[!INCLUDE [text-analytics-find-resource-information](../includes/find-azure-resource-info.md)]

Cree un nuevo objeto [BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/textanalytics#New). Pase la clave a la función [autorest.NewCognitiveServicesAuthorizer()](https://godoc.org/github.com/Azure/go-autorest/autorest#NewCognitiveServicesAuthorizer), que a continuación se pasará a la propiedad `authorizer` del cliente.

```go
func GetTextAnalyticsClient() textanalytics.BaseClient {
    var key string = "<paste-your-text-analytics-key-here>"
    var endpoint string = "<paste-your-text-analytics-endpoint-here>"

    textAnalyticsClient := textanalytics.New(endpoint)
    textAnalyticsClient.Authorizer = autorest.NewCognitiveServicesAuthorizer(key)

    return textAnalyticsClient
}
```

## <a name="sentiment-analysis"></a>análisis de opiniones

Cree una nueva función llamada `SentimentAnalysis()` y cree un cliente mediante el método `GetTextAnalyticsClient()` creado anteriormente. Cree una lista de objetos [MultiLanguageInput](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/textanalytics#MultiLanguageBatchInput) que contenga los documentos que desea analizar. Cada objeto contendrá un `id`, `Language` y un atributo `text`. El atributo `text` almacena el texto que se va a analizar, `language` es el idioma del documento y `id` puede ser cualquier valor. 

Llame a la función [Sentiment()](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/textanalytics#BaseClient.Sentiment) del cliente y obtenga el resultado. A continuación, itere por los resultados e imprima el identificador de cada documento y la puntuación de opiniones. Una puntuación cercana a 0 indica una opinión negativa, mientras que una puntuación cercana a 1 indica una opinión positiva.

[!code-go[Sentiment analysis sample](~/azure-sdk-for-go-samples/cognitiveservices/textanalytics.go?name=sentimentAnalysis)]

Llame a `SentimentAnalysis()` en el proyecto.

### <a name="output"></a>Output

```console
Document ID: 1 , Sentiment Score: 0.87
Document ID: 2 , Sentiment Score: 0.11
Document ID: 3 , Sentiment Score: 0.44
Document ID: 4 , Sentiment Score: 1.00
```

## <a name="language-detection"></a>Detección de idiomas

Cree una nueva función llamada `LanguageDetection()` y cree un cliente mediante el método `GetTextAnalyticsClient()` creado anteriormente. Cree una lista de objetos [LanguageInput](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/textanalytics#LanguageInput) que contenga los documentos que desea analizar. Cada objeto contendrá un `id` y un atributo `text`. El atributo `text` almacena el texto que se va a analizar y el `id` puede ser cualquier valor. 

Llame a la función [DetectLanguage()](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/textanalytics#BaseClient.DetectLanguage) del cliente y obtenga el resultado. A continuación, itere por los resultados e imprima el identificador de cada documento y el idioma detectado.

[!code-go[Language detection sample](~/azure-sdk-for-go-samples/cognitiveservices/textanalytics.go?name=languageDetection)]

Llame a `LanguageDetection()` en el proyecto.

### <a name="output"></a>Output

```console
Document ID: 0 , Language: English 
Document ID: 1 , Language: Spanish
Document ID: 2 , Language: Chinese_Simplified
```

## <a name="entity-recognition"></a>Reconocimiento de entidades

Cree una nueva función llamada `ExtractEntities()` y cree un cliente mediante el método `GetTextAnalyticsClient()` creado anteriormente. Cree una lista de objetos [MultiLanguageInput](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/textanalytics#MultiLanguageBatchInput) que contenga los documentos que desea analizar. Cada objeto contendrá un `id`, `language` y un atributo `text`. El atributo `text` almacena el texto que se va a analizar, `language` es el idioma del documento y `id` puede ser cualquier valor. 

Llame a la función [Entities()](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/textanalytics#BaseClient.Entities) del cliente y obtenga el resultado. A continuación, recorra en iteración los resultados e imprima el identificador de cada documento y la puntuación de las entidades extraídas.

[!code-go[entity recognition sample](~/azure-sdk-for-go-samples/cognitiveservices/textanalytics.go?name=entityRecognition)]

Llame a `ExtractEntities()` en el proyecto.

### <a name="output"></a>Output

```console
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

## <a name="key-phrase-extraction"></a>Extracción de la frase clave

Cree una nueva función llamada `ExtractKeyPhrases()` y cree un cliente mediante el método `GetTextAnalyticsClient()` creado anteriormente. Cree una lista de objetos [MultiLanguageInput](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/textanalytics#MultiLanguageBatchInput) que contenga los documentos que desea analizar. Cada objeto contendrá un `id`, `language` y un atributo `text`. El atributo `text` almacena el texto que se va a analizar, `language` es el idioma del documento y `id` puede ser cualquier valor.

Llame a la función [KeyPhrases()](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/textanalytics#BaseClient.KeyPhrases) del cliente y obtenga el resultado. A continuación, recorra en iteración los resultados e imprima el identificador de cada documento y las frases clave extraídas.

[!code-go[key phrase extraction sample](~/azure-sdk-for-go-samples/cognitiveservices/textanalytics.go?name=keyPhrases)]

Llame a `ExtractKeyPhrases()` en el proyecto.

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
                veterinarian
Document ID: 4
         Key phrases:
                fútbol
```
