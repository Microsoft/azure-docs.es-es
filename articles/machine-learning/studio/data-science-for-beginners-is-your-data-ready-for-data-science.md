---
title: 'ML Studio (clásico): Evaluación de datos: Azure'
description: Cuatro criterios que sus datos deben cumplir para estar listos para la ciencia de datos. Este vídeo presenta ejemplos concretos para facilitar la evaluación de datos básicos.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: sdgilley
ms.author: sgilley
ms.custom: seodec18
ms.date: 03/22/2019
ms.openlocfilehash: 12e939b8fb66b3cffbee49a4b3bedd74b71d7232
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318766"
---
# <a name="is-your-data-ready-for-data-science"></a>¿Están sus datos preparados para la ciencia de datos?
## <a name="video-2-data-science-for-beginners-series"></a>Vídeo 2: Ciencia de datos para principiantes
Aprenda cómo evaluar los datos para asegurarse de que cumplen los criterios básicos para la ciencia de datos.

Para obtener el máximo partido de la serie, véalos en orden. [Ir a la lista de vídeos](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Shows/SupervisionNotRequired/9/player]
>
>

## <a name="other-videos-in-this-series"></a>Otros vídeos de la serie
*Ciencia de datos para principiantes* es una introducción rápida a la ciencia de datos en cinco vídeos de corta duración.

* Vídeo 1: [Las cinco preguntas a las que responde la ciencia de datos](data-science-for-beginners-the-5-questions-data-science-answers.md) *(5 minutos y 14 segundos)*
* Vídeo 2: ¿Están sus datos preparados para la ciencia de datos?
* Vídeo 3: [Realización de preguntas que pueden responderse con datos](data-science-for-beginners-ask-a-question-you-can-answer-with-data.md) *(4 minutos y 17 segundos)*
* Vídeo 4: [Predicción de respuestas con un modelo sencillo](data-science-for-beginners-predict-an-answer-with-a-simple-model.md) *(7 minutos y 42 segundos)*
* Vídeo 5: [Copia del trabajo de otras personas para realizar ciencia de datos](data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md) *(3 minutos y 18 segundos)*

## <a name="transcript-is-your-data-ready-for-data-science"></a>Transcripción: ¿Están sus datos preparados para la ciencia de datos?
¿Están sus datos preparados para la ciencia de datos? es el segundo vídeo de la serie *Ciencia de datos para principiantes*.  

Antes de que la ciencia de datos pueda proporcionar las respuestas que desea, tendrá que darle materia prima de gran calidad con la que trabajar. Al igual que al hacer una pizza, cuanto mejor sean los ingredientes con los que empiece, mejor será el producto final. 

## <a name="criteria-for-data"></a>Criterios para los datos
En ciencia de datos, hay ciertos ingredientes que deben extraerse juntos, como por ejemplo:

* Pertinentes
* Conectado
* Precisos
* Suficientes para trabajar con ellos

## <a name="is-your-data-relevant"></a>¿Son sus datos pertinentes?
El primer ingrediente: necesitamos que los datos sean relevantes.

![Datos pertinentes frente a datos no pertinentes: evaluar los datos](./media/data-science-for-beginners-is-your-data-ready-for-data-science/relevant-and-irrelevant-data.png)

En el lado izquierdo, la tabla presenta el nivel de alcohol en sangre de siete personas a las que se les hizo la prueba de alcoholemia fuera de un bar de Boston, el promedio de bateo de los Red Sox en su último partido y el precio de la leche en la tienda más cercana.

Todos ellos son datos perfectamente legítimos. El único problema es que no son pertinentes. No hay ninguna relación obvia entre estos números. Si alguien le da el precio actual de la leche y el promedio de bateo de los Red Sox, no hay forma de que pueda adivinar su contenido de alcohol en sangre.

Ahora observe la tabla de la derecha. Esta vez, se midió la masa corporal de cada persona y contamos el número de bebidas que consumieron.  Los números de cada fila ahora son pertinentes entre sí. Si le di mi masa corporal y el número de margaritas he bebido, podría realizar una estimación de mi contenido de alcohol en sangre.

## <a name="do-you-have-connected-data"></a>¿Tiene datos conectados?
El siguiente ingrediente son unos datos conectados.

![Datos conectados frente a datos desconectados: criterios de datos y datos preparados](./media/data-science-for-beginners-is-your-data-ready-for-data-science/connected-vs-disconnected-data.png)

Estos son algunos datos que afectan a la calidad de las hamburguesas: temperatura de la parrilla, el peso de la carne y la clasificación en la revista de comida local. Pero observe los espacios vacíos en la tabla de la izquierda.

A la mayoría de los conjuntos de datos les faltan algunos valores. Es habitual tener espacios vacíos como estos y hay formas de trabajar para solucionarlos. Pero si faltan demasiados, los datos se empiezan a parecerse a un queso suizo.

Si observa la tabla de la izquierda, faltan tantos datos que es difícil establecer cualquier tipo de relación entre la temperatura de la parrilla y el peso de la carne. En este ejemplo se muestran datos desconectados.

Sin embargo, la tabla de la derecha está llena y completa; un ejemplo de datos conectados.

## <a name="is-your-data-accurate"></a>¿Son precisos los datos?
El siguiente ingrediente es la precisión. Estos son los cuatro objetivos que hay que alcanzar.

![Datos precisos frente a datos imprecisos: criterios de datos](./media/data-science-for-beginners-is-your-data-ready-for-data-science/inaccurate-vs-accurate-data.png)

Mire la diana en la parte superior derecha. Existe una agrupación apretada justo en el blanco. Esto, por supuesto, es preciso. Curiosamente, en el lenguaje de la ciencia de datos, el rendimiento en la diana inferior derecha también se considera preciso.

Si señaló el centro de estas flechas, verá que está muy cerca del blanco. Las flechas se reparten alrededor de la diana, por lo que se consideran imprecisas, pero están centradas alrededor del blanco, por lo que se consideran precisas.

Ahora examinemos la diana superior izquierda. Aquí las flechas dieron todas muy cerca, en una agrupación apretada. Son precisas, pero no lo son porque el centro está muy lejos del blanco. Las flechas en la diana inferior izquierda son inexactas e imprecisas. Este arquero necesita más práctica.

## <a name="do-you-have-enough-data-to-work-with"></a>¿Tiene suficientes datos con los que trabajar?
Por último, el cuarto ingrediente son datos suficientes.

![¿Tiene suficientes datos para el análisis? Evaluación de datos](./media/data-science-for-beginners-is-your-data-ready-for-data-science/barely-enough-data.png)

Piense que cada punto de datos de la tabla es una pincelada en una pintura. Si solo tiene algunas, la pintura puede ser difuminada; es difícil saber qué es.

Si agrega algunas pinceladas más, la pintura empieza a estar un poco más nítida.

Cuando ya casi haya suficientes pinceladas, solo podrá ver lo suficiente como para tomar algunas decisiones generales. ¿Es un lugar que me gustaría visitar? Parece soleado y con agua limpia; sí, ahí es a donde voy a ir de vacaciones.

A medida que agrega más datos, la imagen se vuelve más clara y puede tomar decisiones más detalladas. Ahora puede ver los tres hoteles en la orilla izquierda. Puede percibir las características arquitectónicas del hotel en primer plano. Incluso puede elegir quedarse en la tercera planta debido a la vista.

Con datos pertinentes, conectados, precisos y suficientes, tiene todos los ingredientes necesarios para llevar a cabo ciencia de datos de gran calidad.

Asegúrese de ver los otros cuatro vídeos de *Ciencia de datos para principiantes* en Microsoft Azure Machine Learning Studio (clásico).

## <a name="next-steps"></a>Pasos siguientes
* [Prueba de su primer experimento de ciencia de datos con Machine Learning Studio (clásico)](create-experiment.md)
* [Introducción a Machine Learning en Microsoft Azure](/azure/machine-learning/overview-what-is-azure-ml)
