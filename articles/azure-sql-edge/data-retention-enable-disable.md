---
title: 'Habilitación y deshabilitación de directivas de retención de datos: Azure SQL Edge'
description: Información sobre cómo habilitar y deshabilitar directivas de retención de datos en Azure SQL Edge
keywords: SQL Edge, retención de datos
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/04/2020
ms.openlocfilehash: ee2d65d66caef5cd9405d6e3d0e094de2e30ae87
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90902506"
---
# <a name="enable-and-disable-data-retention-policies"></a>Habilitación y deshabilitación de directivas de retención de datos

En este tema se describe cómo habilitar y deshabilitar las directivas de retención de datos para una tabla y una base de datos. 

## <a name="enable-data-retention-for-a-database"></a>Habilitación de la retención de datos para una base de datos

En el siguiente ejemplo se muestra cómo habilitar la retención de datos mediante [Alter Database](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-set-options).

```sql
ALTER DATABASE [<DatabaseName>] SET DATA_RETENTION  ON;
```

## <a name="check-if-data-retention-is-enabled-for-a-database"></a>Comprobación de si la retención de datos está habilitada para una base de datos

El siguiente comando se puede usar para comprobar si se ha habilitado la retención de datos para una base de datos
```sql
SELECT is_data_retention_enabled, name
FROM sys.databases;
```

## <a name="enable-data-retention-for-a-table"></a>Habilitación de la retención de datos para una tabla

La retención de datos debe estar habilitada para cada tabla cuyos datos quiere que se purguen automáticamente. Cuando se habilita la retención de información en la base de datos y en la tabla, una tarea de sistema en segundo plano examinará periódicamente la tabla para identificar y eliminar las filas obsoletas (antiguas). La retención de datos se puede habilitar en una tabla durante la creación de la tabla mediante [Crear tabla](https://docs.microsoft.com/sql/t-sql/statements/create-table-transact-sql) o [Modificar tabla](https://docs.microsoft.com/sql/t-sql/statements/alter-table-transact-sql).

En el siguiente ejemplo se muestra cómo habilitar la retención de datos para una tabla mediante [Crear tabla](https://docs.microsoft.com/sql/t-sql/statements/create-table-transact-sql). 

```sql
CREATE TABLE [dbo].[data_retention_table] 
(
[dbdatetime2] datetime2(7), 
[product_code] int, 
[value] char(10),  
CONSTRAINT [pk_current_data_retention_table] PRIMARY KEY CLUSTERED ([product_code])
) WITH (DATA_DELETION = ON ( FILTER_COLUMN = [dbdatetime2], RETENTION_PERIOD = 1 day ) )
```

La parte `WITH (DATA_DELETION = ON ( FILTER_COLUMN = [dbdatetime2], RETENTION_PERIOD = 1 day ) )` del comando para crear una tabla establece la retención de datos en la tabla. El comando usa los siguientes parámetros necesarios 

- DATA_DELETION: indica si la retención de datos está activada o desactivada.
- FILTER_COLUMN: nombre en la columna de la tabla que se utilizará para determinar si las filas están obsoletas o no. La columna de filtro solo puede ser una columna con los siguientes tipos de datos 
    - Date
    - SmallDateTime
    - DateTime
    - DateTime2
    - DateTimeOffset
- RETENTION_PERIOD: valor entero seguido de un descriptor de unidad. Las unidades permitidas son DAY, DAYS, WEEK, WEEKS, MONTH, MONTHS, YEAR y YEARS.

En el siguiente ejemplo se muestra cómo habilitar la retención de datos para una tabla mediante [Modificar tabla](https://docs.microsoft.com/sql/t-sql/statements/alter-table-transact-sql).  

```sql
Alter Table [dbo].[data_retention_table]
SET (DATA_DELETION = On (FILTER_COLUMN = [timestamp], RETENTION_PERIOD = 1 day))
```

## <a name="check-if-data-retention-is-enabled-for-a-table"></a>Comprobación de si la retención de datos está habilitada para una tabla

El siguiente comando se puede usar para comprobar las tablas para las que se ha habilitado la retención de datos

```sql
select name, data_retention_period, data_retention_period_unit from sys.tables
```

Un valor de data_retention_period = -1 y data_retention_period_unit como INFINITO, indica que no se ha establecido la retención de datos en la tabla.

La siguiente consulta se puede utilizar para identificar la columna utilizada como filter_column para la retención de los datos. 

```sql
Select name from sys.columns
where is_data_deletion_filter_column =1 
and object_id = object_id(N'dbo.data_retention_table', N'U')
```

## <a name="corelating-db-and-table-data-retention-settings"></a>Correlación de la configuración de la retención de la información en las bases de datos y en las tablas

La configuración de la retención de la información en la base de datos y la tabla se usa conjuntamente para determinar si la limpieza automática de las filas antiguas se ejecutará en las tablas. 

|Opción de base de datos | Opción de tabla | Comportamiento |
|----------------|--------------|----------|
| Apagado | Apagado | La Directiva de retención de datos está deshabilitada y la limpieza, tanto manual como automática de los registros antiguos, está deshabilitada.|
| Apagado | ACTIVAR  | La Directiva de retención de datos se ha habilitado para la tabla. La limpieza automática de los registros obsoletos está deshabilitada; sin embargo, se puede usar el método de limpieza manual para limpiarlos. |
| ACTIVAR | Apagado | La Directiva de retención de datos está habilitada en el nivel de base de datos. Sin embargo, dado que la opción está deshabilitada en el nivel de tabla, no hay ninguna limpieza de filas antiguas basada en la retención.|
| ACTIVAR | ACTIVAR | La Directiva de retención de datos está habilitada para la base de datos y para las tablas. La limpieza automática de los registros obsoletos está habilitada. |

## <a name="disable-data-retention-on-a-table"></a>Deshabilitación de la retención de datos en una tabla 

La retención de datos se puede deshabilitar en una tabla mediante [Modificar tabla](https://docs.microsoft.com/sql/t-sql/statements/alter-table-transact-sql). El siguiente comando se puede usar para deshabilitar la retención de datos en una tabla.

```sql
Alter Table [dbo].[data_retention_table]
Set (DATA_DELETION = OFF)
```

## <a name="disable-data-retention-on-a-database"></a>Deshabilitación de la retención de la información en una base de datos

La retención de la información se puede deshabilitar en una base de datos mediante [Modificar base de datos](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-set-options). El siguiente comando se puede usar para deshabilitar la retención de la información en una base de datos.

```sql
ALTER DATABASE <DatabaseName> SET DATA_RETENTION  OFF;
```

## <a name="next-steps"></a>Pasos siguientes
- [Retención de datos y purga automática de datos](data-retention-overview.md)
- [Administración de datos históricos con directivas de retención](data-retention-cleanup.md)
