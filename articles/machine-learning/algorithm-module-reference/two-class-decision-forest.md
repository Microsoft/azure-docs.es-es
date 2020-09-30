---
title: 'Bosque de decisión de dos clases: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo de bosque de decisión de dos clases en Azure Machine Learning para crear un modelo de aprendizaje automático basado en el algoritmo de bosques de decisión.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 09/03/2020
ms.openlocfilehash: 9870b5366ac629fe1b10cfde15a58b85c7c6fbc3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907725"
---
# <a name="two-class-decision-forest-module"></a>Módulo de bosque de decisión de dos clases

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Utilice este módulo para crear un modelo de Machine Learning basado en el algoritmo de bosques de decisión.  

Los bosques de decisión son modelos de conjunto rápidos y supervisados. Este módulo es una buena opción si quiere predecir un destino con un máximo de dos resultados. 

## <a name="understanding-decision-forests"></a>Descripción de los bosques de decisión

Este algoritmo de bosque de decisión es un método de aprendizaje de conjunto diseñado para las tareas de clasificación. Los métodos de conjunto se basan en el principio general de que, en lugar de depender de un único modelo, puede obtener mejores resultados y un modelo más generalizado mediante la creación de varios modelos relacionados y combinarlos de alguna manera. Por lo general, los modelos de conjunto proporcionan mejor cobertura y precisión que los árboles de decisión únicos. 

Hay muchas maneras de crear modelos individuales y combinarlos en un conjunto. En esta implementación concreta de un bosque de decisión se crean varios árboles de decisión y, a continuación, **se vota** la clase de resultado más popular. Votar es uno de los métodos más conocidos para generar resultados en un modelo de conjunto. 

+ Se crean muchos árboles de clasificación individuales, con todo el conjunto de datos, pero puntos de partida diferentes (normalmente aleatorios). Esto difiere del enfoque de bosque aleatorio, en el que los árboles de decisión individuales pueden usar solo una parte aleatoria de los datos o de las características.
+ Cada árbol del bosque de decisión produce un histograma de etiquetas de frecuencia no normalizada. 
+ El proceso de agregación suma estos histogramas y normaliza el resultado para obtener las "probabilidades" de cada etiqueta. 
+ Los árboles con un nivel alto de confianza en la predicción tendrán un peso mayor en la decisión final del conjunto.

En general, los árboles de decisión tienen numerosas ventajas para las tareas de clasificación:
  
- Pueden capturar los límites de decisión no lineales.
- Usted puede entrenar grandes cantidades de datos y hacer predicciones con ellos, ya que son eficaces en el cálculo y el uso de memoria.
- La selección de características está integrada en los procesos de entrenamiento y clasificación.  
- Los árboles pueden adaptarse a los datos con ruido y a muchas características.  
- Son modelos no paramétricos, lo que significa que pueden controlar datos con distribuciones variadas. 

Sin embargo, los árboles de decisión simples pueden sobreajustar los datos y son menos generalizables que los conjuntos de árboles.

Para obtener más información, consulte [Bosques de decisión](https://go.microsoft.com/fwlink/?LinkId=403677).  

## <a name="how-to-configure"></a>Cómo se configura
  
1.  Agregue el módulo **Bosque de decisión de dos clases** a la canalización en Azure Machine Learning y abra el panel **Propiedades** del módulo. 

    Puede encontrar el módulo en **Machine Learning**. Expanda **Inicializar** y, a continuación, **Clasificación**.  
  
2.  Para obtener información sobre el **método de nuevo muestreo**, elija el método utilizado para crear los árboles individuales.  Puede elegir entre **Bagging** (Agregación) o **Replicate** (Replicación).  
  
    -   **Bagging** (agregación): la agregación también se denomina *agregación de arranque*. En este método, cada árbol crece en una muestra nueva, creada al muestrear de forma aleatoria el conjunto de datos original con el conjunto de reemplazo hasta que haya un conjunto de datos con el tamaño del original.  
  
         Los resultados de los modelos se combinan mediante *votación*, que es una forma de agregación. Cada árbol de un bosque de decisión de clasificación produce un histograma de etiquetas de frecuencia no normalizada. En el proceso de agregación se suman estos histogramas y se normalizan para obtener las "probabilidades" de cada etiqueta. De esta manera, los árboles con un nivel alto de confianza en la predicción tendrán un peso mayor en la decisión final del conjunto.  
  
         Para obtener más información, consulte la entrada de Wikipedia sobre la agregación de arranque.  
  
    -   **Replicate** (replicación): en la replicación, cada árbol se entrena exactamente con los mismos datos de entrada. La determinación de qué predicado de división se utiliza para cada nodo de árbol sigue siendo aleatoria y los árboles serán diversos.   
  
3.  Especifique cómo quiere que se entrene el modelo, estableciendo la opción **Create trainer mode** (Crear modo entrenador).  
  
    -   **Single Parameter** (Parámetro único): Si sabe cómo quiere configurar el modelo, puede proporcionar un conjunto específico de valores como argumentos.

    -   **Parameter Range** (Intervalo de parámetros): si no está seguro de los mejores parámetros, puede encontrar los óptimos mediante el módulo [Optimización de hiperparámetros de un modelo](tune-model-hyperparameters.md). Proporcione un rango de valores y el entrenador itera varias combinaciones de ellos para determinar la combinación de valores que genera el mejor resultado.
  
4.  En **Número de árboles de decisión**, escriba el número máximo de árboles de decisión que se pueden crear en el conjunto. Al crear más árboles de decisión, puede obtener una mejor cobertura, pero el tiempo de entrenamiento aumenta.  
  
    > [!NOTE]
    >  Este valor también controla el número de árboles que se muestran al visualizar el modelo entrenado. Si quiere ver o imprimir un único árbol, puede establecer el valor en 1. Sin embargo, solo se puede producir un único árbol (el árbol con el conjunto inicial de parámetros) y no se realizan iteraciones adicionales.
  
5.  En **Maximum depth of the decision trees** (Profundidad máxima de los árboles de decisión), escriba un número para limitar la profundidad máxima de cualquier árbol de decisión. Al aumentar la profundidad del árbol podría aumentar la precisión, a riesgo de que se produzca un sobreajuste y aumente el tiempo de entrenamiento.
  
  
7.  En **Minimum number of samples per leaf node** (Número mínimo de muestras por nodo hoja), indique el número mínimo de casos que son necesarios para crear cualquier nodo terminal (hoja) en un árbol.
  
     Al aumentar este valor, aumenta el umbral para crear reglas nuevas. Por ejemplo, con el valor predeterminado de 1, incluso un solo caso puede provocar que se cree una regla nueva. Si aumenta el valor a 5, los datos de entrenamiento tendrían que contener cinco casos como mínimo que cumplan las mismas condiciones.  
  
8.  Seleccione la opción **Permitir valores desconocidos para las características categóricas** para crear un grupo de valores desconocidos en los conjuntos de entrenamiento o validación. El modelo podría ser menos preciso con valores conocidos, pero puede proporcionar mejores predicciones para los valores nuevos (desconocidos). 

     Si desactiva esta opción, el modelo puede aceptar únicamente los valores que se encuentran en los datos de entrenamiento.
  
9. Adjunte un conjunto de datos etiquetado y entrene el modelo:

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