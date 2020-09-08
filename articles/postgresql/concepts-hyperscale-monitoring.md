---
title: 'Supervisión y ajuste de Azure Database for PostgreSQL: Hiperescala (Citus)'
description: 'En este artículo se describen las características de supervisión y ajuste de Azure Database for PostgreSQL: Hiperescala (Citus).'
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: dcce4485e00415f9caa706966cac1c936c1f15f6
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690415"
---
# <a name="monitor-and-tune-azure-database-for-postgresql---hyperscale-citus"></a>Supervisar y ajustar Azure Database for PostgreSQL: Hiperescala (Citus)

La supervisión de los datos sobre los servidores le permite solucionar problemas y optimizar la carga de trabajo. Hiperescala (Citus) proporciona diversas opciones de supervisión para ofrecer información detallada sobre el comportamiento de los nodos en un grupo de servidores.

## <a name="metrics"></a>Métricas

Hiperescala (Citus) proporciona métricas para cada nodo de un grupo de servidores. Las métricas proporcionan información detallada sobre el comportamiento de los recursos complementarios. Cada métrica se emite con una frecuencia de un minuto y tiene un historial de hasta 30 días.

Además de ver los gráficos de las métricas, puede configurar alertas. Para obtener instrucciones paso a paso, consulte [How to set up alerts](howto-hyperscale-alert-on-metric.md) (Configuración de alertas).  Otras tareas incluyen la configuración de acciones automatizadas, la ejecución de análisis avanzados y el archivo del historial. Para obtener más información, consulte [Información general sobre las métricas en Microsoft Azure](../monitoring-and-diagnostics/monitoring-overview-metrics.md).

### <a name="list-of-metrics"></a>Lista de métricas

Estas métricas están disponibles para los nodos de Hiperescala (Citus):

|Métrica|Nombre de métrica para mostrar|Unidad|Descripción|
|---|---|---|---|
|active_connections|Conexiones activas|Count|Número de conexiones activas al servidor.|
|cpu_percent|Porcentaje de CPU|Percent|Porcentaje de CPU en uso.|
|iops|E/S|Count|Consulte la [definición de IOPS](../virtual-machines/premium-storage-performance.md#iops) y el [rendimiento de hiperescala](concepts-hyperscale-configuration-options.md)|
|memory_percent|Porcentaje de memoria|Percent|Porcentaje de memoria en uso.|
|network_bytes_ingress|Red interna|Bytes|Red interna a través de conexiones activas.|
|network_bytes_egress|Red interna|Bytes|Red externa a través de conexiones activas.|
|storage_percent|Porcentaje de almacenamiento|Percent|Porcentaje de almacenamiento que se usa más allá del límite máximo del servidor.|
|storage_used|Almacenamiento utilizado|Bytes|Cantidad de almacenamiento en uso. El almacenamiento que usa el servicio puede incluir los archivos de base de datos, los registros de transacciones y los registros de servidor.|

Azure no proporciona métricas agregadas para el clúster en conjunto, pero las métricas de varios nodos se pueden colocar en el mismo gráfico.

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Configuración de alertas](howto-hyperscale-alert-on-metric.md) para obtener instrucciones sobre cómo crear una alerta en una métrica.
