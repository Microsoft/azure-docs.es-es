---
title: 'Introducción a la tecnología de texto a voz: el servicio Voz'
titleSuffix: Azure Cognitive Services
description: La característica de texto a voz del servicio de voz permite que sus aplicaciones, herramientas o dispositivos conviertan el texto en una voz sintetizada natural similar a la humana. En este artículo, encontrará información general sobre las ventajas y las funcionalidades del servicio de conversión de texto a voz.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: trbye
ms.custom: cog-serv-seo-aug-2020
keywords: texto a voz
ms.openlocfilehash: 8b8430135ffd15e8331b654628236a315f10cce3
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401187"
---
# <a name="what-is-text-to-speech"></a>¿Qué es el texto a voz?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

En esta introducción, encontrará información sobre las ventajas y las funcionalidades del servicio de conversión de texto a voz. Con este servicio, sus aplicaciones, herramientas o dispositivos podrán convertir un texto en voz sintetizada similar a la de los humanos. Puede elegir entre voces estándar y neuronales, o puede crear una voz personalizada única para su producto o marca. Tiene más de 75 voces estándar disponibles en más de 45 idiomas y configuraciones regionales, y 5 voces neuronales que están disponibles en varios idiomas y configuraciones regionales. Para obtener una lista completa de las voces, los idiomas y las configuraciones regionales compatibles, consulte [Idiomas admitidos](language-support.md#text-to-speech).

> [!NOTE]
> Bing Speech se ha retirado el 15 de octubre de 2019. Si sus aplicaciones, herramientas o productos usan Bing Speech API o Habla personalizada, hemos creado guías para que le ayuden a migrar al servicio de voz.
> - [Migración de Bing Speech al servicio de voz](how-to-migrate-from-bing-speech.md)

## <a name="core-features"></a>Características principales

* Síntesis de voz: use el [SDK de voz](quickstarts/text-to-speech-audio-file.md) o la [API de REST](rest-text-to-speech.md) para convertir texto a voz mediante las voces estándar, neuronal o personalizada.

* Síntesis asincrónica de audio de larga duración: use [Long Audio API](long-audio-api.md) para sintetizar asincrónicamente archivos de texto a voz de más de 10 minutos (por ejemplo, audiolibros o audioconferencias). A diferencia de la síntesis realizada mediante el SDK de voz o la API de REST de voz a texto, las respuestas no se devuelven en tiempo real. La expectativa es que las solicitudes se envíen de forma asincrónica, se sondeen las respuestas y el audio sintetizado se descargue cuando esté disponible en el servicio. Solo se admiten las voces neuronales personalizadas.

* Voces estándar: se crean mediante técnicas de síntesis paramétrica estadística y de síntesis de concatenación. Estas voces son realmente inteligibles y suenan muy naturales. Puede habilitar fácilmente sus aplicaciones para que hablen en más de 45 idiomas, con una amplia gama de opciones de voz. Estas voces proporcionan una alta precisión de pronunciación, admiten abreviaturas, expanden acrónimos, interpretan la fecha y la hora, son polifónicas y ofrecen muchas cosas más. Hay una lista completa de voces estándar en [Idiomas admitidos](language-support.md#text-to-speech).

* Voces neuronales: las redes neuronales profundas se usan para superar los límites de la síntesis de voz tradicional con respecto al acento y la entonación del lenguaje hablado. La predicción de la prosodia y la síntesis de voz se realizan simultáneamente, lo que resulta en una voz más fluida y natural. Las voces neuronales se pueden usar para que las interacciones con los bots de chat y los asistentes de voz sean más naturales y atractivas, para convertir textos digitales (por ejemplo, los libros electrónicos) en audiolibros y para mejorar los sistemas de navegación de los automóviles. Gracias a su prosodia natural similar a la humana y a la clara articulación de las palabras, las voces neuronales reducen considerablemente la fatiga al escucharlas que suele aparecer cuando los usuarios interactúan con sistemas de inteligencia artificial. Hay una lista completa de voces neuronales en [Idiomas admitidos](language-support.md#text-to-speech).

* Lenguaje de marcado de síntesis de voz (SSML): lenguaje de marcado basado en XML que se usa para personalizar las salidas de voz a texto. Con SSML, puede ajustar el tono, agregar pausas, mejorar la pronunciación, acelerar o ralentizar la velocidad del habla, subir o bajar el volumen y atribuir varias voces a un solo documento. Consulte [SSML](speech-synthesis-markup.md).

## <a name="get-started"></a>Introducción

Consulte el [inicio rápido](get-started-text-to-speech.md) para empezar a usar texto a voz. El servicio de texto a voz está disponible con el [SDK de voz](speech-sdk.md), la [API REST](rest-text-to-speech.md) y la [CLI de voz](spx-overview.md).

## <a name="sample-code"></a>Código de ejemplo

El ejemplo de código para texto a voz está disponible en GitHub. Estos ejemplos tratan la conversión de texto a voz en los lenguajes de programación más populares.

- [Ejemplos de conversión de texto a voz (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
- [Ejemplos de texto a voz (REST)](https://github.com/Azure-Samples/Cognitive-Speech-TTS)

## <a name="customization"></a>Personalización

Además de las voces estándar y neuronales, puede crear y ajustar las voces personalizadas exclusivas del producto o la marca. Todo lo que se necesita para empezar son unos cuantos archivos de audio y las transcripciones asociadas. Para obtener información, consulte [Introducción a voz personalizada](how-to-custom-voice.md).

## <a name="pricing-note"></a>Nota de precios

Al usar el servicio de texto a voz, se le facturará por cada carácter que se convierte a voz, incluida la puntuación. Si bien el documento SSML en sí no es facturable, los elementos opcionales que se usan para ajustar el modo de convertir el texto a voz, como los fonemas y el tono, se cuentan como caracteres facturables. Aquí tiene una lista de lo que se puede facturar:

- El texto que se ha pasado al servicio de texto a voz en el cuerpo SSML de la solicitud.
- Todas las marcas en el campo de texto del cuerpo de la solicitud que están en formato SSML, excepto las etiquetas `<speak>` y `<voice>`.
- Letras, puntuación, espacios, tabulaciones, marcas y todos los caracteres de espacios en blanco.
- Cada punto de código que se define en Unicode

Para obtener más información, consulte [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

> [!IMPORTANT]
> Cada carácter en chino, japonés y coreano se cuenta como dos caracteres para la facturación.

## <a name="reference-docs"></a>Documentos de referencia

- [Acerca del SDK de Voz](speech-sdk.md)
- [API REST: Text-to-speech](rest-text-to-speech.md) (API de REST: Texto a voz)

## <a name="next-steps"></a>Pasos siguientes

- [Obtención de una suscripción de gratuita al servicio de voz](get-started.md)
- [Obtención del SDK de voz](speech-sdk.md)
