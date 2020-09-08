---
title: Especificación del idioma de origen para la conversión de voz en texto
titleSuffix: Azure Cognitive Services
description: El SDK de Voz permite especificar el lenguaje fuente al convertir voz en texto. En este artículo se describe cómo usar los métodos FromConfig y SourceLanguageConfig para permitir que el servicio de voz conozca el lenguaje de origen y proporcione un destino de modelo personalizado.
services: cognitive-services
author: susanhu
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: qiohu
zone_pivot_groups: programming-languages-set-two
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: 3d761bbad4cb2cd2cdd1c34459f25c811bb41c7e
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88918696"
---
# <a name="specify-source-language-for-speech-to-text"></a>Especificación del lenguaje fuente para la conversión de voz en texto

En este artículo, obtendrá información sobre cómo especificar el lenguaje fuente de una entrada de audio que se pasa al SDK de Voz para el reconocimiento de voz. Además, se proporciona código de ejemplo para especificar un modelo de voz personalizado para el reconocimiento mejorado.

::: zone pivot="programming-language-csharp"

## <a name="how-to-specify-source-language-in-c"></a>Cómo especificar el lenguaje fuente en C#

En este ejemplo, el lenguaje fuente se proporciona de forma explícita como un parámetro mediante la construcción `SpeechRecognizer`.

```csharp
var recognizer = new SpeechRecognizer(speechConfig, "de-DE", audioConfig);
```

En este ejemplo, el lenguaje de origen se proporciona mediante `SourceLanguageConfig`. Luego, se pasa `sourceLanguageConfig` como un parámetro a la construcción `SpeechRecognizer`.

```csharp
var sourceLanguageConfig = SourceLanguageConfig.FromLanguage("de-DE");
var recognizer = new SpeechRecognizer(speechConfig, sourceLanguageConfig, audioConfig);
```

En este ejemplo, el lenguaje de origen y el punto de conexión personalizado se proporcionan mediante `SourceLanguageConfig`. Luego, se pasa `sourceLanguageConfig` como un parámetro a la construcción `SpeechRecognizer`.

```csharp
var sourceLanguageConfig = SourceLanguageConfig.FromLanguage("de-DE", "The Endpoint ID for your custom model.");
var recognizer = new SpeechRecognizer(speechConfig, sourceLanguageConfig, audioConfig);
```

>[!Note]
> `SpeechRecognitionLanguage` y `EndpointId` son métodos Set en desuso de la clase `SpeechConfig` de C#. No se recomienda el uso de estos métodos y no deben usarse al construir un `SpeechRecognizer`.

::: zone-end

::: zone pivot="programming-language-cpp"


## <a name="how-to-specify-source-language-in-c"></a>Cómo especificar el lenguaje fuente en C++

En este ejemplo, el lenguaje de origen se proporciona explícitamente como un parámetro mediante el método `FromConfig`.

```C++
auto recognizer = SpeechRecognizer::FromConfig(speechConfig, "de-DE", audioConfig);
```

En este ejemplo, el lenguaje de origen se proporciona mediante `SourceLanguageConfig`. A continuación, se pasa el método `sourceLanguageConfig` como un parámetro a `FromConfig` al crear el elemento `recognizer`.

```C++
auto sourceLanguageConfig = SourceLanguageConfig::FromLanguage("de-DE");
auto recognizer = SpeechRecognizer::FromConfig(speechConfig, sourceLanguageConfig, audioConfig);
```

En este ejemplo, el lenguaje de origen y el punto de conexión personalizado se proporcionan mediante `SourceLanguageConfig`. Se pasa el método `sourceLanguageConfig` como un parámetro a `FromConfig` al crear el elemento `recognizer`.

```C++
auto sourceLanguageConfig = SourceLanguageConfig::FromLanguage("de-DE", "The Endpoint ID for your custom model.");
auto recognizer = SpeechRecognizer::FromConfig(speechConfig, sourceLanguageConfig, audioConfig);
```

>[!Note]
> `SetSpeechRecognitionLanguage` y `SetEndpointId` son métodos en desuso de la clase `SpeechConfig` en C++ y Java. No se recomienda el uso de estos métodos y no deben usarse al construir un `SpeechRecognizer`.

::: zone-end

::: zone pivot="programming-language-java"

## <a name="how-to-specify-source-language-in-java"></a>Cómo especificar el lenguaje fuente en Java

En este ejemplo, el lenguaje de origen se proporciona explícitamente cuando se crea un nuevo elemento `SpeechRecognizer`.

```Java
SpeechRecognizer recognizer = new SpeechRecognizer(speechConfig, "de-DE", audioConfig);
```

En este ejemplo, el lenguaje de origen se proporciona mediante `SourceLanguageConfig`. A continuación, se pasa el método `sourceLanguageConfig` como parámetro al crear el elemento `SpeechRecognizer`.

```Java
SourceLanguageConfig sourceLanguageConfig = SourceLanguageConfig.fromLanguage("de-DE");
SpeechRecognizer recognizer = new SpeechRecognizer(speechConfig, sourceLanguageConfig, audioConfig);
```

En este ejemplo, el lenguaje de origen y el punto de conexión personalizado se proporcionan mediante `SourceLanguageConfig`. A continuación, se pasa el método `sourceLanguageConfig` como parámetro al crear el elemento `SpeechRecognizer`.

```Java
SourceLanguageConfig sourceLanguageConfig = SourceLanguageConfig.fromLanguage("de-DE", "The Endpoint ID for your custom model.");
SpeechRecognizer recognizer = new SpeechRecognizer(speechConfig, sourceLanguageConfig, audioConfig);
```

>[!Note]
> `setSpeechRecognitionLanguage` y `setEndpointId` son métodos en desuso de la clase `SpeechConfig` en C++ y Java. No se recomienda el uso de estos métodos y no deben usarse al construir un `SpeechRecognizer`.

::: zone-end

::: zone pivot="programming-language-python"

## <a name="how-to-specify-source-language-in-python"></a>Cómo especificar el lenguaje fuente en Python

En este ejemplo, el lenguaje fuente se proporciona de forma explícita como un parámetro mediante la construcción `SpeechRecognizer`.

```Python
speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config, language="de-DE", audio_config=audio_config)
```

En este ejemplo, el lenguaje de origen se proporciona mediante `SourceLanguageConfig`. Luego, se pasa `SourceLanguageConfig` como un parámetro a la construcción `SpeechRecognizer`.

```Python
source_language_config = speechsdk.languageconfig.SourceLanguageConfig("de-DE")
speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config, source_language_config=source_language_config, audio_config=audio_config)
```

En este ejemplo, el lenguaje de origen y el punto de conexión personalizado se proporcionan mediante `SourceLanguageConfig`. Luego, se pasa `SourceLanguageConfig` como un parámetro a la construcción `SpeechRecognizer`.

```Python
source_language_config = speechsdk.languageconfig.SourceLanguageConfig("de-DE", "The Endpoint ID for your custom model.")
speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config, source_language_config=source_language_config, audio_config=audio_config)
```

>[!Note]
> Las propiedades `speech_recognition_language` y `endpoint_id` están en desuso de la clase `SpeechConfig` en Python. No se recomienda el uso de estas propiedades y no deben usarse al construir un `SpeechRecognizer`.

::: zone-end

::: zone pivot="programming-language-more"

## <a name="how-to-specify-source-language-in-javascript"></a>Cómo especificar el lenguaje fuente en JavaScript

El primer paso consiste en crear un `SpeechConfig`:

```Javascript
var speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionkey", "YourRegion");
```

A continuación, especifique el lenguaje fuente del audio con `speechRecognitionLanguage`:

```Javascript
speechConfig.speechRecognitionLanguage = "de-DE";
```

Si utiliza un modelo personalizado para el reconocimiento, puede especificar el punto de conexión con `endpointId`:

```Javascript
speechConfig.endpointId = "The Endpoint ID for your custom model.";
```

## <a name="how-to-specify-source-language-in-objective-c"></a>Cómo especificar el lenguaje fuente en Objective-C

En este ejemplo, el lenguaje fuente se proporciona de forma explícita como un parámetro mediante la construcción `SPXSpeechRecognizer`.

```Objective-C
SPXSpeechRecognizer* speechRecognizer = \
    [[SPXSpeechRecognizer alloc] initWithSpeechConfiguration:speechConfig language:@"de-DE" audioConfiguration:audioConfig];
```

En este ejemplo, el lenguaje de origen se proporciona mediante `SPXSourceLanguageConfiguration`. Luego, se pasa `SPXSourceLanguageConfiguration` como un parámetro a la construcción `SPXSpeechRecognizer`.

```Objective-C
SPXSourceLanguageConfiguration* sourceLanguageConfig = [[SPXSourceLanguageConfiguration alloc]init:@"de-DE"];
SPXSpeechRecognizer* speechRecognizer = [[SPXSpeechRecognizer alloc] initWithSpeechConfiguration:speechConfig
                                                                     sourceLanguageConfiguration:sourceLanguageConfig
                                                                              audioConfiguration:audioConfig];
```

En este ejemplo, el lenguaje de origen y el punto de conexión personalizado se proporcionan mediante `SPXSourceLanguageConfiguration`. Luego, se pasa `SPXSourceLanguageConfiguration` como un parámetro a la construcción `SPXSpeechRecognizer`.

```Objective-C
SPXSourceLanguageConfiguration* sourceLanguageConfig = \
        [[SPXSourceLanguageConfiguration alloc]initWithLanguage:@"de-DE"
                                                     endpointId:@"The Endpoint ID for your custom model."];
SPXSpeechRecognizer* speechRecognizer = [[SPXSpeechRecognizer alloc] initWithSpeechConfiguration:speechConfig
                                                                     sourceLanguageConfiguration:sourceLanguageConfig
                                                                              audioConfiguration:audioConfig];
```

>[!Note]
> Las propiedades `speechRecognitionLanguage` y `endpointId` están en desuso de la clase `SPXSpeechConfiguration` en Objective-C. No se recomienda el uso de estas propiedades y no deben usarse al construir un `SPXSpeechRecognizer`.

::: zone-end

## <a name="see-also"></a>Consulte también

* Para obtener una lista de idiomas y configuraciones regionales admitidos para la conversión de voz en texto, consulte [compatibilidad de idioma](language-support.md).

## <a name="next-steps"></a>Pasos siguientes

* [Documentación de referencia del SDK de voz](speech-sdk.md)