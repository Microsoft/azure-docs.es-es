---
title: ResNet
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo crear un modelo de clasificación de imágenes con el algoritmo ResNet.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 05/26/2020
ms.openlocfilehash: 5d8806b8c93f5a8cbceaa6efa16dfff978dda42e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905203"
---
# <a name="resnet"></a>ResNet

En este artículo se describe cómo usar el módulo **ResNet** en el diseñador de Azure Machine Learning para crear un modelo de clasificación de imágenes con el algoritmo ResNet.  

Este algoritmo de clasificación es un método de aprendizaje supervisado y requiere un conjunto de datos etiquetado. Para más información sobre cómo obtener un directorio de imagen con etiquetas, consulte [Conversión a directorio de imagen](convert-to-image-directory.md). Puede entrenar el modelo proporcionando un modelo y un directorio de imagen con etiquetas como entradas para [Entrenamiento del modelo de PyTorch](train-pytorch-model.md). Después, el modelo entrenado puede utilizarse para predecir valores para los nuevos ejemplos de entrada mediante [Puntuación del modelo de imagen](score-image-model.md).

### <a name="more-about-resnet"></a>Más información sobre ResNet

Consulte [este documento](https://pytorch.org/docs/stable/torchvision/models.html?highlight=resnext101_32x8d#torchvision.models.resnext101_32x8d) para obtener más información sobre ResNet.

## <a name="how-to-configure-resnet"></a>Configuración de ResNet

1.  Agregue el módulo **ResNet** a la canalización en el diseñador.  

2.  En **Nombre del modelo**, especifique el nombre de una determinada estructura ResNet y puede seleccionar entre ResNet compatibles: "resnet18", "resnet34", "resnet50", "resnet101", "resnet152", "resnet152", "resnext50\_32x4d", "resnext101\_32x8d", "wide_resnet50\_2", "wide_resnet101\_2".

3.  En **Pretrained** (Entrenado previamente), especifique si desea usar un modelo entrenado previamente en ImageNet. Si se selecciona esta opción, puede ajustar el modelo en función del modelo seleccionado previamente entrenado. Si se anula la selección, puede entrenar desde cero.

4.  Conecte la salida del módulo **DenseNet**, el módulo de entrenamiento y validación del conjunto de datos de imágenes, a [Entrenamiento del modelo de PyTorch](train-pytorch-model.md). 

5. Envíe la canalización.

## <a name="results"></a>Results

Una vez completada la ejecución de la canalización, para usar el modelo para la puntuación, conecte [Entrenamiento del modelo de PyTorch](train-pytorch-model.md) a [Puntuación del modelo de imagen](score-image-model.md), para predecir valores para los nuevos ejemplos de entrada.

## <a name="technical-notes"></a>Notas técnicas  

###  <a name="module-parameters"></a>Parámetros del módulo  

| Nombre       | Intervalo | Tipo    | Valor predeterminado           | Descripción                              |
| ---------- | ----- | ------- | ----------------- | ---------------------------------------- |
| Nombre del modelo | Any   | Mode    | resnext101\_32x8d | Nombre de una determinada estructura ResNet       |
| Entrenado previamente | Any   | Boolean | True              | Si desea usar un modelo entrenado previamente en ImageNet |
|            |       |         |                   |                                          |

###  <a name="output"></a>Output  

| Nombre            | Tipo                    | Descripción                              |
| --------------- | ----------------------- | ---------------------------------------- |
| Modelo no entrenado | UntrainedModelDirectory | Un modelo ResNet no entrenado que se puede conectar a Entrenamiento del modelo de PyTorch. |

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 