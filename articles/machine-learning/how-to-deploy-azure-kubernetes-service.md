---
title: Implementación de modelos de Machine Learning en Kubernetes Service
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo implementar modelos de Azure Machine Learning como un servicio web con Azure Kubernetes Service.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 06/23/2020
ms.openlocfilehash: 9503abf147ee89ec03e7e1317df823426ea37b1c
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87758890"
---
# <a name="deploy-a-model-to-an-azure-kubernetes-service-cluster"></a>Implementación de un modelo en un clúster de Azure Kubernetes Service
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Aprenda a usar Azure Machine Learning para implementar un modelo como un servicio web en Azure Kubernetes Service (AKS). Azure Kubernetes Service se recomienda para implementaciones de producción a gran escala. Úselo si necesita una o varias de las siguientes funcionalidades:

- __Tiempo de respuesta rápido__.
- __Escalado automático__ del servicio implementado.
- Opciones de  __aceleración de hardware__, como GPU y matrices de puertas programables (FPGA).

> [!IMPORTANT]
> El escalado de clústeres no se proporciona a través del SDK de Azure Machine Learning. Para más información acerca del escalado de nodos en un clúster de Azure Kubernetes Service, consulte [Escalado del número de nodos en un clúster de Azure Kubernetes Service](../aks/scale-cluster.md).

En Azure Kubernetes Service, la implementación se realiza en un clúster de AKS que está __conectado a su área de trabajo__. Hay dos formas de conectar un clúster de AKS a un área de trabajo:

* Cree el clúster de AKS mediante el SDK de Azure Machine Learning, la CLI de Machine Learning o [Azure Machine Learning Studio](https://ml.azure.com). Este proceso conecta automáticamente el clúster al área de trabajo.
* Conecte el clúster de AKS existente a un área de trabajo de Azure Machine Learning. Un clúster se puede conectar mediante el SDK de Azure Machine Learning, la CLI de Machine Learning o Azure Machine Learning Studio.

El clúster de AKS y el área de trabajo de AML pueden estar en distintos grupos de recursos.

> [!IMPORTANT]
> El proceso de creación o de conexión es una tarea que se realiza una sola vez. Una vez que un clúster de AKS está conectado al área de trabajo, puede usarlo para las implementaciones. Cuando deje de necesitar el clúster de AKS puede desasociarlo o eliminarlo. Una vez que lo haga, ya no podrá realizar ninguna implementación en el clúster.

> [!IMPORTANT]
> Se recomienda realizar una depuración local antes de la implementación en el servicio web. Para obtener más información, vea [Depuración local](https://docs.microsoft.com/azure/machine-learning/how-to-troubleshoot-deployment#debug-locally).
>
> También puede consultar Azure Machine Learning: [Implementación en el cuaderno local](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/deployment/deploy-to-local).

## <a name="prerequisites"></a>Requisitos previos

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

- Un modelo de Machine Learning registrado en el área de trabajo. Si no tiene un modelo registrado, consulte el artículo en el que se explica [cómo y dónde se implementan los modelos](how-to-deploy-and-where.md).

- La [extensión de la CLI de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md), el [SDK de Python para Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py) o la [extensión de Visual Studio Code para Azure Machine Learning](tutorial-setup-vscode-extension.md).

- En los fragmentos de código de __Python__ de este artículo se supone que se han establecido las siguientes variables:

    * `ws`: establézcalo en su área de trabajo.
    * `model`: establézcalo en el modelo registrado.
    * `inference_config`: establézcalo en la configuración de inferencia del modelo.

    Para más información acerca de cómo establecer estas variables, consulte el artículo en el que se explica [cómo y dónde se implementan los modelos](how-to-deploy-and-where.md).

- En los fragmentos de código de la __CLI__ de este artículo se supone que ha creado un documento `inferenceconfig.json`. Para más información acerca cómo crear este documento, consulte el artículo en el que se explica [cómo y dónde se implementan los modelos](how-to-deploy-and-where.md).

- Si necesita implementar un Standard Load Balancer (SLB) en el clúster en lugar de un Basic Load Balancer (BLB), cree un clúster en el portal de AKS, la CLI o el SDK y, a continuación, asócielo al área de trabajo de AML.

- Si adjunta un clúster de AKS, que tiene un [intervalo IP autorizado habilitado para tener acceso al servidor de API](https://docs.microsoft.com/azure/aks/api-server-authorized-ip-ranges), habilite los intervalos IP del plano de control de AML del clúster de AKS. El plano de control de AML se implementa entre regiones emparejadas e implementa pods de inferencia en el clúster de AKS. Sin acceso al servidor de la API, no se pueden implementar los pods de inferencia. Use el [intervalo IP](https://www.microsoft.com/en-us/download/confirmation.aspx?id=56519) para las [regiones emparejadas]( https://docs.microsoft.com/azure/best-practices-availability-paired-regions) al habilitar los intervalos IP en un clúster de AKS.

__Los intervalos IP autorizados solo funcionan con Standard Load Balancer.__
 
 - El nombre del proceso DEBE ser único dentro de un área de trabajo.
   - El nombre es obligatorio y debe tener una longitud de entre 3 y 24 caracteres.
   - Los caracteres válidos son mayúsculas y minúsculas, dígitos y el carácter -.
   - El nombre debe empezar con una letra
   - El nombre debe ser único en todos los procesos existentes dentro de una región de Azure. Verá una alerta si el nombre elegido no es único
   
 - Si quiere implementar modelos en nodos de GPU o en nodos de FPGA (o en cualquier SKU específica), debe crear un clúster con la SKU específica. No se admite la creación de un grupo de nodos secundarios en un clúster existente ni la implementación de modelos en el grupo de nodos secundarios.
 
 



## <a name="create-a-new-aks-cluster"></a>Creación de un clúster de AKS

**Tiempo estimado**: Aproximadamente 10 minutos.

Crear o asociar un clúster de AKS es un proceso único en el área de trabajo. Puede volver a usar este clúster con diferentes implementaciones. Si elimina el clúster o el grupo de recursos que lo contiene, tendrá que crear un nuevo clúster la próxima vez que tenga que realizar una implementación. Puede tener varios clústeres de AKS asociados al área de trabajo.
 
Azure Machine Learning ya admite el uso de un servicio Azure Kubernetes Service que tenga habilitado Private Link.
Para crear un clúster de AKS privado, siga los documentos que se muestran [aquí](https://docs.microsoft.com/azure/aks/private-clusters).

> [!TIP]
> Si quiere proteger el clúster de AKS mediante una instancia de Azure Virtual Network, primero debe crear la red virtual. Para más información, consulte [Protección de los trabajos de experimentación e inferencia con Azure Virtual Network](how-to-enable-virtual-network.md#aksvnet).

Si desea crear un clúster de AKS para __desarrollo__,  __validación__y __pruebas__, en lugar de producción, en __cluster purpose__ puede especificar __dev test__.

> [!WARNING]
> Si establece `cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST`, el clúster que se crea no es adecuado para un tráfico de nivel de producción y puede aumentar los tiempos de inferencia. Los clústeres de desarrollo y pruebas tampoco garantizan la tolerancia a errores. Se recomiendan al menos dos CPU virtuales para los clústeres de desarrollo y pruebas.

En los siguientes ejemplos se muestra cómo crear un clúster de AKS mediante el SDK y la CLI:

**Uso del SDK**

```python
from azureml.core.compute import AksCompute, ComputeTarget

# Use the default configuration (you can also provide parameters to customize this).
# For example, to create a dev/test cluster, use:
# prov_config = AksCompute.provisioning_configuration(cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST)
prov_config = AksCompute.provisioning_configuration()

aks_name = 'myaks'
# Create the cluster
aks_target = ComputeTarget.create(workspace = ws,
                                    name = aks_name,
                                    provisioning_configuration = prov_config)

# Wait for the create process to complete
aks_target.wait_for_completion(show_output = True)
```

> [!IMPORTANT]
> Para [`provisioning_configuration()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py), si elige valores personalizados para `agent_count` y `vm_size`, y `cluster_purpose` no es `DEV_TEST`, deberá asegurarse de que `agent_count` multiplicado por `vm_size` sea mayor o igual que 12 CPU virtuales. Por ejemplo, si usa un valor de `vm_size` de "Standard_D3_v2", que tiene cuatro CPU virtuales, debe elegir un valor de `agent_count` de 3 o mayor.
>
> El SDK de Azure Machine Learning no admite escalar un clúster de AKS. Para escalar los nodos en el clúster, use la interfaz de usuario para el clúster de AKS en Azure Machine Learning Studio. Solo puede cambiar el número de nodos, no el tamaño de máquina virtual del clúster.

Para más información acerca de las clases, los métodos y los parámetros que se usan en este ejemplo, consulte los siguientes documentos de referencia:

* [AksCompute.ClusterPurpose](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose?view=azure-ml-py)
* [AksCompute.provisioning_configuration](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)
* [ComputeTarget.create](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#create-workspace--name--provisioning-configuration-)
* [ComputeTarget.wait_for_completion](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#wait-for-completion-show-output-false-)

**Uso de la CLI**

```azurecli
az ml computetarget create aks -n myaks
```

Para más información, consulte la referencia de [az ml computetarget create aks](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/create?view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-create-aks).

## <a name="attach-an-existing-aks-cluster"></a>Asociación de un clúster de AKS ya existente

**Tiempo estimado:** Aproximadamente 5 minutos.

Si ya dispone de un clúster de AKS en su suscripción a Azure y es de la versión 1.17 o anterior, puede usarlo para implementar la imagen.

> [!TIP]
> El clúster de AKS existente puede estar en una región de Azure diferente que su área de trabajo de Azure Machine Learning.
>
> Si quiere proteger el clúster de AKS mediante una instancia de Azure Virtual Network, primero debe crear la red virtual. Para más información, consulte [Protección de los trabajos de experimentación e inferencia con Azure Virtual Network](how-to-enable-virtual-network.md#aksvnet).

Cuando se asocia un clúster de AKS a un área de trabajo, puede definir cómo utilizará el clúster estableciendo el parámetro `cluster_purpose`.

Si no establece el parámetro `cluster_purpose`, o establece `cluster_purpose = AksCompute.ClusterPurpose.FAST_PROD`, el clúster deberá tener al menos 12 CPU virtuales disponibles.

Si establece `cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST`, el clúster no necesitará tener 12 CPU virtuales. Se recomiendan al menos dos CPU virtuales para desarrollo y pruebas. No obstante, un clúster que está configurado para desarrollo y pruebas no es adecuado para un tráfico de nivel de producción y puede aumentar los tiempos de inferencia. Los clústeres de desarrollo y pruebas tampoco garantizan la tolerancia a errores.

> [!WARNING]
> No cree varios datos adjuntos simultáneos en el mismo clúster de AKS desde su área de trabajo. Por ejemplo, adjuntar un clúster de AKS a un área de trabajo con dos nombres diferentes. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.
>
> Si desea volver a conectar un clúster de AKS, por ejemplo, para cambiar la configuración de TLS u otra configuración del clúster, primero debe eliminar la conexión existente mediante [AksCompute.detach()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#detach--).

Para más información acerca de cómo crear un clúster de AKS mediante la CLI de Azure o Azure Portal, consulte los artículos siguientes:

* [Creación de un clúster de AKS (CLI)](https://docs.microsoft.com/cli/azure/aks?toc=%2Fazure%2Faks%2FTOC.json&bc=%2Fazure%2Fbread%2Ftoc.json&view=azure-cli-latest#az-aks-create)
* [Creación de un clúster de AKS: Portal](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough-portal?view=azure-cli-latest)
* [Creación de un clúster de AKS (plantilla de ARM en las plantillas de inicio rápido de Azure)](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aks-azml-targetcompute)

En los ejemplos siguientes se muestra cómo asociar un clúster de AKS existente a un área de trabajo:

**Uso del SDK**

```python
from azureml.core.compute import AksCompute, ComputeTarget
# Set the resource group that contains the AKS cluster and the cluster name
resource_group = 'myresourcegroup'
cluster_name = 'myexistingcluster'

# Attach the cluster to your workgroup. If the cluster has less than 12 virtual CPUs, use the following instead:
# attach_config = AksCompute.attach_configuration(resource_group = resource_group,
#                                         cluster_name = cluster_name,
#                                         cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST)
attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                         cluster_name = cluster_name)
aks_target = ComputeTarget.attach(ws, 'myaks', attach_config)

# Wait for the attach process to complete
aks_target.wait_for_completion(show_output = True)
```

Para más información acerca de las clases, los métodos y los parámetros que se usan en este ejemplo, consulte los siguientes documentos de referencia:

* [AksCompute.attach_configuration()](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)
* [AksCompute.ClusterPurpose](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose?view=azure-ml-py)
* [AksCompute.attach](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#attach-workspace--name--attach-configuration-)

**Uso de la CLI**

Para asociar un clúster existente mediante la CLI, es preciso obtener el identificador de recurso del clúster existente. Para obtener este valor, use el comando siguiente. Reemplace `myexistingcluster` por el nombre del clúster de AKS. Reemplace `myresourcegroup` por el grupo de recursos que contiene el clúster:

```azurecli
az aks show -n myexistingcluster -g myresourcegroup --query id
```

Este comando devuelve un valor similar al siguiente texto:

```text
/subscriptions/{GUID}/resourcegroups/{myresourcegroup}/providers/Microsoft.ContainerService/managedClusters/{myexistingcluster}
```

Para conectar el clúster existente a un área de trabajo, use el siguiente comando. Reemplace `aksresourceid` por el valor devuelto por el comando anterior. Reemplace `myresourcegroup` por el grupo de recursos que contiene el área de trabajo. Reemplace `myworkspace` por el nombre del área de trabajo.

```azurecli
az ml computetarget attach aks -n myaks -i aksresourceid -g myresourcegroup -w myworkspace
```

Para más información, consulte la referencia de [az ml computetarget attach aks](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/attach?view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-attach-aks).

## <a name="deploy-to-aks"></a>Implementación en AKS

Para implementar un modelo en Azure Kubernetes Service, cree una __configuración de implementación__ que describa los recursos de proceso necesarios. Por ejemplo, el número de núcleos y la memoria. También necesita una __configuración de inferencia__, que describe el entorno necesario para hospedar el modelo y el servicio web. Para más información sobre cómo crear la configuración de inferencia, consulte [Cómo y dónde implementar modelos](how-to-deploy-and-where.md).

### <a name="using-the-sdk"></a>Uso del SDK

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

* [AksCompute](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.akscompute?view=azure-ml-py)
* [AksWebservice.deploy_configuration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aks.aksservicedeploymentconfiguration?view=azure-ml-py)
* [Model.deploy](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-)
* [Webservice.wait_for_deployment](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice%28class%29?view=azure-ml-py#wait-for-deployment-show-output-false-)

### <a name="using-the-cli"></a>Uso de la CLI

Para realizar una implementación con la CLI, use el siguiente comando. Reemplace `myaks` por el nombre del destino de proceso de AKS. Reemplace `mymodel:1` por el nombre y la versión del modelo registrado. Reemplace `myservice` por el nombre que quiere asignar a este servicio:

```azurecli-interactive
az ml model deploy -ct myaks -m mymodel:1 -n myservice -ic inferenceconfig.json -dc deploymentconfig.json
```

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-aks-deploy-config.md)]

Para obtener más información, consulte la referencia [az ml model deploy](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy).

### <a name="using-vs-code"></a>Uso de Visual Studio Code

Para obtener información acerca del uso de Visual Studio Code, consulte cómo se [realiza la implementación en AKS a través de la extensión de Visual Studio Code](tutorial-train-deploy-image-classification-model-vscode.md#deploy-the-model).

> [!IMPORTANT]
> La implementación a través de Visual Studio Code requiere que el clúster de AKS se cree o se adjunte al área de trabajo de antemano.

### <a name="understand-the-deployment-processes"></a>Comprender los procesos de implementación

La palabra "implementación" se usa en Kubernetes y Azure Machine Learning. "Implementación" tiene significados muy diferentes en estos dos contextos. En Kubernetes, una `Deployment` es una entidad concreta, que se especifica con un archivo YAML declarativo. Una `Deployment` de Kubernetes tiene un ciclo de vida definido y relaciones concretas con otras entidades de Kubernetes, como `Pods` y `ReplicaSets`. Puede obtener información sobre Kubernetes desde documentos y vídeos en [¿Qué es Kubernetes?](https://aka.ms/k8slearning).

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
    1. Si no existe, el sistema genera una imagen (que se almacenará en la memoria caché y se registrará en la instancia de ACR del área de trabajo).
1. Descargar el archivo de proyecto comprimido en el almacenamiento temporal del nodo de proceso.
1. Descomprimir el archivo de proyecto.
1. El nodo de proceso que ejecuta `python <entry script> <arguments>`.
1. Guardar registros, archivos de modelo y otros archivos escritos en `./outputs` en la cuenta de almacenamiento asociada con el área de trabajo.
1. Reducir verticalmente el proceso, incluida la eliminación del almacenamiento temporal (se relaciona con Kubernetes).

Cuando se usa AKS, Kubernetes controla el escalado vertical y la reducción vertical del proceso mediante el Dockerfile generado o encontrado como se describió anteriormente. 

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
> Si necesita regenerar una clave, use [`service.regen_key`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py).

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

* [Protección de experimentos e inferencias en una red virtual](how-to-enable-virtual-network.md)
* [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md)
* [Solución de problemas de implementación](how-to-troubleshoot-deployment.md)
* [Actualización de servicio web](how-to-deploy-update-web-service.md)
* [Uso de TLS para proteger un servicio web con Azure Machine Learning](how-to-secure-web-service.md)
* [Consumir un modelo de ML que está implementado como un servicio web](how-to-consume-web-service.md)
* [Supervisión de los modelos de Azure Machine Learning con Application Insights](how-to-enable-app-insights.md)
* [Recopilar datos de modelos en producción](how-to-enable-data-collection.md)
