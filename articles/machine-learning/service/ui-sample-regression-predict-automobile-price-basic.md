---
title: 'Regresión: Predicción del precio'
titleSuffix: Azure Machine Learning service
description: Obtenga información sobre cómo crear un modelo de machine learning para predecir el precio de un automóvil sin necesidad de escribir una sola línea de código.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: article
author: xiaoharper
ms.author: zhanxia
ms.reviewer: sgilley
ms.date: 05/10/2019
ms.openlocfilehash: 9dfa4b62f5cb79a5716f6f29651e85d0f8a3a409
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787855"
---
# <a name="sample-1---regression-predict-price"></a>Ejemplo 1: regresión: Predicción del precio

Obtenga información sobre cómo crear un modelo de regresión de aprendizaje sin necesidad de escribir una sola línea de código mediante la interfaz visual automático.

Este experimento entrena un **regresor de bosque de decisión** para predecir un automóvil del precio en función de las características técnicas, como la marca, modelo, caballos de fuerza y tamaño. Dado que estamos intentando responder a la pregunta "¿cuánto?" Esto se denomina un problema de regresión. Sin embargo, puede aplicar los mismos pasos fundamentales en este experimento para abordar a cualquier tipo de problema del aprendizaje automático, ya sea una regresión, clasificación, agrupación en clústeres y así sucesivamente.

Los pasos fundamentales de un modelo de aprendizaje automático de aprendizaje son:

1. Obtener los datos
1. Procesar previamente los datos
1. Entrenamiento del modelo
1. Evaluación del modelo

Este es el gráfico final, completado del experimento que vamos a trabajar en. Le presentaremos el razonamiento de todos los módulos para que pueda tomar decisiones similares en sus propios.

![Gráfico del experimento](media/ui-sample-regression-predict-automobile-price-basic/overall-graph.png)

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [aml-ui-prereq](../../../includes/aml-ui-prereq.md)]

4. Seleccione el **abierto** botón para el experimento de ejemplo 1:

    ![Abra el experimento](media/ui-sample-regression-predict-automobile-price-basic/open-sample1.png)

## <a name="get-the-data"></a>Obtener los datos

En este experimento, usamos el **datos de precios de automóviles (Raw)** conjunto de datos, que está en el repositorio de aprendizaje automático de UCI. El conjunto de datos contiene 26 columnas que contienen información acerca de automóviles, incluida la marca, modelo, precio, características de vehículo (como el número de cilindros), MPG y una puntuación de riesgo de seguros. El objetivo de este experimento es predecir el precio del automóvil.

## <a name="pre-process-the-data"></a>Procesar previamente los datos

Las tareas de preparación de datos principal incluyen la limpieza de datos, integración, transformación, reducción y la discretización o cuantificación. En la interfaz visual, puede encontrar módulos para realizar estas operaciones y otros datos de procesamiento previo de las tareas en el **transformación de datos** grupo en el panel izquierdo.

Usamos el **Select Columns in Dataset** módulo para excluir normalized-losses que tienen muchos valores que faltan. A continuación, utilizamos **Clean Missing Data** para quitar las filas que tienen valores que faltan. Esto ayuda a crear un conjunto de datos de entrenamiento limpio.

![Procesamiento previo de los datos](./media/ui-sample-regression-predict-automobile-price-basic/data-processing.png)

## <a name="train-the-model"></a>Entrenamiento del modelo

Problemas de aprendizaje automático varían. Tareas de aprendizaje automático comunes incluyen la clasificación, agrupación en clústeres, regresión y los sistemas de recomendación, cada uno de los cuales puede requerir un algoritmo diferente. La elección del algoritmo a menudo depende de los requisitos del caso de uso. Después de elegir un algoritmo, deberá ajustar sus parámetros para entrenar un modelo más preciso. A continuación, deberá evaluar todos los modelos basados en métricas, como la precisión, inteligibilidad y eficacia.

Dado que el objetivo de este experimento es predecir precios de automóviles y porque la columna de etiqueta (precio) contiene números reales, un modelo de regresión es una buena elección. Teniendo en cuenta que el número de características es relativamente pequeño (inferior a 100) y estas características no están dispersas, el límite de decisión es probable que sea no lineal. Por lo que usamos **regresión de bosque de decisión** para este experimento.

Usamos el **dividir datos** módulo dividir aleatoriamente los datos de entrada para que el conjunto de datos de entrenamiento contiene un 70% de los datos originales y el conjunto de datos de prueba contiene un 30% de los datos originales.

## <a name="test-evaluate-and-compare"></a>Probar, evaluar y comparar

 Se divide el conjunto de datos y utilizan diferentes conjuntos de datos para entrenar y probar el modelo para realizar la evaluación del modelo más objetivas.

Después de que el modelo se entrena, usamos el **Score Model** y **Evaluate Model** módulos para generar los resultados de predicción y evaluar los modelos.

**Puntuar modelo** genera predicciones del conjunto de datos de prueba mediante el modelo entrenado. Para comprobar el resultado, seleccione el puerto de salida **Score Model** y, a continuación, seleccione **visualizar**.

![Resultado de puntuación](./media/ui-sample-regression-predict-automobile-price-basic/score-result.png)

A continuación, pasamos las puntuaciones para el **Evaluate Model** módulo para generar métricas de evaluación. Para comprobar el resultado, seleccione el puerto de salida de la **Evaluate Model** y, a continuación, seleccione **visualizar**.

![Evaluar los resultados](./media/ui-sample-regression-predict-automobile-price-basic/evaluate-result.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [aml-ui-cleanup](../../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

Explore los ejemplos disponibles para la interfaz visual:

- [Ejemplo 2: regresión: Compare de algoritmos para la predicción de precios de automóviles](ui-sample-regression-predict-automobile-price-compare-algorithms.md)
- [Ejemplo 3: clasificación: Predecir el riesgo de crédito](ui-sample-classification-predict-credit-risk-basic.md)
- [Ejemplo 4: clasificación: Predecir el riesgo de crédito (costo confidencial)](ui-sample-classification-predict-credit-risk-cost-sensitive.md)
- [Ejemplo 5: clasificación: Predecir el abandono](ui-sample-classification-predict-churn.md)