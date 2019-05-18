---
title: 'Guardar salidas de trabajos y tareas en Azure Storage con la biblioteca de convenciones de archivo para .NET: Azure Batch | Microsoft Docs'
description: Aprenda a usar la biblioteca de convenciones de archivo para .NET de Azure Batch para guardar las salidas de trabajos y tareas en Azure Storage, y ver las salidas guardadas en Azure Portal.
services: batch
documentationcenter: .net
author: laurenhughes
manager: jeconnoc
editor: ''
ms.assetid: 16e12d0e-958c-46c2-a6b8-7843835d830e
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 11/14/2018
ms.author: lahugh
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d4e16ca40e8026861d492f950396d6aa39d4c445
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65791555"
---
# <a name="persist-job-and-task-data-to-azure-storage-with-the-batch-file-conventions-library-for-net"></a>Guardar datos de trabajos y tareas en Azure Storage con la biblioteca de convenciones de archivo para .NET

[!INCLUDE [batch-task-output-include](../../includes/batch-task-output-include.md)]

Una manera de guardar los datos de tareas consiste en usar la [biblioteca de convenciones de archivo para .NET de Azure Batch][nuget_package]. La biblioteca de convenciones de archivo simplifica el proceso de almacenamiento de los datos de salida de tareas en Azure Storage y su recuperación posterior. Puede usarla en el código de tarea y en el de cliente.&mdash;En el código de tarea, para almacenar los archivos y en el código de cliente, para mostrarlos y recuperarlos. El código de la tarea también puede usar la biblioteca para recuperar las salidas de las tareas precedentes de la cadena, como en un escenario con [dependencias entre tareas](batch-task-dependencies.md).

Para recuperar los archivos de salida con la biblioteca de convenciones de archivo, puede buscar los archivos de un trabajo o tarea determinados enumerándolos por identificador y finalidad. No es necesario conocer los nombres o ubicaciones de los archivos. Por ejemplo, puede usar la biblioteca de convenciones de archivo para mostrar todos los archivos intermedios de una tarea determinada u obtener un archivo de vista previa de un trabajo determinado.

> [!TIP]
> Empezando por la versión del 1 de mayo de 2017, la API del servicio Batch permite guardar datos de salida en Azure Storage para tareas simples y tareas de administrador de trabajos que se ejecutan en los grupos creados con la configuración de máquina virtual. La API del servicio Batch proporciona una manera sencilla para guardar la salida desde dentro del código que permite crear una tarea y que actúa como una alternativa a la biblioteca de convenciones de archivo. Puede modificar las aplicaciones de cliente de Batch para guardar la salida sin necesidad de actualizar la aplicación que la tarea está ejecutando. Para más información, consulte [Almacenamiento de datos de tareas en Azure Storage con la API del servicio Batch](batch-task-output-files.md).

## <a name="when-do-i-use-the-file-conventions-library-to-persist-task-output"></a>¿Cuándo se puede usar la biblioteca de convenciones de archivo para guardar salidas de tareas?

Azure Batch proporciona más de una manera de guardar las salidas de tareas. Las convenciones de archivo se adaptan mejor a estos escenarios:

- Puede modificar fácilmente el código de la aplicación que la tarea está ejecutando para guardar los archivos mediante la biblioteca de convenciones de archivo.
- Quiere transmitir datos a Azure Storage mientras la tarea está aún en funcionamiento.
- Quiere almacenar los datos de grupos creados con la configuración de servicios en la nube o la configuración de máquina virtual.
- La aplicación cliente u otras tareas del trabajo deben encontrar y descargar los archivos de salida de tarea por identificador o finalidad.
- Quiere ver la salida de tarea en Azure Portal.

Si su escenario es diferente de los mencionados anteriormente, considere la adopción de un enfoque diferente. Para más información sobre otras opciones para guardar la salida de tareas, consulte [Guardar salidas de trabajos y tareas en Azure Storage](batch-task-output.md).

## <a name="what-is-the-batch-file-conventions-standard"></a>¿Qué es el estándar de convenciones de archivo de Batch?

El [estándar de convenciones de archivo de Batch](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions) proporciona un esquema de nombres para los contenedores de destino y las rutas de acceso de blob en los que se escriben los archivos de salida. Los archivos guardados en Azure Storage que cumplen el estándar de convenciones de archivo están automáticamente disponibles para su visualización en Azure Portal. El portal es compatible con la convención de nomenclatura y, por eso, puede mostrar los archivos que lo cumplen.

La biblioteca de convenciones de archivo para .NET asigna automáticamente nombres a los contenedores de almacenamiento y a los archivos de salidas de tareas según el estándar de convenciones de archivo. La biblioteca de convenciones de archivo también proporciona métodos para consultar los archivos de salida en Azure Storage según el identificador de trabajo, de tarea o la finalidad.

Si va a desarrollar con un lenguaje distinto de. NET, puede implementar el estándar de convenciones de archivo en su aplicación. Para más información, consulte [Implementar el estándar Batch File Conventions](batch-task-output.md#implement-the-batch-file-conventions-standard).

## <a name="link-an-azure-storage-account-to-your-batch-account"></a>Vinculación de una cuenta de Azure Storage a la cuenta de Batch

Para guardar los datos de salida en Azure Storage mediante la biblioteca de convenciones de archivo y poder verlos en Azure Portal, debe vincular primero una cuenta de Azure Storage a su cuenta de Batch. Si aún no lo ha hecho, vincule una cuenta de Azure Storage a la cuenta de Batch mediante [Azure Portal](https://portal.azure.com):

1. Vaya a la cuenta de Batch en Azure Portal.
1. En **Configuración**, seleccione **Cuenta de Storage**.
1. Si no dispone de una cuenta de Storage asociada con su cuenta de Batch, haga clic en **Storage Account (None)** [Cuenta de Storage (ninguna)].
1. Seleccione una cuenta de Storage de la lista para su suscripción. Para obtener el mejor rendimiento, utilice una cuenta de Azure Storage que esté en la misma región que la cuenta de Batch en la que se ejecutan las tareas.

## <a name="persist-output-data"></a>Guardar datos de salida

Para guardar los datos de salida de trabajos y tareas con la biblioteca de convenciones de archivo, cree un contenedor en Azure Storage y, a continuación, guarde la salida en el contenedor. Use la [biblioteca de cliente de Azure Storage para .NET](https://www.nuget.org/packages/WindowsAzure.Storage) en el código de tarea para cargar la salida de la tarea en el contenedor. 

Para más información sobre el trabajo con contenedores y blobs en Azure Storage, consulte [Introducción al Almacenamiento de blobs de Azure mediante .NET](../storage/blobs/storage-dotnet-how-to-use-blobs.md).

> [!WARNING]
> Todas las salidas de trabajos y tareas guardados con la biblioteca de convenciones de archivo se almacenan en el mismo contenedor. Si un gran número de tareas intenta guardar archivos al mismo tiempo, se pueden aplicar [límites de almacenamiento](../storage/common/storage-performance-checklist.md#blobs).

### <a name="create-storage-container"></a>Creación de contenedores de almacenamiento

Para guardar las salidas de tareas en Azure Storage, primero cree un contenedor mediante una llamada a [CloudJob][net_cloudjob].[ PrepareOutputStorageAsync][net_prepareoutputasync]. Este método de extensión toma un objeto [CloudStorageAccount][net_cloudstorageaccount] como parámetro. Crea un contenedor denominado según el estándar File Conventions, por lo que su contenido es reconocible para Azure Portal y los métodos de recuperación que se describen más adelante en este artículo.

Normalmente se coloca el código para crear un contenedor en la aplicación cliente &mdash;, la aplicación que crea los grupos, los trabajos y las tareas.

```csharp
CloudJob job = batchClient.JobOperations.CreateJob(
    "myJob",
    new PoolInformation { PoolId = "myPool" });

// Create reference to the linked Azure Storage account
CloudStorageAccount linkedStorageAccount =
    new CloudStorageAccount(myCredentials, true);

// Create the blob storage container for the outputs
await job.PrepareOutputStorageAsync(linkedStorageAccount);
```

### <a name="store-task-outputs"></a>Almacenamiento de las salidas de tarea

Ahora que ha preparado un contenedor en Azure Storage, las tareas pueden guardar las salidas en el contenedor mediante la clase [TaskOutputStorage][net_taskoutputstorage] que se encuentra en la biblioteca de convenciones de archivos.

En el código de la tarea, cree primero un objeto [TaskOutputStorage][net_taskoutputstorage] y, luego, cuando la tarea haya finalizado su trabajo, llame al método [TaskOutputStorage][net_taskoutputstorage].[SaveAsync][net_saveasync] para guardar su salida en Azure Storage.

```csharp
CloudStorageAccount linkedStorageAccount = new CloudStorageAccount(myCredentials);
string jobId = Environment.GetEnvironmentVariable("AZ_BATCH_JOB_ID");
string taskId = Environment.GetEnvironmentVariable("AZ_BATCH_TASK_ID");

TaskOutputStorage taskOutputStorage = new TaskOutputStorage(
    linkedStorageAccount, jobId, taskId);

/* Code to process data and produce output file(s) */

await taskOutputStorage.SaveAsync(TaskOutputKind.TaskOutput, "frame_full_res.jpg");
await taskOutputStorage.SaveAsync(TaskOutputKind.TaskPreview, "frame_low_res.jpg");
```

El parámetro `kind` del método [TaskOutputStorage](/dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage).[SaveAsync](/dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage.saveasync#overloads) permite clasificar los archivos guardados. Hay cuatro tipos predefinidos de [TaskOutputKind][net_taskoutputkind]: `TaskOutput`, `TaskPreview`, `TaskLog`, y `TaskIntermediate.`. También puede definir categorías personalizadas de salida.

Estos tipos de salidas le permiten especificar qué tipo de salidas se deben mostrar cuándo se realiza una consulta a Batch relacionada con las salidas guardadas de una tarea dada. En otras palabras, cuando muestra las salidas de una tarea, puede filtrar la lista por uno de los tipos de salida. Por ejemplo, "dame la salida *preview* de la tarea *109*". Aparece más información sobre la presentación y recuperación de salidas en la sección Recuperación de salidas más adelante en el artículo.

> [!TIP]
> El tipo de salida también determina dónde se mostrará un determinado archivo en Azure Portal: Los archivos clasificados como *TaskOutput* aparecen en **Task output files** (Archivos de salida de la tarea), y los archivos *TaskLog* en **Task logs** (Registros de tareas).

### <a name="store-job-outputs"></a>Almacenamiento de salidas de trabajos

Además de almacenar las salidas de tareas, puede almacenar las salidas asociadas a un trabajo completo. Por ejemplo, en la tarea de combinación de un trabajo de representación de una película, podría guardar la película totalmente representada como una salida de trabajo. Cuando se completa el trabajo, la aplicación cliente puede mostrar y recuperar las salidas del trabajo y no es necesario consultar las tareas individuales.

Almacene la salida del trabajo mediante una llamada al método [JobOutputStorage][net_joboutputstorage].[SaveAsync][net_joboutputstorage_saveasync] y especifique el [JobOutputKind][net_joboutputkind] y el nombre de archivo:

```csharp
CloudJob job = new JobOutputStorage(acct, jobId);
JobOutputStorage jobOutputStorage = job.OutputStorage(linkedStorageAccount);

await jobOutputStorage.SaveAsync(JobOutputKind.JobOutput, "mymovie.mp4");
await jobOutputStorage.SaveAsync(JobOutputKind.JobPreview, "mymovie_preview.mp4");
```

Al igual que con el tipo **TaskOutputKind** para las salidas de tareas, utilice el tipo [JobOutputKind][net_joboutputkind] para clasificar los archivos guardados de un trabajo. Este parámetro permite realizar consultas posteriores (mostrar) para un tipo específico de salida. El tipo **JobOutputKind** incluye las categorías de salida y de vista previa y admite la creación de categorías personalizadas.

### <a name="store-task-logs"></a>Almacenamiento de los registros de tareas

Además de guardar un archivo en un almacenamiento duradero cuando se completa una tarea o trabajo, es posible que necesite guardar los archivos que se actualizan durante la ejecución de una tarea, los archivos de registro de &mdash;, o `stdout.txt` y `stderr.txt`, por ejemplo. Para ello, la biblioteca Azure Batch File Conventions proporciona el método [TaskOutputStorage][net_taskoutputstorage].[SaveTrackedAsync][net_savetrackedasync]. Con [SaveTrackedAsync][net_savetrackedasync], puede realizar un seguimiento de las actualizaciones de un archivo del nodo (durante un intervalo especificado) y almacenar esas actualizaciones en Azure Storage.

En el siguiente fragmento de código, utilizamos [SaveTrackedAsync][net_savetrackedasync] para actualizar `stdout.txt` en Azure Storage cada 15 segundos durante la ejecución de la tarea:

```csharp
TimeSpan stdoutFlushDelay = TimeSpan.FromSeconds(3);
string logFilePath = Path.Combine(
    Environment.GetEnvironmentVariable("AZ_BATCH_TASK_DIR"), "stdout.txt");

// The primary task logic is wrapped in a using statement that sends updates to
// the stdout.txt blob in Storage every 15 seconds while the task code runs.
using (ITrackedSaveOperation stdout =
        await taskStorage.SaveTrackedAsync(
        TaskOutputKind.TaskLog,
        logFilePath,
        "stdout.txt",
        TimeSpan.FromSeconds(15)))
{
    /* Code to process data and produce output file(s) */

    // We are tracking the disk file to save our standard output, but the
    // node agent may take up to 3 seconds to flush the stdout stream to
    // disk. So give the file a moment to catch up.
     await Task.Delay(stdoutFlushDelay);
}
```

La sección comentada `Code to process data and produce output file(s)` es simplemente un marcador de posición para el código que normalmente realizaría la tarea. Por ejemplo, es posible que tenga código que descargue datos desde Azure Storage y realice alguna transformación o cálculo en él. La parte importante de este fragmento de código demuestra cómo puede encapsular ese código en un bloque `using` para actualizar periódicamente un archivo con [SaveTrackedAsync][net_savetrackedasync].

El agente de nodo es un programa que se ejecuta en cada nodo del grupo y proporciona la interfaz de comandos y control entre el nodo y el servicio Batch. Se necesita la llamada `Task.Delay` al final de este bloque `using` para asegurarse de que el agente de nodo tiene tiempo de vaciar el contenido de la salida estándar en el archivo stdout.txt del nodo. Sin este retraso, es posible que se pierdan los últimos segundos de salida. Este retraso puede no ser necesario para todos los archivos.

> [!NOTE]
> Si habilita el seguimiento de archivos con **SaveTrackedAsync**, solo se guardan en Azure Storage los *anexos* al archivo de seguimiento. Utilice este método solo para el seguimiento de archivos de registro no rotatorios u otros archivos escritos con operaciones de anexión al final del archivo.

## <a name="retrieve-output-data"></a>Recuperación de datos de salidas

Cuando recupera la salida guardada mediante la biblioteca de convenciones de archivos de Azure Batch, lo hace basándose en las tareas y trabajos. Puede solicitar la salida de una tarea o trabajo determinado sin necesidad de conocer su ruta de acceso en Azure Storage o incluso sin conocer el nombre del archivo. En su lugar, puede solicitar archivos de salida por el identificador de tarea o trabajo.

El fragmento de código siguiente procesa una iteración de las tareas de un trabajo, imprime información sobre los archivos de salida de la tarea y luego descarga los archivos desde Storage.

```csharp
foreach (CloudTask task in myJob.ListTasks())
{
    foreach (OutputFileReference output in
        task.OutputStorage(storageAccount).ListOutputs(
            TaskOutputKind.TaskOutput))
    {
        Console.WriteLine($"output file: {output.FilePath}");

        output.DownloadToFileAsync(
            $"{jobId}-{output.FilePath}",
            System.IO.FileMode.Create).Wait();
    }
}
```

## <a name="view-output-files-in-the-azure-portal"></a>Visualización de archivos de salida en Azure Portal

Azure Portal muestra los archivos de salidas y los registros de las tareas que se guardan en una cuenta de Azure Storage vinculada mediante el [estándar de convenciones de archivo de Batch](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions). Puede implementar usted mismo estas convenciones en un lenguaje de su elección, o puede usar la biblioteca de convenciones de archivo en las aplicaciones. NET.

Para habilitar la presentación de los archivos de salidas en el portal, debe cumplir con los siguientes requisitos:

1. Vincular una cuenta de Azure Storage a la cuenta de Batch.
1. Cumplir las convenciones de nomenclatura predefinidas para los contenedores y archivos de almacenamiento al guardar salidas. Puede encontrar la definición de estas convenciones en el archivo [LÉAME][github_file_conventions_readme] de la biblioteca de convenciones de archivo. Si utiliza la biblioteca de [convenciones de archivo de Azure Batch][nuget_package] para guardar la salida, los archivos se guardarán según el estándar de convenciones de archivo.

Para ver los archivos de salidas y registros de las tareas en Azure Portal, navegue hasta la tarea en cuya salida está interesado y, después, haga clic en **Archivos de salida guardados** o **Registros guardados**. Esta imagen muestra los **archivos de salida guardados** para la tarea con identificador "007":

![Hoja de salidas de tareas de Azure Portal][2]

## <a name="code-sample"></a>Código de ejemplo

El proyecto de ejemplo [PersistOutputs][github_persistoutputs] es uno de los ejemplos de código de [Azure Batch][github_samples] de GitHub. Esta solución de Visual Studio muestra cómo utilizar la biblioteca de convenciones de archivos de Azure Batch para guardar la salida de las tareas en un almacenamiento duradero. Para ejecutar el ejemplo, siga estos pasos:

1. Abra el proyecto en **2019 de Visual Studio**.
2. Agregue las **credenciales de cuenta** de Batch y Storage a **AccountSettings.settings** al proyecto Microsoft.Azure.Batch.Samples.Common.
3. **Compile** (pero no ejecute) la solución. Si se le solicita, restaure los paquetes NuGet.
4. Use el Portal de Azure para cargar un [paquete de aplicación](batch-application-packages.md) para **PersistOutputsTask**. Incluya el archivo `PersistOutputsTask.exe` y los ensamblados dependientes en el paquete zip, establezca el identificador de la aplicación en "PersistOutputsTask" y la versión del paquete de aplicación en "1.0".
5. **Inicie** (ejecute) el proyecto**PersistOutputs**.
6. Cuando se le solicite que elija la tecnología de guardado que desea usar para ejecutar el ejemplo, escriba **1** para ejecutarlo mediante la biblioteca de convenciones de archivo para guardar la salida de la tarea. 

## <a name="next-steps"></a>Pasos siguientes

### <a name="get-the-batch-file-conventions-library-for-net"></a>Obtención de la biblioteca de convenciones de archivo para .NET de Batch

Esta biblioteca está disponible en [NuGet][nuget_package]. La biblioteca permite extender las clases [CloudJob][net_cloudjob] y [CloudTask][net_cloudtask] con nuevos métodos. Consulte también la [documentación de referencia](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.conventions.files) de la biblioteca de convenciones de archivo.

Puede encontrar el [código fuente][github_file_conventions] de la biblioteca de convenciones de archivo en GitHub en el repositorio del Microsoft Azure SDK para .NET. 

### <a name="explore-other-approaches-for-persisting-output-data"></a>Exploración de otros métodos para guardar los datos de salida

- Consulte [Guardar salidas de trabajos y tareas en Azure Storage](batch-task-output.md) para obtener información general sobre cómo guardar datos de tareas y trabajos.
- Consulte [Almacenamiento de datos de tareas en Azure Storage con la API del servicio Batch](batch-task-output-files.md) para aprender a usar la API del servicio Batch para guardar los datos de salida.

[forum_post]: https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[github_file_conventions]: https://github.com/Azure/azure-sdk-for-net/tree/AutoRest/src/Batch/FileConventions
[github_file_conventions_readme]: https://github.com/Azure/azure-sdk-for-net/blob/AutoRest/src/Batch/FileConventions/README.md
[github_persistoutputs]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs
[github_samples]: https://github.com/Azure/azure-batch-samples
[net_batchclient]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudjob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_cloudstorageaccount]: https://docs.microsoft.com/java/api/com.microsoft.azure.storage._cloud_storage_account
[net_cloudtask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_fileconventions_readme]: https://github.com/Azure/azure-sdk-for-net/blob/AutoRest/src/Batch/FileConventions/README.md
[net_joboutputkind]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.joboutputkind.aspx
[net_joboutputstorage]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.joboutputstorage.aspx
[net_joboutputstorage_saveasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.joboutputstorage.saveasync.aspx
[net_msdn]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[net_prepareoutputasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.cloudjobextensions.prepareoutputstorageasync.aspx
[net_saveasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.saveasync.aspx
[net_savetrackedasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.savetrackedasync.aspx
[net_taskoutputkind]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputkind.aspx
[net_taskoutputstorage]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.aspx
[nuget_manager]: https://docs.nuget.org/consume/installing-nuget
[nuget_package]: https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files
[portal]: https://portal.azure.com
[storage_explorer]: https://storageexplorer.com/

[1]: ./media/batch-task-output/task-output-01.png "Selectores de archivos de salida guardados y registros guardados en el portal"
[2]: ./media/batch-task-output/task-output-02.png "Hoja de salidas de tareas de Azure Portal"
