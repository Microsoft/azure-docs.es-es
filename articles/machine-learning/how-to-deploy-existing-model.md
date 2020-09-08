---
title: Uso e implementación de modelos existentes
titleSuffix: Azure Machine Learning
description: Aprenda a traer sus modelos de aprendizaje automático entrenados localmente a la nube de Azure con Azure Machine Learning.  Puede registrar modelos creados fuera de Azure Machine Learning y, a continuación, implementarlos como un servicio web o un módulo de Azure IoT Edge.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 07/17/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 04442ad2c6f12960a6c27cc96b52eae20b046851
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008209"
---
# <a name="deploy-your-existing-model-with-azure-machine-learning"></a>Implementación de un modelo existente con Azure Machine Learning
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo, aprenderá a registrar e implementar un modelo de aprendizaje automático entrenado fuera de Azure Machine Learning. Puede implementarlo como un servicio web o en un dispositivo IoT Edge.  Una vez implementado, puede supervisar el modelo y detectar el desfase de datos en Azure Machine Learning. 

Para más información sobre los conceptos y términos usados en este artículo, consulte [Administración, implementación y supervisión de modelos de Machine Learning](concept-model-management-and-deployment.md).

## <a name="prerequisites"></a>Requisitos previos

* [Un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md)
  + En los ejemplos de Python se supone que la variable `ws` está establecida en el área de trabajo de Azure Machine Learning. Para obtener más información sobre cómo conectarse al área de trabajo, consulte la documentación del SDK de [Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/?view=azure-ml-py#workspace).
  
  + En los ejemplos de la CLI se usan los marcadores de posición `myworkspace` y `myresourcegroup`, que debe reemplazar por el nombre del área de trabajo y el grupo de recursos que lo contiene.

* El [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py).  

* La [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) y la [extensión de la CLI de Machine Learning](reference-azure-machine-learning-cli.md).

* Un modelo entrenado. El modelo debe almacenarse en uno o varios archivos del entorno de desarrollo. <br><br>Para demostrar el registro de un modelo entrenado, en el código de ejemplo de este artículo se usan los modelos del [proyecto de análisis de opinión de Twitter de Paolo Ripamonti](https://www.kaggle.com/paoloripamonti/twitter-sentiment-analysis).

## <a name="register-the-models"></a>Registrar los modelos

Al registrar un modelo, podrá almacenar, editar y realizar un seguimiento de los metadatos correspondientes a los modelos de su área de trabajo. En los siguientes ejemplos de Python y CLI, el directorio `models` contiene los archivos `model.h5`, `model.w2v`, `encoder.pkl` y `tokenizer.pkl`. En este ejemplo se cargan los archivos contenidos en el directorio `models` como un nuevo registro de modelo denominado `sentiment`:

```python
from azureml.core.model import Model
# Tip: When model_path is set to a directory, you can use the child_paths parameter to include
#      only some of the files from the directory
model = Model.register(model_path = "./models",
                       model_name = "sentiment",
                       description = "Sentiment analysis model trained outside Azure Machine Learning",
                       workspace = ws)
```

Para obtener más información, consulte la referencia [Model.register()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model(class)?view=azure-ml-py#register-workspace--model-path--model-name--tags-none--properties-none--description-none--datasets-none--model-framework-none--model-framework-version-none--child-paths-none--sample-input-dataset-none--sample-output-dataset-none--resource-configuration-none-).

```azurecli
az ml model register -p ./models -n sentiment -w myworkspace -g myresourcegroup
```

> [!TIP]
> También puede agregar los objetos de diccionario `tags` y `properties` al modelo registrado. Estos valores se pueden usar más adelante para identificar un modelo específico. Por ejemplo, el marco usado, los parámetros de entrenamiento, etc.

Para obtener más información, consulte la referencia [az ml model register](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-register).


Para obtener más información sobre el registro de modelos en general, consulte [Manage, deploy, and monitor machine learning models](concept-model-management-and-deployment.md) (Administrar, implementar y supervisar los modelos de Machine Learning).

## <a name="define-inference-configuration"></a>Definir la configuración de inferencia

La configuración de inferencia define el entorno que se usa para ejecutar el modelo implementado. La configuración de inferencia hace referencia a las siguientes entidades, las cuales se usan para ejecutar el modelo cuando se implementa:

* Un script de entrada, llamado `score.py`, carga el modelo cuando se inicia el servicio implementado. Este script también se encarga de recibir datos, pasarlos al modelo y, luego, devolver una respuesta.
* Un [entorno](how-to-use-environments.md) de Azure Machine Learning. Un entorno define las dependencias de software necesarias para ejecutar el modelo y el script de entrada.

En el ejemplo siguiente se muestra cómo usar el SDK para crear un entorno y, a continuación, usarlo con una configuración de inferencia:

```python
from azureml.core.model import InferenceConfig
from azureml.core.environment import Environment
from azureml.core.conda_dependencies import CondaDependencies

# Create the environment
myenv = Environment(name="myenv")
conda_dep = CondaDependencies()

# Define the packages needed by the model and scripts
conda_dep.add_conda_package("tensorflow")
conda_dep.add_conda_package("numpy")
conda_dep.add_conda_package("scikit-learn")
# You must list azureml-defaults as a pip dependency
conda_dep.add_pip_package("azureml-defaults")
conda_dep.add_pip_package("keras")
conda_dep.add_pip_package("gensim")

# Adds dependencies to PythonSection of myenv
myenv.python.conda_dependencies=conda_dep

inference_config = InferenceConfig(entry_script="score.py",
                                   environment=myenv)
```

Para más información, consulte los siguientes artículos.

+ [Cómo usar entornos](how-to-use-environments.md).
+ Referencia de [InferenceConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig?view=azure-ml-py).


La CLI carga la configuración de inferencia de un archivo YAML:

```yaml
{
   "entryScript": "score.py",
   "runtime": "python",
   "condaFile": "myenv.yml"
}
```

Con la CLI, el entorno de Conda se define en el archivo `myenv.yml` al que hace referencia la configuración de inferencia. El siguiente código YAML es el contenido de este archivo:

```yaml
name: inference_environment
dependencies:
- python=3.6.2
- tensorflow
- numpy
- scikit-learn
- pip:
    - azureml-defaults
    - keras
    - gensim
```

Para más información sobre la configuración de inferencia, consulte [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

### <a name="entry-script-scorepy"></a>Script de entrada (score.py)

El script de entrada tiene solo dos funciones requeridas: `init()` y `run(data)`. Estas funciones se usan para inicializar el servicio en el inicio y ejecutar el modelo usando para ello los datos de solicitud que pase un cliente. El resto del script se encarga de cargar y ejecutar los modelos.

> [!IMPORTANT]
> No hay un script de entrada genérico que funcione para todos los modelos. Siempre es específico para el modelo que se usa. Debe comprender cómo cargar el modelo, el formato de datos que espera el modelo y cómo puntuar los datos mediante ese modelo.

El siguiente código de Python es un script de entrada de ejemplo (`score.py`):

```python
import os
import pickle
import json
import time
from keras.models import load_model
from keras.preprocessing.sequence import pad_sequences
from gensim.models.word2vec import Word2Vec

# SENTIMENT
POSITIVE = "POSITIVE"
NEGATIVE = "NEGATIVE"
NEUTRAL = "NEUTRAL"
SENTIMENT_THRESHOLDS = (0.4, 0.7)
SEQUENCE_LENGTH = 300

# Called when the deployed service starts
def init():
    global model
    global tokenizer
    global encoder
    global w2v_model

    # Get the path where the deployed model can be found.
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), './models')
    # load models
    model = load_model(model_path + '/model.h5')
    w2v_model = Word2Vec.load(model_path + '/model.w2v')

    with open(model_path + '/tokenizer.pkl','rb') as handle:
        tokenizer = pickle.load(handle)

    with open(model_path + '/encoder.pkl','rb') as handle:
        encoder = pickle.load(handle)

# Handle requests to the service
def run(data):
    try:
        # Pick out the text property of the JSON request.
        # This expects a request in the form of {"text": "some text to score for sentiment"}
        data = json.loads(data)
        prediction = predict(data['text'])
        #Return prediction
        return prediction
    except Exception as e:
        error = str(e)
        return error

# Determine sentiment from score
def decode_sentiment(score, include_neutral=True):
    if include_neutral:
        label = NEUTRAL
        if score <= SENTIMENT_THRESHOLDS[0]:
            label = NEGATIVE
        elif score >= SENTIMENT_THRESHOLDS[1]:
            label = POSITIVE
        return label
    else:
        return NEGATIVE if score < 0.5 else POSITIVE

# Predict sentiment using the model
def predict(text, include_neutral=True):
    start_at = time.time()
    # Tokenize text
    x_test = pad_sequences(tokenizer.texts_to_sequences([text]), maxlen=SEQUENCE_LENGTH)
    # Predict
    score = model.predict([x_test])[0]
    # Decode sentiment
    label = decode_sentiment(score, include_neutral=include_neutral)

    return {"label": label, "score": float(score),
       "elapsed_time": time.time()-start_at}  
```

Para más información sobre los scripts de entrada, consulte [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

## <a name="define-deployment"></a>Definir la implementación

El paquete [Webservice](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice?view=azure-ml-py) contiene las clases que se usan en la implementación. La clase que use determinará dónde se implementa el modelo. Por ejemplo, para implementar un servicio web en Azure Kubernetes Service, use [AksWebService.deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.akswebservice?view=azure-ml-py#deploy-configuration-autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--primary-key-none--secondary-key-none--tags-none--properties-none--description-none--gpu-cores-none--period-seconds-none--initial-delay-seconds-none--timeout-seconds-none--success-threshold-none--failure-threshold-none--namespace-none--token-auth-enabled-none--compute-target-name-none-) para crear la configuración de implementación.

El código Python siguiente define una configuración de implementación para una implementación local. Esta configuración implementa el modelo como un servicio web en el equipo local.

> [!IMPORTANT]
> Asimismo, una implementación local requiere una instalación de [Docker](https://www.docker.com/) que funcione en el equipo local:

```python
from azureml.core.webservice import LocalWebservice

deployment_config = LocalWebservice.deploy_configuration()
```

Para obtener más información sobre, consulte la referencia [LocalWebservice.deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.localwebservice?view=azure-ml-py#deploy-configuration-port-none-).

La CLI carga la configuración de implementación de un archivo YAML:

```YAML
{
    "computeType": "LOCAL"
}
```

La implementación en un destino de proceso diferente, como Azure Kubernetes Service en la nube de Azure, es tan fácil como cambiar la configuración de la implementación. Para obtener más información, consulte [How and where to deploy models](how-to-deploy-and-where.md) (Cómo y dónde implementar modelos).

## <a name="deploy-the-model"></a>Implementación del modelo

En el ejemplo siguiente se carga la información en el modelo registrado denominado `sentiment` y, a continuación, se implementa como un servicio denominado `sentiment`. Durante la implementación, la configuración de inferencia y la configuración de implementación se usan para crear y configurar el entorno de servicio:

```python
from azureml.core.model import Model

model = Model(ws, name='sentiment')
service = Model.deploy(ws, 'myservice', [model], inference_config, deployment_config)

service.wait_for_deployment(True)
print(service.state)
print("scoring URI: " + service.scoring_uri)
```

Para obtener más información, consulte la referencia [Model.deploy()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-).

Para implementar el modelo de la CLI, use el comando siguiente. Este comando implementa la versión 1 del modelo registrado (`sentiment:1`) mediante la configuración de implementación y la inferencia almacenadas en los archivos `inferenceConfig.json` y `deploymentConfig.json`:

```azurecli
az ml model deploy -n myservice -m sentiment:1 --ic inferenceConfig.json --dc deploymentConfig.json
```

Para obtener más información, consulte la referencia [az ml model deploy](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy).

Para obtener más información sobre la implementación, consulte [How and where to deploy models](how-to-deploy-and-where.md) (Cómo y dónde implementar modelos).

## <a name="request-response-consumption"></a>Consumo de solicitud-respuesta

Después de la implementación, se muestra el URI de puntuación. Los clientes pueden usar este URI para enviar solicitudes al servicio. En el ejemplo siguiente se muestra un cliente básico de Python que envía datos al servicio y muestra la respuesta:

```python
import requests
import json

scoring_uri = 'scoring uri for your service'
headers = {'Content-Type':'application/json'}

test_data = json.dumps({'text': 'Today is a great day!'})

response = requests.post(scoring_uri, data=test_data, headers=headers)
print(response.status_code)
print(response.elapsed)
print(response.json())
```

Para obtener más información sobre cómo consumir el servicio implementado, consulte [Create a client](how-to-consume-web-service.md) (Crear un cliente).

## <a name="next-steps"></a>Pasos siguientes

* [Supervisión de los modelos de Azure Machine Learning con Application Insights](how-to-enable-app-insights.md)
* [Recopilar datos de modelos en producción](how-to-enable-data-collection.md)
* [Cómo crear un cliente para un modelo implementado](how-to-consume-web-service.md)
