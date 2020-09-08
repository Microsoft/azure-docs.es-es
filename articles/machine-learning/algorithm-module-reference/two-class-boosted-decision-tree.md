---
title: 'Árbol de decisión ampliado de dos clases: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo de árbol de decisión ampliado de dos clases en Azure Machine Learning para crear un modelo de Machine Learning basado en el algoritmo de árboles de decisión ampliados.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 08/24/2020
ms.openlocfilehash: 1e66774d4239bdad6c8af426a28af6081bb0da3c
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88782182"
---
# <a name="two-class-boosted-decision-tree-module"></a>Módulo de árbol de decisión ampliado de dos clases

En este artículo se describe un módulo del diseñador de Azure Machine Learning (versión preliminar).

Utilice este módulo para crear un modelo de Machine Learning basado en el algoritmo de árboles de decisión ampliados. 

Un árbol de decisión ampliado es un método de aprendizaje de conjuntos en que el segundo árbol corrige los errores del primer árbol, el tercer árbol corrige los errores del primer y del segundo árbol y así sucesivamente. Las predicciones se basan en el conjunto completo de árboles juntos que establecen la predicción.
  
Por lo general, cuando se configuran correctamente, los árboles de decisión ampliados son los métodos más sencillos para obtener el máximo rendimiento en una amplia variedad de tareas de aprendizaje automático. Sin embargo, también son uno de los elementos aprendices que consumen más recursos de memoria y la implementación actual retiene todo en la memoria. Por tanto, un modelo de árbol de decisión ampliado podría no ser capaz de procesar los grandes conjuntos de datos que algunos elementos aprendices lineales pueden administrar.

Este módulo se basa en el algoritmo LightGBM.

## <a name="how-to-configure"></a>Cómo se configura

En este módulo se crea un modelo de clasificación no entrenado. Dado que la clasificación es un método de aprendizaje supervisado, para entrenar el modelo, necesita un *conjunto de datos etiquetado* que incluye una columna de etiqueta con un valor para todas las filas.

Puede entrenar este tipo de modelo mediante [Entrenar modelo](././train-model.md). 

1.  En Azure Machine Learning, agregue el módulo **Boosted Decision Tree** (Árbol de decisión ampliado) a la canalización.
  
2.  Especifique cómo quiere que se entrene el modelo, estableciendo la opción **Create trainer mode** (Crear modo entrenador).
  
    + **Single Parameter** (Parámetro único): Si sabe cómo quiere configurar el modelo, puede proporcionar un conjunto específico de valores como argumentos.
  
    + **Parameter Range** (Intervalo de parámetros): si no está seguro de los mejores parámetros, puede encontrar los óptimos mediante el módulo [Optimización de hiperparámetros de un modelo](tune-model-hyperparameters.md). Proporcione un rango de valores y el entrenador itera varias combinaciones de ellos para determinar la combinación de valores que genera el mejor resultado.
  
3.  En **Número máximo de hojas por árbol**, indique el número máximo de nodos terminales (hojas) que se pueden crear en un árbol.
  
     Al aumentar este valor, podría aumentar el tamaño del árbol y obtener una mayor precisión, a riesgo de experimentar un sobreajuste y un mayor tiempo de entrenamiento.
  
4.  En **Número mínimo de muestras por nodo hoja**, indique el número de casos necesario para crear cualquier nodo terminal (hoja) en un árbol.  
  
     Al aumentar este valor, aumenta el umbral para crear reglas nuevas. Por ejemplo, con el valor predeterminado de 1, incluso un solo caso puede provocar que se cree una regla nueva. Si aumenta el valor a 5, los datos de entrenamiento tendrían que contener cinco casos como mínimo que cumplan las mismas condiciones.
  
5.  En **Velocidad de aprendizaje**, escriba un número entre 0 y 1 que defina el tamaño de paso durante el aprendizaje.  
  
     La velocidad de aprendizaje determina la rapidez o lentitud con la que el aprendiz converge en la solución óptima. Si el tamaño del paso es demasiado grande, puede pasar por alto la solución óptima. Si el tamaño del paso es demasiado pequeño, el entrenamiento tarda más tiempo en converger en la mejor solución.
  
6.  En **Número de árboles construidos**, indique el número total de árboles de decisión que se van a crear en el conjunto. Si crea más árboles de decisión, puede obtener una cobertura potencialmente mejor, pero aumentará el tiempo de entrenamiento.
  
     Este valor también controla el número de árboles que se muestran al visualizar el modelo entrenado. Si quiere ver o imprimir un único árbol, establezca el valor en 1. Sin embargo, al hacerlo, solo se produce un único árbol (el árbol con el conjunto inicial de parámetros) y no se realizan iteraciones adicionales.
  
7.  En **Inicialización con número aleatorio**, escriba opcionalmente un número entero no negativo para que se use como valor de inicialización aleatorio. Al especificar un valor, se garantiza la reproducibilidad durante las ejecuciones que tienen los mismos datos y parámetros.  
  
     De forma predeterminada, el valor de inicialización aleatorio se establece en 0, lo que significa que el valor de inicialización inicial se obtiene a partir del reloj del sistema.  Las ejecuciones sucesivas, con un valor de inicialización aleatorio, pueden tener resultados diferentes.
  

9. Entrenamiento del modelo:

    + Si establece **Create trainer mode** (Crear modo entrenador) en **Single Parameter** (Parámetro único), conecte un conjunto de datos etiquetado y el módulo [Entrenar modelo](train-model.md).  
  
    + Si establece **Create trainer mode** (Crear el modo de entrenador) en **Parameter Range** (Intervalo de parámetros), conecte un conjunto de datos etiquetado y entrene el modelo mediante [Tune Model Hyperparameters](tune-model-hyperparameters.md) (Optimizar los hiperparámetros del modelo).  
  
    > [!NOTE]
    > 
    > Si pasa un intervalo de parámetros a [Train Model](train-model.md) (Entrenar modelo), solo utiliza el valor predeterminado en la lista de parámetros única.  
    > 
    > Si pasa un único conjunto de valores de parámetro al módulo [Tune Model Hyperparameters](tune-model-hyperparameters.md) (Optimizar los hiperparámetros del modelo), cuando espera un intervalo de valores para cada parámetro, omite los valores y usa los valores predeterminados para el aprendiz.  
    > 
    > Si selecciona la opción **Parameter Range** (Intervalo de parámetros) y especifica un valor único para algún parámetro, ese valor único que haya especificado se utilizará en todo el barrido, incluso si otros parámetros cambian en un intervalo de valores.  
   
## <a name="results"></a>Results

Una vez completado el entrenamiento:

+ Para guardar una instantánea del modelo entrenado, seleccione la pestaña **Outputs** (Salidas) en el panel derecho del módulo **Train model** (Entrenar modelo). Seleccione el icono **Register dataset** (Registrar conjunto de datos) para guardar el modelo como un módulo reutilizable.

+ Para usar el modelo de puntuación, agregue el módulo **Score Model** (Puntuar modelo) a una canalización.

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 