---
title: 'Referencia para el módulo: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo Evaluar modelo en Azure Machine Learning para medir la precisión de un modelo entrenado.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/27/2020
ms.openlocfilehash: 7f37a598c31f340e66437a6478512fad1f79121f
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87285958"
---
# <a name="evaluate-model-module"></a>Módulo Evaluate Model

En este artículo se describe un módulo del diseñador de Azure Machine Learning (versión preliminar).

Use este módulo para medir la precisión de un modelo entrenado. Hay que proporcionar un conjunto de datos que contenga las puntuaciones generadas a partir de un modelo y el módulo **Evaluate Model** calculará un conjunto de métricas de evaluación estándar del sector.
  
 Las métricas devueltas por **Evaluate Model** dependen del tipo de modelo que va a evaluar:  
  
-   **Modelos de clasificación**    
-   **Modelos de regresión**  
-   **Modelos de agrupación en clústeres**  


> [!TIP]
> Si no está familiarizado con la evaluación de modelos, le recomendamos la serie de vídeos del Dr. Stephen Elston, como parte del [curso de aprendizaje automático](https://blogs.technet.microsoft.com/machinelearning/2015/09/08/new-edx-course-data-science-machine-learning-essentials/) de EdX. 


## <a name="how-to-use-evaluate-model"></a>Cómo usar Evaluar modelo
1. Conecte la salida de **Scored dataset** (Conjunto de datos puntuados) de [Score Model](./score-model.md) (Puntuar modelo) o la salida del conjunto de datos Resultado de [Assign Data to Clusters](./assign-data-to-clusters.md) (Asignar datos a los clústeres) al puerto de entrada izquierdo de **Evaluate Model** (Evaluar modelo). 
    > [!NOTE] 
    > Si se usan módulos como "Seleccionar columnas en conjunto de datos" para seleccionar parte del conjunto de datos de entrada, asegúrese de que la columna de etiqueta Real (usada en el entrenamiento) y las columnas "Probabilidades puntuadas" y "Etiquetas puntuadas" existen para calcular métricas como AUC o Precisión para la clasificación binaria/detección de anomalías.
    > La columna de etiqueta Real y la columna "Etiquetas puntuadas" existen para calcular las métricas para la clasificación o regresión de varias clases.
    > La columna "Assignments" y las columnas "DistancesToClusterCenter no.X" (X es índice centroide, que va de 0, ..., a número de centroides -1) existen para calcular las métricas de la agrupación en clústeres.

2. [Opcional] Conecte la salida de **Scored dataset** (Conjunto de datos puntuados) de [Score Model](./score-model.md) (Puntuar modelo) o la salida del conjunto de datos Resultado de Assign Data to Clusters (Asignar datos a los clústeres) del segundo modelo al puerto de entrada **derecho** de **Evaluate Model** (Evaluar modelo). Puede comparar fácilmente los resultados de dos modelos distintos con los mismos datos. Los dos algoritmos de entrada deben ser del mismo tipo. O bien, puede comparar las puntuaciones de las dos ejecuciones diferentes con los mismos datos con parámetros diferentes.

    > [!NOTE]
    > El tipo de algoritmo hace referencia a la "clasificación de dos clases", la "clasificación multiclase", la "regresión" y la "agrupación en clústeres" en los "algoritmos de aprendizaje automático". 

3. Envíe la canalización para generar las puntuaciones de evaluación.

## <a name="results"></a>Results

Después de ejecutar **Evaluate Model** (Evaluar modelo), seleccione el módulo para abrir el panel de navegación **Evaluate Model** (Evaluar modelo) a la derecha.  Luego, elija la pestaña **Outputs + Logs** (Salidas y registros) y, en esa pestaña, la sección **Data Outputs** (Salidas de datos) tiene varios iconos. El icono **Visualizar** tiene un icono gráfico de barras y es una primera forma de ver los resultados.

En el caso de la clasificación binaria, después de hacer clic en el icono **Visualizar**, puede visualizar la matriz de confusión binaria.
En el caso de la clasificación múltiple, puede encontrar el archivo de trazado de la matriz de confusión en la pestaña **Salidas y registros**, como se muestra a continuación:
> [!div class="mx-imgBorder"]
> ![Vista previa de la imagen cargada](media/module/multi-class-confusion-matrix.png)

Si conecta los conjuntos de datos a las dos entradas de **Evaluate Model**, los resultados contendrán las métricas para ambos conjuntos de datos, o ambos modelos.
El modelo o los datos conectados al puerto izquierdo aparecen en primer lugar en el informe, seguido de las métricas del conjunto de datos, o el modelo conectado al puerto derecho.  

Por ejemplo, la imagen siguiente representa una comparación de los resultados de dos modelos de agrupación en clústeres que se crearon con los mismos datos, pero con distintos parámetros.  

![Comparing2Models](media/module/evaluate-2-models.png)  

Debido a que se trata de un modelo de agrupación en clústeres, los resultados de evaluación son diferentes de si compara las puntuaciones de dos modelos de regresión o si compara dos modelos de clasificación. Pero la presentación general es la misma. 

## <a name="metrics"></a>Métricas

Esta sección describe las métricas devueltas para los tipos específicos de los modelos admitidos para su uso con **Evaluate Model**:

+ [Modelos de clasificación](#metrics-for-classification-models)
+ [Modelos de regresión](#metrics-for-regression-models)
+ [Modelos de agrupación en clústeres](#metrics-for-clustering-models)

### <a name="metrics-for-classification-models"></a>Métricas para modelos de clasificación


Las siguientes métricas se notifican al evaluar los modelos de clasificación binaria.
  
-   **Precisión** (Accuracy) mide la calidad de un modelo de clasificación como la proporción de resultados verdaderos en el total de casos.  
  
-   **Precisión** es la proporción de resultados verdaderos de todos los resultados positivos. Precisión = TP/(TP+FP)  
  
-   **Recuperación** es la fracción de la cantidad total de instancias pertinentes que se recuperaron en realidad. Recuperación = TP/(TP+FN)  
  
-   **Puntuación F1** se calcula como el promedio ponderado de precisión y recuperación entre 0 y 1, donde el valor ideal de puntuación F1 es 1.  
  
-   **AUC** mide el área bajo la curva trazada con los verdaderos positivos en el eje y los falsos positivos en el eje x. Esta métrica es útil porque proporciona un número único que le permite comparar los modelos de diferentes tipos.  


### <a name="metrics-for-regression-models"></a>Métricas para los modelos de regresión
 
Las métricas devueltas para los modelos de regresión están diseñadas para estimar la cantidad de errores.  Se considera que un modelo se ajusta a los datos correctamente si la diferencia entre los valores observados y los previstos es pequeña. Pero el patrón de los valores residuales (la diferencia entre un punto previsto y su valor real correspondiente) puede indicarle mucho sobre el sesgo potencial en el modelo.  
  
 Las siguientes métricas se notifican para evaluar los modelos de regresión.
  
- **Error medio absoluto (MAE)** : mide la proximidad de las predicciones con respecto a los resultados reales; por lo tanto, cuanto menor es la puntuación, mejor.  
  
- **Error cuadrático medio (RMSE)** : crea un valor único que resume el error en el modelo. Al elevar al cuadrado la diferencia, la métrica no tiene en cuenta la diferencia entre un exceso o un defecto de predicción.  
  
- **Error absoluto relativo (RAE)** : es la diferencia absoluta relativa entre los valores esperados y los reales; es relativa porque la diferencia media se divide por la media aritmética.  
  
- **Error cuadrático relativo (RSE)** : del mismo modo, normaliza el error cuadrático medio total de los valores previstos dividiendo entre el total de errores al cuadrado de los valores reales.  
  

  
- **Coeficiente de determinación**: a menudo conocido como R<sup>2</sup>, representa la eficacia predictiva del modelo como un valor entre 0 y 1. Cero significa que el modelo es aleatorio (no explica nada); 1 significa que hay un ajuste perfecto. Pero hay que tener precaución al interpretar los valores de R<sup>2</sup>, ya que los valores bajos pueden ser completamente normales y los valores altos pueden ser sospechosos.

###  <a name="metrics-for-clustering-models"></a>Métricas para modelos de agrupación en clústeres

Dado que los modelos de agrupación en clústeres se diferencian considerablemente de los modelos de clasificación y regresión en muchos aspectos, [Evaluar modelo](evaluate-model.md) también devuelve un conjunto distinto de estadísticas para los modelos de agrupación en clústeres.  
  
 Las estadísticas devueltas para un modelo de agrupación en clústeres describen el número de puntos de datos que se han asignado a cada clúster, la cantidad de separación entre clústeres y cómo de estrechamente están agrupados los puntos de datos dentro de cada clúster.  
  
 El promedio de las estadísticas del modelo de agrupación en clústeres se realiza en todo el conjunto de datos, con filas adicionales que contienen las estadísticas por clúster.  
  
Se notifican las siguientes métricas para evaluar los modelos de agrupación en clústeres.
    
-   Las puntuaciones de la columna **Average Distance to Other Center** (Distancia media a otro centro) representan la cercanía, como media, de cada punto del clúster a los centroides de todos los demás clústeres.   

-   Las puntuaciones de la columna **Average Distance to Cluster Center** (Distancia media al centro del clúster) representan la cercanía de todos los puntos del clúster al centroide de ese clúster.  
  
-   La columna **Number of Points** (Número de puntos) muestra cuántos puntos de datos se han asignado a cada clúster, junto con el número total de puntos de datos en cualquier clúster.  
  
     Si el número de puntos de datos asignado a los clústeres es menor que el número total de puntos de datos disponibles, significa que no se han podido asignar los puntos de datos a un clúster.  
  
-   Las puntuaciones de la columna **Maximal Distance to Cluster Center** (Distancia máxima al centro del clúster) representan las distancias máximas entre cada punto y el centroide del clúster de ese punto.  
  
     Si este número es alto, puede significar que el clúster está muy disperso. Debe revisar esta estadística junto con **Average Distance to Cluster Center** (Distancia media al centro del clúster) para determinar la dispersión del clúster.   

-   La puntuación de **Combined Evaluation** (Evaluación combinada) en la parte inferior de cada sección de resultados indica las puntuaciones medias de los clústeres creados en ese modelo determinado.  
  

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
