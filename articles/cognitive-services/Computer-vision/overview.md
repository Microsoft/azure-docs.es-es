---
title: ¿Qué es Computer Vision?
titleSuffix: Azure Cognitive Services
description: El servicio Computer Vision proporciona acceso a algoritmos avanzados para procesar imágenes y devolver información.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: overview
ms.date: 08/31/2020
ms.author: pafarley
ms.custom:
- seodec18
- cog-serv-seo-aug-2020
keywords: computer vision, computer vision applications, computer vision service
ms.openlocfilehash: f613080ec24fdfc4c739d4d0494aa2f3c36705cd
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89323125"
---
# <a name="what-is-computer-vision"></a>¿Qué es Computer Vision?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

El servicio Computer Vision de Azure proporciona acceso a algoritmos avanzados que procesan imágenes y devuelven información basada en las características visuales de interés. Por ejemplo, Computer Vision puede determinar si una imagen tiene contenido para adultos, buscar marcas u objetos específicos o buscar rostros humanos.

Se pueden crear aplicaciones de Computer Vision mediante un SDK de la biblioteca cliente o la llamada directa a la API REST. Esta página cubre ampliamente lo que puede hacer con Computer Vision.

## <a name="computer-vision-for-digital-asset-management"></a>Computer Vision para la administración de activos digitales

Computer Vision puede funcionar en muchos escenarios de administración de activos digitales (DAM). DAM es el proceso empresarial de organización, almacenamiento y recuperación de recursos multimedia enriquecidos, y la administración de los permisos y derechos digitales. Por ejemplo, es posible que una empresa desee agrupar e identificar imágenes basadas en logotipos, caras, objetos, colores visibles, etc. O bien, puede que desee [generar automáticamente leyendas para las imágenes](./Tutorials/storage-lab-tutorial.md) y adjuntar palabras clave para que admitan búsquedas. Para una solución DAM todo en uno que use Cognitive Services, Azure Cognitive Search e informes inteligentes, vea la [Guía del acelerador de la solución de minería del conocimiento](https://github.com/Azure-Samples/azure-search-knowledge-mining) en GitHub. Para ver otros ejemplos de DAM, consulte el repositorio [plantillas de la solución Computer Vision](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates).

## <a name="optical-character-recognition-ocr"></a>Reconocimiento óptico de caracteres (OCR)

Computer Vision incluye funcionalidades de [reconocimiento óptico de caracteres (OCR)](concept-recognizing-text.md). Puede usar la nueva Read API para extraer texto impreso y manuscrito de imágenes y documentos. Usa los modelos más recientes y funciona con texto en diversas superficies y fondos. Entre estos se incluyen recibos, pósteres, tarjetas de presentación, cartas y pizarras. Las dos API de OCR admiten la extracción de texto impreso en [varios idiomas](./language-support.md). Siga un [inicio rápido](#next-steps) para comenzar.

## <a name="analyze-images-for-insight"></a>Análisis de imágenes para obtener información

Puede analizar imágenes para detectar y proporcionar información detallada acerca de las características y funciones visuales. Todas las características de la tabla siguiente se proporcionan gracias a la API [de análisis de imágenes](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa). Siga un [inicio rápido](#next-steps) para comenzar.


### <a name="tag-visual-features"></a>Etiquetar características visuales

Identifique y etiquete las características visuales de una imagen a partir de un conjunto de miles de objetos, seres vivos, paisajes y acciones reconocibles. Cuando las etiquetas son ambiguas o no muy comunes, la respuesta de la API contiene “indicaciones” para aclarar el contexto de la etiqueta. El etiquetado no se limita al sujeto principal, como una persona en primer plano, sino que también incluye el entorno (interior o exterior), muebles, herramientas, plantas, animales, accesorios, gadgets, etc. [Etiquetar características visuales](concept-tagging-images.md)

### <a name="detect-objects"></a>Detección de objetos

La detección de objetos es similar al etiquetado, pero la API devuelve las coordenadas del rectángulo delimitador para cada etiqueta aplicada. Por ejemplo, si una imagen contiene un perro, un gato y una persona, la operación de detección mostrará estos objetos junto con sus coordenadas en la imagen. Puede usar esta funcionalidad para procesar más las relaciones entre los objetos de una imagen. También permite saber cuando hay varias instancias de la misma etiqueta en una imagen. [Detección de objetos](concept-object-detection.md)

### <a name="detect-brands"></a>Detección de marcas

Identifique las marcas comerciales en imágenes o vídeos desde una base de datos de miles de logotipos globales. Puede usar esta característica, por ejemplo, para detectar qué marcas son más populares en medios sociales o más frecuentes en la ubicación de los productos multimedia. [Detección de marcas](concept-brand-detection.md)

### <a name="categorize-an-image"></a>Clasificar una imagen

Identifique y clasifique toda una imagen mediante una [taxonomía de categoría](Category-Taxonomy.md) con jerarquías hereditarias de elementos primarios y secundarios. Las categorías se pueden usar solas o con nuestros nuevos modelos de etiquetado.<br/>Actualmente, el inglés es el único idioma que se admite para etiquetar y clasificar imágenes. [Clasificar una imagen](concept-categorizing-images.md)

### <a name="describe-an-image"></a>Describir una imagen

Genere una descripción de toda una imagen en lenguaje natural, con frases completas. Los algoritmos de Computer Vision generan varias descripciones en función de los objetos identificados en la imagen. Cada una de estas descripciones se evalúa y se genera una puntuación de confianza. Después, se devuelve una lista de puntuaciones de confianza ordenadas de más alta a más baja. [Describir una imagen](concept-describing-images.md)

### <a name="detect-faces"></a>Detección de caras

Detecte caras en una imagen y proporcione información acerca de ellas. Computer Vision devuelve las coordenadas, el rectángulo, el género y la edad de los rostros que detecta.<br/>Computer Vision proporciona un subconjunto de la funcionalidad del servicio [Face](/azure/cognitive-services/face/). Puede usar el servicio Face para realizar un análisis más detallado, como la identificación facial y la detección de poses. [Detección de caras](concept-detecting-faces.md)

### <a name="detect-image-types"></a>Detectar tipos de imagen

Detecte las características de una imagen, como por ejemplo, si una imagen es un dibujo lineal o la probabilidad de que sea una imagen prediseñada. [Detectar tipos de imagen](concept-detecting-image-types.md)

### <a name="detect-domain-specific-content"></a>Detectar contenido específico del dominio

Use los modelos de dominio para detectar e identificar el contenido específico del dominio en una imagen, como celebridades y monumentos. Por ejemplo, si una imagen contiene personas, Computer Vision puede usar un modelo de dominio para celebridades para determinar si las personas que se han detectado en la imagen son famosos conocidos. [Detectar contenido específico del dominio](concept-detecting-domain-content.md)

### <a name="detect-the-color-scheme"></a>Detectar la combinación de colores

Analice el uso del color en una imagen. Computer Vision puede determinar si una imagen está en blanco y negro o en color, y en las imágenes de color, identificar los colores dominantes y de énfasis. [Detectar la combinación de colores](concept-detecting-color-schemes.md)

### <a name="generate-a-thumbnail"></a>Generar una miniatura

Analice el contenido de una imagen para generar una miniatura adecuada de la misma. En primer lugar, Computer Vision genera una miniatura de alta calidad y, después, analiza los objetos de la imagen para determinar el *área de interés*. Luego, Computer Vision recorta la imagen para ajustarla a los requisitos del área de interés. La miniatura generada se puede presentar con una relación de aspecto diferente de la de la imagen original en función de sus necesidades. [Generar una miniatura](concept-generating-thumbnails.md)

### <a name="get-the-area-of-interest"></a>Obtener el área de interés

Analice el contenido de una imagen para devolver las coordenadas del *área de interés*. En lugar de recortar la imagen y generar una miniatura, Computer Vision devuelve las coordenadas del rectángulo delimitador de la región, por lo que la aplicación que realiza la llamada puede modificar la imagen original según sea necesario. [Obtener el área de interés](concept-generating-thumbnails.md#area-of-interest)

## <a name="moderate-content-in-images"></a>Moderación del contenido de las imágenes

Puede usar Computer Vision para [detectar contenido para adultos](concept-detecting-adult-content.md) en una imagen y devolver puntuaciones de confianza en las distintas clasificaciones. El umbral para el etiquetado de contenido se puede establecer en una escala deslizante, con el fin de que pueda ajustarlo a sus preferencias.

## <a name="use-containers"></a>Uso de contenedores

[Use contenedores de Computer Vision](computer-vision-how-to-install-containers.md) para reconocer texto impreso y manuscrito localmente, mediante la instalación de un contenedor de Docker estándar más cercano a los datos.

## <a name="image-requirements"></a>Requisitos de imagen

Computer Vision puede analizar las imágenes que cumplan los requisitos siguientes:

- La imagen se debe presentar en formato JPEG, PNG, GIF o BMP
- El tamaño de archivo de la imagen debe ser inferior a 4 megabytes (MB)
- Las dimensiones de la imagen deben ser mayores que 50 x 50 píxeles
  - Para la API Read, las dimensiones de la imagen deben estar entre 50 x 50 y 10 000 x 10 000 píxeles.

## <a name="data-privacy-and-security"></a>Seguridad y privacidad de datos

Al igual que sucede con todas las instancias de Cognitive Services, los desarrolladores que usan el servicio Computer Vision deben estar al tanto de las directivas de Microsoft sobre los datos de clientes. Para más información, consulte la [página de Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) en Microsoft Trust Center.

## <a name="next-steps"></a>Pasos siguientes

Introducción a Computer Vision, con una guía de inicio rápido:

- [Inicio rápido: Biblioteca cliente .NET de Computer Vision](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)
- [Inicio rápido: Biblioteca cliente Python de Computer Vision](./quickstarts-sdk/client-library.md?pivots=programming-language-python)
- [Inicio rápido: Biblioteca cliente Java de Computer Vision](./quickstarts-sdk/client-library.md?pivots=programming-language-java)
