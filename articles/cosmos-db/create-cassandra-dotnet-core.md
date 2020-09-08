---
title: 'Inicio rápido: Cassandra API con .NET Core: Azure Cosmos DB'
description: En este inicio rápido se muestra cómo usar Cassandra API de Azure Cosmos DB para crear una aplicación de perfil con Azure Portal y .NET Core
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
author: TheovanKraay
ms.author: thvankra
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 05/18/2020
ms.custom: devx-track-dotnet
ms.openlocfilehash: d24116701347c7d4b60494219b56877e0b121ea3
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89017476"
---
# <a name="quickstart-build-a-cassandra-app-with-net-core-and-azure-cosmos-db"></a>Inicio rápido: Compilación de una aplicación de Cassandra con .NET Core y Azure Cosmos DB

> [!div class="op_single_selector"]
> * [.NET](create-cassandra-dotnet.md)
> * [.NET Core](create-cassandra-dotnet-core.md)
> * [Java v3](create-cassandra-java.md)
> * [Java v4](create-cassandra-java-v4.md)
> * [Node.js](create-cassandra-nodejs.md)
> * [Python](create-cassandra-python.md)
>  

En esta guía de inicio rápido se muestra cómo usar .NET Core y [Cassandra API](cassandra-introduction.md) de Azure Cosmos DB para compilar una aplicación de perfil mediante la clonación de un ejemplo de GitHub. En esta guía de inicio rápido también se muestra cómo usar Azure Portal basado en web para crear una cuenta de Azure Cosmos DB.

Azure Cosmos DB es un servicio de base de datos con varios modelos y de distribución global de Microsoft. Puede crear rápidamente bases de datos de documentos, tablas, clave-valor y grafos, y realizar consultas en ellas. Todas las bases de datos se beneficiarán de las funcionalidades de distribución global y escala horizontal en Azure Cosmos DB. 

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] También puede [probar gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin suscripción de Azure, sin cargos y sin compromiso.

Además, necesita: 
* Si no tiene Visual Studio 2019 instalado, puede descargar y usar la **versión gratuita** de [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Asegúrese de que habilita **Desarrollo de Azure** durante la instalación de Visual Studio.
* Instale [Git](https://www.git-scm.com/) para clonar el ejemplo.

<a id="create-account"></a>
## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

[!INCLUDE [cosmos-db-create-dbaccount-cassandra](../../includes/cosmos-db-create-dbaccount-cassandra.md)]


## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ahora vamos a empezar a trabajar con el código. Vamos a clonar una aplicación de la API de Cassandra desde GitHub, establecer la cadena de conexión y ejecutarla. Verá lo fácil que es trabajar con datos mediante programación. 

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
    git clone https://github.com/Azure-Samples/azure-cosmos-db-cassandra-dotnet-core-getting-started.git
    ```

4. Luego abra el archivo de la solución CassandraQuickStartSample en Visual Studio. 

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si le interesa aprender cómo el código crea los recursos de base de datos, puede revisar los siguientes fragmentos de código. Todos los fragmentos de código se toman del archivo `Program.cs` del método `async Task ProcessAsync()`, que está instalado en la carpeta `C:\git-samples\azure-cosmos-db-cassandra-dotnet-core-getting-started\CassandraQuickStart`. En caso contrario, puede ir directamente a [Actualización de la cadena de conexión](#update-your-connection-string).

* Inicialice la sesión mediante la conexión a un punto de conexión del clúster de Cassandra. La API de Cassandra API en Azure Cosmos DB solo admite TLSv1.2. 

  ```csharp
      var options = new Cassandra.SSLOptions(SslProtocols.Tls12, true, ValidateServerCertificate);
      options.SetHostNameResolver((ipAddress) => CASSANDRACONTACTPOINT);
      Cluster cluster = Cluster
          .Builder()
          .WithCredentials(USERNAME, PASSWORD)
          .WithPort(CASSANDRAPORT)
          .AddContactPoint(CASSANDRACONTACTPOINT)
          .WithSSL(options)
          .Build()
      ;
      ISession session = await cluster.ConnectAsync();
   ```

* Quite el espacio de claves existente, en caso de que exista.

    ```csharp
    await session.ExecuteAsync(new SimpleStatement("DROP KEYSPACE IF EXISTS uprofile")); 
    ```

* Cree un nuevo espacio de claves.

    ```csharp
    await session.ExecuteAsync(new SimpleStatement("CREATE KEYSPACE uprofile WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 1 };"));
    ```

* Cree una nueva tabla.

   ```csharp
  await session.ExecuteAsync(new SimpleStatement("CREATE TABLE IF NOT EXISTS uprofile.user (user_id int PRIMARY KEY, user_name text, user_bcity text)"));
   ```

* Inserte las entidades de usuario mediante el uso del objeto IMapper con una sesión nueva que se conecta al espacio de claves del perfil.

    ```csharp
    await mapper.InsertAsync<User>(new User(1, "LyubovK", "Dubai"));
    ```

* Realice una consulta para obtener la información de todos los usuarios.

    ```csharp
    foreach (User user in await mapper.FetchAsync<User>("Select * from user"))
    {
        Console.WriteLine(user);
    }
    ```

* Realice una consulta para obtener la información de un único usuario.

    ```csharp
    mapper.FirstOrDefault<User>("Select * from user where user_id = ?", 3);
    ```

## <a name="update-your-connection-string"></a>Actualización de la cadena de conexión

Ahora vuelva a Azure Portal para obtener la información de la cadena de conexión y cópiela en la aplicación. La información de la cadena de conexión permite que la aplicación se comunique con la base de datos hospedada.

1. En [Azure Portal](https://portal.azure.com/) seleccione **Cadena de conexión**.

1. Utilice el botón :::image type="icon" source="./media/create-cassandra-dotnet/copy.png"::: del lado derecho de la pantalla para copiar el valor de USERNAME.

   :::image type="content" source="./media/create-cassandra-dotnet/keys.png" alt-text="Visualizar y copiar una clave de acceso en Azure Portal, página Cadena de conexión":::

1. En Visual Studio, abra el archivo Program.cs. 

1. Pegue el valor de USERNAME del portal sobre `<PROVIDE>` en la línea 13.

    La línea 13 de Program.cs debe tener ahora un aspecto similar a 

    `private const string UserName = "cosmos-db-quickstart";`

    También puede pegar el mismo valor en `<PROVIDE>` en la línea 15 para el valor de CONTACT POINT:

    `private const string CassandraContactPoint = "cosmos-db-quickstarts.cassandra.cosmosdb.azure.com"; //  DnsName`

1. Vuelva al portal y copie el valor de PASSWORD. Pegue el valor de PASSWORD del portal sobre `<PROVIDE>` en la línea 14.

    La línea 14 de Program.cs debe tener ahora un aspecto similar a 

    `private const string Password = "2Ggkr662ifxz2Mg...==";`

1. Guarde el archivo Program.cs.
    
## <a name="run-the-net-core-app"></a>Ejecución de la aplicación .NET Core

1. En Visual Studio, seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del administrador de paquetes**.

2. En el símbolo del sistema, use el comando siguiente para instalar el paquete NuGet del controlador .NET. 

    ```cmd
    Install-Package CassandraCSharpDriver
    ```
3. Presione CTRL+F5 para ejecutar la aplicación. La aplicación aparece en la ventana de la consola. 

    :::image type="content" source="./media/create-cassandra-dotnet/output.png" alt-text="Visualizar y comprobar el resultado":::

    Presione CTRL+C para detener la ejecución del programa y cerrar la ventana de la consola. 
    
4. En Azure Portal abra **Explorador de datos** para consultar, modificar y trabajar con estos nuevos datos.

    :::image type="content" source="./media/create-cassandra-dotnet/data-explorer.png" alt-text="Visualizar los datos en el Explorador de datos":::

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido ha aprendido a crear una cuenta de Azure Cosmos DB, a crear un contenedor mediante el Explorador de datos y a ejecutar una aplicación. Ahora puede importar datos adicionales en la cuenta de Cosmos DB. 

> [!div class="nextstepaction"]
> [Importación de datos de Cassandra en Azure Cosmos DB](cassandra-import-data.md)
