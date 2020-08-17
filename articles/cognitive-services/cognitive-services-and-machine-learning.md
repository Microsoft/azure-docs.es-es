---
title: Cognitive Services y Machine Learning
titleSuffix: Azure Cognitive Services
description: Sepa cómo emplear Azure Cognitive Services con otras ofertas de Azure para el aprendizaje automático.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 08/22/2019
ms.openlocfilehash: 0b034d0135c66852b07e0a2ed2fd69e8c3fdc4a6
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132847"
---
# <a name="cognitive-services-and-machine-learning"></a>Cognitive Services y Machine Learning

Cognitive Services proporciona funciones de aprendizaje automático para solucionar problemas generales, como el análisis de texto para la opinión emocional o el análisis de imágenes para reconocer objetos o caras. No es necesario tener conocimientos de aprendizaje automático ni ciencia de datos para usar estos servicios. 

[Cognitive Services](welcome.md) es un grupo de servicios, cada uno de los cuales admite diferentes funcionalidades de predicción generalizadas. Los servicios se dividen en distintas categorías para ayudarle a encontrar el servicio adecuado. 

|Categoría del servicio|Propósito|
|--|--|
|[Decisión](https://azure.microsoft.com/services/cognitive-services/directory/decision/)|Cree aplicaciones que presenten recomendaciones para tomar decisiones informadas y eficaces.|
|[Lenguaje](https://azure.microsoft.com/services/cognitive-services/directory/lang/)|Permita que sus aplicaciones procesen lenguaje natural con scripts precompilados, evalúen sentimientos y aprendan a reconocer lo que los usuarios quieren.|
|[Búsqueda](https://azure.microsoft.com/services/cognitive-services/directory/search/)|Agregue Bing Search API a las aplicaciones e implemente la funcionalidad que permite combinar miles de millones de páginas web, imágenes, vídeos y noticias con una sola llamada API.|
|[Voz](https://azure.microsoft.com/services/cognitive-services/directory/speech/)|Convierta voz en texto y texto en voz que suene natural. Traduzca de un idioma a otro y habilite el reconocimiento y la verificación del hablante.|
|[Visión](https://azure.microsoft.com/services/cognitive-services/directory/vision/)|Reconozca, identifique, ponga título, indexe y modere sus imágenes, vídeos y contenido de lápiz digital.|
||||

Use Cognitive Services cuando:

* Puede usar una solución generalizada.
* Obtiene acceso a la solución desde un SDK o una API de REST de programación. 

Use otra solución de aprendizaje automático cuando:

* Tiene que elegir el algoritmo y necesita entrenar datos muy específicos.

## <a name="what-is-machine-learning"></a>¿Qué es el aprendizaje automático?

El aprendizaje automático es un concepto en el que reúne los datos y un algoritmo para resolver una necesidad concreta. Una vez que los datos y el algoritmo están entrenados, la salida es un modelo que puede volver a usar con datos diferentes. El modelo entrenado proporciona información en función de los nuevos datos. 

El proceso de creación de un sistema de aprendizaje automático requiere cierto conocimiento de aprendizaje automático o ciencia de datos.

El aprendizaje automático se proporciona con [productos y servicios de Azure Machine Learning (AML)](https://docs.microsoft.com/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning?context=azure/machine-learning/studio/context/ml-context).

## <a name="what-is-a-cognitive-service"></a>¿Qué es un servicio de Cognitive Services?

Un servicio de Cognitive Service proporciona una parte o todos los componentes de una solución de aprendizaje automático: datos, algoritmos y modelos entrenados. Estos servicios están diseñados para requerir conocimientos generales sobre los datos sin necesidad de tener experiencia con el aprendizaje automático o la ciencia de datos. Estos servicios proporcionan las API de REST y los SDK basados en lenguaje. Como resultado, debe tener conocimientos de lenguaje de programación para usar los servicios.

## <a name="how-are-cognitive-services-and-azure-machine-learning-aml-similar"></a>¿En qué se parecen Cognitive Services y Azure Machine Learning (AML)?

Ambos tienen el objetivo final de aplicar inteligencia artificial (IA) para mejorar las operaciones empresariales, aunque el modo en que cada uno lo proporciona en las ofertas correspondientes es diferente. 

Por lo general, las audiencias son diferentes:

* Cognitive Services es para desarrolladores sin experiencia de aprendizaje automático.
* Azure Machine Learning se adapta a los científicos de datos. 

## <a name="how-is-a-cognitive-service-different-from-machine-learning"></a>¿En qué se diferencia un servicio de Cognitive Services del aprendizaje automático?

Un servicio de Cognitive Service proporciona un modelo entrenado. Une los datos y un algoritmo y está disponible desde un SDK o una API de REST. Puede implementar este servicio en cuestión de minutos, en función de su escenario.  Un servicio de Cognitive Services proporciona respuestas a problemas generales, como frases clave en texto o identificación de elementos en imágenes. 

El aprendizaje automático es un proceso que generalmente requiere un período de tiempo más largo para implementarse correctamente. Este tiempo se emplea en la recopilación de datos, la limpieza, la transformación, la selección de algoritmos, el entrenamiento de modelos y la implementación para llegar al mismo nivel de funcionalidad proporcionado por un servicio de Cognitive Services. Con el aprendizaje automático, es posible proporcionar respuestas a problemas muy especializados o específicos. Para los problemas de aprendizaje automático, es necesario estar familiarizado con la materia y los datos específicos del problema en cuestión, así como tener conocimientos de ciencia de datos.

## <a name="what-kind-of-data-do-you-have"></a>¿Qué tipo de datos tiene?

Cognitive Services, como grupo de servicios, puede requerir ninguno, algunos o todos los datos personalizados para el modelo entrenado. 

### <a name="no-additional-training-data-required"></a>No se requieren datos de entrenamiento adicionales

Los servicios que proporcionan un modelo totalmente entrenado se pueden tratar como una _caja opaca_. No es necesario saber cómo funcionan ni qué datos se usaron para entrenarlos. Puede llevar los datos a un modelo completamente entrenado para obtener una predicción. 

### <a name="some-or-all-training-data-required"></a>Algunos o todos los datos de entrenamiento son necesarios

Algunos servicios le permiten llevar sus propios datos y, a continuación, entrenar un modelo. Esto le permite ampliar el modelo con los datos y el algoritmo del servicio con sus propios datos. La salida coincide con sus necesidades. Al llevar sus propios datos, puede que tenga que etiquetar los datos de una manera específica en el servicio. Por ejemplo, si va a entrenar un modelo para identificar flores, puede proporcionar un catálogo de imágenes de flores junto con la ubicación de la flor en cada imagen para entrenar el modelo. 

Un servicio puede _permitirle_ proporcionar datos para mejorar sus propios datos. Un servicio puede _requerir_ que proporcione datos. 

### <a name="real-time-or-near-real-time-data-required"></a>Se necesitan datos en tiempo real o casi en tiempo real

Un servicio puede necesitar datos en tiempo real o casi en tiempo real para crear un modelo eficaz. Estos servicios procesan grandes cantidades de datos de modelo. 

## <a name="service-requirements-for-the-data-model"></a>Requisitos de servicio para el modelo de datos

Los datos siguientes categorizan cada servicio según el tipo de datos que permite o requiere.

|Cognitive Service|No se necesita ningún dato de entrenamiento|Proporciona algunos o todos los datos de entrenamiento|Recopilación de datos en tiempo real o casi en tiempo real|
|--|--|--|--|
|[Anomaly Detector](./Anomaly-Detector/overview.md)|x|x|x|
|Bing Search |x|||
|[Computer Vision](./Computer-vision/Home.md)|x|||
|[Content Moderator](./Content-Moderator/overview.md)|x||x|
|[Custom Vision](./Custom-Vision-Service/home.md)||x||
|[Face](./Face/Overview.md)|x|x||
|[Form Recognizer](./form-recognizer/overview.md)||x||
|[Lector inmersivo](./immersive-reader/overview.md)|x|||
|[Ink Recognizer](./Ink-recognizer/overview.md)|x|x||
|[Language Understanding (LUIS)](./LUIS/what-is-luis.md)||x||
|[Personalizer](./personalizer/what-is-personalizer.md)|x*|x*|x|
|[QnA Maker](./QnAMaker/Overview/overview.md)||x||
|[Speaker Recognizer](./speaker-recognition/home.md)||x||
|[Speech: Texto a voz (TTS)](speech-service/text-to-speech.md)|x|x||
|[Speech: Voz a texto (STT)](speech-service/speech-to-text.md)|x|x||
|[Traducción de voz](speech-service/speech-translation.md)|x|||
|[Text Analytics](./text-analytics/overview.md)|x|||
|[Traductor](./translator/translator-info-overview.md)|x|||
|[Traductor: traductor personalizado](./translator/custom-translator/overview.md)||x||

*Personalizer solo necesita datos de entrenamiento recopilados por el servicio (como funciona en tiempo real) para evaluar la directiva y los datos. Personalizer no necesita grandes conjuntos de datos históricos para entrenamiento inicial o por lotes. 

## <a name="where-can-you-use-cognitive-services"></a>¿Dónde se puede usar Cognitive Services?
 
Los servicios se usan en cualquier aplicación que pueda realizar llamadas API de REST o SDK. Entre los ejemplos de aplicaciones se incluyen sitios web, bots, realidad virtual o mixta, aplicaciones móviles y de escritorio. 

## <a name="how-is-azure-cognitive-search-related-to-cognitive-services"></a>¿Cómo se relaciona Azure Cognitive Search con Cognitive Services?

[Azure Cognitive Search](../search/search-what-is-azure-search.md) es un servicio de búsqueda en la nube independiente que, de manera opcional, usa Cognitive Services para agregar procesamiento de imágenes y lenguaje natural a las cargas de trabajo de indexación. Cognitive Services se expone en Azure Cognitive Search con [aptitudes integradas](../search/cognitive-search-predefined-skills.md) que encapsulan API individuales. Puede usar un recurso gratuito para los tutoriales, pero debe tener planeado crear y adjuntar un [recurso facturable](../search/cognitive-search-attach-cognitive-services.md) para volúmenes más grandes.

## <a name="how-can-you-use-cognitive-services"></a>¿Cómo se puede usar Cognitive Services?

Cada servicio proporciona información sobre los datos. Puede combinar servicios para encadenar soluciones como convertir voz (audio) a texto, traducir el texto a varios idiomas y usar los idiomas traducidos para obtener respuestas de una base de conocimiento. Si bien Cognitive Services se puede usar para crear soluciones inteligentes por su cuenta, también se pueden combinar con los proyectos de aprendizaje automático tradicionales para complementar los modelos o acelerar el proceso de desarrollo. 

Cognitive Services que proporcionan modelos exportados para otras herramientas de aprendizaje automático:

|Cognitive Service|Información del modelo|
|--|--|
|[Custom Vision](./custom-vision-service/home.md)|[Exportación](./Custom-Vision-Service/export-model-python.md) para Tensorflow para Android, CoreML para iOS11, ONNX para Windows ML|

## <a name="learn-more"></a>Más información

* [Guía de arquitectura: ¿Cuáles son los productos de aprendizaje automático de Microsoft?](https://docs.microsoft.com/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning)
* [Aprendizaje automático: Introducción a la relación entre aprendizaje profundo y aprendizaje automático](../machine-learning/concept-deep-learning-vs-machine-learning.md)

## <a name="next-steps"></a>Pasos siguientes

* Cree su cuenta de servicio de Cognitive Services ne [Azure Portal](cognitive-services-apis-create-account.md) o con el [CLI de Azure](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account-cli).
* Obtenga información sobre cómo [autenticarse](authentication.md) en un servicio de Cognitive Services.
* Use el [registro de diagnóstico](diagnostic-logging.md) para identificar y depurar problemas. 
* Implemente un servicio de Cognitive Services en un [contenedor](cognitive-services-container-support.md) de Docker.
* Manténgase al día con las [actualizaciones de servicio](https://azure.microsoft.com/updates/?product=cognitive-services).
