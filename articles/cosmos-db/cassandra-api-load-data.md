---
title: 'Tutorial: Carga de datos de ejemplo en una tabla de Cassandra API en Azure Cosmos DB mediante una aplicación Java'
description: En este tutorial se explica cómo cargar datos de usuario de ejemplo en una tabla de Cassandra API en Azure Cosmos DB mediante una aplicación Java.
author: kanshiG
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 05/20/2019
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: 6a12db8e34421dd16c12d167896ef66b3377d524
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "82853026"
---
# <a name="tutorial-load-sample-data-into-a-cassandra-api-table-in-azure-cosmos-db"></a>Tutorial: Carga de datos de ejemplo en una tabla de Cassandra API en Azure Cosmos DB

Como desarrollador, puede que tenga aplicaciones que usan pares clave/valor. Puede usar una cuenta de Cassandra API en Azure Cosmos DB para almacenar y administrar los datos de los pares clave/valor. En este tutorial se explica cómo cargar datos de usuario de ejemplo en una tabla de una cuenta de Cassandra API en Azure Cosmos DB mediante una aplicación Java. La aplicación Java emplea el [controlador de Java](https://github.com/datastax/java-driver) para cargar datos del usuario, como el identificador, el nombre o la ciudad del usuario. 

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Carga de datos en una tabla de Cassandra
> * Ejecución la aplicación

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

* Este artículo pertenece a un tutorial de varias partes. Antes de empezar con este documento, no olvide [crear la cuenta, el espacio de claves y la tabla de Cassandra API](create-cassandra-api-account-java.md).   

## <a name="load-data-into-the-table"></a>Carga de datos en la tabla

Siga estos pasos para cargar datos en la tabla de Cassandra API:

1. Abra el archivo "UserRepository.java", que se encuentra en la carpeta "src\main\java\com\azure\cosmosdb\cassandra", y anexe el código para insertar los campos user_id, user_name y user_bcity en la tabla:

   ```java
   /**
   * Insert a row into user table
   *
   * @param id   user_id
   * @param name user_name
   * @param city user_bcity
   */
   public void insertUser(PreparedStatement statement, int id, String name, String city) {
        BoundStatement boundStatement = new BoundStatement(statement);
        session.execute(boundStatement.bind(id, name, city));
   }

   /**
   * Create a PrepareStatement to insert a row to user table
   *
   * @return PreparedStatement
   */
   public PreparedStatement prepareInsertStatement() {
      final String insertStatement = "INSERT INTO  uprofile.user (user_id, user_name , user_bcity) VALUES (?,?,?)";
   return session.prepare(insertStatement);
   }
   ```
 
2. Abra el archivo "UserProfile.java", que se encuentra en la carpeta "src\main\java\com\azure\cosmosdb\cassandra". Esta clase contiene el método principal que llama a los métodos createKeyspace y createTable definidos anteriormente. Ahora, anexe el código siguiente para insertar algunos datos de ejemplo en la tabla de Cassandra API.

   ```java
   //Insert rows into user table
   PreparedStatement preparedStatement = repository.prepareInsertStatement();
     repository.insertUser(preparedStatement, 1, "JohnH", "Seattle");
     repository.insertUser(preparedStatement, 2, "EricK", "Spokane");
     repository.insertUser(preparedStatement, 3, "MatthewP", "Tacoma");
     repository.insertUser(preparedStatement, 4, "DavidA", "Renton");
     repository.insertUser(preparedStatement, 5, "PeterS", "Everett");
   ```

## <a name="run-the-app"></a>Ejecución la aplicación

Abra un símbolo del sistema o la ventana de terminal y cambie la ruta de acceso a la carpeta en la que ha creado el proyecto. Ejecute el comando "mvn clean install" para generar el archivo cosmosdb-cassandra-examples.jar dentro de la carpeta de destino y ejecute la aplicación. 

```bash
cd "cassandra-demo"

mvn clean install

java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile
```

Ya puede abrir el Explorador de datos en Azure Portal para confirmar que la información del usuario se agrega a la tabla.
    
## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a cargar datos de ejemplo a una cuenta de Cassandra API en Azure Cosmos DB. Avance al siguiente artículo:

> [!div class="nextstepaction"]
> [Consulta de los datos de la cuenta de Cassandra API](cassandra-api-query-data.md)
