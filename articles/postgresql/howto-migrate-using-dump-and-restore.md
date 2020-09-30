---
title: 'Volcado y restauración en Azure Database for PostgreSQL: servidor único'
description: Describe cómo extraer una base de datos de PostgreSQL en un archivo de volcado y cómo restaurar desde un archivo creado por pg_dump en Azure Database for PostgreSQL con un único servidor.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 529573bd18dbdbd16a795619d488beedfb532b11
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90902673"
---
# <a name="migrate-your-postgresql-database-using-dump-and-restore"></a>Migración de una base de datos de PostgreSQL mediante volcado y restauración
[!INCLUDE[applies-to-postgres-single-flexible-server](includes/applies-to-postgres-single-flexible-server.md)]

Puede usar [pg_dump](https://www.postgresql.org/docs/current/static/app-pgdump.html) para extraer una base de datos de PostgreSQL a un archivo de volcado y [pg_restore](https://www.postgresql.org/docs/current/static/app-pgrestore.html) para restaurar la base de datos de PostgreSQL desde un archivo de almacenamiento creado por pg_dump.

## <a name="prerequisites"></a>Prerrequisitos
Para seguir esta guía, necesitará:
- Un [servidor de Azure Database for PostgreSQL](quickstart-create-server-database-portal.md) con reglas de firewall para permitir el acceso a las bases de datos que hay en él.
- Utilidades de línea de comandos [pg_dump](https://www.postgresql.org/docs/current/static/app-pgdump.html) y [pg_restore](https://www.postgresql.org/docs/current/static/app-pgrestore.html) instaladas

Siga estos pasos para realizar un volcado y restaurar la base de datos de PostgreSQL:

## <a name="create-a-dump-file-using-pg_dump-that-contains-the-data-to-be-loaded"></a>Creación de un archivo de volcado mediante pg_dump con los datos que se van a cargar
Para hacer una copia de seguridad de una base de datos de PostgreSQL existente en el entorno local o en una máquina virtual, ejecute el comando siguiente:
```bash
pg_dump -Fc -v --host=<host> --username=<name> --dbname=<database name> -f <database>.dump
```
Por ejemplo, si tiene un servidor local y una base de datos llamada **testdb** en él:
```bash
pg_dump -Fc -v --host=localhost --username=masterlogin --dbname=testdb -f testdb.dump
```


## <a name="restore-the-data-into-the-target-azure-database-for-postgresql-using-pg_restore"></a>Restauración de los datos en la instancia de Azure Database for PostgreSQL de destino con pg_restore
Después de crear la base de datos de destino, puede usar el comando pg_restore y el parámetro -d, --dbname para restaurar los datos en la base de datos de destino desde el archivo de volcado.
```bash
pg_restore -v --no-owner --host=<server name> --port=<port> --username=<user-name> --dbname=<target database name> <database>.dump
```

Al incluir el parámetro --no-owner, todos los objetos creados durante la restauración pasan a ser propiedad del usuario especificado con --username. Para más información, consulte la documentación oficial de PostgreSQL en [pg_restore](https://www.postgresql.org/docs/9.6/static/app-pgrestore.html).

> [!NOTE]
> Si el servidor de PostgreSQL requiere conexiones TLS/SSL (que lo sucede de forma predeterminada en los servidores de Azure Database for PostgreSQL), establezca una variable de entorno `PGSSLMODE=require` para que la herramienta pg_restore se conecte con TLS. Sin TSL, el error podría ser `FATAL:  SSL connection is required. Please specify SSL options and retry.`
>
> En la línea de comandos de Windows, ejecute el comando `SET PGSSLMODE=require` antes de ejecutar el comando pg_restore. En Linux o Bash, ejecute el comando `export PGSSLMODE=require` antes de ejecutar el comando pg_restore.
>

En este ejemplo, restaure los datos del archivo de volcado **testdb.dump** en la base de datos **mypgsqldb** en el servidor de destino **mydemoserver.postgres.database.azure.com**.

Este es un ejemplo de cómo usar este elemento **pg_restore** para el **Servidor único**:

```bash
pg_restore -v --no-owner --host=mydemoserver.postgres.database.azure.com --port=5432 --username=mylogin@mydemoserver --dbname=mypgsqldb testdb.dump
```
Este es un ejemplo de cómo usar este elemento **pg_restore** para el **Servidor flexible**:

```bash
pg_restore -v --no-owner --host=mydemoserver.postgres.database.azure.com --port=5432 --username=mylogin --dbname=mypgsqldb testdb.dump
```
---

## <a name="optimizing-the-migration-process"></a>Optimización del proceso de migración

Una manera de migrar la base de datos de PostgreSQL existente al servicio Azure Database for PostgreSQL es realizar una copia de seguridad de la base de datos en el origen y restaurarla en Azure. Para minimizar el tiempo necesario para completar la migración, considere el uso de los parámetros siguientes con los comandos de copia de seguridad y restauración.

> [!NOTE]
> Para obtener información de sintaxis detallada, vea los artículos [pg_dump](https://www.postgresql.org/docs/current/static/app-pgdump.html) y [pg_restore](https://www.postgresql.org/docs/current/static/app-pgrestore.html).
>

### <a name="for-the-backup"></a>Para la copia de seguridad
- Realice la copia de seguridad con el modificador -Fc para poder realizar la restauración en paralelo a fin de acelerar. Por ejemplo:

    ```bash
    pg_dump -h my-source-server-name -U source-server-username -Fc -d source-databasename -f Z:\Data\Backups\my-database-backup.dump
    ```

### <a name="for-the-restore"></a>Para la restauración
- Se sugiere mover el archivo de copia de seguridad a una máquina virtual de Azure de la misma región que el servidor de Azure Database for PostgreSQL al que se está migrando y ejecutar pg_restore desde esa máquina virtual para reducir la latencia de red. También se recomienda crear la máquina virtual con [redes aceleradas](../virtual-network/create-vm-accelerated-networking-powershell.md) habilitadas.

- Ya debería estar así de forma predeterminada, pero abra el archivo de volcado para comprobar que las instrucciones de creación de índice están después de la inserción de los datos. Si no es así, mueva las instrucciones de creación de índice una vez insertados los datos.

- Restaure con los modificadores -Fc y -j *#* a fin de ejecutar la restauración en paralelo. *#* es el número de núcleos en el servidor de destino. También puede probar con *#* establecido en dos veces el número de núcleos del servidor de destino para ver el impacto. Por ejemplo:

Este es un ejemplo de cómo usar este elemento **pg_restore** para el **Servidor único**:
```bash
 pg_restore -h my-target-server.postgres.database.azure.com -U azure-postgres-username@my-target-server -Fc -j 4 -d my-target-databasename Z:\Data\Backups\my-database-backup.dump
```
Este es un ejemplo de cómo usar este elemento **pg_restore** para el **Servidor flexible**:
```bash
 pg_restore -h my-target-server.postgres.database.azure.com -U azure-postgres-username@my-target-server -Fc -j 4 -d my-target-databasename Z:\Data\Backups\my-database-backup.dump
 ```

- También puede editar el archivo de volcado si agrega el comando *set synchronous_commit = off;* al principio y el comando *set synchronous_commit = on;* al final. No activarlo al final, antes de que las aplicaciones cambien los datos, podría provocar la consiguiente pérdida de datos.

- En el servidor Azure Database for PostgreSQL de destino, considere la posibilidad de hacer lo siguiente antes de la restauración:
    - Desactive el seguimiento del rendimiento de las consultas, ya que estas estadísticas no son necesarios durante la migración. Para ello, puede establecer pg_stat_statements.track pg_qs.query_capture_mode y pgms_wait_sampling.query_capture_mode en NONE.

    - Use una sku de memoria alta y proceso elevado, como 32 núcleos virtuales con optimización para memoria, para acelerar la migración. Puede escalar fácilmente su sku preferida una vez completada la restauración. Cuanto mayor sea la sku, mayor paralelismo puede lograr mediante el aumento del parámetro `-j` correspondiente en el comando pg_restore.

    - Si se aumenta la cantidad de IOPS en el servidor de destino, podría mejorar el rendimiento de la restauración. Puede aprovisionar más IOPS si aumenta el tamaño de almacenamiento del servidor. Esta configuración no es reversible, pero tenga en cuenta si una IOPS mayor beneficiaría la carga de trabajo real en el futuro.

No olvide probar y validar estos comandos en un entorno de prueba antes de usarlos en producción.

## <a name="next-steps"></a>Pasos siguientes
- Para migrar una base de datos de PostgreSQL mediante exportación e importación, vea [Migración de una base de datos de PostgreSQL mediante exportación e importación](howto-migrate-using-export-and-import.md).
- Para obtener más información sobre cómo migrar bases de datos a Azure Database for PostgreSQL, vea la [Guía de migración de base de datos](https://aka.ms/datamigration).
