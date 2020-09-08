---
title: Traducción de voz con el servicio de voz
titleSuffix: Azure Cognitive Services
description: El servicio de voz le permite agregar a sus aplicaciones, herramientas y dispositivos una traducción de voz completa, de varios idiomas y en tiempo real. La misma API puede usarse para la traducción de voz a voz y de voz a texto.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: erhopf
ms.custom: devx-track-csharp
ms.openlocfilehash: 670d7f160285283bd44371b893c63904b2685926
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934504"
---
# <a name="what-is-speech-translation"></a>¿Qué es la traducción de voz?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

La traducción de voz del servicio de voz permite la traducción de voz a voz y voz a texto de secuencias de audio en varios idiomas en tiempo real. Con el SDK de voz, sus aplicaciones, herramientas y los dispositivos tienen acceso a las transcripciones de origen y a las salidas de traducción del audio proporcionadas. Se devuelven resultados provisionales de transcripción y traducción cuando se detecta la voz y los resultados finales se pueden convertir en voz sintetizada.

El motor de traducción de Microsoft usa tecnología de dos enfoques diferentes: traducción automática estadística (SMT) y traducción automática neuronal (NMT). SMT usa análisis estadísticos avanzados para estimar las mejores traducciones posibles dado el contexto de unas pocas palabras. Con NMT, las redes neuronales se utilizan para proporcionar traducciones más precisas y naturales mediante el contexto completo de las oraciones para traducir palabras.

En la actualidad, Microsoft utiliza NMT para la traducción a los idiomas más populares. Todos los [idiomas disponibles para la traducción de voz a voz](language-support.md#speech-translation) cuentan con la tecnología de NMT. La traducción de voz a texto puede utilizar SMT o NMT, según el par de idiomas. Si el idioma de destino admite NMT, la traducción completa se realiza con NMT. Si el idioma de destino no admite NMT, la traducción es un híbrido de NMT y SMT, con el idioma inglés como "enlace" entre los dos idiomas.

## <a name="core-features"></a>Características principales

Estas son las características disponibles en el SDK de voz y las API REST de Speech Services:

| Caso de uso | SDK | REST |
|----------|-----|------|
| Traducción de voz a texto con resultados de reconocimiento. | Sí | No |
| Traducción de voz a voz. | Sí | No |
| Resultados de reconocimiento y traducción provisionales. | Sí | No |

## <a name="get-started-with-speech-translation"></a>Introducción a la traducción de voz

Le ofrecemos inicios rápidos diseñados para que ejecute el código en menos de 10 minutos. Esta tabla incluye una lista de inicios rápidos de traducción de voz ordenados por idioma.

| Guía de inicio rápido | Plataforma | Referencia de API |
|------------|----------|---------------|
| [C#, .NET Core](~/articles/cognitive-services/Speech-Service/quickstarts/translate-speech-to-text.md?pivots=programming-language-csharp&tabs=dotnetcore) | Windows | [Browse](https://aka.ms/csspeech/csharpref) |
| [C#, .NET Framework](~/articles/cognitive-services/Speech-Service/quickstarts/translate-speech-to-text.md?pivots=programming-language-csharp&tabs=dotnet) | Windows | [Browse](https://aka.ms/csspeech/csharpref) |
| [C#, UWP](~/articles/cognitive-services/Speech-Service/quickstarts/translate-speech-to-text.md?pivots=programming-language-csharp&tabs=uwp) | Windows | [Browse](https://aka.ms/csspeech/csharpref) |
| [C++](~/articles/cognitive-services/Speech-Service/quickstarts/translate-speech-to-text.md?pivots=programming-language-cpp&tabs=windows) | Windows | [Browse](https://aka.ms/csspeech/cppref)|
| [Java](~/articles/cognitive-services/Speech-Service/quickstarts/translate-speech-to-text.md?pivots=programming-language-java&tabs=jre) | Windows, Linux, macOS | [Browse](https://aka.ms/csspeech/javaref) |

## <a name="sample-code"></a>Código de ejemplo

Hay un ejemplo de código para el SDK de voz disponible en GitHub. En estos ejemplos se tratan escenarios comunes como la lectura de audio de un archivo o secuencia, el reconocimiento o traducción continuos y de una sola emisión, y el trabajo con modelos personalizados.

* [Ejemplos de conversión de voz a texto y de traducción (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)

## <a name="migration-guides"></a>Guías de migración

Si sus aplicaciones, herramientas o productos usan [Translator Speech API](https://docs.microsoft.com/azure/cognitive-services/translator-speech/overview), hemos creado guías que le ayudarán a migrar al servicio de voz.

* [Migración de Translator Speech API al servicio de voz](how-to-migrate-from-translator-speech-api.md)

## <a name="reference-docs"></a>Documentos de referencia

* [Acerca del SDK de Voz](speech-sdk-reference.md)
* [Speech Devices SDK](speech-devices-sdk.md)
* [API REST: Speech-to-text](rest-speech-to-text.md) (API de REST: Voz a texto)
* [API REST: Text-to-speech](rest-text-to-speech.md) (API de REST: Texto a voz)
* [API REST: Batch transcription and customization](https://westus.cris.ai/swagger/ui/index) (API de REST: Transcripción y personalización de Azure Batch)

## <a name="next-steps"></a>Pasos siguientes

* [Obtenga una clave de suscripción gratuita a los servicios de Voz](get-started.md)
* [Obtención del SDK de voz](speech-sdk.md)
