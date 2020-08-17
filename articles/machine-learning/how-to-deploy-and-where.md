---
title: Cómo y dónde implementar los modelos
titleSuffix: Azure Machine Learning
description: Aprenda cómo y dónde implementar sus modelos de Azure Machine Learning, incluidos Azure Container Instances, Azure Kubernetes Service, Azure IoT Edge y matrices de puertas programables por campo.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 07/08/2020
ms.custom: seoapril2019, tracking-python
ms.openlocfilehash: ee116d668b9c351ecf5b130a39e418a3da8fc053
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86536392"
---
# <a name="deploy-models-with-azure-machine-learning"></a>Implementación de modelos con Azure Machine Learning
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Aprenda a implementar el modelo de Machine Learning como un servicio web en la nube de Azure o en dispositivos de Azure IoT Edge.

El flujo de trabajo es siempre parecido independientemente de [donde implemente](#target) el modelo:

1. Registre el modelo.
1. Prepárese para la implementación. (Especifique los recursos, el uso y el destino de proceso).
1. Implemente el modelo en el destino de proceso.
1. Pruebe el modelo implementado, también denominado servicio web.

Para más información sobre los conceptos implicados en el flujo de trabajo de implementación, consulte [Administración, implementación y supervisión de modelos con Azure Machine Learning](concept-model-management-and-deployment.md).

## <a name="prerequisites"></a>Requisitos previos

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

- Un modelo. Si no tiene un modelo entrenado, puede usar el modelo y los archivos de dependencia que se proporcionan en [este tutorial](https://aka.ms/azml-deploy-cloud).

- La [extensión de la CLI de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md), el [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py) o la [extensión de Visual Studio Code para Azure Machine Learning](tutorial-setup-vscode-extension.md).

## <a name="connect-to-your-workspace"></a>Conexión con su área de trabajo

En el código siguiente se muestra cómo conectarse a un área de trabajo de Azure Machine Learning con la información almacenada en caché del entorno de desarrollo local:

+ **Uso del SDK**

   ```python
   from azureml.core import Workspace
   ws = Workspace.from_config(path=".file-path/ws_config.json")
   ```

  Para más información sobre el uso del SDK para conectarse a un área de trabajo, consulte la documentación del [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py#workspace).

+ **Uso de la CLI**

   Al usar la CLI, use el parámetro `-w` o `--workspace-name` para especificar el área de trabajo para el comando.

+ **Uso de Visual Studio Code**

   Cuando use Visual Studio Code, seleccione el área de trabajo mediante una interfaz gráfica. Para más información, consulte [Implementación y administración de modelos](how-to-manage-resources-vscode.md#endpoints) en la documentación de la extensión de Visual Studio Code.

## <a name="register-your-model"></a><a id="registermodel"></a> Registro del modelo

Un modelo registrado es un contenedor lógico para uno o varios archivos que componen el modelo. Por ejemplo, si tiene un modelo que se almacena en varios archivos, puede registrarlos como un único modelo en el área de trabajo. Después de registrar los archivos, puede descargar o implementar el modelo registrado y recibir todos los archivos que registró.

> [!TIP]
> Al registrar un modelo, se proporciona una ruta de acceso de una ubicación en la nube (desde una ejecución de entrenamiento) o de un directorio local. Esta ruta de acceso es solo para localizar los archivos que se van a cargar como parte del proceso de registro. No es necesario que coincida con la ruta de acceso que se usa en el script de entrada. Para obtener más información, consulte [Locate model files in your entry script](#load-model-files-in-your-entry-script) (Localización de archivos de modelo en el script de entrada).

Los modelos de Machine Learning se registran en el área de trabajo de Azure Machine Learning. El modelo puede proceder de Azure Machine Learning o de otro lugar. Al registrar un modelo, opcionalmente, puede proporcionar metadatos sobre el modelo. Los diccionarios `tags` y `properties` que se aplican a un registro de modelo se pueden usar para filtrar modelos.

En los ejemplos siguientes se muestra cómo registrar un modelo.

### <a name="register-a-model-from-an-experiment-run"></a>Registro de un modelo a partir de una ejecución de experimento

En los fragmentos de código de esta sección se muestra cómo registrar un modelo a partir de una ejecución de entrenamiento:

> [!IMPORTANT]
> Para usar estos fragmentos de código, necesita haber realizado previamente una ejecución de entrenamiento y tener acceso al objeto `Run` (ejemplo del SDK) o al valor de identificador de ejecución (ejemplo de la CLI). Para más información sobre los modelos de entrenamiento, consulte [Configuración y uso de destinos de proceso para el entrenamiento del modelo](how-to-set-up-training-targets.md).

+ **Uso del SDK**

  Al usar el SDK para entrenar un modelo, puede recibir un objeto [Run](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run.run?view=azure-ml-py) o [AutoMLRun](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun), en función de cómo haya entrenado el modelo. Cada objeto se puede usar para registrar un modelo creado por una ejecución del experimento.

  + Registre un modelo desde un objeto `azureml.core.Run`:
 
    ```python
    model = run.register_model(model_name='sklearn_mnist',
                               tags={'area': 'mnist'},
                               model_path='outputs/sklearn_mnist_model.pkl')
    print(model.name, model.id, model.version, sep='\t')
    ```

    El parámetro `model_path` hace referencia a la ubicación del modelo en la nube. En este ejemplo, se usa la ruta de acceso de un único archivo. Para incluir varios archivos en el registro del modelo, establezca `model_path` en la ruta de acceso de una carpeta que contiene los archivos. Para obtener más información, consulte la documentación [Run.register_model](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run.run?view=azure-ml-py#register-model-model-name--model-path-none--tags-none--properties-none--model-framework-none--model-framework-version-none--description-none--datasets-none--sample-input-dataset-none--sample-output-dataset-none--resource-configuration-none----kwargs-).

  + Registre un modelo desde un objeto `azureml.train.automl.run.AutoMLRun`:

    ```python
        description = 'My AutoML Model'
        model = run.register_model(description = description,
                                   tags={'area': 'mnist'})

        print(run.model_id)
    ```

    En este ejemplo, los parámetros `metric` y `iteration`, no se especifican, por lo que se registrará la iteración con la mejor métrica principal. Se utiliza el valor `model_id` devuelto de la ejecución en lugar de un nombre de modelo.

    Para más información, consulte la documentación de [AutoMLRun.register_model](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun#register-model-model-name-none--description-none--tags-none--iteration-none--metric-none-).

+ **Uso de la CLI**

  ```azurecli-interactive
  az ml model register -n sklearn_mnist  --asset-path outputs/sklearn_mnist_model.pkl  --experiment-name myexperiment --run-id myrunid --tag area=mnist
  ```

  [!INCLUDE [install extension](../../includes/machine-learning-service-install-extension.md)]

  El parámetro `--asset-path` hace referencia a la ubicación del modelo en la nube. En este ejemplo, se usa la ruta de acceso de un único archivo. Para incluir varios archivos en el registro del modelo, establezca `--asset-path` en la ruta de acceso de una carpeta que contiene los archivos.

+ **Uso de Visual Studio Code**

  Registre los modelos usando todas las carpetas o archivos de modelo mediante la extensión de [Visual Studio Code](tutorial-train-deploy-image-classification-model-vscode.md#deploy-the-model).

### <a name="register-a-model-from-a-local-file"></a>Registrar un modelo desde un archivo local

Puede registrar un modelo proporcionando la ruta de acceso local de dicho modelo. Puede proporcionar la ruta de acceso de una carpeta o un único archivo. Puede usar este método para registrar los modelos entrenados con Azure Machine Learning y, luego, descargados. También puede usar este método para registrar modelos entrenados fuera de Azure Machine Learning.

[!INCLUDE [trusted models](../../includes/machine-learning-service-trusted-model.md)]

+ **Uso del SDK y ONNX**

    ```python
    import os
    import urllib.request
    from azureml.core.model import Model
    # Download model
    onnx_model_url = "https://www.cntk.ai/OnnxModels/mnist/opset_7/mnist.tar.gz"
    urllib.request.urlretrieve(onnx_model_url, filename="mnist.tar.gz")
    os.system('tar xvzf mnist.tar.gz')
    # Register model
    model = Model.register(workspace = ws,
                            model_path ="mnist/model.onnx",
                            model_name = "onnx_mnist",
                            tags = {"onnx": "demo"},
                            description = "MNIST image classification CNN from ONNX Model Zoo",)
    ```

  Para incluir varios archivos en el registro del modelo, establezca `model_path` en la ruta de acceso de una carpeta que contiene los archivos.

+ **Uso de la CLI**

  ```azurecli-interactive
  az ml model register -n onnx_mnist -p mnist/model.onnx
  ```

  Para incluir varios archivos en el registro del modelo, establezca `-p` en la ruta de acceso de una carpeta que contiene los archivos.

**Tiempo estimado**: Aproximadamente 10 segundos.

Para más información, consulte la documentación de la [clase Model](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py).

Para más información sobre cómo trabajar con modelos entrenados al margen de Azure Machine Learning, consulte el artículo sobre cómo [implementar un modelo existente](how-to-deploy-existing-model.md).

<a name="target"></a>

## <a name="single-versus-multi-model-endpoints"></a>Puntos de conexión de modelos únicos o múltiples
Azure ML admite la implementación de modelos únicos o múltiples tras un único punto de conexión.

Los puntos de conexión de modelos múltiples usan un contenedor compartido para hospedar varios modelos. Esto ayuda a reducir los costos de sobrecarga, mejora la utilización y permite encadenar módulos en conjunto. Los modelos que se especifican en el script de implementación se montan y se ponen a disposición en el disco del contenedor de servicio: se pueden cargar en memoria a petición y puntuar en función del modelo específico que se solicita en el momento de la puntuación.

Para ver un ejemplo de E2E que muestra cómo usar varios modelos detrás de un solo punto de conexión en contenedor, consulte [aquí](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/deployment/deploy-multi-model).

## <a name="prepare-to-deploy"></a>Preparación de la actualización

Para implementar el modelo como un servicio, necesita los siguientes componentes:

* **Definición del entorno de inferencia**. Este entorno encapsula las dependencias necesarias para ejecutar el modelo para la inferencia.
* **Definición del código de puntuación**. Este script acepta solicitudes, puntúa las solicitudes mediante el modelo y devuelve los resultados.
* **Definición de la configuración de inferencia**. La configuración de inferencia especifica la configuración del entorno, el script de entrada y otros componentes necesarios para ejecutar el modelo como un servicio.

Una vez que tenga los componentes necesarios, puede generar un perfil del servicio que se creará como resultado de la implementación del modelo para comprender sus requisitos de CPU y memoria.

### <a name="1-define-inference-environment"></a>1. Definición del entorno de inferencia.

Una configuración de inferencia describe cómo establecer el servicio web que contiene el modelo. Se usa más adelante, cuando se implementa el modelo.

La configuración de inferencia usa entornos Azure Machine Learning para definir las dependencias de software necesarias para la implementación. Los entornos permiten crear, administrar y reutilizar las dependencias de software necesarias para el entrenamiento y la implementación. Puede crear un entorno a partir de archivos de dependencia personalizados o usar uno de los entornos de Azure Machine Learning mantenidos. El siguiente código YAML es un ejemplo de un archivo de dependencias de Conda para inferencia. Tenga en cuenta que debe indicar valores predeterminados de azureml con versión > = 1.0.45 como una dependencia de PIP, ya que contiene la funcionalidad necesaria para hospedar el modelo como un servicio web. Si desea usar la generación automática de esquemas, el script de entrada debe importar también los paquetes `inference-schema`.

```YAML
name: project_environment
dependencies:
  - python=3.6.2
  - scikit-learn=0.20.0
  - pip:
      # You must list azureml-defaults as a pip dependency
    - azureml-defaults>=1.0.45
    - inference-schema[numpy-support]
```

> [!IMPORTANT]
> Si la dependencia está disponible tanto a través de Conda como de PIP (de PyPi), Microsoft recomienda usar la versión de Conda, ya que los paquetes de Conda normalmente incluyen archivos binarios pregenerados que hacen que la instalación sea más confiable.
>
> Para obtener más información, consulte la [descripción de Conda y PIP](https://www.anaconda.com/understanding-conda-and-pip/).
>
> Para comprobar si la dependencia está disponible a través de Conda, use el comando `conda search <package-name>` o los índices de paquetes en [https://anaconda.org/anaconda/repo](https://anaconda.org/anaconda/repo) y [https://anaconda.org/conda-forge/repo](https://anaconda.org/conda-forge/repo).

Puede usar el archivo de dependencias para crear un objeto de entorno y guardarlo en el área de trabajo para su uso futuro:

```python
from azureml.core.environment import Environment
myenv = Environment.from_conda_specification(name = 'myenv',
                                             file_path = 'path-to-conda-specification-file'
myenv.register(workspace=ws)
```

### <a name="2-define-scoring-code"></a><a id="script"></a> 2. Definición del código de puntuación

El script de entrada recibe los datos enviados a un servicio web implementado y los pasa al modelo. A continuación, toma la respuesta devuelta por el modelo y la envía al cliente. *El script es específico para su modelo*. Debe entender los datos que el modelo espera y devuelve.

El script contiene dos funciones que cargan y ejecutan el modelo:

* `init()`: normalmente, esta función carga el modelo en un objeto global. Esta función solo se ejecuta una vez cuando se inicia el contenedor de Docker para el servicio web.

* `run(input_data)`: esta función usa el modelo para predecir un valor basado en los datos de entrada. Las entradas y salidas de la ejecución suelen usar el formato JSON para la serialización y deserialización. También puede trabajar con datos binarios sin formato. Puede transformar los datos antes de enviarlos al modelo o antes de devolverlos al cliente.

#### <a name="load-model-files-in-your-entry-script"></a>Carga de archivos de modelo en el script de entrada

Hay dos maneras de buscar modelos en el script de entrada:
* `AZUREML_MODEL_DIR`: variable de entorno que contiene la ruta de acceso a la ubicación del modelo.
* `Model.get_model_path`: API que devuelve la ruta de acceso al archivo de modelo mediante el nombre del modelo registrado.

##### <a name="azureml_model_dir"></a>AZUREML_MODEL_DIR

AZUREML_MODEL_DIR es una variable de entorno que se crea durante la implementación del servicio. Puede usar esta variable de entorno para buscar la ubicación de los modelos implementados.

En la tabla siguiente se describe el valor de AZUREML_MODEL_DIR en función del número de modelos implementados:

| Implementación | Valor de la variable de entorno |
| ----- | ----- |
| Modelo único | Ruta de acceso a la carpeta que contiene el modelo. |
| Varios modelos | Ruta de acceso a la carpeta que contiene todos los modelos. Los modelos se encuentran por nombre y versión en esta carpeta (`$MODEL_NAME/$VERSION`). |

Durante el registro y la implementación de un modelo, este se coloca en la ruta de acceso AZUREML_MODEL_DIR y se conserva su nombre de archivo original.

Para obtener la ruta de acceso a un archivo de modelo en el script de entrada, combine la variable de entorno con la ruta de acceso al archivo que busca.

**Ejemplo de modelo único**
```python
# Example when the model is a file
model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_regression_model.pkl')

# Example when the model is a folder containing a file
file_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'my_model_folder', 'sklearn_regression_model.pkl')
```

**Ejemplo de varios modelos**

En este escenario, se registran dos modelos con el área de trabajo:

* `my_first_model`: contiene un archivo (`my_first_model.pkl`) y solo hay una versión (`1`).
* `my_second_model`: contiene un archivo (`my_second_model.pkl`) y hay dos versiones; `1` y `2`.

Cuando se implementó el servicio, ambos modelos se proporcionan en la operación de implementación:

```python
first_model = Model(ws, name="my_first_model", version=1)
second_model = Model(ws, name="my_second_model", version=2)
service = Model.deploy(ws, "myservice", [first_model, second_model], inference_config, deployment_config)
```

En la imagen de Docker que hospeda el servicio, la variable de entorno `AZUREML_MODEL_DIR` contiene el directorio donde se encuentran los modelos.
En este directorio, cada uno de los modelos se encuentra en una ruta de acceso al directorio de `MODEL_NAME/VERSION`. Donde `MODEL_NAME` es el nombre del modelo registrado y `VERSION` es la versión del modelo. Los archivos que componen el modelo registrado se almacenan en estos directorios.

En este ejemplo, las rutas de acceso serían `$AZUREML_MODEL_DIR/my_first_model/1/my_first_model.pkl` y `$AZUREML_MODEL_DIR/my_second_model/2/my_second_model.pkl`.


```python
# Example when the model is a file, and the deployment contains multiple models
first_model_name = 'my_first_model'
first_model_version = '1'
first_model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), first_model_name, first_model_version, 'my_first_model.pkl')
second_model_name = 'my_second_model'
second_model_version = '2'
second_model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), second_model_name, second_model_version, 'my_second_model.pkl')
```

##### <a name="get_model_path"></a>get_model_path

Cuando registra un modelo, puede proporcionar un nombre de modelo que se usa para administrar este en el registro. Usará este nombre con el método [Model.get_model_path()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#get-model-path-model-name--version-none---workspace-none-) para recuperar la ruta de acceso del archivo del modelo en el sistema de archivos local. Si registra una carpeta o una colección de archivos, esta API devolverá la ruta de acceso del directorio que contiene esos archivos.

Al registrar un modelo, se le asigna un nombre. El nombre se corresponde con la ubicación donde se coloque el modelo, ya sea localmente o durante la implementación del servicio.

#### <a name="optional-define-model-web-service-schema"></a>(Opcional) Definición del esquema del servicio web del modelo

Para generar automáticamente un esquema para el servicio web, proporcione un ejemplo de la entrada y salida del constructor de uno de los objetos de tipo definidos. El tipo y ejemplo se usan para crear automáticamente el esquema. Azure Machine Learning creará, a continuación, una especificación de [OpenAPI](https://swagger.io/docs/specification/about/) (Swagger) para el servicio web durante la implementación.

Actualmente se admiten estos tipos:

* `pandas`
* `numpy`
* `pyspark`
* Objeto estándar de Python

Para usar la generación de esquemas, incluya el paquete `inference-schema` de código abierto en el archivo de dependencias. Para más información sobre este paquete, consulte [https://github.com/Azure/InferenceSchema](https://github.com/Azure/InferenceSchema). Defina los formatos de ejemplo de entrada y salida de las variables `input_sample` y `output_sample`, que representan los formatos de solicitud y respuesta del servicio web. Use estos ejemplos en los decoradores de entrada y salida de la función `run()`. El siguiente ejemplo de Scikit-learn usa generación de esquemas.

##### <a name="example-entry-script"></a>Script de entrada de ejemplo

En el siguiente ejemplo se muestra cómo aceptar y devolver datos JSON:

```python
#Example: scikit-learn and Swagger
import json
import numpy as np
import os
from sklearn.externals import joblib
from sklearn.linear_model import Ridge

from inference_schema.schema_decorators import input_schema, output_schema
from inference_schema.parameter_types.numpy_parameter_type import NumpyParameterType


def init():
    global model
    # AZUREML_MODEL_DIR is an environment variable created during deployment. Join this path with the filename of the model file.
    # It holds the path to the directory that contains the deployed model (./azureml-models/$MODEL_NAME/$VERSION).
    # If there are multiple models, this value is the path to the directory containing all deployed models (./azureml-models).
    # Alternatively: model_path = Model.get_model_path('sklearn_mnist')
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    # Deserialize the model file back into a sklearn model
    model = joblib.load(model_path)


input_sample = np.array([[10, 9, 8, 7, 6, 5, 4, 3, 2, 1]])
output_sample = np.array([3726.995])


@input_schema('data', NumpyParameterType(input_sample))
@output_schema(NumpyParameterType(output_sample))
def run(data):
    try:
        result = model.predict(data)
        # You can return any data type, as long as it is JSON serializable.
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```

##### <a name="power-bi-compatible-endpoint"></a>Punto de conexión compatible con Power BI 

En el ejemplo siguiente se muestra cómo definir datos de entrada como un diccionario `<key: value>` mediante DataFrame. Este método se admite para consumir el servicio Web implementado desde Power BI. ([Obtenga más información sobre cómo consumir el servicio web desde Power BI](https://docs.microsoft.com/power-bi/service-machine-learning-integration)).

```python
import json
import pickle
import numpy as np
import pandas as pd
import azureml.train.automl
from sklearn.externals import joblib
from azureml.core.model import Model

from inference_schema.schema_decorators import input_schema, output_schema
from inference_schema.parameter_types.numpy_parameter_type import NumpyParameterType
from inference_schema.parameter_types.pandas_parameter_type import PandasParameterType


def init():
    global model
    # Replace filename if needed.
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'model_file.pkl')
    # Deserialize the model file back into a sklearn model.
    model = joblib.load(model_path)


input_sample = pd.DataFrame(data=[{
    # This is a decimal type sample. Use the data type that reflects this column in your data.
    "input_name_1": 5.1,
    # This is a string type sample. Use the data type that reflects this column in your data.
    "input_name_2": "value2",
    # This is an integer type sample. Use the data type that reflects this column in your data.
    "input_name_3": 3
}])

# This is an integer type sample. Use the data type that reflects the expected result.
output_sample = np.array([0])

# To indicate that we support a variable length of data input,
# set enforce_shape=False
@input_schema('data', PandasParameterType(input_sample, enforce_shape=False))
@output_schema(NumpyParameterType(output_sample))
def run(data):
    try:
        result = model.predict(data)
        # You can return any data type, as long as it is JSON serializable.
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```

Para más ejemplos, consulte los siguientes scripts:

* [PyTorch](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/pytorch)
* [TensorFlow](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/tensorflow)
* [Keras](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-keras)
* [AutoML](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning/classification-bank-marketing-all-features)
* [ONNX](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/onnx/)
* [Binary Data](#binary)
* [CORS](#cors)

### <a name="3-define-inference-configuration"></a><a id="script"></a> 3. Definir la configuración de inferencia
    
En el ejemplo siguiente se muestra cómo cargar un entorno desde el área de trabajo y luego usarlo con la configuración de inferencia:

```python
from azureml.core.environment import Environment
from azureml.core.model import InferenceConfig


myenv = Environment.get(workspace=ws, name='myenv', version='1')
inference_config = InferenceConfig(entry_script='path-to-score.py',
                                   environment=myenv)
```

Para obtener más información sobre los entornos, consulte el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md).

Para más información sobre la configuración de inferencia, consulte la documentación de la clase [InferenceConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig?view=azure-ml-py).

Para información sobre el uso de una imagen personalizada de Docker con una configuración de inferencia, consulte [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md).

#### <a name="cli-example-of-inferenceconfig"></a>Ejemplo de la CLI de InferenceConfig

[!INCLUDE [inference config](../../includes/machine-learning-service-inference-config.md)]

El comando siguiente muestra cómo implementar un modelo mediante la CLI:

```azurecli-interactive
az ml model deploy -n myservice -m mymodel:1 --ic inferenceconfig.json
```

En este ejemplo, la configuración especifica la siguiente configuración:

* Que el modelo requiere Python.
* El [script de entrada](#script), que se usa para controlar las solicitudes web que se envían al servicio implementado
* El archivo de Conda que describe los paquetes de Python necesarios para realizar la inferencia.

Para información sobre el uso de una imagen personalizada de Docker con una configuración de inferencia, consulte [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md).

### <a name="4-optional-profile-your-model-to-determine-resource-utilization"></a><a id="profilemodel"></a> 4. (Opcional) Generación de perfiles del modelo para determinar el uso de recursos

Una vez que haya registrado el modelo y preparado los otros componentes necesarios para su implementación, puede determinar la CPU y la memoria que necesitará el servicio implementado. La generación de perfiles prueba el servicio que ejecuta el modelo y devuelve información como el uso de la CPU, el uso de memoria y la latencia de respuesta. También proporciona una recomendación para la CPU y la memoria en función del uso de recursos.

Para generar un perfil del modelo, necesitará lo siguiente:
* Un modelo registrado.
* Una configuración de inferencia basada en el script de entrada y en la definición de entorno de inferencia.
* Un conjunto de datos tabular de una sola columna, donde cada fila contiene una cadena que representa datos de solicitud de ejemplo.

> [!IMPORTANT]
> En este momento, solo se admite la generación de perfiles de servicios que esperan que sus datos de solicitud sean una cadena, por ejemplo: JSON serializado en cadena, texto, imagen serializada en cadena, etc. El contenido de cada fila del conjunto de resultados (cadena) se colocará en el cuerpo de la solicitud HTTP y se enviará al servicio que encapsula el modelo para la puntuación.

A continuación se muestra un ejemplo de cómo se puede construir un conjunto de datos de entrada para generar perfiles de un servicio que espera que los datos de solicitudes entrantes contengan JSON serializado. En este caso, creamos un conjunto de datos basado en 100 instancias del mismo contenido de datos de la solicitud. En escenarios reales, se recomienda usar conjuntos de datos más grandes que contengan varias entradas, especialmente si el comportamiento o el uso de recursos del modelo depende de la entrada.

```python
import json
from azureml.core import Datastore
from azureml.core.dataset import Dataset
from azureml.data import dataset_type_definitions

input_json = {'data': [[1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
                       [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]]}
# create a string that can be utf-8 encoded and
# put in the body of the request
serialized_input_json = json.dumps(input_json)
dataset_content = []
for i in range(100):
    dataset_content.append(serialized_input_json)
dataset_content = '\n'.join(dataset_content)
file_name = 'sample_request_data.txt'
f = open(file_name, 'w')
f.write(dataset_content)
f.close()

# upload the txt file created above to the Datastore and create a dataset from it
data_store = Datastore.get_default(ws)
data_store.upload_files(['./' + file_name], target_path='sample_request_data')
datastore_path = [(data_store, 'sample_request_data' +'/' + file_name)]
sample_request_data = Dataset.Tabular.from_delimited_files(
    datastore_path, separator='\n',
    infer_column_types=True,
    header=dataset_type_definitions.PromoteHeadersBehavior.NO_HEADERS)
sample_request_data = sample_request_data.register(workspace=ws,
                                                   name='sample_request_data',
                                                   create_new_version=True)
```

Una vez que tenga listo el conjunto de datos que contiene los datos de solicitud de ejemplo, cree una configuración de inferencia. La configuración de inferencia se basa en score.py y en la definición de entorno. En el siguiente ejemplo le indicaremos cómo crear una configuración de inferencia y ejecutar la generación de perfiles:

```python
from azureml.core.model import InferenceConfig, Model
from azureml.core.dataset import Dataset


model = Model(ws, id=model_id)
inference_config = InferenceConfig(entry_script='path-to-score.py',
                                   environment=myenv)
input_dataset = Dataset.get_by_name(workspace=ws, name='sample_request_data')
profile = Model.profile(ws,
            'unique_name',
            [model],
            inference_config,
            input_dataset=input_dataset)

profile.wait_for_completion(True)

# see the result
details = profile.get_details()
```

El comando siguiente muestra cómo generar el perfil de un modelo mediante la CLI:

```azurecli-interactive
az ml model profile -g <resource-group-name> -w <workspace-name> --inference-config-file <path-to-inf-config.json> -m <model-id> --idi <input-dataset-id> -n <unique-name>
```

> [!TIP]
> Para conservar la información que devuelve la generación de perfiles, use las etiquetas o las propiedades del modelo. El uso de etiquetas o propiedades almacena los datos con el modelo en el registro del modelo. En los siguientes ejemplos se muestra cómo agregar una nueva etiqueta que contiene la información de `requestedCpu` y `requestedMemoryInGb`:
>
> ```python
> model.add_tags({'requestedCpu': details['requestedCpu'],
>                 'requestedMemoryInGb': details['requestedMemoryInGb']})
> ```
>
> ```azurecli-interactive
> az ml model profile -g <resource-group-name> -w <workspace-name> --i <model-id> --add-tag requestedCpu=1 --add-tag requestedMemoryInGb=0.5
> ```

## <a name="deploy-to-target"></a>Implementación en el destino

La implementación usa la configuración de implementación de la configuración de inferencia para implementar los modelos. El proceso de implementación es similar, independientemente del destino de proceso. La implementación en Azure Kubernetes Service (AKS) es ligeramente diferente, ya que debe proporcionar una referencia al clúster de AKS.

### <a name="choose-a-compute-target"></a>Elección de un destino de proceso

Puede usar los siguientes destinos o recursos de proceso para hospedar la implementación del servicio web:

[!INCLUDE [aml-compute-target-deploy](../../includes/aml-compute-target-deploy.md)]

> [!NOTE]
> * ACI solo es adecuado para pequeños modelos con un tamaño inferior a 1 GB. 
> * Se recomienda usar AKS de nodo único para desarrollo y pruebas de modelos más grandes.

### <a name="define-your-deployment-configuration"></a>Definición de la configuración de la implementación

Antes de implementar el modelo, debe definir la configuración de esta. *La configuración de implementación es específica del destino de proceso que hospedará el servicio web.* Por ejemplo, cuando implementa un modelo de forma local, debe especificar el puerto en el que el servicio aceptará las solicitudes. La configuración de la implementación no forma parte del script de entrada. Se usa para definir las características del destino de proceso que hospedará el modelo y el script de entrada.

Es posible que también tenga que crear el recurso de proceso si, por ejemplo, aún no tiene una instancia de Azure Kubernetes Service (AKS) asociada con el área de trabajo.

La tabla siguiente proporciona un ejemplo de creación de una configuración de implementación para cada destino de proceso:

| Destino de proceso | Ejemplo de configuración de implementación |
| ----- | ----- |
| Local | `deployment_config = LocalWebservice.deploy_configuration(port=8890)` |
| Azure Container Instances | `deployment_config = AciWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)` |
| Azure Kubernetes Service | `deployment_config = AksWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)` |

Las clases de servicios web locales, de Azure Container Instances y AKS se puede importar desde `azureml.core.webservice`:

```python
from azureml.core.webservice import AciWebservice, AksWebservice, LocalWebservice
```

### <a name="securing-deployments-with-tls"></a>Protección de implementaciones con TLS

Para obtener más información sobre cómo proteger una implementación de servicio web, vea [Habilitación de TLS e implementación](how-to-secure-web-service.md#enable).

### <a name="local-deployment"></a><a id="local"></a> Implementación local

Para implementar un modelo de forma local, debe tener Docker instalado en la máquina local.

#### <a name="using-the-sdk"></a>Uso del SDK

```python
from azureml.core.webservice import LocalWebservice, Webservice

deployment_config = LocalWebservice.deploy_configuration(port=8890)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config)
service.wait_for_deployment(show_output = True)
print(service.state)
```

Para más información, consulte la documentación de [LocalWebservice](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.local.localwebservice?view=azure-ml-py), [Model.deploy()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) y [Webservice](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.webservice?view=azure-ml-py).

#### <a name="using-the-cli"></a>Uso de la CLI

Para implementar un modelo con la CLI, use el siguiente comando. Reemplace `mymodel:1` por el nombre y la versión del modelo registrado:

```azurecli-interactive
az ml model deploy -m mymodel:1 --ic inferenceconfig.json --dc deploymentconfig.json
```

[!INCLUDE [aml-local-deploy-config](../../includes/machine-learning-service-local-deploy-config.md)]

Para más información, consulte la documentación de [az ml model deploy](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy).

### <a name="understanding-service-state"></a>Descripción del estado del servicio

Durante la implementación del modelo, es posible que vea el cambio de estado del servicio mientras se implementa por completo.

En la tabla siguiente se describen los diferentes estados del servicio:

| Estado de WebService | Descripción | ¿Estado final?
| ----- | ----- | ----- |
| En transición | El servicio está en proceso de implementación. | No |
| Unhealthy (Incorrecto) | El servicio se ha implementado pero actualmente no se puede acceder a él.  | No |
| No programable | El servicio no se puede implementar en este momento debido a la falta de recursos. | No |
| Con error | No se pudo implementar el servicio debido a un error o a un bloqueo. | Sí |
| Healthy | El servicio está en buen estado y el punto de conexión está disponible. | Sí |

### <a name="compute-instance-web-service-devtest"></a><a id="notebookvm"></a> Servicio web de instancia de proceso (desarrollo y pruebas)

Vea [Implementación de un modelo en instancias de proceso de Azure Machine Learning](how-to-deploy-local-container-notebook-vm.md).

### <a name="azure-container-instances-devtest"></a><a id="aci"></a> Azure Container Instances (desarrollo/pruebas)

Consulte [Implementación en Azure Container Instances](how-to-deploy-azure-container-instance.md).

### <a name="azure-kubernetes-service-devtest-and-production"></a><a id="aks"></a>Azure Kubernetes Service (desarrollo/pruebas y producción)

Consulte [Implementación en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).

### <a name="ab-testing-controlled-rollout"></a>Pruebas A/B (implementación controlada)
Para obtener más información, consulte [Lanzamiento controlado de modelos de ML](how-to-deploy-azure-kubernetes-service.md#deploy-models-to-aks-using-controlled-rollout-preview).

## <a name="consume-web-services"></a>Consumo de servicios web

Cada servicio Web implementado proporciona un punto de conexión de REST, por lo que puede crear aplicaciones cliente en cualquier lenguaje de programación.
Si habilitó la autenticación basada en claves para su servicio, debe proporcionar una clave de servicio como un token en el encabezado de solicitud.
Si habilitó la autenticación basada en token para su servicio, debe proporcionar un Azure Machine Learning JSON Web Token (JWT) como un token de portador en el encabezado de solicitud. 

La principal diferencia es que las **claves de son estáticas y se pueden volver a generar manualmente**y los **tokens deben actualizarse tras la expiración**. La autenticación basada en claves es compatible con la instancia de Azure Container Instance y los servicios web implementados por el servicio Kubernetes de Azure, y la autenticación basada en token está **solo** disponible para las implementaciones del servicio Kubernetes de Azure. Consulte el [procedimiento](how-to-setup-authentication.md#web-service-authentication) en la autenticación para más información y ejemplos de código específicos.

> [!TIP]
> Puede recuperar el documento JSON del esquema después de implementar el servicio. Use la [propiedad swagger_uri](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.local.localwebservice?view=azure-ml-py#swagger-uri) del servicio web implementado (por ejemplo, `service.swagger_uri`) para obtener el URI del archivo Swagger del servicio web local.

### <a name="request-response-consumption"></a>Consumo de solicitud-respuesta

Este es un ejemplo de cómo invocar el servicio en Python:
```python
import requests
import json

headers = {'Content-Type': 'application/json'}

if service.auth_enabled:
    headers['Authorization'] = 'Bearer '+service.get_keys()[0]
elif service.token_auth_enabled:
    headers['Authorization'] = 'Bearer '+service.get_token()[0]

print(headers)

test_sample = json.dumps({'data': [
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
]})

response = requests.post(
    service.scoring_uri, data=test_sample, headers=headers)
print(response.status_code)
print(response.elapsed)
print(response.json())
```

Para más información, consulte [Consumir un modelo de Azure Machine Learning que está implementado como un servicio web](how-to-consume-web-service.md).

### <a name="web-service-schema-openapi-specification"></a>Esquema de servicio web (especificación de OpenAPI)

Si ha usado la generación automática de esquemas con la implementación, puede obtener la dirección de la especificación de OpenAPI para el servicio mediante la [propiedad swagger_uri](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.local.localwebservice?view=azure-ml-py#swagger-uri). (Por ejemplo, `print(service.swagger_uri)`). Use una solicitud GET o abra el URI en un explorador para recuperar la especificación.

El siguiente documento JSON es un ejemplo de un esquema (especificación de OpenAPI) generado para una implementación:

```json
{
    "swagger": "2.0",
    "info": {
        "title": "myservice",
        "description": "API specification for Azure Machine Learning myservice",
        "version": "1.0"
    },
    "schemes": [
        "https"
    ],
    "consumes": [
        "application/json"
    ],
    "produces": [
        "application/json"
    ],
    "securityDefinitions": {
        "Bearer": {
            "type": "apiKey",
            "name": "Authorization",
            "in": "header",
            "description": "For example: Bearer abc123"
        }
    },
    "paths": {
        "/": {
            "get": {
                "operationId": "ServiceHealthCheck",
                "description": "Simple health check endpoint to ensure the service is up at any given point.",
                "responses": {
                    "200": {
                        "description": "If service is up and running, this response will be returned with the content 'Healthy'",
                        "schema": {
                            "type": "string"
                        },
                        "examples": {
                            "application/json": "Healthy"
                        }
                    },
                    "default": {
                        "description": "The service failed to execute due to an error.",
                        "schema": {
                            "$ref": "#/definitions/ErrorResponse"
                        }
                    }
                }
            }
        },
        "/score": {
            "post": {
                "operationId": "RunMLService",
                "description": "Run web service's model and get the prediction output",
                "security": [
                    {
                        "Bearer": []
                    }
                ],
                "parameters": [
                    {
                        "name": "serviceInputPayload",
                        "in": "body",
                        "description": "The input payload for executing the real-time machine learning service.",
                        "schema": {
                            "$ref": "#/definitions/ServiceInput"
                        }
                    }
                ],
                "responses": {
                    "200": {
                        "description": "The service processed the input correctly and provided a result prediction, if applicable.",
                        "schema": {
                            "$ref": "#/definitions/ServiceOutput"
                        }
                    },
                    "default": {
                        "description": "The service failed to execute due to an error.",
                        "schema": {
                            "$ref": "#/definitions/ErrorResponse"
                        }
                    }
                }
            }
        }
    },
    "definitions": {
        "ServiceInput": {
            "type": "object",
            "properties": {
                "data": {
                    "type": "array",
                    "items": {
                        "type": "array",
                        "items": {
                            "type": "integer",
                            "format": "int64"
                        }
                    }
                }
            },
            "example": {
                "data": [
                    [ 10, 9, 8, 7, 6, 5, 4, 3, 2, 1 ]
                ]
            }
        },
        "ServiceOutput": {
            "type": "array",
            "items": {
                "type": "number",
                "format": "double"
            },
            "example": [
                3726.995
            ]
        },
        "ErrorResponse": {
            "type": "object",
            "properties": {
                "status_code": {
                    "type": "integer",
                    "format": "int32"
                },
                "message": {
                    "type": "string"
                }
            }
        }
    }
}
```

Para más información, consulte la [especificación de OpenAPI](https://swagger.io/specification/).

Para obtener una utilidad que puede crear bibliotecas de cliente a partir de la especificación, consulte [swagger-codegen](https://github.com/swagger-api/swagger-codegen).

### <a name="batch-inference"></a><a id="azuremlcompute"></a> Inferencia por lotes
Los destinos de proceso de Azure Machine Learning se crean y administran mediante Azure Machine Learning. Estos se pueden usar para la predicción por lotes en canalizaciones de Azure Machine Learning.

Para ver un tutorial sobre la inferencia por lotes con Proceso de Azure Machine Learning, consulte [Cómo ejecutar predicciones por lotes](tutorial-pipeline-batch-scoring-classification.md).

### <a name="iot-edge-inference"></a><a id="iotedge"></a> Inferencia de IoT Edge
La compatibilidad con implementaciones en el perímetro está en versión preliminar. Para más información, consulte [Implementación de Azure Machine Learning como un módulo de IoT Edge](https://docs.microsoft.com/azure/iot-edge/tutorial-deploy-machine-learning).


## <a name="update-web-services"></a><a id="update"></a> Actualización de servicios web

[!INCLUDE [aml-update-web-service](../../includes/machine-learning-update-web-service.md)]

## <a name="continuously-deploy-models"></a>Implementación de modelos de forma continuada

Puede implementar continuamente modelos mediante la extensión de Machine Learning para [Azure DevOps](https://azure.microsoft.com/services/devops/). Puede usar la extensión de Machine Learning para Azure DevOps para desencadenar una canalización de implementación cuando se registra un nuevo modelo de aprendizaje automático en el área de trabajo de Azure Machine Learning.

1. Regístrese en [Azure Pipelines](https://docs.microsoft.com/azure/devops/pipelines/get-started/pipelines-sign-up?view=azure-devops), lo que hace posible la integración continua y la entrega de la aplicación en cualquier plataforma o nube. (Tenga en cuenta que Azure Pipelines no es lo mismo que las [canalizaciones de Machine Learning](concept-ml-pipelines.md#compare)).

1. [Cree un proyecto de Azure DevOps.](https://docs.microsoft.com/azure/devops/organizations/projects/create-project?view=azure-devops)

1. Instale la [extensión de Machine Learning para Azure Pipelines](https://marketplace.visualstudio.com/items?itemName=ms-air-aiagility.vss-services-azureml&targetId=6756afbe-7032-4a36-9cb6-2771710cadc2&utm_source=vstsproduct&utm_medium=ExtHubManageList).

1. Use las conexiones de servicio para configurar una conexión de entidad de servicio para el área de trabajo de Azure Machine Learning para tener acceso a los artefactos. Vaya a configuración del proyecto, seleccione **Conexiones de servicio** y, luego, seleccione **Azure Resource Manager**:

    [![Selección de Azure Resource Manager](media/how-to-deploy-and-where/view-service-connection.png)](media/how-to-deploy-and-where/view-service-connection-expanded.png)

1. En la lista **Nivel de ámbito**, seleccione **AzureMLWorkspace** y luego escriba el resto de los valores:

    ![Selección de AzureMLWorkspace](./media/how-to-deploy-and-where/resource-manager-connection.png)

1. Para implementar continuamente el modelo de aprendizaje automático mediante Azure Pipelines, seleccione **Versión** en Canalizaciones. Agregue un artefacto nuevo y seleccione el artefacto del **modelo de AzureML** y la conexión de servicio que creó en el paso anterior. Seleccione el modelo y la versión para desencadenar una implementación:

    [![Selección del modelo de AzureML](media/how-to-deploy-and-where/enable-modeltrigger-artifact.png)](media/how-to-deploy-and-where/enable-modeltrigger-artifact-expanded.png)

1. Habilite el desencadenador del modelo en el artefacto del modelo. Al activar el desencadenador, cada vez que se registra la versión especificada (es decir, la versión más reciente) de ese modelo en el área de trabajo, se desencadena una canalización de versión de Azure DevOps.

    [![Habilitación del desencadenador de modelo](media/how-to-deploy-and-where/set-modeltrigger.png)](media/how-to-deploy-and-where/set-modeltrigger-expanded.png)

Para más ejemplos y proyectos de muestra, consulte estos repositorios de ejemplo en GitHub:

* [Microsoft/MLOps](https://github.com/Microsoft/MLOps)
* [Microsoft/MLOpsPython](https://github.com/microsoft/MLOpsPython)

## <a name="download-a-model"></a>Descarga de un modelo
Si desea descargar el modelo para usarlo en su propio entorno de ejecución, puede hacerlo con los comandos de SDK/CLI siguientes:

SDK:
```python
model_path = Model(ws,'mymodel').download()
```

CLI:
```azurecli-interactive
az ml model download --model-id mymodel:1 --target-dir model_folder
```

## <a name="preview-no-code-model-deployment"></a>(Versión preliminar) Implementación de modelo sin código

La implementación del modelo sin código se encuentra actualmente en versión preliminar y admite los siguientes marcos de aprendizaje automático:

### <a name="tensorflow-savedmodel-format"></a>Formato Tensorflow SavedModel
Los modelos Tensorflow deben registrarse en **formato de SavedModel** para que funcionen con la implementación de modelos sin código.

Consulte [este vínculo](https://www.tensorflow.org/guide/saved_model) para obtener información sobre cómo crear un formato SavedModel.

```python
from azureml.core import Model

model = Model.register(workspace=ws,
                       model_name='flowers',                        # Name of the registered model in your workspace.
                       model_path='./flowers_model',                # Local Tensorflow SavedModel folder to upload and register as a model.
                       model_framework=Model.Framework.TENSORFLOW,  # Framework used to create the model.
                       model_framework_version='1.14.0',            # Version of Tensorflow used to create the model.
                       description='Flowers model')

service_name = 'tensorflow-flower-service'
service = Model.deploy(ws, service_name, [model])
```

### <a name="onnx-models"></a>Modelos de ONNX

El registro y la implementación del modelo ONNX se admiten en cualquier gráfico de inferencia de ONNX. Los pasos de preprocesamiento y postproceso no se admiten actualmente.

Este es un ejemplo de cómo registrar e implementar un modelo de MNIST ONNX:

```python
from azureml.core import Model

model = Model.register(workspace=ws,
                       model_name='mnist-sample',                  # Name of the registered model in your workspace.
                       model_path='mnist-model.onnx',              # Local ONNX model to upload and register as a model.
                       model_framework=Model.Framework.ONNX ,      # Framework used to create the model.
                       model_framework_version='1.3',              # Version of ONNX used to create the model.
                       description='Onnx MNIST model')

service_name = 'onnx-mnist-service'
service = Model.deploy(ws, service_name, [model])
```

Para puntuar un modelo, consulte [Consumir un modelo de Azure Machine Learning que está implementado como un servicio web](https://docs.microsoft.com/azure/machine-learning/how-to-consume-web-service). Muchos proyectos de ONNX usan archivos protobuf para almacenar de forma compacta datos de entrenamiento y validación, lo que puede hacer que sea difícil saber cuál es el formato de datos esperado por el servicio. Como desarrollador de modelos, debe documentar para los desarrolladores:

* El formato de entrada (JSON o binario)
* La forma y el tipo de los datos de entrada (por ejemplo, una matriz de floats de forma [100,100,3])
* Información de dominio (por ejemplo, para una imagen, el espacio de colores, el orden de los componentes y si los valores están normalizados)

Si usa PyTorch, el artículo [Exportación de modelos de PyTorch a ONNX](https://github.com/onnx/tutorials/blob/master/tutorials/PytorchOnnxExport.ipynb) tiene los detalles sobre la conversión y las limitaciones. 

### <a name="scikit-learn-models"></a>Modelos de Scikit-learn

Se admite la implementación de modelo sin código para todos los tipos de modelos integrados de Scikit-learn.

Este es un ejemplo de cómo registrar e implementar un modelo de sklearn sin código adicional:

```python
from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

model = Model.register(workspace=ws,
                       model_name='my-sklearn-model',                # Name of the registered model in your workspace.
                       model_path='./sklearn_regression_model.pkl',  # Local file to upload and register as a model.
                       model_framework=Model.Framework.SCIKITLEARN,  # Framework used to create the model.
                       model_framework_version='0.19.1',             # Version of scikit-learn used to create the model.
                       resource_configuration=ResourceConfiguration(cpu=1, memory_in_gb=0.5),
                       description='Ridge regression model to predict diabetes progression.',
                       tags={'area': 'diabetes', 'type': 'regression'})
                       
service_name = 'my-sklearn-service'
service = Model.deploy(ws, service_name, [model])
```

NOTA:  Los modelos que admiten predict_proba usarán ese método de forma predeterminada. Para invalidar esta opción para usar la predicción, puede modificar el cuerpo de POST como se indica a continuación:
```python
import json


input_payload = json.dumps({
    'data': [
        [ 0.03807591,  0.05068012,  0.06169621, 0.02187235, -0.0442235,
         -0.03482076, -0.04340085, -0.00259226, 0.01990842, -0.01764613]
    ],
    'method': 'predict'  # If you have a classification model, the default behavior is to run 'predict_proba'.
})

output = service.run(input_payload)

print(output)
```

NOTA:  Estas dependencias se incluyen en el contenedor de inferencia Scikit-learn integrado previamente:

```yaml
    - dill
    - azureml-defaults
    - inference-schema[numpy-support]
    - scikit-learn
    - numpy
    - joblib
    - pandas
    - scipy
    - sklearn_pandas
```

## <a name="package-models"></a>Modelos de paquetes

En algunos casos, es posible que desee crear una imagen de Docker sin implementar el modelo (si, por ejemplo, pretende [implementar en Azure App Service](how-to-deploy-app-service.md)). O bien, puede que desee descargar la imagen y ejecutarla en una instalación local de Docker. Incluso, puede que desee descargar los archivos usados para compilar la imagen, inspeccionarlos, modificarlos y compilar la imagen manualmente.

El empaquetado de modelos permite hacer estas cosas. Empaqueta todos los recursos necesarios para hospedar un modelo como servicio web y le permite descargar una imagen de Docker totalmente compilada o los archivos necesarios para compilar una. Hay dos maneras de usar el empaquetado del modelos:

**Descarga de un modelo empaquetado:** descargue una imagen de Docker que contiene el modelo y otros archivos necesarios para hospedarla como servicio web.

**Generar un Dockerfile:** descargue el Dockerfile, el modelo, el script de entrada y otros recursos necesarios para compilar una imagen de Docker. Después, puede inspeccionar los archivos o realizar cambios antes de compilar la imagen localmente.

Ambos paquetes se pueden usar para obtener una imagen de Docker local.

> [!TIP]
> La creación de un paquete es similar a la implementación de un modelo. Se usa un modelo registrado y una configuración de inferencia.

> [!IMPORTANT]
> Para descargar una imagen completamente compilada o compilar una imagen localmente, debe tener [Docker](https://www.docker.com) instalado en el entorno de desarrollo.

### <a name="download-a-packaged-model"></a>Descarga del modelo empaquetado

En el ejemplo siguiente se muestra cómo compilar una imagen, que se registra en el registro de contenedor de Azure del área de trabajo:

```python
package = Model.package(ws, [model], inference_config)
package.wait_for_creation(show_output=True)
```

Después de crear un paquete, puede usar `package.pull()` para extraer la imagen en el entorno de Docker local. La salida de este comando mostrará el nombre de la imagen. Por ejemplo: 

`Status: Downloaded newer image for myworkspacef78fd10.azurecr.io/package:20190822181338`. 

Después de descargar el modelo, use el comando `docker images` para ver en una lista las imágenes locales:

```text
REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE
myworkspacef78fd10.azurecr.io/package    20190822181338      7ff48015d5bd        4 minutes ago       1.43 GB
```

Para iniciar un contenedor local basándose en esta imagen, use el siguiente comando para iniciar un contenedor con nombre desde el shell o desde la línea de comandos. Reemplace el valor `<imageid>` por el identificador de la imagen que el comando `docker images` devuelve.

```bash
docker run -p 6789:5001 --name mycontainer <imageid>
```

Este comando inicia la versión más reciente de la imagen con nombre `myimage`. Asigna el puerto local 6789 al puerto del contenedor en el que el servicio web está escuchando (5001). También asigna el nombre `mycontainer` al contenedor, lo que facilita la detención de este. Una vez iniciado el contenedor, puede enviar solicitudes a `http://localhost:6789/score`.

### <a name="generate-a-dockerfile-and-dependencies"></a>Generación de un Dockerfile y dependencias

En el ejemplo siguiente se muestra cómo descargar el Dockerfile, el modelo y otros recursos necesarios para compilar una imagen localmente. El parámetro `generate_dockerfile=True` indica que quiere los archivos, no una imagen completamente compilada.

```python
package = Model.package(ws, [model], inference_config, generate_dockerfile=True)
package.wait_for_creation(show_output=True)
# Download the package.
package.save("./imagefiles")
# Get the Azure container registry that the model/Dockerfile uses.
acr=package.get_container_registry()
print("Address:", acr.address)
print("Username:", acr.username)
print("Password:", acr.password)
```

Este código descarga los archivos necesarios para compilar la imagen en el directorio `imagefiles`. El Dockerfile incluido en los archivos guardados hace referencia a una imagen base almacenada en un registro de contenedor de Azure. Al compilar la imagen en la instalación de Docker local, necesita usar la dirección, el nombre de usuario y la contraseña para autenticarse en el registro. Siga estos pasos para compilar la imagen mediante una instalación local de Docker:

1. Desde un shell o una sesión de línea de comandos, use el comando siguiente para autenticar Docker con el registro de contenedor de Azure. Reemplace `<address>`, `<username>` y `<password>` por los valores recuperados con `package.get_container_registry()`.

    ```bash
    docker login <address> -u <username> -p <password>
    ```

2. Para compilar la imagen, utilice el siguiente comando. Reemplace `<imagefiles>` por la ruta de acceso del directorio donde `package.save()` guardó los archivos.

    ```bash
    docker build --tag myimage <imagefiles>
    ```

    Este comando establece el nombre de la imagen en `myimage`.

Para comprobar la imagen está compilada, use el comando `docker images`. Debería ver la imagen `myimage` en la lista:

```text
REPOSITORY      TAG                 IMAGE ID            CREATED             SIZE
<none>          <none>              2d5ee0bf3b3b        49 seconds ago      1.43 GB
myimage         latest              739f22498d64        3 minutes ago       1.43 GB
```

Para iniciar un nuevo contenedor basado en esta imagen, use el comando siguiente:

```bash
docker run -p 6789:5001 --name mycontainer myimage:latest
```

Este comando inicia la versión más reciente de la imagen con nombre `myimage`. Asigna el puerto local 6789 al puerto del contenedor en el que el servicio web está escuchando (5001). También asigna el nombre `mycontainer` al contenedor, lo que facilita la detención de este. Una vez iniciado el contenedor, puede enviar solicitudes a `http://localhost:6789/score`.

### <a name="example-client-to-test-the-local-container"></a>Cliente de ejemplo para probar el contenedor local

El código siguiente es un ejemplo de un cliente de Python que se puede usar con el contenedor:

```python
import requests
import json

# URL for the web service.
scoring_uri = 'http://localhost:6789/score'

# Two sets of data to score, so we get two results back.
data = {"data":
        [
            [ 1,2,3,4,5,6,7,8,9,10 ],
            [ 10,9,8,7,6,5,4,3,2,1 ]
        ]
        }
# Convert to JSON string.
input_data = json.dumps(data)

# Set the content type.
headers = {'Content-Type': 'application/json'}

# Make the request and display the response.
resp = requests.post(scoring_uri, input_data, headers=headers)
print(resp.text)
```

Para ver clientes de ejemplo en otros lenguajes de programación, vea [Consumir modelos implementados como servicios web](how-to-consume-web-service.md).

### <a name="stop-the-docker-container"></a>Detención del contenedor de Docker

Para detener el contenedor, use el siguiente comando desde una línea de comandos o un shell diferente:

```bash
docker kill mycontainer
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar un servicio web implementado, use `service.delete()`.
Para eliminar un modelo registrado, use `model.delete()`.

Para más información, consulte la documentación para [WebService.delete()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py#delete--) y [Model.delete()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#delete--).

<a id="advanced-entry-script"></a>
## <a name="advanced-entry-script-authoring"></a>Creación avanzada de scripts de entrada

<a id="binary"></a>

### <a name="binary-data"></a>Datos binarios

Si el modelo acepta datos binarios, como una imagen, debe modificar el archivo `score.py` usado para la implementación para aceptar solicitudes HTTP sin procesar. Para aceptar los datos sin procesar, use la clase `AMLRequest` en el script de entrada y agregue el elemento decorador `@rawhttp` a la función `run()`.

Este es un ejemplo de `score.py` que acepta datos binarios:

```python
from azureml.contrib.services.aml_request import AMLRequest, rawhttp
from azureml.contrib.services.aml_response import AMLResponse


def init():
    print("This is init()")


@rawhttp
def run(request):
    print("This is run()")
    print("Request: [{0}]".format(request))
    if request.method == 'GET':
        # For this example, just return the URL for GETs.
        respBody = str.encode(request.full_path)
        return AMLResponse(respBody, 200)
    elif request.method == 'POST':
        reqBody = request.get_data(False)
        # For a real-world solution, you would load the data from reqBody
        # and send it to the model. Then return the response.

        # For demonstration purposes, this example just returns the posted data as the response.
        return AMLResponse(reqBody, 200)
    else:
        return AMLResponse("bad request", 500)
```

> [!IMPORTANT]
> La clase `AMLRequest` está en el espacio de nombres `azureml.contrib`. Las entidades de este espacio de nombres cambian con frecuencia mientras trabajamos para mejorar el servicio. Todo el contenido de este espacio de nombres debe considerarse una versión preliminar que no es completamente compatible con Microsoft.
>
> Si necesita probar esto en su entorno de desarrollo local, puede instalar los componentes mediante el comando siguiente:
>
> ```shell
> pip install azureml-contrib-services
> ```

La clase `AMLRequest` solo permite tener acceso a los datos enviados sin procesar en score.py, no hay ningún componente del lado cliente. Desde un cliente, los datos se publican de la forma habitual. Por ejemplo, el siguiente código de Python lee un archivo de imagen y envía los datos:

```python
import requests
# Load image data
data = open('example.jpg', 'rb').read()
# Post raw data to scoring URI
res = requests.post(url='<scoring-uri>', data=data, headers={'Content-Type': 'application/octet-stream'})
```

<a id="cors"></a>

### <a name="cross-origin-resource-sharing-cors"></a>Uso compartido de recursos entre orígenes (CORS)

El uso compartido de recursos entre orígenes es una manera de permitir que los recursos de una página web se soliciten desde otro dominio. CORS funciona a través de los encabezados HTTP enviados con la solicitud del cliente y se devuelven con la respuesta del servicio. Para más información sobre CORS y los encabezados válidos, consulte [Intercambio de recursos de origen cruzado](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) en Wikipedia.

Para configurar la implementación del modelo para que admita CORS, use la clase `AMLResponse` en el script de entrada. Esta clase permite establecer los encabezados en el objeto de respuesta.

En el ejemplo siguiente se establece el encabezado `Access-Control-Allow-Origin` para la respuesta desde el script de entrada:

```python
from azureml.contrib.services.aml_request import AMLRequest, rawhttp
from azureml.contrib.services.aml_response import AMLResponse

def init():
    print("This is init()")

@rawhttp
def run(request):
    print("This is run()")
    print("Request: [{0}]".format(request))
    if request.method == 'GET':
        # For this example, just return the URL for GETs.
        respBody = str.encode(request.full_path)
        return AMLResponse(respBody, 200)
    elif request.method == 'POST':
        reqBody = request.get_data(False)
        # For a real-world solution, you would load the data from reqBody
        # and send it to the model. Then return the response.

        # For demonstration purposes, this example
        # adds a header and returns the request body.
        resp = AMLResponse(reqBody, 200)
        resp.headers['Access-Control-Allow-Origin'] = "http://www.example.com"
        return resp
    else:
        return AMLResponse("bad request", 500)
```

> [!IMPORTANT]
> La clase `AMLResponse` está en el espacio de nombres `azureml.contrib`. Las entidades de este espacio de nombres cambian con frecuencia mientras trabajamos para mejorar el servicio. Todo el contenido de este espacio de nombres debe considerarse una versión preliminar que no es completamente compatible con Microsoft.
>
> Si necesita probar esto en su entorno de desarrollo local, puede instalar los componentes mediante el comando siguiente:
>
> ```shell
> pip install azureml-contrib-services
> ```


> [!WARNING]
> Azure Machine Learning solo enrutará las solicitudes POST y GET a los contenedores que ejecutan el servicio de puntuación. Esto puede producir errores porque los exploradores usan solicitudes OPTIONS para preparar solicitudes CORS.
> 

## <a name="next-steps"></a>Pasos siguientes

* [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md)
* [Solución de problemas de implementación](how-to-troubleshoot-deployment.md)
* [Uso de TLS para proteger un servicio web con Azure Machine Learning](how-to-secure-web-service.md)
* [Consumir un modelo de Azure Machine Learning que está implementado como un servicio web](how-to-consume-web-service.md)
* [Supervisión de los modelos de Azure Machine Learning con Application Insights](how-to-enable-app-insights.md)
* [Recopilar datos de modelos en producción](how-to-enable-data-collection.md)
* [Creación de flujos de trabajo de aprendizaje automático controlados por eventos (versión preliminar)](how-to-use-event-grid.md)

