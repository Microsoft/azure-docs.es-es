---
title: 'Guía de inicio rápido de Azure: Creación de un blob en el almacenamiento de objetos con PHP | Microsoft Docs'
description: Aprenda rápidamente a transferir objetos hacia o desde Azure Blob Storage mediante PHP. Cargue, descargue y enumere blobs en bloques en un contenedor en Azure Blob Storage.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 11/14/2018
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: a19ff153ea0625a14401770de690dcfddeb74d88
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835755"
---
#  <a name="transfer-objects-tofrom-azure-blob-storage-using-php"></a>Transferencia de objetos hacia Azure Blob Storage, y desde él, mediante PHP
En esta guía de inicio rápido, aprenderá a usar PHP para cargar, descargar y enumerar blobs en bloques en un contenedor en Azure Blob Storage. 

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

Asegúrese de tener instalados los siguientes requisitos previos adicionales:

* [PHP](https://php.net/downloads.php)
* [Azure Storage SDK para PHP](https://github.com/Azure/azure-storage-php)

## <a name="download-the-sample-application"></a>Descarga de la aplicación de ejemplo
La [aplicación de ejemplo](https://github.com/Azure-Samples/storage-blobs-php-quickstart.git) utilizada en esta guía de inicio rápido es una aplicación PHP básica.  

Use [git](https://git-scm.com/) para descargar una copia de la aplicación en su entorno de desarrollo. 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-php-quickstart.git
```

Este comando clona el repositorio en la carpeta git local. Para abrir la aplicación de ejemplo PHP, busque la carpeta storage-blobs-ruby-quickstart y abra el archivo example.rb.  

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Configuración de la cadena de conexión de almacenamiento.
En la aplicación, debe proporcionar el nombre y la clave de la cuenta de almacenamiento para crear la instancia **BlobRestProxy** de la aplicación. Se recomienda almacenar estos identificadores dentro de una variable de entorno en la máquina local que ejecuta la aplicación. Utilice uno de los estos ejemplos dependiendo de su sistema operativo para crear la variable de entorno. Reemplace los valores **youraccountname** y **youraccountkey** con el nombre de cuenta y la clave.

# <a name="linux"></a>[Linux](#tab/linux)

```bash
export ACCOUNT_NAME=<youraccountname>
export ACCOUNT_KEY=<youraccountkey>
```

# <a name="windows"></a>[Windows](#tab/windows)

```cmd
setx ACCOUNT_NAME=<youraccountname>
setx ACCOUNT_KEY=<youraccountkey>
```
---

## <a name="configure-your-environment"></a>Configurar su entorno
Tome la carpeta de su carpeta local git y colóquela en un directorio proporcionado por su servidor PHP. Después, abra un símbolo del sistema en ese mismo directorio y escriba: `php composer.phar install`

## <a name="run-the-sample"></a>Ejecución del ejemplo
En este ejemplo se crea un archivo de prueba en la carpeta '.'. El programa de ejemplo carga el archivo de prueba en Blob Storage, enumera los blobs en el contenedor y descarga el archivo con un nuevo nombre. 

Ejecute el ejemplo. La salida siguiente es un ejemplo de la salida devuelta al ejecutar la aplicación:
  
```
Uploading BlockBlob: HelloWorld.txt
These are the blobs present in the container: HelloWorld.txt: https://myexamplesacct.blob.core.windows.net/blockblobsleqvxd/HelloWorld.txt

This is the content of the blob uploaded: Hello Azure!
```
Al presionar el botón mostrado, el programa de ejemplo elimina el contenedor de almacenamiento y los archivos. Antes de continuar, compruebe la carpeta del servidor de los dos archivos. Puede abrirlos y ver que son idénticas.

También puede usar una herramienta como [Explorador de Azure Storage](https://storageexplorer.com) para ver los archivos de Blob Storage. El Explorador de Azure Storage es una herramienta gratuita multiplataforma que permite acceder a la información de la cuenta de almacenamiento. 

Después de haber comprobado los archivos, presione cualquier tecla para finalizar la demostración y eliminar los archivos de prueba. Ahora que sabe lo que hace el ejemplo, abra el archivo example.rb para examinar el código. 

## <a name="understand-the-sample-code"></a>Descripción del código de ejemplo

A continuación, explicaremos el código de ejemplo para ayudarle a comprender saber cómo funciona.

### <a name="get-references-to-the-storage-objects"></a>Obtención de referencias a los objetos de almacenamiento
Lo primero que hay que hacer es crear las referencias a los objetos usados para acceder a Blob Storage y administrarlo. Estos objetos dependen unos de otros, y cada uno es utilizado por el siguiente de la lista.

* Cree una instancia del objeto **BlobRestProxy** de Azure Storage para configurar las credenciales de conexión. 
* Cree el objeto **BlobService**, que apunte a Blob service en la cuenta de almacenamiento. 
* Cree el objeto **Container**, que representa el contenedor al que está accediendo. Los contenedores se usan para organizar los blobs al igual que se usan las carpetas en el equipo para organizar los archivos.

Una vez que tenga el objeto de contenedor **blobClient**, puede crear el objeto de blob **Block** que apunte al blob específico en el que está interesado. Después, puede realizar operaciones como carga, descarga y copia.

> [!IMPORTANT]
> Los nombres de contenedor deben estar en minúsculas. Para más información sobre la nomenclatura de contenedores y blobs, consulte [Naming and Referencing Containers, Blobs, and Metadata](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Asignación de nombres y referencia a contenedores, blobs y metadatos).

En esta sección, se configura una instancia del cliente de Azure Storage, se crean instancias de los objetos, se crea un contenedor y se establecen permisos en el contenedor para que los blobs sean públicos. El contenedor se denomina **quickstartblobs**. 

```PHP
    # Setup a specific instance of an Azure::Storage::Client
    $connectionString = "DefaultEndpointsProtocol=https;AccountName=".getenv('account_name').";AccountKey=".getenv('account_key');
    
    // Create blob client.
    $blobClient = BlobRestProxy::createBlobService($connectionString);
    
    # Create the BlobService that represents the Blob service for the storage account
    $createContainerOptions = new CreateContainerOptions();
    
    $createContainerOptions->setPublicAccess(PublicAccessType::CONTAINER_AND_BLOBS);
    
    // Set container metadata.
    $createContainerOptions->addMetaData("key1", "value1");
    $createContainerOptions->addMetaData("key2", "value2");

    $containerName = "blockblobs".generateRandomString();

    try    {
        // Create container.
        $blobClient->createContainer($containerName, $createContainerOptions);
```

### <a name="upload-blobs-to-the-container"></a>Carga de blobs al contenedor

Blob Storage admite blobs en bloques, blobs en anexos y blobs en páginas. Los blobs en bloques son los que se usan con más frecuencia y serán los que utilicemos en este tutorial de inicio rápido.  

Para cargar un archivo en un blob, obtenga la ruta de acceso completa del archivo combinando el nombre del directorio y el nombre de archivo en la unidad local. A continuación, puede cargar el archivo en la ruta de acceso especificada con el método **createBlockBlob()** . 

El código de ejemplo toma un archivo local y lo carga en Azure. El archivo se almacena como **myfile** y el nombre del blob como **fileToUpload** en el código. En el ejemplo siguiente se carga el archivo al contenedor denominado **quickstartblobs**.

```PHP
    $myfile = fopen("HelloWorld.txt", "w") or die("Unable to open file!");
    fclose($myfile);

    # Upload file as a block blob
    echo "Uploading BlockBlob: ".PHP_EOL;
    echo $fileToUpload;
    echo "<br />";
    
    $content = fopen($fileToUpload, "r");

    //Upload blob
    $blobClient->createBlockBlob($containerName, $fileToUpload, $content);
```

Para realizar una actualización parcial del contenido de un blob en bloques, utilice el método **createblocklist()** . Los blobs en bloques pueden tener un tamaño de hasta 4,7 TB y pueden ser cualquier cosa, desde hojas de cálculo de Excel a archivos de vídeo grandes. Los blobs en páginas se utilizan principalmente para los archivos VHD usados para respaldar VM IaaS. Los blobs en anexos se utilizan para el registro, por ejemplo, cuando desea escribir en un archivo y luego sigue agregando más información. Los blobs en anexos deben usarse en un modelo de escritura único. La mayoría de los objetos almacenados en Blob Storage son blobs en bloques.

### <a name="list-the-blobs-in-a-container"></a>Enumerar los blobs de un contenedor

Puede obtener una lista de archivos del contenedor con el método **listBlobs()** . El código siguiente recupera la lista de blobs, luego los recorre y se muestran los nombres de los blobs encontrados en un contenedor.  

```PHP
    $listBlobsOptions = new ListBlobsOptions();
    $listBlobsOptions->setPrefix("HelloWorld");
    
    echo "These are the blobs present in the container: ";
    
    do{
        $result = $blobClient->listBlobs($containerName, $listBlobsOptions);
        foreach ($result->getBlobs() as $blob)
        {
            echo $blob->getName().": ".$blob->getUrl()."<br />";
        }
        
        $listBlobsOptions->setContinuationToken($result->getContinuationToken());
    } while($result->getContinuationToken());
```

### <a name="get-the-content-of-your-blobs"></a>Obtención del contenido de los blobs

Obtenga el contenido de los blobs mediante el método **getBlob()**. El código siguiente muestra el contenido del blob cargado en una sección anterior.

```PHP
    $blob = $blobClient->getBlob($containerName, $fileToUpload);
    fpassthru($blob->getContentStream());
```

### <a name="clean-up-resources"></a>Limpieza de recursos
Si ya no necesita los blobs cargados en esta guía de inicio rápido, puede eliminar todo el contenedor mediante el método **deleteContainer()** . Si ya no necesita los archivos creados, use el método **deleteBlob()** para eliminarlos.

```PHP
    // Delete blob.
    echo "Deleting Blob".PHP_EOL;
    echo $fileToUpload;
    echo "<br />";
    $blobClient->deleteBlob($_GET["containerName"], $fileToUpload);

    // Delete container.
    echo "Deleting Container".PHP_EOL;
    echo $_GET["containerName"].PHP_EOL;
    echo "<br />";
    $blobClient->deleteContainer($_GET["containerName"]);

    //Deleting local file
    echo "Deleting file".PHP_EOL;
    echo "<br />";
    unlink($fileToUpload);   
```

## <a name="resources-for-developing-php-applications-with-blobs"></a>Recursos para el desarrollo de aplicaciones PHP con blobs

Consulte estos recursos adicionales para el desarrollo de PHP con Blob Storage:

- Consulte, descargue e instale el [código fuente de la biblioteca de cliente de PHP](https://github.com/Azure/azure-storage-php) para Azure Storage en GitHub.
- Explore los [ejemplos de Blob Storage](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=php&term=blob) escritos mediante la biblioteca de cliente de PHP.

## <a name="next-steps"></a>Pasos siguientes
 
En esta guía de inicio rápido, ha aprendido a transferir archivos entre un disco local y Azure Blob Storage mediante PHP. Para más información acerca de cómo trabajar con PHP, vaya nuestro Centro para desarrolladores de PHP.

> [!div class="nextstepaction"]
> [Centro para desarrolladores de PHP](https://azure.microsoft.com/develop/php/)


Para más información sobre el Explorador de Storage y los blobs, consulte [Administración de recursos de Azure Blob Storage con el Explorador de Storage (versión preliminar)](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
