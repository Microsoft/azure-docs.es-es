---
title: 'Parámetros del servidor en Azure Database for PostgreSQL: Hiperescala (Hiperescala [Citus])'
description: Parámetros de la API de SQL de Hiperescala (Citus)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: reference
ms.date: 08/10/2020
ms.openlocfilehash: 57258540f3cd7b4c47b662b0885453cedd188e5b
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88136650"
---
# <a name="server-parameters"></a>Parámetros del servidor

Hay varios parámetros de servidor que afectan al comportamiento de Hiperescala (Citus), tanto parámetros de PostgreSQL estándar como parámetros específicos de Hiperescala (Citus). Para más información sobre los parámetros de configuración de PostgreSQL, puede visitar la sección de [configuración de tiempo de ejecución](http://www.postgresql.org/docs/current/static/runtime-config.html) de la documentación de PostgreSQL.

El resto de esta referencia pretende describir los parámetros de configuración específicos de Hiperescala (Citus). Estos parámetros se pueden establecer en Azure Portal, en **Parámetros del nodo de trabajo**, **Configuración** para un grupo de servidores de Hiperescala (Citus).

> [!NOTE]
>
> Es posible que los grupos de servidores de Hiperescala que ejecutan versiones anteriores del motor de Citus no ofrezcan todos los parámetros que se enumeran a continuación.

## <a name="general-configuration"></a>Configuración general

### <a name="citususe_secondary_nodes-enum"></a>citus.use\_secondary\_nodes (enumeración)

Establece la directiva que se va a usar al elegir nodos para consultas SELECT. Si se establece en "always", el planificador solo consultará los nodos marcados como noderole "secundario" en [pg_dist_node](reference-hyperscale-metadata.md#worker-node-table).

Los valores admitidos para esta enumeración son:

-   **never:** (valor predeterminado), todas las lecturas se producen en los nodos principales.
-   **always:** En su lugar, las lecturas se ejecutan en nodos secundarios y se deshabilitan las instrucciones INSERT/UPDATE.

### <a name="cituscluster_name-text"></a>citus.cluster\_name (texto)

Informa al planificador del nodo de coordinación qué clúster coordina. Una vez establecido cluster\_name, el planificador únicamente consultará los nodos de trabajo de ese clúster.

### <a name="citusenable_version_checks-boolean"></a>citus.enable\_version\_checks (booleano)

La actualización de la versión de Hiperescala (Citus) requiere un reinicio del servidor (para seleccionar la nueva biblioteca compartida), seguido del comando ALTER EXTENSION UPDATE.  La imposibilidad de ejecutar ambos pasos podría provocar errores o bloqueos.
Hiperescala (Citus) valida entonces que la versión del código y la de la extensión coinciden y, si no es así, se resuelven los errores.

El valor predeterminado es "true" y tiene efecto en el coordinador. En raras ocasiones, es posible que los procesos de actualización complejos requieran que este parámetro se establezca en "false" y, por lo tanto, se deshabilite la comprobación.

### <a name="cituslog_distributed_deadlock_detection-boolean"></a>citus.log\_distributed\_deadlock\_detection (booleano)

Indica si se va a registrar el procesamiento relacionado con la detección de interbloqueos distribuidos en el registro del servidor. El valor predeterminado es "false".

### <a name="citusdistributed_deadlock_detection_factor-floating-point"></a>citus.distributed\_deadlock\_detection\_factor (número de punto flotante)

Establece el tiempo de espera antes de comprobar los interbloqueos distribuidos. En concreto, el tiempo de espera será este valor multiplicado por el valor [deadlock\_timeout](https://www.postgresql.org/docs/current/static/runtime-config-locks.html) de PostgreSQL. El valor predeterminado es `2`. Un valor de `-1` deshabilita la detección de interbloqueos distribuidos.

### <a name="citusnode_connection_timeout-integer"></a>citus.node\_connection\_timeout (entero)

El GUC `citus.node_connection_timeout` establece la duración máxima (en milisegundos) que se debe esperar para el establecimiento de la conexión. Hiperescala (Citus) genera un error si el tiempo de espera transcurre antes de que se establezca al menos una conexión de trabajo. Este GUC afecta a las conexiones del coordinador con los roles de trabajo y de los roles de trabajo entre sí.

-   Valor predeterminado: cinco segundos
-   Mínimos: 10 milisegundos
-   Máximo: una hora

```postgresql
-- set to 30 seconds
ALTER DATABASE foo
SET citus.node_connection_timeout = 30000;
```

## <a name="query-statistics"></a>Estadísticas de consultas

### <a name="citusstat_statements_purge_interval-integer"></a>citus.stat\_statements\_purge\_interval (entero)

Establece la frecuencia con la que el demonio de mantenimiento quita los registros de [citus_stat_statements](reference-hyperscale-metadata.md#query-statistics-table) que no coinciden en `pg_stat_statements`. Este valor de configuración establece el intervalo de tiempo entre purgas en segundos, con un valor predeterminado de 10. Un valor de 0 deshabilita las purgas.

```psql
SET citus.stat_statements_purge_interval TO 5;
```

Este parámetro tiene efecto en el coordinador y se puede cambiar en tiempo de ejecución.

## <a name="data-loading"></a>Carga de datos

### <a name="citusmulti_shard_commit_protocol-enum"></a>citus.multi\_shard\_commit\_protocol (enumeración)

Establece el protocolo de confirmación que se va a usar al ejecutar COPY en una tabla con distribución por hash. En cada selección de ubicación de partición individual, la instrucción COPY se ejecuta en un bloque de transacciones para garantizar que no se ingiera ningún dato en caso de error durante la ejecución de dicha instrucción. Sin embargo, hay un caso de error concreto en el que COPY se ejecuta correctamente en todas las selecciones de ubicación, pero se produce un error (de hardware) antes de que se confirmen todas las transacciones. Este parámetro se puede usar para evitar la pérdida de datos en ese caso eligiendo entre los siguientes protocolos de confirmación:

-   **2pc:** (valor predeterminado) las transacciones en las que COPY se ejecuta en las selecciones de ubicación de partición se preparan primero mediante la [confirmación en dos fases](http://www.postgresql.org/docs/current/static/sql-prepare-transaction.html) de PostgreSQL y luego se confirman. Las confirmaciones que no se realicen se podrán recuperar o anular manualmente mediante COMMIT PREPARED o ROLLBACK PREPARED, respectivamente.
    Cuando se usa 2pc, [max\_prepared\_transactions](http://www.postgresql.org/docs/current/static/runtime-config-resource.html) debe aumentarse en todos los roles de trabajo, normalmente hasta el mismo valor de max\_connections.
-   **1pc:** Las transacciones en las que COPY se realiza en las selecciones de ubicación de partición se confirman en una única ronda. Es posible que se pierdan datos si se no se produce una confirmación después de que COPY se ejecute correctamente en todas las selecciones de ubicación (poco frecuentes).

### <a name="citusshard_replication_factor-integer"></a>citus.shard\_replication\_factor (entero)

Establece el factor de replicación para las particiones, es decir, el número de nodos en los que se colocarán las particiones, y su valor predeterminado es 1. Este parámetro se puede establecer en tiempo de ejecución y tiene efecto en el coordinador. El valor ideal para este parámetro depende del tamaño del clúster y de la tasa de errores de nodo.  Por ejemplo, puede que quiera aumentar este factor de replicación si ejecuta clústeres grandes y observa errores de los nodos con mayor frecuencia.

### <a name="citusshard_count-integer"></a>citus.shard\_count (entero)

Establece el número de particiones para las tablas con particiones por hash y su valor predeterminado es 32.  [create_distributed_table](reference-hyperscale-functions.md#create_distributed_table) UDF usa este valor al crear tablas con particiones por hash. Este parámetro se puede establecer en tiempo de ejecución y tiene efecto en el coordinador.

### <a name="citusshard_max_size-integer"></a>citus.shard\_max\_size (entero)

Establece el tamaño máximo al que crecerá una partición antes de dividirse y su valor predeterminado es 1 GB. Cuando el tamaño del archivo de origen de una partición (que se usa para el almacenamiento provisional) supera este valor de configuración, la base de datos garantiza que se crea otra partición. Este parámetro se puede establecer en tiempo de ejecución y tiene efecto en el coordinador.

## <a name="planner-configuration"></a>Configuración de del planificador

### <a name="cituslimit_clause_row_fetch_count-integer"></a>citus.limit\_clause\_row\_fetch\_count (entero)

Establece el número de filas que se van a capturar por tarea para la optimización de las cláusulas de límite.
En algunos casos, puede que sea necesario que las consultas con cláusulas de límite capturan todas las filas de cada tarea para generar resultados. En esos casos, y donde una aproximación produciría resultados significativos, este valor de configuración establece el número de filas de cada partición que se van a capturar. Las aproximaciones de límite están deshabilitadas de forma predeterminada y este parámetro se establece en -1. Este valor se puede establecer en tiempo de ejecución y tiene efecto en el coordinador.

### <a name="cituscount_distinct_error_rate-floating-point"></a>citus.count\_distinct\_error\_rate (número de punto flotante)

Hiperescala (Citus) puede calcular valores aproximados de recuento (único) con la extensión postgresql-hll. Esta entrada de configuración establece la tasa de errores deseada al calcular valores de recuento (único). 0,0, que es el valor predeterminado, deshabilita los valores aproximados de recuento (único); y 1,0 no ofrece ninguna garantía sobre la exactitud de los resultados. Se recomienda establecer este parámetro en 0,005 para obtener los mejores resultados. Este valor se puede establecer en tiempo de ejecución y tiene efecto en el coordinador.

### <a name="citustask_assignment_policy-enum"></a>citus.task\_assignment\_policy (enumeración)

> [!NOTE]
> Este GUC solo es aplicable cuando [shard_replication_factor](reference-hyperscale-parameters.md#citusshard_replication_factor-integer) es mayor que uno o para consultas en [reference_tables](concepts-hyperscale-distributed-data.md#type-2-reference-tables).

Establece la directiva que se va a usar al asignar tareas a roles de trabajo. El coordinador asigna tareas a los roles de trabajo en función de las selecciones de ubicación de partición. Este valor de configuración especifica la directiva que se va a usar al crear estas asignaciones.
Actualmente, hay tres directivas posibles de asignación de tareas que se pueden usar.

-   **greedy:** La directiva greedy es el valor predeterminado y tiene por objeto distribuir tareas uniformemente entre los roles de trabajo.
-   **round-robin:** La directiva round-robin asigna tareas a los roles de trabajo de modo Round Robin alternando entre distintas réplicas. Esta directiva permite un mejor uso del clúster cuando el número de particiones de una tabla es bajo en comparación con el número de roles de trabajo.
-   **first-replica:** La directiva first-replica asigna tareas por orden de inserción de las selecciones de ubicación (réplicas) de las particiones. En otras palabras, la consulta de fragmentos de una partición se asigna al rol de trabajo que tiene la primera réplica de esa partición.
    Este método permite tener garantías seguras sobre qué particiones se utilizarán en cada nodo (es decir, garantías de residencia en memoria más seguras).

Este parámetro se puede establecer en tiempo de ejecución y tiene efecto en el coordinador.

## <a name="intermediate-data-transfer"></a>Transferencia de datos intermedios

### <a name="citusbinary_worker_copy_format-boolean"></a>citus.binary\_worker\_copy\_format (booleano)

Use el formato binario de copia para transferir datos intermedios entre roles de trabajo.
En las combinaciones de tablas grandes, es posible que Hiperescala (Citus) tenga que volver a particionar y ordenar dinámicamente los datos entre distintos roles de trabajo. De forma predeterminada, estos datos se transfieren en formato de texto. Al habilitar este parámetro, se indica a la base de datos que use el formato de serialización binaria de PostgreSQL para transferir estos datos. Este parámetro tiene efecto en los roles de trabajo y debe cambiarse en el archivo postgresql.conf. Después de editar el archivo de configuración, los usuarios pueden enviar una señal SIGHUP o reiniciar el servidor para que el cambio surta efecto.

### <a name="citusbinary_master_copy_format-boolean"></a>citus.binary\_master\_copy\_format (booleano)

Use el formato binario de copia para transferir datos entre el coordinado y los roles de trabajo. Al ejecutar consultas distribuidas, los roles de trabajo transfieren sus resultados intermedios al coordinador para la agregación final. De forma predeterminada, estos datos se transfieren en formato de texto. Al habilitar este parámetro, se indica a la base de datos que use el formato de serialización binaria de PostgreSQL para transferir estos datos.
Este parámetro se puede establecer en tiempo de ejecución y tiene efecto en el coordinador.

### <a name="citusmax_intermediate_result_size-integer"></a>citus.max\_intermediate\_result\_size (entero)

Tamaño máximo en KB de los resultados intermedios para las CTE que no se pueden insertar en los nodos de trabajo para su ejecución y para las subconsultas complejas. El valor predeterminado es 1 GB y un valor de -1 significa que no hay límite.
Las consultas que superen el límite se cancelarán y generarán un mensaje de error.

## <a name="ddl"></a>DDL

### <a name="citusenable_ddl_propagation-boolean"></a>citus.enable\_ddl\_propagation (booleano)

Especifica si los cambios de DDL se propagan automáticamente del coordinador a todos los roles de trabajo. El valor predeterminado es true. Dado que algunos cambios de esquema requieren un bloqueo exclusivo de acceso en las tablas, y dado que la propagación automática se aplica a todos los roles de trabajo secuencialmente, puede hacer que un clúster de Hiperescala (Citus) deje de responder temporalmente. Puede optar por deshabilitar esta configuración y propagar los cambios manualmente.

## <a name="executor-configuration"></a>Configuración del ejecutor

### <a name="general"></a>General

#### <a name="citusall_modifications_commutative"></a>citus.all\_modifications\_commutative

Hiperescala (Citus) aplica las reglas de conmutabilidad y adquiere los bloqueos adecuados para las operaciones de modificación con el fin de garantizar la corrección del comportamiento. Por ejemplo, se supone que una instrucción INSERT se conmuta por otra instrucción INSERT, pero no por una instrucción UPDATE o DELETE. Del mismo modo, se supone que una instrucción UPDATE o DELETE no se conmutará por otra instrucción UPDATE o DELETE. Esta precaución implica que las instrucciones UPDATE y DELETE requieren Hiperescala (Citus) para adquirir bloqueos más fuertes.

Si tiene instrucciones UPDATE que son conmutativas con sus instrucciones INSERT u otras instrucciones UPDATE, puede rebajar estas suposiciones de conmutabilidad estableciendo este parámetro en "true". Cuando este parámetro se establece en "true", todos los comandos se consideran conmutativos y notifican un bloqueo compartido, lo que puede mejorar el rendimiento global. Este parámetro se puede establecer en tiempo de ejecución y tiene efecto en el coordinador.

#### <a name="citusremote_task_check_interval-integer"></a>citus.remote\_task\_check\_interval (entero)

Establece la frecuencia con la que Hiperescala (Citus) comprueba los estados de los trabajos administrados por el ejecutor del rastreador de tareas. Su valor predeterminado es 10 ms. El coordinador asigna tareas a los roles de trabajo y luego comprueba regularmente con ellos el progreso de cada tarea. Este valor de configuración establece el intervalo de tiempo entre dos comprobaciones consiguientes. Este parámetro tiene efecto en el coordinador y se puede establecer en tiempo de ejecución.

#### <a name="citustask_executor_type-enum"></a>citus.task\_executor\_type (enumeración)

Hiperescala (Citus) tiene tres tipos de ejecutor para ejecutar consultas SELECT distribuidas.  El ejecutor deseado se puede seleccionar estableciendo este parámetro de configuración. Los valores aceptables para este parámetro son los siguientes:

-   **adaptive:** El valor predeterminado. Resulta óptimo para obtener respuestas rápidas a consultas que implican agregaciones y combinaciones coubicadas que abarcan varias particiones.
-   **task-tracker:** El ejecutor del rastreador de tareas es adecuado para consultas complejas de larga duración que requieren el orden aleatorio de datos entre los nodos de trabajo y una administración de recursos eficaz.
-   **real-time:** (en desuso) sirve para un propósito similar al del ejecutor adaptable, pero es menos flexible y puede ejercer una mayor presión de conexión en los nodos de trabajo.

Este parámetro se puede establecer en tiempo de ejecución y tiene efecto en el coordinador.

#### <a name="citusmulti_task_query_log_level-enum-multi_task_logging"></a>citus.multi\_task\_query\_log\_level (enumeración) {#multi_task_logging}

Establece un nivel de registro para cualquier consulta que genere más de una tarea (es decir, que afecte a más de una partición). El registro resulta útil durante la migración de aplicaciones multiinquilino, ya que se puede optar por notificar los errores o advertir de estas consultas, para encontrarlas y agregarles un filtro por tenant\_id. Este parámetro se puede establecer en tiempo de ejecución y tiene efecto en el coordinador. El valor predeterminado de este parámetro es \'off\'.

Los valores admitidos para esta enumeración son:

-   **off:** desactiva el registro de las consultas que generan varias tareas (es decir, que abarcan varias particiones)
-   **debug:** registra la instrucción en el nivel de gravedad de DEBUG.
-   **log:** registra la instrucción en el nivel de gravedad de LOG. La línea de registro incluirá la consulta SQL que se ejecutó.
-   **notice:** registra la instrucción en el nivel de gravedad de NOTICE.
-   **warning:** registra la instrucción en el nivel de gravedad de WARNING.
-   **error:** registra la instrucción en el nivel de gravedad de ERROR.

Puede resultar útil usar `error` durante las pruebas de desarrollo y un nivel de registro inferior como `log` durante la implementación de producción real.
La elección de `log` hará que aparezcan consultas de varias tareas en los registros de la base de datos con la propia consulta mostrada después de \"STATEMENT.\"

```
LOG:  multi-task query about to be executed
HINT:  Queries are split to multiple tasks if they have to be split into several queries on the workers.
STATEMENT:  select * from foo;
```

#### <a name="citusenable_repartition_joins-boolean"></a>citus.enable\_repartition\_joins (booleano)

Normalmente, el intento de realizar combinaciones de volver a particionar con el ejecutor adaptable producirá un mensaje de error.  Sin embargo, si `citus.enable_repartition_joins` se establece en "true", se permite a Hiperescala (Citus) cambiar temporalmente al ejecutor del rastreador de tareas para realizar la combinación.  El valor predeterminado es false.

### <a name="task-tracker-executor-configuration"></a>Configuración del ejecutor del rastreador de tareas

#### <a name="citustask_tracker_delay-integer"></a>citus.task\_tracker\_delay (entero)

Este parámetro establece el tiempo de suspensión entre rondas de administración del rastreador de tareas y su valor predeterminado es 200 ms. El proceso del rastreador de tareas se reactiva regularmente, recorre todas las tareas que tiene asignadas y programa y ejecuta esas tareas.  Después, el rastreador de tareas se suspende durante un período de tiempo antes de volver a recorrer esas tareas.
Este valor de configuración determina la duración del período de suspensión. Este parámetro tiene efecto en los roles de trabajo y debe cambiarse en el archivo postgresql.conf. Después de editar el archivo de configuración, los usuarios pueden enviar una señal SIGHUP o reiniciar el servidor para que el cambio surta efecto.

Este parámetro se puede reducir para recortar el retraso causado por el ejecutor del rastreador de tareas reduciendo el intervalo de tiempo entre las rondas de administración.
Reducir el retraso es útil en casos en los que las consultas de particiones son breves y, por tanto, actualizan su estado con regularidad.

#### <a name="citusmax_assign_task_batch_size-integer"></a>citus.max\_assign\_task\_batch\_size (entero)

El ejecutor del rastreador de tareas en el coordinador asigna de forma sincrónica tareas en lotes al demonio en los roles de trabajo. Este parámetro establece el número máximo de tareas que se asignan en un solo lote. Elegir un tamaño de lote mayor permite una asignación de tareas más rápida. Sin embargo, si el número de roles de trabajo es grande, puede que se tarde más para que todos los roles de trabajo obtengan tareas.  Este parámetro se puede establecer en tiempo de ejecución y tiene efecto en el coordinador.

#### <a name="citusmax_running_tasks_per_node-integer"></a>citus.max\_running\_tasks\_per\_node (entero)

El proceso del rastreador de tareas programa y ejecuta las tareas que se le asignan según corresponda. Este valor de configuración establece el número máximo de tareas que se ejecutan simultáneamente en un nodo en un momento dado y el valor predeterminado es 8.

El límite garantiza que no haya muchas tareas que alcanzan el disco al mismo tiempo y ayuda a evitar la contención de E/S del disco. Si las consultas se envían desde la memoria o desde unidades de estado sólido, se puede aumentar la función max\_running\_tasks\_per\_node sin preocuparse.

#### <a name="cituspartition_buffer_size-integer"></a>citus.partition\_buffer\_size (entero)

Establece el tamaño de búfer que se va a usar para las operaciones de partición y su valor predeterminado es 8 MB.
Hiperescala (Citus) permite volver a particionar los datos de la tabla en varios archivos cuando se combinan dos tablas grandes. Cuando el búfer de esta partición se llene, los datos que se han vuelto a particionar se descargarán en archivos del disco.  Esta entrada de configuración se puede establecer en tiempo de ejecución y tiene efecto en los roles de trabajo.

### <a name="explain-output"></a>Salida de EXPLAIN

#### <a name="citusexplain_all_tasks-boolean"></a>citus.explain\_all\_tasks (booleano)

De forma predeterminada, Hiperescala (Citus) muestra la salida de una única tarea arbitraria al ejecutar [EXPLAIN](http://www.postgresql.org/docs/current/static/sql-explain.html) en una consulta distribuida. En la mayoría de los casos, la salida de EXPLAIN será similar en todas las tareas. En ocasiones, algunas de las tareas se planearán de manera diferente o se producirán tiempos de ejecución mucho mayores. En esos casos, puede resultar útil habilitar este parámetro, tras el cual la salida de EXPLAIN incluirá todas las tareas. La explicación de todas las tareas puede hacer que EXPLAIN tarde más.

## <a name="next-steps"></a>Pasos siguientes

* Otra forma de configuración, además de los parámetros de servidor, son las [opciones de configuración](concepts-hyperscale-configuration-options.md) de recursos en un grupo de servidores de Hiperescala (Citus).
* La base de datos PostgreSQL subyacente también tiene [parámetros de configuración](http://www.postgresql.org/docs/current/static/runtime-config.html).
