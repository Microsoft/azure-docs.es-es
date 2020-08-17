---
title: 'Autenticación de usuario final en Python con Data Lake Storage Gen1: Azure'
description: Aprenda a lograr la autenticación del usuario final con Azure Data Lake Storage Gen1 mediante Azure Active Directory con Python
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: has-adal-ref, devx-track-python
ms.openlocfilehash: c3262a2802da8dbd50fc05f7aa2fea71623c9b42
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87872198"
---
# <a name="end-user-authentication-with-azure-data-lake-storage-gen1-using-python"></a>Autenticación de usuario final con Azure Data Lake Storage Gen1 mediante el uso de Python
> [!div class="op_single_selector"]
> * [Uso de Java](data-lake-store-end-user-authenticate-java-sdk.md)
> * [Uso de SDK de .NET](data-lake-store-end-user-authenticate-net-sdk.md)
> * [Uso de Python](data-lake-store-end-user-authenticate-python.md)
> * [Uso de la API de REST](data-lake-store-end-user-authenticate-rest-api.md)
>
>

En este artículo, aprenderá a usar el SDK de Python para realizar la autenticación de usuario final con Azure Data Lake Storage Gen1. La autenticación del usuario final se puede dividir en dos categorías:

* Autenticación de usuario final sin autenticación multifactor
* Autenticación de usuario final con autenticación multifactor

Ambas opciones se describen en este artículo. Para la autenticación entre servicios con Data Lake Storage Gen1. mediante Python, consulte [Service-to-service authentication with Data Lake Storage Gen1 using Java](data-lake-store-service-to-service-authenticate-python.md) (Autenticación entre servicios con Data Lake Storage Gen1 mediante Python).

## <a name="prerequisites"></a>Prerrequisitos

* **Python** Python se puede descargar desde [aquí](https://www.python.org/downloads/). En este artículo se usa Python 3.6.2.

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

* **Cree una aplicación "nativa" de Azure Active Directory**. Debe completar los pasos descritos en [Autenticación de usuario final con Data Lake Storage Gen1 mediante Azure Active Directory](data-lake-store-end-user-authenticate-using-active-directory.md).

## <a name="install-the-modules"></a>Instalación de los módulos

Para trabajar con Data Lake Storage Gen1 mediante Python, debe instalar tres módulos.

* El módulo `azure-mgmt-resource`, que incluye los módulos de Azure para Active Directory, etc.
* El módulo `azure-mgmt-datalake-store`, que incluye las operaciones de administración de cuentas de Azure Data Lake Storage Gen1. Para más información acerca de este módulo, consulte [referencia al módulo de Azure Data Lake Storage Gen1 Management](/python/api/azure-mgmt-datalake-store/).
* El módulo `azure-datalake-store`, que incluye las operaciones de sistema de archivos de Azure Data Lake Storage Gen1. Para obtener más información sobre este módulo, consulte la [referencia al módulo de azure-datalake-store Filesystem](https://docs.microsoft.com/python/api/azure-datalake-store/azure.datalake.store.core/).

Utilice el comando siguiente para instalar los módulos.

```console
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

    ## Required for Azure Data Lake Storage Gen1 account management
    from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
    from azure.mgmt.datalake.store.models import DataLakeStoreAccount

    ## Required for Azure Data Lake Storage Gen1 filesystem management
    from azure.datalake.store import core, lib, multithread

    # Common Azure imports
    import adal
    from azure.mgmt.resource.resources import ResourceManagementClient
    from azure.mgmt.resource.resources.models import ResourceGroup

    ## Use these as needed for your application
    import logging, pprint, uuid, time
    ```

3. Guarde los cambios en mysample.py.

## <a name="end-user-authentication-with-multi-factor-authentication"></a>Autenticación de usuario final con autenticación multifactor

### <a name="for-account-management"></a>Para la administración de cuentas

Use el siguiente fragmento de código para autenticarse en Azure AD para las operaciones de administración de cuentas de una cuenta de Data Lake Storage Gen1. El siguiente fragmento de código se puede utilizar para autenticar una aplicación mediante Multi-Factor Authentication. Especifique los valores siguientes para una aplicación **nativa** de Azure AD existente.

```python
authority_host_url = "https://login.microsoftonline.com"
tenant = "FILL-IN-HERE"
authority_url = authority_host_url + '/' + tenant
client_id = 'FILL-IN-HERE'
redirect = 'urn:ietf:wg:oauth:2.0:oob'
RESOURCE = 'https://management.core.windows.net/'

context = adal.AuthenticationContext(authority_url)
code = context.acquire_user_code(RESOURCE, client_id)
print(code['message'])
mgmt_token = context.acquire_token_with_device_code(RESOURCE, code, client_id)
armCreds = AADTokenCredentials(mgmt_token, client_id, resource = RESOURCE)
```

### <a name="for-filesystem-operations"></a>Para operaciones del sistema de archivos

Se utiliza para autenticarse en Azure AD para las operaciones del sistema de archivos en una cuenta de Data Lake Storage Gen1. El siguiente fragmento de código se puede utilizar para autenticar una aplicación mediante Multi-Factor Authentication. Especifique los valores siguientes para una aplicación **nativa** de Azure AD existente.

```console
adlCreds = lib.auth(tenant_id='FILL-IN-HERE', resource = 'https://datalake.azure.net/')
```

## <a name="end-user-authentication-without-multi-factor-authentication"></a>Autenticación de usuario final sin autenticación multifactor

Esto está en desuso. Para más información, consulte [Autenticación de Azure mediante el SDK de Python](/azure/python/python-sdk-azure-authenticate).

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha aprendido a usar la autenticación de usuario final para autenticarse en Azure Data Lake Storage Gen1 mediante Python. Ahora puede consultar los siguientes artículos, que tratan sobre cómo usar Python con Azure Data Lake Storage Gen1.

* [Account management operations on Data Lake Storage Gen1 using Python](data-lake-store-get-started-python.md) (Operaciones de administración de cuentas en Data Lake Storage Gen1 con Python)
* [Operaciones de datos en Data Lake Storage Gen1 mediante Python](data-lake-store-data-operations-python.md)
