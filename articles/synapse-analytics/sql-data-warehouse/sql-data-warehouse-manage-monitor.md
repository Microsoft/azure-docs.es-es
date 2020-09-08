---
title: Supervisión de la carga de trabajo de grupos de SQL mediante DMV
description: Obtenga información sobre cómo supervisar la carga de trabajo de grupos SQL de Azure Synapse Analytics y la ejecución de consultas mediante DMV.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/24/2020
ms.author: rortloff
ms.reviewer: igorstan
ms.custom: synapse-analytics
ms.openlocfilehash: 9eb1006bdba6c69136c972359bb13420a04f4180
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89048031"
---
# <a name="monitor-your-azure-synapse-analytics-sql-pool-workload-using-dmvs"></a>Supervisión de la carga de trabajo de grupos de SQL de Azure Synapse Analytics mediante DMV

En este artículo se describe cómo usar las vistas de administración dinámica (DMV) para supervisar la carga de trabajo e investigar la ejecución de la consulta en grupos de SQL.

## <a name="permissions"></a>Permisos

Para consultar las DMV de este artículo, necesita el permiso **VER ESTADO DE BASE DE DATOS** o **CONTROL**. Normalmente la concesión **VER ESTADO DE BASE DE DATOS** es el permiso preferido ya que es mucho más restrictivo.

```sql
GRANT VIEW DATABASE STATE TO myuser;
```

## <a name="monitor-connections"></a>Supervisión de conexiones

Todos los inicios de sesión en el almacenamiento de datos se registran en [sys.dm_pdw_exec_sessions](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-sessions-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Esta DMV contiene los 10.000 últimos inicios de sesión.  El campo session_id es la clave principal y se asigna de forma secuencial para cada nuevo inicio de sesión.

```sql
-- Other Active Connections
SELECT * FROM sys.dm_pdw_exec_sessions where status <> 'Closed' and session_id <> session_id();
```

## <a name="monitor-query-execution"></a>Supervisión de ejecuciones de consultas

Todas las consultas ejecutadas en el grupo de SQL se registran en [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Esta DMV contiene las últimas 10.000 consultas ejecutadas.  El campo request_id identifica cada consulta de forma única y es la clave principal de esta DMV.  Este campo se asigna de forma secuencial para cada nueva consulta y lleva el prefijo QID, que representa el identificador de consulta.  Al consultar en esta DMV sobre un campo session_id determinado, se muestran todas las consultas de un inicio de sesión concreto.

> [!NOTE]
> Los procedimientos almacenados utilizan varios identificadores de solicitud.  Los identificadores de solicitud se asignan en orden secuencial.

Estos son los pasos que deben seguirse para investigar los planes de ejecución de consultas y las horas de una consulta determinada.

### <a name="step-1-identify-the-query-you-wish-to-investigate"></a>PASO 1: Identificar la consulta que quiere investigar

```sql
-- Monitor active queries
SELECT *
FROM sys.dm_pdw_exec_requests
WHERE status not in ('Completed','Failed','Cancelled')
  AND session_id <> session_id()
ORDER BY submit_time DESC;

-- Find top 10 queries longest running queries
SELECT TOP 10 *
FROM sys.dm_pdw_exec_requests
ORDER BY total_elapsed_time DESC;

```

En los resultados de la consulta anterior, **fíjese en el id. de solicitud** de la consulta que quiere investigar.

Las consultas en estado **suspendido** se pueden poner en cola si hay un gran número de consultas activas en ejecución. Estas consultas también aparecen en la consulta [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) con un tipo de UserConcurrencyResourceType. Para obtener información sobre los límites de simultaneidad, consulte [Límites de memoria y simultaneidad](memory-concurrency-limits.md) o [Clases de recursos para administración de carga de trabajo](resource-classes-for-workload-management.md). Las consultas también pueden esperar por otros motivos, como los bloqueos de objetos.  Si la consulta está esperando un recurso, consulte la sección [Supervisión de consultas en espera](#monitor-waiting-queries) más adelante en este artículo.

Para simplificar la búsqueda de una consulta en la tabla [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), use [LABEL](/sql/t-sql/queries/option-clause-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) para asignar un comentario a la consulta que se pueda buscar en la vista sys.dm_pdw_exec_requests.

```sql
-- Query with Label
SELECT *
FROM sys.tables
OPTION (LABEL = 'My Query')
;

-- Find a query with the Label 'My Query'
-- Use brackets when querying the label column, as it it a key word
SELECT  *
FROM    sys.dm_pdw_exec_requests
WHERE   [label] = 'My Query';
```

### <a name="step-2-investigate-the-query-plan"></a>PASO 2: Investigar el plan de consulta

Use el identificador de la solicitud para recuperar el plan de SQL distribuido (DSQL) de la consulta desde [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

```sql
-- Find the distributed query plan steps for a specific query.
-- Replace request_id with value from Step 1.

SELECT * FROM sys.dm_pdw_request_steps
WHERE request_id = 'QID####'
ORDER BY step_index;
```

Si un plan DSQL tarda más de lo esperado, es posible que sea un plan complejo con muchos pasos DSQL o con un solo paso que tarda mucho tiempo.  Si el plan tiene muchos pasos con varias operaciones de movimiento, considere la posibilidad de optimizar las distribuciones de la tabla para reducir el movimiento de datos. En el artículo [Distribución de tablas](sql-data-warehouse-tables-distribute.md) se explica por qué los datos deben moverse para resolver una consulta. En el artículo también se explican algunas estrategias de distribución para minimizar el movimiento de datos.

Para investigar más detalles acerca de un solo paso, compruebe la columna *operation_type* del paso de consulta de larga ejecución y anote el valor de **Índice de pasos**:

* Vaya al paso 3 para **operaciones SQL**: OnOperation, RemoteOperation, ReturnOperation.
* Vaya al paso 4 para **operaciones de movimiento de datos**: ShuffleMoveOperation, BroadcastMoveOperation, TrimMoveOperation, PartitionMoveOperation, MoveOperation, CopyOperation.

### <a name="step-3-investigate-sql-on-the-distributed-databases"></a>PASO 3: Investigar SQL en las bases de datos distribuidas

Use el identificador de solicitud y el índice de pasos para recuperar información de [sys.dm_pdw_sql_requests](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), que contiene detalles sobre la ejecución del paso de la consulta en todas las instancias distribuidas.

```sql
-- Find the distribution run times for a SQL step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_sql_requests
WHERE request_id = 'QID####' AND step_index = 2;
```

Si la consulta se está ejecutando, se puede utilizar [DBCC PDW_SHOWEXECUTIONPLAN](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) para recuperar el plan estimado de SQL Server de la caché de planes de SQL Server para el paso que se está ejecutando en una distribución particular.

```sql
-- Find the SQL Server execution plan for a query running on a specific SQL pool or control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(1, 78);
```

### <a name="step-4-investigate-data-movement-on-the-distributed-databases"></a>PASO 4: Investigar el movimiento de datos en las bases de datos distribuidas

Use el identificador de solicitud y el índice de paso para recuperar información sobre el paso de movimiento de datos que se ejecuta en cada distribución desde [sys.dm_pdw_dms_workers](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-dms-workers-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

```sql
-- Find information about all the workers completing a Data Movement Step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_dms_workers
WHERE request_id = 'QID####' AND step_index = 2;
```

* Compruebe la columna *total_elapsed_time* para ver si una distribución determinada tarda bastante más que otras en el movimiento de datos.
* Para la distribución de larga ejecución, compruebe la columna *rows_processed* para ver si el número de filas que se mueven desde esa distribución es mucho mayor que para las demás. En ese caso, este hallazgo puede indicar un sesgo de los datos subyacentes.

Si la consulta se está ejecutando, puede usar [DBCC PDW_SHOWEXECUTIONPLAN](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) para recuperar el plan estimado de SQL Server de la caché de planes de SQL Server para el paso de SQL que se está ejecutando actualmente en una distribución particular.

```sql
-- Find the SQL Server estimated plan for a query running on a specific SQL pool Compute or control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(55, 238);
```

<a name="waiting"></a>

## <a name="monitor-waiting-queries"></a>Supervisión de consultas en espera

Si observa que la consulta no avanza porque está esperando un recurso, a continuación puede encontrar una consulta que muestra todos los recursos que está esperando una consulta.

```sql
-- Find queries
-- Replace request_id with value from Step 1.

SELECT waits.session_id,
      waits.request_id,  
      requests.command,
      requests.status,
      requests.start_time,  
      waits.type,
      waits.state,
      waits.object_type,
      waits.object_name
FROM   sys.dm_pdw_waits waits
   JOIN  sys.dm_pdw_exec_requests requests
   ON waits.request_id=requests.request_id
WHERE waits.request_id = 'QID####'
ORDER BY waits.object_name, waits.object_type, waits.state;
```

Si la consulta espera activamente recursos de otra consulta, el estado será **AcquireResources**.  Si la consulta tiene todos los recursos necesarios, el estado será **Concedido**.

## <a name="monitor-tempdb"></a>Supervisión de tempdb

Tempdb se usa para almacenar resultados intermedios durante la ejecución de la consulta. Un uso alto de la base de datos tempdb puede dar lugar a una disminución del rendimiento de la consulta. Para cada DW100c que se configure, se asignan 399 GB de espacio de tempdb (DW1000c tendría 3,99 TB de espacio total de tempdb).  A continuación se muestran sugerencias para supervisar el uso de tempdb y para reducir el uso de tempdb en las consultas.

### <a name="monitoring-tempdb-with-views"></a>Supervisión de tempdb con vistas

Para supervisar el uso de tempdb, instale primero la vista [microsoft.vw_sql_requests](https://github.com/Microsoft/sql-data-warehouse-samples/blob/master/solutions/monitoring/scripts/views/microsoft.vw_sql_requests.sql) desde el [kit de herramientas de Microsoft para grupos de SQL](https://github.com/Microsoft/sql-data-warehouse-samples/tree/master/solutions/monitoring). A continuación, puede ejecutar la consulta siguiente para ver el uso de tempdb por nodo para todas las consultas ejecutadas:

```sql
-- Monitor tempdb
SELECT
    sr.request_id,
    ssu.session_id,
    ssu.pdw_node_id,
    sr.command,
    sr.total_elapsed_time,
    es.login_name AS 'LoginName',
    DB_NAME(ssu.database_id) AS 'DatabaseName',
    (es.memory_usage * 8) AS 'MemoryUsage (in KB)',
    (ssu.user_objects_alloc_page_count * 8) AS 'Space Allocated For User Objects (in KB)',
    (ssu.user_objects_dealloc_page_count * 8) AS 'Space Deallocated For User Objects (in KB)',
    (ssu.internal_objects_alloc_page_count * 8) AS 'Space Allocated For Internal Objects (in KB)',
    (ssu.internal_objects_dealloc_page_count * 8) AS 'Space Deallocated For Internal Objects (in KB)',
    CASE es.is_user_process
    WHEN 1 THEN 'User Session'
    WHEN 0 THEN 'System Session'
    END AS 'SessionType',
    es.row_count AS 'RowCount'
FROM sys.dm_pdw_nodes_db_session_space_usage AS ssu
    INNER JOIN sys.dm_pdw_nodes_exec_sessions AS es ON ssu.session_id = es.session_id AND ssu.pdw_node_id = es.pdw_node_id
    INNER JOIN sys.dm_pdw_nodes_exec_connections AS er ON ssu.session_id = er.session_id AND ssu.pdw_node_id = er.pdw_node_id
    INNER JOIN microsoft.vw_sql_requests AS sr ON ssu.session_id = sr.spid AND ssu.pdw_node_id = sr.pdw_node_id
WHERE DB_NAME(ssu.database_id) = 'tempdb'
    AND es.session_id <> @@SPID
    AND es.login_name <> 'sa'
ORDER BY sr.request_id;
```

Si tiene una consulta que consume una gran cantidad de memoria o ha recibido un mensaje de error relacionado con la asignación de tempdb, podría deberse a una instrucción [CREATE TABLE AS SELECT (CTAS)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) o [INSERT SELECT](/sql/t-sql/statements/insert-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) muy grande que da error en la operación de movimiento de datos final. Normalmente esto se identifica como una operación ShuffleMove en el plan de consulta distribuida justo antes de la instrucción INSERT SELECT final.  Use [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) para supervisar las operaciones de ShuffleMove.

La solución más común consiste en dividir la instrucción CTAS o INSERT SELECT en varias instrucciones de carga para que el volumen de datos no supere el límite de tempdb de 1 TB por nodo. También puede escalar el clúster a un tamaño mayor, lo cual repartirá el tamaño de tempdb entre varios nodos, reduciendo el tamaño de tempdb en cada uno de los nodos.

Además de las instrucciones CTAS e INSERT SELECT, las consultas grandes y complejas que se ejecutan con memoria insuficiente pueden desbordarse en tempdb, lo que provoca un error en las consultas.  Considere la posibilidad de realizar la ejecución con una [clase de recurso](resource-classes-for-workload-management.md) mayor para evitar el desbordamiento en tempdb.

## <a name="monitor-memory"></a>Supervisión de memoria

La causa principal de los problemas relacionados con el rendimiento lento y la falta de memoria puede ser la memoria. Si observa que el uso de memoria de SQL Server llega a su limite durante la ejecución de consultas, considere la posibilidad de escalar el almacenamiento de datos.

La consulta siguiente devuelve el uso y la presión de memoria de SQL Server por nodo:

```sql
-- Memory consumption
SELECT
  pc1.cntr_value as Curr_Mem_KB,
  pc1.cntr_value/1024.0 as Curr_Mem_MB,
  (pc1.cntr_value/1048576.0) as Curr_Mem_GB,
  pc2.cntr_value as Max_Mem_KB,
  pc2.cntr_value/1024.0 as Max_Mem_MB,
  (pc2.cntr_value/1048576.0) as Max_Mem_GB,
  pc1.cntr_value * 100.0/pc2.cntr_value AS Memory_Utilization_Percentage,
  pc1.pdw_node_id
FROM
-- pc1: current memory
sys.dm_pdw_nodes_os_performance_counters AS pc1
-- pc2: total memory allowed for this SQL instance
JOIN sys.dm_pdw_nodes_os_performance_counters AS pc2
ON pc1.object_name = pc2.object_name AND pc1.pdw_node_id = pc2.pdw_node_id
WHERE
pc1.counter_name = 'Total Server Memory (KB)'
AND pc2.counter_name = 'Target Server Memory (KB)'
```

## <a name="monitor-transaction-log-size"></a>Supervisión del tamaño del registro de transacciones

La siguiente consulta devuelve el tamaño del registro de transacciones en cada distribución. Si uno de los archivos de registro está llegando a 160 GB, debería considerar la posibilidad de escalar verticalmente su instancia o limitar el tamaño de transacción.

```sql
-- Transaction log size
SELECT
  instance_name as distribution_db,
  cntr_value*1.0/1048576 as log_file_size_used_GB,
  pdw_node_id
FROM sys.dm_pdw_nodes_os_performance_counters
WHERE
instance_name like 'Distribution_%'
AND counter_name = 'Log File(s) Used Size (KB)'
```

## <a name="monitor-transaction-log-rollback"></a>Supervisión de la reversión del registro de transacciones

Si las consultas producen errores o tardan mucho tiempo en continuar, puede comprobar y supervisar si tiene alguna reversión en las transacciones.

```sql
-- Monitor rollback
SELECT
    SUM(CASE WHEN t.database_transaction_next_undo_lsn IS NOT NULL THEN 1 ELSE 0 END),
    t.pdw_node_id,
    nod.[type]
FROM sys.dm_pdw_nodes_tran_database_transactions t
JOIN sys.dm_pdw_nodes nod ON t.pdw_node_id = nod.pdw_node_id
GROUP BY t.pdw_node_id, nod.[type]
```

## <a name="monitor-polybase-load"></a>Supervisión de la carga de PolyBase

La siguiente consulta proporciona una estimación aproximada del progreso de la carga. La consulta solo muestra los archivos que se están procesando actualmente.

```sql

-- To track bytes and files
SELECT
    r.command,
    s.request_id,
    r.status,
    count(distinct input_name) as nbr_files,
    sum(s.bytes_processed)/1024/1024/1024 as gb_processed
FROM
    sys.dm_pdw_exec_requests r
    inner join sys.dm_pdw_dms_external_work s
        on r.request_id = s.request_id
GROUP BY
    r.command,
    s.request_id,
    r.status
ORDER BY
    nbr_files desc,
    gb_processed desc;
```

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de las DMV, consulte [Vistas del sistema](../sql/reference-tsql-system-views.md).
