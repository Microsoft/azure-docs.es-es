---
title: Desarrollo con Java para Azure Files | Microsoft Docs
description: Aprenda a desarrollar aplicaciones y servicios Java que usen Azure Files para almacenar datos de archivos.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 09/19/2017
ms.custom: devx-track-java
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: b1edc13888a42086cdcd3c3c7756491bd5f43241
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87314482"
---
# <a name="develop-for-azure-files-with-java"></a>Desarrollo con Java para Azure Files
[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

[!INCLUDE [storage-check-out-samples-java](../../../includes/storage-check-out-samples-java.md)]

## <a name="about-this-tutorial"></a>Acerca de este tutorial
En este tutorial se muestran los aspectos básicos del uso de Java para desarrollar aplicaciones o servicios que usen Azure Files para almacenar datos de archivos. Crearemos una aplicación de consola y mostraremos cómo realizar acciones básicas con Java y Azure Files:

* Crear y eliminar recursos compartidos de archivos de Azure
* Crear y eliminar directorios
* Enumerar los archivos y directorios de un recurso compartido de Azure File
* Cargar, descargar y eliminar un archivo

> [!Note]  
> Dado que se puede acceder a Azure Files a través de SMB, es posible escribir aplicaciones que accedan al recurso compartido de archivos de Azure mediante las clases estándar de E/S de Java. En este artículo se describe cómo escribir aplicaciones que usen el SDK de Java de Microsoft Azure Storage, que emplea la [API de REST de Azure Files](https://docs.microsoft.com/rest/api/storageservices/file-service-rest-api) para comunicarse con Azure Files.

## <a name="create-a-java-application"></a>Creación de una aplicación Java
Para compilar las muestras, se necesitará el Kit de desarrollo de Java (JDK) y el [SDK de Azure Storage para Java](https://github.com/Azure/azure-storage-java). También deberá haber creado una cuenta de almacenamiento de Azure.

## <a name="set-up-your-application-to-use-azure-files"></a>Configuración de la aplicación para usar Azure Files
Para utilizar las API de almacenamiento de Azure, agregue la siguiente instrucción al principio del archivo Java desde el que desea acceder al servicio de almacenamiento.

```java
// Include the following imports to use blob APIs.
import com.microsoft.azure.storage.*;
import com.microsoft.azure.storage.file.*;
```

## <a name="set-up-an-azure-storage-connection-string"></a>Configuración de una cadena de conexión de Almacenamiento de Azure
Para usar Azure Files, debe conectarse a su cuenta de almacenamiento de Azure. El primer paso sería configurar una cadena de conexión que se usará para establecer la conexión con su cuenta de almacenamiento. Definamos una variable estática para hacerlo.

```java
// Configure the connection-string with your values
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account_name;" +
    "AccountKey=your_storage_account_key";
```

> [!NOTE]
> Reemplace your_storage_account_name y your_storage_account_key por los valores reales de la cuenta de almacenamiento.
> 
> 

## <a name="connecting-to-an-azure-storage-account"></a>Conexión a una cuenta de almacenamiento de Azure
Para conectarse a su cuenta de almacenamiento, deberá usar el objeto **CloudStorageAccount**, pasando una cadena de conexión a su método de **análisis**.

```java
// Use the CloudStorageAccount object to connect to your storage account
try {
    CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);
} catch (InvalidKeyException invalidKey) {
    // Handle the exception
}
```

**CloudStorageAccount.parse** produce una InvalidKeyException, por lo que deberá colocarla dentro de un bloque try/catch.

## <a name="create-an-azure-file-share"></a>Creación de un recurso compartido de archivos de Azure
Todos los archivos y directorios de Azure Files residen en un contenedor denominado **recurso compartido**. La cuenta de almacenamiento puede tener tantos recursos compartidos como los que permita la capacidad de su cuenta. Para acceder a un recurso compartido y su contenido, es preciso usar un cliente de Azure Files.

```java
// Create the Azure Files client.
CloudFileClient fileClient = storageAccount.createCloudFileClient();
```

Se usa el cliente de Azure Files para obtener una referencia a un recurso compartido.

```java
// Get a reference to the file share
CloudFileShare share = fileClient.getShareReference("sampleshare");
```

Para crear el recurso compartido, utilice el método **createIfNotExists** del objeto CloudFileShare.

```java
if (share.createIfNotExists()) {
    System.out.println("New share created");
}
```

En este punto, **share** contiene una referencia a un recurso compartido denominado **sampleshare**.

## <a name="delete-an-azure-file-share"></a>Eliminación de un recurso compartido de archivos de Azure
Para eliminar un recurso compartido es necesario efectuar una llamada al método **deleteIfExists** en un objeto CloudFileShare. A continuación se facilita código de ejemplo que lo hace.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

    // Create the file client.
   CloudFileClient fileClient = storageAccount.createCloudFileClient();

   // Get a reference to the file share
   CloudFileShare share = fileClient.getShareReference("sampleshare");

   if (share.deleteIfExists()) {
       System.out.println("sampleshare deleted");
   }
} catch (Exception e) {
    e.printStackTrace();
}
```

## <a name="create-a-directory"></a>Creación de un directorio
También puede organizar el almacenamiento y colocar los archivos dentro de los subdirectorios en lugar de mantenerlos todos en el directorio raíz. Azure Files permite crear tantos directorios como permita su cuenta. El código siguiente creará un subdirectorio denominado **sampledir** en el directorio raíz.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

//Get a reference to the sampledir directory
CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

if (sampleDir.createIfNotExists()) {
    System.out.println("sampledir created");
} else {
    System.out.println("sampledir already exists");
}
```

## <a name="delete-a-directory"></a>Eliminación de un directorio
La eliminación de un directorio es una tarea sencilla, aunque se debe tener en cuenta que no se puede eliminar un directorio que contenga archivos u otros directorios.

```java
// Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

// Get a reference to the directory you want to delete
CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

// Delete the directory
if ( containerDir.deleteIfExists() ) {
    System.out.println("Directory deleted");
}
```

## <a name="enumerate-files-and-directories-in-an-azure-file-share"></a>Enumerar los archivos y directorios de un recurso compartido de Azure File
Obtener una lista de archivos y directorios dentro de un recurso compartido se realiza fácilmente mediante una llamada a **listFilesAndDirectories** en una referencia de CloudFileDirectory. El método devuelve una lista de objetos ListFileItem en los que puede efectuar la iteración. Por ejemplo, el siguiente código enumerará archivos y directorios dentro del directorio raíz.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

for ( ListFileItem fileItem : rootDir.listFilesAndDirectories() ) {
    System.out.println(fileItem.getUri());
}
```

## <a name="upload-a-file"></a>Cargar un archivo
En esta sección, aprenderá cómo cargar un archivo del almacenamiento local en el directorio raíz de un recurso compartido.

El primer paso para cargar un archivo es obtener una referencia al directorio donde debe residir. Para ello, llame al método **getRootDirectoryReference** del objeto del recurso compartido.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();
```

Ahora que tiene una referencia al directorio raíz del recurso compartido, puede cargar un archivo en él utilizando el código siguiente.

```java
        // Define the path to a local file.
        final String filePath = "C:\\temp\\Readme.txt";
    
        CloudFile cloudFile = rootDir.getFileReference("Readme.txt");
        cloudFile.uploadFromFile(filePath);
```

## <a name="download-a-file"></a>Descarga de un archivo
Una de las operaciones más frecuentes que realizará en Azure Files es la descarga de archivos. En el ejemplo siguiente, el código descarga el archivo SampleFile.txt y muestra su contenido.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

//Get a reference to the directory that contains the file
CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

//Get a reference to the file you want to download
CloudFile file = sampleDir.getFileReference("SampleFile.txt");

//Write the contents of the file to the console.
System.out.println(file.downloadText());
```

## <a name="delete-a-file"></a>Eliminación de un archivo
Otra operación habitual en Azure Files es la eliminación de archivos. El código siguiente elimina un archivo denominado SampleFile.txt que se almacena en un directorio denominado **sampledir**.

```java
// Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

// Get a reference to the directory where the file to be deleted is in
CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

String filename = "SampleFile.txt"
CloudFile file;

file = containerDir.getFileReference(filename)
if ( file.deleteIfExists() ) {
    System.out.println(filename + " was deleted");
}
```

## <a name="next-steps"></a>Pasos siguientes
Si desea obtener más información acerca de otras API de almacenamiento de Azure, siga estos vínculos.

* [Azure para desarrolladores de Java](/java/azure)/)
* [SDK de Azure Storage para Java](https://github.com/azure/azure-storage-java)
* [SDK de Azure Storage para Android](https://github.com/azure/azure-storage-android)
* [Referencia del SDK del cliente de Azure Storage](https://javadoc.io/doc/com.microsoft.azure/azure-core/0.8.0/index.html)
* [API de REST de servicios de Azure Storage](https://msdn.microsoft.com/library/azure/dd179355.aspx)
* [Blog del equipo de Azure Storage](https://blogs.msdn.com/b/windowsazurestorage/)
* [Transferencia de datos con la utilidad en línea de comandos AzCopy](../common/storage-use-azcopy.md)
* [Solución de problemas de Azure Files en Windows](storage-troubleshoot-windows-file-connection-problems.md)
