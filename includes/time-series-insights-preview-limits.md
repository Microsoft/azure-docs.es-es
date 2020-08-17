---
title: archivo de inclusión
description: archivo de inclusión
services: digital-twins
ms.service: digital-twins
ms.topic: include
ms.date: 07/09/2020
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.custom: include file
ms.openlocfilehash: 48080bb4d1e24f7f98d3dfe1fd63b65ba46df35e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289908"
---
### <a name="property-limits"></a>Límites de propiedad

Los límites de propiedad de Azure Time Series Insights han aumentado a 1000 desde un límite máximo de 800 en Gen1. Las propiedades de evento proporcionadas tienen columnas JSON, CSV y de gráfico correspondientes que puede ver en el [explorador de Azure Time Series Insights Gen2](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-update-quickstart).

| SKU | Propiedades máximas |
| --- | --- |
| Gen2 (L1) | Propiedades de 1000 (columnas) |
| Gen1 (S1) | 600 propiedades (columnas) |
| Gen1 (S2) | 800 propiedades (columnas) |

### <a name="event-sources"></a>Orígenes de eventos

Se admite un máximo de dos orígenes de evento por instancia.

* Obtenga información sobre cómo [agregar un origen de Event Hub](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-how-to-add-an-event-source-eventhub).
* Configure [un origen de centro de IoT](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-how-to-add-an-event-source-iothub).

De forma predeterminada, los [entornos Gen2 admiten tasas de entrada](https://docs.microsoft.com/azure/time-series-insights/concepts-streaming-ingress-throughput-limits) de hasta **1 megabyte por segundo (MB/s) por entorno**. Los clientes pueden escalar sus entornos hasta un rendimiento de **16 MB/s** si es necesario. Además, hay un límite por partición de **0,5 MB/s**.

### <a name="api-limits"></a>Límites de API

Los límites de la API de REST para Azure Time Series Insights Gen2 están especificadas en la [documentación de referencia de la API de REST](https://docs.microsoft.com/rest/api/time-series-insights/preview#limits-1).
