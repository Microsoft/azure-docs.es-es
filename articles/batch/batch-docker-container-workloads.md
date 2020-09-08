---
title: Cargas de trabajo de contenedor
description: Aprenda a ejecutar y escalar aplicaciones desde imágenes de contenedor en Azure Batch. Cree un grupo de nodos de proceso que admita la ejecución de tareas de contenedor.
ms.topic: how-to
ms.date: 05/20/2020
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: a26582572302f670010f3038147687b47feef84a
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88933552"
---
# <a name="run-container-applications-on-azure-batch"></a>Ejecución de aplicaciones de contenedor en Azure Batch

Azure Batch permite ejecutar y escalar un gran número de trabajos de computación por lotes en Azure. Las tareas por lotes se pueden ejecutar directamente en las máquinas virtuales (nodos) de un grupo de Batch, pero también se puede configurar un grupo de Batch para ejecutar tareas en contenedores compatibles con Docker en los nodos. En este artículo se muestra cómo crear un grupo de nodos de ejecución que admita la ejecución de tareas de contenedor y luego ejecutar las tareas de contenedor en el grupo.

Debe estar familiarizado con los conceptos de contenedor y cómo crear un trabajo y un grupo de Batch. Los ejemplos de código usan los SDK de Python, .NET y Batch. También puede usar otras herramientas y SDK de Batch, incluido Azure Portal, para crear grupos de Batch habilitados para contenedor y para ejecutar tareas de contenedor.

## <a name="why-use-containers"></a>¿Por qué usar contenedores?

El uso de contenedores proporciona una manera sencilla de ejecutar tareas de Batch sin tener que administrar un entorno ni dependencias para ejecutar las aplicaciones. Los contenedores implementan aplicaciones como unidades ligeras, portátiles y autosuficientes que se pueden ejecutar en varios entornos distintos. Por ejemplo, cree y pruebe localmente un contenedor y, después cargue la imagen de contenedor en un registro en Azure o en otro lugar. El modelo de implementación de contenedor garantiza que el entorno en tiempo de ejecución de la aplicación siempre esté correctamente instalado y configurado donde sea que se hospede la aplicación. Las tareas basadas en contenedor en Batch también pueden aprovechar las características de tareas no en contenedor, incluidos los paquetes de aplicación y la administración de archivos de recursos y archivos de salida.

## <a name="prerequisites"></a>Prerequisites

* **Versiones del SDK**: los SDK de Batch admiten las imágenes de contenedor en las siguientes versiones:
    * API de REST de Batch versión 2017-09-01.6.0
    * SDK de .NET de Batch versión 8.0.0
    * SDK de Python de Batch versión 4.0
    * SDK de Java de Batch versión 3.0
    * SDK de Node.js de Batch versión 3.0

* **Cuentas**: en la suscripción de Azure, debe crear una cuenta de Batch y, opcionalmente, una cuenta de Azure Storage.

* **Una imagen de VM admitidas**: los contenedores solo se admiten en grupos creados con la configuración de máquinas virtuales a partir de imágenes que se detallan en la siguiente sección, "Imágenes de máquinas virtuales admitidas". Si proporciona una imagen personalizada, consulte las consideraciones en la siguiente sección y los requisitos en [Uso de una imagen personalizada administrada para crear un grupo de máquinas virtuales](batch-custom-images.md).

### <a name="limitations"></a>Limitaciones

* El servicio Batch proporciona compatibilidad con RDMA solo para contenedores que se ejecutan en grupos Linux

* Para las cargas de trabajo de contenedor de Windows, se recomienda elegir un tamaño de máquina virtual multinúcleo para el grupo

## <a name="supported-virtual-machine-images"></a>Imágenes de máquinas virtuales admitidas

Use una de las siguientes imágenes Windows o Linux compatibles para crear un grupo de nodos de ejecución de máquinas virtuales para cargas de trabajo de contenedor. Para más información sobre las imágenes de Marketplace compatibles con Batch, consulte la [lista de imágenes de máquinas virtuales](batch-linux-nodes.md#list-of-virtual-machine-images).

### <a name="windows-support"></a>Soporte técnico de Windows

Batch admite imágenes de Windows Server que tienen designaciones compatibles con contenedores. Normalmente, estos nombres de SKU de imagen tienen el sufijo `-with-containers` o `-with-containers-smalldisk`. Además, [la API para enumerar todas las imágenes admitidas en Batch](batch-linux-nodes.md#list-of-virtual-machine-images) indicará una funcionalidad `DockerCompatible` si la imagen admite contenedores Docker.

También puede crear imágenes personalizadas desde máquinas virtuales que ejecutan Docker en Windows.

### <a name="linux-support"></a>Compatibilidad de Linux

En el caso de las cargas de trabajo de contenedor Linux, Batch actualmente admite estas imágenes de Linux que Microsoft Azure Batch publica en Azure Marketplace sin necesidad de una imagen personalizada:

#### <a name="vm-sizes-without-rdma"></a>Tamaños de máquina virtual sin RDMA

- Publicador: `microsoft-azure-batch`
  - Oferta: `centos-container`
  - Oferta: `ubuntu-server-container`

#### <a name="vm-sizes-with-rdma"></a>Tamaños de máquina virtual con RDMA

- Publicador: `microsoft-azure-batch`
  - Oferta: `centos-container-rdma`
  - Oferta: `ubuntu-server-container-rdma`

Estas imágenes solo se admiten para su uso en grupos de Azure Batch y están orientadas a la ejecución de contenedores Docker. Cuentan con:

* Un entorno de ejecución de contenedores [Moby](https://github.com/moby/moby) compatible con Docker, preinstalado.

* Controladores NVIDIA GPU y entorno de ejecución de contenedores de NVIDIA, preinstalados, para agilizar la implementación en máquinas virtuales de la serie N de Azure.

* Imagen preinstalada o preconfigurada compatible con los tamaños de máquina virtual de Infiniband RDMA para las imágenes con el sufijo de `-rdma`. Actualmente estas imágenes no admiten los tamaños de máquina virtual SR-IOV IB/RDMA.

También puede crear imágenes personalizadas a partir de máquinas virtuales que ejecutan Docker en una de las distribuciones Linux compatibles con Batch. Si elige proporcionar su propia imagen personalizada de Linux, consulte las instrucciones de [Uso de una imagen personalizada administrada para crear un grupo de máquinas virtuales](batch-custom-images.md).

Para la compatibilidad de una imagen personalizada con Docker, instale [Docker Community Edition (CE)](https://www.docker.com/community-edition) o [Docker Enterprise Edition (EE)](https://www.docker.com/enterprise-edition).

Consideraciones adicionales para usar una imagen Linux personalizada:

* Para aprovechar el rendimiento de GPU de los tamaños de la serie N de Azure al usar una imagen personalizada, instale previamente los controladores NVIDIA. Además, debe instalar la utilidad de motor de Docker para GPU NVIDIA, [NVIDIA Docker](https://github.com/NVIDIA/nvidia-docker).

* Use un tamaño de máquina virtual compatible con RDAM para acceder a la red RDMA de Azure. Los controladores RDMA necesarios están instalados en HPC de CentOS e imágenes de Ubuntu compatibles con Batch. Puede ser necesaria una configuración adicional para ejecutar cargas de trabajo MPI. Consulte [Uso de instancias compatibles con RDMA o habilitadas para GPU en grupos de Batch](batch-pool-compute-intensive-sizes.md).


## <a name="container-configuration-for-batch-pool"></a>Configuración del contenedor de grupo de Batch

Para habilitar un grupo de Batch para ejecutar cargas de trabajo de contenedor, se debe especificar la opción [ContainerConfiguration](/dotnet/api/microsoft.azure.batch.containerconfiguration) en el objeto [VirtualMachineConfiguration](/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration) del grupo. (Este artículo incluye vínculos a la referencia de la API de .NET de Batch. Puede encontrar la configuración correspondiente en la API [Batch Python](/python/api/overview/azure/batch)).

Puede crear un grupo habilitado para contenedores con o sin imágenes de contenedor capturadas previamente, tal como se muestra en los ejemplos siguientes. El proceso de extracción (o captura previa) permite cargar previamente imágenes de contenedor desde Docker Hub u otro registro de contenedor en Internet. Para obtener el mejor rendimiento, use un [registro de contenedor de Azure](../container-registry/container-registry-intro.md) en la misma región que la cuenta de Batch.

La ventaja de capturar previamente las imágenes de contenedor es que, cuando las tareas comienzan a ejecutarse por primera vez, no tienen que esperar a que la imagen del contenedor se descargue. La configuración de contenedor extrae las imágenes de contenedor de las máquinas virtuales cuando se crea el grupo. Las tareas que se ejecutan en el grupo pueden entonces hacer referencia a la lista de las imágenes de contenedor y opciones de ejecución de contenedor.


### <a name="pool-without-prefetched-container-images"></a>Grupo sin imágenes de contenedor previamente capturadas

Para configurar el grupo habilitado para contenedores sin imágenes de contenedor previamente capturadas, defina los objetos `ContainerConfiguration` y `VirtualMachineConfiguration` como se muestra en el ejemplo de Python siguiente. En este ejemplo se usa la imagen de Ubuntu Server para grupos de contenedores de Azure Batch de Marketplace.


```python
image_ref_to_use = batch.models.ImageReference(
    publisher='microsoft-azure-batch',
    offer='ubuntu-server-container',
    sku='16-04-lts',
    version='latest')

"""
Specify container configuration. This is required even though there are no prefetched images.
"""

container_conf = batch.models.ContainerConfiguration()

new_pool = batch.models.PoolAddParameter(
    id=pool_id,
    virtual_machine_configuration=batch.models.VirtualMachineConfiguration(
        image_reference=image_ref_to_use,
        container_configuration=container_conf,
        node_agent_sku_id='batch.node.ubuntu 16.04'),
    vm_size='STANDARD_D1_V2',
    target_dedicated_nodes=1)
...
```


### <a name="prefetch-images-for-container-configuration"></a>Captura previa de imágenes para la configuración de contenedor

Para la captura previa de imágenes de contenedor en el grupo, agregue la lista de imágenes de contenedor (`container_image_names`, en Python) a `ContainerConfiguration`.

En el siguiente ejemplo básico de Python se muestra cómo capturar previamente una imagen de contenedor Ubuntu estándar desde [Docker Hub](https://hub.docker.com).

```python
image_ref_to_use = batch.models.ImageReference(
    publisher='microsoft-azure-batch',
    offer='ubuntu-server-container',
    sku='16-04-lts',
    version='latest')

"""
Specify container configuration, fetching the official Ubuntu container image from Docker Hub.
"""

container_conf = batch.models.ContainerConfiguration(
    container_image_names=['ubuntu'])

new_pool = batch.models.PoolAddParameter(
    id=pool_id,
    virtual_machine_configuration=batch.models.VirtualMachineConfiguration(
        image_reference=image_ref_to_use,
        container_configuration=container_conf,
        node_agent_sku_id='batch.node.ubuntu 16.04'),
    vm_size='STANDARD_D1_V2',
    target_dedicated_nodes=1)
...
```


En el siguiente ejemplo de C# se da por hecho que quiere capturar previamente una imagen de TensorFlow desde [Docker Hub](https://hub.docker.com). Este ejemplo contiene una tarea de inicio que se ejecuta en el host de máquina virtual de los nodos de grupo. Podría ejecutar una tarea de inicio en el host, por ejemplo, para montar un servidor de archivos al que pueda accederse desde los contenedores.

```csharp
ImageReference imageReference = new ImageReference(
    publisher: "microsoft-azure-batch",
    offer: "ubuntu-server-container",
    sku: "16-04-lts",
    version: "latest");

ContainerRegistry containerRegistry = new ContainerRegistry(
    registryServer: "https://hub.docker.com",
    userName: "UserName",
    password: "YourPassword"                
);

// Specify container configuration, prefetching Docker images
ContainerConfiguration containerConfig = new ContainerConfiguration();
containerConfig.ContainerImageNames = new List<string> { "tensorflow/tensorflow:latest-gpu" };
containerConfig.ContainerRegistries = new List<ContainerRegistry> { containerRegistry };

// VM configuration
VirtualMachineConfiguration virtualMachineConfiguration = new VirtualMachineConfiguration(
    imageReference: imageReference,
    nodeAgentSkuId: "batch.node.ubuntu 16.04");
virtualMachineConfiguration.ContainerConfiguration = containerConfig;

// Set a native host command line start task
StartTask startTaskContainer = new StartTask( commandLine: "<native-host-command-line>" );

// Create pool
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    virtualMachineSize: "Standard_NC6",
    virtualMachineConfiguration: virtualMachineConfiguration);

// Start the task in the pool
pool.StartTask = startTaskContainer;
...
```


### <a name="prefetch-images-from-a-private-container-registry"></a>Captura previa de imágenes desde un registro de contenedor privado

También puede capturar previamente imágenes de contenedor mediante la autenticación a un servidor de registro de contenedor privado. En el ejemplo siguiente, los objetos `ContainerConfiguration` y `VirtualMachineConfiguration` capturan previamente una imagen privada de TensorFlow de un registro de contenedor de Azure privado. La referencia de imagen es la misma que en el ejemplo anterior.

```csharp
// Specify a container registry
ContainerRegistry containerRegistry = new ContainerRegistry(
    registryServer: "myContainerRegistry.azurecr.io",
    userName: "myUserName",
    password: "myPassword");

// Create container configuration, prefetching Docker images from the container registry
ContainerConfiguration containerConfig = new ContainerConfiguration();
containerConfig.ContainerImageNames = new List<string> {
        "myContainerRegistry.azurecr.io/tensorflow/tensorflow:latest-gpu" };
containerConfig.ContainerRegistries = new List<ContainerRegistry> { containerRegistry } );

// VM configuration
VirtualMachineConfiguration virtualMachineConfiguration = new VirtualMachineConfiguration(
    imageReference: imageReference,
    nodeAgentSkuId: "batch.node.ubuntu 16.04");
virtualMachineConfiguration.ContainerConfiguration = containerConfig;

// Create pool
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 4,
    virtualMachineSize: "Standard_NC6",
    virtualMachineConfiguration: virtualMachineConfiguration);
...
```

## <a name="container-settings-for-the-task"></a>Configuración de contenedor para la tarea

Para ejecutar una tarea de contenedor en un grupo habilitado para contenedores, indique opciones específicas de contenedor. La configuración incluye la imagen que se usará, el registro y las opciones de ejecución del contenedor.

* Use la propiedad `ContainerSettings` de las clases de tarea para configurar opciones específicas de contenedor. Esta configuración se define por la clase [TaskContainerSettings](/dotnet/api/microsoft.azure.batch.taskcontainersettings). Tenga en cuenta que la opción de contenedor `--rm` no requiere una opción `--runtime` adicional, ya que de esto se encarga Batch.

* Si ejecuta tareas en imágenes de contenedor, la [tarea en la nube](/dotnet/api/microsoft.azure.batch.cloudtask) y la [tarea del administrador de trabajos](/dotnet/api/microsoft.azure.batch.cloudjob.jobmanagertask) requieren la configuración del contenedor. Sin embargo, la [tarea de inicio](/dotnet/api/microsoft.azure.batch.starttask), la [tarea de preparación de trabajos](/dotnet/api/microsoft.azure.batch.cloudjob.jobpreparationtask) y la [tarea de liberación de trabajos](/dotnet/api/microsoft.azure.batch.cloudjob.jobreleasetask) no requieren configuración del contenedor (es decir, pueden ejecutarse en un contexto de contenedor o directamente en el nodo).

### <a name="container-task-command-line"></a>Línea de comandos de tareas de contenedor

Al ejecutar una tarea de contenedor, Batch usa automáticamente el comando [docker create](https://docs.docker.com/engine/reference/commandline/create/) para crear un contenedor con la imagen especificada en la tarea. Batch luego controla la ejecución de tareas en el contenedor.

Al igual que con las tareas de Batch sin contenedor, establece una línea de comandos para una tarea de contenedor. Dado que Batch crea automáticamente el contenedor, la línea de comandos especifica solo el o los comandos que se ejecutarán en el contenedor.

Si la imagen de contenedor para una tarea de Batch se configura con un script [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#exec-form-entrypoint-example), puede establecer la línea de comandos para que use el script ENTRYPOINT predeterminado o lo reemplace:

* Para usar el script ENTRYPOINT predeterminado de la imagen de contenedor, establezca la línea de comandos de la tarea en la cadena vacía `""`.

* Para reemplazar el script ENTRYPOINT predeterminado, o si la imagen no tiene un script ENTRYPOINT, establezca una línea de comandos adecuada para el contenedor, por ejemplo, `/app/myapp` o `/bin/sh -c python myscript.py`.

Los elementos opcionales [ContainerRunOptions](/dotnet/api/microsoft.azure.batch.taskcontainersettings.containerrunoptions) son argumentos adicionales que proporciona al comando `docker create` que usa Batch para crear y ejecutar el contenedor. Por ejemplo, para establecer un directorio de trabajo para el contenedor, establezca la opción `--workdir <directory>`. Consulte la referencia de [docker create](https://docs.docker.com/engine/reference/commandline/create/) para ver opciones adicionales.

### <a name="container-task-working-directory"></a>Directorio de trabajo de tareas de contenedor

Una tarea de contenedor de Batch se ejecuta en un directorio de trabajo del contenedor muy similar al directorio que Batch configura para una tarea normal (que no es de tipo contenedor). Tenga en cuenta que este directorio de trabajo es diferente de la instrucción [WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir) si se ha configurado en la imagen, o el directorio de trabajo de contenedor predeterminado (`C:\` en un contenedor de Windows, o `/` en un contenedor de Linux).

Para una tarea de contenedor de Batch:

* Todos los directorios incluidos de forma recursiva debajo de `AZ_BATCH_NODE_ROOT_DIR` en el nodo host (la raíz de los directorios de Azure Batch) se asignan al contenedor.
* Todas las variables de entorno de tareas se asignan al contenedor.
* El directorio de trabajo `AZ_BATCH_TASK_WORKING_DIR` de la tarea en el nodo se configura igual que para una tarea normal y se asigna al contenedor.

Estas asignaciones le permiten trabajar con tareas de contenedor de manera muy similar a las tareas sin contenedores. Por ejemplo, instale aplicaciones mediante paquetes de aplicación, acceda a los archivos de recursos desde Azure Storage, use la configuración del entorno de la tarea, y conserve los archivos de salida de la tarea una vez que se detenga el contenedor.

### <a name="troubleshoot-container-tasks"></a>Solucionar problemas de tareas de contenedor

Si la tarea de contenedor no se ejecuta según lo previsto, puede que tenga que obtener información sobre la configuración de WORKDIR o ENTRYPOINT de la imagen de contenedor. Para ver la configuración, ejecute el comando [docker image inspect](https://docs.docker.com/engine/reference/commandline/image_inspect/).

Si es necesario, ajuste la configuración de la tarea de contenedor según la imagen:

* Especifique una ruta de acceso absoluta en la línea de comandos de la tarea. Si se usa el script ENTRYPOINT predeterminado de la imagen para la línea de comandos de la tarea, asegúrese de haber establecido una ruta de acceso absoluta.

* En las opciones de ejecución del contenedor de la tarea, cambie el directorio de trabajo para que coincida con el WORKDIR en la imagen. Por ejemplo, establezca `--workdir /app`.

## <a name="container-task-examples"></a>Ejemplos de la tarea de contenedor

En el siguiente fragmento de código de Python se muestra una línea de comandos básica que se ejecuta en un contenedor creado a partir de una imagen ficticia extraída de Docker Hub. En este caso, la opción del contenedor `--rm` quita el contenedor después de que finaliza la tarea, y la opción `--workdir` establece un directorio de trabajo. La línea de comandos reemplaza el script ENTRYPOINT del contenedor con un comando de shell sencillo que escribe un archivo pequeño en el directorio de trabajo de la tarea en el host.

```python
task_id = 'sampletask'
task_container_settings = batch.models.TaskContainerSettings(
    image_name='myimage',
    container_run_options='--rm --workdir /')
task = batch.models.TaskAddParameter(
    id=task_id,
    command_line='/bin/sh -c \"echo \'hello world\' > $AZ_BATCH_TASK_WORKING_DIR/output.txt\"',
    container_settings=task_container_settings
)
```

En el siguiente ejemplo de C# muestra la configuración de contenedor básica para una tarea en la nube:

```csharp
// Simple container task command
string cmdLine = "c:\\app\\myApp.exe";

TaskContainerSettings cmdContainerSettings = new TaskContainerSettings (
    imageName: "myimage",
    containerRunOptions: "--rm --workdir c:\\app"
    );

CloudTask containerTask = new CloudTask (
    id: "Task1",
    commandline: cmdLine);
containerTask.ContainerSettings = cmdContainerSettings;
```

## <a name="next-steps"></a>Pasos siguientes

* Consulte también el kit de herramientas [Batch Shipyard](https://github.com/Azure/batch-shipyard) para facilitar la implementación de las cargas de trabajo del elemento contenedor en Azure Batch a través de la receta [Shipyard](https://github.com/Azure/batch-shipyard/tree/master/recipes).

* Para más información sobre cómo instalar y usar Docker CE en Linux, consulte la documentación de [Docker](https://docs.docker.com/engine/installation/).

* Para más información sobre el uso de imágenes personalizadas, consulte [Uso de una imagen personalizada administrada para crear un grupo de máquinas virtuales](batch-custom-images.md).

* Más información sobre el [proyecto Moby](https://mobyproject.org/), un marco para crear sistemas basados en contenedor.
