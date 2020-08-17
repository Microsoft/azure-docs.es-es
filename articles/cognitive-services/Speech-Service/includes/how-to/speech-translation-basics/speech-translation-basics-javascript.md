---
author: v-demjoh
ms.service: cognitive-services
ms.topic: include
ms.date: 07/14/2020
ms.author: v-demjoh
ms.custom: devx-track-javascript
ms.openlocfilehash: 6fe89e36685a2db0bf7b570ab6e7150bacf34857
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87405845"
---
## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../get-started.md).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

En primer lugar, deberá instalar <a href="https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk" target="_blank">Speech SDK para JavaScript<span class="docon docon-navigate-external x-hidden-focus"></span></a>. Utilice las siguientes instrucciones en función de la plataforma:
- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=nodejs#get-the-speech-sdk" target="_blank">Node.js <span 
class="docon docon-navigate-external x-hidden-focus"></span></a>
- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=browser#get-the-speech-sdk" target="_blank">Explorador web <span class="docon docon-navigate-external x-hidden-focus"></span></a>

Además, en función del entorno de destino, use una de las siguientes:

# <a name="script"></a>[script](#tab/script)

Descargue y extraiga el archivo *microsoft.cognitiveservices.speech.sdk.bundle.js* de <a href="https://aka.ms/csspeech/jsbrowserpackage" target="_blank">Speech SDK para JavaScript <span class="docon docon-navigate-external x-hidden-focus"></span></a> y colóquelo en una carpeta accesible para el archivo HTML.

```html
<script src="microsoft.cognitiveservices.speech.sdk.bundle.js"></script>;
```

> [!TIP]
> Si el destino es un explorador web y usa la etiqueta `<script>`, el prefijo `sdk` no es necesario. El prefijo `sdk` es un alias que se usa para asignar un nombre al módulo `require`.

# <a name="import"></a>[import](#tab/import)

```javascript
import * from "microsoft-cognitiveservices-speech-sdk";
```

Para más información sobre `import`, consulte <a href="https://javascript.info/import-export" target="_blank">export and import <span class="docon docon-navigate-external x-hidden-focus"></span></a> (export e import).

# <a name="require"></a>[require](#tab/require)

```javascript
const sdk = require("microsoft-cognitiveservices-speech-sdk");
```

Para más información sobre `require`, consulte <a href="https://nodejs.org/en/knowledge/getting-started/what-is-require/" target="_blank">what is require? <span class="docon docon-navigate-external x-hidden-focus"></span></a> (¿Qué es require?).

---

## <a name="create-a-translation-configuration"></a>Creación de una configuración de traducción

Para llamar al servicio de traducción con Speech SDK, debe crear un elemento [`SpeechTranslationConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?view=azure-node-latest). Esta clase incluye información sobre la suscripción, como la clave, la región asociada, el punto de conexión, el host o el token de autorización.

> [!NOTE]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.
Existen diversas maneras para inicializar un elemento [`SpeechTranslationConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?view=azure-node-latest):

* Con una suscripción: pase una clave y la región asociada.
* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

Vamos a echar un vistazo al procedimiento de creación de un elemento [`SpeechTranslationConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?view=azure-node-latest) con una clave y una región. Consulte la página de [soporte por regiones](https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#speech-sdk) para buscar el identificador de región.

```javascript
const speechTranslationConfig = SpeechTranslationConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
```

## <a name="initialize-a-translator"></a>Inicialización de un traductor

Una vez creado un elemento [`SpeechTranslationConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?view=azure-node-latest), el paso siguiente consiste en inicializar un elemento [`TranslationRecognizer`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest). Al inicializar un elemento [`TranslationRecognizer`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest), deberá pasar el elemento `speechTranslationConfig`. Esto proporciona las credenciales que necesita el servicio de traducción para validar la solicitud.

Si va a traducir la voz proporcionada a través del micrófono predeterminado del dispositivo, este es el aspecto que el [`TranslationRecognizer`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest) debería tener:

```javascript
const translator = new TranslationRecognizer(speechTranslationConfig);
```

Si desea especificar el dispositivo de entrada de audio, deberá crear un elemento [`AudioConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig?view=azure-node-latest) y proporcionar el parámetro `audioConfig` al inicializar el elemento [`TranslationRecognizer`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest).

> [!TIP]
> [Obtenga información sobre cómo obtener el identificador de dispositivo del dispositivo de entrada de audio](../../../how-to-select-audio-input-devices.md).
Haga referencia al objeto `AudioConfig` como se indica a continuación:

```javascript
const audioConfig = AudioConfig.fromDefaultMicrophoneInput();
const recognizer = new TranslationRecognizer(speechTranslationConfig, audioConfig);
```

Si desea proporcionar un archivo de audio en lugar de usar un micrófono, deberá proporcionar un elemento `audioConfig`. Sin embargo, esto solo se puede hacer cuando el destino es **Node.js** y al crear [`AudioConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig?view=azure-node-latest) se llama a `fromWavFileOutput` y se pasa el parámetro `filename`, en lugar de llamar a `fromDefaultMicrophoneInput`.

```javascript
const audioConfig = AudioConfig.fromWavFileInput("YourAudioFile.wav");
const recognizer = new TranslationRecognizer(speechTranslationConfig, audioConfig);
```

## <a name="translate-speech"></a>Traducir voz

La [clase TranslationRecognizer](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest) de Speech SDK para JavaScript expone algunos métodos que puede usar para la traducción de voz.

* Traducción de una sola captura (asincrónico): realiza la traducción en modo sin bloqueo (asincrónico). Esto traducirá una expresión única. El final de una expresión única se determina mediante la escucha de un silencio al final o hasta que se procesa un máximo de 15 segundos de audio.
* Traducción continua (asincrónica): inicia de forma asincrónica la operación de traducción continua. El usuario se registra a eventos y controla los distintos estados de la aplicación. Para detener la traducción continua asincrónica, llame a [`stopContinuousRecognitionAsync`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest#stopcontinuousrecognitionasync).

> [!NOTE]
> Obtenga información adicional sobre cómo [elegir el modo de reconocimiento de voz](../../../how-to-choose-recognition-mode.md).
### <a name="specify-a-target-language"></a>Especificación de un idioma de destino

Para traducir, debe especificar un idioma de origen y al menos un idioma de destino.
Puede elegir un idioma de origen mediante una configuración regional incluida en la [tabla de traducción de voz](../../../language-support.md#speech-translation). Busque las opciones de idioma de traducción en el mismo vínculo. Las opciones de idiomas de destino se diferencian cuando se quiere ver el texto o cuando quiere oír una voz traducida sintetizada. Para traducir del inglés al alemán, modifique el objeto de configuración de traducción:

```javascript
speechTranslationConfig.speechRecognitionLanguage = "en-US";
speechTranslationConfig.addTargetLanguage("de");
```

### <a name="single-shot-recognition"></a>Reconocimiento de una sola captura

Este es un ejemplo de traducción asincrónica de una sola captura mediante [`recognizeOnceAsync`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest#recognizeonceasync):

```javascript
recognizer.recognizeOnceAsync(result => {
    // Interact with result
});
```

Deberá escribir código para controlar el resultado. Este ejemplo evalúa el elemento [`result.reason`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognitionresult?view=azure-node-latest) de una traducción al alemán:

```javascript
recognizer.recognizeOnceAsync(
  function (result) {
    let translation = result.translations.get("de");
    window.console.log(translation);
    recognizer.close();
  },
  function (err) {
    window.console.log(err);
    recognizer.close();
});
```

El código también puede controlar las actualizaciones proporcionadas mientras se procesa la traducción.
Puede usar estas actualizaciones para proporcionar comentarios visuales sobre el progreso de la traducción.
Consulte [este ejemplo de Node.js de JavaScript](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/js/node/translation.js) para ver el código de ejemplo que muestra las actualizaciones proporcionadas durante el proceso de traducción. En el código siguiente también se muestran los detalles que se han producido durante el proceso de traducción.

```javascript
recognizer.recognizing = function (s, e) {
    var str = ("(recognizing) Reason: " + SpeechSDK.ResultReason[e.result.reason] +
            " Text: " +  e.result.text +
            " Translation:");
    str += e.result.translations.get("de");
    console.log(str);
};
recognizer.recognized = function (s, e) {
    var str = "\r\n(recognized)  Reason: " + SpeechSDK.ResultReason[e.result.reason] +
            " Text: " + e.result.text +
            " Translation:";
    str += e.result.translations.get("de");
    str += "\r\n";
    console.log(str);
};
```

### <a name="continuous-translation"></a>Traducción continua

La traducción continua es un poco más complicada que el reconocimiento de una sola captura. Requiere que se suscriba a los eventos `recognizing`, `recognized` y `canceled` para obtener los resultados del reconocimiento. Para detener la traducción, debe llamar a [`stopContinuousRecognitionAsync`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest#stopcontinuousrecognitionasync). Este es un ejemplo de cómo se realiza la traducción continua en un archivo de entrada de audio.

Comencemos por definir la entrada e inicializar un elemento [`TranslationRecognizer`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest):

```javascript
const translator = new TranslationRecognizer(speechTranslationConfig);
```

Nos suscribiremos a los eventos enviados desde el elemento [`TranslationRecognizer`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest).

* [`recognizing`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest#recognizing): se señalizan los eventos que contienen los resultados intermedios de la traducción.
* [`recognized`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest#recognized): se señalizan los eventos que contienen los resultados finales de la traducción (lo que indica un intento de traducción correcto).
* [`sessionStopped`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest#sessionstopped): se señalizan los eventos que indican el final de una sesión de traducción (operación).
* [`canceled`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?view=azure-node-latest#canceled): se señalizan los eventos que contienen los resultados de traducción cancelados (que indican un intento de traducción que se canceló como resultado de una solicitud de cancelación directa o, de forma alternativa, por un error de protocolo o transporte).

```javascript
recognizer.recognizing = (s, e) => {
    console.log(`TRANSLATING: Text=${e.result.text}`);
};
recognizer.recognized = (s, e) => {
    if (e.result.reason == ResultReason.RecognizedSpeech) {
        console.log(`TRANSLATED: Text=${e.result.text}`);
    }
    else if (e.result.reason == ResultReason.NoMatch) {
        console.log("NOMATCH: Speech could not be translated.");
    }
};
recognizer.canceled = (s, e) => {
    console.log(`CANCELED: Reason=${e.reason}`);
    if (e.reason == CancellationReason.Error) {
        console.log(`"CANCELED: ErrorCode=${e.errorCode}`);
        console.log(`"CANCELED: ErrorDetails=${e.errorDetails}`);
        console.log("CANCELED: Did you update the subscription info?");
    }
    recognizer.stopContinuousRecognitionAsync();
};
recognizer.sessionStopped = (s, e) => {
    console.log("\n    Session stopped event.");
    recognizer.stopContinuousRecognitionAsync();
};
```

Con todo configurado, podemos llamar a [`startContinuousRecognitionAsync`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?view=azure-node-latest#startcontinuousrecognitionasync).

```javascript
// Starts continuous recognition. Uses stopContinuousRecognitionAsync() to stop recognition.
recognizer.startContinuousRecognitionAsync();
// Something later can call, stops recognition.
// recognizer.StopContinuousRecognitionAsync();
```

## <a name="choose-a-source-language"></a>Elección de un idioma de origen

Una tarea común en la traducción de voz es especificar el idioma de entrada (u origen). Echemos un vistazo a cómo se cambiaría el idioma de entrada a italiano. En el código, busque [`SpeechTranslationConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?view=azure-node-latest) y, a continuación, agregue esta línea directamente debajo.

```javascript
speechTranslationConfig.speechRecognitionLanguage = "it-IT";
```

La propiedad [`speechRecognitionLanguage`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?view=azure-node-latest#speechrecognitionlanguage) espera una cadena con formato de configuración regional de idioma. En la columna **Locale**, puede proporcionar cualquier valor de la lista de idiomas o configuraciones regionales [compatibles](../../../language-support.md).

## <a name="choose-one-or-more-target-languages"></a>Elección de uno o varios idiomas de destino

Speech SDK puede traducir a varios idiomas de destino en paralelo. Los idiomas de destino disponibles son ligeramente distintos de la lista de idiomas de origen y los idiomas de destino se especifican con un código de idioma, en lugar de una configuración regional.
Consulte una lista de códigos de idioma para los destinos de texto en [la tabla de traducción de voz en la página de compatibilidad con idiomas](../../../language-support.md#speech-translation). Ahí también puede encontrar detalles sobre la traducción a idiomas sintetizados.

En el código siguiente se agrega el alemán como idioma de destino:

```javascript
translationConfig.addTargetLanguage("de");
```

Dado que es posible realizar traducciones a varios idiomas de destino, el código debe especificar el idioma de destino al examinar el resultado. El código siguiente obtiene los resultados de la traducción para el alemán.

```javascript
recognizer.recognized = function (s, e) {
    var str = "\r\n(recognized)  Reason: " +
            sdk.ResultReason[e.result.reason] +
            " Text: " + e.result.text + " Translations:";
    var language = "de";
    str += " [" + language + "] " + e.result.translations.get(language);
    str += "\r\n";
    // show str somewhere
};
```
