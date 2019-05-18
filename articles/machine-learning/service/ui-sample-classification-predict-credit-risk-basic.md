---
title: 'Clasificación: Predicción del riesgo de crédito'
titleSuffix: Azure Machine Learning service
description: Obtenga información sobre cómo crear un clasificador de sin necesidad de escribir una sola línea de código mediante la interfaz visual del aprendizaje automático.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: article
author: xiaoharper
ms.author: zhanxia
ms.reviewer: sgilley
ms.date: 05/10/2019
ms.openlocfilehash: f37c945758cfbd03889d79acf764e7f67022267a
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65789395"
---
# <a name="sample-3---classification-predict-credit-risk"></a>Ejemplo 3: clasificación: Predicción del riesgo de crédito

Obtenga información sobre cómo crear un clasificador de sin necesidad de escribir una sola línea de código mediante la interfaz visual del aprendizaje automático. En este ejemplo se entrena un **árbol de decisión ampliado de dos clases** para predecir el crédito de riesgo (alta o baja) según la información de aplicación de crédito como historial crediticio, edad y el número de tarjetas de crédito.

Dado que estamos intentando responder a la pregunta "Cuál?" Esto se denomina un problema de clasificación. Sin embargo, puede aplicar el mismo proceso fundamental para abordar a cualquier tipo de problema del aprendizaje automático, ya sea una regresión, clasificación, agrupación en clústeres y así sucesivamente.

Este es el gráfico completado para este experimento:

![Gráfico del experimento](media/ui-sample-classification-predict-credit-risk-basic/overall-graph.png)

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [aml-ui-prereq](../../../includes/aml-ui-prereq.md)]

4. Seleccione el **abierto** botón para el experimento de ejemplo 3:

    ![Abra el experimento](media/ui-sample-classification-predict-credit-risk-basic/open-sample3.png)

## <a name="related-sample"></a>Ejemplo relacionado

[Ejemplo 4: clasificación: (Costo mayúsculas de minúsculas) de predicción de riesgo de crédito](ui-sample-classification-predict-credit-risk-cost-sensitive.md) proporciona un experimento avanzado que resuelve el mismo problema que este experimento. Muestra cómo realizar _costo confidencial_ la clasificación mediante una **Execute Python Script** module y compare el rendimiento de dos algoritmos de clasificación binaria. Hacer referencia a él si desea obtener más información sobre cómo crear experimentos de clasificación.

## <a name="data"></a>Datos

Usamos el conjunto de datos de la tarjeta de crédito alemana desde el repositorio de UC Irvine.
El conjunto de datos contiene 1000 muestras con 20 características y 1 etiqueta. Cada ejemplo representa a una persona. Las características incluyen características de categorías y numéricas. Consulte la [sitio Web de UCI](https://archive.ics.uci.edu/ml/datasets/Statlog+%28German+Credit+Data%29) del significado de las características de categorías. La última columna es la etiqueta, que denota el riesgo de crédito y tiene solo dos valores posibles: riesgo de crédito alto = 2 y el riesgo de crédito bajo = 1.

## <a name="experiment-summary"></a>Resumen del experimento

Seguimos estos pasos para crear el experimento:

1. Arrastre el módulo de conjunto de datos de datos de UCI de tarjeta de crédito alemán en el lienzo del experimento.
1. Agregar un **editar metadatos** módulo, por lo que podemos agregar nombres descriptivos para cada columna.
1. Agregar un **dividir datos** módulo para crear los conjuntos de entrenamiento y prueba. Establezca la fracción de filas en el primer conjunto de datos de salida en 0,7. Esta configuración especifica que un 70% de los datos será de salida para el puerto de la izquierda del módulo y el resto en el puerto correcto. Usaremos el conjunto de datos izquierdo para entrenamiento y el derecho para las pruebas.
1. Agregar un **Two-Class Boosted Decision Tree** módulo para inicializar un clasificador de árbol de decisión ampliado.
1. Agregar un **Train Model** módulo. Conectar el clasificador del paso anterior para el puerto de entrada izquierdo de la **Train Model**. Agregar el conjunto de entrenamiento (puerto de salida de la izquierda el **dividir datos**) al puerto de entrada derecho de la **Train Model**. El **entrenar modelo** a entrenar el clasificador.
1. Agregar un **Score Model** módulo y conecte el **Train Model** módulo a él. A continuación, agregue el conjunto de pruebas (el puerto correcto de la **dividir datos**) a la **Score Model**. El **Score Model** hará que las predicciones. Puede seleccionar su puerto de salida para ver las predicciones y probabilidades de la clase positiva.
1. Agregar un **Evaluate Model** módulo y conecte el conjunto de datos puntuado a su puerto de entrada izquierdo. Para ver los resultados de evaluación, seleccione el puerto de salida de la **Evaluate Model** módulo y seleccione **visualizar**.

Este es el gráfico de experimento completo:

![Gráfico del experimento](media/ui-sample-classification-predict-credit-risk-basic/overall-graph.png)

## <a name="results"></a>Resultados

![Evaluar los resultados](media/ui-sample-classification-predict-credit-risk-basic/evaluate-result.png)

En los resultados de evaluación, puede ver que el AUC del modelo es 0.776. En el umbral de 0,5, la precisión es 0.621, la recuperación es 0,456 y la puntuación F1 es 0.526.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [aml-ui-cleanup](../../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

Explore los ejemplos disponibles para la interfaz visual:

- [Ejemplo 1: regresión: Predecir el precio de un automóvil](ui-sample-regression-predict-automobile-price-basic.md)
- [Ejemplo 2: regresión: Compare de algoritmos para la predicción de precios de automóviles](ui-sample-regression-predict-automobile-price-compare-algorithms.md)
- [Ejemplo 4: clasificación: Predecir el riesgo de crédito (costo confidencial)](ui-sample-classification-predict-credit-risk-cost-sensitive.md)
- [Ejemplo 5: clasificación: Predecir el abandono](ui-sample-classification-predict-churn.md)