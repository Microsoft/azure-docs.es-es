---
title: ¿Qué es Video Indexer de Azure Media Services?
titleSuffix: Azure Media Services
description: En este artículo se proporciona información general sobre el servicio Video Indexer de Azure Media Services.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 08/31/2020
ms.author: juliako
ms.openlocfilehash: bff99cdd44763c0df6bb5b2dd863b27728c23fcd
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89318697"
---
# <a name="what-is-azure-media-services-video-indexer"></a>¿Qué es Video Indexer de Azure Media Services?

[!INCLUDE [regulation](./includes/regulation.md)]

Video Indexer (VI) es la solución de IA de Azure Media Services y parte de la marca Azure Cognitive Services. Video Indexer proporciona la capacidad de extraer información profunda (sin necesidad de análisis de datos o conocimientos de codificación) mediante el uso de modelos de Machine Learning basados en varios canales (voz, voces y objeto visual). Los modelos se pueden personalizar y entrenar aún más. El servicio habilita la búsqueda profunda, reduce los costos operativos, permite nuevas oportunidades de monetización y crea nuevas experiencias de usuario en grandes archivos de vídeos (con barreras de entrada bajas).

Para empezar a extraer información con Video Indexer, tiene que crear una cuenta y cargar vídeos. Cuando se cargan vídeos en Video Indexer, este analiza los objetos visuales y el audio mediante la ejecución de diferentes modelos de IA. A medida que Video Indexer analiza el vídeo, los modelos de IA extraen la información.

Cuando crea una cuenta de Video Indexer y la conecta a Media Services, los archivos multimedia y de metadatos se almacenan en la cuenta de almacenamiento de Azure asociada con esa cuenta de Media Services. Obtenga información sobre cómo [crear una cuenta de Video Indexer conectada a Azure](connect-to-azure.md).

El siguiente diagrama es una ilustración, no una explicación técnica, de cómo funciona Video Indexer en el back-end.

![Diagrama de flujo de Video Indexer de Azure Media Services](./media/video-indexer-overview/model-chart.png)


## <a name="compliance-privacy-and-security"></a>Cumplimiento, privacidad y seguridad

Como recordatorio importante, debe cumplir todas las leyes aplicables al uso de Video Indexer y no puede utilizar este servicio de Azure ni ningún otro de forma que infrinja los derechos de otras personas o que pueda ser perjudicial para ellas.

Antes de cargar un vídeo o una imagen en Video Indexer, debe tener todos los derechos adecuados para usar ese vídeo o imagen, incluidos (cuando lo exija la ley) todos los consentimientos necesarios de las personas (si las hay) que aparecen en ellos, para el uso, el procesamiento y el almacenamiento de sus datos en Video Indexer y Azure. Algunas jurisdicciones pueden imponer requisitos legales especiales sobre la recopilación, el procesamiento en línea y el almacenamiento de determinadas categorías de datos, como los datos biométricos. Antes de usar Video Indexer y Azure para el procesamiento y el almacenamiento de datos sujetos a requisitos legales especiales, debe garantizar el cumplimiento de tales requisitos legales que puedan aplicarse en su caso.

Para información sobre el cumplimiento, la privacidad y la seguridad en Video Indexer, visite el [Centro de confianza](https://www.microsoft.com/TrustCenter/CloudServices/Azure/default.aspx) de Microsoft. En lo que respecta a las obligaciones de privacidad y a las prácticas de retención y tratamiento de datos de Microsoft, entre las que se incluye la forma de eliminar los datos, lea la [declaración de privacidad](https://privacy.microsoft.com/PrivacyStatement), los [términos de Online Services](https://www.microsoft.com/licensing/product-licensing/products?rtc=1) ("OST") y el [anexo de procesamiento de datos](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=67) ("DPA") de Microsoft. Con el uso de Video Indexer, acepta estar vinculado por el OST, el DPA y la declaración de privacidad.

## <a name="what-can-i-do-with-video-indexer"></a>¿Qué puedo hacer con Video Indexer?

La información de Video Indexer se puede aplicar a muchos escenarios, entre ellos:

* *Búsqueda profunda*: use la información extraída del vídeo para mejorar la experiencia de búsqueda en una biblioteca de vídeos. Por ejemplo, la indexación de palabras habladas y de caras, permite la experiencia de búsqueda de momentos en un vídeo en los que una persona ha pronunciado ciertas palabras o cuando se han visto juntas a dos personas. La búsqueda basada en esta información de los vídeos se puede aplicar a agencias de noticias, instituciones educativas, emisoras, propietarios de contenido de entretenimiento, aplicaciones de línea de negocio de empresas y, en general, a cualquier sector que tenga una biblioteca de vídeos en la que los usuarios necesiten buscar.
* *Creación de contenido*: cree clips finales, resalte momentos destacados, contenido de redes sociales o clips de noticias basados en la información que Video Indexer extrae del contenido. Los fotogramas clave, los marcadores de escenas y las marcas de tiempo para las apariciones de personas y etiquetas hacen que el proceso de creación sea mucho más fluido y sencillo y le permiten ir a las partes del vídeo que necesita para el contenido que está creando.
* *Accesibilidad*: si quiere que el contenido esté disponible para personas con discapacidades o que se distribuya en distintas regiones con diferentes idiomas, puede usar la transcripción y traducción que proporciona Video Indexer en varios idiomas.
* *Monetization*: Video Indexer puede ayudar a aumentar el valor de los vídeos. Por ejemplo, los sectores que dependen de los ingresos por publicidad (medios de comunicación, redes sociales, etc.), pueden entregar anuncios relevantes con la información extraída como señales adicionales al servidor de anuncios.
* *Moderación de contenido*: use modelos de moderación de contenido textual y visual para proteger a los usuarios del contenido inadecuado y asegúrese de que el contenido que publica coincide con los valores de la organización. Puede bloquear automáticamente determinados vídeos o avisar a los usuarios sobre el contenido.
* *Recomendaciones*: La información de vídeo se puede usar para mejorar la interacción con los usuarios, resaltando los momentos de vídeo que les puedan resultar más pertinentes. Al etiquetar cada vídeo con metadatos adicionales, puede recomendar a los usuarios los vídeos más importantes y resaltar las partes del vídeo que se adaptan mejor a sus necesidades.

## <a name="features"></a>Características

En la siguiente lista se muestra la información que se puede recuperar de los vídeos mediante los modelos de audio y vídeo de Video Indexer:

### <a name="video-insights"></a>Información de los vídeos

* **Detección de caras**: detecta y agrupa las caras que aparecen en el vídeo.
* **Identificación de celebridades**: Video Indexer identifica automáticamente a más de un millón de famosos (como líderes mundiales, actores y actrices, atletas, investigadores y líderes empresariales y tecnológicos de todo el mundo). Los datos sobre estos famosos también pueden encontrarse en diversos sitios web (IMDB, Wikipedia, etc.).
* **Identificación facial basada en cuentas**:  Video Indexer entrena un modelo para una cuenta específica. A continuación reconoce las caras en el vídeo según el modelo entrenado. Para más información, consulte [Customize a Person model from the Video Indexer website](customize-person-model-with-website.md) (Personalización de un modelo de persona desde el sitio web de Video Indexer) y [Customize a Person model with the Video Indexer API](customize-person-model-with-api.md) (Personalización de un modelo de persona con Video Indexer API).
* **Extracción de miniaturas de caras** ("mejor cara"): identifica automáticamente la mejor cara capturada en cada grupo de caras (según la calidad, el tamaño y la posición frontal) y la extrae como un recurso de imagen.
* **Reconocimiento de texto visual** (OCR): extrae el texto que se muestra visualmente en el vídeo.
* **Moderación de contenido visual**: Detecta los objetos visuales para adultos o subidos de tono.
* **Identificación de etiquetas**: identifica los objetos visuales y las acciones que se muestran.
* **Segmentación de escenas**: determina cuándo cambia una escena en el vídeo en función de las indicaciones visuales. Una escena representa un evento único y se compone de una serie de cortes consecutivos que están semánticamente relacionados.
* **Detección de cortes**: determina cuándo cambia un corte en el vídeo en función de las indicaciones visuales. Un corte es una serie de fotogramas tomados a partir de la misma cámara de imágenes en movimiento. Para más información, consulte [Scenes, shots, and keyframes](scenes-shots-keyframes.md) (Escenas, cortes y fotogramas clave).
* **Detección de fotogramas negros**: identifica los fotogramas negros presentados en el vídeo.
* **Extracción de fotogramas clave**: detecta los fotogramas clave estables en un vídeo.
* **Créditos de rodaje**: identifica el principio y el final de los créditos al final de las series de televisión y las películas.
* **Detección de personajes animados** (versión preliminar): detección, agrupación y reconocimiento de personajes en contenido animado a través de la integración con la [visión personalizada de Cognitive Services](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/). Para más información, consulte [Detección de personajes animados](animated-characters-recognition.md).
* **Detección del tipo de toma editorial**: etiquetado de capturas basado en su tipo (como plano general, plano medio, primer plano, primerísimo primer plano, dos capturas, varias personas, exterior e interior, etc.). Para más información, consulte [Detección del tipo de toma editorial](scenes-shots-keyframes.md#editorial-shot-type-detection).

### <a name="audio-insights"></a>Información de audio

* **Transcripción de audio**: convierte la voz en texto en 12 idiomas y permite extensiones. Los idiomas admitidos son los siguientes: alemán, árabe, chino (mandarín), coreano, español, francés, hindi, inglés, italiano, japonés, portugués y ruso.
* **Detección de idioma automática**: identifica automáticamente el idioma hablado dominante. Los idiomas admitidos son los siguientes: alemán, chino (mandarín), español, francés, inglés, italiano, japonés, portugués y ruso. Si el idioma hablado no se puede identificar con confianza, Video Indexer supone que es inglés. Para obtener más información, vea [Modelo de identificación de idiomas](language-identification-model.md).
* **Identificación y transcripción de voz en varios idiomas** (versión preliminar): identifica automáticamente el idioma hablado en diferentes segmentos del audio. Envía cada segmento del archivo multimedia que se va a transcribir y luego combina la transcripción en una unificada. Para más información, consulte [Identificación y transcripción automáticas del contenido de varios idiomas](multi-language-identification-transcription.md).
* **Subtítulos (CC)** : crea subtítulos en tres formatos: VTT, TTML y SRT.
* **Procesamiento de dos canales**: detecta, separa la transcripción y se combina en una única escala de tiempo de forma automática.
* **Reducción de ruido**: borra el audio telefónico o las grabaciones ruidosas (basado en los filtros de Skype).
* **Personalización de la transcripción** (CRIS): entrena modelos personalizados de conversión de voz en texto para crear transcripciones específicas del sector. Para más información, consulte [Customize a Language model from the Video Indexer website](customize-language-model-with-website.md) (Personalización de un modelo de lenguaje desde el sitio web de Video Indexer) y [Customize a Language model with the Video Indexer API](customize-language-model-with-api.md) (Personalización de un modelo de lenguaje con Video Indexer API).
* **Enumeración de altavoz**: asigna y comprende las palabras que transmitió cada altavoz y cuándo.
* **Estadísticas de altavoz**: Proporciona las estadísticas de las relaciones de voz de los altavoces.
* **Moderación de contenido textual**: detecta texto explícito en la transcripción de audio.
* **Efectos de audio**: identifica los efectos de audio, como palmadas, voces y silencios.
* **Detección de emociones**: identifica emociones en función de la voz (lo que se dice) y el tono (cómo se dice). La emoción podría ser felicidad, tristeza, ira o miedo.
* **Traducción**: crea traducciones de la transcripción de audio en 54 idiomas diferentes.

### <a name="audio-and-video-insights-multi-channels"></a>Información de audio y vídeo (varios canales)

Al indexar por un canal, hay resultados parciales para esos modelos.

* **Extracción de palabras clave**: extrae palabras clave a partir de voz y texto visual.
* **Extracción de entidades con nombre**: extrae marcas, ubicaciones y personas del lenguaje hablado y del texto visual mediante el procesamiento de lenguaje natural (NLP).
* **Inferencia de tema**: saca conclusiones de los temas principales a partir de las transcripciones. Se incluye la taxonomía IPTC de segundo nivel.
* **Artefactos**: extrae una amplia variedad de artefactos de "detalles de siguiente nivel" para cada uno de los modelos.
* **Análisis de opiniones**: identifica opiniones positivas, negativas y neutras a partir de voz y texto visual.

## <a name="how-can-i-get-started-with-video-indexer"></a>¿Cómo puedo empezar a usar Video Indexer?

Puede acceder a las funciones de Video Indexer de tres maneras:

* Portal de Video Indexer: una solución fácil de usar que le permite evaluar el producto, administrar la cuenta y personalizar los modelos.

    Para más información sobre el portal, consulte la [Introducción al sitio web de Video Indexer](video-indexer-get-started.md).  

* Integración de API: todas las funcionalidades de Video Indexer están disponibles a través de una API REST para que pueda integrar la solución en las aplicaciones y la infraestructura.

    Para empezar a trabajar como desarrollador, consulte  [Uso de la API REST de Video Indexer](video-indexer-use-apis.md).

* Widget insertable: permite insertar las experiencias de información, reproductor y editor de Video Indexer en la aplicación.

    Para más información, consulte  [Inserción de widgets de Video Indexer en las aplicaciones](video-indexer-embed-widgets.md).

Si usa el sitio web, la información se agrega como metadatos y es visible en el portal. Si usa las API, la información está disponible como archivo JSON.

## <a name="next-steps"></a>Pasos siguientes

Ya está listo para empezar a trabajar con Video Indexer. Para más información, consulte los siguientes artículos.

- [Introducción al sitio web de Video Indexer](video-indexer-get-started.md).
- [Procesamiento de contenido con la API REST de Video Indexer](video-indexer-use-apis.md).
- [Inserción de widgets visuales en la aplicación](video-indexer-embed-widgets.md).
