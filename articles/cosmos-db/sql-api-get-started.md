---
title: 'Tutorial: Compilación de una aplicación de consola .NET para administrar datos de la cuenta de API de SQL de Azure Cosmos DB'
description: 'Tutorial: Aprenda a crear recursos de SQL API de Azure Cosmos DB mediante una aplicación de consola en C#.'
author: kirankumarkolli
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 11/05/2019
ms.author: kirankk
ms.custom: devx-track-csharp
ms.openlocfilehash: 514fe30da9c0e232c168992c2aabbb484644aa99
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89015312"
---
# <a name="tutorial-build-a-net-console-app-to-manage-data-in-azure-cosmos-db-sql-api-account"></a>Tutorial: Compilación de una aplicación de consola .NET para administrar datos de la cuenta de API de SQL de Azure Cosmos DB

> [!div class="op_single_selector"]
> * [.NET](sql-api-get-started.md)
> * [Java](sql-api-java-get-started.md)
> * [Async Java](sql-api-async-java-get-started.md)
> * [Node.js](sql-api-nodejs-get-started.md)
>

Le damos la bienvenida al tutorial de introducción a la API de SQL de Azure Cosmos DB. Después de seguir este tutorial, tendrá una aplicación de consola que crea recursos de Azure Cosmos DB y los consulta.

En este tutorial se usa la versión 3.0 o posterior del [SDK de Azure Cosmos DB para .NET](https://www.nuget.org/packages/Microsoft.Azure.Cosmos). Puede trabajar con [.NET Framework o .NET Core](https://dotnet.microsoft.com/download).

Esta tutorial abarca lo siguiente:

> [!div class="checklist"]
>
> * Creación de una cuenta de Azure Cosmos DB y conexión a ella
> * Configuración del proyecto en Visual Studio
> * Creación de una base de datos y un contenedor
> * Adición de elementos al contenedor
> * Consulta en el contenedor
> * Operaciones de creación, lectura, actualización y eliminación (CRUD) en el elemento
> * Eliminación de la base de datos

¿No tiene tiempo? ¡No se preocupe! La solución completa está disponible en [GitHub](https://github.com/Azure-Samples/cosmos-dotnet-getting-started). Para obtener instrucciones rápidas, diríjase a la [sección sobre cómo obtener la solución completa del tutorial](#GetSolution).

Comencemos.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure activa. Si no tiene una, puede registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/).

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [!INCLUDE [cosmos-db-emulator-vs](../../includes/cosmos-db-emulator-vs.md)]

## <a name="step-1-create-an-azure-cosmos-db-account"></a>Paso 1: Creación de una cuenta de Azure Cosmos DB

Vamos a crear una cuenta de Azure Cosmos DB. Si ya tiene una cuenta que desea usar, omita esta sección. Para usar el Emulador de Azure Cosmos DB, siga los pasos que se describen en [Emulador de Azure Cosmos DB](local-emulator.md) para configurarlo. Después, vaya al [paso 2: Configuración del proyecto de Visual Studio](#SetupVS).

[!INCLUDE [create-dbaccount-preview](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="step-2-set-up-your-visual-studio-project"></a><a id="SetupVS"></a>Paso 2: Configuración del proyecto de Visual Studio

1. Abra Visual Studio y seleccione **Crear un proyecto**.
1. En **Crear un proyecto**, elija **Aplicación de consola (.NET Framework)** para C# y seleccione **Siguiente**.
1. Asigne al proyecto el nombre *CosmosGettingStartedTutorial* y seleccione **Crear**.

    :::image type="content" source="./media/sql-api-get-started/configure-cosmos-getting-started-2019.png" alt-text="Configuración del proyecto":::

1. En el **Explorador de soluciones**, haga clic con el botón derecho en la nueva aplicación de consola, que se encuentra en la solución de Visual Studio y, a continuación, seleccione **Administrar paquetes NuGet**.
1. En **Administrador de paquetes NuGet**, seleccione **Examinar** y busque *Microsoft.Azure.Cosmos*. Elija **Microsoft.Azure.Cosmos** y seleccione **Instalar**.

   :::image type="content" source="./media/sql-api-get-started/cosmos-getting-started-manage-nuget-2019.png" alt-text="Instalación de NuGet para el SDK de cliente de Azure Cosmos DB":::

   El identificador del paquete de la biblioteca de cliente de la API de SQL de Azure Cosmos DB es [Microsoft Azure Cosmos DB Client Library](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/).

Estupendo. Ahora que hemos terminado la configuración, comencemos a escribir algo de código. Para ver el proyecto completado de este tutorial, consulte [Desarrollo de una aplicación de consola de .NET con Azure Cosmos DB](https://github.com/Azure-Samples/cosmos-dotnet-getting-started).

## <a name="step-3-connect-to-an-azure-cosmos-db-account"></a><a id="Connect"></a>Paso 3: Conexión a una cuenta de Azure Cosmos DB

1. Reemplace las referencias que aparecen al principio de su aplicación de C#, en el archivo *Program.cs*, por estas otras:

   ```csharp
   using System;
   using System.Threading.Tasks;
   using System.Configuration;
   using System.Collections.Generic;
   using System.Net;
   using Microsoft.Azure.Cosmos;
   ```

1. Agregue estas constantes y variables a la clase `Program`.

    ```csharp
    public class Program
    {
        // ADD THIS PART TO YOUR CODE

        // The Azure Cosmos DB endpoint for running this sample.
        private static readonly string EndpointUri = "<your endpoint here>";
        // The primary key for the Azure Cosmos account.
        private static readonly string PrimaryKey = "<your primary key>";

        // The Cosmos client instance
        private CosmosClient cosmosClient;

        // The database we will create
        private Database database;

        // The container we will create.
        private Container container;

        // The name of the database and container we will create
        private string databaseId = "FamilyDatabase";
        private string containerId = "FamilyContainer";
    }
    ```

   > [!NOTE]
   > Si ya está familiarizado con la versión anterior del SDK de .NET, puede que conozca los términos *colección* y *documento*. Dado que Azure Cosmos DB admite varios modelos de API, todas las versiones del SDK de .NET posteriores a la 3.0 usan los términos genéricos *contenedor* y *elemento*. Un *contenedor* puede ser una colección, un grafo o una tabla. Un *elemento* puede ser un documento, un vértice o borde, o una fila, y es el contenido que hay en un contenedor. Para más información, consulte [Trabajo con bases de datos, contenedores y elementos en Azure Cosmos DB](databases-containers-items.md).

1. Abra [Azure Portal](https://portal.azure.com). Busque la cuenta de Azure Cosmos DB y seleccione **Claves**.

   :::image type="content" source="./media/sql-api-get-started/cosmos-getting-started-portal-keys.png" alt-text="Obtención de las claves de Azure Cosmos DB en Azure Portal":::

1. En *Program.cs*, reemplace `<your endpoint URL>` por el valor de **URI**. Reemplace `<your primary key>` por el valor de **CLAVE PRINCIPAL**.

1. Debajo del método **Main**, agregue una nueva tarea asincrónica llamada **GetStartedDemoAsync**, que crea una instancia del nuevo `CosmosClient`.

    ```csharp
    public static async Task Main(string[] args)
    {
    }

    // ADD THIS PART TO YOUR CODE
    /*
        Entry point to call methods that operate on Azure Cosmos DB resources in this sample
    */
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
    }
    ```

    **GetStartedDemoAsync** se usa como punto de entrada que llama a los métodos que operan en recursos de Azure Cosmos DB.

1. Agregue el código siguiente para ejecutar la tarea asincrónica **GetStartedDemoAsync** desde su método **Main**. El método **Main** detecta las excepciones y las escribe en la consola.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=Main)]

1. Presione F5 para ejecutar la aplicación.

    La consola muestra el mensaje: **Final de la demo, presione cualquier tecla para salir.** Este mensaje confirma que la aplicación realizó una conexión con Azure Cosmos DB. A continuación, puede cerrar la ventana de consola.

Felicidades. Se ha conectado correctamente a una cuenta de Azure Cosmos DB.

## <a name="step-4-create-a-database"></a>Paso 4: Crear una base de datos

Una base de datos es el contenedor lógico de elementos con particiones en contenedores. El método `CreateDatabaseIfNotExistsAsync` o `CreateDatabaseAsync` de la clase [CosmosClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient) puede crear una base de datos.

1. Copie y pegue el método `CreateDatabaseAsync` debajo del método `GetStartedDemoAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=CreateDatabaseAsync&highlight=7)]

    `CreateDatabaseAsync` crea una nueva base de datos con el identificador `FamilyDatabase`, si aún no existe, utilizando el identificador especificado en el campo `databaseId`.

1. Copie y pegue el código siguiente donde se crea una instancia de CosmosClient para llamar al método **CreateDatabase** que acaba de agregar.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);

        //ADD THIS PART TO YOUR CODE
        await this.CreateDatabaseAsync();
    }
    ```

    El archivo *Program.cs* debe tener ahora el siguiente aspecto, con el punto de conexión y la clave principal rellenados.

    ```csharp
    using System;
    using System.Threading.Tasks;
    using System.Configuration;
    using System.Collections.Generic;
    using System.Net;
    using Microsoft.Azure.Cosmos;

    namespace CosmosGettingStartedTutorial
    {
        class Program
        {
            // The Azure Cosmos DB endpoint for running this sample.
            private static readonly string EndpointUri = "<your endpoint here>";
            // The primary key for the Azure Cosmos account.
            private static readonly string PrimaryKey = "<your primary key>";

            // The Cosmos client instance
            private CosmosClient cosmosClient;

            // The database we will create
            private Database database;

            // The container we will create.
            private Container container;

            // The name of the database and container we will create
            private string databaseId = "FamilyDatabase";
            private string containerId = "FamilyContainer";

            public static async Task Main(string[] args)
            {
                try
                {
                    Console.WriteLine("Beginning operations...");
                    Program p = new Program();
                    await p.GetStartedDemoAsync();
                }
                catch (CosmosException de)
                {
                    Exception baseException = de.GetBaseException();
                    Console.WriteLine("{0} error occurred: {1}\n", de.StatusCode, de);
                }
                catch (Exception e)
                {
                    Console.WriteLine("Error: {0}\n", e);
                }
                finally
                {
                    Console.WriteLine("End of demo, press any key to exit.");
                    Console.ReadKey();
                }
            }

            /// <summary>
            /// Entry point to call methods that operate on Azure Cosmos DB resources in this sample
            /// </summary>
            public async Task GetStartedDemoAsync()
            {
                // Create a new instance of the Cosmos Client
                this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
                await this.CreateDatabaseAsync();
            }

            /// <summary>
            /// Create the database if it does not exist
            /// </summary>
            private async Task CreateDatabaseAsync()
            {
                // Create a new database
                this.database = await this.cosmosClient.CreateDatabaseIfNotExistsAsync(databaseId);
                Console.WriteLine("Created Database: {0}\n", this.database.Id);
            }
        }
    }
    ```

1. Presione F5 para ejecutar la aplicación.

   > [!NOTE]
   > Si recibe una "excepción 503 de servicio no disponible", es posible que los [puertos](performance-tips.md#networking) necesarios para el modo de conexión directa estén bloqueados por un firewall. Para corregir este problema, abra los puertos necesarios o use la conectividad del modo de puerta de enlace como se indica en el código siguiente:
   ```csharp
     // Create a new instance of the Cosmos Client in Gateway mode
     this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey, new CosmosClientOptions()
            {
                ConnectionMode = ConnectionMode.Gateway
            });
   ```

Felicidades. Ha creado correctamente una base de datos de Azure Cosmos.  

## <a name="step-5-create-a-container"></a><a id="CreateColl"></a>Paso 5: Crear un contenedor

> [!WARNING]
> El método `CreateContainerIfNotExistsAsync` crea un nuevo contenedor, lo que tiene implicaciones en el precio. Para obtener más información, visite nuestra [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/).
>
>

Un contenedor se puede crear con los métodos [**CreateContainerIfNotExistsAsync**](/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync?view=azure-dotnet#Microsoft_Azure_Cosmos_Database_CreateContainerIfNotExistsAsync_Microsoft_Azure_Cosmos_ContainerProperties_System_Nullable_System_Int32__Microsoft_Azure_Cosmos_RequestOptions_System_Threading_CancellationToken_) o [**CreateContainerAsync**](/dotnet/api/microsoft.azure.cosmos.database.createcontainerasync?view=azure-dotnet#Microsoft_Azure_Cosmos_Database_CreateContainerAsync_Microsoft_Azure_Cosmos_ContainerProperties_System_Nullable_System_Int32__Microsoft_Azure_Cosmos_RequestOptions_System_Threading_CancellationToken_) en la clase `CosmosDatabase`. Un contenedor se compone de elementos (que en el caso de SQL API son documentos JSON) y una lógica de aplicación del servidor en JavaScript asociada, como por ejemplo procedimientos almacenados, funciones definidas por el usuario y desencadenadores.

1. Copie y pegue el método `CreateContainerAsync` debajo del método `CreateDatabaseAsync`. `CreateContainerAsync` crea un nuevo contenedor con el identificador `FamilyContainer`, si aún no existe, utilizando el identificador especificado en el campo `containerId` particionado por la propiedad `LastName`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=CreateContainerAsync&highlight=9)]

1. Copie y pegue el código siguiente donde creó una instancia de CosmosClient para llamar al método **CreateContainer** que acaba de agregar.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();

        //ADD THIS PART TO YOUR CODE
        await this.CreateContainerAsync();
    }
    ```

1. Presione F5 para ejecutar la aplicación.

Felicidades. Ha creado correctamente un contenedor de Azure Cosmos.  

## <a name="step-6-add-items-to-the-container"></a><a id="CreateDoc"></a>Paso 6: Agregar elementos al contenedor

El método [**CreateItemAsync**](/dotnet/api/microsoft.azure.cosmos.container.createitemasync?view=azure-dotnet#Microsoft_Azure_Cosmos_Container_CreateItemAsync__1___0_System_Nullable_Microsoft_Azure_Cosmos_PartitionKey__Microsoft_Azure_Cosmos_ItemRequestOptions_System_Threading_CancellationToken_) de la clase `CosmosContainer` puede crear un elemento. Cuando se usa SQL API, los elementos se proyectan como documentos, que son contenido JSON arbitrario definido por el usuario. Ya puede insertar un elemento en el contenedor de Azure Cosmos.

En primer lugar, vamos a crear una clase llamada `Family` que representará los objetos almacenados en Azure Cosmos DB en este ejemplo. También crearemos las subclases `Parent`, `Child`, `Pet`, `Address` que se usan en `Family`. El elemento debe tener una propiedad `Id` serializada como `id` en JSON.

1. Seleccione Ctrl+Mayús+A para abrir **Agregar nuevo elemento**. Agregue una nueva clase `Family.cs` al proyecto.

    :::image type="content" source="./media/sql-api-get-started/cosmos-getting-started-add-family-class-2019.png" alt-text="Captura de pantalla de la adición de una nueva clase de Family.cs al proyecto":::

1. Copie y pegue las clases `Family`, `Parent`, `Child`, `Pet` y `Address` a `Family.cs`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Family.cs)]


1. De nuevo en *Program.cs*, agregue el método `AddItemsToContainerAsync` después del método `CreateContainerAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=AddItemsToContainerAsync)]


    El código comprueba que no exista un elemento con el mismo identificador. Se insertan dos elementos, uno para *Andersen Family* y otro para *Wakefield Family*.

1. Agregue una llamada a `AddItemsToContainerAsync` en el método `GetStartedDemoAsync`.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();

        //ADD THIS PART TO YOUR CODE
        await this.AddItemsToContainerAsync();
    }
    ```

1. Presione F5 para ejecutar la aplicación.

Felicidades. Ha creado correctamente dos elementos de Azure Cosmos.  

## <a name="step-7-query-azure-cosmos-db-resources"></a><a id="Query"></a>Paso 7: Consulta de los recursos de Azure Cosmos DB

Azure Cosmos DB admite consultas enriquecidas en los documentos JSON que se almacenan en los contenedores. Para obtener más información, consulte [Introducción a las consultas SQL](sql-api-sql-query.md). El ejemplo de código siguiente muestra cómo ejecutar una consulta en los elementos que se insertaron en el paso anterior.

1. Copie y pegue el método `QueryItemsAsync` después del método `AddItemsToContainerAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=QueryItemsAsync&highlight=10-11,17-18)]

1. Agregue una llamada a ``QueryItemsAsync`` en el método ``GetStartedDemoAsync``.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();
        await this.AddItemsToContainerAsync();

        //ADD THIS PART TO YOUR CODE
        await this.QueryItemsAsync();
    }
    ```

1. Presione F5 para ejecutar la aplicación.

Felicidades. Ha consultado correctamente un contenedor de Azure Cosmos.

## <a name="step-8-replace-a-json-item"></a><a id="ReplaceItem"></a>Paso 8: Sustitución de un elemento JSON

Ahora, actualizaremos un elemento en Azure Cosmos DB. Cambiaremos la propiedad `IsRegistered` de `Family` y la propiedad `Grade` de uno de los elementos secundarios.

1. Copie y pegue el método `ReplaceFamilyItemAsync` después del método `QueryItemsAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=ReplaceFamilyItemAsync&highlight=15)]

1. Agregue una llamada a `ReplaceFamilyItemAsync` en el método `GetStartedDemoAsync`.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();
        await this.AddItemsToContainerAsync();
        await this.QueryItemsAsync();

        //ADD THIS PART TO YOUR CODE
        await this.ReplaceFamilyItemAsync();
    }
    ```

1. Presione F5 para ejecutar la aplicación.

Felicidades. Ha sustituido correctamente un elemento de Azure Cosmos.

## <a name="step-9-delete-item"></a><a id="DeleteDocument"></a>Paso 9: Eliminar elemento

Ahora, eliminaremos un elemento en Azure Cosmos DB.

1. Copie y pegue el método `DeleteFamilyItemAsync` después del método `ReplaceFamilyItemAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=DeleteFamilyItemAsync&highlight=10)]

1. Agregue una llamada a `DeleteFamilyItemAsync` en el método `GetStartedDemoAsync`.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();
        await this.AddItemsToContainerAsync();
        await this.QueryItemsAsync();
        await this.ReplaceFamilyItemAsync();

        //ADD THIS PART TO YOUR CODE
        await this.DeleteFamilyItemAsync();
    }
    ```

1. Presione F5 para ejecutar la aplicación.

Felicidades. Ha eliminado correctamente un elemento de Azure Cosmos.

## <a name="step-10-delete-the-database"></a><a id="DeleteDatabase"></a>Paso 10: Eliminación de la base de datos

Ahora eliminaremos nuestra base de datos. Al eliminar la base de datos creada, se quitan la base de datos y todos los recursos secundarios. Los recursos incluyen contenedores, elementos y cualquier procedimiento almacenado, funciones definidas por el usuario y desencadenadores. También eliminamos la instancia `CosmosClient`.

1. Copie y pegue el método `DeleteDatabaseAndCleanupAsync` después del método `DeleteFamilyItemAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=DeleteDatabaseAndCleanupAsync)]

1. Agregue una llamada a ``DeleteDatabaseAndCleanupAsync`` en el método ``GetStartedDemoAsync``.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=GetStartedDemoAsync&highlight=14)]

1. Presione F5 para ejecutar la aplicación.

Felicidades. Ha eliminado correctamente una base de datos de Azure Cosmos.

## <a name="step-11-run-your-c-console-application-all-together"></a><a id="Run"></a>Paso 11: Ejecutar la aplicación de consola de C#

Presione F5 en Visual Studio para compilar y ejecutar la aplicación en modo de depuración.

Verá la salida de la aplicación entera en una ventana de consola. La salida muestra los resultados de las consultas que hemos agregado. Debe coincidir con el texto de ejemplo siguiente.

```cmd
Beginning operations...

Created Database: FamilyDatabase

Created Container: FamilyContainer

Created item in database with id: Andersen.1 Operation consumed 11.43 RUs.

Created item in database with id: Wakefield.7 Operation consumed 14.29 RUs.

Running query: SELECT * FROM c WHERE c.LastName = 'Andersen'

        Read {"id":"Andersen.1","LastName":"Andersen","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null,"FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":5,"Pets":[{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":false}

Updated Family [Wakefield,Wakefield.7].
        Body is now: {"id":"Wakefield.7","LastName":"Wakefield","Parents":[{"FamilyName":"Wakefield","FirstName":"Robin"},{"FamilyName":"Miller","FirstName":"Ben"}],"Children":[{"FamilyName":"Merriam","FirstName":"Jesse","Gender":"female","Grade":6,"Pets":[{"GivenName":"Goofy"},{"GivenName":"Shadow"}]},{"FamilyName":"Miller","FirstName":"Lisa","Gender":"female","Grade":1,"Pets":null}],"Address":{"State":"NY","County":"Manhattan","City":"NY"},"IsRegistered":true}

Deleted Family [Wakefield,Wakefield.7]

Deleted Database: FamilyDatabase

End of demo, press any key to exit.
```

Felicidades. Ha completado este tutorial y tiene una aplicación de consola de C# en funcionamiento.

## <a name="get-the-complete-tutorial-solution"></a><a id="GetSolution"></a> Obtención de la solución completa del tutorial

Puede descargar este tutorial si no tuvo tiempo para completar los pasos o simplemente desea descargar los ejemplos de código.

Para compilar la solución `GetStarted`, necesita los siguientes requisitos previos:

* Una cuenta de Azure activa. Si no tiene una, puede registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una [cuenta de Azure Cosmos DB][cosmos-db-create-account].
* La solución [GetStarted](https://github.com/Azure-Samples/cosmos-dotnet-getting-started) está disponible en GitHub.

Para restaurar las referencias al SDK de Azure Cosmos DB para .NET en Visual Studio, haga clic con el botón derecho en la solución en el **Explorador de soluciones** y, después, seleccione **Restaurar paquetes NuGet**. Después, en el archivo *App.config*, actualice los valores de `EndPointUri` y `PrimaryKey`, tal y como se describe en el [paso 3: Conexión a una cuenta de Azure Cosmos DB](#Connect).

Y, eso es todo, compílelo y habrá terminado.

## <a name="next-steps"></a>Pasos siguientes

* ¿Desea un tutorial de ASP.NET MVC más complejo? Consulte [Tutorial: Desarrollar una aplicación web ASP.NET Core MVC con Azure Cosmos DB mediante el SDK de .NET](sql-api-dotnet-application.md).
* ¿Quiere realizar pruebas de escalado y rendimiento con Azure Cosmos DB? Consulte [Pruebas de escala y rendimiento con Azure Cosmos DB](performance-testing.md).
* Para obtener información sobre cómo supervisar las solicitudes, el uso y el almacenamiento de Azure Cosmos DB, consulte [Supervisión de las métricas de rendimiento y almacenamiento en Azure Cosmos DB](monitor-accounts.md).
* Para ejecutar consultas en nuestro conjunto de datos de ejemplo, visite nuestro [Sitio de prueba de consultas](https://www.documentdb.com/sql/demo).
* Para más información sobre Azure Cosmos DB, consulte [Bienvenido a Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/introduction).

[cosmos-db-create-account]: create-sql-api-java.md#create-a-database-account
