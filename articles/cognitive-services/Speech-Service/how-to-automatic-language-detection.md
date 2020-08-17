---
title: Uso de detección automática de idioma para la conversión de voz en texto
titleSuffix: Azure Cognitive Services
description: El SDK de Voz admite la detección automática de idioma para la conversión de voz en texto. Al usar esta característica, el audio proporcionado se compara con una lista de idiomas proporcionada y se determina la coincidencia más probable. Después, el valor devuelto se puede usar para seleccionar el modelo de lenguaje usado para la conversión de voz en texto.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: trbye
zone_pivot_groups: programming-languages-speech-services-nomore-variant
ms.custom: devx-track-javascript
ms.openlocfilehash: 3a648b11e795a7d5c8680ac8c3df95e7d0c6bb7d
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406782"
---
# <a name="automatic-language-detection-for-speech-to-text"></a>Detección automática de idioma para la conversión de voz en texto

La detección automática de idioma se usa para determinar la coincidencia más probable para el audio que se pasa al SDK de Voz cuando se compara con una lista de idiomas proporcionados. El valor devuelto por la detección automática de idioma se usa para seleccionar el modelo de idioma para la conversión de voz en texto, lo que le proporciona una transcripción más precisa. Para ver los idiomas que están disponibles, consulte la [compatibilidad de idioma](language-support.md).

En este artículo, aprenderá a usar `AutoDetectSourceLanguageConfig` para construir un objeto `SpeechRecognizer` y recuperar el idioma detectado.

> [!IMPORTANT]
> Esta característica solo está disponible para el SDK de Voz de C#, C++, Java, Python y Objective-C.

## <a name="automatic-language-detection-with-the-speech-sdk"></a>Detección automática de idioma con el SDK de Voz

La detección automática de idioma actualmente tiene un límite de servicio de cuatro idiomas por detección. Tenga en cuenta esta limitación al construir el objeto `AudoDetectSourceLanguageConfig`. En los ejemplos siguientes, creará un objeto `AutoDetectSourceLanguageConfig` y lo usará para construir un `SpeechRecognizer`.

> [!TIP]
> También puede especificar un modelo personalizado para utilizarlo al realizar la conversión de voz en texto. Para obtener más información, consulte [Uso de un modelo personalizado para la detección automática de idioma](#use-a-custom-model-for-automatic-language-detection).

En los fragmentos de código siguientes se muestra cómo usar la detección automática de idioma en las aplicaciones:

::: zone pivot="programming-language-csharp"

```csharp
var autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.FromLanguages(
        new string[] { "en-US", "de-DE" });

using (var recognizer = new SpeechRecognizer(
    speechConfig,
    autoDetectSourceLanguageConfig,
    audioConfig))
{
    var speechRecognitionResult = await recognizer.RecognizeOnceAsync();
    var autoDetectSourceLanguageResult =
        AutoDetectSourceLanguageResult.FromResult(speechRecognitionResult);
    var detectedLanguage = autoDetectSourceLanguageResult.Language;
}
```

::: zone-end

::: zone pivot="programming-language-cpp"

```cpp
auto autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig::FromLanguages({ "en-US", "de-DE" });

auto recognizer = SpeechRecognizer::FromConfig(
    speechConfig,
    autoDetectSourceLanguageConfig,
    audioConfig);

speechRecognitionResult = recognizer->RecognizeOnceAsync().get();
auto autoDetectSourceLanguageResult =
    AutoDetectSourceLanguageResult::FromResult(speechRecognitionResult);
auto detectedLanguage = autoDetectSourceLanguageResult->Language;
```

::: zone-end

::: zone pivot="programming-language-java"

```java
AutoDetectSourceLanguageConfig autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.fromLanguages(Arrays.asList("en-US", "de-DE"));

SpeechRecognizer recognizer = new SpeechRecognizer(
    speechConfig,
    autoDetectSourceLanguageConfig,
    audioConfig);

Future<SpeechRecognitionResult> future = recognizer.recognizeOnceAsync();
SpeechRecognitionResult result = future.get(30, TimeUnit.SECONDS);
AutoDetectSourceLanguageResult autoDetectSourceLanguageResult =
    AutoDetectSourceLanguageResult.fromResult(result);
String detectedLanguage = autoDetectSourceLanguageResult.getLanguage();

recognizer.close();
speechConfig.close();
autoDetectSourceLanguageConfig.close();
audioConfig.close();
result.close();
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
auto_detect_source_language_config = \
        speechsdk.languageconfig.AutoDetectSourceLanguageConfig(languages=["en-US", "de-DE"])
speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config, 
        auto_detect_source_language_config=auto_detect_source_language_config, 
        audio_config=audio_config)
result = speech_recognizer.recognize_once()
auto_detect_source_language_result = speechsdk.AutoDetectSourceLanguageResult(result)
detected_language = auto_detect_source_language_result.language
```

::: zone-end

::: zone pivot="programming-language-objectivec"

```Objective-C
NSArray *languages = @[@"zh-CN", @"de-DE"];
SPXAutoDetectSourceLanguageConfiguration* autoDetectSourceLanguageConfig = \
        [[SPXAutoDetectSourceLanguageConfiguration alloc]init:languages];
SPXSpeechRecognizer* speechRecognizer = \
        [[SPXSpeechRecognizer alloc] initWithSpeechConfiguration:speechConfig
                           autoDetectSourceLanguageConfiguration:autoDetectSourceLanguageConfig
                                              audioConfiguration:audioConfig];
SPXSpeechRecognitionResult *result = [speechRecognizer recognizeOnce];
SPXAutoDetectSourceLanguageResult *languageDetectionResult = [[SPXAutoDetectSourceLanguageResult alloc] init:result];
NSString *detectedLanguage = [languageDetectionResult language];
```

::: zone-end

::: zone pivot="programming-language-javascript"

```Javascript
var autoDetectConfig = SpeechSDK.AutoDetectSourceLanguageConfig.fromLanguages(["en-US", "de-DE"]);
var speechRecognizer = SpeechSDK.SpeechRecognizer.FromConfig(speechConfig, audioConfig, autoDetectConfig);
speechRecognizer.recognizeOnceAsync((result: SpeechSDK.SpeechRecognitionResult) => {
        var languageDetectionResult = SpeechSDK.AutoDetectSourceLanguageResult.fromResult(result);
        var detectedLanguage = languageDetectionResult.language;
},
{});
```

::: zone-end

## <a name="use-a-custom-model-for-automatic-language-detection"></a>Uso de un modelo personalizado para la detección automática de idioma

Además de la detección de idiomas mediante los modelos del servicio de Voz, puede especificar un modelo personalizado para obtener reconocimiento mejorado. Si no se proporciona un modelo personalizado, el servicio utilizará el modelo de lenguaje predeterminado.

En los fragmentos de código siguientes se muestra cómo especificar un modelo personalizado en la llamada al servicio de Voz. Si el idioma detectado es `en-US`, se usa el modelo predeterminado. Si el idioma detectado es `fr-FR`, se usa el punto de conexión para el modelo personalizado:

::: zone pivot="programming-language-csharp"

```csharp
var sourceLanguageConfigs = new SourceLanguageConfig[]
{
    SourceLanguageConfig.FromLanguage("en-US"),
    SourceLanguageConfig.FromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR")
};
var autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.FromSourceLanguageConfigs(
        sourceLanguageConfigs);
```

::: zone-end

::: zone pivot="programming-language-cpp"

```cpp
std::vector<std::shared_ptr<SourceLanguageConfig>> sourceLanguageConfigs;
sourceLanguageConfigs.push_back(
    SourceLanguageConfig::FromLanguage("en-US"));
sourceLanguageConfigs.push_back(
    SourceLanguageConfig::FromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR"));

auto autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig::FromSourceLanguageConfigs(
        sourceLanguageConfigs);
```

::: zone-end

::: zone pivot="programming-language-java"

```java
List sourceLanguageConfigs = new ArrayList<SourceLanguageConfig>();
sourceLanguageConfigs.add(
    SourceLanguageConfig.fromLanguage("en-US"));
sourceLanguageConfigs.add(
    SourceLanguageConfig.fromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR"));

AutoDetectSourceLanguageConfig autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.fromSourceLanguageConfigs(
        sourceLanguageConfigs);
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
 en_language_config = speechsdk.languageconfig.SourceLanguageConfig("en-US")
 fr_language_config = speechsdk.languageconfig.SourceLanguageConfig("fr-FR", "The Endpoint Id for custom model of fr-FR")
 auto_detect_source_language_config = speechsdk.languageconfig.AutoDetectSourceLanguageConfig(
        sourceLanguageConfigs=[en_language_config, fr_language_config])
```

::: zone-end

::: zone pivot="programming-language-objectivec"

```Objective-C
SPXSourceLanguageConfiguration* enLanguageConfig = [[SPXSourceLanguageConfiguration alloc]init:@"en-US"];
SPXSourceLanguageConfiguration* frLanguageConfig = \
        [[SPXSourceLanguageConfiguration alloc]initWithLanguage:@"fr-FR"
                                                     endpointId:@"The Endpoint Id for custom model of fr-FR"];
NSArray *languageConfigs = @[enLanguageConfig, frLanguageConfig];
SPXAutoDetectSourceLanguageConfiguration* autoDetectSourceLanguageConfig = \
        [[SPXAutoDetectSourceLanguageConfiguration alloc]initWithSourceLanguageConfigurations:languageConfigs];
```

::: zone-end

::: zone pivot="programming-language-javascript"

```Javascript
var enLanguageConfig = SpeechSDK.SourceLanguageConfig.fromLanguage("en-US");
var frLanguageConfig = SpeechSDK.SourceLanguageConfig.fromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR");
var autoDetectConfig = SpeechSDK.AutoDetectSourceLanguageConfig.fromSourceLanguageConfigs([enLanguageConfig, frLanguageConfig]);
```

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

- [Documentación de referencia del SDK de voz](speech-sdk.md)