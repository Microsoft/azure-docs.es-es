---
title: Implementación de modelo para inferencia con GPU
titleSuffix: Azure Machine Learning service
description: Obtenga información sobre cómo implementar un modelo de aprendizaje profundo como un servicio web que utiliza una GPU para inferencia. En este artículo, se implementa un modelo de Tensorflow en un clúster de Azure Kubernetes Service. El clúster usa una máquina virtual basados en GPU para hospedar el servicio web y las solicitudes de inferencia de puntuación.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: vaidyas
author: csteegz
ms.reviewer: larryfr
ms.date: 05/02/2019
ms.openlocfilehash: 7796e8dc07889c9816e4227f3b38904d91a24da3
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595687"
---
# <a name="deploy-a-deep-learning-model-for-inferencing-with-gpu"></a>Implementar un modelo de aprendizaje profundo para inferencia con GPU

Obtenga información sobre cómo usar la inferencia GPU para un modelo implementado como un servicio web de aprendizaje automático. En este artículo, aprenderá a usar el servicio Azure Machine Learning para implementar un modelo de aprendizaje profundo de Tensorflow de ejemplo. El modelo se implementa en un clúster de Azure Kubernetes Service (AKS) que usa una máquina virtual habilitada para GPU para hospedar el servicio. Cuando se envían las solicitudes al servicio, el modelo usa la GPU para realizar la inferencia.

Las GPU ofrecen ventajas de rendimiento con respecto a CPU en cálculo paralelizar. Aprendizaje e inferencia de modelos (especialmente para grandes lotes de solicitudes) de aprendizaje profundo son los casos de uso excelente para GPU.  

En este ejemplo le mostrará cómo implementar un modelo de TensorFlow que guardó en Azure Machine Learning mediante:
* Creación de un clúster AKS habilitadas para GPU
* Implementar un modelo con GPU de Tensorflow

## <a name="prerequisites"></a>Requisitos previos

* El área de trabajo de Azure Machine Learning services
* Python
* Tensorflow SavedModel registrado. Para obtener información sobre cómo registrar modelos vea [implementar modelos](https://docs.microsoft.com/azure/machine-learning/service/how-to-deploy-and-where#registermodel)

En este artículo se basa en el cuaderno de Jupyter, [implementar modelos de Tensorflow en AKS](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/production-deploy-to-aks-gpu/production-deploy-to-aks-gpu.ipynb), que utiliza TensorFlow guardado modela e implementa en un clúster de AKS. Sin embargo, con pequeños cambios en el archivo de puntuación y el archivo de entorno es aplicable a cualquier marco de aprendizaje automático que son compatibles con GPU.  

## <a name="provision-aks-cluster-with-gpus"></a>Clúster de AKS aprovisionar con GPU
Azure tiene muchas opciones diferentes de GPU, que se puede usar para inferencia. Consulte [la lista de la serie N](https://azure.microsoft.com/pricing/details/virtual-machines/linux/#n-series) para obtener un desglose completo de funcionalidades y los costos. 

Para obtener más información sobre el uso de AKS con el servicio de Azure Machine Learning, consulte el [cómo implementar y donde el artículo.](https://docs.microsoft.com/azure/machine-learning/service/how-to-deploy-and-where#create-a-new-cluster)

```python
# Provision AKS cluster with GPU machine
prov_config = AksCompute.provisioning_configuration(vm_size="Standard_NC6")

# Create the cluster
aks_target = ComputeTarget.create(
    workspace=ws, name=aks_name, provisioning_configuration=prov_config
)

aks_target.wait_for_deployment()
```

> [!IMPORTANT]
> Azure le facturará siempre y cuando se aprovisiona el clúster de AKS. Asegúrese de eliminar el clúster de AKS cuando haya terminado con él.


## <a name="write-entry-script"></a>Escribir el script de entrada

Guarde lo siguiente en el directorio de trabajo como `score.py`. Este archivo se utilizará para puntuar imágenes a medida que se envían al servicio. Este archivo carga el TensorFlow guarda el modelo y, a continuación, en cada entrada de la solicitud pasa la imagen de entrada a la sesión de TensorFlow y devuelve las puntuaciones resultantes.
Otros marcos de inferencia requerirá distintos archivos de puntuación.

```python
import tensorflow as tf
import numpy as np
import ujson
from azureml.core.model import Model
from azureml.contrib.services.aml_request import AMLRequest, rawhttp
from azureml.contrib.services.aml_response import AMLResponse

def init():
    global session
    global input_name
    global output_name
    
    session = tf.Session()

    model_path = Model.get_model_path('resnet50')
    model = tf.saved_model.loader.load(session, ['serve'], model_path)
    if len(model.signature_def['serving_default'].inputs) > 1:
        raise ValueError("This score.py only supports one input")
    input_name = [tensor.name for tensor in model.signature_def['serving_default'].inputs.values()][0]
    output_name = [tensor.name for tensor in model.signature_def['serving_default'].outputs.values()]
    

@rawhttp
def run(request):
    if request.method == 'POST':
        reqBody = request.get_data(False)
        resp = score(reqBody)
        return AMLResponse(resp, 200)
    if request.method == 'GET':
        respBody = str.encode("GET is not supported")
        return AMLResponse(respBody, 405)
    return AMLResponse("bad request", 500)

def score(data):
    result = session.run(output_name, {input_name: [data]})
    return ujson.dumps(result[1])

if __name__ == "__main__":
    init()
    with open("lynx.jpg", 'rb') as f: #load file for testing locally
        content = f.read()
        print(score(content))

```

## <a name="define-conda-environment"></a>Definir el entorno de Conda
Cree un archivo de entorno de conda denominado `myenv.yml` para especificar las dependencias para el servicio. Es importante especificar que está usando `tensorflow-gpu` para lograr un rendimiento acelerado.
```yaml
name: aml-accel-perf
channels:
  - defaults
dependencies:
  - tensorflow-gpu = 1.12
  - numpy
  - ujson
  - pip:
    - azureml-core
    - azureml-contrib-services
```

## <a name="define-gpu-inferenceconfig"></a>Definir InferenceConfig GPU

Crear un [ `InferenceConfig` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig?view=azure-ml-py) que especifica que va a habilitar la GPU. Esto garantizará que CUDA se instala con la imagen.

```python
from azureml.core.model import Model
from azureml.core.model import InferenceConfig

aks_service_name ='gpu-rn'
gpu_aks_config = AksWebservice.deploy_configuration(autoscale_enabled = False, 
                                                    num_replicas = 3, 
                                                    cpu_cores=2, 
                                                    memory_gb=4)
model = Model(ws,"resnet50")

inference_config = InferenceConfig(runtime= "python", 
                                   entry_script="score.py",
                                   conda_file="myenv.yml", 
                                   gpu_enabled=True)
```

Para obtener más información, consulte [InferenceConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig?view=azure-ml-py) y [AksServiceDeploymentConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aks.aksservicedeploymentconfiguration?view=azure-ml-py).
## <a name="deploy-the-model"></a>Implementación del modelo

Implementar el modelo en el clúster de AKS y espere a que se crea el servicio.

```python
aks_service = Model.deploy(ws,
                           models=[model],
                           inference_config=inference_config, 
                           deployment_config=aks_config,
                           deployment_target=aks_target,
                           name=aks_service_name)

aks_service.wait_for_deployment(show_output = True)
print(aks_service.state)
```

> [!NOTE]
> Servicio Azure Machine Learning no implementará un modelo con un `InferenceConfig` que espera la GPU a un clúster sin GPU.

Para obtener más información, consulte [modelo](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py).

## <a name="issue-sample-query-to-deployed-model"></a>Consulta de ejemplo de problema para implementa el modelo

Emitir una consulta de ejemplo para el modelo implementado. Este modelo puntuar cualquier imagen jpeg que enviar al servicio como una solicitud post. 

```python
scoring_url = aks_service.scoring_uri
api_key = aks_service.get_key()(0)
IMAGEURL = "https://upload.wikimedia.org/wikipedia/commons/thumb/6/68/Lynx_lynx_poing.jpg/220px-Lynx_lynx_poing.jpg"

headers = {'Authorization':('Bearer '+ api_key)}
img_data = read_image_from(IMAGEURL).read()
r = requests.post(scoring_url, data = img_data, headers=headers)
```

> [!IMPORTANT]
> Para optimizar la latencia y el rendimiento, el cliente debe estar en la misma región de Azure que el punto de conexión.  Actualmente, las API se crean en la región de Azure Este de EE. UU.

## <a name="cleaning-up-the-resources"></a>Limpiar los recursos

Elimine los recursos cuando haya terminado con la demostración.

> [!IMPORTANT]
> Azure le facturará en función de cuánto se implementa el clúster de AKS. Asegúrese de que limpiar una vez haya terminado con él.

```python
aks_service.delete()
aks_target.delete()
```

## <a name="next-steps"></a>Pasos siguientes

* [Implementar el modelo en FPGA](https://docs.microsoft.com/azure/machine-learning/service/how-to-deploy-fpga-web-service)
* [Implementar el modelo con ONNX](https://docs.microsoft.com/azure/machine-learning/service/how-to-build-deploy-onnx#deploy)
* [Entrenar modelos de DNN de Tensorflow](https://docs.microsoft.com/azure/machine-learning/service/how-to-train-tensorflow)
