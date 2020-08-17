---
title: Entrenamiento de modelos de Machine Learning con scikit-learn
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo ejecutar los scripts de entrenamiento de scikit-learn en Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: jordane
author: jpe316
ms.date: 07/24/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 4221ed6a927d0c589407dc38b5371ad8a65d2174
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88054398"
---
# <a name="build-scikit-learn-models-at-scale-with-azure-machine-learning"></a>Creación de modelos de Scikit-learn a escala con Azure Machine Learning
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo, obtendrá información sobre cómo ejecutar los scripts de entrenamiento de scikit-learn con Azure Machine Learning.

Los scripts de ejemplo de este artículo se usan para clasifica imágenes de flores Iris para crear un modelo de Machine Learning basado en el [conjunto de datos de Iris](https://archive.ics.uci.edu/ml/datasets/iris) de scikit-learn.

Tanto si va a entrenar un modelo de scikit-learn de Machine Learning desde el principio como si va a incorporar un modelo existente a la nube, puede usar Azure Machine Learning para escalar horizontalmente trabajos de entrenamiento de código abierto mediante recursos de proceso en la nube elástica. Puede compilar, implementar y supervisar modelos de nivel de producción, así como crear versiones de dichos mismos, mediante Azure Machine Learning.

## <a name="prerequisites"></a>Requisitos previos

Ejecute este código en cualquiera de estos entornos:
 - Instancia de Proceso de Azure Machine Learning: no se necesitan descargas ni instalación

    - Complete el [Tutorial: Configuración del entorno y el área de trabajo](tutorial-1st-experiment-sdk-setup.md) para crear un servidor de cuadernos dedicado en el que se habrán cargado previamente el SDK y el repositorio de ejemplos.
    - Para encontrar un cuaderno completado y expandido, en la carpeta de entrenamiento de ejemplos en el servidor de cuadernos, vaya a este directorio: carpeta **how-to-use-azureml > ml-frameworks > scikit-learn > training > train-hyperparameter-tune-deploy-with-sklearn**.

 - Su propio servidor de Jupyter Notebook

    - [Instalación del SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py).
    - [Cree un archivo de configuración del área de trabajo](how-to-configure-environment.md#workspace).

## <a name="set-up-the-experiment"></a>Configuración del experimento

En esta sección, para configurar el experimento de entrenamiento, se cargan los paquetes de Python necesarios, se inicializa un área de trabajo, se crea un experimento y se cargan los datos y scripts de entrenamiento.

### <a name="initialize-a-workspace"></a>Inicialización de un área de trabajo

El [área de trabajo de Azure Machine Learning](concept-workspace.md) es el recurso de nivel superior para el servicio. Proporciona un lugar centralizado para trabajar con todos los artefactos que cree. En el SDK de Python, puede acceder a los artefactos del área de trabajo mediante la creación de un objeto [`workspace`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py).

Cree un objeto de área de trabajo a partir del archivo `config.json` creado en la [sección de requisitos previos](#prerequisites).

```Python
from azureml.core import Workspace

ws = Workspace.from_config()
```


### <a name="prepare-scripts"></a>Preparación de los scripts

En este tutorial ya se proporciona el script de entrenamiento **train_iris.py** [aquí](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/scikit-learn/training/train-hyperparameter-tune-deploy-with-sklearn/train_iris.py). En la práctica, debería poder usar cualquier script de entrenamiento personalizado tal cual y ejecutarlo con Azure Machine Learning sin tener que modificar el código.

Notas:
- El script de entrenamiento proporcionado muestra cómo registrar algunas métricas para la ejecución de Azure Machine Learning mediante el objeto `Run` dentro del script.
- El script de entrenamiento proporcionado usa datos de ejemplo de la función `iris = datasets.load_iris()`.  Para sus propios datos, puede que necesite realizar pasos como [cargar el conjunto de datos y los scripts](how-to-train-keras.md#data-upload) para que los datos estén disponibles durante el entrenamiento.

### <a name="define-your-environment"></a>Definición del entorno

#### <a name="create-a-custom-environment"></a>Creación de un entorno personalizado

Cree el entorno de Conda (sklearn-env.yml).
Para escribir el entorno de Conda desde un cuaderno, puede agregar la línea ```%%writefile sklearn-env.yml``` en la parte superior de la celda.

```yaml
name: sklearn-training-env
dependencies:
  - python=3.6.2
  - scikit-learn
  - numpy
  - pip:
    - azureml-defaults
```

Cree un entorno de Azure Machine Learning a partir de esta especificación de entorno de Conda. El entorno se empaquetará en un contenedor de Docker en tiempo de ejecución.
```python
from azureml.core import Environment

myenv = Environment.from_conda_specification(name = "myenv", file_path = "sklearn-env.yml")
myenv.docker.enabled = True
```

#### <a name="use-a-curated-environment"></a>Uso de un entorno mantenido
Azure Machine Learning proporciona entornos de contenedor previamente creados y mantenidos si no desea crear su propia imagen. Para más información, consulte [aquí](resource-curated-environments.md).
Si desea usar un entorno mantenido, puede ejecutar el siguiente comando en su lugar:

```python
env = Environment.get(workspace=ws, name="AzureML-Tutorial")
```

### <a name="create-a-scriptrunconfig"></a>Creación de ScriptRunConfig

Este ScriptRunConfig enviará el trabajo para su ejecución en el destino de proceso local.

```python
from azureml.core import ScriptRunConfig

sklearnconfig = ScriptRunConfig(source_directory='.', script='train_iris.py')
sklearnconfig.run_config.environment = myenv
```

Si desea realizar el envío en un clúster remoto, puede cambiar run_config.target al destino de proceso deseado.

### <a name="submit-your-run"></a>Envío de la ejecución
```python
from azureml.core import Experiment

run = Experiment(ws,'train-sklearn').submit(config=sklearnconfig)
run.wait_for_completion(show_output=True)

```

> [!WARNING]
> Azure Machine Learning ejecuta scripts de entrenamiento mediante la copia de todo el directorio de origen. Si tiene información confidencial que no quiere cargar, use un [archivo .ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) o no lo incluya en el directorio de origen. En su lugar, acceda a los datos mediante un [almacén de datos](https://docs.microsoft.com/python/api/azureml-core/azureml.data?view=azure-ml-py).

Para más información sobre cómo personalizar el entorno de Python, vea el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md). 

## <a name="what-happens-during-run-execution"></a>¿Qué ocurre mientras se lleva a cabo la ejecución?
Durante la ejecución del objeto, pasa por las fases siguientes:

- **Preparando**: se crea una imagen de Docker según el estimador de TensorFlow. La imagen se carga en el registro de contenedor del área de trabajo y se almacena en memoria caché para ejecuciones posteriores. Los registros también se transmiten al historial de ejecución y se pueden consultar para supervisar el progreso.

- **Escalado**: el clúster intenta escalar verticalmente si el clúster de Batch AI requiere más nodos para realizar la ejecución de los que se encuentran disponibles actualmente.

- **En ejecución**: todos los scripts de la carpeta de scripts se cargan en el destino de proceso, se montan o se copian los almacenes de datos y se ejecuta entry_script. Las salidas de stdout y la carpeta ./logs se transmiten al historial de ejecución y se pueden usar para supervisar la ejecución.

- **Posprocesamiento**: la carpeta ./outputs de la ejecución se copia en el historial de ejecución.

## <a name="save-and-register-the-model"></a>Guardado y registro del modelo

Una vez que haya entrenado el modelo, puede guardarlo y registrarlo en el área de trabajo. El registro del modelo permite almacenar los modelos y crear versiones de ellos en el área de trabajo para simplificar la [administración e implementación de modelos](concept-model-management-and-deployment.md).

Para guardar el modelo, agregue el código siguiente al script de entrenamiento, train_iris.py. 

``` Python
import joblib

joblib.dump(svm_model_linear, 'model.joblib')
```

Registre el modelo al área de trabajo con el código siguiente. Al especificar los parámetros `model_framework`, `model_framework_version` y `resource_configuration`, la implementación del modelo sin código está disponible. La implementación de un modelo sin código permite implementar directamente el modelo como un servicio web desde el modelo registrado, y el objeto [`ResourceConfiguration`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.resource_configuration.resourceconfiguration?view=azure-ml-py) define el recurso de proceso del servicio web.

```Python
from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

model = run.register_model(model_name='sklearn-iris', 
                           model_path='outputs/model.joblib',
                           model_framework=Model.Framework.SCIKITLEARN,
                           model_framework_version='0.19.1',
                           resource_configuration=ResourceConfiguration(cpu=1, memory_in_gb=0.5))
```

## <a name="deployment"></a>Implementación

El modelo que acaba de registrar se puede implementar exactamente de la misma manera que cualquier otro modelo registrado en Azure Machine Learning, independientemente del estimador que haya usado para el entrenamiento. El procedimiento de implementación contiene una sección sobre el registro de modelos, pero puede ir directamente a la [creación de un destino de proceso](how-to-deploy-and-where.md#choose-a-compute-target) para la implementación, dado que ya tiene un modelo registrado.

### <a name="preview-no-code-model-deployment"></a>(Versión preliminar) Implementación de modelo sin código

En lugar de la ruta de implementación tradicional, también puede usar la característica de implementación sin código (versión preliminar) para Scikit-learn. Se admite la implementación de modelo sin código para todos los tipos de modelos integrados de Scikit-learn. Mediante el registro del modelo como se ha indicado anteriormente con los parámetros `model_framework`, `model_framework_version` y `resource_configuration`, solo tiene que usar la función estática [`deploy()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model%28class%29?view=azure-ml-py#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) para implementar el modelo.

```python
web_service = Model.deploy(ws, "scikit-learn-service", [model])
```

NOTA:  Estas dependencias se incluyen en el contenedor de inferencia Scikit-learn integrado previamente.

```yaml
    - azureml-defaults
    - inference-schema[numpy-support]
    - scikit-learn
    - numpy
```

El [procedimiento](how-to-deploy-and-where.md) completo trata la implementación en Azure Machine Learning más detalladamente.


## <a name="next-steps"></a>Pasos siguientes

En este artículo ha entrenado y registrado un modelo de Scikit-learn y ha aprendido sobre las opciones de implementación. Consulte estos otros artículos para más información sobre Azure Machine Learning.

* [Seguir métricas de ejecución durante el entrenamiento](how-to-track-experiments.md)
* [Ajustar los hiperparámetros](how-to-tune-hyperparameters.md)
* [Arquitectura de referencia para el entrenamiento del aprendizaje profundo distribuido en Azure](/azure/architecture/reference-architectures/ai/training-deep-learning)
