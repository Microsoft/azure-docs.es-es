---
title: 'Idiomas admitidos: QnA Maker'
titleSuffix: Azure Cognitive Services
description: Una lista de referencias culturales y lenguajes naturales admitidos por QnA Maker para la base de conocimiento. No mezcle idiomas en la misma base de conocimiento.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: reference
ms.date: 09/24/2019
ms.openlocfilehash: c990b6980dea871679b0b301e293e4fb94748db7
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89650903"
---
# <a name="language-support-for-a-qna-maker-resource-and-knowledge-bases"></a>Idiomas admitidos en el recurso de QnA Maker y en las bases de conocimiento

El idioma del servicio se selecciona al crear la primera base de conocimiento en el recurso. Todas las bases de conocimiento adicionales del recurso deben estar en el mismo idioma. 

El idioma determina la relevancia de los resultados que QnA Maker proporciona en respuesta a las consultas de los usuarios. El recurso de QnA Maker y todas las bases de conocimiento dentro de ese recurso admiten solo un idioma. Solo debe ser un idioma para proporcionar los mejores resultados de respuesta para las consultas.

## <a name="single-language-per-resource"></a>Un solo idioma por recurso

Tenga en cuenta lo siguiente.

* Un servicio de QnA Maker y todas sus bases de conocimiento solo admiten un idioma.
* El idioma se establece de forma explícita cuando se crea la primera base de conocimiento del servicio.
* El idioma se determina a partir de los archivos y las direcciones URL que se agregan al crear la base de conocimiento.
* No se puede cambiar el idioma de ninguna otra base de conocimiento en el servicio.
* El servicio de Cognitive Search (clasificador n.º 1) y el servicio de QnA Maker (clasificador n.º 2) usan el idioma para generar la mejor respuesta de una consulta.

## <a name="supporting-multiple-languages"></a>Compatibilidad con varios idiomas

Si necesita admitir un sistema de base de conocimiento que incluye varios idiomas, puede:

* Usar el [servicio Translator](../../translator/translator-info-overview.md) para traducir una pregunta a un solo idioma antes de enviarla a la base de conocimiento. Esto le permite concentrarse en la calidad de un solo idioma, así como en la calidad de las preguntas y respuestas alternativas.
* Crear un recurso de QnA Maker (y una base de conocimiento dentro de ese recurso) para cada idioma. Esta opción le permite administrar preguntas alternativas independientes y textos de respuesta con más matices para cada idioma. De este modo, obtiene mucha más flexibilidad, aunque tiene un costo de mantenimiento mucho mayor cuando las preguntas o respuestas son distintas en todos los idiomas.

Consulte los [idiomas admitidos](../overview/language-support.md) en QnA Maker.

### <a name="support-each-language-with-a-qna-maker-resource"></a>Inclusión de un recurso de QnA Maker para cada idioma

* Cree un recurso de QnA Maker para cada idioma
* Agregue solo archivos y direcciones URL para ese idioma
* Use una convención de nomenclatura para que el recurso identifique el idioma. Por ejemplo, `qna-maker-fr` es para todas las bases de conocimiento de documentos en francés


## <a name="languages-supported"></a>Idiomas admitidos

La siguiente lista contiene los idiomas que admiten los recursos de QnA Maker. 

|Idioma|
|--|
|Árabe|
|Armenio|
|Bengalí|
|Vasco|
|Búlgaro|
|Catalán|
|Chino simplificado|
|Chino tradicional|
|Croata|
|Checo|
|Danés|
|Neerlandés|
|Inglés|
|Estonio|
|Finés|
|Francés|
|Gallego|
|Alemán|
|Griego|
|Gujarati|
|Hebreo|
|Hindi|
|Húngaro|
|Islandés|
|Indonesio|
|Irlandés|
|Italiano|
|Japonés|
|Canarés|
|Coreano|
|Letón|
|Lituano|
|Malayalam|
|Malayo|
|Noruego|
|Polaco|
|Portugués|
|Punjabi|
|Rumano|
|Ruso|
|Serbio cirílico|
|Serbio latín|
|Eslovaco|
|Esloveno|
|Español|
|Sueco|
|Tamil|
|Telugu|
|Tailandés|
|Turco|
|Ucraniano|
|Urdu|
|Vietnamita|

## <a name="query-matching-and-relevance"></a>Coincidencia y relevancia de las consultas
QnA Maker depende de los [analizadores de idioma de Azure Cognitive Search](https://docs.microsoft.com/rest/api/searchservice/language-support) para proporcionar resultados.

Aunque las funcionalidades de Azure Cognitive Search están en el mismo nivel que los idiomas admitidos, QnA Maker tiene un clasificador adicional que está por encima de los resultados de búsqueda de Azure. En este modelo de clasificador, se usan características semánticas y basadas en palabras especiales en los siguientes idiomas.

|Idiomas con un clasificador adicional|
|--|
|Chino|
|Checo|
|Neerlandés|
|Inglés|
|Francés|
|Alemán|
|Húngaro|
|Italiano|
|Japonés|
|Coreano|
|Polaco|
|Portugués|
|Español|
|Sueco|

Esta clasificación adicional es un elemento de trabajo interno del clasificador de QnA Maker.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Selección de idioma](../how-to/language-knowledge-base.md)
