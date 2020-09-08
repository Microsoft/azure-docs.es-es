---
title: Introducción al almacenamiento de tablas mediante Visual Studio (servicios en la nube)
description: Cómo empezar a usar el almacenamiento de tablas de Azure en un proyecto de servicio en la nube en Visual Studio después de conectarse a una cuenta de almacenamiento mediante los servicios conectados de Visual Studio
services: storage
author: ghogen
manager: jillfra
ms.assetid: a3a11ed8-ba7f-4193-912b-e555f5b72184
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure, devx-track-csharp
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 12/02/2016
ms.author: ghogen
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 8579efcee45e08fec3331df8f55b61618edb4f4f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89000420"
---
# <a name="getting-started-with-azure-table-storage-and-visual-studio-connected-services-cloud-services-projects"></a>Introducción al almacenamiento de tablas de Azure y a los servicios conectados de Visual Studio (proyectos de servicios en la nube)
[!INCLUDE [storage-try-azure-tools-tables](../../includes/storage-try-azure-tools-tables.md)]

## <a name="overview"></a>Información general
En este artículo se describe cómo empezar a usar el almacenamiento de tablas de Azure en Visual Studio después de haber creado o hecho referencia a una cuenta de almacenamiento de Azure en un proyecto de servicios en la nube mediante el cuadro de diálogo **Agregar servicios conectados** de Visual Studio. La operación **Agregar servicios conectados** instala los paquetes de NuGet adecuados para tener acceso al almacenamiento de Azure en el proyecto y agrega la cadena de conexión para la cuenta de almacenamiento a los archivos de configuración del proyecto.

El servicio de almacenamiento de tabla de Azure permite almacenar una gran cantidad de datos estructurados. El servicio es un almacén de datos NoSQL que acepta llamadas autenticadas desde dentro y fuera de la nube de Azure. Las tablas de Azure son ideales para el almacenamiento de datos estructurados no relacionales.

Para comenzar, necesita crear una tabla en su cuenta de almacenamiento. También le mostraremos cómo crear una tabla de Azure en código, además de cómo realizar operaciones básicas de tabla y de entidad, como agregar, modificar y leer entidades de tabla. Los ejemplos están escritos en código C\# y usan la [biblioteca del cliente de Microsoft Azure Storage para .NET](https://msdn.microsoft.com/library/azure/dn261237.aspx).

**NOTA:** Algunas de las API que realizan llamadas al almacenamiento de Azure son asincrónicas. Vea [Programación asincrónica con Async y Await](https://msdn.microsoft.com/library/hh191443.aspx) para más información. El código siguiente supone que se están utilizando métodos de programación asincrónica.

* Consulte [Introducción al Almacenamiento de tablas de Azure mediante .NET](../storage/storage-dotnet-how-to-use-tables.md) para más información sobre la manipulación de tablas mediante programación.
* Vea [Documentación sobre Almacenamiento](https://azure.microsoft.com/documentation/services/storage/) para información general sobre Azure Storage.
* Vea [Documentación sobre Cloud Services](https://azure.microsoft.com/documentation/services/cloud-services/) para información general sobre Azure Cloud Services.
* Vea [ASP.NET](https://www.asp.net) para más información sobre la programación de aplicaciones ASP.NET.

## <a name="access-tables-in-code"></a>Acceso a tablas en código
Para obtener acceso a las tablas en los proyectos de servicios en la nube, deberá incluir los elementos siguientes en los archivos de origen C# que acceden al almacenamiento Tabla de Azure.

1. Asegúrese de que las declaraciones del espacio de nombres de la parte superior del archivo de C# incluyen estas instrucciones **using** .
   
    ```csharp
    using Microsoft.Framework.Configuration;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Table;
    using System.Threading.Tasks;
    using LogLevel = Microsoft.Framework.Logging.LogLevel;
    ```
2. Obtenga un objeto **CloudStorageAccount** que represente la información de su cuenta de almacenamiento. Use el código siguiente para obtener la cadena de conexión de almacenamiento y la información de la cuenta de almacenamiento de la configuración del servicio de Azure.
   
    ```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("<storage account name>
    _AzureStorageConnectionString"));
    ```
   > [!NOTE]
   > Use todo el código anterior delante del código que aparece en los ejemplos siguientes.
   > 
   > 
3. Obtenga un objeto **CloudTableClient** para hacer referencia a los objetos de tabla de la cuenta de almacenamiento.
   
    ```csharp
    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
    ```
4. Obtenga un objeto de referencia **CloudTable** para hacer referencia a una tabla y a entidades específicas.
   
    ```csharp
    // Get a reference to a table named "peopleTable".
    CloudTable peopleTable = tableClient.GetTableReference("peopleTable");
    ```

## <a name="create-a-table-in-code"></a>Crear una tabla en el código
Para crear la tabla de Azure, basta con que agregue una llamada a **CreateIfNotExistsAsync** después de obtener el objeto **CloudTable**, tal como se describe en la sección "Acceso a tablas en código".

```csharp
// Create the CloudTable if it does not exist.
await peopleTable.CreateIfNotExistsAsync();
```

## <a name="add-an-entity-to-a-table"></a>Adición de una entidad a una tabla
Para agregar una entidad a una tabla, cree una clase que defina las propiedades de la entidad. El código siguiente define una clase de entidad llamada **CustomerEntity** que usa el nombre del cliente como clave de fila y el apellido como clave de partición.

```csharp
public class CustomerEntity : TableEntity
{
    public CustomerEntity(string lastName, string firstName)
    {
        this.PartitionKey = lastName;
        this.RowKey = firstName;
    }

    public CustomerEntity() { }

    public string Email { get; set; }

    public string PhoneNumber { get; set; }
}
```

Las operaciones de tablas que afectan a las entidades se realizan con el objeto **CloudTable** que se creó anteriormente en el apartado "Acceso a tablas en código". El objeto **TableOperation** representa la operación que se va a realizar. En el ejemplo de código siguiente se muestra cómo crear un objeto **CloudTable** y un objeto **CustomerEntity**. Para preparar la operación, se crea un objeto **TableOperation** para insertar la entidad de cliente en la tabla. Por último, la operación se ejecuta llamando a **CloudTable.ExecuteAsync**.

```csharp
// Create a new customer entity.
CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
customer1.Email = "Walter@contoso.com";
customer1.PhoneNumber = "425-555-0101";

// Create the TableOperation that inserts the customer entity.
TableOperation insertOperation = TableOperation.Insert(customer1);

// Execute the insert operation.
await peopleTable.ExecuteAsync(insertOperation);
```


## <a name="insert-a-batch-of-entities"></a>Inserción de un lote de entidades
Puede insertar varias entidades en una tabla mediante una única operación de escritura. En el ejemplo de código siguiente se crean dos objetos de entidad ("Jeff Smith" y "Ben Smith") que se agregan a un objeto **TableBatchOperation** mediante el método Insert y, a continuación, se inicia la operación llamando a **CloudTable.ExecuteBatchAsync**.

```csharp
// Create the batch operation.
TableBatchOperation batchOperation = new TableBatchOperation();

// Create a customer entity and add it to the table.
CustomerEntity customer1 = new CustomerEntity("Smith", "Jeff");
customer1.Email = "Jeff@contoso.com";
customer1.PhoneNumber = "425-555-0104";

// Create another customer entity and add it to the table.
CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
customer2.Email = "Ben@contoso.com";
customer2.PhoneNumber = "425-555-0102";

// Add both customer entities to the batch insert operation.
batchOperation.Insert(customer1);
batchOperation.Insert(customer2);

// Execute the batch operation.
await peopleTable.ExecuteBatchAsync(batchOperation);
```

## <a name="get-all-of-the-entities-in-a-partition"></a>Obtención de todas las entidades en una partición
Para consultar una tabla a fin de obtener todas las entidades de una partición, use un objeto **TableQuery** . El ejemplo de código siguiente especifica un filtro para las entidades en las que “Smith” es la clave de partición. En este ejemplo, los campos de cada entidad se imprimen en la consola, como parte de los resultados de la consulta.

```csharp
// Construct the query operation for all customer entities where PartitionKey="Smith".
TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>()
    .Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

// Print the fields for each customer.
TableContinuationToken token = null;
do
{
    TableQuerySegment<CustomerEntity> resultSegment = await peopleTable.ExecuteQuerySegmentedAsync(query, token);
    token = resultSegment.ContinuationToken;

    foreach (CustomerEntity entity in resultSegment.Results)
    {
        Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
        entity.Email, entity.PhoneNumber);
    }
} while (token != null);

return View();
```


## <a name="get-a-single-entity"></a>Obtención de una sola entidad
Puede escribir una consulta para obtener una sola entidad concreta. El código siguiente utiliza un objeto **TableOperation** para especificar el cliente llamado "Ben Smith". Este método devuelve solo una entidad, en lugar de una colección, y el valor devuelto en **TableResult.Result** es un objeto **CustomerEntity**. La forma más rápida de recuperar una sola entidad del servicio **Tabla** es especificar claves tanto de partición como de fila en las consultas.

```csharp
// Create a retrieve operation that takes a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the retrieve operation.
TableResult retrievedResult = await peopleTable.ExecuteAsync(retrieveOperation);

// Print the phone number of the result.
if (retrievedResult.Result != null)
    Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
else
    Console.WriteLine("The phone number could not be retrieved.");
```

## <a name="delete-an-entity"></a>Eliminación de una entidad
Puede eliminar fácilmente una entidad después de haberla encontrado. El código siguiente busca una entidad de cliente denominada "Ben Smith" y, si la encuentra, la elimina.

```csharp
// Create a retrieve operation that expects a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the operation.
TableResult retrievedResult = peopleTable.Execute(retrieveOperation);

// Assign the result to a CustomerEntity object.
CustomerEntity deleteEntity = (CustomerEntity)retrievedResult.Result;

// Create the Delete TableOperation and then execute it.
if (deleteEntity != null)
{
    TableOperation deleteOperation = TableOperation.Delete(deleteEntity);

    // Execute the operation.
    await peopleTable.ExecuteAsync(deleteOperation);

    Console.WriteLine("Entity deleted.");
}

else
    Console.WriteLine("Couldn't delete the entity.");
```

## <a name="next-steps"></a>Pasos siguientes
[!INCLUDE [vs-storage-dotnet-tables-next-steps](../../includes/vs-storage-dotnet-tables-next-steps.md)]

