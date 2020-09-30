---
title: 'Máquina de vectores de soporte de dos clases: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo de **Máquina de vectores de soporte de dos clases** en Azure Machine Learning para crear un modelo basado en el algoritmo de máquina de vectores de soporte.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 04/22/2020
ms.openlocfilehash: 1dc2f8bdf6ed2823b44d25bdf65766b1f7ae060c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907663"
---
# <a name="two-class-support-vector-machine-module"></a>Máquina de vectores de soporte de dos clases

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Use este módulo para crear un modelo basado en el algoritmo de máquina de vectores de soporte. 

Las máquinas de vectores de soporte (SVM) son una clase contrastada de métodos de aprendizaje supervisado. Esta implementación determinada es adecuada para la predicción de dos resultados posibles basados en las variables continuas o de categoría.

Después de definir los parámetros del modelo, entrene el modelo mediante los módulos de entrenamiento y proporcione un *conjunto de datos etiquetado* que incluya una columna de etiqueta o resultado.

## <a name="about-support-vector-machines"></a>Acerca de las máquinas de vectores de soporte

Las máquinas de vectores de soporte están entre los primeros algoritmos de aprendizaje automático, y los modelos de SVM se han usado en muchas aplicaciones, desde la recuperación de información a la clasificación de texto e imágenes. Las SVM pueden usarse para tareas de clasificación y regresión.

Este modelo de SVM es un modelo de aprendizaje supervisado que requiere datos etiquetados. En el proceso de entrenamiento, el algoritmo analiza los datos de entrada y reconoce patrones en un espacio de características multidimensionales denominado *hiperplano*.  Todos los ejemplos de entrada se representan como puntos en este espacio y se asignan a categorías de salida, de tal manera que las categorías se dividen de forma tan amplia y clara como sea posible.

Para la predicción, el algoritmo de SVM asigna ejemplos nuevos a una categoría u otra, asignándolos en ese mismo espacio. 

## <a name="how-to-configure"></a>Cómo se configura 

Para este tipo de modelo, se recomienda normalizar el conjunto de datos antes de usarlo para entrenar al clasificador.
  
1.  Agregue el módulo **Máquina de vectores de soporte de dos clases** a la canalización.  
  
2.  Especifique cómo quiere que se entrene el modelo, estableciendo la opción **Create trainer mode** (Crear modo entrenador).  
  
    -   **Single Parameter** (Parámetro único): Si sabe cómo quiere configurar el modelo, puede proporcionar un conjunto específico de valores como argumentos.  

    -   **Parameter Range** (Intervalo de parámetros): si no está seguro de los mejores parámetros, puede encontrar los óptimos mediante el módulo [Optimización de hiperparámetros de un modelo](tune-model-hyperparameters.md). Proporcione un rango de valores y el entrenador itera varias combinaciones de ellos para determinar la combinación de valores que genera el mejor resultado.

3.  En **Número de iteraciones**, escriba un número que indique el número de iteraciones usadas al compilar el modelo.  
  
     Este parámetro puede usarse para controlar el equilibrio entre velocidad de entrenamiento y precisión.  
  
4.  En **Lambda**, escriba un valor que se usará como peso para la regularización L1.  
  
     Este coeficiente de regularización puede usarse para optimizar el modelo. Los valores mayores penalizan modelos más complejos.  
  
5.  Seleccione la opción **Normalize features** (Normalizar características) si quiere normalizar las características antes del entrenamiento.
  
     Si aplica la normalización antes del entrenamiento, los puntos de datos se centran según la media y se escalan para que tengan una unidad de desviación estándar.
  
6.  Seleccione la opción **Project to the unit sphere** (Proyectar en la esfera de la unidad) para normalizar los coeficientes.
  
     La proyección de valores en el espacio de la unidad significa que, antes del entrenamiento, los puntos de datos se centran en 0 y se escalan para que tengan una unidad de desviación estándar.
  
7.  En **Random number seed** (Inicialización de números aleatorios), escriba un valor entero para usarlo como valor de inicialización si quiere garantizar la reproducibilidad entre ejecuciones.  En caso contrario, se usa un valor del reloj del sistema como valor de inicialización, que puede dar lugar a resultados ligeramente diferentes entre ejecuciones.
  
9. Conecte un conjunto de datos etiquetado y entrene el modelo:

    + Si establece **Create trainer mode** (Crear modo entrenador) en **Single Parameter** (Parámetro único), conecte un conjunto de datos etiquetado y el módulo [Entrenar modelo](train-model.md).  
  
    + Si establece **Create trainer mode** (Crear el modo de entrenador) en **Parameter Range** (Intervalo de parámetros), conecte un conjunto de datos etiquetado y entrene el modelo mediante [Tune Model Hyperparameters](tune-model-hyperparameters.md) (Optimizar los hiperparámetros del modelo).  
  
    > [!NOTE]
    > 
    > Si pasa un intervalo de parámetros a [Train Model](train-model.md) (Entrenar modelo), solo utiliza el valor predeterminado en la lista de parámetros única.  
    > 
    > Si pasa un único conjunto de valores de parámetro al módulo [Tune Model Hyperparameters](tune-model-hyperparameters.md) (Optimizar los hiperparámetros del modelo), cuando espera un intervalo de valores para cada parámetro, omite los valores y usa los valores predeterminados para el aprendiz.  
    > 
    > Si selecciona la opción **Parameter Range** (Intervalo de parámetros) y especifica un valor único para algún parámetro, ese valor único que haya especificado se utilizará en todo el barrido, incluso si otros parámetros cambian en un intervalo de valores.
  
10. Envíe la canalización.

## <a name="results"></a>Results

Una vez completado el entrenamiento:

+ Para guardar una instantánea del modelo entrenado, seleccione la pestaña **Outputs** (Salidas) en el panel derecho del módulo **Train model** (Entrenar modelo). Seleccione el icono **Register dataset** (Registrar conjunto de datos) para guardar el modelo como un módulo reutilizable.

+ Para usar el modelo de puntuación, agregue el módulo **Score Model** (Puntuar modelo) a una canalización.


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 