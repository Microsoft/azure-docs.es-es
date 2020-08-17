---
title: 'Problemas conocidos: Migraciones en línea a Azure Database for MySQL'
titleSuffix: Azure Database Migration Service
description: Obtenga información sobre limitaciones y problemas conocidos de migración con las migraciones en línea a Azure Database for MySQL al usar Azure Database Migration Service.
services: database-migration
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom:
- seo-lt-2019
- seo-dt-2019
ms.topic: article
ms.date: 02/20/2020
ms.openlocfilehash: 9a2e28439efaa1983c4deeff4c6746108fc28e4e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090712"
---
# <a name="online-migration-issues--limitations-to-azure-db-for-mysql-with-azure-database-migration-service"></a>Limitaciones y problemas de migración en línea de Azure DB for MySQL con Azure Database Migration Service

Los problemas conocidos y las limitaciones relacionadas con las migraciones en línea de MySQL a Azure Database for MySQL se describen en las siguientes secciones.

## <a name="online-migration-configuration"></a>Configuración de la migración en línea


- La versión del servidor MySQL Server de origen debe ser la 5.6.35, 5.7.18 o una posterior.
- Azure Database for MySQL es compatible con:
  - MySQL Community Edition
  - Motor de InnoDB
- Migración de la misma versión. No se admite la migración de MySQL 5.6 a Azure Database for MySQL 5.7.
- Habilite el registro binario en my.ini (Windows) o my.cnf (Unix).
  - Establezca Server_id en cualquier número mayor o igual a 1, por ejemplo, Server_id = 1 (solo para MySQL 5.6).
  - Establezca log-bin = \<path> (solo para MySQL 5.6).
  - Establezca binlog_format = row.
  - Expire_logs_days = 5 (recomendado - solo para MySQL 5.6).
- El usuario debe tener el rol ReplicationAdmin.
- Las intercalaciones definidas para la base de datos de MySQL de origen son las mismas que las que están definidas en Azure Database for MySQL de destino.
- El esquema debe coincidir entre la base de datos de MySQL de origen y la base de datos de destino de Azure Database for MySQL.
- El esquema de Azure Database for MySQL de destino no debe tener claves externas. Utilice la consulta siguiente para eliminar las claves externas:
    ```
    SET group_concat_max_len = 8192;
    SELECT SchemaName, GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery, GROUP_CONCAT(AddQuery SEPARATOR ';\n') as AddQuery
    FROM
    (SELECT 
    KCU.REFERENCED_TABLE_SCHEMA as SchemaName, KCU.TABLE_NAME, KCU.COLUMN_NAME,
        CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' DROP FOREIGN KEY ', KCU.CONSTRAINT_NAME) AS DropQuery,
        CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' ADD CONSTRAINT ', KCU.CONSTRAINT_NAME, ' FOREIGN KEY (`', KCU.COLUMN_NAME, '`) REFERENCES `', KCU.REFERENCED_TABLE_NAME, '` (`', KCU.REFERENCED_COLUMN_NAME, '`) ON UPDATE ',RC.UPDATE_RULE, ' ON DELETE ',RC.DELETE_RULE) AS AddQuery
        FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE KCU, information_schema.REFERENTIAL_CONSTRAINTS RC
        WHERE
          KCU.CONSTRAINT_NAME = RC.CONSTRAINT_NAME
          AND KCU.REFERENCED_TABLE_SCHEMA = RC.UNIQUE_CONSTRAINT_SCHEMA
      AND KCU.REFERENCED_TABLE_SCHEMA = ['schema_name']) Queries
      GROUP BY SchemaName;
    ```

    Ejecute la clave externa que desea eliminar (que es la segunda columna) en el resultado de la consulta.
- El esquema de Azure Database for MySQL de destino no debe tener ningún desencadenador. Para eliminar los desencadenadores en la base de datos de destino:
    ```
    SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
    ```

## <a name="datatype-limitations"></a>Limitaciones del tipo de datos

- **Limitación**: si hay un tipo de datos JSON en la base de datos MySQL de origen, se producirá un error en la migración durante la sincronización continua.

    **Solución alternativa**: modificar el tipo de datos JSON a un texto medio o largo en la base de datos MySQL de origen.

- **Limitación**: si no hay ninguna clave principal en las tablas, se producirá un error en la sincronización continua.

    **Solución alternativa**: establecer temporalmente una clave principal para la tabla para que continúe la migración. Puede quitar la clave principal una vez completada la migración de datos.

## <a name="lob-limitations"></a>Limitaciones de LOB

Las columnas de objetos grandes (LOB) son columnas que pueden alcanzar un tamaño considerable. Para MySQL, algunos de los tipos de datos de LOB son Medium text, Longtext, Blob, Mediumblob, Longblob, etc.

- **Limitación**: si se usan tipos de datos de LOB como claves principales, se producirá un error en la migración.

    **Solución alternativa**: reemplazar la clave principal por otros tipos de datos o columnas que no sean de LOB.

- **Limitación**: Si la longitud de columna de objetos grandes (LOB) es mayor que 32 KB, es posible que se trunquen los datos en el destino. Puede comprobar la longitud de columna de LOB mediante esta consulta:
    ```
    SELECT max(length(description)) as LEN from catalog;
    ```

    **Solución alternativa**: si tiene un objeto LOB mayor de 32 KB, póngase en contacto con el equipo de ingeniería en [Ask Azure Database Migrations](mailto:AskAzureDatabaseMigrations@service.microsoft.com).

## <a name="limitations-when-migrating-online-from-aws-rds-mysql"></a>Limitaciones al migrar en línea desde AWS RDS MySQL

Al intentar realizar una migración en línea desde AWS RDS MySQL hasta Azure Database for MySQL, pueden producirse los siguientes errores.

- **Error:** la base de datos "{0}" tiene claves externas en el destino. Corrija el destino e inicie una nueva actividad de migración de datos. Ejecute el siguiente script en el destino para mostrar las claves externas.

  **Limitación**: Si tiene claves externas en el esquema, se producirá un error en la carga inicial y la sincronización continua de la migración.
  **Solución alternativa**: Ejecute el siguiente script en MySQL Workbench para extraer el script para eliminar la clave externa y el script para agregar clave externa:

  ```
  SET group_concat_max_len = 8192; SELECT SchemaName, GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery, GROUP_CONCAT(AddQuery SEPARATOR ';\n') as AddQuery FROM (SELECT KCU.REFERENCED_TABLE_SCHEMA as SchemaName, KCU.TABLE_NAME, KCU.COLUMN_NAME, CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' DROP FOREIGN KEY ', KCU.CONSTRAINT_NAME) AS DropQuery, CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' ADD CONSTRAINT ', KCU.CONSTRAINT_NAME, ' FOREIGN KEY (`', KCU.COLUMN_NAME, '`) REFERENCES `', KCU.REFERENCED_TABLE_NAME, '` (`', KCU.REFERENCED_COLUMN_NAME, '`) ON UPDATE ',RC.UPDATE_RULE, ' ON DELETE ',RC.DELETE_RULE) AS AddQuery FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE KCU, information_schema.REFERENTIAL_CONSTRAINTS RC WHERE KCU.CONSTRAINT_NAME = RC.CONSTRAINT_NAME AND KCU.REFERENCED_TABLE_SCHEMA = RC.UNIQUE_CONSTRAINT_SCHEMA AND KCU.REFERENCED_TABLE_SCHEMA = 'SchemaName') Queries GROUP BY SchemaName;
  ```

- **Error:** la base de datos "{0}" no existe en el servidor. El servidor de origen de MySQL proporcionado distingue mayúsculas de minúsculas. Compruebe el nombre de la base de datos.

  **Limitación**: Al migrar una base de datos MySQL a Azure mediante la interfaz de la línea de comandos (CLI), los usuarios pueden encontrarse este error. El servicio no pudo ubicar la base de datos en el servidor de origen. Puede que haya proporcionado un nombre de base de datos incorrecto o que la base de datos no exista en el servidor mostrado. Tenga en cuenta que los nombres de las bases de datos distinguen mayúsculas de minúsculas.

  **Solución alternativa**: proporcione el nombre exacto de la base de datos e inténtelo de nuevo.

- **Error:** hay tablas con el mismo nombre en la base de datos "{database}". Azure Database for MySQL no admite tablas que distinguen mayúsculas de minúsculas.

  **Limitación**: Este error se produce cuando se tienen dos tablas con el mismo nombre en la base de datos de origen. Azure Database for MySQL no admite tablas que distingan mayúsculas de minúsculas.

  **Solución alternativa**: actualice los nombres de tabla para que sean únicos y vuelva a intentarlo.

- **Error:** la base de datos de destino {database} está vacía. Migre el esquema.

  **Limitación**: Este error se produce cuando la base de datos de Azure Database for MySQL de destino no tiene el esquema necesario. La migración del esquema es necesaria para habilitar la migración de datos al destino.

  **Solución alternativa**: [migre el esquema](https://docs.microsoft.com/azure/dms/tutorial-mysql-azure-mysql-online#migrate-the-sample-schema) de la base de datos de origen a la de destino.

## <a name="other-limitations"></a>Otras limitaciones

- No se admite una cadena de contraseña que tenga llaves de apertura y cierre {  } al principio y al final de la cadena de contraseña. Esta limitación se aplica tanto a conectarse a MySQL de origen como a Azure Database for MySQL de destino.
- No se admiten los DDL siguientes:
  - Todos los DDL de la partición
  - Quitar tabla
  - Cambio de nombre de una tabla
- No está admitido el uso de la instrucción *alter table <nombre_tabla> add column <nombre_columna>* para agregar columnas al principio o en medio de una tabla. La instrucción *alter table <nombre_tabla> add column <nombre_columna>* agrega la columna al final de la tabla.
- No se admiten índices creados solo en parte de la columna de datos. La siguiente instrucción es un ejemplo que crea un índice utilizando solo parte de la columna de datos:

    ``` 
    CREATE INDEX partial_name ON customer (name(10));
    ```

- En Azure Database Migration Service, el número de bases de datos que se van a migrar en una única actividad de migración se limita a cuatro.

- Azure DMS no admite la acción referencial CASCADE, que ayuda a eliminar o actualizar automáticamente una fila coincidente en la tabla secundaria cuando se elimina o se actualiza una fila en la tabla primaria. Para obtener más información, consulte la sección sobre acciones referenciales del artículo [FOREIGN KEY Constraints](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html) en la documentación de MySQL. Azure DMS requiere que se quiten las restricciones de claves externas en el servidor de base de datos de destino durante la carga de datos inicial, y no se pueden usar acciones referenciales. Si la carga de trabajo depende de la actualización de una tabla secundaria relacionada a través de esta acción referencial, se recomienda realizar un [volcado y restauración](https://docs.microsoft.com/azure/mysql/concepts-migrate-dump-restore) en su lugar. 

- **Error:** Tamaño de fila demasiado grande (> 8126). Puede ser útil cambiar algunas columnas a TEXTO o BLOB. En el formato de fila actual, el prefijo BLOB de 0 bytes se almacena en línea.

  **Limitación**: Este error se produce cuando se realiza la migración a Azure Database for MySQL mediante el motor de almacenamiento InnoDB y el tamaño de la fila de la tabla es demasiado grande (> 8126 bytes).

  **Solución alternativa**: Actualice el esquema de la tabla que tiene un tamaño de fila superior a 8126 bytes. No se recomienda cambiar el modo strict porque los datos se truncarán. No se admite el cambio de page_size.
