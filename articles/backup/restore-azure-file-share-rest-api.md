---
title: Restauración de recursos compartidos de archivos de Azure con la API REST
description: Aprenda a usar la API REST para restaurar recursos compartidos de archivos de Azure a partir de un punto de restauración creado con Azure Backup.
ms.topic: conceptual
ms.date: 02/17/2020
ms.openlocfilehash: 60c73caa5db684e38b94b4d5786f2fd24aa65d08
ms.sourcegitcommit: f1b18ade73082f12fa8f62f913255a7d3a7e42d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88761804"
---
# <a name="restore-azure-file-shares-using-rest-api"></a>Restauración de recursos compartidos de archivos de Azure con la API REST

En este artículo se explica cómo restaurar un recurso compartido de archivos completo o archivos específicos desde un punto de restauración creado por [Azure Backup](./backup-overview.md) mediante API REST.

Al acabar este tutorial, habrá aprendido cómo realizar las siguientes operaciones con API REST:

* Visualización de los puntos de restauración de un recurso compartido de archivos de Azure con copia de seguridad.
* Restauración de un recurso compartido de archivos de Azure completo.
* Restauración de archivos o carpetas individuales.

## <a name="prerequisites"></a>Prerrequisitos

Damos por sentado que ya tiene un recurso compartido de archivos que quiere restaurar. Si no es así, consulte [Copia de seguridad de recurso compartido de archivos de Azure con API REST](backup-azure-file-share-rest-api.md) para aprender a crear uno.

En este artículo, usaremos los siguientes recursos:

* **Almacén de Recovery Services**: *azurefilesvault*
* **Grupo de recursos**: *azurefiles*
* **Cuenta de almacenamiento**: *afsaccount*
* **Recurso compartido de archivos**: *azurefiles*

## <a name="fetch-containername-and-protecteditemname"></a>Captura de ContainerName y ProtectedItemName

Para la mayoría de las llamadas API relacionadas con la restauración, debe pasar valores para los parámetros de URI {containerName} y {protectedItemName}. Use el atributo ID en el cuerpo de respuesta de la operación [GET backupprotectableitems](/rest/api/backup/protecteditems/get) para recuperar los valores de estos parámetros. En nuestro ejemplo, el identificador del recurso compartido de archivos que queremos proteger es el siguiente:

`"/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;afsaccount/protectableItems/azurefileshare;azurefiles`

Por lo tanto, los valores se convierten de la siguiente manera:

* {containername}: *storagecontainer;storage;azurefiles;afsaccount*
* {protectedItemName}: *azurefileshare;azurefiles*

## <a name="fetch-recovery-points-for-backed-up-azure-file-share"></a>Captura de puntos de recuperación para el recurso compartido de archivos de Azure con copia de seguridad

Para restaurar los archivos o recursos compartidos de archivos con copia de seguridad, seleccione primero un punto de recuperación para realizar la operación de restauración. Se pueden enumerar los puntos de recuperación disponibles de un elemento con copia de seguridad mediante la llamada API REST [para obtener la lista e puntos de recuperación](/rest/api/site-recovery/recoverypoints/listbyreplicationprotecteditems). Es una operación GET con todos los valores pertinentes.

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/recoveryPoints?api-version=2019-05-13&$filter={$filter}
```

Establezca los valores de URI de la manera siguiente:

* {fabricName}: *Azure*
* {vaultName}: *azurefilesvault*
* {containername}: *storagecontainer;storage;azurefiles;afsaccount*
* {protectedItemName}: *azurefileshare;azurefiles*
* {ResourceGroupName}: *azurefiles*

El identificador URI de GET tiene todos los parámetros necesarios. No es necesario un cuerpo de solicitud adicional.

```http
GET https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;afsaccount/protectedItems/AzureFileShare;azurefiles/recoveryPoints?api-version=2019-05-13
```

### <a name="example-response-for-fetch-recovery-points"></a>Respuesta de ejemplo para la captura de puntos de recuperación

Una vez que se emita el identificador URI de GET, se devuelve una respuesta 200:

```http
HTTP/1.1" 200 None
'Cache-Control': 'no-cache'
'Pragma': 'no-cache'
'Transfer-Encoding': 'chunked'
'Content-Type': 'application/json'
'Content-Encoding': 'gzip'
'Expires': '-1'
'Vary': 'Accept-Encoding'
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': 'd68d7951-7d97-4c49-9a2d-7fbaab55233a'
'x-ms-client-request-id': '4edb5a58-47ea-11ea-a27a-0a580af41908, 4edb5a58-47ea-11ea-a27a-0a580af41908'
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'Server': 'Microsoft-IIS/10.0'
'X-Powered-By': 'ASP.NET'
'x-ms-ratelimit-remaining-subscription-reads': '11998'
'x-ms-correlation-request-id': 'd68d7951-7d97-4c49-9a2d-7fbaab55233a'
'x-ms-routing-request-id': 'WESTEUROPE:20200205T073708Z:d68d7951-7d97-4c49-9a2d-7fbaab55233a'
'Date': 'Wed, 05 Feb 2020 07:37:08 GMT'
{
“value”:[
  {
    "eTag": null,
    "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;afsaccount/protectedItems/AzureFileShare;azurefiles/recoveryPoints/932881138555802864",
    "location": null,
    "name": "932881138555802864",
    "properties": {
      "fileShareSnapshotUri": "https://afsaccount.file.core.windows.net/azurefiles?sharesnapshot=2020-02-04T08:01:35.0000000Z",
      "objectType": "AzureFileShareRecoveryPoint",
      "recoveryPointSizeInGb": 1,
      "recoveryPointTime": "2020-02-04T08:01:35+00:00",
      "recoveryPointType": "FileSystemConsistent"
    },
    "resourceGroup": "azurefiles",
    "tags": null,
    "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints"
  },
  {
    "eTag": null,
    "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;afsaccount/protectedItems/AzureFileShare;azurefiles/recoveryPoints/932878582606969225",
    "location": null,
    "name": "932878582606969225",
    "properties": {
      "fileShareSnapshotUri": "https://afsaccount.file.core.windows.net/azurefiles?sharesnapshot=2020-02-03T08:05:30.0000000Z",
      "objectType": "AzureFileShareRecoveryPoint",
      "recoveryPointSizeInGb": 1,
      "recoveryPointTime": "2020-02-03T08:05:30+00:00",
      "recoveryPointType": "FileSystemConsistent"
    },
    "resourceGroup": "azurefiles",
    "tags": null,
    "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints"
  },
  {
    "eTag": null,
    "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;afsaccount/protectedItems/AzureFileShare;azurefiles/recoveryPoints/932890167574511261",
    "location": null,
    "name": "932890167574511261",
    "properties": {
      "fileShareSnapshotUri": "https://afsaccount.file.core.windows.net/azurefiles?sharesnapshot=2020-02-02T08:03:50.0000000Z",
      "objectType": "AzureFileShareRecoveryPoint",
      "recoveryPointSizeInGb": 1,
      "recoveryPointTime": "2020-02-02T08:03:50+00:00",
      "recoveryPointType": "FileSystemConsistent"
    },
    "resourceGroup": "azurefiles",
    "tags": null,
    "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints"
  },
```

El punto de recuperación se identifica con el campo {name} en la respuesta anterior.

## <a name="full-share-recovery-using-rest-api"></a>Recuperación de recursos compartidos completos mediante API REST

Use esta opción de restauración para restaurar el recurso compartido de archivos completo en la ubicación original o en otra alternativa.
Desencadenar la restauración es una solicitud POST y puede realizar esta operación mediante API REST [para desencadenar la restauración](/rest/api/backup/restores/trigger).

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/recoveryPoints/{recoveryPointId}/restore?api-version=2019-05-13
```

Los valores {containerName} y {protectedItemName} son los que se establecen [aquí](#fetch-containername-and-protecteditemname) y recoveryPointID es el campo {name} del punto de recuperación mencionado anteriormente.

```http
POST https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;afsaccount/protectedItems/AzureFileShare%3Bazurefiles/recoveryPoints/932886657837421071/restore?api-version=2019-05-13'
```

### <a name="create-request-body"></a>Creación del cuerpo de la solicitud

Para desencadenar una restauración para un recurso compartido de archivos, a continuación se incluyen los componentes del cuerpo de la solicitud:

Nombre |  Tipo   |   Descripción
--- | ---- | ----
Propiedades | AzureFileShareRestoreRequest | Propiedades de RestoreRequestResource

Para obtener una lista completa de las definiciones del cuerpo de la solicitud y otros detalles, consulte el [documento de API REST para desencadenar la restauración](/rest/api/backup/restores/trigger#request-body).

### <a name="restore-to-original-location"></a>Restauración en una ubicación original

#### <a name="request-body-example-for-restore-to-original-location"></a>Ejemplo de cuerpo de la solicitud para efectuar la restauración en la ubicación original

El cuerpo de solicitud siguiente define las propiedades necesarias para desencadenar una restauración del recurso compartido de archivos de Azure:

```json
{
   "properties":{
      "objectType":"AzureFileShareRestoreRequest",
      "recoveryType":"OriginalLocation",
      "sourceResourceId":"/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/afsaccount",
      "copyOptions":"Overwrite",
      "restoreRequestType":"FullShareRestore"
}
}
```

### <a name="restore-to-alternate-location"></a>Restauración a una ubicación alternativa

Especifique los parámetros siguientes para la recuperación en una ubicación alternativa:

* **targetResourceId**: La cuenta de almacenamiento en la que se restaura el contenido de la copia de seguridad. La cuenta de almacenamiento de destino debe estar en la misma ubicación que el almacén.
* **name**: recurso compartido de archivos de la cuenta de almacenamiento de destino en la que se restaura el contenido de la copia de seguridad.
* **targetFolderPath**: La carpeta del recurso compartido de archivos en la que se restauran los datos.

#### <a name="request-body-example-for-restore-to-alternate-location"></a>Ejemplo de cuerpo de la solicitud para efectuar la restauración en una ubicación alternativa

El siguiente cuerpo de la solicitud restaura el recurso compartido de archivos *azurefiles* de la cuenta de almacenamiento *afsaccount* en el recurso compartido de archivos *azurefiles1* de la cuenta de almacenamiento *afaccount1*.

```json
{
   "properties":{
      "objectType":"AzureFileShareRestoreRequest",
      "recoveryType":"AlternateLocation",
      "sourceResourceId":"/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/afsaccount",
      "copyOptions":"Overwrite",
      "restoreRequestType":"FullShareRestore",
      "restoreFileSpecs":[
         {
            "targetFolderPath":"restoredata"
}
],
      "targetDetails":{
         "name":"azurefiles1",
         "targetResourceId":"/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/afaccount1"
}
}
}
```

### <a name="response"></a>Response

La activación de una operación de restauración es una [operación asincrónica](../azure-resource-manager/management/async-operations.md). Esta operación crea otra operación de la que se debe hacer un seguimiento por separado.
Devuelve las dos respuestas: 202 (Aceptado) cuando se crea otra operación y 200 (Correcto) cuando se completa dicha operación.

#### <a name="response-example"></a>Ejemplo de respuesta

Una vez enviado el URI de *POST* para desencadenar una restauración, la respuesta inicial es 202 (Aceptado) con un encabezado de ubicación o Azure-async-header.

```http
HTTP/1.1" 202
'Cache-Control': 'no-cache'
'Pragma': 'no-cache'
'Expires': '-1'
'Location': 'https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;afsaccount/protectedItems/AzureFileShare;azurefiles/operationResults/68ccfbc1-a64f-4b29-b955-314b5790cfa9?api-version=2019-05-13'
'Retry-After': '60'
'Azure-AsyncOperation': 'https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;afsaccount/protectedItems/AzureFileShare;azurefiles/operationsStatus/68ccfbc1-a64f-4b29-b955-314b5790cfa9?api-version=2019-05-13'
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': '2426777d-c5ec-44b6-a324-384f8947460c'
'x-ms-client-request-id': '3c743096-47eb-11ea-ae90-0a580af41908, 3c743096-47eb-11ea-ae90-0a580af41908'
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'X-Powered-By': 'ASP.NET'
'x-ms-ratelimit-remaining-subscription-writes': '1198'
'x-ms-correlation-request-id': '2426777d-c5ec-44b6-a324-384f8947460c'
'x-ms-routing-request-id': 'WESTEUROPE:20200205T074347Z:2426777d-c5ec-44b6-a324-384f8947460c'
'Date': 'Wed, 05 Feb 2020 07:43:47 GMT'
```

A continuación, realice un seguimiento de la operación resultante con el encabezado de ubicación o el encabezado Azure-AsyncOperation y un comando GET.

```http
GET https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupOperations/68ccfbc1-a64f-4b29-b955-314b5790cfa9?api-version=2016-12-01
```

Una vez completada la operación, devuelve 200 (OK) con el identificador del trabajo de restauración resultante en el cuerpo de respuesta.

```http
HTTP/1.1" 200
'Cache-Control': 'no-cache'
'Pragma': 'no-cache'
'Transfer-Encoding': 'chunked'
'Content-Type': 'application/json'
'Content-Encoding': 'gzip'
'Expires': '-1'
'Vary': 'Accept-Encoding'
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': '41ee89b2-3be4-40d8-8ff6-f5592c2571e3'
'x-ms-client-request-id': '3c743096-47eb-11ea-ae90-0a580af41908, 3c743096-47eb-11ea-ae90-0a580af41908'
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'Server': 'Microsoft-IIS/10.0'
'X-Powered-By': 'ASP.NET'
'x-ms-ratelimit-remaining-subscription-reads': '11998'
'x-ms-correlation-request-id': '41ee89b2-3be4-40d8-8ff6-f5592c2571e3'
'x-ms-routing-request-id': 'WESTEUROPE:20200205T074348Z:41ee89b2-3be4-40d8-8ff6-f5592c2571e3'
'Date': 'Wed, 05 Feb 2020 07:43:47 GMT'
{
  "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupJobs/a7e97e42-4e54-4d4b-b449-26fcf946f42c",
  "location": null,
  "name": "a7e97e42-4e54-4d4b-b449-26fcf946f42c",
  "properties": {
    "actionsInfo": [
      "Cancellable"
    ],
    "activityId": "3c743096-47eb-11ea-ae90-0a580af41908",
    "backupManagementType": "AzureStorage",
    "duration": "0:00:01.863098",
    "endTime": null,
    "entityFriendlyName": "azurefiles",
    "errorDetails": null,
    "extendedInfo": {
      "dynamicErrorMessage": null,
      "propertyBag": {},
      "tasksList": []
    },
    "jobType": "AzureStorageJob",
    "operation": "Restore",
    "startTime": "2020-02-05T07:43:47.144961+00:00",
    "status": "InProgress",
    "storageAccountName": "afsaccount",
    "storageAccountVersion": "Storage"
  },
  "resourceGroup": "azurefiles",
  "tags": null,
  "type": "Microsoft.RecoveryServices/vaults/backupJobs"
}
```

En la recuperación de ubicación alternativa, el cuerpo de la respuesta será similar al siguiente:

```http
{
  "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupJobs/7e0ee41e-6e31-4728-a25c-98ff6b777641",
  "location": null,
  "name": "7e0ee41e-6e31-4728-a25c-98ff6b777641",
  "properties": {
    "actionsInfo": [
      "Cancellable"
    ],
    "activityId": "6077be6e-483a-11ea-a915-0a580af4ad72",
    "backupManagementType": "AzureStorage",
    "duration": "0:00:02.171965",
    "endTime": null,
    "entityFriendlyName": "azurefiles",
    "errorDetails": null,
    "extendedInfo": {
      "dynamicErrorMessage": null,
      "propertyBag": {
        "Data Transferred (in MB)": "0",
        "Job Type": "Recover to an alternate file share",
        "Number Of Failed Files": "0",
        "Number Of Restored Files": "0",
        "Number Of Skipped Files": "0",
        "RestoreDestination": "afaccount1/azurefiles1/restoredata",
        "Source File Share Name": "azurefiles",
        "Source Storage Account Name": "afsaccount",
        "Target File Share Name": "azurefiles1",
        "Target Storage Account Name": "afaccount1"
      },
      "tasksList": []
    },
    "jobType": "AzureStorageJob",
    "operation": "Restore",
    "startTime": "2020-02-05T17:10:18.106532+00:00",
    "status": "InProgress",
    "storageAccountName": "afsaccount",
    "storageAccountVersion": "ClassicCompute"
  },
  "resourceGroup": "azurefiles",
  "tags": null,
  "type": "Microsoft.RecoveryServices/vaults/backupJobs"
}
```

Puesto que el trabajo de copia de seguridad es una operación de larga duración, se debe seguir como se explica en el [documento sobre la supervisión de trabajos con API REST](./backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

## <a name="item-level-recovery-using-rest-api"></a>Recuperación de nivel de elemento mediante API REST

Puede usar esta opción para restaurar archivos o carpetas en la ubicación original o en otra alternativa.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/recoveryPoints/{recoveryPointId}/restore?api-version=2019-05-13
```

Los valores {containerName} y {protectedItemName} son los que se establecen [aquí](#fetch-containername-and-protecteditemname) y recoveryPointID es el campo {name} del punto de recuperación mencionado anteriormente.

```http
POST https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;afsaccount/protectedItems/AzureFileShare%3Bazurefiles/recoveryPoints/932886657837421071/restore?api-version=2019-05-13'
```

### <a name="create-request-body-for-item-level-recovery-using-rest-api"></a>Creación del cuerpo de la solicitud para la recuperación de nivel de elemento mediante la API de REST

Para desencadenar una restauración para un recurso compartido de archivos, a continuación se incluyen los componentes del cuerpo de la solicitud:

Nombre |  Tipo   |   Descripción
--- | ---- | ----
Propiedades | AzureFileShareRestoreRequest | Propiedades de RestoreRequestResource

Para obtener una lista completa de las definiciones del cuerpo de la solicitud y otros detalles, consulte el [documento de API REST para desencadenar la restauración](/rest/api/backup/restores/trigger#request-body).

### <a name="restore-to-original-location-for-item-level-recovery-using-rest-api"></a>Restauración en la ubicación original para la recuperación de nivel de elemento mediante la API de REST

El siguiente cuerpo de solicitud es para restaurar el archivo *Restoretest.txt* en el recurso compartido de archivos *azurefiles* de la cuenta de almacenamiento *afsaccount*.

Creación del cuerpo de la solicitud

```json
{
   "properties":{
      "objectType":"AzureFileShareRestoreRequest",
      "copyOptions":"Overwrite",
      "recoveryType":"OriginalLocation",
      "restoreFileSpecs":[
         {
            "fileSpecType":"File",
            "path":"RestoreTest.txt",
            "targetFolderPath":null
}
],
      "restoreRequestType":"ItemLevelRestore",
      "sourceResourceId":"/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.storage/storageAccounts/afsaccount",
      "targetDetails":null
}
}
```

### <a name="restore-to-alternate-location-for-item-level-recovery-using-rest-api"></a>Restauración en una ubicación alternativa para la recuperación de nivel de elemento mediante la API de REST

El siguiente cuerpo de la solicitud es para restaurar el archivo *Restoretest.txt* en el recurso compartido de archivos *azurefiles* de la cuenta de almacenamiento *afsaccount* en la carpeta *restoredata* del recurso compartido de archivos *azurefiles1* de la cuenta de almacenamiento *afaccount1*.

Creación del cuerpo de la solicitud

```json
{
   "properties":{
      "objectType":"AzureFileShareRestoreRequest",
      "recoveryType":"AlternateLocation",
      "sourceResourceId":"/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/afsaccount",
      "copyOptions":"Overwrite",
      "restoreRequestType":"ItemLevelRestore",
      "restoreFileSpecs":[
         {
            "path":"Restore/RestoreTest.txt",
            "fileSpecType":"File",
            "targetFolderPath":"restoredata"
}
],
      "targetDetails":{
         "name":"azurefiles1",
         "targetResourceId":"/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/afaccount1"
}
}
}
```

La respuesta debe controlarse de la misma manera que se explicó anteriormente para las [restauraciones de recursos compartidos completos](#full-share-recovery-using-rest-api).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [administrar copias de seguridad de recursos compartidos de archivos de Azure mediante API REST](manage-azure-file-share-rest-api.md).
