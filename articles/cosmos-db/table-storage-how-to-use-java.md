---
title: Uso de Azure Storage Table o Table API de Azure Cosmos DB desde Java
description: Almacene datos estructurados en la nube mediante Azure Table Storage o Table API de Azure Cosmos DB.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: Java
ms.topic: sample
ms.date: 07/23/2020
author: sakash279
ms.author: akshanka
ms.custom: devx-track-java
ms.openlocfilehash: e28770bae9f845ae8f5edd3b67bc55175392052a
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056676"
---
# <a name="how-to-use-azure-table-storage-or-azure-cosmos-db-table-api-from-java"></a>Uso de Azure Table Storage y Table API de Azure Cosmos DB desde Java

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

En este artículo se muestra cómo crear tablas, y almacenar los datos y realizar operaciones CRUD en ellos. Elija usar el servicio Azure Table Storage o Table API de Azure Cosmos DB. Los ejemplos están escritos en Java y utilizan el [SDK de Azure Storage para Java][Azure Storage SDK for Java]. Entre los escenarios descritos se incluyen **crear**, **enumerar** y **eliminar** tablas, así como **insertar**, **consultar**, **modificar** y **eliminar** entidades de una tabla. Para obtener más información acerca de las tablas, consulte la sección [Pasos siguientes](#next-steps) .

> [!NOTE]
> hay un SDK disponible para los desarrolladores que usen Azure Storage en dispositivos Android. Para obtener más información, vea el [SDK de Azure Storage para Android][Azure Storage SDK for Android].
>

## <a name="create-an-azure-service-account"></a>Creación de una cuenta de servicio de Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

**Creación de una cuenta de Azure Storage**

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

**Creación de una cuenta de Azure Cosmos DB**

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="create-a-java-application"></a>Creación de una aplicación Java

En esta guía utilizará funciones del almacenamiento que puede ejecutar en una aplicación Java localmente o bien mediante código a través de un rol web o un rol de trabajo de Azure.

Para usar los ejemplos de este artículo, deberá instalar el Kit de desarrollo de Java (JDK) y crear una cuenta de Azure Storage o una cuenta de Azure Cosmos DB en su suscripción a Azure. Después de esto, compruebe que su sistema de desarrollo cumple los requisitos mínimos y las dependencias que se indican en el repositorio del [SDK de Azure Storage para Java][Azure Storage SDK for Java] en GitHub. Si su sistema cumple esos requisitos, puede seguir las instrucciones para descargar e instalar las bibliotecas de Azure Storage para Java en su sistema desde ese repositorio. Después de completar esas tareas puede crear una aplicación Java que use los ejemplos de este artículo.

## <a name="configure-your-application-to-access-table-storage"></a>Configuración de la aplicación para acceder al almacenamiento de tablas

Agregue las siguientes instrucciones de importación en la parte superior del archivo Java en el que desea utilizar las API de Azure Storage o Table API de Azure Cosmos DB para obtener acceso a las tablas:

```java
// Include the following imports to use table APIs
import com.microsoft.azure.storage.*;
import com.microsoft.azure.storage.table.*;
import com.microsoft.azure.storage.table.TableQuery.*;
```

## <a name="add-your-connection-string"></a>Incorporación de la cadena de conexión

Puede conectarse a la cuenta de almacenamiento de Azure o a la de Table API de Azure Cosmos DB. Obtenga la cadena de conexión en función del tipo de cuenta que esté usando.

### <a name="add-an-azure-storage-connection-string"></a>Adición de una cadena de conexión de almacenamiento de Azure

Un cliente de almacenamiento de Azure utiliza una cadena de conexión de almacenamiento para almacenar extremos y credenciales con el fin de obtener acceso a los servicios de administración de datos. Al ejecutarse en una aplicación cliente, debe proporcionar la cadena de conexión de almacenamiento en el siguiente formato, usando el nombre de su cuenta de almacenamiento y la clave de acceso principal de la cuenta de almacenamiento que se muestra en [Azure Portal](https://portal.azure.com) para los valores **AccountName** y **AccountKey**. 

En este ejemplo se muestra cómo puede declarar un campo estático para mantener la cadena de conexión:

```java
// Define the connection-string with your values.
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account;" +
    "AccountKey=your_storage_account_key";
```

### <a name="add-an-azure-cosmos-db-table-api-connection-string"></a>Adición de una cadena de conexión de Table API de Azure Cosmos DB

Una cuenta de Azure Cosmos DB utiliza una cadena de conexión para almacenar el punto de conexión de la tabla y sus credenciales. Al ejecutarse en una aplicación cliente, debe proporcionar la cadena de conexión de Azure Cosmos DB en el siguiente formato, usando el nombre de su cuenta de Azure Cosmos DB y la clave de acceso principal de la cuenta que se muestra en [Azure Portal](https://portal.azure.com) para los valores **AccountName** y **AccountKey**. 

En este ejemplo se muestra cómo puede declarar un campo estático para mantener la cadena de conexión de Azure Cosmos DB:

```java
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=https;" + 
    "AccountName=your_cosmosdb_account;" + 
    "AccountKey=your_account_key;" + 
    "TableEndpoint=https://your_endpoint;" ;
```

En una aplicación que se ejecuta en un rol de Azure, puede almacenar esta cadena en el archivo de configuración de servicio, *ServiceConfiguration.cscfg*, y se puede obtener acceso a él con una llamada al método **RoleEnvironment.getConfigurationSettings**. A continuación se muestra un ejemplo de cómo obtener la cadena de conexión desde un elemento de **configuración** denominado *StorageConnectionString* en el archivo de configuración del servicio:

```java
// Retrieve storage account from connection-string.
String storageConnectionString =
    RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");
```

También puede almacenar la cadena de conexión en el archivo config.properties del proyecto:

```java
StorageConnectionString = DefaultEndpointsProtocol=https;AccountName=your_account;AccountKey=your_account_key;TableEndpoint=https://your_table_endpoint/
```

En los ejemplos siguientes se supone que usó uno de estos métodos para obtener la cadena de conexión de almacenamiento.

## <a name="create-a-table"></a>Creación de una tabla

Los objetos `CloudTableClient` permiten obtener objetos de referencia para tablas y entidades. El siguiente código crea un objeto `CloudTableClient` y lo usa para crear un nuevo objeto `CloudTable` que representa una tabla llamada "people". 

> [!NOTE]
> Existen otras maneras de crear objetos `CloudStorageAccount`. Para más información, consulte `CloudStorageAccount` en la [Referencia del SDK del cliente de Azure Storage].
>

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create the table if it doesn't exist.
    String tableName = "people";
    CloudTable cloudTable = tableClient.getTableReference(tableName);
    cloudTable.createIfNotExists();
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="list-the-tables"></a>Enumeración de las tablas

Para obtener una lista de las tablas, llame al método **CloudTableClient.listTables()** para recuperar una lista que se puede iterar de nombres de tablas.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Loop through the collection of table names.
    for (String table : tableClient.listTables())
    {
        // Output each table name.
        System.out.println(table);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="add-an-entity-to-a-table"></a>Adición de una entidad a una tabla

Las entidades se asignan a objetos de Java con una clase personalizada que implementa `TableEntity`. Para mayor comodidad, la clase `TableServiceEntity` implementa `TableEntity` y usa la reflexión para asignar propiedades a los métodos de captador y establecedor indicados para las propiedades. Para agregar una entidad a una tabla, cree primero una clase que defina las propiedades de la entidad. El código siguiente define una clase de entidad que utiliza el nombre de pila del cliente como clave de fila y el apellido como clave de partición. En conjunto, la clave de partición y la clave de fila de una entidad la identifican inequívocamente en la tabla. Puede realizarse una consulta en las entidades con la misma clave de partición de manera más rápida que en aquellas que tienen claves de partición distintas.

```java
public class CustomerEntity extends TableServiceEntity {
    public CustomerEntity(String lastName, String firstName) {
        this.partitionKey = lastName;
        this.rowKey = firstName;
    }

    public CustomerEntity() { }

    String email;
    String phoneNumber;

    public String getEmail() {
        return this.email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPhoneNumber() {
        return this.phoneNumber;
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }
}
```

Las operaciones de tabla que afectan a las entidades requieren un objeto `TableOperation`. Este objeto define la operación que va a realizarse en una entidad, que puede ejecutarse con un objeto `CloudTable`. El código siguiente crea una instancia `CustomerEntity` con algunos datos del cliente que se van a almacenar. El código siguiente llama a `TableOperation`.insertOrReplace** para crear un objeto `TableOperation` a fin de insertar una entidad en una tabla y le asocia el nuevo valor `CustomerEntity`. Por último, el código llama al método `execute` en el objeto `CloudTable`, especificando la tabla "people" y el nuevo `TableOperation`, que posteriormente envía una solicitud al servicio de almacenamiento para insertar la nueva entidad de cliente en la tabla "people" o sustituir la entidad si ya existe.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a new customer entity.
    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    customer1.setEmail("Walter@contoso.com");
    customer1.setPhoneNumber("425-555-0101");

    // Create an operation to add the new customer to the people table.
    TableOperation insertCustomer1 = TableOperation.insertOrReplace(customer1);

    // Submit the operation to the table service.
    cloudTable.execute(insertCustomer1);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="insert-a-batch-of-entities"></a>Inserción de un lote de entidades

Puede insertar un lote de entidades en el servicio Tabla mediante una operación de escritura. El siguiente código crea un objeto `TableBatchOperation` y le agrega tres operaciones de inserción. Cada operación de inserción se agrega mediante la creación de un nuevo objeto de entidad, la configuración de sus valores y, a continuación, la llamada al método `insert` en el objeto `TableBatchOperation` para asociar la entidad a una nueva operación de inserción. A continuación, el código llama a `execute` en el objeto `CloudTable`, especificando la tabla "people" y el objeto `TableBatchOperation`, que envía el lote de operaciones de tabla al servicio de almacenamiento en una única solicitud.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Define a batch operation.
    TableBatchOperation batchOperation = new TableBatchOperation();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a customer entity to add to the table.
    CustomerEntity customer = new CustomerEntity("Smith", "Jeff");
    customer.setEmail("Jeff@contoso.com");
    customer.setPhoneNumber("425-555-0104");
    batchOperation.insertOrReplace(customer);

    // Create another customer entity to add to the table.
    CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
    customer2.setEmail("Ben@contoso.com");
    customer2.setPhoneNumber("425-555-0102");
    batchOperation.insertOrReplace(customer2);

    // Create a third customer entity to add to the table.
    CustomerEntity customer3 = new CustomerEntity("Smith", "Denise");
    customer3.setEmail("Denise@contoso.com");
    customer3.setPhoneNumber("425-555-0103");
    batchOperation.insertOrReplace(customer3);

    // Execute the batch of operations on the "people" table.
    cloudTable.execute(batchOperation);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

Algunos aspectos que cabe tener en cuenta acerca de las operaciones por lotes:

* Puede ejecutar cualquier combinación de 100 operaciones de inserción, eliminación, combinación, reemplazo, inserción o combinación e inserción o reemplazo en un único lote.
* Una operación por lotes puede tener una operación de recuperación, si es que es la única operación del lote.
* Todas las entidades de la misma operación por lotes deben compartir la misma clave de partición.
* Una operación por lotes se limita a una carga de datos de 4 MB.

## <a name="retrieve-all-entities-in-a-partition"></a>todas las entidades de una partición

Para consultar una tabla a fin de obtener las entidades de una partición, use un objeto `TableQuery`. Llame a `TableQuery.from` para crear una consulta en una tabla concreta que devuelva un tipo de resultado específico. El código siguiente especifica un filtro para las entidades en las que "Smith" es la clave de partición. `TableQuery.generateFilterCondition` es un método auxiliar para crear filtros para las consultas. Llame a `where` en la referencia devuelta por el método `TableQuery.from` para aplicar el filtro a la consulta. Cuando la consulta se ejecuta con una llamada a `execute` en el objeto `CloudTable`, devuelve un `Iterator` con el tipo de resultado `CustomerEntity` especificado. A continuación, puede usar el `Iterator` devuelto para cada bucle "ForEach" para consumir los resultados. En este código, los campos de cada entidad se imprimen en la consola, como parte de los resultados de la consulta.

```java
try
{
    // Define constants for filters.
    final String PARTITION_KEY = "PartitionKey";
    final String ROW_KEY = "RowKey";
    final String TIMESTAMP = "Timestamp";

    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a filter condition where the partition key is "Smith".
    String partitionFilter = TableQuery.generateFilterCondition(
        PARTITION_KEY,
        QueryComparisons.EQUAL,
        "Smith");

    // Specify a partition query, using "Smith" as the partition key filter.
    TableQuery<CustomerEntity> partitionQuery =
        TableQuery.from(CustomerEntity.class)
        .where(partitionFilter);

    // Loop through the results, displaying information about the entity.
    for (CustomerEntity entity : cloudTable.execute(partitionQuery)) {
        System.out.println(entity.getPartitionKey() +
            " " + entity.getRowKey() +
            "\t" + entity.getEmail() +
            "\t" + entity.getPhoneNumber());
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="retrieve-a-range-of-entities-in-a-partition"></a>Recuperación de un rango de entidades de una partición

Si no quiere consultar todas las entidades de una partición, puede especificar un rango mediante el uso de operadores de comparación en un filtro. El código siguiente combina dos filtros para obtener todas las entidades de la partición "Smith" en las que la clave de fila (nombre de pila) empieza por una letra hasta "E" en el alfabeto. A continuación, imprime los resultados de la consulta. Si usa las entidades agregadas a la tabla en la sección de inserción por lotes de esta guía, solo se devuelven dos entidades en este momento (Ben y Denise Smith); Jeff Smith no se incluye.

```java
try
{
    // Define constants for filters.
    final String PARTITION_KEY = "PartitionKey";
    final String ROW_KEY = "RowKey";
    final String TIMESTAMP = "Timestamp";

    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a filter condition where the partition key is "Smith".
    String partitionFilter = TableQuery.generateFilterCondition(
        PARTITION_KEY,
        QueryComparisons.EQUAL,
        "Smith");

    // Create a filter condition where the row key is less than the letter "E".
    String rowFilter = TableQuery.generateFilterCondition(
        ROW_KEY,
        QueryComparisons.LESS_THAN,
        "E");

    // Combine the two conditions into a filter expression.
    String combinedFilter = TableQuery.combineFilters(partitionFilter,
        Operators.AND, rowFilter);

    // Specify a range query, using "Smith" as the partition key,
    // with the row key being up to the letter "E".
    TableQuery<CustomerEntity> rangeQuery =
        TableQuery.from(CustomerEntity.class)
        .where(combinedFilter);

    // Loop through the results, displaying information about the entity
    for (CustomerEntity entity : cloudTable.execute(rangeQuery)) {
        System.out.println(entity.getPartitionKey() +
            " " + entity.getRowKey() +
            "\t" + entity.getEmail() +
            "\t" + entity.getPhoneNumber());
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="retrieve-a-single-entity"></a>una sola entidad

Puede enviar una consulta para recuperar una sola entidad concreta. El código siguiente llama a `TableOperation.retrieve` con los parámetros de clave de partición y clave de fila para especificar el cliente "Jeff Smith", en lugar de crear un elemento `Table Query` y usar filtros para realizar la misma operación. Cuando se ejecuta, la operación de recuperación devuelve solo una entidad, en lugar de una colección de entidades. El método `getResultAsType` convierte el resultado en el tipo de objetivo de asignación, un objeto `CustomerEntity`. Si este tipo no es compatible con el tipo especificado para la consulta, se muestra una excepción. Se devuelve un valor nulo si no coincide exactamente la clave de fila y de partición de ninguna entidad. La forma más rápida de recuperar una sola entidad de Table service es especificar claves tanto de partición como de fila en las consultas.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Retrieve the entity with partition key of "Smith" and row key of "Jeff"
    TableOperation retrieveSmithJeff =
        TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // Submit the operation to the table service and get the specific entity.
    CustomerEntity specificEntity =
        cloudTable.execute(retrieveSmithJeff).getResultAsType();

    // Output the entity.
    if (specificEntity != null)
    {
        System.out.println(specificEntity.getPartitionKey() +
            " " + specificEntity.getRowKey() +
            "\t" + specificEntity.getEmail() +
            "\t" + specificEntity.getPhoneNumber());
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="modify-an-entity"></a>Modificación de una entidad

Para modificar una entidad, recupérela del servicio Tabla, realice los cambios en el objeto de entidad y vuelva a guardar los cambios en dicho servicio con una operación de reemplazo o combinación. El código siguiente cambia el número de teléfono de un cliente. En lugar de llamar a **TableOperation.insert** como hicimos para la inserción, este código llama a **TableOperation.replace**. El método **CloudTable.execute** llama al servicio Tabla y la entidad se reemplaza, a no ser que otra aplicación la haya modificado desde que la aplicación la recuperó. Cuando se produce esa situación, se muestra una excepción y la entidad debe recuperarse, modificarse y guardarse de nuevo. Este patrón de reintento de simultaneidad optimista es común en un sistema de almacenamiento distribuido.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Retrieve the entity with partition key of "Smith" and row key of "Jeff".
    TableOperation retrieveSmithJeff =
        TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // Submit the operation to the table service and get the specific entity.
    CustomerEntity specificEntity =
        cloudTable.execute(retrieveSmithJeff).getResultAsType();

    // Specify a new phone number.
    specificEntity.setPhoneNumber("425-555-0105");

    // Create an operation to replace the entity.
    TableOperation replaceEntity = TableOperation.replace(specificEntity);

    // Submit the operation to the table service.
    cloudTable.execute(replaceEntity);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="query-a-subset-of-entity-properties"></a>Consulta de un subconjunto de propiedades de las entidades

Una consulta de tabla puede recuperar solo algunas propiedades de una entidad. Esta técnica, denominada proyección, reduce el ancho de banda y puede mejorar el rendimiento de las consultas, en especial en el caso de entidades de gran tamaño. La consulta del siguiente código usa el método `select` para devolver solo las direcciones de correo electrónico de las entidades de la tabla. Los resultados se proyectan en una colección de `String` con la ayuda de un `Entity Resolver`, que hace la conversión del tipo de entidades que el servidor devuelve. Puede aprender más sobre la proyección en [Tablas de Azure: Introducción de Upsert y proyección de consultas][Tablas de Azure: Introducción de Upsert y proyección de consultas]. La proyección no es compatible con el emulador de almacenamiento local, por lo que este código solo se ejecuta cuando se utiliza una cuenta del servicio Table service.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Define a projection query that retrieves only the Email property
    TableQuery<CustomerEntity> projectionQuery =
        TableQuery.from(CustomerEntity.class)
        .select(new String[] {"Email"});

    // Define an Entity resolver to project the entity to the Email value.
    EntityResolver<String> emailResolver = new EntityResolver<String>() {
        @Override
        public String resolve(String PartitionKey, String RowKey, Date timeStamp, HashMap<String, EntityProperty> properties, String etag) {
            return properties.get("Email").getValueAsString();
        }
    };

    // Loop through the results, displaying the Email values.
    for (String projectedString :
        cloudTable.execute(projectionQuery, emailResolver)) {
            System.out.println(projectedString);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="insert-or-replace-an-entity"></a>Inserción o reemplazo de una entidad

En ocasiones, es posible que desee agregar una entidad a una tabla sin saber si ya existe en la tabla. Las operaciones de inserción o reemplazo permiten realizar una consulta única que inserte la entidad si no existe o que reemplace la existente si la hubiera. Según los ejemplos anteriores, el siguiente código inserta o reemplaza la entidad de "Walter Harp". Después de crear una nueva entidad, este código llama al método **TableOperation.insertOrReplace** . A continuación, este código llama a **execute** en el objeto **Cloud Table** con la tabla y las operaciones de inserción o reemplazo de tabla como parámetros. Para actualizar solo parte de una entidad, en su lugar, se puede usar el método **TableOperation.insertOrMerge** . La inserción o el reemplazo no son compatibles con el emulador de almacenamiento local, por lo que este código solo se ejecuta cuando se utiliza una cuenta del servicio Table service. Puede aprender más sobre las operaciones de inserción o reemplazo y de inserción o fusión en [Tablas de Azure: Introducción de Upsert y proyección de consultas][Tablas de Azure: Introducción de Upsert y proyección de consultas].

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a new customer entity.
    CustomerEntity customer5 = new CustomerEntity("Harp", "Walter");
    customer5.setEmail("Walter@contoso.com");
    customer5.setPhoneNumber("425-555-0106");

    // Create an operation to add the new customer to the people table.
    TableOperation insertCustomer5 = TableOperation.insertOrReplace(customer5);

    // Submit the operation to the table service.
    cloudTable.execute(insertCustomer5);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="delete-an-entity"></a>Eliminación de una entidad

Puede eliminar fácilmente una entidad después de que la haya recuperado. Tras recuperar la entidad, llame a `TableOperation.delete` con la entidad que se va a eliminar. A continuación, llame a `execute` en el objeto `CloudTable`. El código siguiente recupera y elimina una entidad de cliente.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create an operation to retrieve the entity with partition key of "Smith" and row key of "Jeff".
    TableOperation retrieveSmithJeff = TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // Retrieve the entity with partition key of "Smith" and row key of "Jeff".
    CustomerEntity entitySmithJeff =
        cloudTable.execute(retrieveSmithJeff).getResultAsType();

    // Create an operation to delete the entity.
    TableOperation deleteSmithJeff = TableOperation.delete(entitySmithJeff);

    // Submit the delete operation to the table service.
    cloudTable.execute(deleteSmithJeff);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="delete-a-table"></a>Eliminar una tabla

Finalmente, el código siguiente elimina una tabla de una cuenta de almacenamiento. Aproximadamente 40 segundos después de eliminar una tabla, no puede volver a crearla. 

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Delete the table and all its data if it exists.
    CloudTable cloudTable = tableClient.getTableReference("people");
    cloudTable.deleteIfExists();
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

[!INCLUDE [storage-check-out-samples-java](../../includes/storage-check-out-samples-java.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Getting Started with Azure Table service in Java](https://github.com/Azure-Samples/storage-table-java-getting-started) (Introducción a Azure Table service en Java)
* El [Explorador de Microsoft Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md) es una aplicación independiente y gratuita de Microsoft que permite trabajar visualmente con los datos de Azure Storage en Windows, macOS y Linux.
* [SDK de Azure Storage para Java][Azure Storage SDK for Java]
* [Referencia del SDK del cliente de Azure Storage][Azure Storage Client SDK Reference]
* [API de REST de Azure Storage][Azure Storage REST API]
* [Blog del equipo de Azure Storage][Azure Storage Team Blog]

Para más información, visite [Azure para desarrolladores de Java](/java/azure).

[Azure SDK for Java]: https://go.microsoft.com/fwlink/?LinkID=525671
[Azure Storage SDK for Java]: https://github.com/azure/azure-storage-java
[Azure Storage SDK for Android]: https://github.com/azure/azure-storage-android
[Referencia del SDK del cliente de Azure Storage]: https://azure.github.io/azure-storage-java/
[Azure Storage REST API]: https://msdn.microsoft.com/library/azure/dd179355.aspx
[Azure Storage Team Blog]: https://blogs.msdn.microsoft.com/windowsazurestorage/
