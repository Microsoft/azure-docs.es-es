---
title: Envío de una ejecución de entrenamiento a un destino de proceso
titleSuffix: Azure Machine Learning
description: Configure el modelo de Machine Learning en varios entornos de entrenamiento (destinos de proceso). Es fácil cambiar entre entornos de entrenamiento. Inicie el entrenamiento de forma local. Si necesita escalar horizontalmente, cambie a un destino de proceso basado en la nube.
services: machine-learning
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 08/28/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, contperfq1
ms.openlocfilehash: 8b07d19ca88a2d680a4f9efbb85fcf60b895a2b3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907603"
---
# <a name="submit-a-training-run-to-a-compute-target"></a>Envío de una ejecución de entrenamiento a un destino de proceso

En este artículo obtendrá información acerca de cómo usar varios entornos de entrenamiento ([destinos de proceso](concept-compute-target.md)) para entrenar los modelos de Machine Learning.

Cuando se entrena, es normal comenzar en el equipo local y después ejecutar ese script de entrenamiento en un destino de proceso diferente. Con Azure Machine Learning, puede ejecutar el script en varios destinos de proceso sin tener que cambiar el script de entrenamiento.

Todo lo que debe hacer es definir el entorno de cada destino de proceso con una **configuración de ejecución de script**.  Después, cuando desee ejecutar el experimento de entrenamiento en un destino de proceso diferente, especifique la configuración de ejecución para ese proceso.

## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
* El [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py&preserve-view=true)
* Un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md), `ws`
* Un destino de proceso, `my_compute_target`.  Cree un destino de proceso con:
  * [SDK de Python](how-to-create-attach-compute-sdk.md) 
  * [Azure Machine Learning Studio](how-to-create-attach-compute-studio.md)

## <a name="whats-a-script-run-configuration"></a><a name="whats-a-run-configuration"></a>¿En qué consiste una configuración de ejecución de script?

Envíe el experimento de entrenamiento mediante un objeto [ScriptRunConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.scriptrunconfig?view=azure-ml-py&preserve-view=true).  Este objeto incluye:

* **source_directory**: el directorio de origen que contiene el script de entrenamiento.
* **script**: Identificación del script de entrenamiento
* **run_config**: la [configuración de ejecución](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfiguration?view=azure-ml-py&preserve-view=true), que a su vez define la ubicación en que tendrá lugar el entrenamiento. En el elemento `run_config` se especifican el destino de proceso y el entorno que se usarán al ejecutar el script de entrenamiento.  

## <a name="whats-an-environment"></a>¿Qué es un entorno?

Los [entornos](concept-environments.md) de Azure Machine Learning son una encapsulación del entorno en el que se produce el entrenamiento del aprendizaje automático. Especifican los paquetes, las variables de entorno y la configuración de software de Python en torno a los scripts de entrenamiento y puntuación. También especifican los entornos de ejecución (Python, Spark o Docker).  

Los entornos se especifican en el objeto `run_config` dentro de un `ScriptRunConfig`.

## <a name="train-your-model"></a><a id="submit"></a>Entrenamiento de un modelo

El patrón de código para enviar una ejecución de entrenamiento es el mismo para todos los tipos de destinos de proceso:

1. Creación de un experimento para su ejecución
1. Cree el entorno en el que se ejecutará el script
1. Cree una configuración de ejecución de script que haga referencia al entorno y al destino de proceso
1. Envío de la ejecución
1. Espere a que la ejecución se complete

También puede:

* Enviar el experimento con un objeto `Estimator`, tal como se muestra en [Entrenamiento de modelos de aprendizaje automático con estimadores](how-to-train-ml-models.md).
* Envíe una ejecución de HyperDrive para el [ajuste de hiperparámetros](how-to-tune-hyperparameters.md).
* Enviar un experimento mediante la [extensión de VS Code](tutorial-train-deploy-image-classification-model-vscode.md#train-the-model).

## <a name="create-an-experiment"></a>Creación de un experimento

Cree un experimento en el área de trabajo.

```python
from azureml.core import Experiment

experiment_name = 'my_experiment'

experiment = Experiment(workspace=ws, name=experiment_name)
```

## <a name="create-an-environment"></a>Creación de un entorno

Los entornos seleccionados contienen colecciones de paquetes de Python y están disponibles en el área de trabajo de forma predeterminada. Estos entornos están respaldados por imágenes de Docker en caché, lo que reduce el costo de preparación de la ejecución. En el caso de un destino de proceso remoto, puede usar uno de los siguientes entornos seleccionados populares para comenzar:

```python
from azureml.core import Workspace, Environment

ws = Workspace.from_config()
my_environment = Environment.get(workspace=ws, name="AzureML-Minimal")
```

Para obtener más información y detalles sobre los entornos, consulte [Creación y uso de entornos de software en Azure Machine Learning](how-to-use-environments.md).
  
### <a name="local-compute-target"></a>Destino de proceso local

Si el destino de proceso es su **máquina local**, usted es responsable de garantizar que todos los paquetes necesarios estén disponibles en el entorno de Python en el que se ejecutará el script.  Use `python.user_managed_dependencies` para usar el entorno de Python actual (o el de Python en la ruta de acceso que especifique).

```python
from azureml.core import Environment

# Editing a run configuration property on-fly.
my_environment = Environment("user-managed-env")

my_environment.python.user_managed_dependencies = True

# You can choose a specific Python environment by pointing to a Python path 
#my_environment.python.interpreter_path = '/home/johndoe/miniconda3/envs/myenv/bin/python'
```

## <a name="create-script-run-configuration"></a>Creación de una configuración de ejecución de script

Ahora que tiene un destino de proceso (`compute_target`) y un entorno (`my_environment`), cree una configuración de ejecución de script que ejecute el script de entrenamiento (`train.py`) ubicado en el directorio `project_folder`:

```python
from azureml.core import ScriptRunConfig

script_run_config = ScriptRunConfig(source_directory=project_folder, script='train.py')

# Set compute target
script_run_config.run_config.target = my_compute_target

# Set environment.   If you don't do this, a default environment will be created.
script_run_config.run_config.environment = my_environment
```

También debería configurar el marco para la ejecución.

* En el caso de un clúster de HDI:
    ```python
    src.run_config.framework = "pyspark"
    ```

* En el caso de una máquina virtual remota:
    ```python
    src.run_config.framework = "python"
    ```

## <a name="submit-the-experiment"></a>Envío del experimento

```python
run = experiment.submit(config=script_run_config)
```

> [!IMPORTANT]
> Cuando envía la ejecución de entrenamiento, se crea una instantánea del directorio que contiene los scripts de entrenamiento y se envía al destino de proceso. También se almacena como parte del experimento del área de trabajo. Si cambia los archivos y vuelve a enviar la ejecución, solo se cargarán los archivos cambiados.
>
> [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]
> 
> Para más información sobre las instantáneas, consulte [Instantáneas](concept-azure-machine-learning-architecture.md#snapshots).


<a id="gitintegration"></a>

## <a name="git-tracking-and-integration"></a>Integración y seguimiento de Git

Cuando se inicia una ejecución de entrenamiento en la que el directorio de origen es un repositorio de GIT local, se almacena información sobre el repositorio en el historial de ejecución. Para más información, consulte [Integración de Git con Azure Machine Learning](concept-train-model-git-integration.md).

## <a name="notebook-examples"></a>Ejemplos de cuadernos

Consulte estos cuadernos para ver ejemplos de entrenamiento con varios destinos de proceso:
* [how-to-use-azureml/training](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training)
* [tutorials/img-classification-part1-training.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/image-classification-mnist-data/img-classification-part1-training.ipynb)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Entrenamiento de un modelo](tutorial-train-models-with-aml.md) utiliza un destino de proceso administrado para entrenar un modelo.
* Obtenga información sobre cómo [ajustar los hiperparámetros eficazmente](how-to-tune-hyperparameters.md) para crear modelos mejores.?view=azure-ml-py&preserve-view=true).
* Cuando tenga un modelo entrenado, aprenda [cómo y dónde implementar los modelos](how-to-deploy-and-where.md).
* Consulte la referencia del SDK de la [clase RunConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfig.runconfiguration?view=azure-ml-py&preserve-view=true).
* [Uso de Azure Machine Learning con Azure Virtual Networks](how-to-enable-virtual-network.md)