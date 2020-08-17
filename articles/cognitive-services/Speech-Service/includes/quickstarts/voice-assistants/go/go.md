---
author: trrwilson
ms.service: cognitive-services
ms.topic: include
ms.date: 05/25/2020
ms.author: trrwilson
ms.openlocfilehash: ee6b6c29c89c39c72144371af8268760da843170
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87375057"
---
## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar:

> [!div class="checklist"]
> * [Ha creado un recurso de Voz de Azure](../../../../get-started.md)
> * [Configuración del entorno de desarrollo y creación de un proyecto vacío](../../../../quickstarts/setup-platform.md)
> * Ha creado un bot conectado al [canal Direct Line Speech](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-directlinespeech)
> * Asegúrese de que tiene acceso a un micrófono para capturar el audio.
>
  > [!NOTE]
  > Consulte [la lista de regiones admitidas para los asistentes de voz ](~/articles/cognitive-services/speech-service/regions.md#voice-assistants) y asegúrese de que sus recursos se implementan en una de esas regiones.

## <a name="setup-your-environment"></a>Configuración del entorno

Actualice el archivo go.mod con la versión más reciente del SDK agregando esta línea:
```sh
require (
    github.com/Microsoft/cognitive-services-speech-sdk-go v1.13.0
)
```

## <a name="start-with-some-boilerplate-code"></a>Inicio con código reutilizable
Reemplace el contenido del archivo de código fuente (por ejemplo, `quickstart.go`) por el siguiente, que incluye:

- Definición del paquete "main"
- Importación de los módulos necesarios desde el SDK de voz
- Variables para almacenar la información de bots que se sustituirá más adelante en este inicio rápido
- Una implementación sencilla con el micrófono para la entrada de audio
- Controladores de eventos para varios eventos que tienen lugar durante una interacción de voz

```sh
package main

import (
    "fmt"
    "time"

    "github.com/Microsoft/cognitive-services-speech-sdk-go/audio"
    "github.com/Microsoft/cognitive-services-speech-sdk-go/dialog"
    "github.com/Microsoft/cognitive-services-speech-sdk-go/speech"
)

func main() {
    subscription :=  "YOUR_SUBSCRIPTION_KEY"
    region := "YOUR_BOT_REGION"

    audioConfig, err := audio.NewAudioConfigFromDefaultMicrophoneInput()
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer audioConfig.Close()
    config, err := dialog.NewBotFrameworkConfigFromSubscription(subscription, region)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer config.Close()
    connector, err := dialog.NewDialogServiceConnectorFromConfig(config, audioConfig)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer connector.Close()
    activityReceivedHandler := func(event dialog.ActivityReceivedEventArgs) {
        defer event.Close()
        fmt.Println("Received an activity.")
    }
    connector.ActivityReceived(activityReceivedHandler)
    recognizedHandle := func(event speech.SpeechRecognitionEventArgs) {
        defer event.Close()
        fmt.Println("Recognized ", event.Result.Text)
    }
    connector.Recognized(recognizedHandle)
    recognizingHandler := func(event speech.SpeechRecognitionEventArgs) {
        defer event.Close()
        fmt.Println("Recognizing ", event.Result.Text)
    }
    connector.Recognizing(recognizingHandler)
    connector.ListenOnceAsync()
    <-time.After(10 * time.Second)
}
```

Reemplace los valores `YOUR_SUBSCRIPTION_KEY` y `YOUR_BOT_REGION` por los valores reales del recurso de voz.

- Vaya a Azure Portal y abra el recurso Voz.
- En **Keys and Endpoint** (Claves y puntos de conexión), a la izquierda, hay dos claves de suscripción disponibles.
    - Use cualquiera de ellas como reemplazo del valor `YOUR_SUBSCRIPTION_KEY`
- En **Información general**, a la izquierda, anote la región y asígnela al identificador de región.
    - Use el identificador de región como reemplazo del valor `YOUR_BOT_REGION`. Por ejemplo, `"westus"` para **Oeste de EE. UU**.

   > [!NOTE]
   > Consulte [la lista de regiones admitidas para los asistentes de voz ](~/articles/cognitive-services/speech-service/regions.md#voice-assistants) y asegúrese de que sus recursos se implementan en una de esas regiones.

   > [!NOTE]
   > Para obtener información sobre la configuración de un bot, consulte la documentación de Bot Framework para el [canal Direct Line Speech](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-directlinespeech).

## <a name="code-explanation"></a>Explicación del código
Se necesita la clave y la región de la suscripción de Voz para crear un objeto de configuración de voz. El objeto de configuración es necesario para crear una instancia de un objeto del reconocedor de voz.

La instancia del reconocedor ofrece varias formas de reconocer la voz. En este ejemplo, la voz se reconoce continuamente. Esta funcionalidad permite que el servicio de voz sepa que está enviando muchas frases para su reconocimiento y cuándo termina el programa para detener el reconocimiento de voz. A medida que se producen los resultados, el código los escribirá en la consola.

## <a name="build-and-run"></a>Compilación y ejecución
Ya está listo para compilar el proyecto y probar el asistente de voz personalizado mediante el servicio de voz.
1. Compile el proyecto, por ejemplo, **"vaya a Compilar"** .
2. Ejecute el módulo y diga una frase por el micrófono del dispositivo. Lo que diga se transmitirá al canal Direct Line Speech y se transcribirá a texto, que aparece como salida.


> [!NOTE]
> El SDK de Voz se usará de forma predeterminada para reconocer el uso de en-us como idioma. Para más información sobre cómo elegir el idioma de origen, consulte [Especificación del idioma de origen para la conversión de voz a texto](../../../../how-to-specify-source-language.md).

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [footer](./footer.md)]
