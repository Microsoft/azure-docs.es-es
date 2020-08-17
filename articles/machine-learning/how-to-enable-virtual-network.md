---
title: Aislamiento de red y privacidad
titleSuffix: Azure Machine Learning
description: Use una instancia aislada de Azure Virtual Network con Azure Machine Learning para proteger los trabajos de experimentación o aprendizaje, así como los trabajos de inferencia o puntuación.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.date: 07/07/2020
ms.topic: conceptual
ms.custom: how-to, contperfq4, tracking-python
ms.openlocfilehash: 16065b45a6afea25615b985d3c89445dee48bd1d
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88167732"
---
# <a name="network-isolation-during-training--inference-with-private-virtual-networks"></a>Aislamiento de red durante el entrenamiento e inferencia con redes virtuales privadas
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo, obtendrá información sobre cómo proteger los ciclos de vida de aprendizaje automático mediante el aislamiento de los trabajos de aprendizaje e inferencia de Azure Machine Learning en una instancia de Azure Virtual Network (red virtual). Azure Machine Learning se basa en otros servicios de Azure para recursos de proceso, también conocidos como [destinos de proceso](concept-compute-target.md), a fin de entrenar e implementar modelos. Los destinos se pueden crear dentro de una red virtual. Por ejemplo, puede usar el proceso de Azure Machine Learning para entrenar un modelo y, después, implementarlo en Azure Kubernetes Service (AKS). 

Una __red virtual__ actúa como un límite de seguridad, aislando los recursos de Azure de la internet pública. También pude unir una red virtual de Azure a la red local. Mediante la unión de redes puede entrenar de forma segura sus modelos y acceder a los modelos implementados para la inferencia.

## <a name="prerequisites"></a>Requisitos previos

+ Un [área de trabajo](how-to-manage-workspace.md) de Azure Machine Learning.

+ Conocimientos prácticos generales sobre el [servicio Azure Virtual Network](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview) y las [redes IP](https://docs.microsoft.com/azure/virtual-network/virtual-network-ip-addresses-overview-arm).

+ Una red virtual y una subred preexistentes que se usarán con los recursos de proceso.

## <a name="private-endpoints"></a>Puntos de conexión privados

También puede [habilitar Azure Private Link](how-to-configure-private-link.md) para conectarse a su área de trabajo mediante un punto de conexión privado. El punto de conexión privado es un conjunto de direcciones IP privadas dentro de la red virtual. [Obtenga información sobre cómo configurar este punto de conexión privado.](how-to-configure-private-link.md)



> [!TIP]
> Puede combinar la red virtual y Private Link para proteger la comunicación entre el área de trabajo y otros recursos de Azure. Sin embargo, algunas combinaciones requieren un área de trabajo de Enterprise Edition. Use la tabla siguiente para comprender qué escenarios requieren Enterprise Edition:
>
> | Escenario | Enterprise</br>edition | Básico</br>edition |
> | ----- |:-----:|:-----:| 
> | Sin red virtual ni Private Link | ✔ | ✔ |
> | Área de trabajo sin Private Link. Otros recursos (excepto Azure Container Registry) en una red virtual | ✔ | ✔ |
> | Área de trabajo sin Private Link. Otros recursos con Private Link | ✔ | |
> | Área de trabajo con Private Link. Otros recursos (excepto Azure Container Registry) en una red virtual | ✔ | ✔ |
> | Área de trabajo y cualquier otro recurso con Private Link | ✔ | |
> | Área de trabajo con Private Link. Otros recursos sin Private Link ni red virtual | ✔ | ✔ |
> | Azure Container Registry en una red virtual | ✔ | |
> | Claves administradas por el cliente para el área de trabajo | ✔ | |
> 

> [!WARNING]
> 
> No se admite la vista previa de instancias de proceso de Azure Machine Learning en un área de trabajo en la que Private Link está habilitado.

<a id="amlcompute"></a>

## <a name="machine-learning-studio"></a>Machine Learning Studio

Si los datos están almacenados en una red virtual, debe usar una [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md) de área de trabajo para conceder a Studio acceso a los datos.

> [!IMPORTANT]
> Aunque la mayor parte de Studio funciona con datos almacenados en una red virtual, __no__ es así en el caso de los cuadernos integrados. Los notebooks integrados no admiten el uso de almacenamiento que se encuentra en una red virtual. En su lugar, puede usar cuadernos de Jupyter Notebook en una instancia de Compute. Para obtener más información, consulte la sección [Acceso a los datos en un cuaderno de instancia de Compute](#access-data-in-a-compute-instance-notebook).

Si no puede conceder el acceso a Studio, recibirá este error `Error: Unable to profile this dataset. This might be because your data is stored behind a virtual network or your data does not support profile.` y deshabilitará las operaciones siguientes:

* Vista previa de los datos en Studio.
* Visualización de los datos en el diseñador.
* Envío de un experimento de AutoML.
* Inicio de un proyecto de etiquetado.

Studio admite la lectura de datos de los siguientes tipos de almacén de datos en una red virtual:

* Blob de Azure
* Azure Data Lake Storage Gen1
* Azure Data Lake Storage Gen2
* Azure SQL Database

### <a name="add-resources-to-the-virtual-network"></a>Adición de recursos a la red virtual 

Agregue el área de trabajo y la cuenta de almacenamiento a la misma red virtual para que puedan acceder entre sí.

1. Para conectar el área de trabajo a la red virtual, [habilite Azure Private Link](how-to-configure-private-link.md). Esta funcionalidad está actualmente en versión preliminar y está disponible en las regiones Este de EE. UU., Oeste de EE. UU. 2 y Centro y Sur de EE. UU.

1. Para conectar la cuenta de almacenamiento a la red virtual, [configure las opciones de firewalls y redes virtuales](#use-a-storage-account-for-your-workspace).

### <a name="configure-a-datastore-to-use-managed-identity"></a>Configuración de un almacén de datos para usar identidad administrada

Después de agregar el área de trabajo y la cuenta de servicio de almacenamiento a la red virtual, debe configurar almacenes de datos para que usen la identidad administrada a fin de acceder a los datos. En estos pasos se agrega la identidad administrada del área de trabajo como __Lector__ al servicio de almacenamiento mediante el control de acceso basado en recursos (RBAC) de Azure. El acceso __Lector__ permite que el área de trabajo recupere la configuración del firewall y se asegura de que los datos no salgan de la red virtual.

1. En Studio, seleccione __Almacenes de datos__.

1. Para crear un almacén de datos, seleccione __+ Nuevo almacén de datos__. Para actualizar uno existente, seleccione el almacén de datos y después __Actualizar credenciales__.

1. En la configuración del almacén de datos, seleccione __Sí__ para __Permitir que Azure Machine Learning Service acceda al almacenamiento mediante la identidad administrada del área de trabajo__.

> [!NOTE]
> Estos cambios pueden tardar hasta 10 minutos en surtir efecto.

### <a name="azure-blob-storage-blob-data-reader"></a>Lector de datos de blob de Azure Blob Storage

Para el __almacenamiento de blobs de Azure__, la identidad administrada del área de trabajo también se agrega como un [lector de datos de blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader) para que pueda leer datos del almacenamiento de blobs.


### <a name="azure-machine-learning-designer-default-datastore"></a>Almacén de datos predeterminado del diseñador de Azure Machine Learning

El diseñador usa la cuenta de almacenamiento asociada al área de trabajo para almacenar la salida de forma predeterminada. Sin embargo, puede especificarla para almacenar la salida en cualquier almacén de información al que tenga acceso. Si su entorno usa redes virtuales, puede usar estos controles para asegurarse de que los datos se mantienen seguros y son accesibles.

Para configurar un nuevo almacenamiento predeterminado para una canalización:

1. En un borrador de canalización, seleccione el **icono de engranaje de la configuración** junto al título de la canalización.
1. Seleccione la opción **Seleccionar almacén de datos predeterminado**.
1. Especifique un nuevo almacén de datos.

También puede invalidar el almacén de datos predeterminado por cada módulo. Esto le ofrece control sobre la ubicación de almacenamiento de cada módulo individual.

1. Seleccione el módulo cuya salida quiere especificar.
1. Expanda la sección **Configuración de salida**.
1. Seleccione la opción **Override default output settings** (Invalidar configuración de salida predeterminada).
1. Seleccione **Set output settings** (Establecer configuración de salida).
1. Especifique un nuevo almacén de datos.

### <a name="azure-data-lake-storage-gen2-access-control"></a>Control de acceso de Azure Data Lake Storage Gen2

Puede usar listas de control de acceso (ACL) de tipo RBAC y POSIX para controlar el acceso a los datos dentro de una red virtual.

Para usar RBAC, agregue la identidad administrada del área de trabajo al rol [Lector de datos de blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader). Para más información, consulte [Control de acceso basado en rol](../storage/blobs/data-lake-storage-access-control.md#role-based-access-control).

Para usar ACL, se puede asignar el acceso de la identidad administrada del área de trabajo como cualquier otra entidad de seguridad. Para obtener más información, vea [Listas de control de acceso en archivos y directorios](../storage/blobs/data-lake-storage-access-control.md#access-control-lists-on-files-and-directories).


### <a name="azure-data-lake-storage-gen1-access-control"></a>Control de acceso de Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 solo admite listas de control de acceso de estilo POSIX. Puede asignar el acceso de la identidad administrada del área de trabajo a los recursos como cualquier otro principio de seguridad. Para obtener más información, vea [Control de acceso en Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-access-control.md).


### <a name="azure-sql-database-contained-user"></a>Usuario independiente de Azure SQL Database

Para acceder a los datos almacenados en una instancia de Azure SQL Database mediante la identidad administrada, debe crear un usuario independiente de SQL que se asigne a la identidad administrada. Para obtener más información sobre cómo crear un usuario desde un proveedor externo, vea [Creación de usuarios independientes asignados a identidades de Azure AD](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities).

Después de crear un usuario independiente de SQL, utilice el [comando GRANT de T-SQL](https://docs.microsoft.com/sql/t-sql/statements/grant-object-permissions-transact-sql) para concederle permisos.

### <a name="connect-to-the-studio"></a>Conexión a Studio

Si accede a Studio desde un recurso dentro de una red virtual (por ejemplo, una instancia de proceso o una máquina virtual), tendrá que permitir el tráfico de salida desde la red virtual a Studio. 

Por ejemplo, si usa grupos de seguridad de red (NSG) para restringir el tráfico de salida, agregue una regla a un destino de __etiqueta de servicio__ de __AzureFrontDoor.Frontend__.

## <a name="use-a-storage-account-for-your-workspace"></a>Uso de una cuenta de almacenamiento para el área de trabajo

> [!IMPORTANT]
> Se puede colocar tanto la _cuenta de almacenamiento predeterminada_ para Azure Machine Learning o las _cuentas de almacenamiento no predeterminadas_ en una red virtual.
>
> La cuenta de almacenamiento predeterminada se aprovisiona automáticamente al crear un área de trabajo.
>
> En las cuentas de almacenamiento no predeterminadas, el parámetro `storage_account` de la [función `Workspace.create()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py#create-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--exist-ok-false--show-output-true-) permite especificar una cuenta de almacenamiento personalizada mediante el identificador de recursos de Azure.

Para usar un servicio de almacenamiento de Azure para el área de trabajo en una red virtual, siga estos pasos:

1. Cree una instancia de proceso (por ejemplo, un clúster o una instancia de proceso de Machine Learning) detrás de una red virtual o vincule un recurso de proceso al área de trabajo (por ejemplo, un clúster de HDInsight, una máquina virtual o un clúster de Azure Kubernetes Service). El recurso de proceso puede ser para experimentación o implementación de modelo.

   Para más información, consulte las secciones [Uso de una instancia de Proceso de Machine Learning](#amlcompute), [Uso de una máquina virtual o un clúster de HDInsight](#vmorhdi) y [Uso de Azure Kubernetes Service](#aksvnet) de este artículo.

1. En Azure Portal, vaya al servicio de almacenamiento que quiera usar en el área de trabajo.

   [![Almacenamiento asociado al área de trabajo de Azure Machine Learning](./media/how-to-enable-virtual-network/workspace-storage.png)](./media/how-to-enable-virtual-network/workspace-storage.png#lightbox)

1. En la página de la cuenta de servicio de almacenamiento, seleccione __Firewalls y redes virtuales__.

   ![El área "Firewalls y redes virtuales" de la página de Azure Storage de Azure Portal](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks.png)

1. En la página __Firewalls y redes virtuales__, realice las acciones siguientes:
    - Seleccione __Redes seleccionadas__.
    - En __Redes virtuales__, seleccione el vínculo __Agregar red virtual existente__. Esta acción agrega la red virtual en la que reside el proceso (vea el paso 1).

        > [!IMPORTANT]
        > La cuenta de almacenamiento debe estar en la misma red virtual y subred que las instancias de proceso o clústeres usados para entrenamiento o inferencia.

    - Seleccione la casilla __Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento__.

    > [!IMPORTANT]
    > Al trabajar con el SDK de Azure Machine Learning, el entorno de desarrollo debe poder conectarse a la cuenta de Azure Storage. Si la cuenta de almacenamiento se encuentra dentro de una red virtual, el firewall debe permitir el acceso desde la dirección IP del entorno de desarrollo.
    >
    > Para habilitar el acceso a la cuenta de almacenamiento, visite __Firewalls y redes virtuales__ de la cuenta de almacenamiento *desde un explorador web en el cliente de desarrollo*. A continuación, use la casilla __Agregar la dirección IP del cliente__ para agregar la dirección IP del cliente al campo __INTERVALO DE DIRECCIONES__. También puede usar el campo __INTERVALO DE DIRECCIONES__ para especificar manualmente la dirección IP del entorno de desarrollo. Una vez agregada la dirección IP del cliente, este puede acceder a la cuenta de almacenamiento mediante el SDK.

   [![Panel "Firewalls y redes virtuales" de Azure Portal](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks-page.png)](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks-page.png#lightbox)

## <a name="use-datastores-and-datasets"></a>Uso de almacenes de datos y conjuntos de datos

En esta sección se describe el uso de almacenes de datos y conjuntos de datos para la experiencia del SDK. Para obtener más información sobre la experiencia de Studio, vea la sección [Machine Learning Studio](#machine-learning-studio).

De forma predeterminada, Azure Machine Learning realiza comprobaciones de las credenciales y la validez de los datos al intentar acceder a ellos mediante el SDK. Si los datos están detrás de una red virtual, Azure Machine Learning no pueden acceder a ellos y se produce un error en sus comprobaciones. Para evitarlo, debe crear almacenes de datos y conjuntos de datos que omitan la validación.

### <a name="use-a-datastore"></a>Uso de un almacén de datos

 En Azure Data Lake Store Gen1 y Azure Data Lake Store Gen2 la validación se omite de forma predeterminada, por lo que no es necesario realizar ninguna otra acción. Pero para los servicios siguientes puede usar una sintaxis similar a fin de omitir la validación del almacén de datos:

- Azure Blob Storage
- Recurso compartido de archivos de Azure
- PostgreSQL
- Azure SQL Database

En el ejemplo de código siguiente se crea un almacén de datos de blobs de Azure y se establece `skip_validation=True`.

```python
blob_datastore = Datastore.register_azure_blob_container(workspace=ws,  

                                                         datastore_name=blob_datastore_name,  

                                                         container_name=container_name,  

                                                         account_name=account_name, 

                                                         account_key=account_key, 

                                                         skip_validation=True ) // Set skip_validation to true
```

### <a name="use-a-dataset"></a>Uso de un conjunto de datos

La sintaxis para omitir la validación de conjuntos de datos es similar para los siguientes tipos de datos:
- Archivo delimitado
- JSON 
- Parquet
- SQL
- Archivo

En el código siguiente se crea un conjunto de datos JSON y se establece `validate=False`.

```python
json_ds = Dataset.Tabular.from_json_lines_files(path=datastore_paths, 

validate=False) 

```


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



![Reglas de NSG de salida para Proceso de Machine Learning](./media/how-to-enable-virtual-network/experimentation-virtual-network-outbound.png)

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
    
    Cuando agregue las rutas definidas por el usuario, defina la ruta del prefijo de cada dirección IP de Batch relacionada y en __Tipo del próximo salto__, seleccione __Internet__. En la imagen siguiente se muestra un ejemplo de esta UDR en Azure Portal:

    ![Ejemplo de una ruta definida por el usuario para un prefijo de dirección](./media/how-to-enable-virtual-network/user-defined-route.png)

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

<a id="aksvnet"></a>

## <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Para agregar Azure Kubernetes Service (AKS) de una red virtual al área de trabajo, use los pasos siguientes:

> [!IMPORTANT]
> Antes de comenzar el siguiente procedimiento, debe cumplir los requisitos previos indicados en el procedimiento de [configuración de redes avanzadas en Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/configure-azure-cni#prerequisites) y planear el direccionamiento IP para el clúster.
>
> La instancia de AKS y la instancia de Azure Virtual Network deben estar en la misma región. Si protege las cuentas de Azure Storage usadas por el área de trabajo de una red virtual, deben estar en la misma red virtual que la instancia de AKS.

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

### <a name="use-private-ips-with-azure-kubernetes-service"></a>Uso de IP privadas con Azure Kubernetes Service

De forma predeterminada, se asigna una dirección IP pública a las implementaciones de AKS. Al usar AKS dentro de una red virtual, puede usar una dirección IP privada en su lugar. Solo se puede acceder a las direcciones IP privadas desde dentro de la red virtual o las redes unidas.

Una dirección IP privada se habilita mediante la configuración de AKS para usar un _equilibrador de carga interno_. 

#### <a name="network-contributor-role"></a>Rol de colaborador de red

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

#### <a name="enable-private-ip"></a>Habilitación de la IP privada

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

Para obtener información sobre cómo asociar un clúster, consulte [Asociación de un clúster de AKS ya existente](how-to-deploy-azure-kubernetes-service.md#attach-an-existing-aks-cluster).

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

## <a name="use-azure-container-instances-aci"></a>Uso de Azure Container Instances (ACI)

Azure Container Instances se crean dinámicamente al implementar un modelo. Para habilitar Azure Machine Learning para crear ACI dentro de la red virtual, debe habilitar la __delegación de subred__ para la subred que usa la implementación.

> [!WARNING]
> Al usar Azure Container Instances en una red virtual, la red virtual debe estar en el mismo grupo de recursos que el área de trabajo de Azure Machine Learning.
>
> Al usar Azure Container Instances dentro de la red virtual, Azure Container Registry (ACR) del área de trabajo no puede estar también en la red virtual.

A fin de usar ACI en una red virtual para su área de trabajo, siga los pasos siguientes:

1. Para habilitar la delegación de subred en la red virtual, use la información del artículo [Adición o eliminación de una delegación de subred](../virtual-network/manage-subnet-delegation.md). Puede habilitar la delegación al crear una red virtual o agregarla a una red existente.

    > [!IMPORTANT]
    > Al habilitar la delegación, use `Microsoft.ContainerInstance/containerGroups` como el valor __Delegar subred en un servicio__.

2. Implemente el modelo mediante [AciWebservice.deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aci.aciwebservice?view=azure-ml-py#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none--vnet-name-none--subnet-name-none-), use los parámetros `vnet_name` y `subnet_name`. Establezca estos parámetros en el nombre de red virtual y subred donde habilitó la delegación.

## <a name="azure-firewall"></a>Azure Firewall

Para obtener información sobre el uso de Azure Machine Learning con Azure Firewall, consulte [Uso del área de trabajo de Azure Machine Learning detrás de un firewall](how-to-access-azureml-behind-firewall.md).

## <a name="azure-container-registry"></a>Azure Container Registry

> [!IMPORTANT]
> Azure Container Registry (ACR) se puede colocar en una red virtual, pero debe cumplir los siguientes requisitos previos:
>
> * Un área de trabajo de Azure Machine Learning debe tener Enterprise Edition. Para obtener información sobre la actualización, vea [Actualización a Enterprise Edition](how-to-manage-workspace.md#upgrade).
> * La región del área de trabajo de Azure Machine Learning debe ser una [región con vínculo privado habilitado](https://docs.microsoft.com/azure/private-link/private-link-overview#availability). 
> * La instancia de Azure Container Registry debe tener la versión Premium. Para más información sobre la actualización, vea [Cambio de SKU](/azure/container-registry/container-registry-skus#changing-skus).
> * La instancia de Azure Container Registry debe estar en la misma red virtual y subred que la cuenta de almacenamiento y los destinos de proceso utilizados para entrenamiento o inferencia.
> * El área de trabajo de Azure Machine Learning debe contener un [clúster de proceso de Azure Machine Learning](how-to-set-up-training-targets.md#amlcompute).
>
>     Cuando la instancia de ACR está detrás de una red virtual, Azure Machine Learning no puede usarla para compilar imágenes de Docker directamente. En su lugar, se usa el clúster de proceso para compilar las imágenes.

1. Para encontrar el nombre de la instancia de Azure Container Registry para el área de trabajo, use alguno de los métodos siguientes:

    __Azure Portal__

    En la sección de información general del área de trabajo, el valor __Registro__ se vincula a la instancia de Azure Container Registry.

    :::image type="content" source="./media/how-to-enable-virtual-network/azure-machine-learning-container-registry.png" alt-text="Azure Container Registry para el área de trabajo" border="true":::

    __CLI de Azure__

    Si ha [instalado la extensión de Machine Learning para la CLI de Azure](reference-azure-machine-learning-cli.md), puede usar el comando `az ml workspace show` para mostrar la información del área de trabajo.

    ```azurecli-interactive
    az ml workspace show -w yourworkspacename -g resourcegroupname --query 'containerRegistry'
    ```

    Este comando devuelve un valor similar a `"/subscriptions/{GUID}/resourceGroups/{resourcegroupname}/providers/Microsoft.ContainerRegistry/registries/{ACRname}"`. La última parte de la cadena es el nombre de Azure Container Registry para el área de trabajo.

1. Para limitar el acceso a la red virtual, siga los pasos descritos en [Configuración del acceso de red al registro](../container-registry/container-registry-vnet.md#configure-network-access-for-registry). Al agregar la red virtual, seleccione la red virtual y la subred para los recursos de Azure Machine Learning.

1. Use el SDK de Python para Azure Machine Learning para configurar un clúster de proceso para compilar imágenes de Docker. El siguiente fragmento de código muestra cómo hacerlo:

    ```python
    from azureml.core import Workspace
    # Load workspace from an existing config file
    ws = Workspace.from_config()
    # Update the workspace to use an existing compute cluster
    ws.update(image_build_compute = 'mycomputecluster')
    ```

    > [!IMPORTANT]
    > La cuenta de almacenamiento, el clúster de proceso y Azure Container Registry deben estar en la misma subred de la red virtual.
    
    Para más información, vea la referencia del método [update()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py#update-friendly-name-none--description-none--tags-none--image-build-compute-none--enable-data-actions-none-).

1. Debe aplicar la siguiente plantilla de Azure Resource Manager. Esta plantilla permite que el área de trabajo se comunique con ACR.

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "keyVaultArmId": {
        "type": "string"
        },
        "workspaceName": {
        "type": "string"
        },
        "containerRegistryArmId": {
        "type": "string"
        },
        "applicationInsightsArmId": {
        "type": "string"
        },
        "storageAccountArmId": {
        "type": "string"
        },
        "location": {
        "type": "string"
        }
    },
    "resources": [
        {
        "type": "Microsoft.MachineLearningServices/workspaces",
        "apiVersion": "2019-11-01",
        "name": "[parameters('workspaceName')]",
        "location": "[parameters('location')]",
        "identity": {
            "type": "SystemAssigned"
        },
        "sku": {
            "tier": "enterprise",
            "name": "enterprise"
        },
        "properties": {
            "sharedPrivateLinkResources":
    [{"Name":"Acr","Properties":{"PrivateLinkResourceId":"[concat(parameters('containerRegistryArmId'), '/privateLinkResources/registry')]","GroupId":"registry","RequestMessage":"Approve","Status":"Pending"}}],
            "keyVault": "[parameters('keyVaultArmId')]",
            "containerRegistry": "[parameters('containerRegistryArmId')]",
            "applicationInsights": "[parameters('applicationInsightsArmId')]",
            "storageAccount": "[parameters('storageAccountArmId')]"
        }
        }
    ]
    }
    ```

## <a name="key-vault-instance"></a>Instancia de almacén de claves 

Azure Machine Learning usa la instancia de Key Vault que está asociada al área de trabajo para almacenar las siguientes credenciales:
* La cadena de conexión de cuenta de almacenamiento asociada
* Contraseñas para las instancias de Azure Container Repository
* Cadenas de conexión a almacenes de datos

Para usar las funcionalidades de experimentación de Azure Machine Learning con Azure Key Vault detrás de una red virtual, siga los pasos siguientes:

1. Vaya a la instancia de Key Vault asociada al área de trabajo.

   [![Instancia de Key Vault asociada al área de trabajo de Azure Machine Learning](./media/how-to-enable-virtual-network/workspace-key-vault.png)](./media/how-to-enable-virtual-network/workspace-key-vault.png#lightbox)

1. En la __página de Key Vault__, en el panel izquierdo, seleccione __Firewalls y redes virtuales__.

   ![Sección "Firewalls y redes virtuales" del panel de Key Vault](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks.png)

1. En la página __Firewalls y redes virtuales__, realice las acciones siguientes:
    - Haga clic en __Redes seleccionadas__ en __Permitir el acceso desde__.
    - En __Redes virtuales__, seleccione __Agregar redes virtuales existentes__ para agregar la red virtual donde reside el proceso de experimentación.
    - En __¿Quiere permitir que los servicios de confianza de Microsoft puedan omitir este firewall?__ , seleccione __Sí__.

   [![Sección "Firewalls y redes virtuales" del panel de Key Vault](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks-page.png)](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks-page.png#lightbox)


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

Para usar una máquina virtual o un clúster de Azure HDInsight en una red virtual con el área de trabajo, siga estos pasos:

1. Cree una máquina virtual o un clúster de HDInsight mediante Azure Portal o la CLI de Azure y colóquelo en una red virtual de Azure. Para más información, consulte los siguientes artículos.
    * [Creación y administración de redes virtuales de Azure para máquinas virtuales Linux](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-virtual-network)

    * [Extender HDInsight mediante una red virtual de Azure](https://docs.microsoft.com/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)

1. Para permitir que Azure Machine Learning se comunique con el puerto SSH en la máquina virtual o el clúster, configure una entrada de origen para el grupo de seguridad de red. El puerto SSH suele ser el 22. Para permitir el tráfico desde este origen, realice las acciones siguientes:

    * En la lista desplegable __Origen__, seleccione __Etiqueta de servicio__.

    * En la lista desplegable __Etiqueta de servicio de origen__ , seleccione __AzureMachineLearning__.

    * En la lista desplegable __Intervalos de puertos de origen__, seleccione __*__ .

    * En la lista desplegable __Destino__, seleccione __Cualquiera__.

    * En la lista desplegable __Intervalos de puertos de destino__, seleccione __22__.

    * En __Protocolo__, seleccione __Cualquiera__.

    * En __Acción__, seleccione __Permitir__.

   ![Reglas de entrada para realizar la experimentación en una máquina virtual o un clúster de HDInsight dentro de una red virtual](./media/how-to-enable-virtual-network/experimentation-virtual-network-inbound.png)

    Conserve las reglas de salida predeterminadas para el grupo de seguridad de red. Para más información, consulte las reglas de seguridad predeterminadas en [Grupos de seguridad](https://docs.microsoft.com/azure/virtual-network/security-overview#default-security-rules).

    Si no desea usar las reglas de salida predeterminadas, pero desea limitar el acceso de salida de la red virtual, consulte la sección [Limitación de la conectividad saliente de la red virtual](#limiting-outbound-from-vnet).

1. Conecte el clúster de HDInsight o de máquina virtual a su área de trabajo de Azure Machine Learning. Para más información, consulte [Configuración de destinos de proceso del entrenamiento del modelo](how-to-set-up-training-targets.md).


## <a name="next-steps"></a>Pasos siguientes

* [Configuración de entornos de entrenamiento](how-to-set-up-training-targets.md)
* [Configuración de puntos de conexión privados](how-to-configure-private-link.md)
* [Lugar de implementación de modelos](how-to-deploy-and-where.md)
* [Uso de TLS para proteger un servicio web con Azure Machine Learning](how-to-secure-web-service.md)