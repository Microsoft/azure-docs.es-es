---
title: Implementación de un servicio de inferencia cifrada (versión preliminar)
titleSuffix: Azure Machine Learning
description: Información sobre cómo usar Microsoft SEAL para implementar un servicio de predicción cifrada para la clasificación de imágenes
author: luisquintanilla
ms.author: luquinta
ms.date: 07/09/2020
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: b241220168a5cd81ce23be729944df387e87e567
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87844549"
---
# <a name="how-to-deploy-an-encrypted-inferencing-web-service-preview"></a>Implementación de un servicio web de inferencia cifrada (versión preliminar)

Obtenga información sobre cómo implementar un modelo de clasificación de imágenes como un servicio web de inferencia cifrada en [Azure Container Instances](https://docs.microsoft.com/azure/container-instances/) (ACI). El servicio web es una imagen de contenedor de Docker que contiene el modelo y la lógica de puntuación.

En esta guía, se usa Azure Machine Learning Service para:

> [!div class="checklist"]
> * Configurar los entornos.
> * Implementar el servicio web de inferencia cifrada.
> * Preparar los recursos de prueba.
> * Realizar predicciones cifradas.
> * Limpieza de recursos

ACI es una excelente solución para probar y conocer el flujo de trabajo de implementación de modelo. Para implementaciones de producción escalables, considere la posibilidad de usar Azure Kubernetes Service. Para más información, consulte [cómo y dónde realizar la implementación](https://docs.microsoft.com/azure/machine-learning/service/how-to-deploy-and-where).

El método de cifrado que se usa en este ejemplo es el [cifrado homomórfico](https://github.com/Microsoft/SEAL#homomorphic-encryption). El cifrado homomórfico permite realizar cálculos en datos cifrados sin necesidad de tener acceso a una clave secreta (descifrado). Los resultados de los cálculos están cifrados y solo los puede revelar el propietario de la clave secreta. 

## <a name="prerequisites"></a>Requisitos previos

En esta guía se da por supuesto que tiene un modelo de clasificación de imágenes registrado en Azure Machine Learning. Si no es así, registre el modelo con un [modelo previamente entrenado](https://github.com/Azure/MachineLearningNotebooks/raw/master/tutorials/image-classification-mnist-data/sklearn_mnist_model.pkl) o cree el suyo propio; para ello, complete el [tutorial de entrenamiento de un modelo de clasificación de imágenes con Azure Machine Learning](tutorial-train-models-with-aml.md).

## <a name="configure-local-environment"></a>Configuración del entorno local

En un cuaderno de Jupyter

1. Importe los paquetes de Python necesarios para este ejemplo.

    ```python
    %matplotlib inline
    import numpy as np
    import matplotlib.pyplot as plt

    import azureml.core

    # display the core SDK version number
    print("Azure ML SDK Version: ", azureml.core.VERSION)
    ```

2. Instale la biblioteca de cifrado homomórfico para la inferencia segura.

    > [!NOTE]
    > El paquete `encrypted-inference` actualmente está en versión preliminar.

    [`encrypted-inference`](https://pypi.org/project/encrypted-inference) es una biblioteca que contiene enlaces para la inferencia cifrada basada en [Microsoft SEAL](https://github.com/Microsoft/SEAL).

    ```python
    !pip install encrypted-inference==0.9
    ```

## <a name="configure-the-inferencing-environment"></a>Configuración del entorno de inferencia

Cree un entorno para la inferencia y agregue el paquete de `encrypted-inference` como una dependencia de Conda.

```python
from azureml.core.environment import Environment
from azureml.core.conda_dependencies import CondaDependencies

# to install required packages
env = Environment('tutorial-env')
cd = CondaDependencies.create(pip_packages=['azureml-dataprep[pandas,fuse]>=1.1.14', 'azureml-defaults', 'azure-storage-blob', 'encrypted-inference==0.9'], conda_packages = ['scikit-learn==0.22.1'])

env.python.conda_dependencies = cd

# Register environment to re-use later
env.register(workspace = ws)
```

## <a name="deploy-encrypted-inferencing-web-service"></a>Implementar el servicio web de inferencia cifrada.

Implementación del modelo como un servicio web hospedado en ACI

Para crear el entorno correcto para ACI, proporcione lo siguiente:

* Un script de puntuación para mostrar cómo usar el modelo.
* Un archivo de configuración para compilar el ACI.
* Un modelo entrenado.

### <a name="create-scoring-script"></a>Creación del script de puntuación

Cree el script de puntuación `score.py` usado por el servicio web para la inferencia.

Debe incluir dos funciones necesarias en el script de puntuación:

* La función `init()`, que normalmente carga el modelo en un objeto global. Esta función solo se ejecuta una vez cuando se inicia el contenedor de Docker.
* La función `run(input_data)` usa el modelo para predecir un valor basado en los datos de entrada. Los datos de entrada y salida de la ejecución suelen usan el formato JSON para la serialización y deserialización, pero se admiten otros formatos. La función captura las claves públicas basadas en cifrado homomórfico cargadas por el llamador del servicio.

```python
%%writefile score.py
import json
import os
import pickle
import joblib
from azure.storage.blob import BlobServiceClient, BlobClient, ContainerClient, PublicAccess
from encrypted.inference.eiserver import EIServer

def init():
    global model
    # AZUREML_MODEL_DIR is an environment variable created during deployment.
    # It is the path to the model folder (./azureml-models/$MODEL_NAME/$VERSION)
    # For multiple models, it points to the folder containing all deployed models (./azureml-models)
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

    global server
    server = EIServer(model.coef_, model.intercept_, verbose=True)

def run(raw_data):

    json_properties = json.loads(raw_data)

    key_id = json_properties['key_id']
    conn_str = json_properties['conn_str']
    container = json_properties['container']
    data = json_properties['data']

    # download the public keys from blob storage
    blob_service_client = BlobServiceClient.from_connection_string(conn_str=conn_str)
    blob_client = blob_service_client.get_blob_client(container=container, blob=key_id)
    public_keys = blob_client.download_blob().readall()

    result = {}
    # make prediction
    result = server.predict(data, public_keys)

    # you can return any data type as long as it is JSON-serializable
    return result
```

### <a name="create-configuration-file"></a>Creación de un archivo de configuración

Cree un archivo de configuración de implementación y especifique el número de CPU y gigabytes de RAM necesario para el contenedor de ACI. Aunque depende de su modelo, el valor predeterminado de 1 núcleo y 1 gigabyte de RAM suele ser suficiente para muchos modelos. Si más adelante cree que necesita más, tendrá que volver a crear la imagen y volver a implementar el servicio.

```python
from azureml.core.webservice import AciWebservice

aciconfig = AciWebservice.deploy_configuration(cpu_cores=1, 
                                               memory_gb=1, 
                                               tags={"data": "MNIST",  "method" : "sklearn"}, 
                                               description='Encrypted Predict MNIST with sklearn + SEAL')
```

### <a name="deploy-to-azure-container-instances"></a>Implementación en Azure Container Instances

Tiempo estimado para completar el tutorial: **entre 2 y 5 minutos**

Configuración de la imagen e implementación. El código siguiente realiza estos pasos:

1. Crea el objeto de entorno que contiene las dependencias que necesita el modelo mediante el archivo de entorno (`myenv.yml`).
1. Crea una configuración de inferencia necesaria para implementar el modelo como un servicio web mediante:
   * El archivo de puntuación (`score.py`).
   * Objeto de entorno creado en el paso anterior.
1. Implementa el modelo en el contenedor de ACI.
1. Obtenga el punto de conexión HTTP del servicio web.

```python
%%time
from azureml.core.webservice import Webservice
from azureml.core.model import InferenceConfig
from azureml.core.environment import Environment
from azureml.core import Workspace
from azureml.core.model import Model

ws = Workspace.from_config()
model = Model(ws, 'sklearn_mnist')

myenv = Environment.get(workspace=ws, name="tutorial-env")
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)

service = Model.deploy(workspace=ws,
                       name='sklearn-encrypted-mnist-svc',
                       models=[model],
                       inference_config=inference_config,
                       deployment_config=aciconfig)

service.wait_for_deployment(show_output=True)
```

Obtenga punto de conexión HTTP del servicio web de puntuación, que acepta llamadas de cliente REST. Este punto de conexión se puede compartir con todo aquel que desee probar el servicio web, o se puede integrar en una aplicación.

```python
print(service.scoring_uri)
```

## <a name="prepare-test-data"></a>Preparación de los datos de prueba

1. Descargue los datos de prueba. En este caso, se guarda en un directorio denominado *data*.

    ```python
    import os
    from azureml.core import Dataset
    from azureml.opendatasets import MNIST
    
    data_folder = os.path.join(os.getcwd(), 'data')
    os.makedirs(data_folder, exist_ok=True)
    
    mnist_file_dataset = MNIST.get_file_dataset()
    mnist_file_dataset.download(data_folder, overwrite=True)
    ```

1. Cargue los datos de prueba del directorio *data*.

    ```python
    from utils import load_data
    import os
    import glob
    
    data_folder = os.path.join(os.getcwd(), 'data')
    # note we also shrink the intensity values (X) from 0-255 to 0-1. This helps the neural network converge faster
    X_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-images-idx3-ubyte.gz"), recursive=True)[0], False) / 255.0
    y_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-labels-idx1-ubyte.gz"), recursive=True)[0], True).reshape(-1)
    ```

## <a name="make-encrypted-predictions"></a>Realizar predicciones cifradas.

Use el conjunto de datos de prueba con el modelo para obtener predicciones.

Para realizar predicciones cifradas:

1. Cree un nuevo elemento `EILinearRegressionClient` (un cliente basado en cifrado homomórfico) y claves públicas.

    ```python
    from encrypted.inference.eiclient import EILinearRegressionClient

    # Create a new Encrypted inference client and a new secret key.
    edp = EILinearRegressionClient(verbose=True)

    public_keys_blob, public_keys_data = edp.get_public_keys()
    ```

1. Cargue las claves públicas generadas por el cifrado homomórfico en el almacén de blobs predeterminado del área de trabajo. Esto le permitirá compartir las claves con el servidor de inferencia.

    ```python
    import azureml.core
    from azureml.core import Workspace, Datastore
    import os

    ws = Workspace.from_config()

    datastore = ws.get_default_datastore()
    container_name=datastore.container_name

    # Create a local file and write the keys to it
    public_keys = open(public_keys_blob, "wb")
    public_keys.write(public_keys_data)
    public_keys.close()

    # Upload the file to blob store
    datastore.upload_files([public_keys_blob])

    # Delete the local file
    os.remove(public_keys_blob)
    ```

1. Cifre los datos de prueba.

    ```python
    #choose any one sample from the test data 
    sample_index = 1

    #encrypt the data
    raw_data = edp.encrypt(X_test[sample_index])

    ```

1. Use la API `run` del SDK para invocar el servicio y proporcionar el conjunto de datos de prueba al modelo para obtener predicciones. Tendremos que enviar la cadena de conexión a Blob Storage donde se cargaron las claves públicas.

    ```python
    import json
    from azureml.core import Webservice

    service = Webservice(ws, 'sklearn-encrypted-mnist-svc')

    #pass the connection string for blob storage to give the server access to the uploaded public keys 
    conn_str_template = 'DefaultEndpointsProtocol={};AccountName={};AccountKey={};EndpointSuffix=core.windows.net'
    conn_str = conn_str_template.format(datastore.protocol, datastore.account_name, datastore.account_key)

    #build the json 
    data = json.dumps({"data": raw_data, "key_id" : public_keys_blob, "conn_str" : conn_str, "container" : container_name })
    data = bytes(data, encoding='ASCII')

    print ('Making an encrypted inference web service call ')
    eresult = service.run(input_data=data)

    print ('Received encrypted inference results')
    ```

1. Use el cliente de para descifrar los resultados.

    ```python
    import numpy as np

    results = edp.decrypt(eresult)

    print ('Decrypted the results ', results)

    #Apply argmax to identify the prediction result
    prediction = np.argmax(results)

    print ( ' Prediction : ', prediction)
    print ( ' Actual Label : ', y_test[sample_index])
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Elimine el servicio web creado en este ejemplo:

```python
service.delete()
```

Si ya no va a usar los recursos de Azure que ha creado, elimínelos. Esto le evitará cargos por recursos no utilizados que todavía se están ejecutando. Consulte esta guía sobre cómo [limpiar recursos](how-to-manage-workspace.md#clean-up-resources) para obtener más información.
