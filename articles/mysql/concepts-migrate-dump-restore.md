---
title: Migración mediante volcado y restauración de Azure Database for MySQL
description: En este artículo se explican dos maneras comunes de realizar una copia de seguridad de las bases de datos y restaurarlas en Azure Database for MySQL, con herramientas como mysqldump, MySQL Workbench y PHPMyAdmin.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 2/27/2020
ms.openlocfilehash: a0171481b97cff2ea085a80b387bff13590529a5
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905900"
---
# <a name="migrate-your-mysql-database-to-azure-database-for-mysql-using-dump-and-restore"></a>Migre su Base de datos MySQL a Azure Database for MySQL mediante el volcado y la restauración.

[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]

En este artículo se explican dos formas habituales de hacer una copia de seguridad y restaurar bases de datos en Azure Database for MySQL.
- Volcado y restauración desde la línea de comandos (mediante mysqldump)
- Volcado y restauración mediante PHPMyAdmin

## <a name="before-you-begin"></a>Antes de empezar
Para seguir esta guía de procedimientos, necesita lo siguiente:
- Conocer la [creación de una base de datos de Azure para el servidor MySQL: Azure Portal](quickstart-create-mysql-server-database-using-azure-portal.md)
- Utilidad de línea de comandos [mysqldump](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html) instalada en la máquina.
- [MySQL Workbench](https://dev.mysql.com/downloads/workbench/) u otra herramienta de MySQL de terceros para ejecutar los comandos de volcado y restauración.

> [!TIP]
> Si desea migrar bases de datos de gran tamaño con tamaños de base de datos superiores a 1 TB, considere la posibilidad de usar herramientas de la comunidad como **mydumper/myloader**, que admite la importación y exportación paralelas. Aprenda [cómo migrar bases de datos de MySQL grandes](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/best-practices-for-migrating-large-databases-to-azure-database/ba-p/1362699).

## <a name="common-use-cases-for-dump-and-restore"></a>Casos de uso comunes de volcado y restauración
Puede emplear utilidades de MySQL, como **mysqldump** y **mysqlpump** para realizar un volcado de las bases de datos y cargarlas en una base de datos de Azure MySQL en varios escenarios comunes. En otros escenarios, puede usar en su lugar el enfoque de [importación y exportación](concepts-migrate-import-export.md).

- **Use volcados de base de datos cuando vaya a migrar toda la base de datos**. Esta recomendación es adecuada si va a mover una gran cantidad de datos de MySQL o si quiere reducir la interrupción del servicio en sitios o aplicaciones dinámicos.
-  **Use el volcado de base de datos si todas las tablas de la base de datos usan el motor de almacenamiento InnoDB**. Azure Database for MySQL solo admite el motor de almacenamiento InnoDB y, por tanto, no es compatible con otros alternativos. Si las tablas están configuradas con otros motores de almacenamiento, conviértalos al formato del motor InnoDB antes de realizar la migración a Azure Database for MySQL.

    Por ejemplo, si tiene una aplicación WordPress o WebApp que usa las tablas MyISAM, convierta primero esas tablas; para ello, mígrelas al formato InnoDB antes de restaurarlas en Azure Database for MySQL. Use la cláusula `ENGINE=InnoDB` para configurar el motor utilizado al crear una nueva tabla y luego transfiera los datos a la tabla compatible antes de la restauración.

   ```sql
   INSERT INTO innodb_table SELECT * FROM myisam_table ORDER BY primary_key_columns
   ```
> [!Important]
> - Para evitar problemas de compatibilidad, asegúrese de usar la misma versión de MySQL en los sistemas de origen y de destino al realizar el volcado de las bases de datos. Por ejemplo, si el servidor MySQL existente tiene la versión 5.7, debe migrar a Azure Database for MySQL configurado para ejecutar esta versión. El comando `mysql_upgrade` no funciona en un servidor de Azure Database for MySQL y no se admite. 
> - Si tiene que actualizar entre versiones de MySQL, primero vuelque o exporte la base de datos con una versión menor a una versión superior de MySQL en su propio entorno. A continuación, ejecute `mysql_upgrade` antes de intentar la migración a una instancia de Azure Database for MySQL.

## <a name="performance-considerations"></a>Consideraciones de rendimiento
Para optimizar el rendimiento, tenga en cuenta estas consideraciones al volcar grandes bases de datos:
-   Use la opción `exclude-triggers` en mysqldump al volcar las bases de datos. Excluya los desencadenadores de los archivos de volcado para evitar que los comandos de desencadenamiento se disparen durante la restauración de datos.
-   Use la opción `single-transaction` para establecer el modo de aislamiento de transacción a REPEATABLE READ y enviar una instrucción SQL START TRANSACTION al servidor antes de volcar datos. El volcado de muchas tablas en una única transacción provoca el consumo de almacenamiento adicional durante la restauración. Las opciones `single-transaction` y `lock-tables` son mutuamente excluyentes porque LOCK TABLES hace que las transacciones pendientes se confirmen implícitamente. Para volcar las tablas grandes, combine la opción `single-transaction` con la opción `quick`.
-   Use la sintaxis de varias filas `extended-insert` que incluye varias listas VALUE. Esto da como resultado un archivo de volcado de memoria más pequeño y acelera las inserciones cuando se vuelve a cargar el archivo.
-  Use la opción `order-by-primary` de mysqldump al volcar las bases de datos, para que el script de los datos se genere en el orden de la clave principal.
-   Use la opción `disable-keys` de mysqldump al volcar los datos para deshabilitar las restricciones de clave externa antes de la carga. El hecho de deshabilitar las comprobaciones de clave externa favorece un aumento del rendimiento. Habilite las restricciones y compruebe los datos después de la carga para garantizar la integridad referencial.
-   Use tablas con particiones cuando sea necesario.
-   Cargue los datos en paralelo. Evite demasiado paralelismo que podría provocar que se alcanzara un límite de recursos, y supervise los recursos mediante las métricas disponibles en Azure Portal.
-   Use la opción `defer-table-indexes` de mysqlpump al volcar las bases de datos, para que la creación de índices tenga lugar una vez cargados los datos de las tablas.
-   Use la opción `skip-definer` de mysqlpump para omitir las cláusulas DEFINER y SQL SECURITY de las instrucciones CREATE para las vistas y los procedimientos almacenados.  Al volver a cargar el archivo de volcado de memoria, se crean objetos que usan los valores DEFINER y SQL SECURITY predeterminados.
-   Copie los archivos de copia de seguridad en un blob o almacén de Azure y realice la restauración desde allí, lo que debería ser mucho más rápido que realizar la restauración a través de Internet.

## <a name="create-a-database-on-the-target-azure-database-for-mysql-server"></a>Creación de una base de datos en el servidor de destino de Azure Database for MySQL
Cree una base de datos vacía en el servidor de destino de Azure Database for MySQL donde se van a migrar los datos. Use una herramienta como MySQL Workbench o mysql.exe para crear la base de datos. La base de datos puede tener el mismo nombre que la base de datos que contiene los datos volcados, o puede crear una base de datos con un nombre diferente.

Para conectarse, busque la información de conexión en la página **Introducción** de su instancia de Azure Database for MySQL.

:::image type="content" source="./media/concepts-migrate-dump-restore/1_server-overview-name-login.png" alt-text="Obtención de la información de conexión en Azure Portal":::

Agregue la información de conexión a MySQL Workbench.

:::image type="content" source="./media/concepts-migrate-dump-restore/2_setup-new-connection.png" alt-text="Cadena de conexión de MySQL Workbench":::

## <a name="preparing-the-target-azure-database-for-mysql-server-for-fast-data-loads"></a>Preparación del servidor de Azure Database for MySQL de destino para cargas de datos rápidas
Para preparar el servidor de Azure Database for MySQL de destino para cargas de datos más rápidas, es necesario cambiar los siguientes parámetros y configuración del servidor.
- max_allowed_packet: establézcalo en 1073741824 (es decir, 1 GB) para evitar cualquier problema de desbordamiento debido a filas largas.
- slow_query_log: establézcalo en OFF para desactivar el registro de consultas lentas. Esto eliminará la sobrecarga causada por un registro de consultas lento durante las cargas de datos.
- query_store_capture_mode: establézcalo en NONE para desactivar el Almacén de consultas. Esto eliminará la sobrecarga causada por las actividades de muestreo en el Almacén de consultas.
- innodb_buffer_pool_size: escale verticalmente el servidor a 32 núcleo virtual de SKU con optimización de memoria desde el plan de tarifa del portal durante la migración para aumentar el innodb_buffer_pool_size. Innodb_buffer_pool_size solo se puede aumentar escalando el proceso para Azure Database for MySQL Server.
- innodb_io_capacity e innodb_io_capacity_max: cambie a 9000 de los parámetros del servidor en Azure Portal para mejorar el uso de la E/S a fin de optimizar la velocidad de la migración.
- innodb_write_io_threads e innodb_write_io_threads: cambie a 4 desde los parámetros del servidor en Azure Portal para mejorar la velocidad de la migración.
- Escalar verticalmente la capa de almacenamiento: las IOPs para el servidor de Azure Database for MySQL aumentan progresivamente con el aumento de la capa de almacenamiento. Para agilizar las cargas, puede aumentar la capa de almacenamiento para aumentar la IOPs aprovisionada. Recuerde que el almacenamiento solo se puede escalar verticalmente, no reducir.

Una vez completada la migración, puede revertir los parámetros del servidor y la configuración del nivel de proceso a sus valores anteriores.

## <a name="dump-and-restore-using-mysqldump-utility"></a>Volcado y restauración mediante la utilidad mysqldump

### <a name="create-a-backup-file-from-the-command-line-using-mysqldump"></a>Creación de un archivo de copia de seguridad a partir de la línea de comandos mediante mysqldump
Para hacer copia de seguridad de una base de datos MySQL existente en el servidor local en el entorno local o en una máquina virtual, ejecute el siguiente comando:
```bash
$ mysqldump --opt -u [uname] -p[pass] [dbname] > [backupfile.sql]
```

Los parámetros que se proporcionan son los siguientes:
- [uname] El nombre de usuario de base de datos
- [pass] La contraseña de la base de datos (observe que no hay ningún espacio entre -p y la contraseña)
- [dbname] El nombre de la base de datos
- [backupfile.sql] El nombre de archivo para la copia de seguridad de la base de datos
- [--opt] La opción mysqldump

Por ejemplo, para hacer una copia de seguridad de una base de datos llamada "testdb" en el servidor MySQL con el nombre de usuario "testuser" y sin contraseña en archivo untestdb_backup.sql, use el siguiente comando. El comando realiza una copia de la base de datos `testdb` en un archivo denominado `testdb_backup.sql`, que contiene todas las instrucciones SQL necesarias para volver a crear la base de datos. Asegúrese de que el nombre de usuario "testuser" tenga al menos el privilegio SELECT para las tablas volcadas, SHOW VIEW para las vistas volcadas, TRIGGER para los desencadenadores volcados y LOCK TABLES si no se usa la opción de una transacción.

```bash
GRANT SELECT, LOCK TABLES, SHOW VIEW ON *.* TO 'testuser'@'hostname' IDENTIFIED BY 'password';
```
Ahora, ejecute mysqldump para crear la copia de seguridad de la base de datos `testdb`

```bash
$ mysqldump -u root -p testdb > testdb_backup.sql
```
Si quiere incluir en la copia de seguridad solo algunas de las tablas, ordene los nombres de tabla en una lista separados por espacios y seleccione los que desee. Por ejemplo, para realizar una copia de seguridad solo de las tablas table1 y table2 de "testdb", siga este ejemplo:

```bash
$ mysqldump -u root -p testdb table1 table2 > testdb_tables_backup.sql
```
Para hacer una copia de seguridad de más de una base de datos a la vez, use el conmutador --database y ordene los nombres de las bases de datos en una lista separados por espacios.
```bash
$ mysqldump -u root -p --databases testdb1 testdb3 testdb5 > testdb135_backup.sql
```

### <a name="restore-your-mysql-database-using-command-line-or-mysql-workbench"></a>Restauración de la base de datos MySQL mediante la línea de comandos o MySQL Workbench
Una vez que haya creado la base de datos de destino, puede usar el comando mysql o MySQL Workbench para restaurar los datos en la base de datos específica recién creada desde el archivo de volcado.
```bash
mysql -h [hostname] -u [uname] -p[pass] [db_to_restore] < [backupfile.sql]
```
En este ejemplo, restaure los datos en la base de datos recién creada en el servidor de destino de Azure Database for MySQL server.

Este es un ejemplo de cómo usar este elemento **mysql** para un **servidor único**:

```bash
$ mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p testdb < testdb_backup.sql
```
Este es un ejemplo de cómo usar este elemento **mysql** para el **servidor flexible**:

```bash
$ mysql -h mydemoserver.mysql.database.azure.com -u myadmin -p testdb < testdb_backup.sql
```
---

## <a name="dump-and-restore-using-phpmyadmin"></a>Volcado y restauración mediante PHPMyAdmin
Siga estos pasos para realizar un volcado y restaurar una base de datos mediante PHPMyadmin.

> [!NOTE]
> En el caso de un servidor único, el nombre de usuario debe tener este formato, "username@servername" pero, en el de un servidor flexible, puede usar solo "username". Si usa "username@servername" para un servidor flexible, se producirá un error en la conexión.

### <a name="export-with-phpmyadmin"></a>Exportación con PHPMyadmin
Para exportar, puede usar la herramienta común phpMyAdmin que puede haber instalado ya localmente en su entorno. Para exportar la Base de datos MySQL mediante PHPMyAdmin:
1. Abra phpMyAdmin.
2. Seleccione la base de datos. Haga clic en el nombre de la base de datos en la lista de la izquierda.
3. Haga clic en el vínculo **Exportar**. Aparece una nueva página para ver el volcado de la base de datos.
4. En el área de exportación, haga clic en el vínculo **Select All** (Seleccionar todo) para seleccionar todas las tablas de la base de datos.
5. En el área de opciones de SQL, haga clic en las opciones adecuadas.
6. Haga clic en la opción **Save as file** (Guardar como archivo) y en la opción correspondiente de compresión y luego haga clic en el botón **Go** (Ir). Debería aparecer un cuadro de diálogo en el que se le pide que guarde el archivo localmente.

### <a name="import-using-phpmyadmin"></a>Importación mediante PHPMyAdmin
La importación de la base de datos es similar a la exportación. Haga lo siguiente:
1. Abra phpMyAdmin.
2. En la página de instalación de phpMyAdmin, haga clic en **Add** (Agregar) para agregar el servidor de Azure Database for MySQL. Proporcione la información de conexión e inicio de sesión.
3. Cree una base de datos con el nombre adecuado y selecciónela en la parte izquierda de la pantalla. Para volver a escribir la base de datos existente, haga clic en el nombre de la base de datos, active todas las casillas situadas al lado de los nombres de tabla y seleccione **Eliminar** para eliminar las tablas existentes.
4. Haga clic en el vínculo **SQL** para mostrar la página donde puede escribir comandos SQL, o bien cargue su archivo SQL.
5. Use el botón **Browse** (Examinar) para buscar el archivo de base de datos.
6. Haga clic en **Go** (Ir) para exportar la copia de seguridad, ejecutar los comandos SQL y volver a crear la base de datos.

## <a name="known-issues"></a>Problemas conocidos
Para obtener información sobre problemas conocidos, sugerencias y trucos, le recomendamos que consulte nuestro [blog de techcommunity](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/tips-and-tricks-in-using-mysqldump-and-mysql-restore-to-azure/ba-p/916912).

## <a name="next-steps"></a>Pasos siguientes
- [Conexión de aplicaciones a Azure Database for MySQL](./howto-connection-string.md)
- Para más información sobre cómo migrar bases de datos a Azure Database for MySQL, consulte la [Guía de migración de base de datos](https://aka.ms/datamigration).
