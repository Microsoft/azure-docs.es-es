---
title: 'Formulación de preguntas que se puedan responder con datos: ML Studio (clásico) - Azure'
description: Aprenda a formular una buena pregunta de ciencia de datos en Ciencia de datos para principiantes, vídeo 3. Incluye una comparación de preguntas de clasificación y regresión.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: sdgilley
ms.author: sgilley
ms.date: 03/22/2019
ms.openlocfilehash: ff08af00649cf61134c3e27d07b329e40169ea66
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318800"
---
# <a name="ask-a-question-you-can-answer-with-data"></a>Realización de preguntas que pueden responderse con datos
## <a name="video-3-data-science-for-beginners-series"></a>Vídeo 3: Ciencia de datos para principiantes
Aprenda a formular una pregunta de ciencia de datos en Ciencia de datos para principiantes, vídeo 3. Este vídeo incluye una comparación de preguntas para algoritmos de clasificación y regresión.

Para obtener el máximo partido de la serie, véalos en orden. [Ir a la lista de vídeos](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Data-science-for-beginners-Ask-a-question-you-can-answer-with-data/player]
>
>

## <a name="other-videos-in-this-series"></a>Otros vídeos de la serie
*Ciencia de datos para principiantes* es una introducción rápida a la ciencia de datos en cinco vídeos de corta duración.

* Vídeo 1: [Las cinco preguntas a las que responde la ciencia de datos](data-science-for-beginners-the-5-questions-data-science-answers.md) *(5 minutos y 14 segundos)*
* Vídeo 2: [¿Están sus datos preparados para la ciencia de datos?](data-science-for-beginners-is-your-data-ready-for-data-science.md) *(4 minutos y 56 segundos)*
* Vídeo 3: Realización de preguntas que pueden responderse con datos
* Vídeo 4: [Predicción de respuestas con un modelo sencillo](data-science-for-beginners-predict-an-answer-with-a-simple-model.md) *(7 minutos y 42 segundos)*
* Vídeo 5: [Copia del trabajo de otras personas para realizar ciencia de datos](data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md) *(3 minutos y 18 segundos)*

## <a name="transcript-ask-a-question-you-can-answer-with-data"></a>Transcripción: Realización de preguntas que pueden responderse con datos
Este es el tercer vídeo de la serie "Ciencia de datos para principiantes".  

En este vídeo, recibirá algunas sugerencias para formular una pregunta que se pueda responder con datos.

Aprovechará mejor este vídeo si ve primero los dos vídeos anteriores de esta serie. "Las cinco preguntas a las que responde la ciencia de datos" y "¿Están sus datos preparados para la ciencia de datos?"

## <a name="ask-a-sharp-question"></a>Formulación de una pregunta directa
Hemos hablado de que la ciencia de datos es el proceso de usar nombres (llamados también categorías o etiquetas) y números para predecir una respuesta a una pregunta. Pero no sirve cualquier pregunta; debe ser una *pregunta directa.*

Una pregunta vaga no se tiene que responder con un nombre o un número. Una pregunta directa sí.

Imagine que encuentra una lámpara mágica con un genio que responde de forma veraz a cualquier pregunta que formule. Pero este genio es travieso, e intentará responder de forma tan vaga y confusa mientras pueda salirse con la suya. Quiere precisar una pregunta de forma tan irrefutable que a la fuerza le diga lo que quiere saber.

Si formulara una pregunta vaga como "¿Qué va a pasar con mis acciones?", el genio podría responder: "El precio cambiará". Es una respuesta veraz, pero no sirve de mucha ayuda.

Pero si tuviera que formular una pregunta directa, como "¿Cuál será el precio de venta de mis acciones la próxima semana?", el genio no podrá evitar darle una respuesta específica y predecir un precio de venta.

## <a name="examples-of-your-answer-target-data"></a>Ejemplos de la respuesta: datos de destino
Después de formular la pregunta, compruebe si tiene ejemplos de la respuesta en sus datos.

Si la pregunta es "¿A qué precio de venta estarán mis acciones la semana próxima?", tenemos para asegurarnos de que nuestros datos incluyan el historial de precios de cotización.

Si la pregunta es "¿Qué automóvil de mi flota va a caer primero?", tenemos que asegurarnos de que nuestros datos incluyan información de los errores anteriores.

![Datos de destino: ejemplos de la respuesta. Formular una pregunta de ciencia de datos.](./media/data-science-for-beginners-ask-a-question-you-can-answer-with-data/target-data.png)

Estos ejemplos de respuestas se denominan destino. Un destino es lo que intentamos predecir sobre los puntos de datos futuros, ya sea una categoría o un número.

Si no tiene los datos de destino, deberá obtener algunos. Sin ellos, no podrá responder a su pregunta.

## <a name="reformulate-your-question"></a>Reformulación de la pregunta
A veces, puede formular la pregunta con otras palabras a fin de obtener una respuesta más útil.

La pregunta "¿Es este punto de datos A o B?" predice la categoría (o nombre o etiqueta) de algo. Para responderla, usamos un *algoritmo de clasificación*.

La pregunta "¿Cuánto?" o "¿Cuántos?" predice una cantidad. Para responderla, usamos un *algoritmo de regresión*.

Para ver cómo podemos transformarlas, examinemos la pregunta: "¿Qué reportaje de noticias es el más interesante para este lector?" Se pregunta por una predicción de una sola opción entre muchas posibilidades, es decir: ¿Es A, B, C o D?, y se usaría un algoritmo de clasificación.

Pero esta pregunta puede ser más fácil de responder si la formula de esta forma: "¿Cuánto interés tiene cada reportaje de la lista para este lector?" Ahora puede dar a cada artículo una puntuación numérica, de esta forma será fácil identificar el artículo con la puntuación más alta. Se trata de reformular la pregunta de clasificación como una de regresión o ¿cuánto?

![Reformular la pregunta. Pregunta de clasificación frente a pregunta de regresión.](./media/data-science-for-beginners-ask-a-question-you-can-answer-with-data/classification-question-vs-regression-question.png)

El modo de formular una pregunta es una pista de qué algoritmo puede darle una respuesta.

Encontrará que ciertas familias de algoritmos, como los de nuestro ejemplo de reportajes de noticias, están estrechamente relacionadas. Puede reformular la pregunta para usar el algoritmo que le proporcione la respuesta más útil.

Y más importante aún, formule esa pregunta directa, la pregunta que se pueda responder con datos. Y asegúrese de tener los datos adecuados para responderla.

Hemos hablado de algunos principios básicos para formular una pregunta que se pueda responder con datos.

Asegúrese de consultar los demás vídeos de la serie "Ciencia de datos para principiantes" de Microsoft Azure Machine Learning Studio (clásico).

## <a name="next-steps"></a>Pasos siguientes
* [Prueba de su primer experimento de ciencia de datos con Machine Learning Studio (clásico)](create-experiment.md)
* [Introducción a Machine Learning en Microsoft Azure](/azure/machine-learning/overview-what-is-azure-ml)
