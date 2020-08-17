---
ms.topic: include
ms.service: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.date: 07/09/2020
ms.openlocfilehash: de5d3f8f32e928c77ffd6028ec764793ab7229ac
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86495351"
---
## <a name="business-disaster-recovery"></a>Recuperación ante desastres de la empresa

En esta sección se describen las características de Azure Time Series Insights que mantienen las aplicaciones y los servicios en ejecución, incluso si se produce un desastre (lo que se conoce como *recuperación ante desastres de la empresa*).

### <a name="high-availability"></a>Alta disponibilidad

Como servicio de Azure, Azure Time Series Insights proporciona determinadas características de *alta disponibilidad* mediante el uso de redundancias al nivel de región de Azure. Por ejemplo, Azure admite funcionalidades de recuperación ante desastres mediante la característica de *disponibilidad entre regiones* de Azure.

Entre las características de alta disponibilidad adicionales proporcionadas mediante Azure (también disponibles para cualquier instancia de Azure Time Series Insights), se encuentran las siguientes:

- **Conmutación por error**: Azure proporciona [replicación geográfica y equilibrio de carga](https://docs.microsoft.com/azure/architecture/resiliency/recovery-loss-azure-region).
- **Restauración de datos** y **recuperación del almacenamiento**: Azure proporciona [varias opciones para conservar y recuperar datos](https://docs.microsoft.com/azure/architecture/resiliency/recovery-data-corruption).
- **Azure Site Recovery**: Azure proporciona características de recuperación del sitio a través de [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/).
- **Azure Backup**: [Azure Backup ](https://docs.microsoft.com/azure/backup/backup-architecture) admite la copia de seguridad local y en la nube de las máquinas virtuales de Azure.

Asegúrese de habilitar las características de Azure pertinentes para proporcionar una alta disponibilidad global entre regiones para los dispositivos y usuarios.

> [!NOTE]
> Si Azure está configurado para habilitar la disponibilidad entre regiones, no es necesario establecer ninguna configuración adicional de disponibilidad entre regiones en Azure Time Series Insights.

### <a name="iot-and-event-hubs"></a>IoT y centro de eventos

Algunos servicios IoT de Azure también incluyen características de recuperación ante desastres de empresa integradas:

- [Recuperación ante desastres de alta disponibilidad de Azure IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-ha-dr), incluida la redundancia entre regiones
- [Directivas de Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/event-hubs-geo-dr)
- [Redundancia de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-redundancy)

La integración de Azure Time Series Insights con los demás servicios proporciona oportunidades adicionales de recuperación ante desastres. Por ejemplo, los datos de telemetría enviados al centro de eventos pueden conservarse en una copia de seguridad de la base de datos de Azure Blob Storage.

### <a name="azure-time-series-insights"></a>Azure Time Series Insights

Existen varias maneras de mantener los datos, las aplicaciones y los servicios de Azure Time Series Insights en ejecución, incluso si se ha interrumpido. 

Sin embargo, puede determinar que también se requiera una copia de seguridad completa del entorno de Azure Time Series, con los siguientes objetivos:

- Como una *instancia de conmutación por error* específicamente para que Azure Time Series Insights le redirija los datos y el tráfico
- Para conservar los datos y la información de auditoría

Por lo general, la mejor forma de duplicar un entorno de Azure Time Series Insights es crear un segundo entorno de Azure Time Series Insights en una región de Azure de copia de seguridad. Los eventos también se envían a este secundario desde el origen del evento principal. Asegúrese de usar un segundo grupo de consumidores dedicado. Siga las pautas de recuperación ante desastres de la empresa de ese origen, como se han descrito anteriormente.

Para crear un entorno duplicado:

1. Cree un entorno en una segunda región. Para más información, lea [Creación de un entorno de Azure Time Series Insights en Azure Portal](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-get-started).
1. Cree un segundo grupo de consumidores dedicado para el origen de eventos.
1. Conecte ese origen de eventos al nuevo entorno. Asegúrese de designar el segundo grupo de consumidores dedicado.
1. Revise la documentación de Azure Time Series Insights sobre [IoT Hub](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-how-to-add-an-event-source-iothub) y [Event Hubs](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-data-access).

Si se produce un evento:

1. Si la región primaria resulta afectada durante un incidente de desastre, vuelva a enrutar las operaciones al entorno de copia de Azure Time Series Insights de respaldo.
1. Use la segunda región para realizar copias de seguridad y recuperar todos los datos de telemetría y consulta de Azure Time Series Insights.

> [!IMPORTANT]
> Si se produce una conmutación por error:
> 
> * También puede producirse un retraso.
> * Podría producirse un pico momentáneo en el procesamiento de los mensajes dado que las operaciones se vuelven a enrutar.
> 
> Para más información, lea [Mitigación de la latencia en Azure Time Series Insights](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-environment-mitigate-latency).

