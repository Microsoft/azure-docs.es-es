---
title: 'Límites en Azure Database for PostgreSQL: servidor flexible'
description: En este artículo, se describen los límites de Azure Database for PostgreSQL con un servidor flexible, como el número de opciones del motor de almacenamiento y de conexión.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 30c2da4ac750375c66b92cdca552e1a51a8dbc40
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932476"
---
# <a name="limits-in-azure-database-for-postgresql---flexible-server"></a>Límites en Azure Database for PostgreSQL con servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: servidor flexible está en versión preliminar

En las secciones siguientes se describen los límites de capacidad y funcionales en el servicio de base de datos. Para obtener más información sobre los niveles de recursos (proceso, memoria y almacenamiento), consulte el artículo acerca del [proceso y el almacenamiento](concepts-compute-storage.md).

## <a name="maximum-connections"></a>Número máximo de conexiones

A continuación se muestran el número máximo de conexiones por plan de tarifa y los núcleos virtuales. El sistema de Azure requiere tres conexiones para supervisar Azure Database for PostgreSQL con servidor flexible.

| Nombre de SKU             | Núcleos virtuales | Tamaño de memoria | Conexiones máximas | Número máximo de conexiones de usuario |
|----------------------|--------|-------------|-----------------|----------------------|
| **Flexible**        |        |             |                 |                      |
| B1ms                 | 1      | 2 GiB       | 50              | 47                   |
| B2s                  | 2      | 4 GiB       | 100             | 97                   |
| **Uso general**  |        |             |                 |                      |
| D2s_v3               | 2      | 8 GiB       | 214             | 211                  |
| D4s_v3               | 4      | 16 GiB      | 429             | 426                  |
| D8s_v3               | 8      | 32 GiB      | 859             | 856                  |
| D16s_v3              | 16     | 64 GiB      | 1718            | 1715                 |
| D32s_v3              | 32     | 128 GB     | 3437            | 3434                 |
| D48s_v3              | 48     | 192 GiB     | 5000            | 4997                 |
| D64s_v3              | 64     | 256 GiB     | 5000            | 4997                 |
| **Memoria optimizada** |        |             |                 |                      |
| E2s_v3               | 2      | 16 GiB      | 1718            | 1715                 |
| E4s_v3               | 4      | 32 GiB      | 3437            | 3434                 |
| E8s_v3               | 8      | 64 GiB      | 5000            | 4997                 |
| E16s_v3              | 16     | 128 GB     | 5000            | 4997                 |
| E32s_v3              | 32     | 256 GiB     | 5000            | 4997                 |
| E48s_v3              | 48     | 384 GiB     | 5000            | 4997                 |
| E64s_v3              | 64     | 432 GiB     | 5000            | 4997                 |

Si las conexiones superan el límite, puede que reciba el error siguiente:
> FATAL:  sorry, too many clients already

> [!IMPORTANT]
> Para obtener la mejor experiencia posible, se recomienda usar un agrupador de conexiones, como PgBouncer, para administrar las conexiones de forma eficaz.

Una conexión de PostgreSQL, aunque no esté activa, puede ocupar aproximadamente 10 MB de memoria. Además, la creación de conexiones lleva su tiempo. La mayoría de las aplicaciones solicitan muchas conexiones de corta duración, y esto es lo que conforma esta situación. El resultado es que hay menos recursos disponibles para la carga de trabajo real, lo que baja el rendimiento. Puede usarse un agrupador de conexiones para reducir las conexiones inactivas y reutilizar las existentes. Para más información, visite nuestra [entrada de blog](https://techcommunity.microsoft.com/t5/azure-database-for-postgresql/not-all-postgres-connection-pooling-is-equal/ba-p/825717).

## <a name="functional-limitations"></a>Limitaciones funcionales

### <a name="scale-operations"></a>Operaciones de escalado

- El escalado del almacenamiento del servidor requiere un reinicio del servidor.
- El almacenamiento del servidor solo se puede escalar en incrementos de 2x; consulte [Proceso y almacenamiento](concepts-compute-storage.md) para obtener más información.
- La reducción del tamaño de almacenamiento del servidor no se admite actualmente.

### <a name="server-version-upgrades"></a>Actualizaciones de la versión de servidor

- La migración automatizada entre versiones principales del motor de base de datos no se admite en este momento. Si quiere actualizar a la siguiente versión principal, realice un [volcado y restáurelo](../howto-migrate-using-dump-and-restore.md) a un servidor que se haya creado con la nueva versión del motor.

### <a name="networking"></a>Redes

- Actualmente no se admite ni la entrada ni la salida de la red virtual.
- Actualmente no se admite la combinación de acceso público con la implementación en una red virtual.
- No se admiten las reglas de firewall en la red virtual; en su lugar, se pueden usar grupos de seguridad de red.
- Los servidores de bases de datos de acceso público pueden conectarse a Internet público, por ejemplo, a través de `postgres_fdw`, y este acceso no se puede restringir. A los servidores basados en la red virtual se les puede restringir el acceso de salida mediante grupos de seguridad de red.

### <a name="high-availability"></a>Alta disponibilidad

- La alta disponibilidad con redundancia de zona no se admite actualmente en los servidores que pueden aumentar la velocidad.
- La dirección IP del servidor de bases de datos cambia cuando el servidor conmuta por error al modo de espera de alta disponibilidad. Use el registro DNS en lugar de la dirección IP del servidor.

### <a name="availability-zones"></a>Zonas de disponibilidad

- Actualmente no se admite el traslado manual de servidores a una zona de disponibilidad diferente.
- La zona de disponibilidad del servidor en espera de alta disponibilidad no se puede configurar manualmente.

### <a name="postgres-engine-extensions-and-pgbouncer"></a>Motor de postgres, extensiones y PgBouncer

- Postgres 10 y las versiones anteriores no se admiten. Se recomienda usar la opción [Servidor único](../overview-single-server.md) si necesita versiones anteriores de Postgres.
- La compatibilidad con la extensión está limitada actualmente a las extensiones `contrib` de Postgres.
- El agrupador de conexiones PgBouncer integrado no está disponible actualmente para los servidores de bases de datos de una red virtual ni para los servidores que pueden aumentar la velocidad.

### <a name="stopstart-operation"></a>Parada o puesta en marcha

- No se puede detener el servidor durante más de siete días.

### <a name="scheduled-maintenance"></a>Mantenimiento programado

- El cambio de la ventana de mantenimiento en menos de cinco días antes de una actualización ya planeada no afectará a esa actualización. Los cambios solo surten efecto con el siguiente mantenimiento programado.

### <a name="backing-up-a-server"></a>Copia de seguridad de un servidor

- El sistema administra las copias de seguridad, actualmente no hay ninguna manera de ejecutar estas copias de seguridad manualmente. Se recomienda usar `pg_dump` en su lugar.
- Las copias de seguridad son siempre copias de seguridad completas basadas en instantáneas (no copias de seguridad diferenciales), lo que podría provocar una mayor utilización del espacio de almacenamiento de copias de seguridad. Tenga en cuenta que los registros de transacciones (registros de escritura previa-WAL) son independientes de las copias de seguridad completas o diferenciales y que se almacenan continuamente.

### <a name="restoring-a-server"></a>Restauración de un servidor

- Al usar la función de restauración a un momento dado, se crea un servidor con las mismas configuraciones de proceso y almacenamiento que el servidor en el que se basa.
- Los servidores de bases de datos basados en la red virtual se restauran en la misma red virtual cuando se restaura a partir de una copia de seguridad.
- El servidor creado durante una restauración no tiene las reglas de firewall que existían en el servidor original. Las reglas del firewall para este nuevo servidor deben crearse por separado.
- La restauración a un servidor que se ha eliminado no se admite en este momento.
- No se admite la restauración entre regiones.

### <a name="other-features"></a>Otras características

* Todavía no se admite la autenticación de Azure AD. Se recomienda usar la opción [Servidor único](../overview-single-server.md) si necesita la autenticación de Azure AD.
* Todavía no se admiten las réplicas de lectura. Se recomienda usar la opción [Servidor único](../overview-single-server.md) si necesita réplicas de lectura.


## <a name="next-steps"></a>Pasos siguientes

- Comprensión de [las opciones de proceso y almacenamiento disponibles](concepts-compute-storage.md)
- Conozca las [versiones de base de datos de PostgreSQL admitidas](concepts-supported-versions.md).
- Revise [cómo hacer una copia de seguridad de un servidor y restaurarlo en Azure Database for PostgreSQL mediante Azure Portal](how-to-restore-server-portal.md).
