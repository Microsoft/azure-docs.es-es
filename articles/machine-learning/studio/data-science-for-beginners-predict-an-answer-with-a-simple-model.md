---
title: 'ML Studio (clásico): Predicción de respuestas con modelos de regresión: Azure'
description: Descubra cómo crear un modelo de regresión simple para predecir un precio en el vídeo 4 de Ciencia de datos para principiantes. Incluye una regresión lineal con los datos de destino.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: sdgilley
ms.author: sgilley
ms.custom: seodec18
ms.date: 03/22/2019
ms.openlocfilehash: 1cfe617ab424fbb29e2866cd98d4b8c390f306c9
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318749"
---
# <a name="predict-an-answer-with-a-simple-model"></a>Predicción de respuestas con un modelo sencillo


## <a name="video-4-data-science-for-beginners-series"></a>Vídeo 4: Ciencia de datos para principiantes
Aprenda a crear un modelo de regresión simple para predecir el precio de un diamante en el vídeo 4 de Ciencia de datos para principiantes. Dibujaremos un modelo de regresión con los datos de destino.

Para obtener el máximo partido de la serie, véalos en orden. [Ir a la lista de vídeos](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/data-science-for-beginners-series-predict-an-answer-with-a-simple-model/player]
>
>

## <a name="other-videos-in-this-series"></a>Otros vídeos de la serie
*Ciencia de datos para principiantes* es una introducción rápida a la ciencia de datos en cinco vídeos de corta duración.

* Vídeo 1: [Las cinco preguntas a las que responde la ciencia de datos](data-science-for-beginners-the-5-questions-data-science-answers.md) *(5 minutos y 14 segundos)*
* Vídeo 2: [¿Están sus datos preparados para la ciencia de datos?](data-science-for-beginners-is-your-data-ready-for-data-science.md) *(4 minutos y 56 segundos)*
* Vídeo 3: [Realización de preguntas que pueden responderse con datos](data-science-for-beginners-ask-a-question-you-can-answer-with-data.md) *(4 minutos y 17 segundos)*
* Vídeo 4: Predicción de respuestas con un modelo sencillo
* Vídeo 5: [Copia del trabajo de otras personas para realizar ciencia de datos](data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md) *(3 minutos y 18 segundos)*

## <a name="transcript-predict-an-answer-with-a-simple-model"></a>Transcripción: Predicción de respuestas con un modelo sencillo
Este es el cuarto vídeo de la serie "Ciencia de datos para principiantes". En este caso, crearemos un modelo simple y realizaremos una predicción.

Un *modelo* es un caso simplificado sobre nuestros datos. Le mostraré lo que quiero decir.

## <a name="collect-relevant-accurate-connected-enough-data"></a>Recopilación de datos pertinentes, precisos, conectados y suficientes
Supongamos que deseo comprar un diamante. Tengo un anillo que pertenecía a mi abuela con un engarce para un diamante de 1,35 quilates, y quiero tener una idea de cuánto costará. Tomo un lápiz y un cuaderno en la joyería y escribo el precio de todos los diamantes de la vitrina y cuántos quilates tienen. Empiezo por el primer diamante: tiene 1,01 quilates y cuesta 7366 USD.

Ahora hago lo mismo para todos los diamantes de la joyería.

![Columnas de datos de los diamantes](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/diamond-data.png)

Observe que nuestra lista tiene dos columnas. Cada columna tiene un atributo diferente, quilates y precio, y cada fila es un único punto de datos que representa un solo diamante.

En realidad, hemos creado un pequeño conjunto de datos aquí: una tabla. Observe que cumple los criterios de calidad:

* Los datos son **pertinentes** : sin duda el peso está relacionado con el precio.
* Son **precisos** : hemos comprobado dos veces los precios que escribimos.
* Están **conectados** : no hay ningún espacio en blanco en ninguna de estas columnas.
* Y, como veremos, hay **suficientes** datos para responder a nuestra pregunta.

## <a name="ask-a-sharp-question"></a>Formulación de una pregunta directa
Ahora platearemos nuestra pregunta de forma directa: "¿Cuánto costará comprar un diamante de 1,35 quilates?"

Nuestra lista no contiene ningún diamante de 1,35 quilates, por lo que debemos utilizar el resto de nuestros datos para obtener una respuesta a la pregunta.

## <a name="plot-the-existing-data"></a>Trazado de los datos existentes
Lo primero que haremos es dibujar una línea horizontal de números, denominada un eje, para colocar los pesos. El intervalo de peso es de 0 a 2, por lo que dibujaremos una línea que cubra ese intervalo y colocaremos marcas para cada medio quilate.

A continuación, dibujaremos un eje vertical para registrar el precio y conectarlo al eje horizontal de peso. Utilizaremos unidades en dólares. Ahora tenemos un conjunto de ejes de coordenadas.

![Ejes de peso y precio](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/weight-and-price-axes.png)

Ahora vamos a tomar estos datos y los convertimos en un *gráfico de dispersión*. Esta es una excelente forma de ver conjuntos de datos numéricos.

Para el primer punto de datos, dibujamos mentalmente una línea vertical en 1,01 quilates. A continuación, dibujamos mentalmente una línea horizontal en 7366 USD. Donde se encuentran, dibujamos un punto. Esto representa nuestro primer diamante.

Ahora podemos hacer lo mismo con cada diamante de esta lista. Cuando hayamos terminado, esto es lo que obtenemos: una serie de puntos, uno para cada diamante.

![gráfico de dispersión](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/scatter-plot.png)

## <a name="draw-the-model-through-the-data-points"></a>Dibujado del modelo siguiendo los puntos de datos
Si observa los puntos con los ojos entrecerrados, la colección parece una línea gruesa y difuminada. Podemos tomar nuestro marcador y dibujar una línea recta a través de ellos.

Al dibujar una línea, hemos creado un *modelo*. Piense en esto como tomar el mundo real y hacer un cómic simplista de él. Ahora el cómic es incorrecto: la línea no pasa por todos los puntos de datos. Pero es una simplificación útil.

![Línea de regresión lineal](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/linear-regression-line.png)

El hecho de que todos los puntos no pasen exactamente por la línea es correcto. Los científicos de datos lo explican diciendo que existe el modelo, que es la línea, y que cada punto tiene cierto *ruido* o *varianza* asociado. Existe la relación subyacente perfecta, y después está la cruda realidad, que agrega ruido e incertidumbre.

Dado que estamos intentando responder a la pregunta *¿cuánto?* , esto se denomina una *regresión*. Y puesto que estamos usando una línea recta, es una *regresión lineal*.

## <a name="use-the-model-to-find-the-answer"></a>Uso del modelo para encontrar la respuesta
Ahora tenemos un modelo y le planteamos nuestra pregunta: ¿Cuánto costará un diamante de 1,35 quilates?

Para responder a la pregunta, calculamos la posición de 1,35 quilates y dibujamos una línea vertical. Donde cruza la línea del modelo, dibujamos una línea horizontal hasta el eje de dólares. Se encuentra justo en 10 000. ¡Bum! Esa es la respuesta: Un diamante de 1,35 quilates cuesta aproximadamente 10 000 USD.

![Encontrar la respuesta en el modelo](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/find-the-answer.png)

## <a name="create-a-confidence-interval"></a>Creación de un intervalo de confianza
Es natural preguntarse si es muy precisa esta predicción. Resulta útil saber si el precio del diamante de 1,35 quilates estará muy cerca de los 10 000 USD, o será mucho mayor o menor. Para averiguarlo, marcaremos una zona alrededor de la línea de regresión que incluya la mayoría de los puntos. Esta zona es nuestro *intervalo de confianza*: estamos bastante seguros de que los precios estarán dentro de esta zona, porque en el pasado, la mayoría de ellos lo han estado. Podemos dibujar dos líneas horizontales más donde la línea de 1,35 quilates cruza la parte superior e inferior de dicha zona.

![intervalo de confianza](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/confidence-interval.png)

Ahora podemos decir algo sobre nuestro intervalo de confianza:  podemos decir con seguridad que el precio de un diamante de 1,35 quilates es aproximadamente de 10 000 USD, con un mínimo de 8000 USD y un máximo de 12 000 USD.

## <a name="were-done-with-no-math-or-computers"></a>Hemos terminado, sin matemáticas ni equipos informáticos.
Hemos hecho lo que hacen los científicos de datos, y lo hemos hecho simplemente con un dibujo:

* Hemos planteado una pregunta a la que hemos podido responder con datos.
* Hemos creado un *modelo* mediante la *regresión lineal*.
* Hemos realizado una *predicción* y la hemos completado con un *intervalo de confianza*.

Y no hemos usamos ni matemáticas ni equipos informáticos para hacerlo.

Ahora bien, si hubiéramos tenido más información, como...

* el corte del diamante
* las variaciones de color (la cercanía al blanco del diamante)
* la cantidad de inclusiones en el diamante

...habríamos tenido más columnas. En ese caso, las matemáticas hubieran sido útiles. Si tiene más de dos columnas, resulta difícil dibujar los puntos en papel. Las matemáticas permiten ajustar esa línea o ese plano a los datos perfectamente.

Además, si en lugar de un puñado de diamantes, tuviéramos dos mil o dos millones, podría hacer ese trabajo de forma mucho más rápida con un equipo informático.

Hoy, hemos hablado sobre cómo realizar la regresión lineal y hemos realizado una predicción con datos.

Asegúrese de consultar los demás vídeos de la serie "Ciencia de datos para principiantes" de Microsoft Azure Machine Learning Studio (clásico).

## <a name="next-steps"></a>Pasos siguientes
* [Prueba de su primer experimento de ciencia de datos con Machine Learning Studio (clásico)](create-experiment.md)
* [Introducción a Machine Learning en Microsoft Azure](/azure/machine-learning/overview-what-is-azure-ml)
