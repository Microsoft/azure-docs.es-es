---
title: 'Supervisión de Azure Database for MySQL: servidor flexible'
description: 'En este artículo se explican las métricas de supervisión y alerta de Azure Database for MySQL: servidor flexible, incluidas las estadísticas de CPU, almacenamiento y conexión.'
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: conceptual
ms.date: 9/21/2020
ms.openlocfilehash: f3a58a6766c42385dd6611c2014ba36fd1078710
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932263"
---
# <a name="monitor-azure-database-for-mysql-flexible-servers-with-built-in-metrics"></a>Supervisión de servidores flexibles de Azure Database for MySQL con métricas integradas

> [!IMPORTANT] 
> Azure Database for MySQL: servidor flexible está actualmente en versión preliminar pública.

Azure Database for MySQL: servidor flexible proporciona supervisión de servidores mediante Azure Monitor. Las métricas son valores numéricos que describen algún aspecto de los recursos del servidor en un momento dado. La supervisión de los recursos del servidor ayuda a solucionar problemas y a optimizar la carga de trabajo al permitir supervisar lo que más importa. La supervisión de las métricas adecuadas ayuda a mantener el rendimiento, la confiabilidad y la disponibilidad del servidor y las aplicaciones.

En este artículo se proporciona información sobre las distintas métricas disponibles para su servidor flexible que ofrecen datos sobre el comportamiento del servidor.

## <a name="available-metrics"></a>Métricas disponibles

Azure Database for MySQL: servidor flexible proporciona distintas métricas para comprender cómo rinde la carga de trabajo y, en función de estos datos, comprender el impacto en el servidor y la aplicación. Por ejemplo, en un servidor flexible, puede supervisar el **porcentaje de CPU de host**, las **conexiones activas**, el **porcentaje de E/S** y el **porcentaje de memoria de host** para identificar si hay impacto en el rendimiento. A partir de esos datos, es posible que tenga que optimizar la carga de trabajo, y escalarla verticalmente mediante el cambio de niveles de proceso u horizontalmente mediante réplica de lectura.

Todas las métricas de Azure tienen una frecuencia de un minuto y cada métrica proporciona 30 días de historial. Puede configurar alertas en las métricas. Para obtener instrucciones paso a paso, vea [Configuración de alertas](./how-to-alert-on-metric.md). Otras tareas incluyen la configuración de acciones automatizadas, la realización de análisis avanzados y el archivo del historial. Para obtener más información, consulte [Información general sobre las métricas en Microsoft Azure](../../monitoring-and-diagnostics/monitoring-overview-metrics.md).

### <a name="list-of-metrics"></a>Lista de métricas
Estas métricas están disponibles para Azure Database for MySQL:

|Nombre para mostrar de la métrica|Métrica|Unidad|Descripción|
|---|---|---|---|
|Porcentaje de CPU de host|cpu_percent|Percent|Se trata del porcentaje de uso de la CPU en el servidor, incluido el uso de CPU de la carga de trabajo cliente y los procesos de Azure MySQL.|
|Entrada de red de host |network_bytes_ingress|Bytes|Tráfico de red entrante en el servidor, incluido el tráfico de la base de datos cliente y características de Azure MySQL como la replicación, la supervisión, los registros, etc.|
|Salida de red de host|network_bytes_egress|Bytes|Tráfico de red saliente en el servidor, incluido el tráfico de la base de datos cliente y características de Azure MySQL como la replicación, la supervisión, los registros, etc.|
|Intervalo de replicación|replication_lag|Segundos|El tiempo transcurrido desde la última transacción reproducida. Esta métrica está disponible únicamente para los servidores de réplica.|
|Conexiones activas|active_connection|Count|Número de conexiones activas al servidor.|
|Almacenamiento de copia de seguridad utilizado|backup_storage_used|Bytes|Cantidad de almacenamiento de copia de seguridad utilizado.|
|Porcentaje de E/S|io_consumption_percent|Percent|Porcentaje de E/S en uso.|
|Porcentaje de memoria de host|memory_percent|Percent|Porcentaje de memoria en uso en el servidor, incluido el uso de memoria de la carga de trabajo cliente y los procesos de Azure MySQL.|
|Límite de almacenamiento|storage_limit|Bytes|Almacenamiento máximo de este servidor.|
|Porcentaje de almacenamiento|storage_percent|Percent|Porcentaje de almacenamiento que se usa más allá del límite máximo del servidor.|
|Almacenamiento usado|storage_used|Bytes|Cantidad de almacenamiento en uso. El almacenamiento que usa el servicio puede incluir los archivos de base de datos, los registros de transacciones y los registros de servidor.|
|Total de conexiones|total_connections|Count|Número de conexiones totales al servidor|
|Conexiones anuladas|aborted_connections|Count|Número de intentos fallidos de conexión a MySQL, por ejemplo, conexión errónea debido a credenciales no válidas.|
|Consultas|Consultas|Count|Número de consultas por segundo|

## <a name="next-steps"></a>Pasos siguientes
- Consulte [How to set up alerts](./how-to-alert-on-metric.md) (Configuración de alertas) para obtener instrucciones sobre cómo crear una alerta en una métrica.
- Obtenga más información sobre el [escalado de IOPS](./concepts/../concepts-compute-storage.md#iops) para mejorar el rendimiento.
