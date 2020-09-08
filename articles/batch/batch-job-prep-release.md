---
title: Creación de tareas para preparar trabajos y completarlos en nodos de proceso
description: Utilice las tareas de preparación en el nivel de trabajo para minimizar la transferencia de datos a los nodos de proceso de Azure Batch y las tareas de liberación para la limpieza del nodo tras la finalización del trabajo.
ms.topic: how-to
ms.date: 02/17/2020
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 5b1084cfdd5995b7983badcdce71460f7bdec3d5
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919461"
---
# <a name="run-job-preparation-and-job-release-tasks-on-batch-compute-nodes"></a>Ejecución de tareas de preparación y liberación de trabajos en nodos de proceso de Batch

 A menudo, Azure Batch requiere algún tipo de configuración antes de ejecutar sus tareas y un mantenimiento posterior al trabajo una vez completadas sus tareas. Puede ser necesario descargar los datos de entrada de tareas comunes en los nodos de proceso, o bien cargar datos de salida de tareas en Azure Storage una vez completado el trabajo. Puede usar las tareas de **preparación del trabajo** y **liberación del trabajo** para realizar estas operaciones.

## <a name="what-are-job-preparation-and-release-tasks"></a>Tareas de preparación y liberación de trabajos
Antes de ejecutar las tareas de un trabajo, la tarea de preparación del trabajo se ejecuta en todos los nodos de proceso programados para ejecutar al menos una tarea. Cuando el trabajo se ha completado, la tarea de liberación del trabajo se ejecuta en cada nodo del grupo que ejecutó al menos una tarea. Al igual que con las tareas normales de Batch, puede especificar una línea de comandos para que se invoque cuando se ejecute una tarea de preparación o liberación.

Las tareas de preparación y liberación ofrecen características de tareas de Batch familiares, como la descarga de archivos ([archivos de recursos][net_job_prep_resourcefiles]), la ejecución con elevación de privilegios, las variables de entorno personalizadas, la duración máxima de ejecución, el número de reintentos y el tiempo de retención de archivos.

En las siguientes secciones, aprenderá cómo utilizar las clases [JobPreparationTask][net_job_prep] y [JobReleaseTask][net_job_release] que se encuentran en la biblioteca de [Batch para .NET][api_net].

> [!TIP]
> Las tareas de preparación y liberación de trabajos son especialmente útiles en entornos de "grupo compartido", en los que un grupo de nodos de proceso persiste entre ejecuciones de trabajo y muchos trabajos lo usan.
> 
> 

## <a name="when-to-use-job-preparation-and-release-tasks"></a>Uso de las tareas de preparación y liberación de trabajos
Las tareas de preparación y liberación de trabajos son una buena opción en las situaciones siguientes:

**Descarga de datos de tareas comunes**

A menudo, los trabajos de Batch requieren un conjunto común de datos como entrada para las tareas del trabajo. Por ejemplo, en cálculos de análisis de riesgos diarios, los datos de mercado son específicos del trabajo, pero comunes a todas las tareas incluidas en él. Estos datos de mercado, a menudo con un tamaño de varios gigabytes, deben descargarse en cada nodo de proceso una sola vez, para que cualquier tarea que se ejecuta en el nodo pueda usarlos. Puede usar una **tarea de preparación del trabajo** para descargar estos datos en cada nodo antes de la ejecución de otras tareas del trabajo.

**Trabajo persistente y resultado de la tarea**

En un entorno de "grupo compartido", cuando no se dan de baja los nodos de proceso de un grupo entre los trabajos, puede ser necesario eliminar datos del trabajo entre ejecuciones. Puede ser necesario conservar espacio en disco en los nodos o cumplir las directivas de seguridad de su organización. Use una **tarea de liberación del trabajo** para eliminar los datos descargados por una tarea de preparación del trabajo o generados durante la ejecución de la tarea.

**Retención de registro**

Puede que desee conservar una copia de los archivos de registro generados por las tareas o quizás los archivos de volcado de memoria generados por aplicaciones con errores. Puede usar una **tarea de liberación del trabajo** en estos casos para comprimir y cargar estos datos en una cuenta de [Azure Storage][azure_storage].

> [!TIP]
> Otra manera de conservar los registros y otros datos de salida de los trabajos y las tareas es usar la biblioteca [Azure Batch File Conventions](batch-task-output.md) (Convenciones de archivos de Azure Batch).
>
>

## <a name="job-preparation-task"></a>tarea de preparación del trabajo


Antes de ejecutar las tareas de un trabajo, Batch ejecuta la tarea de preparación del trabajo en cada nodo de proceso programado para ejecutar una tarea. De forma predeterminada, Batch esperará hasta que la tarea de preparación del trabajo se complete antes de ejecutar las tareas programadas para ejecutarse en el nodo. Sin embargo, puede configurar el servicio para que no espere. Si se reinicia el nodo, la tarea de preparación del trabajo se ejecuta de nuevo. También puede deshabilitar este comportamiento. Si tiene un trabajo con una tarea de preparación del trabajo y una tarea del administrador de trabajos configurada, la tarea de preparación del trabajo se ejecuta antes que la tarea del administrador de trabajos, al igual que en todas las demás tareas. La tarea de preparación del trabajo siempre se ejecuta primero.

La tarea de preparación del trabajo solo se ejecuta en los nodos programados para ejecutar una tarea. Esto impide la ejecución innecesaria de una tarea de preparación en caso de que un nodo no tenga una tarea asignada. Esto puede ocurrir cuando el número de tareas de un trabajo es menor que el número de nodos de un grupo. También se aplica cuando la [ejecución de tareas simultáneas](batch-parallel-node-tasks.md) está habilitada, lo que deja algunos nodos inactivos si el número de tareas es menor que el total de posibles tareas simultáneas. Si no ejecuta la tarea de preparación del trabajo en nodos inactivos, puede ahorrar dinero en gastos de transferencia de datos.

> [!NOTE]
> [JobPreparationTask][net_job_prep_cloudjob] difiere de [CloudPool.StartTask][pool_starttask] en que JobPreparationTask se ejecuta al principio de cada trabajo, mientras que StartTask solo se ejecuta cuando un nodo de proceso se une por primera vez a un grupo o se reinicia.
>


>## <a name="job-release-task"></a>tarea de liberación del trabajo

Cuando un trabajo se marca como completado, se ejecuta la tarea de liberación del trabajo en cada nodo del grupo que ejecutó al menos una tarea. El trabajo se marca como completado mediante la emisión de una solicitud de finalización. A continuación, el servicio Batch establece el estado del trabajo en *terminando*, finaliza las tareas activas o en ejecución asociadas al trabajo y ejecuta la tarea de liberación del trabajo. A continuación, el trabajo se mueve al estado *completado* .

> [!NOTE]
> La eliminación de un trabajo también ejecuta la tarea de liberación del trabajo. Sin embargo, si un trabajo ya se ha terminado, la tarea no se ejecuta una segunda vez si el trabajo se va a eliminar más adelante.

Las tareas de liberación de trabajos pueden ejecutarse durante un máximo de 15 minutos antes de que el servicio Batch las finalice. Para más información, consulte la [Documentación de referencia de API de REST](/rest/api/batchservice/job/add#jobreleasetask).
> 
> 

## <a name="job-prep-and-release-tasks-with-batch-net"></a>Tareas de preparación y liberación de trabajos con Batch para .NET
Para usar una tarea de preparación del trabajo, asigne un objeto [JobPreparationTask][net_job_prep] a la propiedad [CloudJob.JobPreparationTask][net_job_prep_cloudjob] del trabajo. De forma similar, para establecer la tarea de liberación del trabajo, inicialice una [JobReleaseTask][net_job_release] y asígnela a la propiedad [CloudJob.JobReleaseTask][net_job_prep_cloudjob] del trabajo.

En este fragmento de código, `myBatchClient` es una instancia de [BatchClient][net_batch_client] y `myPool` es un grupo existente en la cuenta de Batch.

```csharp
// Create the CloudJob for CloudPool "myPool"
CloudJob myJob =
    myBatchClient.JobOperations.CreateJob(
        "JobPrepReleaseSampleJob",
        new PoolInformation() { PoolId = "myPool" });

// Specify the command lines for the job preparation and release tasks
string jobPrepCmdLine =
    "cmd /c echo %AZ_BATCH_NODE_ID% > %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";
string jobReleaseCmdLine =
    "cmd /c del %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";

// Assign the job preparation task to the job
myJob.JobPreparationTask =
    new JobPreparationTask { CommandLine = jobPrepCmdLine };

// Assign the job release task to the job
myJob.JobReleaseTask =
    new JobReleaseTask { CommandLine = jobReleaseCmdLine };

await myJob.CommitAsync();
```

Como se mencionó antes, la tarea de liberación se ejecuta cuando se finaliza o se elimina un trabajo. Finalice un trabajo con [JobOperations.TerminateJobAsync][net_job_terminate]. Elimine un trabajo con [JobOperations.DeleteJobAsync][net_job_delete]. Normalmente un trabajo se termina o elimina cuando se han completado sus tareas o cuando se ha alcanzado el tiempo de espera que haya definido.

```csharp
// Terminate the job to mark it as Completed; this will initiate the
// Job Release Task on any node that executed job tasks. Note that the
// Job Release Task is also executed when a job is deleted, thus you
// need not call Terminate if you typically delete jobs after task completion.
await myBatchClient.JobOperations.TerminateJobAsync("JobPrepReleaseSampleJob");
```

## <a name="code-sample-on-github"></a>Código de ejemplo en GitHub
Para ver cómo funcionan las tareas de preparación y liberación del trabajo, consulte el proyecto de ejemplo [JobPrepRelease][job_prep_release_sample] en GitHub. Esta aplicación de consola hace lo siguiente:

1. Crea un grupo con dos nodos.
2. Crea un trabajo con las tareas de preparación y de liberación del trabajo, además de las estándar.
3. Ejecuta la tarea de preparación del trabajo, que en primer lugar escribe el identificador de nodo en un archivo de texto en el directorio "shared" de un nodo.
4. Ejecuta una tarea en cada nodo que escribe su identificador de tarea en el mismo archivo de texto.
5. Una vez que se han completado todas las tareas (o se alcanza el tiempo de espera), imprime el contenido del archivo de texto de cada nodo en la consola.
6. Cuando se ha completado el trabajo, ejecuta la tarea de liberación del trabajo para eliminar el archivo del nodo.
7. Imprime los códigos de salida de las tareas de preparación y liberación del trabajo para cada nodo donde se ejecutaron.
8. Pausa la ejecución para permitir la confirmación de la eliminación del trabajo o el grupo.

La salida de la aplicación de ejemplo es similar a la siguiente:

```
Attempting to create pool: JobPrepReleaseSamplePool
Created pool JobPrepReleaseSamplePool with 2 nodes
Checking for existing job JobPrepReleaseSampleJob...
Job JobPrepReleaseSampleJob not found, creating...
Submitting tasks and awaiting completion...
All tasks completed.

Contents of shared\job_prep_and_release.txt on tvm-2434664350_1-20160623t173951z:
-------------------------------------------
tvm-2434664350_1-20160623t173951z tasks:
  task001
  task004
  task005
  task006

Contents of shared\job_prep_and_release.txt on tvm-2434664350_2-20160623t173951z:
-------------------------------------------
tvm-2434664350_2-20160623t173951z tasks:
  task008
  task002
  task003
  task007

Waiting for job JobPrepReleaseSampleJob to reach state Completed
...

tvm-2434664350_1-20160623t173951z:
  Prep task exit code:    0
  Release task exit code: 0

tvm-2434664350_2-20160623t173951z:
  Prep task exit code:    0
  Release task exit code: 0

Delete job? [yes] no
yes
Delete pool? [yes] no
yes

Sample complete, hit ENTER to exit...
```

> [!NOTE]
> Debido a la creación de variables y a la hora de inicio de los nodos en un nuevo grupo (algunos nodos están listos para las tareas antes que otros), puede que la salida sea diferente. En concreto, como las tareas se realizan rápidamente, uno de los nodos del grupo podría ejecutar todas las tareas del trabajo. Si esto sucede, observará que las tareas de preparación y liberación del trabajo no existen para el nodo que no ha ejecutado ninguna tarea.
> 
> 

### <a name="inspect-job-preparation-and-release-tasks-in-the-azure-portal"></a>Inspección de las tareas de preparación y liberación en Azure Portal
Cuando ejecute la aplicación de ejemplo, puede usar [Azure Portal][portal] para ver las propiedades del trabajo y sus tareas, o incluso descargar el archivo de texto compartido modificado por las tareas del trabajo.

La captura de pantalla siguiente muestra la **hoja de tareas de preparación** en Azure Portal después de una ejecución de la aplicación de ejemplo. Vaya a las propiedades *JobPrepReleaseSampleJob* después de que sus tareas se hayan completado (pero antes de eliminar el trabajo y el grupo) y haga clic en **Tareas de preparación** o **Tareas de liberación** para ver sus propiedades.

![Propiedades de preparación del trabajo en el Portal de Azure][1]

## <a name="next-steps"></a>Pasos siguientes
### <a name="application-packages"></a>paquetes de aplicación
Además de la tarea de preparación del trabajo, puede usar la característica de [paquetes de aplicación](batch-application-packages.md) de Batch para preparar los nodos de proceso de cara a la ejecución de tareas. Esta característica es especialmente útil para implementar aplicaciones que no requieren que se ejecute un instalador, aplicaciones que contienen muchos archivos (más de 100) o aplicaciones que requieren un control estricto de la versión.

### <a name="installing-applications-and-staging-data"></a>Instalación de aplicaciones y datos de ensayo
Este foro de MSDN proporciona información general de varios métodos de preparación de los nodos para la ejecución de tareas:

[Instalación de aplicaciones y datos de ensayo en nodos de proceso de Batch][forum_post]

Escrito por uno de los miembros del equipo de Azure Batch, describe varias técnicas que puede utilizar para implementar aplicaciones y datos en los nodos de proceso.

[api_net]: /dotnet/api/microsoft.azure.batch
[api_net_listjobs]: /dotnet/api/microsoft.azure.batch.joboperations
[api_rest]: /rest/api/batchservice/
[azure_storage]: https://azure.microsoft.com/services/storage/
[portal]: https://portal.azure.com
[job_prep_release_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/JobPrepRelease
[forum_post]: https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[net_batch_client]: /dotnet/api/microsoft.azure.batch.batchclient
[net_cloudjob]:https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_job_prep]: /dotnet/api/microsoft.azure.batch.jobpreparationtask
[net_job_prep_cloudjob]: /dotnet/api/microsoft.azure.batch.cloudjob
[net_job_prep_resourcefiles]: /dotnet/api/microsoft.azure.batch.jobpreparationtask
[net_job_delete]: /previous-versions/azure/mt281411(v=azure.100)
[net_job_terminate]: /previous-versions/azure/mt188985(v=azure.100)
[net_job_release]: /dotnet/api/microsoft.azure.batch.jobreleasetask
[net_job_release_cloudjob]: /dotnet/api/microsoft.azure.batch.cloudjob
[pool_starttask]: /dotnet/api/microsoft.azure.batch.cloudpool

[net_list_certs]: /dotnet/api/microsoft.azure.batch.certificateoperations
[net_list_compute_nodes]: /dotnet/api/microsoft.azure.batch.pooloperations
[net_list_job_schedules]: /dotnet/api/microsoft.azure.batch.jobscheduleoperations
[net_list_jobprep_status]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_jobs]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_nodefiles]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_pools]: /dotnet/api/microsoft.azure.batch.pooloperations
[net_list_schedule_jobs]: /dotnet/api/microsoft.azure.batch.jobscheduleoperations
[net_list_task_files]: /dotnet/api/microsoft.azure.batch.cloudtask
[net_list_tasks]: /dotnet/api/microsoft.azure.batch.joboperations

[1]: ./media/batch-job-prep-release/portal-jobprep-01.png
