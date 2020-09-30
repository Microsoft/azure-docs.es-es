---
title: Cómo y dónde implementar los modelos
titleSuffix: Azure Machine Learning
description: Aprenda cómo y dónde implementar sus modelos de Azure Machine Learning, incluidos Azure Container Instances, Azure Kubernetes Service, Azure IoT Edge y matrices de puertas programables por campo.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.reviewer: larryfr
ms.date: 07/08/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: b77d48fa8af8be05123ee17f218ddb4ae5e8af98
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90886225"
---
# <a name="deploy-models-with-azure-machine-learning"></a>Implementación de modelos con Azure Machine Learning


Aprenda a implementar el modelo de Machine Learning como un servicio web en la nube de Azure o en dispositivos de Azure IoT Edge.

El flujo de trabajo es siempre parecido independientemente de donde implemente el modelo:

1. Registre el modelo.
1. Prepare una configuración de inferencia.
1. Prepare un script de entrada (a menos que se use una [implementación sin código](how-to-deploy-no-code-deployment.md))
1. Implemente el modelo en el destino de proceso.
1. Pruebe el modelo implementado, también denominado servicio web.

Para más información sobre los conceptos implicados en el flujo de trabajo de implementación, consulte [Administración, implementación y supervisión de modelos con Azure Machine Learning](concept-model-management-and-deployment.md).

## <a name="prerequisites"></a>Requisitos previos

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
- Un modelo. Si no tiene un modelo entrenado, puede usar el modelo y los archivos de dependencia que se proporcionan en [este tutorial](https://aka.ms/azml-deploy-cloud).
- La [extensión de la interfaz de la línea de comandos (CLI) de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md).

# <a name="python"></a>[Python](#tab/python)

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
- Un modelo. Si no tiene un modelo entrenado, puede usar el modelo y los archivos de dependencia que se proporcionan en [este tutorial](https://aka.ms/azml-deploy-cloud).
- El [kit de desarrollo de software (SDK) de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true).

---

## <a name="connect-to-your-workspace"></a>Conexión con su área de trabajo

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Siga las instrucciones de la documentación de la CLI de Azure para [configurar el contexto de la suscripción](/cli/azure/manage-azure-subscriptions-azure-cli#change-the-active-subscription).

A continuación, haga lo siguiente:

```azurecli-interactive
az ml workspace list --resource-group=<my resource group>
```

para ver las áreas de trabajo a las que tiene acceso.

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core import Workspace
ws = Workspace.from_config(path=".file-path/ws_config.json")
```

Para más información sobre el uso del SDK para conectarse a un área de trabajo, consulte la documentación del [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py#&preserve-view=trueworkspace).


---


## <a name="register-your-model"></a><a id="registermodel"></a> Registro del modelo

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Un modelo registrado es un contenedor lógico para uno o varios archivos que componen el modelo. Por ejemplo, si tiene un modelo que se almacena en varios archivos, puede registrarlos como un único modelo en el área de trabajo. Después de registrar los archivos, puede descargar o implementar el modelo registrado y recibir todos los archivos que registró.

> [!TIP]
> Al registrar un modelo, se proporciona una ruta de acceso de una ubicación en la nube (desde una ejecución de entrenamiento) o de un directorio local. Esta ruta de acceso es solo para localizar los archivos que se van a cargar como parte del proceso de registro. No es necesario que coincida con la ruta de acceso que se usa en el script de entrada. Para obtener más información, consulte [Locate model files in your entry script](how-to-deploy-advanced-entry-script.md#load-registered-models) (Localización de archivos de modelo en el script de entrada).

Los modelos de Machine Learning se registran en el área de trabajo de Azure Machine Learning. El modelo puede proceder de Azure Machine Learning o de otro lugar. Al registrar un modelo, opcionalmente, puede proporcionar metadatos sobre el modelo. Los diccionarios `tags` y `properties` que se aplican a un registro de modelo se pueden usar para filtrar modelos.

En los ejemplos siguientes se muestra cómo registrar un modelo.

### <a name="register-a-model-from-an-azure-ml-training-run"></a>Registro de un modelo a partir de una ejecución de entrenamiento de Azure ML

```azurecli-interactive
az ml model register -n sklearn_mnist  --asset-path outputs/sklearn_mnist_model.pkl  --experiment-name myexperiment --run-id myrunid --tag area=mnist
```

[!INCLUDE [install extension](../../includes/machine-learning-service-install-extension.md)]

El parámetro `--asset-path` hace referencia a la ubicación del modelo en la nube. En este ejemplo, se usa la ruta de acceso de un único archivo. Para incluir varios archivos en el registro del modelo, establezca `--asset-path` en la ruta de acceso de una carpeta que contiene los archivos.

### <a name="register-a-model-from-a-local-file"></a>Registrar un modelo desde un archivo local

```azurecli-interactive
az ml model register -n onnx_mnist -p mnist/model.onnx
```

Para incluir varios archivos en el registro del modelo, establezca `-p` en la ruta de acceso de una carpeta que contiene los archivos.

Para más información sobre `az ml model register`, consulte la [documentación de referencia](/cli/azure/ext/azure-cli-ml/ml/model).

# <a name="python"></a>[Python](#tab/python)


Un modelo registrado es un contenedor lógico para uno o varios archivos que componen el modelo. Por ejemplo, si tiene un modelo que se almacena en varios archivos, puede registrarlos como un único modelo en el área de trabajo. Después de registrar los archivos, puede descargar o implementar el modelo registrado y recibir todos los archivos que registró.

> [!TIP]
> Al registrar un modelo, se proporciona una ruta de acceso de una ubicación en la nube (desde una ejecución de entrenamiento) o de un directorio local. Esta ruta de acceso es solo para localizar los archivos que se van a cargar como parte del proceso de registro. No es necesario que coincida con la ruta de acceso que se usa en el script de entrada. Para obtener más información, consulte [Locate model files in your entry script](how-to-deploy-advanced-entry-script.md#load-registered-models) (Localización de archivos de modelo en el script de entrada).

Los modelos de Machine Learning se registran en el área de trabajo de Azure Machine Learning. El modelo puede proceder de Azure Machine Learning o de otro lugar. Al registrar un modelo, opcionalmente, puede proporcionar metadatos sobre el modelo. Los diccionarios `tags` y `properties` que se aplican a un registro de modelo se pueden usar para filtrar modelos.

En los ejemplos siguientes se muestra cómo registrar un modelo.

### <a name="register-a-model-from-an-azure-ml-training-run"></a>Registro de un modelo a partir de una ejecución de entrenamiento de Azure ML

  Al usar el SDK para entrenar un modelo, puede recibir un objeto [Run](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run.run?view=azure-ml-py&preserve-view=true) o [AutoMLRun](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun), en función de cómo haya entrenado el modelo. Cada objeto se puede usar para registrar un modelo creado por una ejecución del experimento.

  + Registre un modelo desde un objeto `azureml.core.Run`:
 
    ```python
    model = run.register_model(model_name='sklearn_mnist',
                               tags={'area': 'mnist'},
                               model_path='outputs/sklearn_mnist_model.pkl')
    print(model.name, model.id, model.version, sep='\t')
    ```

    El parámetro `model_path` hace referencia a la ubicación del modelo en la nube. En este ejemplo, se usa la ruta de acceso de un único archivo. Para incluir varios archivos en el registro del modelo, establezca `model_path` en la ruta de acceso de una carpeta que contiene los archivos. Para obtener más información, consulte la documentación [Run.register_model](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run.run?view=azure-ml-py#&preserve-view=trueregister-model-model-name--model-path-none--tags-none--properties-none--model-framework-none--model-framework-version-none--description-none--datasets-none--sample-input-dataset-none--sample-output-dataset-none--resource-configuration-none----kwargs-).

  + Registre un modelo desde un objeto `azureml.train.automl.run.AutoMLRun`:

    ```python
        description = 'My AutoML Model'
        model = run.register_model(description = description,
                                   tags={'area': 'mnist'})

        print(run.model_id)
    ```

    En este ejemplo, los parámetros `metric` y `iteration`, no se especifican, por lo que se registrará la iteración con la mejor métrica principal. Se utiliza el valor `model_id` devuelto de la ejecución en lugar de un nombre de modelo.

    Para más información, consulte la documentación de [AutoMLRun.register_model](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun#register-model-model-name-none--description-none--tags-none--iteration-none--metric-none-).


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

Para más información, consulte la documentación de la [clase Model](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py&preserve-view=true).

Para más información sobre cómo trabajar con modelos entrenados al margen de Azure Machine Learning, consulte el artículo sobre cómo [implementar un modelo existente](how-to-deploy-existing-model.md).

---

## <a name="define-an-entry-script"></a>Definición de un script de entrada

[!INCLUDE [write entry script](../../includes/machine-learning-entry-script.md)]


## <a name="define-an-inference-configuration"></a>Definición de una configuración de inferencia

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

[!INCLUDE [inference config](../../includes/machine-learning-service-inference-config.md)]

El comando siguiente muestra cómo implementar un modelo mediante la CLI:

```azurecli-interactive
az ml model deploy -n myservice -m mymodel:1 --ic inferenceconfig.json
```

En este ejemplo, la configuración especifica la siguiente configuración:

* Que el modelo requiere Python.
* El [script de entrada](#define-an-entry-script), que se usa para controlar las solicitudes web que se envían al servicio implementado
* El archivo de Conda que describe los paquetes de Python necesarios para realizar la inferencia.

Para información sobre el uso de una imagen personalizada de Docker con una configuración de inferencia, consulte [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md).

# <a name="python"></a>[Python](#tab/python)

Una configuración de inferencia describe cómo establecer el servicio web que contiene el modelo. Se usa más adelante, cuando se implementa el modelo.

La configuración de inferencia usa entornos Azure Machine Learning para definir las dependencias de software necesarias para la implementación. Los entornos permiten crear, administrar y reutilizar las dependencias de software necesarias para el entrenamiento y la implementación. Puede crear un entorno a partir de archivos de dependencia personalizados o usar uno de los entornos de Azure Machine Learning mantenidos. El siguiente código YAML es un ejemplo de un archivo de dependencias de Conda para inferencia. Tenga en cuenta que debe indicar valores predeterminados de azureml con versión > = 1.0.45 como una dependencia de PIP, ya que contiene la funcionalidad necesaria para hospedar el modelo como un servicio web. Si desea usar la generación automática de esquemas, el script de entrada debe importar también los paquetes `inference-schema`.

```YAML

name: project_environment
dependencies:
- python=3.6.2
- scikit-learn=0.22.1
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

Puede encontrar una discusión minuciosa sobre el uso y la personalización de los entornos de Python con Azure Machine Learning en [Creación y uso de entornos de software en Azure Machine Learning](how-to-use-environments.md).

Para información sobre el uso de una imagen personalizada de Docker con una configuración de inferencia, consulte [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md).


En el ejemplo siguiente se muestra cómo cargar un entorno desde el área de trabajo y luego usarlo con la configuración de inferencia:

```python
from azureml.core.environment import Environment
from azureml.core.model import InferenceConfig


myenv = Environment.get(workspace=ws, name='myenv', version='1')
inference_config = InferenceConfig(entry_script='path-to-score.py',
                                    environment=myenv)
```

Para obtener más información sobre los entornos, consulte el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md).

Para más información sobre la configuración de inferencia, consulte la documentación de la clase [InferenceConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig?view=azure-ml-py&preserve-view=true).

---

## <a name="choose-a-compute-target"></a>Elección de un destino de proceso

[!INCLUDE [aml-compute-target-deploy](../../includes/aml-compute-target-deploy.md)]

## <a name="define-a-deployment-configuration"></a>Definición de una configuración de implementación

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Las opciones disponibles para una configuración de implementación varían en función del destino de proceso que elija.

[!INCLUDE [aml-local-deploy-config](../../includes/machine-learning-service-local-deploy-config.md)]

Para más información, consulte la documentación de [az ml model deploy](/cli/azure/ext/azure-cli-ml/ml/model#ext-azure-cli-ml-az-ml-model-deploy).

# <a name="python"></a>[Python](#tab/python)

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

---



## <a name="deploy-your-model"></a>Implementación del modelo

Ahora está preparado para implementar el modelo. 

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

### <a name="using-a-registered-model"></a>Uso de un modelo registrado

Si registró el modelo en el área de trabajo de Azure Machine Learning, sustituya "mymodel:1" por el nombre del modelo y su número de versión.

```azurecli-interactive
az ml model deploy -m mymodel:1 --ic inferenceconfig.json --dc deploymentconfig.json
```

### <a name="using-a-local-model"></a>Uso de un modelo local

Si prefiere no registrar el modelo, puede pasar el parámetro "sourceDirectory" del archivo inferenceconfig.json para especificar un directorio local desde el que se va atender el modelo.

```azurecli-interactive
az ml model deploy --ic inferenceconfig.json --dc deploymentconfig.json
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra una implementación local. La sintaxis variará en función del destino de proceso elegido en el paso anterior.

```python
from azureml.core.webservice import LocalWebservice, Webservice

deployment_config = LocalWebservice.deploy_configuration(port=8890)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config)
service.wait_for_deployment(show_output = True)
print(service.state)
```

Para más información, consulte la documentación de [LocalWebservice](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.local.localwebservice?view=azure-ml-py&preserve-view=true), [Model.deploy()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#&preserve-view=truedeploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) y [Webservice](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.webservice?view=azure-ml-py&preserve-view=true).

---

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


### <a name="batch-inference"></a><a id="azuremlcompute"></a> Inferencia por lotes
Los destinos de proceso de Azure Machine Learning se crean y administran mediante Azure Machine Learning. Estos se pueden usar para la predicción por lotes en canalizaciones de Azure Machine Learning.

Para ver un tutorial sobre la inferencia por lotes con Proceso de Azure Machine Learning, consulte [Cómo ejecutar predicciones por lotes](tutorial-pipeline-batch-scoring-classification.md).

### <a name="iot-edge-inference"></a><a id="iotedge"></a> Inferencia de IoT Edge
La compatibilidad con implementaciones en el perímetro está en versión preliminar. Para más información, consulte [Implementación de Azure Machine Learning como un módulo de IoT Edge](https://docs.microsoft.com/azure/iot-edge/tutorial-deploy-machine-learning).

## <a name="delete-resources"></a>Eliminar recursos

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Para eliminar un servicio web implementado, use `az ml service <name of webservice>`.

Para eliminar un modelo registrado del área de trabajo, use `az ml model delete <model id>`.

Más información sobre la [eliminación de un servicio web](/cli/azure/ext/azure-cli-ml/ml/service#ext-azure-cli-ml-az-ml-service-delete) y la [eliminación de un modelo](/cli/azure/ext/azure-cli-ml/ml/model#ext-azure-cli-ml-az-ml-model-delete).

# <a name="python"></a>[Python](#tab/python)

Para eliminar un servicio web implementado, use `service.delete()`.
Para eliminar un modelo registrado, use `model.delete()`.

Para más información, consulte la documentación para [WebService.delete()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py#&preserve-view=truedelete--) y [Model.delete()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#&preserve-view=truedelete--).

---


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

