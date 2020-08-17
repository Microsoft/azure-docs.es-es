---
author: IEvangelist
ms.service: cognitive-services
ms.topic: include
ms.date: 04/03/2020
ms.author: trbye
ms.custom: devx-track-javascript
ms.openlocfilehash: 1c998c9a4574b31427cd5fdd2a12dbd6f0dd81b1
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406335"
---
## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar:

> [!div class="checklist"]
> * <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices" target="_blank">Creación de un recurso de voz de Azure <span class="docon docon-navigate-external x-hidden-focus"></span></a>
> * [Configuración del entorno de desarrollo y creación de un proyecto vacío](../../../../quickstarts/setup-platform.md)

## <a name="start-with-some-boilerplate-code"></a>Inicio con código reutilizable

Vamos a agregar código que funcione como el esqueleto del proyecto. Cree un archivo index.js y agregue este código.

Asegúrese de rellenar los valores de `subscriptionKey`, `servcieRegion` y `filename`.

```JavaScript
(function() {
  // <code>
  "use strict";
  
  // pull in the required packages.
  var sdk = require("microsoft-cognitiveservices-speech-sdk");
  var fs = require("fs");
  
  // replace with your own subscription key,
  // service region (e.g., "westus"), and
  // the name of the file you want to run
  // through the speech recognizer.
  var subscriptionKey = "YourSubscriptionKey";
  var serviceRegion = "YourServiceRegion"; // e.g., "westus"
  var filename = "YourAudioFile.wav"; // 16000 Hz, Mono
 
}());
  
```
## <a name="load-the-file-into-an-pushaudioinputstream"></a>Carga del archivo en un elemento PushAudioInputStream

En NodeJS, el SDK de voz no admite el acceso a archivos de forma nativa directamente, por lo que se cargará el archivo y se incluirá en un elemento `PushAudioInputStream`. Esto funcionará bien con archivos más pequeños, pero con uno más grande el uso de `PullAudioInputStream` será más eficaz para la memoria.

```JavaScript
// create the push stream we need for the speech sdk.
  var pushStream = sdk.AudioInputStream.createPushStream();
  
  // open the file and push it to the push stream.
  fs.createReadStream(filename).on('data', function(arrayBuffer) {
    pushStream.write(arrayBuffer.slice());
  }).on('end', function() {
    pushStream.close();
  });
```

## <a name="create-a-speech-configuration"></a>Creación de una configuración de Voz

Antes de inicializar un objeto `SpeechRecognizer`, debe crear una configuración que use la clave y la región de suscripción. Inserte este código a continuación.

> [!NOTE]
> El SDK de Voz se usará de forma predeterminada para reconocer el uso de en-us como idioma. Para más información sobre cómo elegir el idioma de origen, consulte [Especificación del idioma de origen para la conversión de voz a texto](../../../../how-to-specify-source-language.md).

 ```JavaScript
   // now create the audio-config pointing to our stream and
  // the speech config specifying the language.
  var speechConfig = sdk.SpeechConfig.fromSubscription(subscriptionKey, serviceRegion);
  
  // setting the recognition language to English.
  speechConfig.speechRecognitionLanguage = "en-US";
  
```
## <a name="create-an-audio-configuration"></a>Creación de una configuración de audio

Ahora, debe crear un objeto `AudioConfig` que apunte al archivo `PushAudioInputStream`. Inserte este código justo debajo de la configuración de Voz.

```JavaScript
    var audioConfig = sdk.AudioConfig.fromStreamInput(pushStream);
```

## <a name="initialize-a-speechrecognizer"></a>Inicialización de un objeto SpeechRecognizer

Ahora, se creará el objeto `SpeechRecognizer` con los objetos `SpeechConfig` y `AudioConfig` creados anteriormente.

```JavaScript
  // create the speech recognizer.
  var recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);
  
```
## <a name="recognize-a-phrase-and-display-results"></a>Reconocimiento de una frase y visualización de los resultados

En el objeto `SpeechRecognizer`, va a llamar al método `recognizeOnceAsync()`. Este método permite que el servicio Voz sepa que solo va a enviar una frase para el reconocimiento y que, una vez que se identifica la frase, se detendrá el reconocimiento de voz.

También escribiremos el texto devuelto, o cualquier error, en la consola y, por último, cerraremos el reconocedor.
```JavaScript
 // we are done with the setup
  console.log("Now recognizing from: " + filename);
  
  // start the recognizer and wait for a result.
  recognizer.recognizeOnceAsync(
    function (result) {
      console.log(result);
  
      recognizer.close();
      recognizer = undefined;
    },
    function (err) {
      console.trace("err - " + err);
  
      recognizer.close();
      recognizer = undefined;
    });
```
## <a name="check-your-code"></a>Comprobación del código
```JavaScript
(function() {
  "use strict";
  
  // pull in the required packages.
  var sdk = require("microsoft-cognitiveservices-speech-sdk");
  var fs = require("fs");
  
  // replace with your own subscription key,
  // service region (e.g., "westus"), and
  // the name of the file you want to run
  // through the speech recognizer.
  var subscriptionKey = "YourSubscriptionKey";
  var serviceRegion = "YourServiceRegion"; // e.g., "westus"
  var filename = "YourAudioFile.wav"; // 16000 Hz, Mono
  
  // create the push stream we need for the speech sdk.
  var pushStream = sdk.AudioInputStream.createPushStream();
  
  // open the file and push it to the push stream.
  fs.createReadStream(filename).on('data', function(arrayBuffer) {
    pushStream.write(arrayBuffer.slice());
  }).on('end', function() {
    pushStream.close();
  });
  
  // now create the audio-config pointing to our stream and
  // the speech config specifying the language.
  var speechConfig = sdk.SpeechConfig.fromSubscription(subscriptionKey, serviceRegion);
  
  // setting the recognition language to English.
  speechConfig.speechRecognitionLanguage = "en-US";
  
  var audioConfig = sdk.AudioConfig.fromStreamInput(pushStream);
  
  // create the speech recognizer.
  var recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);
  
  // we are done with the setup
  console.log("Now recognizing from: " + filename);
  
  // start the recognizer and wait for a result.
  recognizer.recognizeOnceAsync(
    function (result) {
      console.log(result);
  
      recognizer.close();
      recognizer = undefined;
    },
    function (err) {
      console.trace("err - " + err);
  
      recognizer.close();
      recognizer = undefined;
    });

}());
```
## <a name="run-the-sample-locally"></a>Ejecución local del código

Ejecución del código mediante NodeJs
```bash
node index.js
```

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [Speech recognition basics](../../speech-to-text-next-steps.md)]
