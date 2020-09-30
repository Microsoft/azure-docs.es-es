---
title: 'Multiclass Neural Network: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Multiclass Neural Network (Red neuronal multiclase) de Azure Machine Learning Service para crear un modelo de red neuronal que pueda usarse para predecir un destino que tenga varios valores.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 04/22/2020
ms.openlocfilehash: 1b5ccc9deb3c11d847cd194992d7972f82a5c160
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905289"
---
# <a name="multiclass-neural-network-module"></a>Módulo Multiclass Neural Network

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Use este módulo para crear un modelo de red neuronal que se pueda usar para predecir un destino que tenga varios valores. 

Por ejemplo, las redes neurales de este tipo podrían usarse en tareas complejas de visión informática, como el reconocimiento de números o letras, la clasificación de documentos y el reconocimiento de patrones.

La clasificación mediante redes neuronales es un método de aprendizaje supervisado y, por lo tanto, requiere un *conjunto de datos etiquetado* que incluya una columna de etiquetas.

Puede entrenar el modelo proporcionando el modelo y el conjunto de datos etiquetado como entrada para [Entrenar modelo](./train-model.md). Después, el modelo entrenado puede utilizarse para predecir valores para los nuevos ejemplos de entrada.  

## <a name="about-neural-networks"></a>Acerca de las redes neuronales

Una red neuronal es un conjunto de capas interconectadas. Las entradas son la primera capa y se conectan a una capa de salida mediante un grafo acíclico que consta de nodos y aristas ponderadas.

Entre las capas de entrada y salida puede insertar varias capas ocultas. La mayoría de tareas predictivas pueden realizarse fácilmente con solo una o varias capas ocultas. Sin embargo, las investigaciones recientes han demostrado que las redes neuronales profundas (DNN) con muchas capas pueden ser eficaces en tareas complejas, como en el reconocimiento de imágenes o de voz. Las capas sucesivas se usan para modelar los crecientes niveles de profundidad semántica.

La relación entre entradas y salidas se aprende mediante el entrenamiento de la red neuronal con los datos de entrada. La dirección del grafo procede desde las entradas a través de la capa oculta y hacia la capa de salida. Todos los nodos de una capa se conectan mediante las aristas ponderadas a los nodos de la capa siguiente.

Para calcular la salida de la red para una entrada determinada, se calcula un valor en cada nodo en las capas ocultas y en la capa de salida. El valor se establece calculando la suma ponderada de los valores de los nodos de la capa anterior. A continuación, se aplica una función de activación a esa suma ponderada.

## <a name="configure-multiclass-neural-network"></a>Configurar una red neuronal multiclase

1. Agregue el módulo **MultiClass Neural Network** (Red neuronal multiclase) a la canalización del diseñador. Puede encontrar este módulo en **Machine Learning**, **Inicializar**, en la categoría **Clasificación**.

2. **Create trainer mode** (Crear modo entrenador): Use esta opción para especificar cómo quiere que se entrene al modelo:

    - **Single Parameter** (Parámetro único): Elija esta opción si ya sabe cómo desea configurar el modelo.

    - **Parameter Range** (Intervalo de parámetros): seleccione esta opción si no está seguro de los mejores parámetros y quiere ejecutar un barrido de parámetros. Seleccione un rango de valores que iterar y el módulo [Optimización de hiperparámetros de un modelo](tune-model-hyperparameters.md) itera en todas las combinaciones posibles de los valores proporcionados para determinar los hiperparámetros que generan los resultados óptimos.  

3. **Hidden layer specification** (Especificación de capa oculta): Seleccione el tipo de arquitectura de red que se va a crear.

    - **Fully connected case** (Caso completamente conectado): Seleccione esta opción para crear un modelo con la arquitectura de red neuronal predeterminada. Para los modelos de red neuronal multiclase, los valores predeterminados son los siguientes:

        - Una capa oculta.
        - La capa de salida está conectada por completo a la capa oculta.
        - La capa oculta está conectada por completo a la capa de entrada.
        - El número de nodos de la capa de entrada está determinado por el número de características de los datos de entrenamiento.
        - El usuario puede establecer el número de nodos de la capa oculta. El valor predeterminado es 100.
        - El número de nodos de la capa de salida depende del número de clases.
  
   

5. **Number of hidden nodes** (Número de nodos ocultos): Esta opción le permite personalizar el número de nodos ocultos en la arquitectura predeterminada. Escriba el número de nodos ocultos. El valor predeterminado es una capa oculta con 100 nodos.

6. **Velocidad de aprendizaje**: Defina el tamaño del paso llevado a cabo en cada iteración, antes de la corrección. Un valor mayor para la velocidad de aprendizaje puede hacer que el modelo converja con mayor rapidez, pero puede superar los mínimos locales.

7. **Number of learning iterations** (Número de iteraciones de aprendizaje): Especifique el número máximo de veces que el algoritmo debe procesar los casos de entrenamiento.

8. **The initial learning weights diameter** (El diámetro de pesos de aprendizaje inicial): Escriba un valor que determine los pesos de nodo en el inicio del proceso de aprendizaje.

9. **The momentum** (El momento): Especifique una ponderación que se debe aplicar durante el aprendizaje a los nodos de iteraciones anteriores.
  
11. **Shuffle examples** (Ordenar ejemplos aleatoriamente): Seleccione esta opción para ordenar de forma aleatoria los casos entre iteraciones.

    Si anula la selección de esta opción, los casos se procesan exactamente en el mismo orden cada vez que se ejecuta la canalización.

12. **Número de iniciación aleatorio**: Escriba un valor para usar como inicialización si desea asegurar la repetibilidad entre ejecuciones de la misma canalización.

14. Entrenamiento del modelo:

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

- Para guardar una instantánea del modelo entrenado, seleccione la pestaña **Outputs** (Salidas) en el panel derecho del módulo **Train model** (Entrenar modelo). Seleccione el icono **Register dataset** (Registrar conjunto de datos) para guardar el modelo como un módulo reutilizable.

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 