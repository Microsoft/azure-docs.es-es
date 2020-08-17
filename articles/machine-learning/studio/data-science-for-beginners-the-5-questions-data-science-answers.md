---
title: 'ML Studio (clásico): Ciencia de datos para principiantes: Azure'
description: Ciencia de datos para principiantes explica los conceptos básicos en 5 vídeos breves, empezando por las 5 preguntas a las que responde la ciencia de datos. De Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: sdgilley
ms.author: sgilley
ms.custom: seodec18
ms.date: 03/22/2019
ms.openlocfilehash: a0c99c05701c7f2d64fcfc2f70081f2d7475ce5f
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318732"
---
# <a name="data-science-for-beginners-video-1-the-5-questions-data-science-answers"></a>Ciencia de datos para principiantes, vídeo 1: Las cinco preguntas a las que responde la ciencia de datos

Obtenga una rápida introducción a la ciencia de datos en la serie *Ciencia de datos para principiantes*, con cinco vídeos breves, que protagoniza una de los científicos de datos más importantes. Estos vídeos son básicos pero útiles si está interesado en elaborar ciencia de datos o trabajar con científicos de datos.

Este primer vídeo trata de los tipos de preguntas a los que puede responder la ciencia de datos. Para obtener el máximo partido de la serie, véalos en orden. [Ir a la lista de vídeos](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Shows/SupervisionNotRequired/8/player]
>
>

## <a name="other-videos-in-this-series"></a>Otros vídeos de la serie
*Ciencia de datos para principiantes* es una introducción rápida a la ciencia de datos de aproximadamente 25 minutos en total. Consulte los cinco vídeos:

* Vídeo 1: Las cinco preguntas a las que responde la ciencia de datos
* Vídeo 2: [¿Están sus datos preparados para la ciencia de datos?](data-science-for-beginners-is-your-data-ready-for-data-science.md) *(4 minutos y 56 segundos)*
* Vídeo 3: [Realización de preguntas que pueden responderse con datos](data-science-for-beginners-ask-a-question-you-can-answer-with-data.md) *(4 minutos y 17 segundos)*
* Vídeo 4: [Predicción de respuestas con un modelo sencillo](data-science-for-beginners-predict-an-answer-with-a-simple-model.md) *(7 minutos y 42 segundos)*
* Vídeo 5: [Copia del trabajo de otras personas para realizar ciencia de datos](data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md) *(3 minutos y 18 segundos)*

## <a name="transcript-the-5-questions-data-science-answers"></a>Transcripción: Las cinco preguntas a las que responde la ciencia de datos
¡Hola! Esta es la serie de vídeos *Ciencia de datos para principiantes*.

La ciencia de datos puede ser algo intimidante, por lo que he presentado aquí los aspectos básicos, sin ecuaciones ni jerga de programación.

En este primer vídeo, hablaremos sobre "Las cinco preguntas a las que responde la ciencia de datos".

La ciencia de datos utiliza números y nombres (también denominados categorías o etiquetas) y predice respuestas a preguntas.

Puede sorprenderle, pero *solo hay cinco preguntas a las que responde la ciencia de datos*:

* ¿Esto es A o B?
* ¿Es extraño?
* ¿Cuánto? o ¿cuántos?
* ¿Cómo está organizado?
* ¿Qué debo hacer a continuación?

Cada una de estas preguntas se responde mediante una familia de métodos de aprendizaje automático independientes, denominados algoritmos.

Resulta útil pensar en un algoritmo como una receta y los datos como los ingredientes. Un algoritmo indica cómo combinar y mezclar los datos para obtener una respuesta. Los equipos son similares a una batidora. Llevan a cabo una gran parte del trabajo duro del algoritmo y lo hacen bastante rápido.

## <a name="question-1-is-this-a-or-b-uses-classification-algorithms"></a>Pregunta 1: ¿esto es A o B? utiliza algoritmos de clasificación
Comencemos con la pregunta: ¿Esto es A o B?

![Algoritmos de clasificación: ¿Esto es A o B?](./media/data-science-for-beginners-the-5-questions-data-science-answers/classification-algorithms.png)

Esta familia de algoritmos se llama clasificación de dos clases.

Es útil para cualquier pregunta que solo tenga dos respuestas posibles.

Por ejemplo:

* Este neumático se romperá en las próximas 1000 millas: ¿Sí o no?
* Qué aporta más clientes ¿un cupón de 5 USD o un descuento del 25 %?

Esta pregunta también puede modificarse para que incluya más de dos opciones: ¿esto es A, B, C, D u otras?  Esto se denomina clasificación multiclase y es útil cuando hay varias respuestas (o varios miles de respuestas) posibles. La clasificación multiclase elige la más probable.

## <a name="question-2-is-this-weird-uses-anomaly-detection-algorithms"></a>Pregunta 2: ¿es extraño? utiliza algoritmos de detección de anomalías
La siguiente pregunta que la ciencia de datos puede responder es: ¿Es extraño? Esta pregunta se responde mediante una familia de algoritmos denominada detección de anomalías.

![Algoritmos de detección de anomalías: ¿Es extraño?](./media/data-science-for-beginners-the-5-questions-data-science-answers/anomaly-detection-algorithms.png)

Si tiene una tarjeta de crédito, ya se habrá beneficiado de la detección de anomalías. Su compañía de tarjetas de crédito analiza sus patrones de compra para poder alertarle de un posible fraude. Cargos "extraños" podrían ser una compra en una tienda donde normalmente no compra o comprar un artículo inusualmente caro.

Esta pregunta puede ser útil de muchas formas. Por ejemplo:

* Si tiene un automóvil con indicadores de presión, puede que quiera saber: ¿la lectura de este medidor de presión es normal?
* Si está supervisando Internet, quizá quiera saber: ¿Es este un mensaje típico de internet?

La detección de anomalías marca eventos o comportamientos inesperados o poco habituales. Proporciona pistas sobre dónde buscar problemas.

## <a name="question-3-how-much-or-how-many-uses-regression-algorithms"></a>Pregunta 3: ¿cuánto? o ¿cuántos? utiliza algoritmos de regresión
Machine Learning también puede predecir la respuesta a las preguntas ¿cuánto? o ¿cuántos? La familia de algoritmos que responde a esta pregunta se denomina regresión.

![Algoritmos de regresión: ¿cuánto? o ¿cuántos?](./media/data-science-for-beginners-the-5-questions-data-science-answers/regression-algorithms.png)

Los algoritmos de regresión realizan predicciones numéricas, como:

* ¿Qué temperatura hará el martes que viene?  
* ¿Cuáles serán las ventas del cuarto trimestre?

Ayudan a responder a cualquier pregunta que pueda solicitar un número.

## <a name="question-4-how-is-this-organized-uses-clustering-algorithms"></a>Pregunta 4: ¿cómo está organizado? utiliza algoritmos de clústeres
Ahora bien, las dos últimas preguntas son un poco más avanzadas.

A veces deseará comprender la estructura de un conjunto de datos: ¿cómo está organizado? Para esta pregunta, no hay ejemplos para los que ya conozca resultados.

Hay muchas maneras de determinar la estructura de datos. Un enfoque son los clústeres. Separa los datos en grupos naturales para interpretarlos más fácilmente. Con los clústeres no hay una respuesta correcta.

![Algoritmos de agrupación en clústeres: ¿Cómo está organizado?](./media/data-science-for-beginners-the-5-questions-data-science-answers/clustering-algorithms.png)

Algunos ejemplos comunes de preguntas con agrupación en clústeres son:

* ¿A qué espectadores les gusta el mismo tipo de películas?
* ¿Qué modelos de impresora generan errores del mismo modo?

Al comprender cómo se organizan los datos, puede comprender y predecir mejor eventos y comportamientos.  

## <a name="question-5-what-should-i-do-now-uses-reinforcement-learning-algorithms"></a>Pregunta 5: ¿qué debo hacer ahora? utiliza algoritmos de aprendizaje reforzado
La última pregunta: ¿qué debo hacer ahora? utiliza una familia de algoritmos llamada aprendizaje reforzado.

El aprendizaje reforzado se inspiró en cómo responde el cerebro de las ratas y los seres humanos a los castigos y las recompensas. Estos algoritmos aprenden de los resultados y deciden la acción siguiente.

Normalmente, el aprendizaje reforzado es una buena elección para los sistemas automatizados que tengan que tomar una gran cantidad de pequeñas decisiones sin ayuda humana.

![Algoritmos de aprendizaje de refuerzo: ¿Qué debo hacer a continuación?](./media/data-science-for-beginners-the-5-questions-data-science-answers/reinforcement-learning-algorithms.png)

Las preguntas a las que responde son siempre sobre la acción que se debe llevar a cabo; normalmente dicha acción será realizada por una máquina o un robot. Algunos ejemplos son:

* Si soy un sistema de control de la temperatura de una casa: ¿ajusto la temperatura o la dejo como está?  
* Si soy un automóvil sin conductor: en un semáforo en ámbar, ¿freno o acelero?  
* Para un robot aspirador: ¿sigo aspirando o vuelvo a la estación de carga?

Los algoritmos de aprendizaje reforzado recopilan datos a medida que avanzan, aprendiendo mediante prueba y error.

Eso es todo: las cinco preguntas a las que puede responder la ciencia de datos.

## <a name="next-steps"></a>Pasos siguientes
* [Prueba de su primer experimento de ciencia de datos con Machine Learning Studio (clásico)](create-experiment.md)
* [Introducción a Machine Learning en Microsoft Azure](/azure/machine-learning/overview-what-is-azure-ml)
