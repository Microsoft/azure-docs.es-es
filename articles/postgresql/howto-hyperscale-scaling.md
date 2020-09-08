---
title: Escalado de un grupo de servidor de Hiperescala (Citus) en Azure Database for PostgreSQL
description: Ajuste la memoria del grupo de servidores, el disco y los recursos de CPU para tratar un aumento de la carga
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 8/10/2020
ms.openlocfilehash: 85a1f0dcc2e778a09cf0d19b2a85d6faf371f032
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88134534"
---
# <a name="server-group-size"></a>Tamaño del grupo de servidores

La opción de implementación de Hiperescala (Citus) usa servidores de bases de datos cooperativas para paralelizar la ejecución de consultas y almacenar más datos. El grupo de servidores "size" hace referencia al número de servidores y a los recursos de hardware de cada uno.

## <a name="picking-initial-size"></a>Selección del tamaño inicial

El tamaño de un grupo de servidores, en términos de número de nodos y su capacidad de hardware, es fácil de cambiar ([ver más abajo](#scale-a-hyperscale-citus-server-group)). Sin embargo, sigue siendo necesario elegir un tamaño inicial para un nuevo grupo de servidores. Estas son algunas sugerencias para una opción razonable.

### <a name="multi-tenant-saas-use-case"></a>Caso de uso de SaaS multiinquilino

Para aquellos que migran a Hiperescala (Citus) desde una instancia existente de base de datos de PostgreSQL de un solo nodo, se recomienda elegir un clúster en el que el número de núcleos virtuales de trabajo y la RAM total sean iguales a los de la instancia original. En estos escenarios se han detectado mejoras de rendimiento de dos a tres veces mayores porque el particionamiento mejora el uso de recursos, lo que permite, por ejemplo, índices más pequeños.

El número de núcleos virtuales necesarios para el nodo de coordinación depende de la carga de trabajo existente (rendimiento de escritura y lectura). El nodo de coordinación no requiere tanta RAM como los nodos de trabajo, pero la asignación de RAM se determina en función del número de núcleos virtuales (tal y como se describe en el artículo sobre [opciones de configuración de Hiperescala](concepts-hyperscale-configuration-options.md)) para que el número de núcleos virtuales sea en esencia la decisión real.

### <a name="real-time-analytics-use-case"></a>Caso de uso de análisis en tiempo real

Total de núcleos virtuales: cuando los datos de trabajo se ajustan a la RAM, se puede esperar una mejora de rendimiento lineal en Hiperescala (Citus) proporcional al número de núcleos de trabajo. Para determinar el número adecuado de núcleos virtuales para sus necesidades, tenga en cuenta la latencia actual de las consultas en la base de datos de un solo nodo y la latencia necesaria en Hiperescala (Citus). Divida la latencia actual según la latencia deseada y redondee el resultado.

RAM del trabajo: el mejor caso sería proporcionar suficiente memoria para que la mayor parte del espacio de trabajo se ajuste a la memoria. El tipo de consultas que utiliza la aplicación afecta a los requisitos de memoria. Puede ejecutar EXPLAIN ANALYZE en una consulta para determinar cuánta memoria se requiere. Recuerde que los núcleos virtuales y la RAM se escalan a la vez, tal y como se describe en el artículo sobre [opciones de configuración de Hiperescala](concepts-hyperscale-configuration-options.md).

## <a name="scale-a-hyperscale-citus-server-group"></a>Escalado de un grupo de servidores Hiperescala (Citus)

Azure Database for PostgreSQL:Hiperescala (Citus) proporciona escalado de autoservicio para tratar el aumento de la carga. Azure Portal permite agregar nuevos nodos de trabajo y aumentar los núcleos virtuales de los nodos existentes. Agregar nodos no provoca ningún tiempo de inactividad e incluso mover particiones a los nuevos nodos (llamado [reequilibrio de particiones](#rebalance-shards)) se produce sin interrumpir las consultas.

### <a name="add-worker-nodes"></a>Incorporación de nodos de trabajo

Para agregar nodos, vaya a la pestaña **Configurar** en el grupo de servidores de Hiperescala (Citus).  Arrastre el control deslizante de **Números de nodo de trabajo** para cambiar el valor.

![Controles deslizantes de recursos](./media/howto-hyperscale-scaling/01-sliders-workers.png)

Haga clic en el botón **Guardar** para que el valor cambiado surta efecto.

> [!NOTE]
> Una vez que se ha aumentado y guardado, el número de nodos de trabajo no se puede reducir con el control deslizante.

#### <a name="rebalance-shards"></a>Reequilibrio de particiones

Para aprovechar los nodos recién agregados, tiene que reequilibrar la tabla distribuida [particiones de base de datos](concepts-hyperscale-distributed-data.md#shards), lo que significa mover algunas particiones de los nodos existentes a las nuevas. En primer lugar, compruebe que los trabajos nuevos hayan finalizado correctamente el aprovisionamiento. Luego, inicie el reequilibrador de particiones conectándose al nodo coordinador de clústeres con psql y ejecute:

```sql
SELECT rebalance_table_shards('distributed_table_name');
```

La función `rebalance_table_shards` vuelve a equilibrar todas las tablas del grupo [colocación](concepts-hyperscale-colocation.md)de la tabla nombrada en su argumento. Por lo tanto, no tiene que llamar a la función para cada tabla distribuida, simplemente llámela en una tabla representativa de cada grupo de ubicación.

### <a name="increase-or-decrease-vcores-on-nodes"></a>Aumentar o disminuir los núcleos virtuales de los nodos

> [!NOTE]
> Esta funcionalidad actualmente está en su versión preliminar. Para solicitar un cambio en los núcleos virtuales de los nodos del grupo de servidores, [póngase en contacto con el servicio de soporte técnico de Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

Además de agregar nodos nuevos, puede aumentar las capacidades de los nodos existentes. Ajustar la capacidad de proceso hacia arriba y hacia abajo puede ser útil en los experimentos de rendimiento, así como en el caso de cambios a corto o largo plazo en las demandas de tráfico.

Para cambiar los núcleos virtuales de todos los nodos de trabajo, ajuste el control deslizante **Núcleos virtuales** en **Configuración (por nodo de trabajo)** . Los núcleos virtuales del nodo de coordinación se pueden ajustar de forma independiente. Ajuste el control deslizante **Núcleos virtuales** en **Configuración (nodo de coordinación)** .

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el grupo de servidores en [opciones de rendimiento](concepts-hyperscale-configuration-options.md).
