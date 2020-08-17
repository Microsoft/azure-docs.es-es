---
title: 'Inicio rápido: Biblioteca cliente de Azure Blob Storage v8 para Java'
description: Cree una cuenta de almacenamiento y un contenedor en el almacenamiento de objetos (Blob). Después, use la biblioteca cliente de Azure Storage v8 para Java a fin de cargar un blob en Azure Storage, descargar un blob o enumerar los blobs de un contenedor.
author: mhopkins-msft
ms.custom: devx-track-java
ms.author: mhopkins
ms.date: 07/24/2020
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.openlocfilehash: 9ab45495283f8751912d93e9799958a8effc887a
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323747"
---
# <a name="quickstart-manage-blobs-with-java-v8-sdk"></a>Inicio rápido: Administración de blobs con el SDK de Java v8

En este inicio rápido, se aprende a administrar blobs mediante Java. Los blobs son objetos que pueden contener grandes cantidades de datos de texto o binarios, como imágenes, documentos, elementos multimedia de streaming y datos de archivo. Va a cargar, descargar y enumerar blobs. También va a crear y eliminar contenedores y a establecer sus permisos.

> [!NOTE]
> Esta guía de inicio rápido usa una versión heredada de la biblioteca cliente de Azure Blob Storage. Para empezar a trabajar con la versión más reciente, consulte [Inicio rápido: Administración de blobs con el SDK de Java v12](storage-quickstart-blobs-java.md).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- Una cuenta de Azure Storage. [Crear una cuenta de almacenamiento](../common/storage-account-create.md).
- Un IDE que tenga la integración de Maven. En esta guía se usa [Eclipse](https://www.eclipse.org/downloads/) con la configuración del "IDE de Eclipse para desarrolladores de Java".

## <a name="download-the-sample-application"></a>Descarga de la aplicación de ejemplo

La [aplicación de ejemplo](https://github.com/Azure-Samples/storage-blobs-java-quickstart) es una aplicación de consola básica.

Use [git](https://git-scm.com/) para descargar una copia de la aplicación en su entorno de desarrollo.

```bash
git clone https://github.com/Azure-Samples/storage-blobs-java-quickstart.git
```

Este comando clona el repositorio en la carpeta git local. Para abrir el proyecto, inicie Eclipse y cierre la pantalla de bienvenida. Seleccione **File** (Archivo) y después **Open Projects from File System** (Abrir proyectos desde el sistema de archivos). Asegúrese de la casilla **Detect and configure project natures** (Detectar y configurar los tipos de proyecto) está activada. Seleccione **Directory** (Directorio) y vaya a donde se haya almacenado el repositorio clonado. En el repositorio clonado, seleccione la carpeta **blobAzureApp**. Asegúrese de que el proyecto **blobAzureApp** aparece como un proyecto de Eclipse y, después, seleccione **Finish** (Finalizar).

Cuando se termine de importar el proyecto, abra **AzureApp.java** (ubicado en **blobQuickstart.blobAzureApp** dentro de **src/main/java**) y reemplace `accountname` y `accountkey` dentro de la cadena `storageConnectionString`. Después, ejecute la aplicación. Las instrucciones concretas para completar estas tareas se describen en las secciones siguientes.

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Configuración de la cadena de conexión de almacenamiento.

En la aplicación, debe proporcionar la cadena de conexión para la cuenta de almacenamiento. Abra el archivo **AzureApp.Java**. Busque la variable `storageConnectionString` y pegue el valor de la cadena de conexión que ha copiado en la sección anterior. La variable `storageConnectionString` debe tener una apariencia similar a la del siguiente ejemplo de código:

```java
public static final String storageConnectionString =
"DefaultEndpointsProtocol=https;" +
"AccountName=<account-name>;" +
"AccountKey=<account-key>";
```

## <a name="run-the-sample"></a>Ejecución del ejemplo

En esta aplicación de ejemplo se crea un archivo de prueba en el directorio predeterminado (*C:\Users\<user>\AppData\Local\Temp*, para usuarios de Windows), se carga en Blob Storage, se enumeran los blobs del contenedor y luego se descarga el archivo con un nombre nuevo para que se puedan comparar los archivos antiguos y los nuevos.

Ejecute el ejemplo con Maven en la línea de comandos. Abra un shell y vaya a **blobAzureApp** dentro del directorio clonado. A continuación, escriba `mvn compile exec:java`.

El siguiente ejemplo muestra la salida si fuera a ejecutar la aplicación en Windows.

```
Azure Blob storage quick start sample
Creating container: quickstartcontainer
Creating a sample file at: C:\Users\<user>\AppData\Local\Temp\sampleFile514658495642546986.txt
Uploading the sample file
URI of blob is: https://myexamplesacct.blob.core.windows.net/quickstartcontainer/sampleFile514658495642546986.txt
The program has completed successfully.
Press the 'Enter' key while in the console to delete the sample files, example container, and exit the application.

Deleting the container
Deleting the source, and downloaded files
```

Antes de continuar, revise el directorio predeterminado (*C:\Users\<user>\AppData\Local\Temp*, para usuarios de Windows) del archivo de ejemplo. Copie la dirección URL para el blob fuera de la ventana de consola y péguela en un explorador para ver el contenido del archivo de Blob Storage. Si el archivo de ejemplo en el directorio se compara con el contenido almacenado en Blob Storage, verá que son los mismos.

  >[!NOTE]
  >También puede usar una herramienta como [Explorador de Azure Storage](https://storageexplorer.com/?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) para ver los archivos de Blob Storage. El Explorador de Azure Storage es una herramienta gratuita multiplataforma que permite acceder a la información de la cuenta de almacenamiento.

Después de haber comprobado los archivos, presione la tecla **Entrar** para finalizar la demostración y eliminar los archivos de prueba. Ahora que sabe lo que hace el ejemplo, abra el archivo **AzureApp.java** para examinar el código.

## <a name="understand-the-sample-code"></a>Descripción del código de ejemplo

A continuación, explicaremos el código de ejemplo para ayudarle a comprender saber cómo funciona.

### <a name="get-references-to-the-storage-objects"></a>Obtención de referencias a los objetos de almacenamiento

Lo primero que hay que hacer es crear las referencias a los objetos usados para acceder a Blob Storage y administrarlo. Estos objetos dependen unos de otros: cada uno es utilizado por el siguiente de la lista.

* Cree una instancia del objeto [CloudStorageAccount](/java/api/com.microsoft.azure.management.storage.storageaccount) que apunte a la cuenta de almacenamiento.

    El objeto **CloudStorageAccount** es una representación de su cuenta de almacenamiento, que le permite establecer las propiedades de la cuenta de almacenamiento mediante programación, así como acceder a estas. Con el objeto **CloudStorageAccount** se puede crear una instancia del objeto **CloudBlobClient**, que es necesario para acceder a Blob service.

* Cree una instancia del objeto **CloudBlobClient**, que apunte a [Blob service](/java/api/com.microsoft.azure.storage.blob.cloudblobclient) de la cuenta de almacenamiento.

    El objeto **CloudBlobClient** proporciona un punto de acceso a Blob service, lo que le permite establecer las propiedades de Blob Storage mediante programación, así como acceder a estas. Con el objeto **CloudBlobClient** se puede crear una instancia del objeto **CloudBlobContainer**, que es necesario para crear contenedores.

* Cree una instancia del objeto [CloudBlobContainer](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer), que representa el contenedor al que está accediendo. Use los contenedores para organizar los blobs al igual que se usan las carpetas del equipo para organizar los archivos.

    Una vez que tenga **CloudBlobContainer**, puede crear una instancia del objeto [CloudBlockBlob](/java/api/com.microsoft.azure.storage.blob.cloudblockblob) que apunte al blob específico en el que está interesado y realizar una operación de carga, descarga, copia, etc.

> [!IMPORTANT]
> Los nombres de contenedor deben estar en minúsculas. Para más información acerca de los contenedores, consulte [Naming and Referencing Containers, Blobs, and Metadata](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Asignación de nombres y referencia a contenedores, blobs y metadatos).

### <a name="create-a-container"></a>Crear un contenedor

En esta sección, se crean instancias de los objetos, se crea un nuevo contenedor y luego se establecen permisos en el contenedor para que los blobs sean públicos y se pueda acceder a ellos con tan solo una dirección URL. El contenedor se denomina **quickstartcontainer**.

En este ejemplo se utiliza [CreateIfNotExists](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.createifnotexists) porque queremos crear un nuevo contenedor cada vez que se ejecuta el ejemplo. En un entorno de producción donde se usa el mismo contenedor en toda una aplicación, lo más recomendable es llamar solo a **CreateIfNotExists** una vez. O bien se puede crear el contenedor con antelación para que no necesite crearlo en el código.

```java
// Parse the connection string and create a blob client to interact with Blob storage
storageAccount = CloudStorageAccount.parse(storageConnectionString);
blobClient = storageAccount.createCloudBlobClient();
container = blobClient.getContainerReference("quickstartcontainer");

// Create the container if it does not exist with public access.
System.out.println("Creating container: " + container.getName());
container.createIfNotExists(BlobContainerPublicAccessType.CONTAINER, new BlobRequestOptions(), new OperationContext());
```

### <a name="upload-blobs-to-the-container"></a>Carga de blobs al contenedor

Para cargar un archivo a un blob en bloques, obtenga una referencia al blob en el contenedor de destino. Cuando tenga la referencia de blob, puede cargar datos en él mediante [CloudBlockBlob.Upload](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob.cloudblockblob.upload). Esta operación crea el blob si todavía no existe o lo sobrescribe si ya existe.

El código de ejemplo crea un archivo local que se usará para la carga y descarga, almacenando el archivo que se va a cargar como **source** y el nombre del blob en **blob**. En el ejemplo siguiente se carga el archivo al contenedor llamado **quickstartcontainer**.

```java
//Creating a sample file
sourceFile = File.createTempFile("sampleFile", ".txt");
System.out.println("Creating a sample file at: " + sourceFile.toString());
Writer output = new BufferedWriter(new FileWriter(sourceFile));
output.write("Hello Azure!");
output.close();

//Getting a blob reference
CloudBlockBlob blob = container.getBlockBlobReference(sourceFile.getName());

//Creating blob and uploading file to it
System.out.println("Uploading the sample file ");
blob.uploadFromFile(sourceFile.getAbsolutePath());
```

Hay varios métodos `upload`, incluidos [upload](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.upload), [uploadBlock](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadblock), [uploadFullBlob](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadfullblob), [uploadStandardBlobTier](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadstandardblobtier)y [uploadText](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadtext) que se pueden usar con Blob Storage. Por ejemplo, si tiene una cadena, puede usar el método `UploadText` en lugar de emplear `Upload`.

Los blobs en bloques pueden ser cualquier tipo de archivo de texto o binario. Los blobs en páginas se usan principalmente con los archivos VHD que respaldan las máquinas virtuales IaaS. Use los blobs en anexos para el registro, por ejemplo, cuando desea escribir en un archivo y luego sigue agregando más información. La mayoría de los objetos almacenados en Blob Storage son blobs en bloques.

### <a name="list-the-blobs-in-a-container"></a>Enumerar los blobs de un contenedor

Puede obtener una lista de archivos del contenedor mediante [CloudBlobContainer.ListBlobs](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.listblobs). El código siguiente recupera la lista de blobs, luego los recorre y se muestran los URI de los blobs encontrados. Puede copiar el URI desde la ventana de comandos y pegarlo en un explorador para ver el archivo.

```java
//Listing contents of container
for (ListBlobItem blobItem : container.listBlobs()) {
    System.out.println("URI of blob is: " + blobItem.getUri());
}
```

### <a name="download-blobs"></a>Descargar blobs

Descargue blobs al disco local mediante [CloudBlob.DownloadToFile](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob.cloudblob.downloadtofile).

El código siguiente descarga el blob cargado en una sección anterior, agregando el sufijo "_DOWNLOADED" al nombre de blob para que pueda ver ambos archivos en el disco local.

```java
// Download blob. In most cases, you would have to retrieve the reference
// to cloudBlockBlob here. However, we created that reference earlier, and
// haven't changed the blob we're interested in, so we can reuse it.
// Here we are creating a new file to download to. Alternatively you can also pass in the path as a string into downloadToFile method: blob.downloadToFile("/path/to/new/file").
downloadedFile = new File(sourceFile.getParentFile(), "downloadedFile.txt");
blob.downloadToFile(downloadedFile.getAbsolutePath());
```

### <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita los blobs cargados, puede eliminar todo el contenedor mediante [CloudBlobContainer.DeleteIfExists](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.deleteifexists). Este método también elimina los archivos del contenedor.

```java
try {
if(container != null)
    container.deleteIfExists();
} catch (StorageException ex) {
System.out.println(String.format("Service error. Http code: %d and error code: %s", ex.getHttpStatusCode(), ex.getErrorCode()));
}

System.out.println("Deleting the source, and downloaded files");

if(downloadedFile != null)
downloadedFile.deleteOnExit();

if(sourceFile != null)
sourceFile.deleteOnExit();
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo aprendió a transferir archivos entre un disco local y Azure Blob Storage mediante Java. Para más información sobre cómo trabajar con Java, vaya a nuestro repositorio de código fuente de GitHub.

> [!div class="nextstepaction"]
> [Referencia de API de Java](https://docs.microsoft.com/java/api/overview/azure/storage?view=azure-java-legacy)
> [Ejemplos de código para Java](../common/storage-samples-java.md)
