---
title: Solución de problemas de implementación de Docker
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo abordar, solucionar y resolver los errores comunes de implementación de Docker con Azure Kubernetes Service y Azure Container Instances mediante Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: clauren42
ms.author: clauren
ms.reviewer: jmartens
ms.date: 08/06/2020
ms.topic: conceptual
ms.custom: troubleshooting, contperfq4, devx-track-python
ms.openlocfilehash: 3f8a3c705878e212e6a26670e20b5a81a3f2a6ba
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87904384"
---
# <a name="troubleshoot-docker-deployment-of-models-with-azure-kubernetes-service-and-azure-container-instances"></a>Solución de problemas con la implementación de Docker de modelos con Azure Kubernetes Service y Azure Container Instances 

Obtenga información sobre cómo abordar y solucionar los errores comunes de implementación de Docker con Azure Container Instances (ACI) y Azure Kubernetes Service (AKS) mediante Azure Machine Learning.

## <a name="prerequisites"></a>Requisitos previos

* Una **suscripción de Azure**. Si no tiene una ya, pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
* El [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py).
* La[CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).
* La [extensión de la CLI para Azure Machine Learning](reference-azure-machine-learning-cli.md).
* Para depurar localmente, debe tener una instalación de Docker en funcionamiento en el sistema local.

    Para comprobar la instalación de Docker, use el comando `docker run hello-world` desde un símbolo del sistema o terminal. Para obtener información sobre la instalación de Docker o la solución de problemas de Docker, consulte la [Documentación de Docker](https://docs.docker.com/).

## <a name="steps-for-docker-deployment-of-machine-learning-models"></a>Pasos para la implementación de Docker de modelos de Machine Learning

Al implementar un modelo en Azure Machine Learning, el sistema realiza una serie de tareas.

El enfoque recomendado para la implementación de modelos se realiza a través de la API [Model.deploy()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model%28class%29?view=azure-ml-py#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) mediante un objeto de [entorno](how-to-use-environments.md) como parámetro de entrada. En este caso, el servicio crea una imagen base de Docker durante la fase de implementación y monta los modelos necesarios en una llamada. Las tareas de implementación básicas son:

1. Registrar el modelo en el registro de modelos del área de trabajo.

2. Defina la configuración de inferencia:
    1. Cree un objeto [Entorno](how-to-use-environments.md) basado en las dependencias que especifique en el archivo yaml de entorno o use uno de nuestros entornos adquiridos.
    2. Cree una configuración de inferencia (objeto InferenceConfig) basada en el entorno y el script de puntuación.

3. Implemente el modelo en el servicio de instancia de contenedor de Azure (ACI) o Azure Kubernetes Service (AKS).

Más información sobre este proceso en la introducción a la [administración de modelos](concept-model-management-and-deployment.md).

## <a name="before-you-begin"></a>Antes de empezar

Si tiene algún problema, lo primero es dividir la tarea de implementación (descrita anteriormente) en pasos individuales para aislar el problema.

Suponiendo que esté usando el método de implementación nuevo o recomendado a través de la API [Model.deploy()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model%28class%29?view=azure-ml-py#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) con un objeto [Entorno](how-to-use-environments.md) como parámetro de entrada, su código puede dividirse en tres importantes pasos:

1. Registre el modelo. Este es un código de ejemplo:

    ```python
    from azureml.core.model import Model


    # register a model out of a run record
    model = best_run.register_model(model_name='my_best_model', model_path='outputs/my_model.pkl')

    # or, you can register a file or a folder of files as a model
    model = Model.register(model_path='my_model.pkl', model_name='my_best_model', workspace=ws)
    ```

2. Defina la configuración de inferencia para la implementación:

    ```python
    from azureml.core.model import InferenceConfig
    from azureml.core.environment import Environment


    # create inference configuration based on the requirements defined in the YAML
    myenv = Environment.from_conda_specification(name="myenv", file_path="myenv.yml")
    inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
    ```

3. Implemente el modelo mediante la configuración de inferencia creada en el paso anterior:

    ```python
    from azureml.core.webservice import AciWebservice


    # deploy the model
    aci_config = AciWebservice.deploy_configuration(cpu_cores=1, memory_gb=1)
    aci_service = Model.deploy(workspace=ws,
                           name='my-service',
                           models=[model],
                           inference_config=inference_config,
                           deployment_config=aci_config)
    aci_service.wait_for_deployment(show_output=True)
    ```

Una vez que haya dividido el proceso de implementación en tareas individuales, se pueden observar algunos de los errores más comunes.

## <a name="debug-locally"></a>Depuración local

Si tiene problemas al implementar un modelo en ACI o AKS, intente implementarlo como un servicio web local. El uso de un servicio web local facilita la solución de problemas. Se descarga la imagen de Docker que contiene el modelo y se inicia en el sistema local.

Puede encontrar un ejemplo de [cuaderno de implementación local](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/deploy-to-local/register-model-deploy-local.ipynb) en el repositorio de [MachineLearningNotebooks](https://github.com/Azure/MachineLearningNotebooks) para explorar un ejemplo ejecutable.

> [!WARNING]
> No se admiten las implementaciones de servicios web locales en escenarios de producción.

Para implementar de forma local, modifique el código para usar `LocalWebservice.deploy_configuration()` con el fin de crear una configuración de implementación. Luego use `Model.deploy()` para implementar el servicio. En el ejemplo siguiente se implementa un modelo (incluido en la variable de modelo) como un servicio web local:

```python
from azureml.core.environment import Environment
from azureml.core.model import InferenceConfig, Model
from azureml.core.webservice import LocalWebservice


# Create inference configuration based on the environment definition and the entry script
myenv = Environment.from_conda_specification(name="env", file_path="myenv.yml")
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
# Create a local deployment, using port 8890 for the web service endpoint
deployment_config = LocalWebservice.deploy_configuration(port=8890)
# Deploy the service
service = Model.deploy(
    ws, "mymodel", [model], inference_config, deployment_config)
# Wait for the deployment to complete
service.wait_for_deployment(True)
# Display the port that the web service is available on
print(service.port)
```

Si va a definir su propio YAML de especificación de Conda, debe enumerar azureml-defaults con la versión >= 1.0.45 como dependencia de PIP. Este paquete contiene la funcionalidad necesaria para hospedar el modelo como un servicio web.

En este punto, se puede trabajar con el servicio de forma habitual. Por ejemplo, en el código siguiente se muestra cómo enviar datos al servicio:

```python
import json

test_sample = json.dumps({'data': [
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
]})

test_sample = bytes(test_sample, encoding='utf8')

prediction = service.run(input_data=test_sample)
print(prediction)
```

Para obtener más información sobre cómo personalizar el entorno de Python, consulte el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md). 

### <a name="update-the-service"></a>Actualizar el servicio

Durante las pruebas locales, es posible que deba actualizar el archivo `score.py` para agregar un registro o intentar resolver los problemas que haya descubierto. Para recargar los cambios realizados en el archivo `score.py`, use `reload()`. Por ejemplo, el código siguiente recarga el script para el servicio y luego envía datos. Los datos se puntúan con el archivo `score.py` actualizado:

> [!IMPORTANT]
> El método `reload` solo está disponible para las implementaciones locales. Para información sobre cómo actualizar una implementación en otro destino de proceso, consulte [cómo actualizar el servicio web](how-to-deploy-update-web-service.md).

```python
service.reload()
print(service.run(input_data=test_sample))
```

> [!NOTE]
> El script se recarga desde la ubicación que especifica el objeto `InferenceConfig` que usa el servicio.

Para cambiar el modelo, las dependencias de Conda o la configuración de implementación, use [update()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice%28class%29?view=azure-ml-py#update--args-). En el siguiente ejemplo se actualiza el modelo que usa el servicio:

```python
service.update([different_model], inference_config, deployment_config)
```

### <a name="delete-the-service"></a>Eliminación del servicio

Para eliminar el servicio, use [delete()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice%28class%29?view=azure-ml-py#delete--).

### <a name="inspect-the-docker-log"></a><a id="dockerlog"></a> Inspección del registro de Docker

Puede imprimir los mensajes detallados de registro del motor de Docker desde el objeto de servicio. Puede ver el registro para ACI, AKS y las implementaciones locales. En el ejemplo siguiente se muestra cómo imprimir los registros.

```python
# if you already have the service object handy
print(service.get_logs())

# if you only know the name of the service (note there might be multiple services with the same name but different version number)
print(ws.webservices['mysvc'].get_logs())
```
Si ve que la línea `Booting worker with pid: <pid>` se produce varias veces en los registros, significa que no hay suficiente memoria para iniciar el trabajo.
Puede solucionar el error aumentando el valor de `memory_gb` en `deployment_config`.
 
## <a name="container-cannot-be-scheduled"></a>No se puede programar el contenedor

Al implementar un servicio en un destino de proceso de Azure Kubernetes Service, Azure Machine Learning intentará programar el servicio con la cantidad de recursos solicitada. Si pasados 5 minutos no hay ningún nodo disponible en el clúster con la cantidad adecuada de recursos disponibles, se producirá un error en la implementación con el mensaje `Couldn't Schedule because the kubernetes cluster didn't have available resources after trying for 00:05:00`. Para solucionar este error, se pueden agregar más nodos, cambiar la SKU de los nodos o cambiar los requisitos de recursos de su servicio. 

El mensaje de error suele indicar el recurso del cual se necesita más. Por ejemplo, si ve un mensaje de error que indica `0/3 nodes are available: 3 Insufficient nvidia.com/gpu`, significa que el servicio requiere GPU y que hay tres nodos en el clúster que no tienen GPU disponibles. Esto se podría solucionar con la incorporación de más nodos si se usa una SKU de GPU, el cambio a una SKU habilitada para GPU si no se estaba usando o el cambio del entorno para que no se necesite GPU.  

## <a name="service-launch-fails"></a>Error en el inicio del servicio

Después de que la imagen se haya compilado correctamente, el sistema intenta iniciar un contenedor con el uso de la configuración de implementación. Como parte del proceso de puesta en marcha del contenedor, el sistema invoca la función `init()` en el script de puntuación. Si hay excepciones no detectadas en la función `init()`, es posible que vea el error **CrashLoopBackOff** en el mensaje de error.

Use la información que aparece en la sección [Inspeccionar el registro de Docker](#dockerlog) para comprobar los registros.

## <a name="function-fails-get_model_path"></a>Error en la función: get_model_path()

A menudo, en la función `init()` en el script de puntuación, se llama a la función [Model.get_model_path()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#get-model-path-model-name--version-none---workspace-none-) para buscar un archivo de modelo o una carpeta de archivos de modelo en el contenedor. Si no se encuentra el archivo de modelo o la carpeta, se produce un error en la función. La manera más fácil de depurar este error es ejecutar el siguiente código de Python en el shell del contenedor:

```python
from azureml.core.model import Model
import logging
logging.basicConfig(level=logging.DEBUG)
print(Model.get_model_path(model_name='my-best-model'))
```

En este ejemplo se imprime la ruta de acceso local (relativa a `/var/azureml-app`) en el contenedor donde el script de puntuación espera encontrar el archivo de modelo o la carpeta. A continuación, puede comprobar si el archivo o la carpeta están realmente donde se espera que estén.

Establecer el nivel de registro en DEBUG puede provocar el registro de información adicional, que puede resultar útil para identificar el error.

## <a name="function-fails-runinput_data"></a>Error en la función: run(input_data)

Si el servicio se implementa correctamente, pero se bloquea al publicar datos en el punto de conexión de puntuación, puede agregar una instrucción de captura de errores en la función `run(input_data)` para que devuelva un mensaje de error detallado en su lugar. Por ejemplo:

```python
def run(input_data):
    try:
        data = json.loads(input_data)['data']
        data = np.array(data)
        result = model.predict(data)
        return json.dumps({"result": result.tolist()})
    except Exception as e:
        result = str(e)
        # return error message back to the client
        return json.dumps({"error": result})
```

**Nota**: La devolución de mensajes de error de la llamada `run(input_data)` se debe realizar solo con fines de depuración. Por motivos de seguridad, no debe devolver los mensajes de error de este modo en un entorno de producción.

## <a name="http-status-code-502"></a>Código de estado HTTP 502

Un código de estado 502 indica que el servicio ha producido una excepción o se ha bloqueado en el método `run()` del archivo score.py. Utilice la información de este artículo para depurar el archivo.

## <a name="http-status-code-503"></a>Código de estado HTTP 503

Las implementaciones de Azure Kubernetes Service admiten el escalado automático, que permite que las réplicas se agreguen para admitir una carga adicional. Sin embargo, el escalador automático está diseñado para controlar cambios **graduales** en la carga. Si recibe grandes picos en las solicitudes por segundo, los clientes pueden recibir un código de estado HTTP 503.

Hay dos cosas que ayudan a impedir los códigos de estado 503:

* Cambiar el nivel de uso en el que el escalado automático crea nuevas réplicas.
    
    De forma predeterminada, se establece el uso de destino de escalado automático en el 70 %, lo que significa que el servicio puede controlar los picos en las solicitudes por segundo (RPS) de hasta un 30 %. Puede ajustar el objetivo de uso mediante el establecimiento de `autoscale_target_utilization` en un valor inferior.

    > [!IMPORTANT]
    > Este cambio no hace que las réplicas se creen *más rápidamente*. En lugar de eso, se crean con un umbral de uso más bajo. En lugar de esperar a que el servicio se use en un 70 %, si se cambia el valor a un 30 %, las réplicas se crearán cuando se produzca este 30 % de uso.
    
    Si el servicio web ya usa el número máximo de réplicas actuales y siguen apareciendo los códigos de estado 503, aumente el valor de `autoscale_max_replicas` con el fin de aumentar el número máximo de réplicas.

* Cambie el número mínimo de réplicas. El aumento en el número mínimo de réplicas proporciona un grupo más grande para controlar los picos entrantes.

    Para aumentar el número mínimo de réplicas, establezca `autoscale_min_replicas` en un valor superior. Puede calcular las réplicas necesarias mediante el código siguiente, reemplazando los valores por valores específicos del proyecto:

    ```python
    from math import ceil
    # target requests per second
    targetRps = 20
    # time to process the request (in seconds)
    reqTime = 10
    # Maximum requests per container
    maxReqPerContainer = 1
    # target_utilization. 70% in this example
    targetUtilization = .7

    concurrentRequests = targetRps * reqTime / targetUtilization

    # Number of container replicas
    replicas = ceil(concurrentRequests / maxReqPerContainer)
    ```

    > [!NOTE]
    > Si recibe picos de solicitudes más grandes que el número mínimo nuevo de réplicas que se puede controlar, es posible que reciba códigos de estado 503 otra vez. Por ejemplo, a medida que el tráfico al servicio aumente, es posible que deba aumentar el número mínimo de réplicas.

Para obtener más información sobre cómo configurar `autoscale_target_utilization`, `autoscale_max_replicas`, y `autoscale_min_replicas`, vea la referencia de módulo [AksWebservice](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.akswebservice?view=azure-ml-py).

## <a name="http-status-code-504"></a>Código de estado HTTP 504

Un código de estado 504 indica que la solicitud ha agotado el tiempo de espera. El tiempo de espera predeterminado es 1 minuto.

Puede aumentar el tiempo de espera o intentar acelerar el servicio modificando el archivo score.py para quitar llamadas innecesarias. Si estas acciones no solucionan el problema, use la información de este artículo para depurar el archivo score.py. El código puede estar en un estado sin respuesta o en un bucle infinito.

## <a name="advanced-debugging"></a>Depuración avanzada

En algunos casos, es posible que tenga que depurar interactivamente el código de Python incluido en la implementación de modelo. Por ejemplo, si el script de entrada presenta errores y no se puede determinar el motivo mediante un registro adicional. Mediante el uso de Visual Studio Code y debugpy, puede conectarse al código que se ejecuta en el contenedor de Docker. Para más información, visite la [guía de depuración interactiva en VS Code](how-to-debug-visual-studio-code.md#debug-and-troubleshoot-deployments).

## <a name="next-steps"></a>Pasos siguientes

Más información acerca de la implementación:

* [Cómo implementar y dónde](how-to-deploy-and-where.md)
* [Tutorial: Entrenamiento e implementación de modelos](tutorial-train-models-with-aml.md)
