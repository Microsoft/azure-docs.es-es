---
title: archivo de inclusión
description: archivo de inclusión
ms.topic: include
ms.custom: include file
services: time-series-insights
ms.service: time-series-insights
author: kingdomofends
ms.author: adgera
ms.date: 04/29/2019
ms.openlocfilehash: e87a82e985ed1d1794f9da00546f167ef01e1779
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65815501"
---
## <a name="business-disaster-recovery"></a>Recuperación ante desastres de la empresa

Esta sección describen las características de Azure Time Series Insights que mantener las aplicaciones y servicios que se ejecutan aunque se produzca un desastre (**negocio recuperación ante desastres**).

### <a name="high-availability"></a>Alta disponibilidad

Como un servicio de Azure Time Series Insights proporciona ciertos **alta disponibilidad** características usando redundancias en el nivel de región de Azure. Por ejemplo, Microsoft Azure admite funcionalidades de recuperación ante desastres a través de Azure **disponibilidad entre regiones** característica.

Las características de alta disponibilidad adicionales proporcionadas a través de Azure (y también está disponible en cualquier instancia de Time Series Insights) incluyen:

1. **Conmutación por error**: Azure proporciona [equilibrio de carga y la replicación geográfica](https://docs.microsoft.com/azure/architecture/resiliency/recovery-loss-azure-region).
1. **Restauración de datos** y **recuperación almacenamiento**: Azure proporciona [varias opciones para conservar y recuperar datos](https://docs.microsoft.com/azure/architecture/resiliency/recovery-data-corruption).
1. **Site Recovery** características a través de [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/).

Asegúrese de habilitar estas características de Azure proporcionar global, entre regiones, alta disponibilidad para sus dispositivos y usuarios.

> [!NOTE]
> Si Azure está configurado para permitir **disponibilidad entre regiones**, hace falta ninguna configuración adicional disponibilidad entre regiones dentro de Azure Time Series Insights.

### <a name="iot-and-event-hubs"></a>IoT y Event hubs

Algunos servicios IoT de Azure también incluyen características de recuperación ante desastres de negocios integrados:

1. [Recuperación ante desastres de alta disponibilidad de IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-ha-dr) incluidos redundancia dentro de una región.
1. [Las directivas del centro de eventos](https://docs.microsoft.com/azure/event-hubs/event-hubs-geo-dr).
1. [Redundancia de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-redundancy).

Integración con estos otros servicios de Time Series Insights proporciona ante desastres más oportunidades de recuperación. Por ejemplo, los datos de telemetría enviados al centro de eventos pueden conservarse en una base de datos de Azure Blob Storage de copia de seguridad.

### <a name="time-series-insights"></a>Time Series Insights

Hay varias maneras de mantener sus datos de Time Series Insights, aplicaciones y servicios en ejecución incluso si le interrumpe. También puede determinar que se requiere una copia completa, duplicada, copia de seguridad de su entorno Azure Time Series:

1. Como una serie de tiempo específica Insights **instancia de conmutación por error** para redirigir los datos y al tráfico.
1. Para fines de conservación de la auditoría y los datos.

En general, la mejor manera para duplicar un entorno de Time Series Insights es crear un segundo entorno de Time Series Insights en una región de Azure de la copia de seguridad. Los eventos también se envían a este entorno secundario del origen de eventos principal. Asegúrese de usar un grupo de consumidores dedicado, segundo y seguir las directrices de recuperación ante desastres de ese origen negocio (proporcionadas anteriormente).

En concreto, para crear un entorno duplicado:

1. Crear un entorno en una segunda región ([crear un nuevo entorno de Time Series Insights en Azure portal](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-get-started)).
1. Cree un segundo grupo de consumidores dedicado para el origen de eventos.
1. Conectar ese origen de evento en el nuevo entorno asegurándose de designar el grupo de consumidores en segundo lugar, dedicado.
1. Revise el valor de la serie temporal [IoT Hub](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-how-to-add-an-event-source-iothub) y [centro de eventos](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-data-access) documentación.

Por último:

* Si la región primaria resulta afectada durante un incidente de desastre, vuelva a enrutar las operaciones al entorno de copia de Time Series Insights de respaldo.
* Use la segunda región para realizar copias de seguridad y recuperar todos los datos de telemetría y consulta de Time Series Insights.

> [!IMPORTANT]
> * Tenga en cuenta que puede experimentarse un retraso en caso de una conmutación por error.
> * Conmutación por error también podría producir un pico momentáneo en que se vuelven a enrutar las operaciones de procesamiento de mensajes.
> * Para más información, consulte [Mitigación de la latencia en Time Series Insights](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-environment-mitigate-latency).