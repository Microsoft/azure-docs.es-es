---
title: 'Tablas del sistema en Azure Database for PostgreSQL: Hiperescala (Citus)'
description: Metadatos para la ejecución de consultas distribuidas
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: reference
ms.date: 08/10/2020
ms.openlocfilehash: c11fd7a9cb6fdd3eb976d0b9e6a91fdc69bf9fba
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88136652"
---
# <a name="system-tables-and-views"></a>Selección de tablas y vistas

Hiperescala (Citus) crea y mantiene tablas especiales que contienen información sobre los datos distribuidos en el grupo de servidores. El nodo de coordinación consulta estas tablas al planear cómo ejecutar consultas en los nodos de trabajo.

## <a name="coordinator-metadata"></a>Metadatos del coordinador

Hiperescala (Citus) divide cada tabla distribuida en varias particiones lógicas basadas en la columna de distribución. El coordinador entonces mantiene las tablas de metadatos para realizar un seguimiento de las estadísticas y la información sobre el estado y la ubicación de estas particiones.

En esta sección, se describe cada una de estas tablas de metadatos y su esquema.
Puede ver y consultar estas tablas mediante SQL después de iniciar sesión en el nodo de coordinación.

> [!NOTE]
>
> Es posible que los grupos de servidores de Hiperescala que ejecutan versiones anteriores del motor de Citus no ofrezcan todas las tablas que se enumeran a continuación.

### <a name="partition-table"></a>Tabla de particiones

La tabla pg\_dist\_partition almacena metadatos sobre qué tablas de la base de datos se distribuyen. Para cada tabla distribuida, también almacena información sobre el método de distribución e información detallada sobre la columna de distribución.

| Nombre         | Tipo     | Descripción                                                                                                                                                                                                                                           |
|--------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| logicalrelid | regclass | Tabla distribuida a la que corresponde esta fila. Este valor hace referencia a la columna relfilenode de la tabla de catálogo del sistema pg_class.                                                                                                                   |
| partmethod   | char     | Método utilizado para la creación de particiones o la distribución. Se anexan los valores de esta columna correspondientes a distintos métodos de distribución: "a", por hash: "h", por tabla de referencia: "n"                                                                          |
| partkey      | text     | Información detallada sobre la columna de distribución, que incluye el número de columna, el tipo y otra información relevante.                                                                                                                                      |
| colocationid | integer  | Grupo de coubicación al que pertenece esta tabla. Las tablas del mismo grupo permiten combinaciones colocadas y resúmenes distribuidos entre otras optimizaciones. Este valor hace referencia a la columna colocationid de la tabla pg_dist_colocation.                      |
| repmodel     | char     | Método utilizado para la replicación de datos. Los valores de esta columna correspondientes a diferentes métodos de replicación son: Replicación basada en instrucciones de Citus: "c ", replicación streaming de PostgreSQL: "s", confirmación en dos fases (para tablas de referencia): "t" |

```
SELECT * from pg_dist_partition;
 logicalrelid  | partmethod |                                                        partkey                                                         | colocationid | repmodel 
---------------+------------+------------------------------------------------------------------------------------------------------------------------+--------------+----------
 github_events | h          | {VAR :varno 1 :varattno 4 :vartype 20 :vartypmod -1 :varcollid 0 :varlevelsup 0 :varnoold 1 :varoattno 4 :location -1} |            2 | c
 (1 row)
```

### <a name="shard-table"></a>Tabla de particiones

La tabla pg\_dist\_shard almacena metadatos sobre las particiones individuales de una tabla. Pg_dist_shard tiene información sobre la tabla de distribución a la que pertenecen las particiones y estadísticas sobre la columna de distribución de las particiones.
En el caso de las tablas con distribución por anexión, estas estadísticas corresponden a los valores mínimo y máximo de la columna de distribución. En el caso de las tablas con distribución por hash, corresponden a los intervalos de tokens hash asignados a la partición. Estas estadísticas se usan para la eliminación de particiones no relacionadas durante las consultas SELECT.

| Nombre          | Tipo     | Descripción                                                                                                                                                                                  |
|---------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| logicalrelid  | regclass | Tabla distribuida a la que corresponde esta fila. Este valor hace referencia a la columna relfilenode de la tabla de catálogo del sistema pg_class.                                                          |
| shardid       | bigint   | Identificador único global asignado a la partición.                                                                                                                                           |
| shardstorage  | char     | Tipo de almacenamiento utilizado para la partición. Los distintos tipos de almacenamiento se describen en la tabla siguiente.                                                                                               |
| shardminvalue | text     | En el caso de las tablas con distribución por anexión, el valor mínimo de la columna de distribución de esta partición (inclusive). En el caso de las tablas con distribución por hash, el valor mínimo del token hash asignado a la partición (inclusive). |
| shardmaxvalue | text     | En el caso de las tablas con distribución por anexión, el valor máximo de la columna de distribución de esta partición (inclusive). En el caso de las tablas con distribución por hash, el valor máximo del token hash asignado a la partición (inclusive). |

```
SELECT * from pg_dist_shard;
 logicalrelid  | shardid | shardstorage | shardminvalue | shardmaxvalue 
---------------+---------+--------------+---------------+---------------
 github_events |  102026 | t            | 268435456     | 402653183
 github_events |  102027 | t            | 402653184     | 536870911
 github_events |  102028 | t            | 536870912     | 671088639
 github_events |  102029 | t            | 671088640     | 805306367
 (4 rows)
```

#### <a name="shard-storage-types"></a>Tipos de almacenamiento de particiones

La columna shardstorage de pg\_dist\_shard indica el tipo de almacenamiento utilizado para la partición. A continuación se muestra una breve descripción de los distintos tipos de almacenamiento de particiones y su representación.

| Tipo de almacenamiento | Valor de shardstorage | Descripción                                                                        |
|--------------|--------------------|------------------------------------------------------------------------------------|
| TABLE        | "t"                | Indica que la partición almacena datos que pertenecen a una tabla distribuida normal.         |
| COLUMNAR     | "c"                | Indica que la partición almacena datos en columnas. (Se usa en tablas cstore_fdw distribuidas). |
| FOREIGN      | "f"                | Indica que la partición almacena datos externos. (Se usa en tablas file_fdw distribuidas).    |

### <a name="shard-placement-table"></a>Tabla de selecciones de ubicación de partición

La tabla pg\_dist\_placement realiza un seguimiento de la ubicación de las réplicas de la partición en los nodos de trabajo. Cada réplica de una partición asignada a un nodo específico se denomina selección de ubicación de partición. Esta tabla almacena información sobre el estado y la ubicación de cada selección de ubicación de partición.

| Nombre        | Tipo   | Descripción                                                                                                                               |
|-------------|--------|-------------------------------------------------------------------------------------------------------------------------------------------|
| shardid     | bigint | Identificador de la partición asociada a esta selección de ubicación. Este valor hace referencia a la columna shardid de la tabla de catálogo pg_dist_shard.             |
| shardstate  | int    | Describe el estado de esta selección de ubicación. Los distintos estados de una partición se describen en la sección siguiente.                                         |
| shardlength | bigint | En el caso de las tablas con distribución por anexión, el tamaño de la selección de ubicación de partición en el nodo de trabajo (en bytes). En el caso de las tablas con distribución por hash, cero.            |
| placementid | bigint | Identificador único generado automáticamente para cada selección de ubicación individual.                                                                           |
| groupid     | int    | Denota un grupo de un servidor principal y ninguno o más servidores secundarios cuando se usa el modelo de replicación streaming. |

```
SELECT * from pg_dist_placement;
  shardid | shardstate | shardlength | placementid | groupid
 ---------+------------+-------------+-------------+---------
   102008 |          1 |           0 |           1 |       1
   102008 |          1 |           0 |           2 |       2
   102009 |          1 |           0 |           3 |       2
   102009 |          1 |           0 |           4 |       3
   102010 |          1 |           0 |           5 |       3
   102010 |          1 |           0 |           6 |       4
   102011 |          1 |           0 |           7 |       4
```

#### <a name="shard-placement-states"></a>Estados de selección de ubicación de partición

Hiperescala (Citus) administra el estado de las particiones por selección de ubicación. Si una selección de ubicación coloca el sistema en un estado incoherente, Citus la marca automáticamente como no disponible. El estado de la selección de ubicación se registra en la tabla pg_dist_shard_placement, dentro de la columna shardstate. Esta es una breve descripción de los distintos estados de selección de ubicación de partición:

| Nombre del estado | Shardstate value | Descripción                                                                                                                                                                                                                                                                                                                                                                                                                         |
|------------|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| FINALIZED  | 1                | Estado en el que se crean las particiones de estado. Las ubicaciones de selección de partición en este estado se consideran actualizadas y se usan en el planeamiento y la ejecución de consultas.                                                                                                                                                                                                                                                                                 |
| INACTIVE   | 3                | Las ubicaciones de selección de partición en este estado se consideran inactivas debido a que no están sincronizadas con otras réplicas de la misma partición. El estado puede producirse cuando se produce un error en una operación de anexión, modificación (INSERT, UPDATE, DELETE) o DDL para esta selección de ubicación. El planeador de consultas omitirá las selecciones de ubicación en este estado durante el planeamiento y la ejecución. Los usuarios pueden sincronizar los datos de estas particiones con una réplica finalizada como actividad en segundo plano. |
| TO_DELETE  | 4                | Si Citus intenta quitar una selección de ubicación de partición en respuesta a una llamada master_apply_delete_command y se produce un error, la selección de ubicación pasa a este estado. Los usuarios pueden entonces eliminar estas particiones como actividad en segundo plano posterior.                                                                                                                                                                                                              |

### <a name="worker-node-table"></a>Tabla de nodos de trabajo

La tabla pg\_dist\_node contiene información sobre los nodos de trabajo del clúster.

| Nombre             | Tipo    | Descripción                                                                                                                                                                                |
|------------------|---------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| nodeid           | int     | Identificador generado automáticamente para un nodo individual.                                                                                                                                          |
| groupid          | int     | Identificador utilizado para denotar un grupo de un servidor principal y ninguno o más servidores secundarios cuando se usa el modelo de replicación streaming. De manera predeterminada, es igual al de nodeid.         |
| nodename         | text    | Nombre de host o dirección IP del nodo de trabajo de PostgreSQL.                                                                                                                                     |
| nodeport         | int     | Número del puerto en el que escucha el nodo de trabajo de PostgreSQL.                                                                                                                              |
| noderack         | text    | (Opcional) Información de selección de ubicación del bastidor del nodo de trabajo.                                                                                                                                 |
| hasmetadata      | boolean | Reservado para uso interno.                                                                                                                                                                 |
| isactive         | boolean | Indica si el nodo está activo y acepta selecciones de ubicación de partición.                                                                                                                                     |
| noderole         | text    | Indica si el nodo es principal o secundario.                                                                                                                                                 |
| nodecluster      | text    | Nombre del clúster que contiene este nodo.                                                                                                                                               |
| shouldhaveshards | boolean | Si es "false", las particiones se quitarán del nodo (se purgan) al reequilibrar y no se colocarán particiones de las nuevas tablas distribuidas en el nodo, a menos que estén coubicadas con particiones que ya estén allí. |

```
SELECT * from pg_dist_node;
 nodeid | groupid | nodename  | nodeport | noderack | hasmetadata | isactive | noderole | nodecluster | shouldhaveshards
--------+---------+-----------+----------+----------+-------------+----------+----------+-------------+------------------
      1 |       1 | localhost |    12345 | default  | f           | t        | primary  | default     | t
      2 |       2 | localhost |    12346 | default  | f           | t        | primary  | default     | t
      3 |       3 | localhost |    12347 | default  | f           | t        | primary  | default     | t
(3 rows)
```

### <a name="distributed-object-table"></a>Tabla de objetos distribuidos

La tabla citus.pg\_dist\_object contiene una lista de objetos tales como los tipos y funciones que se han creado en el nodo de coordinación y propagado a los nodos de trabajo. Cuando un administrador agrega nuevos nodos de trabajo al clúster, Hiperescala (Citus) crea automáticamente copias de los objetos distribuidos en los nuevos nodos (en el orden correcto para satisfacer las dependencias de los objetos).

| Nombre                        | Tipo    | Descripción                                          |
|-----------------------------|---------|------------------------------------------------------|
| classid                     | oid     | Clase del objeto distribuido                      |
| objid                       | oid     | Identificador de objeto del objeto distribuido                  |
| objsubid                    | integer | Subidentificador de objeto del objeto distribuido, por ejemplo, attnum |
| type                        | text    | Parte de la dirección estable usada durante las actualizaciones de PG   |
| object_names                | text[]  | Parte de la dirección estable usada durante las actualizaciones de PG   |
| object_args                 | text[]  | Parte de la dirección estable usada durante las actualizaciones de PG   |
| distribution_argument_index | integer | Solo válido para funciones o procedimientos distribuidos      |
| colocationid                | integer | Solo válido para funciones o procedimientos distribuidos      |

Las \"direcciones estables\" identifican de forma única los objetos independientemente de un servidor específico. Hiperescala (Citus) realiza un seguimiento de los objetos durante una actualización de PostgreSQL mediante direcciones estables creadas con la función [pg\_identify\_object\_as\_address()](https://www.postgresql.org/docs/current/functions-info.html#FUNCTIONS-INFO-OBJECT-TABLE).

Este es un ejemplo de cómo `create_distributed_function()` agrega entradas a la tabla `citus.pg_dist_object`:

```psql
CREATE TYPE stoplight AS enum ('green', 'yellow', 'red');

CREATE OR REPLACE FUNCTION intersection()
RETURNS stoplight AS $$
DECLARE
        color stoplight;
BEGIN
        SELECT *
          FROM unnest(enum_range(NULL::stoplight)) INTO color
         ORDER BY random() LIMIT 1;
        RETURN color;
END;
$$ LANGUAGE plpgsql VOLATILE;

SELECT create_distributed_function('intersection()');

-- will have two rows, one for the TYPE and one for the FUNCTION
TABLE citus.pg_dist_object;
```

```
-[ RECORD 1 ]---------------+------
classid                     | 1247
objid                       | 16780
objsubid                    | 0
type                        |
object_names                |
object_args                 |
distribution_argument_index |
colocationid                |
-[ RECORD 2 ]---------------+------
classid                     | 1255
objid                       | 16788
objsubid                    | 0
type                        |
object_names                |
object_args                 |
distribution_argument_index |
colocationid                |
```

### <a name="colocation-group-table"></a>Tabla de grupos de coubicación

La tabla pg\_dist\_colocation contiene información sobre las tablas\' en las que las particiones se deben colocar juntas o [coubicadas](concepts-hyperscale-colocation.md).
Cuando dos tablas están en el mismo grupo de coubicación, Hiperescala (Citus) garantiza que las particiones con los mismos valores de partición se colocarán en los mismos nodos de trabajo.
La coubicación permite usar optimizaciones de combinación, determinadas acumulaciones distribuidas y compatibilidad con claves externas. La coubicación de particiones se deduce cuando el número de particiones, los factores de replicación y los tipos de columna de partición coinciden entre dos tablas; aunque, si se quiere, se puede especificar un grupo de coubicación personalizado al crear una tabla distribuida.

| Nombre                   | Tipo | Descripción                                                                   |
|------------------------|------|-------------------------------------------------------------------------------|
| colocationid           | int  | Identificador único del grupo de coubicación al que corresponde esta fila.          |
| shardcount             | int  | Número de particiones para todas las tablas de este grupo de coubicación.                          |
| replicationfactor      | int  | Factor de replicación para todas las tablas de este grupo de coubicación.                  |
| distributioncolumntype | oid  | Tipo de la columna de distribución para todas las tablas de este grupo de coubicación. |

```
SELECT * from pg_dist_colocation;
  colocationid | shardcount | replicationfactor | distributioncolumntype 
 --------------+------------+-------------------+------------------------
             2 |         32 |                 2 |                     20
  (1 row)
```

### <a name="rebalancer-strategy-table"></a>Tabla de estrategias de reequilibrador

En esta tabla se definen estrategias que [rebalance_table_shards](reference-hyperscale-functions.md#rebalance_table_shards) puede utilizar para determinar a dónde se deben trasladar las particiones.

| Nombre                           | Tipo    | Descripción                                                                                                                                       |
|--------------------------------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| default_strategy               | boolean | Indica si rebalance_table_shards debe elegir esta estrategia de forma predeterminada. Use citus_set_default_rebalance_strategy para actualizar esta columna.             |
| shard_cost_function            | regproc | Identificador de una función de costo, que debe tomar un shardid como bigint y devolver su noción de costo como tipo real.                                |
| node_capacity_function         | regproc | Identificador de una función de capacidad, que debe tomar un nodeid como int y devolver su noción de capacidad de nodo como tipo real.                          |
| shard_allowed_on_node_function | regproc | Identificador de una función que determinado shardid bigint (y nodeidarg int) devuelve como valor booleano si Citus puede almacenar la partición en el nodo. |
| default_threshold              | float4  | Umbral según el que se considera que un nodo está demasiado lleno o demasiado vacío, lo que determina cuándo rebalance_table_shards debe intentar trasladar las particiones.                    |
| minimum_threshold              | float4  | Medida de seguridad para evitar que el argumento de umbral de rebalance_table_shards () se establezca demasiado bajo                                                  |

Una instalación de Hiperescala (Citus) incluye estas estrategias en la tabla:

```postgresql
SELECT * FROM pg_dist_rebalance_strategy;
```

```
-[ RECORD 1 ]-------------------+-----------------------------------
Name                            | by_shard_count
default_strategy                | true
shard_cost_function             | citus_shard_cost_1
node_capacity_function          | citus_node_capacity_1
shard_allowed_on_node_function  | citus_shard_allowed_on_node_true
default_threshold               | 0
minimum_threshold               | 0
-[ RECORD 2 ]-------------------+-----------------------------------
Name                            | by_disk_size
default_strategy                | false
shard_cost_function             | citus_shard_cost_by_disk_size
node_capacity_function          | citus_node_capacity_1
shard_allowed_on_node_function  | citus_shard_allowed_on_node_true
default_threshold               | 0.1
minimum_threshold               | 0.01
```

La estrategia predeterminada, `by_shard_count`, asigna a cada partición el mismo costo. Su efecto es igualar el número de particiones entre los nodos. La otra estrategia predefinida, `by_disk_size`, asigna un costo a cada partición que corresponde a su tamaño de disco en bytes más el de las particiones que se coubican con ella. El tamaño del disco se calcula mediante `pg_total_relation_size`, por lo que incluye índices. Esta estrategia intenta lograr el mismo espacio en disco en todos los nodos. Observe el umbral de 0,1: evita el movimiento de particiones innecesario debido a diferencias insignificantes en el espacio en disco.

#### <a name="creating-custom-rebalancer-strategies"></a>Creación de estrategias de reequilibrador personalizadas

Estos son ejemplos de funciones que se pueden usar en las nuevas estrategias de reequilibrador de particiones y se registran en [pg_dist_rebalance_strategy](reference-hyperscale-metadata.md?#rebalancer-strategy-table) con la función [citus_add_rebalance_strategy](reference-hyperscale-functions.md#citus_add_rebalance_strategy).

-   Establecimiento de una excepción de capacidad de nodo por patrón de nombre de host:

    ```postgresql
    CREATE FUNCTION v2_node_double_capacity(nodeidarg int)
        RETURNS boolean AS $$
        SELECT
            (CASE WHEN nodename LIKE '%.v2.worker.citusdata.com' THEN 2 ELSE 1 END)
        FROM pg_dist_node where nodeid = nodeidarg
        $$ LANGUAGE sql;
    ```

-   Reequilibrio por número de consultas que van a una partición, que [citus_stat_statements](reference-hyperscale-metadata.md#query-statistics-table) mide:

    ```postgresql
    -- example of shard_cost_function

    CREATE FUNCTION cost_of_shard_by_number_of_queries(shardid bigint)
        RETURNS real AS $$
        SELECT coalesce(sum(calls)::real, 0.001) as shard_total_queries
        FROM citus_stat_statements
        WHERE partition_key is not null
            AND get_shard_id_for_distribution_column('tab', partition_key) = shardid;
    $$ LANGUAGE sql;
    ```

-   Aislamiento de una partición específica (10000) en un nodo (dirección \'10.0.0.1\'):

    ```postgresql
    -- example of shard_allowed_on_node_function

    CREATE FUNCTION isolate_shard_10000_on_10_0_0_1(shardid bigint, nodeidarg int)
        RETURNS boolean AS $$
        SELECT
            (CASE WHEN nodename = '10.0.0.1' THEN shardid = 10000 ELSE shardid != 10000 END)
        FROM pg_dist_node where nodeid = nodeidarg
        $$ LANGUAGE sql;

    -- The next two definitions are recommended in combination with the above function.
    -- This way the average utilization of nodes is not impacted by the isolated shard.
    CREATE FUNCTION no_capacity_for_10_0_0_1(nodeidarg int)
        RETURNS real AS $$
        SELECT
            (CASE WHEN nodename = '10.0.0.1' THEN 0 ELSE 1 END)::real
        FROM pg_dist_node where nodeid = nodeidarg
        $$ LANGUAGE sql;
    CREATE FUNCTION no_cost_for_10000(shardid bigint)
        RETURNS real AS $$
        SELECT
            (CASE WHEN shardid = 10000 THEN 0 ELSE 1 END)::real
        $$ LANGUAGE sql;
    ```

### <a name="query-statistics-table"></a>Tabla de estadísticas de consultas

Hiperescala (Citus) proporciona `citus_stat_statements` para estadísticas sobre cómo se ejecutan las consultas y para quién. Es similar a la vista [pg\_stat\_statements](https://www.postgresql.org/docs/current/static/pgstatstatements.html) de PostgreSQL (y se puede combinar con ella), que realiza un seguimiento de las estadísticas sobre la velocidad de consulta.

Esta vista puede realizar un seguimiento de las consultas a los inquilinos de origen en una aplicación multiinquilino, lo que ayuda a decidir cuándo se debe aislar a los inquilinos.

| Nombre          | Tipo   | Descripción                                                                      |
|---------------|--------|----------------------------------------------------------------------------------|
| queryid       | bigint | identificador (adecuado para combinaciones de pg_stat_statements)                                   |
| userid        | oid    | usuario que ejecutó la consulta                                                           |
| dbid          | oid    | instancia de base de datos de coordinador                                                 |
| Query         | text   | cadena de consulta anonimizada                                                          |
| ejecutor      | text   | ejecutor de Citus usado: adaptive, real-time, task-tracker, router o insert-select |
| partition_key | text   | valor de la columna de distribución en las consultas ejecutadas por enrutador; de lo contrario, NULL               |
| calls         | bigint | número de veces que se ejecutó la consulta                                                |

```sql
-- create and populate distributed table
create table foo ( id int );
select create_distributed_table('foo', 'id');
insert into foo select generate_series(1,100);

-- enable stats
-- pg_stat_statements must be in shared_preload libraries
create extension pg_stat_statements;

select count(*) from foo;
select * from foo where id = 42;

select * from citus_stat_statements;
```

Resultados:

```
-[ RECORD 1 ]-+----------------------------------------------
queryid       | -909556869173432820
userid        | 10
dbid          | 13340
query         | insert into foo select generate_series($1,$2)
executor      | insert-select
partition_key |
calls         | 1
-[ RECORD 2 ]-+----------------------------------------------
queryid       | 3919808845681956665
userid        | 10
dbid          | 13340
query         | select count(*) from foo;
executor      | adaptive
partition_key |
calls         | 1
-[ RECORD 3 ]-+----------------------------------------------
queryid       | 5351346905785208738
userid        | 10
dbid          | 13340
query         | select * from foo where id = $1
executor      | adaptive
partition_key | 42
calls         | 1
```

Advertencias:

-   Los datos de estadísticas no se replican y no sobrevivirán a bloqueos de la base de datos ni a la conmutación por error.
-   Realiza un seguimiento de un número limitado de consultas, establecido por el GUC `pg_stat_statements.max` (valor predeterminado 5000).
-   Para truncar la tabla, se usa la función `citus_stat_statements_reset()`

### <a name="distributed-query-activity"></a>Actividad de consulta distribuida

Hiperescala (Citus) ofrece vistas especiales para ver las consultas y los bloqueos en todo el clúster, incluidas las consultas específicas de particiones que se usan internamente para generar los resultados de las consultas distribuidas.

-   **citus\_dist\_stat\_activity**: muestra las consultas distribuidas que se ejecutan en todos los nodos. Superconjunto de `pg_stat_activity`, que se puede usar dondequiera que se encuentre este último.
-   **citus\_worker\_stat\_activity**: muestra las consultas en los trabajos, incluidas las consultas de fragmentos en particiones individuales.
-   **citus\_lock\_waits**: consultas bloqueadas en todo el clúster.

Las dos primeras vistas incluyen todas las columnas de [pg\_stat\_activity](https://www.postgresql.org/docs/current/static/monitoring-stats.html#PG-STAT-ACTIVITY-VIEW) más el host/puerto del host del trabajo que inició la consulta y el host/puerto del nodo de coordinación del clúster.

Por ejemplo, plantéese la posibilidad de contar las filas de una tabla distribuida:

```postgresql
-- run from worker on localhost:9701

SELECT count(*) FROM users_table;
```

Podemos ver que la consulta aparece en `citus_dist_stat_activity`:

```postgresql
SELECT * FROM citus_dist_stat_activity;

-[ RECORD 1 ]----------+----------------------------------
query_hostname         | localhost
query_hostport         | 9701
master_query_host_name | localhost
master_query_host_port | 9701
transaction_number     | 1
transaction_stamp      | 2018-10-05 13:27:20.691907+03
datid                  | 12630
datname                | postgres
pid                    | 23723
usesysid               | 10
usename                | citus
application\_name      | psql
client\_addr           | 
client\_hostname       | 
client\_port           | -1
backend\_start         | 2018-10-05 13:27:14.419905+03
xact\_start            | 2018-10-05 13:27:16.362887+03
query\_start           | 2018-10-05 13:27:20.682452+03
state\_change          | 2018-10-05 13:27:20.896546+03
wait\_event_type       | Client
wait\_event            | ClientRead
state                  | idle in transaction
backend\_xid           | 
backend\_xmin          | 
query                  | SELECT count(*) FROM users_table;
backend\_type          | client backend
```

Esta consulta requiere información de todas las particiones. Parte de la información se encuentra en la `users_table_102038` de la partición, que se almacena en `localhost:9700`. Podemos ver una consulta que tiene acceso a la partición examinando la vista `citus_worker_stat_activity`:

```postgresql
SELECT * FROM citus_worker_stat_activity;

-[ RECORD 1 ]----------+-----------------------------------------------------------------------------------------
query_hostname         | localhost
query_hostport         | 9700
master_query_host_name | localhost
master_query_host_port | 9701
transaction_number     | 1
transaction_stamp      | 2018-10-05 13:27:20.691907+03
datid                  | 12630
datname                | postgres
pid                    | 23781
usesysid               | 10
usename                | citus
application\_name      | citus
client\_addr           | ::1
client\_hostname       | 
client\_port           | 51773
backend\_start         | 2018-10-05 13:27:20.75839+03
xact\_start            | 2018-10-05 13:27:20.84112+03
query\_start           | 2018-10-05 13:27:20.867446+03
state\_change          | 2018-10-05 13:27:20.869889+03
wait\_event_type       | Client
wait\_event            | ClientRead
state                  | idle in transaction
backend\_xid           | 
backend\_xmin          | 
query                  | COPY (SELECT count(*) AS count FROM users_table_102038 users_table WHERE true) TO STDOUT
backend\_type          | client backend
```

En el campo `query` se muestran los datos que se copian de la partición que se van a contar.

> [!NOTE]
> Si una consulta de enrutador (por ejemplo, un solo inquilino en una aplicación multiinquilino, se ejecuta
> * "SELECT FROM table WHERE tenant_id = X") sin un bloque de transacciones, y luego las columnas master\_query\_host\_name y master\_query\_host\_port serán NULL en citus\_worker\_stat\_activity.

Para ver cómo funciona `citus_lock_waits`, podemos generar manualmente una situación de bloqueo. Configuraremos en primer lugar una tabla de prueba desde el coordinador:

```postgresql
CREATE TABLE numbers AS
  SELECT i, 0 AS j FROM generate_series(1,10) AS i;
SELECT create_distributed_table('numbers', 'i');
```

Luego, con dos sesiones en el coordinador, podemos ejecutar esta secuencia de instrucciones:

```postgresql
-- session 1                           -- session 2
-------------------------------------  -------------------------------------
BEGIN;
UPDATE numbers SET j = 2 WHERE i = 1;
                                       BEGIN;
                                       UPDATE numbers SET j = 3 WHERE i = 1;
                                       -- (this blocks)
```

En la vista `citus_lock_waits` se muestra la situación.

```postgresql
SELECT * FROM citus_lock_waits;

-[ RECORD 1 ]-------------------------+----------------------------------------
waiting_pid                           | 88624
blocking_pid                          | 88615
blocked_statement                     | UPDATE numbers SET j = 3 WHERE i = 1;
current_statement_in_blocking_process | UPDATE numbers SET j = 2 WHERE i = 1;
waiting_node_id                       | 0
blocking_node_id                      | 0
waiting_node_name                     | coordinator_host
blocking_node_name                    | coordinator_host
waiting_node_port                     | 5432
blocking_node_port                    | 5432
```

En este ejemplo, las consultas se originaron en el coordinador, pero la vista también puede mostrar los bloqueos entre consultas que se originan en los trabajos (ejecutados, por ejemplo, con MX de Hiperescala [Citus]).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre cómo algunas [funciones de Hiperescala](reference-hyperscale-functions.md) modifican tablas del sistema
* Revisión de los conceptos de [nodos y tablas](concepts-hyperscale-nodes.md)
