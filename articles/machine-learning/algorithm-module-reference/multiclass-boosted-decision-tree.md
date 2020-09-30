---
title: 'Árbol de decisión ampliado multiclase: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Multiclass Boosted Decision Tree (Árbol de decisión ampliado multiclase) en Azure Machine Learning para crear un clasificador mediante datos etiquetados.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/19/2020
ms.openlocfilehash: 1a1cb7661ae01dd89d45afe004978813ac90eaff
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905311"
---
# <a name="multiclass-boosted-decision-tree"></a>Árbol de decisión ampliado multiclase

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Utilice este módulo para crear un modelo de Machine Learning basado en el algoritmo de árboles de decisión ampliados.

Un árbol de decisión ampliado es un método de aprendizaje de conjuntos en que el segundo árbol corrige los errores del primer árbol, el tercer árbol corrige los errores del primer y del segundo árbol y así sucesivamente. Las predicciones se basan en el conjunto de árboles juntos.

## <a name="how-to-configure"></a>Cómo se configura 

En este módulo se crea un modelo de clasificación no entrenado. Dado que la clasificación es un método de aprendizaje supervisado, necesita un *conjunto de datos etiquetado* que incluya una columna de etiqueta con un valor para todas las filas.

Puede entrenar este tipo de modelo mediante [Entrenar modelo](././train-model.md). 

1.  Agregue el módulo **Árbol de decisión ampliado multiclase** a la canalización.

1.  Especifique cómo quiere que se entrene el modelo. Para ello, establezca la opción **Create trainer mode** (Crear modo entrenador).

    + **Single Parameter** (Parámetro único): Si sabe cómo quiere configurar el modelo, puede proporcionar un conjunto específico de valores como argumentos.
    
    + **Parameter Range** (Intervalo de parámetros): seleccione esta opción si no está seguro de los mejores parámetros y quiere ejecutar un barrido de parámetros. Seleccione un rango de valores que iterar y el módulo [Optimización de hiperparámetros de un modelo](tune-model-hyperparameters.md) itera en todas las combinaciones posibles de los valores proporcionados para determinar los hiperparámetros que generan los resultados óptimos.  

1. **Maximum number of leaves per tree** (Número máximo de hojas por árbol) limita el número máximo de nodos terminales (hojas) que se pueden crear en un árbol.
    
    Al aumentar este valor, podría aumentar el tamaño del árbol y obtener una mayor precisión, a riesgo de experimentar un sobreajuste y un mayor tiempo de entrenamiento.
  
1. **Minimum number of samples per leaf node** (Número mínimo de muestras por nodo hoja) indica el número de casos necesarios para crear cualquier nodo terminal (hoja) en un árbol.  

    Al aumentar este valor, aumenta el umbral para crear reglas nuevas. Por ejemplo, con el valor predeterminado de 1, incluso un solo caso puede provocar que se cree una regla nueva. Si aumenta el valor a 5, los datos de entrenamiento tendrían que contener cinco casos como mínimo que cumplan las mismas condiciones.

1. La **velocidad de aprendizaje** define el tamaño del paso durante el aprendizaje. Especifique un número comprendido entre 0 y 1.

    La velocidad de aprendizaje determina la rapidez o la lentitud con la que el aprendiz converge en una solución óptima. Si el tamaño del paso es demasiado grande, puede pasar por alto la solución óptima. Si el tamaño del paso es demasiado pequeño, el entrenamiento tarda más tiempo en converger en la mejor solución.

1. **Number of trees constructed** (Número de árboles construidos) indica el número total de árboles de decisión que se van a crear en el conjunto. Si crea más árboles de decisión, puede obtener una cobertura potencialmente mejor, pero aumentará el tiempo de entrenamiento.

1. **Random number seed** (Inicialización con número aleatorio) establece opcionalmente un número entero no negativo para que se use como valor de inicialización aleatorio. Al especificar un valor, se garantiza la reproducibilidad durante las ejecuciones que tienen los mismos datos y parámetros.  

    El valor de inicialización aleatorio se establece de forma predeterminada en 42. Las ejecuciones sucesivas que usen un valor de inicialización aleatorio diferente podrían tener resultados diferentes.

1. Entrenamiento del modelo:

    + Si establece **Create trainer mode** (Crear modo entrenador) en **Single Parameter** (Parámetro único), conecte un conjunto de datos etiquetado y el módulo [Entrenar modelo](train-model.md).  
  
    + Si define **Create trainer mode** (Crear modo de entrenador) como **Parameter Range** (Rango de parámetros), conecte un conjunto de datos etiquetado y entrene el modelo mediante [Tune Model Hyperparameters](tune-model-hyperparameters.md) (Hiperparámetros de modelo de ajuste).  
  
    > [!NOTE]
    > 
    > Si pasa un intervalo de parámetros a [Train Model](train-model.md) (Entrenar modelo), solo utiliza el valor predeterminado en la lista de parámetros única.  
    > 
    > Si pasa un único conjunto de valores de parámetro al módulo [Tune Model Hyperparameters](tune-model-hyperparameters.md) (Optimizar los hiperparámetros del modelo), cuando espera un intervalo de valores para cada parámetro, omite los valores y usa los valores predeterminados para el aprendiz.  
    > 
    > Si selecciona la opción **Parameter Range** (Intervalo de parámetros) y especifica un valor único para algún parámetro, ese valor único que haya especificado se utilizará en todo el barrido, incluso si otros parámetros cambian en un intervalo de valores.

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
