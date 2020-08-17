---
title: Administración de Azure Data Lake Storage Gen1 con Python
description: Aprenda a usar el SDK de Python para realizar operaciones de administración de cuentas de Azure Data Lake Storage Gen1.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: devx-track-python
ms.openlocfilehash: 57414828394e7f241cd33b7d56190a3ce9a7eab2
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876007"
---
# <a name="account-management-operations-on-azure-data-lake-storage-gen1-using-python"></a>Operaciones de administración de cuentas en Azure Data Lake Storage Gen1 mediante Python.
> [!div class="op_single_selector"]
> * [SDK de .NET](data-lake-store-get-started-net-sdk.md)
> * [REST API](data-lake-store-get-started-rest-api.md)
> * [Python](data-lake-store-get-started-python.md)
>
>

Aprenda a usar el SDK de Python para Azure Data Lake Storage Gen1 para realizar operaciones de administración de cuentas básicas, como la creación o la enumeración de cuentas de Data Lake Storage Gen1. Para obtener instrucciones sobre cómo realizar operaciones del sistema de archivos en Data Lake Storage Gen1 con Python, consulte [Filesystem operations on Data Lake Storage Gen1 using Python](data-lake-store-data-operations-python.md) (Operaciones del sistema de archivos en Data Lake Storage Gen1 con Python).

## <a name="prerequisites"></a>Prerrequisitos

* **Python** Python se puede descargar desde [aquí](https://www.python.org/downloads/). En este artículo se usa Python 3.6.2.

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

* **Un grupo de recursos de Azure**. Para obtener instrucciones, vea [Creación de un grupo de recursos de Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).

## <a name="install-the-modules"></a>Instalación de los módulos

Para trabajar con Data Lake Storage Gen1 mediante Python, debe instalar tres módulos.

* El módulo `azure-mgmt-resource`, que incluye los módulos de Azure para Active Directory, etc.
* El módulo `azure-mgmt-datalake-store`, que incluye las operaciones de administración de cuentas de Azure Data Lake Storage Gen1. Para más información acerca de este módulo, consulte [referencia al módulo de Azure Data Lake Storage Gen1 Management](/python/api/azure-mgmt-datalake-store/).
* El módulo `azure-datalake-store`, que incluye las operaciones de sistema de archivos de Azure Data Lake Storage Gen1. Para obtener más información acerca de este módulo, consulte [referencia al módulo de azure-datalake-store Filesystem](https://docs.microsoft.com/python/api/azure-datalake-store/azure.datalake.store.core/).

Utilice el comando siguiente para instalar los módulos.

```console
pip install azure-mgmt-resource
pip install azure-mgmt-datalake-store
pip install azure-datalake-store
```

## <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

1. En el IDE que prefiera, cree una nueva aplicación de Python, por ejemplo, **mysample.py**.

2. Agregue el fragmento de código siguiente para importar los módulos necesarios

    ```python
    ## Use this only for Azure AD service-to-service authentication
    from azure.common.credentials import ServicePrincipalCredentials

    ## Use this only for Azure AD end-user authentication
    from azure.common.credentials import UserPassCredentials

    ## Use this only for Azure AD multi-factor authentication
    from msrestazure.azure_active_directory import AADTokenCredentials

    ## Required for Data Lake Storage Gen1 account management
    from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
    from azure.mgmt.datalake.store.models import DataLakeStoreAccount

    ## Required for Data Lake Storage Gen1 filesystem management
    from azure.datalake.store import core, lib, multithread

    # Common Azure imports
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

## <a name="create-client-and-data-lake-storage-gen1-account"></a>Crear un cliente y una cuenta de Data Lake Storage Gen1.

En el siguiente fragmento de código se crea primero el cliente de la cuenta de Data Lake Storage Gen1. Usa el objeto de cliente para crear una cuenta de Data Lake Storage Gen1. Por último, el fragmento de código crea un objeto de cliente del sistema de archivos.

```python
## Declare variables
subscriptionId = 'FILL-IN-HERE'
adlsAccountName = 'FILL-IN-HERE'
resourceGroup = 'FILL-IN-HERE'
location = 'eastus2'

## Create Data Lake Storage Gen1 account management client object
adlsAcctClient = DataLakeStoreAccountManagementClient(armCreds, subscriptionId)

## Create a Data Lake Storage Gen1 account
adlsAcctResult = adlsAcctClient.account.create(
    resourceGroup,
    adlsAccountName,
    DataLakeStoreAccount(
        location=location
    )
).wait()
```

    
## <a name="list-the-data-lake-storage-gen1-accounts"></a>Enumerar las cuentas de Data Lake Storage Gen1.

```python
## List the existing Data Lake Storage Gen1 accounts
result_list_response = adlsAcctClient.account.list()
result_list = list(result_list_response)
for items in result_list:
    print(items)
```

## <a name="delete-the-data-lake-storage-gen1-account"></a>Eliminar la cuenta de Data Lake Storage Gen1

```python
## Delete an existing Data Lake Storage Gen1 account
adlsAcctClient.account.delete(adlsAccountName)
```
    

## <a name="next-steps"></a>Pasos siguientes
* [Filesystem operations on Data Lake Storage Gen1 using Python](data-lake-store-data-operations-python.md) (Operaciones del sistema de archivos en Data Lake Storage Gen1 mediante Python).

## <a name="see-also"></a>Consulte también

* [azure-datalake-store Python (Filesystem) reference](https://docs.microsoft.com/python/api/azure-datalake-store/azure.datalake.store.core) [Referencia de azure-datalake-store Python (sistema de archivos)]
* [Abrir aplicaciones de macrodatos de código abierto que funcionan con Azure Data Lake Storage Gen1](data-lake-store-compatible-oss-other-applications.md)
