---
title: 'Tutorial: Diseño de un panel en tiempo real: Hiperescala (Citus): Azure Database for PostgreSQL'
description: Este tutorial muestra cómo crear, rellenar y consultar tablas distribuidas en Azure Database for PostgreSQL Hiperescala (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.topic: tutorial
ms.date: 05/14/2019
ms.openlocfilehash: db3cd95c5a833b299ee85c1e68b15644ae0e0226
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387581"
---
# <a name="tutorial-design-a-real-time-analytics-dashboard-by-using-azure-database-for-postgresql--hyperscale-citus"></a>Tutorial: Diseño de un panel de análisis en tiempo real on Azure Database for PostgreSQL: Hiperescala (Citus)

En este tutorial, usará Azure Database for PostgreSQL Hiperescala (Citus) para aprender lo siguiente:

> [!div class="checklist"]
> * Creación de un grupo de servidores Hiperescala (Citus)
> * Uso de la utilidad psql para crear un esquema
> * Particiones de tablas entre nodos
> * Generación de datos de ejemplo
> * Realización de acumulaciones
> * Consulta de datos sin procesar y agregados
> * Expiración de los datos

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [azure-postgresql-hyperscale-create-db](../../includes/azure-postgresql-hyperscale-create-db.md)]

## <a name="use-psql-utility-to-create-a-schema"></a>Uso de la utilidad psql para crear un esquema

Una vez conectado a Azure Database for PostgreSQL Hiperescala (Citus) mediante psql, puede completar algunas tareas básicas. Este tutorial lo guiará a través de la ingesta de datos de tráfico provenientes del análisis web y posterior acumulación de los datos para proporcionar paneles en tiempo real basados en esos datos.

Vamos a crear una tabla que consumirá todos los datos de tráfico web sin procesar. Ejecute los comandos siguientes en el terminal psql:

```sql
CREATE TABLE http_request (
  site_id INT,
  ingest_time TIMESTAMPTZ DEFAULT now(),

  url TEXT,
  request_country TEXT,
  ip_address TEXT,

  status_code INT,
  response_time_msec INT
);
```

También vamos a crear una tabla que contendrá los agregados por minuto y una tabla que conserva la posición de la última acumulación. Ejecute los siguientes comandos en psql:

```sql
CREATE TABLE http_request_1min (
  site_id INT,
  ingest_time TIMESTAMPTZ, -- which minute this row represents

  error_count INT,
  success_count INT,
  request_count INT,
  average_response_time_msec INT,
  CHECK (request_count = error_count + success_count),
  CHECK (ingest_time = date_trunc('minute', ingest_time))
);

CREATE INDEX http_request_1min_idx ON http_request_1min (site_id, ingest_time);

CREATE TABLE latest_rollup (
  minute timestamptz PRIMARY KEY,

  CHECK (minute = date_trunc('minute', minute))
);
```

Ahora puede ver las tablas recién creadas en la lista de tablas con este comando de psql:

```postgres
\dt
```

## <a name="shard-tables-across-nodes"></a>Particiones de tablas entre nodos

Una implementación de hiperescala almacena filas de tablas en distintos nodos en función del valor de una columna designada por el usuario. Esta "columna de distribución" marca cómo los datos se particionan entre los distintos nodos.

Establezcamos que la columna de distribución sea site\_id, la clave de partición. En psql, ejecute estas funciones:

  ```sql
SELECT create_distributed_table('http_request',      'site_id');
SELECT create_distributed_table('http_request_1min', 'site_id');
```

[!INCLUDE [azure-postgresql-hyperscale-dist-alert](../../includes/azure-postgresql-hyperscale-dist-alert.md)]

## <a name="generate-sample-data"></a>Generación de datos de ejemplo

Ahora, el grupo de servidores debería estar listo para ingerir algunos datos. Podemos ejecutar lo siguiente de manera local desde la conexión de `psql` para insertar datos continuamente.

```sql
DO $$
  BEGIN LOOP
    INSERT INTO http_request (
      site_id, ingest_time, url, request_country,
      ip_address, status_code, response_time_msec
    ) VALUES (
      trunc(random()*32), clock_timestamp(),
      concat('http://example.com/', md5(random()::text)),
      ('{China,India,USA,Indonesia}'::text[])[ceil(random()*4)],
      concat(
        trunc(random()*250 + 2), '.',
        trunc(random()*250 + 2), '.',
        trunc(random()*250 + 2), '.',
        trunc(random()*250 + 2)
      )::inet,
      ('{200,404}'::int[])[ceil(random()*2)],
      5+trunc(random()*150)
    );
    COMMIT;
    PERFORM pg_sleep(random() * 0.25);
  END LOOP;
END $$;
```

La consulta inserta aproximadamente ocho filas cada segundo. Las filas se almacenan en distintos nodos de trabajo según las indicaciones de la columna de distribución, `site_id`.

   > [!NOTE]
   > Deje que se ejecute la consulta de generación de datos y abra una segunda conexión de psql para el resto de los comandos de este tutorial.
   >

## <a name="query"></a>Consultar

La opción de hospedaje de hiperescala permite que varios nodos procesen consultas en paralelo para así acelerar el proceso. Por ejemplo, la base de datos calcula agregados como SUM y COUNT en nodos de trabajo y combina los resultados en una respuesta final.

Esta es una consulta para contar las solicitudes web por minuto junto con algunas estadísticas.
Intente ejecutarla en psql y observe los resultados.

```sql
SELECT
  site_id,
  date_trunc('minute', ingest_time) as minute,
  COUNT(1) AS request_count,
  SUM(CASE WHEN (status_code between 200 and 299) THEN 1 ELSE 0 END) as success_count,
  SUM(CASE WHEN (status_code between 200 and 299) THEN 0 ELSE 1 END) as error_count,
  SUM(response_time_msec) / COUNT(1) AS average_response_time_msec
FROM http_request
WHERE date_trunc('minute', ingest_time) > now() - '5 minutes'::interval
GROUP BY site_id, minute
ORDER BY minute ASC;
```

## <a name="rolling-up-data"></a>Acumulación de datos

La consulta anterior funciona bien en las primeras etapas, pero su rendimiento se degrada a medida que aumentan los datos. Incluso con el procesamiento distribuido, resulta más rápido calcular previamente estos datos que volver a calcularlos varias veces.

Es posible garantizar que el panel siga siendo rápido si acumulamos de manera frecuente los datos sin procesar en una tabla de agregado. Puede experimentar con la duración de la agregación. Hemos utilizado una tabla de agregación por minuto, pero los datos se pueden dividir en 5, 15 o 60 minutos en su lugar.

Para ejecutar esta acumulación más fácilmente, vamos a ponerlo en una función plpgsql. Ejecute estos comandos en psql para crear la función `rollup_http_request`.

```sql
-- initialize to a time long ago
INSERT INTO latest_rollup VALUES ('10-10-1901');

-- function to do the rollup
CREATE OR REPLACE FUNCTION rollup_http_request() RETURNS void AS $$
DECLARE
  curr_rollup_time timestamptz := date_trunc('minute', now());
  last_rollup_time timestamptz := minute from latest_rollup;
BEGIN
  INSERT INTO http_request_1min (
    site_id, ingest_time, request_count,
    success_count, error_count, average_response_time_msec
  ) SELECT
    site_id,
    date_trunc('minute', ingest_time),
    COUNT(1) as request_count,
    SUM(CASE WHEN (status_code between 200 and 299) THEN 1 ELSE 0 END) as success_count,
    SUM(CASE WHEN (status_code between 200 and 299) THEN 0 ELSE 1 END) as error_count,
    SUM(response_time_msec) / COUNT(1) AS average_response_time_msec
  FROM http_request
  -- roll up only data new since last_rollup_time
  WHERE date_trunc('minute', ingest_time) <@
          tstzrange(last_rollup_time, curr_rollup_time, '(]')
  GROUP BY 1, 2;

  -- update the value in latest_rollup so that next time we run the
  -- rollup it will operate on data newer than curr_rollup_time
  UPDATE latest_rollup SET minute = curr_rollup_time;
END;
$$ LANGUAGE plpgsql;
```

Una vez agregada la función, ejecútela para acumular los datos:

```sql
SELECT rollup_http_request();
```

Y, con los datos en un formulario previamente agregado, podemos consultar la tabla de acumulación para obtener el mismo informe que antes. Ejecute la siguiente consulta:

```sql
SELECT site_id, ingest_time as minute, request_count,
       success_count, error_count, average_response_time_msec
  FROM http_request_1min
 WHERE ingest_time > date_trunc('minute', now()) - '5 minutes'::interval;
 ```

## <a name="expiring-old-data"></a>Expiración de datos antiguos

Las acumulaciones agilizan las consultas, pero de todos modos es necesario que los datos antiguos expiren para evitar costos de almacenamiento ilimitados. Debe decidir durante cuánto tiempo quiere conservar los datos para cada granularidad y usar consultas estándar para eliminar los datos expirados. En el ejemplo siguiente, decidimos conservar los datos sin procesar durante un día y las agregaciones por minuto, durante un mes:

```sql
DELETE FROM http_request WHERE ingest_time < now() - interval '1 day';
DELETE FROM http_request_1min WHERE ingest_time < now() - interval '1 month';
```

En el entorno de producción, podría encapsular estas consultas en una función y llamarla cada minuto en un trabajo de cron.

## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, creó recursos de Azure en un grupo de servidores. Si no cree que vaya a necesitar estos recursos en el futuro, elimine el grupo de servidores. Presione el botón *Eliminar* en la página *Información general* del grupo de servidores. Cuando aparezca una página emergente en la que se le pida hacerlo, confirme el nombre del grupo de servidores y haga clic en el botón *Eliminar* final.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a aprovisionar un grupo de servidores Hiperescala (Citus). Se conectó a él con psql, creó un esquema y distribuyó datos. Ha aprendido a consultar datos en formato sin procesar, a agregar esos datos de manera habitual, a consultar las tablas de agregados y a hacer expirar los datos antiguos.

A continuación, conozca los conceptos de hiperescala.
> [!div class="nextstepaction"]
> [Tipos de nodo de hiperescala](https://aka.ms/hyperscale-concepts)
