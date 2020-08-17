---
title: 'Idiomas admitidos: Text Analytics API'
titleSuffix: Azure Cognitive Services
description: Una lista de los idiomas naturales admitidos por Text Analytics API. En este artículo se explica qué idiomas se admiten para cada operación.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: aahi
ms.openlocfilehash: b3cec783b43d0169a3d0d56ca040e1397701102e
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87986175"
---
# <a name="text-analytics-api-v3-language-support"></a>Idiomas admitidos en Text Analytics API v3 

> [!IMPORTANT]
> La versión 3. x de Text Analytics API no está disponible actualmente en las siguientes regiones: Centro de la India, Norte de Emiratos Árabes Unidos, Norte de China 2, Este de China.


#### <a name="sentiment-analysis"></a>[Análisis de sentimiento](#tab/sentiment-analysis)

| Idioma              | Código de lenguaje | Compatibilidad con la versión 2 | Compatibilidad con la versión v3 | A partir de la versión del modelo v3: |              Notas |
|:----------------------|:-------------:|:----------:|:----------:|:--------------------------:|-------------------:|
| Chino simplificado    |   `zh-hans`   |     ✓      |     ✓      |         2019-10-01         | También se acepta `zh` |
| Chino (tradicional)   |   `zh-hant`   |            |     ✓      |         2019-10-01         |                    |
| Danés               |     `da`      |     ✓      |            |                            |                    |
| Neerlandés                 |     `nl`      |     ✓      |            |                            |                    |
| Inglés               |     `en`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Finés               |     `fi`      |     ✓      |            |                            |                    |
| Francés                |     `fr`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Alemán                |     `de`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Griego                 |     `el`      |     ✓      |            |                            |                    |
| Italiano               |     `it`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Japonés              |     `ja`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Coreano                |     `ko`      |            |     ✓      |         2019-10-01         |                    |
| Noruego (Bokmål)   |     `no`      |     ✓      |            |                            |                    |
| Polaco                |     `pl`      |     ✓      |            |                            |                    |
| Portugués (Portugal) |    `pt-PT`    |     ✓      |     ✓      |         2019-10-01         | También se acepta `pt` |
| Ruso               |     `ru`      |     ✓      |            |                            |                    |
| Español               |     `es`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Sueco               |     `sv`      |     ✓      |            |                            |                    |
| Turco               |     `tr`      |     ✓      |            |                            |                    |

### <a name="opinion-mining-v31-preview-only"></a>Minería de opiniones (solo v3.1-preview)

| Idioma              | Código de lenguaje | A partir de la versión del modelo v3: |              Notas |
|:----------------------|:-------------:|:------------------------------------:|-------------------:|
| Inglés               |     `en`      |              2020-04-01              |                    |


#### <a name="named-entity-recognition-ner"></a>[Reconocimiento de entidades con nombre (NER)](#tab/named-entity-recognition)

> [!NOTE]
> * NER v3 actualmente solo admite los idiomas inglés y español. Si llama a NER v3 con un idioma diferente, la API devolverá los resultados de v2.1, siempre que el idioma sea compatible con la versión 2.1.
> * La versión 2.1 solo devuelve el conjunto completo de entidades para los idiomas inglés, chino simplificado, francés, alemán y español.  Se devuelven las entidades "Persona", "Ubicación" y "Organización" para los demás idiomas admitidos.

| Idioma               | Código de lenguaje | Compatibilidad con la versión 2.1 | Compatibilidad con la versión 3 | A partir de la versión del modelo de la versión 3: |       Notas        |
|:-----------------------|:-------------:|:----------:|:----------:|:-------------------------------:|:------------------:|
| Árabe                |     `ar`      |     ✓      |            |                                 |                    |
| Checo                 |     `cs`      |     ✓      |            |                                 |                    |
| Chino simplificado     |   `zh-hans`   |     ✓      |            |                                 | También se acepta `zh` |
| Chino (tradicional)   |   `zh-hant`   |     ✓      |            |                                 |                    |
| Danés                |     `da`      |     ✓      |            |                                 |                    |
| Neerlandés                 |     `nl`      |     ✓      |            |                                 |                    |
| Inglés                |     `en`      |     ✓      |     ✓      |           2019-10-01            |                    |
| Finés               |     `fi`      |     ✓      |            |                                 |                    |
| Francés                 |     `fr`      |     ✓      |            |                                 |                    |
| Alemán                 |     `de`      |     ✓      |            |                                 |                    |
| Hebreo                |     `he`      |     ✓      |            |                                 |                    |
| Húngaro             |     `hu`      |     ✓      |            |                                 |                    |
| Italiano               |     `it`      |     ✓      |            |                                 |                    |
| Japonés              |     `ja`      |     ✓      |            |                                 |                    |
| Coreano                |     `ko`      |     ✓      |            |                                 |                    |
| Noruego (Bokmål)   |     `no`      |     ✓      |            |                                 | También se acepta `nb` |
| Polaco                |     `pl`      |     ✓      |            |                                 |                    |
| Portugués (Portugal) |    `pt-PT`    |     ✓      |            |                                 | También se acepta `pt` |
| Portugués (Brasil)   |    `pt-BR`    |     ✓      |            |                                 |                    |
| Ruso              |     `ru`      |     ✓      |            |                                 |                    |
| Español               |     `es`      |     ✓      |     ✓       |              2020-04-01                   |                    |
| Sueco               |     `sv`      |     ✓      |            |                                 |                    |
| Turco               |     `tr`      |     ✓      |            |                                 |                    |

#### <a name="key-phrase-extraction"></a>[Extracción de frases clave](#tab/key-phrase-extraction)

| Idioma              | Código de lenguaje | Compatibilidad con la versión 2 | Compatibilidad con la versión 3 | Disponible a partir de la versión del modelo 3: |       Notas        |
|:----------------------|:-------------:|:----------:|:----------:|:-----------------------------------------:|:------------------:|
| Neerlandés                 |     `nl`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Inglés               |     `en`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Finés               |     `fi`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Francés                |     `fr`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Alemán                |     `de`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Italiano               |     `it`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Japonés              |     `ja`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Coreano                |     `ko`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Noruego (Bokmål)   |     `no`      |     ✓      |     ✓      |                2019-10-01                 | También se acepta `nb` |
| Polaco                |     `pl`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Portugués (Portugal) |    `pt-PT`    |     ✓      |     ✓      |                2019-10-01                 | También se acepta `pt` |
| Portugués (Brasil)   |    `pt-BR`    |     ✓      |     ✓      |                2019-10-01                 |                    |
| Ruso               |     `ru`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Español               |     `es`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Sueco               |     `sv`      |     ✓      |     ✓      |                2019-10-01                 |                    |

#### <a name="entity-linking"></a>[Vinculación de entidad](#tab/entity-linking)

| Idioma | Código de lenguaje | Compatibilidad con la versión 2 | Compatibilidad con la versión 3 | Disponible a partir de la versión del modelo 3: | Notas |
|:---------|:-------------:|:----------:|:----------:|:-----------------------------------------:|:-----:|
| Inglés  |     `en`      |     ✓      |     ✓      |                2019-10-01                 |       |
| Español  |     `es`      |     ✓      |     ✓      |                2019-10-01                 |       |

#### <a name="language-detection"></a>[Detección de idioma](#tab/language-detection)

La API Text Analytics puede detectar una amplia variedad de idiomas, variantes, dialectos y algunos idiomas regionales o culturales.  La detección de idioma devuelve el "script" de un idioma. Por ejemplo, en el caso de la frase "Tengo un perro", se devolverá `en` en lugar de `en-US`. El único caso especial es el chino, donde la funcionalidad de detección de idioma devolverá `zh_CHS` o `zh_CHT` si puede determinar el script dado el texto proporcionado. En situaciones donde no se puede identificar un script específico para un documento chino, devolverá simplemente `zh`.

No se ha publicado la lista exacta de idiomas de esta característica, pero puede detectar una amplia gama de idiomas, variantes, dialectos y algunos lenguajes cultural o configuración regional. 

Si tiene contenido que se expresa en un idioma que se usa con menos frecuencia, puede probar Detección de idioma para ver si devuelve un código. La respuesta para los idiomas que no se pueden detectar es `unknown`.

---

## <a name="see-also"></a>Consulte también

* [¿Qué es Text Analytics API?](overview.md)   
