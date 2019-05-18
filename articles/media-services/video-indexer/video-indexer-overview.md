---
title: ¿Qué es Video Indexer?
titlesuffix: Azure Media Services
description: En este tema se proporciona información general del servicio Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: juliako
ms.openlocfilehash: 2c72c7c493c0a887adab147054c725a2e1c0659f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799137"
---
# <a name="what-is-video-indexer"></a>¿Qué es Video Indexer?

Azure Video Indexer es una aplicación en la nube creada en Azure Media Analytics, Azure Search y Cognitive Services (como Face API, Microsoft Translator, Computer Vision API y Custom Speech Service). Permite extraer la información de los vídeos con los modelos de vídeo y audio de Video Indexer que se describen a continuación:
  
## <a name="video-insights"></a>Información de los vídeos

- **Detección de caras**: detecta y agrupa las caras que aparecen en el vídeo.
- **Identificación de celebridades**: Video Indexer identifica automáticamente las celebridades de más de 1 millón – como líderes mundiales, actores, actresses, atletas, los investigadores, negocios y líderes tecnológicos en todo el mundo. Los datos de estos famosos también pueden encontrarse en varios sitios web famosos, como IMDB y Wikipedia.
- **Identificación facial basada en cuentas**:  Video Indexer entrena un modelo para una cuenta específica. A continuación, reconozca caras en el vídeo según el modelo entrenado. Para obtener más información, consulte [personalizar un modelo de persona desde el sitio Web de Video Indexer](customize-person-model-with-website.md) y [personalizar un modelo de la persona con la API de Video Indexer](customize-person-model-with-api.md).
- **Extracción de miniaturas de caras** ("mejor cara"): identifica automáticamente la mejor cara capturada en cada grupo de caras (según la calidad, el tamaño y la posición frontal) y la extrae como un recurso de imagen.
- **Reconocimiento de texto visual** (OCR): extrae el texto que se muestra visualmente en el vídeo.
- **Moderación de contenido visual**: Detecta los objetos visuales para adultos o subidos de tono.
- **Identificación de etiquetas**: identifica los objetos visuales y las acciones que se muestran.
- **Segmentación de la escena**: determina cuándo los cambios de una escena en según las indicaciones visuales de vídeo. Una escena representa un evento único y está compuesto por una serie de capturas consecutivas, que son semánticamente relacionados. 
- **Detección de captura**: determina cuando se cambia una captura de vídeo en función de las indicaciones visuales. Una captura es una serie de marcos que se toman de la misma cámara imágenes en movimiento. Para obtener más información, consulte [escenas, capturas y fotogramas clave](scenes-shots-keyframes.md).
- **Detección de fotogramas negros**: identifica los fotogramas negros presentados en el vídeo.
- **Extracción de fotogramas clave**: detecta los fotogramas clave estables en un vídeo.
- **Las sucesivas créditos**: identificar el principio y final de los créditos graduales al final de programas de TV y películas.

## <a name="audio-insights"></a>Información de audio

- **Detección de idioma automática**: identifica automáticamente el idioma hablado dominante. Idiomas admitidos son inglés, español, francés, alemán, italiano, chino (simplificado), japonés, ruso y portugués brasileño portugués le reserva a inglés cuando no se puede detectar el idioma.
- **Transcripción de audio**: convierte la voz en texto en 12 idiomas y permite extensiones. Idiomas admitidos son inglés, español, francés, alemán, italiano, chino (simplificado), japonés, árabe, ruso, portugués (Brasil), Hindi y coreano.
- **Subtítulos (CC)**: crea subtítulos en tres formatos: VTT, TTML y SRT.
- **Procesamiento de dos canales**: detecta, separa la transcripción y se combina en una única escala de tiempo de forma automática.
- **Reducción de ruido**: borra el audio telefónico o las grabaciones ruidosas (basado en los filtros de Skype).
- **Personalización de la transcripción** (CRIS): Entrena personalizado de voz a los modelos de texto para crear una transcripción específicas del sector. Para obtener más información, consulte [personalizar un modelo de lenguaje desde el sitio Web de Video Indexer](customize-language-model-with-website.md) y [personalizar un modelo de lenguaje con las API de indizador de vídeo](customize-language-model-with-api.md).
- **Enumeración de altavoz**: asigna y comprende las palabras que transmitió cada altavoz y cuándo.
- **Estadísticas de altavoz**: proporciona las estadísticas de las relaciones de voz de los altavoces.
- **Moderación de contenido textual**: detecta texto explícito en la transcripción de audio.
- **Efectos de audio**: identifica los efectos de audio, como palmadas, voces y silencios.
- **Detección de emociones**: Identifica emoticonos en función de voz (lo que se dice) y la tonalidad de voz (cómo es que se va a dicho).  La emoción podría ser: felicidad, tristeza, ira o miedo.
- **Traducción**: crea traducciones de la transcripción de audio en 54 idiomas diferentes.

## <a name="audio-and-video-insights-multi-channels"></a>Información de audio y vídeo (varios canales)

Al indexar por un canal, puede haber resultados parciales para esos modelos disponibles

- **Extracción de palabras clave**: extrae palabras clave a partir de voz y texto visual.
- **Extracción de marcas**: extrae marcas a partir de voz y texto visual.
- **Inferencia de tema**: saca conclusiones de los temas principales a partir de las transcripciones. Se incluye la taxonomía IPTC de primer nivel.
- **Artefactos**: extrae una amplia variedad de artefactos de "detalles de siguiente nivel" para cada uno de los modelos.
- **Análisis de opiniones**: identifica opiniones positivas, negativas y neutras a partir de voz y texto visual.
 
Cuando Video Indexer termina de procesar y analizar, se puede revisar, ajustar, buscar y publicar la información de vídeo.

Tanto si su rol es el de administrador de contenido como el de desarrollador, el servicio Video Indexer puede satisfacer sus necesidades. Los administradores de contenido pueden utilizar el portal web de Video Indexer para consumir el servicio sin tener que escribir una sola línea de código; consulte el artículo sobre la [introducción al sitio web de Video Indexer](video-indexer-get-started.md). Los desarrolladores pueden aprovechar las API para procesar el contenido a escala; consulte [Uso de Video Indexer REST API](video-indexer-use-apis.md). El servicio también permite a los clientes utilizar widgets para publicar secuencias de vídeo y extraer información en sus propias aplicaciones; consulte el tema sobre la [inserción de widgets visuales en la aplicación](video-indexer-embed-widgets.md).

Puede suscribirse al servicio mediante una cuenta existente de AAD, LinkedIn, Facebook, Google o MSA. Para más información, consulte la [introducción](video-indexer-get-started.md).

## <a name="scenarios"></a>Escenarios

A continuación se presentan algunos escenarios en los que Video Indexer puede ser útil.

- Búsqueda: las información extraída del vídeo se puede utilizar para mejorar la experiencia de búsqueda en una biblioteca de vídeos. Por ejemplo, la indexación de palabras habladas y caras puede permitir la experiencia de búsqueda de momentos en un vídeo en los que una persona en particular ha pronunciado ciertas palabras o cuando se han visto juntas a dos personas. La búsqueda basada en esta información de los vídeos se puede aplicar a agencias de noticias, instituciones educativas, emisoras, propietarios de contenido de entretenimiento, aplicaciones de línea de negocio de empresas y, en general, a cualquier sector que tenga una biblioteca de vídeos en la que los usuarios necesiten buscar.
- Creación de contenido: insights extraen de vídeos y ayudan a crear eficazmente los contenidos como finalizadores, contenido de medios sociales, noticias clips etc. del contenido existente en el archivo de la organización 
- Monetización: Video Indexer puede ayudar a mejorar el valor de los vídeos. Por ejemplo, los sectores que dependen de los ingresos por publicidad (por ejemplo, los medios de comunicación, las redes sociales, etc.), pueden entregar anuncios más relevantes con los conocimientos extraídos como señales adicionales para el servidor de anuncios (la presentación de un anuncio de calzado deportivo es más relevante en medio de un partido de fútbol que en una competición de natación).
- Involucración del usuario: la información del vídeo se puede utilizar para mejorar la involucración de los usuarios al posicionar momentos de vídeo más relevantes para ellos. Como ejemplo, considere un vídeo educativo que explique las esferas durante los primeros 30 minutos y las pirámides durante los siguientes 30 minutos. El estudiante que lee sobre las pirámides se beneficiaría más si el vídeo se posiciona a partir del marcador de 30 minutos.

## <a name="next-steps"></a>Pasos siguientes

Ya está listo para empezar a trabajar con Video Indexer. Para más información, consulte los siguientes artículos.

- [Introducción al sitio web de Video Indexer](video-indexer-get-started.md)
- [Proceso de contenido con Video Indexer REST API](video-indexer-use-apis.md)
- [Inserción de widgets visuales en la aplicación](video-indexer-embed-widgets.md)
