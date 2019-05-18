---
title: 'Clasificación: Predecir el abandono, apetencia y venta vertical '
titleSuffix: Azure Machine Learning service
description: Este experimento de ejemplo de la interfaz visual muestra predicción clasificador binario de renovación, una tarea común para la administración de relaciones con clientes (CRM).
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: article
author: xiaoharper
ms.author: zhanxia
ms.reviewer: sgilley
ms.date: 05/10/2019
ms.openlocfilehash: 42724f5fcb3101015cef0d218a3d548f349646be
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65785817"
---
# <a name="sample-5---classification-predict-churn-appetency-and-up-selling"></a>Ejemplo 5: clasificación: Predecir el abandono, apetencia y venta vertical 

Obtenga información sobre cómo crear un experimento de aprendizaje automático complejos sin necesidad de escribir una sola línea de código mediante la interfaz visual.

Este experimento entrena tres, **árbol de decisión ampliado de dos clases** clasificadores para predecir las tareas comunes para los sistemas de administración (CRM) de relación de cliente: renovación, apetencia y venta vertical. Los valores de datos y las etiquetas se dividen entre varios orígenes de datos y codificadas para anonimizar la información del cliente, sin embargo, todavía podemos usar la interfaz visual para combinar conjuntos de datos y entrenar un modelo con los valores codificados.

Dado que estamos intentando responder a la pregunta "Cuál?" Esto se denomina un problema de clasificación. Sin embargo, puede aplicar los mismos pasos en este experimento para abordar a cualquier tipo de problema del aprendizaje automático, ya sea una regresión, clasificación, agrupación en clústeres y así sucesivamente.

Este es el gráfico completado para este experimento:

![Gráfico de experimento](./media/ui-sample-classification-predict-churn/experiment-graph.png)

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [aml-ui-prereq](../../../includes/aml-ui-prereq.md)]

4. Seleccione el **abierto** botón para el experimento de ejemplo 5.

    ![Abra el experimento](media/ui-sample-classification-predict-churn/open-sample5.png)

## <a name="data"></a>Datos

Los datos que se utilizan en este experimento están de KDD Cup 2009. El conjunto de datos tiene 50 000 filas y columnas de característica de 230. La tarea consiste en predecir el abandono, apetencia y venta vertical para los clientes que usan estas características. Para obtener más información sobre los datos y la tarea, vea el [sitio Web KDD](https://www.kdd.org/kdd-cup/view/kdd-cup-2009).

## <a name="experiment-summary"></a>Resumen del experimento

Este experimento de ejemplo de la interfaz visual muestra la predicción de clasificador binario de renovación, apetencia y venta de arriba, una tarea común para la administración de relaciones con clientes (CRM).

En primer lugar, se realizan un procesamiento de datos simple.

- El conjunto de datos sin procesar contiene una gran cantidad de valores que faltan. Usamos el **Clean Missing Data** los valores de módulo para reemplazar el campo que falta con 0.

    ![Limpiar el conjunto de datos](./media/ui-sample-classification-predict-churn/cleaned-dataset.png)

- Las características y la correspondiente renovación, apetencia, y las ventas hasta las etiquetas están en diferentes conjuntos de datos. Usamos el **agregar columnas** módulo para anexar las columnas de etiqueta a las columnas de característica. La primera columna, **Col1**, es la columna de etiqueta. El resto de las columnas, **Var1**, **Var2**, etc., son las columnas de característica.

    ![Agregar el conjunto de datos de columna](./media/ui-sample-classification-predict-churn/added-column1.png)

- Usamos el **dividir datos** módulo para dividir el conjunto de datos "Train" y conjuntos de pruebas.

    A continuación, usamos el clasificador binario del árbol de decisión impulsado con los parámetros predeterminados para crear los modelos de predicción. Creamos un modelo por tarea, es decir, un modelo para predecir ventas cruzadas arriba, apetencia y renovación.

## <a name="results"></a>Resultados

Visualizar el resultado de la **Evaluate Model** módulo para ver el rendimiento del modelo en el conjunto de pruebas. Para la tarea incrementando las ventas, la curva ROC se muestra que el modelo no mejor que un modelo aleatorio. El área bajo la curva (AUC) es 0.857. En el umbral de 0,5, la precisión es 0,7, la recuperación es 0.463 y la puntuación F1 es 0.545.

![Evaluar los resultados](./media/ui-sample-classification-predict-churn/evaluate-result.png)

 Puede mover el **umbral** control deslizante y vea Cambiar las métricas para la tarea de clasificación binaria.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [aml-ui-cleanup](../../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

Explore los ejemplos disponibles para la interfaz visual:

- [Ejemplo 1: regresión: Predecir el precio de un automóvil](ui-sample-regression-predict-automobile-price-basic.md)
- [Ejemplo 2: regresión: Compare de algoritmos para la predicción de precios de automóviles](ui-sample-regression-predict-automobile-price-compare-algorithms.md)
- [Ejemplo 3: clasificación: Predecir el riesgo de crédito](ui-sample-classification-predict-credit-risk-basic.md)
- [Ejemplo 4: clasificación: Predecir el riesgo de crédito (costo confidencial)](ui-sample-classification-predict-credit-risk-cost-sensitive.md)
