---
title: Implementación de modelos con una imagen de Docker personalizada
titleSuffix: Azure Machine Learning
description: Aprenda a usar una imagen base de Docker personalizada al implementar los modelos de Azure Machine Learning. Aunque Azure Machine Learning proporciona una imagen base predeterminada, también puede usar una propia.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 06/17/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 76eed22052b8c9fe2cc849e68dd926ef2c85208a
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87843222"
---
# <a name="deploy-a-model-using-a-custom-docker-base-image"></a>Implementación de un modelo con una imagen base de Docker personalizada
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Aprenda a usar una imagen base de Docker al implementar modelos entrenados con Azure Machine Learning.

Cuando se implementa un modelo entrenado en un servicio web o un dispositivo IoT Edge, se crea un paquete que contiene un servidor web para controlar las solicitudes entrantes.

Azure Machine Learning proporciona una imagen base de Docker predeterminada para que no tenga que preocuparse de crear una. También puede usar __entornos__ de Azure Machine Learning para seleccionar una imagen base específica o usar una personalizada que proporcione.

Una imagen base se usa como punto de partida cuando se crea una imagen para una implementación. Proporciona el sistema operativo y los componentes subyacentes. A continuación, el proceso de implementación agrega componentes adicionales, como el modelo, el entorno de Conda y otros recursos, a la imagen antes de implementarla.

Normalmente, se crea una imagen base personalizada cuando se desea usar Docker para administrar las dependencias, mantener un mayor control sobre las versiones de los componentes o ahorrar tiempo durante la implementación. Por ejemplo, es posible que quiera estandarizar a partir de una versión específica de Python, Conda u otro componente. También puede querer instalar el software necesario para el modelo, caso en que el proceso de instalación lleva mucho tiempo. Instalar el software al crear la imagen base significa que no tendrá que instalarlo para cada implementación.

> [!IMPORTANT]
> Al implementar un modelo, no se pueden invalidar los componentes principales, como el servidor web o los componentes de IoT Edge. Estos componentes proporcionan un entorno conocido de trabajo que Microsoft ha probado y admite.

> [!WARNING]
> Es posible que Microsoft no pueda ayudar a solucionar los problemas derivados de una imagen personalizada. Si tiene problemas, se le pedirá que use la imagen predeterminada o una de las imágenes que proporciona Microsoft para ver si el problema es específico de la imagen.

Este documento se divide en dos secciones:

* Creación de una imagen base personalizada: proporciona información para administradores y DevOps para crear una imagen personalizada y configurar la autenticación en Azure Container Registry mediante la CLI de Azure y la CLI de Machine Learning.
* Implementación de un modelo con una imagen base personalizada: proporciona información a los científicos de datos y DevOps/ingenieros de ML sobre el uso de imágenes personalizadas al implementar un modelo entrenado desde el SDK de Python o la CLI de ML.

## <a name="prerequisites"></a>Requisitos previos

* Un grupo de trabajo de Azure Machine Learning. Para más información, consulte el artículo [Crear un área de trabajo](how-to-manage-workspace.md).
* El [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py). 
* La[CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).
* La [extensión de la CLI para Azure Machine Learning](reference-azure-machine-learning-cli.md).
* Una instancia de [Azure Container Registry](/azure/container-registry) u otro registro de Docker accesible en Internet.
* En este documento, se da por hecho que está familiarizado con la creación y el uso de un objeto de __configuración de inferencia__ como parte de la implementación de modelos. Para más información, consulte [Dónde y cómo implementar](how-to-deploy-and-where.md).

## <a name="create-a-custom-base-image"></a>Creación de una imagen base personalizada

Para la información de esta sección, se da por supuesto que usa Azure Container Registry para almacenar imágenes de Docker. Use la lista de comprobación siguiente a la hora de planear la creación de imágenes personalizadas para Azure Machine Learning:

* ¿Usará la instancia de Azure Container Registry creada para el área de trabajo de Azure Machine Learning, o una instancia independiente?

    Al usar imágenes almacenadas en el __registro de contenedor para el área de trabajo__, no es necesario autenticarse en el registro. El área de trabajo controla la autenticación.

    > [!WARNING]
    > La instancia de Azure Container Registry para el área de trabajo __se crea la primera vez que entrena o implementa un modelo__ con el área de trabajo. Si ha creado una nueva área de trabajo, pero no ha entrenado ni ha creado ningún modelo, no existirá ninguna instancia de Azure Container Registry para el área de trabajo.

    Para obtener información sobre cómo recuperar el nombre de la instancia de Azure Container Registry para el área de trabajo, vea la sección [Obtener el nombre del registro de contenedor](#getname) de este artículo.

    Al usar imágenes almacenadas en un __registro de contenedor independiente__, deberá configurar una entidad de servicio que tenga, al menos, acceso de lectura. A continuación, proporcione el id. de la entidad de servicio (nombre de usuario) y la contraseña a cualquier persona que use imágenes del registro. La excepción es si hace el registro de contenedor accesible públicamente.

    Para obtener información sobre cómo crear una instancia de Azure Container Registry, consulte [Crear un registro de contenedor privado](/azure/container-registry/container-registry-get-started-azure-cli).

    Para obtener más información sobre cómo usar entidades de servicio con Azure Container Registry, consulte [Autenticación de Azure Container Registry con entidades de servicio](/azure/container-registry/container-registry-auth-service-principal).

* Información de la imagen y Azure Container Registry: Proporcione el nombre de imagen a cualquier persona que necesite usarlo. Por ejemplo, para hacer referencia a una imagen denominada `myimage` almacenada en un registro denominado `myregistry`, se usa `myregistry.azurecr.io/myimage` si la imagen se utiliza para la implementación de modelos.

* Requisitos de imagen: Azure Machine Learning solo admite imágenes de Docker que proporcionan el software siguiente:

    * Ubuntu 16.04 o posterior.
    * Conda 4.5.# o posterior.
    * Python 3.5.#, 3.6.# o 3.7.#.

<a id="getname"></a>

### <a name="get-container-registry-information"></a>Obtener información sobre el registro de contenedor

En esta sección, aprenderá a obtener el nombre de la instancia de Azure Container Registry para el área de trabajo de Azure Machine Learning.

> [!WARNING]
> La instancia de Azure Container Registry para el área de trabajo __se crea la primera vez que entrena o implementa un modelo__ con el área de trabajo. Si ha creado una nueva área de trabajo, pero no ha entrenado ni ha creado ningún modelo, no existirá ninguna instancia de Azure Container Registry para el área de trabajo.

Si ya ha entrenado o implementado modelos mediante Azure Machine Learning, se creó un registro de contenedor para el área de trabajo. Para buscar el nombre de este registro de contenedor, siga estos pasos:

1. Abra un shell o símbolo del sistema nuevo y use el comando siguiente para autenticarse en su suscripción de Azure:

    ```azurecli-interactive
    az login
    ```

    Siga las indicaciones para autenticarse en la suscripción.

    [!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)] 

2. Use el siguiente comando para mostrar el registro de contenedor para el área de trabajo. Reemplace `<myworkspace>` por el nombre del área de trabajo de Azure Machine Learning. Reemplace `<resourcegroup>` por el grupo de recursos de Azure que contiene el área de trabajo:

    ```azurecli-interactive
    az ml workspace show -w <myworkspace> -g <resourcegroup> --query containerRegistry
    ```

    [!INCLUDE [install extension](../../includes/machine-learning-service-install-extension.md)]

    La información devuelta es similar al texto siguiente:

    ```text
    /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.ContainerRegistry/registries/<registry_name>
    ```

    El valor `<registry_name>` es el nombre de Azure Container Registry para el área de trabajo.

### <a name="build-a-custom-base-image"></a>Compilación de una imagen base personalizada

Los pasos de esta sección le orientan a lo largo de la creación de una imagen de Docker personalizada en Azure Container Registry.

1. Cree un nuevo archivo de texto denominado `Dockerfile` y use el texto siguiente como contenido:

    ```text
    FROM ubuntu:16.04

    ARG CONDA_VERSION=4.5.12
    ARG PYTHON_VERSION=3.6

    ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
    ENV PATH /opt/miniconda/bin:$PATH

    RUN apt-get update --fix-missing && \
        apt-get install -y wget bzip2 && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/*

    RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-${CONDA_VERSION}-Linux-x86_64.sh -O ~/miniconda.sh && \
        /bin/bash ~/miniconda.sh -b -p /opt/miniconda && \
        rm ~/miniconda.sh && \
        /opt/miniconda/bin/conda clean -tipsy

    RUN conda install -y conda=${CONDA_VERSION} python=${PYTHON_VERSION} && \
        conda clean -aqy && \
        rm -rf /opt/miniconda/pkgs && \
        find / -type d -name __pycache__ -prune -exec rm -rf {} \;
    ```

2. Desde un shell o una línea de comandos, use lo siguiente para autenticarse en Azure Container Registry. Reemplace `<registry_name>` con el nombre del registro de contenedor en que quiera almacenar la imagen:

    ```azurecli-interactive
    az acr login --name <registry_name>
    ```

3. Para cargar el archivo Dockerfile y compilarlo, use el siguiente comando. Reemplace `<registry_name>` con el nombre del registro de contenedor en que quiera almacenar la imagen:

    ```azurecli-interactive
    az acr build --image myimage:v1 --registry <registry_name> --file Dockerfile .
    ```

    > [!TIP]
    > En este ejemplo, se aplica una etiqueta de `:v1` a la imagen. Si no se proporciona ninguna etiqueta, se aplica una etiqueta de `:latest`.

    Durante el proceso de compilación, se transmite en secuencias información para respaldar la línea de comandos. Si la compilación se completa correctamente, recibirá un mensaje similar al texto siguiente:

    ```text
    Run ID: cda was successful after 2m56s
    ```

Para obtener más información sobre la creación de imágenes con Azure Container Registry, consulte [Inicio rápido: Compilación y ejecución de una imagen de contenedor en Azure Container Registry Tasks](https://docs.microsoft.com/azure/container-registry/container-registry-quickstart-task-cli)

Para más información acerca de cómo cargar imágenes existentes en una instancia de Azure Container Registry, consulte [Inserción de la primera imagen en un registro de contenedor privado de Docker](/azure/container-registry/container-registry-get-started-docker-cli).

## <a name="use-a-custom-base-image"></a>Uso de una imagen base personalizada

Para usar una imagen personalizada, necesitará la siguiente información:

* El __nombre de la imagen__. Por ejemplo, `mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda:latest` es la ruta de acceso a una imagen de Docker básica que proporciona Microsoft.

    > [!IMPORTANT]
    > En el caso de las imágenes personalizadas que haya creado, asegúrese de incluir todas las etiquetas que se usaron con la imagen. Por ejemplo, si la imagen se creó con una etiqueta específica, como `:v1`. Si no usó ninguna etiqueta específica al crear la imagen, se aplicó una etiqueta de `:latest`.

* Si la imagen está en un __repositorio privado__, necesita la siguiente información:

    * La __dirección__ del registro. Por ejemplo, `myregistry.azureecr.io`.
    * Un __nombre de usuario__ y una __contraseña__ de entidad de servicio con acceso de lectura al registro.

    Si no tiene esta información, hable con el administrador de la instancia de Azure Container Registry que contiene la imagen.

### <a name="publicly-available-base-images"></a>Imágenes base disponibles públicamente

Microsoft proporciona varias imágenes de Docker en un repositorio accesible públicamente que se pueden usar mediante los pasos descritos en esta sección:

| Imagen | Descripción |
| ----- | ----- |
| `mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda` | Imagen básica para Azure Machine Learning |
| `mcr.microsoft.com/azureml/onnxruntime:latest` | Contiene el tiempo de ejecución de ONNX para la inferencia de CPU |
| `mcr.microsoft.com/azureml/onnxruntime:latest-cuda` | Contiene el tiempo de ejecución ONNX y CUDA para GPU |
| `mcr.microsoft.com/azureml/onnxruntime:latest-tensorrt` | Contiene el tiempo de ejecución ONNX y TensorRT para GPU |
| `mcr.microsoft.com/azureml/onnxruntime:latest-openvino-vadm ` | Contiene el tiempo de ejecución de ONNX y OpenVINO para <sup></sup>Vision Accelerator Design de Intel basado en Movidius <sup>TM</sup> MyriadX VPUs |
| `mcr.microsoft.com/azureml/onnxruntime:latest-openvino-myriad` | Contiene el tiempo de ejecución de ONNX y OpenVINO para las llaves USB <sup></sup>Movidius <sup>TM</sup> de Intel |

Para más información acerca de las imágenes base de tiempo de ejecución de ONNX, consulte [la sección ONNX Runtime dockerfile](https://github.com/microsoft/onnxruntime/blob/master/dockerfiles/README.md) en el repositorio de GitHub.

> [!TIP]
> Puesto que estas imágenes están disponibles públicamente, no es necesario proporcionar ninguna dirección, nombre de usuario ni contraseña para usarlas.

Para más información, consulte el repositorio de GitHub de [Contenedores de Azure Machine Learning](https://github.com/Azure/AzureML-Containers).

### <a name="use-an-image-with-the-azure-machine-learning-sdk"></a>Usar una imagen con el SDK de Azure Machine Learning

Para usar una imagen almacenada en **Azure Container Registry para el área de trabajo** o un **registro de contenedor que sea accesible públicamente**, establezca los siguientes atributos de [entorno](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.environment?view=azure-ml-py):

+ `docker.enabled=True`
+ `docker.base_image`: Establézcalo en el registro y la ruta de acceso a la imagen.

```python
from azureml.core.environment import Environment
# Create the environment
myenv = Environment(name="myenv")
# Enable Docker and reference an image
myenv.docker.enabled = True
myenv.docker.base_image = "mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda:latest"
```

Para usar una imagen desde un __registro de contenedor privado__ que no esté en el área de trabajo, debe usar `docker.base_image_registry` para especificar la dirección del repositorio y un nombre de usuario y contraseña:

```python
# Set the container registry information
myenv.docker.base_image_registry.address = "myregistry.azurecr.io"
myenv.docker.base_image_registry.username = "username"
myenv.docker.base_image_registry.password = "password"

myenv.inferencing_stack_version = "latest"  # This will install the inference specific apt packages.

# Define the packages needed by the model and scripts
from azureml.core.conda_dependencies import CondaDependencies
conda_dep = CondaDependencies()
# you must list azureml-defaults as a pip dependency
conda_dep.add_pip_package("azureml-defaults")
myenv.python.conda_dependencies=conda_dep
```

Debe agregar azureml-defaults con la versión 1.0.45 o posterior como una dependencia pip. Este paquete contiene la funcionalidad necesaria para hospedar el modelo como un servicio web. También debe establecer la propiedad inferencing_stack_version en el entorno en "latest", lo que instalará los paquetes apt específicos que el servicio web necesita. 

Después de definir el entorno, úselo con un objeto [InferenceConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig?view=azure-ml-py) para definir el entorno de inferencia en el que se ejecutará el modelo y el servicio web.

```python
from azureml.core.model import InferenceConfig
# Use environment in InferenceConfig
inference_config = InferenceConfig(entry_script="score.py",
                                   environment=myenv)
```

En este momento, puede continuar con la implementación. Por ejemplo, el siguiente fragmento de código implementaría un servicio web localmente mediante la configuración de inferencia y la imagen personalizada:

```python
from azureml.core.webservice import LocalWebservice, Webservice

deployment_config = LocalWebservice.deploy_configuration(port=8890)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config)
service.wait_for_deployment(show_output = True)
print(service.state)
```

Para más información sobre la implementación, consulte [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

Para más información sobre cómo personalizar el entorno de Python, consulte el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md). 

### <a name="use-an-image-with-the-machine-learning-cli"></a>Usar una imagen con la CLI de Machine Learning

> [!IMPORTANT]
> Actualmente, la CLI de Machine Learning puede usar imágenes de Azure Container Registry para su área de trabajo o los repositorios accesibles públicamente. No puede usar las imágenes de registros privados independientes.

Antes de implementar un modelo mediante la CLI de Machine Learning, cree un [entorno](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.environment?view=azure-ml-py) que use la imagen personalizada. Después, cree un archivo de configuración de la inferencia que haga referencia al entorno. También puede definir el entorno directamente en el archivo de configuración de la inferencia. El documento JSON siguiente muestra cómo hacer referencia a una imagen en un registro de contenedor público. En este ejemplo, el entorno se define insertado:

```json
{
    "entryScript": "score.py",
    "environment": {
        "docker": {
            "arguments": [],
            "baseDockerfile": null,
            "baseImage": "mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda:latest",
            "enabled": false,
            "sharedVolumes": true,
            "shmSize": null
        },
        "environmentVariables": {
            "EXAMPLE_ENV_VAR": "EXAMPLE_VALUE"
        },
        "name": "my-deploy-env",
        "python": {
            "baseCondaEnvironment": null,
            "condaDependencies": {
                "channels": [
                    "conda-forge"
                ],
                "dependencies": [
                    "python=3.6.2",
                    {
                        "pip": [
                            "azureml-defaults",
                            "azureml-telemetry",
                            "scikit-learn",
                            "inference-schema[numpy-support]"
                        ]
                    }
                ],
                "name": "project_environment"
            },
            "condaDependenciesFile": null,
            "interpreterPath": "python",
            "userManagedDependencies": false
        },
        "version": "1"
    }
}
```

Este archivo se usa con el comando `az ml model deploy`. El parámetro `--ic` se usa para especificar el archivo de configuración de la inferencia.

```azurecli
az ml model deploy -n myservice -m mymodel:1 --ic inferenceconfig.json --dc deploymentconfig.json --ct akscomputetarget
```

Para más información sobre la implementación de un modelo mediante la CLI de ML, consulte la sección sobre el registro, la creación de perfiles y la implementación del modelo del artículo [Extensión de la CLI para Azure Machine Learning](reference-azure-machine-learning-cli.md#model-registration-profiling-deployment).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [Cómo y dónde implementar un modelo](how-to-deploy-and-where.md).
* Información sobre cómo [Entrenar e implementar modelos de Machine Learning mediante Azure Pipelines](/azure/devops/pipelines/targets/azure-machine-learning?view=azure-devops).
