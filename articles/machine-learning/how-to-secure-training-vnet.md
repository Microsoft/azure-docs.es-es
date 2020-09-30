---
title: Protección de entornos de entrenamiento con redes virtuales
titleSuffix: Azure Machine Learning
description: Use una instancia aislada de Azure Virtual Network para proteger el entorno de entrenamiento de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.date: 07/16/2020
ms.custom: contperfq4, tracking-python
ms.openlocfilehash: cf2c19297fdd56fb376a54b231bbb021c53b25bd
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90882976"
---
# <a name="secure-an-azure-machine-learning-training-environment-with-virtual-networks"></a>Protección de un entorno de entrenamiento de Azure Machine Learning con redes virtuales

En este artículo, aprenderá a proteger los entornos de entrenamiento con una red virtual en Azure Machine Learning.

Este artículo es la tercera parte de una serie de cinco capítulos que le guía a través de la protección de un flujo de trabajo de Azure Machine Learning. Le recomendamos encarecidamente que lea [Parte uno: Introducción a las redes virtuales](how-to-network-security-overview.md) para comprender la arquitectura general en primer lugar. 

Consulte los demás artículos de esta serie:

[1. Introducción a las redes virtuales](how-to-network-security-overview.md) > [Protección del área de trabajo](how-to-secure-workspace-vnet.md) > **3. Protección del entorno de entrenamiento** > [4. Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md)  > [5. Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)

En este artículo aprenderá a proteger los siguientes recursos de proceso de entrenamiento en una red virtual:
> [!div class="checklist"]
> - Clúster de proceso de Azure Machine Learning
> - Instancia de proceso de Azure Machine Learning
> - Azure Databricks
> - Máquina virtual
> - Clúster de HDInsight

## <a name="prerequisites"></a>Requisitos previos

+ Lea el artículo [Introducción a la seguridad de red](how-to-network-security-overview.md) para comprender los escenarios comunes de redes virtuales y la arquitectura de red virtual general.

+ Una red virtual y una subred existentes que se usarán con los recursos de proceso.

+ Para implementar recursos en una red virtual o subred, la cuenta de usuario debe tener permisos para realizar las siguientes acciones en los controles de acceso basados en rol (RBAC) de Azure:

    - "Microsoft.Network/virtualNetworks/join/action" en el recurso de red virtual.
    - "Microsoft.Network/virtualNetworks/subnet/join/action" en el recurso de subred.

    Para obtener más información sobre RBAC con redes, vea los [roles integrados de redes](/azure/role-based-access-control/built-in-roles#networking).


## <a name="compute-clusters--instances"></a><a name="compute-instance"></a>Clústeres e instancias de proceso 

Para usar un [__destino de proceso__ de Azure Machine Learning](concept-compute-target.md#azure-machine-learning-compute-managed) o una [__instancia__ de proceso de Azure Machine Learning](concept-compute-instance.md) en una red virtual, deben cumplirse los siguientes requisitos de red:

> [!div class="checklist"]
> * La red virtual debe estar en la misma suscripción y región que el área de trabajo de Azure Machine Learning.
> * La subred que se especifica para la instancia de proceso o el clúster debe tener suficientes direcciones IP sin asignar para acomodar el número de máquinas virtuales de destino. Si la subred no tiene suficientes direcciones IP sin asignar, un clúster de proceso se asignará parcialmente.
> * Compruebe si sus directivas de seguridad o bloqueos del grupo de recursos o la suscripción de la red virtual restringen los permisos para administrar las redes virtuales. Si va a proteger la red virtual mediante la restricción del tráfico, deje abiertos algunos puertos para el servicio de proceso. Para más información, vea la sección [Puertos obligatorios](#mlcports).
> * Si va a colocar varios clústeres o instancias de proceso en una red virtual, es posible que tenga que solicitar un aumento de la cuota para uno o varios de los recursos.
> * Si las cuentas de Azure Storage del área de trabajo también están protegidas en una red virtual, deben estar en la misma red virtual que el clúster o la instancia de Proceso de Azure Machine Learning. 
> * Para que la funcionalidad de Jupyter de instancia de proceso haga su trabajo, asegúrese de que la comunicación de socket web no esté deshabilitada.

> [!TIP]
> El clúster o la instancia de proceso de Machine Learning asigna automáticamente recursos de red adicionales __al grupo de recursos que contiene la red virtual__. Para cada clúster o instancia de proceso, el servicio asigna los recursos siguientes:
> 
> * Un grupo de seguridad de red
> * Una dirección IP pública
> * Un equilibrador de carga
> 
> En el caso de los clústeres, estos recursos se eliminan (y se vuelven a crear) cada vez que el clúster se reduce verticalmente hasta quedarse sin ningún nodo. Sin embargo, para una instancia, los recursos se retienen hasta que esta se elimina completamente (los recursos no se eliminan con la detención). 
> Estos recursos están limitados por las [cuotas de recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits) de la suscripción.


### <a name="required-ports"></a><a id="mlcports"></a> Puertos necesarios

Si planea proteger la red virtual mediante la restricción del tráfico de red hacia y desde la red pública de Internet, debe permitir las comunicaciones de entrada desde el servicio Azure Batch.

El servicio Batch agrega grupos de seguridad de red (NSG) en el nivel de las interfaces de red (NIC) que están asociadas a las máquinas virtuales. Estos grupos de seguridad de red configuran automáticamente las reglas de entrada y salida para permitir el siguiente tráfico:

- Tráfico de entrada TCP en los puertos 29876 y 29877 desde una __etiqueta de servicio__ __BatchNodeManagement__.

    ![Una regla de entrada que usa la etiqueta de servicio BatchNodeManagement](./media/how-to-enable-virtual-network/batchnodemanagement-service-tag.png)

- (Opcional) Tráfico TCP de entrada en el puerto 22 para permitir el acceso remoto. Use este puerto solo si desea conectarse mediante SSH en la IP pública.

- Tráfico saliente en cualquier puerto para la red virtual.

- Tráfico saliente en cualquier puerto para Internet.

- Para el tráfico TCP de entrada de la instancia de proceso en el puerto 44224 desde una __etiqueta de servicio__ de __AzureMachineLearning__.

> [!IMPORTANT]
> Tenga cuidado si modifica reglas de entrada o salida en los grupos de seguridad de red configurados para Batch o las agrega a ellos. Si un grupo de seguridad de red bloquea la comunicación con los nodos de ejecución, el servicio de proceso establece el estado de los nodos de ejecución en inutilizable.
>
> No es necesario especificar grupos de seguridad de red en el nivel de subred, porque el servicio Azure Batch configura los suyos propios. Pero si la subred que contiene el proceso de Azure Machine Learning tiene grupos de seguridad de red asociados o un firewall, también debe permitir el tráfico que se ha mostrado antes.

La configuración de la regla de NSG en Azure Portal se muestra en las siguientes imágenes:

:::image type="content" source="./media/how-to-enable-virtual-network/amlcompute-virtual-network-inbound.png" alt-text="Reglas de grupo de seguridad de red de entrada para Proceso de Machine Learning" border="true":::



![Reglas de grupo de seguridad de red de entrada para Proceso de Machine Learning](./media/how-to-enable-virtual-network/experimentation-virtual-network-outbound.png)

### <a name="limit-outbound-connectivity-from-the-virtual-network"></a><a id="limiting-outbound-from-vnet"></a> Limitación de la conectividad saliente de la red virtual

Si no quiere usar las reglas de salida predeterminadas y quiere limitar el acceso de salida de la red virtual, siga los pasos siguientes:

- Deniegue la conexión saliente a Internet mediante las reglas de NSG.

- En el caso de una __instancia de proceso__ o un __clúster de proceso__, limite el tráfico saliente a los siguientes elementos:
   - Azure Storage, mediante la __etiqueta de servicio__ de __Storage.RegionName__, donde `{RegionName}` es el nombre de una región de Azure.
   - Azure Container Registry, mediante la __etiqueta de servicio__ de __AzureContainerRegistry.RegionName__ donde `{RegionName}` es el nombre de una región de Azure.
   - Azure Machine Learning, mediante la __etiqueta de servicio__ de __AzureMachineLearning__
   - Azure Resource Manager, mediante la __etiqueta de servicio__ de __AzureResourceManager__
   - Azure Active Directory, mediante la __etiqueta de servicio__ de __AzureActiveDirectory__

La configuración de la regla de NSG en Azure Portal se muestra en la siguiente imagen:

[![Reglas de grupo de seguridad de red de salida para Proceso de Machine Learning](./media/how-to-enable-virtual-network/limited-outbound-nsg-exp.png)](./media/how-to-enable-virtual-network/limited-outbound-nsg-exp.png#lightbox)

> [!NOTE]
> Si planea usar las imágenes de Docker predeterminadas proporcionadas por Microsoft y habilitar las dependencias administradas por el usuario, también debe usar las siguientes __etiquetas de servicio__:
>
> * __MicrosoftContainerRegistry__
> * __AzureFrontDoor.FirstParty__
>
> Esta configuración es necesaria cuando se tiene código similar a los fragmentos de código siguientes como parte de los scripts de entrenamiento:
>
> __Aprendizaje de RunConfig__
> ```python
> # create a new runconfig object
> run_config = RunConfiguration()
> 
> # configure Docker 
> run_config.environment.docker.enabled = True
> # For GPU, use DEFAULT_GPU_IMAGE
> run_config.environment.docker.base_image = DEFAULT_CPU_IMAGE 
> run_config.environment.python.user_managed_dependencies = True
> ```
>
> __Entrenamiento de Estimador__
> ```python
> est = Estimator(source_directory='.',
>                 script_params=script_params,
>                 compute_target='local',
>                 entry_script='dummy_train.py',
>                 user_managed=True)
> run = exp.submit(est)
> ```

### <a name="forced-tunneling"></a>Tunelización forzada

Si usa la [tunelización forzada](/azure/vpn-gateway/vpn-gateway-forced-tunneling-rm) con el proceso de Azure Machine Learning, debe permitir la comunicación con la red pública de Internet desde la subred que contiene el recurso de proceso. Esta comunicación se usa para la programación de tareas y el acceso a Azure Storage.

Hay dos maneras en que puede lograrlo:

* Use una instancia de [Virtual Network NAT](../virtual-network/nat-overview.md). Una puerta de enlace NAT proporciona conectividad de salida a Internet para una o más subredes en su red virtual. Para obtener más información, consulte [Diseño de redes virtuales con recursos de puertas de enlace NAT](../virtual-network/nat-gateway-resource.md).

* Agregue [rutas definidas por el usuario (UDR)](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview) a la subred que contiene el recurso de proceso. Establezca una ruta definida por el usuario para cada dirección IP que use el servicio Azure Batch en la región en la que existen sus recursos. Estas UDR permiten que el servicio Batch se comunique con los nodos de proceso para programar tareas. Agregue también la dirección IP de Azure Machine Learning Service en el que existen los recursos, ya que esto es necesario para acceder a las instancias de proceso. Para obtener una lista de direcciones IP del servicio Batch y de Azure Machine Learning Service, utilice uno de los métodos siguientes:

    * Descargue los [intervalos de direcciones IP y las etiquetas de servicio de Azure](https://www.microsoft.com/download/details.aspx?id=56519) y busque `BatchNodeManagement.<region>` y `AzureMachineLearning.<region>` en el archivo, donde `<region>` es su región de Azure.

    * Use la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) para descargar la información. En el ejemplo siguiente se descarga la información de la dirección IP, que se filtra para la región Este de EE. UU. 2:

        ```azurecli-interactive
        az network list-service-tags -l "East US 2" --query "values[?starts_with(id, 'Batch')] | [?properties.region=='eastus2']"
        az network list-service-tags -l "East US 2" --query "values[?starts_with(id, 'AzureMachineLearning')] | [?properties.region=='eastus2']"
        ```

        > [!TIP]
        > Si usa las regiones US-Virginia o US-Arizona, o las regiones China-East-2, estos comandos no devuelven direcciones IP. Use mejor uno de los siguientes vínculos para descargar una lista de direcciones IP:
        >
        > * [Intervalos de direcciones IP y etiquetas de servicio de Azure para Azure Government](https://www.microsoft.com/download/details.aspx?id=57063)
        > * [Intervalos de direcciones IP y etiquetas de servicio de Azure para Azure China](https://www.microsoft.com//download/details.aspx?id=57062)
    
    Cuando agregue las rutas definidas por el usuario, defina la ruta del prefijo de cada dirección IP de Batch relacionada y en __Tipo del próximo salto__, seleccione __Internet__. En la imagen siguiente se muestra un ejemplo de esta UDR en Azure Portal:

    ![Ejemplo de una ruta definida por el usuario para un prefijo de dirección](./media/how-to-enable-virtual-network/user-defined-route.png)

    > [!IMPORTANT]
    > Las direcciones IP pueden cambiar con el tiempo.

    Además de cualquier UDR que defina, se debe permitir el tráfico de salida a Azure Storage a través del dispositivo de red local. En concreto, las direcciones URL de este tráfico están en los siguientes formatos: `<account>.table.core.windows.net`, `<account>.queue.core.windows.net` y `<account>.blob.core.windows.net`. 

    Para más información, consulte [Creación de un grupo de Azure Batch en una red virtual](../batch/batch-virtual-network.md#user-defined-routes-for-forced-tunneling).


### <a name="create-a-compute-cluster-in-a-virtual-network"></a>Creación de un clúster de proceso en una red virtual

Para crear un clúster de Proceso de Machine Learning, siga los pasos siguientes:

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/) y, después, seleccione la suscripción y el área de trabajo.

1. Seleccione __Proceso__ a la izquierda.

1. Seleccione __Clústeres de entrenamiento__ en el centro y, después, seleccione __+__ .

1. En el cuadro de diálogo __Nuevo clúster de entrenamiento__, expanda la sección __Configuración avanzada__.

1. Para configurar este recurso de proceso para que use una red virtual, realice las acciones siguientes en la sección __Configurar la red virtual__:

    1. En la lista desplegable __Grupo de recursos__, seleccione el grupo de recursos que contiene la red virtual.
    1. En la lista desplegable __Red virtual__, seleccione la red que contiene la subred.
    1. En la lista desplegable __Subred__, seleccione la subred que se va a usar.

   ![Configuración de la red virtual de Proceso de Machine Learning](./media/how-to-enable-virtual-network/amlcompute-virtual-network-screen.png)

También puede crear un clúster de Proceso de Machine Learning con el SDK de Azure Machine Learning. El código siguiente crea un nuevo clúster de Proceso de Machine Learning en la `default` subred de una red virtual denominada `mynetwork`:

```python
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException

# The Azure virtual network name, subnet, and resource group
vnet_name = 'mynetwork'
subnet_name = 'default'
vnet_resourcegroup_name = 'mygroup'

# Choose a name for your CPU cluster
cpu_cluster_name = "cpucluster"

# Verify that cluster does not exist already
try:
    cpu_cluster = ComputeTarget(workspace=ws, name=cpu_cluster_name)
    print("Found existing cpucluster")
except ComputeTargetException:
    print("Creating new cpucluster")

    # Specify the configuration for the new cluster
    compute_config = AmlCompute.provisioning_configuration(vm_size="STANDARD_D2_V2",
                                                           min_nodes=0,
                                                           max_nodes=4,
                                                           vnet_resourcegroup_name=vnet_resourcegroup_name,
                                                           vnet_name=vnet_name,
                                                           subnet_name=subnet_name)

    # Create the cluster with the specified name and configuration
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)

    # Wait for the cluster to be completed, show the output log
    cpu_cluster.wait_for_completion(show_output=True)
```

Cuando finaliza el proceso de creación, el modelo se entrena mediante el clúster en un experimento. Para más información, consulte [Selección y uso de un destino de proceso para entrenamiento](how-to-set-up-training-targets.md).

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

### <a name="access-data-in-a-compute-instance-notebook"></a>Acceso a los datos en un cuaderno de instancia de Compute

Si usa cuadernos en una instancia de Azure Compute, debe asegurarse de que el cuaderno se ejecuta en un recurso de proceso detrás de la misma red virtual y subred que los datos. 

Tendrá que configurar la instancia de Compute para que esté en la misma red virtual durante la creación en **Configuración avanzada** > **Configurar la red virtual**. No se puede agregar una instancia de Compute existente a una red virtual.

## <a name="azure-databricks"></a>Azure Databricks

Para usar Azure Databricks en una red virtual con el área de trabajo, deben cumplirse los siguientes requisitos:

> [!div class="checklist"]
> * La red virtual debe estar en la misma suscripción y región que el área de trabajo de Azure Machine Learning.
> * Si las cuentas de Azure Storage del área de trabajo también están protegidas en una red virtual, deben estar en la misma red virtual que el clúster de proceso de Azure Databricks.
> * Además de las subredes __databricks-private__ y __databricks-public__ utilizadas por Azure Databricks, también se requiere la subred __predeterminada__ creada para la red virtual.

Para obtener información específica sobre el uso de Azure Databricks con una red virtual, consulte [Implementar Azure Databricks en su red virtual de Azure](https://docs.azuredatabricks.net/administration-guide/cloud-configurations/azure/vnet-inject.html).

<a id="vmorhdi"></a>

## <a name="virtual-machine-or-hdinsight-cluster"></a>Máquina virtual o clúster de HDInsight

> [!IMPORTANT]
> Azure Machine Learning solo admite máquinas virtuales que ejecuten Ubuntu.

En esta sección aprenderá a usar una máquina virtual o un clúster de Azure HDInsight en una red virtual con su área de trabajo.

### <a name="create-the-vm-or-hdinsight-cluster"></a>Creación de la máquina virtual o el clúster de HDInsight

Cree una máquina virtual o un clúster de HDInsight mediante Azure Portal o la CLI de Azure y colóquelo en una red virtual de Azure. Para más información, consulte los siguientes artículos.
* [Creación y administración de redes virtuales de Azure para máquinas virtuales Linux](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-virtual-network)

* [Extender HDInsight mediante una red virtual de Azure](https://docs.microsoft.com/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)

### <a name="configure-network-ports"></a>Configuración de los puertos de red 

Permita que Azure Machine Learning se comunique con el puerto SSH en la máquina virtual o el clúster y configure una entrada de origen para el grupo de seguridad de red. El puerto SSH suele ser el 22. Para permitir el tráfico desde este origen, realice las acciones siguientes:

1. En la lista desplegable __Origen__, seleccione __Etiqueta de servicio__.

1. En la lista desplegable __Etiqueta de servicio de origen__ , seleccione __AzureMachineLearning__.

    ![Reglas de entrada para realizar la experimentación en una máquina virtual o un clúster de HDInsight dentro de una red virtual](./media/how-to-enable-virtual-network/experimentation-virtual-network-inbound.png)

1. En la lista desplegable __Intervalos de puertos de origen__, seleccione __*__ .

1. En la lista desplegable __Destino__, seleccione __Cualquiera__.

1. En la lista desplegable __Intervalos de puertos de destino__, seleccione __22__.

1. En __Protocolo__, seleccione __Cualquiera__.

1. En __Acción__, seleccione __Permitir__.

Conserve las reglas de salida predeterminadas para el grupo de seguridad de red. Para más información, consulte las reglas de seguridad predeterminadas en [Grupos de seguridad](https://docs.microsoft.com/azure/virtual-network/security-overview#default-security-rules).

Si no desea usar las reglas de salida predeterminadas, pero desea limitar el acceso de salida de la red virtual, consulte la sección [Limitación de la conectividad saliente de la red virtual](#limiting-outbound-from-vnet).

### <a name="attach-the-vm-or-hdinsight-cluster"></a>Conexión de la máquina virtual o el clúster de HDInsight

Conecte el clúster de HDInsight o de máquina virtual a su área de trabajo de Azure Machine Learning. Para más información, consulte [Configuración de destinos de proceso del entrenamiento del modelo](how-to-set-up-training-targets.md).

## <a name="next-steps"></a>Pasos siguientes

Este artículo es la tercera parte de una serie de cuatro capítulos sobre redes virtuales. Vea el resto de los artículos para obtener información sobre cómo proteger una red virtual:

* [Parte 1: Introducción a las redes virtuales](how-to-network-security-overview.md)
* [Parte 2: Protección de los recursos de un área de trabajo](how-to-secure-workspace-vnet.md)
* [Parte 4: Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md)
* [Parte 5: Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)