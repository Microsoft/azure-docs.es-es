---
title: 'Inicio rápido: Versión 2.1 de la biblioteca cliente de Azure Blob Storage para Python'
description: En esta guía de inicio rápido, creará una cuenta de almacenamiento y un contenedor en el almacenamiento de objetos (Blob). Después, puede usar la versión 2.1 de la biblioteca cliente de Storage para Python a fin de cargar un blob en Azure Storage, descargar un blob y enumerar los blobs de un contenedor.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/24/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.custom: seo-python-october2019, devx-track-python
ms.openlocfilehash: a01b6e644ce3afda451d94d3e00eda278a645adf
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87851331"
---
# <a name="quickstart-manage-blobs-with-python-v21-sdk"></a>Inicio rápido: Administración de blobs con el SDK de Python v2.1

En este inicio rápido, se aprende a administrar blobs mediante Python. Los blobs son objetos que pueden contener grandes cantidades de texto o datos binarios, como imágenes, documentos, medios de transmisión y datos de archivo. Cargará, descargará y enumerará los blobs, y creará y eliminará los contenedores.

> [!NOTE]
> Esta guía de inicio rápido usa una versión heredada de la biblioteca cliente de Azure Blob Storage. Para empezar a trabajar con la versión más reciente, consulte [Inicio rápido: Administración de blobs con el SDK de Python, versión 12](storage-quickstart-blobs-python.md).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- Una cuenta de Azure Storage. [Crear una cuenta de almacenamiento](../common/storage-account-create.md).
- [Python](https://www.python.org/downloads/)
- [SDK de Azure Storage para Python](https://github.com/Azure/azure-sdk-for-python).

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="download-the-sample-application"></a>Descarga de la aplicación de ejemplo

La [aplicación de ejemplo](https://github.com/Azure-Samples/storage-blobs-python-quickstart.git) usada en esta guía de inicio rápido es una aplicación Python básica.  

Use el siguiente comando de [git](https://git-scm.com/) para descargar la aplicación en su entorno de desarrollo. 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-python-quickstart.git 
```

Para examinar el programa de Python, abra el archivo *example.py*, que se encuentra en la raíz del repositorio.  

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Configuración de la cadena de conexión de almacenamiento.

En la aplicación, proporcione el nombre y la clave de la cuenta de almacenamiento para crear un objeto `BlockBlobService`.

1. Abra el archivo *example.py* desde el Explorador de soluciones en el IDE.

1. Reemplace los valores `accountname` y `accountkey` por el nombre y la clave de la cuenta de almacenamiento:

    ```python
    block_blob_service = BlockBlobService(
        account_name='accountname', account_key='accountkey')
    ```

1. Guarde y cierre el archivo.

## <a name="run-the-sample"></a>Ejecución del ejemplo

El programa de ejemplo crea un archivo de prueba en la carpeta *Documentos*, carga el archivo en Blob Storage, enumera los blobs del archivo y descarga el archivo con un nuevo nombre.

1. Instale las dependencias:

    ```console
    pip install azure-storage-blob==2.1.0
    ```

1. Vaya a la aplicación de ejemplo:

    ```console
    cd storage-blobs-python-quickstart
    ```

1. Ejecución del ejemplo:

    ```console
    python example.py
    ```

    Verá mensajes similares a la siguiente salida:
  
    ```output
    Temp file = C:\Users\azureuser\Documents\QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

    Uploading to Blob storage as blobQuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

    List blobs in the container
             Blob name: QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

    Downloading blob to     C:\Users\azureuser\Documents\QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078_DOWNLOADED.txt
    ```

1. Antes de continuar, vaya a la carpeta *Documentos* y compruebe los dos archivos.

    * *QuickStart_\<universally-unique-identifier\>*
    * *QuickStart_\<universally-unique-identifier\>_DOWNLOADED*

1. Puede abrirlos y ver que son idénticos.

    También puede usar una herramienta como el [Explorador de Azure Storage](https://storageexplorer.com). Es útil para los archivos en Blob Storage. El Explorador de Azure Storage es una herramienta multiplataforma gratuita que le permite acceder a la información de su cuenta de almacenamiento. 

1. Después de haber examinado los archivos, presione cualquier tecla para finalizar la demostración y eliminar los archivos de prueba.

## <a name="learn-about-the-sample-code"></a>Información acerca del código de ejemplo

Ahora que sabe lo que hace el ejemplo, abra el archivo *example.py* para examinar el código.

### <a name="get-references-to-the-storage-objects"></a>Obtención de referencias a los objetos de almacenamiento

En esta sección, se crean instancias de los objetos, se crea un contenedor y luego se establecen permisos en el contenedor para que los blobs sean públicos. Llamará al contenedor `quickstartblobs`. 

```python
# Create the BlockBlockService that the system uses to call the Blob service for the storage account.
block_blob_service = BlockBlobService(
    account_name='accountname', account_key='accountkey')

# Create a container called 'quickstartblobs'.
container_name = 'quickstartblobs'
block_blob_service.create_container(container_name)

# Set the permission so the blobs are public.
block_blob_service.set_container_acl(
    container_name, public_access=PublicAccess.Container)
```

En primer lugar, creará las referencias a los objetos usados para acceder a Blob Storage y administrarlo. Estos objetos dependen unos de otros, y cada uno es utilizado por el siguiente de la lista.

* Cree una instancia del objeto **BlockBlobService**, que apunta a Blob service en la cuenta de almacenamiento. 

* Cree una instancia del objeto **CloudBlobContainer**, que representa el contenedor al que está accediendo. El sistema usa contenedores para organizar los blobs, de la misma forma que se usan carpetas en un equipo para organizar los archivos.

Una vez que tenga el contenedor de blobs en la nube, cree una instancia del objeto **CloudBlockBlob** que apunte al blob específico en el que está interesado. Luego, puede cargar, descargar o copiar el blob según sea necesario.

> [!IMPORTANT]
> Los nombres de contenedor deben estar en minúsculas. Para más información sobre la nomenclatura de contenedores y blobs, consulte [Naming and Referencing Containers, Blobs, and Metadata](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Asignación de nombres y referencia a contenedores, blobs y metadatos).

### <a name="upload-blobs-to-the-container"></a>Carga de blobs al contenedor

Blob Storage admite blobs en bloques, blobs en anexos y blobs en páginas. Los blobs en bloques pueden tener un tamaño de hasta 4,7 TB y pueden ser cualquier cosa, desde hojas de cálculo de Excel a archivos de vídeo grandes. Puede usar blobs en anexos para el registro cuando desee escribir en un archivo y, después, seguir agregando más información. Los blobs en páginas se usan principalmente para los archivos de disco duro virtual (VHD) que respaldan las máquinas virtuales de infraestructura como servicio (IaaS VM). Los blobs en bloques son los que se usan con más frecuencia. En este inicio rápido se usan blobs en bloques.

Para cargar un archivo en un blob, obtenga la ruta de acceso completa del archivo combinando el nombre del directorio y el nombre de archivo en la unidad local. Luego, puede cargar el archivo en la ruta de acceso especificada con el método `create_blob_from_path`. 

El código de ejemplo crea un archivo local que el sistema usará para la carga y descarga, almacenando el archivo que el sistema carga como *full_path_to_file* y el nombre del blob como *local_file_name*. En este ejemplo se carga el archivo en un contenedor llamado `quickstartblobs`:

```python
# Create a file in Documents to test the upload and download.
local_path = os.path.expanduser("~\Documents")
local_file_name = "QuickStart_" + str(uuid.uuid4()) + ".txt"
full_path_to_file = os.path.join(local_path, local_file_name)

# Write text to the file.
file = open(full_path_to_file, 'w')
file.write("Hello, World!")
file.close()

print("Temp file = " + full_path_to_file)
print("\nUploading to Blob storage as blob" + local_file_name)

# Upload the created file, use local_file_name for the blob name.
block_blob_service.create_blob_from_path(
    container_name, local_file_name, full_path_to_file)
```

Existen varios métodos de carga que puede usar con Blob Storage. Por ejemplo, si tiene una secuencia de memoria, puede usar el método `create_blob_from_stream` en lugar de `create_blob_from_path`. 

### <a name="list-the-blobs-in-a-container"></a>Enumerar los blobs de un contenedor

El siguiente código crea un elemento `generator` para el método `list_blobs`. El código recorre en bucle la lista de blobs del contenedor e imprime sus nombres en la consola.

```python
# List the blobs in the container.
print("\nList blobs in the container")
generator = block_blob_service.list_blobs(container_name)
for blob in generator:
    print("\t Blob name: " + blob.name)
```

### <a name="download-the-blobs"></a>Descarga de los blobs


Descargue los blobs en el disco local con el método `get_blob_to_path`.
El código siguiente descarga el blob que se cargó anteriormente. El sistema anexa *_DOWNLOADED* al nombre del blob para que pueda ver ambos archivos en el disco local.

```python
# Download the blob(s).
# Add '_DOWNLOADED' as prefix to '.txt' so you can see both files in Documents.
full_path_to_file2 = os.path.join(local_path, local_file_name.replace(
   '.txt', '_DOWNLOADED.txt'))
print("\nDownloading blob to " + full_path_to_file2)
block_blob_service.get_blob_to_path(
    container_name, local_file_name, full_path_to_file2)
```

### <a name="clean-up-resources"></a>Limpieza de recursos
Si ya no necesita los blobs cargados en esta guía de inicio rápido, puede eliminar todo el contenedor mediante el método `delete_container`. Si lo que quiere es eliminar archivos individuales, use el método `delete_blob`.

```python
# Clean up resources. This includes the container and the temp files.
block_blob_service.delete_container(container_name)
os.remove(full_path_to_file)
os.remove(full_path_to_file2)
```

## <a name="resources-for-developing-python-applications-with-blobs"></a>Recursos para el desarrollo de aplicaciones Python con blobs

Para más información sobre el desarrollo de Python con Blob Storage, consulte estos recursos adicionales:

### <a name="binaries-and-source-code"></a>Archivos binarios y código fuente

- Consulte, descargue e instale el [código fuente de la biblioteca de cliente de Python](https://github.com/Azure/azure-storage-python) para Azure Storage en GitHub.

### <a name="client-library-reference-and-samples"></a>Referencia a la biblioteca de clientes y ejemplos

- Para más información sobre la biblioteca cliente de Python, consulte las [bibliotecas de Azure Storage para Python](https://docs.microsoft.com/python/api/overview/azure/storage).
- Explore los [ejemplos de Blob Storage](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=python&term=blob) escritos mediante la biblioteca de cliente de Python.

## <a name="next-steps"></a>Pasos siguientes
 
En esta guía de inicio rápido aprendió a transferir archivos entre un disco local y Azure Blob Storage mediante Python. 

Para más información sobre el Explorador de Storage y los blobs, consulte [Administración de recursos de Azure Blob Storage con el Explorador de Storage (versión preliminar)](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
