---
title: 'Uso de tareas de instancias múltiples para ejecutar aplicaciones de MPI: Azure Batch | Microsoft Docs'
description: Obtenga información sobre cómo ejecutar aplicaciones de Interfaz de paso de mensajes (MPI) con el tipo de tarea de instancias múltiples en Azure Batch.
services: batch
documentationcenter: ''
author: laurenhughes
manager: jeconnoc
editor: ''
ms.assetid: 83e34bd7-a027-4b1b-8314-759384719327
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.date: 03/13/2019
ms.author: lahugh
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7a4184fa361ba863cdaf916ef4853eda8b6e8188
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595949"
---
# <a name="use-multi-instance-tasks-to-run-message-passing-interface-mpi-applications-in-batch"></a>Uso de tareas de instancias múltiples para ejecutar aplicaciones de la Interfaz de paso de mensajes (MPI) en Batch

Las tareas de instancias múltiples le permiten ejecutar una tarea de Azure Batch en varios nodos de proceso al mismo tiempo. Estas tareas permiten escenarios de informática de alto rendimiento como las aplicaciones de interfaz de paso de mensajes (MPI) en Batch. En este artículo, aprenderá a ejecutar tareas de instancias múltiples mediante la biblioteca [.NET de Batch][api_net].

> [!NOTE]
> Aunque los ejemplos de este artículo se centran en .NET de Batch, MS-MPI y los nodos de proceso de Windows, los conceptos de tareas de múltiples instancias aquí tratados son aplicables a otras plataformas y tecnologías (Python e Intel MPI en nodos de Linux, por ejemplo).
>
>

## <a name="multi-instance-task-overview"></a>Información general de las tareas de instancias múltiples
En Batch, cada tarea se ejecuta normalmente en un solo nodo de proceso: se envían varias tareas a un trabajo y el servicio Batch programa la ejecución de cada tarea en un nodo. No obstante, si establece la **configuración de instancias múltiples** en una tarea, le estará diciendo a Batch que cree una tarea principal y varias tareas secundarias que se ejecutarán después en varios nodos.

![Información general de las tareas de instancias múltiples][1]

Al enviar una tarea con configuración de instancias múltiples a un trabajo, el servicio Batch realiza  varios pasos que son específicos de las tareas de instancias múltiples:

1. El servicio Batch crea una tarea **principal** y varias tareas **secundarias** en función de la configuración de instancias múltiples definida. El número total de tareas (principales y todas las subtareas) coincide con el número de **instancias** (nodos de proceso) especificado en la configuración de múltiples instancias.
2. Batch designa uno de los nodos de proceso como el **maestro**, y programa la tarea principal para que se ejecute en el maestro. Programa las subtareas para que se ejecuten en el resto de los nodos de proceso asignados a la tarea de múltiples instancias, una subtarea por nodo.
3. Estas tareas, tanto la principal como las subtareas, descargan los **archivos de recursos comunes** que se especifican en la configuración de múltiples instancias.
4. Cuando se han descargado los archivos de recursos comunes, la tarea principal y las subtareas ejecutan el **comando de coordinación** que se especifica en la configuración de instancias múltiples. El comando de coordinación normalmente se utiliza para preparar los nodos para ejecutar la tarea. Esto puede incluir iniciar servicios en segundo plano (como los de [Microsoft MPI][msmpi_msdn]`smpd.exe`) y comprobar que los nodos están listos para procesar los mensajes entre nodos.
5. La tarea principal ejecuta el **comando de aplicación** en el nodo maestro *después de* que la principal y todas las subtareas hayan completado correctamente el comando de coordinación. Solo la tarea principal ejecuta el comando de aplicación, que es la línea de comandos especificada de la tarea de múltiples instancias. En una solución basada en [MS-MPI][msmpi_msdn], se trata del lugar donde ejecuta la aplicación habilitada para MPI mediante `mpiexec.exe`.

> [!NOTE]
> Aunque es funcionalmente distinta, la "tarea de instancias múltiples" no es un tipo de tarea única como [StartTask][net_starttask] o [JobPreparationTask][net_jobprep]. La tarea de instancias múltiples es simplemente una tarea de Batch estándar ([CloudTask][net_task] en .NET de Batch) cuya opción de instancias múltiples se ha configurado. En este artículo, nos referiremos a ella como **tarea de instancias múltiples**.
>
>

## <a name="requirements-for-multi-instance-tasks"></a>Requisitos de las tareas de instancias múltiples
Las tareas de múltiples instancias requieren un grupo con la **comunicación ente nodos habilitada** y la **ejecución simultánea de tareas deshabilitada**. Para deshabilitar la ejecución de tareas simultáneas, establezca la propiedad [CloudPool.MaxTasksPerComputeNode](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool) en 1.

> [!NOTE]
> Batch [limita](batch-quota-limit.md#pool-size-limits) el tamaño de un grupo que tiene habilitada la comunicación entre nodos.


Este fragmento de código muestra cómo crear un grupo para tareas de varias instancias mediante la biblioteca de .NET de Batch.

```csharp
CloudPool myCloudPool =
    myBatchClient.PoolOperations.CreatePool(
        poolId: "MultiInstanceSamplePool",
        targetDedicatedComputeNodes: 3
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

// Multi-instance tasks require inter-node communication, and those nodes
// must run only one task at a time.
myCloudPool.InterComputeNodeCommunicationEnabled = true;
myCloudPool.MaxTasksPerComputeNode = 1;
```

> [!NOTE]
> Si intenta ejecutar una tarea de instancias múltiples en un grupo con la comunicación entre nodos deshabilitada o con un valor de *maxTasksPerNode* superior a 1, la tarea nunca será programada, sino que permanecerá indefinidamente en estado "activo". 


### <a name="use-a-starttask-to-install-mpi"></a>Uso de StartTask para instalar MPI
Para ejecutar aplicaciones MPI con una tarea de instancias múltiples, primero debe instalar una implementación de MPI (por ejemplo, MS-MPI o Intel MPI) en los nodos de proceso del grupo. Es un buen momento para utilizar una instancia de [StartTask][net_starttask], que se ejecuta cada vez que un nodo se une a un grupo o se ha reiniciado. En este fragmento de código se crea un objeto StartTask que especifica el paquete de instalación de MS-MPI como un [archivo de recursos][net_resourcefile]. La línea de comandos de la tarea de inicio se ejecuta una vez que el archivo de recurso se ha descargado en el nodo. En este caso, la línea de comandos realiza una instalación desatendida de MS-MPI.

```csharp
// Create a StartTask for the pool which we use for installing MS-MPI on
// the nodes as they join the pool (or when they are restarted).
StartTask startTask = new StartTask
{
    CommandLine = "cmd /c MSMpiSetup.exe -unattend -force",
    ResourceFiles = new List<ResourceFile> { new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MSMpiSetup.exe", "MSMpiSetup.exe") },
    UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.Admin)),
    WaitForSuccess = true
};
myCloudPool.StartTask = startTask;

// Commit the fully configured pool to the Batch service to actually create
// the pool and its compute nodes.
await myCloudPool.CommitAsync();
```

### <a name="remote-direct-memory-access-rdma"></a>Acceso directo a memoria remota (RDMA)
Cuando elige el [tamaño con capacidad RDMA](../virtual-machines/windows/sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) como A9 para los nodos de proceso del grupo de Batch, la aplicación de MPI puede aprovechar la red de acceso directo a memoria remota (RDMA) de alto rendimiento y baja latencia de Azure.

Consulte los tamaños compatibles con RDMA en los siguientes artículos:

* Grupos de **CloudServiceConfiguration**

  * [Tamaños de Cloud Services](../cloud-services/cloud-services-sizes-specs.md) (solo Windows)
* Grupos de **VirtualMachineConfiguration**

  * [Tamaños de las máquinas virtuales en Azure](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (Linux)
  * [Tamaños de las máquinas virtuales en Azure](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Windows)

> [!NOTE]
> Para sacar el máximo provecho a RDMA en los [nodos de proceso de Linux](batch-linux-nodes.md), debe utilizar **Intel MPI** en esos nodos. 
>

## <a name="create-a-multi-instance-task-with-batch-net"></a>Creación de una tarea de instancias múltiples con .NET de Batch
Ahora que hemos analizado los requisitos de grupo y la instalación del paquete MPI, vamos a crear la tarea de instancias múltiples. En este fragmento de código, creamos una instancia de [CloudTask][net_task] estándar y luego configuramos su propiedad [MultiInstanceSettings][net_multiinstance_prop]. Como se mencionó anteriormente, la tarea de instancias múltiples no es un tipo de tarea distinto, sino una tarea de Batch estándar configurada con la opción de instancias múltiples.

```csharp
// Create the multi-instance task. Its command line is the "application command"
// and will be executed *only* by the primary, and only after the primary and
// subtasks execute the CoordinationCommandLine.
CloudTask myMultiInstanceTask = new CloudTask(id: "mymultiinstancetask",
    commandline: "cmd /c mpiexec.exe -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe");

// Configure the task's MultiInstanceSettings. The CoordinationCommandLine will be executed by
// the primary and all subtasks.
myMultiInstanceTask.MultiInstanceSettings =
    new MultiInstanceSettings(numberOfNodes) {
    CoordinationCommandLine = @"cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d",
    CommonResourceFiles = new List<ResourceFile> {
    new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MyMPIApplication.exe",
                     "MyMPIApplication.exe")
    }
};

// Submit the task to the job. Batch will take care of splitting it into subtasks and
// scheduling them for execution on the nodes.
await myBatchClient.JobOperations.AddTaskAsync("mybatchjob", myMultiInstanceTask);
```

## <a name="primary-task-and-subtasks"></a>Tarea principal y subtareas
Cuando se crea la configuración de instancias múltiples para una tarea, se especifica el número de nodos de proceso que ejecutarán la tarea. Cuando se envía la tarea a un trabajo, el servicio Batch crea una tarea **principal** y suficientes **subtareas** que, juntas, coinciden con el número de nodos especificado.

A estas tareas se les asigna a un identificador entero del intervalo de 0 a *numberOfInstances* - 1. La tarea con el identificador 0 es la tarea principal y todos los demás identificadores son subtareas. Por ejemplo, si crea la siguiente configuración de instancias múltiples para una tarea, la tarea principal tendrá un identificador de 0 y las subtareas tendrán los identificadores del 1 al 9.

```csharp
int numberOfNodes = 10;
myMultiInstanceTask.MultiInstanceSettings = new MultiInstanceSettings(numberOfNodes);
```

### <a name="master-node"></a>Nodo maestro
Cuando se envía una tarea de múltiples instancias, el servicio Batch designa uno de los nodos de proceso como el nodo "maestro" y programa la tarea principal para que se ejecute en el nodo maestro. Las subtareas se programan para ejecutarse en el resto de los nodos asignados a la tarea de múltiples instancias.

## <a name="coordination-command"></a>comando de coordinación
El **comando de coordinación** ejecuta tanto tareas principales como subtareas.

La invocación del comando de coordinación se bloquea: el servicio Batch no ejecuta el comando de aplicación hasta que el comando de coordinación se ha devuelto correctamente para todas las subtareas. Por lo tanto, el comando de coordinación debe iniciar los servicios en segundo plano necesarios, comprobar que están listos para utilizarse y luego cerrarse. Por ejemplo, este comando de coordinación para una solución que utiliza la versión 7 de MS-MPI inicia el servicio SMPD en el nodo y luego se cierra:

```
cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d
```

Observe el uso de `start` en este comando de coordinación. Esto es necesario porque la aplicación `smpd.exe` no devuelve resultados inmediatamente después de la ejecución. Sin el uso del comando [start][cmd_start], este comando de coordinación no devolvería resultados y, por tanto, impediría que se ejecutara el comando de aplicación.

## <a name="application-command"></a>Comando de aplicación
Una vez que la tarea principal y todas las subtareas han terminado de ejecutar el comando de coordinación, *solo*la tarea principal ejecuta la línea de comandos de la tarea de instancias múltiples. Llamaremos a este el **comando de aplicación** para distinguirlo del comando de coordinación.

Para las aplicaciones de MS-MPI, use el comando de aplicación para ejecutar la aplicación habilitada para MPI con `mpiexec.exe`. Por ejemplo, este es un comando de aplicación para una solución mediante la versión 7 de MS-MPI:

```
cmd /c ""%MSMPI_BIN%\mpiexec.exe"" -c 1 -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe
```

> [!NOTE]
> Dado que `mpiexec.exe` de MS-MPI utiliza la variable `CCP_NODES` de forma predeterminada (consulte [Variables de entorno](#environment-variables)), la línea de comandos de aplicación del ejemplo anterior la excluye.
>
>

## <a name="environment-variables"></a>Variables de entorno
Batch crea varias [variables de entorno][msdn_env_var] específicas de las tareas de múltiples instancias en los nodos de proceso asignados a una tarea de múltiples instancias. Las líneas de comando de coordinación y de la aplicación pueden hacer referencia a estas variables de entorno, como los scripts y programas que se ejecutan.

Las siguientes variables de entorno las crea el servicio Batch para su uso por las tareas de múltiples instancias:

* `CCP_NODES`
* `AZ_BATCH_NODE_LIST`
* `AZ_BATCH_HOST_LIST`
* `AZ_BATCH_MASTER_NODE`
* `AZ_BATCH_TASK_SHARED_DIR`
* `AZ_BATCH_IS_CURRENT_NODE_MASTER`

Para obtener detalles completos sobre estas y las demás variables de entorno del nodo de proceso de Batch, incluido su contenido y la visibilidad, consulte [Variables de entorno del nodo de proceso][msdn_env_var].

> [!TIP]
> El ejemplo de código de Linux MPI de Batch contiene un ejemplo de cómo se pueden usar varias de estas variables de entorno. El script de Batch [coordination-cmd][coord_cmd_example] descarga archivos de aplicación y entrada comunes desde Azure Storage, permite a un recurso compartido de Network File System (NFS) en el nodo maestro y configura los demás nodos asignados a la tarea de múltiples instancias, como clientes NFS.
>
>

## <a name="resource-files"></a>Archivos de recursos
Hay dos conjuntos de archivos de recursos que se deben tener en cuenta para las tareas de múltiples instancias: **archivos de recursos comunes** que descargan *todas* las tareas (tanto principales como subtareas) y **archivos de recursos** especificados para la propia tarea de múltiples instancias, que descarga *solo la tarea principal*.

Puede especificar uno o más **archivos de recursos comunes** en la configuración de instancias múltiples de una tarea. La tarea principal y todas las subtareas descargan estos archivos de recursos comunes desde el [Azure Storage](../storage/common/storage-introduction.md) en el **directorio compartido de tareas** de cada nodo. Puede tener acceso al directorio compartido de tareas desde las líneas de comandos de coordinación y aplicación mediante la variable de entorno `AZ_BATCH_TASK_SHARED_DIR` . La ruta de acceso `AZ_BATCH_TASK_SHARED_DIR` es idéntica en todos los nodos asignados a la tarea de múltiples instancias, por lo que puede compartir un único comando de coordinación entre el principal y todas las subtareas. Batch no "comparte" el directorio en un sentido de acceso remoto, pero puede usarlo como un montaje o punto de recurso compartido, tal como se mencionó anteriormente en la información sobre las variables de entorno.

Los archivos de recursos que especifique para la propia tarea de múltiples instancias se descargan en el directorio de trabajo de la tarea, `AZ_BATCH_TASK_WORKING_DIR`, de forma predeterminada. Como se mencionó, a diferencia de los archivos de recursos comunes, solo la tarea principal descarga los archivos de recursos especificados para la propia tarea de múltiples instancias.

> [!IMPORTANT]
> Utilice siempre las variables de entorno `AZ_BATCH_TASK_SHARED_DIR` y `AZ_BATCH_TASK_WORKING_DIR` para hacer referencia a estos directorios en las líneas de comando. No intente construir las rutas de acceso manualmente.
>
>

## <a name="task-lifetime"></a>Duración de la tarea
La duración de la tarea principal controla la duración de toda la tarea de instancias múltiples. Cuando se cierra la tarea principal, todas las subtareas se terminan. El código de salida de la tarea principal es el código de salida de la tarea y, por tanto, se utiliza para determinar el éxito o fracaso de la tarea con fines de reintento.

Si se produce un error en alguna de las subtareas, por ejemplo, se cierra con un código de error distinto de cero, la tarea de instancias múltiples entera dará error. Entonces la tarea de instancias múltiples se termina y se reintenta, hasta su límite de reintento.

Cuando se elimina una tarea de instancias múltiples, el servicio Batch también elimina la tarea principal y todas las subtareas. Todos los directorios de subtarea y sus archivos se eliminan de los nodos de proceso, igual que en el caso de una tarea estándar.

Los valores de [TaskConstraints][net_taskconstraints] para una tarea de instancias múltiples, como las propiedades [MaxTaskRetryCount][net_taskconstraint_maxretry], [MaxWallClockTime][net_taskconstraint_maxwallclock] y [RetentionTime][net_taskconstraint_retention], se respetan ya que son para una tarea estándar, y se aplican a la tarea principal y a todas las subtareas. Sin embargo, si cambia la propiedad [RetentionTime][net_taskconstraint_retention] después de agregar la tarea de instancias múltiples al trabajo, este cambio solo se aplica a la tarea principal. Todas las subtareas seguirán usando la propiedad [RetentionTime][net_taskconstraint_retention] original.

La lista de tareas recientes de un nodo de proceso reflejará el identificador de una subtarea si la tarea reciente era parte de una tarea de instancias múltiples.

## <a name="obtain-information-about-subtasks"></a>Obtención de información sobre las subtareas
Para obtener información sobre las subtareas mediante la biblioteca .NET de Batch, llame al método [CloudTask.ListSubtasks][net_task_listsubtasks]. Este método devuelve información sobre todas las subtareas e información sobre el nodo de proceso que ejecuta las tareas. A partir de esta información, puede determinar el directorio raíz de cada subtarea, el identificador de grupo, su estado actual, el código de salida, etc. Esta información se puede utilizar en combinación con el método [PoolOperations.GetNodeFile][poolops_getnodefile] para obtener los archivos de la subtarea. Tenga en cuenta que este método no devuelve información de la tarea principal (id. 0).

> [!NOTE]
> A menos que se indique lo contrario, los métodos .NET de Batch que operan en la propia clase [CloudTask][net_task] de varias instancias, *solo* se aplican a la tarea principal. Por ejemplo, al llamar al método [CloudTask.ListNodeFiles][net_task_listnodefiles] en una tarea de instancias múltiples, solo se devuelven los archivos de la tarea principal.
>
>

El fragmento de código siguiente muestra cómo obtener información de la subtarea y cómo solicitar contenido de archivos de los nodos en los que se ejecuta.

```csharp
// Obtain the job and the multi-instance task from the Batch service
CloudJob boundJob = batchClient.JobOperations.GetJob("mybatchjob");
CloudTask myMultiInstanceTask = boundJob.GetTask("mymultiinstancetask");

// Now obtain the list of subtasks for the task
IPagedEnumerable<SubtaskInformation> subtasks = myMultiInstanceTask.ListSubtasks();

// Asynchronously iterate over the subtasks and print their stdout and stderr
// output if the subtask has completed
await subtasks.ForEachAsync(async (subtask) =>
{
    Console.WriteLine("subtask: {0}", subtask.Id);
    Console.WriteLine("exit code: {0}", subtask.ExitCode);

    if (subtask.State == SubtaskState.Completed)
    {
        ComputeNode node =
            await batchClient.PoolOperations.GetComputeNodeAsync(subtask.ComputeNodeInformation.PoolId,
                                                                 subtask.ComputeNodeInformation.ComputeNodeId);

        NodeFile stdOutFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\\" + Constants.StandardOutFileName);
        NodeFile stdErrFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\\" + Constants.StandardErrorFileName);
        stdOut = await stdOutFile.ReadAsStringAsync();
        stdErr = await stdErrFile.ReadAsStringAsync();

        Console.WriteLine("node: {0}:", node.Id);
        Console.WriteLine("stdout.txt: {0}", stdOut);
        Console.WriteLine("stderr.txt: {0}", stdErr);
    }
    else
    {
        Console.WriteLine("\tSubtask {0} is in state {1}", subtask.Id, subtask.State);
    }
});
```

## <a name="code-sample"></a>Código de ejemplo
En el ejemplo de código [MultiInstanceTasks][github_mpi] que se encuentra en GitHub se muestra cómo puede usarse una tarea de instancias múltiples para ejecutar una aplicación de [MS-MPI][msmpi_msdn] en los nodos de proceso de Batch. Siga los pasos que se describen en las secciones [Preparación](#preparation) y [Ejecución](#execution) para ejecutar el ejemplo.

### <a name="preparation"></a>Preparación
1. Siga los dos primeros pasos que se indican en [Compilación y ejecución de un sencillo programa de MS-MPI][msmpi_howto]. De este modo, cumplirá los requisitos previos necesarios para el siguiente paso.
2. Compile una versión de *lanzamiento* del programa de ejemplo [MPIHelloWorld][helloworld_proj] de MPI. La tarea de instancias múltiples ejecutará este programa en los nodos de proceso.
3. Cree un archivo zip que contenga `MPIHelloWorld.exe` (creado en el paso 2) y `MSMpiSetup.exe` (descargado en el paso 1). En el siguiente paso, cargará este archivo zip como paquete de aplicación.
4. Use [Azure Portal][portal] para crear una [aplicación](batch-application-packages.md) de Batch llamada "MPIHelloWorld" y establezca el archivo zip que creó en el paso anterior como versión "1.0" del paquete de aplicación. Para más información, consulte [Carga y administración de aplicaciones](batch-application-packages.md#upload-and-manage-applications).

> [!TIP]
> Cree una versión de *lanzamiento* de `MPIHelloWorld.exe` para que no tenga que incluir otras dependencias (por ejemplo, `msvcp140d.dll` o `vcruntime140d.dll`) en el paquete de aplicación.
>
>

### <a name="execution"></a>Ejecución
1. Descargue [azure-batch-samples][github_samples_zip] de GitHub.
2. Abra el MultiInstanceTasks **solución** en Visual Studio de 2019. El archivo de solución de `MultiInstanceTasks.sln` se encuentra en:

    `azure-batch-samples\CSharp\ArticleProjects\MultiInstanceTasks\`
3. En el proyecto **Microsoft.Azure.Batch.Samples.Common**, especifique las credenciales de la cuenta de Batch y Storage en `AccountSettings.settings`.
4. **Compile y ejecute** la solución MultiInstanceTasks. De este modo, ejecutará la aplicación de ejemplo de MPI en los nodos de proceso de un grupo de Batch.
5. *Opcional*: Puede usar [Azure Portal][portal] o el [Batch Explorer][batch_labs] para examinar el grupo, el trabajo y la tarea de ejemplo ("MultiInstanceSamplePool", "MultiInstanceSampleJob" y "MultiInstanceSampleTask") antes de eliminar los recursos.

> [!TIP]
> Si no tiene Visual Studio, puede descargar gratis [Visual Studio Community][visual_studio].
>
>

La salida de `MultiInstanceTasks.exe` será similar a la siguiente:

```
Creating pool [MultiInstanceSamplePool]...
Creating job [MultiInstanceSampleJob]...
Adding task [MultiInstanceSampleTask] to job [MultiInstanceSampleJob]...
Awaiting task completion, timeout in 00:30:00...

Main task [MultiInstanceSampleTask] is in state [Completed] and ran on compute node [tvm-1219235766_1-20161017t162002z]:
---- stdout.txt ----
Rank 2 received string "Hello world" from Rank 0
Rank 1 received string "Hello world" from Rank 0

---- stderr.txt ----

Main task completed, waiting 00:00:10 for subtasks to complete...

---- Subtask information ----
subtask: 1
        exit code: 0
        node: tvm-1219235766_3-20161017t162002z
        stdout.txt:
        stderr.txt:
subtask: 2
        exit code: 0
        node: tvm-1219235766_2-20161017t162002z
        stdout.txt:
        stderr.txt:

Delete job? [yes] no: yes
Delete pool? [yes] no: yes

Sample complete, hit ENTER to exit...
```

## <a name="next-steps"></a>Pasos siguientes
* En el blog Microsoft HPC & Azure Batch Team se trata sobre la [compatibilidad de MPI para Linux en Azure Batch][blog_mpi_linux], e incluye información sobre el uso de [OpenFOAM][openfoam] con Batch. Puede encontrar ejemplos de código de Python para el [ejemplo de OpenFOAM en GitHub][github_mpi].
* Aprenda a [crear grupos de nodos de proceso de Linux](batch-linux-nodes.md) para utilizarlos en soluciones de MPI con Azure Batch.

[helloworld_proj]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/MultiInstanceTasks/MPIHelloWorld

[api_net]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: https://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_labs]: https://azure.github.io/BatchExplorer/
[blog_mpi_linux]: https://blogs.technet.microsoft.com/windowshpc/2016/07/20/introducing-mpi-support-for-linux-on-azure-batch/
[cmd_start]: https://technet.microsoft.com/library/cc770297.aspx
[coord_cmd_example]: https://github.com/Azure/azure-batch-samples/blob/master/Python/Batch/article_samples/mpi/data/linux/openfoam/coordination-cmd
[github_mpi]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/MultiInstanceTasks
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_samples_zip]: https://github.com/Azure/azure-batch-samples/archive/master.zip
[msdn_env_var]: https://msdn.microsoft.com/library/azure/mt743623.aspx
[msmpi_msdn]: https://msdn.microsoft.com/library/bb524831.aspx
[msmpi_sdk]: https://go.microsoft.com/FWLink/p/?LinkID=389556
[msmpi_howto]: https://blogs.technet.com/b/windowshpc/archive/2015/02/02/how-to-compile-and-run-a-simple-ms-mpi-program.aspx
[openfoam]: http://www.openfoam.com/
[visual_studio]: https://www.visualstudio.com/vs/community/

[net_jobprep]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobpreparationtask.aspx
[net_multiinstance_class]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.aspx
[net_multiinstance_prop]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.multiinstancesettings.aspx
[net_multiinsance_commonresfiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.commonresourcefiles.aspx
[net_multiinstance_coordcmdline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.coordinationcommandline.aspx
[net_multiinstance_numinstances]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.numberofinstances.aspx
[net_pool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_pool_create]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[net_pool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx
[net_resourcefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.resourcefile.aspx
[net_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.aspx
[net_starttask_cmdline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.commandline.aspx
[net_task]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_taskconstraints]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.aspx
[net_taskconstraint_maxretry]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.maxtaskretrycount.aspx
[net_taskconstraint_maxwallclock]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.maxwallclocktime.aspx
[net_taskconstraint_retention]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.retentiontime.aspx
[net_task_listsubtasks]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listsubtasks.aspx
[net_task_listnodefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listnodefiles.aspx
[poolops_getnodefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.getnodefile.aspx

[portal]: https://portal.azure.com
[rest_multiinstance]: https://msdn.microsoft.com/library/azure/mt637905.aspx

[1]: ./media/batch-mpi/batch_mpi_01.png "Información general de instancias múltiples"
