---
title: 'Autenticación entre servicios de Python: Data Lake Storage Gen1'
description: Aprenda a realizar la autenticación de servicio a servicio con Azure Data Lake Storage Gen1 mediante Azure Active Directory usando Python.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: has-adal-ref, devx-track-python
ms.openlocfilehash: 95da32c6f684946bf6d394c282246f2f329b8620
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87873983"
---
# <a name="service-to-service-authentication-with-azure-data-lake-storage-gen1-using-python"></a>Autenticación de servicio a servicio con Azure Data Lake Storage Gen1 mediante Python
> [!div class="op_single_selector"]
> * [Uso de Java](data-lake-store-service-to-service-authenticate-java.md)
> * [Uso de SDK de .NET](data-lake-store-service-to-service-authenticate-net-sdk.md)
> * [Uso de Python](data-lake-store-service-to-service-authenticate-python.md)
> * [Uso de la API de REST](data-lake-store-service-to-service-authenticate-rest-api.md)
>
>

En este artículo, obtendrá información sobre cómo usar el SDK de Python para realizar la autenticación de servicio a servicio con Azure Data Lake Storage Gen1. Para la autenticación del usuario final con Data Lake Storage Gen1 mediante Python, vea [End-user authentication with Data Lake Storage using Python](data-lake-store-end-user-authenticate-python.md) (Autenticación del usuario final con Data Lake Storage Gen1 mediante Python).


## <a name="prerequisites"></a>Prerrequisitos

* **Python** Python se puede descargar desde [aquí](https://www.python.org/downloads/). En este artículo se usa Python 3.6.2.

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

* **Cree una aplicación "web" de Azure Active Directory**. Debe haber completado los pasos descritos en [Service-to-service authentication with Data Lake Storage Gen1 using Azure Active Directory](data-lake-store-service-to-service-authenticate-using-active-directory.md) (Autenticación entre servicios con Data Lake Storage Gen1 mediante Azure Active Directory).

## <a name="install-the-modules"></a>Instalación de los módulos

Para trabajar con Data Lake Storage Gen1 mediante Python, debe instalar tres módulos.

* El módulo `azure-mgmt-resource`, que incluye los módulos de Azure para Active Directory, etc.
* El módulo `azure-mgmt-datalake-store`, que incluye las operaciones de administración de cuentas de Data Lake Storage Gen1. Para más información acerca de este módulo, consulte [referencia al módulo de Azure Data Lake Storage Gen1 Management](/python/api/azure-mgmt-datalake-store/).
* El módulo `azure-datalake-store`, que incluye las operaciones del sistema de archivos de Data Lake Storage Gen1. Para obtener más información sobre este módulo, consulte la [referencia al módulo de azure-datalake-store Filesystem](https://docs.microsoft.com/python/api/azure-datalake-store/azure.datalake.store.core/).

Utilice el comando siguiente para instalar los módulos.

```
pip install azure-mgmt-resource
pip install azure-mgmt-datalake-store
pip install azure-datalake-store
```

## <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

1. En el IDE que prefiera, cree una nueva aplicación de Python, por ejemplo, **mysample.py**.

2. Agregue el fragmento de código siguiente para importar los módulos necesarios

    ```
    ## Use this for Azure AD authentication
    from msrestazure.azure_active_directory import AADTokenCredentials

    ## Required for Data Lake Storage Gen1 account management
    from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
    from azure.mgmt.datalake.store.models import DataLakeStoreAccount

    ## Required for Data Lake Storage Gen1 filesystem management
    from azure.datalake.store import core, lib, multithread

    # Common Azure imports
    import adal
    from azure.mgmt.resource.resources import ResourceManagementClient
    from azure.mgmt.resource.resources.models import ResourceGroup

    ## Use these as needed for your application
    import logging, getpass, pprint, uuid, time
    ```

3. Guarde los cambios en mysample.py.

## <a name="service-to-service-authentication-with-client-secret-for-account-management"></a>Autenticación de servicio a servicio con el secreto de cliente para la administración de cuentas

Use este fragmento de código para autenticarse en Azure AD y realizar operaciones de administración de cuentas en Data Lake Storage Gen1, tales como crear o eliminar una cuenta de Data Lake Storage Gen1, entre otras. El siguiente fragmento de código se puede utilizar para autenticar la aplicación de forma no interactiva, para lo que se usa el secreto del cliente para una entidad de servicio o aplicación de una aplicación de "aplicación web" de Azure AD existente.

```python
authority_host_uri = 'https://login.microsoftonline.com'
tenant = '<TENANT>'
authority_uri = authority_host_uri + '/' + tenant
RESOURCE = 'https://management.core.windows.net/'
client_id = '<CLIENT_ID>'
client_secret = '<CLIENT_SECRET>'

context = adal.AuthenticationContext(authority_uri, api_version=None)
mgmt_token = context.acquire_token_with_client_credentials(RESOURCE, client_id, client_secret)
armCreds = AADTokenCredentials(mgmt_token, client_id, resource=RESOURCE)
```

## <a name="service-to-service-authentication-with-client-secret-for-filesystem-operations"></a>Autenticación de servicio a servicio con el secreto de cliente para las operaciones del sistema de archivos

Use el siguiente fragmento de código para autenticarse en Azure AD y realizar operaciones del sistema de archivos en Data Lake Storage Gen1, como crear una carpeta, cargar un archivo, etc. El siguiente fragmento de código se puede utilizar para autenticar la aplicación de forma no interactiva, para lo que se usa el secreto del cliente para una entidad de servicio o aplicación. Utilícelo con una aplicación de "aplicación web" de Azure AD.

```python
tenant = '<TENANT>'
RESOURCE = 'https://datalake.azure.net/'
client_id = '<CLIENT_ID>'
client_secret = '<CLIENT_SECRET>'

adlCreds = lib.auth(tenant_id = tenant,
                client_secret = client_secret,
                client_id = client_id,
                resource = RESOURCE)
```

<!-- ## Service-to-service authentication with certificate for account management

Use this snippet to authenticate with Azure AD for account management operations on Data Lake Storage Gen1 such as create a Data Lake Storage Gen1 account, delete a Data Lake Storage Gen1 account, etc. The following snippet can be used to authenticate your application non-interactively, using the certificate of an existing Azure AD "Web App" application. For instructions on how to create an Azure AD application, see [Create service principal with certificates](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-self-signed-certificate).

    authority_host_uri = 'https://login.microsoftonline.com'
    tenant = '<TENANT>'
    authority_uri = authority_host_uri + '/' + tenant
    resource_uri = 'https://management.core.windows.net/'
    client_id = '<CLIENT_ID>'
    client_cert = '<CLIENT_CERT>'
    client_cert_thumbprint = '<CLIENT_CERT_THUMBPRINT>'

    context = adal.AuthenticationContext(authority_uri, api_version=None)
    mgmt_token = context.acquire_token_with_client_certificate(resource_uri, client_id, client_cert, client_cert_thumbprint)
    credentials = AADTokenCredentials(mgmt_token, client_id) -->

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha aprendido a usar la autenticación de servicio a servicio para autenticarse en Data Lake Storage Gen1 con Python. Ahora puede consultar los siguientes artículos, que tratan sobre cómo usar Python para trabajar con Data Lake Storage Gen1.

* [Account management operations on Data Lake Storage Gen1 using Python](data-lake-store-get-started-python.md) (Operaciones de administración de cuentas en Data Lake Storage Gen1 con Python)
* [Operaciones de datos en Data Lake Storage Gen1 mediante Python](data-lake-store-data-operations-python.md)
