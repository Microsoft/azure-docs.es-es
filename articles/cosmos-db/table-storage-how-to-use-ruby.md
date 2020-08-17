---
title: Uso de Table API de Azure Cosmos DB y Azure Table Storage con Ruby
description: Almacene datos estructurados en la nube mediante Azure Table Storage o Table API de Azure Cosmos DB.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: ruby
ms.topic: sample
ms.date: 07/23/2020
author: sakash279
ms.author: akshanka
ms.reviewer: sngun
ms.openlocfilehash: 2229eea7b91168507ea9568a1e53930cf983b1df
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87171953"
---
# <a name="how-to-use-azure-table-storage-and-the-azure-cosmos-db-table-api-with-ruby"></a>Uso de Azure Table Storage y Table API de Azure Cosmos DB con Ruby

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

En este artículo se muestra cómo crear tablas, y almacenar los datos y realizar operaciones CRUD en ellos. Elija usar el servicio Azure Table Storage o Table API de Azure Cosmos DB. Los ejemplos de este artículo están escritos en Ruby y usan la [biblioteca de cliente de Azure Storage Table para Ruby](https://github.com/azure/azure-storage-ruby/tree/master/table). Entre los escenarios se incluyen la creación de una tabla, la eliminación de una tabla, la inserción de entidades y la consulta de entidades de la tabla.

## <a name="create-an-azure-service-account"></a>Creación de una cuenta de servicio de Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

**Creación de una cuenta de Azure Storage**

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

**Creación de una cuenta de Azure Cosmos DB**

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="add-access-to-azure-storage-or-azure-cosmos-db"></a>Incorporación de acceso a Azure Storage o Azure Cosmos DB

Para usar Azure Storage o Azure Cosmos DB, debe descargar y usar el paquete de Azure para Ruby que incluye un conjunto de bibliotecas útiles que se comunican con los servicios REST de Table.

### <a name="use-rubygems-to-obtain-the-package"></a>Uso de RubyGems para obtener el paquete

1. Use una interfaz de línea de comandos como **PowerShell** (Windows), **Terminal** (Mac) o **Bash** (Unix).
2. Escriba **gem install azure-storage-table** en la ventana de comandos para instalar la gema y las dependencias.

### <a name="import-the-package"></a>Importación del paquete

Con el editor de texto que prefiera, agregue lo siguiente al principio del archivo de Ruby en el que pretenda utilizar el almacenamiento:

```ruby
require "azure/storage/table"
```

## <a name="add-your-connection-string"></a>Incorporación de la cadena de conexión

Puede conectarse a la cuenta de almacenamiento de Azure o a la de Table API de Azure Cosmos DB. Obtenga la cadena de conexión en función del tipo de cuenta que esté usando.

### <a name="add-an-azure-storage-connection"></a>Incorporación de una conexión de Azure Storage

El módulo de Azure Storage lee las variables de entorno **AZURE_STORAGE_ACCOUNT** y **AZURE_STORAGE_ACCESS_KEY** para la información necesaria para conectarse a su cuenta de Azure Storage. Si no se establecen estas variables de entorno, debe especificar la información de la cuenta antes de usar **Azure::Storage::Table::TableService** con el código siguiente:

```ruby
Azure.config.storage_account_name = "<your Azure Storage account>"
Azure.config.storage_access_key = "<your Azure Storage access key>"
```

Para obtener estos valores desde una cuenta de almacenamiento de Azure Resource Manager o clásica en el Portal de Azure:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya a la cuenta de almacenamiento que desea usar.
3. En la hoja Configuración que se encuentra a la derecha, haga clic en **Claves de acceso**.
4. En la hoja Claves de acceso que aparece, verá la clave de acceso 1 y 2. Puede usar cualquiera de estas.
5. Haga clic en el icono de copia para copiar la clave en el Portapapeles.

### <a name="add-an-azure-cosmos-db-connection"></a>Adición de una conexión de Azure Cosmos DB

Para conectarse a Azure Cosmos DB, copie la cadena de conexión principal de Azure Portal y cree un objeto **Client** con la cadena de conexión que ha copiado. Puede pasar el objeto **Client** cuando crea un objeto **TableService**:

```ruby
common_client = Azure::Storage::Common::Client.create(storage_account_name:'myaccount', storage_access_key:'mykey', storage_table_host:'mycosmosdb_endpoint')
table_client = Azure::Storage::Table::TableService.new(client: common_client)
```

## <a name="create-a-table"></a>Creación de una tabla

El objeto **Azure::Storage::Table::TableService** permite trabajar con tablas y entidades. Para crear una tabla, use el método **create_table()** . En el siguiente ejemplo se crea una tabla o se imprime el error, si hay alguno.

```ruby
azure_table_service = Azure::Storage::Table::TableService.new
begin
    azure_table_service.create_table("testtable")
rescue
    puts $!
end
```

## <a name="add-an-entity-to-a-table"></a>Adición de una entidad a una tabla

Para agregar una entidad, primero cree un objeto hash que defina las propiedades de la entidad. Tenga en cuenta que para cada entidad debe especificar valores en **PartitionKey** y **RowKey**. Estos son los identificadores exclusivos de sus entidades y son valores que se pueden consultar más rápidamente que las demás propiedades. Azure Storage usa **PartitionKey** para distribuir automáticamente las entidades de la tabla entre varios nodos de almacenamiento. Las entidades con la misma **PartitionKey** se almacenan en el mismo nodo. La **RowKey** es el identificador exclusivo de la entidad de la partición a la que pertenece.

```ruby
entity = { "content" => "test entity",
    :PartitionKey => "test-partition-key", :RowKey => "1" }
azure_table_service.insert_entity("testtable", entity)
```

## <a name="update-an-entity"></a>Actualización de una entidad

Hay varios métodos para actualizar una entidad existente:

* **update_entity():** actualiza una entidad existente al reemplazarla.
* **merge_entity():** actualiza una entidad que ya existe combinando los valores de las nuevas propiedades con la entidad existente.
* **insert_or_merge_entity():** actualiza una entidad existente al reemplazarla. Si no hay entidades, se insertará una nueva.
* **insert_or_replace_entity():** actualiza una entidad que ya existe combinando los valores de las nuevas propiedades con la entidad existente. Si no hay entidades, se insertará una nueva.

En el ejemplo siguiente se demuestra cómo actualizar una entidad usando **update_entity()** :

```ruby
entity = { "content" => "test entity with updated content",
    :PartitionKey => "test-partition-key", :RowKey => "1" }
azure_table_service.update_entity("testtable", entity)
```

Con **update_entity()** y **merge_entity()** , si la entidad que va a actualizar no existe, la operación de actualización generará un error. Por lo tanto, si desea almacenar una entidad independientemente de la que ya existe, debe usar **insert_or_replace_entity()** o **insert_or_merge_entity()** .

## <a name="work-with-groups-of-entities"></a>Trabajo con grupos de entidades

A veces resulta útil enviar varias operaciones juntas en un lote a fin de garantizar el procesamiento atómico por parte del servidor. Para ello, primero debe crear un objeto **Batch** y, a continuación, usar el método **execute_batch()** en **TableService**. El siguiente ejemplo muestra el envío de dos entidades con RowKey 2 y 3 en un lote. Tenga en cuenta que solo funciona para entidades con el mismo valor de PartitionKey.

```ruby
azure_table_service = Azure::TableService.new
batch = Azure::Storage::Table::Batch.new("testtable",
    "test-partition-key") do
    insert "2", { "content" => "new content 2" }
    insert "3", { "content" => "new content 3" }
end
results = azure_table_service.execute_batch(batch)
```

## <a name="query-for-an-entity"></a>Consulta de una entidad

Para consultar una entidad de una tabla, use el método **get_entity()** , pasando el nombre de la tabla, **PartitionKey** y **RowKey**.

```ruby
result = azure_table_service.get_entity("testtable", "test-partition-key",
    "1")
```

## <a name="query-a-set-of-entities"></a>Consulta de un conjunto de entidades

Para realizar una consulta de un conjunto de entidades de una tabla, cree un objeto hash de consulta y use el método **query_entities()** . El siguiente ejemplo muestra la obtención de todas las entidades con el mismo valor de **PartitionKey**:

```ruby
query = { :filter => "PartitionKey eq 'test-partition-key'" }
result, token = azure_table_service.query_entities("testtable", query)
```

> [!NOTE]
> Si el resultado establecido es demasiado largo para que lo devuelva una única consulta, se devuelve un token de continuación que puede usar para recuperar las páginas siguientes.


## <a name="query-a-subset-of-entity-properties"></a>Consulta de un subconjunto de propiedades de las entidades

Una consulta de tabla puede recuperar solo algunas propiedades de una entidad. Esta técnica, denominada "proyección", reduce el ancho de banda y puede mejorar el rendimiento de las consultas, en especial en el caso de entidades de gran tamaño. Utilice la cláusula select y pase los nombres de las propiedades que quiera que lleguen al cliente.

```ruby
query = { :filter => "PartitionKey eq 'test-partition-key'",
    :select => ["content"] }
result, token = azure_table_service.query_entities("testtable", query)
```

## <a name="delete-an-entity"></a>Eliminación de una entidad

Para eliminar una entidad, use el método **delete_entity()** . Pase el nombre de la tabla que contiene la entidad, la PartitionKey y la RowKey de la entidad.

```ruby
azure_table_service.delete_entity("testtable", "test-partition-key", "1")
```

## <a name="delete-a-table"></a>Eliminar una tabla

Para eliminar una tabla, use el método **delete_table()** y pase el nombre de la tabla que desea eliminar.

```ruby
azure_table_service.delete_table("testtable")
```

## <a name="next-steps"></a>Pasos siguientes

* El [Explorador de Microsoft Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md) es una aplicación independiente y gratuita de Microsoft que permite trabajar visualmente con los datos de Azure Storage en Windows, macOS y Linux.
* [Centro para desarrolladores de Ruby](https://azure.microsoft.com/develop/ruby/)
* [Biblioteca del cliente de Microsoft Azure Storage Table para Ruby](https://github.com/azure/azure-storage-ruby/tree/master/table)