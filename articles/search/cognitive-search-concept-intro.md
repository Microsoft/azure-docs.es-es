---
title: Conceptos de enriquecimiento con IA
titleSuffix: Azure Cognitive Search
description: La extracción de contenido, el procesamiento de lenguaje natural (NLP) y el procesamiento de imágenes se usan para crear contenido en el que se puedan realizar búsquedas en índices de Azure Cognitive Search mediante aptitudes cognitivas predefinidas y algoritmos de inteligencia artificial personalizados.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/18/2020
ms.openlocfilehash: d6fbfc7dced59580e91c3beceb6054f223a0a17d
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319055"
---
# <a name="ai-enrichment-in-azure-cognitive-search"></a>Enriquecimiento con IA en Azure Cognitive Search

El enriquecimiento con IA es una extensión de [indexadores](search-indexer-overview.md) que se puede usar para extraer texto de imágenes, blobs y otros orígenes de datos no estructurados. El enriquecimiento y la extracción permiten que se puedan realizar búsquedas en los objetos de salida de indexadores, ya sea un [índice de búsqueda](search-what-is-an-index.md) o un [almacén de conocimiento](knowledge-store-concept-intro.md). 

La extracción y el enriquecimiento se implementan mediante *aptitudes cognitivas* asociadas a una canalización controlada por indexadores. Puede usar las aptitudes integradas de Microsoft o insertar el procesamiento externo en una [*aptitud personalizada*](cognitive-search-create-custom-skill-example.md) que cree. Algunos ejemplos de aptitudes personalizadas pueden ser un módulo de entidad personalizado o un clasificador de documentos que tiene como destino un dominio específico, como finanzas, publicaciones científicas o medicina.

Las aptitudes integradas se encuadran en estas categorías: 

+ Las aptitudes del **procesamiento de lenguaje natural** incluyen [reconocimiento de entidades](cognitive-search-skill-entity-recognition.md), [detección de idioma](cognitive-search-skill-language-detection.md), [extracción de frases clave](cognitive-search-skill-keyphrases.md), manipulación de texto, [detección de opiniones](cognitive-search-skill-sentiment.md) y [detección de información de identificación personal](cognitive-search-skill-pii-detection.md). Con estas aptitudes, un texto no estructurado se asigna como campos en los que se pueden realizar búsquedas y aplicar filtros en un índice.

+ Las aptitudes de **procesamiento de imágenes** incluyen [reconocimiento óptico de caracteres (OCR)](cognitive-search-skill-ocr.md) e identificación de [características visuales](cognitive-search-skill-image-analysis.md), como detección facial, interpretación de imágenes, reconocimiento de imágenes (personas famosas y puntos de referencia) o atributos como orientación de la imagen. Estas aptitudes crean representaciones de texto del contenido de las imágenes, lo que permite la realización de búsquedas en dicho contenido mediante las funcionalidades de consulta de Azure Cognitive Search.

![Diagrama de una canalización de enriquecimiento](./media/cognitive-search-intro/cogsearch-architecture.png "introducción a la canalización de enriquecimiento")

Las aptitudes integradas de Azure Cognitive Search se basan en los modelos de aprendizaje automático previamente entrenados de Cognitive Services APIs: [Computer Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/) y [Text Analytics](https://docs.microsoft.com/azure/cognitive-services/text-analytics/overview). Puede adjuntar un recurso de Cognitive Services si desea aprovechar estos recursos durante el procesamiento de contenido.

Tanto el lenguaje natural como el procesamiento de imágenes se aplican durante la fase de ingesta de datos, con resultados que forman parte de la composición de un documento en un índice en el que se pueden realizar búsquedas en Azure Cognitive Search. Los datos se obtienen como un conjunto de datos de Azure y luego se insertan a través de una canalización de indexación mediante las [aptitudes integradas](cognitive-search-predefined-skills.md) que necesite.  

## <a name="when-to-use-ai-enrichment"></a>Cuándo usar el enriquecimiento con IA

Considere la posibilidad de usar las aptitudes cognitivas integradas si el contenido sin procesar es texto no estructurado, contenido de imagen o contenido que requiere la detección y traducción de idiomas. La aplicación de inteligencia artificial a través de las aptitudes cognitivos integradas puede desbloquear este contenido, lo que aumenta su valor y utilidad en las aplicaciones de ciencia de datos y búsqueda. 

Además, puede considerar la posibilidad de agregar una aptitud personalizada si tiene código de código abierto, de terceros o de otro fabricante que le gustaría integrar en la canalización. Los modelos de clasificación que identifican las características destacadas de varios tipos de documento se encuentran en esta categoría, pero se puede usar cualquier paquete que agregue valor al contenido.

### <a name="more-about-built-in-skills"></a>Más información sobre las aptitudes integradas

Los [conjuntos de aptitudes](cognitive-search-defining-skillset.md) que se ensamblan mediante aptitudes integradas son apropiados para los siguientes escenarios de la aplicación:

+ Documentos digitalizados (JPEG) en los que desea realizar búsquedas de texto completo. Puede adjuntar una aptitud de reconocimiento óptico de caracteres (OCR) para identificar, extraer e ingerir texto de archivos JPEG.

+ PDF con combinación de imagen y texto. El texto de los archivos PDF se puede extraer durante la indexación sin el uso de los pasos del enriquecimiento, pero la adición del procesamiento de imágenes y del lenguaje natural a menudo puede producir un mejor resultado que el de una indexación estándar.

+ Contenido multilingüe en el que desea aplicar la detección de idioma y, posiblemente, la traducción del texto.

+ Documentos no estructurados o semiestructurados cuyo contenido tenga un significado o contexto inherentes que está oculto en el documento mayor. 

  Los blobs a menudo contienen un cuerpo de contenido grande que se empaqueta en un "campo" único. Al adjuntar aptitudes de procesamiento de imágenes y de lenguaje natural a un indizador, puede crear información que exista en el contenido sin procesar, pero que no se expone como campos distintos. Algunas aptitudes cognitivas integradas listas para usar que pueden servidor de ayuda: extracción de frases clave, análisis de opiniones y reconocimiento de entidades (personas, organizaciones y ubicaciones).

  Además, las aptitudes integradas también se pueden usar para reestructurar el contenido mediante operaciones de división de texto, combinación y modelado de forma.

### <a name="more-about-custom-skills"></a>Más información sobre las aptitudes personalizadas

Las aptitudes personalizadas pueden admitir escenarios más complejos, como el reconocimiento de formularios o la detección de entidades personalizadas mediante un modelo que se proporciona y se encapsula en la [interfaz web de aptitudes personalizadas](cognitive-search-custom-skill-interface.md). Entre los ejemplos de aptitudes personalizadas se incluyen [Form Recognizer](/azure/cognitive-services/form-recognizer/overview), la integración de [Bing Entity Search API](https://docs.microsoft.com/azure/search/cognitive-search-create-custom-skill-example) y el [reconocimiento de entidades personalizadas](https://github.com/Microsoft/SkillsExtractorCognitiveSearch).

## <a name="steps-in-an-enrichment-pipeline"></a>Pasos en una canalización de enriquecimiento <a name="enrichment-steps"></a>

Una canalización de enriquecimiento se basa en [*indizadores*](search-indexer-overview.md). Los indizadores rellenan un índice basándose en las asignaciones de campo a campo entre el índice y el origen de datos para la descifrado de documentos. Las aptitudes, ahora asociadas a los indizadores, interceptan y enriquecen los documentos en función del conjunto de aptitudes que se definen. Una vez que se indexa, puede acceder al contenido mediante solicitudes de búsqueda a través de todos los [tipos de consulta compatibles con Azure Cognitive Search](search-query-overview.md).  Si es la primera vez que usa los indexadores, esta sección lo guiará a través de los pasos.

### <a name="step-1-connection-and-document-cracking-phase"></a>Paso 1: Fase de conexión y descifrado de documentos

Al comienzo de la canalización, hay texto no estructurado o contenido que no es de texto, como imágenes, documentos escaneados o archivos JPEG. Los datos deben estar en un servicio de almacenamiento de datos de Azure al que un indexador puede acceder. Los indexadores pueden "descifrar" los documentos de origen para extraer texto de los datos de origen. El descifrado de documentos es el proceso de extraer o crear contenido de texto a partir de orígenes que no son de texto durante el indexado.

![Fase de descifrado de documentos](./media/cognitive-search-intro/document-cracking-phase-blowup.png "descifrado de documentos")

 Entre los orígenes compatibles se incluyen Azure Blob Storage, Azure Table Storage, Azure SQL Database y Azure Cosmos DB. El contenido basado en texto se puede extraer de estos tipos de archivo: archivos PDF, Word, PowerPoint, archivos CSV. Para ver la lista completa, consulte los [formatos compatibles](search-howto-indexing-azure-blob-storage.md#supported-document-formats). La indexación toma tiempo, por lo que empiece con un conjunto de datos representativo y pequeño para ir compilando el índice de manera incremental a medida que la solución madura.

### <a name="step-2-cognitive-skills-and-enrichment-phase"></a>Paso 2: Fase de enriquecimiento y aptitudes cognitivas

El enriquecimiento se lleva a cabo mediante *aptitudes cognitivas* que realizan operaciones atómicas. Por ejemplo, una vez que ha descifrado un archivo PDF, puede aplicar la detección de idioma del reconocimiento de entidades o la extracción de frases clave para generar campos nuevos en el índice que no están disponibles de manera nativa en el origen. La colección completa de aptitudes que usa en la canalización se conoce como un *conjunto de aptitudes*.  

![Fase de enriquecimiento](./media/cognitive-search-intro/enrichment-phase-blowup.png "fase de enriquecimiento")

Un conjunto de aptitudes se basa en las [aptitudes cognitivas integradas](cognitive-search-predefined-skills.md) o en las [aptitudes personalizadas](cognitive-search-create-custom-skill-example.md) que proporciona y conecta con el conjunto de aptitudes. Un conjunto de aptitudes puede ser mínimo o altamente complejo y no solo determina el tipo de procesamiento, sino también el orden de las operaciones. Un conjunto de aptitudes más las asignaciones de campos que se definen como parte de un indexador especifica completamente la canalización de enriquecimiento. Para más información sobre cómo unir todos estos elementos, consulte el artículo sobre cómo [definir un conjunto de aptitudes](cognitive-search-defining-skillset.md).

Internamente, la canalización genera una colección de documentos enriquecidos. Puede decidir qué elementos de los documentos enriquecidos se deben asignar a los campos del índice de búsqueda que se pueden indexar. Por ejemplo, si aplicara las aptitudes de reconocimiento de entidades y de extracción de frases clave, esos campos nuevos se volverían parte del documento enriquecido y se podrían asignar a los campos del índice. Consulte las [anotaciones](cognitive-search-concept-annotations-syntax.md) para más información sobre las formaciones de entrada/salida.

#### <a name="add-a-knowledgestore-element-to-save-enrichments"></a>Adición de un elemento knowledgeStore para guardar enriquecimientos

El elemento [api-version=2020-06-30 de REST de Search](https://docs.microsoft.com/rest/api/searchservice/) extiende los conjuntos de aptitudes con una definición de `knowledgeStore` que proporciona una conexión al almacenamiento de Azure y proyecciones que describen cómo se almacenan los enriquecimientos. Esto se suma al índice. En una canalización estándar de inteligencia artificial, los documentos enriquecidos son transitorios, solo se usan durante la indexación y después se descartan. Los documentos enriquecidos se conservan mediante el almacén de conocimiento. Para obtener más información, consulte [Almacén de conocimiento](knowledge-store-concept-intro.md).

### <a name="step-3-search-index-and-query-based-access"></a>Paso 3: Índice de búsqueda y acceso basado en consulta

Al finalizar el procesamiento, obtiene un índice de búsqueda que consta de documentos enriquecidos con texto totalmente apto para búsquedas en Azure Cognitive Search. Para acceder al contenido enriquecido que la canalización genera, los desarrolladores y usuarios [consultan el índice](search-query-overview.md). 

![Índice con icono de búsqueda](./media/cognitive-search-intro/search-phase-blowup.png "Índice con icono de búsqueda")

El índice es como cualquier otro elemento que se pueda crear para Azure Cognitive Search: puede complementarlo con analizadores personalizados, invocar consultas de búsqueda aproximada, agregar la búsqueda filtrada o experimentar con perfiles de puntuación para volver a dar forma a los resultados de la búsqueda.

Los índices se generan a partir de un esquema de índice que define los campos, atributos y otras construcciones adjuntas a un índice específico, como los perfiles de puntuación y las asignaciones de sinónimos. Una vez que se define y rellena un índice, puede indexar de manera incremental para seleccionar documentos de origen nuevos y actualizados. Algunas de las determinaciones requieren recompilar completamente el índice. Debe usar un conjunto de datos pequeño hasta que el diseño del esquema sea estable. Para más información, consulte el artículo sobre cómo [recompilar un índice](search-howto-reindex.md).

**Lista de comprobación: Un flujo de trabajo típico**

1. Subdivida los datos de origen de Azure en un ejemplo representativo. La indexación toma tiempo, por lo que empiece con un conjunto de datos representativo y pequeño para ir compilando el índice de manera incremental a medida que la solución madura.

1. Cree en Azure Cognitive Search un [objeto de origen de datos](https://docs.microsoft.com/rest/api/searchservice/create-data-source) que proporcione una cadena de conexión para la recuperación de datos.

1. Cree un [conjunto de aptitudes](https://docs.microsoft.com/rest/api/searchservice/create-skillset) con pasos de enriquecimiento.

1. Defina el [esquema de índice](https://docs.microsoft.com/rest/api/searchservice/create-index). La colección *Fields* (Campos) incluye campos de los datos de origen. También debe simular campos adicionales para contener valores generados para el contenido creado durante el enriquecimiento.

1. Defina el [indexador](https://docs.microsoft.com/rest/api/searchservice/create-indexer) que hace referencia al origen de datos, conjunto de aptitudes e índice.

1. Dentro del indexador, agregue *outputFieldMappings*. En esta sección, la salida se asigna desde el conjunto de aptitudes (en el paso 3) a los campos de entrada del esquema de índice (en el paso 4).

1. Envíe la solicitud *Create Indexer* (Crear indexador) que acaba de crear (una solicitud POST con una definición de indexador en el cuerpo de la solicitud) para expresar el indexador en Azure Cognitive Search. Este paso se refiere a cómo ejecuta el indexador mediante la invocación de la canalización.

1. Ejecute consultas para evaluar los resultados y modifique el código para actualizar la configuración del indexador, el esquema o los conjuntos de aptitudes.

1. [Restablezca el indexador](search-howto-reindex.md) antes de recompilar la canalización.

## <a name="next-steps"></a>Pasos siguientes

+ [Vínculos de la documentación del enriquecimiento con inteligencia artificial](cognitive-search-resources-documentation.md)
+ [Ejemplo: Creación de una aptitud personalizada para el enriquecimiento con inteligencia artificial (C#)](cognitive-search-create-custom-skill-example.md)
+ [Inicio rápido: Tutorial de la prueba del enriquecimiento con IA en un portal](cognitive-search-quickstart-blob.md)
+ [Tutorial: Más información sobre las API de enriquecimiento con inteligencia artificial](cognitive-search-tutorial-blob.md)
+ [Almacén de conocimiento](knowledge-store-concept-intro.md)
+ [Creación de un almacén de conocimiento con REST](knowledge-store-create-rest.md)
+ [Sugerencias para la solución de problemas](cognitive-search-concept-troubleshooting.md)
