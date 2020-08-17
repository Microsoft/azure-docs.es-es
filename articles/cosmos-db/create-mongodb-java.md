---
title: 'Inicio rápido: Compilación de una aplicación web mediante Azure Cosmos DB API para Mongo DB y el SDK de Java'
description: Aprenda a compilar un ejemplo de código de Java que se puede usar para conectarse a la API para MongoDB de Azure Cosmos DB y realizar consultas.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: java
ms.topic: quickstart
ms.date: 12/26/2018
ms.custom: seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 301bedc75d243f427120c955bda539508ff43200
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323645"
---
# <a name="quickstart-create-a-console-app-with-java-and-the-mongodb-api-in-azure-cosmos-db"></a>Inicio rápido: Creación de una aplicación de consola con Java y la API MongoDB en Azure Cosmos DB

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

En este inicio rápido, se crea y administra una cuenta de Azure Cosmos DB para MongoDB API desde Azure Portal, y se agregan datos mediante una aplicación del SDK de Java clonada desde GitHub. Azure Cosmos DB es un servicio de base de datos multimodelo que permite crear y consultar rápidamente bases de datos de documentos, tablas, claves-valores y grafos con funcionalidades de distribución global y escala horizontal.

## <a name="prerequisites"></a>Prerrequisitos
- Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). O bien, [pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin una suscripción de Azure. También puede usar el [emulador de Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) con la cadena de conexión `.mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true`.
- [Java Development Kit (JDK), versión 8](https://www.azul.com/downloads/azure-only/zulu/?&version=java-8-lts&architecture=x86-64-bit&package=jdk). 
- [Maven](https://maven.apache.org/download.cgi). O bien, ejecute `apt-get install maven` para instalar Maven.
- [Git](https://git-scm.com/downloads). 

## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

[!INCLUDE [mongodb-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

## <a name="add-a-collection"></a>Agregar una colección

Asigne un nombre a la nueva base de datos **db** y a la nueva colección **coll**.

[!INCLUDE [cosmos-db-create-collection](../../includes/cosmos-db-mongodb-create-collection.md)] 

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ahora vamos a clonar una aplicación desde GitHub, establecer la cadena de conexión y ejecutarla. Verá lo fácil que es trabajar con datos mediante programación. 

1. Abra un símbolo del sistema, cree una carpeta nueva denominada ejemplos de GIT y, después, cierre el símbolo del sistema.

    ```bash
    md "C:\git-samples"
    ```

2. Abra una ventana de terminal de Git, como git bash y utilice el comando `cd` para cambiar a la nueva carpeta para instalar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

3. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-getting-started.git
    ```

3. A continuación, abra el código en su editor favorito. 

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si está interesado en aprender cómo se crean los recursos de base de datos en el código, puede revisar los siguientes fragmentos de código. En caso contrario, puede ir directamente a [Actualización de la cadena de conexión](#update-your-connection-string). 

Los fragmentos de código siguientes se han tomado del archivo *Program.java*.

Esta aplicación de consola usa el [controlador Java de MongoDB](https://docs.mongodb.com/ecosystem/drivers/java/). 

* Se inicializa DocumentClient.

    ```java
    MongoClientURI uri = new MongoClientURI("FILLME");`

    MongoClient mongoClient = new MongoClient(uri);            
    ```

* Se crean una base de datos y una colección.

    ```java
    MongoDatabase database = mongoClient.getDatabase("db");

    MongoCollection<Document> collection = database.getCollection("coll");
    ```

* Se insertan algunos documentos mediante `MongoCollection.insertOne`.

    ```java
    Document document = new Document("fruit", "apple")
    collection.insertOne(document);
    ```

* Se realizan algunas consultas mediante `MongoCollection.find`.

    ```java
    Document queryResult = collection.find(Filters.eq("fruit", "apple")).first();
    System.out.println(queryResult.toJson());       
    ```

## <a name="update-your-connection-string"></a>Actualización de la cadena de conexión

Ahora vuelva a Azure Portal para obtener la información de la cadena de conexión y cópiela en la aplicación.

1. En la cuenta de Azure Cosmos DB, seleccione **Inicio rápido**, **Java** y copie la cadena de conexión en el Portapapeles.

2. Abra el archivo *Program.java* y reemplace el argumento del constructor MongoClientURI por la cadena de conexión. Ya ha actualizado la aplicación con toda la información que necesita para comunicarse con Azure Cosmos DB. 
    
## <a name="run-the-console-app"></a>Ejecutar la aplicación de consola

1. Ejecute `mvn package` en un terminal para instalar los módulos de NPM necesarios.

2. Ejecute `mvn exec:java -D exec.mainClass=GetStarted.Program` en un terminal para iniciar la aplicación de Java.

Ahora puede usar [Robomongo](mongodb-robomongo.md) / [Studio 3T](mongodb-mongochef.md) para consultar, modificar y trabajar con estos nuevos datos.

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cuenta de Azure Cosmos DB para MongoDB API, a agregar una base de datos y un contenedor mediante el Explorador de datos, y a agregar datos mediante una aplicación de consola de Java. Ahora puede importar datos adicionales en la base de datos de Cosmos. 

> [!div class="nextstepaction"]
> [Importación de datos de MongoDB a Azure Cosmos DB](mongodb-migrate.md)
