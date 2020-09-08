---
title: Copia de seguridad de recursos compartidos de archivos de Azure con API REST
description: Aprenda a usar la API REST para realizar copias de seguridad de recursos compartidos de archivos de Azure en el almacén de Recovery Services
ms.topic: conceptual
ms.date: 02/16/2020
ms.openlocfilehash: 8d2d8ed88da133986540a293185c8e37000ab87b
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88824872"
---
# <a name="backup-azure-file-share-using-azure-backup-via-rest-api"></a>Copia de seguridad de un recurso compartido de archivos de Azure con Azure Backup mediante API REST

En este artículo se describe cómo hacer una copia de seguridad de un recurso compartido de archivos con Azure Backup mediante la API REST.

En este artículo se da por sentado que ya ha creado un almacén de Recovery Services y una directiva para configurar la copia de seguridad del recurso compartido de archivos. Si aún no lo hecho, consulte los tutoriales de API REST para [crear un almacén](./backup-azure-arm-userestapi-createorupdatevault.md) y [crear una directiva](./backup-azure-arm-userestapi-createorupdatepolicy.md).

En este artículo, usaremos los siguientes recursos:

- **Almacén de Recovery Services**: *azurefilesvault*

- **Directiva:** *schedule1*

- **Grupo de recursos**: *azurefiles*

- **Cuenta de almacenamiento**: *testvault2*

- **Recurso compartido de archivos**: *testshare*

## <a name="configure-backup-for-an-unprotected-azure-file-share-using-rest-api"></a>Configuración de la copia de seguridad para un recurso compartido de archivos de Azure sin protección con API REST

### <a name="discover-storage-accounts-with-unprotected-azure-file-shares"></a>Detección de cuentas de almacenamiento con recursos compartidos de archivos de Azure sin protección

El almacén debe detectar todas las cuentas de almacenamiento de Azure de la suscripción que tengan recursos compartidos de archivos de los que se pueda realizar una copia de seguridad en el almacén de Recovery Services. Esta acción se desencadena con la [operación de actualización](/rest/api/backup/protectioncontainers/refresh). Se trata de una operación *POST* asincrónica que garantiza que el almacén obtiene la lista más reciente de todos los recursos compartidos de archivos sin protección de Azure de la suscripción actual y los "almacena en caché". Una vez que el recurso compartido de archivos se almacena en caché, Recovery Services puede acceder a este y protegerlo.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupname}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/refreshContainers?api-version=2016-12-01&$filter={$filter}
```

El URI de POST tiene los parámetros `{subscriptionId}`, `{vaultName}`, `{vaultresourceGroupName}` y `{fabricName}`. En nuestro ejemplo, el valor de los distintos parámetros será el siguiente:

- `{fabricName}` es *Azure*

- `{vaultName}` es *azurefilesvault*

- `{vaultresourceGroupName}` es *azurefiles*

- $filter=backupManagementType eq 'AzureStorage'

Como todos los parámetros necesarios se proporcionan en el URI, no hay necesidad de tener un cuerpo de solicitud independiente.

```http
POST https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/refreshContainers?api-version=2016-12-01&$filter=backupManagementType eq 'AzureStorage'
```

#### <a name="responses-to-the-refresh-operation"></a>Respuestas para la operación de actualización

La operación 'refresh' es una [operación asincrónica](../azure-resource-manager/management/async-operations.md). Significa que esta operación crea otra que tiene que ser seguida por separado.

Devuelve las dos respuestas: 202 (Aceptado) cuando se crea otra operación y 200 (Correcto) cuando se completa dicha operación.

##### <a name="example-responses-to-the-refresh-operation"></a>Respuestas de ejemplo para la operación de actualización

Una vez que se envía la solicitud *POST*, se devuelve una respuesta 202 (Accepted).

```http
HTTP/1.1 202 Accepted
'Pragma': 'no-cache'
'Expires': '-1'
'Location': ‘https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/ResourceGroups
/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/operationResults/
cca47745-12d2-42f9-b3a4-75335f18fdf6?api-version=2016-12-01’
'Retry-After': '60'
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': '6cc12ceb-90a2-430d-a1ec-9b6b6fdea92b'
'x-ms-client-request-id': ‘3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61’
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'X-Powered-By': 'ASP.NET'
'x-ms-ratelimit-remaining-subscription-reads': '11996'
'x-ms-correlation-request-id': '6cc12ceb-90a2-430d-a1ec-9b6b6fdea92b'
'x-ms-routing-request-id': CENTRALUSEUAP:20200203T091326Z:6cc12ceb-90a2-430d-a1ec-9b6b6fdea92b'
'Date': 'Mon, 03 Feb 2020 09:13:25 GMT'
```

Realice el seguimiento de la operación resultante con el encabezado "Location" (ubicación) y un simple comando *GET*.

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/operationResults/cca47745-12d2-42f9-b3a4-75335f18fdf6?api-version=2016-12-01
```

Una vez que se detectan todas las cuentas de almacenamiento de Azure, el comando GET devuelve una respuesta 200 (Sin contenido). El almacén ahora puede detectar cualquier cuenta de almacenamiento con recursos compartidos de archivos de los que se puede realizar una copia de seguridad dentro de la suscripción.

```http
HTTP/1.1 200 NoContent
Cache-Control  : no-cache
Pragma   : no-cache
X-Content-Type-Options  : nosniff
x-ms-request-id    : d9bdb266-8349-4dbd-9688-de52f07648b2
x-ms-client-request-id  : 3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61
Strict-Transport-Security  : max-age=31536000; includeSubDomains
X-Powered-By    : ASP.NET
x-ms-ratelimit-remaining-subscription-reads: 11933
x-ms-correlation-request-id   : d9bdb266-8349-4dbd-9688-de52f07648b2
x-ms-routing-request-id  : CENTRALUSEUAP:20200127T105304Z:d9bdb266-8349-4dbd-9688-de52f07648b2
Date   : Mon, 27 Jan 2020 10:53:04 GMT
```

### <a name="get-list-of-storage-accounts-with-file-shares-that-can-be-backed-up-with-recovery-services-vault"></a>Obtenga una lista de cuentas de almacenamiento con recursos compartidos de archivos de los que se puede realizar una copia de seguridad con el almacén de Recovery Services

Para confirmar que se ha realizado el "almacenamiento en caché", enumere todas las cuentas de almacenamiento de la suscripción con recursos compartidos de archivos de los que se pueda realizar una copia de seguridad con el almacén de Recovery Services. A continuación, busque la cuenta de almacenamiento deseada en la respuesta. Esto se hace mediante la operación [GET ProtectableContainers](/rest/api/backup/protectablecontainers/list).

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectableContainers?api-version=2016-12-01&$filter=backupManagementType eq 'AzureStorage'
```

El identificador URI de *GET* tiene todos los parámetros necesarios. No se necesita ningún cuerpo de solicitud adicional.

Ejemplo de cuerpo de respuesta:

```json
{

  "value": [

    {

      "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers
 /Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/

protectableContainers/StorageContainer;Storage;AzureFiles;testvault2",

      "name": "StorageContainer;Storage;AzureFiles;testvault2",

      "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectableContainers",

      "properties": {

        "friendlyName": "testvault2",

        "backupManagementType": "AzureStorage",

        "protectableContainerType": "StorageContainer",

        "healthStatus": "Healthy",

        "containerId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/
 AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2"

      }

    }

  ]

}
```

Dado que podemos encontrar la cuenta de almacenamiento *testvault2* en el cuerpo de la respuesta con el nombre descriptivo, la operación de actualización anterior se realizó correctamente. Ahora, el almacén de Recovery Services puede detectar correctamente las cuentas de almacenamiento con recursos compartidos de archivos sin protección en la misma suscripción.

### <a name="register-storage-account-with-recovery-services-vault"></a>Registro de la cuenta de almacenamiento con el almacén de Recovery Services

Este paso solo es necesario si no registró la cuenta de almacenamiento con el almacén anteriormente. Puede registrar el almacén mediante la [operación ProtectionContainers-Register](/rest/api/backup/protectioncontainers/register).

```http
PUT https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}?api-version=2016-12-01
```

Defina las variables de los URI como se indica a continuación:

- {resourceGroupName}: *azurefiles*
- {fabricName}: *Azure*
- {vaultName}: *azurefilesvault*
- {containerName}: es el atributo de nombre del cuerpo de la respuesta de la operación GET ProtectableContainers.
   En nuestro ejemplo, es *StorageContainer;Storage;AzureFiles;testvault2*

>[!NOTE]
> Tome siempre el atributo de nombre de la respuesta y rellénelo en esta solicitud. No cree ni codifique de forma rígida el formato del nombre de contenedor. Si lo crea o codifica de forma rígida, se producirá un error en la llamada API si el formato contenedor-nombre cambia en el futuro.

<br>

```http
PUT https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2?api-version=2016-12-01
```

La creación del cuerpo de la solicitud se lleva a cabo como se indica a continuación:

```json
{

 "properties": {


  "containerType": "StorageContainer",


  "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",


  "resourceGroup": "AzureFiles",


  "friendlyName": "testvault2",


  "backupManagementType": "AzureStorage"


 }
```

Para obtener una lista completa de las definiciones del cuerpo de la solicitud y otros detalles, consulte [ProtectionContainers-Register](/rest/api/backup/protectioncontainers/register#azurestoragecontainer).

Se trata de una operación asincrónica y devuelve dos respuestas: "202 Aceptado" cuando se acepta la operación y "200 Correcto" cuando se completa la operación.  Para realizar un seguimiento del estado de la operación, use el encabezado de ubicación para obtener el estado más reciente de la operación.

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/operationresults/1a3c8ee7-e0e5-43ed-b8b3-73cc992b6db9?api-version=2016-12-01
```

Ejemplo de cuerpo de respuesta cuando se completa la operación:

```json
{
    "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/
protectionContainers/StorageContainer;Storage;AzureFiles;testvault2",
    "name": "StorageContainer;Storage;AzureFiles;testvault2",
    "properties": {
        "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
        "protectedItemCount": 0,
        "friendlyName": "testvault2",
        "backupManagementType": "AzureStorage",
        "registrationStatus": "Registered",
        "healthStatus": "Healthy",
        "containerType": "StorageContainer",
        "protectableObjectType": "StorageContainer"
    }
}
```

Puede comprobar si el registro se realizó correctamente a partir del valor del parámetro *registrationstatus* en el cuerpo de la respuesta. En nuestro caso, muestra el estado registrado para *testvault2*, por lo que la operación de registro se realizó correctamente.

### <a name="inquire-all-unprotected-files-shares-under-a-storage-account"></a>Consultar todos los recursos compartidos de archivos sin protección en una cuenta de almacenamiento

Puede consultar los elementos que se pueden proteger de una cuenta de almacenamiento mediante la operación [Protection Containers-Inquire](/rest/api/backup/protectioncontainers/inquire). Es una operación asincrónica y se debe realizar un seguimiento de los resultados mediante el encabezado de ubicación.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/inquire?api-version=2016-12-01
```

Defina las variables de los URI anteriores como se indica a continuación:

- {vaultName}: *azurefilesvault*
- {fabricName}: *Azure*
- {containerName}: consulte el atributo de nombre del cuerpo de la respuesta de la operación GET ProtectableContainers. En nuestro ejemplo, es *StorageContainer;Storage;AzureFiles;testvault2*

```http
https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/inquire?api-version=2016-12-01
```

Una vez que la solicitud se realiza correctamente, devuelve el código de estado "Correcto".

```http
Cache-Control : no-cache
Pragma   : no-cache
X-Content-Type-Options: nosniff
x-ms-request-id  : 68727f1e-b8cf-4bf1-bf92-8e03a9d96c46
x-ms-client-request-id  : 3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61
Strict-Transport-Security: max-age=31536000; includeSubDomains
Server  : Microsoft-IIS/10.0
X-Powered-B : ASP.NET
x-ms-ratelimit-remaining-subscription-reads: 11932
x-ms-correlation-request-id  : 68727f1e-b8cf-4bf1-bf92-8e03a9d96c46
x-ms-routing-request-id   : CENTRALUSEUAP:20200127T105305Z:68727f1e-b8cf-4bf1-bf92-8e03a9d96c46
Date  : Mon, 27 Jan 2020 10:53:05 GMT
```

### <a name="select-the-file-share-you-want-to-back-up"></a>Seleccione el recurso compartido de archivos del que quiere hacer una copia de seguridad

Puede enumerar todos los elementos que se pueden proteger en la suscripción y buscar el recurso compartido de archivos que se va a incluir en la copia de seguridad mediante la operación [GET backupprotectableItems](/rest/api/backup/backupprotectableitems/list).

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupProtectableItems?api-version=2016-12-01&$filter={$filter}
```

Construya el URI de la forma siguiente:

- {vaultName}: *azurefilesvault*
- {$filter}: *backupManagementType eq 'AzureStorage'*

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupProtectableItems?$filter=backupManagementType eq 'AzureStorage'&api-version=2016-12-01
```

Respuesta de ejemplo:

```json
Status Code:200

{
    "value": [
        {
            "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;afaccount1/protectableItems/azurefileshare;azurefiles1",
            "name": "azurefileshare;azurefiles1",
            "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
            "properties": {
                "parentContainerFabricId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/afaccount1",
                "parentContainerFriendlyName": "afaccount1",
                "azureFileShareType": "XSMB",
                "backupManagementType": "AzureStorage",
                "workloadType": "AzureFileShare",
                "protectableItemType": "AzureFileShare",
                "friendlyName": "azurefiles1",
                "protectionState": "NotProtected"
            }
        },
        {
            "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;afsaccount/protectableItems/azurefileshare;afsresource",
            "name": "azurefileshare;afsresource",
            "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
            "properties": {
                "parentContainerFabricId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/afsaccount",
                "parentContainerFriendlyName": "afsaccount",
                "azureFileShareType": "XSMB",
                "backupManagementType": "AzureStorage",
                "workloadType": "AzureFileShare",
                "protectableItemType": "AzureFileShare",
                "friendlyName": "afsresource",
                "protectionState": "NotProtected"
            }
        },
        {
            "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;testvault2/protectableItems/azurefileshare;testshare",
            "name": "azurefileshare;testshare",
            "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
            "properties": {
                "parentContainerFabricId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
                "parentContainerFriendlyName": "testvault2",
                "azureFileShareType": "XSMB",
                "backupManagementType": "AzureStorage",
                "workloadType": "AzureFileShare",
                "protectableItemType": "AzureFileShare",
                "friendlyName": "testshare",
                "protectionState": "NotProtected"
            }
        }
    ]
}
```

La respuesta contiene la lista de todos los recursos compartidos de archivos sin protección e incluye toda la información que requiere Azure Recovery Services para configurar la copia de seguridad.

### <a name="enable-backup-for-the-file-share"></a>Habilitación de la copia de seguridad para el recurso compartido de archivos

Una vez que el recurso compartido de archivos correspondiente esté "identificado" con el nombre descriptivo, seleccione la directiva para la protección. Para más información acerca de las directivas existentes en el almacén, consulte el artículo sobre [la API de enumeración de directivas](/rest/api/backup/backuppolicies/list). A continuación, seleccione la [directiva pertinente](/rest/api/backup/protectionpolicies/get) haciendo referencia al nombre de la directiva. Para crear las directivas, consulte el [ tutorial sobre la creación de directivas](./backup-azure-arm-userestapi-createorupdatepolicy.md).

La habilitación de la protección es una operación asincrónica *PUT* que crea un "elemento protegido".

```http
PUT https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}?api-version=2019-05-13
```

Establezca las variables **containername** y **protecteditemname** mediante el atributo ID en el cuerpo de respuesta de la operación GET backupprotectableitems.

En nuestro ejemplo, el identificador del recurso compartido de archivos que queremos proteger es el siguiente:

```output
"/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;testvault2/protectableItems/azurefileshare;testshare
```

- {containername}: *storagecontainer;storage;azurefiles;testvault2*
- {protectedItemName}: *azurefileshare;azurefiles*

También puede hacer referencia al atributo **name** del contenedor de protección y las respuestas de los elementos que se pueden proteger.

>[!NOTE]
>Tome siempre el atributo de nombre de la respuesta y rellénelo en esta solicitud. No cree ni codifique de forma rígida el formato del nombre de contenedor, ni el formato del nombre del elemento protegido. Si lo crea o codifica de forma rígida, se producirá un error en la llamada API si el formato contenedor-nombre o el formato del nombre del elemento protegido cambia en el futuro.

<br>

```http
PUT https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/protectedItems/azurefileshare;testshare?api-version=2016-12-01
```

Crear un cuerpo de la solicitud:

El cuerpo de solicitud siguiente define las propiedades necesarias para crear un elemento protegido.

```json
{
  "properties": {
    "protectedItemType": "AzureFileShareProtectedItem",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
    "policyId": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupPolicies/schedule1"
  }
}
```

**sourceResourceId** es **parentcontainerFabricID** en respuesta a GET backupprotectableItems.

Respuesta de ejemplo

La creación de un elemento protegido es una operación asincrónica que crea otra operación de la que es necesario realizar un seguimiento. Devuelve las dos respuestas: 202 (Aceptado) cuando se crea otra operación y 200 (Correcto) cuando se completa dicha operación.

Una vez enviada la solicitud *PUT* para la creación o actualización de elementos protegidos, la respuesta inicial es 202 (Aceptado) con un encabezado de ubicación.

```http
HTTP/1.1 202 Accepted
Cache-Control  : no-cache
Pragma  : no-cache
Location : https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/protectedItems/azurefileshare;testshare/operationResults/c3a52d1d-0853-4211-8141-477c65740264?api-version=2016-12-01
Retry-Afte  : 60
Azure-AsyncOperation  : https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/protectedItems/azurefileshare;testshare/operationResults/c3a52d1d-0853-4211-8141-477c65740264?api-version=2016-12-01
X-Content-Type-Options : nosniff
x-ms-request-id : b55527fa-f473-4f09-b169-9cc3a7a39065
x-ms-client-request-id: 3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61
Strict-Transport-Security : max-age=31536000; includeSubDomains
X-Powered-By  : ASP.NET
x-ms-ratelimit-remaining-subscription-writes: 1198
x-ms-correlation-request-id : b55527fa-f473-4f09-b169-9cc3a7a39065
x-ms-routing-request-id  : CENTRALUSEUAP:20200127T105412Z:b55527fa-f473-4f09-b169-9cc3a7a39065
Date : Mon, 27 Jan 2020 10:54:12 GMT
```

A continuación, realice un seguimiento de la operación resultante con el encabezado de ubicación o el encabezado Azure-AsyncOperation y un comando *GET*.

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupOperations/c3a52d1d-0853-4211-8141-477c65740264?api-version=2016-12-01
```

Una vez completada la operación, devuelve 200 (OK) con el contenido del elemento protegido en el cuerpo de respuesta.

Ejemplo de cuerpo de respuesta:

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

Así se confirma que la protección está habilitada para el recurso compartido de archivos y la primera copia de seguridad se desencadenará según la programación de la directiva.

## <a name="trigger-an-on-demand-backup-for-file-share"></a>Desencadenamiento de una copia de seguridad a petición para recursos compartidos de archivos

Una vez que un recurso compartido de archivos de Azure está configurado para la copia de seguridad, las copias de seguridad se realizan según la programación de la directiva. Puede esperar a la primera copia de seguridad programada o desencadenar una copia de seguridad a petición en cualquier momento.

Desencadenar una copia de seguridad a petición es una operación POST.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/backup?api-version=2016-12-01
```

{containerName} y {protectedItemName} se han creado anteriormente al habilitar la copia de seguridad. En nuestro ejemplo, esto se traduce en:

```http
POST https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/backup?api-version=2017-07-01
```

### <a name="create-request-body"></a>Creación del cuerpo de la solicitud

Para desencadenar una copia de seguridad a petición, los siguientes son los componentes del cuerpo de la solicitud.

| Nombre       | Tipo                       | Descripción                       |
| ---------- | -------------------------- | --------------------------------- |
| Propiedades | AzurefilesharebackupReques | Propiedades de BackupRequestResource |

Para obtener una lista completa de las definiciones del cuerpo de la solicitud y otros detalles, consulte el [documento de la API REST sobre desencadenar copias de seguridad de los elementos protegidos](/rest/api/backup/backups/trigger#request-body).

Ejemplo de cuerpo de la solicitud

```json
{

  "properties":{

   "objectType":"AzureFileShareBackupRequest",
    "recoveryPointExpiryTimeInUTC":"2020-03-07T18:29:59.000Z"
}

}
```

### <a name="responses-to-the-on-demand-backup-operation"></a>Respuestas a la operación de copia de seguridad a petición

Desencadenar una copia de seguridad a petición es una [operación asincrónica](../azure-resource-manager/management/async-operations.md). Significa que esta operación crea otra que tiene que ser seguida por separado.

Devuelve las dos respuestas: 202 (Aceptado) cuando se crea otra operación y 200 (Correcto) cuando se completa dicha operación.

### <a name="example-responses-to-the-on-demand-backup-operation"></a>Respuestas de ejemplo a la operación de copia de seguridad a petición

Una vez enviada la solicitud *POST* para una copia de seguridad a petición, la respuesta inicial es 202 (Accepted) con un encabezado de ubicación o Azure-async-header.

```http
'Cache-Control': 'no-cache'
'Pragma': 'no-cache'
'Expires': '-1'
'Location': https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/operationResults/dc62d524-427a-4093-968d-e951c0a0726e?api-version=2017-07-01
'Retry-After': '60'
'Azure-AsyncOperation': https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/operationsStatus/dc62d524-427a-4093-968d-e951c0a0726e?api-version=2017-07-01
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': '2e03b8d4-66b1-48cf-8094-aa8bff57e8fb'
'x-ms-client-request-id': 'a644712a-4895-11ea-ba57-0a580af42708, a644712a-4895-11ea-ba57-0a580af42708'
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'X-Powered-By': 'ASP.NET'
'x-ms-ratelimit-remaining-subscription-writes': '1199'
'x-ms-correlation-request-id': '2e03b8d4-66b1-48cf-8094-aa8bff57e8fb'
'x-ms-routing-request-id': 'WESTEUROPE:20200206T040339Z:2e03b8d4-66b1-48cf-8094-aa8bff57e8fb'
'Date': 'Thu, 06 Feb 2020 04:03:38 GMT'
'Content-Length': '0'
```

A continuación, realice un seguimiento de la operación resultante con el encabezado de ubicación o el encabezado Azure-AsyncOperation y un comando *GET*.

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupOperations/dc62d524-427a-4093-968d-e951c0a0726e?api-version=2016-12-01
```

Una vez completada la operación, devuelve 200 (OK) con el identificador del trabajo de copia de seguridad resultante en el cuerpo de respuesta.

#### <a name="sample-response-body"></a>Muestra de cuerpo de respuesta

```json
{
    "id": "dc62d524-427a-4093-968d-e951c0a0726e",
    "name": "dc62d524-427a-4093-968d-e951c0a0726e",
    "status": "Succeeded",
    "startTime": "2020-02-06T11:06:02.1327954Z",
    "endTime": "2020-02-06T11:06:02.1327954Z",
    "properties": {
        "objectType": "OperationStatusJobExtendedInfo",
        "jobId": "39282261-cb52-43f5-9dd0-ffaf66beeaef"
    }
}
```

Puesto que el trabajo de copia de seguridad es una operación de larga duración, se debe seguir como se explica en el [documento sobre la supervisión de trabajos con API REST](./backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre cómo [restaurar recursos compartidos de archivos de Azure mediante la API REST](restore-azure-file-share-rest-api.md).
