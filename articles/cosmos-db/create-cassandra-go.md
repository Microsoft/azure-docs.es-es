---
title: Creación de una aplicación de Go con Cassandra API de Azure Cosmos DB mediante el cliente gocql
description: En esta guía de inicio rápido se muestra cómo usar un cliente Go para interactuar con Cassandra API de Azure Cosmos DB.
ms.service: cosmos-db
author: abhirockzz
ms.author: abhishgu
ms.subservice: cosmosdb-cassandra
ms.devlang: go
ms.topic: quickstart
ms.date: 07/14/2020
ms.openlocfilehash: ba53fb786b1d1f61535168cda2152049a12dfb99
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86535648"
---
# <a name="quickstart-build-a-go-app-with-the-gocql-client-to-manage-azure-cosmos-db-cassandra-api-data"></a>Inicio rápido: Creación de una aplicación de Go con el cliente `gocql` para administrar los datos de Cassandra API de Azure Cosmos DB

> [!div class="op_single_selector"]
> * [.NET](create-cassandra-dotnet.md)
> * [.NET Core](create-cassandra-dotnet-core.md)
> * [Java v3](create-cassandra-java.md)
> * [Java v4](create-cassandra-java-v4.md)
> * [Node.js](create-cassandra-nodejs.md)
> * [Python](create-cassandra-python.md)
> * [Golang](create-cassandra-go.md)
>

Azure Cosmos DB es un servicio de base de datos multimodelo que permite crear y consultar rápidamente bases de datos de documentos, tablas, claves-valores y grafos con funcionalidades de distribución global y escala horizontal. En esta guía de inicio rápido, comenzará por crear una cuenta de Cassandra API de Azure Cosmos DB. A continuación, ejecutará una aplicación de Go para crear un espacio de claves de Cassandra, una tabla y ejecutar algunas operaciones. Esta aplicación de Go usa [gocql](https://github.com/gocql/gocql), que es un cliente de Cassandra para el lenguaje Go. 

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?WT.mc_id=cassandrago-docs-abhishgu). O bien, [pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/?WT.mc_id=cassandrago-docs-abhishgu) sin una suscripción de Azure.
- Instalación de [Go](https://golang.org/) en el equipo y conocimientos de cómo funciona.
- [Git](https://git-scm.com/downloads).

## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

Para poder crear una base de datos, debe crear una cuenta de Cassandra con Azure Cosmos DB.

[!INCLUDE [cosmos-db-create-dbaccount-cassandra](../../includes/cosmos-db-create-dbaccount-cassandra.md)]

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Comience por clonar la aplicación desde GitHub.

1. Abra un símbolo del sistema y cree una nueva carpeta llamada `git-samples`.

    ```bash
    md "C:\git-samples"
    ```

2. Abra una ventana de terminal de Git, como Git Bash. Use el comando `cd` para cambiar a la nueva carpeta e instalar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

3. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-cassandra-go-getting-started.git
    ```

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si le interesa aprender cómo crea el código los recursos de base de datos, puede revisar los siguientes fragmentos de código. En caso contrario, puede ir directamente a [Ejecución de la aplicación](#run-the-application).

La función `GetSession` (parte de `utils\utils.go`) devuelve un elemento [`*gocql.Session`](https://godoc.org/github.com/gocql/gocql#Session) que se usa para ejecutar operaciones del clúster como insertar, buscar, etc.

```go
func GetSession(cosmosCassandraContactPoint, cosmosCassandraPort, cosmosCassandraUser, cosmosCassandraPassword string) *gocql.Session {
    clusterConfig := gocql.NewCluster(cosmosCassandraContactPoint)
    port, err := strconv.Atoi(cosmosCassandraPort)
    
    clusterConfig.Authenticator = gocql.PasswordAuthenticator{Username: cosmosCassandraUser, Password: cosmosCassandraPassword}
    clusterConfig.Port = port
    clusterConfig.SslOpts = &gocql.SslOptions{Config: &tls.Config{MinVersion: tls.VersionTLS12}}
    clusterConfig.ProtoVersion = 4
    
    session, err := clusterConfig.CreateSession()
    ...
    return session
}
```

El host de Cassandra de Azure Cosmos DB se pasa a la función [`gocql.NewCluster`](https://godoc.org/github.com/gocql/gocql#NewCluster) para obtener una estructura [`*gocql.ClusterConfig`](https://godoc.org/github.com/gocql/gocql#ClusterConfig) que se configura para usar el nombre de usuario, la contraseña, el puerto y la versión de TLS adecuada ([requisito de seguridad de cifrado HTTPS/SSL/TLS](https://docs.microsoft.com/azure/cosmos-db/database-security?WT.mc_id=cassandrago-docs-abhishgu#how-does-azure-cosmos-db-secure-my-database)).

A continuación, se llama a la función `GetSession` desde la función `main` (`main.go`).

```go
func main() {
    session := utils.GetSession(cosmosCassandraContactPoint, cosmosCassandraPort, cosmosCassandraUser, cosmosCassandraPassword)
    defer session.Close()
    ...
}
```

La información de conectividad y las credenciales se aceptan en forma de variables de entorno (se resuelven en el método `init`).

```go
func init() {
    cosmosCassandraContactPoint = os.Getenv("COSMOSDB_CASSANDRA_CONTACT_POINT")
    cosmosCassandraPort = os.Getenv("COSMOSDB_CASSANDRA_PORT")
    cosmosCassandraUser = os.Getenv("COSMOSDB_CASSANDRA_USER")
    cosmosCassandraPassword = os.Getenv("COSMOSDB_CASSANDRA_PASSWORD")

    if cosmosCassandraContactPoint == "" || cosmosCassandraUser == "" || cosmosCassandraPassword == "" {
        log.Fatal("missing mandatory environment variables")
    }
}
```

A continuación, se usan para ejecutar varias operaciones (parte de `operations\setup.go`) en Azure Cosmos DB que comienzan con la creación de los elementos `keyspace` y `table`.

Como sugiere el nombre, la función `DropKeySpaceIfExists` elimina el elemento `keyspace` solo si este existe.

```go
const dropKeyspace = "DROP KEYSPACE IF EXISTS %s"

func DropKeySpaceIfExists(keyspace string, session *gocql.Session) {
    err := utils.ExecuteQuery(fmt.Sprintf(dropKeyspace, keyspace), session)
    if err != nil {
        log.Fatal("Failed to drop keyspace", err)
    }
    log.Println("Keyspace dropped")
}
```

La función `CreateKeySpace` se usa para crear el elemento `keyspace` (`user_profile`).

```go
const createKeyspace = "CREATE KEYSPACE %s WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 1 }"

func CreateKeySpace(keyspace string, session *gocql.Session) {
    err := utils.ExecuteQuery(fmt.Sprintf(createKeyspace, keyspace), session)
    if err != nil {
        log.Fatal("Failed to create keyspace", err)
    }
    log.Println("Keyspace created")
}
```

Esto va seguido de la creación de la tabla (`user`), de lo que se ocupa la función `CreateUserTable`.

```go
const createTable = "CREATE TABLE %s.%s (user_id int PRIMARY KEY, user_name text, user_bcity text)"

func CreateUserTable(keyspace, table string, session *gocql.Session) {
    err := session.Query(fmt.Sprintf(createTable, keyspace, table)).Exec()
    if err != nil {
        log.Fatal("failed to create table ", err)
    }
    log.Println("Table created")
}
```

Una vez que se crean el espacio de claves y la tabla, se invocan las operaciones CRUD (parte de `operations\crud.go`). 

Se utiliza `InsertUser` para crear un elemento `User`. Establece la información de usuario (identificador, nombre y ciudad) como argumentos de consulta mediante. [`Bind`](https://godoc.org/github.com/gocql/gocql#Query.Bind).

```go
const createQuery = "INSERT INTO %s.%s (user_id, user_name , user_bcity) VALUES (?,?,?)"

func InsertUser(keyspace, table string, session *gocql.Session, user model.User) {
    err := session.Query(fmt.Sprintf(createQuery, keyspace, table)).Bind(user.ID, user.Name, user.City).Exec()
    if err != nil {
        log.Fatal("Failed to create user", err)
    }
    log.Println("User created")
}
```

`FindUser` se utiliza para buscar un usuario (`model\user.go`) mediante un identificador de usuario específico, mientras que [`Scan`](https://godoc.org/github.com/gocql/gocql#Iter.Scan) enlaza los atributos del usuario (devueltos por Cassandra) a variables individuales (`userid`, `name` y `city`); esta es solo una de las formas en las que se puede usar el resultado obtenido como resultado de la consulta de búsqueda.

```go
const selectQuery = "SELECT * FROM %s.%s where user_id = ?"

func FindUser(keyspace, table string, id int, session *gocql.Session) model.User {
    var userid int
    var name, city string
    err := session.Query(fmt.Sprintf(selectQuery, keyspace, table)).Bind(id).Scan(&userid, &name, &city)

    if err != nil {
        if err == gocql.ErrNotFound {
            log.Printf("User with id %v does not exist\n", id)
        } else {
            log.Printf("Failed to find user with id %v - %v\n", id, err)
        }
    }
    return model.User{ID: userid, Name: name, City: city}
}
```

Se utiliza `FindAllUsers` para recuperar todos los usuarios. [`SliceMap`](https://godoc.org/github.com/gocql/gocql#Iter.SliceMap) se utiliza como una forma abreviada para obtener toda la información del usuario en forma de un segmento de `map`. Piense en cada elemento `map` como pares clave-valor donde el nombre de la columna (por ejemplo, `user_id`) es la clave, junto con su valor respectivo.

```go
const findAllUsersQuery = "SELECT * FROM %s.%s"

func FindAllUsers(keyspace, table string, session *gocql.Session) []model.User {
    var users []model.User
    results, _ := session.Query(fmt.Sprintf(findAllUsersQuery, keyspace, table)).Iter().SliceMap()

    for _, u := range results {
        users = append(users, mapToUser(u))
    }
    return users
}
```

Cada elemento `map` de información de usuario se convierte en un elemento `User` mediante la función `mapToUser`, que simplemente extrae el valor de su columna respectiva y lo usa para crear una instancia de la estructura `User`.

```go
func mapToUser(m map[string]interface{}) model.User {
    id, _ := m["user_id"].(int)
    name, _ := m["user_name"].(string)
    city, _ := m["user_bcity"].(string)

    return model.User{ID: id, Name: name, City: city}
}
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Como se mencionó anteriormente, la aplicación acepta los datos de conectividad y las credenciales en forma de variables de entorno. 

1. En la cuenta de Azure Cosmos DB, en [Azure Portal](https://portal.azure.com/), seleccione **Cadena de conexión**. 

    :::image type="content" source="./media/create-cassandra-go/copy-username-connection-string-azure-portal.png" alt-text="Ver y copiar los detalles de la página Cadena de conexión en Azure Portal":::

Copie los valores de los siguientes atributos (`CONTACT POINT`, `PORT`, `USERNAME` y `PRIMARY PASSWORD`) y establézcalos en las variables de entorno respectivas.

```shell
set COSMOSDB_CASSANDRA_CONTACT_POINT=<value for "CONTACT POINT">
set COSMOSDB_CASSANDRA_PORT=<value for "PORT">
set COSMOSDB_CASSANDRA_USER=<value for "USERNAME">
set COSMOSDB_CASSANDRA_PASSWORD=<value for "PRIMARY PASSWORD">
```

En la ventana de terminal, cambie a la carpeta correcta. Por ejemplo:

```shell
cd "C:\git-samples\azure-cosmosdb-cassandra-go-getting-started"
```

2. En el terminal, ejecute el comando siguiente para iniciar la aplicación.

```shell
go run main.go
```

3. La ventana de terminal muestra las notificaciones de las distintas operaciones, como la configuración del espacio de claves y la tabla, la creación de usuarios, etc.

4. En Azure Portal abra **Explorador de datos** para consultar, modificar y trabajar con estos nuevos datos. 

    :::image type="content" source="./media/create-cassandra-go/view-data-explorer-go-app.png" alt-text="Visualizar los datos en el Explorador de datos: Azure Cosmos DB":::

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido ha aprendido a crear una cuenta de Azure Cosmos DB con Cassandra API y a ejecutar una aplicación de Go que crea un contenedor y una base de datos de Cassandra. Ya puede importar datos adicionales en la cuenta de Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importación de datos de Cassandra en Azure Cosmos DB](cassandra-import-data.md)
