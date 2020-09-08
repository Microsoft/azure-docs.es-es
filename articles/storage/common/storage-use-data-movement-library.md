---
title: Transferencia de datos con la biblioteca de movimiento de datos para .NET
titleSuffix: Azure Storage
description: Use la biblioteca de movimiento de datos para realizar operaciones de movimiento o copia de datos con contenido de blobs y archivos. Copie datos a Azure Storage desde archivos locales o copie datos en o entre cuentas de almacenamiento. Migre fácilmente sus datos a Azure Storage.
services: storage
author: tamram
ms.service: storage
ms.devlang: dotnet
ms.topic: how-to
ms.date: 06/16/2020
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 8ec35a651d4d3ef9e0877463329a654bc7491f4c
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89001883"
---
# <a name="transfer-data-with-the-data-movement-library"></a>Transferencia de datos con la biblioteca de movimiento de datos

La biblioteca de movimiento de datos de Azure Storage es una biblioteca multiplataforma de código abierto que se ha diseñado para carga, descarga y copia de alto rendimiento de blobs y archivos. La biblioteca de movimiento de datos proporciona métodos útiles que no están disponibles en la biblioteca de cliente de Azure Storage para .NET. Estos métodos proporcionan la capacidad de establecer el número de operaciones paralelas, seguir el progreso de la transferencia, reanudar con facilidad una transferencia cancelada, etc.

Esta biblioteca también utiliza .NET Core, lo que significa que puede usarla para crear aplicaciones .NET para Windows, Linux y MacOS. Para más información sobre .NET Core, consulte la [documentación de .NET Core](https://dotnet.github.io/). Esta biblioteca también funciona para las aplicaciones de .NET Framework tradicionales para Windows.

En este documento se muestra cómo crear una aplicación de consola de .NET Core que se ejecuta en Windows, Linux y MacOS y se llevan a cabo los siguientes escenarios:

- Cargar archivos y directorios en Blob Storage.
- Definir el número de operaciones paralelas al transferir datos.
- Seguir el progreso de la transferencia de datos.
- Reanudar la transferencia de datos cancelada.
- Copiar el archivo desde la dirección URL en Blob Storage.
- Copiar de Blob Storage a Blob Storage.

## <a name="prerequisites"></a>Prerrequisitos

- [Visual Studio Code](https://code.visualstudio.com/)
- Una [cuenta de almacenamiento de Azure](storage-account-create.md)

## <a name="setup"></a>Configurar

1. Visite la [guía de instalación de .NET Core](https://www.microsoft.com/net/core) para instalar .NET Core. Cuando seleccione el entorno, elija la opción de línea de comandos.
2. Desde la línea de comandos, cree un directorio para el proyecto. Vaya a este directorio y escriba `dotnet new console -o <sample-project-name>` para crear un proyecto de consola en C#.
3. Abra este directorio en Visual Studio Code. Este paso puede realizarse rápidamente a través de la línea de comandos; para ello, escriba `code .` en Windows.
4. Instale la [extensión de C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) desde Visual Studio Code Marketplace. Reinicie Visual Studio Code.
5. Llegados a este punto, debe ver dos mensajes. Uno es para agregar "recursos necesarios para compilar y depurar". Haga clic "sí". El otro mensaje es para restaurar dependencias no resueltas. Haga clic en "restaurar".
6. Modifique `launch.json` de `.vscode` para usar un terminal externo como consola. Esta configuración debe leerse como `"console": "externalTerminal"`
7. Visual Studio Code permite depurar aplicaciones de .NET Core. Presione `F5` para ejecutar la aplicación y compruebe que la configuración funciona. Debe ver "Hello World!" impreso en la consola.

## <a name="add-the-data-movement-library-to-your-project"></a>Incorporación de la biblioteca de movimiento de datos al proyecto

1. Agregue la versión más reciente de la biblioteca de movimiento de datos a la sección `dependencies` del archivo `<project-name>.csproj`. En el momento de la escritura, esta versión sería `"Microsoft.Azure.Storage.DataMovement": "0.6.2"`.
2. Debe aparecer un mensaje para restaurar el proyecto. Haga clic en el botón "restaurar". También puede restaurar el proyecto desde la línea de comandos escribiendo el comando `dotnet restore` en la raíz del directorio del proyecto.

Modifique `<project-name>.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>netcoreapp2.0</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="Microsoft.Azure.Storage.DataMovement" Version="0.6.2" />
        </ItemGroup>
    </Project>
```

## <a name="set-up-the-skeleton-of-your-application"></a>Configuración del esqueleto de la aplicación

Lo primero que hacemos es configurar el código de "esqueleto" de la aplicación. Este código nos pide una clave de cuenta y un nombre de cuenta de Storage y usa esas credenciales para crear un objeto `CloudStorageAccount`. Este objeto se usa para interactuar con nuestra cuenta de Storage en todos los escenarios de transferencia. El código también nos pedirá que elijamos el tipo de operación de transferencia que se desea ejecutar.

Modifique `Program.cs`:

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Diagnostics;
using Microsoft.Azure.Storage;
using Microsoft.Azure.Storage.Blob;
using Microsoft.Azure.Storage.DataMovement;

namespace DMLibSample
{
    public class Program
    {
        public static void Main()
        {
            Console.WriteLine("Enter Storage account name:");
            string accountName = Console.ReadLine();

            Console.WriteLine("\nEnter Storage account key:");
            string accountKey = Console.ReadLine();

            string storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=" + accountName + ";AccountKey=" + accountKey;
            CloudStorageAccount account = CloudStorageAccount.Parse(storageConnectionString);

            ExecuteChoice(account);
        }

        public static void ExecuteChoice(CloudStorageAccount account)
        {
            Console.WriteLine("\nWhat type of transfer would you like to execute?\n1. Local file --> Azure Blob\n2. Local directory --> Azure Blob directory\n3. URL (e.g. Amazon S3 file) --> Azure Blob\n4. Azure Blob --> Azure Blob");
            int choice = int.Parse(Console.ReadLine());

            if(choice == 1)
            {
                TransferLocalFileToAzureBlob(account).Wait();
            }
            else if(choice == 2)
            {
                TransferLocalDirectoryToAzureBlobDirectory(account).Wait();
            }
            else if(choice == 3)
            {
                TransferUrlToAzureBlob(account).Wait();
            }
            else if(choice == 4)
            {
                TransferAzureBlobToAzureBlob(account).Wait();
            }
        }

        public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
        {

        }

        public static async Task TransferLocalDirectoryToAzureBlobDirectory(CloudStorageAccount account)
        {

        }

        public static async Task TransferUrlToAzureBlob(CloudStorageAccount account)
        {

        }

        public static async Task TransferAzureBlobToAzureBlob(CloudStorageAccount account)
        {

        }
    }
}
```

## <a name="upload-a-local-file-to-a-blob"></a>Carga de un archivo local a un blob

Agregue los métodos `GetSourcePath` y `GetBlob` a `Program.cs`:

```csharp
public static string GetSourcePath()
{
    Console.WriteLine("\nProvide path for source:");
    string sourcePath = Console.ReadLine();

    return sourcePath;
}

public static CloudBlockBlob GetBlob(CloudStorageAccount account)
{
    CloudBlobClient blobClient = account.CreateCloudBlobClient();

    Console.WriteLine("\nProvide name of Blob container:");
    string containerName = Console.ReadLine();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExistsAsync().Wait();

    Console.WriteLine("\nProvide name of new Blob:");
    string blobName = Console.ReadLine();
    CloudBlockBlob blob = container.GetBlockBlobReference(blobName);

    return blob;
}
```

Modifique el método `TransferLocalFileToAzureBlob`:

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    Console.WriteLine("\nTransfer started...");
    await TransferManager.UploadAsync(localFilePath, blob);
    Console.WriteLine("\nTransfer operation complete.");
    ExecuteChoice(account);
}
```

Este código nos pide la ruta de acceso a un archivo local, el nombre de un contenedor nuevo o existente y el nombre de un nuevo blob. El método `TransferManager.UploadAsync` realiza la carga con esta información.

Presione `F5` para ejecutar la aplicación. Puede comprobar que se ha producido la carga mediante la visualización de la cuenta de Storage con el [Explorador de Microsoft Azure Storage](https://storageexplorer.com/).

## <a name="set-the-number-of-parallel-operations"></a>Establecimiento del número de operaciones paralelas

Una característica que ofrece la biblioteca de movimiento de datos es la capacidad para establecer el número de operaciones paralelas para aumentar el rendimiento de la transferencia de datos. De forma predeterminada, la biblioteca de movimiento de datos establece el número de operaciones paralelas en 8 veces el número de núcleos de la máquina.

Tenga en cuenta que muchas operaciones paralelas en un entorno con poco ancho de banda puede saturar la conexión de red e impedir que las operaciones se completen. Debe experimentar con esta configuración para determinar qué funciona mejor basándose en el ancho de banda de red disponible.

Vamos a agregar código que nos permita establecer el número de operaciones paralelas. Vamos a agregar también código que cronometra el tiempo que tarda la transferencia en completarse.

Agregar un método `SetNumberOfParallelOperations` a `Program.cs`:

```csharp
public static void SetNumberOfParallelOperations()
{
    Console.WriteLine("\nHow many parallel operations would you like to use?");
    string parallelOperations = Console.ReadLine();
    TransferManager.Configurations.ParallelOperations = int.Parse(parallelOperations);
}
```

Modifique el método `ExecuteChoice` para usar `SetNumberOfParallelOperations`:

```csharp
public static void ExecuteChoice(CloudStorageAccount account)
{
    Console.WriteLine("\nWhat type of transfer would you like to execute?\n1. Local file --> Azure Blob\n2. Local directory --> Azure Blob directory\n3. URL (e.g. Amazon S3 file) --> Azure Blob\n4. Azure Blob --> Azure Blob");
    int choice = int.Parse(Console.ReadLine());

    SetNumberOfParallelOperations();

    if(choice == 1)
    {
        TransferLocalFileToAzureBlob(account).Wait();
    }
    else if(choice == 2)
    {
        TransferLocalDirectoryToAzureBlobDirectory(account).Wait();
    }
    else if(choice == 3)
    {
        TransferUrlToAzureBlob(account).Wait();
    }
    else if(choice == 4)
    {
        TransferAzureBlobToAzureBlob(account).Wait();
    }
}
```

Modifique el método `TransferLocalFileToAzureBlob` para usar un temporizador:

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    Console.WriteLine("\nTransfer started...");
    Stopwatch stopWatch = Stopwatch.StartNew();
    await TransferManager.UploadAsync(localFilePath, blob);
    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

## <a name="track-transfer-progress"></a>Seguimiento del progreso de transferencia

Saber cuánto tiempo tardaron los datos en transferirse resulta muy útil. Sin embargo, tener la posibilidad de ver el progreso de la transferencia *durante* la operación de transferencia sería incluso mejor. Para lograr este escenario, es necesario crear un objeto `TransferContext`. El objeto `TransferContext` se proporciona en dos formas: `SingleTransferContext` y `DirectoryTransferContext`. La primera de ellas es para transferir un archivo único y la segunda es para transferir un directorio de archivos.

Agregue los métodos `GetSingleTransferContext` y `GetDirectoryTransferContext` a `Program.cs`:

```csharp
public static SingleTransferContext GetSingleTransferContext(TransferCheckpoint checkpoint)
{
    SingleTransferContext context = new SingleTransferContext(checkpoint);

    context.ProgressHandler = new Progress<TransferStatus>((progress) =>
    {
        Console.Write("\rBytes transferred: {0}", progress.BytesTransferred );
    });

    return context;
}

public static DirectoryTransferContext GetDirectoryTransferContext(TransferCheckpoint checkpoint)
{
    DirectoryTransferContext context = new DirectoryTransferContext(checkpoint);

    context.ProgressHandler = new Progress<TransferStatus>((progress) =>
    {
        Console.Write("\rBytes transferred: {0}", progress.BytesTransferred );
    });

    return context;
}
```

Modifique el método `TransferLocalFileToAzureBlob` para usar `GetSingleTransferContext`:

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    Console.WriteLine("\nTransfer started...\n");
    Stopwatch stopWatch = Stopwatch.StartNew();
    await TransferManager.UploadAsync(localFilePath, blob, null, context);
    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

## <a name="resume-a-canceled-transfer"></a>Reanudación de una transferencia cancelada

Otra característica muy útil que ofrece la biblioteca de movimiento de datos es la capacidad de reanudar una transferencia cancelada. Vamos a agregar código que nos permite cancelar la transferencia temporalmente escribiendo `c` y luego reanudar dicha transferencia 3 segundos más tarde.

Modifique `TransferLocalFileToAzureBlob`:

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.UploadAsync(localFilePath, blob, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.UploadAsync(localFilePath, blob, null, context);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

Hasta ahora, nuestro valor de `checkpoint` siempre se ha establecido en `null`. Ahora, si se cancela la transferencia, recuperaremos el último punto de comprobación de dicha transferencia y luego utilizaremos este nuevo punto de comprobación en nuestro contexto de la transferencia.

## <a name="transfer-a-local-directory-to-blob-storage"></a>Transferencia de un directorio local a Blob Storage

Sería decepcionante que la biblioteca de movimiento de datos solo pudiera transferir un archivo a la vez. Afortunadamente, no es el caso. La biblioteca de movimiento de datos proporciona la capacidad de transferir un directorio de archivos y todos sus subdirectorios. Vamos a agregar código que nos permita hacer exactamente eso.

En primer lugar, agregue el método `GetBlobDirectory` a `Program.cs`:

```csharp
public static CloudBlobDirectory GetBlobDirectory(CloudStorageAccount account)
{
    CloudBlobClient blobClient = account.CreateCloudBlobClient();

    Console.WriteLine("\nProvide name of Blob container. This can be a new or existing Blob container:");
    string containerName = Console.ReadLine();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExistsAsync().Wait();

    CloudBlobDirectory blobDirectory = container.GetDirectoryReference("");

    return blobDirectory;
}
```

A continuación, modifique `TransferLocalDirectoryToAzureBlobDirectory`:

```csharp
public static async Task TransferLocalDirectoryToAzureBlobDirectory(CloudStorageAccount account)
{
    string localDirectoryPath = GetSourcePath();
    CloudBlobDirectory blobDirectory = GetBlobDirectory(account);
    TransferCheckpoint checkpoint = null;
    DirectoryTransferContext context = GetDirectoryTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    UploadDirectoryOptions options = new UploadDirectoryOptions()
    {
        Recursive = true
    };

    try
    {
        task = TransferManager.UploadDirectoryAsync(localDirectoryPath, blobDirectory, options, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetDirectoryTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.UploadDirectoryAsync(localDirectoryPath, blobDirectory, options, context);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

Hay algunas diferencias entre este método y el método para cargar un único archivo. Ahora estamos usando `TransferManager.UploadDirectoryAsync` y el método `getDirectoryTransferContext` que hemos creado anteriormente. Además, ahora proporcionamos un valor de `options` a nuestra operación de carga, lo que nos permite indicar que deseamos incluir los subdirectorios en nuestra carga.

## <a name="copy-a-file-from-url-to-a-blob"></a>Copia de un archivo desde una dirección URL a un blob

Ahora, vamos a agregar código que nos permite copiar un archivo desde una dirección URL a un blob de Azure.

Modifique `TransferUrlToAzureBlob`:

```csharp
public static async Task TransferUrlToAzureBlob(CloudStorageAccount account)
{
    Uri uri = new Uri(GetSourcePath());
    CloudBlockBlob blob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.CopyAsync(uri, blob, true, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.CopyAsync(uri, blob, true, null, context, cancellationSource.Token);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

Un caso de uso importante de esta característica es cuando necesita mover datos de otro servicio en la nube (p. ej., AWS) a Azure. Siempre que tengan una dirección URL que le proporcione acceso al recurso, podrá mover fácilmente ese recurso a blobs de Azure mediante el uso del método `TransferManager.CopyAsync`. Este método también introduce un nuevo parámetro booleano. El establecimiento de este parámetro en `true` indica que se desea hacer una copia asincrónica del lado servidor. El establecimiento de este parámetro en `false` indica una copia sincrónica, lo que significa que el recurso se descarga primero en el equipo local y después se carga en el blob de Azure. Sin embargo, la copia asincrónica solo está disponible actualmente para copiar desde un recurso de Azure Storage a otro.

## <a name="copy-a-blob"></a>Copia de un blob

Otra característica que proporciona de forma única la biblioteca de movimiento de datos es la capacidad de copiar desde un recurso de Azure Storage a otro.

Modifique `TransferAzureBlobToAzureBlob`:

```csharp
public static async Task TransferAzureBlobToAzureBlob(CloudStorageAccount account)
{
    CloudBlockBlob sourceBlob = GetBlob(account);
    CloudBlockBlob destinationBlob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.CopyAsync(sourceBlob, destinationBlob, CopyMethod.ServiceSideAsyncCopy, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.CopyAsync(sourceBlob, destinationBlob, false, null, context, cancellationSource.Token);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

En este ejemplo, se establece el parámetro booleano de `TransferManager.CopyAsync` en `false` para indicar que desea hacer una copia sincrónica. Esto significa que el recurso se descarga primero en la máquina local y después se carga en el blob de Azure. La opción de copia sincrónica es una excelente manera de asegurarse de que la operación de copia tiene una velocidad constante. En cambio, la velocidad de una copia asincrónica del lado servidor depende del ancho de banda de red disponible en el servidor, el cual puede fluctuar. Sin embargo, la copia sincrónica puede generar un costo de salida adicional en comparación con la copia asincrónica. El enfoque recomendado es usar una copia asincrónica en una VM de Azure que se encuentre en la misma región que la cuenta de almacenamiento de origen, para evitar el costo de salida.

La aplicación de movimiento de datos ahora está completa. [El ejemplo de código completo está disponible en GitHub](https://github.com/azure-samples/storage-dotnet-data-movement-library-app).

## <a name="next-steps"></a>Pasos siguientes

[Documentación de referencia de la biblioteca de movimiento de datos de Azure Storage](https://azure.github.io/azure-storage-net-data-movement).

[!INCLUDE [storage-try-azure-tools-blobs](../../../includes/storage-try-azure-tools-blobs.md)]
