---
title: 'Problemas conocidos: Migraciones en línea de PostgreSQL a Azure Database for PostgreSQL'
titleSuffix: Azure Database Migration Service
description: Aprenda sobre problemas conocidos y limitaciones de migración con las migraciones en línea de PostgreSQL a Azure Database for PostgreSQL con Azure Database Migration Service.
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
ms.openlocfilehash: 564581a102ac3fab504e82db00ef54b3e45d0c19
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090746"
---
# <a name="known-issuesmigration-limitations-with-online-migrations-from-postgresql-to-azure-db-for-postgresql"></a>Problemas conocidos y limitaciones de migración con las migraciones en línea de PostgreSQL a Azure DB for PostgreSQL

Los problemas conocidos y las limitaciones relacionadas con las migraciones en línea de PostgreSQL a Azure Database for PostgreSQL se describen en las siguientes secciones.

## <a name="online-migration-configuration"></a>Configuración de la migración en línea

- El servidor PostgreSQL de origen debe ejecutar las versiones 9.4, 9.5, 9.6, 10 u 11. Para más información, consulte el artículo acerca de las [versiones de base de datos admitidas de PostgreSQL](../postgresql/concepts-supported-versions.md).
- Solo se admiten las migraciones a la misma versión o a otra superior. Por ejemplo, se admite la migración de PostgreSQL 9.5 a Azure Database for PostgreSQL 9.6 o 10, pero no de PostgreSQL 11 a PostgreSQL 9.6.
- Para habilitar la replicación lógica en el archivo de origen **PostgreSQL postgresql.conf**, establezca los parámetros siguientes:
  - **wal_level** = logical
  - **max_replication_slots** = [al menos el número máximo de bases de datos para la migración]; si quiere migrar cuatro bases de datos, establezca el valor en al menos 4.
  - **max_wal_senders** = [número de bases de datos que se ejecutan simultáneamente]; el valor recomendado es 10
- Agregue la dirección IP del agente DMS al archivo pg_hba.conf de PostgreSQL de origen.
  1. Cuando termine de aprovisionar una instancia de Azure Database Migration Service, anote la dirección IP de DMS.
  2. Agregue la dirección IP al archivo pg_hba.conf, tal como se muestran:

      ```
          host  all     172.16.136.18/10    md5
          host  replication postgres    172.16.136.18/10    md5
      ```

- El usuario debe tener el rol REPLICACIÓN en el servidor que hospeda la base de datos de origen.
- Los esquemas de base de datos de origen y destino deben coincidir.
- El esquema de Azure Database for PostgreSQL (servidor único) de destino no debe tener claves externas. Utilice la consulta siguiente para eliminar las claves externas:

    ```
                                SELECT Queries.tablename
           ,concat('alter table ', Queries.tablename, ' ', STRING_AGG(concat('DROP CONSTRAINT ', Queries.foreignkey), ',')) as DropQuery
                ,concat('alter table ', Queries.tablename, ' ', 
                                                STRING_AGG(concat('ADD CONSTRAINT ', Queries.foreignkey, ' FOREIGN KEY (', column_name, ')', 'REFERENCES ', foreign_table_name, '(', foreign_column_name, ')' ), ',')) as AddQuery
        FROM
        (SELECT
        tc.table_schema, 
        tc.constraint_name as foreignkey, 
        tc.table_name as tableName, 
        kcu.column_name, 
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name 
    FROM 
        information_schema.table_constraints AS tc 
        JOIN information_schema.key_column_usage AS kcu
          ON tc.constraint_name = kcu.constraint_name
          AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu
          ON ccu.constraint_name = tc.constraint_name
          AND ccu.table_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY') Queries
      GROUP BY Queries.tablename;
    
    ```

    Ejecute la clave externa que desea eliminar (que es la segunda columna) en el resultado de la consulta.

- El esquema de Azure Database for PostgreSQL (servidor único) de destino no debe tener ningún desencadenador. Use el siguiente comando para deshabilitar los desencadenadores en la base de datos:

     ```
    SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
     ```

## <a name="datatype-limitations"></a>Limitaciones del tipo de datos

  **Limitación**: Si no hay ninguna clave principal en las tablas, es posible que no se sincronicen los cambios en la base de datos de destino.

  **Solución alternativa**: establecer temporalmente una clave principal para la tabla para que continúe la migración. Puede quitar la clave principal una vez completada la migración de datos.

## <a name="limitations-when-migrating-online-from-aws-rds-postgresql"></a>Limitaciones al migrar en línea desde AWS RDS PostgreSQL

Al intentar realizar una migración en línea desde AWS RDS PostgreSQL a Azure Database for PostgreSQL, pueden producirse los siguientes errores.

- **Error**: El valor predeterminado de la columna "{column}" de la tabla "{table}" de la base de datos "{database}" es diferente en los servidores de origen y destino. Es "{valor en origen}" en el origen y "{valor en destino}" en el destino.

  **Limitación**: este error se produce cuando el valor predeterminado de un esquema de columna es diferente entre las bases de datos de origen y de destino.
  **Solución alternativa**: asegúrese de que el esquema en el destino coincide con el esquema en el origen. Para obtener detalles sobre la migración de esquemas, consulte la [documentación sobre la migración en línea de Azure PostgreSQL](https://docs.microsoft.com/azure/dms/tutorial-postgresql-azure-postgresql-online#migrate-the-sample-schema).

- **Error**: La base de datos de destino "{database}" tiene "{número de tablas}" tablas mientras que la base de datos de origen "{database}" tiene "{número de tablas}" tablas. El número de tablas en las bases de datos de origen y de destino deben coincidir.

  **Limitación**: este error se produce cuando el número de tablas es diferente entre las bases de datos de origen y de destino.

  **Solución alternativa**: asegúrese de que el esquema en el destino coincide con el esquema en el origen. Para obtener detalles sobre la migración de esquemas, consulte la [documentación sobre la migración en línea de Azure PostgreSQL](https://docs.microsoft.com/azure/dms/tutorial-postgresql-azure-postgresql-online#migrate-the-sample-schema).

- **Error:** la base de datos de origen {database} está vacía.

  **Limitación**: este error se produce cuando la base de datos de origen está vacía. La causa más probable es que haya seleccionado la base de datos incorrecta como origen.

  **Solución alternativa**: vuelva a comprobar la base de datos de origen que ha seleccionado para la migración y, a continuación, inténtelo de nuevo.

- **Error:** la base de datos de destino {database} está vacía. Migre el esquema.

  **Limitación**: este error se produce cuando no hay ningún esquema en la base de datos de destino. Asegúrese de que el esquema en el destino coincide con el esquema en el origen.
  **Solución alternativa**: asegúrese de que el esquema en el destino coincide con el esquema en el origen. Para obtener detalles sobre la migración de esquemas, consulte la [documentación sobre la migración en línea de Azure PostgreSQL](https://docs.microsoft.com/azure/dms/tutorial-postgresql-azure-postgresql-online#migrate-the-sample-schema).

## <a name="other-limitations"></a>Otras limitaciones

- El nombre de la base de datos no puede incluir ningún punto y coma (;).
- Una tabla capturada debe tener una clave principal. Si una tabla no tiene una clave principal, el resultado de las operaciones de registro DELETE y UPDATE será impredecible.
- Se omitirá la actualización de un segmento de clave principal. En estos casos, el destino identificará la aplicación de dicha actualización como una actualización que no actualizó ninguna fila y se escribirá un registro en la tabla de excepciones.
- La migración de varias tablas con el mismo nombre pero con un caso diferente (por ejemplo, table1, TABLE1 y Table1) puede provocar un comportamiento impredecible y, por tanto, no se admite.
- No se admite cambiar el procesamiento de los DDL de la tabla [CREATE | ALTER | DROP | TRUNCATE].
- En Azure Database Migration Service, una única actividad de migración solo puede alojar hasta cuatro bases de datos.
