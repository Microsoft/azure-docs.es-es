---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/06/2020
ms.author: trbye
ms.openlocfilehash: ff171dfce0bcbb04ec017a8d5e3310cf3162e8e2
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89565004"
---
## <a name="prerequisites"></a>Prerrequisitos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../get-started.md).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

En primer lugar, deberá instalar Speech SDK. Utilice las siguientes instrucciones en función de la plataforma:

* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=linux&pivots=programming-language-cpp" target="_blank">Linux <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=macos&pivots=programming-language-cpp" target="_blank">macOS <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=windows&pivots=programming-language-cpp" target="_blank">Windows <span class="docon docon-navigate-external x-hidden-focus"></span></a>

## <a name="create-a-speech-configuration"></a>Creación de una configuración de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechConfig`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechconfig). Esta clase incluye información sobre la suscripción, como la clave, la región asociada, el punto de conexión, el host o el token de autorización.

> [!NOTE]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.

Existen diversas maneras para inicializar un elemento [`SpeechConfig`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechconfig):

* Con una suscripción: pase una clave y la región asociada.
* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

Vamos a echar un vistazo al procedimiento de creación de un elemento [`SpeechConfig`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechconfig) con una clave y una región. Consulte la página de [soporte por regiones](https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#speech-sdk) para buscar el identificador de región.

```cpp
auto config = SpeechConfig::FromSubscription("YourSubscriptionKey", "YourServiceRegion");
```

## <a name="initialize-a-recognizer"></a>Inicialización de un reconocedor

Una vez creado un elemento [`SpeechConfig`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechconfig), el paso siguiente consiste en inicializar un elemento [`SpeechRecognizer`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechrecognizer). Al inicializar un elemento [`SpeechRecognizer`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechrecognizer), deberá pasar el elemento `speech_config`. Esto proporciona las credenciales que necesita el servicio de voz para validar la solicitud.

```cpp
auto recognizer = SpeechRecognizer::FromConfig(config);
```

## <a name="recognize-from-microphone-or-file"></a>Reconocimiento desde un micrófono o archivo

Si desea especificar el dispositivo de entrada de audio, es preciso que cree [`AudioConfig`](https://docs.microsoft.com/cpp/cognitive-services/speech/audio-audioconfig) y lo use como parámetro al inicializar [`SpeechRecognizer`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechrecognizer).

Para reconocer la voz mediante el micrófono del dispositivo, cree un elemento `AudioConfig` mediante `FromDefaultMicrophoneInput()`y, después, utilice la configuración de audio al crear el objeto `SpeechRecognizer`.

```cpp
using namespace Microsoft::CognitiveServices::Speech::Audio;

auto audioConfig = AudioConfig::FromDefaultMicrophoneInput();
auto recognizer = SpeechRecognizer::FromConfig(config, audioConfig);
```

> [!TIP]
> [Obtenga información sobre cómo obtener el identificador de dispositivo del dispositivo de entrada de audio](../../../how-to-select-audio-input-devices.md).

Si desea reconocer la voz de un archivo de audio, en lugar de usar un micrófono, tiene que crear un elemento `AudioConfig`. Sin embargo, al crear [`AudioConfig`](https://docs.microsoft.com/cpp/cognitive-services/speech/audio-audioconfig), en lugar de llamar a `FromDefaultMicrophoneInput()`, llama a `FromWavFileInput()` y usa el parámetro `filename`.

```cpp
auto audioInput = AudioConfig::FromWavFileInput("YourAudioFile.wav");
auto recognizer = SpeechRecognizer::FromConfig(config, audioInput);
```

## <a name="recognize-speech"></a>Reconocer la voz

La [clase Recognizer](https://docs.microsoft.com/cpp/cognitive-services/speech/speechrecognizer) de Speech SDK para C++ expone algunos métodos que puede usar para el reconocimiento de voz.

* Reconocimiento de una sola captura (asincrónico): realiza el reconocimiento en modo sin bloqueo (asincrónico). Esto reconocerá una expresión única. El final de una expresión única se determina mediante la escucha de un silencio al final o hasta que se procesa un máximo de 15 segundos de audio.
* Reconocimiento continuo (asincrónico): inicia de forma asincrónica la operación de reconocimiento continuo. El usuario se tiene que conectar para controlar el evento para recibir los resultados del reconocimiento. Para detener el reconocimiento continuo asincrónico, llame a [`StopContinuousRecognitionAsync`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechrecognizer#stopcontinuousrecognitionasync).

> [!NOTE]
> Obtenga información adicional sobre cómo [elegir el modo de reconocimiento de voz](../../../how-to-choose-recognition-mode.md).

### <a name="single-shot-recognition"></a>Reconocimiento de una sola captura

Este es un ejemplo de reconocimiento asincrónico de una sola captura mediante [`RecognizeOnceAsync`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechrecognizer#recognizeonceasync):

```cpp
auto result = recognizer->RecognizeOnceAsync().get();
```

Deberá escribir código para controlar el resultado. Este ejemplo evalúa el elemento [`result->Reason`](https://docs.microsoft.com/cpp/cognitive-services/speech/recognitionresult#reason):

* Imprime el resultado del reconocimiento: `ResultReason::RecognizedSpeech`.
* Si no hay ninguna coincidencia de reconocimiento, se informa al usuario: `ResultReason::NoMatch`.
* Si se produce un error, se imprime el mensaje de error: `ResultReason::Canceled`.

```cpp
switch (result->Reason)
{
    case ResultReason::RecognizedSpeech:
        cout << "We recognized: " << result->Text << std::endl;
        break;
    case ResultReason::NoMatch:
        cout << "NOMATCH: Speech could not be recognized." << std::endl;
        break;
    case ResultReason::Canceled:
        {
            auto cancellation = CancellationDetails::FromResult(result);
            cout << "CANCELED: Reason=" << (int)cancellation->Reason << std::endl;
    
            if (cancellation->Reason == CancellationReason::Error) {
                cout << "CANCELED: ErrorCode= " << (int)cancellation->ErrorCode << std::endl;
                cout << "CANCELED: ErrorDetails=" << cancellation->ErrorDetails << std::endl;
                cout << "CANCELED: Did you update the subscription info?" << std::endl;
            }
        }
        break;
    default:
        break;
}
```

### <a name="continuous-recognition"></a>Reconocimiento continuo

El reconocimiento continuo es un poco más complicado que el reconocimiento de una sola captura. Requiere que se suscriba a los eventos `Recognizing`, `Recognized` y `Canceled` para obtener los resultados del reconocimiento. Para detener el reconocimiento, debe llamar a [StopContinuousRecognitionAsync](https://docs.microsoft.com/cpp/cognitive-services/speech/speechrecognizer#stopcontinuousrecognitionasync). Este es un ejemplo de cómo se realiza el reconocimiento continuo en un archivo de entrada de audio.

Comencemos por definir la entrada e inicializar un elemento [`SpeechRecognizer`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechrecognizer):

```cpp
auto audioInput = AudioConfig::FromWavFileInput("YourAudioFile.wav");
auto recognizer = SpeechRecognizer::FromConfig(config, audioInput);
```

A continuación, vamos a crear una variable para administrar el estado del reconocimiento de voz. Para empezar, vamos a declarar un elemento `promise<void>`, ya que al inicio del reconocimiento podemos suponer que no ha terminado.

```cpp
promise<void> recognitionEnd;
```

Nos suscribiremos a los eventos enviados desde el elemento [`SpeechRecognizer`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechrecognizer).

* [`Recognizing`](https://docs.microsoft.com/cpp/cognitive-services/speech/asyncrecognizer#recognizing): se señalizan los eventos que contienen los resultados intermedios del reconocimiento.
* [`Recognized`](https://docs.microsoft.com/cpp/cognitive-services/speech/asyncrecognizer#recognized): se señalizan los eventos que contienen los resultados finales del reconocimiento (lo que indica un intento de reconocimiento correcto).
* [`SessionStopped`](https://docs.microsoft.com/cpp/cognitive-services/speech/asyncrecognizer#sessionstopped): se señalizan los eventos que indican el final de una sesión de reconocimiento (operación).
* [`Canceled`](https://docs.microsoft.com/cpp/cognitive-services/speech/asyncrecognizer#canceled): se señalizan los eventos que contienen los resultados de reconocimiento cancelados (que indican un intento de reconocimiento que se canceló como resultado de una solicitud de cancelación directa o, de forma alternativa, por un error de protocolo o transporte).

```cpp
recognizer->Recognizing.Connect([](const SpeechRecognitionEventArgs& e)
    {
        cout << "Recognizing:" << e.Result->Text << std::endl;
    });

recognizer->Recognized.Connect([](const SpeechRecognitionEventArgs& e)
    {
        if (e.Result->Reason == ResultReason::RecognizedSpeech)
        {
            cout << "RECOGNIZED: Text=" << e.Result->Text 
                 << " (text could not be translated)" << std::endl;
        }
        else if (e.Result->Reason == ResultReason::NoMatch)
        {
            cout << "NOMATCH: Speech could not be recognized." << std::endl;
        }
    });

recognizer->Canceled.Connect([&recognitionEnd](const SpeechRecognitionCanceledEventArgs& e)
    {
        cout << "CANCELED: Reason=" << (int)e.Reason << std::endl;
        if (e.Reason == CancellationReason::Error)
        {
            cout << "CANCELED: ErrorCode=" << (int)e.ErrorCode << "\n"
                 << "CANCELED: ErrorDetails=" << e.ErrorDetails << "\n"
                 << "CANCELED: Did you update the subscription info?" << std::endl;

            recognitionEnd.set_value(); // Notify to stop recognition.
        }
    });

recognizer->SessionStopped.Connect([&recognitionEnd](const SessionEventArgs& e)
    {
        cout << "Session stopped.";
        recognitionEnd.set_value(); // Notify to stop recognition.
    });
```

Con todo configurado, podemos llamar a [`StopContinuousRecognitionAsync`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechrecognizer#startcontinuousrecognitionasync).

```cpp
// Starts continuous recognition. Uses StopContinuousRecognitionAsync() to stop recognition.
recognizer->StartContinuousRecognitionAsync().get();

// Waits for recognition end.
recognitionEnd.get_future().get();

// Stops recognition.
recognizer->StopContinuousRecognitionAsync().get();
```

### <a name="dictation-mode"></a>Modo de dictado

Al usar el reconocimiento continuo, puede habilitar el procesamiento de dictado mediante la función "habilitar dictado" correspondiente. Este modo hará que la instancia de configuración de voz interprete las descripciones de palabras de estructuras de oraciones como puntuación. Por ejemplo, la expresión "Interrogación de apertura vive en la ciudad interrogación de cierre" se interpretaría como el texto "¿Vive en la ciudad?".

Para habilitar el modo de dictado, use el método [`EnableDictation`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechconfig#enabledictation) del elemento [`SpeechConfig`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechconfig).

```cpp
config->EnableDictation();
```

## <a name="change-source-language"></a>Cambio del idioma de origen

Una tarea común en el reconocimiento de voz es especificar el idioma de entrada (u origen). Echemos un vistazo a cómo se cambiaría el idioma de entrada a alemán. En el código, busque [`SpeechConfig`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechconfig) y, a continuación, agregue esta línea directamente debajo.

```cpp
config->SetSpeechRecognitionLanguage("de-DE");
```

[`SetSpeechRecognitionLanguage`](https://docs.microsoft.com/cpp/cognitive-services/speech/speechconfig#setspeechrecognitionlanguage) es un parámetro que toma una cadena como argumento. Puede proporcionar cualquier valor de la lista de [idiomas o configuraciones regionales](../../../language-support.md) compatibles.

## <a name="improve-recognition-accuracy"></a>Mejora de la precisión del reconocimiento

Hay varias maneras de mejorar la precisión del reconocimiento con Speech SDK. Echemos un vistazo a las listas de frases. Las listas de frases se usan para identificar frases conocidas en datos de audio, como el nombre de una persona o una ubicación específica. A una lista de frases se pueden agregar palabras solas o frases completas. Durante el reconocimiento, se usa una entrada de una lista de frases si el audio incluye una coincidencia exacta de una frase completa. Si no se encuentra una coincidencia exacta con la frase, el reconocimiento no está asistido.

> [!IMPORTANT]
> La característica Lista de frases solo está disponible en inglés.

Para usar una lista de frases, primero debe crear un objeto [`PhraseListGrammar`](https://docs.microsoft.com/cpp/cognitive-services/speech/phraselistgrammar) y, a continuación, agregar palabras y frases específicas con [`AddPhrase`](https://docs.microsoft.com/cpp/cognitive-services/speech/phraselistgrammar#addphrase).

Los cambios realizados en el objeto [`PhraseListGrammar`](https://docs.microsoft.com/cpp/cognitive-services/speech/phraselistgrammar) se aplican en el siguiente reconocimiento o después de una reconexión al servicio de voz.

```cpp
auto phraseListGrammar = PhraseListGrammar::FromRecognizer(recognizer);
phraseListGrammar->AddPhrase("Supercalifragilisticexpialidocious");
```

Si necesita borrar la lista de frases: 

```cpp
phraseListGrammar->Clear();
```

### <a name="other-options-to-improve-recognition-accuracy"></a>Otras opciones para mejorar la precisión del reconocimiento

Las listas de frases son solo una opción para mejorar la precisión del reconocimiento. También puede: 

* [Mejora de la precisión con Habla personalizada](../../../how-to-custom-speech.md)
* [Mejora de la precisión con modelos de inquilino](../../../tutorial-tenant-model.md)
