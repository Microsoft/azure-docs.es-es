---
title: Uso de Azure Table Storage y Table API de Azure Cosmos DB con C++
description: Almacene datos estructurados en la nube mediante Azure Table Storage o Table API de Azure Cosmos DB.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: cpp
ms.topic: sample
ms.date: 10/07/2019
author: sakash279
ms.author: akshanka
ms.openlocfilehash: 3d38fa2afe35976283e5129eab7d7f8ef3a1103b
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88236495"
---
# <a name="how-to-use-azure-table-storage-and-azure-cosmos-db-table-api-with-c"></a>Uso de Azure Table Storage y Table API de Azure Cosmos DB con C++

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

En esta guía se muestran escenarios comunes en los que se usa el servicio Azure Table Storage o Table API de Azure Cosmos DB. Los ejemplos están escritos en C++ y usan la [biblioteca de cliente de Azure Storage para C++](https://github.com/Azure/azure-storage-cpp/blob/master/README.md). En este artículo se tratan los siguientes escenarios:

* Creación y eliminación de una tabla
* Trabajo con entidades de tabla

> [!NOTE]
> Esta guía se destina a la biblioteca de cliente de Azure Storage para C++, versión 1.0.0 y posteriores. La versión recomendada es la biblioteca cliente de Storage 2.2.0, que se encuentra disponible a través de [NuGet](https://www.nuget.org/packages/wastorage) o [GitHub](https://github.com/Azure/azure-storage-cpp/).
>

## <a name="create-accounts"></a>Creación de cuentas

### <a name="create-an-azure-service-account"></a>Creación de una cuenta de servicio de Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

### <a name="create-an-azure-storage-account"></a>Creación de una cuenta de Azure Storage

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

### <a name="create-an-azure-cosmos-db-table-api-account"></a>Creación de una cuenta de Table API de Azure Cosmos DB

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="create-a-c-application"></a>Creación de una aplicación de C++

En esta guía, usará las características de almacenamiento de una aplicación de C++. Para ello, instale la biblioteca cliente de Azure Storage para C++.

Para instalar la biblioteca cliente de Azure Storage para C++, use los métodos siguientes:

* **Linux:** Follow the instructions given in the [Azure Storage Client Library for C++ README: Getting Started on Linux](https://github.com/Azure/azure-storage-cpp#getting-started-on-linux) page.
* **Windows:** En Windows, use [vcpkg](https://github.com/microsoft/vcpkg) como administrador de dependencias. Siga la [guía de inicio rápido](https://github.com/microsoft/vcpkg#quick-start) para inicializar vcpkg. Después, use el comando siguiente para instalar la biblioteca:

```powershell
.\vcpkg.exe install azure-storage-cpp
```

Puede encontrar una guía sobre cómo compilar el código fuente y exportarlo a Nuget en el archivo [Léame](https://github.com/Azure/azure-storage-cpp#download--install).

### <a name="configure-access-to-the-table-client-library"></a>Configuración de acceso a la biblioteca cliente de Table

Si desea usar las API de Azure Storage para acceder a las tablas, agregue las siguientes instrucciones `include` en la parte superior del archivo de C++:

```cpp
#include <was/storage_account.h>
#include <was/table.h>
```

Un cliente de Azure Storage o Cosmos DB usa una cadena de conexión para almacenar puntos de conexión y credenciales con el fin de acceder a los servicios de administración de datos. Cuando ejecuta una aplicación cliente, debe proporcionar la cadena de conexión de Storage o la cadena de conexión de Azure Cosmos DB con el formato apropiado.

### <a name="set-up-an-azure-storage-connection-string"></a>Configuración de una cadena de conexión de Azure Storage

En este ejemplo se muestra cómo declarar un campo estático para mantener la cadena de conexión de Azure Storage:  

```cpp
// Define the Storage connection string with your values.
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=<your_storage_account>;AccountKey=<your_storage_account_key>"));
```

Use el nombre de la cuenta de Storage en `<your_storage_account>`. En <your_storage_account_key>, use la clave de acceso de la cuenta de Storage que se muestra en [Azure Portal](https://portal.azure.com). Para obtener información sobre las cuentas de Storage y las claves de acceso, consulte [Crear una cuenta de almacenamiento](../storage/common/storage-create-storage-account.md).

### <a name="set-up-an-azure-cosmos-db-connection-string"></a>Configuración de una cadena de conexión de Azure Cosmos DB

En este ejemplo se muestra cómo declarar un campo estático para mantener la cadena de conexión de Azure Cosmos DB:

```cpp
// Define the Azure Cosmos DB connection string with your values.
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=<your_cosmos_db_account>;AccountKey=<your_cosmos_db_account_key>;TableEndpoint=<your_cosmos_db_endpoint>"));
```

Use el nombre de la cuenta de Azure Cosmos DB en `<your_cosmos_db_account>`. Escriba la clave principal en `<your_cosmos_db_account_key>`. Escriba el punto de conexión que aparece en [Azure Portal](https://portal.azure.com) en `<your_cosmos_db_endpoint>`.

Para probar la aplicación en el equipo local de Windows, puede usar Emulador de Azure Storage que se instala con el [SDK de Azure](https://azure.microsoft.com/downloads/). Emulador de Storage es una utilidad que simula los servicios Azure Blob, Queue y Table disponibles en la máquina de desarrollo local. En el ejemplo siguiente se muestra cómo declarar un campo estático para mantener la cadena de conexión al emulador de almacenamiento local:  

```cpp
// Define the connection string with Azure storage emulator.
const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));  
```

Para iniciar Emulador de Azure Storage, en el escritorio de Windows, seleccione el botón **Iniciar** o la tecla Windows. Escriba y ejecute *Emulador de Microsoft Azure Storage*. Para más información, consulte [Uso del emulador de Azure Storage para desarrollo y pruebas](../storage/common/storage-use-emulator.md).

### <a name="retrieve-your-connection-string"></a>Recuperación de la cadena de conexión

Puede usar la clase `cloud_storage_account` para representar la información de la cuenta de almacenamiento. Para recuperar la información de la cuenta de almacenamiento a partir de la cadena de conexión de almacenamiento, use el método `parse`.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);
```

A continuación, obtenga una referencia a una clase `cloud_table_client`. Esta clase permite obtener objetos de referencia para las tablas y entidades almacenadas en el servicio Table Storage. En el código siguiente se crea un objeto `cloud_table_client` mediante el objeto de cuenta de almacenamiento recuperado anteriormente:  

```cpp
// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();
```

## <a name="create-and-add-entities-to-a-table"></a>Creación e incorporación de entidades a una tabla

### <a name="create-a-table"></a>Creación de una tabla

Los objetos `cloud_table_client` permiten obtener objetos de referencia para tablas y entidades. El código siguiente crea un objeto `cloud_table_client` y lo usa para crear una tabla.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);  

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Retrieve a reference to a table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create the table if it doesn't exist.
table.create_if_not_exists();  
```

### <a name="add-an-entity-to-a-table"></a>Adición de una entidad a una tabla

Para agregar una entidad a una tabla, cree un nuevo objeto `table_entity` y páselo a `table_operation::insert_entity`. El código siguiente usa el nombre de pila del cliente como clave de fila y el apellido como clave de partición. En conjunto, la clave de partición y la clave de fila de una entidad la identifican inequívocamente en la tabla. Puede realizarse una consulta en las entidades con la misma clave de partición de manera más rápida que en aquellas que tienen claves de partición distintas. El uso de claves de partición diversas favorece una mayor escalabilidad de las operaciones paralelas. Para obtener más información, consulte [Lista de comprobación de rendimiento y escalabilidad de Almacenamiento de Microsoft Azure](../storage/common/storage-performance-checklist.md).

El código siguiente crea una nueva instancia de `table_entity` con algunos datos de clientes que se van a almacenar. El código siguiente llama a `table_operation::insert_entity` para crear un objeto `table_operation` a fin de insertar una entidad en una tabla y le asocia la nueva entidad de tabla. Por último, el código llama al método `execute` en el objeto `cloud_table`. El nuevo objeto `table_operation` envía una solicitud a Table service para insertar la nueva entidad de cliente en la tabla `people`.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Retrieve a reference to a table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create the table if it doesn't exist.
table.create_if_not_exists();

// Create a new customer entity.
azure::storage::table_entity customer1(U("Harp"), U("Walter"));

azure::storage::table_entity::properties_type& properties = customer1.properties();
properties.reserve(2);
properties[U("Email")] = azure::storage::entity_property(U("Walter@contoso.com"));

properties[U("Phone")] = azure::storage::entity_property(U("425-555-0101"));

// Create the table operation that inserts the customer entity.
azure::storage::table_operation insert_operation = azure::storage::table_operation::insert_entity(customer1);

// Execute the insert operation.
azure::storage::table_result insert_result = table.execute(insert_operation);
```

### <a name="insert-a-batch-of-entities"></a>Inserción de un lote de entidades

Puede insertar un lote de entidades en Table service mediante una operación de escritura. El siguiente código crea un objeto `table_batch_operation` y, luego, le agrega tres operaciones de inserción. Cada operación de inserción se agrega mediante la creación de un nuevo objeto de entidad, la configuración de sus valores y, a continuación, la llamada al método `insert` en el objeto `table_batch_operation` para asociar la entidad a una nueva operación de inserción. A continuación, el código llama a `cloud_table.execute` para ejecutar la operación.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Define a batch operation.
azure::storage::table_batch_operation batch_operation;

// Create a customer entity and add it to the table.
azure::storage::table_entity customer1(U("Smith"), U("Jeff"));

azure::storage::table_entity::properties_type& properties1 = customer1.properties();
properties1.reserve(2);
properties1[U("Email")] = azure::storage::entity_property(U("Jeff@contoso.com"));
properties1[U("Phone")] = azure::storage::entity_property(U("425-555-0104"));

// Create another customer entity and add it to the table.
azure::storage::table_entity customer2(U("Smith"), U("Ben"));

azure::storage::table_entity::properties_type& properties2 = customer2.properties();
properties2.reserve(2);
properties2[U("Email")] = azure::storage::entity_property(U("Ben@contoso.com"));
properties2[U("Phone")] = azure::storage::entity_property(U("425-555-0102"));

// Create a third customer entity to add to the table.
azure::storage::table_entity customer3(U("Smith"), U("Denise"));

azure::storage::table_entity::properties_type& properties3 = customer3.properties();
properties3.reserve(2);
properties3[U("Email")] = azure::storage::entity_property(U("Denise@contoso.com"));
properties3[U("Phone")] = azure::storage::entity_property(U("425-555-0103"));

// Add customer entities to the batch insert operation.
batch_operation.insert_or_replace_entity(customer1);
batch_operation.insert_or_replace_entity(customer2);
batch_operation.insert_or_replace_entity(customer3);

// Execute the batch operation.
std::vector<azure::storage::table_result> results = table.execute_batch(batch_operation);
```

Algunos aspectos que cabe tener en cuenta acerca de las operaciones por lotes:

* Puede realizar hasta 100 operaciones `insert`, `delete`, `merge`, `replace`, `insert-or-merge` y `insert-or-replace` con cualquier combinación en un solo lote.  
* Una operación por lotes puede tener una operación de recuperación, si es la única operación del lote.  
* Todas las entidades de la misma operación por lotes deben compartir la misma clave de partición.  
* Una operación por lotes se limita a una carga de datos de 4 MB.  

## <a name="query-and-modify-entities"></a>Consulta y modificación de entidades

### <a name="retrieve-all-entities-in-a-partition"></a>todas las entidades de una partición

Para consultar una tabla a fin de obtener todas las entidades de una partición, use un objeto `table_query`. El siguiente ejemplo de código especifica un filtro para las entidades en las que `Smith` es la clave de partición. En este ejemplo, los campos de cada entidad se imprimen en la consola, como parte de los resultados de la consulta.  

> [!NOTE]
> Estos métodos no son compatibles actualmente con C++ en Azure Cosmos DB.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Construct the query operation for all customer entities where PartitionKey="Smith".
azure::storage::table_query query;

query.set_filter_string(azure::storage::table_query::generate_filter_condition(U("PartitionKey"), azure::storage::query_comparison_operator::equal, U("Smith")));

// Execute the query.
azure::storage::table_query_iterator it = table.execute_query(query);

// Print the fields for each customer.
azure::storage::table_query_iterator end_of_results;
for (; it != end_of_results; ++it)
{
    const azure::storage::table_entity::properties_type& properties = it->properties();

    std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key()
        << U(", Property1: ") << properties.at(U("Email")).string_value()
        << U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
}  
```

La consulta de este ejemplo devuelve todas las entidades que coinciden con los criterios de filtro. Si tiene tablas grandes y necesita descargar las entidades de tabla con frecuencia, se recomienda almacenar los datos en blobs de Almacenamiento de Azure en su lugar.

### <a name="retrieve-a-range-of-entities-in-a-partition"></a>Recuperación de un rango de entidades de una partición

Si no desea consultar todas las entidades de una partición, puede especificar un rango. Combine el filtro de clave de partición con un filtro de clave de fila. El siguiente ejemplo de código utiliza dos filtros para obtener todas las entidades de la partición `Smith` en las que la clave de fila (nombre) empieza por una letra anterior a `E` en el alfabeto y, a continuación, imprime los resultados de la consulta.  

> [!NOTE]
> Estos métodos no son compatibles actualmente con C++ en Azure Cosmos DB.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create the table query.
azure::storage::table_query query;

query.set_filter_string(azure::storage::table_query::combine_filter_conditions(
    azure::storage::table_query::generate_filter_condition(U("PartitionKey"),
    azure::storage::query_comparison_operator::equal, U("Smith")),
    azure::storage::query_logical_operator::op_and,
    azure::storage::table_query::generate_filter_condition(U("RowKey"), azure::storage::query_comparison_operator::less_than, U("E"))));

// Execute the query.
azure::storage::table_query_iterator it = table.execute_query(query);

// Loop through the results, displaying information about the entity.
azure::storage::table_query_iterator end_of_results;
for (; it != end_of_results; ++it)
{
    const azure::storage::table_entity::properties_type& properties = it->properties();

    std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key()
        << U(", Property1: ") << properties.at(U("Email")).string_value()
        << U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
}  
```

### <a name="retrieve-a-single-entity"></a>una sola entidad

Puede enviar una consulta para recuperar una sola entidad concreta. En el código siguiente se utiliza `table_operation::retrieve_entity` para especificar el cliente `Jeff Smith`. Este método devuelve una sola entidad, en lugar de una colección, y el valor devuelto se incluye en `table_result`. La forma más rápida de recuperar una sola entidad de Table service es especificar claves tanto de partición como de fila en las consultas.  

```cpp
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Retrieve the entity with partition key of "Smith" and row key of "Jeff".
azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

// Output the entity.
azure::storage::table_entity entity = retrieve_result.entity();
const azure::storage::table_entity::properties_type& properties = entity.properties();

std::wcout << U("PartitionKey: ") << entity.partition_key() << U(", RowKey: ") << entity.row_key()
    << U(", Property1: ") << properties.at(U("Email")).string_value()
    << U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
```

### <a name="replace-an-entity"></a>una entidad

Para reemplazar una entidad, recupérela de Table service, modifique su objeto y, luego, guarde los cambios de nuevo en Table service. El código siguiente cambia el número de teléfono y la dirección de correo electrónico de un cliente. En lugar de llamar a `table_operation::insert_entity`, este código utiliza `table_operation::replace_entity`. De este modo, la entidad se reemplaza por completo en el servidor, a menos que la entidad del servidor hubiese cambiado desde que se recuperó. Si se modificó, se produce un error en la operación. Este error evita que la aplicación sobrescriba un cambio realizado por parte de otro componente entre la recuperación y la actualización. La manera correcta de actuar ante este error es recuperar de nuevo la entidad, hacer los cambios oportunos —si siguen siendo válidos— y, a continuación, realizar otra operación `table_operation::replace_entity`.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Replace an entity.
azure::storage::table_entity entity_to_replace(U("Smith"), U("Jeff"));
azure::storage::table_entity::properties_type& properties_to_replace = entity_to_replace.properties();
properties_to_replace.reserve(2);

// Specify a new phone number.
properties_to_replace[U("Phone")] = azure::storage::entity_property(U("425-555-0106"));

// Specify a new email address.
properties_to_replace[U("Email")] = azure::storage::entity_property(U("JeffS@contoso.com"));

// Create an operation to replace the entity.
azure::storage::table_operation replace_operation = azure::storage::table_operation::replace_entity(entity_to_replace);

// Submit the operation to the Table service.
azure::storage::table_result replace_result = table.execute(replace_operation);
```

### <a name="insert-or-replace-an-entity"></a>Inserción o reemplazo de una entidad

Las operaciones `table_operation::replace_entity` darán error si se cambió la entidad desde que se recuperó del servidor. Asimismo, para que la operación `table_operation::replace_entity` se realice correctamente, antes debe recuperar la entidad del servidor. En ocasiones, no sabe si la entidad existe en el servidor. Los valores actuales almacenados en ella son irrelevantes, ya que la actualización deberá sobrescribirlos todos. Para lograr este resultado, use una operación `table_operation::insert_or_replace_entity`. Esta operación inserta la entidad si no existe. La operación reemplaza la entidad si existe. En el siguiente ejemplo de código, también se recupera la entidad de cliente `Jeff Smith` pero, a continuación, se guarda de nuevo en el servidor mediante `table_operation::insert_or_replace_entity`. Las actualizaciones realizadas en la entidad entre la operación de recuperación y actualización se sobrescribirán.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Insert or replace an entity.
azure::storage::table_entity entity_to_insert_or_replace(U("Smith"), U("Jeff"));
azure::storage::table_entity::properties_type& properties_to_insert_or_replace = entity_to_insert_or_replace.properties();

properties_to_insert_or_replace.reserve(2);

// Specify a phone number.
properties_to_insert_or_replace[U("Phone")] = azure::storage::entity_property(U("425-555-0107"));

// Specify an email address.
properties_to_insert_or_replace[U("Email")] = azure::storage::entity_property(U("Jeffsm@contoso.com"));

// Create an operation to insert or replace the entity.
azure::storage::table_operation insert_or_replace_operation = azure::storage::table_operation::insert_or_replace_entity(entity_to_insert_or_replace);

// Submit the operation to the Table service.
azure::storage::table_result insert_or_replace_result = table.execute(insert_or_replace_operation);
```

### <a name="query-a-subset-of-entity-properties"></a>Consulta de un subconjunto de propiedades de las entidades

Una consulta de tabla puede recuperar solo algunas propiedades de una entidad. La consulta del siguiente código usa el método `table_query::set_select_columns` para devolver solo las direcciones de correo electrónico de las entidades de la tabla.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Define the query, and select only the Email property.
azure::storage::table_query query;
std::vector<utility::string_t> columns;

columns.push_back(U("Email"));
query.set_select_columns(columns);

// Execute the query.
azure::storage::table_query_iterator it = table.execute_query(query);

// Display the results.
azure::storage::table_query_iterator end_of_results;
for (; it != end_of_results; ++it)
{
    std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key();

    const azure::storage::table_entity::properties_type& properties = it->properties();
    for (auto prop_it = properties.begin(); prop_it != properties.end(); ++prop_it)
    {
        std::wcout << ", " << prop_it->first << ": " << prop_it->second.str();
    }

    std::wcout << std::endl;
}
```

> [!NOTE]
> Consultar una serie de propiedades de una entidad es una operación más eficaz que recuperar todas las propiedades.
>

## <a name="delete-content"></a>Eliminación de contenido

### <a name="delete-an-entity"></a>Eliminación de una entidad

Puede eliminar una entidad después de recuperarla. Después de recuperar una entidad, llame a `table_operation::delete_entity` con la entidad que va a eliminar. A continuación, llame al método `cloud_table.execute`. El siguiente código recupera y elimina una entidad con la clave de partición `Smith` y la clave de fila `Jeff`.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create an operation to retrieve the entity with partition key of "Smith" and row key of "Jeff".
azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

// Create an operation to delete the entity.
azure::storage::table_operation delete_operation = azure::storage::table_operation::delete_entity(retrieve_result.entity());

// Submit the delete operation to the Table service.
azure::storage::table_result delete_result = table.execute(delete_operation);  
```

### <a name="delete-a-table"></a>Eliminar una tabla

Finalmente, el ejemplo de código siguiente elimina una tabla de una cuenta de almacenamiento. Una tabla que se ha eliminado no podrá volver a crearse durante un tiempo tras la eliminación.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Delete the table if it exists
if (table.delete_table_if_exists())
{
    std::cout << "Table deleted!";
}
else
{
    std::cout << "Table didn't exist";
}
```

## <a name="troubleshooting"></a>Solución de problemas

En Visual Studio Community Edition, si en el proyecto se generan errores de compilación debido a los archivos de inclusión *storage_account.h* y *table.h*, quite el conmutador de compilador **/permissive-** :

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Propiedades**.
1. En el cuadro de diálogo **Páginas de propiedades**, expanda **Propiedades de configuración**, expanda **C/C++** y seleccione **Lenguaje**.
1. Establezca **Modo de conformidad** en **No**.

## <a name="next-steps"></a>Pasos siguientes

El [Explorador de Microsoft Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md) es una aplicación independiente y gratuita de Microsoft que permite trabajar visualmente con los datos de Azure Storage en Windows, macOS y Linux.

Siga estos vínculos para obtener más información sobre Azure Storage y Table API de Azure Cosmos DB:

* [Introducción a Table API](table-introduction.md)
* [Enumeración de los recursos de Azure Storage en C++](../storage/common/storage-c-plus-plus-enumeration.md)
* [Referencia de la biblioteca de clientes de almacenamiento para C++](https://azure.github.io/azure-storage-cpp)
* [Documentación de Azure Storage](https://azure.microsoft.com/documentation/services/storage/)
