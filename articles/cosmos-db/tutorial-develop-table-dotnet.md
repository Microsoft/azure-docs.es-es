---
title: Table API de Azure Cosmos DB con el SDK de .NET Standard
description: Obtenga información acerca de cómo almacenar y consultar los datos estructurados en una cuenta de Table API de Azure Cosmos DB.
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 12/03/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 80918c63c1fef82c13b23569bcda10545628e8e8
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998023"
---
# <a name="get-started-with-azure-cosmos-db-table-api-and-azure-table-storage-using-the-net-sdk"></a>Introducción a Table API de Azure Cosmos DB y Azure Table Storage mediante el SDK de .NET

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]

[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

Table API de Azure Cosmos DB o Azure Table Storage se pueden usar para almacenar datos NoSQL estructurados en la nube y proporcionar un almacén de claves y atributos con un diseño sin esquema. Dado que tanto Table API de Azure Cosmos DB como Table Storage carecen de esquema, es fácil adaptar los datos a medida que evolucionan las necesidades de la aplicación. Tanto Table API de Azure Cosmos DB como Table Storage se pueden usar para almacenar conjuntos de datos flexibles, como datos de usuarios para aplicaciones web, libretas de direcciones, información de dispositivos u otros tipos de metadatos que el servicio requiera. 

Este tutorial describe un ejemplo que muestra cómo usar [Microsoft Azure Cosmos DB Table Library for .NET](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) con escenarios de almacenamiento de Table API de Azure Cosmos DB y Azure Table Storage. Debe usar la conexión específica del servicio de Azure. Estos escenarios se describen mediante ejemplos de C# que ilustran cómo crear tablas, insertar y actualizar datos, consultar los datos y eliminar las tablas.

## <a name="prerequisites"></a>Requisitos previos

Necesitará lo siguiente para completar este ejemplo correctamente:

* [Microsoft Visual Studio](https://www.visualstudio.com/downloads/)

* [Microsoft Azure Cosmos DB Table Library for .NET](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table): esta biblioteca está disponible actualmente para .NET Standard y .NET Framework. 

* [Cuenta de Table API de Azure Cosmos DB](create-table-dotnet.md#create-a-database-account).

## <a name="create-an-azure-cosmos-db-table-api-account"></a>Creación de una cuenta de Table API de Azure Cosmos DB

[!INCLUDE [cosmos-db-create-dbaccount-table](../../includes/cosmos-db-create-dbaccount-table.md)]

## <a name="create-a-net-console-project"></a>Creación de un proyecto de consola de .NET

En Visual Studio, cree una nueva aplicación de consola de .NET. Los siguientes pasos muestran cómo crear una aplicación de consola en Visual Studio 2019. Azure Cosmos DB Table Library se puede usar en cualquier tipo de aplicación. NET, incluidos cualquier servicio en la nube o aplicación web de Azure, y aplicaciones de escritorio o móviles. En esta guía, usamos una aplicación de consola para hacerlo más sencillo.

1. Seleccione **File (Archivo)**  > **New (Nuevo)**  > **Project (Proyecto)** .

1. Elija **Aplicación de consola (.NET Core)** y, a continuación, seleccione **Siguiente**.

1. En el campo **Nombre del proyecto**, escriba un nombre para la aplicación, como **CosmosTableSamples**. (Puede proporcionar un nombre diferente para adaptarse a sus necesidades).

1. Seleccione **Crear**.

Todos los ejemplos de código de este ejemplo se pueden agregar al método Main() del archivo **Program.cs** de la aplicación de consola.

## <a name="install-the-required-nuget-package"></a>Instalación del paquete NuGet requerido

Para obtener el paquete NuGet, siga estos pasos:

1. Haga clic con el botón derecho en el proyecto, en el **Explorador de soluciones**, y elija **Administrar paquetes NuGet**.

1. Busque en línea [`Microsoft.Azure.Cosmos.Table`](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table), [`Microsoft.Extensions.Configuration`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration), [`Microsoft.Extensions.Configuration.Json`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json) y [`Microsoft.Extensions.Configuration.Binder`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Binder) y seleccione **Instalar** para instalar Microsoft Azure Cosmos DB Table Library.

## <a name="configure-your-storage-connection-string"></a>Configuración de la cadena de conexión de almacenamiento.

1. Desde [Azure Portal](https://portal.azure.com/), vaya a su cuenta de Azure Cosmos o la cuenta de Table Storage. 

1. Abra el panel **Cadena de conexión** o **Claves de acceso**. Use el botón de copia en el lado derecho de la ventana para copiar la **CADENA DE CONEXIÓN PRINCIPAL**.

   :::image type="content" source="./media/create-table-dotnet/connection-string.png" alt-text="Visualización y copia de la cadena de conexión principal (PRIMARY CONNECTION STRING) en el panel Cadena de conexión":::
   
1. Para configurar la cadena de conexión, desde Visual Studio haga clic con el botón derecho en el proyecto **CosmosTableSamples**.

1. Seleccione **Agregar** y, a continuación, **Nuevo elemento**. Cree un nuevo archivo **Settings.json** con el tipo de archivo como **Configuración de JSON de TypeScript**. 

1. Reemplace el código en el archivo Settings.json por el código siguiente y asigne la cadena de conexión principal:

   ```csharp
   {
   "StorageConnectionString": <Primary connection string of your Azure Cosmos DB account>
   }
   ```

1. Haga clic con el botón derecho en el proyecto **CosmosTableSamples**. Seleccione **Agregar**, **Nuevo elemento** y agregue una clase denominada **AppSettings.cs**.

1. Agregue el siguiente código al archivo AppSettings.cs. Este archivo lee la cadena de conexión del archivo Settings.json y la asigna al parámetro de configuración:

   ```csharp
   namespace CosmosTableSamples
   {
    using Microsoft.Extensions.Configuration;
    public class AppSettings
    {
        public string StorageConnectionString { get; set; }
        public static AppSettings LoadAppSettings()
        {
            IConfigurationRoot configRoot = new ConfigurationBuilder()
                .AddJsonFile("Settings.json")
                .Build();
            AppSettings appSettings = configRoot.Get<AppSettings>();
            return appSettings;
        }
    }
   }
   ```

## <a name="parse-and-validate-the-connection-details"></a>Análisis y validación de los detalles de conexión 

1. Haga clic con el botón derecho en el proyecto **CosmosTableSamples**. Seleccione **Agregar**, **Nuevo elemento** y agregue una clase denominada **Common.cs**. Escribirá código para validar los detalles de conexión y crear una tabla dentro de esta clase.

1. Defina un método `CreateStorageAccountFromConnectionString` tal como se muestra a continuación. Este método analizará los detalles de la cadena de conexión y comprobará que el nombre de la cuenta y los detalles de la clave de cuenta proporcionados en el archivo "Settings.json" son válidos. 

 ```csharp
using System;

namespace CosmosTableSamples
{
    using System.Threading.Tasks;
    using Microsoft.Azure.Cosmos.Table;
    using Microsoft.Azure.Documents;

    public class Common
    {
        public static CloudStorageAccount CreateStorageAccountFromConnectionString(string storageConnectionString)
        {
            CloudStorageAccount storageAccount;
            try
            {
                storageAccount = CloudStorageAccount.Parse(storageConnectionString);
            }
            catch (FormatException)
            {
                Console.WriteLine("Invalid storage account information provided. Please confirm the AccountName and AccountKey are valid in the app.config file - then restart the application.");
                throw;
            }
            catch (ArgumentException)
            {
                Console.WriteLine("Invalid storage account information provided. Please confirm the AccountName and AccountKey are valid in the app.config file - then restart the sample.");
                Console.ReadLine();
                throw;
            }

            return storageAccount;
        }
    }
}
   ```


## <a name="create-a-table"></a>Creación de una tabla 

La clase [CloudTableClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.cloudtableclient) le permite recuperar las tablas y las entidades almacenadas en Almacenamiento de tablas. Dado que no hay ninguna tabla en la cuenta de Table API de Cosmos DB, vamos a agregar el método `CreateTableAsync` a la clase **Common.cs** para crear una tabla:

```csharp
public static async Task<CloudTable> CreateTableAsync(string tableName)
  {
    string storageConnectionString = AppSettings.LoadAppSettings().StorageConnectionString;

    // Retrieve storage account information from connection string.
    CloudStorageAccount storageAccount = CreateStorageAccountFromConnectionString(storageConnectionString);

    // Create a table client for interacting with the table service
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient(new TableClientConfiguration());

    Console.WriteLine("Create a Table for the demo");

    // Create a table client for interacting with the table service 
    CloudTable table = tableClient.GetTableReference(tableName);
    if (await table.CreateIfNotExistsAsync())
    {
      Console.WriteLine("Created Table named: {0}", tableName);
    }
    else
    {
      Console.WriteLine("Table {0} already exists", tableName);
    }

    Console.WriteLine();
    return table;
}
```

Si recibe una "excepción 503 de servicio no disponible", es posible que los puertos necesarios para el modo de conexión estén bloqueados por un firewall. Para corregir este problema, abra los puertos necesarios o use la conectividad del modo de puerta de enlace como se indica en el código siguiente:

```csharp
tableClient.TableClientConfiguration.UseRestExecutorForCosmosEndpoint = true;
```

## <a name="define-the-entity"></a>Definición de la entidad 

Las entidades se asignan a objetos C# utilizando una clase personalizada derivada de [TableEntity](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.tableentity). Para agregar una entidad a una tabla, cree una clase que defina las propiedades de la entidad.

Haga clic con el botón derecho en el proyecto **CosmosTableSamples**. Seleccione **Agregar**, **Nueva carpeta** y asígnele el nombre **Model**. En la carpeta Model, agregue una clase llamada **CustomerEntity.cs** y agréguele el código siguiente.

```csharp
namespace CosmosTableSamples.Model
{
    using Microsoft.Azure.Cosmos.Table;
    public class CustomerEntity : TableEntity
    {
        public CustomerEntity()
        {
        }

        public CustomerEntity(string lastName, string firstName)
        {
            PartitionKey = lastName;
            RowKey = firstName;
        }

        public string Email { get; set; }
        public string PhoneNumber { get; set; }
    }
}
```

El código define una clase de entidad que utiliza el nombre de pila del cliente como clave de fila y el apellido como clave de partición. En conjunto, las claves de partición y de fila de una entidad la identifican inequívocamente en la tabla. Las entidades con la misma clave de partición pueden consultarse más rápidamente que aquellas con diferentes claves de partición, pero el uso de claves de partición diversas permite una mayor escalabilidad de las operaciones paralelas. Las entidades que se almacenan en tablas deben ser de un tipo compatible, por ejemplo, derivadas de la clase [TableEntity](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.tableentity). Las propiedades de entidad que desee almacenar en una tabla deben ser propiedades públicas del tipo, y deben admitir que se obtengan y establezcan valores. Además, el tipo de entidad debe exponer un constructor sin parámetros.

## <a name="insert-or-merge-an-entity"></a>Inserción o combinación de la entidad

En el ejemplo de código siguiente se crea un objeto de entidad y se agrega a la tabla. El método InsertOrMerge dentro de la clase [TableOperation](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.tableoperation) se utiliza para insertar o combinar una entidad. Se llama al método [CloudTable.ExecuteAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.cloudtable.executeasync?view=azure-dotnet) para ejecutar la operación. 

Haga clic con el botón derecho en el proyecto **CosmosTableSamples**. Seleccione **Agregar**, **Nuevo elemento** y agregue una clase denominada **SamplesUtils.cs**. Esta clase almacena todo el código necesario para realizar operaciones CRUD en las entidades. 

```csharp
 public static async Task<CustomerEntity> InsertOrMergeEntityAsync(CloudTable table, CustomerEntity entity)
 {
     if (entity == null)
     {
         throw new ArgumentNullException("entity");
     }
     try
     {
         // Create the InsertOrReplace table operation
         TableOperation insertOrMergeOperation = TableOperation.InsertOrMerge(entity);

         // Execute the operation.
         TableResult result = await table.ExecuteAsync(insertOrMergeOperation);
         CustomerEntity insertedCustomer = result.Result as CustomerEntity;

         // Get the request units consumed by the current operation. RequestCharge of a TableResult is only applied to Azure Cosmos DB
         if (result.RequestCharge.HasValue)
         {
             Console.WriteLine("Request Charge of InsertOrMerge Operation: " + result.RequestCharge);
         }

         return insertedCustomer;
     }
     catch (StorageException e)
     {
         Console.WriteLine(e.Message);
         Console.ReadLine();
         throw;
     }
 }
```

## <a name="get-an-entity-from-a-partition"></a>Obtención de una entidad de una partición

Puede obtener la entidad de una partición mediante el método Retrieve de la clase [TableOperation](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.tableoperation). En el ejemplo de código siguiente se obtiene la clave de fila de la clave de partición, el correo electrónico y el número de teléfono de una entidad de cliente. Este ejemplo también imprime las unidades de solicitud utilizadas para consultar la entidad. Para consultar una entidad, anexe el código siguiente al archivo **SamplesUtils.cs**: 

```csharp
public static async Task<CustomerEntity> RetrieveEntityUsingPointQueryAsync(CloudTable table, string partitionKey, string rowKey)
    {
      try
      {
        TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>(partitionKey, rowKey);
        TableResult result = await table.ExecuteAsync(retrieveOperation);
        CustomerEntity customer = result.Result as CustomerEntity;
        if (customer != null)
        {
          Console.WriteLine("\t{0}\t{1}\t{2}\t{3}", customer.PartitionKey, customer.RowKey, customer.Email, customer.PhoneNumber);
        }

        // Get the request units consumed by the current operation. RequestCharge of a TableResult is only applied to Azure CosmoS DB 
        if (result.RequestCharge.HasValue)
        {
           Console.WriteLine("Request Charge of Retrieve Operation: " + result.RequestCharge);
        }

        return customer;
        }
        catch (StorageException e)
        {
           Console.WriteLine(e.Message);
           Console.ReadLine();
           throw;
        }
    }
```

## <a name="delete-an-entity"></a>Eliminación de una entidad

Puede eliminar fácilmente una entidad una vez recuperada utilizando el mismo patrón mostrado para actualizar una entidad. El código siguiente recupera y elimina una entidad de cliente. Para eliminar una entidad, anexe el código siguiente al archivo **SamplesUtils.cs**: 

```csharp
public static async Task DeleteEntityAsync(CloudTable table, CustomerEntity deleteEntity)
   {
     try
     {
        if (deleteEntity == null)
     {
        throw new ArgumentNullException("deleteEntity");
     }

    TableOperation deleteOperation = TableOperation.Delete(deleteEntity);
    TableResult result = await table.ExecuteAsync(deleteOperation);

    // Get the request units consumed by the current operation. RequestCharge of a TableResult is only applied to Azure CosmoS DB 
    if (result.RequestCharge.HasValue)
    {
       Console.WriteLine("Request Charge of Delete Operation: " + result.RequestCharge);
    }

    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

## <a name="execute-the-crud-operations-on-sample-data"></a>Ejecución de operaciones CRUD en datos de ejemplo

Después de definir los métodos para crear una tabla e insertar o combinar entidades, ejecute estos métodos en los datos de ejemplo. Para ello, haga clic con el botón derecho en el proyecto **CosmosTableSamples**. Seleccione **Agregar**, **Nuevo elemento**, agregue una clase denominada **BasicSamples.cs** y agréguele el código siguiente. Este código crea una tabla y le agrega las entidades. Si desea eliminar la entidad y la tabla al final del proyecto, quite los comentarios de los métodos `table.DeleteIfExistsAsync()` y `SamplesUtils.DeleteEntityAsync(table, customer)` del código siguiente:

```csharp
using System;
namespace CosmosTableSamples
{
    using System.Threading.Tasks;
    using Microsoft.Azure.Cosmos.Table;
    using Model;

    class BasicSamples
    {
        public async Task RunSamples()
        {
            Console.WriteLine("Azure Cosmos DB Table - Basic Samples\n");
            Console.WriteLine();

            string tableName = "demo" + Guid.NewGuid().ToString().Substring(0, 5);

            // Create or reference an existing table
            CloudTable table = await Common.CreateTableAsync(tableName);

            try
            {
                // Demonstrate basic CRUD functionality 
                await BasicDataOperationsAsync(table);
            }
            finally
            {
                // Delete the table
                // await table.DeleteIfExistsAsync();
            }
        }

        private static async Task BasicDataOperationsAsync(CloudTable table)
        {
            // Create an instance of a customer entity. See the Model\CustomerEntity.cs for a description of the entity.
            CustomerEntity customer = new CustomerEntity("Harp", "Walter")
            {
                Email = "Walter@contoso.com",
                PhoneNumber = "425-555-0101"
            };

            // Demonstrate how to insert the entity
            Console.WriteLine("Insert an Entity.");
            customer = await SamplesUtils.InsertOrMergeEntityAsync(table, customer);

            // Demonstrate how to Update the entity by changing the phone number
            Console.WriteLine("Update an existing Entity using the InsertOrMerge Upsert Operation.");
            customer.PhoneNumber = "425-555-0105";
            await SamplesUtils.InsertOrMergeEntityAsync(table, customer);
            Console.WriteLine();

            // Demonstrate how to Read the updated entity using a point query 
            Console.WriteLine("Reading the updated Entity.");
            customer = await SamplesUtils.RetrieveEntityUsingPointQueryAsync(table, "Harp", "Walter");
            Console.WriteLine();

            // Demonstrate how to Delete an entity
            //Console.WriteLine("Delete the entity. ");
            //await SamplesUtils.DeleteEntityAsync(table, customer);
            //Console.WriteLine();
        }
    }
}
```

El código anterior crea una tabla que empieza por "demo" y el GUID generado se anexa al nombre de la tabla. A continuación, agrega una entidad de cliente con nombre y apellido como "Walter Harp" y más tarde actualiza el número de teléfono de este usuario. 

En este tutorial, ha compilado código para realizar operaciones CRUD básicas en los datos almacenados en la cuenta de Table API. También puede realizar operaciones avanzadas, como insertar datos por lotes, consultar todos los datos de una partición, consultar un intervalo de datos de una partición y enumerar tablas en la cuenta cuyos nombres comiencen por el prefijo especificado. Puede descargar el ejemplo completo del repositorio de GitHub [azure-cosmos-table-dotnet-core-getting-started](https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started). La clase [AdvancedSamples.cs](https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started/blob/master/CosmosTableSamples/AdvancedSamples.cs) tiene más operaciones que puede realizar en los datos.  

## <a name="run-the-project"></a>Ejecución del proyecto

En el proyecto **CosmosTableSamples**. Abra la clase denominada **Program.CS** y agréguele el siguiente código para llamar a BasicSamples cuando se ejecute el proyecto.

```csharp
using System;

namespace CosmosTableSamples
{
    class Program
    {
        public static void Main(string[] args)
        {
            Console.WriteLine("Azure Cosmos Table Samples");
            BasicSamples basicSamples = new BasicSamples();
            basicSamples.RunSamples().Wait();
           
            Console.WriteLine();
            Console.WriteLine("Press any key to exit");
            Console.Read();
        }
    }
}
```

Ahora, compile la solución y presione F5 para ejecutar el proyecto. Cuando se ejecute el proyecto, verá la siguiente salida en el símbolo del sistema:

:::image type="content" source="./media/tutorial-develop-table-standard/output-from-sample.png" alt-text="Salida del símbolo del sistema":::

Si recibe un error que indica que no se encuentra el archivo Settings.json al ejecutar el proyecto, puede resolverlo agregando la siguiente entrada XML a la configuración del proyecto. Haga clic con el botón derecho en CosmosTableSamples, seleccione Editar CosmosTableSamples.csproj y agregue el itemGroup siguiente: 

```csharp
  <ItemGroup>
    <None Update="Settings.json">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>
```
Ahora puede iniciar sesión en Azure Portal y comprobar que los datos existen en la tabla. 

:::image type="content" source="./media/tutorial-develop-table-standard/results-in-portal.png" alt-text="Resultados en el portal":::

## <a name="next-steps"></a>Pasos siguientes

Ahora puede pasar al siguiente tutorial y aprender a migrar datos a la cuenta de Table API de Azure Cosmos DB. 

> [!div class="nextstepaction"]
>[Consulta de datos](../cosmos-db/table-import.md)
