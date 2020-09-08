---
title: 'Inicio rápido: Compilación de una aplicación de consola .NET para administrar recursos de la API de SQL de Azure Cosmos DB'
description: En este inicio rápido, aprenderá a compilar una aplicación de consola .NET para administrar recursos de cuenta de la API de SQL de Azure Cosmos DB.
author: anfeldma-ms
ms.author: anfeldma
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 05/11/2020
ms.custom: devx-track-dotnet
ms.openlocfilehash: c76a6666be805aa088bab7c5716ffd88a30519c1
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002086"
---
# <a name="quickstart-build-a-net-console-app-to-manage-azure-cosmos-db-sql-api-resources"></a>Inicio rápido: Compilar una aplicación de consola .NET para administrar recursos de API de SQL de Azure Cosmos DB

> [!div class="op_single_selector"]
> * [.NET V3](create-sql-api-dotnet.md)
> * [.NET V4](create-sql-api-dotnet-V4.md)
> * [SDK para Java v4](create-sql-api-java.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

Introducción a la biblioteca cliente de API de SQL de Azure Cosmos DB para .NET. Siga los pasos de este documento para instalar el paquete .NET, compilar una aplicación y probar el código de ejemplo para las operaciones CRUD básicas en los datos almacenados en Azure Cosmos DB. 

Azure Cosmos DB es un servicio de base de datos con varios modelos y de distribución global de Microsoft. Puede usar Azure Cosmos DB para crear y consultar rápidamente las bases de datos de gráficos, documentos, claves y valores. Use la biblioteca cliente de API de SQL de Azure Cosmos DB para .NET para:

* Crear una base de datos y un contenedor de Azure Cosmos
* Agregar datos de ejemplo al contenedor
* Consultar los datos 
* Eliminación de la base de datos

[Documentación de referencia de la API](/dotnet/api/microsoft.azure.cosmos?view=azure-dotnet) | [Código fuente de la biblioteca](https://github.com/Azure/azure-cosmos-dotnet-v3) | [Paquete (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.Cosmos)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción de Azure: [cree una de forma gratuita](https://azure.microsoft.com/free/) o [pruebe Azure Cosmos DB de forma gratuita](https://azure.microsoft.com/try/cosmosdb/) sin necesidad de tener una suscripción de Azure, sin cargos ni compromiso. 
* [SDK de .NET Core 2.1 o posterior](https://dotnet.microsoft.com/download/dotnet-core/2.1).

## <a name="setting-up"></a>Instalación

En esta sección se le guía por el proceso de creación de una cuenta de Azure Cosmos y de configuración de un proyecto que usa una biblioteca cliente de API de SQL de Azure Cosmos DB para .NET para administrar recursos. El código de ejemplo descrito en este artículo crea una base de datos de `FamilyDatabase` y miembros de la familia (cada miembro de la familia es un elemento) en la base de datos. Cada miembro de la familia tiene propiedades como `Id, FamilyName, FirstName, LastName, Parents, Children, Address,`. La propiedad `LastName` se usa como clave de partición para el contenedor. 

### <a name="create-an-azure-cosmos-account"></a><a id="create-account"></a>Crear una cuenta de Azure Cosmos

Si usa la opción [Prueba gratuita del emulador de Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) para crear una cuenta de Azure Cosmos, debe crear una cuenta de Azure Cosmos DB del tipo de **SQL API**. Ya se ha creado una cuenta de prueba de Azure Cosmos DB. No tiene que crear la cuenta explícitamente, por lo que puede omitir esta sección y pasar a la siguiente.

Si tiene su propia suscripción de Azure o creó una suscripción gratuita, debe crear una cuenta de Azure Cosmos explícitamente. El código siguiente creará una cuenta de Azure Cosmos con coherencia de sesión. La cuenta se replica en `South Central US` y `North Central US`.  

Puede usar Azure Cloud Shell para crear la cuenta de Azure Cosmos. Azure Cloud Shell es un shell interactivo, autenticado y al que se puede acceder desde un explorador para administrar recursos de Azure. Ofrece la flexibilidad de poder elegir la experiencia de shell que mejor se adapte a la forma de trabajar de cada uno, Bash o PowerShell. En esta guía de inicio rápido, elija el modo **Bash**. Azure Cloud Shell también requiere una cuenta de almacenamiento; puede crear una cuando se le solicite.

Seleccione el botón **Pruébelo** junto al siguiente código, elija el modo **Bash**, seleccione **Crear una cuenta de almacenamiento** e inicie sesión en Cloud Shell. Después, copie y pegue el código siguiente en Azure Cloud Shell y ejecútelo. El nombre de la cuenta de Azure Cosmos debe ser único globalmente; asegúrese de actualizar el valor de `mysqlapicosmosdb` antes de ejecutar el comando.

```azurecli-interactive

# Set variables for the new SQL API account, database, and container
resourceGroupName='myResourceGroup'
location='southcentralus'

# The Azure Cosmos account name must be globally unique, make sure to update the `mysqlapicosmosdb` value before you run the command
accountName='mysqlapicosmosdb'

# Create a resource group
az group create \
    --name $resourceGroupName \
    --location $location

# Create a SQL API Cosmos DB account with session consistency and multi-master enabled
az cosmosdb create \
    --resource-group $resourceGroupName \
    --name $accountName \
    --kind GlobalDocumentDB \
    --locations regionName="South Central US" failoverPriority=0 --locations regionName="North Central US" failoverPriority=1 \
    --default-consistency-level "Session" \
    --enable-multiple-write-locations true

```

La creación de la cuenta de Azure Cosmos tarda un rato;, una vez que la operación se realiza correctamente, puede ver la salida de confirmación. Una vez que el comando se complete correctamente, inicie sesión en [Azure Portal](https://portal.azure.com/) y compruebe que existe la cuenta de Azure Cosmos con el nombre especificado. Puede cerrar la ventana de Azure Cloud Shell una vez creado el recurso. 

### <a name="create-a-new-net-app"></a><a id="create-dotnet-core-app"></a>Crear una nueva aplicación .NET

Cree una nueva aplicación .NET en el IDE o editor que prefiera. Abra el símbolo del sistema de Windows o una ventana de Terminal desde el equipo local. Ejecutará todos los comandos de las secciones siguientes desde el símbolo del sistema o desde el terminal.  Ejecute el comando dotnet new para crear una nueva aplicación llamada `todo`. El parámetro --langVersion establece la propiedad LangVersion en el archivo de proyecto creado.

```console
dotnet new console --langVersion 7.1 -n todo
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```console
cd todo
dotnet build
```

El resultado esperado de la compilación debe parecerse a lo siguiente:

```console
  Restore completed in 100.37 ms for C:\Users\user1\Downloads\CosmosDB_Samples\todo\todo.csproj.
  todo -> C:\Users\user1\Downloads\CosmosDB_Samples\todo\bin\Debug\netcoreapp2.2\todo.dll
  todo -> C:\Users\user1\Downloads\CosmosDB_Samples\todo\bin\Debug\netcoreapp2.2\todo.Views.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:34.17
```

### <a name="install-the-azure-cosmos-db-package"></a><a id="install-package"></a>Instalar el paquete de Azure Cosmos DB

Mientras sigue en el directorio de aplicaciones, instale el paquete de la biblioteca cliente de Azure Cosmos DB para .NET Core con el comando dotnet add package.

```console
dotnet add package Microsoft.Azure.Cosmos
```

### <a name="copy-your-azure-cosmos-account-credentials-from-the-azure-portal"></a>Copiar las credenciales de la cuenta de Azure Cosmos desde Azure Portal

La aplicación de ejemplo debe autenticarse para la cuenta de Azure Cosmos. Para realizar la autenticación, debe pasar las credenciales de cuenta de Azure Cosmos a la aplicación. Para obtener las credenciales de cuenta de Azure Cosmos, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Vaya a la cuenta de Azure Cosmos.

1. Abra el panel **Claves** y copia el **URI** y la **CLAVE PRINCIPAL** de la cuenta. En el paso siguiente, agregará los valores de URI y claves a una variable de entorno.

### <a name="set-the-environment-variables"></a>Establecer las variables de entorno

Una vez que haya copiado el **URI** y la **CLAVE PRINCIPAL** de la cuenta, guárdelos en una variable de entorno nueva en la máquina local que ejecuta la aplicación. Para establecer la variable de entorno, abra una ventana de consola y ejecute el siguiente comando. Asegúrese de reemplazar los valores `<Your_Azure_Cosmos_account_URI>` y `<Your_Azure_Cosmos_account_PRIMARY_KEY>`.

**Windows**

```console
setx EndpointUrl "<Your_Azure_Cosmos_account_URI>"
setx PrimaryKey "<Your_Azure_Cosmos_account_PRIMARY_KEY>"
```

**Linux**

```bash
export EndpointUrl = "<Your_Azure_Cosmos_account_URI>"
export PrimaryKey = "<Your_Azure_Cosmos_account_PRIMARY_KEY>"
```

**macOS**

```bash
export EndpointUrl = "<Your_Azure_Cosmos_account_URI>"
export PrimaryKey = "<Your_Azure_Cosmos_account_PRIMARY_KEY>"
```

 ## <a name="object-model"></a><a id="object-model"></a>Modelo de objetos

Antes de empezar a compilar la aplicación, consulte la jerarquía de recursos de Azure Cosmos DB y el modelo de objetos usado para crear estos recursos y obtener acceso a ellos. Azure Cosmos DB crea recursos en el orden siguiente:

* Cuenta de Azure Cosmos 
* Bases de datos 
* Contenedores 
* Elementos

Para obtener más información acerca de la jerarquía de diferentes entidades, consulte el artículo sobre cómo [trabajar con bases de datos, contenedores y elementos en Azure Cosmos DB](databases-containers-items.md). Usará las siguientes clases de .NET para interactuar con estos recursos:

* [CosmosClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient?view=azure-dotnet): esta clase proporciona una representación lógica del lado cliente para el servicio Azure Cosmos DB. El objeto de cliente se usa para configurar y ejecutar solicitudes en el servicio.

* [CreateDatabaseIfNotExistsAsync](/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync?view=azure-dotnet): este método crea (si no existe) u obtiene (si ya existe) un recurso de base de datos como operación asincrónica. 

* [CreateContainerIfNotExistsAsync](/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync?view=azure-dotnet): este método crea (si no existe) u obtiene (si ya existe) un contenedor como operación asincrónica. Puede comprobar el código de estado de la respuesta para determinar si el contenedor se creó recientemente (201) o si se devolvió un contenedor existente (200). 
* [CreateItemAsync](/dotnet/api/microsoft.azure.cosmos.container.createitemasync?view=azure-dotnet): este método crea un elemento en el contenedor. 

* [UpsertItemAsync](/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync?view=azure-dotnet): este método crea un elemento en el contenedor si aún no existe o lo reemplaza si ya existe. 

* [GetItemQueryIterator](/dotnet/api/microsoft.azure.cosmos.container.GetItemQueryIterator?view=azure-dotnet
): este método crea una consulta para los elementos de un contenedor en una base de datos de Azure Cosmos mediante una instrucción SQL con valores con parámetros. 

* [DeleteAsync](/dotnet/api/microsoft.azure.cosmos.database.deleteasync?view=azure-dotnet): elimina la base de datos especificada de la cuenta de Azure Cosmos. El método `DeleteAsync` solo elimina la base de datos. La eliminación de la instancia `Cosmosclient` debe producirse por separado (lo que ocurre en el método DeleteDatabaseAndCleanupAsync). 

 ## <a name="code-examples"></a><a id="code-examples"></a>Ejemplos de código

En el código de ejemplo que se describe en este artículo, se crea una base de datos de familia en Azure Cosmos DB. La base de datos de la familia contiene detalles de la familia como el nombre, la dirección, la ubicación, los padres, los hijos y las mascotas. Antes de rellenar los datos en su cuenta de Azure Cosmos, defina las propiedades de un elemento de familia. Cree una nueva clase denominada `Family.cs` en el nivel raíz de la aplicación de ejemplo y agréguele el código siguiente:

```csharp
using Newtonsoft.Json;

namespace todo
{
    public class Family
    {
        [JsonProperty(PropertyName = "id")]
        public string Id { get; set; }
        public string LastName { get; set; }
        public Parent[] Parents { get; set; }
        public Child[] Children { get; set; }
        public Address Address { get; set; }
        public bool IsRegistered { get; set; }
        // The ToString() method is used to format the output, it's used for demo purpose only. It's not required by Azure Cosmos DB
        public override string ToString()
        {
            return JsonConvert.SerializeObject(this);
        }
    }

    public class Parent
    {
        public string FamilyName { get; set; }
        public string FirstName { get; set; }
    }

    public class Child
    {
        public string FamilyName { get; set; }
        public string FirstName { get; set; }
        public string Gender { get; set; }
        public int Grade { get; set; }
        public Pet[] Pets { get; set; }
    }

    public class Pet
    {
        public string GivenName { get; set; }
    }

    public class Address
    {
        public string State { get; set; }
        public string County { get; set; }
        public string City { get; set; }
    }
}
```

### <a name="add-the-using-directives--define-the-client-object"></a>Agregar las directivas de uso y definir el objeto de cliente

Desde el directorio del proyecto, abra el archivo `Program.cs` en el editor y agregue lo siguiente con directivas en la parte superior de la aplicación:

```csharp

using System;
using System.Threading.Tasks;
using System.Configuration;
using System.Collections.Generic;
using System.Net;
using Microsoft.Azure.Cosmos;
```

Para el archivo **Program.cs**, agregue código para leer las variables de entorno que ha establecido en el paso anterior. Defina los objetos `CosmosClient`, `Database`, y `Container`. A continuación, agregue código al método principal que llama al método `GetStartedDemoAsync` en el que administra los recursos de cuenta de Azure Cosmos. 

```csharp
namespace todo
{
public class Program
{

    /// The Azure Cosmos DB endpoint for running this GetStarted sample.
    private string EndpointUrl = Environment.GetEnvironmentVariable("EndpointUrl");

    /// The primary key for the Azure DocumentDB account.
    private string PrimaryKey = Environment.GetEnvironmentVariable("PrimaryKey");

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
           Console.WriteLine("Beginning operations...\n");
           Program p = new Program();
           await p.GetStartedDemoAsync();

        }
        catch (CosmosException de)
        {
           Exception baseException = de.GetBaseException();
           Console.WriteLine("{0} error occurred: {1}", de.StatusCode, de);
        }
        catch (Exception e)
        {
            Console.WriteLine("Error: {0}", e);
        }
        finally
        {
            Console.WriteLine("End of demo, press any key to exit.");
            Console.ReadKey();
        }
    }
}
}
```

### <a name="create-a-database"></a>Crear una base de datos 

Defina el método `CreateDatabaseAsync` dentro de la clase `program.cs`. Este método crea `FamilyDatabase` si todavía no existe.

```csharp
private async Task CreateDatabaseAsync()
{
    // Create a new database
    this.database = await this.cosmosClient.CreateDatabaseIfNotExistsAsync(databaseId);
    Console.WriteLine("Created Database: {0}\n", this.database.Id);
}
```

### <a name="create-a-container"></a>Crear un contenedor

Defina el método `CreateContainerAsync` dentro de la clase `program.cs`. Este método crea `FamilyContainer` si todavía no existe. 

```csharp
/// Create the container if it does not exist. 
/// Specifiy "/LastName" as the partition key since we're storing family information, to ensure good distribution of requests and storage.
private async Task CreateContainerAsync()
{
    // Create a new container
    this.container = await this.database.CreateContainerIfNotExistsAsync(containerId, "/LastName");
    Console.WriteLine("Created Container: {0}\n", this.container.Id);
}
```

### <a name="create-an-item"></a>Crear un elemento

Para crear un elemento de familia, agregue el método `AddItemsToContainerAsync` con el código siguiente. Puede usar los métodos `CreateItemAsync` o `UpsertItemAsync` para crear un elemento:

```csharp
private async Task AddItemsToContainerAsync()
{
    // Create a family object for the Andersen family
    Family andersenFamily = new Family
    {
        Id = "Andersen.1",
        LastName = "Andersen",
        Parents = new Parent[]
        {
           new Parent { FirstName = "Thomas" },
           new Parent { FirstName = "Mary Kay" }
        },
        Children = new Child[]
        {
           new Child
            {
                FirstName = "Henriette Thaulow",
                Gender = "female",
                Grade = 5,
                Pets = new Pet[]
                {
                    new Pet { GivenName = "Fluffy" }
                }
            }
        },
        Address = new Address { State = "WA", County = "King", City = "Seattle" },
        IsRegistered = false
    };

    try
    {
        // Create an item in the container representing the Andersen family. Note we provide the value of the partition key for this item, which is "Andersen".
        ItemResponse<Family> andersenFamilyResponse = await this.container.CreateItemAsync<Family>(andersenFamily, new PartitionKey(andersenFamily.LastName));
        // Note that after creating the item, we can access the body of the item with the Resource property of the ItemResponse. We can also access the RequestCharge property to see the amount of RUs consumed on this request.
        Console.WriteLine("Created item in database with id: {0} Operation consumed {1} RUs.\n", andersenFamilyResponse.Resource.Id, andersenFamilyResponse.RequestCharge);
    }
    catch (CosmosException ex) when (ex.StatusCode == HttpStatusCode.Conflict)
    {
        Console.WriteLine("Item in database with id: {0} already exists\n", andersenFamily.Id);                
    }
}

```

### <a name="query-the-items"></a>Consultar elementos

Después de insertar un elemento, puede ejecutar una consulta para obtener los detalles de la familia "Andersen". En el código siguiente se muestra cómo ejecutar la consulta directamente con la consulta SQL. La consulta SQL para obtener los detalles de la familia "Anderson" es: `SELECT * FROM c WHERE c.LastName = 'Andersen'`. Defina el método `QueryItemsAsync` en la clase `program.cs` y agréguele el código siguiente:


```csharp
private async Task QueryItemsAsync()
{
    var sqlQueryText = "SELECT * FROM c WHERE c.LastName = 'Andersen'";

    Console.WriteLine("Running query: {0}\n", sqlQueryText);

    QueryDefinition queryDefinition = new QueryDefinition(sqlQueryText);
    FeedIterator<Family> queryResultSetIterator = this.container.GetItemQueryIterator<Family>(queryDefinition);

    List<Family> families = new List<Family>();

    while (queryResultSetIterator.HasMoreResults)
    {
        FeedResponse<Family> currentResultSet = await queryResultSetIterator.ReadNextAsync();
        foreach (Family family in currentResultSet)
        {
            families.Add(family);
            Console.WriteLine("\tRead {0}\n", family);
        }
    }
}

```

### <a name="delete-the-database"></a>Eliminación de la base de datos 

Por último, para eliminar la base de datos, agregue el método `DeleteDatabaseAndCleanupAsync` con el código siguiente:

```csharp
private async Task DeleteDatabaseAndCleanupAsync()
{
   DatabaseResponse databaseResourceResponse = await this.database.DeleteAsync();
   // Also valid: await this.cosmosClient.Databases["FamilyDatabase"].DeleteAsync();

   Console.WriteLine("Deleted Database: {0}\n", this.databaseId);

   //Dispose of CosmosClient
    this.cosmosClient.Dispose();
}
```

### <a name="execute-the-crud-operations"></a>Ejecutar operaciones CRUD

Después de definir todos los métodos necesarios, ejecútelos en el método `GetStartedDemoAsync`. El método `DeleteDatabaseAndCleanupAsync` se marcó como comentado en este código porque no verá ningún recurso si se ejecuta ese método. Puede quitar la marca de comentario después de validar que los recursos de Azure Cosmos DB se crearon en Azure Portal. 

```csharp
public async Task GetStartedDemoAsync()
{
    // Create a new instance of the Cosmos Client
    this.cosmosClient = new CosmosClient(EndpointUrl, PrimaryKey);
    await this.CreateDatabaseAsync();
    await this.CreateContainerAsync();
    await this.AddItemsToContainerAsync();
    await this.QueryItemsAsync();
}
```

Después de agregar todos los métodos necesarios, guarde el archivo `Program.cs`. 

## <a name="run-the-code"></a>Ejecución del código

A continuación, compile y ejecute la aplicación para crear los recursos de Azure Cosmos DB. Asegúrese de abrir una nueva ventana del símbolo del sistema, no use la misma instancia que ha usado para establecer las variables de entorno. Puesto que las variables de entorno no se establecen en la ventana abierta actual. Tendrá que abrir un nuevo símbolo del sistema para ver las actualizaciones. 

```console
dotnet build
```

```console
dotnet run
```

Al ejecutar la aplicación, se genera el siguiente resultado. También puede iniciar sesión en Azure Portal y validar la creación de los recursos:

```console
Created Database: FamilyDatabase

Created Container: FamilyContainer

Created item in database with id: Andersen.1 Operation consumed 11.62 RUs.

Running query: SELECT * FROM c WHERE c.LastName = 'Andersen'

        Read {"id":"Andersen.1","LastName":"Andersen","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null,"FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":5,"Pets":[{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":false}

End of demo, press any key to exit.
```

Para validar la creación de los datos, inicie sesión en Azure Portal y vea los elementos necesarios en su cuenta de Azure Cosmos. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar el CLI de Azure o Azure PowerShell para quitar la cuenta de Azure Cosmos y el grupo de recursos correspondiente. El siguiente comando muestra cómo eliminar el grupo de recursos con el CLI de Azure:

```azurecli
az group delete -g "myResourceGroup"
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una base de datos y un contenedor en Azure Portal, y agregado datos de ejemplo mediante el ejemplo de .NET. Ahora puede importar datos adicionales a su cuenta de Azure Cosmos con las instrucciones del siguiente artículo. 

> [!div class="nextstepaction"]
> [Importación de datos a Azure Cosmos DB](import-data.md)
