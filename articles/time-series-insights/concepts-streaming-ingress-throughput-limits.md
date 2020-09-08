---
title: 'Limitaciones de rendimiento de ingesta de streaming: Azure Time Series Insights Gen2 | Microsoft Docs'
description: Obtenga información sobre los límites de rendimiento de entrada en Azure Time Series Insights Gen2.
author: lyrana
ms.author: lyhughes
manager: dpalled
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: seodec18
ms.openlocfilehash: a9ac55802e4bcc435bb4bd6fd4af8977db9fd293
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88950466"
---
# <a name="streaming-ingestion-throughput-limits"></a>Límites de rendimiento de ingesta de streaming

A continuación, se describen las limitaciones de entrada de datos de streaming de Azure Time Series Insights Gen2.

> [!TIP]
> Para obtener una lista completa de todos los límites, lea [Planeamiento del entorno de Azure Time Series Insights Gen2](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-update-plan#review-preview-limits).

## <a name="per-environment-limitations"></a>Limitaciones de cada entorno

En general, las tasas de entrada se ven como el factor del número de dispositivos que hay en la organización, la frecuencia de emisión de eventos y el tamaño de cada evento:

*  **Número de dispositivos** × **Frecuencia de emisión de eventos** × **Tamaño de cada evento**.

De forma predeterminada, Azure Time Series Insights Gen2 puede ingerir los datos entrantes a una velocidad de **hasta 1 megabyte por segundo (MBps) por entorno de Azure Time Series Insights Gen2**. Existen limitaciones adicionales [por partición de centro de conectividad](./concepts-streaming-ingress-throughput-limits.md#hub-partitions-and-per-partition-limits).

> [!TIP]
>
> * La compatibilidad del entorno con velocidades de ingesta de hasta 8 Mbps se puede proporcionar a petición.
> * Póngase en contacto con nosotros si necesita mayor rendimiento mediante el envío de una incidencia de soporte técnico a través de Azure Portal.
 
* **Ejemplo 1:**

    Contoso Shipping tiene 100.000 dispositivos que emiten un evento tres veces por minuto. El tamaño de un evento es de 200 bytes. Usan una instancia de IoT Hub con cuatro particiones como origen de eventos de Azure Time Series Insights Gen2.

    * La tasa de ingesta para su entorno de Azure Time Series Insights Gen2 sería: **100 000 dispositivos * 200 bytes/evento * (3/60 evento/segundo) = 1 MBps**.
    * Si se da por hecho que las particiones están equilibradas, la tasa de ingesta por partición sería de 0,25 Mbps.
    * La tasa de ingesta de Contoso Shipping estaría dentro de los límites de escala.

* **Ejemplo 2:**

    Contoso Fleet Analytics tiene 40 000 dispositivos que emiten un evento cada segundo. Se usa un centro de eventos con un recuento de particiones de 2 como origen del evento de Azure Time Series Insights Gen2. El tamaño de un evento es de 200 bytes.

    * La tasa de ingesta de entorno sería: **40 000 dispositivos * 200 bytes/evento * 1 evento/segundo = 8 Mbps**.
    * Si se da por hecho que las particiones están equilibradas, la tasa por partición sería de 4 Mbps.
    * La tasa de ingesta de análisis de Contoso Fleet Analytics supera los límites del entorno y de la partición. Es posible enviar una solicitud a Azure Time Series Insights Gen2 a través de Azure Portal para aumentar la tasa de ingesta del entorno y crear un centro de eventos con más particiones para estar dentro de los límites.

## <a name="hub-partitions-and-per-partition-limits"></a>Particiones del centro y límites por partición

Al planear el entorno de Azure Time Series Insights Gen2, es importante tener en cuenta la configuración de los orígenes de eventos que se van a conectar a Azure Time Series Insights Gen2. Tanto Azure IoT Hub como Event Hubs usan particiones para habilitar el escalado horizontal para el procesamiento de eventos. 

Una *partición* es una secuencia ordenada de eventos que se mantiene en un centro. El número de particiones se establece durante la fase de creación del centro y no se puede cambiar.

Para conocer los procedimientos recomendados de partición de Event Hubs, revise [¿Cuántas particiones necesito?](https://docs.microsoft.com/azure/event-hubs/event-hubs-faq#how-many-partitions-do-i-need)

> [!NOTE]
> La mayoría de las instancias de IoT Hub que se usan con Azure Time Series Insights Gen2 solo necesitan cuatro particiones.

Tanto si está creando un nuevo centro para el entorno de Azure Time Series Insights Gen2 como si usa uno existente, deberá calcular la tasa de ingesta por partición para determinar si se encuentra dentro de los límites. 

Azure Time Series Insights Gen2 tiene actualmente un **límite general por partición de 0,5 MBps**.

### <a name="iot-hub-specific-considerations"></a>Consideraciones específicas de IoT Hub

Cuando se crea un dispositivo en IoT Hub, se asigna de forma permanente a una partición. Al hacerlo, IoT Hub puede garantizar el orden de los eventos (puesto que la asignación nunca cambia).

Una asignación de partición fija también afecta a las instancias de Azure Time Series Insights Gen2 que ingieren datos enviados desde la instancia de IoT Hub de bajada. Cuando los mensajes de varios dispositivos se reenvían al centro con el mismo identificador de dispositivo de puerta de enlace, pueden llegar a la misma partición al mismo tiempo, lo que posiblemente haga superar los límites de escala por partición.

**Impacto**:

* Si una sola partición experimenta una tasa sostenida de ingesta sobre el límite, existe la posibilidad de que Azure Time Series Insights Gen2 no sincronice los datos de telemetría de todos los dispositivos antes de que se haya superado el período de retención de datos de IoT Hub. Como resultado, los datos enviados se pueden perder si los límites de ingesta se superan constantemente.

Para mitigar esta circunstancia, se recomiendan los siguientes procedimientos recomendados:

* Calcule sus tasas de ingesta por entorno y por partición antes de implementar la solución.
* Asegúrese de que los dispositivos IoT Hub tengan una carga equilibrada en la medida de lo posible.

> [!IMPORTANT]
> En el caso de los entornos que usan IoT Hub como origen de eventos, calcule la tasa de ingesta usando el número de dispositivos del centro en uso para asegurarse de que la velocidad esté por debajo de 0,5 MBps por límite de partición.
>
> * Aunque varios eventos lleguen simultáneamente, no se superará el límite.

  ![Diagrama de particiones de IoT Hub](media/concepts-ingress-overview/iot-hub-partiton-diagram.png)

Consulte los siguientes recursos para obtener más información sobre cómo optimizar el rendimiento y las particiones del centro:

* [Escala de IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-scaling)
* [Escala del centro de eventos](https://docs.microsoft.com/azure/event-hubs/event-hubs-scalability#throughput-units)
* [Particiones del centro de eventos](https://docs.microsoft.com/azure/event-hubs/event-hubs-features#partitions)

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre el [almacenamiento](./concepts-storage.md) de datos