---
title: Entrenamiento de modelos de PyTorch de aprendizaje profundo
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo ejecutar los scripts de entrenamiento de PyTorch a escala empresarial mediante la clase estimator de PyTorch de Azure Machine Learning.  Los scripts de ejemplo clasifican imágenes de pollos y pavos para crear una red neuronal de aprendizaje profundo basada en el tutorial de aprendizaje de transferencia de PyTorch.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: peterlu
author: peterclu
ms.reviewer: peterlu
ms.date: 08/01/2019
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 7f805a702e85b897f9807eb2294e4263a0ea2d8c
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876332"
---
# <a name="train-pytorch-deep-learning-models-at-scale-with-azure-machine-learning"></a>Entrenamiento de modelos de aprendizaje profundo de PyTorch a escala con Azure Machine Learning
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo aprenderá a ejecutar los scripts de entrenamiento de [PyTorch](https://pytorch.org/) a escala empresarial mediante la clase [estimator de PyTorch](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.pytorch?view=azure-ml-py) de Azure Machine Learning.  

Los scripts de ejemplo de este artículo clasifican imágenes de pollos y pavos para crear una red neuronal de aprendizaje profundo basada en el [tutorial](https://pytorch.org/tutorials/beginner/transfer_learning_tutorial.html) de aprendizaje de transferencia de PyTorch. 

Tanto si va a entrenar un modelo de PyTorch de aprendizaje profundo desde el principio como si va a incorporar un modelo existente a la nube, puede usar Azure Machine Learning para escalar horizontalmente trabajos de entrenamiento de código abierto mediante recursos de proceso en la nube elástica. Puede compilar, implementar y supervisar modelos de nivel de producción, así como crear versiones de dichos mismos, mediante Azure Machine Learning. 

Más información sobre las diferencias entre [aprendizaje profundo y aprendizaje automático](concept-deep-learning-vs-machine-learning.md).

## <a name="prerequisites"></a>Requisitos previos

Ejecute este código en cualquiera de estos entornos:

- Instancia de Proceso de Azure Machine Learning: no se necesitan descargas ni instalación

    - Complete el [Tutorial: Configuración del entorno y el área de trabajo](tutorial-1st-experiment-sdk-setup.md) para crear un servidor de cuadernos dedicado en el que se habrán cargado previamente el SDK y el repositorio de ejemplos.
    - En la carpeta de aprendizaje profundo de ejemplos en el servidor de cuadernos, vaya a este directorio: carpeta **how-to-use-azureml > training-with-deep-learning > train-hyperparameter-tune-deploy-with-pytorch**, para encontrar un cuaderno completado y expandido. 
 
 - Su propio servidor de Jupyter Notebook

    - [Instalación del SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py).
    - [Cree un archivo de configuración del área de trabajo](how-to-configure-environment.md#workspace).
    - [Descarga de los archivos de script de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/pytorch/deployment/train-hyperparameter-tune-deploy-with-pytorch) `pytorch_train.py`
     
    También puede encontrar una [versión de Jupyter Notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/pytorch/deployment/train-hyperparameter-tune-deploy-with-pytorch/train-hyperparameter-tune-deploy-with-pytorch.ipynb) completada de esta guía en la página de ejemplos de GitHub. El cuaderno incluye secciones expandidas que abarcan el ajuste de hiperparámetros inteligente, la implementación de modelos y los widgets del cuaderno.

## <a name="set-up-the-experiment"></a>Configuración del experimento

En esta sección, para configurar el experimento de entrenamiento, se cargan los paquetes de Python necesarios, se inicializa un área de trabajo, se crea un experimento y se cargan los datos y scripts de entrenamiento.

### <a name="import-packages"></a>Importación de paquetes

En primer lugar, importe las bibliotecas de Python necesarias.

```Python
import os
import shutil

from azureml.core.workspace import Workspace
from azureml.core import Experiment

from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException
from azureml.train.dnn import PyTorch
```

### <a name="initialize-a-workspace"></a>Inicialización de un área de trabajo

El [área de trabajo de Azure Machine Learning](concept-workspace.md) es el recurso de nivel superior para el servicio. Proporciona un lugar centralizado para trabajar con todos los artefactos que cree. En el SDK de Python, puede acceder a los artefactos del área de trabajo mediante la creación de un objeto [`workspace`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py).

Cree un objeto de área de trabajo a partir del archivo `config.json` creado en la [sección de requisitos previos](#prerequisites).

```Python
ws = Workspace.from_config()
```

### <a name="create-a-deep-learning-experiment"></a>Creación de un experimento de aprendizaje profundo

Cree un experimento y una carpeta para almacenar los scripts de entrenamiento. En este ejemplo, cree un experimento denominado "pytorch-birds".

```Python
project_folder = './pytorch-birds'
os.makedirs(project_folder, exist_ok=True)

experiment_name = 'pytorch-birds'
experiment = Experiment(ws, name=experiment_name)
```

### <a name="get-the-data"></a>Obtener los datos

El conjunto de datos consta de aproximadamente 120 imágenes de entrenamiento de pavos y pollos, con 100 imágenes de validación para cada clase. Descargaremos y extraeremos el conjunto de datos como parte de nuestro script de entrenamiento `pytorch_train.py`. Las imágenes son un subconjunto del [conjunto de datos Open Images v5](https://storage.googleapis.com/openimages/web/index.html).

### <a name="prepare-training-scripts"></a>Preparación de los scripts de entrenamiento

En este tutorial, el script de entrenamiento, `pytorch_train.py`, ya se proporciona. En la práctica, puede tomar cualquier script de entrenamiento personalizado tal cual y ejecutarlo con Azure Machine Learning.

Cargue el script de entrenamiento de Pytorch, `pytorch_train.py`.

```Python
shutil.copy('pytorch_train.py', project_folder)
```

No obstante, si quiere usar las funcionalidades de seguimiento y las métricas de Azure Machine Learning, debe agregar una pequeña cantidad de código dentro del script de entrenamiento. En `pytorch_train.py` encontrará ejemplos de seguimiento de métricas.

## <a name="create-a-compute-target"></a>Creación de un destino de proceso

Cree un destino de proceso en el que ejecutar el trabajo de PyTorch. En este ejemplo, cree un clúster de proceso de Azure Machine Learning habilitado para GPU.

```Python
cluster_name = "gpucluster"

try:
    compute_target = ComputeTarget(workspace=ws, name=cluster_name)
    print('Found existing compute target')
except ComputeTargetException:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_NC6', 
                                                           max_nodes=4)

    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

Para más información sobre los destinos de proceso, vea el artículo [¿Qué es un destino de proceso?](concept-compute-target.md)

## <a name="create-a-pytorch-estimator"></a>Crear un estimador de PyTorch

El [estimador de PyTorch](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.pytorch?view=azure-ml-py) proporciona una manera sencilla de iniciar un trabajo de entrenamiento de PyTorch en un destino de proceso.

El estimador de PyTorch se implementa a través de la clase genérica [`estimator`](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator.estimator?view=azure-ml-py), que se puede usar para admitir cualquier marco de trabajo. Para obtener más información sobre cómo entrenar modelos con el estimador genérico, consulte [Entrenamiento de modelos con Azure Machine Learning mediante un objeto Estimator](how-to-train-ml-models.md).

Si el script de entrenamiento necesita paquetes adicionales pip o conda para ejecutarse, puede hacer que los paquetes se instalen en la imagen de Docker resultante. Para ello, pase sus nombres mediante los argumentos `pip_packages` y `conda_packages`.

```Python
script_params = {
    '--num_epochs': 30,
    '--output_dir': './outputs'
}

estimator = PyTorch(source_directory=project_folder, 
                    script_params=script_params,
                    compute_target=compute_target,
                    entry_script='pytorch_train.py',
                    use_gpu=True,
                    pip_packages=['pillow==5.4.1'])
```

> [!WARNING]
> Azure Machine Learning ejecuta scripts de entrenamiento mediante la copia de todo el directorio de origen. Si tiene información confidencial que no quiere cargar, use un [archivo .ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) o no lo incluya en el directorio de origen. En su lugar, acceda a los datos mediante un [almacén de datos](https://docs.microsoft.com/python/api/azureml-core/azureml.data?view=azure-ml-py).

Para más información sobre cómo personalizar el entorno de Python, vea el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md).

## <a name="submit-a-run"></a>Envío de una ejecución

El [objeto Run](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run%28class%29?view=azure-ml-py) proporciona la interfaz para el historial de ejecución mientras se ejecuta el trabajo y cuando se ha completado.

```Python
run = experiment.submit(estimator)
run.wait_for_completion(show_output=True)
```

Durante la ejecución de Run, pasa por las fases siguientes:

- **Preparando**: se crea una imagen de Docker según el estimador de PyTorch. La imagen se carga en el registro de contenedor del área de trabajo y se almacena en memoria caché para ejecuciones posteriores. Los registros también se transmiten al historial de ejecución y se pueden consultar para supervisar el progreso.

- **Escalado**: el clúster intenta escalar verticalmente si el clúster de Batch AI requiere más nodos para realizar la ejecución de los que se encuentran disponibles actualmente.

- **En ejecución**: todos los scripts de la carpeta de scripts se cargan en el destino de proceso, se montan o se copian los almacenes de datos y se ejecuta entry_script. Las salidas de stdout y la carpeta ./logs se transmiten al historial de ejecución y se pueden usar para supervisar la ejecución.

- **Posprocesamiento**: la carpeta ./outputs de la ejecución se copia en el historial de ejecución.

## <a name="register-or-download-a-model"></a>Registro o descarga de un modelo

Una vez que haya entrenado el modelo, puede registrarlo en el área de trabajo. El registro del modelo permite almacenar los modelos y crear versiones de ellos en el área de trabajo para simplificar la [administración e implementación de modelos](concept-model-management-and-deployment.md).

```Python
model = run.register_model(model_name='pt-dnn', model_path='outputs/')
```

> [!TIP]
> El modelo que acaba de registrar se implementa exactamente de la misma manera que cualquier otro modelo registrado en Azure Machine Learning, independientemente del estimador que haya usado para el entrenamiento. El procedimiento de implementación contiene una sección sobre el registro de modelos, pero puede ir directamente a la [creación de un destino de proceso](how-to-deploy-and-where.md#choose-a-compute-target) para la implementación, dado que ya tiene un modelo registrado.

También puede descargar una copia local del modelo mediante el uso del objeto Run. En el script de entrenamiento, `pytorch_train.py`, un objeto de protector de PyTorch guarda el modelo en una carpeta local (local para el destino de proceso). Puede usar el objeto Run para descargar una copia.

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)

for f in run.get_file_names():
    if f.startswith('outputs/model'):
        output_file_path = os.path.join('./model', f.split('/')[-1])
        print('Downloading from {} to {} ...'.format(f, output_file_path))
        run.download_file(name=f, output_file_path=output_file_path)
```

## <a name="distributed-training"></a>Entrenamiento distribuido

El estimador [`PyTorch`](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.pytorch?view=azure-ml-py) también admite el entrenamiento distribuido en clústeres de CPU y GPU. Puede ejecutar fácilmente los trabajos distribuidos de PyTorch y Azure Machine Learning se ocupará de administrar la orquestación automáticamente.

### <a name="horovod"></a>Horovod
[Horovod](https://github.com/uber/horovod) es una plataforma de código abierto en que todo se reduce para el entrenamiento distribuido desarrollada por Uber. Ofrece una ruta de acceso fácil a los trabajos distribuidos de PyTorch GPU.

Para usar Horovod, especifique un objeto [`MpiConfiguration`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration?view=azure-ml-py) para el parámetro `distributed_training` en el constructor de PyTorch. Este parámetro garantiza que se instala la biblioteca de Horovod para su uso en el script de entrenamiento.


```Python
from azureml.train.dnn import PyTorch

estimator= PyTorch(source_directory=project_folder,
                      compute_target=compute_target,
                      script_params=script_params,
                      entry_script='script.py',
                      node_count=2,
                      process_count_per_node=1,
                      distributed_training=MpiConfiguration(),
                      framework_version='1.6',
                      use_gpu=True)
```
Horovod y sus dependencias se instalarán automáticamente, para que pueda importarlo en el script de entrenamiento `train.py` como sigue:

```Python
import torch
import horovod
```
## <a name="export-to-onnx"></a>Exportación a ONNX

Para optimizar la inferencia con el [runtime de ONNX](concept-onnx.md), convierta el modelo entrenado de PyTorch al formato ONNX. Inferencia o modelo de puntuación, es la fase donde se usa el modelo implementado para la predicción, frecuentemente en datos de producción. Si quiere ver un ejemplo, consulte el [tutorial](https://github.com/onnx/tutorials/blob/master/tutorials/PytorchOnnxExport.ipynb).

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha entrenado y registrado un aprendizaje profundo y una red neuronal mediante PyTorch en Azure Machine Learning. Para obtener información sobre cómo implementar un modelo, continúe con nuestro artículo sobre implementación de modelos.

> [!div class="nextstepaction"]
> [Cómo y dónde implementar los modelos](how-to-deploy-and-where.md)
* [Seguir métricas de ejecución durante el entrenamiento](how-to-track-experiments.md)
* [Ajustar los hiperparámetros](how-to-tune-hyperparameters.md)
* [Implementar un modelo entrenado](how-to-deploy-and-where.md)
* [Arquitectura de referencia para el entrenamiento del aprendizaje profundo distribuido en Azure](/azure/architecture/reference-architectures/ai/training-deep-learning)

