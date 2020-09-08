---
title: Creación y actualización de estadísticas en tablas mediante Azure Synapse SQL
description: Recomendaciones y ejemplos para crear y actualizar las estadísticas de optimización de consultas en las tablas del grupo de SQL de Synapse.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 05/09/2018
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 914c3128805c9875249bb1998fcdb6e456e73b16
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799322"
---
# <a name="table-statistics-in-synapse-sql-pool"></a>Estadísticas de tablas en el grupo de SQL de Synapse

En este artículo, encontrará recomendaciones y ejemplos para crear y actualizar las estadísticas de optimización de consultas en las tablas del grupo de SQL.

## <a name="why-use-statistics"></a>¿Por qué usar estadísticas?

Cuanto más sepa el grupo de SQL acerca de los datos, más rápido puede ejecutar consultas en ellos. Después de cargar los datos en el grupo de SQL, la recopilación de estadísticas de los datos es una de las cosas más importantes que puede hacer para optimizar sus consultas.

El optimizador de consultas del grupo de SQL está basado en el costo. Compara el costo de varios planes de consulta y elige el menor de ellos. En la mayoría de los casos, elige el plan que se ejecutará más rápidamente.

Por ejemplo, si el optimizador estima que la fecha en que se filtra la consulta devolverá una fila, se elegirá un plan. Si estima que la fecha seleccionada devolverá 1 millón de filas, devolverá un plan diferente.

## <a name="automatic-creation-of-statistic"></a>Creación automática de estadísticas

Cuando la opción de base de datos AUTO_CREATE_STATISTICS está activada, el grupo de SQL analiza las consultas entrantes de los usuarios para buscar las estadísticas que faltan.

Si faltan las estadísticas, el optimizador de consultas crea las estadísticas en columnas individuales en el predicado de consulta o en la condición de combinación para mejorar las estimaciones de cardinalidad del plan de consulta.

> [!NOTE]
> Actualmente, la creación automática de estadísticas está activada de forma predeterminada.

Puede comprobar si el grupo de SQL tiene configurado AUTO_CREATE_STATISTICS; para ello, ejecute el siguiente comando:

```sql
SELECT name, is_auto_create_stats_on
FROM sys.databases
```

Si no lo tiene, se recomienda habilitar esta propiedad, para lo que se ejecuta el siguiente comando:

```sql
ALTER DATABASE <yourdatawarehousename>
SET AUTO_CREATE_STATISTICS ON
```

Estas instrucciones activarán la creación automática de estadísticas:

- SELECT
- INSERT-SELECT
- CTAS
- UPDATE
- Delete
- EXPLAIN, cuando contienen una combinación o se ha detectado la presencia de un predicado.

> [!NOTE]
> La creación automática de estadísticas no funciona en tablas temporales o externas.

La creación automática de estadísticas se realiza de forma sincrónica, por lo que puede suponer una ligera degradación del rendimiento de consulta si a las columnas les faltan estadísticas. El tiempo para crear estadísticas de una sola columna depende del tamaño de la tabla.

Para evitar la degradación del rendimiento cuantificable, debe asegurarse de que antes se han creado estadísticas mediante la ejecución de la carga de trabajo de la prueba comparativa antes de la generación de perfiles del sistema.

> [!NOTE]
> La creación de estadísticas también se registrará en [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) en un contexto de usuario diferente.

Cuando se crean las estadísticas automáticas, estas adoptarán la forma de: _WA_Sys_<identificador de columna de 8 dígitos en hexadecimal>_<identificador de tabla de 8 dígitos en hexadecimal>. Para ver las estadísticas que ya se han creado, ejecute el comando [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest):

```sql
DBCC SHOW_STATISTICS (<table_name>, <target>)
```

El valor de table_name es el nombre de la tabla que contiene las estadísticas para mostrar. La tabla no puede ser externa. El destino es el nombre del índice, de las estadísticas o de la columna de destino para los que se va a mostrar información estadística.

## <a name="update-statistics"></a>Actualizar estadísticas

Uno de los procedimientos recomendados es la actualización diaria de estadísticas en columnas de fecha al agregarse nuevas fechas. Cada vez que se cargan nuevas filas en el grupo de SQL, se agregan nuevas fechas de carga o de transacción. Estas incorporaciones cambian la distribución de los datos y hacen que las estadísticas se queden obsoletas.

Es posible que las estadísticas de una columna de países o regiones en una tabla de clientes nunca tengan que actualizarse, ya que la distribución de valores no suele cambiar. Suponiendo que la distribución es constante entre los clientes, agregar nuevas filas a la variación de tabla no va a cambiar la distribución de datos.

Sin embargo, si su grupo de SQL solo contiene un país o región y trae datos de un nuevo país o región, lo que provoca que se almacenen datos de varios países o regiones, debe actualizar las estadísticas en la columna de país o región.

Las siguientes son recomendaciones para actualizar las estadísticas:

|||
|-|-|
| **Frecuencia de actualizaciones de estadísticas**  | Conservadora: Diario </br> Después de cargar o transformar los datos |
| **Muestreo** |  Con menos de mil millones de filas, use el muestreo predeterminado (20 por ciento) </br> Con más de mil millones de filas, use el muestreo del dos por ciento. |

Una de las primeras preguntas que se deben formular para la solución de problemas de una consulta es **"¿Están actualizadas las estadísticas?"**

No se trata de una pregunta que se pueda responder por la antigüedad de los datos. Un objeto de estadísticas actualizadas podría ser antiguo si no ha habido ningún cambio material en los datos subyacentes. Si el número de filas ha cambiado significativamente o hay un cambio material en la distribución de valores para una columna, *entonces* es el momento de actualizar las estadísticas. 

No hay una vista de administración dinámica para determinar si los datos de la tabla han cambiado desde la última vez que se actualizaron las estadísticas.  Las dos consultas siguientes pueden ayudarle a determinar si las estadísticas están obsoletas.

**Consulta 1**:  averiguar la diferencia entre el recuento de filas de las estadísticas (**stats_row_count**) y el recuento de filas real (**actual_row_count**). 

```sql
select 
objIdsWithStats.[object_id], 
actualRowCounts.[schema], 
actualRowCounts.logical_table_name, 
statsRowCounts.stats_row_count, 
actualRowCounts.actual_row_count,
row_count_difference = CASE
    WHEN actualRowCounts.actual_row_count >= statsRowCounts.stats_row_count THEN actualRowCounts.actual_row_count - statsRowCounts.stats_row_count
    ELSE statsRowCounts.stats_row_count - actualRowCounts.actual_row_count
END,
percent_deviation_from_actual = CASE
    WHEN actualRowCounts.actual_row_count = 0 THEN statsRowCounts.stats_row_count
    WHEN statsRowCounts.stats_row_count = 0 THEN actualRowCounts.actual_row_count
    WHEN actualRowCounts.actual_row_count >= statsRowCounts.stats_row_count THEN CONVERT(NUMERIC(18, 0), CONVERT(NUMERIC(18, 2), (actualRowCounts.actual_row_count - statsRowCounts.stats_row_count)) / CONVERT(NUMERIC(18, 2), actualRowCounts.actual_row_count) * 100)
    ELSE CONVERT(NUMERIC(18, 0), CONVERT(NUMERIC(18, 2), (statsRowCounts.stats_row_count - actualRowCounts.actual_row_count)) / CONVERT(NUMERIC(18, 2), actualRowCounts.actual_row_count) * 100)
END
from
(
    select distinct object_id from sys.stats where stats_id > 1
) objIdsWithStats
left join
(
    select object_id, sum(rows) as stats_row_count from sys.partitions group by object_id
) statsRowCounts
on objIdsWithStats.object_id = statsRowCounts.object_id 
left join
(
    SELECT sm.name [schema] ,
    tb.name logical_table_name ,
    tb.object_id object_id ,
    SUM(rg.row_count) actual_row_count
    FROM sys.schemas sm
    INNER JOIN sys.tables tb ON sm.schema_id = tb.schema_id
    INNER JOIN sys.pdw_table_mappings mp ON tb.object_id = mp.object_id
    INNER JOIN sys.pdw_nodes_tables nt ON nt.name = mp.physical_name
    INNER JOIN sys.dm_pdw_nodes_db_partition_stats rg
    ON rg.object_id = nt.object_id
    AND rg.pdw_node_id = nt.pdw_node_id
    AND rg.distribution_id = nt.distribution_id
    WHERE 1 = 1
    GROUP BY sm.name, tb.name, tb.object_id
) actualRowCounts
on objIdsWithStats.object_id = actualRowCounts.object_id

```

>[!TIP]
> Para mejorar el rendimiento de Synapse SQL, considere la posibilidad de usar **sys.pdw_permanent_table_mappings** en lugar de **sys.pdw_table_mappings** en las tablas de usuario permanentes. Para obtener más información, consulte **[sys.pdw_permanent_table_mappings &#40;Transact-SQL&#41;](/sql/relational-databases/system-catalog-views/sys-pdw-permanent-table-mappings-transact-sql?view=azure-sqldw-latest)** .

**Consulta 2**: averiguar la antigüedad de las estadísticas comprobando la última vez que se actualizaron las estadísticas en cada tabla. 

> [!NOTE]
> Recuerde que, si hay un cambio material en la distribución de valores para una columna, debe actualizar las estadísticas independientemente de la última vez que se actualizaran.

```sql
SELECT
    sm.[name] AS [schema_name],
    tb.[name] AS [table_name],
    co.[name] AS [stats_column_name],
    st.[name] AS [stats_name],
    STATS_DATE(st.[object_id],st.[stats_id]) AS [stats_last_updated_date]
FROM
    sys.objects ob
    JOIN sys.stats st
        ON  ob.[object_id] = st.[object_id]
    JOIN sys.stats_columns sc
        ON  st.[stats_id] = sc.[stats_id]
        AND st.[object_id] = sc.[object_id]
    JOIN sys.columns co
        ON  sc.[column_id] = co.[column_id]
        AND sc.[object_id] = co.[object_id]
    JOIN sys.types  ty
        ON  co.[user_type_id] = ty.[user_type_id]
    JOIN sys.tables tb
        ON  co.[object_id] = tb.[object_id]
    JOIN sys.schemas sm
        ON  tb.[schema_id] = sm.[schema_id]
WHERE
    st.[user_created] = 1;
```

Por ejemplo, las **columnas de fecha** de un grupo de SQL normalmente necesitan que las estadísticas se actualicen con frecuencia. Cada vez que se cargan nuevas filas en el grupo de SQL, se agregan nuevas fechas de carga o de transacción. Estas incorporaciones cambian la distribución de los datos y hacen que las estadísticas se queden obsoletas.

Por el contrario, es posible que las estadísticas en una columna de género de una tabla de clientes no necesiten actualizarse nunca. Suponiendo que la distribución es constante entre los clientes, agregar nuevas filas a la variación de tabla no va a cambiar la distribución de datos.

Si el grupo de SQL contiene solo un género y un nuevo requisito da como resultado varios géneros, tiene que actualizar las estadísticas en la columna de género.

Para más información, consulte [Estadísticas](/sql/relational-databases/statistics/statistics?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) en la guía general.

## <a name="implementing-statistics-management"></a>Implementación de administración de estadísticas

A menudo resulta conveniente extender el proceso de carga de datos para asegurarse de que las estadísticas están actualizadas al final de la carga, con el fin de evitar el bloqueo o la contención de recursos entre consultas simultáneas, o al menos minimizarlos.  

La carga de datos se produce cuando las tablas cambian su tamaño con mayor frecuencia o la distribución de los valores. La carga de datos es un lugar lógico para implementar algunos procesos de administración.

Los siguientes principios fundamentales se proporcionan para actualizar las estadísticas:

- Asegúrese de que cada tabla cargada tiene al menos un objeto de estadísticas actualizado. Así se actualiza la información del tamaño de las tablas (recuento de filas y recuento de páginas) como parte de la actualización de las estadísticas.
- Céntrese en las columnas que participan en las cláusulas JOIN, GROUP BY, ORDER BY y DISTINCT.
- Considere la posibilidad de actualizar con más frecuencia las columnas mediante "clave ascendente", como las fechas de transacción, ya que estos valores no se incluirán en el histograma de estadísticas.
- Considere la posibilidad de actualizar columnas de distribución estática con menor frecuencia.
- Recuerde que cada objeto de estadística se actualiza en secuencia. Implementar solo `UPDATE STATISTICS <TABLE_NAME>` no es siempre recomendable, especialmente para las tablas amplias con muchos objetos de estadísticas.

Para obtener más información, consulte [Estimación de cardinalidad](/sql/relational-databases/performance/cardinality-estimation-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="examples-create-statistics"></a>Ejemplos: Creación de estadísticas

Estos ejemplos muestran cómo utilizar diversas opciones de creación de estadísticas. Las opciones que utiliza para cada columna dependen de las características de los datos y cómo se utilizará la columna en las consultas.

### <a name="create-single-column-statistics-with-default-options"></a>Crear estadísticas de columna única con las opciones predeterminadas

Para crear estadísticas de una columna, especifique un nombre para el objeto de estadística y el nombre de la columna.

Esta sintaxis utiliza todas las opciones predeterminadas. De forma predeterminada, un grupo de SQL muestrea el **20 %** de la tabla cuando crea estadísticas.

```sql
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]);
```

Por ejemplo:

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1);
```

### <a name="create-single-column-statistics-by-examining-every-row"></a>Crear estadísticas de columna única mediante el examen de todas las filas

La velocidad de muestreo predeterminada del 20 % es suficiente para la mayoría de las situaciones. Sin embargo, puede ajustar la velocidad de muestreo.

Para probar la tabla completa, utilice esta sintaxis:

```sql
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]) WITH FULLSCAN;
```

Por ejemplo:

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH FULLSCAN;
```

### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Crear estadísticas de columna única especificando el tamaño de muestra

Como alternativa, se puede especificar el tamaño de muestra en forma de porcentaje:

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH SAMPLE = 50 PERCENT;
```

### <a name="create-single-column-statistics-on-only-some-of-the-rows"></a>Crear estadísticas de columna única solo en algunas filas

También puede crear estadísticas en una parte de las filas de la tabla. A esto se le denomina una estadística filtrada.

Por ejemplo, puede utilizar las estadísticas filtradas si piensa consultar una partición específica de una tabla grande con particiones. Al crear estadísticas basadas únicamente en los valores de partición, mejorará la precisión de las estadísticas y, por tanto, también el rendimiento de consulta.

En este ejemplo se crean estadísticas sobre un intervalo de valores. Los valores pueden definirse fácilmente para que coincidan con el intervalo de valores de una partición.

```sql
CREATE STATISTICS stats_col1 ON table1(col1) WHERE col1 > '2000101' AND col1 < '20001231';
```

> [!NOTE]
> Para que el optimizador de consultas considere utilizar estadísticas filtradas al elegir el plan de consulta distribuida, la consulta debe adecuarse a la definición del objeto de estadísticas. Usando el ejemplo anterior, la cláusula WHERE de la consulta tiene que especificar valores de col1 entre 2 000 101 y 20 001 231.

### <a name="create-single-column-statistics-with-all-the-options"></a>Crear estadísticas de columna única con todas las opciones

También puede combinar las opciones. En el ejemplo siguiente se crea un objeto de estadísticas filtradas con un tamaño personalizado de ejemplo:

```sql
CREATE STATISTICS stats_col1 ON table1 (col1) WHERE col1 > '2000101' AND col1 < '20001231' WITH SAMPLE = 50 PERCENT;
```

Para obtener la referencia completa, consulte [CREATE STATISTICS](/sql/t-sql/statements/create-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

### <a name="create-multi-column-statistics"></a>Crear estadísticas de varias columnas

Para crear un objeto de estadísticas de varias columnas, use los ejemplos anteriores, pero indique más columnas.

> [!NOTE]
> El histograma, que se utiliza para calcular el número de filas en el resultado de la consulta, solo está disponible para la primera columna de la definición del objeto de estadísticas.

En este ejemplo, el histograma se encuentra en *product\_category*. Las estadísticas entre columnas se calculan en *product\_category* y *product\_sub_category*:

```sql
CREATE STATISTICS stats_2cols ON table1 (product_category, product_sub_category) WHERE product_category > '2000101' AND product_category < '20001231' WITH SAMPLE = 50 PERCENT;
```

Dado que no hay una correlación entre *product\_category* y *product\_sub\_category*, un objeto de estadística de varias columnas puede ser útil si se tiene acceso a estas columnas al mismo tiempo.

### <a name="create-statistics-on-all-columns-in-a-table"></a>Creación de estadísticas en todas las columnas de una tabla

Una forma de crear estadísticas consiste en emitir comandos CREATE STATISTICS después de crear la tabla:

```sql
CREATE TABLE dbo.table1
(
   col1 int
,  col2 int
,  col3 int
)
WITH
  (
    CLUSTERED COLUMNSTORE INDEX
  )
;

CREATE STATISTICS stats_col1 on dbo.table1 (col1);
CREATE STATISTICS stats_col2 on dbo.table2 (col2);
CREATE STATISTICS stats_col3 on dbo.table3 (col3);
```

### <a name="use-a-stored-procedure-to-create-statistics-on-all-columns-in-a-database"></a>Utilizar un procedimiento almacenado para crear estadísticas de todas las columnas de una base de datos

Un grupo de SQL no tiene un procedimiento almacenado del sistema equivalente a sp_create_stats en SQL Server. Este procedimiento almacenado crea un objeto de estadísticas de columna única en todas las columnas de la base de datos que ya no tienen estadísticas.

El ejemplo siguiente le ayudará a empezar a trabajar con el diseño de la base de datos. Puede adaptarlo a sus necesidades.

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_create_stats]
(   @create_type    tinyint -- 1 default 2 Fullscan 3 Sample
,   @sample_pct     tinyint
)
AS

IF @create_type IS NULL
BEGIN
    SET @create_type = 1;
END;

IF @create_type NOT IN (1,2,3)
BEGIN
    THROW 151000,'Invalid value for @stats_type parameter. Valid range 1 (default), 2 (fullscan) or 3 (sample).',1;
END;

IF @sample_pct IS NULL
BEGIN;
    SET @sample_pct = 20;
END;

IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN;
    DROP TABLE #stats_ddl;
END;

CREATE TABLE #stats_ddl
WITH    (   DISTRIBUTION    = HASH([seq_nmbr])
        ,   LOCATION        = USER_DB
        )
AS
WITH T
AS
(
SELECT      t.[name]                        AS [table_name]
,           s.[name]                        AS [table_schema_name]
,           c.[name]                        AS [column_name]
,           c.[column_id]                   AS [column_id]
,           t.[object_id]                   AS [object_id]
,           ROW_NUMBER()
            OVER(ORDER BY (SELECT NULL))    AS [seq_nmbr]
FROM        sys.[tables] t
JOIN        sys.[schemas] s         ON  t.[schema_id]       = s.[schema_id]
JOIN        sys.[columns] c         ON  t.[object_id]       = c.[object_id]
LEFT JOIN   sys.[stats_columns] l   ON  l.[object_id]       = c.[object_id]
                                    AND l.[column_id]       = c.[column_id]
                                    AND l.[stats_column_id] = 1
LEFT JOIN    sys.[external_tables] e    ON    e.[object_id]        = t.[object_id]
WHERE       l.[object_id] IS NULL
AND            e.[object_id] IS NULL -- not an external table
)
SELECT  [table_schema_name]
,       [table_name]
,       [column_name]
,       [column_id]
,       [object_id]
,       [seq_nmbr]
,       CASE @create_type
        WHEN 1
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+')' AS VARCHAR(8000))
        WHEN 2
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH FULLSCAN' AS VARCHAR(8000))
        WHEN 3
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH SAMPLE '+CONVERT(varchar(4),@sample_pct)+' PERCENT' AS VARCHAR(8000))
        END AS create_stat_ddl
FROM T
;

DECLARE @i INT              = 1
,       @t INT              = (SELECT COUNT(*) FROM #stats_ddl)
,       @s NVARCHAR(4000)   = N''
;

WHILE @i <= @t
BEGIN
    SET @s=(SELECT create_stat_ddl FROM #stats_ddl WHERE seq_nmbr = @i);

    PRINT @s
    EXEC sp_executesql @s
    SET @i+=1;
END

DROP TABLE #stats_ddl;
```

Para crear estadísticas de todas las columnas de la tabla mediante los valores predeterminados, ejecute el procedimiento almacenado.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 1, NULL;
```

Para crear estadísticas de todas las columnas de la tabla mediante fullscan, llame a este procedimiento.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 2, NULL;
```

Para crear estadísticas muestreadas de todas las columnas de la tabla, escriba 3 y el porcentaje de muestra. Este procedimiento utiliza una frecuencia de muestreo del 20 %.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 3, 20;
```

## <a name="examples-update-statistics"></a>Ejemplos: Actualizar estadísticas

Para actualizar estadísticas, puede:

- Actualizar un objeto de estadísticas. Especificar el nombre del objeto de estadísticas que desea actualizar.
- Actualizar todos los objetos de estadísticas de una tabla. Especificar el nombre de la tabla en lugar de un objeto de estadísticas específico.

### <a name="update-one-specific-statistics-object"></a>Actualizar un objeto de estadísticas específico

Para actualizar un objeto de estadísticas específico, use la siguiente sintaxis:

```sql
UPDATE STATISTICS [schema_name].[table_name]([stat_name]);
```

Por ejemplo:

```sql
UPDATE STATISTICS [dbo].[table1] ([stats_col1]);
```

Al actualizar los objetos de estadísticas específicos, puede minimizar el tiempo y los recursos necesarios para administrar las estadísticas. Para ello es preciso pensar cómo elegir los mejores objetos de estadísticas que se recomienda actualizar.

### <a name="update-all-statistics-on-a-table"></a>Actualizar todas las estadísticas en una tabla

Un método sencillo para actualizar todos los objetos de estadísticas de una tabla es el siguiente:

```sql
UPDATE STATISTICS [schema_name].[table_name];
```

Por ejemplo:

```sql
UPDATE STATISTICS dbo.table1;
```

Es fácil usar la instrucción UPDATE STATISTICS. Solo tiene que recordar que se actualizan *todas* las estadísticas de la tabla y, por tanto, puede realizar más trabajo del necesario. Si el rendimiento no es un problema, es la forma más fácil y completa de garantizar que las estadísticas están actualizadas.

> [!NOTE]
> Al actualizar todas las estadísticas de una tabla, un grupo de SQL realiza un análisis para crear muestras de la tabla para cada objeto de estadística. Si la tabla es grande y tiene muchas columnas y estadísticas, puede resultar más eficaz actualizar las estadísticas individualmente en función de las necesidades.

Para obtener una implementación de un procedimiento `UPDATE STATISTICS`, consulte [Tablas temporales](sql-data-warehouse-tables-temporary.md). El método de implementación difiere ligeramente del procedimiento `CREATE STATISTICS`, pero el resultado es el mismo.

Para obtener la sintaxis completa, consulte [UPDATE STATISTICS](/sql/t-sql/statements/update-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="statistics-metadata"></a>Metadatos de las estadísticas

Hay varias funciones y vistas del sistema que puede usar para encontrar información sobre las estadísticas. Por ejemplo, puede ver si un objeto de estadísticas podría estar obsoleto usando la función stats-date para consultar la fecha en que las estadísticas se crearon o actualizaron por última vez.

### <a name="catalog-views-for-statistics"></a>Vistas de catálogo para las estadísticas

Estas vistas del sistema proporcionan información acerca de las estadísticas:

| Vista de catálogo | Descripción |
|:--- |:--- |
| [sys.columns](/sql/relational-databases/system-catalog-views/sys-columns-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada columna. |
| [sys.objects](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada objeto de la base de datos. |
| [sys.schemas](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada esquema de la base de datos. |
| [sys.stats](/sql/relational-databases/system-catalog-views/sys-stats-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada objeto de estadísticas. |
| [sys.stats_columns](/sql/relational-databases/system-catalog-views/sys-stats-columns-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada columna del objeto de estadísticas. Vínculos a la sys.columns. |
| [sys.tables](/sql/relational-databases/system-catalog-views/sys-tables-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada tabla (incluye tablas externas). |
| [sys.table_types](/sql/relational-databases/system-catalog-views/sys-table-types-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada tipo de datos. |

### <a name="system-functions-for-statistics"></a>Funciones del sistema para las estadísticas

Estas funciones del sistema son útiles para trabajar con las estadísticas:

| Función del sistema | Descripción |
|:--- |:--- |
| [STATS_DATE](/sql/t-sql/functions/stats-date-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Fecha en que se actualizó por última vez el objeto de estadísticas. |
| [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Información resumida y detallada acerca de la distribución de valores según lo entiende el objeto de estadísticas. |

### <a name="combine-statistics-columns-and-functions-into-one-view"></a>Combinar funciones y columnas de estadísticas en una vista

Esta vista agrupa las columnas relacionadas con las estadísticas y los resultados de la función STATS_DATE().

```sql
CREATE VIEW dbo.vstats_columns
AS
SELECT
        sm.[name]                           AS [schema_name]
,       tb.[name]                           AS [table_name]
,       st.[name]                           AS [stats_name]
,       st.[filter_definition]              AS [stats_filter_definition]
,       st.[has_filter]                     AS [stats_is_filtered]
,       STATS_DATE(st.[object_id],st.[stats_id])
                                            AS [stats_last_updated_date]
,       co.[name]                           AS [stats_column_name]
,       ty.[name]                           AS [column_type]
,       co.[max_length]                     AS [column_max_length]
,       co.[precision]                      AS [column_precision]
,       co.[scale]                          AS [column_scale]
,       co.[is_nullable]                    AS [column_is_nullable]
,       co.[collation_name]                 AS [column_collation_name]
,       QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                            AS two_part_name
,       QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                            AS three_part_name
FROM    sys.objects                         AS ob
JOIN    sys.stats           AS st ON    ob.[object_id]      = st.[object_id]
JOIN    sys.stats_columns   AS sc ON    st.[stats_id]       = sc.[stats_id]
                            AND         st.[object_id]      = sc.[object_id]
JOIN    sys.columns         AS co ON    sc.[column_id]      = co.[column_id]
                            AND         sc.[object_id]      = co.[object_id]
JOIN    sys.types           AS ty ON    co.[user_type_id]   = ty.[user_type_id]
JOIN    sys.tables          AS tb ON  co.[object_id]        = tb.[object_id]
JOIN    sys.schemas         AS sm ON  tb.[schema_id]        = sm.[schema_id]
WHERE   1=1
AND     st.[user_created] = 1
;
```

## <a name="dbcc-show_statistics-examples"></a>Ejemplos de DBCC SHOW_STATISTICS()

DBCC SHOW_STATISTICS() muestra los datos contenidos en un objeto de estadísticas. Estos datos se presentan en tres partes:

- Encabezado
- Vector de densidad
- Histograma

Los metadatos de encabezado sobre las estadísticas. El histograma muestra la distribución de valores en la primera columna de clave del objeto de estadísticas. El vector de densidad mide la correlación entre las columnas.

> [!NOTE]
> Un grupo de SQL calcula las estimaciones de cardinalidad con cualquiera de los datos en el objeto de estadísticas.

### <a name="show-header-density-and-histogram"></a>Mostrar el encabezado, la densidad y el histograma

Este ejemplo muestra las tres partes de un objeto de estadísticas:

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
```

Por ejemplo:

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1);
```

### <a name="show-one-or-more-parts-of-dbcc-show_statistics"></a>Mostrar una o varias partes de DBCC SHOW_STATISTICS()

Si solo está interesado en ver partes específicas, use la cláusula `WITH` y especifique qué partes desea ver:

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>) WITH stat_header, histogram, density_vector
```

Por ejemplo:

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1) WITH histogram, density_vector
```

## <a name="dbcc-show_statistics-differences"></a>Diferencias de DBCC SHOW_STATISTICS()

DBCC SHOW_STATISTICS() se implementa de forma más estricta en un grupo de SQL en comparación con SQL Server:

- No se admiten las características no documentadas.
- No se puede usar Stats_stream.
- No se pueden combinar resultados de subconjuntos específicos de datos de estadísticas. Por ejemplo, STAT_HEADER JOIN DENSITY_VECTOR.
- No se puede establecer NO_INFOMSGS para la supresión de mensajes.
- No se pueden usar corchetes alrededor de los nombres de las estadísticas.
- No se pueden usar nombres de columna para identificar objetos de estadísticas.
- No se admite el error personalizado 2767.

## <a name="next-steps"></a>Pasos siguientes

Para mejorar aún más el rendimiento de las consultas, vea [Supervisión de la carga de trabajo](sql-data-warehouse-manage-monitor.md).
