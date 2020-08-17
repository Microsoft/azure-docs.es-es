---
title: 'Acerca del SDK de Voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: El kit de desarrollo de software (SDK) de voz expone muchas de las funcionalidades del servicio de voz, lo que facilita el desarrollo de aplicaciones habilitadas para la voz.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/03/2020
ms.author: trbye
ms.openlocfilehash: 79f7924b021de9426eeb66adf2ec12f8033efcea
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056863"
---
# <a name="about-the-speech-sdk"></a>Acerca del SDK de Voz

El kit de desarrollo de software (SDK) de voz expone muchas de las funcionalidades del servicio de voz, lo que le permite el desarrollo de aplicaciones habilitadas para la voz. El SDK de voz está disponible en muchos lenguajes de programación y en todas las plataformas.

[!INCLUDE [Speech SDK Platforms](../../../includes/cognitive-services-speech-service-speech-sdk-platforms.md)]

## <a name="scenario-capabilities"></a>Funcionalidades del escenario

El SDK de voz expone muchas características del servicio de voz, pero no todas ellas. Las funcionalidades del SDK de voz suelen estar asociadas con escenarios. El SDK de voz es perfecto para escenarios en tiempo real y no en tiempo real, mediante dispositivos locales, archivos, almacenamiento de blobs de Azure e incluso flujos de entrada y salida. Cuando un escenario no sea factible con el SDK de voz, busque una alternativa de la API REST.

### <a name="speech-to-text"></a>Voz a texto

La [conversión de voz en texto](speech-to-text.md) (también conocida como *reconocimiento de voz*) transcribe secuencias de audio en texto en tiempo real que las aplicaciones, herramientas o dispositivos pueden usar o mostrar. Use voz a texto con [Language Understanding (LUIS)](../luis/index.yml) para derivar las intenciones del usuario a partir de voz transcrita y actuar en los comandos de voz. Utilice [Speech Translation](speech-translation.md) para traducir la entrada de voz a un idioma diferente con una sola llamada. Para más información, consulte [Aspectos básicos del reconocimiento de voz](speech-to-text-basics.md).

### <a name="text-to-speech"></a>Texto a voz

La [conversión de texto en voz](text-to-speech.md) (también conocido como *síntesis de voz*) convierte el texto en voz sintetizada similar a la voz humana. El texto de entrada está formado por literales de cadena o mediante el [lenguaje de marcado de síntesis de voz (SSML)](speech-synthesis-markup.md). Para más información sobre las voces estándar o neuronales, consulte [Compatibilidad con idiomas y voces en el servicio de voz](language-support.md#text-to-speech).

### <a name="voice-assistants"></a>Asistentes de voz

Los [asistentes de voz](voice-assistants.md) que usan el SDK de voz permiten a los desarrolladores crear interfaces de conversación naturales, similares a la humana, para sus aplicaciones y experiencias. El servicio del asistente de voz proporciona una interacción rápida y confiable entre un dispositivo y un asistente. La implementación utiliza el canal Direct Line Speech de Bot Framework o el servicio integrado de comandos personalizados (versión preliminar) para la finalización de tareas. Además, los asistentes de voz pueden usar las voces personalizadas creadas en el [Portal de voz personalizado](https://aka.ms/customvoice) para agregar una experiencia de salida de voz única.

#### <a name="keyword-spotting"></a>Detección de palabras clave

El concepto de [detección de palabras clave](speech-devices-sdk-create-kws.md) es compatible con el SDK de voz. La detección de palabras clave es el acto de identificar una palabra clave en el habla, seguido de una acción al escuchar la palabra clave. Por ejemplo, "Hola Cortana" activa el asistente Cortana.

### <a name="meeting-scenarios"></a>Escenarios de reuniones

El SDK de voz es idóneo para la transcripción de escenarios de reuniones, ya sea en una conversación en único dispositivo o en varios.

#### <a name="conversation-transcription"></a>Transcripción de conversaciones

La [transcripción de conversaciones](conversation-transcription.md) permite el reconocimiento de voz en tiempo real (y asincrónico), la identificación del hablante y la atribución de frases a cada hablante (también conocido como *diarización*). Es perfecto para transcribir reuniones en persona con la capacidad de distinguir a los oradores.

#### <a name="multi-device-conversation"></a>Conversación entre varios dispositivos

Con la [conversación entre varios dispositivos](multi-device-conversation.md), puede conectar varios dispositivos o clientes en una conversación para enviar mensajes basados en voz o texto, con compatibilidad sencilla con la transcripción y traducción.

### <a name="custom--agent-scenarios"></a>Escenarios personalizados o de agente

El SDK de voz se puede usar para transcribir escenarios de centros de llamadas, donde se generan datos de telefonía.

#### <a name="call-center-transcription"></a>Transcripción para los centros de llamadas

La [transcripción para los centros de llamadas](call-center-transcription.md) es un escenario común para la conversión de voz en texto, ya que se transcriben grandes volúmenes de datos de telefonía que pueden provenir de varios sistemas, como la respuesta interactiva de voz (IVR). Los últimos modelos de reconocimiento de voz del servicio Voz destacan en la transcripción de estos datos de telefonía, incluso en los casos en que los datos son difíciles de entender para un humano.

### <a name="codec-compressed-audio-input"></a>Entrada de audio comprimido con códec

Algunos de los lenguajes de programación del SDK de voz admiten flujos de entrada de audio comprimido con códecs. Para más información, consulte <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-use-codec-compressed-audio-input-streams" target="_blank">Uso de entradas de audio comprimido con códec con el SDK de voz<span class="docon docon-navigate-external x-hidden-focus"></span></a>.

## <a name="rest-api"></a>API DE REST

Aunque el SDK de voz cubre muchas funcionalidades de características del servicio de voz, en algunos escenarios se puede querer usar la API REST.

### <a name="batch-transcription"></a>Transcripción de Azure Batch

La [transcripción por lotes](batch-transcription.md) permite la transcripción asincrónica de voz en texto de grandes volúmenes de datos. La transcripción por lotes solo es posible desde la API REST. Además de convertir el audio de la voz en texto, la conversión de voz en texto por lotes también permite la diarización y el análisis de sentimiento.

## <a name="customization"></a>Personalización

El servicio de voz ofrece una gran funcionalidad con sus modelos predeterminados de conversión de voz en texto, de texto en voz y de traducción de voz. En ocasiones, puede que desee aumentar el rendimiento de línea de base para que funcione mejor con su caso de uso único. El servicio de voz tiene una variedad de herramientas de personalización sin código que facilitan la tarea y permiten crear una ventaja competitiva con modelos personalizados basados en sus propios datos. Estos modelos solo estarán disponibles para usted y su organización.

### <a name="custom-speech-to-text"></a>Conversión de voz a texto personalizada

Cuando se usa la conversión de voz en texto para el reconocimiento y la transcripción en un entorno único, puede crear y entrenar modelos acústicos, de lenguaje y pronunciación personalizados para dirigir el sonido ambiental o vocabulario específico del sector. La creación y administración de modelos de Habla personalizada sin código está disponible en el [portal de Habla personalizada](https://aka.ms/customspeech). Una vez publicado el modelo de Habla personalizada, el SDK de voz ya puede usarse.

### <a name="custom-text-to-speech"></a>Conversión de texto a voz personalizada

La conversión de texto en voz personalizada, también conocida como Voz personalizada, es un conjunto de herramientas en línea que permiten crear una voz única y reconocible para su marca. La creación y administración de modelos de Voz personalizada sin código está disponible en el [portal de Voz personalizada](https://aka.ms/customvoice). Una vez publicado el modelo de Voz personalizada, el SDK de voz ya puede usarse.

## <a name="get-the-speech-sdk"></a>Obtención del SDK de Voz

# <a name="windows"></a>[Windows](#tab/windows)

[!INCLUDE [Get the Speech SDK](includes/get-speech-sdk-windows.md)]

# <a name="linux"></a>[Linux](#tab/linux)

[!INCLUDE [Get the Speech SDK](includes/get-speech-sdk-linux.md)]

# <a name="ios"></a>[iOS](#tab/ios)

[!INCLUDE [Get the Speech SDK](includes/get-speech-sdk-ios.md)]

# <a name="macos"></a>[macOS](#tab/macos)

[!INCLUDE [Get the Speech SDK](includes/get-speech-sdk-macos.md)]

# <a name="android"></a>[Android](#tab/android)

[!INCLUDE [Get the Speech SDK](includes/get-speech-sdk-android.md)]

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

[!INCLUDE [Get the Node.js Speech SDK](includes/get-speech-sdk-nodejs.md)]

# <a name="browser"></a>[Browser](#tab/browser)

[!INCLUDE [Get the Browser Speech SDK](includes/get-speech-sdk-browser.md)]

---

[!INCLUDE [License notice](../../../includes/cognitive-services-speech-service-license-notice.md)]

[!INCLUDE [Sample source code](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Creación de una cuenta de Azure gratuita](https://azure.microsoft.com/free/cognitive-services/)
* [Vea cómo funciona el reconocimiento de voz en C#](quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-csharp&tabs=dotnet)
