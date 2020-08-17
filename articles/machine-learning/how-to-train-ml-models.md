---
title: Entrenamiento de modelos de aprendizaje automático con estimadores
titleSuffix: Azure Machine Learning
description: Aprenda a realizar un entrenamiento de nodo único y distribuido de modelos de aprendizaje automático tradicional y modelos de aprendizaje profundo con la clase Estimator de Azure Machine Learning
ms.author: maxluk
author: maxluk
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: sgilley
ms.date: 03/09/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 9f63b4215e9b4a67a439e47501876d237a6d3c3b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320925"
---
# <a name="train-models-with-azure-machine-learning-using-estimator"></a>Entrenamiento de modelos con Azure Machine Learning mediante un objeto Estimator
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Con Azure Machine Learning, puede enviar fácilmente el script de entrenamiento a [diversos destinos de proceso](how-to-set-up-training-targets.md#compute-targets-for-training), con un [objeto RunConfiguration](how-to-set-up-training-targets.md#whats-a-run-configuration) y un [objeto ScriptRunConfig](how-to-set-up-training-targets.md#submit). Ese patrón le ofrece mucha flexibilidad y el máximo control.


La clase estimador facilita el entrenamiento de modelos con aprendizaje profundo y aprendizaje de refuerzo. Proporciona una abstracción de alto nivel que le permite construir fácilmente la configuración de ejecución. Puede crear y usar un [Estimator](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator?view=azure-ml-py) genérico para enviar el script de entrenamiento con cualquier marco de aprendizaje que elija (como scikit-learn) en cualquier destino de proceso: ya sea la máquina local, una sola máquina virtual en Azure o un clúster de GPU en Azure. En las tareas de PyTorch, TensorFlow, Chainer y de aprendizaje de refuerzo, Azure Machine Learning también proporciona los estimadores de [PyTorch](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.pytorch?view=azure-ml-py), [TensorFlow](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py), [Chainer](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.chainer?view=azure-ml-py) y [aprendizaje de refuerzo](how-to-use-reinforcement-learning.md) para simplificar el uso de estos marcos.

## <a name="train-with-an-estimator"></a>Entrenamiento con un estimador

Una vez que ha creado el [área de trabajo](concept-workspace.md) y ha configurado el [entorno de desarrollo](how-to-configure-environment.md), el entrenamiento de un modelo en Azure Machine Learning implica los siguientes pasos:  
1. Creación de un [destino de proceso remoto](how-to-set-up-training-targets.md) (tenga en cuenta que también puede usar el equipo local como destino de proceso)
2. Carga de los [datos de aprendizaje](how-to-access-data.md) para el almacén de datos (opcional)
3. Creación del [script de aprendizaje](tutorial-train-models-with-aml.md#create-a-training-script)
4. Creación de un objeto `Estimator`
5. Envío del objeto Estimator a un objeto de experimento en el área de trabajo

Este artículo se centra en los pasos 4 y 5. Para obtener información sobre los pasos del 1 al 3, consulte el [tutorial de entrenamiento de un modelo](tutorial-train-models-with-aml.md) para ver un ejemplo.

### <a name="single-node-training"></a>Entrenamiento de nodo único

Utilice un objeto `Estimator` para una ejecución de entrenamiento de nodo único en proceso remoto en Azure para un modelo scikit-learn. Debería haber creado ya su objeto de [destino de proceso](how-to-set-up-training-targets.md#amlcompute) `compute_target` y su objeto [FileDataset](how-to-create-register-datasets.md) `ds`.

```Python
from azureml.train.estimator import Estimator

script_params = {
    # to mount files referenced by mnist dataset
    '--data-folder': ds.as_named_input('mnist').as_mount(),
    '--regularization': 0.8
}

sk_est = Estimator(source_directory='./my-sklearn-proj',
                   script_params=script_params,
                   compute_target=compute_target,
                   entry_script='train.py',
                   conda_packages=['scikit-learn'])
```

Este fragmento de código especifica los parámetros siguientes para el constructor `Estimator`.

Parámetro | Descripción
--|--
`source_directory`| Directorio local que contiene todo el código necesario para el trabajo de aprendizaje. Esta carpeta se copia desde el equipo local al proceso remoto.
`script_params`| Diccionario que especifica los argumentos de línea de comandos que se pasan al script de entrenamiento `entry_script`, en forma de pares `<command-line argument, value>`. Para especificar una marca detallada en `script_params`, use `<command-line argument, "">`.
`compute_target`| El destino de proceso remoto en el que el script de entrenamiento se ejecutará, en este caso, un clúster de Proceso de Azure Machine Learning ([AmlCompute](how-to-set-up-training-targets.md#amlcompute)). (Tenga en cuenta que, aunque el clúster AmlCompute es el destino más usado, también es posible elegir otros tipos de destino, como máquinas virtuales de Azure o incluso el equipo local).
`entry_script`| Ruta del archivo (relativa al `source_directory`) del script de aprendizaje que se va a ejecutar en el proceso remoto. Este archivo y los archivos adicionales de los que depende deben encontrarse en esta carpeta.
`conda_packages`| Lista de paquetes de Python para instalarse mediante Conda que necesita el script de aprendizaje.  

El constructor tiene otro parámetro llamado `pip_packages` que puede usar para los paquetes de pip necesarios.

Ahora que ha creado el objeto `Estimator`, envíe el trabajo de aprendizaje que se va a ejecutar en el proceso remoto mediante una llamada a la función `submit` en el objeto `experiment` del [experimento](concept-azure-machine-learning-architecture.md#experiments). 

```Python
run = experiment.submit(sk_est)
print(run.get_portal_url())
```

> [!IMPORTANT]
> **Carpetas especiales**: dos carpetas, *outputs* y *logs*, reciben un tratamiento especial por parte de Azure Machine Learning. Durante el entrenamiento, si escribe archivos en las carpetas llamadas *outputs* y *logs* relativas al directorio raíz (`./outputs` y `./logs`, respectivamente), estos archivos se cargarán automáticamente en su historial de ejecución, por lo que tendrá acceso a estos una vez completada su ejecución.
>
> Para crear artefactos durante el entrenamiento (por ejemplo, archivos de modelo, puntos de control, archivos de datos o imágenes trazadas), escríbalos en la carpeta `./outputs`.
>
> De forma similar, puede escribir cualquier registro desde su ejecución de entrenamiento en la carpeta `./logs`. Para usar la [integración TensorBoard](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/export-run-history-to-tensorboard/export-run-history-to-tensorboard.ipynb) de Azure Machine Learning, asegúrese de escribir sus registros de TensorBoard en esta carpeta. Mientras su ejecución está en curso, podrá iniciar TensorBoard y transmitir estos registros.  Posteriormente, también podrá restaurar los registros desde cualquiera de sus ejecuciones anteriores.
>
> Por ejemplo, para descargar un archivo escrito en la carpeta *outputs* en su equipo local después de su ejecución de entrenamiento remoto: `run.download_file(name='outputs/my_output_file', output_file_path='my_destination_path')`

### <a name="distributed-training-and-custom-docker-images"></a>Entrenamiento distribuido e imágenes de Docker personalizadas

Hay dos escenarios de entrenamiento adicionales que puede llevar a cabo con el objeto `Estimator`:
* Uso de una imagen de Docker personalizada
* Entrenamiento distribuido en un clúster de varios nodos

El siguiente código muestra cómo llevar a cabo el entrenamiento distribuido para un modelo Keras. Además, en lugar de usar las imágenes de Azure Machine Learning predeterminadas, especifica una imagen de Docker personalizada de Docker Hub `continuumio/miniconda` para el entrenamiento.

Debería haber creado su objeto de [destino de proceso](how-to-set-up-training-targets.md#amlcompute)`compute_target`. Puede crear el estimador de la siguiente manera:

```Python
from azureml.train.estimator import Estimator
from azureml.core.runconfig import MpiConfiguration

estimator = Estimator(source_directory='./my-keras-proj',
                      compute_target=compute_target,
                      entry_script='train.py',
                      node_count=2,
                      process_count_per_node=1,
                      distributed_training=MpiConfiguration(),        
                      conda_packages=['tensorflow', 'keras'],
                      custom_docker_image='continuumio/miniconda')
```

El código anterior expone los siguientes parámetros nuevos para el constructor `Estimator`:

Parámetro | Descripción | Valor predeterminado
--|--|--
`custom_docker_image`| Nombre de la imagen que desea usar. Solo puede proporcionar imágenes disponibles en repositorios de Docker públicos (en este caso, Docker Hub). Para usar una imagen desde un repositorio de Docker privado, utilice el parámetro `environment_definition` del constructor en su lugar. [Vea el ejemplo](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/how-to-use-estimator/how-to-use-estimator.ipynb). | `None`
`node_count`| Número de nodos que se usará para el trabajo de aprendizaje. | `1`
`process_count_per_node`| Número de procesos (o "trabajos") que se ejecutarán en cada nodo. En este caso, usa las GPU `2` disponibles en cada nodo.| `1`
`distributed_training`| Objeto [MPIConfiguration ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration?view=azure-ml-py) para iniciar el entrenamiento distribuido mediante el back-end de MPI.  | `None`


Por último, envíe el trabajo de aprendizaje:
```Python
run = experiment.submit(estimator)
print(run.get_portal_url())
```

## <a name="registering-a-model"></a>Registro de un modelo

Una vez que haya entrenado el modelo, puede guardarlo y registrarlo en el área de trabajo. El registro del modelo permite almacenar los modelos y crear versiones de ellos en el área de trabajo para simplificar la [administración e implementación de modelos](concept-model-management-and-deployment.md).

Al ejecutar el código siguiente, se registrará el modelo en el área de trabajo y estará disponible para hacer referencia a él por el nombre en contextos de proceso o scripts de implementación remotos. Consulte [`register_model`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run.run?view=azure-ml-py#register-model-model-name--model-path-none--tags-none--properties-none--model-framework-none--model-framework-version-none--description-none--datasets-none--sample-input-dataset-none--sample-output-dataset-none--resource-configuration-none----kwargs-) en los documentos de referencia para más información y conocer parámetros adicionales.

```python
model = run.register_model(model_name='sklearn-sample', model_path=None)
```

## <a name="github-tracking-and-integration"></a>Integración y seguimiento de GitHub

Cuando se inicia una ejecución de entrenamiento en la que el directorio de origen es un repositorio de GIT local, se almacena información sobre el repositorio en el historial de ejecución. Para más información, consulte [Integración de Git con Azure Machine Learning](concept-train-model-git-integration.md).

## <a name="examples"></a>Ejemplos
Para obtener un cuaderno que muestre los aspectos básicos de un patrón del objeto Estimator, vea:
* [how-to-use-azureml/training-with-deep-learning/how-to-use-estimator](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/how-to-use-estimator/how-to-use-estimator.ipynb)

Para obtener un notebook que entrena un modelo scikit-learn con Estimator, vea:
* [tutorials/img-classification-part1-training.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/image-classification-mnist-data/img-classification-part1-training.ipynb)

Para obtener cuadernos de modelos de entrenamiento con objetos Estimator específicos del marco de aprendizaje profundo, vea:

* [how-to-use-azureml/ml-frameworks](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Seguir métricas de ejecución durante el entrenamiento](how-to-track-experiments.md)
* [Entrenar modelos de PyTorch](how-to-train-pytorch.md)
* [Entrenar modelos de TensorFlow](how-to-train-tensorflow.md)
* [Entrenar una red neuronal profunda de aprendizaje de refuerzo](how-to-use-reinforcement-learning.md)
* [Ajustar los hiperparámetros](how-to-tune-hyperparameters.md)
* [Implementar un modelo entrenado](how-to-deploy-and-where.md)
* [Creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md)