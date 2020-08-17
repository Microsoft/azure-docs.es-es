---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/15/2020
ms.author: trbye
ms.custom: devx-track-javascript
ms.openlocfilehash: 931fcd113f883f126c2f94e6f3d06684b9a0e772
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407646"
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


## <a name="create-a-speech-configuration"></a>Creación de una configuración de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?view=azure-node-latest). Esta clase incluye información sobre la suscripción, como la clave, la región asociada, el punto de conexión, el host o el token de autorización.

> [!NOTE]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.

Existen diversas maneras para inicializar un elemento [`SpeechConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?view=azure-node-latest):

* Con una suscripción: pase una clave y la región asociada.
* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

En este ejemplo, se crea un elemento [`SpeechConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?view=azure-node-latest) mediante una clave de suscripción y una región. Consulte la página de [soporte por regiones](https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#speech-sdk) para buscar el identificador de región. Cree también código reutilizable básico para usarlo en el resto del artículo y que modificará cuando realice distintas personalizaciones.

```javascript
function synthesizeSpeech() {
    const speechConfig = SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
}
```

## <a name="synthesize-speech-to-a-file"></a>Síntesis de voz en un archivo

A continuación, creará un objeto [`SpeechSynthesizer`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesizer?view=azure-node-latest), que ejecuta conversiones de texto a voz y envía la salida a altavoces, archivos u otros flujos de salida. [`SpeechSynthesizer`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesizer?view=azure-node-latest) acepta como parámetros tanto el objeto [`SpeechConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?view=azure-node-latest) que se creó creado en el paso anterior y un objeto [`AudioConfig`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig?view=azure-node-latest) que especifica cómo se deben controlar los resultados de la salida.

Para empezar, cree un objeto `AudioConfig` para escribir automáticamente la salida en un archivo `.wav` mediante la función estática `fromAudioFileOutput()`.

```javascript
function synthesizeSpeech() {
    const speechConfig = SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const audioConfig = AudioConfig.fromAudioFileOutput("path/to/file.wav");
}
```

Luego, cree una instancia de `SpeechSynthesizer` y use los objetos `speechConfig` y `audioConfig` como parámetros. Después, para ejecutar la síntesis de voz y realizar la escritura en un archivo solo hay que ejecutar `speakTextAsync()` con una cadena de texto. La devolución de llamada de resultados es una excelente ubicación para llamar a `synthesizer.close()`; de hecho, esta llamada es necesaria para que la síntesis funcione correctamente.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const audioConfig = AudioConfig.fromAudioFileOutput("path-to-file.wav");

    const synthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    synthesizer.speakTextAsync(
        "A simple test to write to a file.",
        result => {
            if (result) {
                console.log(JSON.stringify(result));
            }
            synthesizer.close();
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

Ejecute el programa y se escribirá un archivo `.wav` sintetizado en la ubicación que haya especificado. Este es un buen ejemplo del uso más básico, pero a continuación puede examinar cómo personalizar la salida y controlar la respuesta de la salida como una secuencia en memoria para trabajar con escenarios personalizados.

## <a name="synthesize-to-speaker-output"></a>Síntesis a la salida de altavoz

En algunos casos, puede que desee enviar la voz sintetizada directamente a un altavoz. Para ello, cree una instancia de `AudioConfig` mediante la función estática `fromDefaultSpeakerOutput()`. De esta forma, la salida s realiza a través del dispositivo de salida activo actual.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const audioConfig = AudioConfig.fromDefaultSpeakerOutput();

    const synthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    synthesizer.speakTextAsync(
        "Synthesizing directly to speaker output.",
        result => {
            if (result) {
                console.log(JSON.stringify(result));
            }
            synthesizer.close();
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

## <a name="get-result-as-an-in-memory-stream"></a>Obtención del resultado en forma de secuencia en memoria

Para muchos de los escenarios del desarrollo de aplicaciones de voz, es probable que necesite los datos de audio resultantes en forma de secuencia en memoria, en lugar de que se escriban directamente en un archivo. Esto le permitirá crear un comportamiento personalizado, lo que incluye:

* Abstraer la matriz de bytes resultante como una secuencia que permita la búsqueda para los servicios de bajada personalizados.
* Integrar el resultado con otros servicios o API.
* Modificar los datos de audio, escribir encabezados `.wav` personalizados, etc.

Es sencillo hacer este cambio en el ejemplo anterior. En primer lugar, quite el bloqueo de `AudioConfig`, porque a partir de este momento administrará el comportamiento de la salida de forma manual, ya que así obtiene mayor control. Después, use `undefined` en `AudioConfig` en el constructor `SpeechSynthesizer`. 

> [!NOTE]
> Al usar `undefined` en `AudioConfig`, en lugar de omitirlo como en el ejemplo de salida del altavoz anterior, el audio no se reproducirá de forma predeterminada en el dispositivo de salida activo actual.

Esta vez se guarda el resultado en una variable [`SpeechSynthesisResult`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesisresult?view=azure-node-latest). La propiedad `SpeechSynthesisResult.audioData` devuelve un `ArrayBuffer` de los datos de salida. Puede trabajar con este `ArrayBuffer` manualmente.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const synthesizer = new sdk.SpeechSynthesizer(speechConfig);

    synthesizer.speakTextAsync(
        "Getting the response as an in-memory stream.",
        result => {
            // Interact with the audio ArrayBuffer data
            const audioData = result.audioData;
            console.log(`Audio data byte size: ${audioData.byteLength}.`)

            synthesizer.close();
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

A partir de aquí se puede implementar cualquier comportamiento personalizado mediante el objeto `ArrayBuffer` resultante.

## <a name="customize-audio-format"></a>Personalización del formato de audio

En la siguiente sección se muestra cómo personalizar los atributos de la salida de audio, entre los que se incluyen:

* Tipo de archivo de audio
* Frecuencia de muestreo
* Profundidad de bits

Para cambiar el formato de audio, se usa la propiedad `speechSynthesisOutputFormat` en el objeto `SpeechConfig`. Esta propiedad espera un elemento `enum` del tipo [`SpeechSynthesisOutputFormat`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesisoutputformat?view=azure-node-latest), que se usa para seleccionar el formato de salida. En los documentos de referencia encontrará una [lista de los formatos de audio](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesisoutputformat?view=azure-node-latest) disponibles.

Hay varias opciones para los distintos tipos de archivo, por lo que puede elegir la que necesite. Tenga en cuenta que, por definición, los formatos sin procesar, como `Raw24Khz16BitMonoPcm`, no incluyen encabezados de audio. Los formatos sin procesar solo se deben usar cuando se sepa que la implementación de bajada puede descodificar una secuencia de bits sin procesar, o bien si planea crear manualmente encabezados basados en la profundidad de bits, la frecuencia de muestreo, el número de canales, etc.

En este ejemplo, se especifica un formato RIFF de alta fidelidad `Riff24Khz16BitMonoPcm`, para lo que se establece `speechSynthesisOutputFormat` en el objeto `SpeechConfig`. De forma similar al ejemplo de la sección anterior, obtenga los datos de `ArrayBuffer` de audio e interactúe con ellos.

```javascript
function synthesizeSpeech() {
    const speechConfig = SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");

    // Set the output format
    speechConfig.speechSynthesisOutputFormat = SpeechSynthesisOutputFormat.Riff24Khz16BitMonoPcm;

    const synthesizer = new sdk.SpeechSynthesizer(speechConfig, undefined);
    synthesizer.speakTextAsync(
        "Customizing audio output format.",
        result => {
            // Interact with the audio ArrayBuffer data
            const audioData = result.audioData;
            console.log(`Audio data byte size: ${audioData.byteLength}.`)

            synthesizer.close();
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

Si vuelve a ejecutar el programa, se escribirá un archivo `.wav` en la ruta de acceso especificada.

## <a name="use-ssml-to-customize-speech-characteristics"></a>Uso de SSML para personalizar las características de la voz

El lenguaje de marcado de síntesis de voz (SSML) permite ajustar el tono, la pronunciación, la velocidad del habla, el volumen, etc. de la salida de texto a voz mediante el envío de solicitudes desde un lenguaje de definición de esquema XML. En esta sección se muestran algunos ejemplos de uso prácticos, pero se desea una guía más detallada, consulte el [artículo de procedimientos de SSML](../../../speech-synthesis-markup.md).

Para empezar a usar SSML para la personalización, realice un cambio sencillo que cambie la voz.
En primer lugar, cree un archivo XML para la configuración de SSML en el directorio raíz del proyecto, en este ejemplo `ssml.xml`. El elemento raíz es siempre `<speak>` y si se ajusta el texto en un elemento `<voice>`, se puede cambiar la voz mediante el parámetro `name`. En este ejemplo se cambia la voz a una voz masculina en inglés (Reino Unido). Tenga en cuenta que esta es una voz **estándar**, que tiene distintos precios y disponibilidad que las voces **neuronales**. Consulte la [lista completa](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#standard-voices) de voces **estándar** admitidas.

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
  <voice name="en-GB-George-Apollo">
    When you're on the motorway, it's a good idea to use a sat-nav.
  </voice>
</speak>
```

A continuación, debe cambiar la solicitud de síntesis de voz para que haga referencia al archivo XML. La solicitud es básicamente la misma, pero en lugar de usar la función `speakTextAsync()`, se usa `speakSsmlAsync()`. Esta función espera una cadena XML, por lo que primero debe crear una función para cargar un archivo XML y devolverlo como una cadena.

```javascript
function xmlToString(filePath) {
    const xml = readFileSync(filePath, "utf8");
    return xml;
}
```

Para más información sobre `readFileSync`, consulte <a href="https://nodejs.org/api/fs.html#fs_fs_readlinksync_path_options" target="_blank">Node.js file system<span class="docon docon-navigate-external x-hidden-focus"></span></a> (Sistema de archivos de Node.js). Desde aquí, el objeto del resultado es exactamente el mismo que el de los ejemplos anteriores.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const synthesizer = new sdk.SpeechSynthesizer(speechConfig, undefined);

    const ssml = xmlToString("ssml.xml");
    synthesizer.speakSsmlAsync(
        ssml,
        result => {
            if (result.errorDetails) {
                console.error(result.errorDetails);
            } else {
                console.log(JSON.stringify(result));
            }

            synthesizer.close();
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

La salida funciona, pero se pueden realizar algunos cambios sencillos más para que sea más natural. La velocidad de habla general es un algo más rápida, por lo que agregaremos una etiquete `<prosody>` y reduciremos la velocidad al **90 %** de la tasa predeterminada. Además, la pausa después de la coma de la oración es un poco corta y no suena natural. Para corregir este problema, agregue una etiqueta `<break>` para retrasar la voz y establezca el parámetro time en **200 ms**. Vuelva a ejecutar la síntesis para ver cómo han afectado estas personalizaciones a la salida.

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
  <voice name="en-GB-George-Apollo">
    <prosody rate="0.9">
      When you're on the motorway,<break time="200ms"/> it's a good idea to use a sat-nav.
    </prosody>
  </voice>
</speak>
```

## <a name="neural-voices"></a>Voces neuronales

Las voces neuronales son algoritmos de síntesis de voz con tecnología de redes neuronales profundas. Cuando se usa una voz neuronal, es prácticamente imposible distinguir la voz sintetizada de las grabaciones humanas. Con su prosodia natural similar a la humana y la clara articulación de las palabras, las voces neuronales reducen considerablemente la fatiga de la escucha que aparece cuando los usuarios interactúan con sistemas de inteligencia artificial.

Para cambiar a una voz neuronal, cambie `name` a una de las [opciones de voz neuronal](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#neural-voices). Luego, agregue un espacio de nombres XML para `mstts` y ajuste el texto en la etiqueta `<mstts:express-as>`. Use el parámetro `style` para personalizar el estilo de habla. En este ejemplo se utiliza `cheerful`, pero pruebe a establecerlo en `customerservice` o `chat` para ver la diferencia en el estilo de habla.

> [!IMPORTANT]
> Las voces neuronales **solo** se admiten para los recursos de voz creados en las regiones *Este de EE. UU.* , *Sudeste de Asia* *y Oeste de Europa*.

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
    xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
  <voice name="en-US-AriaNeural">
    <mstts:express-as style="cheerful">
      This is awesome!
    </mstts:express-as>
  </voice>
</speak>
```
