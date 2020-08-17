---
title: Implementación de un modelo para la inferencia con GPU
titleSuffix: Azure Machine Learning
description: En este artículo se enseña a usar Azure Machine Learning para implementar un modelo de aprendizaje profundo de TensorFlow habilitado para GPU como un servicio web y las solicitudes de inferencia de puntuación.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: vaidyas
author: csteegz
ms.reviewer: larryfr
ms.date: 06/17/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 6bb6b373482a937819984cea7d1dc264d4f08a74
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87846843"
---
# <a name="deploy-a-deep-learning-model-for-inference-with-gpu"></a>Implementación de un modelo de aprendizaje profundo para la inferencia con GPU
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo aprenderá a usar Azure Machine Learning para implementar un modelo habilitado para GPU como un servicio web. La información de este artículo se basa en la implementación de un modelo en Azure Kubernetes Service (AKS). El clúster de AKS proporciona un recurso de GPU que el modelo usa para la inferencia.

La inferencia, o la puntuación del modelo, es la fase en que se usa el modelo implementado para realizar predicciones. El uso de GPU en lugar de CPU ofrece ventajas de rendimiento en el cálculo que se puede paralelizar considerablemente.

> [!IMPORTANT]
> En el caso de las implementaciones de servicios web, la inferencia de GPU solo se admite en Azure Kubernetes Service. Para realizar la inferencia mediante __una canalización de aprendizaje automático__, las GPU solo se admiten en Proceso de Azure Machine Learning. Para más información sobre el uso de canalizaciones de aprendizaje automático, consulte [Ejecución de predicciones por lotes](how-to-use-parallel-run-step.md). 

> [!TIP]
> Aunque los fragmentos de código de este artículo usan un modelo TensorFlow, puede aplicar la información a cualquier marco de aprendizaje automático que admita GPU.

> [!NOTE]
> La información de este artículo se basa en la información del artículo [Realización de implementaciones en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md). Aunque en ese artículo se trata de forma general la realización de implementaciones en Azure Container Service, este trata la implementación específica de GPU.

## <a name="prerequisites"></a>Requisitos previos

* Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* Un entorno de desarrollo de Python con el SDK de Azure Machine Learning instalado. Para más información, consulte [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py).  

* Un modelo registrado que use una GPU.

    * Para obtener información sobre cómo registrar modelos, vea [Implementación de modelos](how-to-deploy-and-where.md#registermodel).

    * Para crear y registrar el modelo de TensorFlow que se usa para crear este documento, consulte [Entrenamiento de un modelo de TensorFlow](how-to-train-tensorflow.md).

* Conocimientos generales sobre [cómo y dónde implementar los modelos](how-to-deploy-and-where.md).

## <a name="connect-to-your-workspace"></a>Conexión con su área de trabajo

Para conectarse a un área de trabajo existente, use el código siguiente:

> [!IMPORTANT]
> Este fragmento de código espera que la configuración del área de trabajo se guarde en el directorio actual o en su elemento primario. Para más información sobre cómo crear un área de trabajo, consulte [Creación y administración de áreas de trabajo de Azure Machine Learning](how-to-manage-workspace.md).   Para más información sobre cómo guardar la configuración en un archivo, consulte [Creación de un archivo de configuración de área de trabajo](how-to-configure-environment.md#workspace).

```python
from azureml.core import Workspace

# Connect to the workspace
ws = Workspace.from_config()
```

## <a name="create-a-kubernetes-cluster-with-gpus"></a>Creación de un clúster de Kubernetes con GPU

Azure Kubernetes Service proporciona muchas opciones de GPU diferentes. Puede usar cualquiera de ellas para la inferencia del modelo. Consulte [la lista de máquinas virtuales de la serie N](https://azure.microsoft.com/pricing/details/virtual-machines/linux/#n-series) para obtener un desglose completo de funcionalidades y costos.

En el código siguiente se muestra cómo crear un nuevo clúster de AKS para su área de trabajo:

```python
from azureml.core.compute import ComputeTarget, AksCompute
from azureml.exceptions import ComputeTargetException

# Choose a name for your cluster
aks_name = "aks-gpu"

# Check to see if the cluster already exists
try:
    aks_target = ComputeTarget(workspace=ws, name=aks_name)
    print('Found existing compute target')
except ComputeTargetException:
    print('Creating a new compute target...')
    # Provision AKS cluster with GPU machine
    prov_config = AksCompute.provisioning_configuration(vm_size="Standard_NC6")

    # Create the cluster
    aks_target = ComputeTarget.create(
        workspace=ws, name=aks_name, provisioning_configuration=prov_config
    )

    aks_target.wait_for_completion(show_output=True)
```

> [!IMPORTANT]
> Azure le facturará siempre y cuando exista el clúster de AKS. Asegúrese de eliminar el clúster de AKS cuando haya terminado con él.

Para más información acerca del uso de Azure Kubernetes Service con Azure Machine Learning, consulte [Realización de implementaciones en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).

## <a name="write-the-entry-script"></a>Escribir el script de entrada

El script de entrada recibe los datos enviados al servicio web, los pasa al modelo y devuelve los resultados de la puntuación. El siguiente script carga el modelo de TensorFlow en el inicio y, luego, usa el modelo para puntuar los datos.

> [!TIP]
> El script de entrada es específico para su modelo. Por ejemplo, el script debe conocer el marco que se va a usar con el modelo, los formatos de datos, etc.

```python
import json
import numpy as np
import os
import tensorflow as tf

from azureml.core.model import Model


def init():
    global X, output, sess
    tf.reset_default_graph()
    model_root = os.getenv('AZUREML_MODEL_DIR')
    # the name of the folder in which to look for tensorflow model files
    tf_model_folder = 'model'
    saver = tf.train.import_meta_graph(
        os.path.join(model_root, tf_model_folder, 'mnist-tf.model.meta'))
    X = tf.get_default_graph().get_tensor_by_name("network/X:0")
    output = tf.get_default_graph().get_tensor_by_name("network/output/MatMul:0")

    sess = tf.Session()
    saver.restore(sess, os.path.join(model_root, tf_model_folder, 'mnist-tf.model'))


def run(raw_data):
    data = np.array(json.loads(raw_data)['data'])
    # make prediction
    out = output.eval(session=sess, feed_dict={X: data})
    y_hat = np.argmax(out, axis=1)
    return y_hat.tolist()
```

Este archivo se denomina `score.py`. Para obtener más información sobre los scripts de entrada, consulte [Cómo y dónde implementar los modelos](how-to-deploy-and-where.md).

## <a name="define-the-conda-environment"></a>Definir el entorno de conda

El archivo de entorno de Conda especifica las dependencias del servicio. Incluye las dependencias que requiere el modelo y el script de entrada. Tenga en cuenta que debe indicar valores predeterminados de azureml con versión > = 1.0.45 como una dependencia de PIP, ya que contiene la funcionalidad necesaria para hospedar el modelo como un servicio web. En el siguiente código YAML, se define el entorno de un modelo de TensorFlow. Especifica `tensorflow-gpu`, que hará uso de la GPU usada en esta implementación:

```yaml
name: project_environment
dependencies:
  # The python interpreter version.
  # Currently Azure ML only supports 3.5.2 and later.
- python=3.6.2

- pip:
  # You must list azureml-defaults as a pip dependency
  - azureml-defaults>=1.0.45
- numpy
- tensorflow-gpu=1.12
channels:
- conda-forge
```

En este ejemplo, el archivo se guarda como `myenv.yml`.

## <a name="define-the-deployment-configuration"></a>Definición de la configuración de la implementación

> [!IMPORTANT]
> AKS no permite que los pods compartan los GPU, solo puede tener tantas réplicas de un servicio web habilitado para GPU como GPU haya en el clúster.

La configuración de implementación define el entorno de Azure Kubernetes Service que se usa para ejecutar el servicio web:

```python
from azureml.core.webservice import AksWebservice

gpu_aks_config = AksWebservice.deploy_configuration(autoscale_enabled=False,
                                                    num_replicas=3,
                                                    cpu_cores=2,
                                                    memory_gb=4)
```

Para obtener más información, consulte la documentación de referencia de [AksService.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.akswebservice?view=azure-ml-py#deploy-configuration-autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--primary-key-none--secondary-key-none--tags-none--properties-none--description-none--gpu-cores-none--period-seconds-none--initial-delay-seconds-none--timeout-seconds-none--success-threshold-none--failure-threshold-none--namespace-none--token-auth-enabled-none--compute-target-name-none-).

## <a name="define-the-inference-configuration"></a>Definición de la configuración de inferencia

La configuración de inferencia apunta al script de entrada y a un objeto de entorno, que usa una imagen de Docker con compatibilidad con GPU. Tenga en cuenta que el archivo YAML usado para la definición del entorno debe enumerar los valores predeterminados de azureml con la versión > = 1.0.45 como una dependencia de PIP, porque contiene la funcionalidad necesaria para hospedar el modelo como un servicio web.

```python
from azureml.core.model import InferenceConfig
from azureml.core.environment import Environment, DEFAULT_GPU_IMAGE

myenv = Environment.from_conda_specification(name="myenv", file_path="myenv.yml")
myenv.docker.base_image = DEFAULT_GPU_IMAGE
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
```

Para obtener más información sobre los entornos, consulte el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md).
Para obtener más información, consulte la documentación de referencia de [InferenceConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig?view=azure-ml-py).

## <a name="deploy-the-model"></a>Implementación del modelo

Implemente el modelo en el clúster de AKS y espere a que se cree el servicio.

```python
from azureml.core.model import Model

# Name of the web service that is deployed
aks_service_name = 'aks-dnn-mnist'
# Get the registerd model
model = Model(ws, "tf-dnn-mnist")
# Deploy the model
aks_service = Model.deploy(ws,
                           models=[model],
                           inference_config=inference_config,
                           deployment_config=gpu_aks_config,
                           deployment_target=aks_target,
                           name=aks_service_name)

aks_service.wait_for_deployment(show_output=True)
print(aks_service.state)
```

Para obtener más información, consulte la documentación de referencia del [modelo](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py).

## <a name="issue-a-sample-query-to-your-service"></a>Emisión de una consulta de ejemplo para el servicio

Enviar una consulta de prueba para el modelo implementado. Cuando se envía una imagen jpeg al modelo, este puntúa la imagen. En el siguiente ejemplo de código, se descargan los datos de prueba y, a continuación, se selecciona una imagen de prueba aleatoria para enviar al servicio.

```python
# Used to test your webservice
import os
import urllib
import gzip
import numpy as np
import struct
import requests

# load compressed MNIST gz files and return numpy arrays
def load_data(filename, label=False):
    with gzip.open(filename) as gz:
        struct.unpack('I', gz.read(4))
        n_items = struct.unpack('>I', gz.read(4))
        if not label:
            n_rows = struct.unpack('>I', gz.read(4))[0]
            n_cols = struct.unpack('>I', gz.read(4))[0]
            res = np.frombuffer(gz.read(n_items[0] * n_rows * n_cols), dtype=np.uint8)
            res = res.reshape(n_items[0], n_rows * n_cols)
        else:
            res = np.frombuffer(gz.read(n_items[0]), dtype=np.uint8)
            res = res.reshape(n_items[0], 1)
    return res

# one-hot encode a 1-D array
def one_hot_encode(array, num_of_classes):
    return np.eye(num_of_classes)[array.reshape(-1)]

# Download test data
os.makedirs('./data/mnist', exist_ok=True)
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz', filename='./data/mnist/test-images.gz')
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz', filename='./data/mnist/test-labels.gz')

# Load test data from model training
X_test = load_data('./data/mnist/test-images.gz', False) / 255.0
y_test = load_data('./data/mnist/test-labels.gz', True).reshape(-1)

# send a random row from the test set to score
random_index = np.random.randint(0, len(X_test)-1)
input_data = "{\"data\": [" + str(list(X_test[random_index])) + "]}"

api_key = aks_service.get_keys()[0]
headers = {'Content-Type': 'application/json',
           'Authorization': ('Bearer ' + api_key)}
resp = requests.post(aks_service.scoring_uri, input_data, headers=headers)

print("POST to url", aks_service.scoring_uri)
print("label:", y_test[random_index])
print("prediction:", resp.text)
```

Para obtener más información sobre cómo crear una aplicación cliente, consulte [Creación de un cliente para consumir el servicio web implementado](how-to-consume-web-service.md).

## <a name="clean-up-the-resources"></a>Limpiar los recursos

Si creó el clúster de AKS específicamente para este ejemplo, elimine los recursos cuando haya terminado.

> [!IMPORTANT]
> Azure factura en función de cuánto tiempo se implementa el clúster de AKS. Asegúrese de limpiarlo cuando haya terminado con él.

```python
aks_service.delete()
aks_target.delete()
```

## <a name="next-steps"></a>Pasos siguientes

* [Implementar el modelo en FPGA](how-to-deploy-fpga-web-service.md)
* [Implementar el modelo con ONNX](concept-onnx.md#deploy-onnx-models-in-azure)
* [Entrenar modelos DNN de TensorFlow](how-to-train-tensorflow.md)
