---
title: Guardar salidas de trabajos y tareas en Azure Storage con la API del servicio Azure Batch | Microsoft Docs
description: Aprenda a usar la API del servicio Batch para guardar salidas de trabajos y tareas de Batch en Azure Storage.
services: batch
author: laurenhughes
manager: jeconnoc
editor: ''
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 03/05/2019
ms.author: lahugh
ms.custom: seodec18
ms.openlocfilehash: 982fe5cea633d9fd1bbbe7dc862b69d89f5f1c1c
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595278"
---
# <a name="persist-task-data-to-azure-storage-with-the-batch-service-api"></a>Almacenamiento de datos de tareas en Azure Storage con la API del servicio Batch

[!INCLUDE [batch-task-output-include](../../includes/batch-task-output-include.md)]

La API del servicio Batch permite almacenar datos de salida en Azure Storage para tareas simples y tareas de administrador de trabajos que se ejecutan en los grupos con la configuración de máquina virtual. Cuando se agrega una tarea, puede especificar un contenedor de Azure Storage como destino para la salida de la tarea. Posteriormente, el servicio Batch escribe los datos de salida en ese contenedor cuando se completa la tarea.

Una ventaja de utilizar la API del servicio Batch para guardar la salida de las tareas es que no es necesario modificar la aplicación que la tarea ejecuta. En su lugar, con unas modificaciones en la aplicación cliente, puede almacenar la salida de las tareas desde dentro del mismo código que crea la tarea.

## <a name="when-do-i-use-the-batch-service-api-to-persist-task-output"></a>¿Cuándo debo usar la API del servicio Batch para almacenar la salida de tarea?

Azure Batch proporciona más de una manera de guardar las salidas de tareas. El uso de la API de servicio de Batch es el método más adecuado para estos escenarios:

- Desea escribir código para guardar la salida de las tareas desde dentro de la aplicación cliente, sin modificar la aplicación que la tarea ejecuta.
- Quiere guardar las salidas de las tareas y trabajos del administrador de Batch en grupos creados con la configuración de máquina virtual.
- Quiere guardar salidas en un contenedor de Azure Storage con un nombre arbitrario.
- Quiere almacenar salidas en un contenedor de Azure Storage denominado según el [estándar de convenciones de archivo de Batch](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions). 

Si su escenario es diferente de los mencionados anteriormente, considere la adopción de un enfoque diferente. Por ejemplo, la API del servicio Batch no admite actualmente la transmisión de la salida a Azure Storage mientras se ejecuta la tarea. Para transmitir salidas, considere el uso de la biblioteca de convenciones de archivo de Batch, disponible para. NET. Para otros lenguajes, debe implementar su propia solución. Para más información sobre otras opciones para guardar la salida de tareas, consulte [Guardar salidas de trabajos y tareas en Azure Storage](batch-task-output.md).

## <a name="create-a-container-in-azure-storage"></a>Creación de un contenedor en Azure Storage

Para guardar las salidas de tareas en Azure Storage, debe crear un contenedor que actúe como destino de los archivos de salida. Cree el contenedor antes de ejecutar la tarea, preferiblemente antes de enviar el trabajo. Para crear el contenedor, use la biblioteca de cliente o SDK adecuados de Azure Storage. Para más información sobre las API de Azure Storage, consulte [Documentación de Azure Storage](https://docs.microsoft.com/azure/storage/).

Por ejemplo, si va a escribir la aplicación en C#, utilice la [biblioteca de cliente para .NET de Azure Storage](https://www.nuget.org/packages/WindowsAzure.Storage/). En el ejemplo siguiente se muestra cómo crear un contenedor:

```csharp
CloudBlobContainer container = storageAccount.CreateCloudBlobClient().GetContainerReference(containerName);
await container.CreateIfNotExists();
```

## <a name="get-a-shared-access-signature-for-the-container"></a>Obtención de una firma de acceso compartido para el contenedor

Después de crear el contenedor, obtenga una firma de acceso compartido (SAS) con acceso de escritura en el contenedor. Una SAS proporciona acceso delegado al contenedor. La SAS concede acceso con un conjunto de permisos y durante un intervalo de tiempo que puede especificar. El servicio Batch necesita una SAS con permisos de escritura para escribir la salida de la tarea en el contenedor. Para más información sobre las firmas de acceso compartido, consulte [Uso de firmas de acceso compartido \(SAS\) en Azure Storage](../storage/common/storage-dotnet-shared-access-signature-part-1.md).

Cuando se obtiene una SAS mediante las API de Azure Storage, la API devuelve una cadena de token de SAS. Esta cadena de token incluye todos los parámetros de la SAS, incluidos los permisos y el intervalo durante el que la SAS es válida. Para usar la SAS para acceder a un contenedor de Azure Storage, es necesario anexar la cadena de token de SAS al URI del recurso. El identificador URI del recurso, junto con el token de SAS anexado, proporciona acceso autenticado a Azure Storage.

En el ejemplo siguiente se muestra cómo obtener una cadena de token de SAS de solo escritura para el contenedor y, a continuación, anexar la SAS al URI de este:

```csharp
string containerSasToken = container.GetSharedAccessSignature(new SharedAccessBlobPolicy()
{
    SharedAccessExpiryTime = DateTimeOffset.UtcNow.AddDays(1),
    Permissions = SharedAccessBlobPermissions.Write
});

string containerSasUrl = container.Uri.AbsoluteUri + containerSasToken;
```

## <a name="specify-output-files-for-task-output"></a>Especificar archivos de salida para la salida de la tarea

Para especificar archivos de salida para una tarea, cree una colección de objetos [OutputFile](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.outputfile) y asígnela a la propiedad [CloudTask.OutputFiles](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudtask.outputfiles#Microsoft_Azure_Batch_CloudTask_OutputFiles) cuando cree la tarea.

En el siguiente ejemplo de código C# se crea una tarea que escribe números aleatorios en un archivo denominado `output.txt`. En el ejemplo se crea un archivo de salida para `output.txt` que se escribe en el contenedor. En el ejemplo también se crean archivos de salida para todos los archivos de registro que coinciden con el patrón de archivos `std*.txt` (_p. ej._, `stdout.txt` y `stderr.txt`). La URL del contenedor requiere la SAS que creó anteriormente para este. El servicio Batch utiliza la SAS para autenticar el acceso al contenedor:

```csharp
new CloudTask(taskId, "cmd /v:ON /c \"echo off && set && (FOR /L %i IN (1,1,100000) DO (ECHO !RANDOM!)) > output.txt\"")
{
    OutputFiles = new List<OutputFile>
    {
        new OutputFile(
            filePattern: @"..\std*.txt",
            destination: new OutputFileDestination(
         new OutputFileBlobContainerDestination(
                    containerUrl: containerSasUrl,
                    path: taskId)),
            uploadOptions: new OutputFileUploadOptions(
            uploadCondition: OutputFileUploadCondition.TaskCompletion)),
        new OutputFile(
            filePattern: @"output.txt",
            destination: 
         new OutputFileDestination(new OutputFileBlobContainerDestination(
                    containerUrl: containerSasUrl,
                    path: taskId + @"\output.txt")),
            uploadOptions: new OutputFileUploadOptions(
            uploadCondition: OutputFileUploadCondition.TaskCompletion)),
}
```

### <a name="specify-a-file-pattern-for-matching"></a>Especifique un patrón de archivos para buscar coincidencias

Cuando especifica un archivo de salida, puede usar la propiedad [OutputFile.FilePattern](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.outputfile.filepattern#Microsoft_Azure_Batch_OutputFile_FilePattern) para especificar un patrón de archivos para buscar coincidencias. El patrón de archivos puede no coincidir con ningún archivo, coincidir con un solo archivo o con un conjunto de archivos creados por la tarea.

La propiedad **FilePattern** admite caracteres comodín del sistema de archivos estándar, como `*` (para coincidencias no recursivas) y `**` (para coincidencias recursivas). Por ejemplo, el ejemplo de código anterior especifica el patrón de archivos para que coincida con `std*.txt` de manera no recursiva:

`filePattern: @"..\std*.txt"`

Para cargar un único archivo, especifique un patrón de archivos sin ningún carácter comodín. Por ejemplo, el ejemplo de código anterior especifica el patrón de archivos para que coincida con `output.txt`:

`filePattern: @"output.txt"`

### <a name="specify-an-upload-condition"></a>Especificar una condición de carga

La propiedad [OutputFileUploadOptions.UploadCondition](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.outputfileuploadoptions.uploadcondition#Microsoft_Azure_Batch_OutputFileUploadOptions_UploadCondition) permite la carga condicional de archivos de salida. Un escenario común consiste en cargar un conjunto de archivos si la tarea finaliza correctamente y otro diferente si se produce un error. Por ejemplo, puede que desee cargar archivos de registro detallados solo cuando se produce un error en la tarea y finaliza con un código de salida distinto de cero. De igual forma, es posible que quiera cargar los archivos de resultados solo si la tarea finaliza correctamente, ya que puede que esos archivos falten o estén incompletos si se produce un error en la tarea.

El ejemplo de código anterior permite establecer la propiedad **UploadCondition** en **TaskCompletion**. Esta configuración especifica que se debe cargar el archivo una vez completada la tarea, independientemente del valor del código de salida.

`uploadCondition: OutputFileUploadCondition.TaskCompletion`

Para otras configuraciones, consulte la enumeración [OutputFileUploadCondition](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.common.outputfileuploadcondition).

### <a name="disambiguate-files-with-the-same-name"></a>Distinción entre archivos con el mismo nombre

Las tareas de un trabajo pueden producir archivos que tienen el mismo nombre. Por ejemplo, `stdout.txt` y `stderr.txt` se crean para todas las tareas que se ejecutan en un trabajo. Como cada tarea se ejecuta en su propio contexto, estos archivos no están en conflicto en el sistema de archivos del nodo. Sin embargo, al cargar archivos de varias tareas en un contenedor compartido, debe realizar la distinción de los archivos con el mismo nombre.

La propiedad [OutputFileBlobContainerDestination.Path](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.outputfileblobcontainerdestination.path#Microsoft_Azure_Batch_OutputFileBlobContainerDestination_Path) especifica el blob de destino o directorio virtual para los archivos de salida. Puede usar la propiedad **Path** para dar nombre al blob o directorio virtual de tal manera que los archivos de salida con el mismo nombre reciban un nombre único en Azure Storage. El uso del identificador de la tarea en la ruta es una buena manera de asegurarse nombres únicos y una fácil identificación de los archivos.

Si la propiedad **FilePattern** está establecida en una expresión comodín, se cargarán todos los archivos que coincidan con el patrón en el directorio virtual especificado en la propiedad **Path**. Por ejemplo, si el contenedor es `mycontainer`, el identificador de la tarea es `mytask` y el patrón de archivo es `..\std*.txt`, los identificadores URI absolutos de los archivos de salida de Azure Storage serán similares a este:

```
https://myaccount.blob.core.windows.net/mycontainer/mytask/stderr.txt
https://myaccount.blob.core.windows.net/mycontainer/mytask/stdout.txt
```

Si la propiedad **FilePattern** se establece para que coincida con un único nombre de archivo, lo que implica que no contiene caracteres comodín, el valor de la propiedad **Path** especifica el nombre completo del blob. Si prevé conflictos de nomenclatura con un único archivo de varias tareas, incluya el nombre del directorio virtual como parte del nombre de archivo para distinguir entre esos archivos. Por ejemplo, establezca la propiedad **Path** para que incluya el identificador de tarea, el carácter delimitador (normalmente una barra diagonal) y el nombre de archivo:

`path: taskId + @"/output.txt"`

Los identificadores URI absolutos de los archivos de salida de un conjunto de tareas serán parecidos a:

```
https://myaccount.blob.core.windows.net/mycontainer/task1/output.txt
https://myaccount.blob.core.windows.net/mycontainer/task2/output.txt
```

Para más información acerca de los directorios virtuales de Azure Storage, consulte [Enumerar los blobs de un contenedor](../storage/blobs/storage-quickstart-blobs-dotnet.md#list-the-blobs-in-a-container).

## <a name="diagnose-file-upload-errors"></a>Diagnóstico de errores de carga de archivos

Si se produce un error en la carga de archivos de salida en Azure Storage, la tarea pasa al estado **Completado** y se establece la propiedad [TaskExecutionInformation.FailureInformation](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.taskexecutioninformation.failureinformation#Microsoft_Azure_Batch_TaskExecutionInformation_FailureInformation). Examine la propiedad **FailureInformation** para determinar cuál fue el error. Por ejemplo, este es un error que se produce en la carga de archivos si no se puede encontrar el contenedor:

```
Category: UserError
Code: FileUploadContainerNotFound
Message: One of the specified Azure container(s) was not found while attempting to upload an output file
```

En cada carga de archivos, Batch escribe dos archivos de registro en el nodo de proceso, `fileuploadout.txt` y `fileuploaderr.txt`. Puede examinar estos archivos de registro para obtener más información sobre un error específico. En aquellos casos en los que la carga de archivos nunca llegó a comenzar, por ejemplo, porque no se pudo ejecutar la tarea propiamente dicha, estos archivos de registro no existen.

## <a name="diagnose-file-upload-performance"></a>Diagnóstico del rendimiento de la carga de archivos

El archivo `fileuploadout.txt` registra el progreso de la carga. Puede examinar este archivo para obtener más información acerca de cuánto tiempo tardan los archivos en cargarse. Tenga en cuenta que hay muchos factores que influyen en el rendimiento de carga como, por ejemplo, el tamaño del nodo, otra actividad del nodo en el momento de la carga, si el contenedor de destino está en la misma región que el grupo de Batch, cuántos nodos están cargando archivos en la cuenta de almacenamiento al mismo tiempo, etc.

## <a name="use-the-batch-service-api-with-the-batch-file-conventions-standard"></a>Uso de la API del servicio Batch con el estándar de convenciones de archivo de Batch

Cuando guarda la salida de la tarea con la API del servicio Batch, puede asignar el nombre que desee al contenedor y a los blobs de destino. También puede decidir asignarles un nombre conforme al [estándar de convenciones de archivo de Batch](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions). El estándar de convenciones de archivos determina los nombres del contenedor de destino y el blob en Azure Storage para un archivo de salida dado según los nombres del trabajo y de la tarea. Si usa el estándar de convenciones de archivos para la denominación de archivos de salida, estos se podrán ver en [Azure Portal](https://portal.azure.com).

Si va a desarrollar con C#, puede usar los métodos integrados en la [biblioteca de convenciones de archivo para .NET de Batch](https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files). Esta biblioteca crea los contenedores y las rutas de acceso de blobs con los nombres correctos. Por ejemplo, puede llamar a la API para obtener el nombre correcto del contenedor, basándose en el nombre del trabajo:

```csharp
string containerName = job.OutputStorageContainerName();
```

Puede usar el método [CloudJobExtensions.GetOutputStorageContainerUrl](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.conventions.files.cloudjobextensions.getoutputstoragecontainerurl) para devolver una URL de firma de acceso compartido (SAS) que se utiliza para escribir en el contenedor. A continuación, puede pasar esta SAS al constructor [OutputFileBlobContainerDestination](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.outputfileblobcontainerdestination).

Si va a desarrollar con un lenguaje distinto de C#, tendrá que implementar el estándar de convenciones de archivos usted mismo.

## <a name="code-sample"></a>Código de ejemplo

El proyecto de ejemplo [PersistOutputs][github_persistoutputs] es uno de los ejemplos de código de [Azure Batch][github_samples] de GitHub. Esta solución de Visual Studio muestra cómo utilizar la biblioteca de cliente para .NET de Azure Batch para guardar la salida de las tareas en un almacenamiento duradero. Para ejecutar el ejemplo, siga estos pasos:

1. Abra el proyecto en **2019 de Visual Studio**.
2. Agregue las **credenciales de cuenta** de Batch y Storage a **AccountSettings.settings** al proyecto Microsoft.Azure.Batch.Samples.Common.
3. **Compile** (pero no ejecute) la solución. Si se le solicita, restaure los paquetes NuGet.
4. Use el Portal de Azure para cargar un [paquete de aplicación](batch-application-packages.md) para **PersistOutputsTask**. Incluya el archivo `PersistOutputsTask.exe` y los ensamblados dependientes en el paquete zip, establezca el identificador de la aplicación en "PersistOutputsTask" y la versión del paquete de aplicación en "1.0".
5. **Inicie** (ejecute) el proyecto**PersistOutputs**.
6. Cuando se le solicite que elija la tecnología de guardado que desea usar para ejecutar el ejemplo, escriba **2** si desea ejecutarlo mediante la API del servicio Batch para guardar la salida de la tarea.
7. Si lo desea, ejecute el ejemplo de nuevo, y escriba **3** para guardar la salida con la API del servicio Batch y asignar nombre al contenedor de destino y a la ruta del blob según el estándar de convenciones de archivos.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre el almacenamiento de salidas de tareas con la biblioteca File Conventions para .NET, consulte [Persist job and task data to Azure Storage with the Batch File Conventions library for .NET](batch-task-output-file-conventions.md) (Almacenamiento de datos de trabajos y tareas en Azure Storage con la biblioteca de convenciones de archivo de Batch para .NET).
- Para más información sobre otros métodos para guardar la salida de tareas en Azure Batch, consulte [Guardar salidas de trabajos y tareas en Azure Storage](batch-task-output.md).

[github_persistoutputs]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs
[github_samples]: https://github.com/Azure/azure-batch-samples
