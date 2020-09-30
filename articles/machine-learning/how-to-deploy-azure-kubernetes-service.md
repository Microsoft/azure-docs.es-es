---
title: Implementación de modelos de Machine Learning en Kubernetes Service
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo implementar modelos de Azure Machine Learning como un servicio web con Azure Kubernetes Service.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, contperfq1
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 09/01/2020
ms.openlocfilehash: c7c3825d97a5566a43346febc7bbf86a4b242b54
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905699"
---
# <a name="deploy-a-model-to-an-azure-kubernetes-service-cluster"></a>Implementación de un modelo en un clúster de Azure Kubernetes Service


Aprenda a usar Azure Machine Learning para implementar un modelo como un servicio web en Azure Kubernetes Service (AKS). Azure Kubernetes Service se recomienda para implementaciones de producción a gran escala. Úselo si necesita una o varias de las siguientes funcionalidades:

- __Tiempo de respuesta rápido__
- __Escalado automático__ del servicio implementado
- __Logging__
- __Recopilación de datos de modelos__
- __Autenticación__
- __Finalización de TLS__
- Opciones de __aceleración de hardware__, como GPU y matrices de puertas programables (FPGA)

En Azure Kubernetes Service, la implementación se realiza en un clúster de AKS que está __conectado a su área de trabajo__. Para obtener información sobre cómo conectar un clúster de AKS al área de trabajo, consulte [Creación y conexión de un clúster de Azure Kubernetes Service](how-to-create-attach-kubernetes.md).

> [!IMPORTANT]
> Se recomienda depurar localmente antes de la implementación en el servicio web. Para obtener más información, consulte [Depuración de forma local](https://docs.microsoft.com/azure/machine-learning/how-to-troubleshoot-deployment#debug-locally).
>
> También puede consultar Azure Machine Learning: [Implementación en el cuaderno local](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/deployment/deploy-to-local).

## <a name="prerequisites"></a>Requisitos previos

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

- Un modelo de Machine Learning registrado en el área de trabajo. Si no tiene un modelo registrado, consulte el artículo en el que se explica [cómo y dónde se implementan los modelos](how-to-deploy-and-where.md).

- La [extensión de la CLI de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md), el [SDK de Python para Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true) o la [extensión de Visual Studio Code para Azure Machine Learning](tutorial-setup-vscode-extension.md).

- En los fragmentos de código de __Python__ de este artículo se supone que se han establecido las siguientes variables:

    * `ws`: establézcalo en su área de trabajo.
    * `model`: establézcalo en el modelo registrado.
    * `inference_config`: establézcalo en la configuración de inferencia del modelo.

    Para más información acerca de cómo establecer estas variables, consulte el artículo en el que se explica [cómo y dónde se implementan los modelos](how-to-deploy-and-where.md).

- En los fragmentos de código de la __CLI__ de este artículo se supone que ha creado un documento `inferenceconfig.json`. Para más información acerca cómo crear este documento, consulte el artículo en el que se explica [cómo y dónde se implementan los modelos](how-to-deploy-and-where.md).

- Un clúster de Azure Kubernetes Service conectado al área de trabajo. Para obtener más información, consulte [Creación y conexión de un clúster de Azure Kubernetes Service](how-to-create-attach-kubernetes.md).

    - Si quiere implementar modelos en nodos de GPU o en nodos de FPGA (o en cualquier SKU específica), debe crear un clúster con la SKU específica. No se admite la creación de un grupo de nodos secundarios en un clúster existente ni la implementación de modelos en el grupo de nodos secundarios.

## <a name="understand-the-deployment-processes"></a>Comprender los procesos de implementación

La palabra "implementación" se usa en Kubernetes y Azure Machine Learning. "Implementación" tiene significados diferentes en estos dos contextos. En Kubernetes, una `Deployment` es una entidad concreta, que se especifica con un archivo YAML declarativo. Una `Deployment` de Kubernetes tiene un ciclo de vida definido y relaciones concretas con otras entidades de Kubernetes, como `Pods` y `ReplicaSets`. Puede obtener información sobre Kubernetes desde documentos y vídeos en [¿Qué es Kubernetes?](https://aka.ms/k8slearning).

En Azure Machine Learning, "implementación" se usa en el sentido general de poner a disposición y limpiar los recursos del proyecto. Los pasos que Azure Machine Learning considera parte de la implementación son:

1. Comprimir los archivos de la carpeta del proyecto y omitir los especificados en .amlignore o .gitignore.
1. Escalar verticalmente el clúster de proceso (se relaciona con Kubernetes).
1. Compilar o descargar el Dockerfile en el nodo de proceso (se relaciona con Kubernetes).
    1. El sistema calcula un valor de hash de: 
        - La imagen base 
        - Pasos personalizados de Docker (consulte [Implementación de un modelo con una imagen base de Docker personalizada](https://docs.microsoft.com/azure/machine-learning/how-to-deploy-custom-docker-image))
        - El archivo YAML de definición de Conda (consulte [Creación y uso de entornos de software en Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/how-to-use-environments))
    1. El sistema utiliza este valor de hash como clave en una búsqueda de la instancia de Azure Container Registry (ACR) del área de trabajo.
    1. Si no la encuentra, busca una coincidencia en la instancia de ACR global.
    1. Si no existe, el sistema genera una imagen (que se almacenará en la memoria caché y se insertará en la instancia de ACR del área de trabajo).
1. Descargar el archivo de proyecto comprimido en el almacenamiento temporal del nodo de proceso.
1. Descomprimir el archivo de proyecto.
1. El nodo de proceso que ejecuta `python <entry script> <arguments>`.
1. Guardar registros, archivos de modelo y otros archivos escritos en `./outputs` en la cuenta de almacenamiento asociada con el área de trabajo.
1. Reducir verticalmente el proceso, incluida la eliminación del almacenamiento temporal (se relaciona con Kubernetes).

### <a name="azure-ml-router"></a>Enrutador de Azure ML

El componente de front-end (azureml-fe) que enruta las solicitudes de inferencia entrantes a los servicios implementados se escala automáticamente según sea necesario. El escalado de azureml-fe se basa en el propósito y el tamaño (número de nodos) del clúster de AKS. El propósito y los nodos del clúster se configuran cuando se [crea o conecta un clúster de AKS](how-to-create-attach-kubernetes.md). Hay un servicio azureml-fe por clúster, que puede ejecutarse en varios pods.

> [!IMPORTANT]
> Al usar un clúster configurado como __Desarrollo y pruebas__, el escalador automático está **deshabilitado**.

Azureml-fe escala tanto verticalmente para usar más núcleos cono horizontalmente para usar más pods. Al tomar la decisión de escalar verticalmente, se utiliza el tiempo que se tarda en enrutar las solicitudes de inferencia entrantes. Si este tiempo supera el umbral, se produce una escalabilidad vertical. Si el tiempo para enrutar las solicitudes entrantes sigue superando el umbral, se produce una escalabilidad horizontal.

Al reducir vertical y horizontalmente, se emplea el uso de la CPU. Si se cumple el umbral de uso de la CPU, el front-end se reducirá verticalmente en primer lugar. Si el uso de la CPU desciende hasta el umbral de reducción horizontal, se produce una operación de reducción horizontal. El escalado vertical y horizontal solo se producirá si hay suficientes recursos de clúster disponibles.

## <a name="deploy-to-aks"></a>Implementación en AKS

Para implementar un modelo en Azure Kubernetes Service, cree una __configuración de implementación__ que describa los recursos de proceso necesarios. Por ejemplo, el número de núcleos y la memoria. También necesita una __configuración de inferencia__, que describe el entorno necesario para hospedar el modelo y el servicio web. Para más información sobre cómo crear la configuración de inferencia, consulte [Cómo y dónde implementar modelos](how-to-deploy-and-where.md).

> [!NOTE]
> El número de modelos que se implementará se limita a 1000 modelos por implementación (por contenedor).

<a id="using-the-cli"></a>

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.webservice import AksWebservice, Webservice
from azureml.core.model import Model

aks_target = AksCompute(ws,"myaks")
# If deploying to a cluster configured for dev/test, ensure that it was created with enough
# cores and memory to handle this deployment configuration. Note that memory is also used by
# things such as dependencies and AML components.
deployment_config = AksWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config, aks_target)
service.wait_for_deployment(show_output = True)
print(service.state)
print(service.get_logs())
```

Para más información acerca de las clases, los métodos y los parámetros que se usan en este ejemplo, consulte los siguientes documentos de referencia:

* [AksCompute](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.akscompute?view=azure-ml-py&preserve-view=true)
* [AksWebservice.deploy_configuration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aks.aksservicedeploymentconfiguration?view=azure-ml-py&preserve-view=true)
* [Model.deploy](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#&preserve-view=truedeploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-)
* [Webservice.wait_for_deployment](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice%28class%29?view=azure-ml-py#&preserve-view=truewait-for-deployment-show-output-false-)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para realizar una implementación con la CLI, use el siguiente comando. Reemplace `myaks` por el nombre del destino de proceso de AKS. Reemplace `mymodel:1` por el nombre y la versión del modelo registrado. Reemplace `myservice` por el nombre que quiere asignar a este servicio:

```azurecli-interactive
az ml model deploy -ct myaks -m mymodel:1 -n myservice -ic inferenceconfig.json -dc deploymentconfig.json
```

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-aks-deploy-config.md)]

Para obtener más información, consulte la referencia [az ml model deploy](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy).

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Para obtener información acerca del uso de Visual Studio Code, consulte cómo se [realiza la implementación en AKS a través de la extensión de Visual Studio Code](tutorial-train-deploy-image-classification-model-vscode.md#deploy-the-model).

> [!IMPORTANT]
> La implementación a través de Visual Studio Code requiere que el clúster de AKS se cree o se adjunte al área de trabajo de antemano.

---

### <a name="autoscaling"></a>Escalado automático

El componente que controla el escalado automático para las implementaciones de modelos de Azure ML es azureml-fe, que es un enrutador de solicitudes inteligente. Dado que todas las solicitudes de inferencia pasan por él, tiene los datos necesarios para escalar automáticamente los modelos implementados.

> [!IMPORTANT]
> * **No habilite el Escalador horizontal automático de pods (HPA) de Kubernetes para las implementaciones de modelos**. Si lo hace, los dos componentes de escalado automático competirán entre sí. Azureml-fe está diseñado para el escalado automático de modelos implementados por Azure ML, donde HPA tendría que adivinar o aproximar el uso del modelo a partir de una métrica genérica, como el uso de la CPU o una configuración de métricas personalizada.
> 
> * **Azureml-fe no escala el número de nodos en un clúster de AKS**, ya que esto podría provocar un aumento inesperado en los costos. En su lugar, **escala el número de réplicas para el modelo** dentro de los límites del clúster físico. Si necesita escalar el número de nodos dentro del clúster, puede escalar manualmente el clúster o [configurar el escalador automático del clúster de AKS](/azure/aks/cluster-autoscaler).

Para controlar el escalado automático, se puede establecer `autoscale_target_utilization`, `autoscale_min_replicas` y `autoscale_max_replicas` para el servicio web de AKS. En el ejemplo siguiente se muestra cómo habilitar el escalado automático:

```python
aks_config = AksWebservice.deploy_configuration(autoscale_enabled=True, 
                                                autoscale_target_utilization=30,
                                                autoscale_min_replicas=1,
                                                autoscale_max_replicas=4)
```

Las decisiones de escalar o reducir verticalmente se basan en el uso de las réplicas de contenedores actuales. El número de réplicas que están ocupadas (procesando una solicitud) dividido por el número total de réplicas actuales es el uso actual. Si este número supera `autoscale_target_utilization`, se crean más réplicas. Si es menor, se reducen las réplicas. De manera predeterminada, la utilización de destino es del 70 %.

Las decisiones de agregar réplicas son diligentes y rápidas (aproximadamente un segundo). Las decisiones de quitar réplicas son conservadoras (aproximadamente un minuto).

Puede calcular las réplicas necesarias mediante el código siguiente:

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

Para obtener más información sobre cómo configurar `autoscale_target_utilization`, `autoscale_max_replicas` y `autoscale_min_replicas`, consulte la referencia de módulo [AksWebservice](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.akswebservice?view=azure-ml-py).

## <a name="deploy-models-to-aks-using-controlled-rollout-preview"></a>Implementación de modelos en AKS mediante el lanzamiento controlado (versión preliminar)

Analice y promocione las versiones del modelo de un modo controlado mediante puntos de conexión. Puede implementar hasta seis versiones detrás de un solo punto de conexión. Los puntos de conexión proporcionan las siguientes funcionalidades:

* Configure el __porcentaje de tráfico de puntuación que se envía a cada punto de conexión__. Por ejemplo, enrute el 20 % del tráfico al punto de conexión "test" y el 80 % a "production".

    > [!NOTE]
    > Si no tiene en cuenta el 100 % del tráfico, el porcentaje restante se enruta a la versión __predeterminada__ del punto de conexión. Por ejemplo, si configura la versión del punto de conexión "test" para obtener el 10 % del tráfico y "prod" para un 30 %, el 60 % restante se envía a la versión predeterminada del punto de conexión.
    >
    > La primera versión del punto de conexión creada se configura automáticamente como la predeterminada. Para cambiarlo, establezca `is_default=True` al crear o actualizar una versión del punto de conexión.
     
* Etiquete una versión del punto de conexión como __control__ o __tratamiento__. Por ejemplo, la versión del punto de conexión de producción actual podría ser el control, mientras que los nuevos modelos posibles se implementan como versiones de tratamiento. Después de evaluar el rendimiento de las versiones de tratamiento, si una supera el rendimiento del control actual, podría promoverse a ser la nueva versión de producción o control.

    > [!NOTE]
    > Solo puede tener __un__ control. Puede tener varias versiones de tratamiento.

Puede habilitar App Insights para ver las métricas operativas de los puntos de conexión y las versiones implementadas.

### <a name="create-an-endpoint"></a>Crear un punto de conexión
Cuando esté listo para implementar los modelos, cree un punto de conexión de puntuación e implemente la primera versión. En el ejemplo siguiente se muestra cómo implementar y crear el punto de conexión mediante el SDK. La primera implementación se definirá como la versión predeterminada, lo que significa que el percentil de tráfico sin especificar en todas las versiones irá a la versión predeterminada.  

> [!TIP]
> En el ejemplo siguiente, la configuración establece que la versión inicial del punto de conexión va a controlar el 20 % del tráfico. Como este es el primer punto de conexión, también es la versión predeterminada. Y como no tenemos ninguna otra versión para el otro 80 % del tráfico, también se enruta a la versión predeterminada. Hasta que se implementen otras versiones que tomen un porcentaje del tráfico, esta recibe el 100 % del tráfico.

```python
import azureml.core,
from azureml.core.webservice import AksEndpoint
from azureml.core.compute import AksCompute
from azureml.core.compute import ComputeTarget
# select a created compute
compute = ComputeTarget(ws, 'myaks')
namespace_name= endpointnamespace
# define the endpoint and version name
endpoint_name = "mynewendpoint"
version_name= "versiona"
# create the deployment config and define the scoring traffic percentile for the first deployment
endpoint_deployment_config = AksEndpoint.deploy_configuration(cpu_cores = 0.1, memory_gb = 0.2,
                                                              enable_app_insights = True,
                                                              tags = {'sckitlearn':'demo'},
                                                              description = "testing versions",
                                                              version_name = version_name,
                                                              traffic_percentile = 20)
 # deploy the model and endpoint
 endpoint = Model.deploy(ws, endpoint_name, [model], inference_config, endpoint_deployment_config, compute)
 # Wait for he process to complete
 endpoint.wait_for_deployment(True)
 ```

### <a name="update-and-add-versions-to-an-endpoint"></a>Actualizar y agregar versiones a un punto de conexión

Agregue otra versión al punto de conexión y configure el percentil de tráfico de puntuación que va a la versión. Hay dos tipos de versiones, una versión control y otra de tratamiento. Puede haber varias versiones de tratamiento para ayudar a comparar con una sola versión de control.

> [!TIP]
> La segunda versión, creada por el siguiente fragmento de código, acepta el 10 % del tráfico. La primera versión está configurada para un 20 %, por lo que solo el 30 % del tráfico está configurado para versiones específicas. El 70 % restante se envía a la primera versión del punto de conexión, ya que también es la versión predeterminada.

 ```python
from azureml.core.webservice import AksEndpoint

# add another model deployment to the same endpoint as above
version_name_add = "versionb"
endpoint.create_version(version_name = version_name_add,
                        inference_config=inference_config,
                        models=[model],
                        tags = {'modelVersion':'b'},
                        description = "my second version",
                        traffic_percentile = 10)
endpoint.wait_for_deployment(True)
```

Actualice las versiones existentes o elimínelas en un punto de conexión. Puede cambiar el tipo predeterminado de la versión, el tipo de control y el percentil de tráfico. En el ejemplo siguiente, la segunda versión aumenta su tráfico al 40 % y ahora es la predeterminada.

> [!TIP]
> Después del fragmento de código siguiente, la segunda versión es ahora la predeterminada. Ahora está configurada para el 40 %, mientras que la versión original todavía está configurada para un 20 %. Esto significa que las configuraciones de versión no tienen en cuenta el 40 % del tráfico. El tráfico restante se enrutará a la segunda versión, ya que ahora es la predeterminada. Recibe el 80 % del tráfico.

 ```python
from azureml.core.webservice import AksEndpoint

# update the version's scoring traffic percentage and if it is a default or control type
endpoint.update_version(version_name=endpoint.versions["versionb"].name,
                        description="my second version update",
                        traffic_percentile=40,
                        is_default=True,
                        is_control_version_type=True)
# Wait for the process to complete before deleting
endpoint.wait_for_deployment(true)
# delete a version in an endpoint
endpoint.delete_version(version_name="versionb")

```

## <a name="web-service-authentication"></a>Autenticación de servicio web

Cuando se implementa en Azure Kubernetes Service, la autenticación __basada en claves__ se habilita de manera predeterminada. También puede habilitar la autenticación __basada en tokens__. Para la autenticación basada en tokens, los clientes deben usar una cuenta de Azure Active Directory para solicitar un token de autenticación, que se usa para hacer solicitudes al servicio implementado.

Para __deshabilitar__ la autenticación, establezca el parámetro `auth_enabled=False` al crear la configuración de implementación. En el siguiente ejemplo se deshabilita la autenticación mediante el SDK:

```python
deployment_config = AksWebservice.deploy_configuration(cpu_cores=1, memory_gb=1, auth_enabled=False)
```

Para obtener información sobre la autenticación desde una aplicación cliente, consulte [Consumir un modelo de Azure Machine Learning que está implementado como un servicio web](how-to-consume-web-service.md).

### <a name="authentication-with-keys"></a>Autenticación con claves

Si la autenticación con claves está habilitada, puede usar el método `get_keys` para recuperar una clave de autenticación primaria y secundaria:

```python
primary, secondary = service.get_keys()
print(primary)
```

> [!IMPORTANT]
> Si necesita regenerar una clave, use [`service.regen_key`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py&preserve-view=true).

### <a name="authentication-with-tokens"></a>Autenticación con tokens

Para habilitar la autenticación por tokens, establezca el parámetro `token_auth_enabled=True` cuando cree o actualice una implementación. En el ejemplo siguiente se habilita la autenticación por tokens con el SDK:

```python
deployment_config = AksWebservice.deploy_configuration(cpu_cores=1, memory_gb=1, token_auth_enabled=True)
```

Si la autenticación por tokens está habilitada, puede usar el método `get_token` para recuperar un token JWT y la hora de expiración de los tokens:

```python
token, refresh_by = service.get_token()
print(token)
```

> [!IMPORTANT]
> Tendrá que solicitar un nuevo token después de la hora del token `refresh_by`.
>
> Microsoft recomienda crear el área de trabajo de Azure Machine Learning en la misma región que el clúster de Azure Kubernetes Service. Para autenticarse con un token, el servicio web hará una llamada a la región en la que se crea el área de trabajo de Azure Machine Learning. Si la región del área de trabajo no está disponible, no se podrá capturar un token para el servicio web, incluso si el clúster está en una región distinta del área de trabajo. Esto produce que la autenticación basada en tokens no esté disponible hasta que la región del área de trabajo vuelva a estar disponible. Además, cuanto mayor sea la distancia entre la región del clúster y la región del área de trabajo, más tiempo tardará la captura de un token.
>
> Para recuperar un token, debe usar el SDK de Azure Machine Learning o el comando [az ml service get-access-token](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/service?view=azure-cli-latest#ext-azure-cli-ml-az-ml-service-get-access-token).

## <a name="next-steps"></a>Pasos siguientes

* [Protección del entorno de inferencia con Azure Virtual Network](how-to-secure-inferencing-vnet.md)
* [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md)
* [Solución de problemas de implementación](how-to-troubleshoot-deployment.md)
* [Actualización de servicio web](how-to-deploy-update-web-service.md)
* [Uso de TLS para proteger un servicio web con Azure Machine Learning](how-to-secure-web-service.md)
* [Consumir un modelo de ML que está implementado como un servicio web](how-to-consume-web-service.md)
* [Supervisión de los modelos de Azure Machine Learning con Application Insights](how-to-enable-app-insights.md)
* [Recopilar datos de modelos en producción](how-to-enable-data-collection.md)
