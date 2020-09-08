---
title: Compilación de una aplicación web mediante la API de Azure Cosmos DB para MongoDB y el SDK de .NET
description: Se presenta un ejemplo de código de .NET que se puede usar para conectarse a la API de Azure Cosmos DB para MongoDB y realizar consultas.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 05/21/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 105578df0a37ba1d53df2be8ebed1d1f6d77499f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018797"
---
# <a name="quickstart-build-a-net-web-app-using-azure-cosmos-dbs-api-for-mongodb"></a>Inicio rápido: Compilación de una aplicación web .NET mediante la API de Azure Cosmos DB para MongoDB 

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

Azure Cosmos DB es un servicio de base de datos con varios modelos y de distribución global de Microsoft. Puede crear rápidamente bases de datos de documentos, clave-valor y grafos, y realizar consultas en ellas. Todas las bases de datos se beneficiarán de las funcionalidades de distribución global y escala horizontal en Cosmos DB. 

En este inicio rápido se muestra cómo crear una cuenta de Cosmos con la [API de Azure Cosmos DB para MongoDB](mongodb-introduction.md). Después, compilará e implementará una aplicación web de lista de tareas compilada mediante el [controlador .NET de MongoDB](https://docs.mongodb.com/ecosystem/drivers/csharp/).

## <a name="prerequisites-to-run-the-sample-app"></a>Requisitos previos para ejecutar la aplicación de ejemplo

Para ejecutar el ejemplo, necesitará [Visual Studio](https://www.visualstudio.com/downloads/) y una cuenta de Azure Cosmos DB válida.

Si no tiene Visual Studio, descargue [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/) con la carga de trabajo **ASP.NET y desarrollo web** instalada con el programa de instalación.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] 

<a id="create-account"></a>
## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

El ejemplo que se describe en este artículo es compatible con la versión 2.6.1 de MongoDB.Driver.

## <a name="clone-the-sample-app"></a>Clonación de la aplicación de ejemplo

En primer lugar, descargue la aplicación de ejemplo de GitHub. 

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
    git clone https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started.git
    ```

Si no desea usar git, también puede [descargar el proyecto como un archivo ZIP](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started/archive/master.zip).

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si está interesado en aprender cómo se crean los recursos de base de datos en el código, puede revisar los siguientes fragmentos de código. En caso contrario, puede ir directamente a [Actualización de la cadena de conexión](#update-your-connection-string). 

Los fragmentos de código siguientes se han tomado del archivo Dal.cs en el directorio DAL.

* Inicialice el cliente.

    ```cs
        MongoClientSettings settings = new MongoClientSettings();
        settings.Server = new MongoServerAddress(host, 10255);
        settings.UseSsl = true;
        settings.SslSettings = new SslSettings();
        settings.SslSettings.EnabledSslProtocols = SslProtocols.Tls12;

        MongoIdentity identity = new MongoInternalIdentity(dbName, userName);
        MongoIdentityEvidence evidence = new PasswordEvidence(password);

        settings.Credential = new MongoCredential("SCRAM-SHA-1", identity, evidence);

        MongoClient client = new MongoClient(settings);
    ```

* Recupere la base de datos y la colección.

    ```cs
    private string dbName = "Tasks";
    private string collectionName = "TasksList";

    var database = client.GetDatabase(dbName);
    var todoTaskCollection = database.GetCollection<MyTask>(collectionName);
    ```

* Recupere todos los documentos.

    ```cs
    collection.Find(new BsonDocument()).ToList();
    ```

Cree una tarea e insértela en la colección.

   ```csharp
    public void CreateTask(MyTask task)
    {
        var collection = GetTasksCollectionForEdit();
        try
        {
            collection.InsertOne(task);
        }
        catch (MongoCommandException ex)
        {
            string msg = ex.Message;
        }
    }
   ```
   De forma similar, puede actualizar y eliminar los documentos mediante los métodos [collection.UpdateOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.updateOne/index.html) y [collection.DeleteOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.deleteOne/index.html). 

## <a name="update-your-connection-string"></a>Actualización de la cadena de conexión

Ahora vuelva a Azure Portal para obtener la información de la cadena de conexión y cópiela en la aplicación.

1. En [Azure Portal](https://portal.azure.com/), en la cuenta de Cosmos, en el panel de navegación izquierdo, haga clic en **Cadena de conexión** y en **Claves de lectura y escritura**. Deberá usar los botones de copia del lado derecho de la pantalla para copiar el nombre de usuario, la contraseña y el host en el archivo Dal.cs en el paso siguiente.

2. Abra el archivo **Dal.cs** en el directorio **DAL**. 

3. Copie el valor del **nombre de usuario** del portal (con el botón de copia) y conviértalo en el valor del **nombre de usuario** en el archivo **Dal.cs**. 

4. Después, copie el valor del **host** del portal y conviértalo en el valor del **host** en el archivo **Dal.cs**. 

5. Por último, copie el valor de la **contraseña** del portal y conviértalo en el valor de la **contraseña** en el archivo **Dal.cs**. 

Ya ha actualizado la aplicación con toda la información que necesita para comunicarse con Cosmos DB. 
    
## <a name="run-the-web-app"></a>Ejecución de la aplicación web

1. En Visual Studio, haga clic con el botón derecho en el proyecto en el **Explorador de soluciones** y, después, haga clic en **Administrar paquetes NuGet**. 

2. En el cuadro **Examinar** de NuGet, escriba *MongoDB.Driver*.

3. En los resultados, instale la biblioteca **MongoDB.Driver**. De este modo, se instalan el paquete de MongoDB.Driver y todas las dependencias.

4. Haga clic en CTRL + F5 para ejecutar la aplicación. La aplicación se muestra en el explorador. 

5. Haga clic en **Crear** en el explorador y cree algunas tareas en la aplicación de lista de tareas.

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cuenta de Cosmos, crear una colección y ejecutar una aplicación de consola. Ahora puede importar datos adicionales en la base de datos de Cosmos. 

> [!div class="nextstepaction"]
> [Importación de datos de MongoDB a Azure Cosmos DB](mongodb-migrate.md)
