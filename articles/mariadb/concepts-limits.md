---
title: 'Limitaciones: Azure Database for MariaDB'
description: En este artículo se describen las limitaciones de Azure Database for MariaDB como el número de conexiones o las opciones de motor de almacenamiento.
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.topic: conceptual
ms.date: 6/25/2020
ms.openlocfilehash: 51aff856aa5bdeb042493d47f100be0ca32dfbbb
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88032686"
---
# <a name="limitations-in-azure-database-for-mariadb"></a>Limitaciones de Azure Database for MariaDB
En las siguientes secciones se describen la capacidad, la compatibilidad del motor de almacenamiento, la compatibilidad de los privilegios, la compatibilidad de las instrucciones de manipulación de datos y los límites funcionales del servicio de base de datos.

## <a name="server-parameters"></a>Parámetros del servidor

> [!NOTE]
> Si busca valores mínimos y máximos para los parámetros del servidor, como `max_connections` y `innodb_buffer_pool_size`, esta información se ha pasado al artículo sobre los **[parámetros del servidor](./concepts-server-parameters.md)** .

Azure Database for MariaDB admite el ajuste de los valores de parámetros del servidor. Los valores mínimo y máximo de algunos parámetros (por ejemplo, `max_connections`, `join_buffer_size`, `query_cache_size`) vienen determinados por el plan de tarifa y los núcleos virtuales del servidor. Consulte los [parámetros del servidor](./concepts-server-parameters.md) para más información sobre estos límites.

Tras la implementación inicial, un servidor de Azure para MariaDB incluye tablas de sistemas para la información de zona horaria, pero no se han rellenado. Las tablas de la zona horaria se pueden rellenar mediante una llamada al procedimiento almacenado `mysql.az_load_timezone` desde una herramienta como la línea de comandos de MySQL o MySQL Workbench. Vea los artículos de [Azure Portal](howto-server-parameters.md#working-with-the-time-zone-parameter) o de la [CLI de Azure](howto-configure-server-parameters-cli.md#working-with-the-time-zone-parameter) sobre cómo llamar al procedimiento almacenado y establecer las zonas horarias globales o de nivel de sesión.

## <a name="storage-engine-support"></a>Compatibilidad del motor de almacenamiento

### <a name="supported"></a>Compatible
- [InnoDB](https://mariadb.com/kb/en/library/xtradb-and-innodb/)
- [MEMORY](https://mariadb.com/kb/en/library/memory-storage-engine/)

### <a name="unsupported"></a>No compatible
- [MyISAM](https://mariadb.com/kb/en/library/myisam-storage-engine/)
- [BLACKHOLE](https://mariadb.com/kb/en/library/blackhole/)
- [ARCHIVE](https://mariadb.com/kb/en/library/archive/)

## <a name="privilege-support"></a>Compatibilidad de los privilegios

### <a name="unsupported"></a>No compatible
- Rol DBA: muchos parámetros y valores de servidor pueden reducir por error el rendimiento del servidor o invalidar las propiedades ACID del sistema de administración de bases de datos (DBMS). Por lo tanto, para mantener la integridad del servicio y el SLA en un nivel de producto, no se expone el rol DBA en este servicio. La cuenta de usuario predeterminada, que se crea a la vez que las instancias de base de datos, permite a los usuarios realizar la mayoría de las instrucciones DDL y DML en la instancia de base de datos administrada.
- Privilegio SUPER: del mismo modo, los [privilegios SUPER](https://mariadb.com/kb/en/library/grant/#global-privileges) también están restringidos.
- DEFINER: requiere privilegios SUPER para crear y está restringido. Si importa datos mediante una copia de seguridad, quite los comandos `CREATE DEFINER` manualmente o mediante el comando `--skip-definer` durante una operación mysqldump.
- Bases de datos del sistema: En Azure Database for MariaDB, la [base de datos del sistema MySQL](https://mariadb.com/kb/en/the-mysql-database-tables/) es de solo lectura, ya que se usa para admitir varias funcionalidades del servicio PaaS. Tenga en cuenta que no puede cambiar nada en la base de datos del sistema `mysql`.

## <a name="data-manipulation-statement-support"></a>Compatibilidad de las instrucciones de manipulación de datos

### <a name="supported"></a>Compatible
- `LOAD DATA INFILE` es compatible, pero el parámetro `[LOCAL]` debe especificarse y dirigirse a una ruta de acceso UNC (Azure Storage montado a través de SMB).

### <a name="unsupported"></a>No compatible
- `SELECT ... INTO OUTFILE`

## <a name="functional-limitations"></a>Limitaciones funcionales

### <a name="scale-operations"></a>Operaciones de escalado
- El escalado dinámico hacia y desde planes de tarifa Básico no se admite en este momento.
- La reducción del tamaño de almacenamiento del servidor no se admite.

### <a name="server-version-upgrades"></a>Actualizaciones de la versión de servidor
- La migración automatizada entre versiones principales del motor de base de datos no se admite en este momento.

### <a name="point-in-time-restore"></a>Restauración a un momento dado
- Al usar la característica PITR, el nuevo servidor se crea con la misma configuración que el servidor en el que se basa.
- La restauración a un servidor que se ha eliminado no se admite en este momento.

### <a name="subscription-management"></a>Administración de suscripciones
- El movimiento dinámico de servidores creados previamente entre grupo de suscripciones y recursos no se admite en este momento.

### <a name="vnet-service-endpoints"></a>Puntos de conexión de servicio de red virtual
- La compatibilidad con puntos de conexión de servicio de red virtual solo existe para servidores de uso general y optimizados para memoria.

### <a name="storage-size"></a>Tamaño de almacenamiento
- Consulte [Planes de tarifa](concepts-pricing-tiers.md) para obtener los límites de tamaño de almacenamiento para cada plan de tarifa.

## <a name="current-known-issues"></a>Problemas conocidos actualmente
- La instancia del servidor MariaDB muestra una versión de servidor incorrecta después de establecer la conexión. Para obtener la versión del motor de instancias de servidor correcta, use el comando `select version();`.

## <a name="next-steps"></a>Pasos siguientes
- [Qué está disponible en cada nivel de servicio](concepts-pricing-tiers.md)
- [Versiones de base de datos de MariaDB admitidas](concepts-supported-versions.md)
