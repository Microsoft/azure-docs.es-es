---
title: 'Procedimientos recomendados: QnA Maker'
titlesuffix: Azure Cognitive Services
description: Siga estos procedimientos recomendados para mejorar la base de conocimiento y ofrecer mejores resultados a los usuarios finales del bot de chat o de la aplicación.
services: cognitive-services
author: tulasim88
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: article
ms.date: 05/10/2019
ms.author: tulasim
ms.custom: seodec18
ms.openlocfilehash: 2677c993b759988b0a9906b357bcd352b243b5a7
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65792677"
---
# <a name="best-practices-of-a-qna-maker-knowledge-base"></a>Procedimientos recomendados para una base de conocimiento de QnA Maker
El artículo sobre el [ciclo de vida de desarrollo de una base de conocimiento](../Concepts/development-lifecycle-knowledge-base.md) indica cómo administrar una base de conocimiento de principio a fin. Siga estos procedimientos recomendados para mejorar la base de conocimiento y ofrecer mejores resultados a los usuarios finales del bot de chat o de la aplicación.

## <a name="extraction"></a>Extracción
El servicio QnA Maker mejora continuamente los algoritmos que extraen preguntas y respuestas del contenido y amplía la lista de formatos HTML y de archivos que admite. Siga las [directrices](../Concepts/data-sources-supported.md) para la extracción de datos según su tipo de documento. 

En general, las páginas de preguntas más frecuentes deben ser independientes y no combinarse con otra información. Los manuales de productos deben tener encabezados claros y, preferiblemente, una página de índice. 

## <a name="creating-good-questions-and-answers"></a>Creación de buenas preguntas y respuestas

### <a name="good-questions"></a>Buenas preguntas

Las mejores preguntas son simples. Identifique la palabra o frase clave de cada pregunta y formule una pregunta sencilla sobre ella. 

Agregue tantas preguntas alternativas como necesite, pero mantenga las variantes simples. Agregar más palabras o expresiones que no forman parte del objetivo principal de la pregunta no ayuda a que QnA Maker encuentre una coincidencia. 

### <a name="good-answers"></a>Buenas respuestas

Las mejores respuestas son sencillas, pero no tanto como Sí y No. Si la respuesta debe remitir a otros orígenes o incluir una experiencia enriquecida con vínculos y multimedia, use [etiquetado](../how-to/metadata-generateanswer-usage.md) para distinguir qué tipo de respuesta espera, y envíe la etiqueta con la consulta para obtener la versión correcta de la respuesta.

## <a name="chit-chat"></a>Charla
Agregue una charla a su bot para que sea más conversacional y atractivo fácilmente. Fácilmente puede agregar conjuntos de datos chit chat desde personalidades predefinidos al crear su KB y cambiarlos en cualquier momento. Obtenga información sobre cómo [agregar una charla a la base de conocimiento](../How-To/chit-chat-knowledge-base.md). 

### <a name="choosing-a-personality"></a>Selección de una personalidad
Chit chat es compatible con varios personalidades predefinidos: 

|Personalidad |Archivo de conjunto de datos de QnA Maker |
|---------|-----|
|Profesional |[qna_chitchat_professional.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_professional.tsv) |
|Descriptiva |[qna_chitchat_friendly.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_friendly.tsv) |
|Astuto |[qna_chitchat_witty.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_witty.tsv) |
|Preocuparse |[qna_chitchat_caring.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_caring.tsv) |
|Entusiasta |[qna_chitchat_enthusiastic.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_enthusiastic.tsv) |

El rango de respuestas va de formal a informal e irreverente. Debe seleccionar la personalidad que sea más acorde con el tono que quiera para el bot. Puede ver el [conjuntos de datos](https://github.com/Microsoft/BotBuilder-PersonalityChat/tree/master/CSharp/Datasets), elija uno que actúe como base para el bot y, a continuación, personalizar las respuestas. 

### <a name="edit-bot-specific-questions"></a>Edición de preguntas específicas de bot
Hay algunas preguntas específicas de bot que forman parte del conjunto de datos de charla y que se han rellenado con respuestas genéricas. Cambie estas respuestas para reflejar mejor los detalles del bot. 

Se recomienda hacer que las siguientes preguntas y respuestas de charla sean más específicas:

* ¿Quién es usted?
* ¿Qué puede hacer?
* Es necesario indicar la edad
* ¿Quién le ha creado?
* Hola
   

## <a name="rankingscoring"></a>Clasificación y puntuación
Asegúrese de aprovechar al máximo las características de clasificación que QnA Maker admite. Eso aumentará las probabilidades de que una determinada consulta de usuario se responda con una respuesta correcta.

### <a name="choosing-a-threshold"></a>Selección de un umbral
La puntuación de confianza predeterminada que se usa como umbral es de 50, pero puede cambiarla para su base de conocimiento según sus necesidades. Dado que cada base de conocimiento es diferente, conviene probar y elegir el umbral que mejor se adapte a cada caso. Obtenga más información sobre la [puntuación de confianza](../Concepts/confidence-score.md). 

### <a name="add-alternate-questions"></a>Agregar preguntas alternativas
[Alternar las preguntas](../How-To/edit-knowledge-base.md) aumenta las probabilidades de encontrar una coincidencia con una consulta de usuario. Las preguntas alternativas son útiles cuando hay varias maneras de plantear la misma pregunta. Esto puede incluir cambios en la estructura de la frase y el estilo de redacción.

|Consulta original|Consultas alternativas|Cambio| 
|--|--|--|
|¿Hay estacionamiento disponible?|¿Dispone de estacionamiento?|estructura de la frase|
 |Hola|Buenas<br>¿Cómo estamos?|estilo de redacción o jerga|

<a name="use-metadata-filters"></a>

### <a name="use-metadata-tags-to-filter-questions-and-answers"></a>Uso de etiquetas de metadatos para el filtrado de preguntas y respuestas

Los [metadatos](../How-To/edit-knowledge-base.md) permiten usar filtros para reducir los resultados de una consulta de usuario en función de etiquetas de metadatos. La respuesta de la base de conocimiento puede diferir según la etiqueta de metadatos, aunque la consulta sea la misma. Por ejemplo, *¿Dónde está ubicado el estacionamiento?* puede tener otra respuesta si la ubicación de la sucursal del restaurante es diferente; es decir, los metadatos son *ubicación: Seattle* frente a *ubicación: Redmond*.

### <a name="use-synonyms"></a>Usar sinónimos
Aunque hay cierta compatibilidad con los sinónimos en inglés, use [alteraciones de palabras](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/alterations/replace) que no distinguen mayúsculas de minúsculas para agregar sinónimos a las palabras clave que tengan una forma diferente. Los sinónimos deben agregarse en el nivel del servicio QnA Maker y todas las bases de conocimiento del servicio deben compartirlos.

|Palabra original|Sinónimos|
|--|--|
|comprar|compra<br>banca por internet<br>banca electrónica|

### <a name="use-distinct-words-to-differentiate-questions"></a>Usar diferentes palabras para diferenciar preguntas
Los algoritmos de clasificación de QnA Maker que buscan la correspondencia entre la consulta de un usuario y una pregunta de la base de conocimiento funcionan mejor si cada pregunta aborda una necesidad distinta. La repetición del mismo conjunto de palabras en diferentes preguntas reduce la probabilidad de que se seleccione la respuesta correcta para una consulta de usuario determinado con esas palabras. 

Por ejemplo, puede tener dos preguntas y respuestas independientes con las siguientes preguntas:

|Preguntas y respuestas|
|--|
|dónde está *ubicado* el estacionamiento|
|dónde está *ubicado* el cajero automático|

Puesto que estas dos preguntas y respuestas están expresadas con palabras muy similares, esta similitud podría provocar que las puntuaciones sean muy parecidas para muchas de las consultas de usuario que están expresadas como *dónde está ubicado `<x>`*. Como alternativa, intente diferenciarlas claramente con consultas como *"dónde está el estacionamiento"* y *"dónde está el cajero automático"*, evitando palabras como "ubicado", que pueden estar en muchas preguntas en la base de conocimiento. 

## <a name="collaborate"></a>Colaborar
QnA Maker permite a los usuarios [colaborar](../How-to/collaborate-knowledge-base.md) en una base de conocimiento. Los usuarios necesitan acceso al grupo de recursos de QnA Maker en Azure para tener acceso a las bases de conocimiento. Algunas organizaciones quizás quieran externalizar la edición y el mantenimiento de la base de conocimiento, sin dejar de proteger el acceso a sus recursos de Azure. Para realizar este modelo de editor-aprobador, se configuran dos [servicios QnA Maker](../How-to/set-up-qnamaker-service-azure.md) idénticos en suscripciones distintas y seleccionar uno para el ciclo de pruebas de edición. Una vez finalizada la prueba, se transfiere el contenido de la base de conocimiento con un proceso de [importación y exportación](../Tutorials/migrate-knowledge-base.md) al servicio QnA Maker del aprobador que, finalmente, publicará la base de conocimiento y actualizará el punto de conexión.

## <a name="active-learning"></a>Aprendizaje activo

[Aprendizaje activo](../How-to/improve-knowledge-base.md) ofrece el mejor resultado al sugerir preguntas alternativas cuando tiene una amplia variedad de calidad y cantidad de consultas basadas en el usuario. Es importante permitir que las consultas de usuario de las aplicaciones cliente participen en el bucle de comentarios de aprendizaje activo sin censura. Una vez que se sugieren preguntas en el portal de QnA Maker, puede **[filtrar por sugerencias](../How-To/improve-knowledge-base.md#add-active-learning-suggestion-to-knowledge-base)** , a continuación, revise y acepte o rechace las sugerencias. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Editar una base de conocimiento](../How-to/edit-knowledge-base.md)
