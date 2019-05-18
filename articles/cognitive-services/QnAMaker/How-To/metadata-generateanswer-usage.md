---
title: 'Metadatos con GenerateAnswer API: QnA Maker'
titleSuffix: Azure Cognitive Services
description: QnA Maker permite agregar metadatos, en forma de pares de clave-valor, a los conjuntos de preguntas y respuestas. Esta información se puede usar para filtrar los resultados de las consultas de usuario y para almacenar información adicional que se puede usar en las conversaciones de seguimiento.
services: cognitive-services
author: tulasim88
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: article
ms.date: 05/10/2019
ms.author: tulasim
ms.openlocfilehash: 2454e07e4fc4600f846acc7afbcc19cc0b677450
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65792233"
---
# <a name="get-a-knowledge-answer-with-the-generateanswer-api-and-metadata"></a>Obtener una respuesta de conocimiento con la API de GenerateAnswer y metadatos

Para obtener la respuesta prevista a la pregunta de un usuario, use la API GenerateAnswer. Al publicar una base de conocimiento, esta información para usar esta API se muestra en la página de publicación. También puede configurar la API para filtrar las respuestas basadas en etiquetas de metadatos y probar la base de conocimiento desde el punto de conexión con el parámetro de cadena de consulta de prueba.

QnA Maker permite agregar metadatos, en forma de pares de clave-valor, a los conjuntos de preguntas y respuestas. Esta información se puede usar para filtrar los resultados de las consultas de usuario y para almacenar información adicional que se puede usar en las conversaciones de seguimiento. Para más información, consulte la [base de conocimiento](../Concepts/knowledge-base.md).

<a name="qna-entity"></a>

## <a name="storing-questions-and-answers-with-a-qna-entity"></a>Almacenar las preguntas y respuestas con una entidad de QnA

En primer lugar, es importante comprender cómo QnA Maker almacena los datos de preguntas y respuestas. En la siguiente ilustración se muestra una entidad QnA:

![Entidad QnA](../media/qnamaker-how-to-metadata-usage/qna-entity.png)

Cada entidad QnA tiene un identificador único y persistente. El identificador puede utilizarse para realizar actualizaciones en una entidad QnA determinada.

<a name="generateanswer-api"></a>

## <a name="get-answer-predictions-with-the-generateanswer-api"></a>Obtener predicciones de respuesta con la API GenerateAnswer

GenerateAnswer API se usa en un bot o aplicación para consultar la base de conocimiento con una pregunta de usuario y obtener la mejor coincidencia de los conjuntos de preguntas y respuestas.

<a name="generateanswer-endpoint"></a>

## <a name="publish-to-get-generateanswer-endpoint"></a>Publicar para obtener el punto de conexión GenerateAnswer 

Después de publicar la base de conocimiento, ya sea desde el [portal de QnA Maker](https://www.qnamaker.ai) o mediante la [API](https://go.microsoft.com/fwlink/?linkid=2092179), puede obtener los detalles de su punto de conexión de GenerateAnswer.

Para obtener los detalles del punto de conexión:
1. Inicie sesión en [https://www.qnamaker.ai](https://www.qnamaker.ai).
1. En **My knowledge bases** (Mis bases de conocimiento), haga clic en la opción **View Code** (Ver código) correspondiente a su base de conocimiento.
    ![mis bases de conocimiento](../media/qnamaker-how-to-metadata-usage/my-knowledge-bases.png)
1. Obtenga los detalles del punto de conexión de GenerateAnswer.

    ![detalles del punto de conexión](../media/qnamaker-how-to-metadata-usage/view-code.png)

También puede obtener los detalles del punto de conexión en la pestaña **Settings** (Configuración) de la base de conocimiento.

<a name="generateanswer-request"></a>

## <a name="generateanswer-request-configuration"></a>Configuración de la solicitud GenerateAnswer

Para llamar a GenerateAnswer, se utiliza una solicitud HTTP POST. Para ver código de ejemplo que muestra cómo llamar a GenerateAnswer, consulte las [guías de inicio rápido](../quickstarts/csharp.md).

El **URL de solicitud** tiene el formato siguiente: 

```
https://{QnA-Maker-endpoint}/knowledgebases/{knowledge-base-ID}/generateAnswer
```

|Propiedad de la solicitud HTTP|NOMBRE|Type|Propósito|
|--|--|--|--|
|Parámetro de ruta de dirección URL|Id. de base de conocimiento|string|GUID de Knowledge Base.|
|Parámetro de ruta de dirección URL|Host del punto de conexión de QnAMaker|string|nombre de host del punto de conexión implementado en la suscripción de Azure. Esto está disponible en la página de configuración después de publicar la base de conocimiento. |
|Encabezado|Content-Type|string|tipo de soporte del cuerpo enviado a la API. Valor predeterminado es: ''|
|Encabezado|Autorización|string|su clave de punto de conexión (EndpointKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx).|
|Cuerpo de POST|Objeto JSON|JSON|La pregunta con la configuración|


El cuerpo JSON tiene varias opciones:

|Propiedad de cuerpo JSON|Obligatorio|Type|Propósito|
|--|--|--|--|
|`question`|requerido|string|Una pregunta de usuario se envíen a la base de conocimiento.|
|`top`|opcional|integer|número de resultados clasificados por orden de prioridad que se incluirá en la salida. El valor predeterminado es 1.|
|`userId`|opcional|string|identificador único que identifica el usuario. Este identificador se registrará en los registros de chat.|
|`scoreThreshold`|opcional|integer|Se devolverá sólo las respuestas con la puntuación de confianza por encima del umbral. El valor predeterminado es 0.|
|`isTest`|opcional|boolean|Si establece en true, devuelve los resultados desde `testkb` índice de búsqueda en lugar de índice publicado.|
|`strictFilters`|opcional|string|si se especifica, indica a QnA Maker que solo devuelva las respuestas que tienen los metadatos especificados. Use `none` para indicar la respuesta no debe tener ningún filtro de metadatos. |

Un cuerpo JSON de ejemplo tiene el siguiente aspecto:

```json
{
    "question": "qna maker and luis",
    "top": 6,
    "isTest": true,
    "scoreThreshold": 20,
    "strictFilters": [
    {
        "name": "category",
        "value": "api"
    }],
    "userId": "sd53lsY="
}
```

<a name="generateanswer-response"></a>

## <a name="generateanswer-response-properties"></a>Propiedades de la respuesta GenerateAnswer

Una respuesta correcta devuelve el estado de 200 y una respuesta JSON. 

|Propiedad de respuestas (se ordenan por puntuación)|Propósito|
|--|--|
|de la aplicación|puntuación de clasificación entre 0 y 100.|
|Identificador|identificador único asignado a la respuesta.|
|Preguntas|preguntas proporcionadas por el usuario.|
|answer (responder)|la respuesta a la pregunta.|
|source|nombre del origen desde el que se extrajo la respuesta o se guardó en la base de conocimiento.|
|metadatos|metadatos asociados a la respuesta.|
|metadata.name|nombre de los metadatos. (cadena, longitud máx.: 100, requerido)|
|metadata.value: valor de los metadatos. (cadena, longitud máx.: 100, requerido)|


```json
{
    "answers": [
        {
            "score": 28.54820341616869,
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

<a name="metadata-example"></a>

## <a name="using-metadata-allows-you-to-filter-answers-by-custom-metadata-tags"></a>Uso de los metadatos le permite filtrar las respuestas por etiquetas de metadatos personalizados

Adición de metadatos permite filtrar las respuestas por estas etiquetas de metadatos. Agregar la columna de metadatos desde el **opciones de vista** menú. Agregar metadatos a la base de conocimiento, haga clic en los metadatos **+** icono para agregar un par de metadatos. Este par consta de una clave y un valor.

![agregar metadatos](../media/qnamaker-how-to-metadata-usage/add-metadata.png)

<a name="filter-results-with-strictfilters-for-metadata-tags"></a>

## <a name="filter-results-with-strictfilters-for-metadata-tags"></a>Filtrar los resultados con strictFilters para las etiquetas de metadatos

Tomemos la pregunta del usuario "¿Cuándo cierra este hotel?" donde la intención es implícita para el restaurante "Paradise".

Puesto que los resultados solo son necesarios para el restaurante "Paradise", puede establecer un filtro en la llamada GenerateAnswer para los metadatos "Nombre del restaurante", tal y como se indica a continuación.

```json
{
    "question": "When does this hotel close?",
    "top": 1,
    "strictFilters": [
      {
        "name": "restaurant",
        "value": "paradise"
      }]
}
```

<name="keep-context"></a>

## <a name="use-question-and-answer-results-to-keep-conversation-context"></a>Use resultados de preguntas y respuestas para mantener el contexto de la conversación

La respuesta a la GenerateAnswer contiene la información de metadatos correspondiente del conjunto de preguntas/respuestas coincidente. Esta información puede utilizarse en la aplicación cliente para almacenar el contexto de la conversación anterior para su uso en las conversaciones más adelante. 

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

## <a name="next-steps"></a>Pasos siguientes

La página de publicación también proporciona información para generar una respuesta [Postman](../Quickstarts/get-answer-from-kb-using-postman.md) y [cURL](../Quickstarts/get-answer-from-kb-using-curl.md). 

> [!div class="nextstepaction"]
> [Creación de una base de conocimientos](./create-knowledge-base.md)
