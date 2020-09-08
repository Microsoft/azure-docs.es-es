---
title: 'Configuración de registros de diagnóstico: Azure Event Hub | Microsoft Docs'
description: Obtenga información sobre cómo configurar registros de actividad y registros de diagnóstico para centros de eventos en Azure.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: ccd38d8924765df7bfd91b4fc26bb5304f6f180d
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88927738"
---
# <a name="set-up-diagnostic-logs-for-an-azure-event-hub"></a>Configuración de registros de diagnóstico de Azure Event Hubs

Puede ver dos tipos de registros para Event Hubs de Azure:

* **[Registros de actividad](../azure-monitor/platform/platform-logs-overview.md)** : Estos registros contienen información sobre las operaciones realizadas en un trabajo. Los registros están siempre habilitados. Puede ver las entradas del registro de actividad si selecciona **Registro de actividad** en el panel izquierdo del espacio de nombres del centro de eventos en Azure Portal. Por ejemplo: "Crear o actualizar el espacio de nombres", "Crear o actualizar el centro de eventos".

    ![Registro de actividad de un espacio de nombres de Event Hubs](./media/event-hubs-diagnostic-logs/activity-log.png)
* **[Registros de diagnóstico](../azure-monitor/platform/platform-logs-overview.md)** : los registros de diagnóstico proporcionan información más completa sobre las operaciones y acciones realizadas en el espacio de nombres mediante la API o mediante clientes de administración en el SDK de lenguaje. 
    
    En la sección siguiente se muestra cómo habilitar los registros de diagnóstico para un espacio de nombres de Event Hubs.

## <a name="enable-diagnostic-logs"></a>Habilitar registros de diagnóstico
Los registros de diagnóstico están inhabilitados de forma predeterminada. Para habilitarlos, siga estos pasos:

1.  En [Azure Portal](https://portal.azure.com), vaya al espacio de nombres de Event Hubs. 
2. Seleccione **Configuración de diagnóstico** en **Supervisión** en el panel izquierdo y, luego, elija **+ Agregar configuración de diagnóstico**. 

    ![Página Configuración de diagnóstico: Agregar configuración de diagnóstico](./media/event-hubs-diagnostic-logs/diagnostic-settings-page.png)
4. En la sección **Detalles de la categoría**, seleccione los **tipos de registros de diagnóstico** que quiere habilitar. Más adelante en este artículo encontrará información sobre estas categorías. 
5. En la sección **Detalles de destino**, establezca el destino de archivo que prefiera; por ejemplo, una cuenta de almacenamiento, un centro de eventos o un área de trabajo de Log Analytics.

    ![Agregar página de configuración de diagnóstico](./media/event-hubs-diagnostic-logs/aDD-diagnostic-settings-page.png)
6.  Seleccione **Guardar** en la barra de herramientas para guardar la configuración de diagnóstico.

    La nueva configuración surte efecto en unos 10 minutos. Después, los registros aparecen en el destino de archivo configurado, en el panel **Registros de diagnóstico**.

    Para obtener más información sobre el diagnóstico de configuraciones, consulte la [información general sobre los registros de diagnóstico de Azure](../azure-monitor/platform/platform-logs-overview.md).

## <a name="diagnostic-logs-categories"></a>Categorías de registros de diagnósticos

Event Hubs captura los registros de diagnóstico de las siguientes categorías:

| Category | Descripción | 
| -------- | ----------- | 
| Registros de archivo | Capturan información sobre las operaciones de [Event Hubs Capture](event-hubs-capture-overview.md), en concreto, registros relacionados con errores de captura. |
| Registros operativos | Capturan todas las operaciones de administración que se realizan en el espacio de nombres de Azure Event Hubs. Las operaciones de datos no se capturan debido al elevado volumen de este tipo de operaciones que se realizan en Azure Event Hubs. |
| Registros de escalabilidad automática | Capturan las operaciones de inflado automático realizadas en un espacio de nombres de Event Hubs. |
| Registros de coordinador de Kafka | Capturan las operaciones del coordinador de Kafka relacionadas con Event Hubs. |
| Registros de errores de usuario de Kafka | Capturan información sobre las API de Kafka llamadas en Event Hubs. |
| Evento de conexión de red virtual de Event Hubs | Captura información sobre las direcciones IP y las redes virtuales que envían tráfico a Event Hubs. |
| Registros de usuario de claves administradas por el cliente | Capturan las operaciones relacionadas con las claves administradas por el cliente. |


Todos los registros se almacenan en el formato de notación de objetos JavaScript (JSON). Cada entrada tiene campos de cadena que usan el formato descrito en las secciones siguientes.

## <a name="archive-logs-schema"></a>Esquema de registros de archivo

Las cadenas JSON de registros de archivo incluyen elementos enumerados en la tabla siguiente:

Nombre | Descripción
------- | -------
`TaskName` | La descripción de la tarea que produjo error
`ActivityId` | El identificador interno, usado con fines de seguimiento
`trackingId` | El identificador interno, usado con fines de seguimiento
`resourceId` | El identificador de recursos de Azure Resource Manager
`eventHub` | Nombre completo del centro de eventos (incluye el nombre del espacio de nombres)
`partitionId` | La partición del centro de eventos en la que se escribe.
`archiveStep` | Valores posibles: ArchiveFlushWriter, DestinationInit
`startTime` | Hora de inicio del error
`failures` | Número de veces que se ha producido el error.
`durationInSeconds` | La duración del error
`message` | Mensaje de error
`category` | ArchiveLogs

El código siguiente es un ejemplo de una cadena JSON de registro de archivo:

```json
{
   "TaskName": "EventHubArchiveUserError",
   "ActivityId": "000000000-0000-0000-0000-0000000000000",
   "trackingId": "0000000-0000-0000-0000-00000000000000000",
   "resourceId": "/SUBSCRIPTIONS/000000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/<Event Hubs Namespace Name>",
   "eventHub": "<Event Hub full name>",
   "partitionId": "1",
   "archiveStep": "ArchiveFlushWriter",
   "startTime": "9/22/2016 5:11:21 AM",
   "failures": 3,
   "durationInSeconds": 360,
   "message": "Microsoft.WindowsAzure.Storage.StorageException: The remote server returned an error: (404) Not Found. ---> System.Net.WebException: The remote server returned an error: (404) Not Found.\r\n   at Microsoft.WindowsAzure.Storage.Shared.Protocol.HttpResponseParsers.ProcessExpectedStatusCodeNoException[T](HttpStatusCode expectedStatusCode, HttpStatusCode actualStatusCode, T retVal, StorageCommandBase`1 cmd, Exception ex)\r\n   at Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob.<PutBlockImpl>b__3e(RESTCommand`1 cmd, HttpWebResponse resp, Exception ex, OperationContext ctx)\r\n   at Microsoft.WindowsAzure.Storage.Core.Executor.Executor.EndGetResponse[T](IAsyncResult getResponseResult)\r\n   --- End of inner exception stack trace ---\r\n   at Microsoft.WindowsAzure.Storage.Core.Util.StorageAsyncResult`1.End()\r\n   at Microsoft.WindowsAzure.Storage.Core.Util.AsyncExtensions.<>c__DisplayClass4.<CreateCallbackVoid>b__3(IAsyncResult ar)\r\n--- End of stack trace from previous location where exception was thrown ---\r\n   at System.",
   "category": "ArchiveLogs"
}
```

## <a name="operational-logs-schema"></a>Esquema de registros operativos

Las cadenas JSON de registros operativos incluyen elementos enumerados en la tabla siguiente:

Nombre | Descripción
------- | -------
`ActivityId` | Identificador interno, usado con fines de seguimiento. |
`EventName` | Nombre de operación |
`resourceId` | El identificador de recursos de Azure Resource Manager |
`SubscriptionId` | Id. de suscripción |
`EventTimeString` | Hora de la operación |
`EventProperties` | Propiedades de la operación |
`Status` | Estado de la operación |
`Caller` | Autor de la llamada de la operación (Azure Portal o Management Client) |
`Category` | OperationalLogs |

El código siguiente es un ejemplo de una cadena JSON de registro operativo:

```json
Example:
{
   "ActivityId": "00000000-0000-0000-0000-00000000000000",
   "EventName": "Create EventHub",
   "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/<Event Hubs namespace name>",
   "SubscriptionId": "000000000-0000-0000-0000-000000000000",
   "EventTimeString": "9/28/2016 8:40:06 PM +00:00",
   "EventProperties": "{\"SubscriptionId\":\"0000000000-0000-0000-0000-000000000000\",\"Namespace\":\"<Namespace Name>\",\"Via\":\"https://<Namespace Name>.servicebus.windows.net/f8096791adb448579ee83d30e006a13e/?api-version=2016-07\",\"TrackingId\":\"5ee74c9e-72b5-4e98-97c4-08a62e56e221_G1\"}",
   "Status": "Succeeded",
   "Caller": "ServiceBus Client",
   "category": "OperationalLogs"
}
```

## <a name="autoscale-logs-schema"></a>Esquema de registros de escalabilidad automática
Las cadenas JSON del registro de escalabilidad automática incluyen los elementos enumerados en la tabla siguiente:

| Nombre | Descripción |
| ---- | ----------- | 
| `TrackingId` | Identificador interno, que se usa con fines de seguimiento. |
| `ResourceId` | Identificador de recursos de Azure Resource Manager. |
| `Message` | Mensaje informativo, que proporciona detalles sobre la acción de inflado automático. El mensaje contiene el valor anterior y actual de la unidad de procesamiento de un espacio de nombres determinado y lo que desencadenó el inflado de la unidad de rendimiento. |

Este es un evento de escalado automático de ejemplo: 

```json
{
    "TrackingId": "fb1b3676-bb2d-4b17-85b7-be1c7aa1967e",
    "Message": "Scaled-up EventHub TUs (UpdateStartTimeUTC: 5/13/2020 7:48:36 AM, PreviousValue: 1, UpdatedThroughputUnitValue: 2, AutoScaleReason: 'IncomingMessagesPerSecond reached 2170')",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name"
}
```

## <a name="kafka-coordinator-logs-schema"></a>Esquema de registros de coordinador de Kafka
Las cadenas JSON del registro de coordinador de Kafka incluyen los elementos enumerados en la tabla siguiente:

| Nombre | Descripción |
| ---- | ----------- | 
| `RequestId` | Identificador de solicitud, que se usa con fines de seguimiento. |
| `ResourceId` | El identificador de recursos de Azure Resource Manager |
| `Operation` | Nombre de la operación que se realiza durante la coordinación de grupos. |
| `ClientId` | Id. de cliente |
| `NamespaceName` | Nombre del espacio de nombres | 
| `SubscriptionId` | Identificador de suscripción de Azure |
| `Message` | Mensaje informativo o de advertencia, que proporciona detalles sobre las acciones realizadas durante la coordinación de grupos. |

### <a name="example"></a>Ejemplo

```json
{
    "RequestId": "FE01001A89E30B020000000304620E2A_KafkaExampleConsumer#0",
    "Operation": "Join.Start",
    "ClientId": "KafkaExampleConsumer#0",
    "Message": "Start join group for new member namespace-name:c:$default:I:KafkaExampleConsumer#0-cc40856f7f3c4607915a571efe994e82, current group size: 0, API version: 2, session timeout: 10000ms, rebalance timeout: 300000ms.",
    "SubscriptionId": "0000000-0000-0000-0000-000000000000",
    "NamespaceName": "namespace-name",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name",
    "Category": "KafkaCoordinatorLogs"
}
```

## <a name="kafka-user-error-logs-schema"></a>Esquema de registros de error de usuario de Kafka
Las cadenas JSON del registro de errores de Kafka incluyen los elementos enumerados en la tabla siguiente:

| Nombre | Descripción |
| ---- | ----------- |
| `TrackingId` | Identificador de seguimiento, que se usa con fines de seguimiento. |
| `NamespaceName` | Nombre del espacio de nombres |
| `Eventhub` | Nombre del centro de eventos |
| `PartitionId` | Id. de partición |
| `GroupId` | Identificador de grupo |
| `ClientId` | Id. de cliente |
| `ResourceId` | Identificador de recursos de Azure Resource Manager. |
| `Message` | Mensaje informativo, que proporciona detalles sobre un error. |

## <a name="event-hubs-virtual-network-connection-event-schema"></a>Esquema de eventos de conexión de red virtual de Event Hubs

Las cadenas JSON del evento de conexión de red virtual (VNet) de Event Hubs contienen los elementos enumerados en la tabla siguiente:

| Nombre | Descripción |
| ---  | ----------- | 
| `SubscriptionId` | Identificador de suscripción de Azure |
| `NamespaceName` | Nombre del espacio de nombres |
| `IPAddress` | Dirección IP de un cliente que se conecta al servicio de Event Hubs. |
| `Action` | Acción realizada por el servicio Event Hubs al evaluar las solicitudes de conexión. Las acciones admitidas son **Accept Connection** (Aceptar conexión) y **Deny Connection** (Denegar conexión). |
| `Reason` | Proporciona el motivo de que se haya realizado la acción. |
| `Count` | Número de repeticiones de una acción dada. |
| `ResourceId` | Identificador de recursos de Azure Resource Manager. |

### <a name="example"></a>Ejemplo

```json
{
    "SubscriptionId": "0000000-0000-0000-0000-000000000000",
    "NamespaceName": "namespace-name",
    "IPAddress": "1.2.3.4",
    "Action": "Deny Connection",
    "Reason": "IPAddress doesn't belong to a subnet with Service Endpoint enabled.",
    "Count": "65",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name",
    "Category": "EventHubVNetConnectionEvent"
}
```

## <a name="customer-managed-key-user-logs"></a>Registros de usuario de claves administradas por el cliente
Las cadenas JSON del registro de usuario de claves administradas por el cliente incluyen los elementos enumerados en la tabla siguiente:

| Nombre | Descripción |
| ---- | ----------- | 
| `Category` | Tipo de categoría de un mensaje. Corresponde a uno de los siguientes valores: **error** e **info** |
| `ResourceId` | Identificador de recurso interno, que incluye el identificador de suscripción y el nombre del espacio de nombres de Azure. |
| `KeyVault` | Nombre del recurso de Key Vault. |
| `Key` | Nombre de la clave de Key Vault. |
| `Version` | Versión de la clave de Key Vault. |
| `Operation` | El nombre de una operación realizada para atender solicitudes. |
| `Code` | Código de estado |
| `Message` | Mensaje, que proporciona detalles sobre un error o un mensaje informativo. |



## <a name="next-steps"></a>Pasos siguientes
- [Introducción a Event Hubs](./event-hubs-about.md)
- [Ejemplos de Event Hubs](sdks.md)
- Introducción a Event Hubs
    - [.NET Core](event-hubs-dotnet-standard-getstarted-send.md)
    - [Java](event-hubs-java-get-started-send.md)
    - [Python](event-hubs-python-get-started-send.md)
    - [JavaScript](event-hubs-java-get-started-send.md)
