---
author: trrwilson
ms.service: cognitive-services
ms.topic: include
ms.date: 05/25/2020
ms.author: trrwilson
ms.openlocfilehash: 99540e8518aa31a2221844aa954ff665609d8217
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87375238"
---
## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar:

> [!div class="checklist"]
> * [Ha creado un recurso de Voz de Azure](../../../../get-started.md)
> * [Ha configurado el entorno de desarrollo y ha creado un proyecto vacío](../../../../quickstarts/setup-platform.md)
> * Asegúrese de que tiene acceso a un micrófono para capturar el audio.

## <a name="setup-your-environment"></a>Configuración del entorno

Actualice el archivo go.mod con la versión más reciente del SDK agregando esta línea:
```sh
require (
    github.com/Microsoft/cognitive-services-speech-sdk-go v1.13.0
)
```

## <a name="start-with-some-boilerplate-code"></a>Inicio con código reutilizable
1. Reemplace el contenido del archivo de código fuente (por ejemplo, `sr-quickstart.go`) por el siguiente, que incluye:

- Definición del paquete "main"
- Importación de los módulos necesarios desde el SDK de voz
- Variables para almacenar la información de suscripción que se sustituirá más adelante en este inicio rápido
- Una implementación sencilla con el micrófono para la entrada de audio
- Controladores de eventos para varios eventos que tienen lugar durante el reconocimiento de voz

```sh
package recognizer

import (
    "bufio"
    "fmt"
    "os"

    "github.com/Microsoft/cognitive-services-speech-sdk-go/audio"
    "github.com/Microsoft/cognitive-services-speech-sdk-go/speech"
)

func recognizingHandler(event speech.SpeechRecognitionEventArgs) {
    defer event.Close()
    fmt.Println("Recognizing:", event.Result.Text)
}

func recognizedHandler(event speech.SpeechRecognitionEventArgs) {
    defer event.Close()
    fmt.Println("Recognized:", event.Result.Text)
}

func cancelledHandler(event speech.SpeechRecognitionCanceledEventArgs) {
    defer event.Close()
    fmt.Println("Received a cancellation: ", event.ErrorDetails)
}

func main() {
    subscription :=  "YOUR_SUBSCRIPTION_KEY"
    region := "YOUR_SUBSCRIPTIONKEY_REGION"

    audioConfig, err := audio.NewAudioConfigFromDefaultMicrophoneInput()
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer audioConfig.Close()
    config, err := speech.NewSpeechConfigFromSubscription(subscription, region)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer config.Close()
    speechRecognizer, err := speech.NewSpeechRecognizerFromConfig(config, audioConfig)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer speechRecognizer.Close()
    speechRecognizer.Recognizing(recognizingHandler)
    speechRecognizer.Recognized(recognizedHandler)
    speechRecognizer.Canceled(cancelledHandler)
    speechRecognizer.StartContinuousRecognitionAsync()
    defer speechRecognizer.StopContinuousRecognitionAsync()
    bufio.NewReader(os.Stdin).ReadBytes('\n')
}
```

Reemplace los valores `YOUR_SUBSCRIPTION_KEY` y `YOUR_SUBSCRIPTIONKEY_REGION` por los valores reales del recurso de voz.

- Vaya a Azure Portal y abra el recurso Voz.
- En **Claves**, situada a la izquierda, hay dos claves de suscripción disponibles.
    - Use cualquiera de ellas como reemplazo del valor `YOUR_SUBSCRIPTION_KEY`
- En **Información general**, a la izquierda, anote la región y asígnela al identificador de región.
- Use el identificador de región como reemplazo del valor `YOUR_SUBSCRIPTIONKEY_REGION`. Por ejemplo, `"westus"` para **Oeste de EE. UU**.

## <a name="code-explanation"></a>Explicación del código
Se necesita la clave y la región de la suscripción de Voz para crear un objeto de configuración de voz. El objeto de configuración es necesario para crear una instancia de un objeto del reconocedor de voz.

La instancia del reconocedor ofrece varias formas de reconocer la voz. En este ejemplo, la voz se reconoce continuamente. Esta funcionalidad permite que el servicio de voz sepa que está enviando muchas frases para su reconocimiento y cuándo termina el programa para detener el reconocimiento de voz. A medida que se producen los resultados, el código los escribirá en la consola.

## <a name="build-and-run"></a>Compilación y ejecución
Ya está preparado para compilar el proyecto y probar el reconocimiento de voz mediante el servicio de voz.
1. Compile el proyecto, por ejemplo, **"vaya a Compilar"** .
2. Ejecute el módulo y diga una frase por el micrófono del dispositivo. Lo que diga se transmite al servicio de voz y se transcribe a texto, que aparece en la salida.


> [!NOTE]
> El SDK de Voz se usará de forma predeterminada para reconocer el uso de en-us como idioma. Para más información sobre cómo elegir el idioma de origen, consulte [Especificación del idioma de origen para la conversión de voz a texto](../../../../how-to-specify-source-language.md).


## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [footer](./footer.md)]
