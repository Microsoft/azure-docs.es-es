---
title: Ejecución de una carga de trabajo en paralelo mediante la API para .NET
description: 'Tutorial: Transcodificación de archivos multimedia en paralelo con ffmpeg en Azure Batch con la biblioteca cliente de .NET de Batch'
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 12/21/2018
ms.custom: mvc
ms.openlocfilehash: afa660a7138f3b69b2a6f7c478550095f357e29b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87062591"
---
# <a name="tutorial-run-a-parallel-workload-with-azure-batch-using-the-net-api"></a>Tutorial: Ejecución de una carga de trabajo paralela con Azure Batch mediante la API de .NET

Use Azure Batch para ejecutar aplicaciones de informática de alto rendimiento (HPC) en paralelo y a gran escala, de manera eficaz en Azure. Este tutorial le guía a través de un ejemplo de C# de ejecución de carga de trabajo paralela con Batch. Aprenderá lo que es un flujo de trabajo de aplicación de Batch común y a interactuar con los recursos de Batch y Storage mediante programación. Aprenderá a:

> [!div class="checklist"]
> * Agregar un paquete de aplicación a la cuenta de Batch
> * Autenticarse en las cuentas de Batch y Storage
> * Cargar archivos de entrada en Storage
> * Crear un grupo de nodos de proceso para ejecutar una aplicación
> * Crear un trabajo y tareas para procesar archivos de entrada
> * Supervisar la ejecución de las tareas
> * Recuperación de archivos de salida

En este tutorial, convertiremos archivos multimedia MP4 a formato MP3 en paralelo con la herramienta de código de abierto [ffmpeg](https://ffmpeg.org/). 

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

* [Visual Studio 2017 o superior](https://www.visualstudio.com/vs) o [.NET Core 2.1](https://www.microsoft.com/net/download/dotnet-core/2.1) para Linux, macOS o Windows.

* Una cuenta de Batch y una cuenta de Azure Storage vinculada. Para crear estas cuentas, consulte las guías de inicio rápido de Batch con [Azure Portal](quick-create-portal.md) o la [CLI de Azure](quick-create-cli.md).

* [Versión de 64 bits de Windows de ffmpeg 3.4](https://ffmpeg.zeranoe.com/builds/win64/static/ffmpeg-3.4-win64-static.zip) (.zip). Descargue el archivo zip en el equipo local. Para este tutorial solo se necesita el archivo zip. No es necesario descomprimir el archivo ni instalarlo localmente.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="add-an-application-package"></a>Incorporación de un paquete de aplicación

Use Azure Portal para agregar ffmpeg a la cuenta de Batch como [paquete de aplicación](batch-application-packages.md). Los paquetes de aplicación ayudan a administrar aplicaciones de tareas y a implementarlas en los nodos de proceso del grupo. 

1. En Azure Portal, haga clic en **Más servicios** > **Cuentas de Batch** y en el nombre de la cuenta de Batch.
3. Haga clic en **Aplicaciones** > **Agregar**.
4. Escriba *ffmpeg* como **identificador de aplicación** e indique *3.4* como versión de paquete. Seleccione el archivo zip de ffmpeg que ha descargado y haga clic en **Aceptar**. El paquete de aplicación de ffmpeg se agrega a la cuenta de Batch.

![Incorporación del paquete de aplicación](./media/tutorial-parallel-dotnet/add-application.png)

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)]

## <a name="download-and-run-the-sample"></a>Descarga y ejecución del ejemplo

### <a name="download-the-sample"></a>Descarga del ejemplo

[Descargue o clone la aplicación de ejemplo](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial) desde GitHub. Para clonar el repositorio de la aplicación de ejemplo con un cliente de Git, use el siguiente comando:

```
git clone https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial.git
```

Vaya al directorio que contiene el archivo `BatchDotNetFfmpegTutorial.sln` de la solución de Visual Studio.

Abra el archivo de la solución en Visual Studio y actualice las cadenas de credenciales de `Program.cs` con los valores obtenidos para las cuentas. Por ejemplo:

```csharp
// Batch account credentials
private const string BatchAccountName = "mybatchaccount";
private const string BatchAccountKey  = "xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ==";
private const string BatchAccountUrl  = "https://mybatchaccount.mybatchregion.batch.azure.com";

// Storage account credentials
private const string StorageAccountName = "mystorageaccount";
private const string StorageAccountKey  = "xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ==";
```

[!INCLUDE [batch-credentials-include](../../includes/batch-credentials-include.md)]

Además, asegúrese de que la referencia del paquete de aplicación de ffmpeg de la solución coincide con el identificador y la versión del paquete de ffmpeg que cargara en la cuenta de Batch.

```csharp
const string appPackageId = "ffmpeg";
const string appPackageVersion = "3.4";
```

### <a name="build-and-run-the-sample-project"></a>Compilación y ejecución del proyecto de ejemplo

Compile y ejecute la aplicación en Visual Studio o en la línea de comandos con los comandos `dotnet build` y `dotnet run`. Después de ejecutar la aplicación, examine el código para ver qué es lo que hace cada parte de la aplicación. Por ejemplo, en Visual Studio:

* Haga clic con el botón derecho en la solución en el Explorador de soluciones y en **Compilar solución**. 

* Si se le solicita, confirme la restauración de los paquetes NuGet. Si necesita descargar los paquetes que faltan, asegúrese de que el [Administrador de paquetes NuGet](https://docs.nuget.org/consume/installing-nuget) está instalado.

A continuación, ejecútelo. Al ejecutar la aplicación de ejemplo, la salida de la consola es similar a la siguiente. Durante la ejecución, se experimenta una pausa en `Monitoring all tasks for 'Completed' state, timeout in 00:30:00...` mientras se inician los nodos de proceso del grupo. 

```
Sample start: 11/19/2018 3:20:21 PM

Container [input] created.
Container [output] created.
Uploading file LowPriVMs-1.mp4 to container [input]...
Uploading file LowPriVMs-2.mp4 to container [input]...
Uploading file LowPriVMs-3.mp4 to container [input]...
Uploading file LowPriVMs-4.mp4 to container [input]...
Uploading file LowPriVMs-5.mp4 to container [input]...
Creating pool [WinFFmpegPool]...
Creating job [WinFFmpegJob]...
Adding 5 tasks to job [WinFFmpegJob]...
Monitoring all tasks for 'Completed' state, timeout in 00:30:00...
Success! All tasks completed successfully within the specified timeout period.
Deleting container [input]...

Sample end: 11/19/2018 3:29:36 PM
Elapsed time: 00:09:14.3418742
```

Vaya a la cuenta de Batch de Azure Portal para supervisar el grupo, los nodos de proceso, el trabajo y las tareas. Por ejemplo, para ver un mapa térmico de los nodos de proceso del grupo, haga clic en **Grupos** > *WinFFmpegPool*.

Con las tareas en ejecución, el mapa térmico es similar al siguiente:

![Mapa térmico de grupo](./media/tutorial-parallel-dotnet/pool.png)

El tiempo de ejecución típico es de aproximadamente **10 minutos** al ejecutar la aplicación con la configuración predeterminada. La creación del grupo es lo que más tarda.

[!INCLUDE [batch-common-tutorial-download](../../includes/batch-common-tutorial-download.md)]

## <a name="review-the-code"></a>Revisión del código

En las secciones siguientes se desglosan los pasos que lleva a cabo la aplicación de ejemplo para procesar una carga de trabajo en el servicio Batch. Consulte el archivo `Program.cs` en la solución mientras lee el resto de este artículo, ya que no se explican todas las líneas de código del ejemplo.

### <a name="authenticate-blob-and-batch-clients"></a>Autenticación de los clientes de Blob y Batch

Para interactuar con la cuenta de almacenamiento vinculada, la aplicación utiliza la biblioteca cliente de Azure Storage para .NET. Crea una referencia a la cuenta con [CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount), con autenticación de clave compartida. A continuación, crea [CloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient).

```csharp
// Construct the Storage account connection string
string storageConnectionString = String.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}",
                                StorageAccountName, StorageAccountKey);

// Retrieve the storage account
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(storageConnectionString);

CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
```

La aplicación crea un objeto [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient) para crear y administrar los grupos, los trabajos y las tareas en el servicio Batch. El cliente de Batch del ejemplo utiliza la autenticación de clave compartida. Batch también admite la autenticación a través de [Azure Active Directory](batch-aad-auth.md) para usuarios individuales o una aplicaciones desatendidas.

```csharp
BatchSharedKeyCredentials sharedKeyCredentials = new BatchSharedKeyCredentials(BatchAccountUrl, BatchAccountName, BatchAccountKey);

using (BatchClient batchClient = BatchClient.Open(sharedKeyCredentials))
...
```

### <a name="upload-input-files"></a>Carga de archivos de entrada

La aplicación pasa el objeto `blobClient` al método `CreateContainerIfNotExistAsync` para crear un contenedor de almacenamiento para los archivos de entrada (formato MP4) y uno para la salida de la tarea.

```csharp
CreateContainerIfNotExistAsync(blobClient, inputContainerName);
CreateContainerIfNotExistAsync(blobClient, outputContainerName);
```

A continuación, los archivos se cargan al contenedor de entrada desde la carpeta `InputFiles` local. Los archivos de almacenamiento se definen como objetos [ResourceFile](/dotnet/api/microsoft.azure.batch.resourcefile) de Batch para que el servicio los descargue después en nodos de proceso. 

En la carga de los archivos participan dos métodos de `Program.cs`:

* `UploadFilesToContainerAsync`: devuelve una colección de objetos ResourceFile y llama internamente a `UploadResourceFileToContainerAsync` para cargar todos los archivos que se pasan en el parámetro `inputFilePaths`.
* `UploadResourceFileToContainerAsync`: carga los archivos como blobs en el contenedor de entrada. Después de cargar el archivo, obtiene una firma de acceso compartido (SAS) para el blob y devuelve un objeto ResourceFile que lo representa.

```csharp
string inputPath = Path.Combine(Environment.CurrentDirectory, "InputFiles");

List<string> inputFilePaths = new List<string>(Directory.GetFileSystemEntries(inputPath, "*.mp4",
    SearchOption.TopDirectoryOnly));

List<ResourceFile> inputFiles = await UploadFilesToContainerAsync(
  blobClient,
  inputContainerName,
  inputFilePaths);
```

Para más información sobre cómo cargar archivos como blobs en una cuenta de almacenamiento con. NET, consulte [Carga, descarga y enumeración de blobs mediante .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md).

### <a name="create-a-pool-of-compute-nodes"></a>Creación de un grupo de nodos de proceso

Después, en el ejemplo se crea un grupo de nodos de proceso en la cuenta de Batch con una llamada a `CreatePoolIfNotExistAsync`. Este método definido usa el método [BatchClient.PoolOperations.CreatePool](/dotnet/api/microsoft.azure.batch.pooloperations.createpool) para establecer el número de nodos, el tamaño de la máquina virtual y la configuración del grupo. En este caso, un objeto [VirtualMachineConfiguration](/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration) especifica un valor de [ImageReference](/dotnet/api/microsoft.azure.batch.imagereference) en una imagen de Windows Server publicada en Azure Marketplace. Batch es compatible con una amplia gama de imágenes de máquina virtual de Azure Marketplace, así como con las imágenes de máquina virtual personalizadas.

El número de nodos y el tamaño de la máquina virtual se establecen mediante constantes definidas. Batch admite nodos especializados y [nodos de prioridad baja](batch-low-pri-vms.md), y en los grupos puede utilizar ambos. Los nodos dedicados están reservados para el grupo. Los nodos de prioridad baja se ofrecen a precio reducido por la capacidad sobrante de las máquinas virtuales de Azure. Los nodos de prioridad baja dejan de estar disponibles si Azure no tiene capacidad suficiente. En el ejemplo, de forma predeterminada se crea un grupo que contiene solo 5 nodos de baja prioridad con el tamaño *Standard_A1_v2*.

>[!Note]
>Asegúrese de comprobar las cuotas de nodo. Consulte [Límites y cuotas del servicio Batch](batch-quota-limit.md#increase-a-quota) para obtener instrucciones sobre cómo crear una solicitud de cuota.

La aplicación ffmpeg se implementa en los nodos de proceso mediante la incorporación de un valor [ApplicationPackageReference](/dotnet/api/microsoft.azure.batch.applicationpackagereference) a la configuración del grupo.

El método [CommitAsync](/dotnet/api/microsoft.azure.batch.cloudpool.commitasync) envía el grupo al servicio Batch.

```csharp
ImageReference imageReference = new ImageReference(
    publisher: "MicrosoftWindowsServer",
    offer: "WindowsServer",
    sku: "2016-Datacenter-smalldisk",
    version: "latest");

VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration(
    imageReference: imageReference,
    nodeAgentSkuId: "batch.node.windows amd64");

pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: DedicatedNodeCount,
    targetLowPriorityComputeNodes: LowPriorityNodeCount,
    virtualMachineSize: PoolVMSize,
    virtualMachineConfiguration: virtualMachineConfiguration);

pool.ApplicationPackageReferences = new List<ApplicationPackageReference>
    {
    new ApplicationPackageReference {
    ApplicationId = appPackageId,
    Version = appPackageVersion}};

await pool.CommitAsync();  
```

### <a name="create-a-job"></a>Creación de un trabajo

Un trabajo de Batch especifica un grupo en el que ejecutar tareas y valores de configuración opcionales, como la prioridad y la programación del trabajo. En el ejemplo se crea un trabajo con una llamada a `CreateJobAsync`. Este método definido usa el método [BatchClient.JobOperations.CreateJob](/dotnet/api/microsoft.azure.batch.joboperations.createjob) para crear un trabajo en el grupo.

El método [CommitAsync](/dotnet/api/microsoft.azure.batch.cloudjob.commitasync) envía el trabajo al servicio Batch. Inicialmente, el trabajo no tiene tareas.

```csharp
CloudJob job = batchClient.JobOperations.CreateJob();
job.Id = JobId;
job.PoolInformation = new PoolInformation { PoolId = PoolId };

await job.CommitAsync();
```

### <a name="create-tasks"></a>Creación de tareas

En el ejemplo se crean tareas en el trabajo con una llamada al método `AddTasksAsync`, que crea una lista de objetos [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask). Los objetos `CloudTask` ejecutan ffmpeg para procesar un objeto `ResourceFile` de entrada con una propiedad [CommandLine](/dotnet/api/microsoft.azure.batch.cloudtask.commandline). Al crear el grupo, ffmpeg se instaló previamente en todos los nodos. En este caso, la línea de comandos ejecuta ffmpeg para convertir los archivos MP4 de entrada (vídeo) en archivos MP3 (audio).

En el ejemplo se crea un objeto [OutputFile](/dotnet/api/microsoft.azure.batch.outputfile) para el archivo MP3 después de ejecutar la línea de comandos. Los archivos de salida de la tarea (en este caso, uno) se cargan en un contenedor en la cuenta de Storage vinculada mediante la propiedad [OutputFiles](/dotnet/api/microsoft.azure.batch.cloudtask.outputfiles) de la tarea. Anteriormente en el ejemplo de código, una dirección URL de firma de acceso compartido (`outputContainerSasUrl`) se ha obtenido para proporcionar acceso de escritura al contenedor de salida. Tenga en cuenta las condiciones establecidas en el objeto `outputFile`. Solo se carga un archivo de salida de una tarea en el contenedor después de que la tarea se haya completado correctamente (`OutputFileUploadCondition.TaskSuccess`). Vea el [código de ejemplo](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial) en GitHub para obtener más detalles sobre la implementación.

A continuación, en el ejemplo se agregan tareas al trabajo con el método [AddTask](/dotnet/api/microsoft.azure.batch.joboperations.addtaskasync), que las pone en cola para que se ejecuten en los nodos de proceso.

```csharp
 // Create a collection to hold the tasks added to the job.
List<CloudTask> tasks = new List<CloudTask>();

for (int i = 0; i < inputFiles.Count; i++)
{
    string taskId = String.Format("Task{0}", i);

    // Define task command line to convert each input file.
    string appPath = String.Format("%AZ_BATCH_APP_PACKAGE_{0}#{1}%", appPackageId, appPackageVersion);
    string inputMediaFile = inputFiles[i].FilePath;
    string outputMediaFile = String.Format("{0}{1}",
        System.IO.Path.GetFileNameWithoutExtension(inputMediaFile),
        ".mp3");
    string taskCommandLine = String.Format("cmd /c {0}\\ffmpeg-3.4-win64-static\\bin\\ffmpeg.exe -i {1} {2}", appPath, inputMediaFile, outputMediaFile);

    // Create a cloud task (with the task ID and command line)
    CloudTask task = new CloudTask(taskId, taskCommandLine);
    task.ResourceFiles = new List<ResourceFile> { inputFiles[i] };

    // Task output file
    List<OutputFile> outputFileList = new List<OutputFile>();
    OutputFileBlobContainerDestination outputContainer = new OutputFileBlobContainerDestination(outputContainerSasUrl);
    OutputFile outputFile = new OutputFile(outputMediaFile,
       new OutputFileDestination(outputContainer),
       new OutputFileUploadOptions(OutputFileUploadCondition.TaskSuccess));
    outputFileList.Add(outputFile);
    task.OutputFiles = outputFileList;
    tasks.Add(task);
}

// Add tasks as a collection
await batchClient.JobOperations.AddTaskAsync(jobId, tasks);
return tasks
```

### <a name="monitor-tasks"></a>Supervisión de las tareas

Cuando Batch agrega tareas a un trabajo, las pone en cola automáticamente y las programa para que se ejecuten en nodos de proceso del grupo asociado. Según la configuración que especifique, Batch controla la administración de las colas, la programación, los reintentos y otras labores de administración de tareas.

Existen varios enfoques para supervisar la ejecución de tareas. En este ejemplo se define un método `MonitorTasks` para informar solo de los estados de finalización y de tarea correcta o incorrecta. El código `MonitorTasks` especifica un valor de [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) para seleccionar eficazmente solo la información mínima sobre las tareas. A continuación, crea [TaskStateMonitor](/dotnet/api/microsoft.azure.batch.taskstatemonitor), que proporciona utilidades auxiliares para supervisar los estados de la tarea. En `MonitorTasks`, el ejemplo espera que todas las tareas alcancen el estado `TaskState.Completed` en un tiempo determinado. A continuación, finaliza el trabajo e informa de las tareas que se completaran, pero que puedan haber producido errores, como un código de salida distinto de cero.

```csharp
TaskStateMonitor taskStateMonitor = batchClient.Utilities.CreateTaskStateMonitor();
try
{
    await taskStateMonitor.WhenAll(addedTasks, TaskState.Completed, timeout);
}
catch (TimeoutException)
{
    batchClient.JobOperations.TerminateJob(jobId);
    Console.WriteLine(incompleteMessage);
    return false;
}
batchClient.JobOperations.TerminateJob(jobId);
 Console.WriteLine(completeMessage);
...

```

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de ejecutar las tareas, la aplicación elimina automáticamente el contenedor de almacenamiento de entrada que creó y ofrece la opción de eliminar el grupo y el trabajo de Batch. Las clases [JobOperations](/dotnet/api/microsoft.azure.batch.batchclient.joboperations) y [Pool Operations](/dotnet/api/microsoft.azure.batch.batchclient.pooloperations) tienen sus métodos de eliminación correspondientes, a los que se llama si el usuario confirma la eliminación. Aunque no se cobran los trabajos y las tareas, sí se cobran los nodos de proceso. Por consiguiente, se recomienda asignar solo los grupos necesarios. Al eliminar el grupo, las salidas de tarea de los nodos también se eliminan. Sin embargo, los archivos de salida permanecen en la cuenta de almacenamiento.

Cuando ya no los necesite, elimine el grupo de recursos, la cuenta de Batch y la de Storage. Para hacerlo desde Azure Portal, seleccione el grupo de recursos de la cuenta de Batch y haga clic en **Eliminar grupo de recursos**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Agregar un paquete de aplicación a la cuenta de Batch
> * Autenticarse en las cuentas de Batch y Storage
> * Cargar archivos de entrada en Storage
> * Crear un grupo de nodos de proceso para ejecutar una aplicación
> * Crear un trabajo y tareas para procesar archivos de entrada
> * Supervisar la ejecución de las tareas
> * Recuperación de archivos de salida

Para más ejemplos de uso de la API de .NET para programar y procesar cargas de trabajo de Batch, consulte los ejemplos de GitHub.

> [!div class="nextstepaction"]
> [Ejemplos de C# de Batch](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp)


Si se establece la variable de instancia LowPriorityNodeCount=0 y DedicatedNodeCount=5 se corrige el problema y se permite que se complete el trabajo.
