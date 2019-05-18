---
title: Ejecución de experimentos y realización de inferencias en una red virtual
titleSuffix: Azure Machine Learning service
description: Ejecute experimentos de aprendizaje automático y realice inferencias de forma segura dentro de una red virtual de Azure. Aprenda a crear destinos de proceso para el entrenamiento de modelos y a realizar inferencias dentro de una red virtual. Conozca los requisitos de las redes virtuales protegidas, como los puertos de entrada y salida necesarios.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: aashishb
author: aashishb
ms.date: 01/08/2019
ms.openlocfilehash: fe51f4589075cb275e867c943c5d7df3e8d5d4a0
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65795042"
---
# <a name="securely-run-experiments-and-inferencing-inside-an-azure-virtual-network"></a>Ejecución de experimentos y realización de inferencias de forma segura dentro de una red virtual de Azure

En este artículo, aprenderá a ejecutar experimentos y a realizar inferencias dentro de una red virtual. Una red virtual actúa como un límite de seguridad, aislando los recursos de Azure desde internet pública. También pude unir una red virtual de Azure a la red local. Permite acceder a los modelos implementados para las inferencias y entrenar los modelos de forma segura.

Azure Machine Learning Service depende de otros servicios de Azure para los recursos de proceso. Los recursos de proceso (destinos de proceso) se usan para entrenar e implementar modelos. Estos destinos de proceso pueden crearse dentro de una red virtual. Por ejemplo, puede usar la instancia de Microsoft Data Science Virtual Machine para entrenar un modelo y, luego, implementarlo en Azure Kubernetes Service (AKS). Para más información sobre las redes virtuales, consulte [Información general sobre Azure Virtual Network](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview).

## <a name="prerequisites"></a>Requisitos previos

Este documento se da por supuesto que está familiarizado con las redes virtuales de Azure y la IP de red en general. Este documento también se supone que ha creado una red virtual y subred para usar con los recursos de proceso. Si no está familiarizado con las redes virtuales de Azure, lea los artículos siguientes para obtener información sobre el servicio:

* [Direccionamiento IP](https://docs.microsoft.com/azure/virtual-network/virtual-network-ip-addresses-overview-arm)
* [Grupos de seguridad](https://docs.microsoft.com/azure/virtual-network/security-overview)
* [Inicio rápido: Crear una red virtual](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)
* [Filtrado del tráfico de red](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

## <a name="storage-account-for-your-workspace"></a>Cuenta de almacenamiento para el área de trabajo

> [!IMPORTANT]
> Puede colocar la cuenta de almacenamiento que está conectada al área de trabajo de servicio de Azure Machine Learning detrás de la red virtual solo mientras se realiza la experimentación. Inferencia requiere acceso sin restricciones a la cuenta de almacenamiento. Si no está seguro de si ha modificado estos valores, consulte __Modificación de la regla de acceso de red predeterminada__ en [Configuración de redes virtuales y firewalls de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-network-security). Use los pasos para permitir el acceso desde todas las redes mientras se realiza la inferencia.

Para usar la experimentación de Azure Machine Learning capacidades de una red virtual con el almacenamiento de Azure siguen estos pasos:

1. Crear un proceso de experimentación ex. Machine Learning Compute detrás de una red virtual o adjuntar un proceso de experimentación en el área de trabajo ex. Máquina virtual o clúster de HDInsight. Para obtener más información, consulte [uso Machine Learning Compute](#use-machine-learning-compute) y [usar una máquina virtual o clúster de HDInsight](#use-a-virtual-machine-or-hdinsight-cluster) secciones de este documento
2. Vaya al almacenamiento conectado al área de trabajo. ![Imagen de Azure portal que muestra el almacenamiento de Azure que está conectado al área de trabajo del servicio de Azure Machine Learning](./media/how-to-enable-virtual-network/workspace-storage.png)
3. En la página de Azure Storage, seleccione __Firewalls y redes virtuales__. ![Imagen de Azure portal que muestra los Firewalls y virtual networks sección en la página de Azure Storage](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks.png)
4. En el __Firewalls y redes virtuales__ página Seleccione lo siguiente:
    - Seleccione __Redes seleccionadas__.
    - En __redes virtuales__ seleccione __agregar red virtual existente__ para agregar la red virtual donde reside el proceso de experimentación. (Vea el paso 1).
    - Seleccione __permitir que los servicios de Microsoft para obtener acceso a esta cuenta de almacenamiento de confianza__.
![Imagen de Azure portal que muestra los Firewalls y virtual networks página en el almacenamiento de Azure](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks-page.png) 

5. Al ejecutar el experimento, en el código de la experimentación, cambie la configuración de ejecución para usar el almacenamiento de blobs:
    ```python
    run_config.source_directory_data_store = "workspaceblobstore"
    ```
    
## <a name="key-vault-for-your-workspace"></a>Almacén de claves para el área de trabajo
Instancia de Key Vault asociada con el área de trabajo se usa por el servicio de Azure Machine Learning para almacenar las credenciales de varias clases:
* La cadena de conexión de la cuenta de almacenamiento asociada
* Contraseñas para las instancias de Azure Container Repository
* Cadenas de conexión a datos almacenes. 

Para usar la experimentación de Azure Machine Learning capacidades con Key Vault detrás de una red virtual siguen los pasos siguientes:
1. Vaya al almacén de claves asociado con el área de trabajo. ![Imagen de Azure portal que muestra el almacén de claves que está asociado con el área de trabajo del servicio de Azure Machine Learning](./media/how-to-enable-virtual-network/workspace-key-vault.png)
2. En el almacén de claves página Seleccione __Firewalls y redes virtuales__ sección. ![Imagen de Azure portal que muestra los Firewalls y virtual networks sección en la página de Key Vault](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks.png)
3. En el __Firewalls y redes virtuales__ página Seleccione lo siguiente:
    - Seleccione __Redes seleccionadas__.
    - En el __redes virtuales__ seleccione __agregar redes virtuales existentes__ para agregar la red virtual donde reside el proceso de experimentación.
    - Seleccione __permitir que los servicios de Microsoft para omitir este firewall confianza__.
![Imagen de Azure portal que muestra los Firewalls y virtual networks página en Key Vault](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks-page.png) 


## <a name="use-machine-learning-compute"></a>Uso del Proceso de Machine Learning

Para usar el Proceso de Azure Machine Learning en una red virtual, utilice la siguiente información sobre los requisitos de red:

- La red virtual debe estar en la misma suscripción y región que el área de trabajo de Azure Machine Learning Service.

- La subred especificada para el clúster del Proceso de Machine Learning debe tener suficientes direcciones IP sin asignar para acoger el número de máquinas virtuales destinadas al clúster. Si la subred no tiene suficientes direcciones IP sin asignar, el clúster se asignará parcialmente.

- Si va a proteger la red virtual mediante la restricción del tráfico, deje abiertos algunos puertos para el servicio Proceso de Machine Learning. Para más información, consulte [Puertos necesarios](#mlcports).

- Compruebe si sus directivas de seguridad o bloqueos del grupo de recursos o la suscripción de la red virtual restringen los permisos para administrar las redes virtuales.

- Si va a colocar varios clústeres de Proceso de Machine Learning en una red virtual, puede que tenga que solicitar un aumento de cuota para uno o varios de los recursos.

    El Proceso de Machine Learning asigna automáticamente recursos de red adicionales del grupo de recursos que contiene la red virtual. Para cada clúster de Proceso de Machine Learning, Azure Machine Learning Service asigna los recursos siguientes:

    - Un grupo de seguridad de red (NSG)

    - Una dirección IP pública

    - Un equilibrador de carga

  Estos recursos están limitados por las [cuotas de recursos](https://docs.microsoft.com/azure/azure-subscription-service-limits) de la suscripción.

### <a id="mlcports"></a> Puertos necesarios

El Proceso de Machine Learning usa actualmente el servicio Azure Batch para aprovisionar máquinas virtuales en la red virtual especificada. La subred debe permitir las comunicaciones entrantes desde el servicio Batch. Esta comunicación se usa para programar ejecuciones en los nodos de Proceso de Machine Learning y para comunicarse con Azure Storage y otros recursos. Batch agrega grupos de seguridad de red en el nivel de interfaces de red (NIC) que están asociadas a las máquinas virtuales. Estos grupos de seguridad de red configuran automáticamente las reglas de entrada y salida para permitir el siguiente tráfico:

- Tráfico de entrada TCP en los puertos 29876 y 29877 desde un __etiqueta de servicio__ de __BatchNodeManagement__.

    ![Imagen de Azure portal que muestra una regla de entrada con la etiqueta de servicio BatchNodeManagement](./media/how-to-enable-virtual-network/batchnodemanagement-service-tag.png)
 
- (opcional) Tráfico TCP entrante en el puerto 22 para permitir el acceso remoto. Esto solo es necesario si desea conectarse mediante SSH en la dirección IP pública.
 
- Tráfico saliente en cualquier puerto para la red virtual.

- Tráfico saliente en cualquier puerto para Internet. 

Tenga cuidado si modifica reglas de entrada o de salida en los grupos de seguridad de red configurados para Batch o las agrega a ellos. Si un grupo de seguridad de red bloquea la comunicación a los nodos de proceso, los servicios de Proceso de Machine Learning establecen el estado de los nodos de proceso como inutilizables.

No es necesario especificar los grupos de seguridad de red a nivel de subred, porque Batch configura los suyos propios. Sin embargo, si la subred especificada tiene asociados grupos de seguridad de red o un firewall, configure las reglas de seguridad de entrada y salida como se ha explicado antes. 

Captura de pantalla siguiente muestra cómo se ve la configuración de reglas NSG en Azure portal:

![Captura de pantalla de las reglas de NSG de entrada para el Proceso de Machine Learning](./media/how-to-enable-virtual-network/amlcompute-virtual-network-inbound.png)

![Captura de pantalla de las reglas de NSG de salida para el Proceso de Machine Learning](./media/how-to-enable-virtual-network/experimentation-virtual-network-outbound.png)

### <a id="limiting-outbound-from-vnet"></a> Limitar la conectividad saliente de la red virtual

Si no desea usar las reglas de salida predeterminado y desea limitar el acceso de salida de la red virtual, siga estos pasos:

- Denegar la conexión a internet saliente mediante las reglas de NSG 

- Limitar el tráfico saliente al almacenamiento de Azure (mediante __etiqueta de servicio__ de __Storage.Region_Name__ ex. Storage.EastUS), Azure Container Registry (mediante __etiqueta de servicio__ de __AzureContainerRegistry.Region_Name__ ex. AzureContainerRegistry.EastUS) y el servicio de Azure Machine Learning (con __etiqueta de servicio__ de __AzureMachineLearning__)

Captura de pantalla siguiente muestra cómo se ve la configuración de reglas NSG en Azure portal:

![Captura de pantalla de las reglas de NSG de salida para el Proceso de Machine Learning](./media/how-to-enable-virtual-network/limited-outbound-nsg-exp.png)




### <a name="create-machine-learning-compute-in-a-virtual-network"></a>Creación del Proceso de Machine Learning en una red virtual

Para crear un clúster de Proceso de Machine Learning mediante Azure Portal, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), seleccione el área de trabajo de Azure Machine Learning Service.

1. En la sección __Aplicación__, seleccione __Proceso__. A continuación, seleccione __Agregar proceso__. 

    ![Cómo agregar un proceso en Azure Machine Learning Service](./media/how-to-enable-virtual-network/add-compute.png)

1. Para configurar este recurso de proceso para usar una red virtual, use estas opciones:

    - __Network configuration__ (Configuración de red): Seleccione __Advanced__ (Avanzadas).

    - __Grupo de recursos__: Seleccione el grupo de recursos que contiene la red virtual.

    - __Red virtual__: Seleccione la red virtual que contiene la subred.

    - __Subred__: Seleccione la subred que se va a usar.

   ![Captura de pantalla que muestra la configuración de la red virtual para los procesos de Machine Learning](./media/how-to-enable-virtual-network/amlcompute-virtual-network-screen.png)

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
                                                           vnet_resourcegroup_name = vnet_resourcegroup_name,
                                                           vnet_name = vnet_name,
                                                           subnet_name = subnet_name)

    # Create the cluster with the specified name and configuration
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)
    
    # Wait for the cluster to complete, show the output log
    cpu_cluster.wait_for_completion(show_output=True)
```

Cuando finalice el proceso de creación, puede entrenar el modelo con el clúster. Para más información, consulte [Selección y uso de un destino de proceso para entrenamiento](how-to-set-up-training-targets.md).

## <a name="use-a-virtual-machine-or-hdinsight-cluster"></a>Uso de una máquina virtual o un clúster de HDInsight

Para usar una máquina virtual o un clúster de HDInsight en una red virtual con el área de trabajo, siga estos pasos:

> [!IMPORTANT]
> Azure Machine Learning Service solo admite máquinas virtuales que ejecuten Ubuntu.

1. Cree una máquina virtual o un clúster de HDInsight mediante Azure Portal o la CLI de Azure y colóquelos en una red virtual de Azure. Para obtener más información, vea los documentos siguientes:
    * [Creación y administración de redes virtuales de Azure para máquinas virtuales Linux](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-virtual-network)

    * [Extender HDInsight mediante una red virtual de Azure](https://docs.microsoft.com/azure/hdinsight/hdinsight-extend-hadoop-virtual-network) 

1. Para permitir que Azure Machine Learning Service se comunique con el puerto SSH en la máquina virtual o el clúster, debe configurar una entrada de origen para el grupo de seguridad de red. El puerto SSH suele ser el 22. Para permitir el tráfico desde este origen, use la siguiente información:

    * __Origen__: Seleccione __Service Tag__ (Etiqueta de servicio).

    * __Etiqueta de servicio de origen__: Seleccione __AzureMachineLearning__.

    * __Intervalos de puertos de origen__: Seleccione __*__.

    * __Destino__: Seleccione __Todo__.

    * __Intervalos de puertos de destino__: Seleccione __22__.

    * __Protocolo__: Seleccione __Todo__.

    * __Acción__: seleccione __Permitir__.

   ![Captura de pantalla de las reglas de entrada para realizar la experimentación en una máquina virtual o un clúster de HDInsight dentro de una red virtual](./media/how-to-enable-virtual-network/experimentation-virtual-network-inbound.png)

    Mantenga las reglas de salida predeterminadas para el grupo de seguridad de red. Para más información, consulte las reglas de seguridad predeterminadas en [Grupos de seguridad](https://docs.microsoft.com/azure/virtual-network/security-overview#default-security-rules).

    Si no desea usar las reglas de salida predeterminado y desea limitar el acceso de salida de la red virtual, consulte [limitan la conectividad saliente de la red virtual](#limiting-outbound-from-vnet)
    
1. Conecte el clúster de HDInsight o de máquina virtual a su área de trabajo de Azure Machine Learning Service. Para más información, consulte [Configuración de destinos de proceso del entrenamiento del modelo](how-to-set-up-training-targets.md).

## <a name="use-azure-kubernetes-service"></a>Uso de Azure Kubernetes Service

> [!IMPORTANT]
> Compruebe los requisitos previos y planee el direccionamiento IP para el clúster antes de continuar con los pasos que vienen a continuación. Para más información, consulte [Configuración de redes avanzadas en Azure Kubernetes Service](https://docs.microsoft.com/azure/aks/configure-advanced-networking).
> 

> Mantenga las reglas de salida predeterminadas para el grupo de seguridad de red. Para más información, consulte las reglas de seguridad predeterminadas en [Grupos de seguridad](https://docs.microsoft.com/azure/virtual-network/security-overview#default-security-rules).
>
> Azure Kubernetes Service y Azure Virtual Network deben estar en la misma región.

Para agregar Azure Kubernetes Service de una red virtual al área de trabajo, siga estos pasos en Azure Portal:

1. NSG realícela que los controles de la red virtual tiene la regla habilitada para el servicio de Azure Machine Learning con entrada __etiqueta de servicio__ de __AzureMachineLearning__

    ![Cómo agregar un proceso en Azure Machine Learning Service](./media/how-to-enable-virtual-network/aks-vnet-inbound-nsg-aml.png)     
 
1. En [Azure Portal](https://portal.azure.com), seleccione el área de trabajo de Azure Machine Learning Service.

1. En la sección __Aplicación__, seleccione __Proceso__. A continuación, seleccione __Agregar proceso__. 

    ![Cómo agregar un proceso en Azure Machine Learning Service](./media/how-to-enable-virtual-network/add-compute.png)

1. Para configurar este recurso de proceso para usar una red virtual, use estas opciones:

    - __Network configuration__ (Configuración de red): Seleccione __Advanced__ (Avanzadas).

    - __Grupo de recursos__: Seleccione el grupo de recursos que contiene la red virtual.

    - __Red virtual__: Seleccione la red virtual que contiene la subred.

    - __Subred__: Seleccione la subred.

    - __Intervalo de direcciones del servicio de Kubernetes__: seleccione el intervalo de direcciones del servicio de Kubernetes. Este intervalo de direcciones utiliza un intervalo de IP de notación CIDR al definir las direcciones IP disponibles para el clúster. No debe superponerse con ningún intervalo IP de subred. Por ejemplo: 10.0.0.0/16.

    - __Dirección IP del servicio DNS de Kubernetes__: Seleccione la dirección IP del servicio DNS de Kubernetes. Esta dirección IP se asigna al servicio DNS de Kubernetes. Debe estar dentro del intervalo de direcciones del servicio Kubernetes. Por ejemplo: 10.0.0.10.

    - __Dirección del puente de Docker__: Seleccione la dirección del puente de Docker. Esta dirección IP se asigna al puente de Docker. No debe estar en ningún intervalo IP de subred o en el intervalo de direcciones del servicio Kubernetes. Por ejemplo: 172.17.0.1/16.

   ![Azure Machine Learning Service: Configuración de la red virtual del Proceso de Machine Learning](./media/how-to-enable-virtual-network/aks-virtual-network-screen.png)

1. NSG realícela que los controles de que la red virtual tiene la regla habilitada para el punto de conexión de puntuación para que se puede llamar desde fuera de la red virtual de entrada

    ![Cómo agregar un proceso en Azure Machine Learning Service](./media/how-to-enable-virtual-network/aks-vnet-inbound-nsg-scoring.png)

    > [!TIP]
    > Si ya tiene un clúster de AKS en una red virtual, puede asociarlo al área de trabajo. Para más información, consulte [Implementación en AKS](how-to-deploy-to-aks.md).

También puede usar el **SDK de Azure Machine Learning** para agregar Azure Kubernetes Service en una red virtual. El código siguiente crea una instancia de Azure Kubernetes Service en la subred `default` de una red virtual denominada `mynetwork`:

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
aks_target = ComputeTarget.create(workspace = ws,
                                  name = "myaks",
                                  provisioning_configuration = config)
```

Cuando finalice el proceso de creación, puede realizar la inferencia en un clúster de AKS detrás de una red virtual. Para más información, consulte [Implementación en AKS](how-to-deploy-to-aks.md).

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de entornos de entrenamiento](how-to-set-up-training-targets.md)
* [Lugar de implementación de modelos](how-to-deploy-and-where.md)
* [Implementación segura de modelos con SSL](how-to-secure-web-service.md)

