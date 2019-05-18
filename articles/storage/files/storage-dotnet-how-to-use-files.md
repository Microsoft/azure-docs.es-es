---
title: Desarrollo para Azure Files con .NET | Microsoft Docs
description: Aprenda a desarrollar aplicaciones y servicios .NET que usen Azure Files para almacenar datos de archivos.
services: storage
author: roygara
ms.service: storage
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/22/2017
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 38bafdb4753b41a9c8acd599e6b7215e1777c6cd
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65779476"
---
# <a name="develop-for-azure-files-with-net"></a>Desarrollo para Azure Files con .NET

[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

En este tutorial se muestran los aspectos básicos del uso de .NET para desarrollar aplicaciones que usan [Azure Files](storage-files-introduction.md) para almacenar datos de archivos. En este tutorial, se crea una aplicación de consola simple para realizar acciones básicas con .NET y Azure Files:

* Obtener el contenido de un archivo
* Establezca la cuota (tamaño máximo) para el recurso compartido de archivos.
* Crear una firma de acceso compartido (clave SAS) para un archivo que utiliza una directiva de acceso compartido definida en el recurso compartido.
* Copie un archivo en otro en la misma cuenta de almacenamiento.
* Copie un archivo en un blob en la misma cuenta de almacenamiento.
* Uso de las métricas de Azure Storage para solucionar problemas

Para más información sobre Azure Files, consulte la [Introducción a Azure Files](storage-files-introduction.md).

[!INCLUDE [storage-check-out-samples-dotnet](../../../includes/storage-check-out-samples-dotnet.md)]

## <a name="understanding-the-net-apis"></a>Descripción de las API de .NET

Azure Files proporciona dos amplios enfoques para las aplicaciones cliente: bloque de mensajes del servidor (SMB) y REST. Dentro de. NET, estos métodos se extraen por las API `System.IO` y `WindowsAzure.Storage`.

API | Cuándo se deben usar | Notas
----|-------------|------
[System.IO](https://docs.microsoft.com/dotnet/api/system.io) | Su aplicación: <ul><li>Es necesario leer o escribir archivos a través de SMB</li><li>Se ejecuta en un dispositivo que tenga acceso a través del puerto 445 a su cuenta de Azure Files</li><li>No es necesario administrar cualquiera de las opciones administrativas del recurso compartido de archivos</li></ul> | El archivo de codificación I/O de Azure Files a través de SMB normalmente es igual que la codificación I/O con cualquier recurso compartido de red o dispositivo de almacenamiento local. Vea [este tutorial](https://docs.microsoft.com/dotnet/csharp/tutorials/console-teleprompter) para obtener una introducción a una serie de características en .NET, incluido el archivo I/O.
[Microsoft.Azure.Storage.File](https://docs.microsoft.com/dotnet/api/overview/azure/storage#client-library) | Su aplicación: <ul><li>No se puede tener acceso a Azure Files a través de SMB en el puerto 445 debido a restricciones de ISP o firewall</li><li>Requiere funcionalidad administrativa, como la capacidad de establecer la cuota de un recurso compartido de archivo o crear una firma de acceso compartido</li></ul> | Este artículo muestra el uso de `Microsoft.Azure.Storage.File` para el archivo I/O con REST (en lugar de SMB) y la administración del recurso compartido de archivos.

## <a name="create-the-console-application-and-obtain-the-assembly"></a>Creación de la aplicación de consola y obtención del ensamblado
En Visual Studio, cree una nueva aplicación de consola de Windows. Los pasos siguientes muestran cómo crear una aplicación de consola en Visual Studio 2017; sin embargo, en otras versiones de Visual Studio los pasos son similares.

1. Seleccione **Archivo** > **Nuevo** > **Proyecto**
2. Seleccione **Instalado** > **Plantillas** > **Visual C#** > **Escritorio clásico de Windows**
3. Seleccione **Aplicación de consola (.NET Framework)**
4. Escriba el nombre de la aplicación en el campo **Nombre:**.
5. Seleccione **Aceptar**.

Todos los ejemplos de código de este tutorial se pueden agregar al método `Main()` del archivo `Program.cs` de la aplicación de consola.

Puede usar la biblioteca de cliente de Azure Storage en cualquier tipo de aplicación. NET, incluida una aplicación web o servicio de nube de Azure y aplicaciones de escritorio y móviles. En esta guía, usamos una aplicación de consola para hacerlo más sencillo.

## <a name="use-nuget-to-install-the-required-packages"></a>Uso de NuGet para instalar los paquetes necesarios
Para completar este tutorial, es preciso que haga referencia a dos paquetes en el proyecto:

* [Biblioteca común de Microsoft Azure Storage para .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/): Este paquete proporciona acceso mediante programación a los recursos comunes en la cuenta de almacenamiento.
* [Biblioteca de Microsoft Azure Storage Blob para .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/): Este paquete proporciona acceso mediante programación a los recursos de Blob en la cuenta de almacenamiento.
* [Biblioteca del Administrador de configuración de Microsoft Azure para .NET](https://www.nuget.org/packages/Microsoft.Azure.ConfigurationManager/): Este paquete proporciona una clase para analizar una cadena de conexión en un archivo de configuración, independientemente del lugar en que se ejecute la aplicación.

Puede usar NuGet para obtener ambos paquetes. Siga estos pasos:

1. Haga clic con el botón derecho en el proyecto, en el **Explorador de soluciones**, y elija **Administrar paquetes NuGet**.
2. Busque "WindowsAzure.Storage" en línea y haga clic en **Instalar** para instalar el paquete Biblioteca del cliente de Storage y sus dependencias.
3. Busque "WindowsAzure.ConfigurationManager" en línea y haga clic en **Instalar** para instalar Azure Configuration Manager.

## <a name="save-your-storage-account-credentials-to-the-appconfig-file"></a>Guardar las credenciales de la cuenta de almacenamiento en el archivo app.config
A continuación, guardará las credenciales en el archivo app.config del proyecto. Edite el archivo app.config de forma que su aspecto sea similar al del siguiente ejemplo, pero reemplace `myaccount` por el nombre de su cuenta de almacenamiento y `mykey` por la clave de su cuenta de almacenamiento.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <startup>
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
    </startup>
    <appSettings>
        <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=StorageAccountKeyEndingIn==" />
    </appSettings>
</configuration>
```

> [!NOTE]
> La versión más reciente del Emulador de Azure Storage no es compatible con Azure Files. La cadena de conexión debe hacer referencia a una cuenta de Azure Storage en la nube para trabajar con Azure Files.

## <a name="add-using-directives"></a>Adición de directivas using
Abra el archivo `Program.cs` desde el Explorador de soluciones y agregue las siguientes directivas using al principio del archivo.

```csharp
using Microsoft.Azure; // Namespace for Azure Configuration Manager
using Microsoft.Azure.Storage; // Namespace for Storage Client Library
using Microsoft.Azure.Storage.Blob; // Namespace for Azure Blobs
using Microsoft.Azure.Storage.File; // Namespace for Azure Files
```

[!INCLUDE [storage-cloud-configuration-manager-include](../../../includes/storage-cloud-configuration-manager-include.md)]

## <a name="access-the-file-share-programmatically"></a>Obtener acceso al recurso compartido de archivos mediante programación
A continuación, agregue el siguiente código al método `Main()` (después del código mostrado anteriormente) para recuperar la cadena de conexión. Este código obtiene una referencia al archivo que creamos anteriormente y envía su contenido a la ventana de consola.

```csharp
// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();

    // Get a reference to the directory we created previously.
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

    // Ensure that the directory exists.
    if (sampleDir.Exists())
    {
        // Get a reference to the file we created previously.
        CloudFile file = sampleDir.GetFileReference("Log1.txt");

        // Ensure that the file exists.
        if (file.Exists())
        {
            // Write the contents of the file to the console window.
            Console.WriteLine(file.DownloadTextAsync().Result);
        }
    }
}
```

Ejecute la aplicación de consola para ver la salida.

## <a name="set-the-maximum-size-for-a-file-share"></a>Establecer el tamaño máximo para un recurso compartido de archivos
A partir de la versión 5.x de la Biblioteca de cliente de Azure Storage se puede establecer la cuota (o el tamaño máximo) de un recurso compartido de archivos, en gigabytes. También puede comprobar cuántos datos se almacenan actualmente en el recurso compartido.

Al establecer la cuota para un recurso compartido, puede limitar el tamaño total de los archivos almacenados en el recurso compartido. Si el tamaño total de archivos del recurso compartido supera la cuota establecida en el recurso compartido, los clientes no podrán aumentar el tamaño de los archivos existentes ni crear nuevos archivos, a menos que estén vacíos.

En el ejemplo siguiente se muestra cómo comprobar el uso actual de un recurso compartido y cómo establecer la cuota para el recurso compartido.

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Check current usage stats for the share.
    // Note that the ShareStats object is part of the protocol layer for the File service.
    Microsoft.Azure.Storage.File.Protocol.ShareStats stats = share.GetStats();
    Console.WriteLine("Current share usage: {0} GB", stats.Usage.ToString());

    // Specify the maximum size of the share, in GB.
    // This line sets the quota to be 10 GB greater than the current usage of the share.
    share.Properties.Quota = 10 + stats.Usage;
    share.SetProperties();

    // Now check the quota for the share. Call FetchAttributes() to populate the share's properties.
    share.FetchAttributes();
    Console.WriteLine("Current share quota: {0} GB", share.Properties.Quota);
}
```

### <a name="generate-a-shared-access-signature-for-a-file-or-file-share"></a>Generar una firma de acceso compartido para un archivo o recurso compartido de archivos
A partir de la versión 5.x de la biblioteca de cliente de Azure Storage, puede generar una firma de acceso compartido (SAS) para un recurso compartido de archivos o para un archivo individual. También puede crear una directiva de acceso compartido en un recurso compartido de archivos para administrar firmas de acceso compartido. Se recomienda la creación de una directiva de acceso compartido, ya que ofrece un medio de revocar la SAS si esta se encuentra en peligro.

En el ejemplo siguiente se crea una directiva de acceso compartido en un recurso compartido y luego se usa esa directiva para ofrecer las restricciones para una SAS en un archivo del recurso compartido.

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    string policyName = "sampleSharePolicy" + DateTime.UtcNow.Ticks;

    // Create a new shared access policy and define its constraints.
    SharedAccessFilePolicy sharedPolicy = new SharedAccessFilePolicy()
        {
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessFilePermissions.Read | SharedAccessFilePermissions.Write
        };

    // Get existing permissions for the share.
    FileSharePermissions permissions = share.GetPermissions();

    // Add the shared access policy to the share's policies. Note that each policy must have a unique name.
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    share.SetPermissions(permissions);

    // Generate a SAS for a file in the share and associate this access policy with it.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");
    CloudFile file = sampleDir.GetFileReference("Log1.txt");
    string sasToken = file.GetSharedAccessSignature(null, policyName);
    Uri fileSasUri = new Uri(file.StorageUri.PrimaryUri.ToString() + sasToken);

    // Create a new CloudFile object from the SAS, and write some text to the file.
    CloudFile fileSas = new CloudFile(fileSasUri);
    fileSas.UploadText("This write operation is authorized via SAS.");
    Console.WriteLine(fileSas.DownloadText());
}
```

Para más información sobre la creación y el uso de firmas de acceso compartido, consulte [Uso de firmas de acceso compartido (SAS)](../common/storage-dotnet-shared-access-signature-part-1.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

## <a name="copy-files"></a>Copiar archivos
A partir de la versión 5.x de la biblioteca de cliente de Azure Storage, puede copiar un archivo en otro, un archivo en un blob o un blob en un archivo. En las siguientes secciones, mostramos cómo realizar estas operaciones de copia mediante programación.

También puede usar AzCopy para copiar un archivo en otro o para copiar un blob en un archivo o viceversa. Consulte [Transferencia de datos con la utilidad en línea de comandos AzCopy](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

> [!NOTE]
> Si va a copiar un blob en un archivo o un archivo en un blob, debe usar una firma de acceso compartido (SAS) para autorizar el acceso al objeto de origen, incluso si está copiando en la misma cuenta de almacenamiento.
> 
> 

**Copiar un archivo en otro archivo** en el ejemplo siguiente se copia un archivo en otro archivo en el mismo recurso compartido. Dado que en esta operación de copia se copia entre archivos de la misma cuenta de almacenamiento, puede usar la autenticación de clave compartida para realizar la copia.

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();

    // Get a reference to the directory we created previously.
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

    // Ensure that the directory exists.
    if (sampleDir.Exists())
    {
        // Get a reference to the file we created previously.
        CloudFile sourceFile = sampleDir.GetFileReference("Log1.txt");

        // Ensure that the source file exists.
        if (sourceFile.Exists())
        {
            // Get a reference to the destination file.
            CloudFile destFile = sampleDir.GetFileReference("Log1Copy.txt");

            // Start the copy operation.
            destFile.StartCopy(sourceFile);

            // Write the contents of the destination file to the console window.
            Console.WriteLine(destFile.DownloadText());
        }
    }
}
```

**Copiar un archivo en un blob** en el ejemplo siguiente se crea un archivo y se copia en un blob en la misma cuenta de almacenamiento. El ejemplo crea una SAS para el archivo de origen, que el servicio usa para autorizar el acceso al archivo de origen durante la operación de copia.

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Create a new file share, if it does not already exist.
CloudFileShare share = fileClient.GetShareReference("sample-share");
share.CreateIfNotExists();

// Create a new file in the root directory.
CloudFile sourceFile = share.GetRootDirectoryReference().GetFileReference("sample-file.txt");
sourceFile.UploadText("A sample file in the root directory.");

// Get a reference to the blob to which the file will be copied.
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
CloudBlobContainer container = blobClient.GetContainerReference("sample-container");
container.CreateIfNotExists();
CloudBlockBlob destBlob = container.GetBlockBlobReference("sample-blob.txt");

// Create a SAS for the file that's valid for 24 hours.
// Note that when you are copying a file to a blob, or a blob to a file, you must use a SAS
// to authorize access to the source object, even if you are copying within the same
// storage account.
string fileSas = sourceFile.GetSharedAccessSignature(new SharedAccessFilePolicy()
{
    // Only read permissions are required for the source file.
    Permissions = SharedAccessFilePermissions.Read,
    SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24)
});

// Construct the URI to the source file, including the SAS token.
Uri fileSasUri = new Uri(sourceFile.StorageUri.PrimaryUri.ToString() + fileSas);

// Copy the file to the blob.
destBlob.StartCopy(fileSasUri);

// Write the contents of the file to the console window.
Console.WriteLine("Source file contents: {0}", sourceFile.DownloadText());
Console.WriteLine("Destination blob contents: {0}", destBlob.DownloadText());
```

Puede copiar un blob en un archivo de la misma manera. Si el objeto de origen es un blob, cree una SAS para autorizar el acceso a dicho blob durante la operación de copia.

## <a name="share-snapshots"></a>Instantáneas de recursos compartido
A partir de la versión 8.5 de la biblioteca de clientes de almacenamiento de Azure, se puede crear una instantánea de recurso compartido. Las instantáneas de recursos compartidos también se pueden enumerar, explorar y eliminar. Las instantáneas de recurso compartido son de solo lectura, por lo que en ellas no se permiten operaciones de escritura.

**Creación de instantáneas de recursos compartidos**

En el siguiente ejemplo se crea una instantánea de recurso compartido.

```csharp
storageAccount = CloudStorageAccount.Parse(ConnectionString); 
fClient = storageAccount.CreateCloudFileClient(); 
string baseShareName = "myazurefileshare"; 
CloudFileShare myShare = fClient.GetShareReference(baseShareName); 
var snapshotShare = myShare.Snapshot();

```
**Enumeración de instantáneas de recursos compartidos**

En el ejemplo siguiente se enumeran las instantáneas de un recurso compartido.

```csharp
var shares = fClient.ListShares(baseShareName, ShareListingDetails.All);
```

**Examen de archivos y directorios en instantáneas de recursos compartidos**

En el ejemplo siguiente se examinan los archivos y el directorio de instantáneas de recursos compartidos.

```csharp
CloudFileShare mySnapshot = fClient.GetShareReference(baseShareName, snapshotTime); 
var rootDirectory = mySnapshot.GetRootDirectoryReference(); 
var items = rootDirectory.ListFilesAndDirectories();
```

**Enumeración de recursos compartidos e instantáneas de recursos compartidos, y restauración de archivos o recursos compartidos de archivos a partir de instantáneas de recursos compartidos** 

La toma de una instantánea de un recurso compartido de archivos permite recuperar archivos individuales o todo el recurso compartido de archivos en el futuro. 

Para restaurar un archivo de una instantánea del recurso compartido de archivos, consulte las instantáneas de recursos compartidos de un recurso compartido de archivos. A partir de ahí puede recuperar un archivo que pertenezca a una instantánea de recurso compartido concreta y usar dicha versión para realizar directamente las operaciones de lectura y comparación, o bien para restaurarlo.

```csharp
CloudFileShare liveShare = fClient.GetShareReference(baseShareName);
var rootDirOfliveShare = liveShare.GetRootDirectoryReference();

       var dirInliveShare = rootDirOfliveShare.GetDirectoryReference(dirName);
var fileInliveShare = dirInliveShare.GetFileReference(fileName);

           
CloudFileShare snapshot = fClient.GetShareReference(baseShareName, snapshotTime);
var rootDirOfSnapshot = snapshot.GetRootDirectoryReference();

       var dirInSnapshot = rootDirOfSnapshot.GetDirectoryReference(dirName);
var fileInSnapshot = dir1InSnapshot.GetFileReference(fileName);

string sasContainerToken = string.Empty;
       SharedAccessFilePolicy sasConstraints = new SharedAccessFilePolicy();
       sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24);
       sasConstraints.Permissions = SharedAccessFilePermissions.Read;
       //Generate the shared access signature on the container, setting the constraints directly on the signature.
sasContainerToken = fileInSnapshot.GetSharedAccessSignature(sasConstraints);

string sourceUri = (fileInSnapshot.Uri.ToString() + sasContainerToken + "&" + fileInSnapshot.SnapshotTime.ToString()); ;
fileInliveShare.StartCopyAsync(new Uri(sourceUri));

```


**Eliminación de instantáneas de recursos compartidos**

En el siguiente ejemplo se elimina una instantánea de recurso compartido.

```csharp
CloudFileShare mySnapshot = fClient.GetShareReference(baseShareName, snapshotTime); mySnapshot.Delete(null, null, null);
```

## <a name="troubleshooting-azure-files-using-metrics"></a>Solución de problemas de Azure Files mediante métricas
Azure Storage Analytics ahora admite métricas para Azure Files. Con los datos de las métricas, es posible seguir paso a paso las solicitudes y diagnosticar problemas.

Puede habilitar las métricas para Azure Files mediante el [portal Azure](https://portal.azure.com). La métrica también se puede habilitar mediante programación. Para ello, hay que llamar a la operación Set File Service Properties a través de la API de REST, o de una de sus análogas de la Biblioteca del cliente de Storage.

En el siguiente ejemplo de código se muestra cómo usar la Biblioteca del cliente de almacenamiento para .NET a fin de habilitar las métricas para Azure Files.

En primer lugar, agregue las siguientes directivas `using` a su archivo `Program.cs`, además de las que ya ha agregado:

```csharp
using Microsoft.Azure.Storage.File.Protocol;
using Microsoft.Azure.Storage.Shared.Protocol;
```

Tenga en cuenta que mientras que Azure Blobs, Azure Table y Azure Queues utilizan el tipo compartido `ServiceProperties` en el espacio de nombres `Microsoft.Azure.Storage.Shared.Protocol`, Azure Files utiliza el suyo propio, el tipo `FileServiceProperties` en el espacio de nombres `Microsoft.Azure.Storage.File.Protocol`. No obstante, se debe hacer referencia a ambos espacios de nombres en el código para que se pueda compilar.

```csharp
// Parse your storage connection string from your application's configuration file.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));
// Create the File service client.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Set metrics properties for File service.
// Note that the File service currently uses its own service properties type,
// available in the Microsoft.Azure.Storage.File.Protocol namespace.
fileClient.SetServiceProperties(new FileServiceProperties()
{
    // Set hour metrics
    HourMetrics = new MetricsProperties()
    {
        MetricsLevel = MetricsLevel.ServiceAndApi,
        RetentionDays = 14,
        Version = "1.0"
    },
    // Set minute metrics
    MinuteMetrics = new MetricsProperties()
    {
        MetricsLevel = MetricsLevel.ServiceAndApi,
        RetentionDays = 7,
        Version = "1.0"
    }
});

// Read the metrics properties we just set.
FileServiceProperties serviceProperties = fileClient.GetServiceProperties();
Console.WriteLine("Hour metrics:");
Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
Console.WriteLine(serviceProperties.HourMetrics.Version);
Console.WriteLine();
Console.WriteLine("Minute metrics:");
Console.WriteLine(serviceProperties.MinuteMetrics.MetricsLevel);
Console.WriteLine(serviceProperties.MinuteMetrics.RetentionDays);
Console.WriteLine(serviceProperties.MinuteMetrics.Version);
```

Además, puede hacer referencia al [artículo de solución de problemas de archivos de Azure](storage-troubleshoot-windows-file-connection-problems.md) para ver una guía completa de solución de problemas.

## <a name="next-steps"></a>Pasos siguientes
Consulte los vínculos siguientes para más información acerca Azure Files.

### <a name="conceptual-articles-and-videos"></a>Artículos y vídeos conceptuales
* [Azure Files: un sistema de archivos SMB en la nube sin dificultades para Windows y Linux](https://azure.microsoft.com/documentation/videos/azurecon-2015-azure-files-storage-a-frictionless-cloud-smb-file-system-for-windows-and-linux/)
* [Uso de Azure Files con Linux](storage-how-to-use-files-linux.md)

### <a name="tooling-support-for-file-storage"></a>Compatibilidad de herramientas con el almacenamiento de archivos
* [Uso de AzCopy con Microsoft Azure Storage](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)
* [Uso de la CLI de Azure con Azure Storage](../common/storage-azure-cli.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json#create-and-manage-file-shares)
* [Solución de problemas de Azure Files](https://docs.microsoft.com/azure/storage/storage-troubleshoot-file-connection-problems)

### <a name="reference"></a>Referencia
* [Referencia de la biblioteca de clientes de almacenamiento para .NET](https://msdn.microsoft.com/library/azure/dn261237.aspx)
* [Referencia de la API REST del servicio de archivos](https://msdn.microsoft.com/library/azure/dn167006.aspx)

### <a name="blog-posts"></a>Publicaciones de blog
* [Azure Files is now generally available (Azure Files ya está disponible con carácter general)](https://azure.microsoft.com/blog/azure-file-storage-now-generally-available/)
* [Inside Azure Files (En el interior de Azure Files)](https://azure.microsoft.com/blog/inside-azure-file-storage/)
* [Introducing Microsoft Azure File Service (Introducción al servicio de archivos de Microsoft Azure)](https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx)
* [Persisting connections to Microsoft Azure Files (Persistencia de conexiones en archivos de Microsoft Azure)](https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx)
