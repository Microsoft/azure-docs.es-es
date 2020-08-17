---
title: Administración de Azure Data Lake Analytics con Python
description: En este artículo se describe cómo usar Python para administrar cuentas, orígenes de datos, usuarios y trabajos de Data Lake Analytics.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: d4213a19-4d0f-49c9-871c-9cd6ed7cf731
ms.topic: how-to
ms.date: 06/08/2018
ms.custom: devx-track-python
ms.openlocfilehash: dd3511fefe39f51a521aa211454a49763224a6d9
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876247"
---
# <a name="manage-azure-data-lake-analytics-using-python"></a>Administración de Azure Data Lake Analytics con Python
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

En este artículo se describe cómo administrar cuentas, orígenes de datos, usuarios y trabajos de Azure Data Lake Analytics mediante Python.

## <a name="supported-python-versions"></a>Versiones de Python compatibles

* Use una versión de Python de 64 bits.
* Puede usar la distribución estándar de Python que encontrará en la sección de **[descargas de Python.org](https://www.python.org/downloads/)** . 
* Muchos desarrolladores consideran conveniente usar la **[distribución de Python Anaconda](https://www.anaconda.com/download/)** .  
* Este artículo se escribió para la versión 3.6 de Python con la distribución de Python estándar

## <a name="install-azure-python-sdk"></a>Instalación del SDK de Python de Azure

Instale los siguientes módulos:

* El módulo **azure-mgmt-resource** incluye otros módulos de Azure para Active Directory, etc.
* El módulo **azure-datalake-store** incluye las operaciones de sistema de archivos de Azure Data Lake Store. 
* El módulo **azure-datalake-store** incluye las operaciones de administración de cuentas de Azure Data Lake Store.
* El módulo **azure-mgmt-datalake-analytics** incluye las operaciones de Azure Data Lake Analytics. 

En primer lugar, asegúrese de que dispone del último `pip`; para ello, ejecute el comando siguiente:

```console
python -m pip install --upgrade pip
```

Este documento se ha escrito con `pip version 9.0.1`.

Use el comando `pip` siguiente para instalar los módulos desde la línea de comandos:

```console
pip install azure-mgmt-resource
pip install azure-datalake-store
pip install azure-mgmt-datalake-store
pip install azure-mgmt-datalake-analytics
```

## <a name="create-a-new-python-script"></a>Creación de un nuevo script de Python

Pegue el código siguiente en el script:

```python
# Use this only for Azure AD service-to-service authentication
#from azure.common.credentials import ServicePrincipalCredentials

# Use this only for Azure AD end-user authentication
#from azure.common.credentials import UserPassCredentials

# Required for Azure Resource Manager
from azure.mgmt.resource.resources import ResourceManagementClient
from azure.mgmt.resource.resources.models import ResourceGroup

# Required for Azure Data Lake Store account management
from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
from azure.mgmt.datalake.store.models import DataLakeStoreAccount

# Required for Azure Data Lake Store filesystem management
from azure.datalake.store import core, lib, multithread

# Required for Azure Data Lake Analytics account management
from azure.mgmt.datalake.analytics.account import DataLakeAnalyticsAccountManagementClient
from azure.mgmt.datalake.analytics.account.models import DataLakeAnalyticsAccount, DataLakeStoreAccountInformation

# Required for Azure Data Lake Analytics job management
from azure.mgmt.datalake.analytics.job import DataLakeAnalyticsJobManagementClient
from azure.mgmt.datalake.analytics.job.models import JobInformation, JobState, USqlJobProperties

# Required for Azure Data Lake Analytics catalog management
from azure.mgmt.datalake.analytics.catalog import DataLakeAnalyticsCatalogManagementClient

# Use these as needed for your application
import logging
import getpass
import pprint
import uuid
import time
```

Ejecute este script para comprobar que se pueden importar los módulos.

## <a name="authentication"></a>Authentication

### <a name="interactive-user-authentication-with-a-pop-up"></a>Autenticación interactiva de usuarios con elemento emergente

Este método no se admite.

### <a name="interactive-user-authentication-with-a-device-code"></a>Autenticación interactiva de usuarios con código de dispositivo

```python
user = input(
    'Enter the user to authenticate with that has permission to subscription: ')
password = getpass.getpass()
credentials = UserPassCredentials(user, password)
```

### <a name="noninteractive-authentication-with-spi-and-a-secret"></a>Autenticación no interactiva con SPI y secreto

```python
credentials = ServicePrincipalCredentials(
    client_id='FILL-IN-HERE', secret='FILL-IN-HERE', tenant='FILL-IN-HERE')
```

### <a name="noninteractive-authentication-with-api-and-a-certificate"></a>Autenticación no interactiva con API y certificado

Este método no se admite.

## <a name="common-script-variables"></a>Variables de script comunes

Estas variables se usan en los ejemplos.

```python
subid = '<Azure Subscription ID>'
rg = '<Azure Resource Group Name>'
location = '<Location>'  # i.e. 'eastus2'
adls = '<Azure Data Lake Store Account Name>'
adla = '<Azure Data Lake Analytics Account Name>'
```

## <a name="create-the-clients"></a>Creación de los clientes

```python
resourceClient = ResourceManagementClient(credentials, subid)
adlaAcctClient = DataLakeAnalyticsAccountManagementClient(credentials, subid)
adlaJobClient = DataLakeAnalyticsJobManagementClient(
    credentials, 'azuredatalakeanalytics.net')
```

## <a name="create-an-azure-resource-group"></a>Creación de un grupo de recursos de Azure

```python
armGroupResult = resourceClient.resource_groups.create_or_update(
    rg, ResourceGroup(location=location))
```

## <a name="create-data-lake-analytics-account"></a>Creación de una cuenta de Análisis de Data Lake

En primer lugar, cree una cuenta de almacenamiento.

```python
adlsAcctResult = adlsAcctClient.account.create(
    rg,
    adls,
    DataLakeStoreAccount(
        location=location)
    )
).wait()
```
A continuación, cree una cuenta de ADLA que utilice ese almacén.

```python
adlaAcctResult = adlaAcctClient.account.create(
    rg,
    adla,
    DataLakeAnalyticsAccount(
        location=location,
        default_data_lake_store_account=adls,
        data_lake_store_accounts=[DataLakeStoreAccountInformation(name=adls)]
    )
).wait()
```

## <a name="submit-a-job"></a>Enviar un trabajo

```python
script = """
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
"""

jobId = str(uuid.uuid4())
jobResult = adlaJobClient.job.create(
    adla,
    jobId,
    JobInformation(
        name='Sample Job',
        type='USql',
        properties=USqlJobProperties(script=script)
    )
)
```

## <a name="wait-for-a-job-to-end"></a>Esperar a que finalice un trabajo

```python
jobResult = adlaJobClient.job.get(adla, jobId)
while(jobResult.state != JobState.ended):
    print('Job is not yet done, waiting for 3 seconds. Current state: ' +
          jobResult.state.value)
    time.sleep(3)
    jobResult = adlaJobClient.job.get(adla, jobId)

print('Job finished with result: ' + jobResult.result.value)
```

## <a name="list-pipelines-and-recurrences"></a>Enumerar las canalizaciones y las repeticiones
Dependiendo de si los trabajos tienen metadatos adjuntos de canalización o repetición, puede enumerar las canalizaciones y las repeticiones.

```python
pipelines = adlaJobClient.pipeline.list(adla)
for p in pipelines:
    print('Pipeline: ' + p.name + ' ' + p.pipelineId)

recurrences = adlaJobClient.recurrence.list(adla)
for r in recurrences:
    print('Recurrence: ' + r.name + ' ' + r.recurrenceId)
```

## <a name="manage-compute-policies"></a>Administración de nodos directivas de proceso

El objeto de DataLakeAnalyticsAccountManagementClient proporciona métodos para administrar las directivas de proceso de una cuenta de Data Lake Analytics.

### <a name="list-compute-policies"></a>Enumeración de directivas de proceso

El código siguiente recupera una lista de directivas de proceso de una cuenta de Data Lake Analytics.

```python
policies = adlaAccountClient.computePolicies.listByAccount(rg, adla)
for p in policies:
    print('Name: ' + p.name + 'Type: ' + p.objectType + 'Max AUs / job: ' +
          p.maxDegreeOfParallelismPerJob + 'Min priority / job: ' + p.minPriorityPerJob)
```

### <a name="create-a-new-compute-policy"></a>Creación de una nueva directiva de proceso

El siguiente código crea una nueva directiva de cálculo para una cuenta de análisis de Data Lake y establece que el número máximo de AU disponibles para el usuario especificado en 50 y la prioridad del trabajo mínimo en 250.

```python
userAadObjectId = "3b097601-4912-4d41-b9d2-78672fc2acde"
newPolicyParams = ComputePolicyCreateOrUpdateParameters(
    userAadObjectId, "User", 50, 250)
adlaAccountClient.computePolicies.createOrUpdate(
    rg, adla, "GaryMcDaniel", newPolicyParams)
```

## <a name="next-steps"></a>Pasos siguientes

- Para ver el mismo tutorial con otras herramientas, haga clic en los selectores de pestañas en la parte superior de la página.
- Para obtener más información sobre U-SQL, consulte [Introducción al lenguaje U-SQL de Análisis de Azure Data Lake](data-lake-analytics-u-sql-get-started.md).
- Para conocer las tareas de administración, consulte [Administración de Azure Data Lake Analytics mediante el Azure Portal](data-lake-analytics-manage-use-portal.md).

