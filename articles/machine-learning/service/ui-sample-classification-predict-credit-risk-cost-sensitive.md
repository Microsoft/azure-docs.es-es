---
title: 'Clasificación: Predecir el riesgo de crédito (costo confidencial)'
titleSuffix: Azure Machine Learning service
description: Este artículo muestra cómo crear un complejo experimento de machine learning mediante la interfaz visual. Obtendrá información sobre cómo implementar secuencias de comandos de Python personalizados y comparar varios modelos para elegir la mejor opción.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: article
author: xiaoharper
ms.author: zhanxia
ms.reviewer: sgilley
ms.date: 05/10/2019
ms.openlocfilehash: d714756c19b94eafc40cc0dbeffbc07704e8f94e
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787810"
---
# <a name="sample-4---classification-predict-credit-risk-cost-sensitive"></a>Ejemplo 4: clasificación: Predecir el riesgo de crédito (costo confidencial)

Este artículo muestra cómo crear un complejo experimento de machine learning mediante la interfaz visual. Obtendrá información sobre cómo implementar la lógica personalizada mediante scripts de Python y comparar varios modelos para elegir la mejor opción.

En este ejemplo se entrena un clasificador para predecir el riesgo de crédito con la información de la aplicación de crédito como historial crediticio, edad y el número de tarjetas de crédito. Sin embargo, puede aplicar los conceptos de este artículo para abordar su propios problemas de aprendizaje automático.

Si está empezando con machine learning, puede tardar un vistazo a la [ejemplo básico de clasificadores](ui-sample-classification-predict-credit-risk-basic.md) primero.

Este es el gráfico completado para este experimento:

[![Gráfico del experimento](media/ui-sample-classification-predict-credit-risk-cost-sensitive/graph.png)](media/ui-sample-classification-predict-credit-risk-cost-sensitive/graph.png#lightbox)

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [aml-ui-prereq](../../../includes/aml-ui-prereq.md)]

4. Seleccione el **abierto** botón para el experimento de ejemplo 4:

    ![Abra el experimento](media/ui-sample-classification-predict-credit-risk-cost-sensitive/open-sample4.png)

## <a name="data"></a>Datos

Usamos el conjunto de datos de la tarjeta de crédito alemana desde el repositorio de UC Irvine. Este conjunto de datos contiene 1000 muestras con 20 características y 1 etiqueta. Cada ejemplo representa a una persona. Las 20 características incluyen características de categorías y numéricas. Consulte la [sitio Web de UCI](https://archive.ics.uci.edu/ml/datasets/Statlog+%28German+Credit+Data%29) para obtener más información sobre el conjunto de datos. La última columna es la etiqueta, que denota el riesgo de crédito y tiene solo dos valores posibles: riesgo de crédito alto = 2 y el riesgo de crédito bajo = 1.

## <a name="experiment-summary"></a>Resumen del experimento

En este experimento, comparamos dos enfoques diferentes para generar modelos para solucionar este problema:

- En el aprendizaje con el conjunto de datos original.
- Cursos de formación con un conjunto de datos replicado.

Con ambos enfoques, Evaluamos los modelos utilizando el conjunto de datos de prueba con la replicación para asegurarse de que los resultados están alineados con la función de costo. Probamos dos clasificadores con ambos enfoques: **Máquina de vectores de soporte de dos clases** y **árbol de decisión ampliado de dos clases**.

El costo de clasificar erróneamente un ejemplo de bajo riesgo tan alto es 1 y el costo de clasificar erróneamente un ejemplo de alto riesgo como bajo es 5. Usamos un **Execute Python Script** módulo para tener en cuenta esta clasificación incorrecta de costos.

Este es el gráfico del experimento:

[![Gráfico del experimento](media/ui-sample-classification-predict-credit-risk-cost-sensitive/graph.png)](media/ui-sample-classification-predict-credit-risk-cost-sensitive/graph.png#lightbox)

## <a name="data-processing"></a>Procesamiento de datos

Comenzamos utilizando el **Editor de metadatos** módulo para agregar nombres de columna para reemplazar los nombres de columna predeterminados por nombres más descriptivos, obtenido de la descripción del conjunto de datos en el sitio UCI. Proporcionamos los nuevos nombres de columna como valores separados por comas en el **nueva columna** campo de nombre de la **Editor de metadatos**.

A continuación, generamos el entrenamiento y utilizados para desarrollar el modelo de predicción del riesgo de conjuntos de pruebas. El conjunto de datos original se divida en conjuntos de entrenamiento y prueba del mismo tamaño utilizando la **dividir datos** módulo. Para crear grupos de igual tamaño, establecemos la **fracción de filas del primer conjunto de datos de salida** opción en 0,5.

### <a name="generate-the-new-dataset"></a>Generar el nuevo conjunto de datos

Dado que el costo de subestimar el riesgo es alto, establecemos el costo de clasificaciones incorrectas similar al siguiente:

- Para casos de alto riesgo clasifican incorrectas como bajo riesgo: 5
- Para los casos de bajo riesgo que se clasificaron incorrectas como de alto riesgo: 1

Para que refleje esta función de costo, se genera un nuevo conjunto de datos. En el nuevo conjunto de datos, cada ejemplo de alto riesgo se replica cinco veces, pero no cambia el número de ejemplos de bajo riesgo. Se dividen los datos en conjuntos de datos antes de la replicación para evitar que la misma fila en ambos conjuntos de entrenamiento y prueba.

Para replicar los datos de alto riesgo, se coloca este código de Python en un **Execute Python Script** módulo:

```Python
import pandas as pd

def azureml_main(dataframe1 = None, dataframe2 = None):

    df_label_1 = dataframe1[dataframe1.iloc[:, 20] == 1]
    df_label_2 = dataframe1[dataframe1.iloc[:, 20] == 2]

    result = df_label_1.append([df_label_2] * 5, ignore_index=True)
    return result,
```

El **Execute Python Script** módulo replica los conjuntos de datos de aprendizaje y prueba.

### <a name="feature-engineering"></a>Ingeniería de características

El **Two-Class Support Vector Machine** algoritmo requiere datos normalizados. Por lo que usamos el **Normalize Data** módulo para normalizar los intervalos de todas las características numéricas con un `tanh` transformación. Un `tanh` transformación convierte todas las características numéricas en valores dentro de un intervalo de 0 y 1 conservando la distribución global de valores.

El **Two-Class Support Vector Machine** módulo controla las características de cadena, convertirlos en características categóricas y características, a continuación, en binario con un valor de 0 ó 1. Por lo que no es necesario normalizar estas características.

## <a name="models"></a>Modelos

Dado que aplicamos dos clasificadores **Two-Class Support Vector Machine** (SVM) y **Two-Class Boosted Decision Tree**y también utilizar dos conjuntos de datos, se genera un total de cuatro modelos:

- SVM entrenado con datos originales.
- SVM entrenado con los datos replicados.
- Árbol de decisión impulsado entrenado con datos originales.
- Árbol de decisión impulsado entrenado con los datos replicados.

Utilizamos el flujo de trabajo experimental estándar para crear, entrenar y probar los modelos:

1. Inicializar los algoritmos de aprendizaje mediante **Two-Class Support Vector Machine** y **Two-Class Boosted Decision Tree**.
1. Use **Train Model** para aplicar el algoritmo a los datos y crear el modelo real.
1. Use **Score Model** para generar puntuaciones con los ejemplos de prueba.

El siguiente diagrama muestra una parte de este experimento, en que se usan los conjuntos de entrenamiento original y se replican para entrenar dos modelos SVM diferentes. **Entrenar modelo** está conectado al conjunto de entrenamiento, y **Score Model** está conectado al conjunto de prueba.

![Gráfico de experimento](media/ui-sample-classification-predict-credit-risk-cost-sensitive/score-part.png)

En la fase de evaluación del experimento, se calcula la precisión de cada uno de los cuatro modelos. En este experimento utilizamos **Evaluate Model** para comparar los ejemplos que tienen la misma clasificación incorrecta de costo.

El **Evaluate Model** módulo puede calcular las métricas de rendimiento de como máximo dos modelos de puntuación. Por lo que usamos una instancia de **Evaluate Model** para evaluar los dos modelos SVM y otra instancia de **Evaluate Model** para evaluar los dos modelos de árbol de decisión impulsado.

Observe que el conjunto de datos replicados de prueba se usa como entrada para **Score Model**. En otras palabras, las puntuaciones de precisión final incluyen el costo para obtener las etiquetas mal.

## <a name="combine-multiple-results"></a>Combinar varios resultados

El **Evaluate Model** módulo genera una tabla con una sola fila que contiene varias métricas. Para crear un único conjunto de resultados de precisión, primero usamos **agregar filas** para combinar los resultados en una sola tabla. A continuación, usamos el siguiente script de Python en el **Execute Python Script** módulo para agregar el nombre del modelo y el enfoque de aprendizaje para cada fila de la tabla de resultados:

```Python
import pandas as pd

def azureml_main(dataframe1 = None, dataframe2 = None):

    new_cols = pd.DataFrame(
            columns=["Algorithm","Training"],
            data=[
                ["SVM", "weighted"],
                ["SVM", "unweighted"],
                ["Boosted Decision Tree","weighted"],
                ["Boosted Decision Tree","unweighted"]
            ])

    result = pd.concat([new_cols, dataframe1], axis=1)
    return result,
```

## <a name="results"></a>Resultados

Para ver los resultados del experimento, haga clic en la salida de visualizar del último **Select Columns in Dataset** módulo.

![Visualizar la salida](media/ui-sample-classification-predict-credit-risk-cost-sensitive/result.png)

La primera columna muestra el algoritmo utilizado para generar el modelo de aprendizaje automático.
La segunda columna indica el tipo de conjunto de entrenamiento.
La tercera columna contiene el valor de precisión sensibles a los costes.

En estos resultados, puede ver que se proporciona la mejor precisión mediante el modelo que se creó con **Two-Class Support Vector Machine** y entrenado en el conjunto de datos de entrenamiento replicada.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [aml-ui-cleanup](../../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

Explore los ejemplos disponibles para la interfaz visual:

- [Ejemplo 1: regresión: Predecir el precio de un automóvil](ui-sample-regression-predict-automobile-price-basic.md)
- [Ejemplo 2: regresión: Compare de algoritmos para la predicción de precios de automóviles](ui-sample-regression-predict-automobile-price-compare-algorithms.md)
- [Ejemplo 3: clasificación: Predecir el riesgo de crédito](ui-sample-classification-predict-credit-risk-basic.md)
- [Ejemplo 5: clasificación: Predecir el abandono](ui-sample-classification-predict-churn.md)
