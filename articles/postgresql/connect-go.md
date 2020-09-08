---
title: 'Inicio rápido: Conexión con Go: Azure Database for PostgreSQL (servidor único)'
description: 'En este inicio rápido se proporciona un ejemplo del lenguaje de programación Go que puede usar para conectarse a Azure Database for PostgreSQL: servidor único y consultar datos de ese servicio.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.custom: mvc
ms.devlang: go
ms.topic: quickstart
ms.date: 5/6/2019
ms.openlocfilehash: e8a44de5b6da4c9a125b5c035d0344115930cfe2
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88182908"
---
# <a name="quickstart-use-go-language-to-connect-and-query-data-in-azure-database-for-postgresql---single-server"></a>Inicio rápido: Uso del lenguaje Go para conectarse y consultar datos en Azure Database for PostgreSQL: servidor único

En este tutorial rápido se muestra cómo conectarse a una instancia de Azure Database for PostgreSQL mediante código escrito en el lenguaje [Go](https://golang.org/) (golang). Se indica cómo usar instrucciones SQL para consultar, insertar, actualizar y eliminar datos en la base de datos. En este artículo se da por hecho que está familiarizado con el desarrollo mediante Go, pero que nunca ha trabajado con Azure Database para PostgreSQL.

## <a name="prerequisites"></a>Prerrequisitos
En este tutorial rápido se usan como punto de partida los recursos creados en una de estas guías:
- [Creación de una base de datos con el portal](quickstart-create-server-database-portal.md)
- [Creación de la base de datos: CLI de Azure](quickstart-create-server-database-azure-cli.md)

## <a name="install-go-and-pq-connector"></a>Instalación de Go y del conector pq
Instale [Go](https://golang.org/doc/install) y el [controlador Pure Go Postgres (pq)](https://github.com/lib/pq) en su máquina. Dependiendo de la plataforma, siga los pasos correspondientes:

### <a name="windows"></a>Windows
1. [Descargue](https://golang.org/dl/) e instale Go para Microsoft Windows de acuerdo con las [instrucciones de instalación](https://golang.org/doc/install).
2. En el menú Inicio, inicie el símbolo del sistema.
3. Cree una carpeta para su proyecto, como `mkdir  %USERPROFILE%\go\src\postgresqlgo`.
4. Cambie el directorio a la carpeta de proyecto, por ejemplo `cd %USERPROFILE%\go\src\postgresqlgo`.
5. Establezca la variable de entorno para GOPATH con el fin de que apunte al directorio de código fuente. `set GOPATH=%USERPROFILE%\go`.
6. Instale el [controlador Pure Go Postgres (pq)](https://github.com/lib/pq) mediante la ejecución del comando `go get github.com/lib/pq`.

   En resumen, instale Go y después ejecute estos comandos en el símbolo del sistema:
   ```cmd
   mkdir  %USERPROFILE%\go\src\postgresqlgo
   cd %USERPROFILE%\go\src\postgresqlgo
   set GOPATH=%USERPROFILE%\go
   go get github.com/lib/pq
   ```

### <a name="linux-ubuntu"></a>Linux (Ubuntu)
1. Inicie el shell de Bash. 
2. Instale Go mediante la ejecución de `sudo apt-get install golang-go`.
3. Cree una carpeta para el proyecto en su directorio principal, como `mkdir -p ~/go/src/postgresqlgo/`.
4. Cambie el directorio a la carpeta, por ejemplo, `cd ~/go/src/postgresqlgo/`.
5. Establezca la variable de entorno GOPATH para que apunte a un directorio de origen válido, como la carpeta go del directorio principal actual. En el shell de Bash, ejecute `export GOPATH=~/go` para agregar el directorio go como GOPATH para la sesión de shell actual.
6. Instale el [controlador Pure Go Postgres (pq)](https://github.com/lib/pq) mediante la ejecución del comando `go get github.com/lib/pq`.

   En resumen, ejecute estos comandos de Bash:
   ```bash
   sudo apt-get install golang-go
   mkdir -p ~/go/src/postgresqlgo/
   cd ~/go/src/postgresqlgo/
   export GOPATH=~/go/
   go get github.com/lib/pq
   ```

### <a name="apple-macos"></a>MacOS de Apple
1. Descargue e instale Go de acuerdo con las [instrucciones de instalación](https://golang.org/doc/install) que coincidan con su plataforma. 
2. Inicie el shell de Bash. 
3. Cree una carpeta para el proyecto en su directorio principal, como `mkdir -p ~/go/src/postgresqlgo/`.
4. Cambie el directorio a la carpeta, por ejemplo, `cd ~/go/src/postgresqlgo/`.
5. Establezca la variable de entorno GOPATH para que apunte a un directorio de origen válido, como la carpeta go del directorio principal actual. En el shell de Bash, ejecute `export GOPATH=~/go` para agregar el directorio go como GOPATH para la sesión de shell actual.
6. Instale el [controlador Pure Go Postgres (pq)](https://github.com/lib/pq) mediante la ejecución del comando `go get github.com/lib/pq`.

   En resumen, instale Go y después ejecute estos comandos de Bash:
   ```bash
   mkdir -p ~/go/src/postgresqlgo/
   cd ~/go/src/postgresqlgo/
   export GOPATH=~/go/
   go get github.com/lib/pq
   ```

## <a name="get-connection-information"></a>Obtención de información sobre la conexión
Obtenga la información de conexión necesaria para conectarse a Azure Database for PostgreSQL. Necesitará el nombre completo del servidor y las credenciales de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En el menú izquierdo de Azure Portal, haga clic en **Todos los recursos** y, luego, busque el servidor que ha creado, por ejemplo, **mydemoserver**.
3. Haga clic en el nombre del servidor.
4. En el panel **Información general** del servidor, anote el **nombre del servidor** y el **nombre de inicio de sesión del administrador del servidor**. Si olvida la contraseña, puede restablecerla en este panel.
 ![Nombre de servidor de Azure Database for PostgreSQL](./media/connect-go/1-connection-string.png)

## <a name="build-and-run-go-code"></a>Compilación y ejecución del código de Go 
1. Para escribir código Golang, puede usar un editor de texto sin formato, como el Bloc de notas en Microsoft Windows, [vi](https://manpages.ubuntu.com/manpages/xenial/man1/nvi.1.html#contenttoc5) o [Nano](https://www.nano-editor.org/) en Ubuntu, o TextEdit en macOS. Si prefiere un entorno de desarrollo integrado (IDE) más enriquecido, pruebe [GoLand](https://www.jetbrains.com/go/) de Jetbrains, [Visual Studio Code](https://code.visualstudio.com/) de Microsoft o [Atom](https://atom.io/).
2. Pegue el código de Golang de las secciones siguientes en archivos de texto y guárdelos en la carpeta del proyecto con la extensión de archivo \*.go, como la ruta de acceso de Windows `%USERPROFILE%\go\src\postgresqlgo\createtable.go` o la ruta de acceso de Linux `~/go/src/postgresqlgo/createtable.go`.
3. Busque las constantes `HOST`, `DATABASE`, `USER` y `PASSWORD` en el código y reemplace los valores de ejemplo con sus propios valores.  
4. Inicie el símbolo del sistema o el shell de Bash. Cambie el directorio a la carpeta de proyecto. Por ejemplo, en Windows `cd %USERPROFILE%\go\src\postgresqlgo\`. En Linux `cd ~/go/src/postgresqlgo/`. Algunos de los entornos de IDE mencionados ofrecen funcionalidades de depuración y de tiempo de ejecución sin necesidad de comandos de shell.
5. Ejecute el código escribiendo el comando `go run createtable.go` para compilar la aplicación y ejecútela. 
6. Además, para compilar el código en una aplicación nativa, `go build createtable.go`, inicie `createtable.exe` para ejecutar la aplicación.

## <a name="connect-and-create-a-table"></a>Conexión y creación de una tabla
Use el código siguiente para conectarse y crear una tabla mediante la instrucción SQL **CREATE TABLE**, seguida de las instrucciones SQL **INSERT INTO** para agregar filas a la tabla.

El código importa tres paquetes: el [paquete sql](https://golang.org/pkg/database/sql/), el [paquete pq](https://godoc.org/github.com/lib/pq), como controlador para la comunicación con el servidor de PostgreSQL, y el [paquete fmt](https://golang.org/pkg/fmt/) para la entrada y la salida impresas en la línea de comandos.

El código llama al método [sql.Open()](https://godoc.org/github.com/lib/pq#Open) para conectarse a la base de datos de Azure Database for PostgreSQL y comprueba la conexión mediante el método [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping). Durante todo el proceso se usa un [identificador de base de datos](https://golang.org/pkg/database/sql/#DB), que contiene el grupo de conexiones del servidor de base de datos. El código llama al método [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) varias veces para ejecutar varios comandos SQL. En todo momento, un método checkError() personalizado comprueba si se ha producido un error y, en caso afirmativo, avisa para salir.

Reemplace los parámetros `HOST`, `DATABASE`, `USER` y `PASSWORD` por sus propios valores. 

```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {
    // Initialize connection string.
    var connectionString string = fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Drop previous table of same name if one exists.
    _, err = db.Exec("DROP TABLE IF EXISTS inventory;")
    checkError(err)
    fmt.Println("Finished dropping table (if existed)")

    // Create table.
    _, err = db.Exec("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
    checkError(err)
    fmt.Println("Finished creating table")

    // Insert some data into table.
    sql_statement := "INSERT INTO inventory (name, quantity) VALUES ($1, $2);"
    _, err = db.Exec(sql_statement, "banana", 150)
    checkError(err)
    _, err = db.Exec(sql_statement, "orange", 154)
    checkError(err)
    _, err = db.Exec(sql_statement, "apple", 100)
    checkError(err)
    fmt.Println("Inserted 3 rows of data")
}
```

## <a name="read-data"></a>Lectura de datos
Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **SELECT**. 

El código importa tres paquetes: el [paquete sql](https://golang.org/pkg/database/sql/), el [paquete pq](https://godoc.org/github.com/lib/pq), como controlador para la comunicación con el servidor de PostgreSQL, y el [paquete fmt](https://golang.org/pkg/fmt/) para la entrada y la salida impresas en la línea de comandos.

El código llama al método [sql.Open()](https://godoc.org/github.com/lib/pq#Open) para conectarse a la base de datos de Azure Database for PostgreSQL y comprueba la conexión mediante el método [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping). Durante todo el proceso se usa un [identificador de base de datos](https://golang.org/pkg/database/sql/#DB), que contiene el grupo de conexiones del servidor de base de datos. Para ejecutar la consulta de selección, se llama al método [db.Query()](https://golang.org/pkg/database/sql/#DB.Query) y las filas resultantes se guardan en una variable de tipo [rows](https://golang.org/pkg/database/sql/#Rows). El código lee los valores de los datos de las columnas de la fila actual mediante el método [rows.Scan()](https://golang.org/pkg/database/sql/#Rows.Scan) y recorre las filas mediante el iterador [rows.Next()](https://golang.org/pkg/database/sql/#Rows.Next) hasta que no existen más filas. Los valores de las columnas de cada fila se imprimen en la salida de la consola. En todo momento, se usa un método checkError() personalizado para comprobar si se ha producido un error y, en caso afirmativo, avisa para salir.

Reemplace los parámetros `HOST`, `DATABASE`, `USER` y `PASSWORD` por sus propios valores. 

```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {

    // Initialize connection string.
    var connectionString string = fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Read rows from table.
    var id int
    var name string
    var quantity int

    sql_statement := "SELECT * from inventory;"
    rows, err := db.Query(sql_statement)
    checkError(err)
    defer rows.Close()

    for rows.Next() {
        switch err := rows.Scan(&id, &name, &quantity); err {
        case sql.ErrNoRows:
            fmt.Println("No rows were returned")
        case nil:
            fmt.Printf("Data row = (%d, %s, %d)\n", id, name, quantity)
        default:
            checkError(err)
        }
    }
}
```

## <a name="update-data"></a>Actualización de datos
Use el código siguiente para conectarse y actualizar los datos mediante la instrucción SQL **UPDATE**.

El código importa tres paquetes: el [paquete sql](https://golang.org/pkg/database/sql/), el [paquete pq](https://godoc.org/github.com/lib/pq), como controlador para la comunicación con el servidor de Postgres, y el [paquete fmt](https://golang.org/pkg/fmt/) para la entrada y la salida impresas en la línea de comandos.

El código llama al método [sql.Open()](https://godoc.org/github.com/lib/pq#Open) para conectarse a la base de datos de Azure Database for PostgreSQL y comprueba la conexión mediante el método [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping). Durante todo el proceso se usa un [identificador de base de datos](https://golang.org/pkg/database/sql/#DB), que contiene el grupo de conexiones del servidor de base de datos. El código llama al método [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) para ejecutar la instrucción SQL que actualiza la tabla. Se usa un método checkError() personalizado para comprobar si se ha producido un error y, en caso afirmativo, avisa para salir.

Reemplace los parámetros `HOST`, `DATABASE`, `USER` y `PASSWORD` por sus propios valores. 
```go
package main

import (
  "database/sql"
  _ "github.com/lib/pq"
  "fmt"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {
    
    // Initialize connection string.
    var connectionString string = 
        fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Modify some data in table.
    sql_statement := "UPDATE inventory SET quantity = $2 WHERE name = $1;"
    _, err = db.Exec(sql_statement, "banana", 200)
    checkError(err)
    fmt.Println("Updated 1 row of data")
}
```

## <a name="delete-data"></a>Eliminación de datos
Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **DELETE**. 

El código importa tres paquetes: el [paquete sql](https://golang.org/pkg/database/sql/), el [paquete pq](https://godoc.org/github.com/lib/pq), como controlador para la comunicación con el servidor de Postgres, y el [paquete fmt](https://golang.org/pkg/fmt/) para la entrada y la salida impresas en la línea de comandos.

El código llama al método [sql.Open()](https://godoc.org/github.com/lib/pq#Open) para conectarse a la base de datos de Azure Database for PostgreSQL y comprueba la conexión mediante el método [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping). Durante todo el proceso se usa un [identificador de base de datos](https://golang.org/pkg/database/sql/#DB), que contiene el grupo de conexiones del servidor de base de datos. El código llama al método [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) para ejecutar la instrucción SQL que elimina una fila de la tabla. Se usa un método checkError() personalizado para comprobar si se ha producido un error y, en caso afirmativo, avisa para salir.

Reemplace los parámetros `HOST`, `DATABASE`, `USER` y `PASSWORD` por sus propios valores. 
```go
package main

import (
  "database/sql"
  _ "github.com/lib/pq"
  "fmt"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {
    
    // Initialize connection string.
    var connectionString string = 
        fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Delete some data from table.
    sql_statement := "DELETE FROM inventory WHERE name = $1;"
    _, err = db.Exec(sql_statement, "orange")
    checkError(err)
    fmt.Println("Deleted 1 row of data")
}
```

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Migración de una base de datos mediante exportación e importación](./howto-migrate-using-export-and-import.md)
