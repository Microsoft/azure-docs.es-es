---
title: Creación de un entrenamiento e implementación de los procesos (Python)
titleSuffix: Azure Machine Learning
description: Uso del SDK de Python de Azure Machine Learning para crear recursos de proceso de entrenamiento e implementación (destinos de proceso) para el aprendizaje automático
services: machine-learning
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 07/08/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, contperfq1
ms.openlocfilehash: ac440db4c1dbddd317743e2d681a62251624d9bd
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898121"
---
# <a name="create-compute-targets-for-model-training-and-deployment-with-python-sdk"></a>Creación de destinos de proceso para la implementación y el entrenamiento de modelos con el SDK de Python

En este artículo, se usa el SDK de Python de Azure Machine Learning para crear y administrar destinos de proceso. También puede crear y administrar destinos de proceso con:
* [Azure Machine Learning Studio](how-to-create-attach-compute-studio.md) 
* La [extensión de la CLI](reference-azure-machine-learning-cli.md#resource-management) para Azure Machine Learning
* La [extensión de VS Code](how-to-manage-resources-vscode.md#compute-clusters) para Azure Machine Learning.


## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
* El [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py&preserve-view=true)
* Un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

## <a name="limitations"></a>Limitaciones

* **No cree varios datos adjuntos simultáneos en el mismo proceso** desde su área de trabajo. Por ejemplo, adjuntar un clúster de Azure Kubernetes Service a un área de trabajo con dos nombres diferentes. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

    Si quiere volver a adjuntar un destino de proceso, por ejemplo, para cambiar la configuración de TLS u otra configuración del clúster, primero debe eliminar los datos adjuntos existentes.

* Algunos de los escenarios que se enumeran en este documento se marcan como __versión preliminar__. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="whats-a-compute-target"></a>¿Qué es un destino de proceso?

Con Azure Machine Learning, puede entrenar el modelo en una variedad de recursos o entornos, denominados colectivamente [__destinos de proceso__](concept-azure-machine-learning-architecture.md#compute-targets). Un destino de proceso puede ser una máquina local o un recurso de nube, como una instancia de proceso de Azure Machine Learning, Azure HDInsight o una máquina virtual remota.  También puede crear destinos de proceso para la implementación de modelos tal como se describe en ["Cómo y dónde implementar los modelos"](how-to-deploy-and-where.md).

## <a name="compute-targets-for-training"></a>Destinos de proceso para entrenamiento

Azure Machine Learning tiene distintas modalidades de soporte técnico en los diferentes destinos de proceso. Un ciclo de vida de desarrollo de modelos típico comienza con el desarrollo y la experimentación en una pequeña cantidad de datos. En esta fase, se recomienda usar un entorno local. Por ejemplo, un equipo local o una máquina virtual basada en la nube. Si escala verticalmente el entrenamiento a conjuntos de datos grandes o realiza el entrenamiento distribuido, se recomienda usar Proceso de Azure Machine Learning para crear un clúster con uno o varios nodos que se escala automáticamente cada vez que se envía una ejecución. También puede adjuntar su propio recurso de proceso, aunque el soporte técnico para varios escenarios puede variar como se detalla a continuación:

[!INCLUDE [aml-compute-target-train](../../includes/aml-compute-target-train.md)]

> [!NOTE]
> Los clústeres de Proceso de Azure Machine Learning pueden crearse como recurso persistente o crearse dinámicamente al solicitar una ejecución. La creación basada en ejecución quita el destino de proceso después de que se completa la ejecución de entrenamiento, por lo que no puede reutilizar los destinos de proceso creados de esta manera.

Use las secciones siguientes para configurar estos destinos de proceso:

* [Equipo local](#local)
* [Clúster de proceso de Azure Machine Learning](#amlcompute)
* [Instancia de proceso de Azure Machine Learning](#instance)
* [Máquinas virtuales remotas](#vm)
* [HDInsight de Azure](#hdinsight)

## <a name="compute-targets-for-inference"></a>Destinos de proceso para inferencia

Al realizar la inferencia, Azure Machine Learning crea un contenedor de Docker que hospeda el modelo y los recursos asociados necesarios para utilizarlo. Este contenedor se usa después en uno de los siguientes escenarios de implementación:

* Como __servicio web__ que se usa para la inferencia en tiempo real. Las implementaciones de servicios web usan uno de los siguientes destinos de proceso:

    * [Equipo local](#local)
    * [Instancia de proceso de Azure Machine Learning](#instance)
    * [Azure Container Instances](#aci)
    * [Azure Kubernetes Services](how-to-create-attach-kubernetes.md)
    * Azure Functions (versión preliminar). La implementación en Azure Functions solo usa Azure Machine Learning para compilar el contenedor de Docker. Desde allí, se implementa mediante Azure Functions. Para obtener más información, consulte [Implementación de un modelo de Machine Learning en Azure Functions (versión preliminar)](how-to-deploy-functions.md).

* Como punto de conexión de __inferencia de lotes__ que se usa para procesar periódicamente lotes de datos. Las inferencias de lotes usan el [clúster de proceso de Azure Machine Learning](#amlcompute).

* A un __dispositivo IoT__ (versión preliminar). La implementación en un dispositivo IoT solo usa Azure Machine Learning para compilar el contenedor de Docker. Desde allí, se implementa mediante Azure IoT Edge. Para más información, consulte [Implementación del módulo IoT Edge (versión preliminar)](/azure/iot-edge/tutorial-deploy-machine-learning).

## <a name="local-computer"></a><a id="local"></a>Equipo local

Cuando se usa el equipo local para el **entrenamiento**, no es necesario crear un destino de proceso.  Simplemente [envíe la ejecución de entrenamiento](how-to-set-up-training-targets.md) desde la máquina local.

Cuando se usa el equipo local para la **inferencia**, debe tener instalado Docker. Para realizar la implementación, utilice [LocalWebservice.deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.local.localwebservice?view=azure-ml-py#deploy-configuration-port-none-) para definir el puerto que utilizará el servicio web. A continuación, use el proceso de implementación normal como se describe en [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

## <a name="azure-machine-learning-compute-cluster"></a><a id="amlcompute"></a>Clúster de Proceso de Azure Machine Learning

El clúster de Proceso de Azure Machine Learning es una infraestructura de proceso administrado que permite al usuario crear fácilmente un proceso de uno o varios nodos. El proceso se crea dentro de la región de su área de trabajo y es un recurso que se puede compartir con otros usuarios del área de trabajo. El proceso se escala verticalmente de forma automática cuando se envía un trabajo y se puede colocar en una instancia de Azure Virtual Network. El proceso se ejecuta en un entorno con contenedores y empaqueta las dependencias del modelo en un [contenedor de Docker](https://www.docker.com/why-docker).

Puede usar Proceso de Azure Machine Learning para distribuir un proceso de entrenamiento o inferencia de lotes en un clúster de nodos de proceso de CPU o GPU en la nube. Para más información sobre los tamaños de máquina virtual que incluyen GPU, consulte [Tamaños de máquinas virtuales optimizadas para GPU](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-gpu). 

Proceso de Azure Machine Learning tiene límites predeterminados, como el número de núcleos que se pueden asignar. Para más información, consulte [Administración y solicitud de cuotas para recursos de Azure](how-to-manage-quotas.md).

> [!TIP]
> Por lo general, los clústeres pueden escalar verticalmente hasta 100 nodos, siempre y cuando tenga la cuota suficiente para el número de núcleos necesarios. De forma predeterminada, los clústeres se configuran con la comunicación entre nodos del clúster habilitada para, por ejemplo, permitir trabajos de MPI. No obstante, puede escalar los clústeres a miles de nodos simplemente [generando una incidencia de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) y solicitando la inclusión en la lista de permitidos de su suscripción, área de trabajo o un clúster específico para deshabilitar la comunicación entre nodos. 

Se puede reutilizar una instancia de Proceso de Azure Machine Learning entre trabajos. El proceso puede compartirse con otros usuarios del área de trabajo y se conserva entre ejecuciones, escalando o reduciendo automáticamente los nodos verticalmente según el número de ejecuciones enviadas y el valor de max_nodes establecido en el clúster. La configuración min_nodes controla los nodos mínimos disponibles.

[!INCLUDE [min-nodes-note](../../includes/machine-learning-min-nodes.md)]

1. **Crear y adjuntar**: para crear un recurso de Proceso de Azure Machine Learning persistente en Python, especifique las propiedades **vm_size** y **max_nodes**. Azure Machine Learning usará valores predeterminados inteligentes para las demás propiedades. El proceso se reduce verticalmente a cero nodos cuando no se utiliza.   Se crean máquinas virtuales dedicadas para ejecutar los trabajos según sea necesario.
    
    * **vm_size**: la familia máquina virtual de los nodos creados por el Proceso de Azure Machine Learning.
    * **max_nodes**: el número máximo de nodos hasta los que se aumenta automáticamente cuando ejecuta un trabajo en Proceso de Azure Machine Learning.
    
   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/amlcompute2.py?name=cpu_cluster)]

   Cuando cree una instancia de Proceso de Azure Machine Learning, puede configurar también varias propiedades avanzadas. Estas propiedades permiten crear un clúster persistente de tamaño fijo o dentro de una instancia existente de Azure Virtual Network de su suscripción.  Consulte la [clase AmlCompute](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute?view=azure-ml-py
    ) para más información.

    O bien puede crear y adjuntar un recurso persistente de Proceso de Azure Machine Learning en [Azure Machine Learning Studio](how-to-create-attach-compute-studio.md#portal-create).

Ahora que ha adjuntado el proceso, el siguiente paso es [enviar la ejecución de entrenamiento](how-to-set-up-training-targets.md) o [ejecutar la inferencia de lotes](how-to-use-parallel-run-step.md).

 ### <a name="lower-your-compute-cluster-cost"></a><a id="low-pri-vm"></a> Reducción del costo del clúster de proceso

También puede optar por usar [VM de prioridad baja](concept-plan-manage-cost.md#low-pri-vm) para ejecutar algunas cargas de trabajo o todas ellas. Estas máquinas virtuales no tienen una disponibilidad garantizada y se pueden reemplazar mientras están en uso. Un trabajo reemplazado se reinicia, no se reanuda. 

Use cualquiera de estas formas para especificar una VM de prioridad baja:
    
* En Studio, elija **Prioridad baja** al crear una máquina virtual.
    
* Con el SDK de Python, establezca el atributo `vm_priority` en la configuración de aprovisionamiento.  
    
    ```python
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                                vm_priority='lowpriority',
                                                                max_nodes=4)
    ```
    
* Con la CLI, establezca `vm-priority`:
    
    ```azurecli-interactive
    az ml computetarget create amlcompute --name lowpriocluster --vm-size Standard_NC6 --max-nodes 5 --vm-priority lowpriority
    ```

 ### <a name="set-up-managed-identity"></a><a id="managed-identity"></a> Configuración de la identidad administrada

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-intro.md)]

Configure la identidad administrada en la configuración de aprovisionamiento:  
    
* Identidad administrada asignada por el sistema:
    ```python
    # configure cluster with a system-assigned managed identity
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                            max_nodes=5,
                                                            identity_type="SystemAssigned",
                                                            )
    ```

* Identidad administrada asignada por el usuario: 

    ```python
    # configure cluster with a user-assigned managed identity
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                            max_nodes=5,
                                                            identity_type="UserAssigned",
                                                            identity_id=['/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'])

    cpu_cluster_name = "cpu-cluster"
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)
    ```

Agregue la identidad administrada a un clúster de proceso existente:  
    
* Identidad administrada asignada por el sistema:

    ```python
    # add a system-assigned managed identity
    cpu_cluster.add_identity(identity_type="SystemAssigned")
    ````

* Identidad administrada asignada por el usuario:

    ```python
    # add a user-assigned managed identity
    cpu_cluster.add_identity(identity_type="UserAssigned", 
                                identity_id=['/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'])
    ```

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-note.md)]

#### <a name="managed-identity-usage"></a>Uso de la identidad administrada

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-default.md)]


## <a name="azure-machine-learning-compute-instance"></a><a id="instance"></a>Instancia de Proceso de Azure Machine Learning

La [instancia de Proceso de Azure Machine Learning](concept-compute-instance.md) es una infraestructura de proceso administrado que permite al usuario crear fácilmente una única VM. El proceso se crea dentro de la región de su área de trabajo, pero, a diferencia de un clúster de proceso, no se puede compartir una instancia con otros usuarios del área de trabajo. Además, la instancia no se reduce verticalmente de manera automática.  Debe detenerse el recurso para evitar que se apliquen cargos continuos.

Una instancia de proceso puede ejecutar varios trabajos en paralelo y tiene una cola de trabajos. 

Las instancias de proceso pueden ejecutar trabajos de manera segura en un [entorno de red virtual](how-to-enable-virtual-network.md#compute-instance), sin necesidad de que las empresas abran puertos SSH. El trabajo se ejecuta en un entorno con contenedores y empaqueta las dependencias del modelo en un contenedor de Docker. 

1. **Crear y adjuntar**: 
    
    ```python
    import datetime
    import time
    
    from azureml.core.compute import ComputeTarget, ComputeInstance
    from azureml.core.compute_target import ComputeTargetException
    
    # Choose a name for your instance
    # Compute instance name should be unique across the azure region
    compute_name = "ci{}".format(ws._workspace_id)[:10]
    
    # Verify that instance does not exist already
    try:
        instance = ComputeInstance(workspace=ws, name=compute_name)
        print('Found existing instance, use it.')
    except ComputeTargetException:
        compute_config = ComputeInstance.provisioning_configuration(
            vm_size='STANDARD_D3_V2',
            ssh_public_access=False,
            # vnet_resourcegroup_name='<my-resource-group>',
            # vnet_name='<my-vnet-name>',
            # subnet_name='default',
            # admin_user_ssh_public_key='<my-sshkey>'
        )
        instance = ComputeInstance.create(ws, compute_name, compute_config)
        instance.wait_for_completion(show_output=True)
    ```

Ahora que ha adjuntado el proceso y ha configurado la ejecución, el siguiente paso es [enviar la ejecución de entrenamiento](how-to-set-up-training-targets.md) o [implementar un modelo para la inferencia](how-to-deploy-local-container-notebook-vm.md).

## <a name="azure-container-instance"></a><a id="aci"></a>Azure Container Instances

Las instancias de Azure Container Instances (ACI) se crean dinámicamente al implementar un modelo. No se puede crear o adjuntar ACI al área de trabajo de ninguna otra manera. Para más información, consulte [Implementación de un modelo en Azure Container Instances](how-to-deploy-azure-container-instance.md).

## <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Azure Kubernetes Service (AKS) permite una variedad de opciones de configuración cuando se usa con Azure Machine Learning. Para obtener más información, consulte [Creación y conexión de Azure Kubernetes Service](how-to-create-attach-kubernetes.md).

## <a name="remote-virtual-machines"></a><a id="vm"></a>Máquinas virtuales remotas

Azure Machine Learning también admite la posibilidad de que traiga su propio recurso de proceso y lo adjunte a su área de trabajo. Uno de estos tipos de recursos es una máquina virtual remota arbitraria, siempre que se pueda acceder desde Azure Machine Learning. El recurso puede ser una máquina virtual de Azure, un servidor remoto de la organización o local. En concreto, dada la dirección IP y las credenciales (nombre de usuario y contraseña o clave SSH), puede usar cualquier máquina virtual a la que se pueda acceder para las ejecuciones remotas.

Puede usar un entorno de Conda integrado en el sistema, un entorno de Python existente o un contenedor de Docker. Para realizar la ejecución en un contenedor de Docker, el motor de Docker debe estar en ejecución en la máquina virtual. Esta funcionalidad es especialmente útil cuando se desea un entorno de desarrollo o experimentación basado en la nube más flexible que un equipo local.

Use Azure Data Science Virtual Machine (DSVM) como máquina virtual de Azure preferida en este escenario. Esta máquina virtual es un entorno de desarrollo de IA y de ciencia de datos preconfigurados de Azure. La máquina virtual ofrece una selección de herramientas y plataformas mantenidas para el desarrollo del aprendizaje automático a lo largo de todo el ciclo de vida. Para obtener más información sobre cómo usar la DSVM con Azure Machine Learning, consulte [Configuración del entorno de desarrollo](https://docs.microsoft.com/azure/machine-learning/how-to-configure-environment#dsvm).

1. **Crear**: Cree una instancia de DSVM antes de usarla para entrenar el modelo. Para crear este recurso, consulte [Aprovisionamiento de Data Science Virtual Machine para Linux (Ubuntu)](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro).

    > [!WARNING]
    > Azure Machine Learning solo admite máquinas virtuales que ejecuten **Ubuntu**. Al crear una máquina virtual o elegir una máquina virtual existente, debe seleccionar una máquina virtual que use Ubuntu.
    > 
    > Azure Machine Learning también requiere que la máquina virtual tenga una __dirección IP pública__.

1. **Adjuntar**: para asociar una máquina virtual existente como destino de proceso, debe proporcionar el identificador de recurso, el nombre de usuario y la contraseña de la máquina virtual. El identificador de recurso de la máquina virtual se puede construir con el identificador de la suscripción, el nombre del grupo de recursos y el nombre de la máquina virtual con el siguiente formato de cadena: `/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Compute/virtualMachines/<vm_name>`.

 
   ```python
   from azureml.core.compute import RemoteCompute, ComputeTarget

   # Create the compute config 
   compute_target_name = "attach-dsvm"
   
   attach_config = RemoteCompute.attach_configuration(resource_id='<resource_id>',
                                                   ssh_port=22,
                                                   username='<username>',
                                                   password="<password>")

   # Attach the compute
   compute = ComputeTarget.attach(ws, compute_target_name, attach_config)

   compute.wait_for_completion(show_output=True)
   ```

   O bien puede asociar la instancia de DSVM al área de trabajo [mediante Azure Machine Learning Studio](how-to-create-attach-compute-studio.md#attached-compute).

    > [!WARNING]
    > No cree varios datos adjuntos simultáneos en el mismo DSVM desde su área de trabajo. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

1. **Configurar**: Cree una configuración de ejecución para el destino de proceso de Data Science Virtual Machine. Docker y Conda se utilizan para crear y configurar el entorno de entrenamiento en la DSVM.

   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/dsvm.py?name=run_dsvm)]


Ahora que ha asociado el proceso y ha configurado la ejecución, el siguiente paso es [enviar la ejecución de entrenamiento](how-to-set-up-training-targets.md).

## <a name="azure-hdinsight"></a><a id="hdinsight"></a>Azure HDInsight 

Azure HDInsight es una plataforma popular para el análisis de macrodatos. La plataforma proporciona Apache Spark, que se puede usar para entrenar el modelo.

1. **Crear**:  Cree el clúster de HDInsight antes de usarlo para entrenar el modelo. Para crear un clúster de Spark en HDInsight, consulte [Creación de un clúster de Spark en HDInsight](https://docs.microsoft.com/azure/hdinsight/spark/apache-spark-jupyter-spark-sql). 

    > [!WARNING]
    > Azure Machine Learning requiere que el clúster de HDInsight tenga una __dirección IP pública__.

    Al crear el clúster, debe especificar un nombre de usuario SSH y una contraseña. Anote estos valores, ya que los necesitará al usar HDInsight como destino de proceso.
    
    Cuando se haya creado el clúster, conéctelo al nombre de host \<clustername>-ssh.azurehdinsight.net, donde \<clustername> es el nombre designado al clúster. 

1. **Adjuntar**: para asociar un clúster de HDInsight como destino de proceso, debe proporcionar el nombre de host, el nombre de usuario y la contraseña del clúster de HDInsight. El identificador de recurso del clúster de HDInsight se puede construir con el identificador de la suscripción, el nombre del grupo de recursos y el nombre del clúster de HDInsight con el siguiente formato de cadena: `/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.HDInsight/clusters/<cluster_name>`.

    ```python
   from azureml.core.compute import ComputeTarget, HDInsightCompute
   from azureml.exceptions import ComputeTargetException

   try:
    # if you want to connect using SSH key instead of username/password you can provide parameters private_key_file and private_key_passphrase

    attach_config = HDInsightCompute.attach_configuration(resource_id='<resource_id>',
                                                          ssh_port=22, 
                                                          username='<ssh-username>', 
                                                          password='<ssh-pwd>')
    hdi_compute = ComputeTarget.attach(workspace=ws, 
                                       name='myhdi', 
                                       attach_configuration=attach_config)

   except ComputeTargetException as e:
    print("Caught = {}".format(e.message))

   hdi_compute.wait_for_completion(show_output=True)
   ```

   O bien puede asociar el clúster de HDInsight al área de trabajo [mediante Azure Machine Learning Studio](how-to-create-attach-compute-studio.md#attached-compute).

    > [!WARNING]
    > No cree varios datos adjuntos simultáneos en el mismo HDInsight desde su área de trabajo. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

1. **Configurar**: Cree una configuración de ejecución para el destino de proceso de HDI. 

   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/hdi.py?name=run_hdi)]


Ahora que ha asociado el proceso y ha configurado la ejecución, el siguiente paso es [enviar la ejecución de entrenamiento](how-to-set-up-training-targets.md).

## <a name="azure-batch"></a><a id="azbatch"></a>Azure Batch 

Azure Batch se usa para ejecutar aplicaciones de informática de alto rendimiento (HPC) en paralelo y a gran escala de manera eficaz en la nube. AzureBatchStep se puede usar en una canalización de Azure Machine Learning para enviar trabajos al grupo de máquinas de Azure Batch.

Para adjuntar Azure Batch como destino de proceso, debe usar el SDK de Azure Machine Learning y proporcionar la siguiente información:

-    **Nombre de proceso de Azure Batch**: nombre descriptivo que se usará para el proceso en el área de trabajo.
-    **Nombre de cuenta de Azure Batch**: nombre de la cuenta de Azure Batch.
-    **Grupo de recursos**: grupo de recursos que contiene la cuenta de Azure Batch.

El código siguiente muestra cómo asociar Azure Batch como destino de proceso:

```python
from azureml.core.compute import ComputeTarget, BatchCompute
from azureml.exceptions import ComputeTargetException

# Name to associate with new compute in workspace
batch_compute_name = 'mybatchcompute'

# Batch account details needed to attach as compute to workspace
batch_account_name = "<batch_account_name>"  # Name of the Batch account
# Name of the resource group which contains this account
batch_resource_group = "<batch_resource_group>"

try:
    # check if the compute is already attached
    batch_compute = BatchCompute(ws, batch_compute_name)
except ComputeTargetException:
    print('Attaching Batch compute...')
    provisioning_config = BatchCompute.attach_configuration(
        resource_group=batch_resource_group, account_name=batch_account_name)
    batch_compute = ComputeTarget.attach(
        ws, batch_compute_name, provisioning_config)
    batch_compute.wait_for_completion()
    print("Provisioning state:{}".format(batch_compute.provisioning_state))
    print("Provisioning errors:{}".format(batch_compute.provisioning_errors))

print("Using Batch compute:{}".format(batch_compute.cluster_resource_id))
```

> [!WARNING]
> No cree varios datos adjuntos simultáneos en la misma instancia de Azure Batch desde su área de trabajo. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

### <a name="azure-databricks"></a><a id="databricks"></a>Azure Databricks

Azure Databricks es un entorno basado en Apache Spark de la nube de Azure. Se puede usar como destino de proceso con una canalización de Azure Machine Learning.

Cree un área de trabajo de Azure Databricks antes de usarlo. Para crear un recurso de área de trabajo, consulte el documento [Ejecución de un trabajo de Spark en Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/quickstart-create-databricks-workspace-portal).

Para adjuntar Azure Databricks como destino de proceso, proporcione la información siguiente:

* __Nombre del proceso de Databricks__: el nombre que desea asignar a este recurso de proceso.
* __Nombre de área de trabajo de Databricks__: el nombre del área de trabajo de Azure Databricks.
* __Token de acceso de Databricks__: el token de acceso usado para autenticarse en Azure Databricks. Para generar un token de acceso, consulte el documento [Autenticación](https://docs.azuredatabricks.net/dev-tools/api/latest/authentication.html).

En el código siguiente se muestra cómo adjuntar Azure Databricks como un destino de proceso con el SDK de Azure Machine Learning (__el área de trabajo de Databricks debe estar presente en la misma suscripción que el área de trabajo de AML__):

```python
import os
from azureml.core.compute import ComputeTarget, DatabricksCompute
from azureml.exceptions import ComputeTargetException

databricks_compute_name = os.environ.get(
    "AML_DATABRICKS_COMPUTE_NAME", "<databricks_compute_name>")
databricks_workspace_name = os.environ.get(
    "AML_DATABRICKS_WORKSPACE", "<databricks_workspace_name>")
databricks_resource_group = os.environ.get(
    "AML_DATABRICKS_RESOURCE_GROUP", "<databricks_resource_group>")
databricks_access_token = os.environ.get(
    "AML_DATABRICKS_ACCESS_TOKEN", "<databricks_access_token>")

try:
    databricks_compute = ComputeTarget(
        workspace=ws, name=databricks_compute_name)
    print('Compute target already exists')
except ComputeTargetException:
    print('compute not found')
    print('databricks_compute_name {}'.format(databricks_compute_name))
    print('databricks_workspace_name {}'.format(databricks_workspace_name))
    print('databricks_access_token {}'.format(databricks_access_token))

    # Create attach config
    attach_config = DatabricksCompute.attach_configuration(resource_group=databricks_resource_group,
                                                           workspace_name=databricks_workspace_name,
                                                           access_token=databricks_access_token)
    databricks_compute = ComputeTarget.attach(
        ws,
        databricks_compute_name,
        attach_config
    )

    databricks_compute.wait_for_completion(True)
```

Para consultar un ejemplo más detallado, vea un [cuaderno de ejemplo](https://aka.ms/pl-databricks) en GitHub.

> [!WARNING]
> No cree varios datos adjuntos simultáneos en la misma instancia de Azure Databricks desde su área de trabajo. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

### <a name="azure-data-lake-analytics"></a><a id="adla"></a>Azure Data Lake Analytics

Azure Data Lake Analytics es una plataforma de análisis de macrodatos de la nube de Azure. Se puede usar como destino de proceso con una canalización de Azure Machine Learning.

Cree una cuenta de Azure Data Lake Analytics antes de usarlo. Para crear este recurso, consulte la [Introducción a Azure Data Lake Analytics](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-get-started-portal) documento.

Para asociar Data Lake Analytics como destino de proceso, debe usar el SDK de Azure Machine Learning y proporcionar la siguiente información:

* __Nombre de proceso__: el nombre que desea asignar a este recurso de proceso.
* __Grupo de recursos__: el grupo de recursos que contiene la cuenta de Data Lake Analytics.
* __Nombre de cuenta__: El nombre de la cuenta de Data Lake Analytics.

El código siguiente muestra cómo asociar Data Lake Analytics como destino de proceso:

```python
import os
from azureml.core.compute import ComputeTarget, AdlaCompute
from azureml.exceptions import ComputeTargetException


adla_compute_name = os.environ.get(
    "AML_ADLA_COMPUTE_NAME", "<adla_compute_name>")
adla_resource_group = os.environ.get(
    "AML_ADLA_RESOURCE_GROUP", "<adla_resource_group>")
adla_account_name = os.environ.get(
    "AML_ADLA_ACCOUNT_NAME", "<adla_account_name>")

try:
    adla_compute = ComputeTarget(workspace=ws, name=adla_compute_name)
    print('Compute target already exists')
except ComputeTargetException:
    print('compute not found')
    print('adla_compute_name {}'.format(adla_compute_name))
    print('adla_resource_id {}'.format(adla_resource_group))
    print('adla_account_name {}'.format(adla_account_name))
    # create attach config
    attach_config = AdlaCompute.attach_configuration(resource_group=adla_resource_group,
                                                     account_name=adla_account_name)
    # Attach ADLA
    adla_compute = ComputeTarget.attach(
        ws,
        adla_compute_name,
        attach_config
    )

    adla_compute.wait_for_completion(True)
```

Para consultar un ejemplo más detallado, vea un [cuaderno de ejemplo](https://aka.ms/pl-adla) en GitHub.

> [!WARNING]
> No cree varios datos adjuntos simultáneos en el mismo ADLA desde su área de trabajo. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

> [!TIP]
> Las canalizaciones de Azure Machine Learning solo pueden trabajar con datos almacenados en el almacén de datos predeterminado de la cuenta de Data Lake Analytics. Si los datos con los que necesita trabajar están en un almacén no predeterminado, puede usar [`DataTransferStep`](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.data_transfer_step.datatransferstep?view=azure-ml-py&preserve-view=true) para copiar los datos antes del entrenamiento.

## <a name="notebook-examples"></a>Ejemplos de cuadernos

Consulte estos cuadernos para ver ejemplos de entrenamiento con varios destinos de proceso:
* [how-to-use-azureml/training](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training)
* [tutorials/img-classification-part1-training.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/image-classification-mnist-data/img-classification-part1-training.ipynb)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes

* Use el recurso de proceso para [enviar una ejecución de entrenamiento](how-to-set-up-training-targets.md).
* [Tutorial: Entrenamiento de un modelo](tutorial-train-models-with-aml.md) utiliza un destino de proceso administrado para entrenar un modelo.
* Obtenga información sobre cómo [ajustar los hiperparámetros eficazmente](how-to-tune-hyperparameters.md) para crear modelos mejores.
* Cuando tenga un modelo entrenado, aprenda [cómo y dónde implementar los modelos](how-to-deploy-and-where.md).
* [Uso de Azure Machine Learning con Azure Virtual Networks](how-to-enable-virtual-network.md)
