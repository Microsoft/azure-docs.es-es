---
title: Configuración de la validación cruzada y las divisiones de datos en experimentos de aprendizaje automático automatizados
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo configurar la validación cruzada y las divisiones de datos en experimentos de aprendizaje automático automatizados
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: cesardl
author: CESARDELATORRE
ms.reviewer: nibaccam
ms.date: 06/16/2020
ms.openlocfilehash: 3e2711243d7c093d3ab8aa5f0e7ebac0a5ec95f9
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90886201"
---
# <a name="configure-data-splits-and-cross-validation-in-automated-machine-learning"></a>Configuración de las divisiones de datos y la validación cruzada en aprendizaje automático automatizado

En este artículo, obtendrá información sobre las distintas opciones para configurar las divisiones de datos y la validación cruzada para entrenamiento y validación para los experimentos de aprendizaje automático automatizado (AutoML).

En Azure Machine Learning, cuando se usa AutoML para crear varios modelos de aprendizaje automático, cada ejecución secundaria debe validar el modelo relacionado al calcular las métricas de calidad para ese modelo, como la precisión o la AUC ponderada. Estas métricas se calculan al comparar las predicciones realizadas con cada modelo con las etiquetas reales de observaciones anteriores en los datos de validación. 

Los experimentos de AutoML realizan la validación de modelos de forma automática. En las secciones siguientes se describe cómo puede personalizar aún más la configuración de validación con el [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/?view=azure-ml-py&preserve-view=true). 

Para obtener una experiencia sin código o con poco código, consulte [Creación de experimentos de aprendizaje automático automatizados en Azure Machine Learning Studio](how-to-use-automated-ml-for-ml-models.md). 

> [!NOTE]
> En la actualidad, Studio admite las opciones de división de datos y validación cruzada para validación cruzada y entrenamiento, pero no permite especificar los archivos de datos individuales para el conjunto de validación. 

## <a name="prerequisites"></a>Requisitos previos

Para realizar este artículo, necesitará lo siguiente

* Un área de trabajo de Azure Machine Learning. Para crear el área de trabajo, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* Familiaridad básica con la configuración de una experimento de aprendizaje de automático automatizado con el SD de Azure Machine Learning. Siga el [tutorial](tutorial-auto-train-models.md) o los [procedimientos](how-to-configure-auto-train.md) para ver los principales modelos de diseño del experimento de aprendizaje automático automatizado.

* Comprensión de los datos de validación cruzada y las divisiones de datos de entrenamiento y validación como conceptos de ML. Para obtener una explicación de alto nivel,

    * [Acerca de los conjuntos de entrenamiento, validación y pruebas en Machine Learning](https://towardsdatascience.com/train-validation-and-test-sets-72cb40cba9e7)

    * [Información sobre la validación cruzada](https://towardsdatascience.com/understanding-cross-validation-419dbd47e9bd)

## <a name="default--data-splits-and-cross-validation"></a>Divisiones de datos y validación cruzada predeterminadas

Use el objeto [AutoMLConfig](https://docs.microsoft.com/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig?view=azure-ml-py&preserve-view=true) para definir la configuración del experimento y el entrenamiento. En el siguiente fragmento de código, observe que solo se han definido los parámetros necesarios; es decir, los parámetros de `n_cross_validation` o `validation_ data` **no** están incluidos.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             label_column_name = 'Class'
                            )
```

Si no especifica explícitamente un parámetro `validation_data` o `n_cross_validation`, AutoML aplica las técnicas predeterminadas en función del número de filas del conjunto de datos `training_data` único proporcionado:

|Tamaño&nbsp;de datos de&nbsp;entrenamiento| Técnica de validación |
|---|-----|
|**Mayor&nbsp;que&nbsp;20 000&nbsp;filas**| Se aplica la división de datos de entrenamiento o validación. El valor predeterminado consiste en usar el 10 % del conjunto de datos de entrenamiento inicial como conjunto de validación. A su vez, ese conjunto de validación se usa para calcular las métricas.
|**Menor&nbsp;que&nbsp;20 000&nbsp;filas**| Se aplica el enfoque de validación cruzada. El número predeterminado de iteraciones depende del número de filas. <br> **Si el conjunto de datos tiene menos de 1000 filas**, se usan diez iteraciones. <br> **Si hay entre 1000 y 20 000 filas**, se usan tres iteraciones.

## <a name="provide-validation-data"></a>Especificación de datos de validación

En este caso, puede empezar con un solo archivo de datos y dividirlo en conjuntos de entrenamiento y validación, o bien puede proporcionar un archivo de datos independiente como conjunto de validación. En cualquier caso, el parámetro `validation_data` del objeto `AutoMLConfig` asigna los datos que se van a usar como conjunto de validación. Este parámetro solo acepta los conjuntos de datos en forma de [conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md) o un dataframe de Pandas.   

En el ejemplo de código siguiente se define de forma explícita qué parte de los datos proporcionados de `dataset` se usará para el entrenamiento y la validación.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

training_data, validation_data = dataset.random_split(percentage=0.8, seed=1)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = training_data,
                             validation_data = validation_data,
                             label_column_name = 'Class'
                            )
```

## <a name="provide-validation-set-size"></a>Especificación del tamaño del conjunto de validación

En este caso, solo se proporciona un único conjunto de datos para el experimento. Es decir, el parámetro `validation_data` **no** se especificó y el conjunto de datos proporcionado se asignó al parámetro `training_data`.  En el objeto `AutoMLConfig`, puede establecer el parámetro `validation_size` para que contenga una parte de los datos de entrenamiento para la validación. Esto significa que AutoML dividirá el conjunto de validación a partir del `training_data` inicial proporcionado. Este valor debe estar entre 0,0 y 1,0 no inclusivo (por ejemplo, 0,2 significa que el 20 % de los datos se conservan para los datos de validación).

Consulte el ejemplo de código siguiente:

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             validation_size = 0.2,
                             label_column_name = 'Class'
                            )
```

## <a name="set-the-number-of-cross-validations"></a>Establecimiento del número de validaciones cruzadas

Para realizar la validación cruzada, incluya el parámetro `n_cross_validations` y configúrelo con un valor. Este parámetro establece cuántas validaciones cruzadas se deben realizar, en función del mismo número de iteraciones.

En el código siguiente, se definen cinco iteraciones para la validación cruzada. Por lo tanto, se usan cinco entrenamientos diferentes, de modo que cada entrenamiento usa 4/5 de los datos, y cada validación usa 1/5 de los datos con una iteración de datos de exclusión cada vez.

Como resultado, las métricas se calculan con el promedio de las cinco métricas de validación.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             n_cross_validations = 5
                             label_column_name = 'Class'
                            )
```

## <a name="specify-custom-cross-validation-data-folds"></a>Especificación de las iteraciones de datos de validación cruzada personalizadas

También puede proporcionar sus propias iteraciones de datos de validación cruzada. Esto se considera un escenario más avanzado porque se especifican las columnas que se van a dividir y usar para la validación.  Incluya las columnas de división de validación cruzada personalizadas en los datos de entrenamiento y especifique las columnas al rellenar los nombres de columna en el parámetro `cv_split_column_names`. Cada columna representa una división de validación cruzada y se rellena con valores enteros 1 o 0, de modo que 1 indica que la fila se debe usar para el entrenamiento y 0 indica que la fila se debe usar para la validación.

El siguiente fragmento de código contiene datos de marketing bancario con dos columnas de división de validación cruzada "cv1" y "cv2".

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_with_cv.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             label_column_name = 'y',
                             cv_split_column_names = ['cv1', 'cv2']
                            )
```

> [!NOTE]
> Para usar `cv_split_column_names` con `training_data` y `label_column_name`, actualice el SDK de Azure Machine Learning para Python versión 1.6.0 o posterior. En el caso de las versiones anteriores del SDK, consulte cómo usar `cv_splits_indices`, pero tenga en cuenta que solo se usa con la entrada de conjunto de datos `X` e `y` únicamente. 

## <a name="next-steps"></a>Pasos siguientes

* [Impedir los datos desequilibrados y el sobreajuste](concept-manage-ml-pitfalls.md).
* [Tutorial: Uso del aprendizaje automático para crear predecir tarifas de taxi: sección de la división de datos](tutorial-auto-train-models.md#split-the-data-into-train-and-test-sets).
* [Entrenamiento automático de un modelo de previsión de series temporales](how-to-auto-train-forecast.md).
