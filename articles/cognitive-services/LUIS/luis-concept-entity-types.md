---
title: 'Tipos de entidades: LUIS'
description: Una entidad extrae datos de una expresión de usuario en tiempo de ejecución de predicción. Un propósito _opcional_ y secundario es impulsar la predicción de la intención o de otras entidades usando la entidad como una característica.
ms.topic: conceptual
ms.date: 08/06/2020
ms.openlocfilehash: 8751bdd52bb1c3738103dc074184a3cf72bfeb09
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88207748"
---
# <a name="extract-data-with-entities"></a>Extracción de datos con entidades

Una entidad extrae datos de una expresión de usuario en tiempo de ejecución de predicción. Un propósito _opcional_ y secundario es impulsar la predicción de la intención o de otras entidades usando la entidad como una característica.

Existen varios tipos de entidades:

* [Entidad de aprendizaje automático](reference-entity-machine-learned-entity.md):Esta es la entidad principal. Debe diseñar el esquema con este tipo de entidad antes de usar otras entidades.
* Las entidades sin aprendizaje automático se usan como [característica](luis-concept-feature.md) obligatoria: para coincidencias de texto exactas, coincidencias de patrones o detecciones por entidades precompiladas
* [Pattern.any](#patternany-entity): para extraer texto de forma libre, como títulos de libros de un [patrón](reference-entity-pattern-any.md)

Las entidades de aprendizaje automático proporcionan la gama más amplia de opciones de extracción de datos. Las entidades sin aprendizaje automático funcionan por coincidencia de texto y se usan como [característica obligatoria](#design-entities-for-decomposition) para una entidad de aprendizaje automático o intención.

## <a name="entities-represent-data"></a>Las entidades representan datos

Las entidades son los datos que desea extraer de la expresión, como nombres, fechas, nombres de producto o cualquier grupo de palabras significativo. Una expresión puede incluir varias o ninguna entidad. _Es posible_ que una aplicación cliente necesite los datos para realizar su tarea.

Las entidades deben etiquetarse de forma coherente en todas las expresiones de entrenamiento para cada intención en un modelo.

 Puede definir sus propias entidades o usar entidades pregeneradas para ahorrar tiempo para conceptos comunes como [datetimeV2](luis-reference-prebuilt-datetimev2.md), [ordinal](luis-reference-prebuilt-ordinal.md), [correo electrónico](luis-reference-prebuilt-email.md) y [número de teléfono](luis-reference-prebuilt-phonenumber.md).

|Expresión|Entidad|data|
|--|--|--|
|Comprar 3 billetes a Nueva York|Número creado previamente<br>Destination|3<br>Nueva York|


### <a name="entities-are-optional-but-recommended"></a>Las entidades son opcionales, pero muy recomendables

Mientras las [intenciones](luis-concept-intent.md) son obligatorias, las entidades son opcionales. No es necesario crear entidades para cada concepto de la aplicación, sino solo para aquellos en los que la aplicación cliente necesite los datos o en los que la entidad actúe como una sugerencia o señal para otra entidad o intención.

A medida que la aplicación se desarrolla y se identifica una nueva necesidad de datos, podrá agregar las entidades adecuadas al modelo LUIS más adelante.

<a name="entity-compared-to-intent"></a>

## <a name="entity-represents-data-extraction"></a>La entidad representa la extracción de datos

La entidad representa un concepto de datos _dentro de la expresión_. Una intención clasifica _toda la expresión_.

Considere las cuatro expresiones siguientes:

|Expresión|Intención pronosticada|Entidades extraídas|Explicación|
|--|--|--|--|
|Ayuda|help|-|No hay nada que extraer.|
|Send something|sendSomething|-|No hay nada que extraer. El modelo no tiene una característica obligatoria para extraer `something` en este contexto y no se ha indicado ningún destinatario.|
|Send Bob a present|sendSomething|`Bob`, `present`|El modelo extrae `Bob` agregando una característica obligatoria de entidad precompilada `personName`. Se ha usado una entidad de aprendizaje automático para extraer `present`.|
|Send Bob a box of chocolates|sendSomething|`Bob`, `box of chocolates`|Las entidades de aprendizaje automático han extraído los dos fragmentos importantes de datos, `Bob` y `box of chocolates`.|

## <a name="label-entities-in-all-intents"></a>Etiquetado de entidades en todas las intenciones

Las entidades extraen los datos independientemente de la intención prevista. Asegúrese de etiquetar _todas_ las expresiones de ejemplo en todas las intenciones. El hecho de que falte el etiquetado de entidades en la intención `None` genera confusión, incluso aunque haya muchas más expresiones de entrenamiento para las otras intenciones.

## <a name="design-entities-for-decomposition"></a>Diseño de entidades para la descomposición

Las entidades de aprendizaje automático permiten diseñar el esquema de la aplicación para la descomposición, mediante la división de un concepto grande en subentidades.

El diseño de la descomposición permite a LUIS devolver un grado profundo de resolución de la entidad a la aplicación cliente. Esto permite a la aplicación cliente centrarse en reglas de negocios y dejar a LUIS la resolución de datos.

Una entidad de aprendizaje automático se desencadena en función del contexto aprendido a través de expresiones de ejemplo.

Las [**entidades de aprendizaje automático**](tutorial-machine-learned-entity.md) son los extractores de nivel superior. Las subentidades son entidades secundarias de las entidades de aprendizaje automático.

## <a name="effective-machine-learned-entities"></a>Entidades de aprendizaje automático eficaces

Para compilar las entidades de aprendizaje automático de forma eficaz:

* El etiquetado debe ser coherente en todas las intenciones. Esto incluye incluso las expresiones que proporcione en la intención **None** que incluyan esta entidad. De lo contrario, el modelo no podrá determinar las secuencias de forma eficaz.
* Si tiene una entidad de aprendizaje automático con subentidades, asegúrese de que los distintos órdenes y variantes de la entidad y las subentidades se presenten en las expresiones etiquetadas. Las expresiones etiquetadas de ejemplo deben incluir todas las formas válidas e incluir entidades que aparecen y que están ausentes, y que también se reordenan dentro de la expresión.
* Debe evitar sobreajustar las entidades a un conjunto muy fijo. El **sobreajuste** tiene lugar cuando el modelo no está bien generalizado, y es un problema común en los modelos de Machine Learning. Esto implica que la aplicación no funcionaría correctamente con datos nuevos. A su vez, debe variar las expresiones etiquetadas de ejemplo para que la aplicación pueda generalizar más allá de los ejemplos limitados que proporcione. Debe variar las distintas subentidades con cambios suficientes para que el modelo piense más en el concepto en lugar de solo en los ejemplos mostrados.

## <a name="effective-prebuilt-entities"></a>Entidades eficaces precompiladas

Para crear entidades eficaces que extraigan datos comunes, como los proporcionados por las [entidades precompiladas](luis-reference-prebuilt-entities.md), se recomienda el siguiente proceso.

Mejore la extracción de datos mediante la incorporación de sus propios datos a una entidad como una característica. De este modo, todas las etiquetas adicionales de los datos aprenderán el contexto de dónde existen los nombres de persona en la aplicación.

<a name="composite-entity"></a>
<a name="list-entity"></a>
<a name="patternany-entity"></a>
<a name="prebuilt-entity"></a>
<a name="regular-expression-entity"></a>
<a name="simple-entity"></a>

## <a name="types-of-entities"></a>Tipos de entidades

Una subentidad de un elemento primario debe ser una entidad de aprendizaje automático. La subentidad puede utilizar una entidad sin aprendizaje automático como [característica](luis-concept-feature.md).

Elija la entidad en función de cómo se deben extraer los datos y cómo se deben representar una vez que se extraen.

|Tipo de entidad|Propósito|
|--|--|
|[**Con aprendizaje automático**](tutorial-machine-learned-entity.md)|Extraer datos anidados y complejos aprendidos de ejemplos con etiquetas. |
|[**Lista**](reference-entity-list.md)|Lista de elementos y sus sinónimos extraída con **coincidencia de texto exacta**.|
|[**Pattern.any**](#patternany-entity)|La entidad donde se encuentra el final de la entidad es difícil de determinar porque la entidad tiene una forma libre. Solo disponibles en los [patrones](luis-concept-patterns.md).|
|[**Creada previamente**](luis-reference-prebuilt-entities.md)|Ya entrenado para extraer un tipo específico de datos, como la dirección URL o el correo electrónico. Algunas de estas entidades precompiladas se definen en el proyecto de código abierto [Recognizers-Text](https://github.com/Microsoft/Recognizers-Text). Si su referencia cultural o entidad específica no se admite actualmente, colabore en el proyecto.|
|[**Expresión regular**](reference-entity-regular-expression.md)|Usa una expresión regular para la **coincidencia de texto exacta**.|


## <a name="extraction-versus-resolution"></a>Extracción frente a resolución

Las entidades extraen los datos a medida que estos aparecen en la expresión. Las entidades no cambian ni resuelven los datos. La entidad no proporcionará ninguna resolución así el texto sea un valor válido para la entidad o no.

Hay formas de aportar resolución a la extracción, pero debe tener en cuenta que esto limita la capacidad de la aplicación para ser inmune a las variaciones y los errores.

Las entidades de lista y las entidades de expresión regular (coincidencia de texto) se pueden usar como [características obligatorias](luis-concept-feature.md#required-features) de una subentidad y esto sirve como filtro para la extracción. Debe usar esto con cuidado para no entorpecer la capacidad de predicción de la aplicación.

## <a name="extracting-contextually-related-data"></a>Extracción de datos relacionados contextualmente

Una expresión puede contener dos o más apariciones de una entidad en la que el significado de los datos se basa en el contexto dentro de la expresión. Un ejemplo es una expresión para reservar un vuelo que tiene dos ubicaciones geográficas, origen y destino.

`Book a flight from Seattle to Cairo`

Las dos ubicaciones deben extraerse de forma que la aplicación cliente conozca el tipo de cada ubicación para completar el billete.

Para extraer el origen y el destino, cree dos subentidades como parte de la entidad de aprendizaje automático de pedido de billetes. Para cada una de las subentidades, cree una característica obligatoria que usa geographyV2.

<a name="using-component-constraints-to-help-define-entity"></a>
<a name="using-subentity-constraints-to-help-define-entity"></a>

### <a name="using-required-features-to-constrain-entities"></a>Uso de características obligatorias para restringir entidades

Más información sobre las [características obligatorias](luis-concept-feature.md)

## <a name="patternany-entity"></a>Entidad Pattern.any

Una entidad Pattern.any solo está disponible en un [Patrón](luis-concept-patterns.md).

<a name="if-you-need-more-than-the-maximum-number-of-entities"></a>
## <a name="exceeding-app-limits-for-entities"></a>Superación de los límites de aplicación para entidades

Si necesita más que el [límite](luis-limits.md#model-limits), póngase en contacto con el soporte técnico. Para ello, recopile información detallada sobre el sistema, vaya al sitio web de [LUIS](luis-reference-regions.md#luis-website) y seleccione **Support** (Soporte). Si la suscripción a Azure incluye servicios de soporte técnico, póngase en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/options/).

## <a name="entity-prediction-status-and-errors"></a>Estado de predicción de entidad y errores

En el portal de LUIS se muestra cuando la entidad tiene una predicción de entidad diferente a la que se ha seleccionado para una expresión de ejemplo. Esta puntuación diferente se basa en el modelo entrenado actual. 

:::image type="content" source="./media/luis-concept-entities/portal-entity-prediction-error.png" alt-text="En el portal de LUIS se muestra cuando la entidad tiene una predicción de entidad diferente a la que se ha seleccionado para una expresión de ejemplo.":::

En la expresión de ejemplo, el texto de error está resaltado, y la línea de la expresión de ejemplo tiene un indicador de error a la derecha, que se muestra como un triángulo rojo. 

Use esta información para solucionar errores de entidad con uno o varios de los siguientes:
* El texto resaltado tiene una etiqueta incorrecta. Para solucionarlo, revise, corrija y vuelva a entrenar. 
* Creación de una [característica](luis-concept-feature.md) para la entidad a fin de ayudar a identificar el concepto de la entidad
* Adición de [expresiones de ejemplo](luis-concept-utterance.md) y etiquetado con la entidad
* [Revisión de las sugerencias de aprendizaje activas](luis-concept-review-endpoint-utterances.md) para cualquier expresión recibida en el punto de conexión de predicción que pueda ayudar a identificar el concepto de la entidad.

## <a name="next-steps"></a>Pasos siguientes

Aprenda conceptos sobre las buenas [expresiones](luis-concept-utterance.md).

Vea [Add entities](luis-how-to-add-entities.md) (Agregar entidades) para obtener más información sobre cómo agregar entidades a la aplicación de LUIS.

Consulte [Tutorial: Extracción de datos estructurados de una expresión de usuario con entidades con aprendizaje automático en Language Understanding (LUIS)](tutorial-machine-learned-entity.md) para obtener información sobre cómo extraer datos estructurados de una expresión mediante la entidad de aprendizaje automático.

