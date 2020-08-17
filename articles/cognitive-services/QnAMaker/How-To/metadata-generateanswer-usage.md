---
title: 'Metadatos con GenerateAnswer API: QnA Maker'
titleSuffix: Azure Cognitive Services
description: QnA Maker permite agregar metadatos, en forma de pares de clave-valor, a los pares de preguntas y respuestas. Puede filtrar los resultados a las consultas de usuario y almacenar la información adicional que se puede usar en las conversaciones de seguimiento.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 07/16/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 7120f95b8b61fc08759f4d15061ec530849dfc05
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406527"
---
# <a name="get-an-answer-with-the-generateanswer-api-and-metadata"></a>Obtención de una respuesta con GenerateAnswer API y metadatos

Para obtener la respuesta prevista a la pregunta de un usuario, use GenerateAnswer API. Al publicar una base de conocimiento, puede consultar información sobre cómo usar esta API en la página **Publicar**. También puede configurar la API para filtrar las respuestas según las etiquetas de metadatos y probar la base de conocimiento desde el punto de conexión con el parámetro de cadena de consulta de prueba.

QnA Maker permite agregar metadatos, en forma de pares clave-valor, a los pares de preguntas y respuestas. Luego puede usar esta información para filtrar los resultados de las consultas de usuario y para almacenar información adicional que se puede usar en las conversaciones de seguimiento. Para más información, consulte la [base de conocimiento](../Concepts/knowledge-base.md).

<a name="qna-entity"></a>

## <a name="store-questions-and-answers-with-a-qna-entity"></a>Almacenamiento de preguntas y respuestas con una entidad QnA

Resulta importante entender cómo QnA Maker almacena los datos de preguntas y respuestas. En la siguiente ilustración se muestra una entidad QnA:

![Ilustración de una entidad QnA](../media/qnamaker-how-to-metadata-usage/qna-entity.png)

Cada entidad QnA tiene un identificador único y persistente. Puede usar el identificador para realizar actualizaciones en una entidad QnA determinada.

<a name="generateanswer-api"></a>

## <a name="get-answer-predictions-with-the-generateanswer-api"></a>Obtención de predicciones de respuestas con GenerateAnswer API

[GenerateAnswer API](https://docs.microsoft.com/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer) se usa en un bot o aplicación para consultar la base de conocimiento con una pregunta de usuario y obtener la mejor coincidencia de los pares de preguntas y respuestas.

<a name="generateanswer-endpoint"></a>

## <a name="publish-to-get-generateanswer-endpoint"></a>Publicación para obtener el punto de conexión de GenerateAnswer

Después de publicar la base de conocimiento, ya sea desde el [portal de QnA Maker](https://www.qnamaker.ai) o mediante la [API](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/publish), puede obtener los detalles de su punto de conexión de GenerateAnswer.

Para obtener los detalles del punto de conexión:
1. Inicie sesión en [https://www.qnamaker.ai](https://www.qnamaker.ai).
1. En **My knowledge bases** (Mis bases de conocimiento), seleccione **View Code** (Ver código) correspondiente a su base de conocimiento.
    ![Captura de pantalla de My knowledge bases (Mis bases de conocimiento)](../media/qnamaker-how-to-metadata-usage/my-knowledge-bases.png)
1. Obtenga los detalles del punto de conexión de GenerateAnswer.

    ![Captura de los detalles del punto de conexión](../media/qnamaker-how-to-metadata-usage/view-code.png)

También puede obtener los detalles del punto de conexión en la pestaña **Settings** (Configuración) de la base de conocimiento.

<a name="generateanswer-request"></a>

## <a name="generateanswer-request-configuration"></a>Configuración de la solicitud GenerateAnswer

Para llamar a GenerateAnswer, se utiliza una solicitud HTTP POST. Para ver código de ejemplo que muestra cómo llamar a GenerateAnswer, consulte las [guías de inicio rápido](../quickstarts/quickstart-sdk.md#generate-an-answer-from-the-knowledge-base).

La solicitud POST usa:

* Los [parámetros de URI](https://docs.microsoft.com/rest/api/cognitiveservices/qnamakerruntime/runtime/train#uri-parameters) obligatorios.
* La propiedad del encabezado obligatoria, `Authorization`, por seguridad.
* Las [propiedades del cuerpo](https://docs.microsoft.com/rest/api/cognitiveservices/qnamakerruntime/runtime/train#feedbackrecorddto) obligatorias.

La dirección URL de GenerateAnswer tiene el formato siguiente:

```
https://{QnA-Maker-endpoint}/knowledgebases/{knowledge-base-ID}/generateAnswer
```

Recuerde establecer la propiedad del encabezado HTTP de `Authorization` con un valor de la cadena `EndpointKey` con un espacio final y, luego, la clave del punto de conexión que se encontró en la página **Configuración**.

Un cuerpo JSON de ejemplo tiene el siguiente aspecto:

```json
{
    "question": "qna maker and luis",
    "top": 6,
    "isTest": true,
    "scoreThreshold": 30,
    "rankerType": "" // values: QuestionOnly
    "strictFilters": [
    {
        "name": "category",
        "value": "api"
    }],
    "userId": "sd53lsY="
}
```

Más información acerca de [rankerType](../concepts/best-practices.md#choosing-ranker-type).

El código JSON anterior solo solicitó aquellas respuestas con una puntuación del 30 % o que están por encima de la puntuación del umbral.

<a name="generateanswer-response"></a>

## <a name="generateanswer-response-properties"></a>Propiedades de la respuesta de GenerateAnswer

La [respuesta](https://docs.microsoft.com/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#successful-query) es un objeto JSON que incluye toda la información necesaria para mostrar la respuesta y el siguiente turno en la conversación, si está disponible.

```json
{
    "answers": [
        {
            "score": 38.54820341616869,
            "Id": 20,
            "answer": "There is no direct integration of LUIS with QnA Maker. But, in your bot code, you can use LUIS and QnA Maker together. [View a sample bot](https://github.com/Microsoft/BotBuilder-CognitiveServices/tree/master/Node/samples/QnAMaker/QnAWithLUIS)",
            "source": "Custom Editorial",
            "questions": [
                "How can I integrate LUIS with QnA Maker?"
            ],
            "metadata": [
                {
                    "name": "category",
                    "value": "api"
                }
            ]
        }
    ]
}
```

El código JSON anterior respondió con una respuesta con una puntuación del 38,5 %.

## <a name="use-qna-maker-with-a-bot-in-c"></a>Uso de QnA Maker con un bot en C#

Bot Framework brinda acceso a las propiedades de QnA Maker con [getAnswer API](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.ai.qna.qnamaker.getanswersasync?view=botbuilder-dotnet-stable#Microsoft_Bot_Builder_AI_QnA_QnAMaker_GetAnswersAsync_Microsoft_Bot_Builder_ITurnContext_Microsoft_Bot_Builder_AI_QnA_QnAMakerOptions_System_Collections_Generic_Dictionary_System_String_System_String__System_Collections_Generic_Dictionary_System_String_System_Double__):

```csharp
using Microsoft.Bot.Builder.AI.QnA;
var metadata = new Microsoft.Bot.Builder.AI.QnA.Metadata();
var qnaOptions = new QnAMakerOptions();

metadata.Name = Constants.MetadataName.Intent;
metadata.Value = topIntent;
qnaOptions.StrictFilters = new Microsoft.Bot.Builder.AI.QnA.Metadata[] { metadata };
qnaOptions.Top = Constants.DefaultTop;
qnaOptions.ScoreThreshold = 0.3F;
var response = await _services.QnAServices[QnAMakerKey].GetAnswersAsync(turnContext, qnaOptions);
```

El código JSON anterior solo solicitó aquellas respuestas con una puntuación del 30 % o que están por encima de la puntuación del umbral.

## <a name="use-qna-maker-with-a-bot-in-nodejs"></a>Uso de QnA Maker con un bot en Node.js

Bot Framework brinda acceso a las propiedades de QnA Maker con [getAnswer API](https://docs.microsoft.com/javascript/api/botbuilder-ai/qnamaker?view=botbuilder-ts-latest#generateanswer-string---undefined--number--number-):

```javascript
const { QnAMaker } = require('botbuilder-ai');
this.qnaMaker = new QnAMaker(endpoint);

// Default QnAMakerOptions
var qnaMakerOptions = {
    ScoreThreshold: 0.30,
    Top: 3
};
var qnaResults = await this.qnaMaker.getAnswers(stepContext.context, qnaMakerOptions);
```

El código JSON anterior solo solicitó aquellas respuestas con una puntuación del 30 % o que están por encima de la puntuación del umbral.

<a name="metadata-example"></a>

## <a name="use-metadata-to-filter-answers-by-custom-metadata-tags"></a>Uso de metadatos para filtrar las respuestas por etiquetas de metadatos personalizadas

Agregar los metadatos le permite filtrar las respuestas según estas etiquetas de metadatos. Agregue la columna de metadatos desde el menú **View Options** (Ver opciones). Para agregar los metadatos a la base de conocimiento, seleccione el icono **+** de los metadatos para agregar un par de metadatos. Este par consta de una clave y un valor.

![Captura de pantalla de incorporación de metadatos](../media/qnamaker-how-to-metadata-usage/add-metadata.png)

<a name="filter-results-with-strictfilters-for-metadata-tags"></a>

## <a name="filter-results-with-strictfilters-for-metadata-tags"></a>Filtrar los resultados con strictFilters para las etiquetas de metadatos

Considere la pregunta del usuario "¿Cuándo cierra este hotel?", donde la intención está implícita para el restaurante "Paradise".

Puesto que los resultados solo son necesarios para el restaurante "Paradise", puede establecer un filtro en la llamada GenerateAnswer para los metadatos "Nombre del restaurante". Esto se muestra en el ejemplo siguiente:

```json
{
    "question": "When does this hotel close?",
    "top": 1,
    "strictFilters": [ { "name": "restaurant", "value": "paradise"}]
}
```

### <a name="logical-and-by-default"></a>AND lógico de manera predeterminada

Para combinar varios filtros de metadatos en la consulta, agregue los filtros de metadatos adicionales a la matriz de la propiedad `strictFilters`. De manera predeterminada, los valores se combinan lógicamente (AND). Una combinación lógica requiere que todos los filtros coincidan con los pares PyR para que se devuelva el par en la respuesta.

Esto es equivalente a utilizar la propiedad `strictFiltersCompoundOperationType` con el valor de `AND`.

### <a name="logical-or-using-strictfilterscompoundoperationtype-property"></a>OR lógico mediante la propiedad strictFiltersCompoundOperationType

Al combinar varios filtros de metadatos, si solo le preocupa uno o algunos de los filtros que coinciden, use la propiedad `strictFiltersCompoundOperationType` con el valor de `OR`.

Esto permite que knowledge base devuelva respuestas cuando algún filtro coincida, pero no devolverá respuestas que no tengan metadatos.

```json
{
    "question": "When do facilities in this hotel close?",
    "top": 1,
    "strictFilters": [
      { "name": "type","value": "restaurant"},
      { "name": "type", "value": "bar"},
      { "name": "type", "value": "poolbar"}
    ],
    "strictFiltersCompoundOperationType": "OR"
}
```

### <a name="metadata-examples-in-quickstarts"></a>Ejemplos de metadatos en los inicios rápidos

Obtenga más información sobre los metadatos en el inicio rápido del portal de QnA Maker sobre metadatos:
* [Creación: adición de metadatos a un par PyR](../quickstarts/add-question-metadata-portal.md#add-metadata-to-filter-the-answers)
* [Predicción de consultas: filtrar respuestas por metadatos](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md)

<a name="keep-context"></a>

## <a name="use-question-and-answer-results-to-keep-conversation-context"></a>Uso de los resultados de las preguntas y respuestas para mantener el contexto de la conversación

La respuesta a GenerateAnswer contiene la información de los metadatos correspondientes al par de pregunta y respuesta coincidente. Puede usar esta información en la aplicación cliente para almacenar el contexto de la conversación anterior para su uso en conversiones posteriores.

```json
{
    "answers": [
        {
            "questions": [
                "What is the closing time?"
            ],
            "answer": "10.30 PM",
            "score": 100,
            "id": 1,
            "source": "Editorial",
            "metadata": [
                {
                    "name": "restaurant",
                    "value": "paradise"
                },
                {
                    "name": "location",
                    "value": "secunderabad"
                }
            ]
        }
    ]
}
```

## <a name="match-questions-only-by-text"></a>Coincidencia solo de preguntas, por texto

De manera predeterminada, QnA Maker busca a través de preguntas y respuestas. Si quiere buscar solo en las preguntas, para generar una respuesta, utilice el objeto `RankerType=QuestionOnly` del cuerpo de POST de la solicitud GenerateAnswer.

Puede buscar en la base de conocimiento publicada, con `isTest=false`, o bien en la base de conocimiento de prueba, con `isTest=true`.

```json
{
  "question": "Hi",
  "top": 30,
  "isTest": true,
  "RankerType":"QuestionOnly"
}
```

## <a name="common-http-errors"></a>Errores HTTP comunes

|Código|Explicación|
|:--|--|
|2xx|Correcto|
|400|Los parámetros de la solicitud no son correctos, lo que significa que faltan parámetros necesarios, o bien que tienen un formato incorrecto o son demasiado grandes|
|400|El cuerpo de la solicitud es incorrecto, lo que significa que falta el código JSON, o bien que tiene un formato incorrecto o es demasiado grande|
|401|Clave no válida|
|403|Prohibido: no tiene los permisos correctos|
|404|KB no existe|
|410|Esta API está en desuso y ya no está disponible|

## <a name="next-steps"></a>Pasos siguientes

La página **Publicar** también proporciona información para [generar una respuesta](../Quickstarts/get-answer-from-knowledge-base-using-url-tool.md) con Postman o cURL.

> [!div class="nextstepaction"]
> [Obtener análisis en la base de conocimiento](../how-to/get-analytics-knowledge-base.md)
