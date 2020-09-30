---
title: Entrenamiento de un modelo con una imagen personalizada de Docker
titleSuffix: Azure Machine Learning
description: Aprenda a entrenar modelos con imágenes personalizadas de Docker en Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sagopal
author: saachigopal
ms.date: 08/11/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: d90b56366cb22e80162983c982e861de608e4e9e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90893121"
---
# <a name="train-a-model-using-a-custom-docker-image"></a>Entrenamiento de un modelo con una imagen personalizada de Docker

En este artículo, aprenderá a usar una imagen personalizada de Docker al entrenar modelos con Azure Machine Learning. 

Los scripts de ejemplo de este artículo se usan para clasificar las imágenes de mascotas mediante la creación de una red neuronal convolucional. 

Aunque Azure Machine Learning proporciona una imagen base de Docker predeterminada, también puede usar entornos de Azure Machine Learning para especificar una imagen base concreta, como uno de los conjuntos de [imágenes base de Azure ML](https://github.com/Azure/AzureML-Containers) o su propia [imagen personalizada](how-to-deploy-custom-docker-image.md#create-a-custom-base-image). Las imágenes base personalizadas permiten administrar estrechamente las dependencias y mantener un mayor control sobre las versiones de los componentes al ejecutar trabajos de entrenamiento. 

## <a name="prerequisites"></a>Requisitos previos 
Ejecute este código en cualquiera de estos entornos:
* Instancia de Proceso de Azure Machine Learning: no se necesitan descargas ni instalación
    * Complete el [Tutorial: Configuración del entorno y el área de trabajo](tutorial-1st-experiment-sdk-setup.md) para crear un servidor de cuadernos dedicado en el que se habrán cargado previamente el SDK y el repositorio de ejemplos.
    * En el [repositorio de ejemplos](https://github.com/Azure/azureml-examples) de Azure Machine Learning, navegue a este directorio **notebooks > fastai > train-pets-resnet34.ipynb** para buscar un cuaderno completo. 

* Su propio servidor de Jupyter Notebook
    * Cree un [archivo de configuración del área de trabajo](how-to-configure-environment.md#workspace).
    * El [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py&preserve-view=true). 
    * Una instancia de [Azure Container Registry](/azure/container-registry) u otro registro de Docker accesible en Internet.

## <a name="set-up-the-experiment"></a>Configuración del experimento 
En esta sección, para configurar el experimento de entrenamiento, se inicializa un área de trabajo, se crea un experimento y se cargan los datos y scripts de entrenamiento.

### <a name="initialize-a-workspace"></a>Inicialización de un área de trabajo
El [área de trabajo de Azure Machine Learning](concept-workspace.md) es el recurso de nivel superior para el servicio. Proporciona un lugar centralizado para trabajar con todos los artefactos que cree. En el SDK de Python, puede acceder a los artefactos del área de trabajo mediante la creación de un objeto [`workspace`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py&preserve-view=true).

Cree un objeto de área de trabajo a partir del archivo `config.json` creado en la [sección de requisitos previos](#prerequisites).

```Python
from azureml.core import Workspace

ws = Workspace.from_config()
```

### <a name="prepare-scripts"></a>Preparación de los scripts
En este tutorial, el script de entrenamiento **train.py** se proporciona [aquí](https://github.com/Azure/azureml-examples/blob/main/code/models/fastai/pets-resnet34/train.py). En la práctica, puede tomar cualquier script de entrenamiento personalizado tal cual y ejecutarlo con Azure Machine Learning.

### <a name="define-your-environment"></a>Definición del entorno
Cree un objeto de entorno y habilite Docker. 

```python
from azureml.core import Environment

fastai_env = Environment("fastai2")
fastai_env.docker.enabled = True
```

Esta imagen base especificada es compatible con la biblioteca fast.ai, que permite capacidades de aprendizaje profundo distribuido. Para más información, consulte [fast.ai DockerHub](https://hub.docker.com/u/fastdotai). 

Al usar la imagen personalizada de Docker, es posible que ya tenga el entorno de Python configurado correctamente. En ese caso, establezca la marca de `user_managed_dependencies` en True para poder aprovechar el entorno de Python integrado de la imagen personalizada. De forma predeterminada, Azure ML creará un entorno de Conda con las dependencias especificadas y realizará la ejecución en ese entorno en lugar de usar las bibliotecas de Python que haya instalado en la imagen base.

```python
fastai_env.docker.base_image = "fastdotai/fastai2:latest"
fastai_env.python.user_managed_dependencies = True
```

Para usar una imagen desde un registro de contenedor privado que no esté en el área de trabajo, debe usar `docker.base_image_registry` para especificar la dirección del repositorio y un nombre de usuario y contraseña:

```python
# Set the container registry information
fastai_env.docker.base_image_registry.address = "myregistry.azurecr.io"
fastai_env.docker.base_image_registry.username = "username"
fastai_env.docker.base_image_registry.password = "password"
```

También es posible usar un Dockerfile personalizado. Use este enfoque si necesita instalar paquetes que no son de Python como dependencias y recuerde establecer la imagen base en Ninguna.

```python 
# Specify docker steps as a string. 
dockerfile = r"""
FROM mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04
RUN echo "Hello from custom container!"
"""

# Set base image to None, because the image is defined by dockerfile.
fastai_env.docker.base_image = None
fastai_env.docker.base_dockerfile = dockerfile

# Alternatively, load the string from a file.
fastai_env.docker.base_image = None
fastai_env.docker.base_dockerfile = "./Dockerfile"
```

### <a name="create-or-attach-existing-amlcompute"></a>Creación o asociación de una instancia de AMlCompute existente
Para entrenar el modelo, necesitará crear un [destino de proceso](concept-azure-machine-learning-architecture.md#compute-targets). En este tutorial, creará una instancia de AmlCompute como recurso de proceso de aprendizaje.

La creación de AmlCompute tarda aproximadamente cinco minutos. Si la instancia de AmlCompute con ese nombre ya está en el área de trabajo, este código omitirá el proceso de creación.

Al igual que en otros servicios de Azure, existen límites en determinados recursos (como AmlCompute) asociados con Azure Machine Learning Service. Lea [este artículo](how-to-manage-quotas.md) sobre los límites predeterminados y cómo solicitar una cuota mayor. 

```python
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException

# choose a name for your cluster
cluster_name = "gpu-cluster"

try:
    compute_target = ComputeTarget(workspace=ws, name=cluster_name)
    print('Found existing compute target.')
except ComputeTargetException:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_NC6',
                                                           max_nodes=4)

    # create the cluster
    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

    compute_target.wait_for_completion(show_output=True)

# use get_status() to get a detailed status for the current AmlCompute
print(compute_target.get_status().serialize())
```

### <a name="create-a-scriptrunconfig"></a>Creación de ScriptRunConfig
Este ScriptRunConfig configurará el trabajo para su ejecución en el [destino de proceso](how-to-set-up-training-targets.md) local.

```python
from azureml.core import ScriptRunConfig

fastai_config = ScriptRunConfig(source_directory='fastai-example', script='train.py')
fastai_config.run_config.environment = fastai_env
fastai_config.run_config.target = compute_target
```

### <a name="submit-your-run"></a>Envío de la ejecución
Cuando se envía una ejecución de entrenamiento mediante un objeto ScriptRunConfig, el método de envío devuelve un objeto de tipo ScriptRun. El objeto ScriptRun devuelto proporciona acceso mediante programación a información sobre la ejecución de entrenamiento. 

```python
from azureml.core import Experiment

run = Experiment(ws,'fastai-custom-image').submit(fastai_config)
run.wait_for_completion(show_output=True)
```

> [!WARNING]
> Azure Machine Learning ejecuta scripts de entrenamiento mediante la copia de todo el directorio de origen. Si tiene información confidencial que no quiere cargar, use un [archivo .ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) o no lo incluya en el directorio de origen. En su lugar, acceda a los datos mediante un [almacén de datos](https://docs.microsoft.com/python/api/azureml-core/azureml.data?view=azure-ml-py&preserve-view=true).

Para obtener más información acerca de cómo personalizar el entorno de Python, consulte [Creación y uso de entornos de software](how-to-use-environments.md). 

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha entrenado un modelo mediante una imagen de Docker personalizada. Consulte estos otros artículos para más información sobre Azure Machine Learning.
* [Seguimiento de las métricas de ejecución](how-to-track-experiments.md) durante el entrenamiento.
* [Implementación de un modelo](how-to-deploy-custom-docker-image.md) con una imagen personalizada de Docker.
