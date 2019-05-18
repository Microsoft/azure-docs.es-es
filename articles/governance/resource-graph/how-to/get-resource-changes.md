---
title: Obtención de los cambios del recurso
description: Comprenda cómo averiguar cuándo se cambió un recurso y obtener una lista de las propiedades que puede cambiar.
services: resource-graph
author: DCtheGeek
ms.author: dacoulte
ms.date: 05/10/2019
ms.topic: conceptual
ms.service: resource-graph
manager: carmonm
ms.openlocfilehash: 4e28ca15197f89caeaeaca0aabb648755b8235f1
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65551542"
---
# <a name="get-resource-changes"></a>Obtención de los cambios del recurso

Los cambios a lo largo de uso diario, reconfiguración y reimplementación incluso los recursos.
Cambio puede proceder de un usuario individual o por un proceso automatizado. La mayoría de cambios es así por diseño, pero a veces no lo es. Con los últimos 14 días del historial de cambios, gráfico de recursos de Azure le permite:

- Buscar cuándo se detectaron cambios en una propiedad de Azure Resource Manager.
- Ver qué propiedades cambiaron como parte de ese evento de cambio.

Detección de cambios y los detalles son valiosos para los escenarios de ejemplo siguiente:

- Durante la administración de incidentes para comprender _potencialmente_ relacionados con los cambios. Consultar los eventos de cambio durante un período de tiempo específico y evaluar los detalles del cambio.
- Mantener una base de datos de administración de configuración, se conoce como una CMDB, actualizada. En lugar de actualizar todos los recursos y sus conjuntos de propiedad completa en una frecuencia programada, solo obtiene qué ha cambiado.
- Descripción de las otras propiedades de que pueden haber cambiado cuando un recurso puede cambiar el estado de cumplimiento. Evaluación de estas propiedades adicionales puede proporcionar información sobre otras propiedades que es posible que deba administrarse a través de una definición de directiva de Azure.

En este artículo se muestra cómo recopilar esta información a través del SDK del gráfico de recursos. Para ver esta información en el portal de Azure, consulte la directiva de Azure [historial de cambios](../../policy/how-to/determine-non-compliance.md#change-history-preview) o Azure Activity Log [historial de cambios](../../../azure-monitor/platform/activity-logs-overview.md#view-change-history).

> [!NOTE]
> Detalles del cambio en el gráfico de recursos son propiedades del Administrador de recursos. Para realizar el seguimiento de cambios dentro de una máquina virtual, consulte Azure Automation [el seguimiento de cambios](../../../automation/automation-change-tracking.md) o la directiva de Azure [configuración de invitado para máquinas virtuales](../../policy/concepts/guest-configuration.md).

> [!IMPORTANT]
> Historial de cambios en el gráfico de recursos de Azure está en versión preliminar pública.

## <a name="find-when-changes-were-detected"></a>Buscar cuando se detectaron cambios

El primer paso para ver qué cambió en un recurso es encontrar los eventos de cambio relacionados con ese recurso dentro de una ventana de tiempo. Este paso se realiza a través de la **resourceChanges** extremo REST.

El **resourceChanges** extremo requiere dos parámetros en el cuerpo de solicitud:

- **resourceId**: El recurso de Azure para buscar cambios.
- **interval**: Una propiedad con _iniciar_ y _final_ fechas para cuándo se debe comprobar si un evento de cambio mediante la **Zulu zona horaria (Z)**.

Ejemplo del cuerpo de solicitud:

```json
{
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "interval": {
        "start": "2019-03-28T00:00:00.000Z",
        "end": "2019-03-31T00:00:00.000Z"
    }
}
```

Con el cuerpo de la solicitud anterior, el URI de la API de REST para **resourceChanges** es:

```http
POST https://management.azure.com/providers/Microsoft.ResourceGraph/resourceChanges?api-version=2018-09-01-preview
```

La respuesta será similar a este ejemplo:

```json
{
    "changes": [{
            "changeId": "{\"beforeId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-09T00:00:00.000Z\",\"afterId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-10T00:00:00.000Z\"}",
            "beforeSnapshot": {
                "timestamp": "2019-03-29T01:32:05.993Z"
            },
            "afterSnapshot": {
                "timestamp": "2019-03-29T01:54:24.42Z"
            }
        },
        {
            "changeId": "9dc352cb-b7c1-4198-9eda-e5e3ed66aec8",
            "beforeSnapshot": {
                "timestamp": "2019-03-28T10:30:19.68Z"
            },
            "afterSnapshot": {
                "timestamp": "2019-03-28T21:12:31.337Z"
            }
        }
    ]
}
```

Cada detectado evento de cambio para el **resourceId** tiene un **changeId** que es único para ese recurso. Mientras el **changeId** cadena a veces puede contener otras propiedades, solo se garantiza que sea único. El registro de cambios incluye las horas que el antes y después de que se realicen las instantáneas.
Se ha producido el evento de cambio en algún momento en esta ventana de tiempo.

## <a name="see-what-properties-changed"></a>Vea lo que cambiar las propiedades

Con el **changeId** desde el **resourceChanges** punto de conexión, el **resourceChangeDetails** extremo REST, a continuación, se utiliza para obtener detalles del evento de cambio.

El **resourceChangeDetails** extremo requiere dos parámetros en el cuerpo de solicitud:

- **resourceId**: El recurso de Azure para buscar cambios.
- **changeId**: El evento de cambio exclusivo para el **resourceId** recopilada desde **resourceChanges**.

Ejemplo del cuerpo de solicitud:

```json
{
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "changeId": "{\"beforeId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-09T00:00:00.000Z\",\"afterId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-10T00:00:00.000Z\"}"
}
```

Con el cuerpo de la solicitud anterior, el URI de la API de REST para **resourceChangeDetails** es:

```http
POST https://management.azure.com/providers/Microsoft.ResourceGraph/resourceChangeDetails?api-version=2018-09-01-preview
```

La respuesta será similar a este ejemplo:

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

**beforeSnapshot** y **afterSnapshot** cada dar a la hora en que se tomó la instantánea y las propiedades en ese momento. Se produjo el cambio en algún momento entre estas instantáneas. Examinar el ejemplo anterior, podemos ver que la propiedad que cambió fue **supportsHttpsTrafficOnly**.

Para comparar los resultados mediante programación, compare la **contenido** parte de cada instantánea para determinar la diferencia. Si se compara la instantánea completa, el **timestamp** siempre se muestra como una diferencia a pesar de que se espera.

## <a name="next-steps"></a>Pasos siguientes

- Puede ver el idioma en uso en [consultas Starter](../samples/starter.md).
- Vea avanzada se usa en [consultas avanzadas](../samples/advanced.md).
- Aprenda a [explore recursos](../concepts/explore-resources.md).