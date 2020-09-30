---
title: DenseNet
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo crear un modelo de clasificación de imágenes con el algoritmo DenseNet.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 05/26/2020
ms.openlocfilehash: 69c18c24ae9a8eb4c1fd54c1f8530e126a40b004
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898528"
---
# <a name="densenet"></a>DenseNet

En este artículo se describe cómo usar el módulo **DenseNet** en el diseñador de Azure Machine Learning para crear un modelo de clasificación de imágenes con el algoritmo DenseNet.  

Este algoritmo de clasificación es un método de aprendizaje supervisado y requiere un conjunto de datos etiquetado. Para más instrucciones sobre cómo obtener un directorio de imagen con etiquetas, consulte [Conversión a directorio de imagen](convert-to-image-directory.md). Puede entrenar el modelo proporcionando el modelo y el directorio de imagen con etiquetas como entradas para [Entrenamiento del modelo de PyTorch](train-pytorch-model.md). Después, el modelo entrenado puede utilizarse para predecir valores para los nuevos ejemplos de entrada mediante [Puntuación del modelo de imagen](score-image-model.md).

### <a name="more-about-densenet"></a>Más información sobre DenseNet

Consulte [Redes circunvolucionales conectadas densamente](https://arxiv.org/abs/1608.06993) para obtener más detalles.

## <a name="how-to-configure-densenet"></a>Configuración de DenseNet

1.  Agregue el módulo **DenseNet** a la canalización en el diseñador.  

2.  Por **Nombre del modelo**, especifique el nombre de una determinada estructura DenseNet y puede seleccionar entre las instancias de DenseNet compatibles: "densenet121", "densenet161", "densenet169", "densenet201".

3.  En **Pretrained** (Entrenado previamente), especifique si desea usar un modelo entrenado previamente en ImageNet. Si se selecciona esta opción, puede ajustar el modelo en función del modelo seleccionado previamente entrenado. Si se anula la selección, puede entrenar desde cero.

4.  Por **Memory efficient** (Eficiencia de memoria), especifique si quiere usar puntos de comprobación, que es mucho mejor para la eficiencia de la memoria, pero más lento. Consulte https://arxiv.org/pdf/1707.06990.pdf para obtener más información.

5.  Conecte la salida del módulo **DenseNet**, el módulo de entrenamiento y validación del conjunto de datos de imágenes, a [Entrenamiento del modelo de PyTorch](train-pytorch-model.md). 

6. Envíe la canalización.


## <a name="results"></a>Results

Una vez completada la ejecución de la canalización, para usar el modelo para la puntuación, conecte [Entrenamiento del modelo de PyTorch](train-pytorch-model.md) a [Puntuación del modelo de imagen](score-image-model.md), para predecir valores para los nuevos ejemplos de entrada.

## <a name="technical-notes"></a>Notas técnicas  

###  <a name="module-parameters"></a>Parámetros del módulo  

| Nombre             | Intervalo | Tipo    | Valor predeterminado     | Descripción                              |
| ---------------- | ----- | ------- | ----------- | ---------------------------------------- |
| Nombre del modelo       | Any   | Mode    | densenet201 | Nombre de una determinada estructura DenseNet     |
| Entrenado previamente       | Any   | Boolean | True        | Si desea usar un modelo entrenado previamente en ImageNet |
| Eficiencia de memoria | Any   | Boolean | False       | Si quiere usar puntos de comprobación, mucho mejor para la eficiencia de la memoria, pero más lento |

###  <a name="output"></a>Output  

| Nombre            | Tipo                    | Descripción                              |
| --------------- | ----------------------- | ---------------------------------------- |
| Modelo no entrenado | UntrainedModelDirectory | Un modelo DenseNet no entrenado que se puede conectar a Entrenamiento del modelo de PyTorch. |

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
