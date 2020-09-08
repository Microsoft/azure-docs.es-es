---
title: 'Inicio rápido: Bibliotecas cliente del SDK de Language Understanding (LUIS)'
description: Con este inicio rápido, creará una aplicación de LUIS, y realizará consultas en ella, con las bibliotecas cliente del SDK de LUIS usando C#, Python o JavaScript.
ms.topic: quickstart
ms.date: 09/01/2020
keywords: Azure, artificial intelligence, ai, natural language processing, nlp, natural language understanding, nlu, ai conversation, conversational ai, ai chatbot, chatbot maker, LUIS, nlp ai, luis ai, azure luis, understanding natural language
ms.custom: devx-track-python, devx-track-javascript, devx-track-csharp, cog-serv-seo-aug-2020
zone_pivot_groups: programming-languages-set-diberry-3core
ms.openlocfilehash: 6bcdca85125d44475fadfd195c1dfda88f761f88
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89323114"
---
# <a name="quickstart-language-understanding-luis-sdk-client-libraries-to-create-and-query-your-luis-app"></a>Inicio rápido: Bibliotecas cliente del SDK de Language Understanding (LUIS) para crear la aplicación de LUIS y realizar consultas en ella

Con este inicio rápido, creará una aplicación de LUIS, y realizará consultas en ella, con las bibliotecas cliente del SDK de LUIS usando C#, Python o JavaScript.

Language Understanding (LUIS) permite aplicar inteligencia de aprendizaje automático personalizado a una conversación o un texto de lenguaje natural de un usuario para predecir el significado global y extraer información pertinente y detallada.

* La biblioteca cliente del **SDK de creación** le permite crear, editar, entrenar y publicar aplicaciones de LUIS. *La biblioteca cliente del **SDK del entorno de ejecución de predicción** le permite consultar la aplicación publicada.

::: zone pivot="programming-language-csharp"
[!INCLUDE [LUIS development with C# SDK](./includes/sdk-csharp.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [LUIS development with Node.js SDK](./includes/sdk-nodejs.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [LUIS development with Python SDK](./includes/sdk-python.md)]
::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

La aplicación se puede eliminar desde el [portal de LUIS](https://www.luis.ai) y los recursos de Azure desde [Azure Portal](https://portal.azure.com/).

## <a name="troubleshooting"></a>Solución de problemas

* Autenticación en la biblioteca cliente: los errores de autenticación indican normalmente que se han usado el punto de conexión y la clave incorrectos. Por comodidad, en este inicio rápido se usan la clave y el punto de conexión de creación del entorno de ejecución de predicción, pero solo funcionarán si aún no ha usado la cuota mensual. Si no puede usarlos, debe utilizar la clave y el punto de conexión del entorno de ejecución de predicción al acceder a la biblioteca cliente del SDK del entorno de ejecución de predicción.
* Creación de entidades: si recibe un error al crear la entidad de aprendizaje automático anidada que se usa en este tutorial, asegúrese de que ha copiado el código y que no lo ha modificado para crear una entidad diferente.
* Creación de expresiones de ejemplo: si recibe un error al crear la expresión del ejemplo con etiquetas usado en este tutorial, asegúrese de que ha copiado el código y de que no lo ha modificado para crear otro ejemplo con etiquetas.
* Entrenamiento: si recibe un error de entrenamiento, normalmente significa que hay una aplicación vacía (sin intenciones con expresiones de ejemplo) o una aplicación con intenciones o entidades con un formato incorrecto.
* Errores varios: dado que el código llama a las bibliotecas cliente con objetos de texto y JSON, asegúrese de que no ha cambiado el código.

Otros errores: si recibe un error que no aparece en la lista anterior, háganoslo saber; para ello, puede enviarnos sus comentarios en la parte inferior de esta página. Incluya el lenguaje de programación y la versión de las bibliotecas cliente que ha instalado. 

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es Language Understanding (LUIS)?](what-is-luis.md)
* [Novedades](whats-new.md)
* [Intenciones](luis-concept-intent.md), [entidades](luis-concept-entity-types.md), [expresiones de ejemplo](luis-concept-utterance.md) y [entidades precompiladas](luis-reference-prebuilt-entities.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code).
