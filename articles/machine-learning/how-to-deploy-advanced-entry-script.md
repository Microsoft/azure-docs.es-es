---
title: Creación de un script de entrada para escenarios avanzados
titleSuffix: Azure Machine Learning entry script authoring
author: gvashishtha
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 07/31/2020
ms.author: gopalv
ms.openlocfilehash: 0499cd6885454604e89ce4cadc313b2f68c45156
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87544505"
---
# <a name="advanced-entry-script-authoring"></a>Creación avanzada de scripts de entrada

En este artículo se muestra cómo escribir scripts de entrada para casos de uso especializados.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por supuesto que ya tiene un modelo de aprendizaje automático entrenado que planea implementar con Azure Machine Learning. Para más información sobre la implementación de modelos, consulte [este tutorial](how-to-deploy-and-where.md).

## <a name="automatically-generate-a-swagger-schema"></a>Generación automática de un esquema de Swagger

Para generar automáticamente un esquema para el servicio web, proporcione un ejemplo de la entrada y salida del constructor de uno de los objetos de tipo definidos. El tipo y ejemplo se usan para crear automáticamente el esquema. Azure Machine Learning creará, a continuación, una especificación de [OpenAPI](https://swagger.io/docs/specification/about/) (Swagger) para el servicio web durante la implementación.

Actualmente se admiten estos tipos:

* `pandas`
* `numpy`
* `pyspark`
* Objeto estándar de Python

Para usar la generación de esquemas, incluya el paquete `inference-schema` de código abierto en el archivo de dependencias. Para más información sobre este paquete, consulte [https://github.com/Azure/InferenceSchema](https://github.com/Azure/InferenceSchema). Defina los formatos de ejemplo de entrada y salida de las variables `input_sample` y `output_sample`, que representan los formatos de solicitud y respuesta del servicio web. Use estos ejemplos en los decoradores de entrada y salida de la función `run()`. El siguiente ejemplo de Scikit-learn usa generación de esquemas.


## <a name="power-bi-compatible-endpoint"></a>Punto de conexión compatible con Power BI 

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


## <a name="binary-ie-image-data"></a><a id="binary-data"></a> Datos binarios (por ejemplo, una imagen)

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

## <a name="cross-origin-resource-sharing-cors"></a>Uso compartido de recursos entre orígenes (CORS)

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


## <a name="load-registered-models"></a>Carga de los modelos registrados

Hay dos maneras de buscar modelos en el script de entrada:
* `AZUREML_MODEL_DIR`: variable de entorno que contiene la ruta de acceso a la ubicación del modelo.
* `Model.get_model_path`: API que devuelve la ruta de acceso al archivo de modelo mediante el nombre del modelo registrado.

#### <a name="azureml_model_dir"></a>AZUREML_MODEL_DIR

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

### <a name="get_model_path"></a>get_model_path

Cuando registra un modelo, puede proporcionar un nombre de modelo que se usa para administrar este en el registro. Usará este nombre con el método [Model.get_model_path()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#get-model-path-model-name--version-none---workspace-none-) para recuperar la ruta de acceso del archivo del modelo en el sistema de archivos local. Si registra una carpeta o una colección de archivos, esta API devolverá la ruta de acceso del directorio que contiene esos archivos.

Al registrar un modelo, se le asigna un nombre. El nombre se corresponde con la ubicación donde se coloque el modelo, ya sea localmente o durante la implementación del servicio.

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de implementaciones con errores](how-to-troubleshoot-deployment.md)
* [Implementación en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Creación de aplicaciones cliente para consumir servicios web](how-to-consume-web-service.md)
* [Actualización de servicios web](how-to-deploy-update-web-service.md)
* [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md)
* [Uso de TLS para proteger un servicio web con Azure Machine Learning](how-to-secure-web-service.md)
* [Supervisión de los modelos de Azure Machine Learning con Application Insights](how-to-enable-app-insights.md)
* [Recopilar datos de modelos en producción](how-to-enable-data-collection.md)
* [Creación de flujos de trabajo de aprendizaje automático controlados por eventos (versión preliminar)](how-to-use-event-grid.md)
