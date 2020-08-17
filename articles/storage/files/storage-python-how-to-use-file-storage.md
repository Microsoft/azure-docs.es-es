---
title: Desarrollo para Azure Files con Python | Microsoft Docs
description: Aprenda a desarrollar aplicaciones y servicios de Python que utilizan Azure Files para almacenar datos de archivos.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 12/14/2018
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-python
ms.openlocfilehash: 46512d61238c45936e7ebed4310993159cb43d34
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87845449"
---
# <a name="develop-for-azure-files-with-python"></a>Desarrollo para Azure Files con Python
[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

[!INCLUDE [storage-try-azure-tools-files](../../../includes/storage-try-azure-tools-files.md)]

Este tutorial mostrará los aspectos básicos del uso de Python para desarrollar aplicaciones o servicios que utilizan Azure Files para almacenar datos de archivos. En este tutorial, se creará una aplicación de consola simple y se mostrará cómo realizar las acciones básicas con Python y Azure Files:

* Crear recursos compartidos de archivos de Azure
* Crear directorios
* Enumerar los archivos y directorios de un recurso compartido de Azure File
* Cargar, descargar y eliminar un archivo

> [!Note]  
> Dado que se puede acceder a Azure Files a través de SMB, es posible escribir aplicaciones simples que accedan al recurso compartido de archivos de Azure mediante las clases y funciones estándar de E/S de Python. En este artículo se describe cómo escribir aplicaciones que usen el SDK de Python de Azure Storage, que emplea la [API de REST de Azure Files](https://docs.microsoft.com/rest/api/storageservices/file-service-rest-api) para comunicarse con Azure Files.

## <a name="download-and-install-azure-storage-sdk-for-python"></a>Descarga e instalación del SDK de Azure Storage para Python

El [SDK de Azure Storage para Python](https://github.com/azure/azure-storage-python) requiere Python 2.7, 3.3, 3.4, 3.5 o 3.6.
 
## <a name="install-via-pypi"></a>Instalación mediante PyPi

Para realizar la instalación mediante el índice de paquetes de Python (PyPI), escriba:

```bash
pip install azure-storage-file
```

> [!NOTE]
> Si va a actualizar desde el SDK de Azure Storage para Python 0.36 o una versión anterior, desinstale el SDK anterior mediante `pip uninstall azure-storage` antes de instalar el paquete más reciente.

Si quiere conocer métodos de instalación alternativos, consulte el [SDK de Azure Storage para Python en GitHub](https://github.com/Azure/azure-storage-python/).

## <a name="view-the-sample-application"></a>Visualización de la aplicación de ejemplo
Para ver y ejecutar una aplicación de ejemplo que muestra cómo usar Python con Azure Files, consulte [Azure Storage: Getting Started with Azure Files in Python](https://github.com/Azure-Samples/storage-file-python-getting-started) (Azure Storage: Introducción a Azure Files en Python). 

Para ejecutar la aplicación de ejemplo, asegúrese de que ha instalado los paquetes `azure-storage-file` y `azure-storage-common`.

## <a name="set-up-your-application-to-use-azure-files"></a>Configuración de la aplicación para usar Azure Files
Agregue lo siguiente cerca de la parte superior de todo archivo de origen de Python en el que desee acceder a Azure Storage mediante programación.

```python
from azure.storage.file import FileService
```

## <a name="set-up-a-connection-to-azure-files"></a>Configuración de una conexión a Azure Files 
El objeto `FileService` le permite trabajar con recursos compartidos, directorios y archivos. El código siguiente crea un objeto `FileService` mediante el nombre de la cuenta de almacenamiento y la clave de la cuenta. Reemplace `<myaccount>` y `<mykey>` por el nombre y la clave de la de cuenta.

```python
file_service = FileService(account_name='myaccount', account_key='mykey')
```

## <a name="create-an-azure-file-share"></a>Creación de un recurso compartido de archivos de Azure
En el siguiente código de ejemplo, puede usar un objeto `FileService` para crear el recurso compartido, en caso de que no exista.

```python
file_service.create_share('myshare')
```

## <a name="create-a-directory"></a>Creación de un directorio
También puede organizar el almacenamiento colocando archivos dentro de los subdirectorios en lugar de mantenerlos a todos en el directorio raíz. Azure Files permite crear tantos directorios como permita su cuenta. El código siguiente creará un subdirectorio denominado **sampledir** bajo el directorio raíz.

```python
file_service.create_directory('myshare', 'sampledir')
```

## <a name="enumerate-files-and-directories-in-an-azure-file-share"></a>Enumerar los archivos y directorios de un recurso compartido de Azure File
Para mostrar los archivos y los directorios de un recurso compartido, use el método **list\_directories\_and\_files**. Este método devuelve un generador. El código siguiente ofrece el **nombre** de todos los archivos y el directorio de un recurso compartido de la consola.

```python
generator = file_service.list_directories_and_files('myshare')
for file_or_dir in generator:
    print(file_or_dir.name)
```

## <a name="upload-a-file"></a>Cargar un archivo 
Un recurso compartido de archivos de Azure contiene como mínimo un directorio raíz donde pueden residir los archivos. En esta sección, aprenderá cómo cargar un archivo del almacenamiento local en el directorio raíz de un recurso compartido.

Para crear un archivo y actualizar los datos, use los métodos `create_file_from_path`, `create_file_from_stream`, `create_file_from_bytes` o `create_file_from_text`. Se trata de métodos de alto nivel que realizan la fragmentación necesaria cuando el tamaño de los datos supera los 64 MB.

`create_file_from_path` carga el contenido de un archivo de la ruta de acceso especificada y `create_file_from_stream` carga el contenido de un archivo o secuencia ya abiertos. `create_file_from_bytes` carga una matriz de bytes y `create_file_from_text`carga el valor de texto especificado con la codificación especificada (el valor predeterminado es UTF-8).

En el siguiente ejemplo se carga el contenido del archivo **sunset.png** en el archivo **myfile**.

```python
from azure.storage.file import ContentSettings
file_service.create_file_from_path(
    'myshare',
    None,  # We want to create this blob in the root directory, so we specify None for the directory_name
    'myfile',
    'sunset.png',
    content_settings=ContentSettings(content_type='image/png'))
```

## <a name="download-a-file"></a>Descarga de un archivo
Para descargar los datos de un archivo, use `get_file_to_path`, `get_file_to_stream`, `get_file_to_bytes` o `get_file_to_text`. Se trata de métodos de alto nivel que realizan la fragmentación necesaria cuando el tamaño de los datos supera los 64 MB.

En el ejemplo siguiente se muestra cómo usar `get_file_to_path` para descargar el contenido en el archivo **myfile** y almacenarlo en el archivo **out-sunset.png**.

```python
file_service.get_file_to_path('myshare', None, 'myfile', 'out-sunset.png')
```

## <a name="delete-a-file"></a>Eliminación de un archivo
Finalmente, para eliminar un archivo, llame a `delete_file`.

```python
file_service.delete_file('myshare', None, 'myfile')
```

## <a name="create-share-snapshot"></a>Creación de una instantánea de recurso compartido
Puede crear una copia a un punto dado de todo el recurso compartido de archivos.

```python
snapshot = file_service.snapshot_share(share_name)
snapshot_id = snapshot.snapshot
```

**Creación de una instantánea de recurso compartido con metadatos**

```python
metadata = {"foo": "bar"}
snapshot = file_service.snapshot_share(share_name, metadata=metadata)
```

## <a name="list-shares-and-snapshots"></a>Enumeración de recursos compartidos e instantáneas 
Puede enumerar todas las instantáneas para un recurso compartido determinado.

```python
shares = list(file_service.list_shares(include_snapshots=True))
```

## <a name="browse-share-snapshot"></a>Examinación de instantáneas de recurso compartido
Puede examinar el contenido de cada instantánea de recurso compartido para recuperar archivos y directorios desde ese punto dado.

```python
directories_and_files = list(
    file_service.list_directories_and_files(share_name, snapshot=snapshot_id))
```

## <a name="get-file-from-share-snapshot"></a>Obtención del archivo desde una instantánea de recurso compartido
Puede descargar un archivo desde una instantánea de recurso compartido para el escenario de restauración.

```python
with open(FILE_PATH, 'wb') as stream:
    file = file_service.get_file_to_stream(
        share_name, directory_name, file_name, stream, snapshot=snapshot_id)
```

## <a name="delete-a-single-share-snapshot"></a>Eliminación de una instantánea de recurso compartido única  
Puede eliminar una instantánea de recurso compartido única.

```python
file_service.delete_share(share_name, snapshot=snapshot_id)
```

## <a name="delete-share-when-share-snapshots-exist"></a>Eliminación de un recurso compartido cuando existen instantáneas de recurso compartido
No se puede eliminar un recurso compartido que contenga instantáneas a no ser que todas las instantáneas se eliminen primero.

```python
file_service.delete_share(share_name, delete_snapshots=DeleteSnapshot.Include)
```

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha aprendido a manipular Azure Files con Python, siga estos vínculos para obtener más información.

* [Centro para desarrolladores de Python](https://azure.microsoft.com/develop/python/)
* [API de REST de servicios de Azure Storage](https://msdn.microsoft.com/library/azure/dd179355)
* [SDK de Microsoft Azure Storage para Python](https://github.com/Azure/azure-storage-python)
