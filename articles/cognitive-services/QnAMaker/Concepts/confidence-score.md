---
title: 'Puntuación de confianza: QnA Maker'
titleSuffix: Azure Cognitive Services
description: Esta puntuación de confianza indica el grado de confianza que se tiene en que la respuesta sea la coincidencia correcta para la consulta de usuario en cuestión.
services: cognitive-services
author: tulasim88
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: article
ms.date: 04/05/2019
ms.author: tulasim
ms.custom: seodec18
ms.openlocfilehash: 4fb5d1e20c4c857dedcec2dc4695f82fccd9269d
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65792750"
---
# <a name="confidence-score-of-a-qna-maker-knowledge-base"></a>Puntuación de confianza de una base de conocimiento de QnA Maker
Cuando una consulta de usuario se compara con una base de conocimientos (KB), QnA Maker devuelve respuestas pertinentes, junto con una puntuación de confianza. Esta puntuación indica el grado de confianza que se tiene en que la respuesta sea la coincidencia correcta para la consulta de usuario en cuestión. 

La puntuación de confianza es un número entre 0 y 100. Una puntuación de 100 es probablemente una coincidencia exacta, mientras que una puntuación de 0 significa que no se ha encontrado ninguna respuesta coincidente. Cuanto mayor sea la puntuación, mayor será la confianza en la respuesta. Para una consulta determinada, podrían devolverse varias respuestas. En ese caso, las respuestas se devuelven en orden de mayor a menor puntuación de confianza.

En el ejemplo siguiente, puede ver una entidad QnA con dos preguntas. 


![Ejemplo de un par de QnA](../media/qnamaker-concepts-confidencescore/ranker-example-qna.png)

En el ejemplo anterior, se pueden esperar puntuaciones como el intervalo de puntuación que se muestra a continuación para los distintos tipos de consultas de usuario:


![Intervalo de puntuación del clasificador](../media/qnamaker-concepts-confidencescore/ranker-score-range.png)


En la tabla siguiente se indica la confianza típico asociada a un resultado determinado.

|Valor de la puntuación|Significado de la puntuación|Consulta de ejemplo|
|--|--|--|
|90-100|Coincidencia casi exacta de la consulta del usuario y una pregunta de KB.|"Mis cambios no se actualizan en KB después de la publicación".|
|>70|Confianza alta. Normalmente, una buena respuesta que responde por completo a la consulta del usuario.|"He publicado mi KB pero no se actualiza".|
|50-70|Confianza media. Normalmente, una respuesta bastante buena que debería responder el propósito principal de la consulta del usuario.|"¿Debo guardar mis actualizaciones antes de publicar mi KB?"|
|30-50|Confianza baja. Normalmente, una respuesta relacionada que responde en parte a la intención del usuario.|"¿Qué hace Guardar y entrenar?"|
|<30|Confianza muy baja. Normalmente, no responde a la consulta del usuario, pero contiene algunas palabras o frases coincidentes. |"¿Donde puedo agregar sinónimos a mi KB?"|
|0|Ninguna coincidencia, por lo que no se devuelve una respuesta.|"¿Cuánto cuesta el servicio?"|

## <a name="choose-a-score-threshold"></a>Seleccionar un umbral de puntuación
En la tabla anterior se muestran las puntuaciones que se pueden esperar en la mayoría de KB. Aun así, como cada KB es diferente y contiene tipos de palabras, intenciones y objetivos distintos, se recomienda que pruebe y seleccione el umbral que mejor funcione en su caso. De forma predeterminada el umbral se establece en 0, por lo que se devuelven todas las respuestas posibles. Es el umbral recomendado que debería funcionar para la mayoría de artículos de Knowledge Base, **50**.

Al elegir el umbral, tenga en cuenta el equilibrio entre los valores de Precisión y Cobertura, y ajuste el umbral según sus requisitos.

- Si la **Precisión** es más importante en su caso, aumente el umbral. De este modo, cada vez que se devuelva una respuesta, el resultado será mucho más confiable y probablemente responderá a lo que buscan los usuarios. En este caso, podrían quedar más preguntas sin responder. *Por ejemplo:* si establece el umbral en **70**, podría perder la oportunidad de responder a algunos ejemplos ambiguos, como "¿Qué es Guardar y entrenar?".

- Si la **Cobertura** (o coincidencia) es más importante y quiere responder a tantas preguntas como sea posible, incluso aunque solo haya una relación parcial con la pregunta del usuario, reduzca el umbral. Esto significa que podría haber más casos en los que el resultado no responde a la consulta en sí del usuario, pero proporciona otra respuesta en cierto modo relacionada. *Por ejemplo:* si realiza el umbral **30**, podría dar respuestas para las consultas como "¿Dónde puedo editar mi KB?"

> [!NOTE]
> Las versiones más recientes de QnA Maker incluyen mejoras en la lógica de puntuación, lo que podría afectar al umbral. Siempre que actualice el servicio, no olvide probar y ajustar el umbral, en caso necesario. Puede comprobar la versión del servicio QnA [aquí](https://www.qnamaker.ai/UserSettings)y obtener información acerca de cómo obtener las actualizaciones más recientes [aquí](../How-To/troubleshooting-runtime.md).

## <a name="improve-confidence-scores"></a>Mejorar las puntuaciones de confianza
Para mejorar la puntuación de confianza de una respuesta concreta a una consulta de usuario, puede agregar la consulta del usuario a la base de conocimiento como una pregunta alternativa en la respuesta. También puede usar [alteraciones de palabras](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/alterations/replace) que no distinguen mayúsculas de minúsculas para agregar sinónimos a las palabras clave en la KB.


## <a name="similar-confidence-scores"></a>Puntuaciones de confianza similares
Si varias respuestas tienen una puntuación de confianza similar, es probable que la consulta fuera demasiado genérica y, por tanto, coincidiera con la misma probabilidad con varias respuestas. Intente estructurar mejor sus preguntas y respuestas para que todas las entidades QnA tengan una intención distinta.


## <a name="confidence-score-differences"></a>Diferencias en las puntuaciones de confianza
La puntuación de confianza de una respuesta puede cambiar de manera apenas perceptible entre la prueba y la versión publicada de la base de conocimiento, aunque el contenido sea el mismo. Esto se debe a que el contenido de la prueba y la base de conocimiento publicada se encuentran en distintos índices de Azure Search. Cuando se publica una base de conocimiento, el contenido de preguntas y respuestas de la base de conocimiento se mueve desde el índice de prueba a un índice de producción en Azure Search. Vea cómo se lleva a cabo la operación de [publicación](../Quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base).

Si tiene una base de conocimiento en regiones diferentes, cada región usa su propio índice de Azure Search. Dado que se usan distintos índices, las puntuaciones no serán las mismas exactamente. 


## <a name="no-match-found"></a>No se encontraron coincidencias
Si el clasificador no encuentra ninguna buena coincidencia, se devuelve una puntuación de confianza de 0,0 o "None" (Ninguna) y la respuesta predeterminada no es "No good match found in the KB" (No se encontró ninguna buena coincidencia en KB)". Puede invalidar esta [respuesta predeterminada](#change-default-answer) en el código de aplicación o bot llamando el punto de conexión. Como alternativa, también puede establecer la respuesta de invalidación en Azure, lo que cambiará el valor predeterminado para todas las knowledge base implementada en un determinado servicio QnA Maker.

## <a name="change-default-answer"></a>Cambio de la respuesta predeterminada

1. Vaya a [Azure Portal](https://portal.azure.com) y navegue hasta el grupo de recursos que representa el servicio QnA Maker que creó.

2. Haga clic para abrir **App Service**.

    ![En Azure Portal, acceda a App Service para QnA Maker](../media/qnamaker-concepts-confidencescore/set-default-response.png)

3. Haga clic en **Configuración de la aplicación** y edite el campo **DefaultAnswer** a la respuesta predeterminada deseada. Haga clic en **Save**(Guardar).

    ![Seleccione Configuración de la aplicación y, a continuación, edite el valor de DefaultAnswer para QnA Maker](../media/qnamaker-concepts-confidencescore/change-response.png)

4. Reiniciar App Service

    ![Después de cambiar DefaultAnswer, reinicie la instancia de App Service de QnA Maker](../media/qnamaker-faq/qnamaker-appservice-restart.png)


## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Orígenes de datos admitidos](./data-sources-supported.md)

