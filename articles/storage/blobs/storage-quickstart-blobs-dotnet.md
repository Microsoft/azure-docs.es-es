---
title: 'Inicio rápido: Uso de .NET para crear un blob en el almacenamiento de objetos: Azure Storage'
description: En esta guía de inicio rápido, obtendrá información sobre cómo usar la biblioteca cliente de Azure Storage para .NET para crear un contenedor y un blob en Blob Storage (objeto). A continuación, aprenderá a descargar el blob en un equipo local y a enumerar todos los blobs en un contenedor.
services: storage
author: mhopkins-msft
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 05/09/2019
ms.author: mhopkins
ms.reviewer: seguler
ms.openlocfilehash: 2728ac93abd18f929dc6e82cfb471ce33fe8866f
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65508022"
---
# <a name="quickstart-use-net-to-create-a-blob-in-object-storage"></a>Inicio rápido: Uso de .NET para crear un blob en el almacenamiento de objetos

En esta guía de inicio rápido, obtendrá información sobre cómo usar la biblioteca cliente de Azure Storage para .NET para crear un contenedor y un blob en Blob Storage (objeto). A continuación, aprenderá a descargar el blob en un equipo local y a enumerar todos los blobs en un contenedor.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

A continuación, descargue e instale .NET Core 2.1 para su sistema operativo. Si usa Windows, puede instalar Visual Studio y utilizar .NET Framework si lo prefiere. También puede elegir instalar un editor y usarlo con el sistema operativo.

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

- Instale [.NET Core para Windows](https://www.microsoft.com/net/download/windows) o [.NET Framework](https://www.microsoft.com/net/download/windows) (se incluye con Visual Studio para Windows).
- Instale [Visual Studio para Windows](https://www.visualstudio.com/). Si usa .NET Core, la instalación de Visual Studio es opcional.  

Para obtener información acerca de cuándo elegir .NET Core o .NET Framework, consulte [Selección entre .NET Core y .NET Framework para aplicaciones de servidor](https://docs.microsoft.com/dotnet/standard/choosing-core-framework-server).

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

- Instale [.NET Core para Linux](https://www.microsoft.com/net/download/linux)
- Opcionalmente, instale [Visual Studio Code](https://www.visualstudio.com/) y la [extensión de C#](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp&dotnetid=963890049.1518206068)

# <a name="macostabmacos"></a>[macOS](#tab/macos)

- Instale [.NET Core para macOS](https://www.microsoft.com/net/download/macos).
- Opcionalmente, instale [Visual Studio para Mac](https://www.visualstudio.com/vs/visual-studio-mac/)

---

## <a name="download-the-sample-application"></a>Descarga de la aplicación de ejemplo

La aplicación de ejemplo utilizada en este tutorial de inicio rápido es una aplicación de consola básica. Puede explorar la aplicación de ejemplo en [GitHub](https://github.com/Azure-Samples/storage-blobs-dotnet-quickstart).

Use [git](https://git-scm.com/) para descargar una copia de la aplicación en su entorno de desarrollo.

```bash
git clone https://github.com/Azure-Samples/storage-blobs-dotnet-quickstart.git
```

Este comando clona el repositorio en la carpeta git local. Para abrir la solución de Visual Studio, busque la carpeta *storage-blobs-dotnet-quickstart*, ábrala y haga doble clic en *storage-blobs-dotnet-quickstart.sln*.

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Configuración de la cadena de conexión de almacenamiento.

Para ejecutar la aplicación, debe proporcionar la cadena de conexión de la cuenta de almacenamiento. La aplicación de ejemplo lee la cadena de conexión de una variable de entorno y la utiliza para autorizar las solicitudes en Azure Storage.

Una vez que haya copiado la cadena de conexión, escríbala en una variable de entorno nueva en la máquina local que ejecuta la aplicación. Para establecer la variable de entorno, abra una ventana de consola y siga las instrucciones de su sistema operativo. Reemplace `<yourconnectionstring>` por la cadena de conexión real:

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

```cmd
setx storageconnectionstring "<yourconnectionstring>"
```

Después de agregar la variable de entorno, puede que tenga que reiniciar todos los programas en ejecución que necesiten leer la variable de entorno, incluida la ventana de consola. Por ejemplo, si usa Visual Studio como editor, reinícielo antes de ejecutar el ejemplo.

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

```bash
export storageconnectionstring=<yourconnectionstring>
```

Después de agregar la variable de entorno, ejecute `source ~/.bashrc` desde la ventana de consola para que los cambios surtan efecto.

# <a name="macostabmacos"></a>[macOS](#tab/macos)

Edite .bash_profile y agregue la variable de entorno:

```bash
export storageconnectionstring=<yourconnectionstring>
```

Después de agregar la variable de entorno, ejecute `source .bash_profile` desde la ventana de consola para que los cambios surtan efecto.

---

## <a name="run-the-sample"></a>Ejecución del ejemplo

Este ejemplo crea un archivo de prueba en la carpeta **Mis documentos** local y lo carga en Blob Storage. Después, en el ejemplo se enumeran los blobs del contenedor y descarga el archivo con un nombre nuevo para que pueda comparar los archivos antiguo y nuevo.

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

Si utiliza Visual Studio como editor, puede presionar **F5** para ejecutarlo.

Otra opción es ir al directorio de la aplicación y ejecutar la aplicación con el comando `dotnet run`.

```console
dotnet run
```

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

Desplácese hasta el directorio de la aplicación y ejecute la aplicación con el comando `dotnet run`.

```console
dotnet run
```

# <a name="macostabmacos"></a>[macOS](#tab/macos)

Desplácese hasta el directorio de la aplicación y ejecute la aplicación con el comando `dotnet run`.

```console
dotnet run
```

---

La salida de la aplicación de ejemplo es similar a la siguiente:

```output
Azure Blob storage - .NET Quickstart sample

Created container 'quickstartblobs33c90d2a-eabd-4236-958b-5cc5949e731f'

Temp file = C:\Users\myusername\Documents\QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt
Uploading to Blob storage as blob 'QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt'

Listing blobs in container.
https://storagesamples.blob.core.windows.net/quickstartblobs33c90d2a-eabd-4236-958b-5cc5949e731f/QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt

Downloading blob to C:\Users\myusername\Documents\QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db_DOWNLOADED.txt

Press any key to delete the sample files and example container.
```

Al presionar la tecla **Entrar**, la aplicación elimina el contenedor de almacenamiento y los archivos. Antes de eliminarlos, compruebe si la carpeta **Mis documentos** contiene los dos archivos. Puede abrirlos y ver que son idénticos. Copie la dirección URL del blob de la ventana de la consola y péguela en un explorador para ver el contenido del blob.

Después de haber comprobado los archivos, presione cualquier tecla para finalizar la demostración y eliminar los archivos de prueba. Ahora que sabe lo que hace el ejemplo, abra el archivo Program.cs para examinar el código.

## <a name="understand-the-sample-code"></a>Descripción del código de ejemplo

Después, explore el código de ejemplo para poder saber cómo funciona.

### <a name="try-parsing-the-connection-string"></a>Pruebe a analizar la cadena de conexión

Lo primero que el ejemplo realiza es comprobar que la variable de entorno contiene una cadena de conexión que se puede analizar para crear un objeto [CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount) que señala a la cuenta de almacenamiento. Para comprobar que la cadena de conexión es válida, use el método [TryParse](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount.tryparse). Si **TryParse** funciona correctamente, inicializa la variable *storageAccount* y devuelve el valor **true**.

```csharp
// Retrieve the connection string for use with the application. The storage connection string is stored
// in an environment variable on the machine running the application called storageconnectionstring.
// If the environment variable is created after the application is launched in a console or with Visual
// Studio, the shell or application needs to be closed and reloaded to take the environment variable into account.
string storageConnectionString = Environment.GetEnvironmentVariable("storageconnectionstring");

// Check whether the connection string can be parsed.
CloudStorageAccount storageAccount;
if (CloudStorageAccount.TryParse(storageConnectionString, out storageAccount))
{
    // If the connection string is valid, proceed with operations against Blob storage here.
    ...
}
else
{
    // Otherwise, let the user know that they need to define the environment variable.
    Console.WriteLine(
        "A connection string has not been defined in the system environment variables. " +
        "Add an environment variable named 'storageconnectionstring' with your storage " +
        "connection string as a value.");
    Console.WriteLine("Press any key to exit the sample application.");
    Console.ReadLine();
}
```

### <a name="create-the-container-and-set-permissions"></a>Cree el contenedor y establezca los permisos

Después, el ejemplo crea un contenedor y establece sus permisos para que los blobs del contenedor sean públicos. Si un blob es público, cualquier cliente puede acceder de forma anónima.

Para crear el contenedor, en primer lugar cree una instancia del objeto [CloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient) que apunte a Blob Storage en la cuenta de almacenamiento. Cree una instancia del objeto [CloudBlobContainer](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer) y, después, cree el contenedor.

En este caso, el ejemplo llama al método [CreateAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createasync) para crear el contenedor. Un valor de GUID se anexa al nombre de contenedor para asegurarse de que sea único. En un entorno de producción, a menudo es preferible utilizar el método [CreateIfNotExistsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createifnotexistsasync) para crear un contenedor solo si no existe y para evitar conflictos de nomenclatura.

> [!IMPORTANT]
> Los nombres de contenedor deben estar en minúsculas. Para más información acerca de los contenedores de nomenclatura y los blobs, consulte [Naming and Referencing Containers, Blobs, and Metadata](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Asignación de nombres y realización de referencias a contenedores, blobs y metadatos).

```csharp
// Create the CloudBlobClient that represents the Blob storage endpoint for the storage account.
CloudBlobClient cloudBlobClient = storageAccount.CreateCloudBlobClient();

// Create a container called 'quickstartblobs' and append a GUID value to it to make the name unique.
CloudBlobContainer cloudBlobContainer = cloudBlobClient.GetContainerReference("quickstartblobs" + Guid.NewGuid().ToString());
await cloudBlobContainer.CreateAsync();

// Set the permissions so the blobs are public.
BlobContainerPermissions permissions = new BlobContainerPermissions
{
    PublicAccess = BlobContainerPublicAccessType.Blob
};
await cloudBlobContainer.SetPermissionsAsync(permissions);
```

### <a name="upload-blobs-to-the-container"></a>Carga de blobs al contenedor

Después, el ejemplo carga un archivo local en un blob en bloques. El código de ejemplo obtiene una referencia a un objeto **CloudBlockBlob** mediante una llamada al método [GetBlockBlobReference](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.getblockblobreference) en el contenedor creado en la sección anterior. Luego carga el archivo seleccionado en el blob mediante una llamada al método [UploadFromFileAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.uploadfromfileasync). Esta operación crea el blob si no existe y lo sobrescribe, en caso de que ya exista.

```csharp
// Create a file in your local MyDocuments folder to upload to a blob.
string localPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
string localFileName = "QuickStart_" + Guid.NewGuid().ToString() + ".txt";
sourceFile = Path.Combine(localPath, localFileName);
// Write text to the file.
File.WriteAllText(sourceFile, "Hello, World!");

Console.WriteLine("Temp file = {0}", sourceFile);
Console.WriteLine("Uploading to Blob storage as blob '{0}'", localFileName);

// Get a reference to the blob address, then upload the file to the blob.
// Use the value of localFileName for the blob name.
CloudBlockBlob cloudBlockBlob = cloudBlobContainer.GetBlockBlobReference(localFileName);
await cloudBlockBlob.UploadFromFileAsync(sourceFile);
```

### <a name="list-the-blobs-in-a-container"></a>Enumerar los blobs de un contenedor

El ejemplo enumera los blobs del contenedor mediante el método [ListBlobsSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmentedasync). En el caso del ejemplo, solo se ha agregado un blob al contenedor, por lo que la operación de enumeración devuelve simplemente dicho blob.

Si hay demasiados blobs para devolverlos en una sola llamada (de manera predeterminada más de 5000), el método **ListBlobsSegmentedAsync** devuelve un segmento del conjunto total de resultados y un token de continuación. Para recuperar el siguiente segmento de blobs, proporciónelo en el token de continuación devuelto por la llamada anterior, etc. hasta que el token de continuación es nulo. Un token de continuación nulo indica que se han recuperado todos los blobs. El código de ejemplo muestra cómo utilizar el token de continuación para los procedimientos recomendados.

```csharp
// List the blobs in the container.
Console.WriteLine("List blobs in container.");
BlobContinuationToken blobContinuationToken = null;
do
{
    var results = await cloudBlobContainer.ListBlobsSegmentedAsync(null, blobContinuationToken);
    // Get the value of the continuation token returned by the listing call.
    blobContinuationToken = results.ContinuationToken;
    foreach (IListBlobItem item in results.Results)
    {
        Console.WriteLine(item.Uri);
    }
} while (blobContinuationToken != null); // Loop while the continuation token is not null.

```

### <a name="download-blobs"></a>Descargar blobs

A continuación, el ejemplo descarga el blob creado anteriormente en el sistema de archivos local mediante el método [DownloadToFileAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.downloadtofileasync). El código de ejemplo agrega el sufijo "_DOWNLOADED" al nombre del blob para que pueda ver ambos archivos en el sistema de archivos local.

```csharp
// Download the blob to a local file, using the reference created earlier.
// Append the string "_DOWNLOADED" before the .txt extension so that you can see both files in MyDocuments.
destinationFile = sourceFile.Replace(".txt", "_DOWNLOADED.txt");
Console.WriteLine("Downloading blob to {0}", destinationFile);
await cloudBlockBlob.DownloadToFileAsync(destinationFile, FileMode.Create);  
```

### <a name="clean-up-resources"></a>Limpieza de recursos

El ejemplo limpia los recursos que creó mediante la eliminación de todo el contenedor con [CloudBlobContainer.DeleteAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteasync). Si lo desea, también puede eliminar los archivos locales.

```csharp
Console.WriteLine("Press the 'Enter' key to delete the sample files, example container, and exit the application.");
Console.ReadLine();
// Clean up resources. This includes the container and the two temp files.
Console.WriteLine("Deleting the container");
if (cloudBlobContainer != null)
{
    await cloudBlobContainer.DeleteIfExistsAsync();
}
Console.WriteLine("Deleting the source, and downloaded files");
File.Delete(sourceFile);
File.Delete(destinationFile);
```

## <a name="resources-for-developing-net-applications-with-blobs"></a>Recursos para el desarrollo de aplicaciones .NET con blobs

Consulte estos recursos adicionales para el desarrollo de .NET con Blob Storage:

### <a name="binaries-and-source-code"></a>Archivos binarios y código fuente

- Descargue el paquete NuGet para la versión más reciente de la [biblioteca de cliente de .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/) para Azure Blob Storage.
- Consulte el [código fuente del SDK de Microsoft Azure Storage Blob para .NET](https://github.com/Azure/azure-storage-net/tree/master/Blob) en GitHub.

### <a name="client-library-reference-and-samples"></a>Referencia a la biblioteca de clientes y ejemplos

- Para más información acerca de la biblioteca de cliente de .NET, consulte la [referencia de la API de .NET](https://docs.microsoft.com/dotnet/api/overview/azure/storage).
- Explore los [ejemplos de Blob Storage](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=dotnet&term=blob) escritos mediante la biblioteca de cliente de .NET.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha aprendido a cargar, descargar y enumerar blobs mediante .NET.

Para aprender a crear una aplicación web que cargue una imagen en Blob Storage, diríjase a:

> [!div class="nextstepaction"]
> [Carga y procesamiento de una imagen](storage-upload-process-images.md).

- Para más información sobre .NET Core, consulte [Get started with .NET in 10 minutes](https://www.microsoft.com/net/learn/get-started/) (Introducción a .NET en 10 minutos).
- Para explorar una aplicación de ejemplo que puede implementar desde Visual Studio para Windows, consulte el artículo [.NET Photo Gallery Web Application Sample with Azure Blob Storage](https://azure.microsoft.com/resources/samples/storage-blobs-dotnet-webapp/) (Ejemplo de aplicación web de galería fotográfica .NET con Azure Blob Storage).
