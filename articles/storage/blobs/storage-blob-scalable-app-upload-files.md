---
title: Carga de grandes cantidades de datos aleatorios en paralelo en Azure Storage
description: Aprenda a usar la biblioteca cliente de Azure Storage para cargar grandes cantidades de datos aleatorios en paralelo en una cuenta de Azure Storage
author: roygara
ms.service: storage
ms.topic: tutorial
ms.date: 10/08/2019
ms.author: rogarana
ms.subservice: blobs
ms.openlocfilehash: dd87e1a9bcff55813dff420976df58351386fb34
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "75371945"
---
# <a name="upload-large-amounts-of-random-data-in-parallel-to-azure-storage"></a>Cargar grandes cantidades de datos aleatorios en paralelo en Azure Storage

Este tutorial es la segunda parte de una serie. En este tutorial se muestra cómo implementar una aplicación que cargue grandes cantidades de datos aleatorios en una cuenta de Azure Storage.

En la segunda parte de la serie, se aprende a:

> [!div class="checklist"]
> * Configurar la cadena de conexión
> * Compilar la aplicación
> * Ejecución de la aplicación
> * Validar el número de conexiones

Azure Blob Storage proporciona un servicio escalable para almacenar los datos. Para asegurarse de que la aplicación tenga el máximo rendimiento posible, se recomienda comprender el funcionamiento de Blob Storage. Es importante conocer los límites de los blobs de Azure. Para más información sobre estos límites, visite: [Objetivos de escalabilidad y rendimiento de Blob Storage](../blobs/scalability-targets.md).

La [nomenclatura de las particiones](../blobs/storage-performance-checklist.md#partitioning) es otro factor posiblemente importante al diseñar una aplicación de alto rendimiento con blobs. En el caso de tamaños de bloques mayores o iguales que 4 MiB, se usan los [blobs en bloques de alto rendimiento](https://azure.microsoft.com/blog/high-throughput-with-azure-blob-storage/) y la nomenclatura de las particiones no afecta al rendimiento. En el caso de tamaños de bloques menores de 4 MiB, Azure Storage usa un esquema de partición basado en intervalo para escalar y equilibrar la carga. Esta configuración significa que los archivos con prefijos o convenciones de nomenclatura similares van a la misma partición. Esta lógica incluye el nombre del contenedor donde se cargan los archivos. En este tutorial, usará archivos que tienen GUID de nombres, así como contenido generado de manera aleatoria. Estos se cargan posteriormente en cinco contenedores diferentes con nombres aleatorios.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, debe haber completado el tutorial anterior sobre el almacenamiento: [Creación de una máquina virtual y una cuenta de almacenamiento para una aplicación escalable][previous-tutorial].

## <a name="remote-into-your-virtual-machine"></a>Sesión remota en la máquina virtual

Ejecute el comando siguiente en el equipo local para crear una sesión de escritorio remoto con la máquina virtual. Reemplace la dirección IP con el valor de publicIPAddress de la máquina virtual. Cuando se le solicite, escriba las credenciales usadas al crear la máquina virtual.

```
mstsc /v:<publicIpAddress>
```

## <a name="configure-the-connection-string"></a>Configurar la cadena de conexión

En Azure Portal, vaya a la cuenta de almacenamiento. Seleccione **Claves de acceso** en **Configuración**, en su cuenta de almacenamiento. Copie la **cadena de conexión** de la clave principal o secundaria. Inicie sesión en la máquina virtual creada en el tutorial anterior. Abra un **símbolo del sistema** como administrador y ejecute el comando `setx` con el conmutador `/m`. Este comando guarda una variable de entorno de configuración de la máquina. La variable de entorno no está disponible hasta que se vuelve a cargar el **símbolo del sistema**. Reemplace **\<storageConnectionString\>** en el ejemplo siguiente:

```
setx storageconnectionstring "<storageConnectionString>" /m
```

Cuando termine, abra otro **símbolo del sistema**, vaya a `D:\git\storage-dotnet-perf-scale-app` y escriba `dotnet build` para compilar la aplicación.

## <a name="run-the-application"></a>Ejecución de la aplicación

Vaya a `D:\git\storage-dotnet-perf-scale-app`.

Escriba `dotnet run` para ejecutar la aplicación. La primera vez que se ejecuta `dotnet`, rellena la caché de paquetes local, para mejorar la velocidad de restauración y habilitar el acceso sin conexión. Este comando tarda un minuto como máximo en ejecutarse y solo lo hace una vez.

```
dotnet run
```

La aplicación crea cinco contenedores con nombres aleatorios y comienza a cargar los archivos del directorio de almacenamiento provisional en la cuenta de almacenamiento. La aplicación establece el mínimo de subprocesos en 100 y el valor de [DefaultConnectionLimit](https://msdn.microsoft.com/library/system.net.servicepointmanager.defaultconnectionlimit(v=vs.110).aspx) en 100 para garantizar que se pueda establecer un gran número de conexiones simultáneas al ejecutar la aplicación.

Además de establecer la configuración del límite de subprocesos y de conexiones, la clase [BlobRequestOptions](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions) del método [UploadFromStreamAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.uploadfromstreamasync) está configurada para usar el paralelismo y deshabilitar la validación de hash MD5. Los archivos se cargan en bloques de 100 MB. Esta configuración proporciona un mejor rendimiento, pero puede aumentar el costo si se usa una red con un rendimiento deficiente, ya que si se produce un error, se reintenta todo el bloque de 100 MB.

|Propiedad|Value|Descripción|
|---|---|---|
|[ParallelOperationThreadCount](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.paralleloperationthreadcount)| 8| La configuración divide el blob en bloques al realizar la carga. Para conseguir el máximo rendimiento, este valor debe ser 8 veces el número de núcleos. |
|[DisableContentMD5Validation](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.disablecontentmd5validation)| true| Esta propiedad deshabilita la función de comprobación del hash MD5 del contenido cargado. Al deshabilitar la validación de MD5, se agiliza la transferencia. Aún así, hay que tener en cuenta que no se puede confirmar la validez o integridad de los archivos que se transfieren.   |
|[StoreBlobContentMD5](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.storeblobcontentmd5)| false| Esta propiedad determina si se calcula y almacena un hash MD5 con el archivo.   |
| [RetryPolicy](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.retrypolicy)| Retroceso de 2 segundos con 10 reintentos como máximo |Determina la directiva de reintentos de las solicitudes. Los errores de conexión se reintentan y, en este ejemplo, una directiva [ExponentialRetry](/dotnet/api/microsoft.azure.batch.common.exponentialretry) está configurada con un retroceso de 2 segundos y un número máximo de reintentos de 10. Esta configuración es importante cuando la aplicación se acerca a los objetivos de escalabilidad y rendimiento de Blob Storage. Para más información, consulte [Objetivos de escalabilidad y rendimiento de Blob Storage](../blobs/scalability-targets.md).  |

La tarea `UploadFilesAsync` se muestra en el ejemplo siguiente:

```csharp
private static async Task UploadFilesAsync()
{
    // Create random 5 characters containers to upload files to.
    CloudBlobContainer[] containers = await GetRandomContainersAsync();
    var currentdir = System.IO.Directory.GetCurrentDirectory();

    // path to the directory to upload
    string uploadPath = currentdir + "\\upload";
    Stopwatch time = Stopwatch.StartNew();
    try
    {
        Console.WriteLine("Iterating in directory: {0}", uploadPath);
        int count = 0;
        int max_outstanding = 100;
        int completed_count = 0;

        // Define the BlobRequestOptions on the upload.
        // This includes defining an exponential retry policy to ensure that failed connections are retried with a backoff policy. As multiple large files are being uploaded
        // large block sizes this can cause an issue if an exponential retry policy is not defined.  Additionally parallel operations are enabled with a thread count of 8
        // This could be should be multiple of the number of cores that the machine has. Lastly MD5 hash validation is disabled for this example, this improves the upload speed.
        BlobRequestOptions options = new BlobRequestOptions
        {
            ParallelOperationThreadCount = 8,
            DisableContentMD5Validation = true,
            StoreBlobContentMD5 = false
        };
        // Create a new instance of the SemaphoreSlim class to define the number of threads to use in the application.
        SemaphoreSlim sem = new SemaphoreSlim(max_outstanding, max_outstanding);

        List<Task> tasks = new List<Task>();
        Console.WriteLine("Found {0} file(s)", Directory.GetFiles(uploadPath).Count());

        // Iterate through the files
        foreach (string path in Directory.GetFiles(uploadPath))
        {
            // Create random file names and set the block size that is used for the upload.
            var container = containers[count % 5];
            string fileName = Path.GetFileName(path);
            Console.WriteLine("Uploading {0} to container {1}.", path, container.Name);
            CloudBlockBlob blockBlob = container.GetBlockBlobReference(fileName);

            // Set block size to 100MB.
            blockBlob.StreamWriteSizeInBytes = 100 * 1024 * 1024;
            await sem.WaitAsync();

            // Create tasks for each file that is uploaded. This is added to a collection that executes them all asyncronously.  
            tasks.Add(blockBlob.UploadFromFileAsync(path, null, options, null).ContinueWith((t) =>
            {
                sem.Release();
                Interlocked.Increment(ref completed_count);
            }));
            count++;
        }

        // Creates an asynchronous task that completes when all the uploads complete.
        await Task.WhenAll(tasks);

        time.Stop();

        Console.WriteLine("Upload has been completed in {0} seconds. Press any key to continue", time.Elapsed.TotalSeconds.ToString());

        Console.ReadLine();
    }
    catch (DirectoryNotFoundException ex)
    {
        Console.WriteLine("Error parsing files in the directory: {0}", ex.Message);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

El siguiente ejemplo es una salida de aplicación truncada que se ejecuta en un sistema Windows.

```
Created container https://mystorageaccount.blob.core.windows.net/9efa7ecb-2b24-49ff-8e5b-1d25e5481076
Created container https://mystorageaccount.blob.core.windows.net/bbe5f0c8-be9e-4fc3-bcbd-2092433dbf6b
Created container https://mystorageaccount.blob.core.windows.net/9ac2f71c-6b44-40e7-b7be-8519d3ba4e8f
Created container https://mystorageaccount.blob.core.windows.net/47646f1a-c498-40cd-9dae-840f46072180
Created container https://mystorageaccount.blob.core.windows.net/38b2cdab-45fa-4cf9-94e7-d533837365aa
Iterating in directory: D:\git\storage-dotnet-perf-scale-app\upload
Found 50 file(s)
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\1d596d16-f6de-4c4c-8058-50ebd8141e4d.txt to container 9efa7ecb-2b24-49ff-8e5b-1d25e5481076.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\242ff392-78be-41fb-b9d4-aee8152a6279.txt to container bbe5f0c8-be9e-4fc3-bcbd-2092433dbf6b.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\38d4d7e2-acb4-4efc-ba39-f9611d0d55ef.txt to container 9ac2f71c-6b44-40e7-b7be-8519d3ba4e8f.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\45930d63-b0d0-425f-a766-cda27ff00d32.txt to container 47646f1a-c498-40cd-9dae-840f46072180.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\5129b385-5781-43be-8bac-e2fbb7d2bd82.txt to container 38b2cdab-45fa-4cf9-94e7-d533837365aa.
...
Upload has been completed in 142.0429536 seconds. Press any key to continue
```

### <a name="validate-the-connections"></a>Validar las conexiones

Mientras se cargan los archivos, puede comprobar el número de conexiones simultáneas de su cuenta de almacenamiento. Abra un **símbolo del sistema** y escriba `netstat -a | find /c "blob:https"`. Este comando muestra el número de conexiones simultáneas que se abrieron mediante `netstat`. En el ejemplo siguiente se muestra un resultado similar a lo que verá cuando ejecute el tutorial usted mismo. Como puede ver en el ejemplo, más de 800 conexiones estaban abiertas al cargar los archivos aleatorios en la cuenta de almacenamiento. Este valor cambia durante la ejecución de la carga. Al realizar la carga en fragmentos de bloques paralelos, se reduce enormemente la cantidad de tiempo necesaria para transferir el contenido.

```
C:\>netstat -a | find /c "blob:https"
800

C:\>
```

## <a name="next-steps"></a>Pasos siguientes

En la segunda parte de la serie aprendió no solo cómo cargar grandes cantidades de datos aleatorios en una cuenta de almacenamiento en paralelo, si no que también aprendió lo siguiente:

> [!div class="checklist"]
> * Configurar la cadena de conexión
> * Compilar la aplicación
> * Ejecución de la aplicación
> * Validar el número de conexiones

Continúe en la tercera parte de la serie para descargar grandes cantidades de datos de una cuenta de almacenamiento.

> [!div class="nextstepaction"]
> [Descarga de grandes cantidades de datos aleatorios desde Azure Storage](storage-blob-scalable-app-download-files.md)

[previous-tutorial]: storage-blob-scalable-app-create-vm.md
