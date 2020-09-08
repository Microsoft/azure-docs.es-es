---
title: 'Inicio rápido: Conexión con PHP: Azure Database for PostgreSQL (servidor único)'
description: 'En este inicio rápido se proporciona un ejemplo de código de PHP que puede usar para conectarse a Azure Database for PostgreSQL: servidor único y consultar datos de ese servicio.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.custom: mvc
ms.devlang: php
ms.topic: quickstart
ms.date: 2/28/2018
ms.openlocfilehash: 000f3cd9598bc4bdffa896be47466d157c7df8c1
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88182721"
---
# <a name="quickstart-use-php-to-connect-and-query-data-in-azure-database-for-postgresql---single-server"></a>Inicio rápido: Uso de PHP para conectarse y consultar datos en Azure Database for PostgreSQL: servidor único

En este tutorial rápido se muestra cómo conectarse a una instancia de Azure Database for PostgreSQL mediante una aplicación de [PHP](https://secure.php.net/manual/intro-whatis.php). Se indica cómo usar instrucciones SQL para consultar, insertar, actualizar y eliminar datos en la base de datos. En los pasos de este artículo se da por hecho que está familiarizado con el desarrollo mediante PHP, pero que nunca ha trabajado con Azure Database for PostgreSQL.

## <a name="prerequisites"></a>Prerrequisitos
En este tutorial rápido se usan como punto de partida los recursos creados en una de estas guías:
- [Creación de una base de datos con el portal](quickstart-create-server-database-portal.md)
- [Creación de la base de datos: CLI de Azure](quickstart-create-server-database-azure-cli.md)

## <a name="install-php"></a>Instalación de PHP
Instale PHP en su propio servidor o cree una [aplicación web](../app-service/overview.md) de Azure que lo incluya.

### <a name="windows"></a>Windows
- Descargue la [versión segura 7.1.4 de PHP sin subprocesos (x64)](https://windows.php.net/download#php-7.1).
- Instale PHP y consulte el [manual de PHP](https://secure.php.net/manual/install.windows.php) para continuar con la configuración.
- El código usa la clase **pgsql** (ext/php_pgsql.dll) que se incluye en la instalación de PHP. 
- Ha habilitado la extensión **pgsql** mediante la edición del archivo de configuración php.ini, que normalmente se encuentra en `C:\Program Files\PHP\v7.1\php.ini`. El archivo de configuración debe contener una línea con el texto `extension=php_pgsql.so`. Si no se muestra, agregue el texto y guarde el archivo. Si el texto está presente, pero se ha comentado con un prefijo de punto y coma, quite el punto y coma para quitar la marca de comentario del texto.

### <a name="linux-ubuntu"></a>Linux (Ubuntu)
- Descargue la [versión segura 7.1.4 de PHP sin subprocesos (x64)](https://secure.php.net/downloads.php). 
- Instale PHP y consulte el [manual de PHP](https://secure.php.net/manual/install.unix.php) para continuar con la configuración.
- El código usa la clase **pgsql** (php_pgsql.so). Instálela mediante la ejecución de `sudo apt-get install php-pgsql`.
- Ha habilitado la extensión **pgsql** mediante la edición del archivo de configuración `/etc/php/7.0/mods-available/pgsql.ini`. El archivo de configuración debe contener una línea con el texto `extension=php_pgsql.so`. Si no se muestra, agregue el texto y guarde el archivo. Si el texto está presente, pero se ha comentado con un prefijo de punto y coma, quite el punto y coma para quitar la marca de comentario del texto.

### <a name="macos"></a>MacOS
- Descargue la [versión 7.1.4 de PHP](https://secure.php.net/downloads.php).
- Instale PHP y consulte el [manual de PHP](https://secure.php.net/manual/install.macosx.php) para continuar con la configuración.

## <a name="get-connection-information"></a>Obtención de información sobre la conexión
Obtenga la información de conexión necesaria para conectarse a Azure Database for PostgreSQL. Necesitará el nombre completo del servidor y las credenciales de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En el menú izquierdo de Azure Portal, haga clic en **Todos los recursos** y, luego, busque el servidor que ha creado, por ejemplo, **mydemoserver**.
3. Haga clic en el nombre del servidor.
4. En el panel **Información general** del servidor, anote el **nombre del servidor** y el **nombre de inicio de sesión del administrador del servidor**. Si olvida la contraseña, puede restablecerla en este panel.
 ![Nombre de servidor de Azure Database for PostgreSQL](./media/connect-php/1-connection-string.png)

## <a name="connect-and-create-a-table"></a>Conexión y creación de una tabla
Use el código siguiente para conectarse y crear una tabla mediante la instrucción SQL **CREATE TABLE**, seguida de las instrucciones SQL **INSERT INTO** para agregar filas a la tabla.

El código llama al método [pg_connect()](https://secure.php.net/manual/en/function.pg-connect.php) para conectar con Azure Database for PostgreSQL. A continuación, llama al método [pg_query()](https://secure.php.net/manual/en/function.pg-query.php) varias veces para ejecutar varios comandos y a [pg_last_error()](https://secure.php.net/manual/en/function.pg-last-error.php) para consultar los detalles si se produjo un error cada vez. A continuación, llama al método [pg_close()](https://secure.php.net/manual/en/function.pg-close.php) para cerrar la conexión.

Reemplace los parámetros `$host`, `$database`, `$user` y `$password` por sus propios valores. 

```php
<?php
    // Initialize connection variables.
    $host = "mydemoserver.postgres.database.azure.com";
    $database = "mypgsqldb";
    $user = "mylogin@mydemoserver";
    $password = "<server_admin_password>";

    // Initialize connection object.
    $connection = pg_connect("host=$host dbname=$database user=$user password=$password") 
        or die("Failed to create connection to database: ". pg_last_error(). "<br/>");
    print "Successfully created connection to database.<br/>";

    // Drop previous table of same name if one exists.
    $query = "DROP TABLE IF EXISTS inventory;";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). "<br/>");
    print "Finished dropping table (if existed).<br/>";

    // Create table.
    $query = "CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). "<br/>");
    print "Finished creating table.<br/>";

    // Insert some data into table.
    $name = '\'banana\'';
    $quantity = 150;
    $query = "INSERT INTO inventory (name, quantity) VALUES ($1, $2);";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). "<br/>");

    $name = '\'orange\'';
    $quantity = 154;
    $query = "INSERT INTO inventory (name, quantity) VALUES ($name, $quantity);";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). "<br/>");

    $name = '\'apple\'';
    $quantity = 100;
    $query = "INSERT INTO inventory (name, quantity) VALUES ($name, $quantity);";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error()). "<br/>";

    print "Inserted 3 rows of data.<br/>";

    // Closing connection
    pg_close($connection);
?>
```

## <a name="read-data"></a>Lectura de datos
Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **SELECT**. 

 El código llama al método [pg_connect()](https://secure.php.net/manual/en/function.pg-connect.php) para conectar con Azure Database for PostgreSQL. A continuación, llama al método [pg_query()](https://secure.php.net/manual/en/function.pg-query.php) para ejecutar el comando SELECT (los resultados se guardan en un conjunto de resultados) y a [pg_last_error()](https://secure.php.net/manual/en/function.pg-last-error.php) para comprobar los detalles si se ha producido un error.  Para leer el conjunto de resultados, se llama al método [pg_fetch_row()](https://secure.php.net/manual/en/function.pg-fetch-row.php) en un bucle, una vez por cada fila, y los datos de la fila se recuperan en una matriz `$row`, con un valor de datos por columna en cada posición de dicha matriz.  Para liberar el conjunto de resultados, se llama al método [pg_free_result()](https://secure.php.net/manual/en/function.pg-free-result.php). A continuación, llama al método [pg_close()](https://secure.php.net/manual/en/function.pg-close.php) para cerrar la conexión.

Reemplace los parámetros `$host`, `$database`, `$user` y `$password` por sus propios valores. 

```php
<?php
    // Initialize connection variables.
    $host = "mydemoserver.postgres.database.azure.com";
    $database = "mypgsqldb";
    $user = "mylogin@mydemoserver";
    $password = "<server_admin_password>";
    
    // Initialize connection object.
    $connection = pg_connect("host=$host dbname=$database user=$user password=$password")
                or die("Failed to create connection to database: ". pg_last_error(). "<br/>");

    print "Successfully created connection to database. <br/>";

    // Perform some SQL queries over the connection.
    $query = "SELECT * from inventory";
    $result_set = pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). "<br/>");
    while ($row = pg_fetch_row($result_set))
    {
        print "Data row = ($row[0], $row[1], $row[2]). <br/>";
    }

    // Free result_set
    pg_free_result($result_set);

    // Closing connection
    pg_close($connection);
?>
```

## <a name="update-data"></a>Actualización de datos
Use el código siguiente para conectarse y actualizar los datos mediante la instrucción SQL **UPDATE**.

El código llama al método [pg_connect()](https://secure.php.net/manual/en/function.pg-connect.php) para conectar con Azure Database for PostgreSQL. A continuación, llama al método [pg_query()](https://secure.php.net/manual/en/function.pg-query.php) para ejecutar un comando, y a [pg_last_error()](https://secure.php.net/manual/en/function.pg-last-error.php) para comprobar los detalles si se ha producido un error. A continuación, llama al método [pg_close()](https://secure.php.net/manual/en/function.pg-close.php) para cerrar la conexión.

Reemplace los parámetros `$host`, `$database`, `$user` y `$password` por sus propios valores. 

```php
<?php
    // Initialize connection variables.
    $host = "mydemoserver.postgres.database.azure.com";
    $database = "mypgsqldb";
    $user = "mylogin@mydemoserver";
    $password = "<server_admin_password>";

    // Initialize connection object.
    $connection = pg_connect("host=$host dbname=$database user=$user password=$password")
                or die("Failed to create connection to database: ". pg_last_error(). ".<br/>");

    print "Successfully created connection to database. <br/>";

    // Modify some data in table.
    $new_quantity = 200;
    $name = '\'banana\'';
    $query = "UPDATE inventory SET quantity = $new_quantity WHERE name = $name;";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). ".<br/>");
    print "Updated 1 row of data. </br>";

    // Closing connection
    pg_close($connection);
?>
```


## <a name="delete-data"></a>Eliminación de datos
Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **DELETE**. 

 El código llama al método [pg_connect()](https://secure.php.net/manual/en/function.pg-connect.php) para conectar con Azure Database for PostgreSQL. A continuación, llama al método [pg_query()](https://secure.php.net/manual/en/function.pg-query.php) para ejecutar un comando, y a [pg_last_error()](https://secure.php.net/manual/en/function.pg-last-error.php) para comprobar los detalles si se ha producido un error. A continuación, llama al método [pg_close()](https://secure.php.net/manual/en/function.pg-close.php) para cerrar la conexión.

Reemplace los parámetros `$host`, `$database`, `$user` y `$password` por sus propios valores. 

```php
<?php
    // Initialize connection variables.
    $host = "mydemoserver.postgres.database.azure.com";
    $database = "mypgsqldb";
    $user = "mylogin@mydemoserver";
    $password = "<server_admin_password>";

    // Initialize connection object.
    $connection = pg_connect("host=$host dbname=$database user=$user password=$password")
            or die("Failed to create connection to database: ". pg_last_error(). ". </br>");

    print "Successfully created connection to database. <br/>";

    // Delete some data from table.
    $name = '\'orange\'';
    $query = "DELETE FROM inventory WHERE name = $name;";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). ". <br/>");
    print "Deleted 1 row of data. <br/>";

    // Closing connection
    pg_close($connection);
?>
```

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Migración de una base de datos mediante exportación e importación](./howto-migrate-using-export-and-import.md)
