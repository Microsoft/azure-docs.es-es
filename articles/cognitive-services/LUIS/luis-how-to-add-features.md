---
title: Uso de características para mejorar la lista de palabras de LUIS
titleSuffix: Azure Cognitive Services
description: Uso de Language Understanding (LUIS) para agregar características de aplicación que pueden mejorar la detección o predicción de intenciones y entidades de categorías y patrones.
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/17/2020
ms.author: diberry
ms.openlocfilehash: 25698fe2b05cbfb564e441e488bfa93221a9618a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075209"
---
# <a name="use-features-to-boost-signal-of-word-list"></a>Uso de características para aumentar la señal de la lista de palabras

Puede agregar características a la aplicación de LUIS para mejorar su precisión. Las características ayudan a LUIS al proporcionar sugerencias sobre ciertas palabras y frases que forman parte de un vocabulario de dominio de aplicación.

Revise los [conceptos](luis-concept-feature.md) para comprender cuándo y por qué usar una característica.

## <a name="add-phrase-list-as-a-feature"></a>Adición de una lista de frases como característica

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai), seleccione su **Suscripción** y **Recurso de creación** para ver las aplicaciones asignadas a ese recurso de creación.
1. Abra la aplicación mediante la selección de su nombre en la página **Mis aplicaciones**.
1. Seleccione **Compilación** y, a continuación, seleccione **Características** en el panel izquierdo de la aplicación.

1. En la página **Características**, seleccione **+ Crear**.

1. En el cuadro de diálogo **Create new phrase list feature** (Crear característica de lista de frases), escriba un nombre, como `Cities`. En el cuadro **Valor**, escriba ejemplos de ciudades, como `Seattle`. Puede escribir un valor a la vez o un conjunto de valores separados por comas y luego presione **ENTRAR**.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la adición de la característica (lista de frases) Cities](./media/luis-add-features/add-phrase-list-cities.png)

    Una vez que haya especificado suficientes valores para LUIS, aparecerán sugerencias. Puede **agregar todos** los valores propuestos o seleccionar términos individuales.

1. Mantenga activado **Estos valores son intercambiables** si las frases se pueden usar indistintamente.

1. La lista de frases se puede aplicar a toda la aplicación con la configuración **Global**, o bien a un modelo específico (intención o entidad). Si crea la lista de frases, como una _característica_ de una intención o entidad, la alternancia no se establece en global. En este caso, el significado de la alternancia es que la característica es solo local para ese modelo, por lo que _no es global_  para la aplicación.

1. Seleccione **Listo**. La nueva característica se agrega a la página **ML Features** (Características de ML).

<a name="edit-phrase-list"></a>
<a name="delete-phrase-list"></a>
<a name="deactivate-phrase-list"></a>


> [!Note]
> Puede eliminar o desactivar una lista de frases desde la barra de herramientas contextual de la página **ML Features**.

## <a name="global-phrase-list-applies-to-entire-app"></a>La lista de frases globales se aplica a toda la aplicación

Una lista de frases debe aplicarse a la intención o entidad a la que se pretende ayudar, pero puede haber ocasiones en las que se debe aplicar una lista de frases a toda la aplicación como característica **global**.

En la página de características de aprendizaje automático, seleccione la lista de frases y, a continuación, seleccione **Hacerlo global** en la barra de herramientas contextual superior.

## <a name="model-as-a-feature"></a>Modelo como característica

Una entidad puede ser una [característica para una intención o entidad](luis-concept-feature.md).

Para agregar una entidad como una característica para una intención, seleccione la intención en la página de intenciones y, a continuación, seleccione la opción **+ Agregar característica** situada sobre la barra de herramientas contextual. La lista incluirá todas las listas de frases y entidades que se pueden aplicar como características.

Para agregar una entidad como una característica a otra entidad, puede agregar la característica en la página de detalles de la intención mediante la [paleta de entidades](label-entity-example-utterance.md#adding-entity-as-a-feature-from-the-entity-palette) o puede [agregar la característica](luis-how-to-add-entities.md#add-a-feature-to-a-machine-learned-entity) en la página de detalles de la entidad.

## <a name="next-steps"></a>Pasos siguientes

Después de agregar, editar, eliminar o desactivar una característica, vuelva a [entrenar y probar la aplicación](luis-interactive-test.md) para ver si mejora el rendimiento.
