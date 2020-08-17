---
title: Obtención de los cambios del recurso
description: Aprenda los métodos para identificar cuándo un recurso se cambió, obtener una lista de las propiedades que cambiaron y evaluar las diferencias.
ms.date: 08/10/2020
ms.topic: how-to
ms.openlocfilehash: 02dfd2b8b66cd09be39a44166a0fd62bb13c8395
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056574"
---
# <a name="get-resource-changes"></a>Obtención de los cambios del recurso

Los recursos se modifican a través del uso diario, la reconfiguración e incluso la reimplementación.
Los cambios pueden proceder de un usuario individual o un proceso automatizado. La mayoría de los cambios son así por diseño, pero a veces no lo son. Con los últimos 14 días de historial de cambios,Azure Resource Graph le permite hacer lo siguiente:

- Buscar cuándo se detectaron cambios en una propiedad de Azure Resource Manager
- Para cada cambio de recurso, consultar los detalles del cambio de propiedad
- Ver una comparación completa del recurso antes y después del cambio detectado

La detección y los detalles de cambios son de utilidad para los siguientes escenarios de ejemplo:

- Durante la administración de incidentes para comprender los cambios _potencialmente_ relacionados. Consultar los eventos de cambio durante un período de tiempo específico y evaluar los detalles del cambio.
- Mantener actualizada una base de datos de administración de configuración, conocida como una CMDB. En lugar de actualizar todos los recursos y sus conjuntos de propiedad completos según una frecuencia programada, obtener solo lo que cambió.
- Comprender qué otras propiedades podrían haber cambiado cuando un recurso puede cambió de estado de cumplimiento. La evaluación de estas propiedades adicionales puede proporcionar información sobre otras propiedades que quizás tengan que administrarse a través de una definición de Azure Policy.

En este artículo se muestra cómo recopilar esta información a través del SDK del Resource Graph. Para ver esta información en Azure Portal, consulte Azure Policy [Historial de cambios](../../policy/how-to/determine-non-compliance.md#change-history) o [Historial de cambios](../../../azure-monitor/platform/activity-log.md#view-the-activity-log) del Registro de actividad de Azure. Para obtener detalles sobre los cambios en sus aplicaciones desde el nivel de infraestructura hasta la implementación de aplicaciones, consulte el artículo sobre el [uso de Application Change Analysis (versión preliminar)](../../../azure-monitor/app/change-analysis.md) en Azure Monitor.

> [!NOTE]
> Los detalles de cambio en Resource Graph son para las propiedades de Resource Manager. Para realizar un seguimiento de los cambios dentro de una máquina virtual, consulte las opciones [Seguimiento de cambios](../../../automation/change-tracking.md) o Azure Policy [Configuración de invitado para VM](../../policy/concepts/guest-configuration.md) de Azure Automation.

> [!IMPORTANT]
> El historial de cambios en Azure Resource Graph está disponible como versión preliminar pública.

## <a name="find-detected-change-events-and-view-change-details"></a>Buscar eventos de cambio detectados y ver detalles de los cambios

El primer paso para ver qué cambió en un recurso es buscar los eventos de cambio relacionados con ese recurso dentro de una ventana de tiempo. Cada evento de cambio también incluye detalles sobre lo que ha cambiado en el recurso. Este paso se realiza a través del punto de conexión REST **resourceChanges**.

El punto de conexión **resourceChanges** acepta los siguientes parámetros en el cuerpo de solicitud:

- **resourceId** \[obligatorio\]: el recurso de Azure para el que se buscan cambios.
- **interval** \[obligatorio\]: una propiedad con fecha de _inicio_ y _finalización_ durante las que se buscará evento de cambio mediante la **zona horaria Zulu (Z)** .
- **fetchPropertyChanges** (opcional): una propiedad booleana que establece si el objeto de respuesta incluye los cambios de propiedad.

Ejemplo del cuerpo de solicitud:

```json
{
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "interval": {
        "start": "2019-09-28T00:00:00.000Z",
        "end": "2019-09-29T00:00:00.000Z"
    },
    "fetchPropertyChanges": true
}
```

Con el cuerpo de solicitud anterior, el URI de la API de REST para **resourceChanges** es:

```http
POST https://management.azure.com/providers/Microsoft.ResourceGraph/resourceChanges?api-version=2018-09-01-preview
```

La respuesta será similar a la siguiente:

```json
{
    "changes": [
        {
            "changeId": "{\"beforeId\":\"3262e382-9f73-4866-a2e9-9d9dbee6a796\",\"beforeTime\":\"2019-09-28T00:45:35.012Z\",\"afterId\":\"6178968e-981e-4dac-ac37-340ee73eb577\",\"afterTime\":\"2019-09-28T00:52:53.371Z\"}",
            "beforeSnapshot": {
                "snapshotId": "3262e382-9f73-4866-a2e9-9d9dbee6a796",
                "timestamp": "2019-09-28T00:45:35.012Z"
            },
            "afterSnapshot": {
                "snapshotId": "6178968e-981e-4dac-ac37-340ee73eb577",
                "timestamp": "2019-09-28T00:52:53.371Z"
            },
            "changeType": "Create"
        },
        {
            "changeId": "{\"beforeId\":\"a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c\",\"beforeTime\":\"2019-09-28T00:43:38.366Z\",\"afterId\":\"3262e382-9f73-4866-a2e9-9d9dbee6a796\",\"afterTime\":\"2019-09-28T00:45:35.012Z\"}",
            "beforeSnapshot": {
                "snapshotId": "a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c",
                "timestamp": "2019-09-28T00:43:38.366Z"
            },
            "afterSnapshot": {
                "snapshotId": "3262e382-9f73-4866-a2e9-9d9dbee6a796",
                "timestamp": "2019-09-28T00:45:35.012Z"
            },
            "changeType": "Delete"
        },
        {
            "changeId": "{\"beforeId\":\"b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c\",\"beforeTime\":\"2019-09-28T00:43:15.518Z\",\"afterId\":\"a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c\",\"afterTime\":\"2019-09-28T00:43:38.366Z\"}",
            "beforeSnapshot": {
                "snapshotId": "b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c",
                "timestamp": "2019-09-28T00:43:15.518Z"
            },
            "afterSnapshot": {
                "snapshotId": "a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c",
                "timestamp": "2019-09-28T00:43:38.366Z"
            },
            "propertyChanges": [
                {
                    "propertyName": "tags.org",
                    "afterValue": "compute",
                    "changeCategory": "User",
                    "changeType": "Insert"
                },
                {
                    "propertyName": "tags.team",
                    "afterValue": "ARG",
                    "changeCategory": "User",
                    "changeType": "Insert"
                }
            ],
            "changeType": "Update"
        },
        {
            "changeId": "{\"beforeId\":\"19d12ab1-6ac6-4cd7-a2fe-d453a8e5b268\",\"beforeTime\":\"2019-09-28T00:42:46.839Z\",\"afterId\":\"b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c\",\"afterTime\":\"2019-09-28T00:43:15.518Z\"}",
            "beforeSnapshot": {
                "snapshotId": "19d12ab1-6ac6-4cd7-a2fe-d453a8e5b268",
                "timestamp": "2019-09-28T00:42:46.839Z"
            },
            "afterSnapshot": {
                "snapshotId": "b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c",
                "timestamp": "2019-09-28T00:43:15.518Z"
            },
            "propertyChanges": [{
                "propertyName": "tags.cgtest",
                "afterValue": "hello",
                "changeCategory": "User",
                "changeType": "Insert"
            }],
            "changeType": "Update"
        }
    ]
}
```

Cada evento de cambio detectado para el **resourceId** tiene las siguientes propiedades:

- **changeId**: este valor es único para ese recurso. Si bien la cadena **changeId** a veces puede contener otras propiedades, solo se garantiza que sea única.
- **beforeSnapshot**: contiene el valor de **snapshotId** y de **timestamp** de la instantánea de recursos que se tomó antes de que se detectara un cambio.
- **afterSnapshot**: contiene el valor de **snapshotId** y de **timestamp** de la instantánea de recursos que se tomó después de que se detectara un cambio.
- **changeType**: describe el tipo de cambio detectado para el registro de cambios completo entre **beforeSnapshot** y **afterSnapshot**. Los valores son: _Create_ (Crear), _Update_ (Actualizar) y _Delete_ (Eliminar). La matriz de propiedades **propertyChanges** solo se incluye cuando **changeType** es _Update_.
- **propertyChanges**: esta matriz de propiedades detalla todas las propiedades de los recursos que se actualizaron entre **beforeSnapshot** y **afterSnapshot**:
  - **propertyName**: el nombre de la propiedad de recurso que se modificó.
  - **changeCategory**: describe lo que hizo el cambio. Los valores son: _System_ (Sistema) y _User_ (Usuario).
  - **changeType**: describe el tipo de cambio detectado para la propiedad de recurso individual.
    Los valores son: _Insert_ (Insertar), _Update_ (Actualizar), _Remove_ (Eliminar).
  - **beforeValue**: el valor de la propiedad de recurso en el **beforeSnapshot**. No se muestra cuando **changeType** es _Insert_.
  - **afterValue**: el valor de la propiedad de recurso en el **afterSnapshot**. No se muestra cuando **changeType** es _Remove_.

## <a name="compare-resource-changes"></a>Comparación de los cambios del recurso

Con el objeto **changeId** del punto de conexión **resourceChanges**, el punto de conexión REST **resourceChangeDetails** luego se usa para obtener las instantáneas de antes y de después del recurso que cambió.

El punto de conexión **resourceChangeDetails** requiere dos parámetros en el cuerpo de solicitud:

- **resourceId**: el recurso de Azure con el que comparar los cambios.
- **changeId**: el evento de cambio único para el objeto **resourceId** recopilado desde **resourceChanges**.

Ejemplo del cuerpo de solicitud:

```json
{
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "changeId": "{\"beforeId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-09T00:00:00.000Z\",\"afterId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"afterTime\":'2019-05-10T00:00:00.000Z\"}"
}
```

Con el cuerpo de solicitud anterior, el URI de la API de REST para **resourceChangeDetails** es:

```http
POST https://management.azure.com/providers/Microsoft.ResourceGraph/resourceChangeDetails?api-version=2018-09-01-preview
```

La respuesta será similar a la siguiente:

```json
{
    "changeId": "{\"beforeId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-09T00:00:00.000Z\",\"afterId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-10T00:00:00.000Z\"}",
    "beforeSnapshot": {
        "timestamp": "2019-03-29T01:32:05.993Z",
        "content": {
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "kind": "Storage",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
            "name": "mystorageaccount",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "westus",
            "tags": {},
            "properties": {
                "networkAcls": {
                    "bypass": "AzureServices",
                    "virtualNetworkRules": [],
                    "ipRules": [],
                    "defaultAction": "Allow"
                },
                "supportsHttpsTrafficOnly": false,
                "encryption": {
                    "services": {
                        "file": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        },
                        "blob": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        }
                    },
                    "keySource": "Microsoft.Storage"
                },
                "provisioningState": "Succeeded",
                "creationTime": "2018-07-27T18:37:21.7708872Z",
                "primaryEndpoints": {
                    "blob": "https://mystorageaccount.blob.core.windows.net/",
                    "queue": "https://mystorageaccount.queue.core.windows.net/",
                    "table": "https://mystorageaccount.table.core.windows.net/",
                    "file": "https://mystorageaccount.file.core.windows.net/"
                },
                "primaryLocation": "westus",
                "statusOfPrimary": "available"
            }
        }
    },
    "afterSnapshot": {
        "timestamp": "2019-03-29T01:54:24.42Z",
        "content": {
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "kind": "Storage",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
            "name": "mystorageaccount",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "westus",
            "tags": {},
            "properties": {
                "networkAcls": {
                    "bypass": "AzureServices",
                    "virtualNetworkRules": [],
                    "ipRules": [],
                    "defaultAction": "Allow"
                },
                "supportsHttpsTrafficOnly": true,
                "encryption": {
                    "services": {
                        "file": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        },
                        "blob": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        }
                    },
                    "keySource": "Microsoft.Storage"
                },
                "provisioningState": "Succeeded",
                "creationTime": "2018-07-27T18:37:21.7708872Z",
                "primaryEndpoints": {
                    "blob": "https://mystorageaccount.blob.core.windows.net/",
                    "queue": "https://mystorageaccount.queue.core.windows.net/",
                    "table": "https://mystorageaccount.table.core.windows.net/",
                    "file": "https://mystorageaccount.file.core.windows.net/"
                },
                "primaryLocation": "westus",
                "statusOfPrimary": "available"
            }
        }
    }
}
```

Los objetos **beforeSnapshot** y **afterSnapshot** proporcionan cada uno la hora en que se capturó la instantánea y las propiedades en ese momento. El cambio se produjo en algún momento entre estas instantáneas. Al examinar el ejemplo anterior, podemos ver que la propiedad que cambió fue **supportsHttpsTrafficOnly**.

Para comparar los resultados, use la propiedad **changes** en **resourceChanges** o bien evalúe la parte **content** de cada instantánea en **resourceChangeDetails** para determinar la diferencia. Si compara las instantáneas, el objeto **timestamp** siempre se muestra como una diferencia, a pesar de que sea esperado.

## <a name="next-steps"></a>Pasos siguientes

- Consulte el lenguaje en uso en[Consultas básicas](../samples/starter.md).
- Consulte los usos avanzados en [Consultas avanzadas](../samples/advanced.md).
- Obtenga más información sobre cómo [explorar recursos](../concepts/explore-resources.md).
- Para instrucciones sobre cómo trabajar con consultas con mucha frecuencia, consulte [Guía para solicitudes limitadas](../concepts/guidance-for-throttled-requests.md).
