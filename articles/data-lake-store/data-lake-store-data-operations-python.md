---
title: 'Python: operaciones de sistema de archivos en Azure Data Lake Storage Gen1 | Microsoft Docs'
description: Aprenda a usar el SDK de Python para trabajar con el sistema de archivos de Data Lake Storage Gen1.
services: data-lake-store
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: devx-track-python
ms.openlocfilehash: 5cbf75ef2d9f0c1a65179e6a423b59f9763ce72b
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87872249"
---
# <a name="filesystem-operations-on-azure-data-lake-storage-gen1-using-python"></a>Operaciones de sistema de archivos en Azure Data Lake Storage Gen1 mediante el uso de Python
> [!div class="op_single_selector"]
> * [SDK de .NET](data-lake-store-data-operations-net-sdk.md)
> * [SDK de Java](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
> 

Con este artículo aprenderá a utilizar el SDK de Python para realizar operaciones de sistema de archivos en Azure Data Lake Storage Gen1. Para instrucciones sobre cómo realizar operaciones de administración de cuentas en Data Lake Storage Gen1 con Python, consulte [Operaciones de administración de cuentas en Data Lake Storage Gen1 con Python](data-lake-store-get-started-python.md).

## <a name="prerequisites"></a>Prerequisites

* **Python** Python se puede descargar desde [aquí](https://www.python.org/downloads/). En este artículo se usa Python 3.6.2.

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

* **Cuenta de Azure Data Lake Storage Gen1**. Siga las instrucciones de [Introducción a Azure Data Lake Storage Gen1 con Azure Portal](data-lake-store-get-started-portal.md).

## <a name="install-the-modules"></a>Instalación de los módulos

Para trabajar con Data Lake Storage Gen1 mediante Python, debe instalar tres módulos.

* El módulo `azure-mgmt-resource`, que incluye los módulos de Azure para Active Directory, etc.
* El módulo `azure-mgmt-datalake-store`, que incluye las operaciones de administración de cuentas de Azure Data Lake Storage Gen1. Para más información acerca de este módulo, consulte [referencia al módulo de azure-mgmt-datalake-store](/python/api/azure-mgmt-datalake-store/).
* El módulo `azure-datalake-store`, que incluye las operaciones de sistema de archivos de Azure Data Lake Storage Gen1. Para más información acerca de este módulo, consulte la [referencia al módulo de azure-datalake-store file-system](https://docs.microsoft.com/python/api/azure-datalake-store/azure.datalake.store.core/).

Utilice el comando siguiente para instalar los módulos.

```console
pip install azure-mgmt-resource
pip install azure-mgmt-datalake-store
pip install azure-datalake-store
```

## <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

1. En el IDE que prefiera, cree una nueva aplicación de Python, por ejemplo, **mysample.py**.

2. Agregue las líneas siguientes para importar los módulos necesarios.

   ```python
   ## Use this only for Azure AD service-to-service authentication
   from azure.common.credentials import ServicePrincipalCredentials

   ## Use this only for Azure AD end-user authentication
   from azure.common.credentials import UserPassCredentials

   ## Use this only for Azure AD multi-factor authentication
   from msrestazure.azure_active_directory import AADTokenCredentials

   ## Required for Azure Data Lake Storage Gen1 account management
   from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
   from azure.mgmt.datalake.store.models import DataLakeStoreAccount

   ## Required for Azure Data Lake Storage Gen1 filesystem management
   from azure.datalake.store import core, lib, multithread

   ## Common Azure imports
   from azure.mgmt.resource.resources import ResourceManagementClient
   from azure.mgmt.resource.resources.models import ResourceGroup

   ## Use these as needed for your application
   import logging, getpass, pprint, uuid, time
   ```

3. Guarde los cambios en mysample.py.

## <a name="authentication"></a>Authentication

En esta sección, se explican las distintas maneras de autenticarse con Azure AD. Las opciones disponibles son:

* Para la autenticación del usuario final para la aplicación, consulte [Autenticación de usuario final con Azure Data Lake Storage Gen1 mediante el uso de Python](data-lake-store-end-user-authenticate-python.md).
* Para la autenticación entre servicios para la aplicación, consulte [Autenticación entre servicios con Data Lake Storage Gen1 usando Python](data-lake-store-service-to-service-authenticate-python.md).

## <a name="create-filesystem-client"></a>Creación del cliente del sistema de archivos

En el siguiente fragmento de código se crea primero el cliente de la cuenta de Data Lake Storage Gen1. Usa el objeto de cliente para crear una cuenta de Data Lake Storage Gen1. Por último, el fragmento de código crea un objeto de cliente del sistema de archivos.

```python
## Declare variables
subscriptionId = 'FILL-IN-HERE'
adlsAccountName = 'FILL-IN-HERE'

## Create a filesystem client object
adlsFileSystemClient = core.AzureDLFileSystem(adlCreds, store_name=adlsAccountName)
```

## <a name="create-a-directory"></a>Creación de un directorio

```python
## Create a directory
adlsFileSystemClient.mkdir('/mysampledirectory')
```

## <a name="upload-a-file"></a>Cargar un archivo

```python
## Upload a file
multithread.ADLUploader(adlsFileSystemClient, lpath='C:\\data\\mysamplefile.txt', rpath='/mysampledirectory/mysamplefile.txt', nthreads=64, overwrite=True, buffersize=4194304, blocksize=4194304)
```


## <a name="download-a-file"></a>Descarga de un archivo

```python
## Download a file
multithread.ADLDownloader(adlsFileSystemClient, lpath='C:\\data\\mysamplefile.txt.out', rpath='/mysampledirectory/mysamplefile.txt', nthreads=64, overwrite=True, buffersize=4194304, blocksize=4194304)
```

## <a name="delete-a-directory"></a>Eliminación de un directorio

```python
## Delete a directory
adlsFileSystemClient.rm('/mysampledirectory', recursive=True)
```

## <a name="next-steps"></a>Pasos siguientes
* [Operaciones de administración de cuentas en Data Lake Storage Gen1 con Python](data-lake-store-get-started-python.md).

## <a name="see-also"></a>Consulte también

* [Referencia de Azure Data Lake Storage Gen1 con Python (sistema de archivos)](https://docs.microsoft.com/python/api/azure-datalake-store/azure.datalake.store.core)
* [Abrir aplicaciones de macrodatos de código abierto que funcionan con Azure Data Lake Storage Gen1](data-lake-store-compatible-oss-other-applications.md)
