---
title: 'Introducción a la traducción de voz: servicio Voz'
titleSuffix: Azure Cognitive Services
description: La traducción de voz le permite incorporar en sus aplicaciones, herramientas y dispositivos una traducción de voz completa, en varios idiomas y en tiempo real. La misma API puede usarse para la traducción de voz a voz y de voz a texto. En este artículo encontrará información general sobre las ventajas y las funcionalidades del servicio de traducción de voz.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: erhopf
ms.custom: devx-track-csharp, cog-serv-seo-aug-2020
keywords: traducción de voz
ms.openlocfilehash: 67bb418926932ebb7e443e77c65dd12c7352049d
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401221"
---
# <a name="what-is-speech-translation"></a>¿Qué es la traducción de voz?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

En esta introducción, descubrirá las ventajas y funcionalidades del servicio de traducción de voz, un servicio que le permite agregar a las secuencias de audio traducciones de voz a voz y de voz a texto en varios idiomas y en tiempo real. Con el SDK de voz, sus aplicaciones, herramientas y los dispositivos tienen acceso a las transcripciones de origen y a las salidas de traducción del audio proporcionadas. A medida que se detecta la voz, se van devolviendo resultados provisionales de transcripción y traducción. Asimismo, los resultados finales pueden convertirse en voz sintetizada.

El motor de traducción de Microsoft usa tecnología de dos enfoques diferentes: traducción automática estadística (SMT) y traducción automática neuronal (NMT). SMT usa análisis estadísticos avanzados para estimar las mejores traducciones posibles dado el contexto de unas pocas palabras. Con NMT, las redes neuronales se utilizan para proporcionar traducciones más precisas y naturales mediante el contexto completo de las oraciones para traducir palabras.

En la actualidad, Microsoft utiliza NMT para la traducción a los idiomas más populares. Todos los [idiomas disponibles para la traducción de voz a voz](language-support.md#speech-translation) cuentan con la tecnología de NMT. La traducción de voz a texto puede utilizar SMT o NMT, según el par de idiomas. Si el idioma de destino admite NMT, la traducción completa se realiza con NMT. Si el idioma de destino no admite NMT, la traducción es un híbrido de NMT y SMT, con el idioma inglés como "enlace" entre los dos idiomas.

## <a name="core-features"></a>Características principales

* Traducción de voz a texto con resultados de reconocimiento.
* Traducción de voz a voz.
* Compatibilidad para traducir a varios idiomas de destino.
* Resultados de reconocimiento y traducción provisionales.

## <a name="get-started"></a>Introducción 

Consulte la [guía de inicio rápido](get-started-speech-translation.md) para empezar a usar la traducción de voz. El servicio de traducción de voz está disponible con el [SDK de Voz](speech-sdk.md) y la [CLI de Voz](spx-overview.md).

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

* Complete la [guía de inicio rápido](get-started-speech-translation.md) de traducción de voz
* [Obtenga una clave de suscripción gratuita a los servicios de Voz](get-started.md)
* [Obtención del SDK de voz](speech-sdk.md)
