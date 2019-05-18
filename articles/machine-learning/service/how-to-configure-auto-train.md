---
title: Creación de experimentos de ML automatizados
titleSuffix: Azure Machine Learning service
description: El aprendizaje automático automatizado elige un algoritmo y genera un modelo listo para la implementación. Obtenga información sobre las opciones que puede usar para configurar experimentos de aprendizaje automático automatizado.
author: nacharya1
ms.author: nilesha
ms.reviewer: sgilley
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 05/02/2019
ms.custom: seodec18
ms.openlocfilehash: 194902cfa2992e4370b68bf140ec3a5e03f364ca
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65597681"
---
# <a name="configure-automated-ml-experiments-in-python"></a>Configurar automatizadas experimentos de aprendizaje automático en Python

En esta guía, aprenderá a definir diversos valores de configuración de su automatizadas experimentos de aprendizaje automático con la [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py). El aprendizaje automático automatizado elige un algoritmo e hiperparámetros, y genera un modelo listo para la implementación. Se pueden usar varias opciones para configurar experimentos de aprendizaje automático automatizado.

Para ver ejemplos de experimentos de aprendizaje automático automatizado, vea [Tutorial: Train a classification model with automated machine learning](tutorial-auto-train-models.md) (Tutorial: Entrenamiento de un modelo de clasificación con aprendizaje automático automatizado) o [Entrenamiento de modelos con aprendizaje automático automatizado en la nube mediante Azure Machine Learning](how-to-auto-train-remote.md).

Opciones de configuración disponibles en el aprendizaje automático automatizado:

* Selección del tipo de experimento: Clasificación, regresión o la predicción de Series temporales
* Origen de datos, formatos y datos de captura
* Elección el destino del proceso: local o remoto
* Configuración de experimentos de aprendizaje automático automatizado
* Ejecución de un experimento de aprendizaje automático automatizado
* Explorar las métricas del modelo
* Registrar e implementar el modelo

Si no prefiere una experiencia de ningún código, también puede [crear sus experimentos en Azure portal de aprendizaje de automático automatizada](how-to-create-portal-experiments.md).

## <a name="select-your-experiment-type"></a>Seleccione el tipo de experimento

Antes de comenzar el experimento, debe determinar el tipo de problema de aprendizaje automático que va a resolver. El aprendizaje automático automatizado admite tipos de tareas de clasificación, regresión y previsión.

El aprendizaje automático automatizado admite los siguientes algoritmos durante el proceso de optimización y automatización. Como usuario, no hay ninguna necesidad de especificar el algoritmo. Aunque los algoritmos DNN disponibles durante el entrenamiento, ML automatizada no compila los modelos de DNN.

Clasificación | Regresión | Previsión de Series temporales
|-- |-- |--
[Regresión logística](https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression)| [Red elástica](https://scikit-learn.org/stable/modules/linear_model.html#elastic-net)| [Red elástica](https://scikit-learn.org/stable/modules/linear_model.html#elastic-net)
[Light GBM](https://lightgbm.readthedocs.io/en/latest/index.html)|[Light GBM](https://lightgbm.readthedocs.io/en/latest/index.html)|[Light GBM](https://lightgbm.readthedocs.io/en/latest/index.html)
[Potenciación del gradiente](https://scikit-learn.org/stable/modules/ensemble.html#classification)|[Potenciación del gradiente](https://scikit-learn.org/stable/modules/ensemble.html#regression)|[Potenciación del gradiente](https://scikit-learn.org/stable/modules/ensemble.html#regression)
[Árbol de decisión](https://scikit-learn.org/stable/modules/tree.html#decision-trees)|[Árbol de decisión](https://scikit-learn.org/stable/modules/tree.html#regression)|[Árbol de decisión](https://scikit-learn.org/stable/modules/tree.html#regression)
[K Vecinos más próximos](https://scikit-learn.org/stable/modules/neighbors.html#nearest-neighbors-regression)|[K Vecinos más próximos](https://scikit-learn.org/stable/modules/neighbors.html#nearest-neighbors-regression)|[K Vecinos más próximos](https://scikit-learn.org/stable/modules/neighbors.html#nearest-neighbors-regression)
[SVC lineal](https://scikit-learn.org/stable/modules/svm.html#classification)|[Lazo LARS](https://scikit-learn.org/stable/modules/linear_model.html#lars-lasso)|[Lazo LARS](https://scikit-learn.org/stable/modules/linear_model.html#lars-lasso)
[Clasificación de vectores de soporte de C (SVC)](https://scikit-learn.org/stable/modules/svm.html#classification)|[Descenso de gradiente estocástico (SGD)](https://scikit-learn.org/stable/modules/sgd.html#regression)|[Descenso de gradiente estocástico (SGD)](https://scikit-learn.org/stable/modules/sgd.html#regression)
[Bosque aleatorio](https://scikit-learn.org/stable/modules/ensemble.html#random-forests)|[Bosque aleatorio](https://scikit-learn.org/stable/modules/ensemble.html#random-forests)|[Bosque aleatorio](https://scikit-learn.org/stable/modules/ensemble.html#random-forests)
[Árboles extremadamente aleatorios](https://scikit-learn.org/stable/modules/ensemble.html#extremely-randomized-trees)|[Árboles extremadamente aleatorios](https://scikit-learn.org/stable/modules/ensemble.html#extremely-randomized-trees)|[Árboles extremadamente aleatorios](https://scikit-learn.org/stable/modules/ensemble.html#extremely-randomized-trees)
[Xgboost](https://xgboost.readthedocs.io/en/latest/parameter.html)|[Xgboost](https://xgboost.readthedocs.io/en/latest/parameter.html)| [Xgboost](https://xgboost.readthedocs.io/en/latest/parameter.html)
[Clasificador DNN](https://www.tensorflow.org/api_docs/python/tf/estimator/DNNClassifier)|[Regressor DNN](https://www.tensorflow.org/api_docs/python/tf/estimator/DNNRegressor) | [Regressor DNN](https://www.tensorflow.org/api_docs/python/tf/estimator/DNNRegressor)|
[DNN de clasificador lineal](https://www.tensorflow.org/api_docs/python/tf/estimator/LinearClassifier)|[Regresores lineales](https://www.tensorflow.org/api_docs/python/tf/estimator/LinearRegressor)|[Regresores lineales](https://www.tensorflow.org/api_docs/python/tf/estimator/LinearRegressor)
[Bayes naive](https://scikit-learn.org/stable/modules/naive_bayes.html#bernoulli-naive-bayes)|
[Descenso de gradiente estocástico (SGD)](https://scikit-learn.org/stable/modules/sgd.html#sgd)|


## <a name="data-source-and-format"></a>Formato y origen de datos
El aprendizaje automático automatizado es compatible con los datos que residen en el escritorio local o en la nube, como Azure Blob Storage. Los datos pueden leerse en los formatos de datos compatibles con scikit-learn. Puede leer los datos en:
* Matrices de Numpy X (características) e y (variable de destino o también denominada etiqueta)
* Dataframe de Pandas

Ejemplos:

*   Matrices de Numpy

    ```python
    digits = datasets.load_digits()
    X_digits = digits.data
    y_digits = digits.target
    ```

*   Dataframe de Pandas

    ```python
    import pandas as pd
    from sklearn.model_selection import train_test_split
    df = pd.read_csv("https://automldemods.blob.core.windows.net/datasets/PlayaEvents2016,_1.6MB,_3.4k-rows.cleaned.2.tsv", delimiter="\t", quotechar='"')
    # get integer labels
    y = df["Label"]
    df = df.drop(["Label"], axis=1)
    df_train, _, y_train, _ = train_test_split(df, y, test_size=0.1, random_state=42)
    ```

## <a name="fetch-data-for-running-experiment-on-remote-compute"></a>Capturar datos para ejecutar un experimento en el proceso remoto

Si usa un proceso remoto para ejecutar el experimento, se debe ajustar la operación de captura de datos en un script de python independiente `get_data()`. Este script se ejecuta en el proceso remoto donde se ejecuta el experimento de aprendizaje automático automatizado. `get_data` elimina la necesidad de capturar los datos a través del cable para cada iteración. Sin `get_data`, el experimento generará un error cuando se ejecute en el proceso remoto.

A continuación se incluye un ejemplo de `get_data`:

```python
%%writefile $project_folder/get_data.py
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import LabelEncoder
def get_data(): # Burning man 2016 data
    df = pd.read_csv("https://automldemods.blob.core.windows.net/datasets/PlayaEvents2016,_1.6MB,_3.4k-rows.cleaned.2.tsv", delimiter="\t", quotechar='"')
    # get integer labels
    le = LabelEncoder()
    le.fit(df["Label"].values)
    y = le.transform(df["Label"].values)
    df = df.drop(["Label"], axis=1)
    df_train, _, y_train, _ = train_test_split(df, y, test_size=0.1, random_state=42)
    return { "X" : df, "y" : y }
```

En su objeto `AutoMLConfig`, especifique el parámetro `data_script` y proporcione la ruta de acceso al archivo de script `get_data`, como se indica a continuación:

```python
automl_config = AutoMLConfig(****, data_script=project_folder + "/get_data.py", **** )
```

El script `get_data` puede devolver:

Clave | Type | Se excluye mutuamente con    | DESCRIPCIÓN
---|---|---|---
X | Dataframe de Pandas o matriz de Numpy | data_train, label, columns |  Todas las características que se usan para el aprendizaje
y | Dataframe de Pandas o matriz de Numpy |   etiqueta   | Datos de etiquetas que se usan para el aprendizaje. Para la clasificación, debe ser una matriz de enteros.
X_valid | Dataframe de Pandas o matriz de Numpy   | data_train, label | _Opcional_ Todas las características con las que se debe validar. Si no se especifica, X se divide entre "train" y "validate"
y_valid |   Dataframe de Pandas o matriz de Numpy | data_train, label | _Opcional_ Los datos de etiqueta con los que se va a validar. Si no se especifica, Y se divide entre "train" y "validate"
sample_weight | Dataframe de Pandas o matriz de Numpy |   data_train, label, columns| _Opcional_ Un valor de ponderación para cada ejemplo. Se usa cuando se quieren asignar ponderaciones distintas a los puntos de datos
sample_weight_valid | Dataframe de Pandas o matriz de Numpy | data_train, label, columns |    _Opcional_ Un valor de ponderación para cada ejemplo de validación. Si no se especifica, sample_weight se divide entre "train" y "validate"
data_train |    Dataframe de Pandas |  X, y, X_valid, y_valid |    Todos los datos (características y etiqueta) que se usarán para el aprendizaje
etiqueta | string  | X, y, X_valid, y_valid |  Qué columna de data_train representa la etiqueta
columnas | Matriz de cadenas  ||  _Opcional_ Lista blanca de columnas que se usarán para las características
cv_splits_indices   | Matriz de enteros ||  _Opcional_ Lista de índices para dividir los datos para la validación cruzada

### <a name="load-and-prepare-data-using-data-prep-sdk"></a>Cargue y prepare los datos con el SDK de preparación de datos.
Automatizadas experimentos de machine learning admite la carga de datos y lo transforma mediante el SDK de la preparación de datos. El uso del SDK proporciona la capacidad de

>* Opciones de carga desde varios tipos de archivos con inferencia de parámetros de análisis (codificación, separador, encabezados).
>* Opciones de conversión de tipo usando la inferencia durante la carga de archivos.
>* Compatibilidad con la conexión para MS SQL Server y Azure Data Lake Storage.
>* Adición de una columna mediante una expresión
>* Atribución de valores que faltan
>* Derivación de columnas por ejemplos
>* Filtrado
>* Transformaciones personalizadas de Python

Para obtener información sobre el SDK de preparación de datos, consulte el artículo [Cómo preparar datos para el modelado](how-to-load-data.md).
A continuación se muestra un ejemplo de carga de datos mediante el SDK de preparación de datos.
```python
# The data referenced here was pulled from `sklearn.datasets.load_digits()`.
simple_example_data_root = 'https://dprepdata.blob.core.windows.net/automl-notebook-data/'
X = dprep.auto_read_file(simple_example_data_root + 'X.csv').skip(1)  # Remove the header row.
# You can use `auto_read_file` which intelligently figures out delimiters and datatypes of a file.

# Here we read a comma delimited file and convert all columns to integers.
y = dprep.read_csv(simple_example_data_root + 'y.csv').to_long(dprep.ColumnSelector(term='.*', use_regex = True))
```

## <a name="train-and-validation-data"></a>Datos de entrenamiento y validación

Puede especificar un conjunto distinto de entrenamiento y validación mediante get_data() o, directamente, en el método `AutoMLConfig`.

## <a name="cross-validation-split-options"></a>Opciones de división de validación cruzada

### <a name="k-folds-cross-validation"></a>Validación cruzada de K iteraciones

Use la opción de configuración `n_cross_validations` para especificar el número de validaciones cruzadas. El conjunto de datos de entrenamiento se dividirá aleatoriamente en `n_cross_validations` iteraciones del mismo tamaño. Durante cada ronda de validación cruzada, una de las iteraciones se usará para la validación del modelo entrenado en las iteraciones restantes. Este proceso se repite durante `n_cross_validations` rondas hasta que cada una se utiliza una vez como conjunto de validación. Se notifican las puntuaciones promedio de todas las rondas de `n_cross_validations` y el modelo correspondiente se puede volver a entrenar en todo el conjunto de datos de entrenamiento. 

### <a name="monte-carlo-cross-validation-repeated-random-sub-sampling"></a>Validación de Monte Carlo entre (repetida submuestra aleatoria)

Use `validation_size` para especificar el porcentaje del conjunto de datos de entrenamiento que se debe usar para la validación y use `n_cross_validations` para especificar el número de validaciones cruzadas. Durante cada ronda de validación cruzada, un subconjunto de tamaño `validation_size` se seleccionará aleatoriamente para la validación del modelo entrenado en los datos restantes. Por último, se notifican las puntuaciones promedias de todas las rondas `n_cross_validations` y el modelo correspondiente se puede volver a entrenar en todo el conjunto de datos de entrenamiento. Monte Carlo no se admite para la predicción de series temporales.

### <a name="custom-validation-dataset"></a>Conjunto de datos de validación personalizado

Utilice el conjunto de datos de validación personalizada si la división aleatoria no es aceptable, normalmente datos de series temporales o datos desequilibrados. Puede especificar un conjunto de datos de validación propio. El modelo se evaluará en el conjunto de datos de validación especificado en lugar de un conjunto de datos aleatorio.

## <a name="compute-to-run-experiment"></a>Proceso para ejecutar el experimento

A continuación, determine dónde se va a entrenar el modelo. Un experimento de entrenamiento de aprendizaje automático automatizado se puede ejecutar en las opciones de proceso siguientes:
*   La máquina local, como un escritorio local o un equipo portátil: generalmente, cuando haya un pequeño conjunto de datos y siga en la fase de exploración.
*   Una máquina remota en la nube: [Azure Machine Learning Managed Compute](concept-azure-machine-learning-architecture.md#managed-and-unmanaged-compute-targets) es un servicio administrado que permite entrenar modelos de aprendizaje automático en clústeres de máquinas virtuales de Azure.

Visite el [sitio de GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning) para obtener cuadernos de ejemplo con destinos de proceso locales y remotos.

*   Un clúster de Azure Databricks en su suscripción de Azure. Puede encontrar más detalles aquí: [programa de instalación de Azure Databricks del clúster para ML automatizada](how-to-configure-environment.md#azure-databricks)

Consulte la [sitio de GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/azure-databricks/automl) por ejemplo equipos portátiles con Azure Databricks.

<a name='configure-experiment'></a>

## <a name="configure-your-experiment-settings"></a>Establecer la configuración de experimento

Se pueden usar varias opciones para configurar el experimento de aprendizaje automático automatizado. Estos parámetros se establecen al crear una instancia un objeto `AutoMLConfig`. Consulte la [clase AutoMLConfig](https://docs.microsoft.com/python/api/azureml-train-automl/azureml.train.automl.automlconfig?view=azure-ml-py) para obtener una lista completa de parámetros.  

Estos son algunos ejemplos:

1.  Experimento de clasificación mediante AUC ponderado como la métrica principal con un tiempo máximo de 12 000 segundos por cada iteración. El experimento finaliza después de 50 iteraciones y 2 iteraciones de validación cruzada.

    ```python
    automl_classifier = AutoMLConfig(
        task='classification',
        primary_metric='AUC_weighted',
        max_time_sec=12000,
        iterations=50,
        blacklist_models='XGBoostClassifier',
        X=X,
        y=y,
        n_cross_validations=2)
    ```
2.  A continuación se incluye un ejemplo de un conjunto de experimentos de regresión que finaliza después de 100 iteraciones. Cada iteración tiene una duración de hasta 600 segundos con 5 iteraciones de validación cruzada.

    ```python
    automl_regressor = AutoMLConfig(
        task='regression',
        max_time_sec=600,
        iterations=100,
        whitelist_models='kNN regressor'
        primary_metric='r2_score',
        X=X,
        y=y,
        n_cross_validations=5)
    ```

Las tres diferentes `task` los valores de parámetro determinan la lista de algoritmos que se aplicará.  Use los parámetros `whitelist` o `blacklist` para modificar aún más las iteraciones con los algoritmos disponibles para incluir o excluir. Encontrará la lista de los modelos admitidos en [SupportedAlgorithms clase](https://docs.microsoft.com/python/api/azureml-train-automl/azureml.train.automl.constants.supportedalgorithms?view=azure-ml-py).

## <a name="primary-metric"></a>Métrica principal
La métrica principal; como se muestra en los ejemplos anteriores determina la métrica que se usará durante el entrenamiento del modelo para la optimización. La métrica principal que puede seleccionar viene determinada por el tipo de tarea que elija. A continuación es una lista de métricas disponibles.

|Clasificación | Regresión | Previsión de Series temporales
|-- |-- |--
|accuracy| spearman_correlation | spearman_correlation
|AUC_weighted | normalized_root_mean_squared_error | normalized_root_mean_squared_error
|average_precision_score_weighted | r2_score | r2_score
|norm_macro_recall | normalized_mean_absolute_error | normalized_mean_absolute_error
|precision_score_weighted |

## <a name="data-preprocessing--featurization"></a>Características y preprocesamiento de datos

En cada experimento de aprendizaje automático automatizada, los datos están [escalan automáticamente y normaliza](concept-automated-ml.md#preprocess) para ayudar a los algoritmos funcionan bien.  Sin embargo, también puede habilitar adicionales preprocesamiento o características, como la falta de imputación de valores, la codificación y las transformaciones. [Más información sobre qué características se incluye](how-to-create-portal-experiments.md#preprocess). 

Para habilitar esta características, especifique `"preprocess": True` para el [ `AutoMLConfig` clase](https://docs.microsoft.com/python/api/azureml-train-automl/azureml.train.automl.automlconfig?view=azure-ml-py).

## <a name="time-series-forecasting"></a>Previsión de Series temporales
Tipo de tarea de previsión de tiempo serie tiene parámetros adicionales que se va a definir.
1. time_column_name - se trata de un parámetro necesario que define el nombre de la columna de la serie de fecha y hora que contiene datos de entrenamiento. 
1. max_horizon: Esto define el período de tiempo que desea predecir horizontalmente en función de la periodicidad de los datos de entrenamiento. Por ejemplo si tiene datos de entrenamiento con intervalos de agregación diaria, definir hasta qué punto fuera de días que desea que entrenar para el modelo.
1. grain_column_names: Esto define el nombre de columnas que contienen datos de series de tiempo individual en los datos de entrenamiento. Por ejemplo, si la previsión de ventas de una marca concreta por almacén, definiría marca y el almacén de columnas como columnas de su nivel de detalle.

Vea el ejemplo de estos ejemplo Bloc de notas de configuración que se usa a continuación, está disponible [aquí](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-orange-juice-sales/auto-ml-forecasting-orange-juice-sales.ipynb).

```python
# Setting Store and Brand as grains for training.
grain_column_names = ['Store', 'Brand']
nseries = data.groupby(grain_column_names).ngroups

# View the number of time series data with defined grains
print('Data contains {0} individual time-series.'.format(nseries))
```

```python
time_series_settings = {
    'time_column_name': time_column_name,
    'grain_column_names': grain_column_names,
    'drop_column_names': ['logQuantity'],
    'max_horizon': n_test_periods
}

automl_config = AutoMLConfig(task='forecasting',
                             debug_log='automl_oj_sales_errors.log',
                             primary_metric='normalized_root_mean_squared_error',
                             iterations=10,
                             X=X_train,
                             y=y_train,
                             n_cross_validations=5,
                             path=project_folder,
                             verbosity=logging.INFO,
                             **time_series_settings)
```

## <a name="run-experiment"></a>Ejecutar experimento

Envíe el experimento para ejecutar y generar un modelo. Pase `AutoMLConfig` al método `submit` para generar el modelo.

```python
run = experiment.submit(automl_config, show_output=True)
```

>[!NOTE]
>Las dependencias se instalan por primera vez en una máquina nueva.  El resultado puede tardar hasta 10 minutos en mostrarse.
>Establecer `show_output` en `True` genera un resultado que se muestra en la consola.

## <a name="exit-criteria"></a>Criterios de salida 
Hay unas cuantas opciones puede definir para completar el experimento.
1. Ningún criterio - si no se define ninguna salga el experimento, continuarán hasta que no se realiza ningún progreso adicional en la métrica principal de parámetros. 
1. Número de iteraciones: defina el número de iteraciones del experimento ejecutar. También puede opcional agregar iteration_timeout_minutes para definir un límite de tiempo en minutos por cada iteración.
1. Cierre después de un período de tiempo - experiment_timeout_minutes utilizando en la configuración que puede definir cuánto tiempo en minutos debe seguir un experimento en ejecución.
1. Salga de una vez alcanzada una puntuación - mediante experiment_exit_score que puede elegir para completar el experimento una vez alcanzada una puntuación según la métrica principal.

## <a name="explore-model-metrics"></a>Explorar las métricas del modelo
Puede ver los resultados en un widget o en línea si se encuentra en un bloc de notas. Vea [Seguimiento y evaluación de modelos](how-to-track-experiments.md#view-run-details) para obtener más información.


### <a name="classification-metrics"></a>Métricas de clasificación
Las métricas siguientes se guardan en cada iteración de una tarea de clasificación.

|Métrica|DESCRIPCIÓN|Cálculo|Parámetros adicionales
--|--|--|--|
AUC_Macro| AUC es el área bajo la Curva de característica operativa del receptor. Macro es la media aritmética del parámetro AUC para cada clase.  | [Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html) | average="macro"|
AUC_Micro| AUC es el área bajo la Curva de característica operativa del receptor. Micro se calcula de forma global mediante la combinación de los verdaderos positivos y los falsos positivos de cada clase| [Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html) | average="micro"|
AUC_Weighted  | AUC es el área bajo la Curva de característica operativa del receptor. Weighted es la media aritmética de la puntuación para cada clase, ponderada por el número de instancias verdaderas en cada clase.| [Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html)|average="weighted"
accuracy|La precisión es el porcentaje de las etiquetas de predicción que coinciden exactamente con las etiquetas verdaderas. |[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html) |None|
average_precision_score_macro|La precisión promedio resume una curva de precisión-recuperación como la media ponderada de las precisiones conseguidas en cada umbral, donde el aumento de la recuperación del umbral anterior se usa como peso. Macro es la media aritmética de la puntuación de precisión media de cada clase|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="macro"|
average_precision_score_micro|La precisión promedio resume una curva de precisión-recuperación como la media ponderada de las precisiones conseguidas en cada umbral, donde el aumento de la recuperación del umbral anterior se usa como peso. Micro se calcula de forma global mediante la combinación de los verdaderos positivos y los falsos positivos en cada límite|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="micro"|
average_precision_score_weighted|La precisión promedio resume una curva de precisión-recuperación como la media ponderada de las precisiones conseguidas en cada umbral, donde el aumento de la recuperación del umbral anterior se usa como peso. Weighted es la media aritmética de la puntuación de precisión promedio para cada clase, ponderada por el número de instancias verdaderas en cada clase.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="weighted"|
balanced_accuracy|La precisión equilibrada es la media aritmética de recuperación para cada clase.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="macro"|
f1_score_macro|La puntuación F1 es la media armónica de precisión y recuperación. Macro es la media aritmética de la puntuación F1 para cada clase.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="macro"|
f1_score_micro|La puntuación F1 es la media armónica de precisión y recuperación. Micro se calcula de forma global mediante el recuento del total de verdaderos positivos, falsos negativos y falsos positivos.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="micro"|
f1_score_weighted|La puntuación F1 es la media armónica de precisión y recuperación. Media ponderada por frecuencia de clase de la puntuación F1 para cada clase.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="weighted"|
log_loss|Se trata de la función de pérdida que se usa en la regresión logística (multinomial) y sus extensiones como las redes neuronales, definida como la verosimilitud logarítmica negativa de las etiquetas verdaderas, dadas las predicciones de un clasificador probabilístico. Para un ejemplo único con la etiqueta verdadera yt en {0,1} y la probabilidad estimada yp de que yt = 1, la pérdida logarítmica es -log P(yt&#124;yp) = -(yt log(yp) + (1 - yt) log(1 - yp))|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.log_loss.html)|None|
norm_macro_recall|La recuperación de macro normalizada es la recuperación de macro que se ha normalizado para que el rendimiento aleatorio tenga una puntuación de 0 y el rendimiento perfecto una puntuación de 1. Esto se logra mediante norm_macro_recall := (recall_score_macro - R)/(1 - R), donde R es el valor esperado de recall_score_macro para las predicciones aleatorias (es decir, R=0,5 para clasificación binaria y R=(1/C) para problemas de clasificación de clase C)|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average = "macro" y después (recall_score_macro - R)/(1 - R), donde R es el valor esperado de recall_score_macro para las predicciones aleatorias (es decir, R=0,5 para clasificación binaria y R=(1/C) para problemas de clasificación de clase C)|
precision_score_macro|La precisión es el porcentaje de los elementos etiquetados como una clase determinada que se encuentran realmente en esa clase. Macro es la media aritmética de la precisión para cada clase.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="macro"|
precision_score_micro|La precisión es el porcentaje de los elementos etiquetados como una clase determinada que se encuentran realmente en esa clase. Micro se calcula de forma global mediante el recuento del total de verdaderos positivos y falsos positivos.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="micro"|
precision_score_weighted|La precisión es el porcentaje de los elementos etiquetados como una clase determinada que se encuentran realmente en esa clase. Weighted es la media aritmética de la precisión para cada clase, ponderada por el número de instancias verdaderas en cada clase.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="weighted"|
recall_score_macro|Recuperación es el porcentaje de elementos que realmente están una clase determinada y que están correctamente etiquetados. Macro es la media aritmética de la recuperación para cada clase.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="macro"|
recall_score_micro|Recuperación es el porcentaje de elementos que realmente están una clase determinada y que están correctamente etiquetados. Micro se calcula de forma global mediante el recuento del total de verdaderos positivos y falsos negativos.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="micro"|
recall_score_weighted|Recuperación es el porcentaje de elementos que realmente están una clase determinada y que están correctamente etiquetados. Weighted es la media aritmética de la recuperación para cada clase, ponderada por el número de instancias verdaderas en cada clase.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="weighted"|
weighted_accuracy|La precisión ponderada es la precisión donde el peso asignado a cada ejemplo es igual a la proporción de instancias verdades en la clase real de ese ejemplo.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html)|sample_weight es un vector igual a la proporción de esa clase para cada elemento en el destino|

### <a name="regression-and-time-series-forecasting-metrics"></a>Las métricas de previsión de series de regresión y hora
Las métricas siguientes se guardan en cada iteración de una tarea de regresión o previsión.

|Métrica|DESCRIPCIÓN|Cálculo|Parámetros adicionales
--|--|--|--|
explained_variance|La varianza explicada es la proporción que tiene en cuenta un modelo matemático la variación de un conjunto de datos determinado. Es el porcentaje de reducción de la varianza de los datos originales con respecto a la varianza de los errores. Cuando la media de los errores es 0, es igual a la varianza explicada.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.explained_variance_score.html)|None|
r2_score|R2 es el coeficiente de determinación o el porcentaje de reducción de los errores cuadráticos en comparación con un modelo de referencia que da como resultado la media. Cuando la media de los errores es 0, es igual a la varianza explicada.|[Cálculo](https://scikit-learn.org/0.16/modules/generated/sklearn.metrics.r2_score.html)|None|
spearman_correlation|La correlación de Spearman es una medida no paramétrica de la monotonicidad de la relación entre dos conjuntos de datos. A diferencia de la correlación de Pearson, la correlación de Spearman no asume que los conjuntos de datos se distribuyen normalmente. Como sucede con otros coeficientes de correlación, este varía entre -1 y + 1, y 0 implica que no hay ninguna correlación. Las correlaciones de -1 o +1 implican una relación monotónica exacta. Las correlaciones positivas implican que cuando aumenta x, también lo hace y. Las correlaciones negativas implican que cuando aumenta x, y disminuye.|[Cálculo](https://docs.scipy.org/doc/scipy-0.16.1/reference/generated/scipy.stats.spearmanr.html)|None|
mean_absolute_error|El error absoluto medio es el valor esperado del valor absoluto de la diferencia entre el destino y la predicción.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_absolute_error.html)|None|
normalized_mean_absolute_error|El error medio absoluto normalizado es un error medio absoluto dividido por el intervalo de los datos|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_absolute_error.html)|Se divide por el intervalo de los datos|
median_absolute_error|El error medio absoluto es la media de todas las diferencias absolutas entre el destino y la predicción. Esta pérdida es estable para los valores atípicos.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.median_absolute_error.html)|None|
normalized_median_absolute_error|El error medio absoluto normalizado es un error medio absoluto dividido por el intervalo de los datos|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.median_absolute_error.html)|Se divide por el intervalo de los datos|
root_mean_squared_error|El error cuadrático medio es la raíz cuadrada de la diferencia cuadrática esperada entre el destino y la predicción.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_error.html)|None|
normalized_root_mean_squared_error|El error cuadrático medio normalizado es el error cuadrático medio dividido por el intervalo de los datos.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_error.html)|Se divide por el intervalo de los datos|
root_mean_squared_log_error|El error logarítmico cuadrático medio es la raíz cuadrada del error logarítmico cuadrático esperado.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_log_error.html)|None|
normalized_root_mean_squared_log_error|El error logarítmico cuadrático medio normalizado es el error logarítmico cuadrático medio dividido por el intervalo de los datos.|[Cálculo](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_log_error.html)|Se divide por el intervalo de los datos|


## <a name="understand-automated-ml-models"></a>Comprender los modelos de aprendizaje automático automatizados

Cualquier modelo generado mediante Machine Learning automatizado incluye los siguientes pasos:
+ Automatizar la ingeniería de características (si preprocesar = True)
+ Escalado o normalización y el algoritmo con valores hypermeter

Es muy transparente para obtener esta información de la salida fitted_model ML automatizada.

```python
automl_config = AutoMLConfig(…)
automl_run = experiment.submit(automl_config …)
best_run, fitted_model = automl_run.get_output()
```

### <a name="automated-feature-engineering"></a>Ingeniería de características automatizadas

Ver la lista de preprocesamiento y [automatizada de ingeniería de características](concept-automated-ml.md#preprocess) que ocurre cuando preprocesar = True.  

En este ejemplo:
+ Hay 4 características de entrada: Una (numérico), B (numérico), (numérico) de C, D (DateTime)
+ Numéricos de características de C se puede quitar porque es una columna de identificador con todos los valores únicos
+ Las características numéricas A y B tienen valores que faltan y, por tanto, se imputen por Media
+ Característica de fecha y hora D es caracterizará en 11 diferentes características de ingeniería

Utilice estos 2 API en el primer paso del modelo ajustado para saber más.  Consulte [este cuaderno de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand).

+ API de 1: `get_engineered_feature_names()` devuelve una lista de nombres de ingeniería de características.

  Uso: 
  ```python
  fitted_model.named_steps['timeseriestransformer']. get_engineered_feature_names ()
  ```

  ```
  Output: ['A', 'B', 'A_WASNULL', 'B_WASNULL', 'year', 'half', 'quarter', 'month', 'day', 'hour', 'am_pm', 'hour12', 'wday', 'qday', 'week']
  ```

  Esta lista incluye todos los nombres de las características de ingeniería.

  >[!Note]
  >Use 'timeseriestransformer' para la tarea = 'previsión', utilice 'datatransformer' para la tarea "regresión" o 'clasificación'. 

+ API 2: `get_featurization_summary()` devuelve características resumen para todas las características de entrada.

  Uso: 
  ```python
  fitted_model.named_steps['timeseriestransformer'].get_featurization_summary()
  ```

  >[!Note]
  >Use 'timeseriestransformer' para la tarea = 'previsión', utilice 'datatransformer' para la tarea "regresión" o 'clasificación'.

  Salida:
  ```
  [{'RawFeatureName': 'A',
    'TypeDetected': 'Numeric',
    'Dropped': 'No',
    'EngineeredFeatureCount': 2,
    'Tranformations': ['MeanImputer', 'ImputationMarker']},
   {'RawFeatureName': 'B',
    'TypeDetected': 'Numeric',
    'Dropped': 'No',
    'EngineeredFeatureCount': 2,
    'Tranformations': ['MeanImputer', 'ImputationMarker']},
   {'RawFeatureName': 'C',
    'TypeDetected': 'Numeric',
    'Dropped': 'Yes',
    'EngineeredFeatureCount': 0,
    'Tranformations': []},
   {'RawFeatureName': 'D',
    'TypeDetected': 'DateTime',
    'Dropped': 'No',
    'EngineeredFeatureCount': 11,
    'Tranformations': ['DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime']}]
  ```
  
   Donde:
   
   |Salida|Definición|
   |----|--------|
   |RawFeatureName|Nombre de función o columna de entrada desde el conjunto de datos proporcionado.| 
   |TypeDetected|Tipo de datos de la característica de entrada detectado.|
   |Quitar|Indica si la característica de entrada se quita o utilizar.|
   |EngineeringFeatureCount|Número de características que se generan a través de transformaciones de ingeniería de característica automatizada.|
   |Transformaciones|Lista de las transformaciones aplicadas a las características para generar características diseñadas de entrada.|  

### <a name="scalingnormalization-and-algorithm-with-hypermeter-values"></a>Escalado o normalización y el algoritmo con valores hypermeter:

Para entender los valores de escalado o normalización y el algoritmo/hiperparámetros de una canalización, utilice fitted_model.steps. [Más información sobre el escalado o normalización](concept-automated-ml.md#preprocess). Este es una salida de ejemplo:

```
[('RobustScaler', RobustScaler(copy=True, quantile_range=[10, 90], with_centering=True, with_scaling=True)), ('LogisticRegression', LogisticRegression(C=0.18420699693267145, class_weight='balanced', dual=False, fit_intercept=True, intercept_scaling=1, max_iter=100, multi_class='multinomial', n_jobs=1, penalty='l2', random_state=None, solver='newton-cg', tol=0.0001, verbose=0, warm_start=False))
```

Para obtener más detalles, use esta función auxiliar que se muestra en [este cuaderno de ejemplo](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/classification/auto-ml-classification.ipynb).

```python
from pprint import pprint
def print_model(model, prefix=""):
    for step in model.steps:
        print(prefix + step[0])
        if hasattr(step[1], 'estimators') and hasattr(step[1], 'weights'):
            pprint({'estimators': list(e[0] for e in step[1].estimators), 'weights': step[1].weights})
            print()
            for estimator in step[1].estimators:
                print_model(estimator[1], estimator[0]+ ' - ')
        else:
            pprint(step[1].get_params())
            print()
                
print_model(fitted_model)
```

Este es el resultado de ejemplo:

+ Canalización mediante un algoritmo concreto (LogisticRegression con RobustScalar en este caso):

  ```
  RobustScaler
  {'copy': True,
   'quantile_range': [10, 90],
   'with_centering': True,
   'with_scaling': True}
  
  LogisticRegression
  {'C': 0.18420699693267145,
   'class_weight': 'balanced',
   'dual': False,
   'fit_intercept': True,
   'intercept_scaling': 1,
   'max_iter': 100,
   'multi_class': 'multinomial',
   'n_jobs': 1,
   'penalty': 'l2',
   'random_state': None,
   'solver': 'newton-cg',
   'tol': 0.0001,
   'verbose': 0,
   'warm_start': False}
  ```
  
+ Canalización mediante el enfoque de conjunto: En este caso, es un conjunto de 2 diferentes canalizaciones

  ```
  prefittedsoftvotingclassifier
  {'estimators': ['1', '18'],
  'weights': [0.6666666666666667,
              0.3333333333333333]}

  1 - RobustScaler
  {'copy': True,
   'quantile_range': [25, 75],
   'with_centering': True,
   'with_scaling': False}
  
  1 - LightGBMClassifier
  {'boosting_type': 'gbdt',
   'class_weight': None,
   'colsample_bytree': 0.2977777777777778,
   'importance_type': 'split',
   'learning_rate': 0.1,
   'max_bin': 30,
   'max_depth': 5,
   'min_child_samples': 6,
   'min_child_weight': 5,
   'min_split_gain': 0.05263157894736842,
   'n_estimators': 200,
   'n_jobs': 1,
   'num_leaves': 176,
   'objective': None,
   'random_state': None,
   'reg_alpha': 0.2631578947368421,
   'reg_lambda': 0,
   'silent': True,
   'subsample': 0.8415789473684211,
   'subsample_for_bin': 200000,
   'subsample_freq': 0,
   'verbose': -10}
  
  18 - StandardScalerWrapper
  {'class_name': 'StandardScaler',
   'copy': True,
   'module_name': 'sklearn.preprocessing.data',
   'with_mean': True,
   'with_std': True}
  
  18 - LightGBMClassifier
  {'boosting_type': 'goss',
   'class_weight': None,
   'colsample_bytree': 0.2977777777777778,
   'importance_type': 'split',
   'learning_rate': 0.07894947368421053,
   'max_bin': 30,
   'max_depth': 6,
   'min_child_samples': 47,
   'min_child_weight': 0,
   'min_split_gain': 0.2631578947368421,
   'n_estimators': 400,
   'n_jobs': 1,
   'num_leaves': 14,
   'objective': None,
   'random_state': None,
   'reg_alpha': 0.5789473684210527,
   'reg_lambda': 0.7894736842105263,
   'silent': True,
   'subsample': 1,
   'subsample_for_bin': 200000,
   'subsample_freq': 0,
   'verbose': -10}
  ```
  
<a name="explain"></a>

## <a name="explain-the-model-interpretability"></a>Explica el modelo (interoperabilidad)

El aprendizaje automático automatizado permite comprender la importancia de las características.  Durante el proceso de entrenamiento, puede obtener la importancia de características global para el modelo.  Para escenarios de clasificación, también puede obtener importancia de características de nivel de clase.  Debe proporcionar un conjunto de datos de validación (X_valid) para obtener la importancia de características.

Hay dos maneras de generar la importancia de características.

*   Una vez completado un experimento, puede usar el método `explain_model` en cualquier iteración.

    ```python
    from azureml.train.automl.automlexplainer import explain_model

    shap_values, expected_values, overall_summary, overall_imp, per_class_summary, per_class_imp = \
        explain_model(fitted_model, X_train, X_test)

    #Overall feature importance
    print(overall_imp)
    print(overall_summary)

    #Class-level feature importance
    print(per_class_imp)
    print(per_class_summary)
    ```

*   Para ver la importancia de características para todas las iteraciones, establezca la marca `model_explainability` en `True` en AutoMLConfig.

    ```python
    automl_config = AutoMLConfig(task = 'classification',
                                 debug_log = 'automl_errors.log',
                                 primary_metric = 'AUC_weighted',
                                 max_time_sec = 12000,
                                 iterations = 10,
                                 verbosity = logging.INFO,
                                 X = X_train,
                                 y = y_train,
                                 X_valid = X_test,
                                 y_valid = y_test,
                                 model_explainability=True,
                                 path=project_folder)
    ```

    Después, puede usar el método retrieve_model_explanation para recuperar la importancia de características para una iteración concreta.

    ```python
    from azureml.train.automl.automlexplainer import retrieve_model_explanation

    shap_values, expected_values, overall_summary, overall_imp, per_class_summary, per_class_imp = \
        retrieve_model_explanation(best_run)

    #Overall feature importance
    print(overall_imp)
    print(overall_summary)

    #Class-level feature importance
    print(per_class_imp)
    print(per_class_summary)
    ```

Puede visualizar el gráfico de importancia de características en el área de trabajo de Azure Portal. El gráfico también se muestra al usar el widget de Jupyter en un cuaderno. Para más información sobre los gráficos, consulte el artículo [Cuadernos de ejemplo de Azure Machine Learning Service](samples-notebooks.md).

```Python
from azureml.widgets import RunDetails
RunDetails(local_run).show()
```
![Gráfico de importancia de características](./media/how-to-configure-auto-train/feature-importance.png)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre [cómo y dónde implementar un modelo](how-to-deploy-and-where.md).

Obtenga más información sobre [cómo entrenar un modelo de regresión con Automated machine learning](tutorial-auto-train-models.md) o [cómo entrenar mediante automatizada de machine learning en un recurso remoto](how-to-auto-train-remote.md).
