---
title: Limitaciones en Azure Database for MySQL
description: En este artículo se describen las limitaciones de Azure Database for MySQL como el número de conexiones o las opciones de motor de almacenamiento.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 12/6/2018
ms.openlocfilehash: 9088e9ad98633b46dc3a7f0ee9002a0dd9fc5a55
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65551890"
---
# <a name="limitations-in-azure-database-for-mysql"></a>Limitaciones en Azure Database for MySQL
En las siguientes secciones se describen la capacidad, la compatibilidad del motor de almacenamiento, la compatibilidad de los privilegios, la compatibilidad de las instrucciones de manipulación de datos y los límites funcionales del servicio de base de datos. Consulte también las [limitaciones generales](https://dev.mysql.com/doc/mysql-reslimits-excerpt/5.6/en/limits.html) que se aplican al motor de base de datos MySQL.

## <a name="maximum-connections"></a>Número máximo de conexiones
El número máximo de conexiones por plan de tarifa y núcleos virtuales es el siguiente: 

|**Plan de tarifa**|**Núcleos virtuales**| **Conexiones máximas**|
|---|---|---|
|Básica| 1| 50|
|Básica| 2| 100|
|Uso general| 2| 300|
|Uso general| 4| 625|
|Uso general| 8| 1250|
|Uso general| 16| 2.500|
|Uso general| 32| 5000|
|Uso general| 64| 10000|
|Memoria optimizada| 2| 600|
|Memoria optimizada| 4| 1250|
|Memoria optimizada| 8| 2.500|
|Memoria optimizada| 16| 5000|
|Memoria optimizada| 32| 10000|

Si las conexiones superan el límite, puede que reciba el error siguiente:
> ERROR 1040 (08004): Demasiadas conexiones

## <a name="storage-engine-support"></a>Compatibilidad del motor de almacenamiento

### <a name="supported"></a>Compatible
- [InnoDB](https://dev.mysql.com/doc/refman/5.7/en/innodb-introduction.html)
- [MEMORY](https://dev.mysql.com/doc/refman/5.7/en/memory-storage-engine.html)

### <a name="unsupported"></a>No compatible
- [MyISAM](https://dev.mysql.com/doc/refman/5.7/en/myisam-storage-engine.html)
- [BLACKHOLE](https://dev.mysql.com/doc/refman/5.7/en/blackhole-storage-engine.html)
- [ARCHIVE](https://dev.mysql.com/doc/refman/5.7/en/archive-storage-engine.html)
- [FEDERATED](https://dev.mysql.com/doc/refman/5.7/en/federated-storage-engine.html)

## <a name="privilege-support"></a>Compatibilidad de los privilegios

### <a name="unsupported"></a>No compatible
- Rol DBA: muchos parámetros de servidor y valores de configuración pueden reducir por error el rendimiento del servidor o invalidar las propiedades ACID del sistema de administración de bases de datos. Por lo tanto, para mantener la integridad del servicio y SLA en un nivel de producto, no se expone el rol DBA de este servicio. La cuenta de usuario predeterminada, que se crea a la vez que las instancias de base de datos, permite a los usuarios realizar la mayoría de las instrucciones DDL y DML en la instancia de base de datos administrados. 
- Privilegio SUPER: del mismo modo, los [privilegios SUPER](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_super) también están restringidos.
- DEFINER: Requiere privilegios SUPER para crear y está restringido. Si importa datos mediante una copia de seguridad, quite los comandos `CREATE DEFINER` manualmente o mediante el comando `--skip-definer` durante una operación mysqldump.

## <a name="data-manipulation-statement-support"></a>Compatibilidad de las instrucciones de manipulación de datos

### <a name="supported"></a>Compatible
- `LOAD DATA INFILE` es compatible, pero el parámetro `[LOCAL]` debe especificarse y dirigirse a una ruta de acceso UNC (Azure Storage montado a través de SMB).

### <a name="unsupported"></a>No compatible
- `SELECT ... INTO OUTFILE`

## <a name="functional-limitations"></a>Limitaciones funcionales

### <a name="scale-operations"></a>Operaciones de escalado
- El escalado dinámico a y desde niveles de precios Básico no se admite en este momento.
- La reducción del tamaño de almacenamiento del servidor no se admite.

### <a name="server-version-upgrades"></a>Actualizaciones de la versión de servidor
- La migración automatizada entre las principales versiones del motor de base de datos no se admite en este momento. Si quiere actualizar a la siguiente versión principal, realice un [volcado y restáurelo ](./concepts-migrate-dump-restore.md) a un servidor que se haya creado con la nueva versión del motor.

### <a name="point-in-time-restore"></a>Restauración a un momento dado
- Al usar la característica PITR, el nuevo servidor se crea con la misma configuración que el servidor en el que se basa.
- La restauración a un servidor que se ha eliminado no se admite en este momento.

### <a name="vnet-service-endpoints"></a>Puntos de conexión de servicio de red virtual
- La compatibilidad con puntos de conexión de servicio de red virtual solo existe para servidores de uso general y optimizados para memoria.

### <a name="storage-size"></a>Tamaño de almacenamiento
- Consulte [planes de tarifa](concepts-pricing-tiers.md) para los límites de tamaño de almacenamiento por cada plan de tarifa.

## <a name="current-known-issues"></a>Problemas conocidos actualmente
- Instancia del servidor MySQL muestra una versión de servidor errónea después de establecer la conexión. Para obtener la versión del motor de instancias de servidor correcta, use el comando `select version();`.

## <a name="next-steps"></a>Pasos siguientes
- [Qué está disponible en cada nivel de servicio](concepts-pricing-tiers.md)
- [Versiones de base de datos de MySQL admitidas](concepts-supported-versions.md)
