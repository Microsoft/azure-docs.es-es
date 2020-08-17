---
title: Creación y actualización de estadísticas mediante recursos de Azure Synapse SQL
description: Recomendaciones y ejemplos para crear y actualizar las estadísticas de optimización de consultas en SQL de Synapse.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 04/19/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.custom: ''
ms.openlocfilehash: 25c92437b350d7329c340fe1ea13b3df40e231ba
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87020606"
---
# <a name="statistics-in-synapse-sql"></a>Estadísticas en SQL de Synapse

En este artículo se proporcionan recomendaciones y ejemplos para crear y actualizar las estadísticas de optimización de consultas mediante los siguientes recursos SQL de Synapse: Grupo de SQL y SQL a petición (versión preliminar).

## <a name="statistics-in-sql-pool"></a>Estadísticas en el grupo de SQL

### <a name="why-use-statistics"></a>¿Por qué usar estadísticas?

Cuanto más sepa el recurso del grupo de SQL acerca de los datos, más rápido puede ejecutar consultas. Después de cargar los datos en el grupo de SQL, la recopilación de estadísticas de los datos es una de las cosas más importantes que puede hacer para la optimización de consultas.  

El optimizador de consultas del grupo de SQL está basado en el costo. Compara el costo de varios planes de consulta y elige el menor de ellos. En la mayoría de los casos, elige el plan que se ejecutará más rápidamente.

Por ejemplo, si el optimizador considera que la fecha en que se filtra la consulta devolverá una fila, se elegirá un plan. Si estima que la fecha seleccionada devolverá 1 millón de filas, devolverá un plan diferente.

### <a name="automatic-creation-of-statistics"></a>Creación automática de estadísticas

El grupo de SQL analizará las consultas entrantes de los usuarios para buscar las estadísticas que faltan cuando la opción AUTO_CREATE_STATISTICS de la base de datos esté establecida en `ON`.  Si faltan estadísticas, el optimizador de consultas crea las estadísticas en columnas individuales del predicado de consulta o en la condición de combinación. 

Esta función se usa para mejorar las estimaciones de cardinalidad del plan de consulta.

> [!IMPORTANT]
> Actualmente, la creación automática de estadísticas está activada de forma predeterminada.

Puede comprobar si su almacenamiento de datos tiene configurado AUTO_CREATE_STATISTICS; para ello, ejecute el siguiente comando:

```sql
SELECT name, is_auto_create_stats_on
FROM sys.databases
```

Si el almacenamiento de datos no tiene habilitada la opción AUTO_CREATE_STATISTICS, se recomienda habilitar esta propiedad con el siguiente comando:

```sql
ALTER DATABASE <yourdatawarehousename>
SET AUTO_CREATE_STATISTICS ON
```

Estas instrucciones desencadenarán la creación automática de estadísticas:

- SELECT
- INSERT-SELECT
- CTAS
- UPDATE
- Delete
- EXPLAIN, cuando contienen una combinación o se ha detectado la presencia de un predicado.

> [!NOTE]
> La creación automática de estadísticas no se genera en las tablas temporales o externas.

En cambio, se realiza sincrónicamente. Por lo tanto, puede generar una ligera degradación del rendimiento de consulta si a las columnas les faltan estadísticas. El tiempo para crear estadísticas de una sola columna depende del tamaño de la tabla.

Para evitar la degradación del rendimiento cuantificable, debe asegurarse de que antes se han creado estadísticas. Para ello, ejecute la carga de trabajo de la prueba comparativa antes de generar los perfiles del sistema.

> [!NOTE]
> La creación de estadísticas también se registra en [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) en un contexto de usuario diferente.

Cuando se crean las estadísticas automáticas, estas adoptan la siguiente forma: _WA_Sys_<identificador de columna de 8 dígitos en hexadecimal>_<identificador de tabla de 8 dígitos en hexadecimal>. Para ver las estadísticas ya creadas, ejecute el comando [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest):

```sql
DBCC SHOW_STATISTICS (<table_name>, <target>)
```

El valor de table_name es el nombre de la tabla que contiene las estadísticas que se mostrarán, y no puede ser una tabla externa. El destino es el nombre del índice, de las estadísticas o de la columna de destino para los que se va a mostrar información estadística.

### <a name="update-statistics"></a>Actualizar estadísticas

Uno de los procedimientos recomendados es la actualización diaria de estadísticas en columnas de fecha al agregarse nuevas fechas. Cada fila nueva de tiempo se carga en el almacenamiento de datos, se agregan nuevas fechas de carga o de transacción. Estas incorporaciones cambian la distribución de los datos y hacen que las estadísticas se queden obsoletas.

Es posible que las estadísticas de una columna de un país o región en una tabla de clientes nunca tengan que actualizarse, pues la distribución de valores no suele cambiar. Suponiendo que la distribución es constante entre los clientes, agregar nuevas filas a la variación de tabla no va a cambiar la distribución de datos.

Sin embargo, cuando el almacenamiento de datos solo contiene un país o región y trae datos de uno nuevo, debe actualizar las estadísticas en la columna de dicho país o región.

Las siguientes son recomendaciones para actualizar las estadísticas:

|||
|-|-|
| **Frecuencia de actualizaciones de estadísticas**  | Conservadora: Diario </br> Después de cargar o transformar los datos |
| **Muestreo** |  Con menos de mil millones de filas, use el muestreo predeterminado (20 por ciento) </br> Con más de mil millones de filas, use el muestreo del dos por ciento. |

### <a name="determine-last-statistics-update"></a>Determinación de la última actualización de estadísticas

Una de las primeras preguntas que se deben formular para la solución de problemas de una consulta es **"¿Están actualizadas las estadísticas?"**

No se trata de una pregunta que se pueda responder por la antigüedad de los datos. Un objeto de estadísticas actualizadas podría ser antiguo si no ha habido ningún cambio material en los datos subyacentes. Si el número de filas ha cambiado significativamente o se produce un cambio material en la distribución de valores para una columna, *entonces* es el momento de actualizar las estadísticas.

No hay disponible una vista de administración dinámica para determinar si los datos de la tabla han cambiado desde la última vez que se actualizaron las estadísticas. Conocer la antigüedad de sus estadísticas puede proporcionarle parte de la totalidad de la imagen. 

Puede usar la siguiente consulta para determinar la última vez que se actualizaron las estadísticas en cada tabla.

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

Las **columnas de fecha** en un almacenamiento de datos, por ejemplo, normalmente necesitan que las estadísticas se actualicen con frecuencia. Cada fila nueva de tiempo se carga en el almacenamiento de datos, se agregan nuevas fechas de carga o de transacción. Estas incorporaciones cambian la distribución de los datos y hacen que las estadísticas se queden obsoletas.

Es posible que las estadísticas en una columna de género de una tabla de clientes no necesiten actualizarse nunca. Suponiendo que la distribución es constante entre los clientes, agregar nuevas filas a la variación de tabla no va a cambiar la distribución de datos.

No obstante, si el almacén de datos contiene solo un género y un nuevo requisito da como resultado varios géneros, tiene que actualizar las estadísticas en la columna de género. 

Para obtener más información, consulte el artículo [Estadísticas](/sql/relational-databases/statistics/statistics).

### <a name="implement-statistics-management"></a>Implementación de la administración de estadísticas

A menudo resulta conveniente extender el proceso de carga de datos para garantizar que las estadísticas están actualizadas al final de la carga. La carga de datos se produce cuando las tablas cambian su tamaño o la distribución de los valores con mayor frecuencia. Por lo tanto, el proceso de carga es un lugar lógico para implementar algunos procesos de administración.

Los siguientes principios fundamentales se proporcionan para actualizar las estadísticas durante el proceso de carga:

- Asegúrese de que cada tabla cargada tiene al menos un objeto de estadísticas actualizado. Este proceso actualiza la información del tamaño de las tablas (recuento de filas y recuento de páginas) como parte de la actualización de las estadísticas.
- Céntrese en las columnas que participan en las cláusulas JOIN, GROUP BY, ORDER BY y DISTINCT.
- Considere la posibilidad de actualizar con más frecuencia las columnas de "clave ascendente", como las fechas de transacción, ya que estos valores no se incluirán en el histograma de estadísticas.
- Considere la posibilidad de actualizar columnas de distribución estática con menor frecuencia.
- Recuerde que cada objeto de estadística se actualiza en secuencia. Implementar solo `UPDATE STATISTICS <TABLE_NAME>` no es siempre recomendable, especialmente para las tablas amplias con muchos objetos de estadísticas.

Para obtener más información, consulte [Estimación de cardinalidad](/sql/relational-databases/performance/cardinality-estimation-sql-server).

### <a name="examples-create-statistics"></a>Ejemplos: Creación de estadísticas

Estos ejemplos muestran cómo utilizar diversas opciones de creación de estadísticas. Las opciones que utiliza para cada columna dependen de las características de los datos y cómo se utilizará la columna en las consultas.

#### <a name="create-single-column-statistics-with-default-options"></a>Crear estadísticas de columna única con las opciones predeterminadas

Para crear estadísticas en una columna, especifique un nombre para el objeto de estadística y el nombre de la columna.
Esta sintaxis utiliza todas las opciones predeterminadas. De forma predeterminada, un grupo de SQL muestrea el **20 %** de la tabla cuando crea estadísticas.

```sql
CREATE STATISTICS [statistics_name]
    ON [schema_name].[table_name]([column_name]);
```

Por ejemplo:

```sql
CREATE STATISTICS col1_stats
    ON dbo.table1 (col1);
```

#### <a name="create-single-column-statistics-by-examining-every-row"></a>Crear estadísticas de columna única mediante el examen de todas las filas

La velocidad de muestreo predeterminada del 20 % es suficiente para la mayoría de las situaciones. Sin embargo, puede ajustar la velocidad de muestreo. Para probar la tabla completa, utilice esta sintaxis:

```sql
CREATE STATISTICS [statistics_name]
    ON [schema_name].[table_name]([column_name])
    WITH FULLSCAN;
```

Por ejemplo:

```sql
CREATE STATISTICS col1_stats
    ON dbo.table1 (col1)
    WITH FULLSCAN;
```

#### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Crear estadísticas de columna única especificando el tamaño de muestra

Otra opción consiste en especificar el tamaño del ejemplo como un porcentaje:

```sql
CREATE STATISTICS col1_stats
    ON dbo.table1 (col1)
    WITH SAMPLE = 50 PERCENT;
```

#### <a name="create-single-column-statistics-on-only-some-of-the-rows"></a>Crear estadísticas de columna única solo en algunas filas

También puede crear estadísticas en una parte de las filas de la tabla, que se denomina estadística filtrada.

Por ejemplo, puede utilizar las estadísticas filtradas si piensa consultar una partición específica de una tabla grande con particiones. Al crear estadísticas basadas únicamente en los valores de partición, mejorará la precisión de las estadísticas. Ademas, experimentará una mejora en el rendimiento de las consultas.

En este ejemplo se crean estadísticas sobre un intervalo de valores. Los valores pueden definirse fácilmente para que coincidan con el intervalo de valores de una partición.

```sql
CREATE STATISTICS stats_col1
    ON table1(col1)
    WHERE col1 > '2000101' AND col1 < '20001231';
```

> [!NOTE]
> Para que el optimizador de consultas considere utilizar estadísticas filtradas al elegir el plan de consulta distribuida, la consulta debe adecuarse a la definición del objeto de estadísticas. Usando el ejemplo anterior, la cláusula WHERE de la consulta tiene que especificar valores de col1 entre 2 000 101 y 20 001 231.

#### <a name="create-single-column-statistics-with-all-the-options"></a>Crear estadísticas de columna única con todas las opciones

También puede combinar las opciones. En el ejemplo siguiente se crea un objeto de estadísticas filtradas con un tamaño personalizado de ejemplo:

```sql
CREATE STATISTICS stats_col1
    ON table1 (col1)
    WHERE col1 > '2000101' AND col1 < '20001231'
    WITH SAMPLE = 50 PERCENT;
```

Para obtener la referencia completa, consulte [CREATE STATISTICS](/sql/t-sql/statements/create-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

#### <a name="create-multi-column-statistics"></a>Crear estadísticas de varias columnas

Para crear un objeto de estadísticas de varias columnas, use los ejemplos anteriores, pero indique más columnas.

> [!NOTE]
> El histograma, que se utiliza para calcular el número de filas en el resultado de la consulta, solo está disponible para la primera columna de la definición del objeto de estadísticas.

En este ejemplo, el histograma se encuentra en *product\_category*. Las estadísticas entre columnas se calculan en *product\_category* y *product\_sub_category*:

```sql
CREATE STATISTICS stats_2cols
    ON table1 (product_category, product_sub_category)
    WHERE product_category > '2000101' AND product_category < '20001231'
    WITH SAMPLE = 50 PERCENT;
```

Dado que no existe una correlación entre *product\_category* y *product\_sub\_category*, un objeto de estadística de varias columnas puede ser útil si se tiene acceso a estas columnas al mismo tiempo.

#### <a name="create-statistics-on-all-columns-in-a-table"></a>Creación de estadísticas en todas las columnas de una tabla

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

#### <a name="use-a-stored-procedure-to-create-statistics-on-all-columns-in-a-database"></a>Utilizar un procedimiento almacenado para crear estadísticas de todas las columnas de una base de datos

Un grupo de SQL no tiene un procedimiento almacenado del sistema equivalente a sp_create_stats en SQL Server. Este procedimiento almacenado crea un objeto de estadísticas de columna única en todas las columnas de la base de datos que ya no tienen estadísticas.

El ejemplo siguiente le ayudará a empezar a trabajar con el diseño de la base de datos. Puede adaptarlo a sus necesidades:

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_create_stats]
(   @create_type    tinyint -- 1 default, 2 Fullscan, 3 Sample
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

Para crear estadísticas de todas las columnas de la tabla mediante fullscan, llame al procedimiento:

```sql
EXEC [dbo].[prc_sqldw_create_stats] 2, NULL;
```

Para crear estadísticas muestreadas de todas las columnas de la tabla, escriba 3 y el porcentaje de muestra. El siguiente procedimiento usa una frecuencia de muestreo del 20 %.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 3, 20;
```

### <a name="examples-update-statistics"></a>Ejemplos: Actualizar estadísticas

Para actualizar estadísticas, puede:

- Actualizar un objeto de estadísticas. Especificar el nombre del objeto de estadísticas que desea actualizar.
- Actualizar todos los objetos de estadísticas de una tabla. Especificar el nombre de la tabla en lugar de un objeto de estadísticas específico.

#### <a name="update-one-specific-statistics-object"></a>Actualizar un objeto de estadísticas específico

Para actualizar un objeto de estadísticas específico, use la siguiente sintaxis:

```sql
UPDATE STATISTICS [schema_name].[table_name]([stat_name]);
```

Por ejemplo:

```sql
UPDATE STATISTICS [dbo].[table1] ([stats_col1]);
```

Al actualizar los objetos de estadísticas específicos, puede minimizar el tiempo y los recursos necesarios para administrar las estadísticas. Esta acción requiere pensar detenidamente cómo se seleccionarán los mejores objetos de estadísticas que se recomienda actualizar.

#### <a name="update-all-statistics-on-a-table"></a>Actualizar todas las estadísticas en una tabla

Un método sencillo para actualizar todos los objetos de estadísticas de una tabla es el siguiente:

```sql
UPDATE STATISTICS [schema_name].[table_name];
```

Por ejemplo:

```sql
UPDATE STATISTICS dbo.table1;
```

Es fácil usar la instrucción UPDATE STATISTICS. Solo tiene que recordar que se actualizan *todas* las estadísticas de la tabla, por lo que se pide más trabajo del necesario. 

Si el rendimiento no es un problema, este método es la forma más fácil y completa de garantizar que las estadísticas están actualizadas.

> [!NOTE]
> Al actualizar todas las estadísticas de una tabla, un grupo de SQL realiza un análisis para crear muestras de la tabla para cada objeto de estadística. Si la tabla es grande y tiene muchas columnas y estadísticas, puede resultar más eficaz actualizar las estadísticas individualmente en función de las necesidades.

Para ver una implementación del procedimiento `UPDATE STATISTICS`, consulte [Tablas temporales](develop-tables-temporary.md). El método de implementación difiere ligeramente del procedimiento `CREATE STATISTICS`, pero el resultado es el mismo.
Para ver la sintaxis completa, consulte [UPDATE STATISTICS](/sql/t-sql/statements/update-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

### <a name="statistics-metadata"></a>Metadatos de las estadísticas

Hay varias funciones y vistas del sistema que puede usar para encontrar información sobre las estadísticas. Por ejemplo, puede ver si un objeto de estadísticas podría estar obsoleto mediante la función STATS_DATE(). STATS_DATE() le permite ver cuándo se crearon o actualizaron las estadísticas por última vez.

#### <a name="catalog-views-for-statistics"></a>Vistas de catálogo para las estadísticas

Estas vistas del sistema proporcionan información acerca de las estadísticas:

| Vista de catálogo | Descripción |
|:--- |:--- |
| [sys.columns](/sql/relational-databases/system-catalog-views/sys-columns-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada columna. |
| [sys.objects](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada objeto de la base de datos. |
| [sys.schemas](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada esquema de la base de datos. |
| [sys.stats](/sql/relational-databases/system-catalog-views/sys-stats-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada objeto de estadísticas. |
| [sys.stats_columns](/sql/relational-databases/system-catalog-views/sys-stats-columns-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada columna del objeto de estadísticas. Vínculos a la sys.columns. |
| [sys.tables](/sql/relational-databases/system-catalog-views/sys-tables-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada tabla (incluye tablas externas). |
| [sys.table_types](/sql/relational-databases/system-catalog-views/sys-table-types-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) |Una fila por cada tipo de datos. |

#### <a name="system-functions-for-statistics"></a>Funciones del sistema para las estadísticas

Estas funciones del sistema son útiles para trabajar con las estadísticas:

| Función del sistema | Descripción |
|:--- |:--- |
| [STATS_DATE](/sql/t-sql/functions/stats-date-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) |Fecha en que se actualizó por última vez el objeto de estadísticas. |
| [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) |Información resumida y detallada acerca de la distribución de valores según lo entiende el objeto de estadísticas. |

#### <a name="combine-statistics-columns-and-functions-into-one-view"></a>Combinar funciones y columnas de estadísticas en una vista

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
JOIN    sys.tables          AS tb ON    co.[object_id]      = tb.[object_id]
JOIN    sys.schemas         AS sm ON    tb.[schema_id]      = sm.[schema_id]
WHERE   1=1
AND     st.[user_created] = 1
;
```

### <a name="dbcc-show_statistics-examples"></a>Ejemplos de DBCC SHOW_STATISTICS()

DBCC SHOW_STATISTICS() muestra los datos contenidos en un objeto de estadísticas. Estos datos se presentan en tres partes:

- Encabezado
- Vector de densidad
- Histograma

El encabezado contiene los metadatos sobre las estadísticas. El histograma muestra la distribución de valores en la primera columna de clave del objeto de estadísticas. 

El vector de densidad mide la correlación entre las columnas. Un grupo de SQL calcula las estimaciones de cardinalidad con cualquiera de los datos en el objeto de estadísticas.

#### <a name="show-header-density-and-histogram"></a>Mostrar el encabezado, la densidad y el histograma

Este ejemplo muestra las tres partes de un objeto de estadísticas:

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
```

Por ejemplo:

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1);
```

#### <a name="show-one-or-more-parts-of-dbcc-show_statistics"></a>Mostrar una o varias partes de DBCC SHOW_STATISTICS()

Si solo está interesado en ver partes específicas, use la cláusula `WITH` y especifique qué partes desea ver:

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
    WITH stat_header, histogram, density_vector
```

Por ejemplo:

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1)
    WITH histogram, density_vector
```

### <a name="dbcc-show_statistics-differences"></a>Diferencias de DBCC SHOW_STATISTICS()

En comparación, `DBCC SHOW_STATISTICS()` se implementa de forma más estricta en un grupo de SQL que en SQL Server:

- No se admiten las características no documentadas.
- No se puede usar Stats_stream.
- No se pueden combinar los resultados de subconjuntos específicos de datos de estadísticas. Por ejemplo, STAT_HEADER JOIN DENSITY_VECTOR.
- No se puede establecer NO_INFOMSGS para la eliminación de mensajes.
- No se pueden usar corchetes alrededor de los nombres de las estadísticas.
- No se pueden usar nombres de columna para identificar objetos de estadísticas.
- No se admite el error personalizado 2767.

### <a name="next-steps"></a>Pasos siguientes

Para mejorar aún más el rendimiento de las consultas, vea [Supervisión de la carga de trabajo](../sql-data-warehouse/sql-data-warehouse-manage-monitor.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="statistics-in-sql-on-demand-preview"></a>Estadísticas en SQL a petición (versión preliminar)

Las estadísticas se crean por cada columna concreta en un conjunto de datos determinado (ruta de acceso de almacenamiento).

### <a name="why-use-statistics"></a>¿Por qué usar estadísticas?

Cuanto más sepa SQL a petición (versión preliminar) acerca de los datos, más rápido puede ejecutar consultas en ellos. La recopilación de estadísticas sobre los datos es una de las tareas más importantes que se pueden realizar para optimizar las consultas. 

El optimizador de consultas de SQL a petición está basado en el costo. Compara el costo de varios planes de consulta y elige el menor de ellos. En la mayoría de los casos, elige el plan que se ejecutará más rápidamente. 

Por ejemplo, si el optimizador estima que la fecha en que se filtra la consulta devolverá una fila, se elegirá un plan. Si estima que la fecha seleccionada devolverá 1 millón de filas, devolverá un plan diferente.

### <a name="automatic-creation-of-statistics"></a>Creación automática de estadísticas

SQL a petición analiza las consultas de usuario entrantes para buscar las estadísticas que faltan. Si faltan las estadísticas, el optimizador de consultas crea las estadísticas en columnas individuales en el predicado de consulta o en la condición de combinación para mejorar las estimaciones de cardinalidad del plan de consulta.

La instrucción SELECT activará la creación automática de estadísticas.

> [!NOTE]
> La creación automática de estadísticas está activada para los archivos Parquet. En el caso de los archivos CSV, debe crear las estadísticas manualmente hasta que se admita la creación automática de estadísticas para archivos CSV.

La creación automática de estadísticas se realiza de forma sincrónica, por lo que puede suponer una ligera degradación del rendimiento de consulta si a las columnas les faltan estadísticas. El tiempo necesario para crear estadísticas de una sola columna depende del tamaño de los archivos de destino.

### <a name="manual-creation-of-statistics"></a>Creación manual de estadísticas

SQL a petición le permite crear estadísticas de forma manual. En el caso de los archivos CSV, debe crear las estadísticas manualmente, ya que la creación automática de estadísticas no está activada para los archivos CSV. 

Consulte los siguientes ejemplos para obtener instrucciones sobre cómo crear estadísticas manualmente.

### <a name="update-statistics"></a>Actualizar estadísticas

Los cambios en los datos de los archivos, así como la eliminación y adición de archivos, producen cambios en la distribución de datos y hace que las estadísticas no estén actualizadas. En ese caso, es necesario actualizar las estadísticas.

SQL a petición vuelve a crear automáticamente las estadísticas si se cambian los datos de forma significativa. Cada vez que se crean estadísticas automáticamente, también se guarda el estado actual del conjunto de datos: las rutas de acceso, los tamaños y las fechas de última modificación de los archivos.

Cuando las estadísticas estén obsoletas, se crean otras nuevas. El algoritmo recorre los datos y los compara con el estado actual del conjunto de datos. Si el tamaño de los cambios supera el umbral especificado, las estadísticas anteriores se eliminan y se vuelven a crear en el nuevo conjunto de información.

Las estadísticas manuales nunca se declaran obsoletas.

> [!NOTE]
> La opción de volver a crear automáticamente las estadísticas está activada para los archivos Parquet. En el caso de los archivos CSV, debe quitar y crear estadísticas manualmente hasta que se admita la creación automática de estadísticas para archivos CSV. Consulte los siguientes ejemplos sobre cómo quitar y crear estadísticas.

Una de las primeras preguntas que se deben formular para la solución de problemas de una consulta es **"¿Están actualizadas las estadísticas?"**

Si el número de filas ha cambiado significativamente o hay un cambio material en la distribución de valores para una columna, *entonces* es el momento de actualizar las estadísticas.

> [!NOTE]
> Recuerde que, si hay un cambio material en la distribución de valores para una columna, debe actualizar las estadísticas independientemente de la última vez que se actualizaran.

### <a name="implement-statistics-management"></a>Implementación de la administración de estadísticas

Debería extender la canalización de datos para asegurarse de que las estadísticas se actualizan cuando los datos se modifican de forma significativa mediante la adición, eliminación o modificación de archivos.

Los siguientes principios fundamentales se proporcionan para actualizar las estadísticas:

- Asegúrese de que el conjunto de datos tenga al menos un objeto de estadísticas actualizado. Así se actualiza la información del tamaño (recuento de filas y recuento de páginas) como parte de la actualización de las estadísticas.
- Céntrese en las columnas que participan en las cláusulas JOIN, GROUP BY, ORDER BY y DISTINCT.
- Actualice con más frecuencia las columnas de "clave ascendente", como las fechas de transacción, ya que estos valores no se incluirán en el histograma de estadísticas.
- Actualice las columnas de distribución estáticas con menor frecuencia.

Para obtener más información, consulte [Estimación de cardinalidad](/sql/relational-databases/performance/cardinality-estimation-sql-server).

### <a name="examples-create-statistics-for-column-in-openrowset-path"></a>Ejemplos: Creación de estadísticas para columnas en la ruta de acceso OPENROWSET

En los siguientes ejemplos se muestra cómo usar diversas opciones de creación de estadísticas. Las opciones que utiliza para cada columna dependen de las características de los datos y cómo se utilizará la columna en las consultas.

> [!NOTE]
> En este momento, solo puede crear estadísticas de columna única.
>
> El nombre del procedimiento sp_create_file_statistics se cambiará por sp_create_openrowset_statistics. El rol de servidor público tiene concedido el permiso ADMINISTER BULK OPERATIONS mientras que el rol de base de datos público tiene permisos EXECUTE en sp_create_file_statistics y sp_drop_file_statistics. Esto podría cambiar en el futuro.

El procedimiento almacenado siguiente se emplea para crear estadísticas:

```sql
sys.sp_create_file_statistics [ @stmt = ] N'statement_text'
```

Argumentos: [ @stmt = ] N“statement_text”: especifica una instrucción Transact-SQL que devolverá los valores de columna que se usarán para las estadísticas. Puede usar TABLESAMPLE para especificar los ejemplos de datos que se usarán. Si no se especifica TABLESAMPLE, se usará FULLSCAN.

```syntaxsql
<tablesample_clause> ::= TABLESAMPLE ( sample_number PERCENT )
```

> [!NOTE]
> El muestreo de CSV no funciona en este momento, solo FULLSCAN es compatible con CSV.

#### <a name="create-single-column-statistics-by-examining-every-row"></a>Crear estadísticas de columna única mediante el examen de todas las filas

Para crear las estadísticas de una columna, especifique una consulta que devuelva la columna para la que necesita estadísticas.

De forma predeterminada, si no se especifica lo contrario, SQL a petición usa el 100 % de los datos proporcionados en el conjunto de datos al crear las estadísticas.

Por ejemplo, haga lo siguiente para crear estadísticas mediante las opciones predeterminadas (FULLSCAN) de la columna year del conjunto de datos basado en el archivo population.csv:

```sql
/* make sure you have credentials for storage account access created
IF EXISTS (SELECT * FROM sys.credentials WHERE name = 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer')
DROP CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]
GO

CREATE CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]  
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = ''
GO
*/

EXEC sys.sp_create_file_statistics N'SELECT year
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/csv/population/population.csv'',
        FORMAT = ''CSV'',
        FIELDTERMINATOR ='','',
        ROWTERMINATOR = ''\n''
    )
WITH (
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
) AS [r]
'
```

#### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Crear estadísticas de columna única especificando el tamaño de muestra

Puede especificar el tamaño de muestra en forma de porcentaje:

```sql
/* make sure you have credentials for storage account access created
IF EXISTS (SELECT * FROM sys.credentials WHERE name = 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer')
DROP CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]
GO

CREATE CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]  
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = ''
GO
*/

EXEC sys.sp_create_file_statistics N'SELECT payment_type
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=2018/month=6/*.parquet'',
         FORMAT = ''PARQUET''
    ) AS [nyc]
    TABLESAMPLE(5 PERCENT)
'
```

### <a name="examples-update-statistics"></a>Ejemplos: Actualizar estadísticas

Para actualizar las estadísticas, debe quitarlas y crearlas. El procedimiento almacenado siguiente se emplea para quitar estadísticas:

```sql
sys.sp_drop_file_statistics [ @stmt = ] N'statement_text'
```

> [!NOTE]
> El nombre del procedimiento sp_drop_file_statistics se cambiará por sp_drop_openrowset_statistics. El rol de servidor público tiene concedido el permiso ADMINISTER BULK OPERATIONS mientras que el rol de base de datos público tiene permisos EXECUTE en sp_create_file_statistics y sp_drop_file_statistics. Esto podría cambiar en el futuro.

Argumentos: [@stmt =] N“statement_text”: especifica la misma instrucción Transact-SQL que se usa al crear las estadísticas.

Para actualizar las estadísticas de la columna year del conjunto de datos, que se basa en el archivo population.csv, debe quitar y crear las estadísticas:

```sql
EXEC sys.sp_drop_file_statistics N'SELECT payment_type
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=2018/month=6/*.parquet'',
         FORMAT = ''PARQUET''
    ) AS [nyc]
    TABLESAMPLE(5 PERCENT)
'
GO

/* make sure you have credentials for storage account access created
IF EXISTS (SELECT * FROM sys.credentials WHERE name = 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer')
DROP CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]
GO

CREATE CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]  
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = ''
GO
*/

EXEC sys.sp_create_file_statistics N'SELECT payment_type
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=2018/month=6/*.parquet'',
         FORMAT = ''PARQUET''
    ) AS [nyc]
    TABLESAMPLE(5 PERCENT)
'
```

### <a name="examples-create-statistics-for-external-table-column"></a>Ejemplos: Creación de estadísticas para columnas de tablas externas

En los siguientes ejemplos se muestra cómo usar diversas opciones de creación de estadísticas. Las opciones que utiliza para cada columna dependen de las características de los datos y cómo se utilizará la columna en las consultas.

> [!NOTE]
> En este momento, solo puede crear estadísticas de columna única.

Para crear estadísticas de una columna, especifique un nombre para el objeto de estadística y el nombre de la columna.

```sql
CREATE STATISTICS statistics_name
ON { external_table } ( column )
    WITH
        { FULLSCAN
          | [ SAMPLE number PERCENT ] }
        , { NORECOMPUTE }
```

Argumentos: external_table especifica la tabla externa en la que debe crear las estadísticas.

FULLSCAN calcula las estadísticas examinando todas las filas. FULLSCAN y SAMPLE 100 PERCENT tienen los mismos resultados. FULLSCAN no se puede utilizar con la opción SAMPLE.

SAMPLE número PERCENT especifica el porcentaje o número de filas aproximado de la tabla o vista indizada que usa el optimizador de consultas al crear las estadísticas. El número puede encontrarse entre 0 y 100.

SAMPLE no se puede utilizar con la opción FULLSCAN.

> [!NOTE]
> El muestreo de CSV no funciona en este momento, solo FULLSCAN es compatible con CSV.

#### <a name="create-single-column-statistics-by-examining-every-row"></a>Crear estadísticas de columna única mediante el examen de todas las filas

```sql
CREATE STATISTICS sState
    on census_external_table (STATENAME)
    WITH FULLSCAN, NORECOMPUTE
```

#### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Crear estadísticas de columna única especificando el tamaño de muestra

```sql
-- following sample creates statistics with sampling 20%
CREATE STATISTICS sState
    on census_external_table (STATENAME)
    WITH SAMPLE 5 percent, NORECOMPUTE
```

### <a name="examples-update-statistics"></a>Ejemplos: Actualizar estadísticas

Para actualizar las estadísticas, debe quitarlas y crearlas. Primero, use DROP STATISTICS:

```sql
DROP STATISTICS census_external_table.sState
```

Y después, use CREATE STATISTICS:

```sql
CREATE STATISTICS sState
    on census_external_table (STATENAME)
    WITH FULLSCAN, NORECOMPUTE
```

## <a name="next-steps"></a>Pasos siguientes

Para mejorar aún más el rendimiento de las consultas, consulte [Procedimientos recomendados para el grupo de SQL](best-practices-sql-pool.md#maintain-statistics).
