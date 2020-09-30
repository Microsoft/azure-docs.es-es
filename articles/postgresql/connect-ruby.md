---
title: 'Inicio rápido: Conexión con Ruby: Azure Database for PostgreSQL (servidor único)'
description: 'En este inicio rápido se proporciona un ejemplo de código de Ruby que puede usar para conectarse a Azure Database for PostgreSQL: servidor único y consultar datos de ese servicio.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.custom: mvc
ms.devlang: ruby
ms.topic: quickstart
ms.date: 5/6/2019
ms.openlocfilehash: e33e80a7363ac27bd7e0983a1ff3b65ee85b86b2
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90882383"
---
# <a name="quickstart-use-ruby-to-connect-and-query-data-in-azure-database-for-postgresql---single-server"></a>Inicio rápido: Uso de Ruby para conectarse y consultar datos en Azure Database for PostgreSQL: servidor único

En este tutorial rápido se muestra cómo conectarse a una instancia de Azure Database for PostgreSQL mediante una aplicación de [Ruby](https://www.ruby-lang.org). Se indica cómo usar instrucciones SQL para consultar, insertar, actualizar y eliminar datos en la base de datos. En los pasos de este artículo se da por hecho que está familiarizado con el desarrollo mediante Ruby, pero que nunca ha trabajado con Azure Database for PostgreSQL.

## <a name="prerequisites"></a>Prerrequisitos
En este tutorial rápido se usan como punto de partida los recursos creados en una de estas guías:
- [Creación de una base de datos con el portal](quickstart-create-server-database-portal.md)
- [Creación de la base de datos: CLI de Azure](quickstart-create-server-database-azure-cli.md)

También necesita tener instalado:
- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [Ruby pg](https://rubygems.org/gems/pg/), el módulo PostgreSQL para Ruby

## <a name="get-connection-information"></a>Obtención de información sobre la conexión
Obtenga la información de conexión necesaria para conectarse a Azure Database for PostgreSQL. Necesitará el nombre completo del servidor y las credenciales de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En el menú izquierdo de Azure Portal, haga clic en **Todos los recursos** y, luego, busque el servidor que ha creado, por ejemplo, **mydemoserver**.
3. Haga clic en el nombre del servidor.
4. En el panel **Información general** del servidor, anote el **nombre del servidor** y el **nombre de inicio de sesión del administrador del servidor**. Si olvida la contraseña, puede restablecerla en este panel.
 :::image type="content" source="./media/connect-ruby/1-connection-string.png" alt-text="Nombre de servidor de Azure Database for PostgreSQL":::

> [!NOTE]
> El símbolo `@` en el nombre de usuario de Azure Postgres se ha codificado con una dirección URL como `%40` en todas las cadenas de conexión.

## <a name="connect-and-create-a-table"></a>Conexión y creación de una tabla
Use el código siguiente para conectarse y crear una tabla mediante la instrucción SQL **CREATE TABLE**, seguida de las instrucciones SQL **INSERT INTO** para agregar filas a la tabla.

El código usa un objeto ```PG::Connection``` con el constructor ```new``` para conectarse a Azure Database for PostgreSQL. A continuación, realiza una llamada al método ```exec()``` para ejecutar los comandos DROP, CREATE TABLE e INSERT INTO. El código comprueba si hay errores mediante la clase ```PG::Error```. A continuación, llama al método ```close()``` para cerrar la conexión antes de terminar. Consulte la [documentación de referencia de Ruby Pg](https://www.rubydoc.info/gems/pg/PG) para obtener más información sobre estas clases y métodos.

Reemplace `host`, `database`, `user` y las cadenas `password` por sus propios valores.


```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database'

    # Drop previous table of same name if one exists
    connection.exec('DROP TABLE IF EXISTS inventory;')
    puts 'Finished dropping table (if existed).'

    # Drop previous table of same name if one exists.
    connection.exec('CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);')
    puts 'Finished creating table.'

    # Insert some data into table.
    connection.exec("INSERT INTO inventory VALUES(1, 'banana', 150)")
    connection.exec("INSERT INTO inventory VALUES(2, 'orange', 154)")
    connection.exec("INSERT INTO inventory VALUES(3, 'apple', 100)")
    puts 'Inserted 3 rows of data.'

rescue PG::Error => e
    puts e.message

ensure
    connection.close if connection
end
```

## <a name="read-data"></a>Lectura de datos
Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **SELECT**.

El código usa un objeto ```PG::Connection``` con el constructor ```new``` para conectarse a Azure Database for PostgreSQL. A continuación, llama al método ```exec()``` para ejecutar el comando SELECT, conservando los resultados en un conjunto de resultados. La colección de resultados se repite con el bucle `resultSet.each do`, conservando los valores de fila actuales de la variable `row`. El código comprueba si hay errores mediante la clase ```PG::Error```. A continuación, llama al método ```close()``` para cerrar la conexión antes de terminar. Consulte la [documentación de referencia de Ruby Pg](https://www.rubydoc.info/gems/pg/PG) para obtener más información sobre estas clases y métodos.

Reemplace `host`, `database`, `user` y las cadenas `password` por sus propios valores.

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :database => dbname, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    resultSet = connection.exec('SELECT * from inventory;')
    resultSet.each do |row|
        puts 'Data row = (%s, %s, %s)' % [row['id'], row['name'], row['quantity']]
    end

rescue PG::Error => e
    puts e.message

ensure
    connection.close if connection
end
```

## <a name="update-data"></a>Actualización de datos
Use el código siguiente para conectarse y actualizar los datos mediante la instrucción SQL **UPDATE**.

El código usa un objeto ```PG::Connection``` con el constructor ```new``` para conectarse a Azure Database for PostgreSQL. A continuación, llama al método ```exec()``` para ejecutar el comando UPDATE. El código comprueba si hay errores mediante la clase ```PG::Error```. A continuación, llama al método ```close()``` para cerrar la conexión antes de terminar. Consulte la [documentación de referencia de Ruby Pg](https://www.rubydoc.info/gems/pg/PG) para obtener más información sobre estas clases y métodos.

Reemplace `host`, `database`, `user` y las cadenas `password` por sus propios valores.

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    # Modify some data in table.
    connection.exec('UPDATE inventory SET quantity = %d WHERE name = %s;' % [200, '\'banana\''])
    puts 'Updated 1 row of data.'

rescue PG::Error => e
    puts e.message

ensure
    connection.close if connection
end
```


## <a name="delete-data"></a>Eliminación de datos
Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **DELETE**.

El código usa un objeto ```PG::Connection``` con el constructor ```new``` para conectarse a Azure Database for PostgreSQL. A continuación, llama al método ```exec()``` para ejecutar el comando UPDATE. El código comprueba si hay errores mediante la clase ```PG::Error```. A continuación, llama al método ```close()``` para cerrar la conexión antes de terminar.

Reemplace `host`, `database`, `user` y las cadenas `password` por sus propios valores.

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    # Modify some data in table.
    connection.exec('DELETE FROM inventory WHERE name = %s;' % ['\'orange\''])
    puts 'Deleted 1 row of data.'

rescue PG::Error => e
    puts e.message

ensure
    connection.close if connection
end
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Migración de una base de datos mediante exportación e importación](./howto-migrate-using-export-and-import.md) <br/>
> [!div class="nextstepaction"]
> [Documentación de referencia sobre Ruby Pg](https://www.rubydoc.info/gems/pg/PG)
