---
title: 'Limitaciones: Azure Database for MySQL con servidor flexible'
description: En este artículo, se describen las limitaciones de Azure Database for MySQL con servidor flexible, como el número de opciones del motor de almacenamiento y de conexión.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 9/21/2020
ms.openlocfilehash: 64f4b6e87f038b265fbd2c3e13f3779fb4c24f74
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932265"
---
# <a name="limitations-in-azure-database-for-mysql---flexible-server-preview"></a>Limitaciones de Azure Database for MySQL con servidor flexible (versión preliminar)

> [!IMPORTANT] 
> Actualmente, Azure Database for MySQL con servidor flexible se encuentra en versión preliminar pública.

En este artículo se describen las limitaciones del servicio Azure Database for MySQL con servidor flexible. También se aplican las [limitaciones generales](https://dev.mysql.com/doc/mysql-reslimits-excerpt/5.7/en/limits.html) en el motor de base de datos de MySQL. Para obtener más información sobre los niveles de recursos (proceso, memoria y almacenamiento), consulte el artículo acerca del [proceso y el almacenamiento](concepts-compute-storage.md).

## <a name="server-parameters"></a>Parámetros del servidor

> [!NOTE]
> Si busca valores mínimos y máximos para los parámetros del servidor, como `max_connections` y `innodb_buffer_pool_size`, esta información se ha pasado al artículo sobre los conceptos de los parámetros del servidor <!-- **[server parameters](./concepts-server-parameters.md)** --> .

Azure Database for MySQL admite el ajuste de los valores de parámetros del servidor. Los valores mínimo y máximo de algunos parámetros (por ejemplo, `max_connections`, `join_buffer_size`, `query_cache_size`) vienen determinados por el nivel de proceso y el tamaño de proceso del servidor. Consulte los conceptos de los parámetros del servidor <!-- [server parameters](./concepts-server-parameters.md)--> para obtener más información acerca de estos límites.

## <a name="storage-engines"></a>Motores de almacenamiento

MySQL es compatible con muchos motores de almacenamiento. En Azure Database for MySQL con servidor flexible, se admiten y no se admiten los siguientes motores de almacenamiento:

### <a name="supported"></a>Compatible
- [InnoDB](https://dev.mysql.com/doc/refman/5.7/en/innodb-introduction.html)
- [MEMORY](https://dev.mysql.com/doc/refman/5.7/en/memory-storage-engine.html)

### <a name="unsupported"></a>No compatible
- [MyISAM](https://dev.mysql.com/doc/refman/5.7/en/myisam-storage-engine.html)
- [BLACKHOLE](https://dev.mysql.com/doc/refman/5.7/en/blackhole-storage-engine.html)
- [ARCHIVE](https://dev.mysql.com/doc/refman/5.7/en/archive-storage-engine.html)
- [FEDERATED](https://dev.mysql.com/doc/refman/5.7/en/federated-storage-engine.html)

## <a name="privileges--data-manipulation-support"></a>Compatibilidad con privilegios y con la manipulación de datos

Muchos parámetros y ajustes del servidor pueden reducir por error el rendimiento del servidor o invalidar las propiedades ACID del servidor de MySQL. Para mantener la integridad del servicio y el SLA en un nivel de producto, no se exponen varios roles en este servicio. 

El servicio MySQL no permite el acceso directo al sistema de archivos subyacente. No se admiten algunos comandos de manipulación de datos. 

### <a name="unsupported"></a>No compatible

No se admite lo siguiente:
- Rol DBA: restringido. De forma alternativa, puede usar el rol de administrador (generado durante la creación del nuevo servidor), que le permite ejecutar la mayoría de las instrucciones DDL y DML. 
- Privilegio SUPER: del mismo modo, el [privilegio SUPER](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_super) también está restringido.
- DEFINER: requiere privilegios SUPER para crear y está restringido. Si importa datos mediante una copia de seguridad, quite los comandos `CREATE DEFINER` manualmente o mediante el comando `--skip-definer` durante una operación mysqldump.
- Bases de datos del sistema: La [base de datos del sistema de MySQL](https://dev.mysql.com/doc/refman/5.7/en/system-schema.html) es de solo lectura y se usa para admitir varias funcionalidades de PaaS. No puede realizar cambios en la base de datos del sistema de `mysql`.
- `SELECT ... INTO OUTFILE`: no se admite en el servicio.

### <a name="supported"></a>Compatible
- `LOAD DATA INFILE` es compatible, pero el parámetro `[LOCAL]` debe especificarse y dirigirse a una ruta de acceso UNC (Azure Storage montado a través de SMB).

## <a name="functional-limitations"></a>Limitaciones funcionales

### <a name="zone-redundant-ha"></a>Alta disponibilidad con redundancia de zona
- Esta opción de configuración solo se puede establecer durante la creación del servidor.
- No se admite en el nivel de proceso flexible.

### <a name="networking"></a>Redes
- El método de conectividad no se puede cambiar después de crear el servidor. Si el servidor se crea con la opción *Acceso privado (integración con red virtual)* , no se puede cambiar a *Acceso público (direcciones IP permitidas)* después de la creación, y viceversa
- TLS/SSL está habilitado de forma predeterminada y no se puede deshabilitar.
- La versión de TLS mínima que admite el servidor es TLS 1.2. Consulte el artículo sobre cómo [conectarse mediante TLS/SSL](./how-to-connect-tls-ssl.md) para obtener más información.

### <a name="stopstart-operation"></a>Operación de inicio/detención
- No es compatible con las configuraciones de alta disponibilidad con redundancia de zona (principal y en espera).
- No es compatible con las configuraciones de réplica de lectura (tanto de origen como de réplicas).

### <a name="scale-operations"></a>Operaciones de escalado
- No se admite la reducción del almacenamiento del servidor aprovisionado.

### <a name="read-replicas"></a>Réplicas de lectura
- No es compatible con las configuraciones de alta disponibilidad con redundancia de zona (principal y en espera).

### <a name="server-version-upgrades"></a>Actualizaciones de la versión de servidor
- No se admite la migración automatizada entre versiones principales del motor de base de datos. Si quiere actualizar a la versión principal, realice un [volcado y restáurelo](../concepts-migrate-dump-restore.md) a un servidor que se haya creado con la nueva versión del motor.

### <a name="restoring-a-server"></a>Restauración de un servidor
- Con la restauración a un momento dado, se crea un nuevo servidor con las mismas configuraciones de proceso y almacenamiento que el servidor de origen en el que se basa. El proceso del servidor recién restaurado se puede reducir verticalmente después de crear el servidor.
- No se admite la restauración de un servidor eliminado.

## <a name="next-steps"></a>Pasos siguientes

- Comprenda las [opciones de proceso y almacenamiento disponibles](concepts-compute-storage.md).
- Obtenga información acerca de las [versiones admitidas de MySQL](concepts-supported-versions.md).
- Revise [cómo hacer una copia de seguridad de un servidor y restaurarlo mediante Azure Portal](how-to-restore-server-portal.md).