---
title: Creación de palabras clave personalizadas - Servicio de voz
titleSuffix: Azure Cognitive Services
description: El dispositivo siempre espera una palabra clave (o frase). Cuando el usuario dice la palabra clave, el dispositivo envía todo el audio subsiguiente a la nube, hasta que el usuario deja de hablar. La personalización de la palabra clave es una forma eficaz de diferenciar el dispositivo y de reforzar la marca.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/20/2019
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: d80f244f7b5e17d730451093070b971e9aa041b9
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919019"
---
# <a name="custom-keyword-basics"></a>Conceptos básicos de palabras claves personalizadas

En este artículo, aprenderá los conceptos básicos de cómo trabajar con palabras clave personalizadas mediante Speech Studio y el SDK de Voz. Una palabra clave es una palabra o frase corta que permite activar por voz el producto. Se crean modelos de palabras clave en Speech Studio y, a continuación, se exporta un archivo de modelo que se usará con el SDK de Voz en sus aplicaciones.

## <a name="prerequisites"></a>Requisitos previos

Los pasos descritos en este artículo requieren una suscripción a Voz y el SDK de Voz. Si no dispone ya de una suscripción, [pruebe el servicio de voz de forma gratuita](get-started.md). Para obtener el SDK, consulte la [guía de instalación](quickstarts/setup-platform.md) para su plataforma.

## <a name="create-a-keyword-in-speech-studio"></a>Creación de una palabra clave en Speech Studio

Antes de que pueda usar una palabra clave personalizada, deberá crear una palabra clave con la página [Custom Keyword](https://aka.ms/sdsdk-wakewordportal) (Palabra clave personalizada) en [Speech Studio](https://aka.ms/sdsdk-speechportal). Después de proporcionar una palabra clave, se genera un archivo `.table` que puede usar con el SDK de Voz.

> [!IMPORTANT]
> Los modelos de palabra clave personalizada y los archivos `.table` resultantes **solo** pueden crearse en Speech Studio.
> No se pueden crear palabras clave personalizadas a partir del SDK o mediante llamadas REST.

1. Vaya a [Speech Studio](https://aka.ms/sdsdk-speechportal) e **inicie sesión** o, si todavía no tiene una suscripción a Voz, elija [**Crear una suscripción**](https://go.microsoft.com/fwlink/?linkid=2086754).

1. En la página [Custom Keyword](https://aka.ms/sdsdk-wakewordportal) (Palabra clave personalizada), cree un **Nuevo proyecto**. 

1. Escriba un **Nombre**, una **Descripción** opcional y seleccione el idioma. Necesita un proyecto por idioma, y la compatibilidad está limitada actualmente al idioma `en-US`.

    ![Descripción del proyecto de palabra clave](media/custom-keyword/custom-kws-portal-new-project.png)

1. Seleccione el proyecto de la lista. 

    ![Selección del proyecto de palabra clave](media/custom-keyword/custom-kws-portal-project-list.png)

1. Para crear un nuevo modelo de palabra clave, haga clic en **Entrenar modelo**.

1. Escriba un **Nombre** para el modelo, una **Descripción** opcional y la **Palabra clave** de su elección y después haga clic en **Siguiente**. Consulte las [directrices](speech-devices-sdk-kws-guidelines.md#choose-an-effective-keyword) sobre cómo elegir una palabra clave eficaz.

    ![Escriba su palabra clave](media/custom-keyword/custom-kws-portal-new-model.png)

1. El portal creará pronunciaciones candidatas para la palabra clave. Para escuchar a cada candidato, haga clic en los botones de reproducción y desactive las pronunciaciones que sean incorrectas. Una vez que solo las pronunciaciones correctas estén marcadas como activas, haga clic en **Entrenar** para empezar a generar el modelo de palabra clave. 

    ![Revisión de la palabra clave](media/custom-keyword/custom-kws-portal-choose-prons.png)

1. El modelo puede tardar hasta treinta minutos en generarse. La lista de palabras clave cambiará de **Procesando** a **Correcto** cuando el modelo esté completo. Luego puede descargar el archivo.

    ![Revisión de la palabra clave](media/custom-keyword/custom-kws-portal-download-model.png)

1. El archivo descargado es un archivo `.zip`. Extraiga el archivo y verá un archivo con la extensión `.table`. Este es el archivo que usará con el SDK en la sección siguiente, por lo que debe asegurarse de anotar la ruta de acceso. El nombre del archivo refleja el nombre de la palabra clave; por ejemplo, la palabra clave **Activate device** (activa el dispositivo) tiene el nombre de archivo `Activate_device.table`.

## <a name="use-a-keyword-model-with-the-sdk"></a>Uso de un modelo de palabras clave con el SDK

En primer lugar, cargue el archivo del modelo de palabra clave mediante la función estática `FromFile()`, que devuelve un `KeywordRecognitionModel`. Use la ruta de acceso al archivo `.table` que descargó de Speech Studio. Además, debe crear una configuración `AudioConfig` con el micrófono predeterminado y, a continuación, cree una instancia nueva de `KeywordRecognizer` mediante la configuración de audio.

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

var keywordModel = KeywordRecognitionModel.FromFile("your/path/to/Activate_device.table");
using var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using var keywordRecognizer = new KeywordRecognizer(audioConfig);
```

A continuación, se ejecuta el reconocimiento de palabras clave mediante una llamada a `RecognizeOnceAsync()` en la que se pasa el objeto de modelo. De este modo, se inicia una sesión de reconocimiento de palabras clave que continúa hasta que se reconoce la palabra clave. Por lo tanto, normalmente se usa este modelo de diseño en aplicaciones multiproceso o en casos de uso en los que puede esperar una palabra de activación indefinidamente.

```csharp
KeywordRecognitionResult result = await keywordRecognizer.RecognizeOnceAsync(keywordModel);
```

> [!NOTE]
> El ejemplo que se muestra aquí usa el reconocimiento de palabras clave local, ya que no requiere un objeto `SpeechConfig` para el contexto de autenticación y no se comunica con el servidor back-end. Sin embargo, puede ejecutar el reconocimiento de palabras clave y la comprobación [mediante una conexión continua con el servidor back-end](https://docs.microsoft.com/azure/cognitive-services/speech-service/tutorial-voice-enable-your-bot-speech-sdk#view-the-source-code-that-enables-keyword).

## <a name="next-steps"></a>Pasos siguientes

Pruebe la palabra clave personalizada con la [guía de inicio rápido del SDK de dispositivos de Voz](https://aka.ms/sdsdk-quickstart).
