---
title: Vínculos de la documentación del enriquecimiento con inteligencia artificial
titleSuffix: Azure Cognitive Search
description: Una lista anotada de artículos, tutoriales, ejemplos y entradas de blog relacionados con cargas de trabajo de enriquecimiento con inteligencia artificial en Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/12/2020
ms.openlocfilehash: eaafb9536a8667417c0abf4ab570609f2db290bc
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935354"
---
# <a name="documentation-resources-for-ai-enrichment-in-azure-cognitive-search"></a>Recursos de documentación para el enriquecimiento con inteligencia artificial en Azure Cognitive Search

El enriquecimiento con inteligencia artificial es un complemento a la indexación basada en indexadores que busca información latente en orígenes no textuales y texto sin diferenciar, y la transforma en contenido que permite búsquedas de texto completo en Azure Cognitive Search. 

Para el procesamiento integrado, se llama internamente a los modelos de inteligencia artificial con entrenamiento previo de Cognitive Services para realizar los análisis. También puede integrar modelos personalizados con Azure Machine Learning, Azure Functions u otros enfoques.

A continuación se muestra una lista consolidada de la documentación sobre el enriquecimiento con IA.

## <a name="concepts"></a>Conceptos

+ [Enriquecimientos con IA](cognitive-search-concept-intro.md)
+ [Conjuntos de aptitudes](cognitive-search-working-with-skillsets.md)
+ [Sesiones de depuración](cognitive-search-debug-session.md)
+ [Almacenes de conocimiento](knowledge-store-concept-intro.md)
+ [Proyecciones](knowledge-store-projection-overview.md)
+ [Enriquecimiento incremental (reutilización de un documento enriquecido en caché)](cognitive-search-incremental-indexing-conceptual.md)

## <a name="hands-on-walkthroughs"></a>Tutoriales prácticos

+ [Inicio rápido: Creación de un conjunto de aptitudes cognitivo en Azure Portal](cognitive-search-quickstart-blob.md)
+ [Tutorial: Enriched indexing with AI](cognitive-search-tutorial-blob.md) (Tutorial: Indexación enriquecida con inteligencia artificial)
+ [Tutorial: Diagnóstico, reparación y confirmación de cambios en el conjunto de aptitudes con sesiones de depuración](cognitive-search-tutorial-debug-sessions.md)

## <a name="knowledge-stores"></a>Almacenes de conocimiento

+ [Inicio rápido: Creación de un almacén de conocimiento en Azure Portal](knowledge-store-create-portal.md)
+ [Creación de un almacén de conocimiento mediante REST y Postman](knowledge-store-create-rest.md)
+ [Visualización de un almacén de conocimiento con el Explorador de Storage](knowledge-store-view-storage-explorer.md)
+ [Conexión a un almacén de conocimiento con Power BI](knowledge-store-connect-power-bi.md)
+ [Ejemplos de proyección (dar forma y exportar enriquecimientos)](knowledge-store-projections-examples.md)

## <a name="custom-skills-advanced"></a>Aptitudes personalizadas (avanzado)

+ [How to define a custom skills interface](cognitive-search-custom-skill-interface.md) (Definición de una interfaz de aptitudes personalizada)
+ [Ejemplo: Creación de una aptitud personalizada con Azure Functions (y Bing Entity Search API)](cognitive-search-create-custom-skill-example.md)
+ [Ejemplo: Creación de una aptitud personalizada con Python](cognitive-search-custom-skill-python.md)
+ [Ejemplo: Creación de una aptitud personalizada mediante Form Recognizer](cognitive-search-custom-skill-form.md) 
+ [Ejemplo: Creación de una aptitud personalizada con Azure Machine Learning](cognitive-search-tutorial-aml-custom-skill.md) 

## <a name="how-to-guidance"></a>Guía de procedimientos

+ [Adjuntar un recurso de Cognitive Services](cognitive-search-attach-cognitive-services.md)
+ [Define a skillset](cognitive-search-defining-skillset.md) (Definición de un conjunto de aptitudes)
+ [Referencia a las anotaciones de un conjunto de aptitudes](cognitive-search-concept-annotations-syntax.md)
+ [Asignación de campos enriquecidos a un índice](cognitive-search-output-field-mapping.md)
+ [Procesamiento y extracción de información de imágenes](cognitive-search-concept-image-scenarios.md)
+ [Configuración del almacenamiento en caché para el enriquecimiento incremental](search-howto-incremental-index.md)
+ [How to rebuild an Azure Cognitive Search index](search-howto-reindex.md) (Reconstrucción de un índice de Azure Cognitive Search)
+ [Sugerencias de diseño](cognitive-search-concept-troubleshooting.md)
+ [Errores y advertencias comunes](cognitive-search-common-errors-warnings.md)

## <a name="skills-reference"></a>Referencia sobre aptitudes

+ [Aptitudes integradas](cognitive-search-predefined-skills.md)
  + [Microsoft.Skills.Text.KeyPhraseExtractionSkill](cognitive-search-skill-keyphrases.md)
  + [Microsoft.Skills.Text.LanguageDetectionSkill](cognitive-search-skill-language-detection.md)
  + [Microsoft.Skills.Text.EntityRecognitionSkill](cognitive-search-skill-entity-recognition.md)
  + [Microsoft.Skills.Text.MergeSkill](cognitive-search-skill-textmerger.md)
  + [Microsoft.Skills.Text.PIIDetectionSkill](cognitive-search-skill-pii-detection.md)
  + [Microsoft.Skills.Text.SplitSkill](cognitive-search-skill-textsplit.md)
  + [Microsoft.Skills.Text.SentimentSkill](cognitive-search-skill-sentiment.md)
  + [Microsoft.Skills.Text.TranslationSkill](cognitive-search-skill-text-translation.md)
  + [Microsoft.Skills.Vision.ImageAnalysisSkill](cognitive-search-skill-image-analysis.md)
  + [Microsoft.Skills.Vision.OcrSkill](cognitive-search-skill-ocr.md)
  + [Microsoft.Skills.Util.ConditionalSkill](cognitive-search-skill-conditional.md)
  + [Microsoft.Skills.Util.DocumentExtractionSkill](cognitive-search-skill-document-extraction.md)
  + [Microsoft.Skills.Util.ShaperSkill](cognitive-search-skill-shaper.md)

+ Aptitudes personalizadas
  + [Microsoft.Skills.Custom.AmlSkill](cognitive-search-aml-skill.md)
  + [Microsoft.Skills.Custom.WebApiSkill](cognitive-search-custom-skill-web-api.md)

+ [Aptitudes en desuso](cognitive-search-skill-deprecated.md)
  + [Microsoft.Skills.Text.NamedEntityRecognitionSkill](cognitive-search-skill-named-entity-recognition.md)

## <a name="apis"></a>API existentes

+ [REST API](/rest/api/searchservice/)
  + [Creación de un conjunto de aptitudes (api-version=2020-06-30)](/rest/api/searchservice/create-skillset)
  + [Create Indexer (api-version=2020-06-30)](/rest/api/searchservice/create-indexer)

## <a name="see-also"></a>Consulte también

+ [API REST de Azure Cognitive Search](/rest/api/searchservice/)
+ [Indexadores de Azure Cognitive Search](search-indexer-overview.md)
+ [¿Qué es Azure Cognitive Search?](search-what-is-azure-search.md)