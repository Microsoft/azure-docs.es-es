---
title: Límites de memoria y simultaneidad
description: Vea los límites de simultaneidad y memoria asignados a los distintos niveles de rendimiento y las clases de recursos en Azure Synapse Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 5b72694f93ed5b712a0f684887df5b69a7b35c72
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89441687"
---
# <a name="memory-and-concurrency-limits-for-azure-synapse-analytics"></a>Límites de memoria y simultaneidad de Azure Synapse Analytics

Vea los límites de simultaneidad y memoria asignados a los distintos niveles de rendimiento y las clases de recursos en Azure Synapse Analytics.  

## <a name="data-warehouse-capacity-settings"></a>Configuración de la capacidad del almacenamiento de datos

Las siguientes tablas muestran la capacidad máxima para el almacenamiento de datos en diferentes niveles de rendimiento. Para cambiar el nivel de rendimiento, consulte [Escalabilidad horizontal de un proceso en Azure SQL Data Warehouse en Azure Portal](quickstart-scale-compute-portal.md).

### <a name="service-levels"></a>Niveles de servicio

Los niveles de servicio van de DW100c a DW30000c.

| Nivel de rendimiento | Nodos de proceso | Distribuciones por nodo de proceso | Memoria por almacenamiento de datos (GB) |
|:-----------------:|:-------------:|:------------------------------:|:------------------------------:|
| DW100c            | 1             | 60                             |    60                          |
| DW200c            | 1             | 60                             |   120                          |
| DW300c            | 1             | 60                             |   180                          |
| DW400c            | 1             | 60                             |   240                          |
| DW500c            | 1             | 60                             |   300                          |
| DW1000c           | 2             | 30                             |   600                          |
| DW1500c           | 3             | 20                             |   900                          |
| DW2000c           | 4             | 15                             |  1200                          |
| DW2500c           | 5             | 12                             |  1\.500                          |
| DW3000c           | 6             | 10                             |  1800                          |
| DW5000c           | 10            | 6                              |  3000                          |
| DW6000c           | 12            | 5                              |  3600                          |
| DW7500c           | 15            | 4                              |  4500                          |
| DW10000c          | 20            | 3                              |  6000                          |
| DW15000c          | 30            | 2                              |  9000                          |
| DW30000c          | 60            | 1                              | 18000                          |

El nivel de servicio máximo es DW30000c, que tiene 60 nodos de proceso y una distribución por nodo de proceso. Por ejemplo, un almacenamiento de datos de 600 TB con DW30000c procesa, aproximadamente, 10 TB por nodo de proceso.

## <a name="concurrency-maximums-for-workload-groups"></a>Valores máximos de simultaneidad para grupos de cargas de trabajo

Con la introducción de los [grupos de cargas de trabajo](sql-data-warehouse-workload-isolation.md), ya no se aplica el concepto de espacios de simultaneidad.  Los recursos por solicitud se asignan de forma porcentual y se especifican en la definición del grupo de cargas de trabajo.  Sin embargo, incluso con la eliminación de los espacios de simultaneidad, se necesita una cantidad mínima de recursos por consultas basada en el nivel de servicio.  En la tabla siguiente se han definido la cantidad mínima de recursos necesarios por consulta en los niveles de servicio y la simultaneidad asociada que se puede lograr.

|Nivel de servicio|Número máximo de consultas concurrentes|% mínimo admitido para REQUEST_MIN_RESOURCE_GRANT_PERCENT|
|---|---|---|
|DW100c|4|25 %|
|DW200c|8|12,5 %|
|DW300c|12|8 %|
|DW400c|16|6,25 %|
|DW500c|20|5 %|
|DW1000c|32|3 %|
|DW1500c|32|3 %|
|DW2000c|48|2 %|
|DW2500c|48|2 %|
|DW3000c|64|1,5 %|
|DW5000c|64|1,5 %|
|DW6000c|128|0,75 %|
|DW7500c|128|0,75 %|
|DW10000c|128|0,75 %|
|DW15000c|128|0,75 %|
|DW30000c|128|0,75 %|
||||

## <a name="concurrency-maximums-for-resource-classes"></a>Valores máximos de simultaneidad para clases de recursos

Para asegurarse de que cada consulta tenga recursos suficientes para ejecutarse de forma eficaz, Synapse SQL realiza un seguimiento del uso de los recursos. Para ello, asigna espacios de simultaneidad a cada consulta. El sistema coloca las consultas en una cola en función de la importancia y las ranuras de simultaneidad. Las consultas esperan en la cola hasta que haya suficientes ranuras de simultaneidad disponibles. La [importancia](sql-data-warehouse-workload-importance.md) y las ranuras de simultaneidad determinan la asignación de prioridades de CPU. Para obtener más información, consulte [Analyze your workload](analyze-your-workload.md) (Análisis de la carga de trabajo).

**Clases de recursos estáticos**

En la tabla siguiente se muestra el número máximo de consultas simultáneas y espacios de simultaneidad para cada [clase de recurso estático](resource-classes-for-workload-management.md).  

| Nivel de servicio | Número máximo de consultas concurrentes | Espacios de simultaneidad disponibles | Ranuras utilizadas por staticrc10 | Ranuras utilizadas por staticrc20 | Ranuras utilizadas por staticrc30 | Ranuras utilizadas por staticrc40 | Ranuras utilizadas por staticrc50 | Ranuras utilizadas por staticrc60 | Ranuras utilizadas por staticrc70 | Ranuras utilizadas por staticrc80 |
|:-------------:|:--------------------------:|:---------------------------:|:---------:|:----------:|:----------:|:----------:|:----------:|:----------:|:----------:|:----------:|
| DW100c        |  4                         |    4                        | 1         | 2          | 4          | 4          | 4         |  4         |  4         |  4         |
| DW200c        |  8                         |    8                        | 1         | 2          | 4          | 8          |  8         |  8         |  8         |  8        |
| DW300c        | 12                         |   12                        | 1         | 2          | 4          | 8          |  8         |  8         |  8         |   8        |
| DW400c        | 16                         |   16                        | 1         | 2          | 4          | 8          | 16         | 16         | 16         |  16        |
| DW500c        | 20                         |   20                        | 1         | 2          | 4          | 8          | 16         | 16         | 16         |  16        |
| DW1000c       | 32                         |   40                        | 1         | 2          | 4          | 8          | 16         | 32         | 32         |  32        |
| DW1500c       | 32                         |   60                        | 1         | 2          | 4          | 8          | 16         | 32         | 32         |  32        |
| DW2000c       | 48                         |   80                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         |  64        |
| DW2500c       | 48                         |  100                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         |  64        |
| DW3000c       | 64                         |  120                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         |  64        |
| DW5000c       | 64                         |  200                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |
| DW6000c       | 128                        |  240                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |
| DW7500c       | 128                        |  300                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |
| DW10000c      | 128                        |  400                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |
| DW15000c      | 128                        |  600                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |
| DW30000c      | 128                        | 1200                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |

**Clases de recursos dinámicos**

En la tabla siguiente se muestra el número máximo de consultas simultáneas y espacios de simultaneidad para cada [clase de recurso dinámico](resource-classes-for-workload-management.md). Las clases de recurso dinámico usan una asignación de porcentaje de memoria de 3-10-22-70 para las clases de recursos small-medium-large-xlarge en todos los niveles de servicio.

| Nivel de servicio | Número máximo de consultas concurrentes | Espacios de simultaneidad disponibles | Ranuras utilizadas por smallrc | Ranuras utilizadas por mediumrc | Ranuras utilizadas por largerc | Ranuras utilizadas por xlargerc |
|:-------------:|:--------------------------:|:---------------------------:|:---------------------:|:----------------------:|:---------------------:|:----------------------:|
| DW100c        |  4                         |    4                        | 1                     |  1                     |  1                    |   2                    |
| DW200c        |  8                         |    8                        | 1                     |  1                     |  1                    |   5                    |
| DW300c        | 12                         |   12                        | 1                     |  1                     |  2                    |   8                    |
| DW400c        | 16                         |   16                        | 1                     |  1                     |  3                    |  11                    |
| DW500c        | 20                         |   20                        | 1                     |  2                     |  4                    |  14                    |
| DW1000c       | 32                         |   40                        | 1                     |  4                     |  8                    |  28                    |
| DW1500c       | 32                         |   60                        | 1                     |  6                     |  13                   |  42                    |
| DW2000c       | 32                         |   80                        | 2                     |  8                     |  17                   |  56                    |
| DW2500c       | 32                         |  100                        | 3                     | 10                     |  22                   |  70                    |
| DW3000c       | 32                         |  120                        | 3                     | 12                     |  26                   |  84                    |
| DW5000c       | 32                         |  200                        | 6                     | 20                     |  44                   | 140                    |
| DW6000c       | 32                         |  240                        | 7                     | 24                     |  52                   | 168                    |
| DW7500c       | 32                         |  300                        | 9                     | 30                     |  66                   | 210                    |
| DW10000c      | 32                         |  400                        | 12                    | 40                     |  88                   | 280                    |
| DW15000c      | 32                         |  600                        | 18                    | 60                     | 132                   | 420                    |
| DW30000c      | 32                         | 1200                        | 36                    | 120                    | 264                   | 840                    |

Cuando no hay suficientes espacios de simultaneidad libres para iniciar la ejecución de consulta, las consultas se ponen en cola y se ejecutan en función de su importancia.  En caso de una importancia equivalente, las consultas se ejecutan según el criterio primero en entrar, primero en salir.  A medida que las consultas finalizan y el número de consultas y espacios desciende por debajo de los límites, Azure Synapse Analytics libera las consultas en cola.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo aprovechar las clases de recursos para optimizar aún más la carga de trabajo, revise los artículos siguientes:

* [Resource classes for workload management](resource-classes-for-workload-management.md) (Clases de recursos para la administración de cargas de trabajo)
* [Analyzing your workload](analyze-your-workload.md) (Análisis de la carga de trabajo)
