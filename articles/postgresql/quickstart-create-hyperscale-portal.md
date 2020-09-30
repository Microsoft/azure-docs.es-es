---
title: 'Inicio rápido: Creación de un grupo de servidores (Citus): Azure Database for PostgreSQL'
description: Inicio rápido para crear y consultar tablas distribuidas en Hiperescala (Citus) de Azure Database for PostgreSQL.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.topic: quickstart
ms.date: 08/17/2020
ms.openlocfilehash: 1a16283f3d04c9ad331a04c3a36b49055635d76e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906499"
---
# <a name="quickstart-create-a-hyperscale-citus-server-group-in-the-azure-portal"></a>Inicio rápido: Creación de un grupo de servidores Citus en Azure Portal

Azure Database for PostgreSQL es un servicio administrado que usa para ejecutar, administrar y escalar bases de datos de PostgreSQL de alta disponibilidad en la nube. Este inicio rápido muestra cómo crear un grupo de servidores de Hiperescala (Citus) de Azure Database for PostgreSQL mediante Azure Portal. Explorará los datos distribuidos: tablas de particionamiento entre nodos, ingesta de datos de ejemplo y ejecución de consultas que se ejecutan en varios nodos.

[!INCLUDE [azure-postgresql-hyperscale-create-db](../../includes/azure-postgresql-hyperscale-create-db.md)]

## <a name="create-and-distribute-tables"></a>Creación y distribución de tablas

Una vez conectado al nodo coordinador de hiperescala mediante psql, puede completar algunas tareas básicas.

Dentro de los servidores Hiperescala hay tres tipos de tablas:

- Tablas particionadas o distribuidas (repartidas para ayudar a escalar para el rendimiento y paralelización)
- Tablas de referencia (se mantienen varias copias)
- Tablas locales (se usan frecuentemente para las tablas de administración interna)

En este inicio rápido nos centraremos principalmente en las tablas distribuidas para familiarizarnos con ellas.

El modelo de datos con el que vamos a trabajar es simple: datos de usuario y eventos desde GitHub. Los eventos incluyen la creación de una bifurcación, confirmaciones git relacionadas con una organización y más.

Una vez que se haya conectado a través de psql vamos a crear nuestras tablas. En la consola de psql ejecute lo siguiente:

```sql
CREATE TABLE github_events
(
    event_id bigint,
    event_type text,
    event_public boolean,
    repo_id bigint,
    payload jsonb,
    repo jsonb,
    user_id bigint,
    org jsonb,
    created_at timestamp
);

CREATE TABLE github_users
(
    user_id bigint,
    url text,
    login text,
    avatar_url text,
    gravatar_id text,
    display_login text
);
```

El campo `payload` de `github_events` tiene un tipo de datos JSONB. JSONB es el tipo de datos JSON en formato binario en Postgres. Este tipo de datos facilita el almacenamiento de un esquema flexible en una sola columna.

Postgres puede crear un índice `GIN` en este tipo que indexará cada clave y valor dentro de él. Con un índice, consultar la carga con varias condiciones pasa a ser una tarea rápida y fácil. Sigamos adelante, ahora vamos a crear un par de índices antes de cargar los datos. En psql:

```sql
CREATE INDEX event_type_index ON github_events (event_type);
CREATE INDEX payload_index ON github_events USING GIN (payload jsonb_path_ops);
```

A continuación con esas tablas Postgres en el nodo de coordinación, indicaremos a Hiperescala que las particione entre los roles de trabajo. Para ello, ejecutaremos una consulta para cada tabla especificando la clave en la que se va a particionar. En el ejemplo actual, vamos a particionar la tabla de usuarios y de eventos en `user_id`:

```sql
SELECT create_distributed_table('github_events', 'user_id');
SELECT create_distributed_table('github_users', 'user_id');
```

[!INCLUDE [azure-postgresql-hyperscale-dist-alert](../../includes/azure-postgresql-hyperscale-dist-alert.md)]

Ya estamos preparados para cargar los datos. En psql todavía, aplique un shell para descargar los archivos:

```sql
\! curl -O https://examples.citusdata.com/users.csv
\! curl -O https://examples.citusdata.com/events.csv
```

A continuación, cargue los datos de los archivos en las tablas distribuidas:

```sql
SET CLIENT_ENCODING TO 'utf8';

\copy github_events from 'events.csv' WITH CSV
\copy github_users from 'users.csv' WITH CSV
```

## <a name="run-queries"></a>Ejecutar consultas

Ahora llega la parte más divertida: vamos a ejecutar algunas consultas. Comencemos con un sencillo `count (*)` para ver cuántos datos hemos cargado:

```sql
SELECT count(*) from github_events;
```

Esto ha ido muy bien. Volveremos a ese tipo de agregación más tarde, pero ahora vamos a echar un vistazo a otras consultas. Dentro de la columna `payload` de JSONB existe una cantidad importante de datos, pero varían en función del tipo de evento. Los eventos `PushEvent` contienen un tamaño que incluye el número de diferentes confirmaciones para la inserción. Se puede usar para buscar el número total de confirmaciones por hora:

```sql
SELECT date_trunc('hour', created_at) AS hour,
       sum((payload->>'distinct_size')::int) AS num_commits
FROM github_events
WHERE event_type = 'PushEvent'
GROUP BY hour
ORDER BY hour;
```

Hasta ahora, las consultas que hemos realizado solo han utilizado github\_events, pero podemos combinar esta información con github\_users. Como hemos particionado tanto los usuarios como los eventos en el mismo identificador (`user_id`), las filas de ambas tablas con identificadores de usuario que coincidan se [coubicarán](concepts-hyperscale-colocation.md) en los mismos nodos de base de datos con lo que pueden combinarse fácilmente.

Si incorporamos `user_id`, Hiperescala puede insertar la ejecución conjunta en particiones para su ejecución en paralelo en nodos de trabajo. Por ejemplo, vamos a buscar los usuarios que hayan creado el mayor número de repositorios:

```sql
SELECT gu.login, count(*)
  FROM github_events ge
  JOIN github_users gu
    ON ge.user_id = gu.user_id
 WHERE ge.event_type = 'CreateEvent'
   AND ge.payload @> '{"ref_type": "repository"}'
 GROUP BY gu.login
 ORDER BY count(*) DESC;
```

## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, creó recursos de Azure en un grupo de servidores. Si no cree que vaya a necesitar estos recursos en el futuro, elimine el grupo de servidores. Presione el botón **Eliminar** en la página **Información general** del grupo de servidores. Cuando aparezca una página emergente en la que se le pida hacerlo, confirme el nombre del grupo de servidores y haga clic en el botón **Eliminar** final.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, aprendió a aprovisionar un grupo de servidores de Hiperescala (Citus). Se conectó a él con psql, creó un esquema y distribuyó datos.

- Siga un tutorial para [crear aplicaciones escalables multiinquilino](https://aka.ms/hyperscale-tutorial-multi-tenant).
- Determine el mejor [tamaño inicial](howto-hyperscale-scaling.md#picking-initial-size) para el grupo de servidores.
