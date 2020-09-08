---
title: Uso de Java y JDBC con Azure Database for MySQL
description: Aprenda a usar Java y JDBC con una instancia de Azure Database for MySQL.
author: jdubois
ms.author: judubois
ms.service: mysql
ms.custom: mvc, devcenter
ms.topic: quickstart
ms.devlang: java
ms.date: 08/17/2020
ms.openlocfilehash: a54e950286a37c207d902090f015b3732e0ff10b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88517589"
---
# <a name="use-java-and-jdbc-with-azure-database-for-mysql"></a>Uso de Java y JDBC con Azure Database for MySQL

En este tema se muestra cómo crear una aplicación de ejemplo que utiliza Java y [JDBC](https://en.wikipedia.org/wiki/Java_Database_Connectivity) para almacenar y recuperar información de [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/).

JDBC es la API de Java estándar para conectarse a bases de datos relacionales tradicionales.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure. Si no tiene una, [obtenga la versión de evaluación gratuita](https://azure.microsoft.com/free/).
- [Azure Cloud Shell](/azure/cloud-shell/quickstart) o la [CLI de Azure](/cli/azure/install-azure-cli). Se recomienda Azure Cloud Shell, ya que se iniciará sesión automáticamente y tendrá acceso a todas las herramientas que necesite.
- Un [kit de desarrollo de Java](https://aka.ms/azure-jdks) compatible, versión 8 (incluido en Azure Cloud Shell).
- La herramienta de compilación [Apache Maven](https://maven.apache.org/).

## <a name="prepare-the-working-environment"></a>Preparación del entorno de trabajo

Vamos a usar variables de entorno para limitar los errores tipográficos y para que sea más fácil personalizar la configuración siguiente para sus necesidades específicas.

Configure esas variables de entorno mediante los comandos siguientes:

```bash
AZ_RESOURCE_GROUP=database-workshop
AZ_DATABASE_NAME=<YOUR_DATABASE_NAME>
AZ_LOCATION=<YOUR_AZURE_REGION>
AZ_MYSQL_USERNAME=demo
AZ_MYSQL_PASSWORD=<YOUR_MYSQL_PASSWORD>
AZ_LOCAL_IP_ADDRESS=<YOUR_LOCAL_IP_ADDRESS>
```

Reemplace los marcadores de posición por los valores siguientes, que se usan a lo largo de este artículo:

- `<YOUR_DATABASE_NAME>`: nombre del servidor de MySQL Server. Debe ser único en Azure.
- `<YOUR_AZURE_REGION>`: región de Azure que va a usar. Puede usar `eastus` de forma predeterminada, pero se recomienda que configure una región más cercana a la ubicación en la que vive. Puede obtener la lista completa de las regiones disponibles; para ello, escriba `az account list-locations`.
- `<YOUR_MYSQL_PASSWORD>`: contraseña del servidor de base de datos MySQL. Esa contraseña debe tener un mínimo de ocho caracteres. Debe contener caracteres de tres de las siguientes categorías: Letras del alfabeto inglés mayúsculas y minúsculas, números (0-9) y caracteres no alfanuméricos (!, $, #, %, etc.).
- `<YOUR_LOCAL_IP_ADDRESS>`: dirección IP del equipo local, desde el que se ejecutará la aplicación de Java. Una manera cómoda de encontrarla es escribir en el explorador web la dirección [whatismyip.akamai.com](http://whatismyip.akamai.com/).

A continuación, cree un grupo de recursos:

```azurecli
az group create \
    --name $AZ_RESOURCE_GROUP \
    --location $AZ_LOCATION \
  	| jq
```

> [!NOTE]
> Vamos a usar la utilidad `jq`, que se instala de forma predeterminada en [Azure Cloud Shell](https://shell.azure.com/), para mostrar los datos JSON y facilitar su lectura.
> Si esa utilidad no es de su agrado, puede quitar de forma segura la parte que hace referencia a `| jq` de todos los comandos que vamos a utilizar.

## <a name="create-an-azure-database-for-mysql-instance"></a>Creación de una instancia de Azure Database for MySQL

Lo primero que crearemos es un servidor administrado de MySQL.

> [!NOTE]
> Puede leer información más detallada sobre la creación de servidores de MySQL en [Creación de un servidor de Azure Database for MySQL mediante Azure Portal](/azure/mysql/quickstart-create-mysql-server-database-using-azure-portal).

En [Azure Cloud Shell](https://shell.azure.com/), ejecute el siguiente script.

```azurecli
az mysql server create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name $AZ_DATABASE_NAME \
    --location $AZ_LOCATION \
    --sku-name B_Gen5_1 \
    --storage-size 5120 \
    --admin-user $AZ_MYSQL_USERNAME \
    --admin-password $AZ_MYSQL_PASSWORD \
  	| jq
```

Este comando crea un servidor de MySQL pequeño.

### <a name="configure-a-firewall-rule-for-your-mysql-server"></a>Configuración de una regla de firewall para el servidor de MySQL

De forma predeterminada, las instancias de Azure Database for MySQL están protegidas. Tienen un firewall que no permite ninguna conexión entrante. Para poder usar la base de datos, es necesario agregar una regla de firewall que permita a la dirección IP local acceder al servidor de base de datos.

Como hemos configurado nuestra dirección IP local al principio de este artículo, puede abrir el firewall del servidor mediante la ejecución de:

```azurecli
az mysql server firewall-rule create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name $AZ_DATABASE_NAME-database-allow-local-ip \
    --server $AZ_DATABASE_NAME \
    --start-ip-address $AZ_LOCAL_IP_ADDRESS \
    --end-ip-address $AZ_LOCAL_IP_ADDRESS \
  	| jq
```

### <a name="configure-a-mysql-database"></a>Configuración de una base de datos MySQL

El servidor de MySQL que creó anteriormente está vacío. No tiene ninguna base de datos que pueda usar con la aplicación de Java. Cree una nueva base de datos llamada `demo`:

```azurecli
az mysql db create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name demo \
    --server-name $AZ_DATABASE_NAME \
  	| jq
```

### <a name="create-a-new-java-project"></a>Creación de un nuevo proyecto de Java

Con su IDE favorito, cree un nuevo proyecto de Java y agregue un archivo `pom.xml` en el directorio raíz:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.20</version>
        </dependency>
    </dependencies>
</project>
```

Este archivo es un archivo de [Apache Maven](https://maven.apache.org/) que configura el proyecto para usar:

- Java 8
- Un reciente controlador de MySQL para Java

### <a name="prepare-a-configuration-file-to-connect-to-azure-database-for-mysql"></a>Preparación de un archivo de configuración para conectarse a Azure Database for MySQL

Cree el archivo *src/main/resources/application.properties* y agregue:

```properties
url=jdbc:mysql://$AZ_DATABASE_NAME.mysql.database.azure.com:3306/demo?serverTimezone=UTC
user=demo@$AZ_DATABASE_NAME
password=$AZ_MYSQL_PASSWORD
```

- Reemplace las dos variables `$AZ_DATABASE_NAME` por el valor que configuró al principio de este artículo.
- Reemplace la variable `$AZ_MYSQL_PASSWORD` por el valor que configuró al principio de este artículo.

> [!NOTE]
> Anexamos `?serverTimezone=UTC` a la propiedad de configuración `url` para indicar al controlador de JDBC que use el formato de fecha UTC (hora universal coordinada) al conectarse a la base de datos. De lo contrario, el servidor Java no usaría el mismo formato de fecha que la base de datos, y daría lugar a un error.

### <a name="create-an-sql-file-to-generate-the-database-schema"></a>Creación de un archivo SQL para generar el esquema de la base de datos

Usaremos el archivo *src/main/resources/`schema.sql`* para crear un esquema de la base de datos. Cree ese archivo con el siguiente contenido:

```sql
DROP TABLE IF EXISTS todo;
CREATE TABLE todo (id SERIAL PRIMARY KEY, description VARCHAR(255), details VARCHAR(4096), done BOOLEAN);
```

## <a name="code-the-application"></a>Incorporación del código de la aplicación

### <a name="connect-to-the-database"></a>Conectarse a la base de datos

A continuación, agregue el código Java que usará JDBC para almacenar y recuperar datos del servidor de MySQL.

Cree un archivo *src/main/java/DemoApplication.java*, que contiene:

```java
package com.example.demo;

import com.mysql.cj.jdbc.AbandonedConnectionCleanupThread;

import java.sql.*;
import java.util.*;
import java.util.logging.Logger;

public class DemoApplication {

    private static final Logger log;

    static {
        System.setProperty("java.util.logging.SimpleFormatter.format", "[%4$-7s] %5$s %n");
        log =Logger.getLogger(DemoApplication.class.getName());
    }

    public static void main(String[] args) throws Exception {
        log.info("Loading application properties");
        Properties properties = new Properties();
        properties.load(DemoApplication.class.getClassLoader().getResourceAsStream("application.properties"));

        log.info("Connecting to the database");
        Connection connection = DriverManager.getConnection(properties.getProperty("url"), properties);
        log.info("Database connection test: " + connection.getCatalog());

        log.info("Create database schema");
        Scanner scanner = new Scanner(DemoApplication.class.getClassLoader().getResourceAsStream("schema.sql"));
        Statement statement = connection.createStatement();
        while (scanner.hasNextLine()) {
            statement.execute(scanner.nextLine());
        }

        /*
        Todo todo = new Todo(1L, "configuration", "congratulations, you have set up JDBC correctly!", true);
        insertData(todo, connection);
        todo = readData(connection);
        todo.setDetails("congratulations, you have updated data!");
        updateData(todo, connection);
        deleteData(todo, connection);
        */

        log.info("Closing database connection");
        connection.close();
        AbandonedConnectionCleanupThread.uncheckedShutdown();
    }
}
```

Este código Java usará los archivos *application.properties* y *schema.sql* que se crearon antes para conectarse al servidor de MySQL y crear un esquema que almacenará los datos.

En este archivo, puede ver que hemos comentado los métodos para insertar, leer, actualizar y eliminar datos: codificaremos esos métodos en el resto de este artículo y podrá quitar los comentarios uno tras otro.

> [!NOTE]
> Las credenciales de la base de datos se almacenan en las propiedades *user* (usuario) y *password* (contraseña) del archivo *application.properties*. Estas credenciales se usan al ejecutar `DriverManager.getConnection(properties.getProperty("url"), properties);`, ya que el archivo de propiedades se pasa como argumento.

> [!NOTE]
> La línea `AbandonedConnectionCleanupThread.uncheckedShutdown();` del final es un comando específico del controlador de MySQL para destruir un subproceso interno al cerrar la aplicación.
> Se puede omitir sin ningún problema. 

Ahora puede ejecutar esta clase principal con su herramienta favorita:

- Con el IDE, debería poder hacer clic con el botón derecho en la clase *DemoApplication* y ejecutarla.
- Con Maven, puede ejecutar la aplicación mediante la ejecución de: `mvn exec:java -Dexec.mainClass="com.example.demo.DemoApplication"`.

La aplicación debería conectarse a Azure Database for MySQL, crear un esquema de base de datos y cerrar la conexión, como debería ver en los registros de la consola:

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Closing database connection 
```

### <a name="create-a-domain-class"></a>Creación de una clase de dominio

Cree la nueva clase de Java `Todo`, junto a la clase `DemoApplication` y agregue el siguiente código:

```java
package com.example.demo;

public class Todo {

    private Long id;
    private String description;
    private String details;
    private boolean done;

    public Todo() {
    }

    public Todo(Long id, String description, String details, boolean done) {
        this.id = id;
        this.description = description;
        this.details = details;
        this.done = done;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getDetails() {
        return details;
    }

    public void setDetails(String details) {
        this.details = details;
    }

    public boolean isDone() {
        return done;
    }

    public void setDone(boolean done) {
        this.done = done;
    }

    @Override
    public String toString() {
        return "Todo{" +
                "id=" + id +
                ", description='" + description + '\'' +
                ", details='" + details + '\'' +
                ", done=" + done +
                '}';
    }
}
```

Esta clase es un modelo de dominio asignado a la tabla `todo` que creó al ejecutar el script *schema.sql*.

### <a name="insert-data-into-azure-database-for-mysql"></a>Inserción de datos en Azure Database for MySQL

En el archivo *src/main/java/DemoApplication.java*, después del método main, agregue el método siguiente para insertar los datos en la base de datos:

```java
private static void insertData(Todo todo, Connection connection) throws SQLException {
    log.info("Insert data");
    PreparedStatement insertStatement = connection
            .prepareStatement("INSERT INTO todo (id, description, details, done) VALUES (?, ?, ?, ?);");

    insertStatement.setLong(1, todo.getId());
    insertStatement.setString(2, todo.getDescription());
    insertStatement.setString(3, todo.getDetails());
    insertStatement.setBoolean(4, todo.isDone());
    insertStatement.executeUpdate();
}
```

Ahora puede quitar la marca de comentario de las dos líneas siguientes en el método `main`:

```java
Todo todo = new Todo(1L, "configuration", "congratulations, you have set up JDBC correctly!", true);
insertData(todo, connection);
```

La ejecución de la clase main debería generar ahora la siguiente salida:

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Closing database connection
```

### <a name="reading-data-from-azure-database-for-mysql"></a>Lectura de datos de Azure Database for MySQL

Vamos a leer los datos que se insertaron previamente para validar que el código funciona correctamente.

En el archivo *src/main/java/DemoApplication.java*, después del método `insertData`, agregue el método siguiente para leer los datos de la base de datos:

```java
private static Todo readData(Connection connection) throws SQLException {
    log.info("Read data");
    PreparedStatement readStatement = connection.prepareStatement("SELECT * FROM todo;");
    ResultSet resultSet = readStatement.executeQuery();
    if (!resultSet.next()) {
        log.info("There is no data in the database!");
        return null;
    }
    Todo todo = new Todo();
    todo.setId(resultSet.getLong("id"));
    todo.setDescription(resultSet.getString("description"));
    todo.setDetails(resultSet.getString("details"));
    todo.setDone(resultSet.getBoolean("done"));
    log.info("Data read from the database: " + todo.toString());
    return todo;
}
```

Ahora puede quitar la marca de comentario de la línea siguiente en el método `main`:

```java
todo = readData(connection);
```

La ejecución de la clase main debería generar ahora la siguiente salida:

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true} 
[INFO   ] Closing database connection 
```

### <a name="updating-data-in-azure-database-for-mysql"></a>Actualización de datos en Azure Database for MySQL

Vamos a actualizar los datos que se insertaron anteriormente.

En el archivo *src/main/java/DemoApplication.java*, después del método `readData`, agregue el método siguiente para actualizar los datos de la base de datos:

```java
private static void updateData(Todo todo, Connection connection) throws SQLException {
    log.info("Update data");
    PreparedStatement updateStatement = connection
            .prepareStatement("UPDATE todo SET description = ?, details = ?, done = ? WHERE id = ?;");

    updateStatement.setString(1, todo.getDescription());
    updateStatement.setString(2, todo.getDetails());
    updateStatement.setBoolean(3, todo.isDone());
    updateStatement.setLong(4, todo.getId());
    updateStatement.executeUpdate();
    readData(connection);
}
```

Ahora puede quitar la marca de comentario de las dos líneas siguientes en el método `main`:

```java
todo.setDetails("congratulations, you have updated data!");
updateData(todo, connection);
```

La ejecución de la clase main debería generar ahora la siguiente salida:

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true} 
[INFO   ] Update data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have updated data!', done=true} 
[INFO   ] Closing database connection 
```

### <a name="deleting-data-in-azure-database-for-mysql"></a>Eliminación de datos en Azure Database for MySQL

Por último, vamos a eliminar los datos que se insertaron anteriormente.

En el archivo *src/main/java/DemoApplication.java*, después del método `updateData`, agregue el método siguiente para eliminar los datos de la base de datos:

```java
private static void deleteData(Todo todo, Connection connection) throws SQLException {
    log.info("Delete data");
    PreparedStatement deleteStatement = connection.prepareStatement("DELETE FROM todo WHERE id = ?;");
    deleteStatement.setLong(1, todo.getId());
    deleteStatement.executeUpdate();
    readData(connection);
}
```

Ahora puede quitar la marca de comentario de la línea siguiente en el método `main`:

```java
deleteData(todo, connection);
```

La ejecución de la clase main debería generar ahora la siguiente salida:

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true} 
[INFO   ] Update data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have updated data!', done=true} 
[INFO   ] Delete data 
[INFO   ] Read data 
[INFO   ] There is no data in the database! 
[INFO   ] Closing database connection 
```

## <a name="conclusion-and-resources-clean-up"></a>Conclusión y limpieza de recursos

Felicidades. Ha creado una aplicación de Java que usa JDBC para almacenar y recuperar datos de Azure Database for MySQL.

Para limpiar todos los recursos utilizados durante esta guía de inicio rápido, elimine el grupo de recursos con el siguiente comando:

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Migre su Base de datos MySQL a Azure Database for MySQL mediante el volcado y la restauración](concepts-migrate-dump-restore.md)
