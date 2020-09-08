---
title: ¿Qué es Language Understanding (LUIS)?
description: 'Language Understanding (LUIS): servicio de API basado en la nube que usa el aprendizaje automático para la conversación y el lenguaje natural para predecir el significado y extraer información.'
keywords: Azure, artificial intelligence, ai, natural language processing, nlp, natural language understanding, nlu, ai conversation, conversational ai, ai chatbot, chatbot maker, LUIS, nlp ai, luis ai, azure luis, understanding natural language
ms.topic: overview
ms.date: 08/07/2020
ms.custom: cog-serv-seo-aug-2020
ms.openlocfilehash: 98d801f6a34feb40d56215f8b6257a68ec628e10
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89320059"
---
# <a name="what-is-language-understanding-luis"></a>¿Qué es Language Understanding (LUIS)?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Language Understanding (LUIS) es un servicio de API basado en la nube que aplica inteligencia de aprendizaje automático personalizado a una conversación o un texto de lenguaje natural de un usuario para predecir el significado global y extraer información pertinente y detallada.

Una aplicación cliente para LUIS es cualquier aplicación conversacional que se comunique con un usuario en lenguaje natural para completar una tarea. Entre estas aplicaciones cliente se encuentran, por ejemplo, aplicaciones de redes sociales, bots de chat y aplicaciones de escritorio habilitadas para voz.

![Imagen conceptual de tres aplicaciones cliente que funcionan con Language Understanding (LUIS) de Cognitive Services](./media/luis-overview/luis-entry-point.png "Imagen conceptual de tres aplicaciones cliente que funcionan con Language Understanding (LUIS) de Cognitive Services")

## <a name="use-luis-in-a-chat-bot"></a>Uso de LUIS en un bot de chat

<a name="Accessing-LUIS"></a>

Una vez que la aplicación de LUIS está publicada, la aplicación cliente envía expresiones (texto) a la [API][endpoint-apis] del punto de conexión de procesamiento de lenguaje natural de LUIS y recibe los resultados como respuestas en formato JSON. Una aplicación cliente común para LUIS es un bot de chat.


![Imagen conceptual del funcionamiento de LUIS con el bot de chat para predecir el texto del usuario con la comprensión del lenguaje natural (NLP)](./media/luis-overview/LUIS-chat-bot-request-response.svg "Imagen conceptual del funcionamiento de LUIS con el bot de chat para predecir el texto del usuario con la comprensión del lenguaje natural (NLP)")

|Paso|Acción|
|:--|:--|
|1|La aplicación cliente envía la _expresión_ del usuario (el texto en sus propias palabras), "I want to call my HR rep" ("Quiero llamar a mi representante de recursos humanos") al punto de conexión de LUIS como una solicitud HTTP.|
|2|LUIS le permite diseñar modelos de lenguaje personalizados para agregar inteligencia a su aplicación. Los modelos de lenguaje de aprendizaje automático toman el texto de entrada no estructurado del usuario y devuelven una respuesta con formato JSON, con una intención alta, `HRContact`. La respuesta en formato JSON del punto de conexión contiene como mínimo la expresión de consulta y la intención con mayor puntuación. También puede extraer datos, como la entidad _Tipo de contacto_.|
|3|La aplicación cliente usa la respuesta JSON para tomar decisiones acerca de cómo responder a las solicitudes del usuario. Estas decisiones pueden incluir árboles de decisión en el código de Bot Framework y llamadas a otros servicios. |

La aplicación de LUIS proporciona inteligencia para que la aplicación cliente pueda tomar decisiones inteligentes. LUIS no proporciona estas opciones.

<a name="Key-LUIS-concepts"></a>
<a name="what-is-a-luis-model"></a>

## <a name="natural-language-understanding-nlu"></a>Comprensión del lenguaje natural (NLU)

[LUIS proporciona inteligencia artificial (AI)](artificial-intelligence.md) en forma de NLU, un subconjunto del procesamiento de lenguaje natural (NLP).

La aplicación de LUIS contiene un modelo de lenguaje natural específico de un dominio. Puede iniciar la aplicación de LUIS con un modelo de dominio creado previamente, crear el suyo propio o combinar partes de un dominio creado previamente con su propia información personalizada.

* **Modelo creado previamente** LUIS tiene muchos modelos de dominio creados previamente, entre los que se incluyen intenciones, expresiones y entidades creadas previamente. Puede usar las entidades creadas previamente sin tener que utilizar las intenciones y expresiones del modelo precompilado. Los [modelos de dominio creados previamente](luis-how-to-use-prebuilt-domains.md) incluyen todo el diseño y suponen una excelente manera de empezar a usar rápidamente LUIS.

* **Modelo personalizado**: LUIS le ofrece varias maneras de identificar sus propios modelos personalizados, incluidas las intenciones y las entidades. Las entidades incluyen entidades de aprendizaje automático, específicas o literales, y una combinación de entidades de aprendizaje automático y literales.

Obtenga más información sobre [NLP](artificial-intelligence.md) y el área de NLU específica de LUIS.

## <a name="step-1-design-and-build-your-model"></a>Paso 1: Diseño y compilación del modelo

Diseñe su modelo con las categorías de intenciones del usuario llamadas **[intenciones](luis-concept-intent.md)** . Cada intención necesita ejemplos de **[expresiones](luis-concept-utterance.md)** del usuario. Cada expresión puede ofrecer datos que se deben extraer con [entidades de aprendizaje automático](luis-concept-entity-types.md#effective-machine-learned-entities).

|Expresión de usuario de ejemplo|Intención|Datos extraídos|
|-----------|-----------|-----------|
|`Book a flight to Seattle?`|BookFlight|Seattle|
|`When does your store open?`|StoreHoursAndLocation|abierto|
|`Schedule a meeting at 1pm with Bob in Distribution`|ScheduleMeeting|1 p.m., Bob|

Cree el modelo con las API de [creación](https://go.microsoft.com/fwlink/?linkid=2092087), con el [**portal de LUIS**](https://www.luis.ai) o con ambos. Obtenga más información sobre cómo crear con el [portal](get-started-portal-build-app.md) y las [bibliotecas de cliente del SDK](azure-sdk-quickstart.md).

## <a name="step-2-get-the-query-prediction"></a>Paso 2: Obtención de la predicción de consulta

Una vez que el modelo de la aplicación se haya entrenado y publicado en el punto de conexión, la aplicación cliente (como un bot de chat) enviará las expresiones a la API del [punto de conexión](https://go.microsoft.com/fwlink/?linkid=2092356) de predicción. La API aplica el modelo a la expresión para su análisis y responde con los resultados de la predicción en formato JSON.

La respuesta en formato JSON del punto de conexión contiene como mínimo la expresión de consulta y la intención con mayor puntuación. También puede extraer datos, como la siguiente entidad **Tipo de contacto** y la opinión general.

```JSON
{
    "query": "I want to call my HR rep",
    "prediction": {
        "topIntent": "HRContact",
        "intents": {
            "HRContact": {
                "score": 0.8582669
            }
        },
        "entities": {
            "Contact Type": [
                "call"
            ]
        },
        "sentiment": {
            "label": "neutral",
            "score": 0.5
        }
    }
}
```

## <a name="step-3-improve-model-prediction"></a>Paso 3: Mejora de la predicción del modelo

Después de publicar la aplicación de LUIS y recibir expresiones de usuario reales, LUIS proporciona [aprendizaje activo](luis-concept-review-endpoint-utterances.md) de las expresiones del punto de conexión para mejorar la precisión de la predicción. Revise estas sugerencias como parte del trabajo de mantenimiento periódico en el ciclo de vida de desarrollo.

<a name="using-luis"></a>

## <a name="development-lifecycle-and-tools"></a>Ciclo de vida de desarrollo y herramientas
LUIS proporciona herramientas, control de versiones y colaboración con otros autores de LUIS para integrarse en el [ciclo de vida de desarrollo completo](luis-concept-app-iteration.md).

Language Understanding (LUIS), como API REST, se puede usar con cualquier producto, servicio o marco con una solicitud HTTP. LUIS también proporciona bibliotecas de cliente (SDK) para varios lenguajes de programación importantes. Obtenga más información sobre los [recursos del desarrollador](developer-reference-resource.md) proporcionados.

Herramientas para usar de forma rápida y sencilla LUIS con un bot:
* [CLI de LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS): paquete NPM que proporciona creación y predicción mediante una herramienta independiente de la línea de comandos o como importación.
* [LUISGen](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUISGen) LUISGen es una herramienta para generar código fuente de C# y de TypeScript fuertemente tipado a partir de un modelo de LUIS exportado.
* [Dispatch](https://aka.ms/dispatch-tool) permite que una aplicación primaria utilice varias aplicaciones de LUIS y QnA Maker mediante el uso de un modelo de distribuidor.
* [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) LUDown es una herramienta de la línea de comandos que ayuda a administrar los modelos de lenguaje para el bot.

## <a name="integrate-with-a-bot"></a>Integración con un bot

Use el [servicio Azure Bot](https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0) con [Microsoft Bot Framework](https://dev.botframework.com/) para compilar e implementar un bot de chat. Diseñe y desarrolle con la herramienta de interfaz gráfica, [Composer](https://docs.microsoft.com/composer/), o [trabajo con los ejemplos de bot](https://github.com/microsoft/BotBuilder-Samples) diseñados para los principales escenarios de bot.

## <a name="integrate-with-other-cognitive-services"></a>Integración con otros servicios de Cognitive Services

Otros servicios de Cognitive Services que utilizan LUIS:
* [QnA Maker][qnamaker] permite la combinación de varios tipos de texto en una base de conocimiento de preguntas y respuestas.
* [Servicio Voz](../Speech-Service/overview.md) convierte solicitudes de lenguaje hablado en texto.

LUIS proporciona una funcionalidad de Text Analytics como parte de los recursos de LUIS existentes. Esta funcionalidad incluye el [análisis de opiniones](luis-how-to-publish-app.md#configuring-publish-settings) y la [extracción de frases clave](luis-reference-prebuilt-keyphrase.md) con la entidad keyPhrase pregenerada.

## <a name="learn-with-the-quickstarts"></a>Aprenda con las guías de inicio rápido

Obtenga información sobre LUIS con inicios rápidos prácticos mediante el [portal](get-started-portal-build-app.md) y las [bibliotecas de cliente del SDK](azure-sdk-quickstart.md).


## <a name="next-steps"></a>Pasos siguientes

* [Novedades](whats-new.md) con el servicio y la documentación
* [Planee la aplicación](luis-how-plan-your-app.md) con [intenciones](luis-concept-intent.md) y [entidades](luis-concept-entity-types.md).
* [Consulte el punto de conexión de predicción](luis-get-started-get-intent-from-browser.md).
* [Recursos para el desarrollador](developer-reference-resource.md) de LUIS

[bot-framework]: https://docs.microsoft.com/bot-framework/
[flow]: https://docs.microsoft.com/connectors/luis/
[authoring-apis]: https://go.microsoft.com/fwlink/?linkid=2092087
[endpoint-apis]: https://go.microsoft.com/fwlink/?linkid=2092356
[qnamaker]: https://qnamaker.ai/
