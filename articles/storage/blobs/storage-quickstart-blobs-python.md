---
title: 'Inicio rápido: Biblioteca de Azure Blob Storage v12: Python'
description: En esta guía de inicio rápido, obtendrá información sobre cómo usar la biblioteca cliente de Azure Blob Storage versión 12 para Python a fin de crear un contenedor y un blob en Blob Storage (objeto). A continuación, aprenderá a descargar el blob en un equipo local y a enumerar todos los blobs en un contenedor.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/24/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.custom: devx-track-python
ms.openlocfilehash: 1a9dc47888110836239c59f6a7542f71f753dd3a
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89568190"
---
# <a name="quickstart-manage-blobs-with-python-v12-sdk"></a>Inicio rápido: Administración de blobs con el SDK de Python v12

En este inicio rápido, se aprende a administrar blobs mediante Python. Los blobs son objetos que pueden contener grandes cantidades de texto o datos binarios, como imágenes, documentos, medios de transmisión y datos de archivo. Cargará, descargará y enumerará los blobs, y creará y eliminará los contenedores.

Recursos adicionales:

* [Documentación de referencia de API](/python/api/azure-storage-blob)
* [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-blob)
* [Paquete (índice de paquetes de Python)](https://pypi.org/project/azure-storage-blob/)
* [Muestras](https://docs.microsoft.com/azure/storage/common/storage-samples-python?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#blob-samples)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- Una cuenta de Azure Storage. [Crear una cuenta de almacenamiento](../common/storage-account-create.md).
- [Python](https://www.python.org/downloads/) 2.7, 3.5 o versiones posteriores

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="setting-up"></a>Instalación

En esta sección se explica cómo preparar un proyecto para que funcione con la biblioteca cliente de Azure Blob Storage v12 para Python.

### <a name="create-the-project"></a>Creación del proyecto

Cree una aplicación de Python llamada *blob-quickstart-v12*.

1. En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para el proyecto.

    ```console
    mkdir blob-quickstart-v12
    ```

1. Cambie al directorio *blob-quickstart-v12* recién creado.

    ```console
    cd blob-quickstart-v12
    ```

1. En el directorio *blob-quickstart-v12*, cree otro directorio denominado *data*. Aquí es donde se crearán y almacenarán los archivos de datos de blobs.

    ```console
    mkdir data
    ```

### <a name="install-the-package"></a>Instalar el paquete

Mientras sigue en el directorio de aplicaciones, instale el paquete de la biblioteca cliente de Azure Blob Storage para Python con el comando `pip install`.

```console
pip install azure-storage-blob
```

Este comando instala el paquete de la biblioteca cliente de Azure Blob Storage para Python y todas las bibliotecas de las que depende. En este caso, se trata solo de la biblioteca principal de Azure para Python.

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

1. Abra un nuevo archivo de texto en el editor de código.
1. Agregue instrucciones `import`.
1. Cree la estructura del programa, incluido un control de excepciones básico.

    Este es el código:

    ```python
    import os, uuid
    from azure.storage.blob import BlobServiceClient, BlobClient, ContainerClient, __version__

    try:
        print("Azure Blob storage v" + __version__ + " - Python quickstart sample")
        # Quick start code goes here
    except Exception as ex:
        print('Exception:')
        print(ex)
    ```

1. Guarde el nuevo archivo como *blob-quickstart-v12.py* en el directorio *blob-quickstart-v12*.

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Modelo de objetos

Azure Blob Storage está optimizado para el almacenamiento de cantidades masivas de datos no estructurados. Los datos no estructurados son datos que no cumplen un modelo de datos o definición concreta, como texto o datos binarios. Blob Storage ofrece tres tipos de recursos:

* La cuenta de almacenamiento
* Un contenedor en la cuenta de almacenamiento
* Un blob en el contenedor

En el siguiente diagrama se muestra la relación entre estos recursos.

![Diagrama de arquitectura de Blob Storage](./media/storage-blobs-introduction/blob1.png)

Use las siguientes clases de Python para interactuar con estos recursos:

* [BlobServiceClient](/python/api/azure-storage-blob/azure.storage.blob.blobserviceclient): La clase `BlobServiceClient` permite manipular recursos de Azure Storage y contenedores de blobs.
* [ContainerClient](/python/api/azure-storage-blob/azure.storage.blob.containerclient): La clase `ContainerClient` permite manipular contenedores de Azure Storage y sus blobs.
* [BlobClient](/python/api/azure-storage-blob/azure.storage.blob.blobclient): La clase `BlobClient` permite manipular los blobs de Azure Storage.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código de ejemplo muestran cómo realizar las siguientes acciones con la biblioteca cliente de Azure Blob Storage para Python:

* [Obtención de la cadena de conexión](#get-the-connection-string)
* [Creación de un contenedor](#create-a-container)
* [Carga de los blobs en un contenedor](#upload-blobs-to-a-container)
* [Enumeración de los blobs de un contenedor](#list-the-blobs-in-a-container)
* [Descarga de los blobs](#download-blobs)
* [Eliminación de un contenedor](#delete-a-container)

### <a name="get-the-connection-string"></a>Obtención de la cadena de conexión

El código siguiente recupera la cadena de conexión de la cuenta de almacenamiento de la variable de entorno creada en la sección [Configuración de la cadena de conexión de almacenamiento](#configure-your-storage-connection-string).

Agregue este código dentro del bloque `try`:

```python
# Retrieve the connection string for use with the application. The storage
# connection string is stored in an environment variable on the machine
# running the application called AZURE_STORAGE_CONNECTION_STRING. If the environment variable is
# created after the application is launched in a console or with Visual Studio,
# the shell or application needs to be closed and reloaded to take the
# environment variable into account.
connect_str = os.getenv('AZURE_STORAGE_CONNECTION_STRING')
```

### <a name="create-a-container"></a>Crear un contenedor

Decida un nombre para el nuevo contenedor. El código siguiente anexa un valor de UUID al nombre de contenedor para asegurarse de que sea único.

> [!IMPORTANT]
> Los nombres de contenedor deben estar en minúsculas. Para más información acerca de los contenedores de nomenclatura y los blobs, consulte [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Asignación de nombres y realización de referencias a contenedores, blobs y metadatos).

Cree una instancia de la clase [BlobServiceClient](/python/api/azure-storage-blob/azure.storage.blob.blobserviceclient) llamando al método [from_connection_string](/python/api/azure-storage-blob/azure.storage.blob.blobserviceclient#from-connection-string-conn-str--credential-none----kwargs-). A continuación, llame al método [create_container](/python/api/azure-storage-blob/azure.storage.blob.blobserviceclient#create-container-name--metadata-none--public-access-none----kwargs-) para crear realmente el contenedor en la cuenta de almacenamiento.

Agregue este código al final del bloque `try`:

```python
# Create the BlobServiceClient object which will be used to create a container client
blob_service_client = BlobServiceClient.from_connection_string(connect_str)

# Create a unique name for the container
container_name = "quickstart" + str(uuid.uuid4())

# Create the container
container_client = blob_service_client.create_container(container_name)
```

### <a name="upload-blobs-to-a-container"></a>Carga de los blobs en un contenedor

El siguiente fragmento de código:

1. Crea un archivo de texto en el directorio local.
1. Obtiene una referencia a un objeto [BlobClient](/python/api/azure-storage-blob/azure.storage.blob.blobclient) llamando al método [get_blob_client](/python/api/azure-storage-blob/azure.storage.blob.containerclient#get-blob-client-blob--snapshot-none-) en la clase [BlobServiceClient](/python/api/azure-storage-blob/azure.storage.blob.blobserviceclient) de la sección [Crear un contenedor](#create-a-container).
1. Carga el archivo de texto local en el blob llamando al método [upload_blob](/python/api/azure-storage-blob/azure.storage.blob.blobclient#upload-blob-data--blob-type--blobtype-blockblob---blockblob----length-none--metadata-none----kwargs-).

Agregue este código al final del bloque `try`:

```python
# Create a file in local data directory to upload and download
local_path = "./data"
local_file_name = "quickstart" + str(uuid.uuid4()) + ".txt"
upload_file_path = os.path.join(local_path, local_file_name)

# Write text to the file
file = open(upload_file_path, 'w')
file.write("Hello, World!")
file.close()

# Create a blob client using the local file name as the name for the blob
blob_client = blob_service_client.get_blob_client(container=container_name, blob=local_file_name)

print("\nUploading to Azure Storage as blob:\n\t" + local_file_name)

# Upload the created file
with open(upload_file_path, "rb") as data:
    blob_client.upload_blob(data)
```

### <a name="list-the-blobs-in-a-container"></a>Enumerar los blobs de un contenedor

Enumere los blobs en el contenedor llamando al método [list_blobs](/python/api/azure-storage-blob/azure.storage.blob.containerclient#list-blobs-name-starts-with-none--include-none----kwargs-). En este caso, solo se ha agregado un blob al contenedor, por lo que la operación de enumeración devuelve simplemente dicho blob.

Agregue este código al final del bloque `try`:

```python
print("\nListing blobs...")

# List the blobs in the container
blob_list = container_client.list_blobs()
for blob in blob_list:
    print("\t" + blob.name)
```

### <a name="download-blobs"></a>Descargar blobs

Descargue el blob creado previamente llamando al método [download_blob](/python/api/azure-storage-blob/azure.storage.blob.blobclient#download-blob-offset-none--length-none----kwargs-). El código de ejemplo agrega el sufijo "DOWNLOAD" al nombre de archivo para que pueda ver ambos archivos en el sistema de archivos local.

Agregue este código al final del bloque `try`:

```python
# Download the blob to a local file
# Add 'DOWNLOAD' before the .txt extension so you can see both files in the data directory
download_file_path = os.path.join(local_path, str.replace(local_file_name ,'.txt', 'DOWNLOAD.txt'))
print("\nDownloading blob to \n\t" + download_file_path)

with open(download_file_path, "wb") as download_file:
    download_file.write(blob_client.download_blob().readall())
```

### <a name="delete-a-container"></a>Eliminación de un contenedor

El código siguiente limpia los recursos que creó la aplicación; para ello, quita todo el contenedor con el método [delete_container](/python/api/azure-storage-blob/azure.storage.blob.containerclient#delete-container---kwargs-). Si lo desea, también puede eliminar los archivos locales.

La aplicación se detiene para la entrada del usuario mediante una llamada a `input()` antes de eliminar el blob, el contenedor y los archivos locales. Esta es una buena oportunidad para verificar que los recursos se han creado correctamente antes de eliminarlos.

Agregue este código al final del bloque `try`:

```python
# Clean up
print("\nPress the Enter key to begin clean up")
input()

print("Deleting blob container...")
container_client.delete_container()

print("Deleting the local source and downloaded files...")
os.remove(upload_file_path)
os.remove(download_file_path)

print("Done")
```

## <a name="run-the-code"></a>Ejecución del código

Esta aplicación crea un archivo de prueba en la carpeta local y lo carga en Blob Storage. Después, en el ejemplo se enumeran los blobs del contenedor y se descarga el archivo con un nombre nuevo para que pueda comparar los archivos antiguo y nuevo.

Navegue hasta el directorio que contiene el archivo *blob-quickstart-v12.py* y, a continuación, ejecute el siguiente comando de `python` para ejecutar la aplicación.

```console
python blob-quickstart-v12.py
```

La salida de la aplicación es similar a la del ejemplo siguiente:

```output
Azure Blob storage v12 - Python quickstart sample

Uploading to Azure Storage as blob:
        quickstartcf275796-2188-4057-b6fb-038352e35038.txt

Listing blobs...
        quickstartcf275796-2188-4057-b6fb-038352e35038.txt

Downloading blob to
        ./data/quickstartcf275796-2188-4057-b6fb-038352e35038DOWNLOAD.txt

Press the Enter key to begin clean up

Deleting blob container...
Deleting the local source and downloaded files...
Done
```

Antes de comenzar a limpiar el proceso, compruebe la carpeta *data* para los dos archivos. Puede abrirlos y ver que son idénticos.

Después de haber comprobado los archivos, presione la tecla **Entrar** para eliminar los archivos de prueba y terminar la demostración.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha aprendido a cargar, descargar y enumerar blobs mediante Python.

Para ver las aplicaciones de ejemplo de Blob Storage, siga estos pasos:

> [!div class="nextstepaction"]
> [Ejemplos de Azure Blob Storage v12 para Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-blob/samples)

* Para más información, consulte el artículo sobre las [bibliotecas cliente de Azure Storage para Python](/azure/developer/python/sdk/storage/overview).
* Para ver tutoriales, ejemplos, artículos de inicio rápido y otra documentación, visite [Azure para desarrolladores de Python](/azure/python/).
