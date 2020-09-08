---
title: 'Modificación de tablas distribuidas: Hiperescala (Citus) (Azure Database for PostgreSQL)'
description: 'Comandos SQL para crear y modificar tablas distribuidas: Hiperescala (Citus) mediante Azure Portal'
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 8/10/2020
ms.openlocfilehash: 628944f9763dc79148e0b64c97158064208412bf
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88136692"
---
# <a name="distribute-and-modify-tables"></a>Distribución y modificación de tablas

## <a name="distributing-tables"></a>Distribución de tablas

Para crear una tabla distribuida, primero debe definir el esquema de la tabla. Para ello, puede definir una tabla con la instrucción [CREATE TABLE](http://www.postgresql.org/docs/current/static/sql-createtable.html) de la misma manera que lo haría con una tabla normal de PostgreSQL.

```sql
CREATE TABLE github_events
(
    event_id bigint,
    event_type text,
    event_public boolean,
    repo_id bigint,
    payload jsonb,
    repo jsonb,
    actor jsonb,
    org jsonb,
    created_at timestamp
);
```

A continuación, puede usar la función create\_distributed\_table() para especificar la columna de distribución de la tabla y crear las particiones de trabajo.

```sql
SELECT create_distributed_table('github_events', 'repo_id');
```

La llamada de la función informa a Hiperescala (Citus) que la tabla github\_events debe distribuirse en la columna repo\_id (mediante el hash del valor de la columna). La función también crea particiones en los nodos de trabajo mediante los valores de configuración citus.shard\_count y citus.shard\_replication\_factor.

Crea un número total de particiones citus.shard\_count, donde cada partición posee una parte de un espacio de hash y se replica en función del valor de configuración predeterminado de citus.shard\_replication\_factor. Las réplicas de la partición creadas en el trabajo tienen el mismo esquema de tabla, índice y definiciones de restricción que la tabla en el coordinador. Una vez creadas las réplicas, la función guarda todos los metadatos distribuidos en el coordinador.

A cada partición creada se le asigna un identificador de partición único, y todas sus réplicas tienen el mismo identificador de partición. Las particiones se representan en el nodo de trabajo como tablas estándar de PostgreSQL con el nombre \'tablename\_shardid\', donde tablename es el nombre de la tabla distribuida y shardid es el identificador único asignado. Puede conectarse a las instancias de Postgres de trabajo para ver o ejecutar comandos en particiones individuales.

Ahora está listo para insertar datos en la tabla distribuida y ejecutar consultas en ella. También puede obtener más información sobre la UDF que se usa en esta sección en la referencia [DDL de tabla y partición](reference-hyperscale-functions.md#table-and-shard-ddl).

### <a name="reference-tables"></a>Tablas de referencia

El método anterior distribuye las tablas en varias particiones horizontales.  Otra posibilidad es distribuir las tablas en una sola partición y replicar la partición en cada nodo de trabajo. Las tablas distribuidas de esta manera se denominan *tablas de referencia*. Se usan para almacenar datos a los que varios nodos de un clúster tienen que acceder con frecuencia.

Entre los candidatos comunes para las tablas de referencia se incluyen:

-   Tablas más pequeñas que deben combinarse con tablas distribuidas de mayor tamaño.
-   Tablas en aplicaciones multiinquilino que carecen de una columna de identificador de inquilino o que no están asociadas a un inquilino. (O bien, durante la migración, incluso para algunas tablas asociadas a un inquilino).
-   Tablas que necesitan restricciones exclusivas en varias columnas y son lo suficientemente pequeñas.

Por ejemplo, suponga que un sitio de comercio electrónico multiinquilino necesita calcular el impuesto de ventas de las transacciones en cualquiera de sus tiendas. La información fiscal no es específica de ningún inquilino. Tiene sentido colocarla en una tabla compartida. Una tabla de referencia con sede en Estados Unidos podría tener el siguiente aspecto:

```postgresql
-- a reference table

CREATE TABLE states (
  code char(2) PRIMARY KEY,
  full_name text NOT NULL,
  general_sales_tax numeric(4,3)
);

-- distribute it to all workers

SELECT create_reference_table('states');
```

Ahora, las consultas como un cálculo de impuestos para un carro de la compra pueden combinarse en la tabla de `states` sin sobrecarga de la red, y pueden agregar una clave externa al código de estado para una mejor validación.

Además de distribuir una tabla como partición única replicada, la UDF `create_reference_table` la marca como tabla de referencia en las tablas de metadatos de Hiperescala (Citus). Hiperescala (Citus) realiza automáticamente confirmaciones en dos fases ([2PC](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)) para las modificaciones en las tablas marcadas de esta manera, lo que proporciona garantías de coherencia alta.

Si tiene una tabla distribuida con un recuento de particiones de uno, puede actualizarla para que sea una tabla de referencia reconocida como sigue:

```postgresql
SELECT upgrade_to_reference_table('table_name');
```

Para ver otro ejemplo de uso de las tablas de referencia, consulte el [tutorial de bases de datos multiinquilino](tutorial-design-database-hyperscale-multi-tenant.md).

### <a name="distributing-coordinator-data"></a>Distribución de datos de coordinador

Si una base de datos PostgreSQL existente se convierte en el nodo de coordinación de un clúster de Hiperescala (Citus), los datos en sus tablas se pueden distribuir de forma eficaz y con una interrupción mínima en una aplicación.

La función `create_distributed_table` que se ha descrito anteriormente sirve tanto para tablas vacías como no vacías y, en este segundo caso, distribuye automáticamente las filas de la tabla en todo el clúster. Sabrá si copia los datos por la presencia del mensaje, \"NOTICE: Copying data from local table\...\" Por ejemplo:

```postgresql
CREATE TABLE series AS SELECT i FROM generate_series(1,1000000) i;
SELECT create_distributed_table('series', 'i');
NOTICE:  Copying data from local table...
 create_distributed_table
 --------------------------

 (1 row)
```

Las operaciones de escritura en la tabla están bloqueadas mientras se migran los datos, y las escrituras pendientes se controlan como consultas distribuidas una vez que se confirma la función. (Si se produce un error en la función, las consultas vuelven a ser locales). Las lecturas pueden continuar de forma normal y se convertirán en consultas distribuidas una vez que la función se confirme.

Al distribuir las tablas A y B, donde A tiene una clave externa hacia B, distribuya primero la tabla de destino de claves B. Si lo hace en el orden equivocado, se producirá un error:

```
ERROR:  cannot create foreign key constraint
DETAIL:  Referenced table must be a distributed table or a reference table.
```

Si no es posible distribuir en el orden correcto, quite las claves externas, distribuya las tablas y vuelva a crear las claves externas.

Al migrar datos desde una base de datos externa, como desde Amazon RDS a la nube de Hiperescala (Citus), cree primero las tablas distribuidas de Hiperescala (Citus) mediante `create_distributed_table` y, a continuación, copie los datos en la tabla.
La copia en tablas distribuidas evita quedarse sin espacio en el nodo de coordinación.

## <a name="colocating-tables"></a>Coubicación de tablas

"Coubicación" significa mantener información relacionada en las mismas máquinas. Permite realizar consultas eficaces, a la vez que aprovecha la escalabilidad horizontal para todo el conjunto de datos. Para obtener más información, consulte [Coubicación](concepts-hyperscale-colocation.md).

Las tablas se coubican en grupos. Para controlar manualmente la asignación del grupo de coubicación de una tabla, use el parámetro opcional `colocate_with` de `create_distributed_table`. Si no le interese la coubicación de una tabla, omita este parámetro. Toma `'default'` como valor predeterminado, lo que agrupa la tabla con cualquier otra tabla de coubicación predeterminada que tenga el mismo tipo de columna de distribución, recuento de particiones y factor de replicación. Si desea interrumpir o actualizar esta coubicación implícita, puede usar `update_distributed_table_colocation()`.

```postgresql
-- these tables are implicitly co-located by using the same
-- distribution column type and shard count with the default
-- co-location group

SELECT create_distributed_table('A', 'some_int_col');
SELECT create_distributed_table('B', 'other_int_col');
```

Cuando una tabla nueva no está relacionada con otras de su supuesto grupo de coubicación implícito, especifique `colocated_with => 'none'`.

```postgresql
-- not co-located with other tables

SELECT create_distributed_table('A', 'foo', colocate_with => 'none');
```

La división de tablas no relacionadas en sus propios grupos de coubicación mejorará el rendimiento de [reequilibrio de particiones](howto-hyperscale-scaling.md#rebalance-shards), ya que las particiones del mismo grupo deben moverse juntas.

Cuando las tablas sí están relacionadas (por ejemplo, cuando se van a combinar), puede tener sentido coubicarlas de manera explícita. Las ventajas de la coubicación adecuada son más importantes que cualquier sobrecarga de reequilibrio.

Para coubicar de manera explícita varias tablas, distribuya una y luego coloque las demás en su grupo de coubicación. Por ejemplo:

```postgresql
-- distribute stores
SELECT create_distributed_table('stores', 'store_id');

-- add to the same group as stores
SELECT create_distributed_table('orders', 'store_id', colocate_with => 'stores');
SELECT create_distributed_table('products', 'store_id', colocate_with => 'stores');
```

La información sobre los grupos de coubicación se almacena en la tabla [pg_dist_colocation](reference-hyperscale-metadata.md#colocation-group-table), mientras que en [pg_dist_partition](reference-hyperscale-metadata.md#partition-table) se muestra qué tablas están asignadas a qué grupos.

## <a name="dropping-tables"></a>Eliminación de tablas

Puede usar el comando DROP TABLE estándar de PostgreSQL para quitar las tablas distribuidas. Al igual que con las tablas normales, DROP TABLE quita los índices, las reglas, los desencadenadores y las restricciones que existen para la tabla de destino. Además, también quita las particiones en los nodos de trabajo y limpia los metadatos.

```sql
DROP TABLE github_events;
```

## <a name="modifying-tables"></a>Modificación de tablas

Hiperescala (Citus) propaga automáticamente muchos tipos de instrucciones DDL.
La modificación de una tabla distribuida en el nodo de coordinación también actualizará las particiones en los trabajos. Otras instrucciones DDL requieren la propagación manual y algunas otras están prohibidas, como las que modificarían una columna de distribución.
Al intentar ejecutar un DDL que no sea válido para la propagación automática, se producirá un error y las tablas se mantendrán sin modificar en el nodo de coordinación.

A continuación se muestra una referencia de las categorías de instrucciones DDL que se propagan.
La propagación automática se puede habilitar o deshabilitar con un [parámetro de configuración](reference-hyperscale-parameters.md#citusenable_ddl_propagation-boolean).

### <a name="addingmodifying-columns"></a>Adición o modificación de columnas

Hiperescala (Citus) propaga la mayoría de los comandos [ALTER TABLE](https://www.postgresql.org/docs/current/static/ddl-alter.html) de forma automática. Agregar columnas o cambiar sus valores predeterminados funcionan como lo harían en una base de datos PostgreSQL de una sola máquina:

```postgresql
-- Adding a column

ALTER TABLE products ADD COLUMN description text;

-- Changing default value

ALTER TABLE products ALTER COLUMN price SET DEFAULT 7.77;
```

Los cambios significativos en una columna existente, como el cambio de nombre o el cambio de su tipo de datos, también se permiten. Sin embargo, no se puede modificar el tipo de datos de la [columna de distribución](concepts-hyperscale-nodes.md#distribution-column).
Esta columna determina cómo se distribuyen los datos de la tabla a través del clúster de Hiperescala (Citus), y la modificación de su tipo de datos exigiría mover los datos.

Si se intenta hacerlo, se producirá un error:

```postgres
-- assumining store_id is the distribution column
-- for products, and that it has type integer

ALTER TABLE products
ALTER COLUMN store_id TYPE text;

/*
ERROR:  XX000: cannot execute ALTER TABLE command involving partition column
LOCATION:  ErrorIfUnsupportedAlterTableStmt, multi_utility.c:2150
*/
```

### <a name="addingremoving-constraints"></a>Adición o eliminación de restricciones

El uso de Hiperescala (Citus) le permite seguir disfrutando de la seguridad de una base de datos relacional, incluidas las restricciones de la base de datos (consulte la [documentación](https://www.postgresql.org/docs/current/static/ddl-constraints.html) de PostgreSQL).
Debido a la naturaleza de los sistemas distribuidos, Hiperescala (Citus) no hará referencia cruzada a las restricciones de unicidad ni a la integridad referencial entre los nodos de trabajo.

Para configurar una clave externa entre las tablas distribuidas coubicadas, incluya siempre la columna de distribución en la clave. La inclusión de la columna de distribución puede implicar que la clave se haga compuesta.

Se pueden crear claves externas en las situaciones siguientes:

-   entre dos tablas locales (no distribuidas);
-   entre dos tablas de referencia;
-   entre dos tablas distribuidas [coubicadas](concepts-hyperscale-colocation.md) cuando la clave incluye la columna de distribución;
-   como tabla distribuida que hace referencia a una [tabla de referencia](concepts-hyperscale-nodes.md#type-2-reference-tables).

No se admiten las claves externas de tablas de referencia a tablas distribuidas.

> [!NOTE]
>
> Las restricciones de clave principal y unicidad deben incluir la columna de distribución. Si se agregan a una columna que no es de distribución, se generará un error.

En este ejemplo se muestra cómo crear claves principales y externas en tablas distribuidas:

```postgresql
--
-- Adding a primary key
-- --------------------

-- We'll distribute these tables on the account_id. The ads and clicks
-- tables must use compound keys that include account_id.

ALTER TABLE accounts ADD PRIMARY KEY (id);
ALTER TABLE ads ADD PRIMARY KEY (account_id, id);
ALTER TABLE clicks ADD PRIMARY KEY (account_id, id);

-- Next distribute the tables

SELECT create_distributed_table('accounts', 'id');
SELECT create_distributed_table('ads',      'account_id');
SELECT create_distributed_table('clicks',   'account_id');

--
-- Adding foreign keys
-- -------------------

-- Note that this can happen before or after distribution, as long as
-- there exists a uniqueness constraint on the target column(s) which
-- can only be enforced before distribution.

ALTER TABLE ads ADD CONSTRAINT ads_account_fk
  FOREIGN KEY (account_id) REFERENCES accounts (id);
ALTER TABLE clicks ADD CONSTRAINT clicks_ad_fk
  FOREIGN KEY (account_id, ad_id) REFERENCES ads (account_id, id);
```

Del mismo modo, incluya la columna de distribución en las restricciones de unicidad:

```postgresql
-- Suppose we want every ad to use a unique image. Notice we can
-- enforce it only per account when we distribute by account id.

ALTER TABLE ads ADD CONSTRAINT ads_unique_image
  UNIQUE (account_id, image_url);
```

Las restricciones que no son nulas se pueden aplicar a cualquier columna (de distribución o no) porque no requieren búsquedas entre trabajos.

```postgresql
ALTER TABLE ads ALTER COLUMN image_url SET NOT NULL;
```

### <a name="using-not-valid-constraints"></a>Uso de restricciones NOT VALID

En algunas situaciones puede ser útil aplicar restricciones a las nuevas filas, al tiempo que se permite que las filas no conformes existentes permanezcan sin cambios. Hiperescala (Citus) admite esta característica para restricciones CHECK y claves externas, con la designación de restricción \"NOT VALID\" de PostgreSQL.

Por ejemplo, considere una aplicación que almacena perfiles de usuario en una [tabla de referencia](concepts-hyperscale-nodes.md#type-2-reference-tables).

```postgres
-- we're using the "text" column type here, but a real application
-- might use "citext" which is available in a postgres contrib module

CREATE TABLE users ( email text PRIMARY KEY );
SELECT create_reference_table('users');
```

Con el paso del tiempo, imagine que algunos datos que no son direcciones se incluyen en la tabla.

```postgres
INSERT INTO users VALUES
   ('foo@example.com'), ('hacker12@aol.com'), ('lol');
```

Querríamos validar las direcciones, pero PostgreSQL no nos permite normalmente agregar una restricción CHECK que produzca un error en las filas existentes. Sin embargo, *sí* permitir una restricción marcada como no válida:

```postgres
ALTER TABLE users
ADD CONSTRAINT syntactic_email
CHECK (email ~
   '^[a-zA-Z0-9.!#$%&''*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$'
) NOT VALID;
```

Ahora las nuevas filas están protegidas.

```postgres
INSERT INTO users VALUES ('fake');

/*
ERROR:  new row for relation "users_102010" violates
        check constraint "syntactic_email_102010"
DETAIL:  Failing row contains (fake).
*/
```

Más adelante, durante las horas de poca actividad, un administrador de bases de datos puede intentar corregir las filas incorrectas y volver a validar la restricción.

```postgres
-- later, attempt to validate all rows
ALTER TABLE users
VALIDATE CONSTRAINT syntactic_email;
```

La documentación de PostgreSQL contiene más información sobre NOT VALID y VALIDATE CONSTRAINT en la sección [ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html).

### <a name="addingremoving-indices"></a>Adición o eliminación de índices

Hiperescala (Citus) permite agregar y quitar [índices](https://www.postgresql.org/docs/current/static/sql-createindex.html):

```postgresql
-- Adding an index

CREATE INDEX clicked_at_idx ON clicks USING BRIN (clicked_at);

-- Removing an index

DROP INDEX clicked_at_idx;
```

Agregar un índice exige un bloqueo de escritura, lo que puede no ser deseable en un \"sistema de registro\" multiinquilino. Para minimizar el tiempo de inactividad de la aplicación, cree el índice [simultáneamente](https://www.postgresql.org/docs/current/static/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY) en su lugar. Este método requiere más trabajo total que una compilación de índice estándar y tarda más tiempo en completarse. Sin embargo, dado que permite que las operaciones normales continúen mientras se crea el índice, este método es útil para agregar nuevos índices en un entorno de producción.

```postgresql
-- Adding an index without locking table writes

CREATE INDEX CONCURRENTLY clicked_at_idx ON clicks USING BRIN (clicked_at);
```
