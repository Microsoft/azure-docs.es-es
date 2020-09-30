---
title: 'Introducción a la conversión de voz en texto: el servicio Voz'
titleSuffix: Azure Cognitive Services
description: El software de conversión de voz en texto permite la transcripción en tiempo real de secuencias de audio en texto. Las aplicaciones, las herramientas o los dispositivos pueden consumir y mostrar esta entrada de texto, así como manipularla. En este artículo, encontrará información general sobre las ventajas y las funcionalidades del servicio de conversión de voz en texto.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: trbye
ms.custom: cog-serv-seo-aug-2020
keywords: conversión de voz a texto, software de conversión de voz en texto
ms.openlocfilehash: 42696728427c34d5000881af8bbbf97ae2313111
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401272"
---
# <a name="what-is-speech-to-text"></a>¿Qué es la conversión de voz a texto?

En esta introducción, descubrirá las ventajas y las funcionalidades del servicio de conversión de voz en texto.
La conversión de voz en texto, que también se conoce como "reconocimiento de voz", permite transcribir secuencias de audio como texto en tiempo real. Las aplicaciones, las herramientas o los dispositivos pueden consumir y mostrar este texto como una entrada de comando, así como manipularlo. Este servicio funciona con la misma tecnología de reconocimiento que Microsoft utiliza para los productos de Cortana y Office. Funciona sin problemas con las ofertas de servicio de <a href="./speech-translation.md" target="_blank">traducción<span class="docon docon-navigate-external x-hidden-focus"></span></a> y <a href="./text-to-speech.md" target="_blank">conversión de texto en voz<span class="docon docon-navigate-external x-hidden-focus"></span></a>. Si desea obtener una lista completa de los idiomas disponibles para la conversión de voz a texto, consulte [Idiomas admitidos](language-support.md#speech-to-text).

De forma predeterminada, el servicio de conversión de voz en texto utiliza el modelo de lenguaje universal. Este modelo se entrenó con datos propiedad de Microsoft y se implementa en la nube. Resulta óptimo para escenarios de conversación y dictado. Si usa la conversión de voz en texto para el reconocimiento y la transcripción en un entorno único, puede crear y entrenar modelos acústicos, de lenguaje y pronunciación personalizados. La personalización es útil para abordar el ruido ambiente o el vocabulario específico del sector.

Con texto de referencia adicional como entrada, el servicio de conversión de voz a texto también habilita la funcionalidad de [evaluación de la pronunciación](rest-speech-to-text.md#pronunciation-assessment-parameters) para evaluar la pronunciación de la voz y proporcionar a los hablantes información sobre la precisión y la fluidez del audio hablado. Con la evaluación de la pronunciación, los estudiantes de idiomas pueden practicar, obtener comentarios instantáneos y mejorar su pronunciación para poder hablar y realizar presentaciones con confianza. Los educadores pueden utilizar la funcionalidad para evaluar la pronunciación de varios hablantes en tiempo real. Actualmente, la característica es compatible con el inglés de Estados Unidos y guarda una alta relación con las evaluaciones de voz realizadas por expertos.

> [!NOTE]
> Bing Speech se ha retirado el 15 de octubre de 2019. Si sus aplicaciones, herramientas o productos usan Bing Speech API, hemos creado guías para que le ayuden a migrar al servicio de voz.
> - [Migración de Bing Speech al servicio de voz](how-to-migrate-from-bing-speech.md)

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

## <a name="get-started-with-speech-to-text"></a>Introducción a la conversión de voz a texto

El servicio de conversión de voz a texto está disponible a través del [SDK de voz](speech-sdk.md). Existen varios escenarios comunes disponibles como inicios rápidos, en diferentes lenguajes y plataformas:

 - [Inicio rápido: Reconocimiento de voz con entrada de micrófono](quickstarts/speech-to-text-from-microphone.md)
 - [Inicio rápido: Reconocimiento de voz a partir de un archivo](quickstarts/speech-to-text-from-file.md)
 - [Inicio rápido: Reconocimiento de voz almacenada en Blob Storage](quickstarts/from-blob.md)

Si prefiere usar el servicio REST de voz a texto, consulte [API REST](rest-speech-to-text.md).

 - [Inicio rápido: Evaluación de la pronunciación con entrada de referencias](rest-speech-to-text.md#pronunciation-assessment-parameters)

## <a name="tutorials-and-sample-code"></a>Tutoriales y ejemplo de código

Una vez que haya tenido la oportunidad de usar el servicio de voz, pruebe nuestro tutorial, que le enseña a reconocer intenciones a partir de contenido de voz mediante el SDK de voz y LUIS.

- [Tutorial: Reconocimiento de intenciones a partir de la voz con el SDK de Voz y LUIS con C#](how-to-recognize-intents-from-speech-csharp.md)

Hay un ejemplo de código para el SDK de voz disponible en GitHub. En estos ejemplos se tratan escenarios comunes como la lectura de audio de un archivo o flujo, el reconocimiento continuo y de una sola emisión, y el trabajo con modelos personalizados.

- [Ejemplos de conversión de voz a texto (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
- [Ejemplos de transcripción de Azure Batch (REST)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/batch)
- [Ejemplos de evaluación de pronunciación (REST)](rest-speech-to-text.md#pronunciation-assessment-parameters)

## <a name="customization"></a>Personalización

Además del modelo de servicio de voz estándar, puede crear modelos personalizados. La personalización ayuda a eliminar las barreras del reconocimiento de voz, como el estilo de habla, el vocabulario y el ruido de fondo. Consulte [Habla personalizada](how-to-custom-speech.md). Las opciones de personalización varían según el idioma o la configuración regional (consulte los [idiomas admitidos](supported-languages.md) para comprobar la compatibilidad).

## <a name="batch-transcription"></a>Transcripción de Azure Batch

La transcripción por lotes es un conjunto de operaciones de API REST que permite transcribir una gran cantidad de audio en almacenamiento. Puede apuntar a archivos de audio con un identificador URI de firma de acceso compartido (SAS) y recibir los resultados de las transcripciones de forma asincrónica. Para más información sobre cómo usar la API de transcripción por lotes, consulte el [procedimiento](batch-transcription.md).

[!INCLUDE [speech-reference-doc-links](includes/speech-reference-doc-links.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Obtenga una clave de suscripción gratuita a los servicios de Voz](get-started.md)
- [Obtención del SDK de voz](speech-sdk.md)
