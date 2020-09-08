---
title: Uso de los destinos de proceso para el entrenamiento de modelos
titleSuffix: Azure Machine Learning
description: Configurar los entornos de entrenamiento (destinos de proceso) del entrenamiento del modelo de Machine Learning. Es fácil cambiar entre entornos de entrenamiento. Inicie el entrenamiento de forma local. Si necesita escalar horizontalmente, cambie a un destino de proceso basado en la nube.
services: machine-learning
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 07/08/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: e83faee7d72026dafc50b21d0a0773e663e5a03a
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88933125"
---
# <a name="set-up-and-use-compute-targets-for-model-training"></a>Configuración y uso de destinos de proceso para el entrenamiento del modelo 
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Con Azure Machine Learning, puede entrenar el modelo en una variedad de recursos o entornos, denominados colectivamente [__destinos de proceso__](concept-azure-machine-learning-architecture.md#compute-targets). Un destino de proceso puede ser una máquina local o un recurso de nube, como una instancia de proceso de Azure Machine Learning, Azure HDInsight o una máquina virtual remota.  También puede crear destinos de proceso para la implementación de modelos tal como se describe en ["Cómo y dónde implementar los modelos"](how-to-deploy-and-where.md).

Los destinos de proceso se pueden crear y administrar mediante el SDK de Azure Machine Learning, Azure Machine Learning Studio, la CLI de Azure o la extensión de VS Code de Azure Machine Learning. Si tiene destinos de proceso creados mediante cualquier otro servicio (por ejemplo, un clúster de HDInsight), para usarlos debe adjuntarlos al área de trabajo de Azure Machine Learning.
 
En este artículo, aprenderá a usar diversos destinos de proceso para el entrenamiento de modelos.  Los pasos para todos los destinos de proceso siguen el mismo flujo de trabajo:
1. __Crear__ un destino de proceso si aún no tiene uno.
2. __Asociar__ el destino de proceso al área de trabajo.
3. __Configurar__ el destino de proceso para que contenga las dependencias de paquete y entorno de Python necesarias por el script.


>[!NOTE]
> El código de este artículo se probó con la versión 1.0.74 del SDK de Azure Machine Learning.

## <a name="compute-targets-for-training"></a>Destinos de proceso para entrenamiento

Azure Machine Learning tiene distintas modalidades de soporte técnico en los diferentes destinos de proceso. Un ciclo de vida de desarrollo de modelos típico comienza con el desarrollo y la experimentación en una pequeña cantidad de datos. En esta fase, se recomienda usar un entorno local. Por ejemplo, un equipo local o una máquina virtual basada en la nube. Si escala verticalmente el entrenamiento a conjuntos de datos grandes o realiza el entrenamiento distribuido, se recomienda usar Proceso de Azure Machine Learning para crear un clúster con uno o varios nodos que se escala automáticamente cada vez que se envía una ejecución. También puede adjuntar su propio recurso de proceso, aunque el soporte técnico para varios escenarios puede variar como se detalla a continuación:

[!INCLUDE [aml-compute-target-train](../../includes/aml-compute-target-train.md)]


> [!NOTE]
> Los clústeres de Proceso de Azure Machine Learning pueden crearse como recurso persistente o crearse dinámicamente al solicitar una ejecución. La creación basada en ejecución quita el destino de proceso después de que se completa la ejecución de entrenamiento, por lo que no puede reutilizar los destinos de proceso creados de esta manera.

## <a name="whats-a-run-configuration"></a>¿En qué consiste una configuración de ejecución?

Cuando se entrena, es normal comenzar en el equipo local y después ejecutar ese script de entrenamiento en un destino de proceso diferente. Con Azure Machine Learning, puede ejecutar el script en varios destinos de proceso sin tener que cambiar el script.

Todo lo que debe hacer es definir el entorno de cada destino de proceso con una **configuración de ejecución**.  Después, cuando desee ejecutar el experimento de entrenamiento en un destino de proceso diferente, especifique la configuración de ejecución para ese proceso. Para obtener más información sobre cómo especificar un entorno y enlazarlo a la configuración de ejecución, consulte [Creación y administración de entornos para el aprendizaje y la implementación](how-to-use-environments.md).

Obtenga más información sobre cómo [enviar experimentos](#submit) al final de este artículo.

## <a name="whats-an-estimator"></a>¿Qué es un objeto "Estimator"?

Para facilitar el entrenamiento del modelo mediante plataformas populares, el SDK de Azure Machine Learning para Python proporciona una abstracción de nivel más alto alternativa, la clase "Estimator".  Esta clase le permite construir fácilmente configuraciones de ejecución. Puede crear y usar un objeto [Estimator](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator?view=azure-ml-py) genérico para enviar scripts de entrenamiento que usen cualquier plataforma de aprendizaje que elija, como scikit-learn. Se recomienda usar un objeto "estimator" para el entrenamiento, ya que crea automáticamente objetos insertados como un entorno u objetos RunConfiguration. Si desea tener más control sobre cómo se crean estos objetos y especificar los paquetes que se deben instalar para la ejecución de su experimento, siga [estos pasos](#amlcompute) para enviar sus experimentos de entrenamiento mediante un objeto RunConfiguration en un Proceso de Azure Machine Learning.

Azure Machine Learning proporciona estimadores específicos para [PyTorch](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.pytorch?view=azure-ml-py), [TensorFlow](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py), [Chainer](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.chainer?view=azure-ml-py) y [Ray RLlib](how-to-use-reinforcement-learning.md).

Para obtener más información, consulte [Entrenamiento de modelos con Azure Machine Learning mediante un objeto Estimator](how-to-train-ml-models.md).

## <a name="whats-an-ml-pipeline"></a>¿Qué es una canalización de ML?

Con las canalizaciones de ML, puede optimizar su flujo de trabajo con sencillez y velocidad, dotarlo de portabilidad y reutilizarlo. Al crear canalizaciones mediante Azure Machine Learning, puede centrarse en lo que domina, el aprendizaje automático, en lugar de en la infraestructura y la automatización.

Las canalizaciones de ML se construyen a partir de varios **pasos**, que son unidades de cálculo diferentes en la canalización. Cada paso se puede ejecutar de forma independiente y usar recursos de proceso aislados. Este enfoque permite que varios científicos de datos trabajen en la misma canalización al mismo tiempo sin sobrecargar los recursos de proceso y facilita el uso de diferentes tamaños o tipos de proceso para cada paso.

> [!TIP]
> Las canalizaciones de ML pueden utilizar la configuración de ejecución o los objetos "Estimator" al entrenar modelos.

Aunque las canalizaciones de ML pueden entrenar modelos, también pueden preparar los datos antes del entrenamiento e implementar los modelos después. Uno de los casos de uso principales para las canalizaciones es la puntuación por lotes. Para obtener más información, vea el artículo [Creación de canalizaciones de ML reutilizables en Azure Machine Learning Service](concept-ml-pipelines.md).

## <a name="set-up-in-python"></a>Configuración en Python

Use las secciones siguientes para configurar estos destinos de proceso:

* [Equipo local](#local)
* [Clúster de proceso de Azure Machine Learning](#amlcompute)
* [Instancia de proceso de Azure Machine Learning](#instance)
* [Máquinas virtuales remotas](#vm)
* [HDInsight de Azure](#hdinsight)


### <a name="local-computer"></a><a id="local"></a>Equipo local

1. **Crear y adjuntar**: no es necesario crear o asociar un destino de proceso para utilizar el equipo local como entorno de entrenamiento.  

1. **Configurar**:  cuando se utiliza el equipo local como destino de proceso, el código de entrenamiento se ejecuta en el [entorno de desarrollo](how-to-configure-environment.md).  Si ese entorno ya tiene los paquetes Python que necesita, utilice el entorno administrado por usuarios.

 [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/local.py?name=run_local)]

Ahora que ha asociado el proceso y ha configurado la ejecución, el siguiente paso es [enviar la ejecución de entrenamiento](#submit).

### <a name="azure-machine-learning-compute-cluster"></a><a id="amlcompute"></a>Clúster de Proceso de Azure Machine Learning

El clúster de Proceso de Azure Machine Learning es una infraestructura de proceso administrado que permite al usuario crear fácilmente un proceso de uno o varios nodos. El proceso se crea dentro de la región de su área de trabajo y es un recurso que se puede compartir con otros usuarios del área de trabajo. El proceso se escala verticalmente de forma automática cuando se envía un trabajo y se puede colocar en una instancia de Azure Virtual Network. El proceso se ejecuta en un entorno con contenedores y empaqueta las dependencias del modelo en un [contenedor de Docker](https://www.docker.com/why-docker).

Proceso Azure Machine Learning Compute para distribuir el proceso de entrenamiento en un clúster de nodos de proceso de CPU o GPU de la nube. Para más información sobre los tamaños de máquina virtual que incluyen GPU, consulte [Tamaños de máquinas virtuales optimizadas para GPU](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-gpu). 

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

    O bien puede crear y adjuntar un recurso persistente de Proceso de Azure Machine Learning en [Azure Machine Learning Studio](#portal-create).

   
1. **Configurar**: Cree una configuración de ejecución para el destino de proceso persistente.

   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/amlcompute2.py?name=run_amlcompute)]

Ahora que ha asociado el proceso y ha configurado la ejecución, el siguiente paso es [enviar la ejecución de entrenamiento](#submit).

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

 Los clústeres de proceso de Azure Machine Learning también admiten [identidades administradas](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) para autenticar el acceso a los recursos de Azure sin incluir credenciales en el código. Hay dos tipos de identidades administradas:

* Una **identidad administrada asignada por el sistema** se habilita directamente en el clúster de proceso de Azure Machine Learning. El ciclo de vida de una identidad asignada por el sistema está vinculado directamente al clúster de proceso. Si se elimina el clúster de proceso, Azure limpia automáticamente las credenciales y la identidad en Azure AD.
* Una **identidad administrada asignada por el usuario** es un recurso de Azure independiente que se proporciona a través del servicio de identidad administrada de Azure. Puede asignar una identidad administrada asignada por el usuario a varios recursos y se conserva durante tanto tiempo como necesite.

Use una de estas formas para especificar una identidad administrada para un clúster de proceso:
    
* En el estudio, al crear el clúster de proceso o al editar los detalles del clúster de proceso, alterne la opción **Asignar una identidad administrada** y especifique una identidad asignada por el sistema o una identidad asignada por el usuario.
    
* Con el SDK de Python, establezca el atributo `identity_type` en la configuración de aprovisionamiento.  
    
    ```python
    # configure cluster with a system-assigned managed identity
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                            max_nodes=5,
                                                            identity_type="SystemAssigned",
                                                            )

    # configure cluster with a user-assigned managed identity
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                            max_nodes=5,
                                                            identity_type="UserAssigned",
                                                            identity_id=['/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'])

    cpu_cluster_name = "cpu-cluster"
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)
    ```

* Con el SDK de Python, establezca los atributos `identity_type` y `identity_id` (si se crea una identidad administrada asignada por el usuario) en la configuración de aprovisionamiento.  
    
    ```python
    # add a system-assigned managed identity
    cpu_cluster.add_identity(identity_type="SystemAssigned")

    # add a user-assigned managed identity
    cpu_cluster.add_identity(identity_type="UserAssigned", 
                                identity_id=['/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'])
    ```
    
* Con la CLI, establezca el atributo `assign-identity` durante la creación del clúster:
    
    ```azurecli
    # create a cluster with a user-assigned managed identity
    az ml computetarget create amlcompute --name cpu-cluster --vm-size Standard_NC6 --max-nodes 5 --assign-identity '/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'

    # create a cluster with a system-managed identity
    az ml computetarget create amlcompute --name cpu-cluster --vm-size Standard_NC6 --max-nodes 5 --assign-identity '[system]'

* Using the CLI, execute the following commands to assign a managed identity on an existing cluster:
    
    ```azurecli
    # add a user-assigned managed identity
    az ml computetarget amlcompute identity assign --name cpu-cluster '/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'

    # add a system-assigned managed identity
    az ml computetarget amlcompute identity assign --name cpu-cluster '[system]'

> [!NOTE]
> Azure Machine Learning compute clusters support only **one system-assigned identity** or **multiple user-assigned identities**, not both concurrently.
> 
> Additionally, you can assign only one managed identity from the studio.

#### Managed identity usage

AML defines the **default managed identity** as the system-assigned managed identity or the first user-assigned managed identity.

During a run there are two applications of an identity:
1. The system uses an identity to setup the user's storage mounts, container registry, and datastores.
    * In this case, the system will use the default managed identity.

1. The user applies an identity to access resources from within the code for a submitted run
    
    * In this case, the user must provide the *client_id* corresponding to the managed identity they want to use to retrieve a credential. 
    * Alternatively, AML exposes the user-assigned identity's client ID through the *DEFAULT_IDENTITY_CLIENT_ID* environment variable.
    
    For example, to retrieve a token for a datastore with the default managed identity:
    
    ```python
    client_id = os.environ.get('DEFAULT_IDENTITY_CLIENT_ID')
    credential = ManagedIdentityCredential(client_id=client_id)
    token = credential.get_token('https://storage.azure.com/')



### <a id="instance"></a>Azure Machine Learning compute instance

[Azure Machine Learning compute instance](concept-compute-instance.md) is a managed-compute infrastructure that allows you to easily create a single VM. The compute is created within your workspace region, but unlike a compute cluster, an instance cannot be shared with other users in your workspace. Also the instance does not automatically scale down.  You must stop the resource to prevent ongoing charges.

A compute instance can run multiple jobs in parallel and has a job queue. 

Compute instances can run jobs securely in a [virtual network environment](how-to-enable-virtual-network.md#compute-instance), without requiring enterprises to open up SSH ports. The job executes in a containerized environment and packages your model dependencies in a Docker container. 

1. **Create and attach**: 
    
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

1. **Configurar**: Creación de una configuración de ejecución.
    
    ```python
    
    from azureml.core import ScriptRunConfig
    from azureml.core.runconfig import DEFAULT_CPU_IMAGE
    
    src = ScriptRunConfig(source_directory='', script='train.py')
    
    # Set compute target to the one created in previous step
    src.run_config.target = instance
    
    # Set environment
    src.run_config.environment = myenv
     
    run = experiment.submit(config=src)
    ```

Ahora que ha asociado el proceso y ha configurado la ejecución, el siguiente paso es [enviar la ejecución de entrenamiento](#submit).


### <a name="remote-virtual-machines"></a><a id="vm"></a>Máquinas virtuales remotas

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

   O bien puede asociar la instancia de DSVM al área de trabajo [mediante Azure Machine Learning Studio](#portal-reuse).

1. **Configurar**: Cree una configuración de ejecución para el destino de proceso de Data Science Virtual Machine. Docker y Conda se utilizan para crear y configurar el entorno de entrenamiento en la DSVM.

   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/dsvm.py?name=run_dsvm)]


Ahora que ha asociado el proceso y ha configurado la ejecución, el siguiente paso es [enviar la ejecución de entrenamiento](#submit).

### <a name="azure-hdinsight"></a><a id="hdinsight"></a>Azure HDInsight 

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

   O bien puede asociar el clúster de HDInsight al área de trabajo [mediante Azure Machine Learning Studio](#portal-reuse).

1. **Configurar**: Cree una configuración de ejecución para el destino de proceso de HDI. 

   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/hdi.py?name=run_hdi)]


Ahora que ha asociado el proceso y ha configurado la ejecución, el siguiente paso es [enviar la ejecución de entrenamiento](#submit).


### <a name="azure-batch"></a><a id="azbatch"></a>Azure Batch 

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

> [!TIP]
> Las canalizaciones de Azure Machine Learning solo pueden trabajar con datos almacenados en el almacén de datos predeterminado de la cuenta de Data Lake Analytics. Si los datos con los que necesita trabajar están en un almacén no predeterminado, puede usar [`DataTransferStep`](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.data_transfer_step.datatransferstep?view=azure-ml-py) para copiar los datos antes del entrenamiento.

## <a name="set-up-in-azure-machine-learning-studio"></a>Configuración de Azure Machine Learning Studio

Desde Azure Machine Learning Studio, puede acceder a destinos de proceso que están asociados con el área de trabajo.  Puede usar Studio para:

* [Ver los destinos de proceso](#portal-view) asociados a un área de trabajo
* [Crear un destino de proceso](#portal-create) en el área de trabajo
* [Adjuntar un destino de proceso](#portal-reuse) que se creó fuera del área de trabajo


Después de crear un destino y de adjuntarlo al área de trabajo, lo utilizará en la configuración de ejecución con un objeto `ComputeTarget`: 

```python
from azureml.core.compute import ComputeTarget
myvm = ComputeTarget(workspace=ws, name='my-vm-name')
```

### <a name="view-compute-targets"></a><a id="portal-view"></a>Visualización de destinos de proceso


Para ver los destinos de proceso del área de trabajo, use los pasos siguientes:

1. Vaya a [Azure Machine Learning Studio](https://ml.azure.com).
 
1. En __Aplicaciones__, seleccione __Proceso__.

    [![Visualización de la pestaña Proceso](./media/how-to-set-up-training-targets/azure-machine-learning-service-workspace.png)](./media/how-to-set-up-training-targets/azure-machine-learning-service-workspace-expanded.png)

### <a name="create-a-compute-target"></a><a id="portal-create"></a>Creación de un destino de proceso

Siga los pasos anteriores para ver la lista de destinos de proceso. A continuación, siga estos pasos para crear un destino de proceso: 

1. Haga clic en el signo más (+) para agregar un destino de proceso.

    ![Agregar un destino de proceso](./media/how-to-set-up-training-targets/add-compute-target.png) 

1. Escriba un nombre para el destino de proceso. 

1. Seleccione **Proceso de Machine Learning** como tipo de proceso que se usará en __Entrenamiento__. 

    >[!NOTE]
    >Proceso de Azure Machine Learning es el único recurso de proceso administrado que se puede crear en Azure Machine Learning Studio.  Todos los demás recursos de proceso se pueden asociar después de su creación.

1. Rellene el formulario. Proporcione valores para las propiedades necesarias, especialmente para **Familia de máquinas virtuales** y **Nodos máximos** que se van a utilizar para acelerar el proceso.  

1. Seleccione __Crear__.


1. Vea el estado de la operación de creación seleccionando el destino de proceso en la lista:

    ![Seleccionar un destino de proceso para ver el estado de la operación de creación](./media/how-to-set-up-training-targets/View_list.png)

1. Verá los detalles del destino de proceso: 

    ![Ver los detalles del destino de proceso](./media/how-to-set-up-training-targets/compute-target-details.png) 

### <a name="attach-compute-targets"></a><a id="portal-reuse"></a>Adjuntar destinos de proceso

Para usar destinos de proceso creados fuera del área de trabajo de Azure Machine Learning, debe adjuntarlos. Adjuntar un destino de proceso hace que esté disponible para el área de trabajo.

Siga los pasos descritos previamente para ver la lista de destinos de proceso. Luego, siga los pasos a continuación para adjuntar un destino de proceso: 

1. Haga clic en el signo más (+) para agregar un destino de proceso. 
1. Escriba un nombre para el destino de proceso. 
1. Seleccione el tipo de proceso que va a adjuntar para __Entrenamiento__:

    > [!IMPORTANT]
    > No todos los tipos de proceso se pueden adjuntar desde Azure Machine Learning Studio. Los tipos de proceso que se pueden adjuntar actualmente para entrenamiento incluyen:
    >
    > * Una máquina virtual remota
    > * Azure Databricks (para su uso en las canalizaciones de aprendizaje automático)
    > * Azure Data Lake Analytics (para su uso en las canalizaciones de aprendizaje automático)
    > * HDInsight de Azure

1. Rellene el formulario y proporcione valores para las propiedades necesarias.

    > [!NOTE]
    > Microsoft recomienda que use claves SSH, que son más seguras que las contraseñas. Las contraseñas son vulnerables a ataques por fuerza bruta. Las claves SSH se basan en las firmas criptográficas. Para información sobre cómo crear claves SSH para usarlas con Azure Virtual Machines, consulte los siguientes documentos:
    >
    > * [Creación y uso de claves SSH en Linux o macOS](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys)
    > * [Creación y uso de claves SSH en Windows](https://docs.microsoft.com/azure/virtual-machines/linux/ssh-from-windows)

1. Seleccione __Adjuntar__. 
1. Consulte el estado de la operación de adjuntar seleccionando el destino de proceso en la lista.

## <a name="set-up-with-cli"></a>Configuración con la CLI

Puede acceder a los destinos de proceso asociados con el área de trabajo mediante la [extensión de la CLI](reference-azure-machine-learning-cli.md) para Azure Machine Learning.  Puede usar la CLI para:

* Crear un destino de proceso administrado
* Actualizar un destino de proceso administrado
* Asociar un destino de proceso no administrado

Para más información, consulte el artículo sobre la [administración de recursos](reference-azure-machine-learning-cli.md#resource-management).

## <a name="set-up-with-vs-code"></a>Configuración con VS Code

Puede crear y administrar los destinos de proceso asociados con el área de trabajo, así como acceder a ellos, mediante la [extensión de VS Code](how-to-manage-resources-vscode.md#compute-clusters) para Azure Machine Learning.

## <a name="submit-training-run-using-azure-machine-learning-sdk"></a><a id="submit"></a>Envío de una ejecución de entrenamiento mediante el SDK de Azure Machine Learning

Después de crear una configuración de ejecución, se utiliza para ejecutar el experimento.  El patrón de código para enviar una ejecución de entrenamiento es el mismo para todos los tipos de destinos de proceso:

1. Creación de un experimento para su ejecución
1. Envíe la ejecución.
1. Espere a que la ejecución se complete.

> [!IMPORTANT]
> Cuando envía la ejecución de entrenamiento, se crea una instantánea del directorio que contiene los scripts de entrenamiento y se envía al destino de proceso. También se almacena como parte del experimento del área de trabajo. Si cambia los archivos y vuelve a enviar la ejecución, solo se cargarán los archivos cambiados.
>
> [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]
> 
> Para más información, consulte [Instantánea](concept-azure-machine-learning-architecture.md#snapshots).

### <a name="create-an-experiment"></a>Creación de un experimento

En primer lugar, cree un experimento en el área de trabajo.

[!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/local.py?name=experiment)]

### <a name="submit-the-experiment"></a>Envío del experimento

Envíe el experimento con un objeto `ScriptRunConfig`.  Este objeto incluye:

* **source_directory**: el directorio de origen que contiene el script de entrenamiento.
* **script**: Identificación del script de entrenamiento
* **run_config**: la configuración de ejecución, que a su vez define donde tendrá lugar el entrenamiento.

Por ejemplo, para usar la configuración del [destino local](#local):

[!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/local.py?name=local_submit)]

Cambie el mismo experimento para que se ejecute en un destino de proceso diferente mediante una configuración de ejecución diferente, como [amlcompute target](#amlcompute):

[!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/amlcompute2.py?name=amlcompute_submit)]

> [!TIP]
> De forma predeterminada, en este ejemplo solo se usa un nodo del destino de proceso para el entrenamiento. Para usar más de un nodo, establezca el `node_count` de la configuración de ejecución en el número deseado de nodos. Por ejemplo, el código siguiente establece el número de nodos que se usan para el entrenamiento en cuatro:
>
> ```python
> src.run_config.node_count = 4
> ```

También puede:

* Enviar el experimento con un objeto `Estimator`, tal como se muestra en [Entrenamiento de modelos de aprendizaje automático con estimadores](how-to-train-ml-models.md).
* Envíe una ejecución de HyperDrive para el [ajuste de hiperparámetros](how-to-tune-hyperparameters.md).
* Enviar un experimento mediante la [extensión de VS Code](tutorial-train-deploy-image-classification-model-vscode.md#train-the-model).

Para obtener más información, consulte la documentación de [ScriptRunConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.scriptrunconfig?view=azure-ml-py) y [RunConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfiguration?view=azure-ml-py).

## <a name="create-run-configuration-and-submit-run-using-azure-machine-learning-cli"></a>Creación de una configuración de ejecución y envío de la ejecución mediante la CLI de Azure Machine Learning

Puede usar la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) y la [extensión de la CLI de Machine Learning](reference-azure-machine-learning-cli.md) para crear configuraciones de ejecución y enviar esas ejecuciones a distintos destinos del proceso. En los ejemplos siguientes se supone que ya tiene un área de trabajo de Azure Machine Learning y que ha iniciado sesión en Azure mediante el comando de la CLI `az login`. 

[!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)] 

### <a name="create-run-configuration"></a>Creación de una configuración de ejecución

La manera más sencilla de crear la configuración de ejecución es navegar por la carpeta que contiene los scripts de Python de Machine Learning y usar el comando de la CLI.

```azurecli
az ml folder attach
```

Este comando crea una subcarpeta `.azureml` que contiene archivos de configuración de ejecución de plantillas para distintos destinos de proceso. Puede copiar y editar estos archivos para personalizar la configuración; por ejemplo, para agregar paquetes de Python o cambiar la configuración de Docker.  

### <a name="structure-of-run-configuration-file"></a>Estructura del archivo de configuración de ejecución

El archivo de configuración de ejecución tiene un formato YAML y las siguientes secciones:
 * El script que se ejecutará y sus argumentos
 * El nombre del destino de proceso, ya sea "local" o el nombre de un proceso del área de trabajo.
 * Los parámetros para realizar la ejecución: el marco de trabajo, el comunicador para ejecuciones distribuidas, una duración máxima y el número de nodos de proceso.
 * La sección del entorno. Consulte [Creación y administración de entornos para el aprendizaje y la implementación](how-to-use-environments.md) para obtener más información sobre los campos de esta sección.
   * Para especificar los paquetes de Python que se van a instalar para la ejecución, cree el [archivo de entorno de CONDA](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#create-env-file-manually) y establezca el campo __condaDependenciesFile__.
 * Detalles del historial de ejecución para especificar la carpeta del archivo de registro y para habilitar o deshabilitar las instantáneas de la colección de salida y del historial de ejecución.
 * Detalles de configuración específicos del marco seleccionado.
 * Detalles de la referencia de datos y del almacén de datos.
 * Detalles de configuración específicos para el Proceso de Machine Learning para crear un nuevo clúster.

Vea el [archivo JSON](https://github.com/microsoft/MLOps/blob/b4bdcf8c369d188e83f40be8b748b49821f71cf2/infra-as-code/runconfigschema.json) de ejemplo para obtener un esquema runconfig completo.

### <a name="create-an-experiment"></a>Creación de un experimento

En primer lugar, cree un experimento para las ejecuciones.

```azurecli
az ml experiment create -n <experiment>
```

### <a name="script-run"></a>Ejecución de script

Para enviar una ejecución de script, ejecute un comando

```azurecli
az ml run submit-script -e <experiment> -c <runconfig> my_train.py
```

### <a name="hyperdrive-run"></a>Ejecución de HyperDrive

Puede usar HyperDrive con la CLI de Azure para realizar ejecuciones de ajuste de parámetros. En primer lugar, cree un archivo de configuración de HyperDrive con el siguiente formato. Consulte el artículo sobre el [ajuste de hiperparámetros del modelo](how-to-tune-hyperparameters.md) para obtener más información sobre los parámetros de ajuste de hiperparámetros.

```yml
# hdconfig.yml
sampling: 
    type: random # Supported options: Random, Grid, Bayesian
    parameter_space: # specify a name|expression|values tuple for each parameter.
    - name: --penalty # The name of a script parameter to generate values for.
      expression: choice # supported options: choice, randint, uniform, quniform, loguniform, qloguniform, normal, qnormal, lognormal, qlognormal
      values: [0.5, 1, 1.5] # The list of values, the number of values is dependent on the expression specified.
policy: 
    type: BanditPolicy # Supported options: BanditPolicy, MedianStoppingPolicy, TruncationSelectionPolicy, NoTerminationPolicy
    evaluation_interval: 1 # Policy properties are policy specific. See the above link for policy specific parameter details.
    slack_factor: 0.2
primary_metric_name: Accuracy # The metric used when evaluating the policy
primary_metric_goal: Maximize # Maximize|Minimize
max_total_runs: 8 # The maximum number of runs to generate
max_concurrent_runs: 2 # The number of runs that can run concurrently.
max_duration_minutes: 100 # The maximum length of time to run the experiment before cancelling.
```

Agregue este archivo junto con los archivos de configuración de ejecución. A continuación, envíe una ejecución de HyperDrive mediante:
```azurecli
az ml run submit-hyperdrive -e <experiment> -c <runconfig> --hyperdrive-configuration-name <hdconfig> my_train.py
```

Observe la sección *arguments* (argumentos) en runconfig y el *espacio de parámetros* en la configuración de HyperDrive. Contienen los argumentos de la línea de comandos que se van a pasar al script de aprendizaje. El valor de runconfig permanece igual para cada iteración, mientras que el intervalo de la configuración de HyperDrive se repite en interación. No especifique el mismo argumento en ambos archivos.

Para obtener más información sobre estos comandos de la CLI ```az ml```, consulte la [documentación de referencia](reference-azure-machine-learning-cli.md).

<a id="gitintegration"></a>

## <a name="git-tracking-and-integration"></a>Integración y seguimiento de Git

Cuando se inicia una ejecución de entrenamiento en la que el directorio de origen es un repositorio de GIT local, se almacena información sobre el repositorio en el historial de ejecución. Para más información, consulte [Integración de Git con Azure Machine Learning](concept-train-model-git-integration.md).

## <a name="notebook-examples"></a>Ejemplos de cuadernos

Consulte estos cuadernos para ver ejemplos de entrenamiento con varios destinos de proceso:
* [how-to-use-azureml/training](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training)
* [tutorials/img-classification-part1-training.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/image-classification-mnist-data/img-classification-part1-training.ipynb)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Entrenamiento de un modelo](tutorial-train-models-with-aml.md) utiliza un destino de proceso administrado para entrenar un modelo.
* Obtenga información sobre cómo [ajustar los hiperparámetros eficazmente](how-to-tune-hyperparameters.md) para crear modelos mejores.
* Cuando tenga un modelo entrenado, aprenda [cómo y dónde implementar los modelos](how-to-deploy-and-where.md).
* Consulte la referencia del SDK de la [clase RunConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfig.runconfiguration?view=azure-ml-py).
* [Uso de Azure Machine Learning con Azure Virtual Networks](how-to-enable-virtual-network.md)
