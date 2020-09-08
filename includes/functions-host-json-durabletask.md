---
title: archivo de inclusión
description: archivo de inclusión
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/14/2019
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 6e253604c57d73c2a89ccfa5cff7efe9e572d11d
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89094133"
---
Configuración de [Durable Functions](../articles/azure-functions/durable-functions-overview.md).

### <a name="durable-functions-1x"></a>Durable Functions 1.x

```json
{
  "durableTask": {
    "hubName": "MyTaskHub",
    "controlQueueBatchSize": 32,
    "partitionCount": 4,
    "controlQueueVisibilityTimeout": "00:05:00",
    "workItemQueueVisibilityTimeout": "00:05:00",
    "maxConcurrentActivityFunctions": 10,
    "maxConcurrentOrchestratorFunctions": 10,
    "maxQueuePollingInterval": "00:00:30",
    "azureStorageConnectionStringName": "AzureWebJobsStorage",
    "trackingStoreConnectionStringName": "TrackingStorage",
    "trackingStoreNamePrefix": "DurableTask",
    "traceInputsAndOutputs": false,
    "logReplayEvents": false,
    "eventGridTopicEndpoint": "https://topic_name.westus2-1.eventgrid.azure.net/api/events",
    "eventGridKeySettingName":  "EventGridKey",
    "eventGridPublishRetryCount": 3,
    "eventGridPublishRetryInterval": "00:00:30",
    "eventGridPublishEventTypes": ["Started", "Completed", "Failed", "Terminated"]
  }
}
```

### <a name="durable-functions-2x"></a><a name="durable-functions-2-0-host-json"></a>Durable Functions 2.x

```json
{
 "extensions": {
  "durableTask": {
    "hubName": "MyTaskHub",
    "storageProvider": {
      "connectionStringName": "AzureWebJobsStorage",
      "controlQueueBatchSize": 32,
      "controlQueueBufferThreshold": 256,
      "controlQueueVisibilityTimeout": "00:05:00",
      "maxQueuePollingInterval": "00:00:30",
      "partitionCount": 4,
      "trackingStoreConnectionStringName": "TrackingStorage",
      "trackingStoreNamePrefix": "DurableTask",
      "useLegacyPartitionManagement": true,
      "workItemQueueVisibilityTimeout": "00:05:00",
    },
    "tracing": {
      "traceInputsAndOutputs": false,
      "traceReplayEvents": false,
    },
    "notifications": {
      "eventGrid": {
        "topicEndpoint": "https://topic_name.westus2-1.eventgrid.azure.net/api/events",
        "keySettingName": "EventGridKey",
        "publishRetryCount": 3,
        "publishRetryInterval": "00:00:30",
        "publishEventTypes": [
          "Started",
          "Pending",
          "Failed",
          "Terminated"
        ]
      }
    },
    "maxConcurrentActivityFunctions": 10,
    "maxConcurrentOrchestratorFunctions": 10,
    "extendedSessionsEnabled": false,
    "extendedSessionIdleTimeoutInSeconds": 30,
    "useAppLease": true,
    "useGracefulShutdown": false
  }
 }
}

```

Los nombres de la central de tareas deben empezar por una letra y estar formados únicamente por letras y números. Si no se especifica, el nombre predeterminado de la central de tareas de la aplicación de función es **DurableFunctionsHub**. Para más información, consulte el artículo sobre las [centrales de tareas](../articles/azure-functions/durable-functions-task-hubs.md).

|Propiedad  |Valor predeterminado | Descripción |
|---------|---------|---------|
|hubName|DurableFunctionsHub|Se pueden usar nombres de [central de tareas](../articles/azure-functions/durable-functions-task-hubs.md) alternativos para aislar varias aplicaciones de Durable Functions unas de otras, incluso si usan el mismo back-end de almacenamiento.|
|controlQueueBatchSize|32|El número de mensajes que se van a extraer a la vez de la cola de control.|
|controlQueueBufferThreshold|256|Número de mensajes de la cola de control que se pueden almacenar en búfer en la memoria a la vez, momento en que el distribuidor esperará para quitar cualquier mensaje adicional de la cola.|
|partitionCount |4|El recuento de particiones para la cola de control. Puede ser un entero positivo comprendido entre 1 y 16.|
|controlQueueVisibilityTimeout |5 minutos|El tiempo de espera de visibilidad de los mensajes de la cola de control quitados de la cola.|
|workItemQueueVisibilityTimeout |5 minutos|El tiempo de espera de visibilidad de los mensajes de la cola de elementos de trabajo quitados de la cola.|
|maxConcurrentActivityFunctions |10 veces el número de procesadores en la máquina actual.|El número máximo de funciones de actividad que se pueden procesar simultáneamente en una única instancia de host.|
|maxConcurrentOrchestratorFunctions |10 veces el número de procesadores en la máquina actual.|El número máximo de funciones de Orchestrator que se pueden procesar simultáneamente en una única instancia de host.|
|maxQueuePollingInterval|30 segundos|Intervalo de sondeo de cola de elementos de trabajo y control máximo en formato *hh:mm:ss:* . Los valores más altos pueden provocar mayores latencias de procesamiento de mensajes. Los valores más bajos pueden provocar mayores costos de almacenamiento debido a transacciones de almacenamiento mayor.|
|azureStorageConnectionStringName |AzureWebJobsStorage|El nombre de la configuración de aplicación que tiene la cadena de conexión de Azure Storage que se usa para administrar los recursos subyacentes de Azure Storage.|
|trackingStoreConnectionStringName||Nombre de una cadena de conexión que se usará para las tablas de historial e instancias. Si no se especifica, se usa las conexiones `connectionStringName` (Durable 2.x) o `azureStorageConnectionStringName` (Durable 1.x).|
|trackingStoreNamePrefix||Prefijo que se usará para las tablas de historial e instancias cuando se especifica `trackingStoreConnectionStringName`. Si no se establece, el valor de prefijo predeterminado será `DurableTask`. Si no se especifica `trackingStoreConnectionStringName`, las tablas de historial e instancias usarán el valor `hubName` como prefijo y se pasarán por alto todos los valores de configuración de `trackingStoreNamePrefix`.|
|traceInputsAndOutputs |false|Un valor que indica si se realizará el seguimiento de las entradas y salidas de las llamadas de función. El comportamiento predeterminado al realizar el seguimiento de eventos de ejecución de funciones es incluir el número de bytes en las entradas y salidas serializadas de las llamadas de función. Este comportamiento proporciona información mínima sobre el aspecto de las entradas y salidas, sin sobredimensionar los registros o exponer por accidente información confidencial. Al establecer esta propiedad en true, el registro de funciones predeterminado registra todo el contenido de las entradas y salidas de función.|
|logReplayEvents|false|Un valor que indica si se debe escribir eventos de reproducción de orquestación en Application Insights.|
|eventGridTopicEndpoint ||La dirección URL de un punto de conexión de tema personalizado de Azure Event Grid. Cuando se establece esta propiedad, se publican eventos de notificación del ciclo de vida de orquestación en este punto de conexión. Esta propiedad admite la resolución de la configuración de la aplicación.|
|eventGridKeySettingName ||El nombre de la configuración de aplicación que contiene la clave usada para autenticarse con el tema personalizado de Azure Event Grid en `EventGridTopicEndpoint`.|
|eventGridPublishRetryCount|0|El número de reintentos, si, al publicar en el tema de Event Grid, se produce un error.|
|eventGridPublishRetryInterval|5 minutos|Event Grid publica el intervalo de reintento en formato *hh:mm:ss*.|
|eventGridPublishEventTypes||Lista de tipos de eventos que se van a publicar en Event Grid. Si no se especifica, se publicarán todos los tipos de evento. Los valores permitidos son `Started`, `Completed`, `Failed` y `Terminated`.|
|useAppLease|true|Si se establece en `true`, las aplicaciones requerirán que se adquiera una concesión de blob de nivel de aplicación antes de procesar los mensajes de la central de tareas. Para más información, consulte la documentación sobre la [recuperación ante desastres y la distribución geográfica](../articles/azure-functions/durable/durable-functions-disaster-recovery-geo-distribution.md). Disponible a partir de la versión 2.3.0.
|useLegacyPartitionManagement|true|Cuando se establece en `false`, usa un algoritmo de administración de particiones que reduce la posibilidad de que se ejecute la función duplicada al realizar un escalado horizontal.  Disponible a partir de la versión 2.3.0. En una fase posterior, el valor predeterminado se cambiará a `false`.|
|useGracefulShutdown|false|(Versión preliminar) Habilite el apagado correcto para reducir la posibilidad de que se produzcan errores de apagado del host en las ejecuciones de función en proceso.|

Muchos de estos valores son para optimizar el rendimiento. Para obtener más información, vea [Rendimiento y escalabilidad](../articles/azure-functions/durable-functions-perf-and-scale.md).
