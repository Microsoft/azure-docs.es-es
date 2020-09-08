---
title: Creación de particiones de tablas
description: Recomendaciones y ejemplos para usar particiones de tablas en el grupo de SQL de Synapse.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/18/2019
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: ed5c0a140c69e9042fc9b85589719a54b65e985e
ms.sourcegitcommit: e2b36c60a53904ecf3b99b3f1d36be00fbde24fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88763140"
---
# <a name="partitioning-tables-in-synapse-sql-pool"></a>Creación de particiones de tablas en el grupo de SQL de Synapse

Recomendaciones y ejemplos para usar particiones de tablas en el grupo de SQL de Synapse.

## <a name="what-are-table-partitions"></a>¿Qué son las particiones de tablas?

Las particiones de tablas permiten dividir los datos en grupos más pequeños de datos. En la mayoría de los casos, se crean particiones de tablas en una columna de fecha. La creación de particiones es compatible con todos los tipos de tabla del grupo de SQL de Synapse, entre los que se incluyen el almacén de columnas en clúster, el índice agrupado y el montón. La creación de particiones también es compatible con todos los tipos de distribución, incluidos la distribución por hash o la distribución Round Robin.  

La creación de particiones puede beneficiar el mantenimiento de datos y el rendimiento de las consultas. El hecho de que beneficie a ambos aspectos, o solo a uno de ellos, dependerá de la forma en que se carguen los datos y de la misma columna se puede usar para ambos propósitos, ya que la creación de particiones solo se puede hacer en una columna.

### <a name="benefits-to-loads"></a>Ventajas para las cargas

La principal ventaja de la creación de particiones en el grupo de SQL de Synapse es que se mejora la eficiencia y el rendimiento de la carga de datos mediante el uso de la eliminación, la conmutación y la combinación de particiones. En la mayoría de los casos, la creación de particiones de los datos se realiza en una columna de fecha que está estrechamente relacionada con el orden con el que se cargan los datos en la base de datos. Una de las mayores ventajas de utilizar particiones para mantener los datos es que se evita el registro de transacciones. Aunque la mera inserción, actualización o eliminación de datos puede ser el enfoque más sencillo, con un poco de esfuerzo, el uso de la creación de particiones durante el proceso de carga puede mejorar considerablemente el rendimiento.

La conmutación de particiones se puede utilizar para quitar o reemplazar rápidamente cualquier sección de una tabla.  Por ejemplo, una tabla de datos de ventas podría contener datos solo de los últimos 36 meses. Al final de cada mes, el mes más antiguo de datos de ventas se elimina de la tabla.  Estos datos pueden eliminarse mediante el uso de una instrucción delete para eliminar los datos del mes más antiguo. 

Sin embargo, la eliminación una gran cantidad de datos fila a fila con una instrucción delete puede tardar mucho tiempo, así como crear el riesgo de transacciones grandes que podría tardar mucho tiempo en revertirse si algo va mal. Un enfoque mejor es colocar la partición de datos más antigua. Aunque la eliminación de las filas individuales podría tardar horas, la de toda una partición puede tardar segundos.

### <a name="benefits-to-queries"></a>Ventajas para las consultas

La creación de particiones también puede utilizarse para mejorar el rendimiento de las consultas. Una consulta que aplica un filtro a los datos con particiones puede limitar el análisis a solo las particiones idóneas. Este método de filtrado puede evitar un recorrido de tabla completa y examinar solo un subconjunto más pequeño de datos. Con la introducción de índices de almacén de columnas en clúster, las ventajas de rendimiento de la eliminación del predicado son menores, pero en algunos casos puede haber beneficios para las consultas. 

Por ejemplo, si la tabla de datos de ventas se particiona en 36 meses mediante el campo de fecha de ventas, las consultas que se filtren por esa fecha de ventas pueden omitir la búsqueda en las particiones que no coincidan con el filtro.

## <a name="sizing-partitions"></a>Ajustar el tamaño de las particiones

Aunque la creación de particiones se puede usar para mejorar el rendimiento en algunos escenarios, la creación de una tabla con **demasiadas** particiones pueden afectar negativamente al rendimiento en algunas circunstancias.  Estas cuestiones se dan especialmente en las tablas de almacén de columnas en clúster. 

Para que la creación de particiones sea útil, es importante saber cuándo usarla y el número de particiones que se deben crear. No hay ninguna regla inamovible con respecto a cuántas particiones son demasiadas, depende de los datos y del número de particiones que se cargan de manera simultánea. Normalmente, un esquema de partición correcta tiene decenas o centenas de particiones, no miles.

Al crear particiones en tablas de **almacén de columnas en clúster**, es importante considerar el número de filas que pertenece a cada partición. Para que tanto la compresión como el rendimiento de las tablas de almacén de columnas en clúster sean óptimos, se necesita un mínimo de un millón de filas por partición y distribución. Antes de que se creen particiones, el grupo de SQL de Synapse ya divide cada tabla en 60 bases de datos distribuidas. 

Todas las particiones que se agreguen a una tabla se sumarán a las distribuciones creadas en segundo plano. Usando este ejemplo, si la tabla de datos de ventas contenía 36 particiones mensuales, y dado que un grupo de SQL de Synapse tiene 60 distribuciones, la tabla de datos de ventas debería contener 60 millones de filas por mes, o 2 100 millones de filas cuando se rellenen todos los meses. Si una tabla contiene menos filas que el número mínimo recomendado de filas por partición, considere utilizar menos particiones para hacer que aumente el número de filas por partición. 

Para obtener más información, consulte el artículo [Indexación](sql-data-warehouse-tables-index.md), que incluye las consultas que pueden evaluar la calidad de los índices de almacén de columnas en clúster.

## <a name="syntax-differences-from-sql-server"></a>Diferencias de sintaxis con respecto a SQL Server

El grupo de SQL de Synapse introduce una manera más sencilla de definir particiones que SQL Server. Las funciones y los esquemas de la creación de particiones no se usan igual en el grupo de SQL de Synapse que en SQL Server. En su lugar, lo único que se debe hacer es identificar la columna con particiones y los puntos limítrofes. 

Aunque la sintaxis de la creación de particiones puede variar ligeramente con respecto a la de SQL Server, los conceptos básicos son los mismos. SQL Server y el grupo de SQL de Synapse admiten una columna de partición por tabla, que puede ser una partición por rangos. Para obtener más información sobre las particiones, consulte [Tablas e índices con particiones](/sql/relational-databases/partitions/partitioned-tables-and-indexes?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

En el siguiente ejemplo se usa una instrucción [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) para crear particiones en la tabla FactInternetSales en la columna OrderDateKey:

```sql
CREATE TABLE [dbo].[FactInternetSales]
(
    [ProductKey]            int          NOT NULL
,   [OrderDateKey]          int          NOT NULL
,   [CustomerKey]           int          NOT NULL
,   [PromotionKey]          int          NOT NULL
,   [SalesOrderNumber]      nvarchar(20) NOT NULL
,   [OrderQuantity]         smallint     NOT NULL
,   [UnitPrice]             money        NOT NULL
,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                    (20000101,20010101,20020101
                    ,20030101,20040101,20050101
                    )
                )
)
;
```

## <a name="migrating-partitioning-from-sql-server"></a>Migración de particiones de SQL Server

Para migrar las definiciones de particiones de SQL Server al grupo de SQL de Synapse, basta con que haga lo siguiente:

- Elimine el [esquema de particiones](/sql/t-sql/statements/create-partition-scheme-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) de SQL Server.
- Agregue la definición de [función de partición](/sql/t-sql/statements/create-partition-function-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) a CREATE TABLE.

Si va a migrar una tabla con particiones de una instancia de SQL Server, la instrucción SQL siguiente puede ayudarle a consultar el número de filas que se encuentran en cada partición. Tenga en cuenta que si se usa la misma granularidad de particiones en el grupo de SQL de Synapse, el número de filas por partición se reduce en un factor de 60.  

```sql
-- Partition information for a SQL Server Database
SELECT      s.[name]                        AS      [schema_name]
,           t.[name]                        AS      [table_name]
,           i.[name]                        AS      [index_name]
,           p.[partition_number]            AS      [partition_number]
,           SUM(a.[used_pages]*8.0)         AS      [partition_size_kb]
,           SUM(a.[used_pages]*8.0)/1024    AS      [partition_size_mb]
,           SUM(a.[used_pages]*8.0)/1048576 AS      [partition_size_gb]
,           p.[rows]                        AS      [partition_row_count]
,           rv.[value]                      AS      [partition_boundary_value]
,           p.[data_compression_desc]       AS      [partition_compression_desc]
FROM        sys.schemas s
JOIN        sys.tables t                    ON      t.[schema_id]         = s.[schema_id]
JOIN        sys.partitions p                ON      p.[object_id]         = t.[object_id]
JOIN        sys.allocation_units a          ON      a.[container_id]      = p.[partition_id]
JOIN        sys.indexes i                   ON      i.[object_id]         = p.[object_id]
                                            AND     i.[index_id]          = p.[index_id]
JOIN        sys.data_spaces ds              ON      ds.[data_space_id]    = i.[data_space_id]
LEFT JOIN   sys.partition_schemes ps        ON      ps.[data_space_id]    = ds.[data_space_id]
LEFT JOIN   sys.partition_functions pf      ON      pf.[function_id]      = ps.[function_id]
LEFT JOIN   sys.partition_range_values rv   ON      rv.[function_id]      = pf.[function_id]
                                            AND     rv.[boundary_id]      = p.[partition_number]
WHERE       p.[index_id] <=1
GROUP BY    s.[name]
,           t.[name]
,           i.[name]
,           p.[partition_number]
,           p.[rows]
,           rv.[value]
,           p.[data_compression_desc]
;
```

## <a name="partition-switching"></a>Modificación de particiones

El grupo de SQL de Synapse admite la separación, la combinación y la modificación de particiones. Cada una de estas funciones se ejecuta mediante la instrucción [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

Para cambiar particiones entre dos tablas, debe asegurarse de que las particiones se alinean en sus límites correspondientes y que las definiciones de tablas coinciden. Como las restricciones CHECK no están disponibles para aplicar el intervalo de valores en una tabla, la tabla de origen debe contener los mismos límites de partición que la tabla de destino. Si los límites de partición no son iguales, el cambio de partición genera un error dado que los metadatos de la partición no se sincronizan.

### <a name="how-to-split-a-partition-that-contains-data"></a>División de una partición que contiene datos

El método más eficaz para dividir una partición que ya contiene datos es usar una instrucción `CTAS` . Si la tabla con particiones es un almacén de columnas en clúster, la partición de tabla debe estar vacía antes de que se pueda dividir.

En el ejemplo siguiente se crea una tabla de almacén de columnas con particiones. Se inserta una fila en cada partición:

```sql
CREATE TABLE [dbo].[FactInternetSales]
(
        [ProductKey]            int          NOT NULL
    ,   [OrderDateKey]          int          NOT NULL
    ,   [CustomerKey]           int          NOT NULL
    ,   [PromotionKey]          int          NOT NULL
    ,   [SalesOrderNumber]      nvarchar(20) NOT NULL
    ,   [OrderQuantity]         smallint     NOT NULL
    ,   [UnitPrice]             money        NOT NULL
    ,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                    (20000101
                    )
                )
)
;

INSERT INTO dbo.FactInternetSales
VALUES (1,19990101,1,1,1,1,1,1);
INSERT INTO dbo.FactInternetSales
VALUES (1,20000101,1,1,1,1,1,1);
```

La siguiente consulta busca el recuento de filas mediante la vista de catálogo `sys.partitions`:

```sql
SELECT  QUOTENAME(s.[name])+'.'+QUOTENAME(t.[name]) as Table_name
,       i.[name] as Index_name
,       p.partition_number as Partition_nmbr
,       p.[rows] as Row_count
,       p.[data_compression_desc] as Data_Compression_desc
FROM    sys.partitions p
JOIN    sys.tables     t    ON    p.[object_id]   = t.[object_id]
JOIN    sys.schemas    s    ON    t.[schema_id]   = s.[schema_id]
JOIN    sys.indexes    i    ON    p.[object_id]   = i.[object_Id]
                            AND   p.[index_Id]    = i.[index_Id]
WHERE t.[name] = 'FactInternetSales'
;
```

El siguiente comando split recibe un mensaje de error:

```sql
ALTER TABLE FactInternetSales SPLIT RANGE (20010101);
```

La cláusula Msg 35346, Level 15, State 1, Line 44 SPLIT de la instrucción ALTER PARTITION ha generado un error porque la partición no está vacía. solo las particiones vacías se pueden dividir cuando existe un índice de almacén de columnas en la tabla. Considere la posibilidad de deshabilitar el índice de almacén de columnas antes de emitir la instrucción ALTER PARTITION y, a continuación, vuelva a generar el índice de almacén de columnas una vez completada la instrucción ALTER PARTITION.

Sin embargo, puede usar `CTAS` para crear una nueva tabla que contenga los datos.

```sql
CREATE TABLE dbo.FactInternetSales_20000101
    WITH    (   DISTRIBUTION = HASH(ProductKey)
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101
                                )
                            )
            )
AS
SELECT *
FROM    FactInternetSales
WHERE   1=2
;
```

Se permite el cambio porque los límites de partición están alineados. Esto deja la tabla de origen con una partición vacía que se puede dividir posteriormente.

```sql
ALTER TABLE FactInternetSales SWITCH PARTITION 2 TO  FactInternetSales_20000101 PARTITION 2;

ALTER TABLE FactInternetSales SPLIT RANGE (20010101);
```

Lo único que falta por hacer es alinear los datos con los nuevos límites de partición mediante `CTAS` y volver a cambiar los datos a la tabla principal.

```sql
CREATE TABLE [dbo].[FactInternetSales_20000101_20010101]
    WITH    (   DISTRIBUTION = HASH([ProductKey])
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101,20010101
                                )
                            )
            )
AS
SELECT  *
FROM    [dbo].[FactInternetSales_20000101]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

ALTER TABLE dbo.FactInternetSales_20000101_20010101 SWITCH PARTITION 2 TO dbo.FactInternetSales PARTITION 2;
```

Una vez que haya completado el movimiento de los datos, es una buena idea actualizar las estadísticas en la tabla de destino. Al actualizar las estadísticas se tiene la seguridad de que estas reflejan con precisión la nueva distribución de los datos en sus respectivas particiones.

```sql
UPDATE STATISTICS [dbo].[FactInternetSales];
```

### <a name="load-new-data-into-partitions-that-contain-data-in-one-step"></a>Cargar en un solo paso datos nuevos en particiones que ya contienen datos

La carga de datos en particiones mediante el cambio de partición es una forma muy útil de organizar nuevos datos en una tabla que no es visible para los usuarios.  Puede un desafío para los sistemas ocupados tratar con la contención de bloqueo asociada con el cambio de partición.  

Para borrar los datos existentes en una partición, solía requerirse un `ALTER TABLE` para cambiar los datos.  Luego se requería otro `ALTER TABLE` para cambiar a los nuevos datos.  

En el grupo de SQL de Synapse, la opción `TRUNCATE_TARGET` se admite en el comando `ALTER TABLE`.  Con `TRUNCATE_TARGET` el comando `ALTER TABLE` sobrescribe los datos existentes en la partición con datos nuevos.  A continuación se muestra un ejemplo que usa `CTAS` para crear una nueva tabla con los datos existentes, inserta datos nuevos y luego cambia todos los datos de nuevo a la tabla de destino, sobrescribiendo los datos existentes.

```sql
CREATE TABLE [dbo].[FactInternetSales_NewSales]
    WITH    (   DISTRIBUTION = HASH([ProductKey])
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101,20010101
                                )
                            )
            )
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

INSERT INTO dbo.FactInternetSales_NewSales
VALUES (1,20000101,2,2,2,2,2,2);

ALTER TABLE dbo.FactInternetSales_NewSales SWITCH PARTITION 2 TO dbo.FactInternetSales PARTITION 2 WITH (TRUNCATE_TARGET = ON);  
```

### <a name="table-partitioning-source-control"></a>Control de código fuente de particiones de tabla

Para evitar que la definición de tabla se **oxide** en el sistema de control de código fuente, es conveniente tener en cuenta lo siguiente:

1. Crear la tabla como una tabla con particiones, pero sin valores de partición

    ```sql
    CREATE TABLE [dbo].[FactInternetSales]
    (
        [ProductKey]            int          NOT NULL
    ,   [OrderDateKey]          int          NOT NULL
    ,   [CustomerKey]           int          NOT NULL
    ,   [PromotionKey]          int          NOT NULL
    ,   [SalesOrderNumber]      nvarchar(20) NOT NULL
    ,   [OrderQuantity]         smallint     NOT NULL
    ,   [UnitPrice]             money        NOT NULL
    ,   [SalesAmount]           money        NOT NULL
    )
    WITH
    (   CLUSTERED COLUMNSTORE INDEX
    ,   DISTRIBUTION = HASH([ProductKey])
    ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES () )
    )
    ;
    ```

1. `SPLIT` la tabla como parte del proceso de implementación:

    ```sql
     -- Create a table containing the partition boundaries

    CREATE TABLE #partitions
    WITH
    (
        LOCATION = USER_DB
    ,   DISTRIBUTION = HASH(ptn_no)
    )
    AS
    SELECT  ptn_no
    ,       ROW_NUMBER() OVER (ORDER BY (ptn_no)) as seq_no
    FROM    (
        SELECT CAST(20000101 AS INT) ptn_no
        UNION ALL
        SELECT CAST(20010101 AS INT)
        UNION ALL
        SELECT CAST(20020101 AS INT)
        UNION ALL
        SELECT CAST(20030101 AS INT)
        UNION ALL
        SELECT CAST(20040101 AS INT)
    ) a
    ;

     -- Iterate over the partition boundaries and split the table

    DECLARE @c INT = (SELECT COUNT(*) FROM #partitions)
    ,       @i INT = 1                                 --iterator for while loop
    ,       @q NVARCHAR(4000)                          --query
    ,       @p NVARCHAR(20)     = N''                  --partition_number
    ,       @s NVARCHAR(128)    = N'dbo'               --schema
    ,       @t NVARCHAR(128)    = N'FactInternetSales' --table
    ;

    WHILE @i <= @c
    BEGIN
        SET @p = (SELECT ptn_no FROM #partitions WHERE seq_no = @i);
        SET @q = (SELECT N'ALTER TABLE '+@s+N'.'+@t+N' SPLIT RANGE ('+@p+N');');

        -- PRINT @q;
        EXECUTE sp_executesql @q;
        SET @i+=1;
    END

     -- Code clean-up

    DROP TABLE #partitions;
    ```

Con este enfoque, el código de control de código fuente permanece estático y se permite que los valores del límite de creación de particiones sean dinámicos, de tal forma que evolucionan con la base de datos con el tiempo.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el desarrollo de tablas, vea los artículos de [Información general sobre tablas](sql-data-warehouse-tables-overview.md).
