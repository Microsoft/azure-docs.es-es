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
ms.openlocfilehash: 1aff5780aa5f4cf21d862a5cee3b140179dac03c
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88703470"
---
A continuación, se resumen los límites clave en Azure Time Series Insights Gen1.

### <a name="sku-ingress-rates-and-capacities"></a>Capacidades y tasas de entrada de SKU

Las capacidades y tasas de entrada de las SKU S1 y S2 proporcionan flexibilidad al configurar un nuevo entorno de Azure Time Series Insights. La capacidad de la SKU indica la tasa de entrada diaria en función del número de eventos o bytes almacenados, lo que suceda primero. Tenga en cuenta que la entrada se mide *por minuto* y la **limitación** se aplica mediante el algoritmo de cubo de tokens. La entrada se mide en bloques de 1 KB. Por ejemplo, un evento real de 0,8 KB se mediría como un evento y un evento de 2,6 KB se cuenta como tres eventos.

| Capacidad de SKU de S1 | Velocidad de entrada | Capacidad máxima de almacenamiento
| --- | --- | --- |
| 1 | 1 GB (1 millón de eventos) al día | 30 GB (30 millones de eventos) al mes |
| 10 | 10 GB (10 millones de eventos) al día | 300 GB (300 millones de eventos) al mes |

| Capacidad de SKU de S2 | Velocidad de entrada | Capacidad máxima de almacenamiento
| --- | --- | --- |
| 1 | 10 GB (10 millones de eventos) al día | 300 GB (300 millones de eventos) al mes |
| 10 | 100 GB (100 millones de eventos) al día | 3 TB (3 mil millones de eventos) al mes |

> [!NOTE]
> Las capacidades se escalan linealmente, por lo que una SKU de S1 con capacidad 2 admite una velocidad de entrada de 2 GB (2 millones) de eventos al día y 60 GB (60 millones de eventos) al mes.

Los entornos de SKU de S2 admiten muchos más eventos al mes y tienen una capacidad de entrada mucho mayor.

| SKU  | Recuento de eventos cada mes  | Recuento de eventos cada minuto | Tamaño de eventos cada minuto  |
|---------|---------|---------|---------|---------|
| S1     |   30 millones   |  720    |  720 KB   |
 |S2     |   300 millones   | 7200   | 7200 KB  |

### <a name="property-limits"></a>Límites de propiedad

Los límites de propiedad de Gen1 dependen del entorno de SKU seleccionado. Las propiedades de evento suministradas tienen las columnas JSON, CSV y de gráfico correspondientes que se pueden ver en el [Explorador de Azure Time Series Insights](https://docs.microsoft.com/azure/time-series-insights/time-series-quickstart).

| SKU | Propiedades máximas |
| --- | --- |
| S1 | 600 propiedades (columnas) |
| S2 | 800 propiedades (columnas) |

### <a name="event-sources"></a>Orígenes de eventos

Se admite un máximo de dos orígenes de evento por instancia.

* Obtenga información sobre cómo [agregar un origen de Event Hub](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-how-to-add-an-event-source-eventhub).
* Configure [un origen de centro de IoT](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-how-to-add-an-event-source-iothub).

### <a name="api-limits"></a>Límites de API

Los límites de la API REST de Azure Time Series Insights Gen1 se especifican en la [documentación de referencia de la API REST](https://docs.microsoft.com/rest/api/time-series-insights/dataaccess(preview)/query/getavailability).