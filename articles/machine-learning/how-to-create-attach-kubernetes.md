---
title: Creación y conexión de Azure Kubernetes Service
titleSuffix: Azure Machine Learning
description: Azure Kubernetes Service (AKS) se puede usar para implementar un modelo de Machine Learning como servicio web. Aprenda a crear un nuevo clúster de AKS a través de Azure Machine Learning. También aprenderá a conectar el clúster de AKS existente a un área de trabajo de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 09/01/2020
ms.openlocfilehash: 034fbf991c97ba7b9f51b5508c35df1889280562
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90897895"
---
# <a name="create-and-attach-an-azure-kubernetes-service-cluster"></a>Creación y conexión de un clúster de Azure Kubernetes Service

Azure Machine Learning puede implementar modelos de Machine Learning entrenados en Azure Kubernetes Service. Sin embargo, primero debe __crear__ un clúster de Azure Kubernetes Service (AKS) desde el área de trabajo de Azure ML o __adjuntar__ un clúster de AKS existente. En este artículo se proporciona información sobre cómo crear y anexar un clúster.

## <a name="prerequisites"></a>Requisitos previos

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

- La [extensión de la CLI de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md), el [SDK de Python para Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true) o la [extensión de Visual Studio Code para Azure Machine Learning](tutorial-setup-vscode-extension.md).

- Si planea usar una Azure Virtual Network para proteger la comunicación entre el área de trabajo de Azure ML y el clúster de AKS, lea el artículo [Aislamiento de red durante el entrenamiento y la inferencia](how-to-enable-virtual-network.md).

## <a name="limitations"></a>Limitaciones

- Si necesita implementar un **Standard Load Balancer (SLB)** en el clúster en lugar de un Basic Load Balancer (BLB), cree un clúster en el portal de AKS, la CLI o el SDK y, a continuación, **adjuntarlo** al área de trabajo de AML.

- Si tiene una instancia de Azure Policy que restringe la creación de direcciones IP públicas, se producirá un error en la creación del clúster de AKS. AKS requiere una dirección IP pública para el [tráfico de salida](/azure/aks/limit-egress-traffic). En este artículo de tráfico de salida también se proporcionan instrucciones para bloquear el tráfico de salida desde el clúster a través de la dirección IP pública, excepto en el caso de algunos nombres de dominio completos. Hay dos formas de habilitar una dirección IP pública:
    - El clúster puede usar la dirección IP pública creada de forma predeterminada con BLB o SLB.
    - El clúster se puede crear sin una dirección IP pública y luego configurar una dirección IP pública con un firewall con una ruta definida por el usuario. Para obtener más información, consulte [Personalización de la salida de un clúster con una ruta definida por el usuario](/azure/aks/egress-outboundtype).
    
    El plano de control AML no se comunica con esta dirección IP pública. Se comunica con el plano de control AKS para las implementaciones. 

- Si **adjunta** un clúster de AKS, que tiene un [intervalo IP autorizado habilitado para tener acceso al servidor de API](/azure/aks/api-server-authorized-ip-ranges), habilite los intervalos IP del plano de control de AML del clúster de AKS. El plano de control de AML se implementa entre regiones emparejadas e implementa pods de inferencia en el clúster de AKS. Sin acceso al servidor de la API, no se pueden implementar los pods de inferencia. Use el [intervalo IP](https://www.microsoft.com/download/confirmation.aspx?id=56519) para las [regiones emparejadas](/azure/best-practices-availability-paired-regions) al habilitar los intervalos IP en un clúster de AKS.

    Los intervalos IP autorizados solo funcionan con Standard Load Balancer.

- Si quiere usar un clúster de AKS privado (mediante Azure Private Link), primero debe crear el clúster y, a continuación, **adjuntarlo** al área de trabajo. Para obtener más información, consulte [Creación de un clúster privado de Azure Kubernetes Service](/azure/aks/private-clusters).

- El nombre de proceso del clúster de AKS DEBE ser único en el área de trabajo de Azure ML.
    - El nombre es obligatorio y debe tener una longitud de entre 3 y 24 caracteres.
    - Los caracteres válidos son mayúsculas y minúsculas, dígitos y el carácter -.
    - El nombre debe empezar con una letra.
    - El nombre debe ser único en todos los procesos existentes dentro de una región de Azure. Verá una alerta si el nombre elegido no es único.
   
 - Si quiere implementar modelos en nodos de **GPU** o en nodos de **FPGA** (o en cualquier SKU específica), debe crear un clúster con la SKU específica. No se admite la creación de un grupo de nodos secundarios en un clúster existente ni la implementación de modelos en el grupo de nodos secundarios.
 
- Al crear o adjuntar un clúster, puede seleccionar si quiere crearlo para __desarrollo y pruebas__ o __producción__. Si quiere crear un clúster de AKS para __desarrollo__,  __validación__y __pruebas__, en lugar de producción, especifique el __propósito del clúster__ en __desarrollo y pruebas__. Si no especifica el propósito del clúster, se crea un clúster de __producción__. 

    > [!IMPORTANT]
    > Un clúster de __desarrollo y pruebas__ no es adecuado para un tráfico de nivel de producción y puede aumentar los tiempos de inferencia. Los clústeres de desarrollo y pruebas tampoco garantizan la tolerancia a errores.

- Al crear o adjuntar un clúster, si se va a usar para __producción__, debe contener al menos 12 __CPU virtuales__. El número de CPU virtuales se puede calcular multiplicando el __número de nodos__ en el clúster por el __número de núcleos__ proporcionados por el tamaño de máquina virtual seleccionado. Por ejemplo, si usa un valor de VM de "Standard_D3_v2", que tiene 4 CPU virtuales, debe seleccionar un número de nodos de 3 o más.

    Para los clústeres de __desarrollo y pruebas__, se recomiendan al menos dos CPU virtuales.

- El SDK de Azure Machine Learning no admite escalar un clúster de AKS. Para escalar los nodos en el clúster, use la interfaz de usuario para el clúster de AKS en Azure Machine Learning Studio. Solo puede cambiar el número de nodos, no el tamaño de máquina virtual del clúster. Para más información acerca del escalado de nodos en un clúster de AKS, consulte los artículos siguientes:

    - [Escalado manual del número de nodos en un clúster de AKS](../aks/scale-cluster.md)
    - [Configuración del escalador automático en AKS](../aks/cluster-autoscaler.md)

## <a name="azure-kubernetes-service-version"></a>Versión de Azure Kubernetes Service

Azure Kubernetes Service le permite crear un clúster mediante una variedad de versiones de Kubernetes. Para obtener más información sobre las versiones disponibles, consulte [Versiones de Kubernetes compatibles en Azure Kubernetes Service (AKS)](/azure/aks/supported-kubernetes-versions).

Cuando **crea** un clúster de Azure Kubernetes Service con uno de los métodos siguientes, *no tiene una opción en la versión* del clúster que se crea:

* Azure Machine Learning Studio o la sección de Azure Machine Learning de Azure Portal.
* Extensión de Machine Learning para la CLI de Azure.
* SDK de Azure Machine Learning.

Estos métodos de creación de un clúster de AKS usan la versión __predeterminada__ del clúster. *La versión predeterminada cambia con el tiempo* a medida que están disponibles las nuevas versiones de Kubernetes.

Cuando **adjunta** un clúster de AKS existente, se admiten todas las versiones de AKS compatibles actualmente.

> [!NOTE]
> Puede haber casos extremos en los que tenga un clúster anterior que ya no se admita. En este caso, la operación de adjuntar devolverá un error y mostrará las versiones admitidas actualmente.
>
> Puede adjuntar **versiones preliminares**. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. La compatibilidad con las versiones preliminares puede ser limitada. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="available-and-default-versions"></a>Versiones disponibles y predeterminadas

Para buscar las versiones de AKS disponibles y predeterminadas, use el comando [az aks get-versions](/cli/azure/aks?view=azure-cli-latest#az_aks_get_versions) de la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest). Por ejemplo, el comando siguiente devuelve las versiones disponibles en la región Oeste de EE. UU.:

```azurecli-interactive
az aks get-versions -l westus -o table
```

La salida de este comando es similar al texto siguiente:

```text
KubernetesVersion    Upgrades
-------------------  ----------------------------------------
1.18.6(preview)      None available
1.18.4(preview)      1.18.6(preview)
1.17.9               1.18.4(preview), 1.18.6(preview)
1.17.7               1.17.9, 1.18.4(preview), 1.18.6(preview)
1.16.13              1.17.7, 1.17.9
1.16.10              1.16.13, 1.17.7, 1.17.9
1.15.12              1.16.10, 1.16.13
1.15.11              1.15.12, 1.16.10, 1.16.13
```

Para buscar la versión predeterminada que se usa al **crear** un clúster a través de Azure Machine Learning, puede usar el parámetro `--query` para seleccionar la versión predeterminada:

```azurecli-interactive
az aks get-versions -l westus --query "orchestrators[?default == `true`].orchestratorVersion" -o table
```

La salida de este comando es similar al texto siguiente:

```text
Result
--------
1.16.13
```

Si quiere **comprobar mediante programación las versiones disponibles**, utilice la API REST [Container Service Client - List Orchestrators](https://docs.microsoft.com/rest/api/container-service/container%20service%20client/listorchestrators). Para buscar las versiones disponibles, examine las entradas en las que `orchestratorType` sea `Kubernetes`. Las entradas `orchestrationVersion` asociadas contienen las versiones disponibles que se pueden **adjuntar** al área de trabajo.

Para buscar la versión predeterminada que se usa al **crear** un clúster a través de Azure Machine Learning, busque la entrada en la que `orchestratorType` sea `Kubernetes` y `default` sea `true`. El valor de `orchestratorVersion` asociado es la versión predeterminada. El siguiente fragmento de código JSON es un ejemplo de entrada:

```json
...
 {
        "orchestratorType": "Kubernetes",
        "orchestratorVersion": "1.16.13",
        "default": true,
        "upgrades": [
          {
            "orchestratorType": "",
            "orchestratorVersion": "1.17.7",
            "isPreview": false
          }
        ]
      },
...
```

## <a name="create-a-new-aks-cluster"></a>Creación de un clúster de AKS

**Tiempo estimado**: Aproximadamente 10 minutos.

Crear o asociar un clúster de AKS es un proceso único en el área de trabajo. Puede volver a usar este clúster con diferentes implementaciones. Si elimina el clúster o el grupo de recursos que lo contiene, tendrá que crear un nuevo clúster la próxima vez que tenga que realizar una implementación. Puede tener varios clústeres de AKS asociados al área de trabajo.

En el ejemplo siguiente se muestra cómo crear un clúster de AKS mediante el SDK y la CLI:

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.compute import AksCompute, ComputeTarget

# Use the default configuration (you can also provide parameters to customize this).
# For example, to create a dev/test cluster, use:
# prov_config = AksCompute.provisioning_configuration(cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST)
prov_config = AksCompute.provisioning_configuration()

# Example configuration to use an existing virtual network
# prov_config.vnet_name = "mynetwork"
# prov_config.vnet_resourcegroup_name = "mygroup"
# prov_config.subnet_name = "default"
# prov_config.service_cidr = "10.0.0.0/16"
# prov_config.dns_service_ip = "10.0.0.10"
# prov_config.docker_bridge_cidr = "172.17.0.1/16"

aks_name = 'myaks'
# Create the cluster
aks_target = ComputeTarget.create(workspace = ws,
                                    name = aks_name,
                                    provisioning_configuration = prov_config)

# Wait for the create process to complete
aks_target.wait_for_completion(show_output = True)
```

Para más información acerca de las clases, los métodos y los parámetros que se usan en este ejemplo, consulte los siguientes documentos de referencia:

* [AksCompute.ClusterPurpose](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose?view=azure-ml-py&preserve-view=true)
* [AksCompute.provisioning_configuration](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)
* [ComputeTarget.create](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#create-workspace--name--provisioning-configuration-)
* [ComputeTarget.wait_for_completion](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#wait-for-completion-show-output-false-)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az ml computetarget create aks -n myaks
```

Para más información, consulte la referencia de [az ml computetarget create aks](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/create?view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-create-aks).

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para obtener información sobre cómo crear un clúster de AKS en el portal, consulte [Creación de destinos de proceso en Azure Machine Learning Studio](how-to-create-attach-compute-studio.md#inference-clusters).

---

## <a name="attach-an-existing-aks-cluster"></a>Asociación de un clúster de AKS ya existente

**Tiempo estimado:** Aproximadamente 5 minutos.

Si ya dispone de un clúster de AKS en su suscripción a Azure y es de la versión 1.17 o anterior, puede usarlo para implementar la imagen.

> [!TIP]
> El clúster de AKS existente puede estar en una región de Azure diferente que su área de trabajo de Azure Machine Learning.


> [!WARNING]
> No cree varios datos adjuntos simultáneos en el mismo clúster de AKS desde su área de trabajo. Por ejemplo, adjuntar un clúster de AKS a un área de trabajo con dos nombres diferentes. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.
>
> Si desea volver a conectar un clúster de AKS, por ejemplo, para cambiar la configuración de TLS u otra configuración del clúster, primero debe eliminar la conexión existente mediante [AksCompute.detach()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#detach--).

Para más información acerca de cómo crear un clúster de AKS mediante la CLI de Azure o Azure Portal, consulte los artículos siguientes:

* [Creación de un clúster de AKS (CLI)](https://docs.microsoft.com/cli/azure/aks?toc=%2Fazure%2Faks%2FTOC.json&bc=%2Fazure%2Fbread%2Ftoc.json&view=azure-cli-latest#az-aks-create)
* [Creación de un clúster de AKS: Portal](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough-portal?view=azure-cli-latest)
* [Creación de un clúster de AKS (plantilla de ARM en las plantillas de inicio rápido de Azure)](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aks-azml-targetcompute)

En el ejemplo siguiente se muestra cómo adjuntar un clúster de AKS existente a un área de trabajo:

# <a name="python"></a>[Python](#tab/python)

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
* [AksCompute.ClusterPurpose](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose?view=azure-ml-py&preserve-view=true)
* [AksCompute.attach](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#attach-workspace--name--attach-configuration-)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

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

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para obtener información sobre cómo adjuntar un clúster de AKS en el portal, consulte [Creación de destinos de proceso en Azure Machine Learning Studio](how-to-create-attach-compute-studio.md#inference-clusters).

---

## <a name="next-steps"></a>Pasos siguientes

* [Cómo y dónde implementar un modelo](how-to-deploy-and-where.md)
* [Implementación de un modelo en un clúster de Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)