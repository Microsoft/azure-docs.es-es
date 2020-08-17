---
title: 'Tutorial: Diseño de una base de datos multiinquilino: Hiperescala (Citus): Azure Database for PostgreSQL'
description: Este tutorial muestra cómo crear, rellenar y consultar tablas distribuidas en Azure Database for PostgreSQL Hiperescala (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 05/14/2019
ms.openlocfilehash: bc7891e157bbffa386396a352fd1d48e4559ecdc
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386406"
---
# <a name="tutorial-design-a-multi-tenant-database-by-using-azure-database-for-postgresql--hyperscale-citus"></a>Tutorial: Diseño de una base de datos multiinquilino con Azure Database for PostgreSQL Hiperescala (Citus)

En este tutorial, usará Azure Database for PostgreSQL Hiperescala (Citus) para aprender lo siguiente:

> [!div class="checklist"]
> * Creación de un grupo de servidores Hiperescala (Citus)
> * Uso de la utilidad psql para crear un esquema
> * Particiones de tablas entre nodos
> * Ingesta de datos de ejemplo
> * Consulta de datos de inquilino
> * Datos compartidos entre los inquilinos
> * Personalización de esquema por inquilino

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [azure-postgresql-hyperscale-create-db](../../includes/azure-postgresql-hyperscale-create-db.md)]

## <a name="use-psql-utility-to-create-a-schema"></a>Uso de la utilidad psql para crear un esquema

Una vez conectado a Azure Database for PostgreSQL Hiperescala (Citus) mediante psql, puede completar algunas tareas básicas. Este tutorial explica cómo crear una aplicación web que permite a los anunciantes realizar un seguimiento de sus campañas.

Varias empresas pueden usar la aplicación, así que crearemos una tabla que contenga las empresas y otra para sus campañas. En la consola de psql, ejecute estos comandos:

```sql
CREATE TABLE companies (
  id bigserial PRIMARY KEY,
  name text NOT NULL,
  image_url text,
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL
);

CREATE TABLE campaigns (
  id bigserial,
  company_id bigint REFERENCES companies (id),
  name text NOT NULL,
  cost_model text NOT NULL,
  state text NOT NULL,
  monthly_budget bigint,
  blacklisted_site_urls text[],
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL,

  PRIMARY KEY (company_id, id)
);
```

Cada campaña pagará por anunciarse. Agregue también una tabla para anuncios, ejecutando el siguiente código en psql después del código anterior:

```sql
CREATE TABLE ads (
  id bigserial,
  company_id bigint,
  campaign_id bigint,
  name text NOT NULL,
  image_url text,
  target_url text,
  impressions_count bigint DEFAULT 0,
  clicks_count bigint DEFAULT 0,
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, campaign_id)
    REFERENCES campaigns (company_id, id)
);
```

Por último, realizaremos un seguimiento de las estadísticas de clics e impresiones por cada anuncio:

```sql
CREATE TABLE clicks (
  id bigserial,
  company_id bigint,
  ad_id bigint,
  clicked_at timestamp without time zone NOT NULL,
  site_url text NOT NULL,
  cost_per_click_usd numeric(20,10),
  user_ip inet NOT NULL,
  user_data jsonb NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, ad_id)
    REFERENCES ads (company_id, id)
);

CREATE TABLE impressions (
  id bigserial,
  company_id bigint,
  ad_id bigint,
  seen_at timestamp without time zone NOT NULL,
  site_url text NOT NULL,
  cost_per_impression_usd numeric(20,10),
  user_ip inet NOT NULL,
  user_data jsonb NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, ad_id)
    REFERENCES ads (company_id, id)
);
```

Ahora puede ver las tablas recién creadas en la lista de tablas con este comando de psql:

```postgres
\dt
```

Las aplicaciones multiinquilino pueden exigir exclusividad solo por inquilino, por lo que todas las claves principales y externas incluyen el identificador de la empresa.

## <a name="shard-tables-across-nodes"></a>Particiones de tablas entre nodos

Una implementación de hiperescala almacena filas de tablas en distintos nodos en función del valor de una columna designada por el usuario. Esta "columna de distribución" indica qué filas pertenecen a qué inquilino.

Vamos a establecer la columna de distribución como company\_id, el identificador del inquilino. En psql, ejecute estas funciones:

```sql
SELECT create_distributed_table('companies',   'id');
SELECT create_distributed_table('campaigns',   'company_id');
SELECT create_distributed_table('ads',         'company_id');
SELECT create_distributed_table('clicks',      'company_id');
SELECT create_distributed_table('impressions', 'company_id');
```

[!INCLUDE [azure-postgresql-hyperscale-dist-alert](../../includes/azure-postgresql-hyperscale-dist-alert.md)]

## <a name="ingest-sample-data"></a>Ingesta de datos de ejemplo

Ahora, fuera de psql, en la línea de comandos normal, descargue conjuntos de datos de ejemplo:

```bash
for dataset in companies campaigns ads clicks impressions geo_ips; do
  curl -O https://examples.citusdata.com/mt_ref_arch/${dataset}.csv
done
```

De vuelta en psql, realice una carga masiva de los datos. No olvide ejecutar psql en el mismo directorio donde descargó los archivos de datos.

```sql
SET CLIENT_ENCODING TO 'utf8';

\copy companies from 'companies.csv' with csv
\copy campaigns from 'campaigns.csv' with csv
\copy ads from 'ads.csv' with csv
\copy clicks from 'clicks.csv' with csv
\copy impressions from 'impressions.csv' with csv
```

Estos datos ahora se distribuirán entre nodos de trabajo.

## <a name="query-tenant-data"></a>Consulta de datos de inquilino

Cuando la aplicación solicita datos de un único inquilino, la base de datos puede ejecutar la consulta en un único nodo de trabajo. Las consultas de un único inquilino filtran por un único identificador de inquilino. Por ejemplo, la siguiente consulta filtra `company_id = 5` para anuncios e impresiones. Intente ejecutarla en psql y observe los resultados.

```sql
SELECT a.campaign_id,
       RANK() OVER (
         PARTITION BY a.campaign_id
         ORDER BY a.campaign_id, count(*) desc
       ), count(*) as n_impressions, a.id
  FROM ads as a
  JOIN impressions as i
    ON i.company_id = a.company_id
   AND i.ad_id      = a.id
 WHERE a.company_id = 5
GROUP BY a.campaign_id, a.id
ORDER BY a.campaign_id, n_impressions desc;
```

## <a name="share-data-between-tenants"></a>Datos compartidos entre los inquilinos

Hasta ahora `company_id` ha distribuido todas las tablas, pero algunos datos no "pertenecen" de manera natural a ningún inquilino en particular y se pueden compartir. Por ejemplo, es posible que todas las empresas de la plataforma de anuncios del ejemplo deseen obtener información geográfica sobre su audiencia según las direcciones IP.

Cree una tabla que contenga información geográfica compartida. Ejecute los siguientes comandos en psql:

```sql
CREATE TABLE geo_ips (
  addrs cidr NOT NULL PRIMARY KEY,
  latlon point NOT NULL
    CHECK (-90  <= latlon[0] AND latlon[0] <= 90 AND
           -180 <= latlon[1] AND latlon[1] <= 180)
);
CREATE INDEX ON geo_ips USING gist (addrs inet_ops);
```

A continuación convierta `geo_ips` en una "tabla de referencia" para almacenar una copia de la tabla en cada nodo de trabajo.

```sql
SELECT create_reference_table('geo_ips');
```

Cargue en ella datos de ejemplo. No olvide ejecutar este comando en psql desde dentro del directorio donde descargó el conjunto de datos.

```sql
\copy geo_ips from 'geo_ips.csv' with csv
```

La combinación de la tabla de clics con geo\_ips es eficaz en todos los nodos.
Esta es una combinación para encontrar las ubicaciones de todas las personas que hicieron clic en el anuncio.
290. Intente ejecutar la consulta en psql.

```sql
SELECT c.id, clicked_at, latlon
  FROM geo_ips, clicks c
 WHERE addrs >> c.user_ip
   AND c.company_id = 5
   AND c.ad_id = 290;
```

## <a name="customize-the-schema-per-tenant"></a>Personalización de esquema por inquilino

Cada inquilino puede necesitar almacenar información especial que otros no necesitan. Sin embargo, todos los inquilinos comparten una infraestructura común con un esquema de base de datos idéntico. ¿Dónde pueden ubicarse los datos adicionales?

Un truco consiste en usar un tipo de columna abierto, como JSONB de PostgreSQL.  Nuestro esquema tiene un campo JSONB en `clicks` llamado `user_data`.
Una empresa (por ejemplo, la número cinco) puede usar la columna para realizar un seguimiento que determine si el usuario utiliza un dispositivo móvil.

Esta es una consulta para determinar qué usuarios hacen más clics: los visitantes tradicionales o los usuarios de dispositivos móviles.

```sql
SELECT
  user_data->>'is_mobile' AS is_mobile,
  count(*) AS count
FROM clicks
WHERE company_id = 5
GROUP BY user_data->>'is_mobile'
ORDER BY count DESC;
```

Podemos optimizar esta consulta para una empresa mediante la creación de un [índice parcial](https://www.postgresql.org/docs/current/static/indexes-partial.html).

```sql
CREATE INDEX click_user_data_is_mobile
ON clicks ((user_data->>'is_mobile'))
WHERE company_id = 5;
```

Por lo general, podemos crear un [índices GIN](https://www.postgresql.org/docs/current/static/gin-intro.html) en cada clave y valor de la columna.

```sql
CREATE INDEX click_user_data
ON clicks USING gin (user_data);

-- this speeds up queries like, "which clicks have
-- the is_mobile key present in user_data?"

SELECT id
  FROM clicks
 WHERE user_data ? 'is_mobile'
   AND company_id = 5;
```

## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, creó recursos de Azure en un grupo de servidores. Si no cree que vaya a necesitar estos recursos en el futuro, elimine el grupo de servidores. Presione el botón *Eliminar* en la página *Información general* del grupo de servidores. Cuando aparezca una página emergente en la que se le pida hacerlo, confirme el nombre del grupo de servidores y haga clic en el botón *Eliminar* final.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a aprovisionar un grupo de servidores Hiperescala (Citus). Se conectó a él con psql, creó un esquema y distribuyó datos. Ha aprendido a consultar los datos dentro de los inquilinos y entre ellos, así como a personalizar el esquema por inquilino.

A continuación, conozca los conceptos de hiperescala.
> [!div class="nextstepaction"]
> [Tipos de nodo de hiperescala](https://aka.ms/hyperscale-concepts)
