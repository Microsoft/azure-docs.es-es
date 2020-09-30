---
title: Protección de entornos de inferencia con redes virtuales
titleSuffix: Azure Machine Learning
description: Use una instancia aislada de Azure Virtual Network para proteger el entorno de inferencia de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.date: 07/16/2020
ms.custom: contperfq4, tracking-python
ms.openlocfilehash: 359c2a27099ca298076edc255b8c30e226af0a18
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90882940"
---
# <a name="secure-an-azure-machine-learning-inferencing-environment-with-virtual-networks"></a>Protección de un entorno de inferencia de Azure Machine Learning con redes virtuales

En este artículo, aprenderá a proteger los entornos de inferencia con una red virtual en Azure Machine Learning.

Este artículo es la cuarta parte de una serie de cinco capítulos que le guía a través de la protección de un flujo de trabajo de Azure Machine Learning. Le recomendamos encarecidamente que lea [Parte uno: Introducción a las redes virtuales](how-to-network-security-overview.md) para comprender la arquitectura general en primer lugar. 

Consulte los demás artículos de esta serie:

[1. Introducción a las redes virtuales](how-to-network-security-overview.md) > [Protección del área de trabajo](how-to-secure-workspace-vnet.md) > [3. Protección del entorno de entrenamiento](how-to-secure-training-vnet.md) > **4. Protección del entorno de inferencia** > [5. Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)

En este artículo aprenderá a proteger los siguientes recursos de inferencia en una red virtual:
> [!div class="checklist"]
> - Clúster de Azure Kubernetes Service (AKS) predeterminado
> - Clúster privado de AKS
> - Azure Container Instances (ACI)


## <a name="prerequisites"></a>Requisitos previos

+ Lea el artículo [Introducción a la seguridad de red](how-to-network-security-overview.md) para comprender los escenarios comunes de redes virtuales y la arquitectura de red virtual general.

+ Una red virtual y una subred existentes que se usarán con los recursos de proceso.

+ Para implementar recursos en una red virtual o subred, la cuenta de usuario debe tener permisos para realizar las siguientes acciones en los controles de acceso basados en rol (RBAC) de Azure:

    - "Microsoft.Network/virtualNetworks/join/action" en el recurso de red virtual.
    - "Microsoft.Network/virtualNetworks/subnet/join/action" en el recurso de subred.

    Para obtener más información sobre RBAC con redes, vea los [roles integrados de redes](/azure/role-based-access-control/built-in-roles#networking).

<a id="aksvnet"></a>

## <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Para usar un clúster de AKS en una red virtual, deben cumplirse los siguientes requisitos de red:

> [!div class="checklist"]
> * Siga los requisitos previos que se indican en [Configuración de redes avanzadas en Azure Kubernetes Service (AKS)](../aks/configure-azure-cni.md#prerequisites).
> * La instancia de AKS y la red virtual deben estar en la misma región. Si protege las cuentas de Azure Storage usadas por el área de trabajo de una red virtual, deben estar en la misma red virtual que la instancia de AKS.


Para agregar AKS en una red virtual a su área de trabajo, siga los pasos siguientes:

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/) y, después, seleccione la suscripción y el área de trabajo.

1. Seleccione __Proceso__ a la izquierda.

1. Seleccione __Clústeres de inferencia__ en el centro y, después, seleccione __+__ .

1. En el cuadro de diálogo __Nuevo clúster de inferencia__, seleccione __Avanzado__ en __Configuración de red__.

1. Para configurar este recurso de proceso para que use una red virtual, realice las acciones siguientes:

    1. En la lista desplegable __Grupo de recursos__, seleccione el grupo de recursos que contiene la red virtual.
    1. En la lista desplegable __Red virtual__, seleccione la red que contiene la subred.
    1. En la lista desplegable __Subred__, seleccione la subred.
    1. En el cuadro __Intervalo de direcciones del servicio Kubernetes__, escriba el intervalo de direcciones de servicio de Kubernetes. Este intervalo de direcciones utiliza un intervalo de IP de notación CIDR (enrutamiento entre dominios sin clases) para definir las direcciones IP que están disponibles para el clúster. No debe superponerse con ningún intervalo IP de subred (por ejemplo, 10.0.0.0/16).
    1. En el cuadro __Dirección IP del servicio DNS de Kubernetes__, escriba la dirección IP del servicio DNS de Kubernetes. Esta dirección IP se asigna al servicio DNS de Kubernetes. Debe estar dentro del intervalo de direcciones del servicio Kubernetes (por ejemplo, 10.0.0.10).
    1. En el cuadro __Dirección de puente de Docker__, escriba la dirección del puente de Docker. Esta dirección IP se asigna al puente de Docker. No debe estar en ningún intervalo IP de subred o en el intervalo de direcciones del servicio Kubernetes (por ejemplo, 172.17.0.1/16).

   ![Azure Machine Learning: Configuración de la red virtual del Proceso de Machine Learning](./media/how-to-enable-virtual-network/aks-virtual-network-screen.png)

1. Asegúrese de que el grupo de seguridad de red que controla la red virtual tiene una regla de seguridad de entrada habilitada para el punto de conexión de puntuación, de modo que se le pueda llamar desde fuera de la red virtual.
   > [!IMPORTANT]
   > Mantenga las reglas de salida predeterminadas para el grupo de seguridad de red. Para más información, consulte las reglas de seguridad predeterminadas en [Grupos de seguridad](https://docs.microsoft.com/azure/virtual-network/security-overview#default-security-rules).

   [![Regla de seguridad de entrada](./media/how-to-enable-virtual-network/aks-vnet-inbound-nsg-scoring.png)](./media/how-to-enable-virtual-network/aks-vnet-inbound-nsg-scoring.png#lightbox)

También puede usar el SDK de Azure Machine Learning para agregar Azure Kubernetes Service en una red virtual. Si ya tiene un clúster de AKS en una red virtual, asócielo al área de trabajo como se describe en el tema sobre la [implementación en AKS](how-to-deploy-and-where.md). El código siguiente crea una instancia de Azure Kubernetes Service en la subred `default` de una red virtual denominada `mynetwork`:

```python
from azureml.core.compute import ComputeTarget, AksCompute

# Create the compute configuration and set virtual network information
config = AksCompute.provisioning_configuration(location="eastus2")
config.vnet_resourcegroup_name = "mygroup"
config.vnet_name = "mynetwork"
config.subnet_name = "default"
config.service_cidr = "10.0.0.0/16"
config.dns_service_ip = "10.0.0.10"
config.docker_bridge_cidr = "172.17.0.1/16"

# Create the compute target
aks_target = ComputeTarget.create(workspace=ws,
                                  name="myaks",
                                  provisioning_configuration=config)
```

Cuando finalice el proceso de creación, puede ejecutar una inferencia, o puntuación de modelos, en un clúster de AKS detrás de una red virtual. Para más información, consulte [Implementación en AKS](how-to-deploy-and-where.md).

## <a name="private-aks-cluster"></a>Clúster privado de AKS

De manera predeterminada, los clústeres de AKS tienen un plano de control o un servidor de API con las direcciones IP públicas. Puede configurar AKS para usar un plano de control privado mediante la creación de un clúster privado de AKS. Para obtener más información, consulte [Creación de un clúster privado de Azure Kubernetes Service](../aks/private-clusters.md).

Después de crear el clúster privado de AKS, [adjunte el clúster a la red virtual](how-to-create-attach-kubernetes.md) para usarlo con Azure Machine Learning.

## <a name="internal-aks-load-balancer"></a>Equilibrador de carga interno de AKS

De manera predeterminada, las implementaciones de AKS usan un [equilibrador de carga público](../aks/load-balancer-standard.md). En esta sección, aprenderá a configurar AKS para usar un equilibrador de carga interno. Un equilibrador de carga interno (o privado) se usa cuando solo se admiten direcciones IP privadas como front-end. Los equilibradores de carga internos se usan para equilibrar la carga del tráfico dentro de una red virtual.

Un equilibrador de carga privado se habilita mediante la configuración de AKS para usar un _equilibrador de carga interno_. 

### <a name="network-contributor-role"></a>Rol de colaborador de red

> [!IMPORTANT]
> Si crea o asocia un clúster de AKS al proporcionar una red virtual creada anteriormente, debe conceder a la entidad de servicio (SP) o la identidad administrada del clúster de AKS el rol _Colaborador de la red_ del grupo de recursos que contiene la red virtual. Debe hacerlo antes de intentar cambiar el equilibrador de carga interno a una dirección IP privada.
>
> Para agregar la identidad como colaborador de red, siga estos pasos:

1. Para buscar la entidad de servicio o el id. de identidad administrada para AKS, use los siguientes comandos de la CLI de Azure. Reemplace `<aks-cluster-name>` por el nombre del clúster. Reemplace `<resource-group-name>` por el nombre del grupo de recursos que _contiene el clúster de AKS_:

    ```azurecli-interactive
    az aks show -n <aks-cluster-name> --resource-group <resource-group-name> --query servicePrincipalProfile.clientId
    ``` 

    Si este comando devuelve el valor `msi`, use el siguiente comando para identificar el identificador de entidad de seguridad de la identidad administrada:

    ```azurecli-interactive
    az aks show -n <aks-cluster-name> --resource-group <resource-group-name> --query identity.principalId
    ```

1. Para buscar el identificador del grupo de recursos que contiene la red virtual, use el siguiente comando. Reemplace `<resource-group-name>` por el nombre del grupo de recursos que _contiene la red virtual_.

    ```azurecli-interactive
    az group show -n <resource-group-name> --query id
    ```

1. Para agregar la entidad de servicio o la identidad administrada como colaborador de la red, use el siguiente comando. Reemplace `<SP-or-managed-identity>` por el identificador devuelto para la entidad de servicio o la identidad administrada. Reemplace `<resource-group-id>` por el identificador revuelto para el grupo de recursos que contiene la red virtual:

    ```azurecli-interactive
    az role assignment create --assignee <SP-or-managed-identity> --role 'Network Contributor' --scope <resource-group-id>
    ```
Para más información sobre el uso del equilibrador de carga interno con AKS, consulte [Uso del equilibrador de carga interno con Azure Kubernetes Service](/azure/aks/internal-lb).

### <a name="enable-private-load-balancer"></a>Habilitación del equilibrador de carga privado

> [!IMPORTANT]
> No se puede habilitar la IP privada al crear el clúster de Azure Kubernetes Service. Debe habilitarse como una actualización de un clúster existente.

En el siguiente fragmento de código se muestra cómo __crear un nuevo clúster de AKS__ y, después, actualizarlo para que use una IP privada y un equilibrador de carga interno:

```python
import azureml.core
from azureml.core.compute.aks import AksUpdateConfiguration
from azureml.core.compute import AksCompute, ComputeTarget

# Verify that cluster does not exist already
try:
    aks_target = AksCompute(workspace=ws, name=aks_cluster_name)
    print("Found existing aks cluster")

except:
    print("Creating new aks cluster")

    # Subnet to use for AKS
    subnet_name = "default"
    # Create AKS configuration
    prov_config = AksCompute.provisioning_configuration(location = "eastus2")
    # Set info for existing virtual network to create the cluster in
    prov_config.vnet_resourcegroup_name = "myvnetresourcegroup"
    prov_config.vnet_name = "myvnetname"
    prov_config.service_cidr = "10.0.0.0/16"
    prov_config.dns_service_ip = "10.0.0.10"
    prov_config.subnet_name = subnet_name
    prov_config.docker_bridge_cidr = "172.17.0.1/16"

    # Create compute target
    aks_target = ComputeTarget.create(workspace = ws, name = "myaks", provisioning_configuration = prov_config)
    # Wait for the operation to complete
    aks_target.wait_for_completion(show_output = True)
    
    # Update AKS configuration to use an internal load balancer
    update_config = AksUpdateConfiguration(None, "InternalLoadBalancer", subnet_name)
    aks_target.update(update_config)
    # Wait for the operation to complete
    aks_target.wait_for_completion(show_output = True)
```

__CLI de Azure__

```azurecli-interactive
az rest --method put --uri https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.MachineLearningServices/workspaces/<workspace>/computes/<compute>?api-version=2018-11-19 --body @body.json
```

El contenido del archivo `body.json` al que hace referencia el comando es similar al siguiente documento JSON:

```json
{ 
    "location": "<region>", 
    "properties": { 
        "resourceId": "/subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.ContainerService/managedClusters/<aks-resource-name>", 
        "computeType": "AKS", 
        "provisioningState": "Succeeded", 
        "properties": { 
            "loadBalancerType": "InternalLoadBalancer", 
            "agentCount": <agent-count>, 
            "agentVmSize": "vm-size", 
            "clusterFqdn": "<cluster-fqdn>" 
        } 
    } 
} 
```

Cuando __asocie un clúster existente__ al área de trabajo, debe esperar a que finalice la operación de adjuntar para configurar el equilibrador de carga.

Para obtener información sobre cómo asociar un clúster, consulte [Asociación de un clúster de AKS ya existente](how-to-create-attach-kubernetes.md).

Después de asociar el clúster existente, puede actualizarlo para usar una dirección IP privada.

```python
import azureml.core
from azureml.core.compute.aks import AksUpdateConfiguration
from azureml.core.compute import AksCompute

# ws = workspace object. Creation not shown in this snippet
aks_target = AksCompute(ws,"myaks")

# Change to the name of the subnet that contains AKS
subnet_name = "default"
# Update AKS configuration to use an internal load balancer
update_config = AksUpdateConfiguration(None, "InternalLoadBalancer", subnet_name)
aks_target.update(update_config)
# Wait for the operation to complete
aks_target.wait_for_completion(show_output = True)
```

## <a name="enable-azure-container-instances-aci"></a>Habilitación de Azure Container Instances (ACI)

Azure Container Instances se crean dinámicamente al implementar un modelo. Para habilitar Azure Machine Learning para crear ACI dentro de la red virtual, debe habilitar la __delegación de subred__ para la subred que usa la implementación.

> [!WARNING]
> Al usar Azure Container Instances en una red virtual, la red virtual debe estar en el mismo grupo de recursos que el área de trabajo de Azure Machine Learning.
>
> Al usar Azure Container Instances dentro de la red virtual, Azure Container Registry (ACR) del área de trabajo no puede estar también en la red virtual.

A fin de usar ACI en una red virtual para su área de trabajo, siga los pasos siguientes:

1. Para habilitar la delegación de subred en la red virtual, use la información del artículo [Adición o eliminación de una delegación de subred](../virtual-network/manage-subnet-delegation.md). Puede habilitar la delegación al crear una red virtual o agregarla a una red existente.

    > [!IMPORTANT]
    > Al habilitar la delegación, use `Microsoft.ContainerInstance/containerGroups` como el valor __Delegar subred en un servicio__.

2. Implemente el modelo mediante [AciWebservice.deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aci.aciwebservice?view=azure-ml-py#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none--vnet-name-none--subnet-name-none-&preserve-view=true), use los parámetros `vnet_name` y `subnet_name`. Establezca estos parámetros en el nombre de red virtual y subred donde habilitó la delegación.


## <a name="next-steps"></a>Pasos siguientes

Este artículo es la tercera parte de una serie de cuatro capítulos sobre redes virtuales. Vea el resto de los artículos para obtener información sobre cómo proteger una red virtual:

* [Parte 1: Introducción a las redes virtuales](how-to-network-security-overview.md)
* [Parte 2: Protección de los recursos de un área de trabajo](how-to-secure-workspace-vnet.md)
* [Parte 3: Protección del entorno de entrenamiento](how-to-secure-training-vnet.md)
* [Parte 5: Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)