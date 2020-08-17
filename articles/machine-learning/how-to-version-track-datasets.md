---
title: Control de versiones del conjunto de datos
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo mejorar la versión de los conjuntos de datos y sobre cómo funciona el control de versiones con las canalizaciones de aprendizaje automático.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sihhu
author: MayMSFT
ms.reviewer: nibaccam
ms.date: 03/09/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 4ae0ebaec011634153e4ac9bbe43ef519c58a251
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87845891"
---
# <a name="version-and-track-datasets-in-experiments"></a>Versión y seguimiento de conjuntos de valores en experimentos
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo, aprenderá a controlar versiones y realizar un seguimiento de los conjuntos de datos de Azure Machine Learning para fines de reproducibilidad. El control de versiones del conjunto de datos es una manera de delimitar el estado de los datos, con el fin de que pueda aplicar una versión específica del conjunto de datos para futuros experimentos.

Escenarios de control de versiones típicos:

* Cuando hay nuevos datos disponibles para el reentrenamiento
* Cuando se aplican diferentes enfoques de preparación de datos o de ingeniería de características

## <a name="prerequisites"></a>Prerrequisitos

En este tutorial, necesitará:

- [El SDK de Azure Machine Learning para Python instalado](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py). Este SDK incluye el paquete de [azureml-datasets](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset?view=azure-ml-py).
    
- Un [área de trabajo de Azure Machine Learning](concept-workspace.md). Para recuperar una existente, ejecute el código siguiente o [cree una nueva área de trabajo](how-to-manage-workspace.md).

    ```Python
    import azureml.core
    from azureml.core import Workspace
    
    ws = Workspace.from_config()
    ```
- Un [conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md).

<a name="register"></a>

## <a name="register-and-retrieve-dataset-versions"></a>Registrar y recuperar versiones del conjunto de datos

Al registrar un conjunto de datos, puede mejorar su versión, volver a utilizarlo y compartirlo entre experimentos y con colegas. Puede registrar varios conjuntos de datos con el mismo nombre y recuperar una versión específica por nombre y número de versión.

### <a name="register-a-dataset-version"></a>Registrar una versión del conjunto de datos

El código siguiente registra una nueva versión del conjunto de datos `titanic_ds` estableciendo el parámetro `create_new_version` en `True`. Si no hay ningún conjunto de datos de `titanic_ds` existente registrado en el área de trabajo, el código crea un nuevo conjunto de datos con el nombre `titanic_ds` y establece su versión en 1.

```Python
titanic_ds = titanic_ds.register(workspace = workspace,
                                 name = 'titanic_ds',
                                 description = 'titanic training data',
                                 create_new_version = True)
```

### <a name="retrieve-a-dataset-by-name"></a>Recuperar un conjunto de datos por nombre

De manera predeterminada, el método [get_by_name()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#get-by-name-workspace--name--version--latest--) de la clase `Dataset` devuelve la versión más reciente del conjunto de datos registrada en el área de trabajo. 

El código siguiente obtiene la versión 1 del conjunto de datos `titanic_ds`.

```Python
from azureml.core import Dataset
# Get a dataset by name and version number
titanic_ds = Dataset.get_by_name(workspace = workspace,
                                 name = 'titanic_ds', 
                                 version = 1)
```

<a name="best-practice"></a>

## <a name="versioning-best-practice"></a>Procedimientos recomendados de control de versiones

Cuando crea una versión de conjunto de datos, *no* está creando una copia adicional de datos con el área de trabajo. Dado que los conjuntos de datos son referencias a los datos en el servicio de almacenamiento, tiene una única fuente verdadera, administrada por el servicio de almacenamiento.

>[!IMPORTANT]
> Si se sobrescriben o se eliminan los datos a los que hace referencia el conjunto de datos, la llamada a una versión específica del conjunto de datos *no* revierte el cambio.

Cuando se cargan datos de un conjunto de datos, siempre se carga el contenido de datos actual al que hace referencia el conjunto de datos. Si quiere asegurarse de que cada versión del conjunto de datos se puede reproducir, se recomienda no modificar el contenido de datos al que hace referencia la versión del conjunto de datos. Cuando lleguen nuevos datos, guarde los nuevos archivos de datos en una carpeta de datos independiente y, después, cree una nueva versión del conjunto de datos para incluir los datos de esa nueva carpeta.

En la siguiente imagen y código de ejemplo se muestra el método recomendado para estructurar las carpetas de datos y crear versiones del conjunto de datos que hacen referencia a esas carpetas:

![Estructura de carpetas](./media/how-to-version-track-datasets/folder-image.png)

```Python
from azureml.core import Dataset

# get the default datastore of the workspace
datastore = workspace.get_default_datastore()

# create & register weather_ds version 1 pointing to all files in the folder of week 27
datastore_path1 = [(datastore, 'Weather/week 27')]
dataset1 = Dataset.File.from_files(path=datastore_path1)
dataset1.register(workspace = workspace,
                  name = 'weather_ds',
                  description = 'weather data in week 27',
                  create_new_version = True)

# create & register weather_ds version 2 pointing to all files in the folder of week 27 and 28
datastore_path2 = [(datastore, 'Weather/week 27'), (datastore, 'Weather/week 28')]
dataset2 = Dataset.File.from_files(path = datastore_path2)
dataset2.register(workspace = workspace,
                  name = 'weather_ds',
                  description = 'weather data in week 27, 28',
                  create_new_version = True)

```

<a name="pipeline"></a>

## <a name="version-a-pipeline-output-dataset"></a>Controlar la versión de un conjunto de datos de salida de canalización

Puede usar un conjunto de datos como entrada y salida de cada paso de canalización de Machine Learning. Al volver a ejecutar las canalizaciones, la salida de cada paso de la canalización se registra como una nueva versión del conjunto de datos.

Dado que las canalizaciones de Machine Learning rellenan la salida de cada paso en una nueva carpeta cada vez que se vuelve a ejecutar la canalización, los conjuntos de datos de salida con versión se pueden reproducir. Más información sobre los [conjuntos de datos en canalizaciones](how-to-create-your-first-pipeline.md#steps).

```Python
from azureml.core import Dataset
from azureml.pipeline.steps import PythonScriptStep
from azureml.pipeline.core import Pipeline, PipelineData
from azureml.core. runconfig import CondaDependencies, RunConfiguration

# get input dataset 
input_ds = Dataset.get_by_name(workspace, 'weather_ds')

# register pipeline output as dataset
output_ds = PipelineData('prepared_weather_ds', datastore=datastore).as_dataset()
output_ds = output_ds.register(name='prepared_weather_ds', create_new_version=True)

conda = CondaDependencies.create(
    pip_packages=['azureml-defaults', 'azureml-dataprep[fuse,pandas]'], 
    pin_sdk_version=False)

run_config = RunConfiguration()
run_config.environment.docker.enabled = True
run_config.environment.python.conda_dependencies = conda

# configure pipeline step to use dataset as the input and output
prep_step = PythonScriptStep(script_name="prepare.py",
                             inputs=[input_ds.as_named_input('weather_ds')],
                             outputs=[output_ds],
                             runconfig=run_config,
                             compute_target=compute_target,
                             source_directory=project_folder)
```

<a name="track"></a>

## <a name="track-datasets-in-experiments"></a>Seguimiento de conjuntos de valores en experimentos

Para cada experimento de Machine Learning, puede realizar fácilmente el seguimiento de los conjuntos de datos que se usan como entrada mediante el objeto `Run` del experimento.

El código siguiente usa el método [`get_details()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run.run?view=azure-ml-py#get-details--) para realizar un seguimiento de los conjuntos de datos de entrada que se utilizaron en la ejecución del experimento:

```Python
# get input datasets
inputs = run.get_details()['inputDatasets']
input_dataset = inputs[0]['dataset']

# list the files referenced by input_dataset
input_dataset.to_path()
```

También encontrará los `input_datasets` en los experimentos con https://ml.azure.com/. 

En la imagen siguiente se muestra dónde encontrar el conjunto de datos de entrada de un experimento en Azure Machine Learning Studio. En este ejemplo, vaya al panel **Experimentos** y abra la pestaña **Propiedades** para una ejecución concreta del experimento, `keras-mnist`.

![Conjuntos de datos de entrada](./media/how-to-version-track-datasets/input-datasets.png)

Use el código siguiente para registrar modelos con conjuntos de datos:

```Python
model = run.register_model(model_name='keras-mlp-mnist',
                           model_path=model_path,
                           datasets =[('training data',train_dataset)])
```

Después del registro, puede ver la lista de modelos registrados con el conjunto de datos mediante Python o ir a https://ml.azure.com/.

La siguiente vista es del panel **Conjunto de datos** en **Activos**. Seleccione el conjunto de datos y, después, seleccione la pestaña **Modelos** para obtener una lista de modelos registrados el conjunto de datos. 

![Modelos de conjuntos de datos de entrada](./media/how-to-version-track-datasets/dataset-models.png)

## <a name="next-steps"></a>Pasos siguientes

* [Entrenamiento con conjuntos de datos](how-to-train-with-datasets.md)
* [Más cuadernos de conjuntos de datos de ejemplo](https://aka.ms/dataset-tutorial)
