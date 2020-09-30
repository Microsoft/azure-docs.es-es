---
title: Uso de extensiones de PostgreSQL
description: Uso de extensiones de PostgreSQL
titleSuffix: Azure Arc enabled data services
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 19b2ec283619df0cc8d3c880cb2df6f53f6fb332
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932225"
---
# <a name="use-postgresql-extensions-in-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Uso de extensiones de PostgreSQL en el grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc

PostgreSQL es idóneo cuando se usa con extensiones. De hecho, un elemento clave de la propia funcionalidad de Hiperescala es la extensión `citus` proporcionada por Microsoft, que se instala de forma predeterminada y que permite a Postgres particionar datos de manera transparente entre varios nodos.


[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="list-of-extensions"></a>Lista de extensiones
Además de las extensiones en [`contrib`](https://www.postgresql.org/docs/12/contrib.html), la lista de extensiones presentes en los contenedores del grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc es la siguiente:
- `citus`, v: 9,4
- `pg_cron`, v: 1.2
- `plpgsql`, v: 1.0
- `postgis`, v: 3.0.2

Esta lista evoluciona con el tiempo y las actualizaciones se publicarán en este documento. Todavía no es posible agregar extensiones más allá de las mencionadas anteriormente.

En esta guía se mostrará un escenario en el que se usan dos de estas extensiones:
- [PostGIS](https://postgis.net/)
- [`pg_cron`](https://github.com/citusdata/pg_cron)


## <a name="manage-extensions"></a>Administración de extensiones

### <a name="enable-extensions"></a>Habilitación de extensiones
Este paso no es necesario para las extensiones que forman parte de `contrib`.
El formato general del comando para habilitar las extensiones es el siguiente:

#### <a name="enable-an-extension-at-the-creation-time-of-a-server-group"></a>Se habilita una extensión en el momento de crear un grupo de servidores:
```console
azdata arc postgres server create -n <name of your postgresql server group> --extensions <extension names>
```
#### <a name="enable-an-extension-on-an-instance-that-already-exists"></a>Se habilita una extensión en una instancia que ya existe:
```console
azdata arc postgres server edit -n <name of your postgresql server group> --extensions <extension names>
```

#### <a name="get-the-list-of-extensions-enabled"></a>Se obtiene la lista de extensiones habilitadas:
Luego, ejecute cualquiera de los comandos siguientes.

##### <a name="with-azdata"></a>Con azdata
```console
azdata arc postgres server show -n <server group name>
```
Desplácese por la salida y observe las secciones engine\extensions en las especificaciones del grupo de servidores. Por ejemplo:
```console
"engine": {
      "extensions": [
        {
          "name": "citus"
        },
        {
          "name": "pg_cron"
        }
      ]
    },
```
##### <a name="with-kubectl"></a>Con kubectl
```console
kubectl describe postgresql-12s/postgres02
```
Desplácese por la salida y observe las secciones engine\extensions en las especificaciones del grupo de servidores. Por ejemplo:
```console
Engine:
    Extensions:
      Name:  citus
      Name:  pg_cron
```


### <a name="create-extensions"></a>Creación de extensiones:
Conéctese al grupo de servidores con la herramienta cliente que prefiera y ejecute la consulta estándar de PostgreSQL:
```console
CREATE EXTENSION <extension name>;
```

### <a name="get-the-list-of-extension-created-in-your-server-group"></a>Obtenga la lista de extensiones creadas en el grupo de servidores:
Conéctese al grupo de servidores con la herramienta cliente que prefiera y ejecute la consulta estándar de PostgreSQL:
```console
select * from pg_extension;
```

### <a name="drop-an-extension-from-your-server-group"></a>Quite una extensión del grupo de servidores:
Conéctese al grupo de servidores con la herramienta cliente que prefiera y ejecute la consulta estándar de PostgreSQL:
```console
drop extension <extension name>;
```

## <a name="use-the-postgis-and-the-pg_cron-extensions"></a>Uso de las extensiones PostGIS y Pg_cron

### <a name="the-postgis-extension"></a>La extensión PostGIS

Se puede habilitar la extensión PostGIS en un grupo de servidores existente, o bien crear uno con la extensión ya habilitada:

**Habilitación de una extensión en el momento de crear un grupo de servidores:**
```console
azdata arc postgres server create -n <name of your postgresql server group> --extensions <extension names>

#Example:
azdata arc postgres server create -n pg2 -w 2 --extensions postgis
```

**Habilitación de una extensión en una instancia que ya existe:**
```console
azdata arc postgres server edit -n <name of your postgresql server group> --extensions <extension names>

#Example:
azdata arc postgres server edit --extensions postgis -n pg2
```

Para comprobar qué extensiones están instaladas, use el siguiente comando estándar de PostgreSQL después de conectarse a la instancia con la herramienta cliente de PostgreSQL que prefiera, como Azure Data Studio:
```console
select * from pg_extension;
```

En primer lugar, para obtener un ejemplo de PostGIS, obtenga [datos de ejemplo](http://duspviz.mit.edu/tutorials/intro-postgis/) del Department of Urban Studies & Planning (Departamento de estudios urbanos y planificación) del MIT. Es posible que tenga que ejecutar `apt-get install unzip` para instalar unzip al usar la máquina virtual para las pruebas.

```console
wget http://duspviz.mit.edu/_assets/data/intro-postgis-datasets.zip
unzip intro-postgis-datasets.zip
```

Ahora se conectará a la base de datos y creará la extensión PostGIS:

```console
CREATE EXTENSION postgis;
```

> [!NOTE]
> Si quiere usar una de las extensiones del paquete `postgis` (por ejemplo `postgis_raster`, `postgis_topology`, `postgis_sfcgal`, `fuzzystrmatch`...), primero debe crear la extensión postgis y después la otra. Por ejemplo: `CREATE EXTENSION postgis`; `CREATE EXTENSION postgis_raster`;

Y cree el esquema:

```sql
CREATE TABLE coffee_shops (
  id serial NOT NULL,
  name character varying(50),
  address character varying(50),
  city character varying(50),
  state character varying(50),
  zip character varying(10),
  lat numeric,
  lon numeric,
  geom geometry(POINT,4326)
);
CREATE INDEX coffee_shops_gist ON coffee_shops USING gist (geom);
```

Ahora, se puede combinar PostGIS con la funcionalidad de escalado horizontal mediante la distribución de la tabla coffee_shops:

```sql
SELECT create_distributed_table('coffee_shops', 'id');
```

Se cargarán algunos datos:

```console
\copy coffee_shops(id,name,address,city,state,zip,lat,lon) from cambridge_coffee_shops.csv CSV HEADER;
```

Rellene el campo `geom` con la latitud y la longitud codificadas correctamente en el tipo de datos `geometry` de PostGIS:

```sql
UPDATE coffee_shops SET geom = ST_SetSRID(ST_MakePoint(lon,lat),4326);
```

Ahora se pueden enumerar las cafeterías más cercanas al MIT (77 Massachusetts Ave; 42.359055, -71.093500):

```sql
SELECT name, address FROM coffee_shops ORDER BY geom <-> ST_SetSRID(ST_MakePoint(-71.093500,42.359055),4326);
```


### <a name="the-pg_cron-extension"></a>La extensión pg_cron

Ahora se habilitará `pg_cron` en el grupo de servidores PostgreSQL, además de PostGIS:

```console
azdata postgres server update -n pg2 -ns arc --extensions postgis,pg_cron
```

Tenga en cuenta que esto reiniciará los nodos e instalará las extensiones adicionales, lo que puede tardar de 2 a 3 minutos.

Ahora se puede volver a conectar y crear la extensión `pg_cron`:

```sql
CREATE EXTENSION pg_cron;
```

Con fines de prueba, cree una tabla `the_best_coffee_shop` que tome un nombre aleatorio de la tabla `coffee_shops` anterior y establezca el contenido de la tabla:

```sql
CREATE TABLE the_best_coffee_shop(name text);
```

Se puede usar `cron.schedule` más algunas instrucciones SQL para obtener un nombre de tabla aleatorio (observe el uso de una tabla temporal para almacenar el resultado de una consulta distribuida) y almacenarla en `the_best_coffee_shop`:

```sql
SELECT cron.schedule('* * * * *', $$
  TRUNCATE the_best_coffee_shop;
  CREATE TEMPORARY TABLE tmp AS SELECT name FROM coffee_shops ORDER BY random() LIMIT 1;
  INSERT INTO the_best_coffee_shop SELECT * FROM tmp;
  DROP TABLE tmp;
$$);
```

Y ahora, una vez por minuto, se obtiene un nombre diferente:

```sql
SELECT * FROM the_best_coffee_shop;
```

```console
      name
-----------------
 B & B Snack Bar
(1 row)
```

Vea el archivo [LÉAME de pg_cron](https://github.com/citusdata/pg_cron) para obtener información completa sobre la sintaxis.

>[!NOTE]
>No se admite la eliminación de la extensión `citus`. La extensión `citus` es necesaria para proporcionar la experiencia de Hiperescala.

