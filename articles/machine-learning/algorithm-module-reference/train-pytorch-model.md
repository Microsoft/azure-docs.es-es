---
title: Entrenamiento del modelo de PyTorch
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo entrenar el modelo de PyTorch desde cero o ajustarlo.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 05/26/2020
ms.openlocfilehash: 0ba603dad7d48be725f308f3a3296676c5f4f108
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90883240"
---
# <a name="train-pytorch-model"></a>Entrenamiento del modelo de PyTorch

En este artículo se describe cómo usar el módulo **Entrenamiento del modelo de PyTorch** en el diseñador de Azure Machine Learning para entrenar modelos de PyTorch como DenseNet. El entrenamiento tiene lugar después de definir un modelo y establecer sus parámetros y requiere datos etiquetados. 

## <a name="how-to-use-train-pytorch-model"></a>Uso de Entrenamiento del modelo de PyTorch 

1. Agregue los módulos [DenseNet](densenet.md) o [ResNet](resnet.md) al borrador de la canalización en el diseñador.

2. Agregue el módulo **Entrenamiento del modelo de PyTorch** a la canalización. Puede encontrar este módulo en la categoría **Entrenamiento del modelo**. Expanda **Entrenar** y luego arrastre el módulo **Entrenamiento del modelo de PyTorch** a la canalización.

   > [!NOTE]
   > El módulo **Entrenamiento del modelo de PyTorch** se ejecuta mejor en un proceso de tipo **GPU** para conjuntos de datos grandes; de lo contrario, se producirá un error en la canalización. Puede seleccionar el proceso para un módulo específico en el panel derecho del módulo estableciendo **Use other compute target** (Utilizar otro destino de proceso).

3.  En la entrada izquierda, adjunte un modo no entrenado. Adjunte el conjunto de datos de entrenamiento y el conjunto de datos de validación a la entrada de la parte central y derecha de **Entrenamiento del modelo de PyTorch**.

    En el caso de un modelo no entrenado, debe ser un modelo de Pytorch como DenseNet; de lo contrario, se producirá un error "InvalidModelDirectoryError".

    En el caso de un conjunto de datos, el conjunto de datos de entrenamiento debe ser un directorio de imagen con etiquetas. Para más información sobre cómo obtener un directorio de imagen con etiquetas, consulte **Conversión a directorio de imagen**. Si no está etiquetado, se producirá una un error "NotLabeledDatasetError".

    El conjunto de datos de entrenamiento y el conjunto de datos de validación deben tener las mismas categorías de etiqueta; de lo contrario, se producirá un error InvalidDatasetError.

4.  En **Epochs** (Épocas), especifique el número de épocas que le gustaría entrenar. Se iterará todo el conjunto de valores cada época, de forma predeterminada 5.

5.  Para **Tamaño del lote**, especifique el número de instancias que se van a entrenar en un lote, de forma predeterminada 16.

6.  En **Velocidad de aprendizaje**, especifique un valor para la *velocidad de aprendizaje*. El valor de velocidad de aprendizaje controla el tamaño del paso que se usa en el optimizador, como sgd, cada vez que se prueba y se corrige el modelo.

    Al disminuir la velocidad, se prueba el modelo más a menudo, con el riesgo de que pueda quedarse atascado en un nivel local. Al aumentar el paso, puede convergir con mayor rapidez, con el riesgo de superar los mínimos verdaderos. De forma predeterminada 0,001.

7.  En **Random seed** (Inicialización aleatoria), escriba opcionalmente un valor de entero que se usará como inicialización. Se recomienda usar una inicialización si desea garantizar la reproducibilidad del experimento a través de las ejecuciones.

8.  Por **Patience** (Paciencia), especifique para cuántas épocas quiere detener el entrenamiento con antelación si la pérdida de validación no se reduce consecutivamente. De forma predeterminada 3.

9.  Envíe la canalización. Si el conjunto de datos tiene un tamaño mayor, tardará un tiempo.

## <a name="results"></a>Results

Una vez completada la ejecución de la canalización, para usar el modelo para la puntuación, conecte [Entrenamiento del modelo de PyTorch](train-pytorch-model.md) a [Puntuación del modelo de imagen](score-image-model.md), para predecir valores para los nuevos ejemplos de entrada.

## <a name="technical-notes"></a>Notas técnicas
###  <a name="expected-inputs"></a>Entradas esperadas  

| Nombre               | Tipo                    | Descripción                              |
| ------------------ | ----------------------- | ---------------------------------------- |
| Modelo no entrenado    | UntrainedModelDirectory | Modelo no entrenado, requerir PyTorch         |
| Conjunto de datos de entrenamiento   | ImageDirectory          | Conjunto de datos de entrenamiento                         |
| Conjunto de datos de validación | ImageDirectory          | Conjunto de datos de validación para la evaluación cada época |

###  <a name="module-parameters"></a>Parámetros del módulo  

| Nombre          | Intervalo            | Tipo    | Valor predeterminado | Descripción                              |
| ------------- | ---------------- | ------- | ------- | ---------------------------------------- |
| Épocas        | >0               | Entero | 5       | Seleccionar la columna que contiene la columna de etiqueta o resultado |
| Tamaño de lote    | >0               | Entero | 16      | Número de instancias que se van a entrenar en un lote   |
| Velocidad de aprendizaje | >=double.Epsilon | Float   | 0,001   | Velocidad de aprendizaje inicial para el optimizador de descenso de gradiente estocástico. |
| Inicialización aleatoria   | Any              | Entero | 1       | La inicialización para el generador de números aleatorios usado por el modelo. |
| Paciencia      | >0               | Entero | 3       | Número de épocas para la detención con antelación del entrenamiento   |

###  <a name="outputs"></a>Salidas  

| Nombre          | Tipo           | Descripción   |
| ------------- | -------------- | ------------- |
| Modelo entrenado | ModelDirectory | Modelo entrenado |

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 



