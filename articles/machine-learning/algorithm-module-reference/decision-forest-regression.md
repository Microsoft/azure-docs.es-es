---
title: 'Regresión de bosque de decisión: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo Regresión de bosque de decisión en Azure Machine Learning para crear un modelo de regresión basado en un conjunto de árboles de decisión.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 04/22/2020
ms.openlocfilehash: 7f9677bfd793e7ff21ff2c6c7e6760b630dc074b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898541"
---
# <a name="decision-forest-regression-module"></a>Módulo Decision Forest Regression (Regresión de bosque de decisión)

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Utilice este módulo para crear un modelo de regresión basado en un conjunto de árboles de decisión.

Una vez que haya configurado el modelo, debe entrenarlo mediante un conjunto de datos etiquetado y el módulo [Entrenar modelo](./train-model.md). A continuación, el modelo entrenado podrá usarse para realizar predicciones. 

## <a name="how-it-works"></a>Funcionamiento

Los árboles de decisión son modelos no paramétricos que llevan a cabo una secuencia de pruebas simples para cada instancia, atravesando una estructura de datos de árbol binario hasta alcanzar un nodo hoja (decisión).

Los árboles de decisión tienen las siguientes ventajas:

- Son eficientes tanto en el cálculo como en la utilización de la memoria durante el entrenamiento y la predicción.

- Pueden representar límites de decisión no lineales.

- Realizan una clasificación y selección de características integradas y son resistentes en presencia de características ruidosas.

Este modelo de regresión consta de un conjunto de árboles de decisión. Cada árbol de un bosque de decisión de regresión da como resultado una predicción en forma de distribución gaussiana. Se realiza una agregación sobre el conjunto de árboles para buscar la distribución gaussiana más cercana a la distribución combinada de todos los árboles del modelo.

Para obtener más información sobre el marco teórico de este algoritmo y su implementación, consulte este artículo: [Bosques de decisión: Marco unificado para la clasificación, la regresión, la estimación de densidades, el aprendizaje múltiple y el aprendizaje supervisado parcialmente](https://www.microsoft.com/en-us/research/publication/decision-forests-a-unified-framework-for-classification-regression-density-estimation-manifold-learning-and-semi-supervised-learning/?from=http%3A%2F%2Fresearch.microsoft.com%2Fapps%2Fpubs%2Fdefault.aspx%3Fid%3D158806#)

## <a name="how-to-configure-decision-forest-regression-model"></a>Cómo configurar el modelo de regresión de bosque de decisión

1. Agregue el módulo **Regresión de bosque de decisión** a la canalización. Puede encontrar el módulo en el diseñador, en **Machine Learning**, **Initialize Model** (Inicializar modelo) y **Regresión**.

2. Abra las propiedades del módulo y, para **Resampling method** (Método de nuevo muestreo), elija el método utilizado para crear los árboles individuales.  Puede elegir entre **Bagging** (Agregación) o **Replicate** (Replicación).

    - **Bagging** (agregación): la agregación también se denomina *agregación de arranque*. Cada árbol de un bosque de decisión de regresión da como resultado una predicción en forma de distribución gaussiana. La agregación consiste en encontrar una distribución gaussiana cuyos dos primeros momentos coincidan con los momentos de la mezcla de distribuciones gaussianas determinada combinando todas las distribuciones devueltas por los árboles individuales.

         Para obtener más información, consulte la entrada de Wikipedia sobre la [agregación de arranque](https://wikipedia.org/wiki/Bootstrap_aggregating).

    - **Replicate** (replicación): en la replicación, cada árbol se entrena exactamente con los mismos datos de entrada. La determinación de qué predicado de división se utiliza para cada nodo de árbol sigue siendo aleatoria y los árboles serán diversos.

         Para obtener más información sobre el proceso de entrenamiento con la opción **Replicate** (Replicación), consulte [Bosques de decisión para Computer Vision y análisis de imágenes médicas. Criminisi y J. Shotton. Springer 2013. ](https://research.microsoft.com/projects/decisionforests/).

3. Especifique cómo quiere que se entrene el modelo, estableciendo la opción **Create trainer mode** (Crear modo entrenador).

    - **Single Parameter** (Parámetro único)

      Si sabe cómo quiere configurar el modelo, puede proporcionar un conjunto específico de valores como argumentos. Es posible que haya obtenido estos valores mediante experimentación o que los haya recibido como guía.

    - **Parameter Range** (Intervalo de parámetros): seleccione esta opción si no está seguro de los mejores parámetros y quiere ejecutar un barrido de parámetros. Seleccione un rango de valores que iterar y el módulo [Optimización de hiperparámetros de un modelo](tune-model-hyperparameters.md) itera en todas las combinaciones posibles de los valores proporcionados para determinar los hiperparámetros que generan los resultados óptimos. 



4. Para **Number of decision trees** (Número de árboles de decisión), indique el número total de árboles de decisión que se creará en el conjunto. Si crea más árboles de decisión, puede obtener una cobertura potencialmente mejor, pero aumentará el tiempo de entrenamiento.

    > [!TIP]
    > Este valor también controla el número de árboles que se muestran al visualizar el modelo entrenado. Si desea ver o imprimir un único árbol, puede establecer el valor en 1; sin embargo, esto significa que solo se producirá un único árbol (el árbol con el conjunto inicial de parámetros) y que no se realizarán más iteraciones.

5. En **Maximum depth of the decision trees** (Profundidad máxima de los árboles de decisión), escriba un número para limitar la profundidad máxima de cualquier árbol de decisión. Al aumentar la profundidad del árbol podría aumentar la precisión, a riesgo de que se produzca un sobreajuste y aumente el tiempo de entrenamiento.

6. En **Number of random splits per node** (Número de divisiones aleatorias por nodo), escriba el número de divisiones que se usarán al crear cada nodo del árbol. Una *división* significa que las características de cada nivel del árbol (nodo) se dividen al azar.

7. En **Minimum number of samples per leaf node** (Número mínimo de muestras por nodo hoja), indique el número mínimo de casos que son necesarios para crear cualquier nodo terminal (hoja) en un árbol.

     Al aumentar este valor, aumenta el umbral para crear reglas nuevas. Por ejemplo, con el valor predeterminado de 1, incluso un solo caso puede provocar que se cree una regla nueva. Si aumenta el valor a 5, los datos de entrenamiento tendrían que contener cinco casos como mínimo que cumplan las mismas condiciones.


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

   

10. Envíe la canalización.

### <a name="results"></a>Results

Una vez completado el entrenamiento:

+ Para guardar una instantánea del modelo entrenado, seleccione el módulo de entrenamiento y cambie a la pestaña **Outputs** (Salidas) del panel derecho. Haga clic en el icono **Register model** (Registrar modelo).  El modelo guardado se encuentra como un módulo más en el árbol de módulos. 

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 