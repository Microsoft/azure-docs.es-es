---
title: Diseño de tablas mediante SQL de Synapse
description: Introducción al diseño de tablas en SQL de Synapse.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 3bf180c2b70a686879082888e45e67936cdbec67
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799237"
---
# <a name="design-tables-using-synapse-sql"></a>Diseño de tablas mediante SQL de Synapse

En este documento se incluyen conceptos clave para diseñar tablas con un grupo de SQL y SQL a petición (versión preliminar).  

[SQL a petición (versión preliminar)](on-demand-workspace-overview.md) es un servicio de consulta de los datos del lago de datos. No tiene almacenamiento local ni ingesta de datos. El [grupo de SQL](best-practices-sql-pool.md) representa una colección de recursos de análisis que se aprovisionan al usar SQL de Synapse. El tamaño del grupo de SQL lo determinan las unidades de almacenamiento de datos (DWU).

En la tabla siguiente se enumeran los temas que son pertinentes para el grupo de SQL frente a SQL a petición:

| Tema                                                        | Grupo de SQL | SQL a petición |
| ------------------------------------------------------------ | ------------------ | ----------------------- |
| [Determinación de la categoría de tabla](#determine-table-category)        | Sí                | No                      |
| [Nombres de esquemas](#schema-names)                                | Sí                | Sí                     |
| [Nombres de tabla](#table-names)                                  | Sí                | No                      |
| [Persistencia de tabla](#table-persistence)                      | Sí                | No                      |
| [Tabla normal](#regular-table)                              | Sí                | No                      |
| [Tabla temporal](#temporary-table)                          | Sí                | Sí                     |
| [Tabla externa](#external-table)                            | Sí                | Sí                     |
| [Tipos de datos](#data-types)                                    | Sí                | Sí                     |
| [Tablas distribuidas](#distributed-tables)                    | Sí                | No                      |
| [Tablas distribuidas mediante una función hash](#hash-distributed-tables)          | Sí                | No                      |
| [Tablas replicadas](#replicated-tables)                      | Sí                | No                      |
| [Tablas round robin](#round-robin-tables)                    | Sí                | No                      |
| [Métodos comunes de distribución para tablas](#common-distribution-methods-for-tables) | Sí                | No                      |
| [Particiones](#partitions)                                    | Sí                | Sí                     |
| [Índices de almacén de columnas](#columnstore-indexes)                  | Sí                | No                      |
| [estadísticas](#statistics)                                    | Sí                | Sí                     |
| [Clave principal y clave única](#primary-key-and-unique-key)    | Sí                | No                      |
| [Comandos para la creación de tablas](#commands-for-creating-tables) | Sí                | No                      |
| [Alineación de los datos de origen con el almacenamiento de datos](#align-source-data-with-the-data-warehouse) | Sí                | No                      |
| [Características no compatibles de las tablas](#unsupported-table-features)    | Sí                | No                      |
| [Consultas de tamaño de tabla](#table-size-queries)                    | Sí                | No                      |

## <a name="determine-table-category"></a>Determinación de la categoría de tabla

Un [esquema de estrella](https://en.wikipedia.org/wiki/Star_schema) organiza los datos en tablas de hechos y dimensiones. Algunas tablas se utilizan para datos de integración o de almacenamiento provisional antes de moverlos a una tabla de hechos o dimensiones. Al diseñar una tabla, decidirá si los datos de la misma pertenecen a una tabla de hechos, dimensiones o integración. Esta decisión informa de la distribución y estructura de tabla adecuadas.

- Las **tablas de hechos** contienen datos cuantitativos que se suelen generar en un sistema transaccional y, después, se cargan en el almacenamiento de datos. Por ejemplo, una empresa minorista genera transacciones de ventas todos los días y, después, carga los datos en una tabla de hechos en el almacenamiento de datos para su análisis.

- Las **tablas de dimensiones** contienen datos de atributos que pueden cambiar, pero normalmente no cambian con frecuencia. Por ejemplo, el nombre y la dirección de un cliente se almacenan en una tabla de dimensiones y solo se actualizan cuando el perfil del cliente cambia. Para minimizar el tamaño de una tabla de hechos de gran tamaño, el nombre y la dirección del cliente no necesitan estar en todas las filas de una tabla de hechos. En su lugar, la tabla de hechos y la tabla de dimensiones pueden compartir un identificador de cliente. Una consulta puede combinar las dos tablas para asociar el perfil y las transacciones de un cliente.

- Las **tablas de integración** proporcionan un lugar para integración o almacenamiento provisional de datos. Puede crear una tabla de integración como una tabla normal, una tabla externa o una tabla temporal. Por ejemplo, puede cargar datos a una tabla de almacenamiento provisional, realizar transformaciones en los datos en el almacenamiento provisional y luego insertar dichos datos en una tabla de producción.

## <a name="schema-names"></a>Nombres de esquemas

Los esquemas son una buena manera de agrupar objetos que se usan de manera similar. El siguiente código crea un [esquema definido por el usuario](/sql/t-sql/statements/create-schema-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) denominado wwi.

```sql
CREATE SCHEMA wwi;
```

## <a name="table-names"></a>Nombres de tabla

Si está migrando varias bases de datos de una solución local al grupo de SQL, el procedimiento recomendado es migrar todas las tablas de hechos, dimensiones e integración a un esquema del grupo de SQL. Por ejemplo, podría almacenar todas las tablas en el almacenamiento de datos de ejemplo [WideWorldImportersDW](/sql/samples/wide-world-importers-dw-database-catalog?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) dentro de un esquema denominado wwi.

Para mostrar la organización de las tablas en el grupo de SQL, puede utilizar fact, dim e int como prefijos para los nombres de tabla. En la tabla a continuación se muestran algunos de los nombres de esquema y tabla para WideWorldImportersDW.  

| Tabla WideWorldImportersDW  | Tipo de tabla. | Grupo de SQL |
|:-----|:-----|:------|:-----|
| City | Dimensión | wwi.DimCity |
| Pedido de | Fact | wwi.FactOrder |

## <a name="table-persistence"></a>Persistencia de tabla

Las tablas almacenan datos de forma permanente en Azure Storage, temporalmente en Azure Storage o en un almacén de datos externo al almacenamiento de datos.

### <a name="regular-table"></a>Tabla normal

Una tabla normal almacena datos en Azure Storage como parte del almacenamiento de datos. La tabla y los datos persisten, independientemente de si hay una sesión abierta o no.  En el ejemplo a continuación se crea una tabla normal con dos columnas.

```sql
CREATE TABLE MyTable (col1 int, col2 int );  
```

### <a name="temporary-table"></a>Tabla temporal

Una tabla temporal solo existe mientras dura la sesión. Puede usar una tabla temporal para evitar que otros usuarios vean los resultados temporales. El uso de tablas temporales también reduce la necesidad de limpieza.  Las tablas temporales utilizan el almacenamiento local, en el grupo de SQL, para ofrecer un rendimiento más rápido.  

SQL a petición admite tablas temporales. Sin embargo, su uso está limitado, ya que usted puede seleccionar de una tabla temporal, pero no puede unirla con los archivos en el almacenamiento.

Para más información, consulte [Tablas temporales](develop-tables-temporary.md).

### <a name="external-table"></a>Tabla externa

Las [tablas externas](develop-tables-external-tables.md) apuntan a datos ubicados en Azure Storage Blob o Azure Data Lake Storage.

Importe los datos de tablas externas al grupo de SQL mediante la instrucción [CREATE TABLE AS SELECT](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json). Para un tutorial sobre la carga, consulte [Uso de PolyBase para cargar datos de Azure Blob Storage en Azure SQL Data Warehouse](../sql-data-warehouse/load-data-from-azure-blob-storage-using-polybase.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

En el caso de SQL a petición, puede utilizar [CETAS](develop-tables-cetas.md) para guardar el resultado de la consulta en una tabla externa en Azure Storage.

## <a name="data-types"></a>Tipos de datos

El grupo de SQL admite los tipos de datos más usados habitualmente. Para obtener una lista de los tipos de datos admitidos, consulte los [tipos de datos en la referencia de CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest#DataTypes) de la instrucción CREATE TABLE. Para obtener más información sobre el uso de los tipos de datos, consulte [Tipos de datos](../sql/develop-tables-data-types.md).

## <a name="distributed-tables"></a>Tablas distribuidas

Una característica fundamental del grupo de SQL es la forma en que puede almacenar y operar en tablas de varias [distribuciones](../sql-data-warehouse/massively-parallel-processing-mpp-architecture.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#distributions).  El grupo de SQL admite tres métodos para distribuir datos:

- Round robin (método predeterminado)
- Hash
- Replicado

### <a name="hash-distributed-tables"></a>Tablas distribuidas mediante una función hash

La distribución hash de la tabla distribuye las filas en función del valor de la columna de distribución. La tabla distribuida de hash está diseñada para lograr un alto rendimiento para consultas en tablas grandes. Hay varios factores que debe tener en cuenta al elegir una columna de distribución.

Para más información, vea [Distribución de tablas en SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="replicated-tables"></a>Tablas replicadas

Una tabla replicada tiene una copia completa de la tabla disponible en cada nodo de proceso. Las consultas se ejecutan rápidamente en tablas replicadas, ya que las combinaciones en las tablas replicadas no requieren movimiento de datos. Sin embargo, la replicación necesita almacenamiento adicional y este método no resulta práctico para tablas de gran tamaño.

Para más información, consulte [Instrucciones de diseño para el uso de tablas replicadas en Azure SQL Data Warehouse](../sql-data-warehouse/design-guidance-for-replicated-tables.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="round-robin-tables"></a>Tablas round robin

Una tabla round robin distribuye las filas de la tabla uniformemente entre todas las distribuciones. Las filas se distribuyen aleatoriamente. Cargar datos en una tabla round robin es rápido.  Pero las consultas pueden requerir más movimiento de datos que los demás métodos de distribución.

Para más información, vea [Distribución de tablas en SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="common-distribution-methods-for-tables"></a>Métodos comunes de distribución para tablas

La categoría de tabla suele determinar la opción óptima para la distribución de tablas.

| Categoría de tabla | Opción de distribución recomendada |
|:---------------|:--------------------|
| Fact           | Utilice la distribución por hash con el índice de almacén de columnas agrupado. El rendimiento mejora cuando se combinan dos tablas hash en la misma columna de distribución. |
| Dimensión      | Use la distribución replicada para tablas más pequeñas. Si las tablas son demasiado grandes para almacenar en cada nodo de proceso, utilice la distribución por hash. |
| Ensayo        | Use round robin para la tabla de almacenamiento provisional. La carga con CTAS es rápida. Una vez que los datos estén en la tabla de almacenamiento provisional, use INSERT... SELECT para mover los datos a las tablas de producción. |

## <a name="partitions"></a>Particiones

En un grupo de SQL, una tabla con particiones almacena y ejecuta operaciones en las filas de la tabla según los intervalos de datos. Por ejemplo, una tabla puede tener particiones por día, mes o año. Puede mejorar el rendimiento de las consultas mediante la eliminación de particiones, ya que limita el examen de una consulta a los datos dentro de una partición.

También puede mantener los datos a través de modificación de particiones. Puesto que los datos del grupo de SQL ya están distribuidos, demasiadas particiones pueden ralentizar el rendimiento de las consultas. Para más información, consulte [Creación de particiones de tablas en SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).  

> [!TIP]
> Cuando cambie de partición a otras particiones de tabla que no estén vacías, puede usar la opción TRUNCATE_TARGET en la instrucción [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) si se deben truncar los datos existentes.

En el código siguiente se cambian los datos diarios transformados a una partición SalesFact y se sobrescriben los datos existentes.

```sql
ALTER TABLE SalesFact_DailyFinalLoad SWITCH PARTITION 256 TO SalesFact PARTITION 256 WITH (TRUNCATE_TARGET = ON);  
```

En SQL a petición, puede limitar los archivos o carpetas (particiones) que leerá la consulta. La creación de particiones por ruta de acceso se admite mediante las funciones filepath y fileinfo descritas en [Consulta de archivos de almacenamiento](develop-storage-files-overview.md). En el ejemplo siguiente se lee una carpeta con datos del año 2017:

```sql
SELECT
    nyc.filepath(1) AS [year],
    payment_type,
    SUM(fare_amount) AS fare_total
FROM  
    OPENROWSET(
        BULK 'https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=*/month=*/*.parquet',
        FORMAT='PARQUET'
    ) AS nyc
WHERE
    nyc.filepath(1) = 2017
GROUP BY
    nyc.filepath(1),
    payment_type
ORDER BY
    nyc.filepath(1),
    payment_type
```

## <a name="columnstore-indexes"></a>Índices de almacén de columnas

De forma predeterminada, el grupo de SQL almacena una tabla como índice de almacén de columnas agrupado. Esta forma de almacenamiento de datos logra una compresión de datos y rendimiento de las consultas altos en tablas grandes.  El índice de almacén de columnas agrupado suele ser la mejor opción, pero en algunos casos un índice agrupado o un montón es la estructura de almacenamiento adecuada.  

> [!TIP]
> Una tabla de montón puede ser sumamente útil para cargar datos transitorios, como una tabla de almacenamiento provisional que se transforma en una tabla final.

Para una lista de características de almacén de columnas, vea [Novedades de los índices de almacén de columnas](/sql/relational-databases/indexes/columnstore-indexes-what-s-new?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest). Para mejorar el rendimiento del índice de almacén de columnas, vea [Maximización de la calidad del grupo de filas del almacén de columnas](../sql/data-load-columnstore-compression.md).

## <a name="statistics"></a>Estadísticas


El optimizador de consultas utiliza estadísticas de columna cuando crea el plan de ejecución de una consulta. Para mejorar el rendimiento de las consultas, es importante crear estadísticas en columnas individuales, especialmente en las columnas que se usan en combinaciones de consultas. SQL de Synapse admite la creación automática de estadísticas. 

La actualización estadística no se realiza automáticamente. Actualice las estadísticas cuando se agregue o cambie un número significativo de filas. Por ejemplo, actualice las estadísticas después de una carga. En el artículo con la [guía de estadísticas](develop-tables-statistics.md) encontrará información adicional.

## <a name="primary-key-and-unique-key"></a>Clave principal y clave única

PRIMARY KEY solo se admite cuando se usan NONCLUSTERED y NOT ENFORCED.  Solo se admite la restricción UNIQUE cuando se usa NOT ENFORCED.  Para obtener más información, consulte el artículo [Restricciones de tablas del grupo de SQL](../sql-data-warehouse/sql-data-warehouse-table-constraints.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="commands-for-creating-tables"></a>Comandos para la creación de tablas

Puede crear una tabla como una nueva tabla vacía. También puede crear y rellenar una tabla con los resultados de una instrucción SELECT. A continuación se muestran los comandos de T-SQL para crear una tabla.

| Instrucción T-SQL | Descripción |
|:----------------|:------------|
| [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) | Crea una tabla vacía mediante la definición de todas las opciones y columnas de la tabla. |
| [CREATE EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) | Crea una tabla externa. La definición de la tabla se almacena en el grupo de SQL. Los datos de la tabla se almacenan en Azure Blob Storage o Azure Data Lake Storage. |
| [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) | Rellena una nueva tabla con los resultados de una instrucción SELECT. Las columnas de tabla y los tipos de datos se basan en los resultados de la instrucción SELECT. Para importar datos, puede seleccionar esta instrucción en una tabla externa. |
| [CREATE EXTERNAL TABLE AS SELECT](/sql/t-sql/statements/create-external-table-as-select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) | Crea una nueva tabla externa mediante la exportación de los resultados de una instrucción SELECT a una ubicación externa.  La ubicación es Azure Blob Storage o Azure Data Lake Storage. |

## <a name="align-source-data-with-the-data-warehouse"></a>Alineación de los datos de origen con el almacenamiento de datos

Las tablas del almacenamiento de datos se rellenan cargando datos desde otro origen de datos. Para lograr una carga correcta, el número y los tipos de datos de las columnas de los datos de origen se deben alinear con la definición de tabla en el almacenamiento de datos.

> [!NOTE]
> Obtener los datos que se van a alinear podría ser la parte más complicada de diseñar las tablas.

Si los datos proceden de varios almacenamientos de datos, puede portar dichos datos en el almacenamiento de datos y almacenarlos en una tabla de integración. Una vez que los datos estén en la tabla de integración, podrá utilizar la eficacia del grupo de SQL para implementar operaciones de transformación. Una vez que los datos están preparados, puede insertarlos en tablas de producción.

## <a name="unsupported-table-features"></a>Características no compatibles de las tablas

El grupo de SQL admite muchas, pero no todas, de las características de tabla que ofrecen otras bases de datos.  En la lista siguiente se muestran algunas de las características de tabla que no se admiten en el grupo de SQL.

- Clave externa, compruebe [Restricciones de tabla](/sql/t-sql/statements/alter-table-table-constraint-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)
- [Columnas calculadas](/sql/t-sql/statements/alter-table-computed-column-definition-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)
- [Vistas indizadas](/sql/relational-databases/views/create-indexed-views?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)
- [Secuencia](/sql/t-sql/statements/create-sequence-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)
- [Columnas dispersas](/sql/relational-databases/tables/use-sparse-columns?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)
- Claves suplentes, implemente con [Identity](../sql-data-warehouse/sql-data-warehouse-tables-identity.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Sinónimos](/sql/t-sql/statements/create-synonym-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)
- [Desencadenadores](/sql/t-sql/statements/create-trigger-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)
- [Índices únicos](/sql/t-sql/statements/create-index-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)
- [Tipos definidos por el usuario](/sql/relational-databases/native-client/features/using-user-defined-types?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)

## <a name="table-size-queries"></a>Consultas de tamaño de tabla

Una forma sencilla de identificar el espacio y las filas que consume una tabla en cada una de las 60 distribuciones es usar [DBCC PDW_SHOWSPACEUSED](/sql/t-sql/database-console-commands/dbcc-pdw-showspaceused-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

```sql
DBCC PDW_SHOWSPACEUSED('dbo.FactInternetSales');
```

Tenga en cuenta que el uso de los comandos DBCC puede resultar muy limitador.  Las vistas de administración dinámica (DMV) muestran más detalles que los comandos DBCC. Comience por crear la vista a continuación.

```sql
CREATE VIEW dbo.vTableSizes
AS
WITH base
AS
(
SELECT
 GETDATE()                                                             AS  [execution_time]
, DB_NAME()                                                            AS  [database_name]
, s.name                                                               AS  [schema_name]
, t.name                                                               AS  [table_name]
, QUOTENAME(s.name)+'.'+QUOTENAME(t.name)                              AS  [two_part_name]
, nt.[name]                                                            AS  [node_table_name]
, ROW_NUMBER() OVER(PARTITION BY nt.[name] ORDER BY (SELECT NULL))     AS  [node_table_name_seq]
, tp.[distribution_policy_desc]                                        AS  [distribution_policy_name]
, c.[name]                                                             AS  [distribution_column]
, nt.[distribution_id]                                                 AS  [distribution_id]
, i.[type]                                                             AS  [index_type]
, i.[type_desc]                                                        AS  [index_type_desc]
, nt.[pdw_node_id]                                                     AS  [pdw_node_id]
, pn.[type]                                                            AS  [pdw_node_type]
, pn.[name]                                                            AS  [pdw_node_name]
, di.name                                                              AS  [dist_name]
, di.position                                                          AS  [dist_position]
, nps.[partition_number]                                               AS  [partition_nmbr]
, nps.[reserved_page_count]                                            AS  [reserved_space_page_count]
, nps.[reserved_page_count] - nps.[used_page_count]                    AS  [unused_space_page_count]
, nps.[in_row_data_page_count]
    + nps.[row_overflow_used_page_count]
    + nps.[lob_used_page_count]                                        AS  [data_space_page_count]
, nps.[reserved_page_count]
 - (nps.[reserved_page_count] - nps.[used_page_count])
 - ([in_row_data_page_count]
         + [row_overflow_used_page_count]+[lob_used_page_count])       AS  [index_space_page_count]
, nps.[row_count]                                                      AS  [row_count]
from
    sys.schemas s
INNER JOIN sys.tables t
    ON s.[schema_id] = t.[schema_id]
INNER JOIN sys.indexes i
    ON  t.[object_id] = i.[object_id]
    AND i.[index_id] <= 1
INNER JOIN sys.pdw_table_distribution_properties tp
    ON t.[object_id] = tp.[object_id]
INNER JOIN sys.pdw_table_mappings tm
    ON t.[object_id] = tm.[object_id]
INNER JOIN sys.pdw_nodes_tables nt
    ON tm.[physical_name] = nt.[name]
INNER JOIN sys.dm_pdw_nodes pn
    ON  nt.[pdw_node_id] = pn.[pdw_node_id]
INNER JOIN sys.pdw_distributions di
    ON  nt.[distribution_id] = di.[distribution_id]
INNER JOIN sys.dm_pdw_nodes_db_partition_stats nps
    ON nt.[object_id] = nps.[object_id]
    AND nt.[pdw_node_id] = nps.[pdw_node_id]
    AND nt.[distribution_id] = nps.[distribution_id]
LEFT OUTER JOIN (select * from sys.pdw_column_distribution_properties where distribution_ordinal = 1) cdp
    ON t.[object_id] = cdp.[object_id]
LEFT OUTER JOIN sys.columns c
    ON cdp.[object_id] = c.[object_id]
    AND cdp.[column_id] = c.[column_id]
WHERE pn.[type] = 'COMPUTE'
)
, size
AS
(
SELECT
   [execution_time]
,  [database_name]
,  [schema_name]
,  [table_name]
,  [two_part_name]
,  [node_table_name]
,  [node_table_name_seq]
,  [distribution_policy_name]
,  [distribution_column]
,  [distribution_id]
,  [index_type]
,  [index_type_desc]
,  [pdw_node_id]
,  [pdw_node_type]
,  [pdw_node_name]
,  [dist_name]
,  [dist_position]
,  [partition_nmbr]
,  [reserved_space_page_count]
,  [unused_space_page_count]
,  [data_space_page_count]
,  [index_space_page_count]
,  [row_count]
,  ([reserved_space_page_count] * 8.0)                                 AS [reserved_space_KB]
,  ([reserved_space_page_count] * 8.0)/1000                            AS [reserved_space_MB]
,  ([reserved_space_page_count] * 8.0)/1000000                         AS [reserved_space_GB]
,  ([reserved_space_page_count] * 8.0)/1000000000                      AS [reserved_space_TB]
,  ([unused_space_page_count]   * 8.0)                                 AS [unused_space_KB]
,  ([unused_space_page_count]   * 8.0)/1000                            AS [unused_space_MB]
,  ([unused_space_page_count]   * 8.0)/1000000                         AS [unused_space_GB]
,  ([unused_space_page_count]   * 8.0)/1000000000                      AS [unused_space_TB]
,  ([data_space_page_count]     * 8.0)                                 AS [data_space_KB]
,  ([data_space_page_count]     * 8.0)/1000                            AS [data_space_MB]
,  ([data_space_page_count]     * 8.0)/1000000                         AS [data_space_GB]
,  ([data_space_page_count]     * 8.0)/1000000000                      AS [data_space_TB]
,  ([index_space_page_count]  * 8.0)                                   AS [index_space_KB]
,  ([index_space_page_count]  * 8.0)/1000                              AS [index_space_MB]
,  ([index_space_page_count]  * 8.0)/1000000                           AS [index_space_GB]
,  ([index_space_page_count]  * 8.0)/1000000000                        AS [index_space_TB]
FROM base
)
SELECT *
FROM size
;
```

>[!TIP]
> Para mejorar el rendimiento de Synapse SQL, considere la posibilidad de usar **sys.pdw_permanent_table_mappings** en lugar de **sys.pdw_table_mappings** en las tablas de usuario permanentes. Para obtener más información, consulte **[sys.pdw_permanent_table_mappings &#40;Transact-SQL&#41;](/sql/relational-databases/system-catalog-views/sys-pdw-permanent-table-mappings-transact-sql?view=azure-sqldw-latest)** .

### <a name="table-space-summary"></a>Resumen de espacio de tabla

Esta consulta devuelve las filas y el espacio por tabla.  El resumen del espacio de tabla le permite ver qué tablas son las más grandes. También verá si son round robin, replicadas o distribuidas por hash.  Para las tablas distribuidas por hash, la consulta muestra la columna de distribución.  

```sql
SELECT
     database_name
,    schema_name
,    table_name
,    distribution_policy_name
,      distribution_column
,    index_type_desc
,    COUNT(distinct partition_nmbr) as nbr_partitions
,    SUM(row_count)                 as table_row_count
,    SUM(reserved_space_GB)         as table_reserved_space_GB
,    SUM(data_space_GB)             as table_data_space_GB
,    SUM(index_space_GB)            as table_index_space_GB
,    SUM(unused_space_GB)           as table_unused_space_GB
FROM
    dbo.vTableSizes
GROUP BY
     database_name
,    schema_name
,    table_name
,    distribution_policy_name
,      distribution_column
,    index_type_desc
ORDER BY
    table_reserved_space_GB desc
;
```

### <a name="table-space-by-distribution-type"></a>Espacio de tabla por tipo de distribución

```sql
SELECT
     distribution_policy_name
,    SUM(row_count)                as table_type_row_count
,    SUM(reserved_space_GB)        as table_type_reserved_space_GB
,    SUM(data_space_GB)            as table_type_data_space_GB
,    SUM(index_space_GB)           as table_type_index_space_GB
,    SUM(unused_space_GB)          as table_type_unused_space_GB
FROM dbo.vTableSizes
GROUP BY distribution_policy_name
;
```

### <a name="table-space-by-index-type"></a>Espacio de tabla por tipo de índice

```sql
SELECT
     index_type_desc
,    SUM(row_count)                as table_type_row_count
,    SUM(reserved_space_GB)        as table_type_reserved_space_GB
,    SUM(data_space_GB)            as table_type_data_space_GB
,    SUM(index_space_GB)           as table_type_index_space_GB
,    SUM(unused_space_GB)          as table_type_unused_space_GB
FROM dbo.vTableSizes
GROUP BY index_type_desc
;
```

### <a name="distribution-space-summary"></a>Resumen de espacio de distribución

```sql
SELECT
    distribution_id
,    SUM(row_count)                as total_node_distribution_row_count
,    SUM(reserved_space_MB)        as total_node_distribution_reserved_space_MB
,    SUM(data_space_MB)            as total_node_distribution_data_space_MB
,    SUM(index_space_MB)           as total_node_distribution_index_space_MB
,    SUM(unused_space_MB)          as total_node_distribution_unused_space_MB
FROM dbo.vTableSizes
GROUP BY     distribution_id
ORDER BY    distribution_id
;
```

## <a name="next-steps"></a>Pasos siguientes

Después de crear las tablas para el almacenamiento de datos, el paso siguiente es cargar datos en la tabla.  Para obtener un tutorial de carga, consulte [Carga de datos en el grupo de SQL](../sql-data-warehouse/load-data-wideworldimportersdw.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#load-the-data-into-sql-pool).
