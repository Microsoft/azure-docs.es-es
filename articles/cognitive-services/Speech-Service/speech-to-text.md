---
title: Voz a texto con servicios de voz de Azure
titleSuffix: Azure Cognitive Services
description: Voz a texto de los servicios de voz de Azure, también conocido como voz a texto, habilita la transcripción en tiempo real de secuencias de audio en texto que pueden consumir las aplicaciones, herramientas o dispositivos, mostrar y actuar en como entrada de comando. Este servicio funciona con la misma tecnología de reconocimiento que Microsoft utiliza para productos de Cortana y Office y funciona sin problemas con la traducción y el texto a voz.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/13/2019
ms.author: erhopf
ms.custom: seodec18
ms.openlocfilehash: 7596670e794c090b04f81cf6b235a4bc54c1f3c4
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65800100"
---
# <a name="what-is-speech-to-text"></a>¿Qué es el texto a voz?

Voz a texto de los servicios de voz de Azure, también conocido como voz a texto, habilita la transcripción en tiempo real de secuencias de audio en texto que pueden consumir las aplicaciones, herramientas o dispositivos, mostrar y actuar en como entrada de comando. Este servicio funciona con la misma tecnología de reconocimiento que Microsoft utiliza para productos de Cortana y Office y funciona sin problemas con la traducción y el texto a voz.  Para obtener una lista completa de idiomas de voz a texto disponibles, consulte [idiomas admitidos](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#speech-to-text).

De forma predeterminada, el servicio de voz a texto utiliza el modelo de lenguaje Universal. Este modelo se entrena con datos de la propiedad de Microsoft y está implementado en la nube. Resulta óptimo para la conversación y escenarios de dictado. Si usa voz a texto para el reconocimiento y la transcripción en un entorno único, puede crear y entrenar modelos acústicos, de lenguaje y pronunciación personalizados para dirigir el sonido ambiental o vocabulario específico del sector. 

Puede capturar audio desde un micrófono fácilmente, leer de una secuencia o tener acceso a archivos de audio desde el almacenamiento con la Speech SDK y API de REST. Speech SDK admite WAV/PCM 16 bits, 8 kHz y 16 kHz, audio de canal único para el reconocimiento de voz. Se admiten los formatos de audio adicionales mediante el [punto de conexión REST de voz a texto](https://docs.microsoft.com/azure/cognitive-services/speech-service/rest-apis) o [transcripción del servicio batch](https://docs.microsoft.com/azure/cognitive-services/speech-service/batch-transcription#supported-formats).

## <a name="core-features"></a>Características principales

Estas son las características disponibles a través de las API de REST y Speech SDK:

| Caso de uso | SDK | REST |
|----------|-----|------|
| Transcribir grabaciones de voz cortos (< 15 segundos). Solo es compatible con el resultado final de transcripción. | Sí | Sí |
| Transcripción continua de grabaciones de voz de larga y secuencias de audio (> 15 segundos). Es compatible con los resultados intermedios y finales de transcripción. | Sí | No |
| Derivar las intenciones de los resultados del reconocimiento con [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/what-is-luis). | Sí | No\* |
| Procesar por lotes de transcripción de los archivos de audio de forma asincrónica. | No | Sí\** |
| Crear y administrar modelos de voz. | No | Sí\** |
| Crear y administrar implementaciones de modelos personalizados. | No | Sí\** |
| Crear pruebas de precisión para medir la exactitud del modelo de línea de base frente a modelos personalizados. | No | Sí\** |
| Administrar suscripciones. | No | Sí\** |

\* *Las entidades e intenciones de LUIS pueden derivarse mediante una suscripción independiente de LUIS. Con esta suscripción, el SDK puede llamar a LUIS para usted y proporcionar entidad y los resultados de la intención. Con la API REST, puede llamar a LUIS usted mismo para deducir las entidades y las intenciones con su suscripción a LUIS.*

\** *Estos servicios están disponibles mediante el punto de conexión cris.ai. Consulte [Swagger referencia](https://westus.cris.ai/swagger/ui/index).*

## <a name="get-started-with-speech-to-text"></a>Empezar a trabajar con texto a voz

Ofrecemos guías de inicio rápido en los lenguajes de programación más populares, cuyo diseño individual le permite ejecutar código en menos de 10 minutos. Esta tabla incluye una lista completa de los inicios rápidos de Speech SDK organizadas por lenguaje.

| Guía de inicio rápido | Plataforma | Referencia de la API |
|------------|----------|---------------|
| [C#, .NET core](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstart-csharp-dotnetcore-windows) |  Windows | [Browse](https://aka.ms/csspeech/csharpref) |
| [C#, .NET framework](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstart-csharp-dotnet-windows) |  Windows | [Browse](https://aka.ms/csspeech/csharpref) |
| [C#, UWP](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstart-csharp-uwp) |  Windows | [Browse](https://aka.ms/csspeech/csharpref) |
| [C++](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstart-cpp-windows) |  Windows | [Browse](https://aka.ms/csspeech/cppref)|
| [C++](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstart-cpp-linux) | Linux | [Browse](https://aka.ms/csspeech/cppref) |
| [Java](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstart-java-android) | Android | [Browse](https://aka.ms/csspeech/javaref) |
| [Java](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstart-java-jre) | Windows, Linux | [Browse](https://aka.ms/csspeech/javaref) |
| [JavaScript, explorador](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstart-js-browser) | Browser, Windows, Linux, macOS | [Browse](https://aka.ms/AA434tv) |
| [JavaScript, Node.js](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstart-js-node) | Windows, Linux, macOS | [Browse](https://aka.ms/AA434tv) |
| [Objective-C](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstart-objectivec-ios) | iOS | [Browse](https://aka.ms/csspeech/objectivecref) |
| [Python](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstart-python) | Windows, Linux, macOS | [Browse](https://aka.ms/AA434tr)  |

Si prefiere usar el servicio REST de voz a texto, consulte [las API de REST](https://docs.microsoft.com/azure/cognitive-services/speech-service/rest-apis).

## <a name="tutorials-and-sample-code"></a>Tutoriales y código de ejemplo

Una vez que haya tenido la oportunidad de usar los servicios de voz, pruebe nuestro tutorial, que le enseña a reconocer intenciones a partir de contenido de voz mediante el SDK de voz y LUIS.

* [Tutorial: Reconocimiento de intenciones a partir de contenido de voz con el SDK de voz y LUIS, C#](how-to-recognize-intents-from-speech-csharp.md)

Código de ejemplo de Speech SDK está disponible en GitHub. En estos ejemplos se tratan escenarios comunes como la lectura de audio de un archivo o flujo, el reconocimiento continuo y de una sola emisión, y el trabajo con modelos personalizados.

* [Ejemplos de voz a texto (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
* [Ejemplos de transcripción de Azure Batch (REST)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/batch)

## <a name="customization"></a>Personalización

Además del modelo Universal usado por los servicios de voz, se pueden crear modelos acústicos, el idioma y la pronunciación personalizados específicos a su experiencia. Esta es una lista de opciones de personalización:

| Modelo | DESCRIPCIÓN |
|-------|-------------|
| [Modelo acústico](how-to-customize-acoustic-models.md) | Creación de un modelo acústico personalizado resulta útil si su aplicación, herramientas o los dispositivos se usan en un entorno determinado, como en un automóvil o una fábrica con condiciones específicas de grabación. Los ejemplos incluyen el habla con acento, ruidos de fondo específicos o el uso de un micrófono específico para la grabación. |
| [Modelo de lenguaje](how-to-customize-language-model.md) | Cree un modelo de lenguaje personalizado para mejorar la transcripción de gramática y vocabulario específicos del sector, como terminología médica o jerga de TI. |
| [Modelo de pronunciación](how-to-customize-pronunciation.md) | Con un modelo de pronunciación personalizado, puede definir el formato fonético y mostrar una palabra o un término. Es útil para controlar términos personalizados, como nombres de producto o acrónimos. Basta con un archivo de pronunciación (un archivo .txt simple). |

> [!NOTE]
> Opciones de personalización varían según el idioma o configuración regional (vea [idiomas admitidos](supported-languages.md)).

## <a name="migration-guides"></a>Guías de migración

> [!WARNING]
> Bing Speech se retirará el 15 de octubre de 2019.

Si sus aplicaciones, herramientas o productos usa voz personalizada o Bing Speech API, hemos creado las guías para ayudarle a migrar a servicios de voz.

* [Migrar de Bing Speech a los servicios de voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-migrate-from-bing-speech)
* [Migrar de voz personalizado a los servicios de voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-migrate-from-custom-speech-service)

## <a name="reference-docs"></a>Documentos de referencia

* [Speech SDK](speech-sdk-reference.md)
* [Speech Devices SDK](speech-devices-sdk.md)
* [API REST: Speech-to-text](rest-speech-to-text.md) (API de REST: Voz a texto)
* [API REST: Text-to-speech](rest-text-to-speech.md) (API de REST: Texto a voz)
* [API REST: Batch transcription and customization](https://westus.cris.ai/swagger/ui/index) (API de REST: Transcripción y personalización de Azure Batch)

## <a name="next-steps"></a>Pasos siguientes

* [Get a Speech Services subscription key for free](get-started.md) (Consiga una clave de suscripción a los servicios de voz gratis)
* [Obtención de Speech SDK](speech-sdk.md)
