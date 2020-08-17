---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/11/2020
ms.custom: devx-track-java
ms.author: trbye
ms.openlocfilehash: 314617554abf8fee430e47eb4b0a0ca5db5bc75f
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87375700"
---
## <a name="prerequisites"></a>Prerrequisitos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../get-started.md).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

En primer lugar, deberá instalar Speech SDK. Utilice las siguientes instrucciones en función de la plataforma:

* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=jre&pivots=programming-language-java" target="_blank">Java Runtime <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=android&pivots=programming-language-java" target="_blank">Android <span class="docon docon-navigate-external x-hidden-focus"></span></a>

## <a name="create-a-speech-configuration"></a>Creación de una configuración de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechConfig`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig?view=azure-java-stable). Esta clase incluye información sobre la suscripción, como la clave, la región asociada, el punto de conexión, el host o el token de autorización.

> [!NOTE]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.

Existen diversas maneras para inicializar un elemento [`SpeechConfig`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig?view=azure-java-stable):

* Con una suscripción: pase una clave y la región asociada.
* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

Vamos a echar un vistazo al procedimiento de creación de un elemento [`SpeechConfig`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig?view=azure-java-stable) con una clave y una región. Consulte la página de [soporte por regiones](https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#speech-sdk) para buscar el identificador de región.

```java
SpeechConfig config = SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
```

## <a name="initialize-a-recognizer"></a>Inicialización de un reconocedor

Una vez creado un elemento [`SpeechConfig`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig?view=azure-java-stable), el paso siguiente consiste en inicializar un elemento [`SpeechRecognizer`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer?view=azure-java-stable). Al inicializar un elemento [`SpeechRecognizer`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer?view=azure-java-stable), deberá pasar el elemento `config`. Esto proporciona las credenciales que necesita el servicio de voz para validar la solicitud.

Si va a realizar un reconocimiento de voz con el micrófono predeterminado del dispositivo, el elemento [`SpeechRecognizer`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer?view=azure-java-stable) debería ser similar a:

```java
SpeechRecognizer recognizer = new SpeechRecognizer(config);
```

Si desea especificar el dispositivo de entrada de audio, deberá crear un elemento [`AudioConfig`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.audio.audioconfig?view=azure-java-stable) y proporcionar el parámetro `audioConfig` al inicializar el elemento [`SpeechRecognizer`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer?view=azure-java-stable).

> [!TIP]
> [Obtenga información sobre cómo obtener el identificador de dispositivo del dispositivo de entrada de audio](../../../how-to-select-audio-input-devices.md).

En primer lugar, agregue las siguientes instrucciones `import`.

```java
import java.util.concurrent.Future;
import com.microsoft.cognitiveservices.speech.*;
```

A continuación, podrá hacer referencia al objeto `AudioConfig`, como se indica a continuación:

```java
AudioConfig audioConfig = AudioConfig.fromDefaultMicrophoneInput();
SpeechRecognizer recognizer = new SpeechRecognizer(config, audioConfig);
```

Si desea proporcionar un archivo de audio en lugar de usar un micrófono, deberá proporcionar un elemento `audioConfig`. Sin embargo, cuando se crea un elemento [`AudioConfig`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.audio.audioconfig?view=azure-java-stable), en lugar de llamar a `fromDefaultMicrophoneInput`, llamará a `fromWavFileOutput` y pasará el parámetro `filename`.

```java
AudioConfig audioConfig = AudioConfig.fromWavFileInput("YourAudioFile.wav");
SpeechRecognizer recognizer = new SpeechRecognizer(config, audioConfig);
```

## <a name="recognize-speech"></a>Reconocer la voz

La [clase Recognizer](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer?view=azure-java-stable) de Speech SDK para Java expone algunos métodos que puede usar para el reconocimiento de voz.

* Reconocimiento de una sola captura (asincrónico): realiza el reconocimiento en modo sin bloqueo (asincrónico). Esto reconocerá una expresión única. El final de una expresión única se determina mediante la escucha de un silencio al final o hasta que se procesa un máximo de 15 segundos de audio.
* Reconocimiento continuo (asincrónico): inicia de forma asincrónica la operación de reconocimiento continuo. Si desea proporcionar un archivo de audio en lugar de usar un micrófono, deberá proporcionar un elemento `audioConfig`. Para detener el reconocimiento continuo asincrónico, llame a [stopContinuousRecognitionAsync](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.stopcontinuousrecognitionasync).

> [!NOTE]
> Obtenga información adicional sobre cómo [elegir el modo de reconocimiento de voz](../../../how-to-choose-recognition-mode.md).

### <a name="single-shot-recognition"></a>Reconocimiento de una sola captura

Este es un ejemplo de reconocimiento asincrónico de una sola captura mediante [`recognizeOnceAsync`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.recognizeonceasync?view=azure-java-stable):

```java
Future<SpeechRecognitionResult> task = recognizer.recognizeOnceAsync();
SpeechRecognitionResult result = task.get();
```

Deberá escribir código para controlar el resultado. Este ejemplo evalúa el elemento [`result.getReason()`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.resultreason?view=azure-java-stable):

* Imprime el resultado del reconocimiento: `ResultReason.RecognizedSpeech`.
* Si no hay ninguna coincidencia de reconocimiento, se informa al usuario: `ResultReason.NoMatch`.
* Si se produce un error, se imprime el mensaje de error: `ResultReason.Canceled`.

```java
switch (result.getReason()) {
    case ResultReason.RecognizedSpeech:
        System.out.println("We recognized: " + result.getText());
        exitCode = 0;
        break;
    case ResultReason.NoMatch:
        System.out.println("NOMATCH: Speech could not be recognized.");
        break;
    case ResultReason.Canceled: {
            CancellationDetails cancellation = CancellationDetails.fromResult(result);
            System.out.println("CANCELED: Reason=" + cancellation.getReason());

            if (cancellation.getReason() == CancellationReason.Error) {
                System.out.println("CANCELED: ErrorCode=" + cancellation.getErrorCode());
                System.out.println("CANCELED: ErrorDetails=" + cancellation.getErrorDetails());
                System.out.println("CANCELED: Did you update the subscription info?");
            }
        }
        break;
}
```

### <a name="continuous-recognition"></a>Reconocimiento continuo

El reconocimiento continuo es un poco más complicado que el reconocimiento de una sola captura. Requiere que se suscriba a los eventos `recognizing`, `recognized` y `canceled` para obtener los resultados del reconocimiento. Para detener el reconocimiento, debe llamar a [`stopContinuousRecognitionAsync`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.stopcontinuousrecognitionasync). Este es un ejemplo de cómo se realiza el reconocimiento continuo en un archivo de entrada de audio.

Comencemos por definir la entrada e inicializar un elemento [`SpeechRecognizer`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer?view=azure-java-stable):

```java
AudioConfig audioConfig = AudioConfig.fromWavFileInput("YourAudioFile.wav");
SpeechRecognizer recognizer = new SpeechRecognizer(config, audioConfig);
```

A continuación, vamos a crear una variable para administrar el estado del reconocimiento de voz. Para empezar, declararemos un elemento `Semaphore` en el ámbito de clase.

```java
private static Semaphore stopTranslationWithFileSemaphore;
```

Nos suscribiremos a los eventos enviados desde el elemento [`SpeechRecognizer`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer?view=azure-java-stable).

* [`recognizing`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.recognizing?view=azure-java-stable): se señalizan los eventos que contienen los resultados intermedios del reconocimiento.
* [`recognized`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.recognized?view=azure-java-stable): se señalizan los eventos que contienen los resultados finales del reconocimiento (lo que indica un intento de reconocimiento correcto).
* [`sessionStopped`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.recognizer.sessionstopped?view=azure-java-stable): se señalizan los eventos que indican el final de una sesión de reconocimiento (operación).
* [`canceled`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.canceled?view=azure-java-stable): se señalizan los eventos que contienen los resultados de reconocimiento cancelados (que indican un intento de reconocimiento que se canceló como resultado de una solicitud de cancelación directa o, de forma alternativa, por un error de protocolo o transporte).

```java
// First initialize the semaphore.
stopTranslationWithFileSemaphore = new Semaphore(0);

recognizer.recognizing.addEventListener((s, e) -> {
    System.out.println("RECOGNIZING: Text=" + e.getResult().getText());
});

recognizer.recognized.addEventListener((s, e) -> {
    if (e.getResult().getReason() == ResultReason.RecognizedSpeech) {
        System.out.println("RECOGNIZED: Text=" + e.getResult().getText());
    }
    else if (e.getResult().getReason() == ResultReason.NoMatch) {
        System.out.println("NOMATCH: Speech could not be recognized.");
    }
});

recognizer.canceled.addEventListener((s, e) -> {
    System.out.println("CANCELED: Reason=" + e.getReason());

    if (e.getReason() == CancellationReason.Error) {
        System.out.println("CANCELED: ErrorCode=" + e.getErrorCode());
        System.out.println("CANCELED: ErrorDetails=" + e.getErrorDetails());
        System.out.println("CANCELED: Did you update the subscription info?");
    }

    stopTranslationWithFileSemaphore.release();
});

recognizer.sessionStopped.addEventListener((s, e) -> {
    System.out.println("\n    Session stopped event.");
    stopTranslationWithFileSemaphore.release();
});
```

Con todo configurado, podemos llamar a [`startContinuousRecognitionAsync`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.startcontinuousrecognitionasync).

```java
// Starts continuous recognition. Uses StopContinuousRecognitionAsync() to stop recognition.
recognizer.startContinuousRecognitionAsync().get();

// Waits for completion.
stopTranslationWithFileSemaphore.acquire();

// Stops recognition.
recognizer.stopContinuousRecognitionAsync().get();
```

### <a name="dictation-mode"></a>Modo de dictado

Al usar el reconocimiento continuo, puede habilitar el procesamiento de dictado mediante la función "habilitar dictado" correspondiente. Este modo hará que la instancia de configuración de voz interprete las descripciones de palabras de estructuras de oraciones como puntuación. Por ejemplo, la expresión "Interrogación de apertura vive en la ciudad interrogación de cierre" se interpretaría como el texto "¿Vive en la ciudad?".

Para habilitar el modo de dictado, use el método [`enableDictation`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig.enabledictation?view=azure-java-stable) del elemento [`SpeechConfig`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig?view=azure-java-stable).

```java
config.enableDictation();
```

## <a name="change-source-language"></a>Cambio del idioma de origen

Una tarea común en el reconocimiento de voz es especificar el idioma de entrada (u origen). Echemos un vistazo a cómo se cambiaría el idioma de entrada a francés. En el código, busque [`SpeechConfig`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig?view=azure-java-stable) y, a continuación, agregue esta línea directamente debajo.

```java
config.setSpeechRecognitionLanguage("fr-FR");
```

[`setSpeechRecognitionLanguage`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig.setspeechrecognitionlanguage?view=azure-java-stable) es un parámetro que toma una cadena como argumento. Puede proporcionar cualquier valor de la lista de [idiomas o configuraciones regionales](../../../language-support.md) compatibles.

## <a name="improve-recognition-accuracy"></a>Mejora de la precisión del reconocimiento

Hay varias maneras de mejorar la precisión del reconocimiento con Speech SDK. Echemos un vistazo a las listas de frases. Las listas de frases se usan para identificar frases conocidas en datos de audio, como el nombre de una persona o una ubicación específica. A una lista de frases se pueden agregar palabras solas o frases completas. Durante el reconocimiento, se usa una entrada de una lista de frases si el audio incluye una coincidencia exacta de una frase completa. Si no se encuentra una coincidencia exacta con la frase, el reconocimiento no está asistido.

> [!IMPORTANT]
> La característica Lista de frases solo está disponible en inglés.

Para usar una lista de frases, primero debe crear un objeto [`PhraseListGrammar`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.phraselistgrammar?view=azure-java-stable) y, a continuación, agregar palabras y frases específicas con [`AddPhrase`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.phraselistgrammar.addphrase?view=azure-java-stable#com_microsoft_cognitiveservices_speech_PhraseListGrammar_addPhrase_String_).

Los cambios realizados en el objeto [`PhraseListGrammar`](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.phraselistgrammar?view=azure-java-stable) se aplican en el siguiente reconocimiento o después de una reconexión al servicio de voz.

```java
PhraseListGrammar phraseList = PhraseListGrammar.fromRecognizer(recognizer);
phraseList.addPhrase("Supercalifragilisticexpialidocious");
```

Si necesita borrar la lista de frases: 

```java
phraseList.clear();
```

### <a name="other-options-to-improve-recognition-accuracy"></a>Otras opciones para mejorar la precisión del reconocimiento

Las listas de frases son solo una opción para mejorar la precisión del reconocimiento. También puede: 

* [Mejora de la precisión con Habla personalizada](../../../how-to-custom-speech.md)
* [Mejora de la precisión con modelos de inquilino](../../../tutorial-tenant-model.md)
