---
title: Elección de un modo de reconocimiento de voz con el SDK de Voz
titleSuffix: Azure Cognitive Services
description: Obtenga información acerca de cómo elegir el mejor modo de reconocimiento al usar el SDK de Voz.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/10/2020
ms.author: trbye
zone_pivot_groups: programming-languages-set-two-with-js
ms.custom: devx-track-javascript
ms.openlocfilehash: 82ff34b945e93ffbf43e3a264a09e0af7b8660e5
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87405099"
---
# <a name="choose-a-speech-recognition-mode"></a>Elección de un modo de reconocimiento de voz

Al considerar las operaciones de reconocimiento de voz a texto, el [SDK de Voz](speech-sdk.md) proporciona varios modos para procesar la voz. Conceptualmente, a veces denominado *modo de reconocimiento*. En este artículo se comparan los distintos modos de reconocimiento.

## <a name="recognize-once"></a>Reconocer una vez

Si quiere procesar cada expresión una "oración" a la vez, use la función "reconocer una vez". Este método detectará una expresión reconocida de la entrada a partir del inicio de la voz detectada hasta la siguiente pausa. Normalmente, una pausa marca el final de una frase o una línea de pensamiento.

Al final de una expresión reconocida, el servicio detiene el procesamiento de audio desde esa solicitud. El límite máximo para el reconocimiento es una duración de frase de 20 segundos.

::: zone pivot="programming-language-csharp"

Para obtener más información sobre el uso de la función `RecognizeOnceAsync`, vea la [Documentación de .NET del SDK de Voz](https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechrecognizer.recognizeonceasync?view=azure-dotnet#Microsoft_CognitiveServices_Speech_SpeechRecognizer_RecognizeOnceAsync).

```csharp
var result = await recognizer.RecognizeOnceAsync();
```

::: zone-end
::: zone pivot="programming-language-cpp"

Para obtener más información sobre el uso de la función `RecognizeOnceAsync`, vea la [Documentación de C++ del SDK de Voz](https://docs.microsoft.com/cpp/cognitive-services/speech/asyncrecognizer#recognizeonceasync).

```cpp
auto result = recognize->RecognizeOnceAsync().get();
```

::: zone-end
::: zone pivot="programming-language-java"

Para obtener más información sobre el uso de la función `recognizeOnceAsync`, vea la [Documentación de Java del SDK de Voz](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.SpeechRecognizer.recognizeOnceAsync?view=azure-java-stable).

```java
SpeechRecognitionResult result = recognizer.recognizeOnceAsync().get();
```

::: zone-end
::: zone pivot="programming-language-python"

Para obtener más información sobre el uso de la función `recognize_once`, vea la [Documentación Python del SDK de Voz](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechrecognizer?view=azure-python#recognize-once------azure-cognitiveservices-speech-speechrecognitionresult).

```python
result = speech_recognizer.recognize_once()
```

::: zone-end

::: zone pivot="programming-language-javascript"

Para más información sobre el uso de la función `recognizeOnceAsync`, consulte [Speech SDK para los documentos de JavaScript](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?view=azure-node-latest#recognizeonceasync--e--speechrecognitionresult-----void---e--string-----void-).

```JavaScript
recognizer.recognizeOnceAsync((result)=>{}, (error)=>{}));
```

::: zone-end

::: zone pivot="programming-language-more"

Para consultar otros lenguajes, consulte la [Documentación de referencia del SDK de Voz](speech-to-text.md#speech-sdk-reference-docs).

::: zone-end

## <a name="continuous"></a>Continuo

Si necesita un reconocimiento de ejecución prolongada, use las funciones de inicio y detención correspondientes para el reconocimiento continuo. La función de inicio iniciará y continuará el procesamiento de todas las expresiones hasta que se invoque la función de detención o hasta que haya transcurrido demasiado tiempo en silencio. Al usar el modo continuo, asegúrese de registrarse en los diversos eventos que se activarán cuando se produzcan. Por ejemplo, el evento "reconocido" se activa cuando se produce el reconocimiento de voz. Debe tener un controlador de eventos para controlar el reconocimiento.

::: zone pivot="programming-language-csharp"

```csharp
// Subscribe to event
recognizer.Recognized += (s, e) => 
{
    if (e.Result.Reason == ResultReason.RecognizedSpeech)
    {
        // Do something with the recognized text
        // e.Result.Text
    }
};

// Start continuous speech recognition
await recognizer.StartContinuousRecognitionAsync();

// Stop continuous speech recognition
await recognizer.StopContinuousRecognitionAsync();
```

::: zone-end
::: zone pivot="programming-language-cpp"

```cpp
// Connect to event
recognizer->Recognized.Connect([] (const SpeechRecognitionEventArgs& e)
{
    if (e.Result->Reason == ResultReason::RecognizedSpeech)
    {
        // Do something with the recognized text
        // e.Result->Text
    }
});

// Start continuous speech recognition
recognizer->StartContinuousRecognitionAsync().get();

// Stop continuous speech recognition
recognizer->StopContinuousRecognitionAsync().get();
```

::: zone-end
::: zone pivot="programming-language-java"

```java
recognizer.recognized.addEventListener((s, e) -> {
    if (e.getResult().getReason() == ResultReason.RecognizedSpeech) {
        // Do something with the recognized text
        // e.getResult().getText()
    }
});

// Start continuous speech recognition
recognizer.startContinuousRecognitionAsync().get();

// Stop continuous speech recognition
recognizer.stopContinuousRecognitionAsync().get();
```

::: zone-end
::: zone pivot="programming-language-python"

```python
def recognized_cb(evt):
    if evt.result.reason == speechsdk.ResultReason.RecognizedSpeech:
        # Do something with the recognized text
        # evt.result.text

speech_recognizer.recognized.connect(recognized_cb)

# Start continuous speech recognition
speech_recognizer.start_continuous_recognition()

# Stop continuous speech recognition
speech_recognizer.stop_continuous_recognition()
```

::: zone-end

::: zone pivot="programming-language-javascript"

```JavaScript
recognizer.recognized = (s, e) => {
    if (e.result.reason == ResultReason.RecognizedSpeech) {
        // Do something with the recognized text
        // e.getResult().getText()
    }
});

// Start continuous speech recognition
recognizer.startContinuousRecognitionAsync(()=>{}, (error)=>{});

// Stop continuous speech recognition
recognizer.stopContinuousRecognitionAsync(()=>{}, (error)=>{});
```

::: zone-end

::: zone pivot="programming-language-more"

Para consultar otros lenguajes, consulte la [Documentación de referencia del SDK de Voz](speech-to-text.md#speech-sdk-reference-docs).

::: zone-end

## <a name="dictation"></a>Dictation

Al usar el reconocimiento continuo, puede habilitar el procesamiento de dictado mediante la función "habilitar dictado" correspondiente. Este modo hará que la instancia de configuración de voz interprete las descripciones de palabras de estructuras de oraciones como puntuación. Por ejemplo, la expresión "Interrogación de apertura vive en la ciudad interrogación de cierre" se interpretaría como el texto "¿Vive en la ciudad?".

::: zone pivot="programming-language-csharp"

Para obtener más información sobre el uso de la función `EnableDictation`, vea la [Documentación de .NET del SDK de Voz](https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.enabledictation?view=azure-dotnet#Microsoft_CognitiveServices_Speech_SpeechConfig_EnableDictation).

```csharp
// Enable diction
SpeechConfig.EnableDictation();
```

::: zone-end
::: zone pivot="programming-language-cpp"

Para obtener más información sobre el uso de la función `EnableDictation`, vea la [Documentación de C++ del SDK de Voz](https://docs.microsoft.com/cpp/cognitive-services/speech/speechconfig#enabledictation).

```cpp
// Enable diction
SpeechConfig->EnableDictation();
```

::: zone-end
::: zone pivot="programming-language-java"

Para obtener más información sobre el uso de la función `enableDictation`, vea la [Documentación de Java del SDK de Voz](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.SpeechConfig.enableDictation?view=azure-java-stable).

```java
// Enable diction
SpeechConfig.enableDictation();
```

::: zone-end
::: zone pivot="programming-language-python"

Para obtener más información sobre el uso de la función `enable_dictation`, vea la [Documentación Python del SDK de Voz](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python#enable-dictation--).

```python
# Enable diction
SpeechConfig.enable_dictation()
```

::: zone-end

::: zone pivot="programming-language-javascript"

Para más información sobre el uso de la función `enableDictation`, consulte [Speech SDK para los documentos de JavaScript](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?view=azure-node-latest#enabledictation--).

```JavaScript
// Enable diction
speechConfig.enableDictation();
```

::: zone-end

::: zone pivot="programming-language-more"

Para consultar otros lenguajes, consulte la [Documentación de referencia del SDK de Voz](speech-to-text.md#speech-sdk-reference-docs).

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Explorar ejemplos adicionales del SDK de Voz en GitHub](https://aka.ms/csspeech/samples)
