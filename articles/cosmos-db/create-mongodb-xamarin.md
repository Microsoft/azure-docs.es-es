---
title: Compilación de una aplicación de Xamarin con .NET y la API de Azure Cosmos DB para MongoDB
description: En este tema se presenta un ejemplo de código de Xamarin que se puede usar para conectarse a la API de Azure Cosmos DB para MongoDB y realizar consultas
author: codemillmatt
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 03/16/2020
ms.author: masoucou
ms.custom: devx-track-csharp
ms.openlocfilehash: 0d5e8643abf41a488dd5f9b8cbc39c3f1bac1c0a
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89005044"
---
# <a name="quickstart-build-a-xamarinforms-app-with-net-sdk-and-azure-cosmos-dbs-api-for-mongodb"></a>Inicio rápido: Creación de una aplicación Xamarin.Forms con .NET SDK y la API de Azure Cosmos DB para MongoDB

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

Azure Cosmos DB es un servicio de base de datos con varios modelos y de distribución global de Microsoft. Puede crear rápidamente bases de datos de documentos, clave-valor y grafos, y realizar consultas en ellas. Todas las bases de datos se beneficiarán de las funcionalidades de distribución global y escala horizontal en Azure Cosmos DB.

En este artículo de inicio rápido se muestra cómo crear una [cuenta de Cosmos configurada con la API de Azure Cosmos DB para MongoDB](mongodb-introduction.md), una base de datos de documentos y una colección desde Azure Portal. Luego compilará una aplicación Xamarin.Forms de aplicación de tareas pendientes mediante el [controlador .NET de MongoDB](https://docs.mongodb.com/ecosystem/drivers/csharp/).

## <a name="prerequisites-to-run-the-sample-app"></a>Requisitos previos para ejecutar la aplicación de ejemplo

Para ejecutar el ejemplo, necesitará [Visual Studio](https://www.visualstudio.com/downloads/) o [Visual Studio para Mac](https://visualstudio.microsoft.com/vs/mac/) y una cuenta de Azure CosmosDB válida.

Si no tiene Visual Studio, descargue [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/) con la carga de trabajo **desarrollo móvil con .NET** instalada con el programa de instalación.

Si prefiere trabajar en un equipo Mac, descargue [Visual Studio para Mac](https://visualstudio.microsoft.com/vs/mac/) y ejecute el programa de instalación.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

<a id="create-account"></a>

## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

El ejemplo que se describe en este artículo es compatible con la versión 2.6.1 de MongoDB.Driver.

## <a name="clone-the-sample-app"></a>Clonación de la aplicación de ejemplo

En primer lugar, descargue la aplicación de ejemplo de GitHub. Implementa una aplicación de tareas pendientes con el modelo de almacenamiento de documentos de MongoDB.



# <a name="windows"></a>[Windows](#tab/windows)

1. En Windows, abra un símbolo del sistema o, en Mac, abra Terminal, cree una nueva carpeta denominada git-samples y, a continuación, cierre la ventana.

    ```batch
    md "C:\git-samples"
    ```

    ```bash
    mkdir '$home\git-samples\
    ```

2. Abra una ventana de terminal de Git, como git bash y utilice el comando `cd` para cambiar a la nueva carpeta para instalar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

3. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-mongodb-xamarin-getting-started.git
    ```

Si no desea usar git, también puede [descargar el proyecto como un archivo ZIP](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-xamarin-getting-started/archive/master.zip)

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si está interesado en aprender cómo se crean los recursos de base de datos en el código, puede revisar los siguientes fragmentos de código. En caso contrario, puede ir directamente a [Actualización de la cadena de conexión](#update-your-connection-string).

Todos los fragmentos de código siguiente se toman de la clase `MongoService`, que se encuentra en la ruta de acceso siguiente: src/TaskList.Core/Services/MongoService.cs.

* Inicialice Mongo Client.
    ```cs
    MongoClientSettings settings = MongoClientSettings.FromUrl(
        new MongoUrl(APIKeys.ConnectionString)
    );

    settings.SslSettings =
        new SslSettings() { EnabledSslProtocols = SslProtocols.Tls12 };

    settings.RetryWrites = false;

    MongoClient mongoClient = new MongoClient(settings);
    ```

* Recupere una referencia a la base de datos y la colección. El SDK de .NET de MongoDB creará automáticamente tanto la base de datos como la colección si todavía no existen.
    ```cs
    string dbName = "MyTasks";
    string collectionName = "TaskList";

    var db = mongoClient.GetDatabase(dbName);

    var collectionSettings = new MongoCollectionSettings {
        ReadPreference = ReadPreference.Nearest
    };

    tasksCollection = db.GetCollection<MyTask>(collectionName, collectionSettings);
    ```
* Recupere todos los documentos como una lista.
    ```cs
    var allTasks = await TasksCollection
                    .Find(new BsonDocument())
                    .ToListAsync();
    ```

* Consulta de documentos determinados.
    ```cs
    public async Task<List<MyTask>> GetIncompleteTasksDueBefore(DateTime date)
    {
        var tasks = await TasksCollection
                        .AsQueryable()
                        .Where(t => t.Complete == false)
                        .Where(t => t.DueDate < date)
                        .ToListAsync();

        return tasks;
    }
    ```

* Cree una tarea e insértela en la colección.
    ```cs
    public async Task CreateTask(MyTask task)
    {
        await TasksCollection.InsertOneAsync(task);
    }
    ```

* Actualice una tarea de una colección.
    ```cs
    public async Task UpdateTask(MyTask task)
    {
        await TasksCollection.ReplaceOneAsync(t => t.Id.Equals(task.Id), task);
    }
    ```

* Elimine una tarea de una colección.
    ```cs
    public async Task DeleteTask(MyTask task)
    {
        await TasksCollection.DeleteOneAsync(t => t.Id.Equals(task.Id));
    }
    ```

<a id="update-your-connection-string"></a>

## <a name="update-your-connection-string"></a>Actualización de la cadena de conexión

Ahora vuelva a Azure Portal para obtener la información de la cadena de conexión y cópiela en la aplicación.

1. En [Azure Portal](https://portal.azure.com/), en la cuenta de Azure Cosmos DB, en el panel de navegación izquierdo, haga clic en **Cadena de conexión** y en **Claves de lectura y escritura**. Usará los botones de copia que están en el lado derecho de la pantalla para copiar la cadena de conexión principal en los pasos siguientes.

2. Abra el archivo **APIKeys.cs** en el directorio **Helpers** (Aplicaciones auxiliares) del proyecto **TaskList.Core**.

3. Copie el valor de la **cadena de conexión principal** del portal (con el botón de copia) y conviértalo en el valor del campo **ConnectionString** en el archivo **APIKeys.cs**.

4. Quite `&replicaSet=globaldb` de la cadena de conexión. Obtendrá un error en tiempo de ejecución si no quita ese valor de la cadena de consulta.

> [!IMPORTANT]
> Debe quitar el par clave-valor `&replicaSet=globaldb` de la cadena de consulta de la cadena de conexión para evitar un error en tiempo de ejecución.

Ya ha actualizado la aplicación con toda la información que necesita para comunicarse con Azure Cosmos DB.

## <a name="run-the-app"></a>Ejecución la aplicación

### <a name="visual-studio-2019"></a>Visual Studio 2019

1. En Visual Studio, haga clic con el botón derecho en cada proyecto en el **Explorador de soluciones** y, después, haga clic en **Administrar paquetes NuGet**.
2. Haga clic en **Restore all NuGet packages** (Restaurar todos los paquetes NuGet).
3. Haga clic con el botón derecho en **TaskList.Android** y seleccione **Establecer como proyecto de inicio**.
4. Presione F5 para iniciar la depuración de la aplicación.
5. Si quiere ejecutar en iOS, primero asegúrese de que la máquina esté conectada a un equipo Mac (aquí hay algunas [instrucciones](https://docs.microsoft.com/xamarin/ios/get-started/installation/windows/introduction-to-xamarin-ios-for-visual-studio) sobre cómo hacerlo).
6. Haga clic con el botón derecho en el proyecto **TaskList.iOS** y seleccione **Establecer como proyecto de inicio**.
7. Haga clic en F5 para iniciar la depuración de la aplicación.

### <a name="visual-studio-for-mac"></a>Visual Studio para Mac

1. En la lista desplegable de plataforma, seleccione TaskList.iOS o TaskList.Android, en función de la plataforma en que quiere realizar la ejecución.
2. Presione cmd+Entrar para empezar a depurar la aplicación.

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha aprendido a crear una cuenta de Azure Cosmos DB y ejecutar una aplicación Xamarin.Forms con la API para MongoDB. Ahora puede importar datos adicionales en la cuenta de Cosmos DB.

> [!div class="nextstepaction"]
> [Importación de datos en Azure Cosmos DB configurado con la API de Azure Cosmos DB para MongoDB](mongodb-migrate.md)
