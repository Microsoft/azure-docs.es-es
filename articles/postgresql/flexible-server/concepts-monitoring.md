---
title: 'Supervisión y métricas: Servidor flexible de Azure Database for PostgreSQL'
description: En este artículo se describen las características de supervisión y métricas del Servidor flexible de Azure Database for PostgreSQL.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 7d1d7f9759fc8e2826da6232981eb3a389576c07
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931891"
---
# <a name="monitor-metrics-azure-database-for-postgresql---flexible-server"></a>Métricas de supervisión: Servidor flexible de Azure Database for PostgreSQL

> [!IMPORTANT]
> Servidor flexible de Azure Database for PostgreSQL está en versión preliminar

La supervisión de los datos sobre los servidores le permite solucionar problemas y optimizar la carga de trabajo. Azure Database for PostgreSQL proporciona diversas opciones de supervisión para ofrecer información detallada sobre el comportamiento del servidor.

## <a name="metrics"></a>Métricas
Azure Database for PostgreSQL proporciona diversas métricas que proporcionan información sobre el comportamiento de los recursos que admiten el servidor PostgreSQL. Cada métrica se emite con una frecuencia de un minuto y tiene un [historial de hasta 93 días](../../azure-monitor/platform/data-platform-metrics.md#retention-of-metrics). Puede configurar alertas en las métricas. Otras opciones incluyen la configuración de acciones automatizadas, la realización de análisis avanzados y el archivado del historial. Para obtener más información, consulte [Información general sobre las métricas en Microsoft Azure](../../azure-monitor/platform/data-platform-metrics.md).

### <a name="list-of-metrics"></a>Lista de métricas
Las métricas siguientes están disponibles para el servidor flexible de PostgreSQL:


|Métrica|Nombre de métrica para mostrar|Unidad|Descripción|
|---|---|---|---|
| backup_storage_used | Almacenamiento de copia de seguridad utilizado | Bytes | Cantidad de almacenamiento de copia de seguridad utilizado. Esta métrica representa la suma del almacenamiento consumido por todas las copias de seguridad de base de datos completas, copias de seguridad diferenciales y copias de seguridad de registros, conservadas según el período de retención de copia de seguridad establecido para el servidor. El servicio administra la frecuencia de las copias de seguridad. En el caso del almacenamiento con redundancia geográfica, el uso del almacenamiento de copia de seguridad es dos veces el del almacenamiento con redundancia local. |
| connections_failed | Conexiones con errores | Count | Conexiones con errores. |
| connections_succeeded | Conexiones correctas | Count | Conexiones correctas. |
| cpu_credits_consumed | Créditos de CPU consumidos | Count | Número de créditos utilizados por el servidor flexible. Aplicable al nivel Flexible. |
| cpu_credits_remaining | Créditos de CPU restantes | Count | Número de créditos disponibles para la ráfaga. Aplicable al nivel Flexible. |
| cpu_percent | Porcentaje de CPU | Percent | Porcentaje de CPU en uso. | 
| disk_queue_depth | Profundidad de la cola de discos | Count | Número de operaciones de E/S pendiente en el disco de datos. |
| iops | E/S | Count | Número de operaciones de E/S en disco por segundo. |
| maximum_used_transactionIDs | Máximo de identificadores de transacción usados | Count | Id. de transacción máximo en uso. |
| memory_percent | Porcentaje de memoria | Percent | Porcentaje de memoria en uso. |
| network_bytes_egress | Red interna | Bytes | Cantidad de tráfico de red saliente. |
| network_bytes_ingress | Red interna | Bytes | Cantidad de tráfico de red entrante. |
| read_iops | IOPS de lectura | Count | Número de operaciones de lectura de E/S de disco de datos por segundo. |
| read_throughput | Rendimiento de lectura | Bytes | Bytes leídos por segundo desde el disco. |
| storage_free | Almacenamiento libre | Bytes | Cantidad de espacio de almacenamiento en disco disponible. |
| storage_percent | Porcentaje de almacenamiento | Porcentaje | Porcentaje de espacio de almacenamiento utilizado. El almacenamiento que usa el servicio puede incluir los archivos de base de datos, los registros de transacciones y los registros de servidor.|
| storage_used | Almacenamiento utilizado | Bytes | Porcentaje de espacio de almacenamiento utilizado. El almacenamiento que usa el servicio puede incluir los archivos de base de datos, los registros de transacciones y los registros de servidor. |
| txlogs_storage_used | Almacenamiento del registro de transacciones usado | Bytes | Cantidad de espacio de almacenamiento utilizado por los registros de transacciones. | 
| write_throughput | Rendimiento de escritura | Bytes | Bytes escritos por segundo en disco. |
| write_iops | IOPS de escritura | Count | Número de operaciones de escritura de E/S de disco de datos por segundo. |

## <a name="server-logs"></a>Registros del servidor
Azure Database for PostgreSQL permite configurar y acceder a los registros estándar de Postgres. Para obtener más información sobre los registros, visite la [documentación de conceptos de registro](concepts-logging.md).
