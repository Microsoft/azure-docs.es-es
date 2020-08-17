---
title: Uso de tablas temporales en Synapse SQL
description: Instrucciones esenciales para el uso de tablas temporales en SQL de Synapse.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 04/15/2020
ms.author: xiaoyul
ms.reviewer: igorstan
ms.openlocfilehash: c6c0e86bc372790cda2de4ff4c1274f414a01ab0
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503215"
---
# <a name="temporary-tables-in-synapse-sql"></a>Tablas temporales en SQL de Synapse

Este artículo contiene directrices esenciales para el uso de tablas temporales y resalta los principios de las tablas temporales de nivel de sesión en SQL de Synapse. 

Los recursos del grupo de SQL y SQL a petición (versión preliminar) pueden emplear tablas temporales. SQL a petición tiene limitaciones que se analizan al final de este artículo. 

## <a name="temporary-tables"></a>Tablas temporales

Las tablas temporales son útiles al procesar datos, especialmente durante la transformación, en la que los resultados intermedios son transitorios. En SQL de Synapse, las tablas temporales existen en el nivel de sesión.  Solo están visibles para la sesión en la que se crearon. Como tales, se quitan automáticamente cuando se cierra la sesión. 

## <a name="temporary-tables-in-sql-pool"></a>Tablas temporales en el grupo de SQL

En el recurso de grupo de SQL, las tablas temporales ofrecen ventajas para el rendimiento porque sus resultados se escriben en el almacenamiento local, en lugar de en el remoto.

### <a name="create-a-temporary-table"></a>Creación de una tabla temporal

Las tablas temporales se crean colocando `#` delante del nombre de la tabla.  Por ejemplo:

```sql
CREATE TABLE #stats_ddl
(
    [schema_name]        NVARCHAR(128) NOT NULL
,    [table_name]            NVARCHAR(128) NOT NULL
,    [stats_name]            NVARCHAR(128) NOT NULL
,    [stats_is_filtered]     BIT           NOT NULL
,    [seq_nmbr]              BIGINT        NOT NULL
,    [two_part_name]         NVARCHAR(260) NOT NULL
,    [three_part_name]       NVARCHAR(400) NOT NULL
)
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
,    HEAP
)
```

También se pueden crear tablas temporales mediante `CTAS` siguiendo exactamente el mismo método:

```sql
CREATE TABLE #stats_ddl
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
,    HEAP
)
AS
(
SELECT
        sm.[name]                                                                AS [schema_name]
,        tb.[name]                                                                AS [table_name]
,        st.[name]                                                                AS [stats_name]
,        st.[has_filter]                                                            AS [stats_is_filtered]
,       ROW_NUMBER()
        OVER(ORDER BY (SELECT NULL))                                            AS [seq_nmbr]
,                                 QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [two_part_name]
,        QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [three_part_name]
FROM    sys.objects            AS ob
JOIN    sys.stats            AS st    ON    ob.[object_id]        = st.[object_id]
JOIN    sys.stats_columns    AS sc    ON    st.[stats_id]        = sc.[stats_id]
                                    AND st.[object_id]        = sc.[object_id]
JOIN    sys.columns            AS co    ON    sc.[column_id]        = co.[column_id]
                                    AND    sc.[object_id]        = co.[object_id]
JOIN    sys.tables            AS tb    ON    co.[object_id]        = tb.[object_id]
JOIN    sys.schemas            AS sm    ON    tb.[schema_id]        = sm.[schema_id]
WHERE    1=1
AND        st.[user_created]   = 1
GROUP BY
        sm.[name]
,        tb.[name]
,        st.[name]
,        st.[filter_definition]
,        st.[has_filter]
)
;
```

> [!NOTE]
> `CTAS` es un comando eficaz y ofrece un uso eficiente del espacio del registro de transacciones. 
> 
> 

### <a name="drop-temporary-tables"></a>Eliminación de tablas temporales
Cuando se crea una nueva sesión, no debe existir ninguna tabla temporal.  Sin embargo, si va a llamar al mismo procedimiento almacenado que crea un archivo temporal con el mismo nombre para tener la seguridad de que las instrucciones `CREATE TABLE` se ejecutan correctamente, puede usar una sencilla comprobación de existencia previa con `DROP`: 

```sql
IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN
    DROP TABLE #stats_ddl
END
```

Por coherencia con la codificación, se recomienda usar este patrón tanto para tablas como para tablas temporales.  También es buena idea usar `DROP TABLE` para quitar las tablas temporales cuando ya no las necesite.  

En el desarrollo de procedimientos almacenados, es habitual que los comandos de eliminación se empaqueten juntos al final de un procedimiento para garantizar que estos objetos se limpian.

```sql
DROP TABLE #stats_ddl
```

### <a name="modularize-code"></a>Modularización de código
Las tablas temporales se pueden usar en cualquier parte de una sesión de usuario. Esta capacidad se puede aprovechar para ayudarle a separar el código de la aplicación en módulos.  Como demostración, el siguiente procedimiento almacenado genera DDL para actualizar todas las estadísticas de la base de datos por nombre de la estadística:

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_update_stats]
(   @update_type    tinyint -- 1 default 2 fullscan 3 sample 4 resample
    ,@sample_pct     tinyint
)
AS

IF @update_type NOT IN (1,2,3,4)
BEGIN;
    THROW 151000,'Invalid value for @update_type parameter. Valid range 1 (default), 2 (fullscan), 3 (sample) or 4 (resample).',1;
END;

IF @sample_pct IS NULL
BEGIN;
    SET @sample_pct = 20;
END;

IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN
    DROP TABLE #stats_ddl
END

CREATE TABLE #stats_ddl
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
)
AS
(
SELECT
        sm.[name]                                                                AS [schema_name]
,        tb.[name]                                                                AS [table_name]
,        st.[name]                                                                AS [stats_name]
,        st.[has_filter]                                                            AS [stats_is_filtered]
,       ROW_NUMBER()
        OVER(ORDER BY (SELECT NULL))                                            AS [seq_nmbr]
,                                 QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [two_part_name]
,        QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [three_part_name]
FROM    sys.objects            AS ob
JOIN    sys.stats            AS st    ON    ob.[object_id]        = st.[object_id]
JOIN    sys.stats_columns    AS sc    ON    st.[stats_id]        = sc.[stats_id]
                                    AND st.[object_id]        = sc.[object_id]
JOIN    sys.columns            AS co    ON    sc.[column_id]        = co.[column_id]
                                    AND    sc.[object_id]        = co.[object_id]
JOIN    sys.tables            AS tb    ON    co.[object_id]        = tb.[object_id]
JOIN    sys.schemas            AS sm    ON    tb.[schema_id]        = sm.[schema_id]
WHERE    1=1
AND        st.[user_created]   = 1
GROUP BY
        sm.[name]
,        tb.[name]
,        st.[name]
,        st.[filter_definition]
,        st.[has_filter]
)
SELECT
    CASE @update_type
    WHEN 1
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+');'
    WHEN 2
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH FULLSCAN;'
    WHEN 3
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH SAMPLE '+CAST(@sample_pct AS VARCHAR(20))+' PERCENT;'
    WHEN 4
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH RESAMPLE;'
    END AS [update_stats_ddl]
,   [seq_nmbr]
FROM    t1
;
GO
```

En este punto, la única acción que se ha producido es la creación de un procedimiento almacenado que genera la tabla temporal, #stats_ddl.  El procedimiento almacenado quita #stats_ddl si ya existe. Al quitarlo, se garantiza que no producirá un error si se ejecuta más de una vez dentro de una sesión.  

Puesto que no hay ningún elemento `DROP TABLE` al final del procedimiento almacenado, cuando este se complete, la tabla creada se conservará y se podrá leer fuera del procedimiento almacenado.  

A diferencia de otras bases de datos SQL Server, SQL de Synapse permite usar la tabla temporal fuera del procedimiento almacenado que la ha creado.  Las tablas temporales que se crearon mediante el grupo de SQL se pueden usar **en cualquier parte** dentro de la sesión. Como resultado, tendrá un código más modular y fácil de administrar, tal como se muestra en el ejemplo siguiente:

```sql
EXEC [dbo].[prc_sqldw_update_stats] @update_type = 1, @sample_pct = NULL;

DECLARE @i INT              = 1
,       @t INT              = (SELECT COUNT(*) FROM #stats_ddl)
,       @s NVARCHAR(4000)   = N''

WHILE @i <= @t
BEGIN
    SET @s=(SELECT update_stats_ddl FROM #stats_ddl WHERE seq_nmbr = @i);

    PRINT @s
    EXEC sp_executesql @s
    SET @i+=1;
END

DROP TABLE #stats_ddl;
```

### <a name="temporary-table-limitations"></a>Limitaciones de tablas temporales

El grupo de SQL tiene algunas limitaciones de implementación para las tablas temporales:

- Solo se admiten las tablas temporales con ámbito de sesión.  No se admiten las tablas temporales globales.
- No se pueden crear vistas en las tablas temporales.
- Solo se pueden crear tablas temporales con distribución hash o round robin.  No se admite la distribución de tablas temporales replicadas. 

## <a name="temporary-tables-in-sql-on-demand-preview"></a>Tablas temporales en SQL a petición (versión preliminar)

Las tablas temporales en SQL a petición son compatibles, pero su uso es limitado. No se pueden usar en consultas que tienen archivos como destino. 

Por ejemplo, no puede combinar una tabla temporal con datos de los archivos almacenados. El número de tablas temporales está limitado a 100 y su tamaño total está limitado a 100 MB.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el desarrollo de tablas, consulte el artículo [Diseño de tablas mediante los recursos de SQL de Synapse](develop-tables-overview.md).

