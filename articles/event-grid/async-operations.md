---
title: Estado de las operaciones asincrónicas de Event Grid
description: Describe cómo realizar un seguimiento de las operaciones asincrónicas de Event Grid en Azure. Muestra los valores que se usan para obtener el estado de una operación de larga ejecución.
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: baae7b097a0b696d405c0e7ea3d3bdeb326f23b1
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011691"
---
# <a name="track-event-grid-asynchronous-azure-operations"></a>Seguimiento de las operaciones asincrónicas de Event Grid en Azure
Algunas operaciones de REST de Azure se ejecutan asincrónicamente porque la operación no se puede completar rápidamente. En este artículo se describe cómo realizar un seguimiento del estado de las operaciones asincrónicas a través de los valores devueltos en la respuesta.  

## <a name="status-codes-for-asynchronous-operations"></a>Códigos de estado para las operaciones asincrónicas
Una operación asincrónica devuelve inicialmente un código de estado HTTP de alguno de estos tipos:

* 201 (Created)
* 202 (Accepted) 

Cuando la operación se completa correctamente, devuelve:

* 200 (OK)
* 204 (No Content) 

Consulte la [documentación de la API REST](/rest/api/) para ver respuestas para la operación que está ejecutando.

## <a name="monitor-status-of-operation"></a>Supervisión del estado de la operación
Las operaciones asincrónicas de REST devuelven valores de encabezado, que se utilizan para determinar el estado de la operación. Hay potencialmente tres valores de encabezado para examinar:

* `Azure-AsyncOperation`: dirección URL para comprobar el estado actual de la operación. Si la operación devuelve este valor, utilícelo siempre (en lugar de Location) para realizar un seguimiento del estado de la operación.
* `Location`: dirección URL para determinar cuándo se ha completado una operación. Use este valor solo cuando no se devuelva Azure-AsyncOperation.
* `Retry-After`: el número de segundos que deben transcurrir antes de comprobar el estado de la operación asincrónica.

Sin embargo, no todas las operaciones asincrónicas devuelven todos estos valores. Por ejemplo, debe evaluar el valor del encabezado Azure-AsyncOperation para una operación y el valor del encabezado Location para otra operación. 

Puede recuperar los valores de encabezado como recuperaría cualquier valor de encabezado de una solicitud. Por ejemplo, en C#, recupere el valor del encabezado de un objeto `HttpWebResponse` denominado `response` con el código siguiente:

```cs
response.Headers.GetValues("Azure-AsyncOperation").GetValue(0)
```

## <a name="azure-asyncoperation-request-and-response"></a>Solicitud y respuesta de Azure-AsyncOperation

Para obtener el estado de la operación asincrónica, envíe una solicitud GET a la dirección URL en el valor del encabezado Azure-AsyncOperation.

El cuerpo de la respuesta de esta operación contiene información sobre la operación. El ejemplo siguiente muestra los posibles valores devueltos por la operación:

```json
{
    "id": "{resource path from GET operation}",
    "name": "{operation-id}", 
    "status" : "Succeeded | Failed | Canceled | {resource provider values}", 
    "startTime": "2017-01-06T20:56:36.002812+00:00",
    "endTime": "2017-01-06T20:56:56.002812+00:00",
    "percentComplete": {double between 0 and 100 },
    "properties": {
        /* Specific resource provider values for successful operations */
    },
    "error" : { 
        "code": "{error code}",  
        "message": "{error description}" 
    }
}
```

Solo se devuelve `status` para todas las respuestas. El objeto de error se devuelve cuando el estado es Failed o Canceled. Todos los demás valores son opcionales; por lo tanto, la respuesta que reciba puede ser diferente del ejemplo.

## <a name="provisioningstate-values"></a>Valores ProvisioningState

Las operaciones que crean, actualizan o eliminan (PUT, PATCH, DELETE) un recurso, normalmente devuelven un valor `provisioningState`. Cuando una operación ha finalizado, se devuelve uno de tres valores siguientes: 

* Correcto
* Con error
* Canceled

Todos los demás valores indican que la operación todavía se está ejecutando. El proveedor de recursos puede devolver un valor personalizado que indica su estado. Por ejemplo, puede recibir **Accepted** cuando la solicitud se ha recibido y está en ejecución.


## <a name="example-requests-and-responses"></a>Solicitudes y respuestas de ejemplo

### <a name="start-virtual-machine-202-with-azure-asyncoperation"></a>Inicio de máquina virtual (202 con Azure-AsyncOperation)
En este ejemplo se muestra cómo determinar el estado de la operación **start** para máquinas virtuales. La solicitud inicial está en el formato siguiente:

```HTTP
POST 
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Compute/virtualMachines/{vm-name}/start?api-version=2016-03-30
```

Devuelve el código de estado 202. Entre los valores de encabezado, verá:

```HTTP
Azure-AsyncOperation : https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2016-03-30
```

Para comprobar el estado de la operación asincrónica, envíe otra solicitud a esa dirección URL.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2016-03-30
```

El cuerpo de respuesta contiene el estado de la operación:

```json
{
  "startTime": "2017-01-06T18:58:24.7596323+00:00",
  "status": "InProgress",
  "name": "9a062a88-e463-4697-bef2-fe039df73a02"
}
```

### <a name="deploy-resources-201-with-azure-asyncoperation"></a>Implementación de recursos (201 con Azure-AsyncOperation)

En este ejemplo se muestra cómo determinar el estado de la operación **deployments** para implementar recursos en Azure. La solicitud inicial está en el formato siguiente:

```HTTP
PUT
https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/microsoft.resources/deployments/{deployment-name}?api-version=2016-09-01
```

Devuelve el código de estado 201. El cuerpo de la respuesta incluye:

```json
"provisioningState":"Accepted",
```

Entre los valores de encabezado, verá:

```HTTP
Azure-AsyncOperation: https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2016-09-01
```

Para comprobar el estado de la operación asincrónica, envíe otra solicitud a esa dirección URL.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2016-09-01
```

El cuerpo de respuesta contiene el estado de la operación:

```json
{"status":"Running"}
```

Cuando haya finalizado la implementación, la respuesta contiene:

```json
{"status":"Succeeded"}
```

### <a name="create-storage-account-202-with-location-and-retry-after"></a>Creación de cuenta de almacenamiento (202 con Location y Retry-After)

Este ejemplo muestra cómo determinar el estado de la operación **create** para cuentas de almacenamiento. La solicitud inicial está en el formato siguiente:

```HTTP
PUT
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}?api-version=2016-01-01
```

Y el cuerpo de solicitud contiene las propiedades de la cuenta de almacenamiento:

```json
{ "location": "South Central US", "properties": {}, "sku": { "name": "Standard_LRS" }, "kind": "Storage" }
```

Devuelve el código de estado 202. Entre los valores de encabezado, vea los dos valores siguientes:

```HTTP
Location: https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2016-01-01
Retry-After: 17
```

Después de esperar el número de segundos especificados en Retry-After, compruebe el estado de la operación asincrónica mediante el envío de otra solicitud a esa dirección URL.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2016-01-01
```

Si la solicitud aún se está ejecutando, recibe un código de estado 202. Si la solicitud se ha completado, recibe un código de estado 200 y el cuerpo de la respuesta contiene las propiedades de la cuenta de almacenamiento que se ha creado.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener documentación sobre cada operación de REST, consulte la [documentación de la API de REST](/rest/api/).
* Para información acerca de la implementación de plantillas a través de la API REST de Resource Manager, consulte [Implementación de recursos con las plantillas de Resource Manager y la API de REST de Resource Manager](../azure-resource-manager/templates/deploy-rest.md).