---
title: Administración de copias de seguridad de recursos compartidos de archivos de Azure con API REST
description: Aprenda a usar API REST para administrar y supervisar los recursos compartidos de archivos de Azure de los que se ha creado una copia de seguridad con Azure Backup.
ms.topic: conceptual
ms.date: 02/17/2020
ms.openlocfilehash: 5e2823472c6a7bdd6b3f9819db3079d7efa78c4e
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88892854"
---
# <a name="manage-azure-file-share-backup-with-rest-api"></a>Administración de copias de seguridad de recursos compartidos de archivos de Azure con API REST

En este artículo se explica cómo realizar tareas para administrar y supervisar los recursos compartidos de archivos de Azure de los que se ha realizado una copia de seguridad mediante [Azure Backup](./backup-overview.md).

## <a name="monitor-jobs"></a>Supervisión de trabajos

El servicio Azure Backup desencadena trabajos que se ejecutan en segundo plano. Esto incluye escenarios como desencadenar copias de seguridad, operaciones de restauración y deshabilitar la copia de seguridad. Estos trabajos se pueden seguir mediante sus identificadores.

### <a name="fetch-job-information-from-operations"></a>Captura de información sobre el trabajo de las operaciones

Una operación como desencadenar la copia de seguridad siempre devolverá un valor de jobID en la respuesta.

Por ejemplo, la respuesta final de una [operación para desencadenar una copia de seguridad en API REST](backup-azure-file-share-rest-api.md#trigger-an-on-demand-backup-for-file-share) es como sigue:

```json
{
    "id": "c3a52d1d-0853-4211-8141-477c65740264",
    "name": "c3a52d1d-0853-4211-8141-477c65740264",
    "status": "Succeeded",
    "startTime": "2020-02-03T18:10:48.296012Z",
    "endTime": "2020-02-03T18:10:48.296012Z",
    "properties": {
        "objectType": "OperationStatusJobExtendedInfo",
        "jobId": "e2ca2cf4-2eb9-4d4b-b16a-8e592d2a658b"
    }
}
```

El trabajo de copia de seguridad del recurso compartido de archivos se identifica mediante el campo **jobId** y se puede realizar su seguimiento como se menciona [aquí](/rest/api/backup/jobdetails/) mediante una solicitud GET.

### <a name="tracking-the-job"></a>Seguimiento del trabajo

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupJobs/{jobName}?api-version=2019-05-13
```

{JobName} es el valor de "jobId" mencionado anteriormente. La respuesta es siempre "200 (Correcto)" y el campo **status** indica el estado del trabajo. Cuando el estado sea "Completed" o "CompletedWithWarnings", la sección **extendedInfo** ofrecerá más detalles sobre el trabajo.

```http
GET https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupJobs/e2ca2cf4-2eb9-4d4b-b16a-8e592d2a658b?api-version=2019-05-13'
```

#### <a name="response"></a>Response

Nombre  | Tipo  |  Descripción
--- | --- | ----
200 OK |  JobResource  | Aceptar

#### <a name="response-example"></a>Ejemplo de respuesta

Una vez que se envía el identificador URI de *GET*, se devuelve una respuesta 200.

```http
HTTP/1.1" 200
'Cache-Control': 'no-cache'
'Pragma': 'no-cache'
'Transfer-Encoding': 'chunked'
'Content-Type': 'application/json'
'Content-Encoding': 'gzip'
'Expires': '-1'
'Vary': 'Accept-Encoding'
'Server': 'Microsoft-IIS/10.0, Microsoft-IIS/10.0'
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': 'dba43f00-5cdb-43b1-a9ec-23e419db67c5'
'x-ms-client-request-id': 'a644712a-4895-11ea-ba57-0a580af42708, a644712a-4895-11ea-ba57-0a580af42708'
'X-Powered-By': 'ASP.NET'
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'x-ms-ratelimit-remaining-subscription-reads': '11999'
'x-ms-correlation-request-id': 'dba43f00-5cdb-43b1-a9ec-23e419db67c5'
'x-ms-routing-request-id': 'WESTEUROPE:20200206T040341Z:dba43f00-5cdb-43b1-a9ec-23e419db67c5'
'Date': 'Thu, 06 Feb 2020 04:03:40 GMT'
{
    "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupJobs/e2ca2cf4-2eb9-4d4b-b16a-8e592d2a658b",
    "name": "e2ca2cf4-2eb9-4d4b-b16a-8e592d2a658b",
    "type": "Microsoft.RecoveryServices/vaults/backupJobs",
    "properties": {
        "jobType": "AzureStorageJob",
        "duration": "00:00:43.1809140",
        "storageAccountName": "testvault2",
        "storageAccountVersion": "Storage",
        "extendedInfo": {
            "tasksList": [],
            "propertyBag": {
                "File Share Name": "testshare",
                "Storage Account Name": "testvault2",
                "Policy Name": "schedule1"
            }
        },
        "entityFriendlyName": "testshare",
        "backupManagementType": "AzureStorage",
        "operation": "ConfigureBackup",
        "status": "Completed",
        "startTime": "2020-02-03T18:10:48.296012Z",
        "endTime": "2020-02-03T18:11:31.476926Z",
        "activityId": "3677cec0-942d-4eac-921f-8f3c873140d7"
    }
}
```

## <a name="modify-policy"></a>Modificación de directivas

Para cambiar la directiva con la que se protege el recurso compartido de archivos, puede usar el mismo formato que para habilitar la protección. Basta con que proporcione el identificador de la nueva directiva en la directiva de solicitud y envíe la solicitud.

Por ejemplo: Para cambiar la directiva de protección de *testshare* de *schedule1* a *schedule2*, proporcione el identificador *schedule2* en el cuerpo de la solicitud.

```json
{
  "properties": {
    "protectedItemType": "AzureFileShareProtectedItem",
    "sourceResourceId": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
    "policyId": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupPolicies/schedule2"
  }
}
```

## <a name="stop-protection-but-retain-existing-data"></a>Detener la protección, pero conservar los datos existentes

Puede quitar la protección de un recurso compartido de archivos protegido, pero conservar los datos de los que ya se ha realizado una copia de seguridad. Para ello, quite la directiva del cuerpo de la solicitud que usó para[habilitar la copia de seguridad](backup-azure-file-share-rest-api.md#enable-backup-for-the-file-share) y enviar la solicitud. Una vez que se quita la asociación con la directiva, ya no se desencadenan las copias de seguridad ni se crea ningún punto de recuperación nuevo.

```json
{
  "properties": {
    "protectedItemType": "AzureFileShareProtectedItem",
    "sourceResourceId": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
    "policyId": “" ,
“protectionState”:”ProtectionStopped”
  }
}
```

### <a name="sample-response"></a>Respuesta de muestra

Detener la protección de un recurso compartido de archivos es una operación asincrónica. La operación crea otra de la que se tiene que hacer el seguimiento. Devuelve las dos respuestas: 202 (Aceptado) cuando se crea otra operación y 200 cuando se completa dicha operación.

Encabezado de respuesta cuando la operación se acepta correctamente:

```http
HTTP/1.1" 202
'Cache-Control': 'no-cache'
'Pragma': 'no-cache'
'Expires': '-1'
'Location': 'https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/operationResults/b300922a-ad9c-4181-b4cd-d42ea780ad77?api-version=2019-05-13'
'Retry-After': '60'
msrest.http_logger :     'Azure-AsyncOperation': 'https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/operationsStatus/b300922a-ad9c-4181-b4cd-d42ea780ad77?api-version=2019-05-13'
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': '3895e8a1-e4b9-4da5-bec7-2cf0266405f8'
'x-ms-client-request-id': 'd331c15e-48ab-11ea-84c0-0a580af46a50, d331c15e-48ab-11ea-84c0-0a580af46a50'
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'X-Powered-By': 'ASP.NET'
'x-ms-ratelimit-remaining-subscription-writes': '1199'
'x-ms-correlation-request-id': '3895e8a1-e4b9-4da5-bec7-2cf0266405f8'
'x-ms-routing-request-id': 'WESTEUROPE:20200206T064224Z:3895e8a1-e4b9-4da5-bec7-2cf0266405f8'
'Date': 'Thu, 06 Feb 2020 06:42:24 GMT'
'Content-Length': '0'
```

A continuación, realice un seguimiento de la operación resultante con el encabezado de ubicación o el encabezado Azure-AsyncOperation y un comando GET:

```http
GET https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupoperations/b300922a-ad9c-4181-b4cd-d42ea780ad77?api-version=2016-12-01
```

### <a name="response-body"></a>Response body

```json
{
    "id": "b300922a-ad9c-4181-b4cd-d42ea780ad77",
    "name": "b300922a-ad9c-4181-b4cd-d42ea780ad77",
    "status": "Succeeded",
    "startTime": "2020-02-06T06:42:24.4001299Z",
    "endTime": "2020-02-06T06:42:24.4001299Z",
    "properties": {
        "objectType": "OperationStatusJobExtendedInfo",
        "jobId": "7816fca8-d5be-4c41-b911-1bbd922e5826"
    }
}
```

## <a name="stop-protection-and-delete-data"></a>Detener la protección y eliminar los datos

Para quitar la protección de un recurso compartido de archivos y eliminar también los datos de la copia de seguridad, realice una operación de eliminación como se detalla [aquí](/rest/api/backup/protecteditems/delete).

```http
DELETE https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}?api-version=2019-05-13
```

Los parámetros {containerName} y {protectedItemName} son los que se establecen [aquí](restore-azure-file-share-rest-api.md#fetch-containername-and-protecteditemname).

En el ejemplo siguiente se desencadena una operación para detener la protección del recurso compartido de archivos *testshare* protegido con *azurefilesvault*.

```http
DELETE https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/protectedItems/azurefileshare;testshare?api-version=2016-12-01
```

### <a name="responses"></a>Respuestas

La eliminación de la protección es una operación asincrónica. La operación crea otra operación de la que se debe hacer un seguimiento por separado.
Devuelve las dos respuestas: 202 (Aceptado) cuando se crea otra operación y 204 (Sin contenido) cuando se completa dicha operación.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [solucionar problemas al configurar la copia de seguridad para recursos compartidos de archivos de Azure](troubleshoot-azure-files.md).
