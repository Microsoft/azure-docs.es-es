---
title: archivo de inclusión
description: archivo de inclusión
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.custom: include file
ms.date: 04/16/2020
ms.openlocfilehash: ac0192da3afa76c3c21056f218f2b249c44d1c36
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87133916"
---
Cognitive Services proporciona dos servicios de procesamiento de lenguaje natural, [Language Understanding](../luis/what-is-luis.md) y [QnA Maker](../qnamaker/overview/overview.md), cada uno con un propósito diferente. Sepa cuándo usar cada servicio y cómo se complementan entre sí.

El procesamiento de lenguaje natural (NLP) permite que la aplicación cliente, como un bot de chat, trabaje con los usuarios mediante el lenguaje natural. Un usuario escribe una frase. El texto del usuario puede tener una gramática, una ortografía y una puntuación deficientes. Cognitive Services puede trabajar con la frase del usuario de todas formas y devuelve la información que el bot de chat necesita para ayudar al usuario.

## <a name="cognitive-services-with-nlp"></a>Cognitive Services con NLP

Language Understanding (LUIS) y QnA Maker proporcionan NLP. La aplicación cliente envía texto en lenguaje natural. El servicio toma el texto, lo procesa y devuelve un resultado.

## <a name="when-to-use-each-service"></a>Cuándo usar cada servicio

Language Understanding (LUIS) y QnA Maker resuelven distintos problemas. LUIS determina la intención del texto de un usuario (lo que se conoce como una expresión), mientras que QnA Maker determina la respuesta al texto de un usuario (lo que se conoce como consulta).

Para elegir el servicio correcto, debe comprender el texto de usuario que procede de la aplicación cliente y la información que la aplicación cliente necesita obtener de Cognitive Services.

Si el bot de chat recibe el texto `How do I get to the Human Resources building on the Seattle North campus?`, use el siguiente gráfico para comprender cómo funciona cada servicio con el texto.

|Servicio|La aplicación cliente determina|
|--|--|
|LUIS|**Determina la intención del usuario** del texto: el servicio no devuelve la respuesta a la pregunta. Por ejemplo, este texto se clasifica como coincidencia con el intento de `FindLocation`.<br>|
|QnA Maker|**Devuelve la respuesta a la pregunta** desde una base de conocimiento personalizada. Por ejemplo, este texto se determina como una pregunta con la respuesta de texto estático `Get on the #9 bus and get off at Franklin street`.|
|||

> [!div class="mx-imgBorder"]
> ![Infografía para determinar cuándo se debe usar LUIS y cuándo utilizar QnA Maker](./luis-qna-maker-together-decision.png)

## <a name="when-do-you-use-luis"></a>¿Cuándo se usa LUIS?

Use LUIS cuando necesite conocer la intención de la expresión como parte de un proceso en el bot de chat. Siguiendo con el texto de ejemplo, `How do I get to the Human Resources building on the Seattle North campus?`, una vez que sepa que la intención del usuario es encontrar una ubicación, puede pasar detalles sobre la expresión (extraída con las entidades) a otro servicio, como un servidor de transporte, para obtener la respuesta.

No es necesario combinar LUIS y QnA Maker para determinar la intención.

Puede combinar los dos servicios de esta expresión, si el bot de chat necesita procesar el texto en función de las intenciones y las entidades (mediante LUIS), así como buscar la respuesta de texto estático específica (mediante QnA Maker).

## <a name="when-do-you-use-qna-maker"></a>¿Cuándo se usa QnA Maker?

Utilice QnA Maker cuando tenga una base de conocimiento estática de respuestas. Esta base de conocimiento está personalizada para sus necesidades, que ha creado con documentos como archivos PDF y direcciones URL.

Siguiendo con la expresión del ejemplo, `How do I get to the Human Resources building on the Seattle North campus?`, envíe el texto, por ejemplo, una consulta, al servicio de QnA Maker publicado y reciba la mejor respuesta.

No es necesario combinar LUIS y QnA Maker para determinar la respuesta a la pregunta.

Puede combinar los dos servicios para esta expresión, si el bot de chat necesita procesar el texto en función de las intenciones y las entidades (mediante LUIS), así como buscar la respuesta (mediante QnA Maker).

## <a name="use-both-services-when-your-knowledge-base-is-incomplete"></a>Uso de ambos servicios cuando la base de conocimiento esté incompleta

Si va a crear la base de conocimiento de QnA Maker, pero sabe que el dominio de asunto está cambiando (por ejemplo, información oportuna), puede combinar los servicios de LUIS y QnA Maker. De este modo, puede usar la información de la base de conocimiento, pero también puede usar LUIS para determinar la intención de un usuario. Una vez que la aplicación cliente tiene la intención, puede solicitar la información pertinente de otro origen.

La aplicación cliente necesitaría supervisar tanto las respuestas de LUIS como las de QnA Maker para las puntuaciones. Si la puntuación de QnA Maker está por debajo de un umbral arbitrario, use la información de intención y de entidad que LUIS devuelve para pasar la información a un servicio de terceros.

Siguiendo con el texto de ejemplo, `How do I get to the Human Resources building on the Seattle North campus?`, supongamos que QnA Maker devuelve una puntuación de confianza baja. Use la intención devuelta por LUIS `FindLocation` y cualquier entidad extraída, como `Human Resources building` y `Seattle North campus`, para enviar esta información a una asignación o servicio de búsqueda para otra respuesta.

Puede presentar esta respuesta de terceros al usuario para la que la valide. Una vez que tenga la aprobación del usuario, puede volver a QnA Maker para agregar la información y aumentar su conocimiento.

## <a name="use-both-services-when-your-chat-bot-needs-more-information"></a>Uso de ambos servicios cuando el bot de chat necesite más información

Si el bot de chat necesita más información de la que proporciona el servicio, para continuar con un árbol de decisión, use ambos servicios y procese ambas respuestas en la aplicación cliente.

Use la herramienta de la **[CLI de Dispatch](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)** de la plataforma de bot para ayudar a crear un proceso de trabajo con ambos servicios. Esta herramienta crea una aplicación principal de LUIS de intenciones que envía entre LUIS y QnA Maker como aplicaciones secundarias. [Más información](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=cs) sobre la integración con LUIS, QnA Maker y Bot Framework.

Use el ejemplo del creador de Bot **NLP con Dispatch**, en [C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/14.nlp-with-dispatch) o [Node.js](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/14.nlp-with-dispatch) para implementar este tipo de bot de chat.

## <a name="best-practices"></a>Procedimientos recomendados

Implementación de procedimientos recomendados para cada servicio:

* Procedimientos recomendados de [LUIS](../luis/luis-concept-best-practices.md)
* Procedimientos recomendados de [QnA Maker](../qnamaker/concepts/best-practices.md)

## <a name="see-also"></a>Consulte también

* [Language Understanding (LUIS)](../luis/what-is-luis.md)
* [QnA Maker](../qnamaker/overview/overview.md)
* [CLI de distribución](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)
* [Ejemplos de plataforma de bot](https://github.com/Microsoft/BotBuilder-Samples)
* [Azure Bot Service](https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0)
* [Emulador de bots de Azure](https://github.com/Microsoft/BotFramework-Emulator)
* [Chat web de la plataforma de bot](https://github.com/microsoft/BotFramework-WebChat)