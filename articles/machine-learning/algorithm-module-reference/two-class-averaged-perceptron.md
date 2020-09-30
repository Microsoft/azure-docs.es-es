---
title: 'Perceptrón promedio de dos clases: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo Perceptrón promedio de dos clases en Azure Machine Learning para crear un modelo de Machine Learning basado en el algoritmo de perceptrón promedio.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 04/22/2020
ms.openlocfilehash: 3e5351ed2d400e0b1a10a913bb62391851030053
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907750"
---
# <a name="two-class-averaged-perceptron-module"></a>Módulo Perceptrón promedio de dos clases

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Utilice este módulo para crear un modelo de Machine Learning basado en el algoritmo de perceptrón promedio.  
  
Este algoritmo de clasificación es un método de aprendizaje supervisado y requiere un *conjunto de datos etiquetado*, que incluya una columna de etiqueta. Puede entrenar el modelo proporcionando el modelo y el conjunto de datos etiquetado como entrada para [Entrenar modelo](./train-model.md). Después, el modelo entrenado puede utilizarse para predecir valores para los nuevos ejemplos de entrada.  

### <a name="about-averaged-perceptron-models"></a>Acerca de los modelos de perceptrón promedio de dos clases

El *método de perceptrón promedio* es una versión anterior y sencilla de una red neuronal. En este enfoque, las entradas se clasifican en varias posibles salidas según una función lineal y, a continuación, se combinan con un conjunto de pesos que se derivan del vector de característica, de ahí, el nombre "perceptrón".

Los modelos de perceptrón más sencillos son adecuados para aprender patrones separables linealmente, mientras que las redes neuronales (especialmente las redes neuronales profundas) pueden modelar límites de clase más complejos. Sin embargo, los perceptrones son más rápidos y, puesto que procesan los casos en serie, pueden usarse en el entrenamiento continuo.

## <a name="how-to-configure-two-class-averaged-perceptron"></a>Procedimiento para configurar el perceptrón promedio de dos clases

1.  Agregue el módulo **Perceptrón promedio de dos clases** a la canalización.  

2.  Especifique cómo quiere que se entrene el modelo, estableciendo la opción **Create trainer mode** (Crear modo entrenador).  
  
    -   **Single Parameter** (Parámetro único): Si sabe cómo quiere configurar el modelo, proporcione un conjunto específico de valores como argumentos.

    -   **Parameter Range** (Intervalo de parámetros): seleccione esta opción si no está seguro de los mejores parámetros y quiere ejecutar un barrido de parámetros. Seleccione un rango de valores que iterar y el módulo [Optimización de hiperparámetros de un modelo](tune-model-hyperparameters.md) itera en todas las combinaciones posibles de los valores proporcionados para determinar los hiperparámetros que generan los resultados óptimos.  
  
3.  En **Velocidad de aprendizaje**, especifique un valor para la *velocidad de aprendizaje*. Los valores de velocidad de aprendizaje controlan el tamaño del paso que se utiliza en el descenso de gradiente estocástico cada vez que se prueba y se corrige el modelo.
  
     Al disminuir la velocidad, se prueba el modelo más a menudo, con el riesgo de que pueda quedarse atascado en un nivel local. Al aumentar el paso, puede convergir con mayor rapidez, con el riesgo de superar los mínimos verdaderos.
  
4.  En **Maximum number of iterations** (Número máximo de iteraciones), escriba el número de veces que desea que el algoritmo examine los datos de entrenamiento.  
  
     Si se detiene pronto proporciona, a menudo, una mejor generalización. Aumentar el número de iteraciones mejora el ajuste, con riesgo de sobreajuste.
  
5.  En **Random number seed** (Inicialización de número aleatorio), escriba opcionalmente un valor de entero que se usará como inicialización. Se recomienda usar una inicialización si desea garantizar la reproducibilidad de la canalización a través de las ejecuciones.  
  
1.  Conecte un conjunto de datos de entrenamiento y entrene el modelo:

    + Si establece **Create trainer mode** (Crear modo entrenador) en **Single Parameter** (Parámetro único), conecte un conjunto de datos etiquetado y el módulo [Entrenar modelo](train-model.md).  
  
    + Si establece **Create trainer mode** (Crear el modo de entrenador) en **Parameter Range** (Intervalo de parámetros), conecte un conjunto de datos etiquetado y entrene el modelo mediante [Tune Model Hyperparameters](tune-model-hyperparameters.md) (Optimizar los hiperparámetros del modelo).  
  
    > [!NOTE]
    > 
    > Si pasa un intervalo de parámetros a [Train Model](train-model.md) (Entrenar modelo), solo utiliza el valor predeterminado en la lista de parámetros única.  
    > 
    > Si pasa un único conjunto de valores de parámetro al módulo [Tune Model Hyperparameters](tune-model-hyperparameters.md) (Optimizar los hiperparámetros del modelo), cuando espera un intervalo de valores para cada parámetro, omite los valores y usa los valores predeterminados para el aprendiz.  
    > 
    > Si selecciona la opción **Parameter Range** (Intervalo de parámetros) y especifica un valor único para algún parámetro, ese valor único que haya especificado se utilizará en todo el barrido, incluso si otros parámetros cambian en un intervalo de valores.




## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 