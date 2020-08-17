---
title: Aplicación Java con Cassandra API de Azure Cosmos DB mediante la versión 3.0 del SDK para Java
description: Esta guía de inicio rápido muestra cómo usar la API de Cassandra de Azure Cosmos DB para crear una aplicación de perfil con Azure Portal y Java
ms.service: cosmos-db
author: TheovanKraay
ms.author: thvankra
ms.subservice: cosmosdb-cassandra
ms.devlang: java
ms.topic: quickstart
ms.date: 05/18/2020
ms.custom: seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 78a8e6cd777c239841f6d26e06a9fc348a221286
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323152"
---
# <a name="quickstart-build-a-java-app-to-manage-azure-cosmos-db-cassandra-api-data-v3-driver"></a>Inicio rápido: Compilación de una aplicación Java para administrar los datos de Cassandra API de Azure Cosmos DB (controlador v3)

> [!div class="op_single_selector"]
> * [.NET](create-cassandra-dotnet.md)
> * [.NET Core](create-cassandra-dotnet-core.md)
> * [Java v3](create-cassandra-java.md)
> * [Java v4](create-cassandra-java-v4.md)
> * [Node.js](create-cassandra-nodejs.md)
> * [Python](create-cassandra-python.md)
>  

En este inicio rápido se crea una cuenta de Cassandra API de Azure Cosmos DB y se utiliza una aplicación Java de Cassandra clonada desde GitHub para crear un contenedor y una base de datos de Cassandra mediante los [ controladores de Apache Cassandra v3.x](https://github.com/datastax/java-driver/tree/3.x) para Java. Azure Cosmos DB es un servicio de base de datos multimodelo que permite crear y consultar rápidamente bases de datos de documentos, tablas, claves-valores y grafos con funcionalidades de distribución global y escala horizontal.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). O bien, [pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin una suscripción de Azure.
- [Java Development Kit (JDK) 8](https://www.azul.com/downloads/azure-only/zulu/?&version=java-8-lts&architecture=x86-64-bit&package=jdk). Apunte su variable de entorno `JAVA_HOME` a la carpeta donde está instalado el JDK.
- Un [archivo binario de Maven](https://maven.apache.org/download.cgi). En Ubuntu, ejecute `apt-get install maven` para instalar Maven.
- [Git](https://www.git-scm.com/downloads). En Ubuntu, ejecute `sudo apt-get install git` para instalar Git.

## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

Para poder crear una base de datos de documentos, debe crear una cuenta de Cassandra con Azure Cosmos DB.

[!INCLUDE [cosmos-db-create-dbaccount-cassandra](../../includes/cosmos-db-create-dbaccount-cassandra.md)]

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ahora vamos a empezar a trabajar con el código. Vamos a clonar una aplicación de Cassandra desde GitHub, establecer la cadena de conexión y ejecutarla. Verá lo fácil que es trabajar con datos mediante programación. 

1. Abra un símbolo del sistema. Cree una carpeta nueva denominada `git-samples`. Después cierre el símbolo del sistema.

    ```bash
    md "C:\git-samples"
    ```

2. Abra una ventana de terminal de Git, como git bash y utilice el comando `cd` para cambiar a la nueva carpeta para instalar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

3. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started.git
    ```

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si le interesa aprender cómo el código crea los recursos de base de datos, puede revisar los siguientes fragmentos de código. En caso contrario, puede ir directamente a [Actualización de la cadena de conexión](#update-your-connection-string). Estos fragmentos de código se han tomado del archivo *src/main/java/com/azure/cosmosdb/cassandra/util/CassandraUtils.java*.  

* Se establecen las opciones de host, puerto, nombre de usuario, contraseña y TLS/SSL de Cassandra. La información de la cadena de conexión procede de la página de cadena de conexión de Azure Portal.

   ```java
   cluster = Cluster.builder().addContactPoint(cassandraHost).withPort(cassandraPort).withCredentials(cassandraUsername, cassandraPassword).withSSL(sslOptions).build();
   ```

* El elemento `cluster` se conecta a la API de Cassandra de Azure Cosmos DB y devuelve una sesión a la que se accede.

    ```java
    return cluster.connect();
    ```

Los fragmentos de código siguientes son del archivo *src/main/java/com/azure/cosmosdb/cassandra/repository/UserRepository.java*.

* Cree un nuevo espacio de claves.

    ```java
    public void createKeyspace() {
        final String query = "CREATE KEYSPACE IF NOT EXISTS uprofile WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '3' } ";
        session.execute(query);
        LOGGER.info("Created keyspace 'uprofile'");
    }
    ```

* Cree una nueva tabla.

   ```java
   public void createTable() {
        final String query = "CREATE TABLE IF NOT EXISTS uprofile.user (user_id int PRIMARY KEY, user_name text, user_bcity text)";
        session.execute(query);
        LOGGER.info("Created table 'user'");
   }
   ```

* Inserte las entidades de usuario mediante un objeto de instrucción preparado.

    ```java
    public PreparedStatement prepareInsertStatement() {
        final String insertStatement = "INSERT INTO  uprofile.user (user_id, user_name , user_bcity) VALUES (?,?,?)";
        return session.prepare(insertStatement);
    }

    public void insertUser(PreparedStatement statement, int id, String name, String city) {
        BoundStatement boundStatement = new BoundStatement(statement);
        session.execute(boundStatement.bind(id, name, city));
    }
    ```

* Realice una consulta para obtener la información de todos los usuarios.

    ```java
   public void selectAllUsers() {
        final String query = "SELECT * FROM uprofile.user";
        List<Row> rows = session.execute(query).all();

        for (Row row : rows) {
            LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
        }
    }
    ```

* Realice una consulta para obtener la información de un único usuario.

    ```java
    public void selectUser(int id) {
        final String query = "SELECT * FROM uprofile.user where user_id = 3";
        Row row = session.execute(query).one();

        LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
    }
    ```

## <a name="update-your-connection-string"></a>Actualización de la cadena de conexión

Ahora vuelva a Azure Portal para obtener la información de la cadena de conexión y cópiela en la aplicación. Los detalles de la cadena de conexión permiten a la aplicación comunicarse con la base de datos hospedada.

1. En la cuenta de Azure Cosmos DB, en [Azure Portal](https://portal.azure.com/), seleccione **Cadena de conexión**. 

    :::image type="content" source="./media/create-cassandra-java/copy-username-connection-string-azure-portal.png" alt-text="Visualizar y copiar un nombre de usuario de Azure Portal, página Cadena de conexión":::

2. Use el botón :::image type="icon" source="./media/create-cassandra-java/copy-button-azure-portal.png"::: del lado derecho de la pantalla para copiar el valor de CONTACT POINT (PUNTO DE CONTACTO). 

3. Abra el archivo *config.properties* desde la carpeta *C:\git-samples\azure-cosmosdb-cassandra-java-getting-started\java-examples\src\main\resources*. 

3. Pegue el valor de CONTACT POINT (PUNTO DE CONTACTO) del portal sobre `<Cassandra endpoint host>` en la línea 2.

    La línea 2 de *config.properties* debería ser similar a esta: 

    `cassandra_host=cosmos-db-quickstart.cassandra.cosmosdb.azure.com`

3. Vuelva al portal y copie el valor de NOMBRE DE USUARIO. Pegue el valor de NOMBRE DE USUARIO del portal sobre `<cassandra endpoint username>` en la línea 4.

    La línea 4 de *config.properties* debería parecerse a esta: 

    `cassandra_username=cosmos-db-quickstart`

4. Vuelva al portal y copie el valor de CONTRASEÑA. Pegue el valor de CONTRASEÑA del portal sobre `<cassandra endpoint password>` en la línea 5.

    La línea 5 de *config.properties* debería parecerse a esta: 

    `cassandra_password=2Ggkr662ifxz2Mg...==`

5. En la línea 6, si quiere usar un certificado TLS/SSL específico, reemplace `<SSL key store file location>` por la ubicación del certificado TLS/SSL. Si no se proporciona un valor, se usa el certificado JDK instalado en <JAVA_HOME>/jre/lib/security/cacerts. 

6. Si cambió la línea 6 para usar un certificado TLS/SSL específico, actualice la línea 7 para usar la contraseña para ese certificado. 

7. Guarde el archivo *config.properties*.

## <a name="run-the-java-app"></a>Ejecución de la aplicación Java

1. En la ventana de terminal de GIT, `cd` a la carpeta `azure-cosmosdb-cassandra-java-getting-started`.

    ```git
    cd "C:\git-samples\azure-cosmosdb-cassandra-java-getting-started"
    ```

2. En la ventana de terminal de GIT, use el siguiente comando para generar el archivo `cosmosdb-cassandra-examples.jar`.

    ```git
    mvn clean install
    ```

3. En la ventana de terminal de Git, ejecute el siguiente comando para iniciar la aplicación de Java.

    ```git
    java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile
    ```

    En la ventana de terminal se notifica que se ha creado el espacio de claves y la tabla. A continuación, se seleccionan y devuelven todos los usuarios de la tabla y se muestra la salida. Seguidamente, se selecciona una fila por identificador y se muestra el valor.  

    Presione Ctrl+C para detener la ejecución del programa y cerrar la ventana de la consola.

4. En Azure Portal abra **Explorador de datos** para consultar, modificar y trabajar con estos nuevos datos. 

    :::image type="content" source="./media/create-cassandra-java/view-data-explorer-java-app.png" alt-text="Visualizar los datos en el Explorador de datos: Azure Cosmos DB":::

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido a crear una cuenta de Azure Cosmos DB con Cassandra API y a ejecutar una aplicación Java de Cassandra que cree un contenedor y una base de datos de Cassandra. Ya puede importar datos adicionales en la cuenta de Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importación de datos de Cassandra en Azure Cosmos DB](cassandra-import-data.md)
