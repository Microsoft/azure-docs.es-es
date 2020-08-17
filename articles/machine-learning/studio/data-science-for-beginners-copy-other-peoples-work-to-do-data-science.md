---
title: 'Ejemplos de copia de ciencia de datos: ML Studio (clásico): Azure'
description: 'Secreto comercial de ciencia de datos: Consiga que otras personas hagan el trabajo por usted. Obtenga ejemplos de Machine Learning en Azure AI Gallery.'
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: sdgilley
ms.author: sgilley
ms.custom: seodec18
ms.date: 03/22/2019
ms.openlocfilehash: 59343c500f50430530b24fbfb7d747108377efd8
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318783"
---
# <a name="copy-other-peoples-work-to-do-data-science"></a>Copia del trabajo de otras personas para realizar ciencia de datos
## <a name="video-5-data-science-for-beginners-series"></a>Vídeo 5: Ciencia de datos para principiantes
Uno de los secretos empresariales de la ciencia de datos es conseguir que otras personas hagan el trabajo por usted. Vea un ejemplo de algoritmo de agrupación en clústeres de Azure AI Gallery que usará en su propio experimento de Machine Learning.

> [!IMPORTANT]
> La **Galería de Cortana Intelligence** ha cambiado de nombre a **Azure AI Gallery**. En consecuencia, el texto y las imágenes de esta transcripción son ligeramente distintos a los del vídeo, en el que se utiliza el nombre anterior.
>

Para obtener el máximo partido de la serie, véalos en orden. [Ir a la lista de vídeos](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/data-science-for-beginners-series-copy-other-peoples-work-to-do-data-science/player]
>
>

## <a name="other-videos-in-this-series"></a>Otros vídeos de la serie
*Ciencia de datos para principiantes* es una introducción rápida a la ciencia de datos en cinco vídeos de corta duración.

* Vídeo 1: [Las cinco preguntas a las que responde la ciencia de datos](data-science-for-beginners-the-5-questions-data-science-answers.md) *(5 minutos y 14 segundos)*
* Vídeo 2: [¿Están sus datos preparados para la ciencia de datos?](data-science-for-beginners-is-your-data-ready-for-data-science.md) *(4 minutos y 56 segundos)*
* Vídeo 3: [Realización de preguntas que pueden responderse con datos](data-science-for-beginners-ask-a-question-you-can-answer-with-data.md) *(4 minutos y 17 segundos)*
* Vídeo 4: [Predicción de respuestas con un modelo sencillo](data-science-for-beginners-predict-an-answer-with-a-simple-model.md) *(7 minutos y 42 segundos)*
* Vídeo 5: Copia del trabajo de otras personas para realizar ciencia de datos

## <a name="transcript-copy-other-peoples-work-to-do-data-science"></a>Transcripción: Copia del trabajo de otras personas para realizar ciencia de datos
Este es el quinto vídeo de la serie "Ciencia de datos para principiantes".

En este caso, descubrirá un lugar donde puede buscar ejemplos que puede usar como punto de partida para su propio trabajo. Aprovechará al máximo este vídeo si ve primero los vídeos anteriores de esta serie.

Uno de los secretos empresariales de la ciencia de datos es conseguir que otras personas hagan el trabajo por usted.

## <a name="find-examples-in-the-azure-ai-gallery"></a>Búsqueda de ejemplos en Azure AI Gallery

Microsoft tiene un servicio basado en la nube llamado [Azure Machine Learning Studio (clásico)](https://azure.microsoft.com/services/machine-learning-studio/). Proporciona un área de trabajo donde puede experimentar con diferentes algoritmos de aprendizaje automático y, si consigue una solución que funciona, puede iniciarla como un servicio web.

Una parte de este servicio recibe el nombre de **[Galería de Azure AI](https://gallery.azure.ai/)** . Contiene recursos, incluida una colección de experimentos o modelos de Azure Machine Learning Studio (clásico), que los usuarios han compilado y con los que han ayudado para que los usen otros usuarios. Estos experimentos son una excelente manera de aprovechar las ideas y el duro trabajo de otras personas para empezar a crear sus propias soluciones. Todo el mundo puede consultarla.

![Galería de Azure AI](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/azure-ai-gallery.png)

Si hace clic en **Experiments** (Experimentos) en la parte superior, verá los experimentos más recientes y populares de la galería. Puede buscar consultar los demás experimentos haciendo clic en **Browse All** (Examinar todo) en la parte superior de la pantalla, ahí puede escribir términos de búsqueda y elegir filtros de búsqueda.

## <a name="find-and-use-a-clustering-algorithm-example"></a>Localización y uso de un ejemplo de algoritmo de clústeres
Por ejemplo, supongamos que desea ver un ejemplo de cómo funcionan los clústeres, por lo que busca experimentos de **"clustering sweep"** .

![Búsqueda de experimentos de clústeres](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/search-for-clustering-experiments.png)

Aquí hay algo interesante que alguien ha aportado a la galería.

![Experimento de clústeres](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/clustering-experiment.png)

Haga clic en ese experimento para obtener una página web en la que se describe el trabajo que hizo este colaborador, junto con algunos de sus resultados.

![Página de descripción del experimento de clústeres](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/clustering-experiment-description-page.png)

Observe el vínculo que dice **Abrir en Studio (clásico)** .

![Botón Abrir en Studio (clásico)](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/open-in-studio.png)

Puedo hacer clic en él y me lleva directamente a **Azure Machine Learning Studio (clásico)** . Crea una copia del experimento y la coloca en mi propia área de trabajo. Incluye el conjunto de datos del colaborador, todo el procesamiento que realizaron, todos los algoritmos que usaron y cómo se guardaron los resultados.

![Abra un experimento de la Galería en Machine Learning Studio (clásico) (ejemplo de algoritmo de agrupación en clústeres)](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/cluster-experiment-open-in-studio.png)

Y ahora tengo un punto de partida. Puedo cambiar sus datos por los míos y retocar el modelo. Esto me proporciona un punto de partida que funciona y me permite trabajar aprovechando el trabajo de las personas que realmente saben lo que hacen.

## <a name="find-experiments-that-demonstrate-machine-learning-techniques"></a>Búsqueda de experimentos que muestran técnicas de aprendizaje automático
Hay otros experimentos en [Azure AI Gallery](https://gallery.azure.ai) incluidos específicamente para proporcionar ejemplos de procedimientos para personas nuevas en la ciencia de datos. Por ejemplo, hay un experimento en la galería que muestra qué hacer cuando faltan valores ([Methods for handling missing values](https://gallery.azure.ai/Experiment/Methods-for-handling-missing-values-1)[Métodos para gestionar valores que faltan]). Le guía por 15 maneras diferentes de sustituir valores vacíos, y habla sobre las ventajas de cada método y cuándo usarlo.

![Experimento de la Galería de Machine Learning Studio (clásico): métodos para valores ausentes](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/experiment-methods-for-handling-missing-values.png)

[Azure AI Gallery](https://gallery.azure.ai) es un lugar para buscar experimentos que funcionan y que puede usar como punto de partida para sus propias soluciones.

Asegúrese de consultar los demás vídeos de la serie "Ciencia de datos para principiantes" de Microsoft Azure Machine Learning Studio (clásico).

## <a name="next-steps"></a>Pasos siguientes
* [Prueba del primer experimento de ciencia de datos con Azure Machine Learning Studio (clásico)](create-experiment.md)
* [Introducción a Machine Learning en Microsoft Azure](/azure/machine-learning/overview-what-is-azure-ml)
