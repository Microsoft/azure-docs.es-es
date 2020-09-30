---
title: Destinos de proceso remoto de ML automatizado
titleSuffix: Azure Machine Learning
description: Aprenda a crear modelos mediante el aprendizaje automático automatizado en un destino de proceso remoto de Azure Machine Learning con Azure Machine Learning.
services: machine-learning
author: cartacioS
ms.author: sacartac
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.workload: data-services
ms.topic: conceptual
ms.custom: how-to
ms.date: 03/09/2020
ms.openlocfilehash: 57108bed97b59513309213d55a0654f4405d509c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90893323"
---
# <a name="train-models-with-automated-machine-learning-in-the-cloud"></a>Entrenamiento de modelos con aprendizaje automático automatizado en la nube



En Azure Machine Learning, puede entrenar un modelo en los diferentes tipos de recursos de proceso que administra. El destino de proceso podría ser un equipo local o un recurso en la nube.

Mediante la incorporación de destinos de procesos adicionales como el proceso de Azure Machine Learning (AmlCompute), resulta fácil escalar vertical y horizontalmente el experimento de aprendizaje automático. AmlCompute es una infraestructura de proceso administrado que permite crear fácilmente un proceso de uno o varios nodos.

En este artículo, aprenderá a crear un modelo mediante ML automatizado con AmlCompute.

## <a name="how-does-remote-differ-from-local"></a>¿En qué se diferencia el modo remoto del local?

Hay más características disponibles si usa un destino de proceso remoto.  Para obtener más información, consulte los [destinos de proceso locales y remotos](concept-automated-ml.md#local-remote).

El tutorial "[Uso del aprendizaje automático para crear un modelo de regresión](tutorial-auto-train-models.md)" le enseña a usar un equipo local para entrenar un modelo con ML automatizado. El flujo de trabajo del entrenamiento local también se aplica a los destinos remotos. Para entrenar de forma remota, en primer lugar cree un destino de proceso remoto, como AmlCompute. A continuación, configure el recurso remoto y envíe el código allí.

En este artículo se muestran los pasos adicionales necesarios para ejecutar un experimento de ML automatizado en un destino AmlCompute remoto. Aquí se usa, en todo el código, el objeto del área de trabajo, `ws` del tutorial.

```python
ws = Workspace.from_config()
```

## <a name="create-resource"></a>Crear el recurso

Cree el destino [`AmlCompute`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.amlcompute%28class%29?view=azure-ml-py&preserve-view=true) en el área de trabajo (`ws`) si no existe aún.

**Tiempo estimado**: La creación del destino AmlCompute tarda aproximadamente cinco minutos.

```python
from azureml.core.compute import AmlCompute
from azureml.core.compute import ComputeTarget
import os

# choose a name for your cluster
compute_name = os.environ.get("AML_COMPUTE_CLUSTER_NAME", "cpu-cluster")
compute_min_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MIN_NODES", 0)
compute_max_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MAX_NODES", 4)

# This example uses CPU VM. For using GPU VM, set SKU to STANDARD_NC6
vm_size = os.environ.get("AML_COMPUTE_CLUSTER_SKU", "STANDARD_D2_V2")


if compute_name in ws.compute_targets:
    compute_target = ws.compute_targets[compute_name]
    if compute_target and type(compute_target) is AmlCompute:
        print('found compute target. just use it. ' + compute_name)
else:
    print('creating a new compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size = vm_size,
                                                                min_nodes = compute_min_nodes, 
                                                                max_nodes = compute_max_nodes)

    # create the cluster
    compute_target = ComputeTarget.create(ws, compute_name, provisioning_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
     # For a more detailed view of current AmlCompute status, use get_status()
    print(compute_target.get_status().serialize())
```

Ahora puede usar el objeto `compute_target` como destino del proceso remoto.

Entre las restricciones en los nombres del clúster se incluyen:
+ La longitud debe ser inferior a 64 caracteres.
+ No puede contener ninguno de los caracteres siguientes: {`\` ~ ! @ # $ % ^ & * ( ) = + _ [ ] { } \\\\ | ; : \' \\" , < > / ?.`

## <a name="access-data-using-tabulardataset-function"></a>Acceso a los datos mediante la función TabularDataset

Se han definido training_data como [`TabularDataset`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.tabulardataset?view=azure-ml-py&preserve-view=true) y la etiqueta, las cuales se pasan a Machine Learning Automatizado en [`AutoMLConfig`](https://docs.microsoft.com/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig?view=azure-ml-py&preserve-view=true). El método `TabularDataset` `from_delimited_files` establece de forma predeterminada `infer_column_types` en true, lo cual hará que se infiera el tipo de columnas automáticamente. 

Si desea establecer manualmente los tipos de columna, puede establecer el argumento `set_column_types` para poder establecer manualmente el tipo de cada una de las columnas. En el siguiente ejemplo de código, los datos proceden del paquete sklearn.

```python
from sklearn import datasets
from azureml.core.dataset import Dataset
from scipy import sparse
import numpy as np
import pandas as pd
import os

# Create a project_folder if it doesn't exist
if not os.path.isdir('data'):
    os.mkdir('data')
    
if not os.path.exists('project_folder'):
    os.makedirs('project_folder')

X = pd.DataFrame(data_train.data[100:,:])
y = pd.DataFrame(data_train.target[100:])

# merge X and y
label = "digit"
X[label] = y

training_data = X

training_data.to_csv('data/digits.csv')
ds = ws.get_default_datastore()
ds.upload(src_dir='./data', target_path='digitsdata', overwrite=True, show_progress=True)

training_data = Dataset.Tabular.from_delimited_files(path=ds.path('digitsdata/digits.csv'))
```

## <a name="configure-experiment"></a>Configuración del experimento
Especifique la configuración de `AutoMLConfig`.  (Consulte una [lista completa de parámetros](how-to-configure-auto-train.md#configure-experiment) y sus posibles valores).

```python
from azureml.train.automl import AutoMLConfig
import time
import logging

automl_settings = {
    "name": "AutoML_Demo_Experiment_{0}".format(time.time()),
    "experiment_timeout_minutes" : 20,
    "enable_early_stopping" : True,
    "iteration_timeout_minutes": 10,
    "n_cross_validations": 5,
    "primary_metric": 'AUC_weighted',
    "max_concurrent_iterations": 10,
}

automl_config = AutoMLConfig(task='classification',
                             debug_log='automl_errors.log',
                             path=project_folder,
                             compute_target=compute_target,
                             training_data=training_data,
                             label_column_name=label,
                             **automl_settings,
                             )
```

## <a name="submit-training-experiment"></a>Envío del experimento de entrenamiento

Ahora, envíe la configuración para seleccionar automáticamente el algoritmo y los hiperparámetros, y entrenar el modelo.

```python
from azureml.core.experiment import Experiment
experiment = Experiment(ws, 'automl_remote')
remote_run = experiment.submit(automl_config, show_output=True)
```

Verá un resultado similar al del siguiente ejemplo:

```output
Running on remote compute: mydsvmParent Run ID: AutoML_015ffe76-c331-406d-9bfd-0fd42d8ab7f6
***********************************************************************************************
ITERATION: The iteration being evaluated.
PIPELINE:  A summary description of the pipeline being evaluated.
DURATION: Time taken for the current iteration.
METRIC: The result of computing score on the fitted pipeline.
BEST: The best observed score thus far.
***********************************************************************************************

 ITERATION     PIPELINE                               DURATION                METRIC      BEST
         2      Standardize SGD classifier            0:02:36                  0.954     0.954
         7      Normalizer DT                         0:02:22                  0.161     0.954
         0      Scale MaxAbs 1 extra trees            0:02:45                  0.936     0.954
         4      Robust Scaler SGD classifier          0:02:24                  0.867     0.954
         1      Normalizer kNN                        0:02:44                  0.984     0.984
         9      Normalizer extra trees                0:03:15                  0.834     0.984
         5      Robust Scaler DT                      0:02:18                  0.736     0.984
         8      Standardize kNN                       0:02:05                  0.981     0.984
         6      Standardize SVM                       0:02:18                  0.984     0.984
        10      Scale MaxAbs 1 DT                     0:02:18                  0.077     0.984
        11      Standardize SGD classifier            0:02:24                  0.863     0.984
         3      Standardize gradient boosting         0:03:03                  0.971     0.984
        12      Robust Scaler logistic regression     0:02:32                  0.955     0.984
        14      Scale MaxAbs 1 SVM                    0:02:15                  0.989     0.989
        13      Scale MaxAbs 1 gradient boosting      0:02:15                  0.971     0.989
        15      Robust Scaler kNN                     0:02:28                  0.904     0.989
        17      Standardize kNN                       0:02:22                  0.974     0.989
        16      Scale 0/1 gradient boosting           0:02:18                  0.968     0.989
        18      Scale 0/1 extra trees                 0:02:18                  0.828     0.989
        19      Robust Scaler kNN                     0:02:32                  0.983     0.989
```

## <a name="explore-results"></a>Exploración de los resultados

Puede usar el mismo [widget de Jupyter](https://docs.microsoft.com/python/api/azureml-widgets/azureml.widgets?view=azure-ml-py&preserve-view=true) que se muestra en el [tutorial de entrenamiento](tutorial-auto-train-models.md#explore-the-results) para ver un gráfico y la tabla de resultados.

```python
from azureml.widgets import RunDetails
RunDetails(remote_run).show()
```

Esta es una imagen estática del widget.  En el cuaderno, puede hacer clic en cualquier línea de la tabla para ver las propiedades de ejecución y los registros de salida para los que se ejecutan.   También puede usar la lista desplegable encima del grafo para ver un grafo de cada métrica disponible para cada iteración.

![tabla de widget](./media/how-to-auto-train-remote/table.png)
![trazado de widget](./media/how-to-auto-train-remote/plot.png)

El widget muestra una dirección URL que puede usar para ver y explorar los detalles de ejecución individuales.  

Si no está en un cuaderno de Jupyter, puede mostrar la URL desde la propia ejecución:

```
remote_run.get_portal_url()
```

La misma información está disponible en el área de trabajo.  Para obtener más información sobre estos resultados, consulte [Descripción de los resultados del aprendizaje automático automatizado](how-to-understand-automated-ml.md).

## <a name="example"></a>Ejemplo

En el siguiente [cuaderno](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/regression/auto-ml-regression.ipynb) se demuestran los conceptos de este artículo.

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de [cómo configurar opciones para el aprendizaje automático](how-to-configure-auto-train.md).
* Consulte el [procedimiento](how-to-machine-learning-interpretability-automl.md) sobre cómo habilitar las características de interpretación de modelos en experimentos de ML automatizados.
