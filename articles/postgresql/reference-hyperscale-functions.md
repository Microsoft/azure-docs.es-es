---
title: Funciones SQL en Hiperescala (Citus) para Azure Database for PostgreSQL
description: Funciones de la API de SQL de Hiperescala (Citus)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: reference
ms.date: 08/10/2020
ms.openlocfilehash: eaada1981929cec890ce3c8ca89fe47393730b05
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88136679"
---
# <a name="functions-in-the-hyperscale-citus-sql-api"></a>Funciones de la API de SQL de Hiperescala (Citus)

Esta sección contiene información de referencia para las funciones definidas por el usuario proporcionadas por Hiperescala (Citus). Estas funciones ayudan a proporcionar una funcionalidad distribuida adicional a Hiperescala (Citus) que no sean los comandos SQL estándar.

> [!NOTE]
>
> Es posible que los grupos de servidores de Hiperescala que ejecutan versiones anteriores del motor de Citus no ofrezcan todas las funciones que se enumeran a continuación.

## <a name="table-and-shard-ddl"></a>DDL de tabla y de partición

### <a name="create_distributed_table"></a>create\_distributed\_table

La función create\_distributed\_table() se usa para definir una tabla distribuida y crear sus particiones si se trata de una tabla distribuida por hash. Esta función adopta un nombre de tabla, la columna de distribución y un método de distribución opcional e inserta los metadatos adecuados para marcar la tabla como distribuida. De forma predeterminada, la función realiza una distribución por"hash" si no se especifica ningún método de distribución. Si la tabla está distribuida por hash, la función también crea particiones de trabajo basadas en los valores de configuración de factor de replicación de particiones y recuento de particiones. Si la tabla contiene filas, se distribuyen automáticamente a los nodos de trabajo.

Esta función reemplaza el uso de master\_create\_distributed\_table() seguido de master\_create\_worker\_shards().

#### <a name="arguments"></a>Argumentos

**table\_name**: nombre de la tabla que se debe distribuir.

**distribution\_column**: columna en la que se va a distribuir la tabla.

**distribution\_type**: (opcional) el método según el cual se distribuirá la tabla. Los valores permitidos son append o hash, con un valor predeterminado de "hash".

**colocate\_with**: (opcional) incluir la tabla actual en el grupo de coubicación de otra tabla. De forma predeterminada, las tablas se coubican cuando se distribuyen según columnas del mismo tipo, tienen el mismo recuento de particiones y tienen el mismo factor de replicación. Los valores posibles para `colocate_with` son `default` y `none` para iniciar un nuevo grupo de coubicación, o bien el nombre de otra tabla que se coubicará con esa tabla.  (Vea [Coubicación en tabla](concepts-hyperscale-colocation.md)).

Tenga en cuenta que el valor predeterminado de `colocate_with` realiza la coubicación implícita. La [coubicación](concepts-hyperscale-colocation.md) puede resultar ideal cuando las tablas están relacionadas o si se van a combinar.  Sin embargo, cuando dos tablas no están relacionadas, pero se usa el mismo tipo de datos para sus columnas de distribución, coubicarlas accidentalmente puede reducir el rendimiento durante el [reequilibrio de particiones](howto-hyperscale-scaling.md#rebalance-shards).  Las particiones de tabla se moverán juntas innecesariamente \"en cascada\".

Si una nueva tabla distribuida no está relacionada con otras tablas, es mejor especificar `colocate_with => 'none'`.

#### <a name="return-value"></a>Valor devuelto

N/D

#### <a name="example"></a>Ejemplo

En este ejemplo, se informa a la base de datos de que la tabla github\_events se debe distribuir por hash en la columna repo\_id.

```postgresql
SELECT create_distributed_table('github_events', 'repo_id');

-- alternatively, to be more explicit:
SELECT create_distributed_table('github_events', 'repo_id',
                                colocate_with => 'github_repo');
```

### <a name="create_reference_table"></a>create\_reference\_table

La función create\_reference\_table() se utiliza para definir una pequeña tabla de referencia o de dimensión. Esta función adopta un nombre de tabla y crea una tabla distribuida con una sola partición, replicada en cada nodo de trabajo.

#### <a name="arguments"></a>Argumentos

**table\_name**: nombre de la tabla pequeña de dimensión o de referencia que se debe distribuir.

#### <a name="return-value"></a>Valor devuelto

N/D

#### <a name="example"></a>Ejemplo

En este ejemplo se informa a la base de datos de que la tabla nation debe definirse como una tabla de referencia.

```postgresql
SELECT create_reference_table('nation');
```

### <a name="upgrade_to_reference_table"></a>upgrade\_to\_reference\_table

La función upgrade\_to\_reference\_table() adopta una tabla distribuida existente que tiene un recuento de una partición y la actualiza para convertirla en una tabla de referencia reconocida. Después de llamar a esta función, parecerá como si la tabla se hubiera creado con [create_reference_table](#create_reference_table).

#### <a name="arguments"></a>Argumentos

**table\_name**: nombre de la tabla distribuida (con el recuento de particiones = 1) que se distribuirá como una tabla de referencia.

#### <a name="return-value"></a>Valor devuelto

N/D

#### <a name="example"></a>Ejemplo

En este ejemplo se informa a la base de datos de que la tabla nation debe definirse como una tabla de referencia.

```postgresql
SELECT upgrade_to_reference_table('nation');
```

### <a name="mark_tables_colocated"></a>mark\_tables\_colocated

La función mark\_tables\_colocated() adopta una tabla distribuida (origen) y una lista de otras tablas (destinos), y coloca los destinos en el mismo grupo de coubicación que el origen. Si el origen aún no está en un grupo, esta función crea uno y le asigna el origen y los destinos.

La coubicación de las tablas debe realizarse en el momento de la distribución de la tabla mediante el parámetro `colocate_with` de [create_distributed_table](#create_distributed_table), pero `mark_tables_colocated` puede ocuparse de ello más adelante si es necesario.

#### <a name="arguments"></a>Argumentos

**source\_table\_name**: nombre de la tabla distribuida a cuyo grupo de coubicación se asignarán los destinos para que coincidan.

**target\_table\_names**: la matriz de nombres de las tablas de destino distribuidas no debe estar vacía. Estas tablas distribuidas deben coincidir con la tabla de origen en:

> -   distribution (método)
> -   tipo de columna de distribución
> -   tipo de replicación
> -   recuento de particiones

Si ninguna de las opciones anteriores es aplicable, Hiperescala (Citus) generará un error. Por ejemplo, al intentar coubicar las tablas `apples` y `oranges` cuyos tipos de columna de distribución difieren, se obtiene el siguiente resultado:

```
ERROR:  XX000: cannot colocate tables apples and oranges
DETAIL:  Distribution column types don't match for apples and oranges.
```

#### <a name="return-value"></a>Valor devuelto

N/D

#### <a name="example"></a>Ejemplo

Este ejemplo coloca `products` y `line_items` en el mismo grupo de coubicación que `stores`. En el ejemplo, se supone que todas estas tablas están distribuidas en una columna con un tipo coincidente, probablemente \"store id\".

```postgresql
SELECT mark_tables_colocated('stores', ARRAY['products', 'line_items']);
```

### <a name="create_distributed_function"></a>create\_distributed\_function

Propaga una función del nodo de coordinación a los nodos de trabajo, y la marca para la ejecución distribuida. Cuando se llama a una función distribuida en el nodo de coordinación, Hiperescala (Citus) usa el valor del \"argumento de distribución\" para seleccionar un nodo de trabajo para ejecutar la función. La ejecución de la función en los nodos de trabajo aumenta el paralelismo y puede aproximar más el código a los datos en las particiones para reducir la latencia.

La ruta de búsqueda de Postgres no se propaga del nodo de coordinación a los nodos de trabajo durante la ejecución de la función distribuida, por lo que el código de la función distribuida debe utilizar los nombres completos de los objetos de base de datos. Además, los avisos emitidos por las funciones no se mostrarán al usuario.

#### <a name="arguments"></a>Argumentos

**function\_name**: nombre de la función que se va a distribuir. El nombre debe incluir los tipos de parámetros de la función entre paréntesis, ya que varias funciones pueden tener el mismo nombre en PostgreSQL. Por ejemplo, `'foo(int)'` es diferente de `'foo(int, text)'`.

**distribution\_arg\_name**: (opcional) nombre del argumento según el cual realizar la distribución. Para mayor comodidad, o si los argumentos de la función no tienen nombres, se permite un marcador de posición posicional, como `'$1'`. Si no se especifica este parámetro, la función con el nombre `function_name` simplemente se crea en los nodos de trabajo. Si se agregan nodos de trabajo en el futuro, la función también se creará automáticamente.

**colocate\_with**: (opcional) cuando la función distribuida lea o escriba en una tabla distribuida, o, a nivel más general, en el grupo de coubicación, asegúrese de asignar un nombre a la tabla mediante el parámetro `colocate_with`. Después, cada invocación de la función se ejecutará en el nodo de trabajo que contiene las particiones pertinentes.

#### <a name="return-value"></a>Valor devuelto

N/D

#### <a name="example"></a>Ejemplo

```postgresql
-- an example function which updates a hypothetical
-- event_responses table which itself is distributed by event_id
CREATE OR REPLACE FUNCTION
  register_for_event(p_event_id int, p_user_id int)
RETURNS void LANGUAGE plpgsql AS $fn$
BEGIN
  INSERT INTO event_responses VALUES ($1, $2, 'yes')
  ON CONFLICT (event_id, user_id)
  DO UPDATE SET response = EXCLUDED.response;
END;
$fn$;

-- distribute the function to workers, using the p_event_id argument
-- to determine which shard each invocation affects, and explicitly
-- colocating with event_responses which the function updates
SELECT create_distributed_function(
  'register_for_event(int, int)', 'p_event_id',
  colocate_with := 'event_responses'
);
```

## <a name="metadata--configuration-information"></a>Información de configuración y metadatos

### <a name="master_get_table_metadata"></a>master\_get\_table\_metadata

La función master\_get\_table\_metadata() se puede utilizar para devolver los metadatos relacionados con la distribución para una tabla distribuida. Estos metadatos incluyen el identificador de relación, el tipo de almacenamiento, el método de distribución, la columna de distribución, el recuento de replicaciones, el tamaño máximo de partición y la directiva de colocación de particiones de la tabla. En segundo plano, esta función consulta las tablas de metadatos de Hiperescala (Citus) para obtener la información necesaria y la concatena en una tupla antes de devolverla al usuario.

#### <a name="arguments"></a>Argumentos

**table\_name**: nombre de la tabla distribuida para la que desea capturar los metadatos.

#### <a name="return-value"></a>Valor devuelto

Una tupla que contiene la siguiente información:

**logical\_relid**: OID de la tabla distribuida. Hace referencia a la columna relfilenode de la tabla de catálogo del sistema pg\_class.

**part\_storage\_type**: tipo de almacenamiento utilizado para la tabla. Puede ser "t" (tabla estándar), "f" (tabla externa) o "c" (tabla en columnas).

**part\_method**: método de distribución utilizado para la tabla. Puede ser "a" (anexar) o "h" (hash).

**part\_key**: columna de distribución de la tabla.

**part\_replica\_count**: recuento de replicaciones de la partición actual.

**part\_max\_size**: tamaño máximo actual de las particiones en bytes.

**part\_placement\_policy**: directiva de colocación de particiones utilizada para colocar las particiones de la tabla. Puede ser 1 (local-node-first) o 2 (round-robin).

#### <a name="example"></a>Ejemplo

En el ejemplo siguiente, se capturan y muestran los metadatos de la tabla github\_events.

```postgresql
SELECT * from master_get_table_metadata('github_events');
 logical_relid | part_storage_type | part_method | part_key | part_replica_count | part_max_size | part_placement_policy 
---------------+-------------------+-------------+----------+--------------------+---------------+-----------------------
         24180 | t                 | h           | repo_id  |                  2 |    1073741824 |                     2
(1 row)
```

### <a name="get_shard_id_for_distribution_column"></a>get\_shard\_id\_for\_distribution\_column

Hiperescala (Citus) asigna cada fila de una tabla distribuida a una partición según el valor de la columna de distribución de la fila y el método de distribución de la tabla. En la mayoría de los casos, la asignación precisa es un detalle de bajo nivel que el administrador de la base de datos puede omitir. Sin embargo, puede ser útil determinar la partición de una fila, ya sea para las tareas manuales de mantenimiento de bases de datos o simplemente por curiosidad. La función `get_shard_id_for_distribution_column` proporciona esta información para las tablas distribuidas por hash e intervalo, así como para las tablas de referencia. No funciona para la distribución mediante append.

#### <a name="arguments"></a>Argumentos

**table\_name**: la tabla distribuida.

**distribution\_value**: el valor de la columna de distribución.

#### <a name="return-value"></a>Valor devuelto

El identificador de la partición de Hiperescala (Citus) se asocia con el valor de la columna de distribución para la tabla especificada.

#### <a name="example"></a>Ejemplo

```postgresql
SELECT get_shard_id_for_distribution_column('my_table', 4);

 get_shard_id_for_distribution_column
--------------------------------------
                               540007
(1 row)
```

### <a name="column_to_column_name"></a>column\_to\_column\_name

Convierte la columna `partkey` de `pg_dist_partition` en un nombre de columna de texto. La conversión es útil para determinar la columna de distribución de una tabla distribuida.

Para obtener una explicación más detallada, vea [Elección de una columna de distribución](concepts-hyperscale-choose-distribution-column.md).

#### <a name="arguments"></a>Argumentos

**table\_name**: la tabla distribuida.

**column\_var\_text**: el valor de `partkey` en la tabla `pg_dist_partition`.

#### <a name="return-value"></a>Valor devuelto

Nombre de la columna de distribución de `table_name`.

#### <a name="example"></a>Ejemplo

```postgresql
-- get distribution column name for products table

SELECT column_to_column_name(logicalrelid, partkey) AS dist_col_name
  FROM pg_dist_partition
 WHERE logicalrelid='products'::regclass;
```

Salida:

```
┌───────────────┐
│ dist_col_name │
├───────────────┤
│ company_id    │
└───────────────┘
```

### <a name="citus_relation_size"></a>citus\_relation\_size

Obtiene el espacio en disco que usan todas las particiones de la tabla distribuida especificada.
El espacio en disco incluye el tamaño de la \"bifurcación principal\", pero excluye el mapa de visibilidad y el mapa de espacio disponible para las particiones.

#### <a name="arguments"></a>Argumentos

**logicalrelid**: el nombre de una tabla distribuida.

#### <a name="return-value"></a>Valor devuelto

Tamaño en bytes como un bigint.

#### <a name="example"></a>Ejemplo

```postgresql
SELECT pg_size_pretty(citus_relation_size('github_events'));
```

```
pg_size_pretty
--------------
23 MB
```

### <a name="citus_table_size"></a>citus\_table\_size

Obtiene el espacio en disco que usan todas las particiones de la tabla distribuida especificada, excluidos los índices (incluidos el AVISO, el mapa de espacio libre y el mapa de visibilidad).

#### <a name="arguments"></a>Argumentos

**logicalrelid**: el nombre de una tabla distribuida.

#### <a name="return-value"></a>Valor devuelto

Tamaño en bytes como un bigint.

#### <a name="example"></a>Ejemplo

```postgresql
SELECT pg_size_pretty(citus_table_size('github_events'));
```

```
pg_size_pretty
--------------
37 MB
```

### <a name="citus_total_relation_size"></a>citus\_total\_relation\_size

Obtiene el espacio total en disco utilizado por todas las particiones de la tabla distribuida especificada, incluidos todos los índices y los datos de los AVISOS.

#### <a name="arguments"></a>Argumentos

**logicalrelid**: el nombre de una tabla distribuida.

#### <a name="return-value"></a>Valor devuelto

Tamaño en bytes como un bigint.

#### <a name="example"></a>Ejemplo

```postgresql
SELECT pg_size_pretty(citus_total_relation_size('github_events'));
```

```
pg_size_pretty
--------------
73 MB
```

### <a name="citus_stat_statements_reset"></a>citus\_stat\_statements\_reset

Quita todas las filas de [citus_stat_statements](reference-hyperscale-metadata.md#query-statistics-table).
Esta función es independiente de `pg_stat_statements_reset()`. Para restablecer todas las estadísticas, llame a ambas funciones.

#### <a name="arguments"></a>Argumentos

N/D

#### <a name="return-value"></a>Valor devuelto

None

## <a name="server-group-management-and-repair"></a>Administración y reparación de grupos de servidores

### <a name="master_copy_shard_placement"></a>master\_copy\_shard\_placement

Si no se puede actualizar una ubicación de partición durante un comando de modificación o una operación DDL, se marca como inactiva. A continuación, se puede llamar a la función master\_copy\_shard\_placement para reparar una ubicación de partición inactiva con datos en una ubicación correcta.

Para reparar una partición, la función quita primero la ubicación de la partición incorrecta y la vuelve a crear con el esquema en el nodo de coordinación. Una vez creada la ubicación de la partición, la función copia los datos de la ubicación correcta y actualiza los metadatos para marcar la nueva ubicación de la partición como correcta. Esta función garantiza la protección de la partición de cualquier modificación simultánea durante la reparación.

#### <a name="arguments"></a>Argumentos

**shard\_id**: identificador de la partición que se va a reparar.

**source\_node\_name**: nombre DNS del nodo en el que está presente la ubicación de la partición correcta (nodo de \"origen\").

**source\_node\_port**: puerto del nodo de trabajo de origen en el que escucha el servidor de bases de datos.

**target\_node\_name**: nombre DNS del nodo en el que está presente la ubicación de la partición no válida (nodo de \"destino\").

**target\_node\_port**: puerto del nodo de trabajo de destino en el que escucha el servidor de bases de datos.

#### <a name="return-value"></a>Valor devuelto

N/D

#### <a name="example"></a>Ejemplo

En el ejemplo siguiente se reparará una ubicación de partición inactiva de la partición 12345, que se encuentra en el servidor de bases de datos que se ejecuta en "bad\_host" en el puerto 5432. Para repararla, utilizará los datos de una ubicación de partición correcta presente en el servidor que se ejecuta en "good\_host" en el puerto.
5432.

```postgresql
SELECT master_copy_shard_placement(12345, 'good_host', 5432, 'bad_host', 5432);
```

### <a name="master_move_shard_placement"></a>master\_move\_shard\_placement

Esta función mueve una partición determinada y las particiones colocadas con ella de un nodo a otro. Normalmente, se usa indirectamente durante el reequilibrio de particiones en lugar de que la llame directamente un administrador de bases de datos.

Hay dos maneras de trasladar los datos: bloqueo o desbloqueo. El enfoque de bloqueo implica que, durante el traslado, se pausan todas las modificaciones realizadas en la partición.
La segunda opción, que evita que se bloqueen las escrituras en las particiones, se basa en la replicación lógica de Postgres 10.

Después de una operación de transición correcta, las particiones del nodo de origen se eliminan. Si se produce un error en la transición en cualquier momento, esta función produce un error y deja sin modificar los nodos de origen y destino.

#### <a name="arguments"></a>Argumentos

**shard\_id**: identificador de la partición que se va a mover.

**source\_node\_name**: nombre DNS del nodo en el que está presente la ubicación de la partición correcta (nodo de \"origen\").

**source\_node\_port**: puerto del nodo de trabajo de origen en el que escucha el servidor de bases de datos.

**target\_node\_name**: nombre DNS del nodo en el que está presente la ubicación de la partición no válida (nodo de \"destino\").

**target\_node\_port**: puerto del nodo de trabajo de destino en el que escucha el servidor de bases de datos.

**shard\_transfer\_mode**: (opcional) especifica el método de replicación, si va a usar la replicación lógica de PostgreSQL o un comando COPY entre nodos de trabajo. Los valores posibles son:

> -   `auto`: requerir la identidad de réplica si es posible la replicación lógica; de lo contrario, use el comportamiento heredado (por ejemplo, para la reparación de particiones, PostgreSQL 9.6). Este es el valor predeterminado.
> -   `force_logical`: utilizar la replicación lógica incluso si la tabla no tiene una identidad de réplica. Se producirá un error en las instrucciones update/delete simultáneas de la tabla durante la replicación.
> -   `block_writes`: usar COPY (escrituras de bloqueo) para las tablas que carecen de identidad de réplica o clave principal.

#### <a name="return-value"></a>Valor devuelto

N/D

#### <a name="example"></a>Ejemplo

```postgresql
SELECT master_move_shard_placement(12345, 'from_host', 5432, 'to_host', 5432);
```

### <a name="rebalance_table_shards"></a>rebalance\_table\_shards

La función rebalance\_table\_shards() mueve las particiones de la tabla especificada para que se distribuyan uniformemente entre los nodos de trabajo. La función calcula primero la lista de movimientos que debe realizar para asegurarse de que el grupo de servidores está equilibrado en el umbral especificado. A continuación, mueve las ubicaciones de las particiones de una en una desde el nodo de origen al nodo de destino y actualiza los metadatos de las particiones correspondientes para reflejar el movimiento.

A cada partición se le asigna un costo al determinar si las particiones se \"distribuyen de manera uniforme\". De forma predeterminada, cada partición tiene el mismo costo (un valor de 1), por lo que la distribución para igualar el costo entre los nodos de trabajo es lo mismo que igualar el número de particiones en cada una. La estrategia de costo constante se denomina \"by\_shard\_count\", y es la estrategia de reequilibrio predeterminada.

La estrategia predeterminada es adecuada en estas circunstancias:

*  Las particiones tienen aproximadamente el mismo tamaño.
*  Las particiones obtienen aproximadamente la misma cantidad de tráfico.
*  Todos los nodos de trabajo tienen el mismo tamaño o tipo.
*  Las particiones no se han anclado a determinados roles de trabajo.

Si no se mantiene alguna de estas suposiciones, el reequilibrio predeterminado puede resultar en un plan incorrecto. En este caso, puede personalizar la estrategia mediante el parámetro `rebalance_strategy`.

Es aconsejable llamar a [get_rebalance_table_shards_plan](#get_rebalance_table_shards_plan) antes de ejecutar rebalance\_table\_shards, para ver y comprobar las acciones que se van a realizar.

#### <a name="arguments"></a>Argumentos

**table\_name**: (opcional) nombre de la tabla cuyas particiones se deben reequilibrar. Si es NULL, vuelva a equilibrar todos los grupos de coubicación existentes.

**threshold**: (opcional) un número de punto flotante entre 0,0 y 1,0 que indica la relación de diferencia máxima del uso del nodo con respecto al promedio de uso. Por ejemplo, si se especifica 0,1, el reequilibrador de particiones intentará equilibrar todos los nodos para que contengan el mismo número de particiones ± 10 %.
En concreto, el reequilibrador de particiones intentará converger la utilización de todos los nodos de trabajo en el intervalo (1 - umbral) \* average\_utilization \... (1
+ umbral) \* average\_utilization.

**max\_shard\_moves**: (opcional) número máximo de particiones que se van a mover.

**excluded\_shard\_list**: (opcional) identificadores de las particiones que no deben moverse durante la operación de reequilibrio.

**shard\_transfer\_mode**: (opcional) especifica el método de replicación, si va a usar la replicación lógica de PostgreSQL o un comando COPY entre nodos de trabajo. Los valores posibles son:

> -   `auto`: requerir la identidad de réplica si es posible la replicación lógica; de lo contrario, use el comportamiento heredado (por ejemplo, para la reparación de particiones, PostgreSQL 9.6). Este es el valor predeterminado.
> -   `force_logical`: utilizar la replicación lógica incluso si la tabla no tiene una identidad de réplica. Se producirá un error en las instrucciones update/delete simultáneas de la tabla durante la replicación.
> -   `block_writes`: usar COPY (escrituras de bloqueo) para las tablas que carecen de identidad de réplica o clave principal.

**drain\_only**: (opcional) cuando es true, mueve las particiones fuera de los nodos de trabajo que tienen `shouldhaveshards` establecido en false en [pg_dist_node](reference-hyperscale-metadata.md#worker-node-table); no se mueve ninguna otra partición.

**rebalance\_strategy**: (opcional) nombre de una estrategia en [pg_dist_rebalance_strategy](reference-hyperscale-metadata.md#rebalancer-strategy-table).
Si se omite este argumento, la función elige la estrategia predeterminada, como se indica en la tabla.

#### <a name="return-value"></a>Valor devuelto

N/D

#### <a name="example"></a>Ejemplo

En el ejemplo siguiente, se intentarán reequilibrar las particiones de la tabla github\_events dentro del umbral predeterminado.

```postgresql
SELECT rebalance_table_shards('github_events');
```

En este ejemplo de uso, se intentará reequilibrar la tabla github\_events sin mover las particiones con los identificadores 1 y 2.

```postgresql
SELECT rebalance_table_shards('github_events', excluded_shard_list:='{1,2}');
```

### <a name="get_rebalance_table_shards_plan"></a>get\_rebalance\_table\_shards\_plan

Se genera la salida de los movimientos de particiones planeados de [rebalance_table_shards](#rebalance_table_shards) sin ejecutarlos.
Aunque es poco probable, get\_rebalance\_table\_shards\_plan puede generar un plan ligeramente distinto al que se obtendrá al realizar una llamada a rebalance\_table\_shards con los mismos argumentos. No se ejecutan al mismo tiempo, por lo que los datos sobre el grupo de servidores, \-por ejemplo, el espacio en disco\-, pueden diferir entre las llamadas.

#### <a name="arguments"></a>Argumentos

Los mismos argumentos que rebalance\_table\_shards: relation, threshold, max\_shard\_moves, excluded\_shard\_list y drain\_only. Consulte la documentación de esa función para ver el significado de los argumentos.

#### <a name="return-value"></a>Valor devuelto

Tuplas que contienen estas columnas:

-   **table\_name**: tabla cuyas particiones se moverán
-   **shardid**: partición en cuestión.
-   **shard\_size**: tamaño en bytes.
-   **sourcename**: nombre de host del nodo de origen
-   **sourceport**: puerto del nodo de origen
-   **targetname**: nombre de host del nodo de destino
-   **targetport**: puerto del nodo de destino

### <a name="get_rebalance_progress"></a>get\_rebalance\_progress

Una vez que comienza el reequilibrio de una partición, la función `get_rebalance_progress()` muestra el progreso de cada una de las particiones involucradas. Supervisa los movimientos planeados y ejecutados por `rebalance_table_shards()`.

#### <a name="arguments"></a>Argumentos

N/D

#### <a name="return-value"></a>Valor devuelto

Tuplas que contienen estas columnas:

-   **sessionid**: PID de Postgres del monitor de reequilibrio
-   **table\_name**: tabla cuyas particiones se van a mover
-   **shardid**: partición en cuestión.
-   **shard\_size**: tamaño en bytes.
-   **sourcename**: nombre de host del nodo de origen
-   **sourceport**: puerto del nodo de origen
-   **targetname**: nombre de host del nodo de destino
-   **targetport**: puerto del nodo de destino
-   **progress**: 0 = en espera de moverse; 1 = en movimiento; 2 = completado

#### <a name="example"></a>Ejemplo

```sql
SELECT * FROM get_rebalance_progress();
```

```
┌───────────┬────────────┬─────────┬────────────┬───────────────┬────────────┬───────────────┬────────────┬──────────┐
│ sessionid │ table_name │ shardid │ shard_size │  sourcename   │ sourceport │  targetname   │ targetport │ progress │
├───────────┼────────────┼─────────┼────────────┼───────────────┼────────────┼───────────────┼────────────┼──────────┤
│      7083 │ foo        │  102008 │    1204224 │ n1.foobar.com │       5432 │ n4.foobar.com │       5432 │        0 │
│      7083 │ foo        │  102009 │    1802240 │ n1.foobar.com │       5432 │ n4.foobar.com │       5432 │        0 │
│      7083 │ foo        │  102018 │     614400 │ n2.foobar.com │       5432 │ n4.foobar.com │       5432 │        1 │
│      7083 │ foo        │  102019 │       8192 │ n3.foobar.com │       5432 │ n4.foobar.com │       5432 │        2 │
└───────────┴────────────┴─────────┴────────────┴───────────────┴────────────┴───────────────┴────────────┴──────────┘
```

### <a name="citus_add_rebalance_strategy"></a>citus\_add\_rebalance\_strategy

Anexar una fila a [pg_dist_rebalance_strategy](reference-hyperscale-metadata.md?#rebalancer-strategy-table).

#### <a name="arguments"></a>Argumentos

Para obtener más información sobre estos argumentos, vea los valores de columna correspondientes en `pg_dist_rebalance_strategy`.

**name**: identificador de la estrategia nueva.

**shard\_cost\_function**: identifica la función utilizada para determinar el \"costo\" de cada partición.

**node\_capacity\_function**: identifica la función para medir la capacidad del nodo.

**shard\_allowed\_on\_node\_function**: identifica la función que determina qué particiones se pueden colocar en determinados nodos.

**default\_threshold**: un umbral de número de punto flotante que optimiza la precisión con la que se debe equilibrar el costo acumulado de las particiones entre los nodos.

**minimum\_threshold**: (opcional) columna de protección que contiene el valor mínimo permitido para el argumento de umbral de rebalance\_table\_shards(). El valor predeterminado es 0.

#### <a name="return-value"></a>Valor devuelto

N/D

### <a name="citus_set_default_rebalance_strategy"></a>citus\_set\_default\_rebalance\_strategy

Actualizar la tabla [pg_dist_rebalance_strategy](reference-hyperscale-metadata.md#rebalancer-strategy-table), cambiando la estrategia denominada por su argumento para que sea el valor predeterminado elegido al reequilibrar las particiones.

#### <a name="arguments"></a>Argumentos

**name**: nombre de la estrategia en pg\_dist\_rebalance\_strategy

#### <a name="return-value"></a>Valor devuelto

N/D

#### <a name="example"></a>Ejemplo

```postgresql
SELECT citus_set_default_rebalance_strategy('by_disk_size');
```

### <a name="citus_remote_connection_stats"></a>citus\_remote\_connection\_stats

La función citus\_remote\_connection\_stats() muestra el número de conexiones activas a cada nodo remoto.

#### <a name="arguments"></a>Argumentos

N/D

#### <a name="example"></a>Ejemplo

```postgresql
SELECT * from citus_remote_connection_stats();
```

```
    hostname    | port | database_name | connection_count_to_node
----------------+------+---------------+--------------------------
 citus_worker_1 | 5432 | postgres      |                        3
(1 row)
```

### <a name="master_drain_node"></a>master\_drain\_node

La función master\_drain\_node() mueve las particiones fuera del nodo designado y a otros nodos que tienen `shouldhaveshards` establecido en true en [pg_dist_node](reference-hyperscale-metadata.md#worker-node-table). Llame a la función antes de quitar un nodo del grupo de servidores y desactivar el servidor físico del nodo.

#### <a name="arguments"></a>Argumentos

**nodename**: nombre de host del nodo que se va a purgar.

**nodeport**: número de puerto del nodo que se va a purgar.

**shard\_transfer\_mode**: (opcional) especifica el método de replicación, si va a usar la replicación lógica de PostgreSQL o un comando COPY entre nodos de trabajo. Los valores posibles son:

> -   `auto`: requerir la identidad de réplica si es posible la replicación lógica; de lo contrario, use el comportamiento heredado (por ejemplo, para la reparación de particiones, PostgreSQL 9.6). Este es el valor predeterminado.
> -   `force_logical`: utilizar la replicación lógica incluso si la tabla no tiene una identidad de réplica. Se producirá un error en las instrucciones update/delete simultáneas de la tabla durante la replicación.
> -   `block_writes`: usar COPY (escrituras de bloqueo) para las tablas que carecen de identidad de réplica o clave principal.

**rebalance\_strategy**: (opcional) nombre de una estrategia en [pg_dist_rebalance_strategy](reference-hyperscale-metadata.md#rebalancer-strategy-table).
Si se omite este argumento, la función elige la estrategia predeterminada, como se indica en la tabla.

#### <a name="return-value"></a>Valor devuelto

N/D

#### <a name="example"></a>Ejemplo

Estos son los pasos típicos para quitar un solo nodo (por ejemplo, "10.0.0.1" en un puerto estándar de PostgreSQL):

1.  Purgue el nodo.

    ```postgresql
    SELECT * from master_drain_node('10.0.0.1', 5432);
    ```

2.  Espere a que finalice el comando.

3.  Quite el nodo.

Al purgar varios nodos, se recomienda usar [rebalance_table_shards](#rebalance_table_shards) en su lugar. Esto permite a Hiperescala (Citus) planear de antemano y mover particiones el número mínimo de veces.

1.  Ejecute este comando para cada nodo que desee quitar:

    ```postgresql
    SELECT * FROM master_set_node_property(node_hostname, node_port, 'shouldhaveshards', false);
    ```

2.  Purgue todos los nodos a la vez con [rebalance_table_shards](#rebalance_table_shards).

    ```postgresql
    SELECT * FROM rebalance_table_shards(drain_only := true);
    ```

3.  Espere hasta que finalice el reequilibrio de la purga.

4.  Quite los nodos.

### <a name="replicate_table_shards"></a>replicate\_table\_shards

La función replicate\_table\_shards() replica las particiones subreplicadas en la tabla especificada. En primer lugar, la función calcula la lista de las particiones subreplicadas y las ubicaciones desde las que se pueden capturar para la replicación. A continuación, la función copia dichas particiones y actualiza los metadatos de la partición correspondientes para reflejar la copia.

#### <a name="arguments"></a>Argumentos

**table\_name**: nombre de la tabla cuyas particiones se deben replicar.

**shard\_replication\_factor**: (opcional) factor de replicación deseado que se alcanzará para cada partición.

**max\_shard\_copies**: (opcional) número máximo de particiones que se van a copiar para alcanzar el factor de replicación deseado.

**excluded\_shard\_list**: (opcional) identificadores de las particiones que no deben copiarse durante la operación de replicación.

#### <a name="return-value"></a>Valor devuelto

N/D

#### <a name="examples"></a>Ejemplos

En el ejemplo siguiente, se intentarán replicar las particiones de la tabla github\_events en shard\_replication\_factor.

```postgresql
SELECT replicate_table_shards('github_events');
```

En este ejemplo, se intentará alcanzar el factor de replicación deseado de las particiones de la tabla github\_events con un máximo de 10 copias de particiones. El reequilibrador copiará un máximo de 10 particiones para intentar alcanzar el factor de replicación deseado.

```postgresql
SELECT replicate_table_shards('github_events', max_shard_copies:=10);
```

### <a name="isolate_tenant_to_new_shard"></a>isolate\_tenant\_to\_new\_shard

Esta función crea una partición que contiene filas con un valor único específico en la columna de distribución. Es especialmente útil para el caso de uso de Hiperescala (Citus) multiinquilino, donde un inquilino grande puede colocarse por sí solo en su propia partición y, en última instancia, en su propio nodo físico.

#### <a name="arguments"></a>Argumentos

**table\_name**: nombre de la tabla para obtener una nueva partición.

**tenant\_id**: valor de la columna de distribución que se asignará a la nueva partición.

**cascade\_option**: (opcional) cuando se establece en \"CASCADE\", también aísla una partición de todas las tablas en el [grupo de coubicación](concepts-hyperscale-colocation.md) de la tabla actual.

#### <a name="return-value"></a>Valor devuelto

**shard\_id**: la función devuelve el identificador único asignado a la partición recién creada.

#### <a name="examples"></a>Ejemplos

Cree una partición para que contenga los elementos lineitems para el inquilino 135:

```postgresql
SELECT isolate_tenant_to_new_shard('lineitem', 135);
```

```
┌─────────────────────────────┐
│ isolate_tenant_to_new_shard │
├─────────────────────────────┤
│                      102240 │
└─────────────────────────────┘
```

## <a name="next-steps"></a>Pasos siguientes

* Muchas de las funciones de este artículo modifican las [tablas de metadatos](reference-hyperscale-metadata.md) del sistema.
* Los [parámetros del servidor](reference-hyperscale-parameters.md) personalizan el comportamiento de algunas funciones.
