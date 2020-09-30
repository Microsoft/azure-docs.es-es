---
title: 'Detección de anomalías basada en PCA: Referencia del módulo'
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo PCA-Based Anomaly Detection (Detección de anomalías basada en PCA) para crear un modelo de detección de anomalías basándose en el análisis de componentes principales (PCA).
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/22/2020
ms.openlocfilehash: fa90fcb9ebc17be9a658b08873234eada98b0fba
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898463"
---
# <a name="pca-based-anomaly-detection-module"></a>Módulo PCA-Based Anomaly Detection

En este artículo se describe cómo usar el módulo Detección de anomalías basada en PCA en el diseñador de Azure Machine Learning para crear un modelo de detección de anomalías basado en el análisis de componentes principales (PCA).

Este módulo ayuda a compilar un modelo en escenarios en los que es sencillo obtener datos de entrenamiento de una clase, como las transacciones válidas, pero es difícil obtener suficientes muestras de las anomalías de destino. 

Por ejemplo, para detectar transacciones fraudulentas, a menudo no hay suficientes ejemplos de fraude para realizar el entrenamiento. Sin embargo puede tener muchos ejemplos de transacciones correctas. El módulo PCA-Based Anomaly Detection (Detección de anomalías basada en PCA) soluciona el problema mediante el análisis de las características disponibles para determinar qué constituye una clase "normal". Después, el módulo aplica las métricas de distancia para identificar los casos que representan anomalías. Este enfoque le permite entrenar un modelo usando los datos descompensados disponibles.

## <a name="more-about-principal-component-analysis"></a>Más información acerca del análisis de componentes principales

PCA es una técnica establecida en el aprendizaje automático. Se usa con frecuencia en análisis de datos exploratorios porque revela la estructura interna de los datos y explica la variación en los mismos.

El PCA analiza datos que contienen varias variables. Busca correlaciones entre las variables y determina la combinación de valores que mejor capta las diferencias en los resultados. Estos valores de características combinados se usan para crear un espacio de características más compacto denominado *componentes principales*.

En la detección de anomalías, cada nueva entrada se analiza. El algoritmo de detección de anomalías calcula su proyección en los vectores propios, junto con un error de reconstrucción normalizado. Este error normalizado se usa como puntuación de anomalías. Cuanto mayor sea el error, más errónea es la instancia.

Para más información sobre cómo funciona el PCA y sobre su implementación para la detección de anomalías, consulte estos documentos:

- [Un algoritmo aleatorio para el análisis de componentes principales](https://arxiv.org/abs/0809.2274) de Rokhlin, Szalan y Tygert.

- [Encontrar estructura con aleatoriedad: Algoritmos probabilísticos para construir descomposiciones de matrices aproximadas](http://users.cms.caltech.edu/~jtropp/papers/HMT11-Finding-Structure-SIREV.pdf) (descarga en PDF) de Halko, Martinsson y Tropp.

## <a name="how-to-configure-pca-based-anomaly-detection"></a>Configuración de la detección de anomalías basada en PCA

1. Agregue el módulo **PCA-Based Anomaly Detection** (Detección de anomalías basada en PCA) a la canalización en el diseñador. Puede encontrar este módulo en la categoría **Anomaly Detection** (Detección de anomalías).

2. En el panel derecho del módulo, seleccione la opción **Training mode** (Modo de entrenamiento). Indique si desea entrenar el modelo con un conjunto específico de parámetros o usar un barrido de parámetros para buscar los mejores.

    Si sabe cómo quiere configurar el modelo seleccione **Single Parameter** (Parámetro único), y proporcione un conjunto específico de valores como argumentos.

3. Para **Number of components to use in PCA** (Número de componentes que se van a usar en PCA), especifique el número de características o componentes de salida que desea.

    La decisión de cuántos componentes incluir es una parte importante del diseño de los experimentos con PCA. La norma general es que no debe incluir el mismo número de componentes de PCA que variables. En su lugar, debe empezar con un número menor de componentes y aumentarlos hasta que se cumplan algunos criterios.

    Los mejores resultados se obtienen cuando el número de componentes de salida es *menor que* el número de columnas de características disponibles en el conjunto de datos.

4. Especifique la cantidad de sobremuestreo que se realizará durante el entrenamiento de PCA aleatorio. En problemas de detección de anomalías, los datos desequilibrados dificultan la aplicación de técnicas estándar de PCA. Al especificar una cantidad de sobremuestreo, puede aumentar el número de instancias de destino.

    Si especifica **1**, no se realiza sobremuestreo. Si especifica un valor superior a **1**, se generan ejemplos adicionales para el entrenamiento del modelo.

    Hay dos opciones, dependiendo de si se usa un barrido de parámetros o no:

    - **Parámetro de sobremuestreo para PCA aleatorio**: escriba un número entero único que represente la proporción de sobremuestreo de la clase minoritaria sobre la clase normal. (Esta opción está disponible cuando se usa el método de entrenamiento **Single parameter**[Parámetro único]).

    > [!NOTE]
    > No puede ver el conjunto de datos sobremuestreado. Para más información sobre cómo se usa el sobremuestreo con PCA, consulte [Notas técnicas](#technical-notes).

5. Seleccione la opción **Enable input feature mean normalization** (Habilitar la característica de entrada media de normalización) para normalizar todas las características de entrada a una media de cero. Por lo general, la normalización o el escalado a cero son recomendables para el PCA, ya que el objetivo del PCA es maximizar la varianza entre las variables.

    Esta opción está activada de forma predeterminada. Anúlela si ya se han normalizado los valores mediante un método o una escala diferentes.

6. Conecte un conjunto de datos de entrenamiento etiquetado y uno de los módulos de entrenamiento.

   Si establece la opción **Create trainer mode** (Crear modo entrenador) en **Single Parameter** (Parámetro único), use el módulo [Train Anomaly Detection Model](train-anomaly-detection-model.md) (Entrenamiento de un modelo de detección de anomalías).

7. Envíe la canalización.

## <a name="results"></a>Results

Una vez completado el entrenamiento, puede guardar el modelo entrenado. O bien, puede conectarlo al módulo [Score Model](score-model.md) (Puntuar el modelo) para predecir puntuaciones de anomalías.

Para evaluar los resultados de un modelo de detección de anomalías:

1. Asegúrese de que hay una columna de puntuación disponible en ambos conjuntos de valores.

    Si intenta evaluar un modelo de detección de anomalías y obtiene el error "There is no score column in scored dataset to compare" (No hay ninguna columna de puntuación en el conjunto de datos puntuado para comparar), quiere decir que está usando un conjunto de datos de evaluación típico que contiene una columna de etiqueta pero no tiene ninguna puntuación de probabilidad. Elija un conjunto de datos que coincida con la salida del esquema para los modelos de detección de anomalías, que incluya una columna **Scored Labels** (Etiquetas puntuadas) y una columna **Scored Probabilities** (Probabilidades puntuadas).

2. Asegúrese de que las columnas de etiqueta estén marcadas.

    A veces, se quitan los metadatos asociados a la columna de etiqueta del gráfico de canalización. Si esto ocurre, al usar el módulo [Evaluate Model](evaluate-model.md) (Evaluar modelo) para comparar los resultados de dos modelos de detección de anomalías, puede obtener el error "There is no label column in scored dataset" (No hay columna de etiqueta en el conjunto de datos puntuado). O podría obtener el error "There is no label column in scored dataset to compare" (No hay columna de etiqueta en el conjunto de datos puntuado para comparar).

    Puede evitar estos errores agregando el módulo [Edit Metadata](edit-metadata.md) (Editar metadatos) antes del módulo [Evaluate Model](evaluate-model.md) (Evaluar modelo). Use el selector de columnas para elegir la columna de clase y, en la lista **Fields** (Campos), seleccione **Label** (Etiqueta).

3. Use el módulo [Execute Python Script](execute-python-script.md) (Ejecutar script de Python) para ajustar las categorías de columna de etiqueta como **1 (positivo, normal)** y **0 (negativo, anómalo)** .

    ````
    label_column_name = 'XXX'
    anomaly_label_category = YY
    dataframe1[label_column_name] = dataframe1[label_column_name].apply(lambda x: 0 if x == anomaly_label_category else 1)
    ````

    
## <a name="technical-notes"></a>Notas técnicas

Este algoritmo utiliza PCA para aproximar el subespacio que contiene la clase normal. El subespacio está distribuido por vectores propios asociados a los valores propios superiores de la matriz de covarianza de datos. 

Para cada entrada nueva, el detector de anomalías calcula su proyección de los vectores propios en primer lugar y, a continuación, calcula el error de reconstrucción normalizado. Este error es la puntuación de anomalías. Cuanto mayor sea el error, más errónea es la instancia. Para obtener información detallada sobre cómo se calcula el espacio normal, consulte Wikipedia: [Análisis de componentes principales](https://wikipedia.org/wiki/Principal_component_analysis). 


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 

Consulte [Excepciones y códigos de error para el diseñador](designer-error-codes.md) para ver una lista de errores específicos de los módulos del diseñador.