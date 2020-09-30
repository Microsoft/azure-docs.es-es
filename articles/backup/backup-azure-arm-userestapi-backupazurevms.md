---
title: copia de seguridad de máquinas virtuales de Azure mediante API REST
description: En este artículo se aprende a configurar, iniciar y administrar las operaciones de copia de seguridad de Azure Backup mediante la API de REST.
ms.topic: conceptual
ms.date: 08/03/2018
ms.assetid: b80b3a41-87bf-49ca-8ef2-68e43c04c1a3
ms.openlocfilehash: aa072cb48e12ac89af3be28a9633a82b50122275
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89006302"
---
# <a name="back-up-an-azure-vm-using-azure-backup-via-rest-api"></a>Copia de seguridad de una máquina virtual de Azure mediante Azure Backup a través de la API REST

En este artículo se describe cómo administrar las copias de seguridad para una máquina virtual de Azure mediante Azure Backup a través de la API REST. Configurará la protección por primera vez para una máquina virtual de Azure que no se haya protegido anteriormente, desencadenará una copia de seguridad a petición para una máquina virtual de Azure protegida y modificará las propiedades de copia de seguridad de una copia de seguridad de máquina virtual a través de la API REST, como se explica aquí.

Consulte los tutoriales de API REST sobre [create vault](backup-azure-arm-userestapi-createorupdatevault.md) y [create polity](backup-azure-arm-userestapi-createorupdatepolicy.md) para crear nuevos almacenes y las directivas.

Supongamos que desea proteger una máquina virtual "testVM" en un grupo de recursos "testRG" para un almacén de Recovery Services "testVault", presente en el grupo de recursos "testVaultRG", con la directiva predeterminada (denominada "DefaultPolicy").

## <a name="configure-backup-for-an-unprotected-azure-vm-using-rest-api"></a>Configuración de la copia de seguridad para una máquina virtual de Azure sin protección con API REST

### <a name="discover-unprotected-azure-vms"></a>Detección de las máquinas virtuales sin protección de Azure

En primer lugar, el almacén debe ser capaz de identificar la máquina virtual de Azure. Esta acción se desencadena con la [operación de actualización](/rest/api/backup/protectioncontainers/refresh). Se trata de una operación *POST* asincrónica que se asegura de que el almacén obtiene la lista más reciente de todas las máquinas virtuales sin protección en la suscripción actual y las "almacena en caché". Una vez que la máquina virtual se "almacene en caché", Recovery Services podrá acceder a ella y protegerla.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupname}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/refreshContainers?api-version=2016-12-01
```

El URI de POST tiene parámetros `{subscriptionId}`, `{vaultName}`, `{vaultresourceGroupName}` y `{fabricName}`. `{fabricName}` es "Azure". Según nuestro ejemplo, `{vaultName}` es "testVault" y `{vaultresourceGroupName}` es "testVaultRG". Como todos los parámetros necesarios se proporcionan en el URI, no hay necesidad de tener un cuerpo de solicitud independiente.

```http
POST https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/refreshContainers?api-version=2016-12-01
```

#### <a name="responses-to-refresh-operation"></a>Respuestas para la operación de actualización

La operación 'refresh' es una [operación asincrónica](../azure-resource-manager/management/async-operations.md). Significa que esta operación crea otra que tiene que ser seguida por separado.

Devuelve las dos respuestas: 202 (Accepted) (aceptado) cuando se crea otra operación y, a continuación, 200 (OK) cuando se completa dicha operación.

|Nombre  |Tipo  |Descripción  |
|---------|---------|---------|
|204 No Content     |         |  Correcto y no se devolvió contenido      |
|202 - Aceptado     |         |     Accepted    |

##### <a name="example-responses-to-refresh-operation"></a>Respuestas de ejemplo para la operación de actualización

Una vez que se envía la solicitud *POST*, se devuelve una respuesta 202 (Accepted).

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
X-Content-Type-Options: nosniff
x-ms-request-id: 43cf550d-e463-421c-8922-37e4766db27d
x-ms-client-request-id: 4910609f-bb9b-4c23-8527-eb6fa2d3253f; 4910609f-bb9b-4c23-8527-eb6fa2d3253f
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: 43cf550d-e463-421c-8922-37e4766db27d
x-ms-routing-request-id: SOUTHINDIA:20180521T105701Z:43cf550d-e463-421c-8922-37e4766db27d
Cache-Control: no-cache
Date: Mon, 21 May 2018 10:57:00 GMT
Location: https://management.azure.com/subscriptions//00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/operationResults/aad204aa-a5cf-4be2-a7db-a224819e5890?api-version=2019-05-13
X-Powered-By: ASP.NET
```

Realice el seguimiento de la operación resultante con el encabezado "Location" (ubicación) y un simple comando *GET*.

```http
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/operationResults/aad204aa-a5cf-4be2-a7db-a224819e5890?api-version=2019-05-13
```

Una vez que se detectan todas las máquinas virtuales de Azure, el comando GET devuelve una respuesta 204 (No Content) (sin contenido). El almacén ahora es capaz de detectar todas las máquinas virtuales dentro de la suscripción.

```http
HTTP/1.1 204 NoContent
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: cf6cd73b-9189-4942-a61d-878fcf76b1c1
x-ms-client-request-id: 25bb6345-f9fc-4406-be1a-dc6db0eefafe; 25bb6345-f9fc-4406-be1a-dc6db0eefafe
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14997
x-ms-correlation-request-id: cf6cd73b-9189-4942-a61d-878fcf76b1c1
x-ms-routing-request-id: SOUTHINDIA:20180521T105825Z:cf6cd73b-9189-4942-a61d-878fcf76b1c1
Cache-Control: no-cache
Date: Mon, 21 May 2018 10:58:25 GMT
X-Powered-By: ASP.NET
```

### <a name="selecting-the-relevant-azure-vm"></a>Selección de la máquina virtual de Azure pertinente

 Puede confirmar que el "almacenamiento en caché" se realiza [enumerando todos los elementos que se pueden proteger](/rest/api/backup/backupprotectableitems/list) en la suscripción y luego busque la máquina virtual deseada en la respuesta. [La respuesta de esta operación](#example-responses-to-get-operation) también le proporciona información sobre el modo en que Recovery Services identifica una máquina virtual.  Una vez que esté familiarizado con el patrón, podrá omitir este paso y proceder directamente a [habilitar la protección](#enabling-protection-for-the-azure-vm).

Esta es una operación *GET*.

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupProtectableItems?api-version=2016-12-01&$filter=backupManagementType eq 'AzureIaasVM'
```

El identificador URI de *GET* tiene todos los parámetros necesarios. No se necesita ningún cuerpo de solicitud adicional.

#### <a name="responses-to-get-operation"></a>Respuestas para la operación GET

|Nombre  |Tipo  |Descripción  |
|---------|---------|---------|
|200 OK     | [WorkloadProtectableItemResourceList](/rest/api/backup/backupprotectableitems/list#workloadprotectableitemresourcelist)        |       Aceptar |

#### <a name="example-responses-to-get-operation"></a>Respuestas de ejemplo para la operación GET

Una vez que se emite la solicitud *GET*, se devuelve una respuesta 200 (OK).

```http
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: 7c2cf56a-e6be-4345-96df-c27ed849fe36
x-ms-client-request-id: 40c8601a-c217-4c68-87da-01db8dac93dd; 40c8601a-c217-4c68-87da-01db8dac93dd
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14979
x-ms-correlation-request-id: 7c2cf56a-e6be-4345-96df-c27ed849fe36
x-ms-routing-request-id: SOUTHINDIA:20180521T071408Z:7c2cf56a-e6be-4345-96df-c27ed849fe36
Cache-Control: no-cache
Date: Mon, 21 May 2018 07:14:08 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "value": [
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainerv2;testRG;testVM/protectableItems/vm;iaasvmcontainerv2;testRG;testVM",
      "name": "iaasvmcontainerv2;testRG;testVM",
      "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
      "properties": {
        "virtualMachineId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
        "virtualMachineVersion": "Compute",
        "resourceGroup": "testRG",
        "backupManagementType": "AzureIaasVM",
        "protectableItemType": "Microsoft.Compute/virtualMachines",
        "friendlyName": "testVM",
        "protectionState": "NotProtected"
      }
    },……………..

```

> [!TIP]
> El número de valores de una respuesta *GET* está limitada a 200 para una página. Utilice el campo "nextLink" para obtener la dirección URL para el siguiente conjunto de respuestas.

La respuesta contiene la lista de todas las máquinas virtuales no protegidas de Azure y cada `{value}` contiene toda la información que requiere Azure Recovery Services para configurar la copia de seguridad. Para configurar la copia de seguridad, tenga en cuenta el campo `{name}` y el campo `{virtualMachineId}` de la sección `{properties}`. Construya dos variables a partir de estos valores de campo como se menciona más abajo.

- containerName = "iaasvmcontainer;"+`{name}`
- protectedItemName = "vm;"+ `{name}`
- `{virtualMachineId}` se usará posteriormente en [el cuerpo de la solicitud](#example-request-body)

En el ejemplo, los valores anteriores se traducen como:

- containerName = "iaasvmcontainer;iaasvmcontainerv2;testRG;testVM"
- protectedItemName = "vm;iaasvmcontainerv2;testRG;testVM"

### <a name="enabling-protection-for-the-azure-vm"></a>Habilitación de la protección de la máquina virtual de Azure

Una vez que la máquina virtual correspondiente está "almacenada en caché" e "identificada", seleccione la directiva para la protección. Para más información acerca de las directivas existentes en el almacén, consulte el artículo sobre [la enumeración de directivas en la API](/rest/api/backup/backuppolicies/list). A continuación, seleccione la [directiva pertinente](/rest/api/backup/protectionpolicies/get) haciendo referencia al nombre de la directiva. Para crear las directivas, consulte el [ tutorial sobre la creación de directivas](backup-azure-arm-userestapi-createorupdatepolicy.md). "DefaultPolicy" se selecciona en el ejemplo siguiente.

La habilitación de la protección es una operación asincrónica *PUT* que crea un elemento protegido.

```http
https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}?api-version=2019-05-13
```

`{containerName}` y `{protectedItemName}` son como se han construido anteriormente. `{fabricName}` es "Azure". En nuestro ejemplo, esto se traduce en:

```http
PUT https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;iaasvmcontainerv2;testRG;testVM?api-version=2019-05-13
```

#### <a name="create-the-request-body"></a>Creación del cuerpo de la solicitud

Para crear un elemento protegido, los siguientes son los componentes del cuerpo de la solicitud.

|Nombre  |Tipo  |Descripción  |
|---------|---------|---------|
|properties     | AzureIaaSVMProtectedItem        |Propiedades del recurso ProtectedItem         |

Para obtener una lista completa de las definiciones del cuerpo de la solicitud y otros detalles, consulte el [documento de la API REST sobre la creación de un elemento protegido](/rest/api/backup/protecteditems/createorupdate#request-body).

##### <a name="example-request-body"></a>Cuerpo de solicitud de ejemplo

El cuerpo de solicitud siguiente define las propiedades necesarias para crear un elemento protegido.

```json
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupPolicies/DefaultPolicy"
  }
}
```

El `{sourceResourceId}` es el identificador `{virtualMachineId}` mencionado anteriormente desde la [respuesta de los elementos de lista que se pueden proteger](#example-responses-to-get-operation).

#### <a name="responses-to-create-protected-item-operation"></a>Respuestas para la operación de creación de elementos protegidos

La creación de un elemento protegido es una [operación asincrónica](../azure-resource-manager/management/async-operations.md). Significa que esta operación crea otra que tiene que ser seguida por separado.

Devuelve las dos respuestas: 202 (Accepted) (aceptado) cuando se crea otra operación y, a continuación, 200 (OK) cuando se completa dicha operación.

|Nombre  |Tipo  |Descripción  |
|---------|---------|---------|
|200 OK     |    [ProtectedItemResource](/rest/api/backup/protecteditemoperationresults/get#protecteditemresource)     |  Aceptar       |
|202 - Aceptado     |         |     Accepted    |

##### <a name="example-responses-to-create-protected-item-operation"></a>Respuestas de ejemplo para la operación de creación de elementos protegidos

Una vez enviada la solicitud *PUT* para la creación o actualización de elementos protegidos, la respuesta inicial es 202 (Accepted) (aceptado) con un encabezado de ubicación o Azure-async-header.

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
Azure-AsyncOperation: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
X-Content-Type-Options: nosniff
x-ms-request-id: db785be0-bb20-4598-bc9f-70c9428b170b
x-ms-client-request-id: e1f94eef-9b2d-45c4-85b8-151e12b07d03; e1f94eef-9b2d-45c4-85b8-151e12b07d03
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: db785be0-bb20-4598-bc9f-70c9428b170b
x-ms-routing-request-id: SOUTHINDIA:20180521T073907Z:db785be0-bb20-4598-bc9f-70c9428b170b
Cache-Control: no-cache
Date: Mon, 21 May 2018 07:39:06 GMT
Location: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationResults/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
X-Powered-By: ASP.NET
```

A continuación, realice un seguimiento de la operación resultante con el encabezado de ubicación o el encabezado Azure-AsyncOperation y un simple comando *GET*.

```http
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
```

Una vez completada la operación, devuelve 200 (OK) con el contenido del elemento protegido en el cuerpo de respuesta.

```json
{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM",
  "name": "VM;testRG;testVM",
  "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems",
  "properties": {
    "friendlyName": "testVM",
    "virtualMachineId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "protectionStatus": "Healthy",
    "protectionState": "IRPending",
    "healthStatus": "Passed",
    "lastBackupStatus": "",
    "lastBackupTime": "2001-01-01T00:00:00Z",
    "protectedItemDataId": "17592691116891",
    "extendedInfo": {
      "recoveryPointCount": 0,
      "policyInconsistent": false
    },
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "backupManagementType": "AzureIaasVM",
    "workloadType": "VM",
    "containerName": "iaasvmcontainerv2;testRG;testVM",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupPolicies/DefaultPolicy",
    "policyName": "DefaultPolicy"
  }
}
```

Así se confirma que la protección está habilitada para la máquina virtual y la primera copia de seguridad se desencadenará según la programación de la directiva.

## <a name="trigger-an-on-demand-backup-for-a-protected-azure-vm"></a>Desencadenar una copia de seguridad a petición para una máquina virtual de Azure protegida

Una vez que una máquina virtual de Azure está configurada para la copia de seguridad, las copias de seguridad se realizan según la programación de la directiva. Puede esperar a la primera copia de seguridad programada o desencadenar una copia de seguridad a petición en cualquier momento. La retención de las copias de seguridad a petición es independiente de la retención de copias de seguridad que marca la directiva y se puede especificar para una determinada fecha y hora. Si no se especifica, se supone que son 30 días a partir del momento en que se desencadene la copia de seguridad a petición.

Desencadenar una copia de seguridad a petición es una operación *POST*.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/backup?api-version=2016-12-01
```

`{containerName}` y `{protectedItemName}` son como se han construido [anteriormente](#responses-to-get-operation). `{fabricName}` es "Azure". En nuestro ejemplo, esto se traduce en:

```http
POST https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;iaasvmcontainerv2;testRG;testVM/backup?api-version=2016-12-01
```

### <a name="create-the-request-body-for-on-demand-backup"></a>Creación del cuerpo de la solicitud para la copia de seguridad a petición

Para desencadenar una copia de seguridad a petición, los siguientes son los componentes del cuerpo de la solicitud.

|Nombre  |Tipo  |Descripción  |
|---------|---------|---------|
|properties     | [IaaSVMBackupRequest](/rest/api/backup/backups/trigger#iaasvmbackuprequest)        |Propiedades de BackupRequestResource         |

Para obtener una lista completa de las definiciones del cuerpo de la solicitud y otros detalles, consulte el [documento de la API REST sobre desencadenar copias de seguridad de los elementos protegidos](/rest/api/backup/backups/trigger#request-body).

#### <a name="example-request-body-for-on-demand-backup"></a>Cuerpo de la solicitud de ejemplo para la copia de seguridad a petición

El cuerpo de solicitud siguiente define las propiedades necesarias para desencadenar la copia de seguridad de un elemento protegido. Si no se especifica el período de retención, se conservarán durante 30 días desde el momento en que se desencadene el trabajo de copia de seguridad.

```json
{
   "properties": {
    "objectType": "IaasVMBackupRequest",
    "recoveryPointExpiryTimeInUTC": "2018-12-01T02:16:20.3156909Z"
  }
}
```

### <a name="responses-for-on-demand-backup"></a>Respuestas para la copia de seguridad a petición

Desencadenar una copia de seguridad a petición es una [operación asincrónica](../azure-resource-manager/management/async-operations.md). Significa que esta operación crea otra que tiene que ser seguida por separado.

Devuelve las dos respuestas: 202 (Accepted) (aceptado) cuando se crea otra operación y, a continuación, 200 (OK) cuando se completa dicha operación.

|Nombre  |Tipo  |Descripción  |
|---------|---------|---------|
|202 - Aceptado     |         |     Accepted    |

#### <a name="example-responses-for-on-demand-backup"></a>Respuestas de ejemplo para la copia de seguridad a petición

Una vez enviada la solicitud *POST* para una copia de seguridad a petición, la respuesta inicial es 202 (Accepted) con un encabezado de ubicación o Azure-async-header.

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
Azure-AsyncOperation: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testVaultRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/b8daecaa-f8f5-44ed-9f18-491a9e9ba01f?api-version=2019-05-13
X-Content-Type-Options: nosniff
x-ms-request-id: 7885ca75-c7c6-43fb-a38c-c0cc437d8810
x-ms-client-request-id: 7df8e874-1d66-4f81-8e91-da2fe054811d; 7df8e874-1d66-4f81-8e91-da2fe054811d
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: 7885ca75-c7c6-43fb-a38c-c0cc437d8810
x-ms-routing-request-id: SOUTHINDIA:20180521T083541Z:7885ca75-c7c6-43fb-a38c-c0cc437d8810
Cache-Control: no-cache
Date: Mon, 21 May 2018 08:35:41 GMT
Location: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testVaultRG;testVM/protectedItems/vm;testRG;testVM/operationResults/b8daecaa-f8f5-44ed-9f18-491a9e9ba01f?api-version=2019-05-13
X-Powered-By: ASP.NET
```

A continuación, realice un seguimiento de la operación resultante con el encabezado de ubicación o el encabezado Azure-AsyncOperation y un simple comando *GET*.

```http
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
```

Una vez completada la operación, devuelve 200 (OK) con el identificador del trabajo de copia de seguridad resultante en el cuerpo de respuesta.

```json
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: a8b13524-2c95-445f-b107-920806f385c1
x-ms-client-request-id: 5a63209d-3708-4e69-a75f-9499f4c8977c; 5a63209d-3708-4e69-a75f-9499f4c8977c
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14995
x-ms-correlation-request-id: a8b13524-2c95-445f-b107-920806f385c1
x-ms-routing-request-id: SOUTHINDIA:20180521T083723Z:a8b13524-2c95-445f-b107-920806f385c1
Cache-Control: no-cache
Date: Mon, 21 May 2018 08:37:22 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "00000000-0000-0000-0000-000000000000",
  "status": "Succeeded",
  "startTime": "2018-05-21T08:35:40.9488967Z",
  "endTime": "2018-05-21T08:35:40.9488967Z",
  "properties": {
    "objectType": "OperationStatusJobExtendedInfo",
    "jobId": "7ddead57-bcb9-4269-ac31-6a1b57588700"
  }
}
```

Puesto que el trabajo de copia de seguridad es una operación de larga duración, se debe seguir como se explica en el [documento sobre la supervisión de trabajos con API REST](backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

## <a name="modify-the-backup-configuration-for-a-protected-azure-vm"></a>Modificación de la configuración de copia de seguridad de una máquina virtual de Azure protegida

### <a name="changing-the-policy-of-protection"></a>Cambiar la directiva de protección

Para cambiar la directiva con la que la máquina virtual se protege, puede usar el mismo formato que para [habilitar la protección](#enabling-protection-for-the-azure-vm). Basta con que proporcione el identificador de la nueva directiva en [el cuerpo de solicitud](#example-request-body) y envíe la solicitud. Por ejemplo: para cambiar la directiva testVM de "DefaultPolicy" a "ProdPolicy", especifique el identificador "ProdPolicy" en el cuerpo de la solicitud.

```http
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupPolicies/ProdPolicy"
  }
}
```

La respuesta seguirá el mismo formato que se mencionó [para habilitar la protección](#responses-to-create-protected-item-operation).

### <a name="stop-protection-but-retain-existing-data"></a>Detener la protección, pero conservar los datos existentes

Para quitar la protección en una máquina virtual protegida, pero conservar los datos que ya se hayan incluido en una copia de seguridad, quite la directiva del cuerpo de solicitud y envíe la solicitud. Una vez que se quita la asociación con la directiva, ya no se desencadenan las copias de seguridad ni se crea ningún punto de recuperación nuevo.

```http
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": ""
  }
}
```

La respuesta seguirá el mismo formato que se mencionó [para desencadenar una copia de seguridad a petición](#example-responses-for-on-demand-backup). El trabajo resultante se debe seguir como se explica en el [documento para supervisar los trabajos con API REST](backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

### <a name="stop-protection-and-delete-data"></a>Detener la protección y eliminar los datos

Para quitar la protección de una máquina virtual protegida y eliminar también los datos de la copia de seguridad, realice una operación de eliminación como se detalla [aquí](/rest/api/backup/protecteditems/delete).

Detener la protección y eliminar los datos es una operación *DELETE*.

```http
DELETE https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}?api-version=2019-05-13
```

`{containerName}` y `{protectedItemName}` son como se han construido [anteriormente](#responses-to-get-operation). `{fabricName}` es "Azure". En nuestro ejemplo, esto se traduce en:

```http
DELETE https://management.azure.com//Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;iaasvmcontainerv2;testRG;testVM?api-version=2019-05-13
```

#### <a name="responses-for-delete-protection"></a>Respuestas para la protección de eliminación

La protección *DELETE* es una [operación asincrónica](../azure-resource-manager/management/async-operations.md). Significa que esta operación crea otra que tiene que ser seguida por separado.

Devuelve las dos respuestas: 202 (Accepted) (aceptado) cuando se crea otra operación y, a continuación, 204 (NoContent) (sin contenido) cuando se completa dicha operación.

|Nombre  |Tipo  |Descripción  |
|---------|---------|---------|
|204 NoContent     |         |  NoContent       |
|202 - Aceptado     |         |     Accepted    |

> [!IMPORTANT]
> Con el fin de protegerse frente a escenarios de eliminación accidental, hay una [característica de eliminación temporal disponible](use-restapi-update-vault-properties.md#soft-delete-state) para el almacén de Recovery Services. Si el estado de eliminación temporal del almacén está establecido en habilitado, la operación de eliminación no eliminará los datos de inmediato. Se conservarán durante 14 días y, a continuación, se purgarán de forma permanente. No se le cobra por el almacenamiento durante este período de 14 días. Para deshacer la operación de eliminación, consulte la sección de [deshacer y eliminar](#undo-the-deletion).

### <a name="undo-the-deletion"></a>Reversión de la eliminación

Deshacer la eliminación accidental es similar a crear el elemento de copia de seguridad. Después de deshacer la eliminación, el elemento se conserva, pero no se desencadenan copias de seguridad futuras.

Deshacer la eliminación es una operación *PUT* muy similar a [cambiar la directiva](#changing-the-policy-of-protection) o [habilitar la protección](#enabling-protection-for-the-azure-vm). Simplemente proporcione la intención de deshacer la eliminación con la variable *isRehydrate* en el [ cuerpo de la solicitud](#example-request-body) y envíe la solicitud. Por ejemplo: Para deshacer la eliminación de testVM, debe usarse el cuerpo de la solicitud siguiente.

```http
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "protectionState": "ProtectionStopped",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "isRehydrate": true
  }
}
```

La respuesta seguirá el mismo formato que se mencionó [para desencadenar una copia de seguridad a petición](#example-responses-for-on-demand-backup). El trabajo resultante se debe seguir como se explica en el [documento para supervisar los trabajos con API REST](backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

## <a name="next-steps"></a>Pasos siguientes

[Restauración de datos desde una copia de seguridad de máquina virtual de Azure](backup-azure-arm-userestapi-restoreazurevms.md).

Para más información sobre las API REST de Azure Backup, consulte los siguientes documentos:

- [API REST del proveedor de Azure Recovery Services](/rest/api/recoveryservices/)
- [Get started with Azure REST API](/rest/api/azure/) (Introducción a la API REST de Azure)
