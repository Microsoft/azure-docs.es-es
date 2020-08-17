---
title: Métricas comunes de escalado automático
description: Aprenda qué métricas se utilizan normalmente para el escalado automático de Cloud Services, Virtual Machines y Web Apps.
ms.topic: conceptual
ms.date: 12/6/2016
ms.subservice: autoscale
ms.openlocfilehash: e1a77fc1b40faca0a339c5e1aaceb71dec8de8bd
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327045"
---
# <a name="azure-monitor-autoscaling-common-metrics"></a>Métricas comunes de escalado automático de Azure Monitor

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

El escalado automático de Azure Monitor le permite escalar verticalmente y reducir horizontalmente el número de instancias en ejecución, basándose en los datos de telemetría (métricas). Este documento describe las métricas comunes que estaría interesado en usar. En Azure Portal, puede elegir la métrica de recurso por la que se va a escalar. Sin embargo, también puede elegir cualquier métrica de un recurso diferente por la que escalar.

La escalabilidad automática de Azure Monitor solo se aplica a [Virtual Machine Scale Sets](https://azure.microsoft.com/services/virtual-machine-scale-sets/), [Cloud Services](https://azure.microsoft.com/services/cloud-services/), [App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) y los [servicios de API Management](../../api-management/api-management-key-concepts.md). Otros servicios de Azure usan distintos métodos de escalado.

## <a name="compute-metrics-for-resource-manager-based-vms"></a>Cálculo de métricas de máquinas virtuales basadas en Resource Manager
De forma predeterminada, las máquinas virtuales basadas en Resource Manager y los conjuntos de escalado de máquinas virtuales emiten métricas básicas (de nivel de host). Además, al configurar la recopilación de datos de diagnóstico para máquinas virtuales y conjuntos de escalado de máquinas virtuales de Azure, la extensión de diagnóstico de Azure también emite contadores de rendimiento de SO invitado (lo que normalmente se conoce como "métricas de SO invitado").  Todas estas métricas se usan en reglas de escalado automático.

Puede usar la CLI, API o PoSH `Get MetricDefinitions` para ver las métricas disponibles para el recurso VMSS.

Si está utilizando conjuntos de escalado de máquinas virtuales y no ve una métrica concreta enumerada, entonces es probable que esté *deshabilitada* en la extensión Diagnostics.

Si una métrica concreta no se muestrea o se transfiere a la frecuencia que desea, puede actualizar la configuración de diagnóstico.

Si se cumple cualquiera de los casos anteriores, revise [Uso de PowerShell para habilitar Diagnósticos de Azure en una máquina virtual con Windows](../../virtual-machines/extensions/diagnostics-windows.md) sobre PowerShell para configurar y actualizar la extensión Diagnostics de máquina virtual de Azure a fin de habilitar la métrica. Ese artículo también incluye un archivo de configuración de diagnósticos de ejemplo.

### <a name="host-metrics-for-resource-manager-based-windows-and-linux-vms"></a>Métricas de host para máquinas virtuales Windows y Linux basadas en Resource Manager
Las siguientes métricas de nivel de host se emiten de forma predeterminada para máquinas virtuales y conjuntos de escalado de máquinas virtuales de Azure en instancias de Windows y Linux. Estas métricas describen la máquina virtual de Azure, pero se recopilan desde el host de dicha máquina en lugar de hacerlo a través de agente instalado en la máquina virtual invitada. Puede usar estas métricas en reglas de escalado automático.

- [Métricas de host para máquinas virtuales Windows y Linux basadas en Resource Manager](./metrics-supported.md#microsoftcomputevirtualmachines)
- [Métricas de host para conjuntos de escalado de máquinas virtuales Windows y Linux basadas en Resource Manager](./metrics-supported.md#microsoftcomputevirtualmachinescalesets)

### <a name="guest-os-metrics-for-resource-manager-based-windows-vms"></a>Métricas de SO invitado para máquinas virtuales Windows basadas en Resource Manager
Cuando crea una nueva máquina virtual en Azure, los diagnósticos se habilitan mediante la extensión Diagnósticos. La extensión de diagnósticos emite un conjunto de métricas realizadas desde dentro de la máquina virtual. Esto significa que puede escalar automáticamente fuera de las métricas que no se emiten de forma predeterminada.

Puede generar una lista de las métricas mediante el siguiente comando en PowerShell.

```
Get-AzMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

Puede crear una alerta para las siguientes métricas:

| Nombre de la métrica | Unidad |
| --- | --- |
| Procesador(_Total)\% Hora del procesador |Percent |
| \Procesador(_Total)\% de tiempo con privilegios |Percent |
| \Procesador(_Total)\% de tiempo de usuario |Percent |
| \Información del procesador(_Total)\Frecuencia del procesador |Count |
| \Sistema\Procesos |Count |
| \Proceso(_Total)\Número de subprocesos |Count |
| \Proceso(_Total)\Número de identificadores |Count |
| \Memoria\% de bytes confirmados en uso |Percent |
| \Memoria\Bytes disponibles |Bytes |
| \Memoria\Bytes confirmados |Bytes |
| \Memoria\Límite de confirmación |Bytes |
| \Memoria\Bytes de bloque paginado |Bytes |
| \Memoria\Bytes de bloque no paginado |Bytes |
| \Disco físico(_Total)\% de tiempo de disco |Percent |
| \Disco físico(_Total)\%Tiempo de lectura de disco |Percent |
| \Disco físico(_Total)\% de tiempo de escritura de disco |Percent |
| \Disco físico(_Total)\Transferencias de disco/s |CountPerSecond |
| \Disco físico(_Total)\Lecturas de disco/s |CountPerSecond |
| \Disco físico(_Total)\Escrituras de disco/s |CountPerSecond |
| \Disco físico(_Total)\Bytes de disco/s |BytesPerSecond |
| \Disco físico(_Total)\Bytes de lectura de disco/s |BytesPerSecond |
| \Disco físico(_Total)\Bytes de escritura de disco/s |BytesPerSecond |
| \Disco físico(_Total)\Promedio Longitud de la cola de disco |Count |
| \Disco físico(_Total)\Promedio Longitud de la cola de lectura de disco |Count |
| \Disco físico(_Total)\Promedio Longitud de la cola de escritura de disco |Count |
| \Disco lógico(_Total)\% de espacio disponible |Percent |
| \Disco lógico(_Total)\Megabytes disponibles |Count |

### <a name="guest-os-metrics-linux-vms"></a>Métricas de SO invitado para máquinas virtuales Linux
Cuando crea una máquina virtual en Azure, los diagnósticos se habilitan de forma predeterminada mediante la extensión Diagnósticos.

Puede generar una lista de las métricas mediante el siguiente comando en PowerShell.

```
Get-AzMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

 Puede crear una alerta para las siguientes métricas:

| Nombre de la métrica | Unidad |
| --- | --- |
| \Memory\AvailableMemory |Bytes |
| \Memory\PercentAvailableMemory |Percent |
| \Memory\UsedMemory |Bytes |
| \Memory\PercentUsedMemory |Percent |
| \Memory\PercentUsedByCache |Percent |
| \Memory\PagesPerSec |CountPerSecond |
| \Memory\PagesReadPerSec |CountPerSecond |
| \Memory\PagesWrittenPerSec |CountPerSecond |
| \Memory\AvailableSwap |Bytes |
| \Memory\PercentAvailableSwap |Percent |
| \Memory\UsedSwap |Bytes |
| \Memory\PercentUsedSwap |Percent |
| \Processor\PercentIdleTime |Percent |
| \Processor\PercentUserTime |Percent |
| \Processor\PercentNiceTime |Percent |
| \Processor\PercentPrivilegedTime |Percent |
| \Processor\PercentInterruptTime |Percent |
| \Processor\PercentDPCTime |Percent |
| \Processor\PercentProcessorTime |Percent |
| \Processor\PercentIOWaitTime |Percent |
| \PhysicalDisk\BytesPerSecond |BytesPerSecond |
| \PhysicalDisk\ReadBytesPerSecond |BytesPerSecond |
| \PhysicalDisk\WriteBytesPerSecond |BytesPerSecond |
| \PhysicalDisk\TransfersPerSecond |CountPerSecond |
| \PhysicalDisk\ReadsPerSecond |CountPerSecond |
| \PhysicalDisk\WritesPerSecond |CountPerSecond |
| \PhysicalDisk\AverageReadTime |Segundos |
| \PhysicalDisk\AverageWriteTime |Segundos |
| \PhysicalDisk\AverageTransferTime |Segundos |
| \PhysicalDisk\AverageDiskQueueLength |Count |
| \NetworkInterface\BytesTransmitted |Bytes |
| \NetworkInterface\BytesReceived |Bytes |
| \NetworkInterface\PacketsTransmitted |Count |
| \NetworkInterface\PacketsReceived |Count |
| \NetworkInterface\BytesTotal |Bytes |
| \NetworkInterface\TotalRxErrors |Count |
| \NetworkInterface\TotalTxErrors |Count |
| \NetworkInterface\TotalCollisions |Count |

## <a name="commonly-used-app-service-server-farm-metrics"></a>Métricas de App Service comúnmente usadas (granja de servidores)
También puede realizar el escalado automático basándose en métricas de servidor web comunes, como la longitud de cola Http. El nombre de la métrica es **HttpQueueLength**.  En la siguiente sección se muestran las métricas de granja de servidores disponibles (App Service).

### <a name="web-apps-metrics"></a>Métricas de Web Apps
Puede generar una lista de las métricas de Web Apps mediante el siguiente comando en PowerShell.

```
Get-AzMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

Puede alertar sobre estas métricas o escalar por las mismas.

| Nombre de la métrica | Unidad |
| --- | --- |
| CpuPercentage |Percent |
| MemoryPercentage |Percent |
| DiskQueueLength |Count |
| HttpQueueLength |Count |
| BytesReceived |Bytes |
| BytesSent |Bytes |

## <a name="commonly-used-storage-metrics"></a>Métricas de almacenamiento utilizadas comúnmente
Puede escalar por la longitud de la cola de almacenamiento, que es el número de mensajes de dicha cola. La longitud de cola de almacenamiento es una métrica especial y el umbral es el número de mensajes por instancia. Por ejemplo, si hay dos instancias y el umbral está establecido en 100, el escalado se produce cuando el número total de mensajes de la cola es 200. Esto puede ser 100 mensajes por instancia, 120 y 80, o cualquier combinación que sume hasta 200 o más.

Esta configuración la puede realizar en Azure Portal, en la hoja **Configuración**. Para los conjuntos de escalado de máquinas virtuales, puede actualizar la configuración de escalado automático en la plantilla de Resource Manager para usar *metricName* como *ApproximateMessageCount* y pasar el identificador de la cola de almacenamiento como *metricResourceUri*.

Por ejemplo, con una cuenta de almacenamiento clásico el parámetro de escalado automático metricTrigger incluiría lo siguiente:

```
"metricName": "ApproximateMessageCount",
"metricNamespace": "",
"metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.ClassicStorage/storageAccounts/STORAGE_ACCOUNT_NAME/services/queue/queues/QUEUE_NAME"
 ```

En el caso de una cuenta de almacenamiento (no clásica) el metricTrigger incluiría lo siguiente:

```
"metricName": "ApproximateMessageCount",
"metricNamespace": "",
"metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.Storage/storageAccounts/STORAGE_ACCOUNT_NAME/services/queue/queues/QUEUE_NAME"
```

## <a name="commonly-used-service-bus-metrics"></a>Métricas más usadas de Service Bus
Puede escalar por la longitud de la cola de Service Bus, que es el número de mensajes de dicha cola. La longitud de cola de Service Bus es una métrica especial y el umbral es el número de mensajes por instancia. Por ejemplo, si hay dos instancias y el umbral está establecido en 100, el escalado se produce cuando el número total de mensajes de la cola es 200. Esto puede ser 100 mensajes por instancia, 120 y 80, o cualquier combinación que sume hasta 200 o más.

Para los conjuntos de escalado de máquinas virtuales, puede actualizar la configuración de escalado automático en la plantilla de Resource Manager para usar *metricName* como *ApproximateMessageCount* y pasar el identificador de la cola de almacenamiento como *metricResourceUri*.

```
"metricName": "ApproximateMessageCount",
 "metricNamespace": "",
"metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.ServiceBus/namespaces/SB_NAMESPACE/queues/QUEUE_NAME"
```

> [!NOTE]
> Para Service Bus, el concepto de grupo de recursos no existe pero Azure Resource Manager crea un grupo de recursos predeterminado por región. El grupo de recursos suele tener el formato 'Default-ServiceBus-[región]'. Por ejemplo, 'Default-ServiceBus-EastUS', 'Default-ServiceBus-WestUS', 'Default-ServiceBus-AustraliaEast', etc.
>
>

