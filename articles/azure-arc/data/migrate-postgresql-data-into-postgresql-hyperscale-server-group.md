---
title: Migración de datos de una base de datos de PostgreSQL a un grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc
titleSuffix: Azure Arc enabled database services
description: Migración de datos de una base de datos de PostgreSQL a un grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 521fd61f18d6673e21c23dbca4cfc12d2ee4bf0b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932330"
---
# <a name="migrate-postgresql-database-to-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Migración de una base de datos de PostgreSQL a un grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc

En este documento se describen los pasos para trasladar la base de datos de PostgreSQL existente (una que no se hospede en Data Services habilitado para Azure Arc) al grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="considerations"></a>Consideraciones

El grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc es la versión de la comunidad de PostgreSQL y se ejecuta con la extensión CitusData habilitada. Por tanto, cualquier herramienta que funcione en PostgreSQL fuera de Azure Arc debería funcionar con el grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc.


Por ese motivo, con el conjunto de herramientas que usa actualmente para postgres, debe ser capaz de:
1. Realizar una copia de seguridad de la base de datos de Postgres desde la instancia hospedada fuera de Azure Arc
2. Restaurarla en el grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc

Lo que queda por hacer es lo siguiente:
- restablecer los parámetros del servidor
- restablecer los contextos de seguridad: volver a crear usuarios, roles y restablecer permisos...

Para realizar esta operación de copia de seguridad y restauración, puede usar cualquier herramienta capaz de realizar la copia de seguridad o restauración para Postgres. Por ejemplo:
- Azure Data Studio y su extensión Postgres
- `pgcli`
- `pgAdmin`
- `pg_dump`
- `pg_restore`
- `psql`
- ...

## <a name="example"></a>Ejemplo
Ahora se ilustrarán estos pasos con la herramienta `pgAdmin`.
Fíjese en la configuración siguiente:
- **Origen:**  
    un servidor Postgres que se ejecuta de forma local en un servidor sin sistema operativo y con el nombre JEANYDSRV. Es de la versión 12 y hospeda una base de datos denominada MyOnPremPostgresDB que tiene una tabla T1 con una fila :::image type="content" source="media/postgres-hyperscale/migrate-pg-source.jpg" alt-text="Migrate-source":::

- **Destino:**  
    un servidor Postgres que se ejecuta en un entorno de Azure Arc con el nombre postgres01. Es de la versión 12. No tiene ninguna base de datos excepto la estándar de Postgres.  
    :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination.jpg" alt-text="Migrate-destination":::


### <a name="take-a-backup-of-the-source-database-on-premises"></a>Realice una copia de seguridad de la base de datos de origen en el entorno local.

:::image type="content" source="media/postgres-hyperscale/Migrate-PG-Source-Backup.jpg" alt-text="Migrate-source-backup":::

Configúrela:
1. Asígnele un nombre de archivo: **MySourceBackup**
2. Establezca el formato en **Personalizado**
:::image type="content" source="media/postgres-hyperscale/Migrate-PG-Source-Backup2.jpg" alt-text="Migrate-source-backup-configure":::

La copia de seguridad se completa correctamente:  
:::image type="content" source="media/postgres-hyperscale/Migrate-PG-Source-Backup3.jpg" alt-text="Migrate-source-backup-completed":::

### <a name="create-an-empty-database-on-the-destination-system-in-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Cree una base de datos vacía en el sistema de destino en el grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc.

> [!NOTE]
> Para registrar una instancia de Postgres en la herramienta `pgAdmin`, debe usar la dirección IP pública de la instancia en el clúster de Kubernetes y establecer el puerto y el contexto de seguridad correctamente. Encontrará estos detalles en la línea del punto de conexión `psql` después de ejecutar el comando siguiente:

```console
azdata arc postgres endpoint list -n postgres01
```
Devuelve un resultado como el siguiente:
```console
[
  {
    "Description": "PostgreSQL Instance",
    "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
  },
  {
    "Description": "Log Search Dashboard",
    "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
  },
  {
    "Description": "Metrics Dashboard",
    "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
  }
]
```

Se le asignará el nombre **RESTORED_MyOnPremPostgresDB** a la base de datos de destino.  
:::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbcreate.jpg" alt-text="Migrate-destination-db-create"lightbox="media/postgres-hyperscale/migrate-pg-destination-dbcreate.jpg":::

### <a name="restore-the-database-in-your-arc-setup"></a>Restaure la base de datos en la instalación de Arc.
:::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestore.jpg" alt-text="Migratre-db-restore":::

Configure la restauración:
1. Seleccione el archivo que contiene la copia de seguridad que quiera restaurar: **MySourceBackup**
2. Mantenga el formato establecido en **Personalizado o tar**
   :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestore2.jpg" alt-text="Migrate-db-restore-configure":::

3. Haga clic en **Restaurar**.  

   La restauración se realiza correctamente.  
   :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestore3.jpg" alt-text="Migrate-db-restore-completed":::

### <a name="verify-that-the-database-was-successfully-restored-in-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Compruebe que la base de datos se ha restaurado correctamente en el grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc.

Use uno de los métodos siguientes:

**Desde `pgAdmin`:**  

expanda la instancia de Postgres hospedada en la instalación de Azure Arc. Verá la tabla en la base de datos que ha restaurado y, al seleccionar los datos, se muestra la misma fila que tiene en la instancia local:

   :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestoreverif.jpg" alt-text="Migrate-db-restore-verification":::

**Desde `psql` dentro de la instalación de Azure Arc:**  

Dentro de la instalación de Arc puede usar `psql` para conectarse a la instancia de Postgres, establecer el contexto de base de datos en `RESTORED_MyOnPremPostgresDB` y consultar los datos:

1. Enumere los puntos de conexión de ayuda de la cadena de conexión `psql`:

   ```console
   azdata arc postgres endpoint list -n postgres01
   [
     {
       "Description": "PostgreSQL Instance",
       "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
     },
     {
       "Description": "Log Search Dashboard",
       "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
     },
     {
       "Description": "Metrics Dashboard",
       "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
     }
   ]
   ```

1. Desde la cadena de conexión `psql`, use el parámetro `-d` para indicar el nombre de la base de datos. Con el comando siguiente, se le pedirá la contraseña:

   ```console
   psql -d RESTORED_MyOnPremPostgresDB -U postgres -h 10.0.0.4 -p 32639
   ```

   Se conecta `psql`.

   ```output
   Password for user postgres:
   psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1), server 12.3 (Debian 12.3-1.pgdg100+1))
   WARNING: psql major version 10, server major version 12.
         Some psql features might not work.
   SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
   Type "help" for help.

   RESTORED_MyOnPremPostgresDB=#   
   ```

1. Seleccione la tabla y verá los datos que ha restaurado desde la instancia de Postgres local:

   ```console
   RESTORED_MyOnPremPostgresDB=# select * from t1;
   ```

   ```output
    col1 |    col2
   ------+-------------
       1 | BobbyIsADog
   (1 row)
   ```

> [!NOTE]
> - No verá muchas ventajas en el rendimiento de la ejecución en Hiperescala de PostgreSQL habilitada para Azure Arc hasta que escale horizontalmente y particione o distribuya los datos entre los nodos de trabajo del grupo de servidores Hiperescala de PostgreSQL. Vea [Pasos siguientes](#next-steps).
>
> - En la actualidad, no es posible "incorporar a Azure Arc" una instancia existente de Postgres que se ejecutaría de forma local o en cualquier otra nube. Es decir, no es posible instalar ningún tipo de "agente de Azure Arc" en la instancia de Postgres existente para que convertirla en habilitada por Azure Arc. En su lugar, debe crear una instancia nueva de Postgres y transferirle los datos. Para hacerlo, puede usar la técnica mostrada antes, o bien la herramienta ETL que prefiera.

## <a name="next-steps"></a>Pasos siguientes

- Lea los conceptos y las guías paso a paso de Hiperescala de Azure Database for PostgreSQL para distribuir los datos entre varios nodos de Hiperescala de PostgreSQL y beneficiarse de toda la eficacia de esta opción:
    * [Nodos y tablas](../../postgresql/concepts-hyperscale-nodes.md)
    * [Determinar el tipo de aplicación](../../postgresql/concepts-hyperscale-app-type.md)
    * [Elección de una columna de distribución](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Colocación de tabla](../../postgresql/concepts-hyperscale-colocation.md)
    * [Distribución y modificación de tablas](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Diseño de una base de datos multiinquilino](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Diseño de un panel de análisis en tiempo real](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

> *En estos documentos, omita las secciones **Inicio de sesión en Azure Portal** y **Creación de una instancia de Azure Database for PostgreSQL: Hiperescala (Citus)** . Implemente los pasos restantes en la implementación de Azure Arc. Esas secciones son específicas de Hiperescala (Citus) de Azure Database for PostgreSQL que se ofrece como un servicio PaaS en la nube de Azure, pero las demás partes de los documentos se aplican directamente a Hiperescala de PostgreSQL habilitada para Azure Arc.

- [Escalado del grupo de servidores Hiperescala de Azure Database for PostgreSQL](scale-out-postgresql-hyperscale-server-group.md)
