---
title: Implementación de modelos de aprendizaje automático en Azure App Service (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a usar Azure Machine Learning para implementar un modelo en una aplicación web en Azure App Service.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 06/23/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 7c98bfe4adb9cbbcc1009c530ba875511ea9ec01
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905111"
---
# <a name="deploy-a-machine-learning-model-to-azure-app-service-preview"></a>Implementación de un modelo de aprendizaje automático en Azure App Service (versión preliminar)


Aprenda a implementar un modelo de Azure Machine Learning como aplicación web en Azure App Service.

> [!IMPORTANT]
> Aunque tanto Azure Machine Learning como Azure App Service están disponibles con carácter general, la posibilidad de implementar un modelo desde Machine Learning Service en App Service está en versión preliminar.

Con Azure Machine Learning, puede crear una imagen de Docker a partir de modelos de aprendizaje automático entrenados. Esta imagen contiene un servicio web que recibe datos, los envía al modelo y, luego, devuelve la respuesta. Azure App Service se puede usar para implementar la imagen y proporciona las siguientes características:

* [Autenticación](/azure/app-service/configure-authentication-provider-aad) avanzada para mejorar la seguridad. Los métodos de autenticación incluyen Azure Active Directory y la autenticación multifactor.
* [Escalado automático](/azure/azure-monitor/platform/autoscale-get-started?toc=%2fazure%2fapp-service%2ftoc.json) sin tener que volver a realizar la implementación.
* [Compatibilidad con TLS](/azure/app-service/configure-ssl-certificate-in-code) para comunicaciones seguras entre los clientes y el servicio.

Para más información sobre las características proporcionadas por Azure App Service, consulte la [introducción a App Service](/azure/app-service/overview).

> [!IMPORTANT]
> Si necesita poder registrar los datos de puntuación que se usan con el modelo implementado, o los resultados de la puntuación, debe implementar Azure Kubernetes Service en su lugar. Para obtener más información, consulte [Recopilar datos en los modelos de producción](how-to-enable-data-collection.md).

## <a name="prerequisites"></a>Prerrequisitos

* Un área de trabajo de Azure Machine Learning. Para más información, consulte el artículo [Crear un área de trabajo](how-to-manage-workspace.md).
* La[CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).
* Un modelo de aprendizaje automático entrenado registrado en el área de trabajo. Si no tiene un modelo, use el [Tutorial: Entrenamiento de modelos de clasificación de imágenes](tutorial-train-models-with-aml.md) para entrenar y registrar uno.

    > [!IMPORTANT]
    > En los fragmentos de código de este artículo se supone que se han establecido las siguientes variables:
    >
    > * `ws`: su área de trabajo de Azure Machine Learning.
    > * `model`: el modelo registrado que se implementará.
    > * `inference_config`: la configuración de inferencia del modelo.
    >
    > Para más información sobre la definición de estas variables, consulte [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

## <a name="prepare-for-deployment"></a>Preparar la implementación

Antes de realizar la implementación, debe definir qué necesita para ejecutar el modelo como un servicio web. En la lista siguiente se describen los principales elementos necesarios para una implementación:

* Un __script de entrada__. Este script acepta solicitudes, puntúa la solicitud mediante el modelo y devuelve los resultados.

    > [!IMPORTANT]
    > El script de entrada es específico del modelo; debe comprender el formato de los datos de la solicitud entrante, el formato de los datos que espera el modelo y el formato de los datos que se devuelven a los clientes.
    >
    > Si los datos de la solicitud están en un formato que el modelo no puede usar, el script puede transformarlos a un formato aceptable. También puede transformar la respuesta antes de devolverla al cliente.

    > [!IMPORTANT]
    > El SDK de Azure Machine Learning no proporciona una manera para que el servicio web acceda al almacén de datos o a los conjuntos de datos. Si necesita que el modelo implementado tenga acceso a los datos almacenados fuera de la implementación, como en una cuenta de Azure Storage, debe desarrollar una solución de código personalizada mediante el SDK pertinente. Por ejemplo, el [SDK de Azure Storage para Python](https://github.com/Azure/azure-storage-python).
    >
    > Otra alternativa que puede funcionar para su escenario son las [predicciones por lotes](how-to-use-parallel-run-step.md), que proporcionan acceso a los almacenes de datos cuando se realiza la puntuación.

    Para más información sobre los scripts de entrada, consulte [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

* **Dependencias**, como scripts de asistente o paquetes de Python/Conda, necesarias para ejecutar el modelo o el script de entrada.

Estas entidades se encapsulan en una __configuración de inferencia__. La configuración de inferencia hace referencia al script de entrada y a otras dependencias.

> [!IMPORTANT]
> Al crear una configuración de inferencia para su uso con Azure App Service, debe usar un objeto [Environment](https://docs.microsoft.com//python/api/azureml-core/azureml.core.environment%28class%29?view=azure-ml-py&preserve-view=true). Tenga en cuenta que si va a definir su propio entorno personalizado, debe enumerar azureml-defaults con la versión >= 1.0.45 como dependencia de PIP. Este paquete contiene la funcionalidad necesaria para hospedar el modelo como un servicio web. En el ejemplo siguiente se muestra cómo crear un objeto de entorno y cómo usarlo con una configuración de inferencia:
>
> ```python
> from azureml.core.environment import Environment
> from azureml.core.conda_dependencies import CondaDependencies
> from azureml.core.model import InferenceConfig
>
> # Create an environment and add conda dependencies to it
> myenv = Environment(name="myenv")
> # Enable Docker based environment
> myenv.docker.enabled = True
> # Build conda dependencies
> myenv.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'],
>                                                            pip_packages=['azureml-defaults'])
> inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
> ```

Para obtener más información sobre los entornos, consulte el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md).

Para más información sobre la configuración de inferencia, consulte [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

> [!IMPORTANT]
> Al realizar la implementación en Azure App Service, no es necesario crear una __configuración de implementación__.

## <a name="create-the-image"></a>Crear la imagen

Para crear la imagen de Docker que se implementa en Azure App Service, use [Model.package](https://docs.microsoft.com//python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#&preserve-view=truepackage-workspace--models--inference-config-none--generate-dockerfile-false-). El siguiente fragmento de código muestra cómo compilar una nueva imagen a partir del modelo y la configuración de inferencia:

> [!NOTE]
> En el fragmento de código se da por supuesto que `model` contiene un modelo registrado y que `inference_config` contiene la configuración del entorno de inferencia. Para más información, consulte [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

```python
from azureml.core import Model

package = Model.package(ws, [model], inference_config)
package.wait_for_creation(show_output=True)
# Display the package location/ACR path
print(package.location)
```

Si `show_output=True`, se muestra la salida del proceso de compilación de Docker. Una vez finalizado el proceso, la imagen se ha creado en Azure Container Registry para su área de trabajo. Una vez creada la imagen, se muestra la ubicación en Azure Container Registry. La ubicación devuelta tiene el formato `<acrinstance>.azurecr.io/package@sha256:<imagename>`. Por ejemplo, `myml08024f78fd10.azurecr.io/package@sha256:20190827151241`.

> [!IMPORTANT]
> Guarde la información de la ubicación, ya que se usa al implementar la imagen.

## <a name="deploy-image-as-a-web-app"></a>Implementación de la imagen como una aplicación web

1. Use el siguiente comando para obtener las credenciales de inicio de sesión del registro de contenedor de Azure que contiene la imagen. Reemplace `<acrinstance>` por el valor devuelto anteriormente de `package.location`:

    ```azurecli-interactive
    az acr credential show --name <myacr>
    ```

    La salida de este comando es similar al siguiente documento JSON:

    ```json
    {
    "passwords": [
        {
        "name": "password",
        "value": "Iv0lRZQ9762LUJrFiffo3P4sWgk4q+nW"
        },
        {
        "name": "password2",
        "value": "=pKCxHatX96jeoYBWZLsPR6opszr==mg"
        }
    ],
    "username": "myml08024f78fd10"
    }
    ```

    Guarde el valor de __username__ y uno de los dos de __passwords__.

1. Si aún no tiene un grupo de recursos o un plan de App Service para implementar el servicio, los siguientes comandos muestran cómo crear ambos:

    ```azurecli-interactive
    az group create --name myresourcegroup --location "West Europe"
    az appservice plan create --name myplanname --resource-group myresourcegroup --sku B1 --is-linux
    ```

    En este ejemplo, se usa un plan de tarifa __Básico__ (`--sku B1`).

    > [!IMPORTANT]
    > Las imágenes que crea Azure Machine Learning usan Linux, por lo que debe usar el parámetro `--is-linux`.

1. Para crear la aplicación web, use el comando siguiente. Reemplace `<app-name>` por el nombre que desea usar. Reemplace `<acrinstance>` y `<imagename>` por los valores de `package.location`devueltos anteriormente:

    ```azurecli-interactive
    az webapp create --resource-group myresourcegroup --plan myplanname --name <app-name> --deployment-container-image-name <acrinstance>.azurecr.io/package@sha256:<imagename>
    ```

    Este comando devuelve información similar al siguiente documento JSON:

    ```json
    {
    "adminSiteName": null,
    "appServicePlanName": "myplanname",
    "geoRegion": "West Europe",
    "hostingEnvironmentProfile": null,
    "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myplanname",
    "kind": "linux",
    "location": "West Europe",
    "maximumNumberOfWorkers": 1,
    "name": "myplanname",
    < JSON data removed for brevity. >
    "targetWorkerSizeId": 0,
    "type": "Microsoft.Web/serverfarms",
    "workerTierName": null
    }
    ```

    > [!IMPORTANT]
    > En este momento ya se ha creado la aplicación web. Sin embargo, dado que no ha proporcionado las credenciales a la instancia de Azure Container Registry que contiene la imagen, la aplicación web no está activa. En el paso siguiente, se proporciona la información de autenticación para el registro de contenedor.

1. Para proporcionar la aplicación web con las credenciales necesarias para acceder al registro de contenedor, use el siguiente comando. Reemplace `<app-name>` por el nombre que desea usar. Reemplace `<acrinstance>` y `<imagename>` por los valores de `package.location`devueltos anteriormente. Reemplace `<username>` y `<password>` por la información de inicio de sesión de ACR recuperada anteriormente:

    ```azurecli-interactive
    az webapp config container set --name <app-name> --resource-group myresourcegroup --docker-custom-image-name <acrinstance>.azurecr.io/package@sha256:<imagename> --docker-registry-server-url https://<acrinstance>.azurecr.io --docker-registry-server-user <username> --docker-registry-server-password <password>
    ```

    Este comando devuelve información similar al siguiente documento JSON:

    ```json
    [
    {
        "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
        "slotSetting": false,
        "value": "false"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_URL",
        "slotSetting": false,
        "value": "https://myml08024f78fd10.azurecr.io"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_USERNAME",
        "slotSetting": false,
        "value": "myml08024f78fd10"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
        "slotSetting": false,
        "value": null
    },
    {
        "name": "DOCKER_CUSTOM_IMAGE_NAME",
        "value": "DOCKER|myml08024f78fd10.azurecr.io/package@sha256:20190827195524"
    }
    ]
    ```

En este momento la aplicación web comienza a cargar la imagen.

> [!IMPORTANT]
> Pueden transcurrir varios minutos hasta que se cargue la imagen. Para supervisar el progreso, use el siguiente comando:
>
> ```azurecli-interactive
> az webapp log tail --name <app-name> --resource-group myresourcegroup
> ```
>
> Una vez que la imagen se ha cargado y el sitio está activo, el registro muestra el siguiente mensaje: `Container <container name> for site <app-name> initialized successfully and is ready to serve requests`.

Una vez implementada la imagen, puede encontrar el nombre de host con el siguiente comando:

```azurecli-interactive
az webapp show --name <app-name> --resource-group myresourcegroup
```

Este comando devuelve información similar al nombre de host siguiente: `<app-name>.azurewebsites.net`. Use este valor como parte de la __dirección URL base__ del servicio.

## <a name="use-the-web-app"></a>Uso de la aplicación web

El servicio web que pasa las solicitudes al modelo se encuentra en `{baseurl}/score`. Por ejemplo, `https://<app-name>.azurewebsites.net/score`. El siguiente código de Python muestra cómo enviar datos a la dirección URL y mostrar la respuesta:

```python
import requests
import json

scoring_uri = "https://mywebapp.azurewebsites.net/score"

headers = {'Content-Type':'application/json'}

test_sample = json.dumps({'data': [
    [1,2,3,4,5,6,7,8,9,10],
    [10,9,8,7,6,5,4,3,2,1]
]})

response = requests.post(scoring_uri, data=test_sample, headers=headers)
print(response.status_code)
print(response.elapsed)
print(response.json())
```

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a configurar la aplicación web en la documentación de [App Service en Linux](/azure/app-service/containers/).
* Obtenga más información sobre el escalado en [Introducción al escalado automático en Azure](/azure/azure-monitor/platform/autoscale-get-started?toc=%2fazure%2fapp-service%2ftoc.json).
* [Uso de un certificado TLS/SSL en Azure App Service](/azure/app-service/configure-ssl-certificate-in-code).
* [Configuración de una aplicación de App Service para usar la información de inicio de sesión de Azure Active Directory](/azure/app-service/configure-authentication-provider-aad).
* [Consumir un modelo de ML que está implementado como un servicio web](how-to-consume-web-service.md)
