---
title: Enlace de salida de Azure Cosmos DB para Functions 2.x y versiones superiores
description: Aprenda a usar el enlace de salida de Azure Cosmos DB en Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/24/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 6a75b0c5b30f60afe51eebc395d21b7c05e8af7f
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212766"
---
# <a name="azure-cosmos-db-output-binding-for-azure-functions-2x-and-higher"></a>Enlace de salida de Azure Cosmos DB para Azure Functions 2.x y versiones superiores

El enlace de salida de Azure Cosmos DB permite escribir un nuevo documento en una base de datos de Azure Cosmos DB mediante SQL API.

Para obtener información sobre los detalles de instalación y configuración, vea la [información general](./functions-bindings-cosmosdb-v2.md).

<a id="example" name="example"></a>

# <a name="c"></a>[C#](#tab/csharp)

En esta sección se incluyen los ejemplos siguientes:

* [Desencadenador de cola, escribir un documento](#queue-trigger-write-one-doc-c)
* [Desencadenador de cola, escribir documentos mediante IAsyncCollector](#queue-trigger-write-docs-using-iasynccollector-c)

Los ejemplos hacen referencia a un tipo de `ToDoItem` simple:

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

<a id="queue-trigger-write-one-doc-c"></a>

### <a name="queue-trigger-write-one-doc"></a>Desencadenador de cola, escribir un documento

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que agrega un documento a una base de datos mediante los datos que se proporcionan en el mensaje desde Queue Storage.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;
using System;

namespace CosmosDBSamplesV2
{
    public static class WriteOneDoc
    {
        [FunctionName("WriteOneDoc")]
        public static void Run(
            [QueueTrigger("todoqueueforwrite")] string queueMessage,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")]out dynamic document,
            ILogger log)
        {
            document = new { Description = queueMessage, id = Guid.NewGuid() };

            log.LogInformation($"C# Queue trigger function inserted one row");
            log.LogInformation($"Description={queueMessage}");
        }
    }
}
```

<a id="queue-trigger-write-docs-using-iasynccollector-c"></a>

### <a name="queue-trigger-write-docs-using-iasynccollector"></a>Desencadenador de cola, escribir documentos mediante IAsyncCollector

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que agrega una colección de documentos a una base de datos mediante los datos que se proporcionan en un mensaje de cola JSON.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class WriteDocsIAsyncCollector
    {
        [FunctionName("WriteDocsIAsyncCollector")]
        public static async Task Run(
            [QueueTrigger("todoqueueforwritemulti")] ToDoItem[] toDoItemsIn,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")]
                IAsyncCollector<ToDoItem> toDoItemsOut,
            ILogger log)
        {
            log.LogInformation($"C# Queue trigger function processed {toDoItemsIn?.Length} items");

            foreach (ToDoItem toDoItem in toDoItemsIn)
            {
                log.LogInformation($"Description={toDoItem.Description}");
                await toDoItemsOut.AddAsync(toDoItem);
            }
        }
    }
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En esta sección se incluyen los ejemplos siguientes:

* [Desencadenador de cola, escribir un documento](#queue-trigger-write-one-doc-c-script)
* [Desencadenador de cola, escribir documentos mediante IAsyncCollector](#queue-trigger-write-docs-using-iasynccollector-c-script)


<a id="queue-trigger-write-one-doc-c-script"></a>

### <a name="queue-trigger-write-one-doc"></a>Desencadenador de cola, escribir un documento

En el ejemplo siguiente se muestra un enlace de salida de Azure Cosmos DB en un archivo *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace. La función usa un enlace de entrada de cola para una cola que recibe JSON en el formato siguiente:

```json
{
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

La función crea documentos de Azure Cosmos DB en el formato siguiente para cada registro:

```json
{
    "id": "John Henry-123456",
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "name": "employeeDocument",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de script de C#:

```cs
    #r "Newtonsoft.Json"

    using Microsoft.Azure.WebJobs.Host;
    using Newtonsoft.Json.Linq;
    using Microsoft.Extensions.Logging;

    public static void Run(string myQueueItem, out object employeeDocument, ILogger log)
    {
      log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");

      dynamic employee = JObject.Parse(myQueueItem);

      employeeDocument = new {
        id = employee.name + "-" + employee.employeeId,
        name = employee.name,
        employeeId = employee.employeeId,
        address = employee.address
      };
    }
```

<a id="queue-trigger-write-docs-using-iasynccollector-c-script"></a>

### <a name="queue-trigger-write-docs-using-iasynccollector"></a>Desencadenador de cola, escribir documentos mediante IAsyncCollector

Para crear varios documentos, puede enlazar a `ICollector<T>` o `IAsyncCollector<T>`, donde `T` es uno de los tipos admitidos.

En este ejemplo se hace referencia a un tipo simple `ToDoItem`:

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

Este es el archivo function.json:

```json
{
  "bindings": [
    {
      "name": "toDoItemsIn",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "todoqueueforwritemulti",
      "connectionStringSetting": "AzureWebJobsStorage"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItemsOut",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

Este es el código de script de C#:

```cs
using System;
using Microsoft.Extensions.Logging;

public static async Task Run(ToDoItem[] toDoItemsIn, IAsyncCollector<ToDoItem> toDoItemsOut, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed {toDoItemsIn?.Length} items");

    foreach (ToDoItem toDoItem in toDoItemsIn)
    {
        log.LogInformation($"Description={toDoItem.Description}");
        await toDoItemsOut.AddAsync(toDoItem);
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra un enlace de salida de Azure Cosmos DB en un archivo *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace. La función usa un enlace de entrada de cola para una cola que recibe JSON en el formato siguiente:

```json
{
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

La función crea documentos de Azure Cosmos DB en el formato siguiente para cada registro:

```json
{
    "id": "John Henry-123456",
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "name": "employeeDocument",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de JavaScript:

```javascript
    module.exports = function (context) {

      context.bindings.employeeDocument = JSON.stringify({
        id: context.bindings.myQueueItem.name + "-" + context.bindings.myQueueItem.employeeId,
        name: context.bindings.myQueueItem.name,
        employeeId: context.bindings.myQueueItem.employeeId,
        address: context.bindings.myQueueItem.address
      });

      context.done();
    };
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra cómo escribir un documento en una base de datos de Azure CosmosDB como la salida de una función.

La definición de enlace se define en *function.json*, donde *type* se establece en `cosmosDB`.

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "cosmosDB",
      "direction": "out",
      "name": "doc",
      "databaseName": "demodb",
      "collectionName": "data",
      "createIfNotExists": "true",
      "connectionStringSetting": "AzureCosmosDBConnectionString"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

Para escribir en la base de datos, pase un objeto de documento al método `set` del parámetro de la base de datos.

```python
import azure.functions as func

def main(req: func.HttpRequest, doc: func.Out[func.Document]) -> func.HttpResponse:

    request_body = req.get_body()

    doc.set(func.Document.from_json(request_body))

    return 'OK'
```

# <a name="java"></a>[Java](#tab/java)

* [Desencadenador de cola, guardar el mensaje en la base de datos a través de un valor devuelto](#queue-trigger-save-message-to-database-via-return-value-java)
* [Desencadenador de HTTP, guardar un documento en la base de datos a través de un valor devuelto](#http-trigger-save-one-document-to-database-via-return-value-java)
* [Desencadenador de HTTP, guardar un documento en la base de datos a través de OutputBinding](#http-trigger-save-one-document-to-database-via-outputbinding-java)
* [Desencadenador de HTTP, guardar varios documentos en la base de datos a través de OutputBinding](#http-trigger-save-multiple-documents-to-database-via-outputbinding-java)


<a id="queue-trigger-save-message-to-database-via-return-value-java"></a>

### <a name="queue-trigger-save-message-to-database-via-return-value"></a>Desencadenador de cola, guardar el mensaje en la base de datos mediante un valor devuelto

En el ejemplo siguiente se muestra una función de Java que agrega un documento a una base de datos con datos de un mensaje de Queue Storage.

```java
@FunctionName("getItem")
@CosmosDBOutput(name = "database",
  databaseName = "ToDoList",
  collectionName = "Items",
  connectionStringSetting = "AzureCosmosDBConnection")
public String cosmosDbQueryById(
    @QueueTrigger(name = "msg",
      queueName = "myqueue-items",
      connection = "AzureWebJobsStorage")
    String message,
    final ExecutionContext context)  {
     return "{ id: \"" + System.currentTimeMillis() + "\", Description: " + message + " }";
   }
```
<a id="http-trigger-save-one-document-to-database-via-return-value-java"></a>

#### <a name="http-trigger-save-one-document-to-database-via-return-value"></a>Desencadenador HTTP, guardar un documento en la base de datos mediante un valor devuelto

En el ejemplo siguiente se muestra una función de Java, cuya firma está anotada con ```@CosmosDBOutput``` y tiene un valor devuelto de tipo ```String```. El documento JSON devuelto por la función se escribirá automáticamente en la colección de COSMOS DB correspondiente.

```java
    @FunctionName("WriteOneDoc")
    @CosmosDBOutput(name = "database",
      databaseName = "ToDoList",
      collectionName = "Items",
      connectionStringSetting = "Cosmos_DB_Connection_String")
    public String run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());

        // Parse query parameter
        String query = request.getQueryParameters().get("desc");
        String name = request.getBody().orElse(query);

        // Generate random ID
        final int id = Math.abs(new Random().nextInt());

        // Generate document
        final String jsonDocument = "{\"id\":\"" + id + "\", " +
                                    "\"description\": \"" + name + "\"}";

        context.getLogger().info("Document to be saved: " + jsonDocument);

        return jsonDocument;
    }
```

<a id="http-trigger-save-one-document-to-database-via-outputbinding-java"></a>

### <a name="http-trigger-save-one-document-to-database-via-outputbinding"></a>Desencadenador HTTP, guardar un documento en la base de datos mediante OutputBinding

En el ejemplo siguiente se muestra una función de Java que escribe un documento en Cosmos DB a través de un parámetro de salida ```OutputBinding<T>```. En este ejemplo, el parámetro ```outputItem``` debe anotarse con ```@CosmosDBOutput```, no con la firma de la función. El uso de ```OutputBinding<T>``` permite que la función aproveche el enlace para escribir el documento en Cosmos DB, mientras que también permite devolver un valor diferente para el autor de llamada de la función, como un documento JSON o XML.

```java
    @FunctionName("WriteOneDocOutputBinding")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBOutput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            OutputBinding<String> outputItem,
            final ExecutionContext context) {

        // Parse query parameter
        String query = request.getQueryParameters().get("desc");
        String name = request.getBody().orElse(query);

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());

        // Generate random ID
        final int id = Math.abs(new Random().nextInt());

        // Generate document
        final String jsonDocument = "{\"id\":\"" + id + "\", " +
                                    "\"description\": \"" + name + "\"}";

        context.getLogger().info("Document to be saved: " + jsonDocument);

        // Set outputItem's value to the JSON document to be saved
        outputItem.setValue(jsonDocument);

        // return a different document to the browser or calling client.
        return request.createResponseBuilder(HttpStatus.OK)
                      .body("Document created successfully.")
                      .build();
    }
```

<a id="http-trigger-save-multiple-documents-to-database-via-outputbinding-java"></a>

### <a name="http-trigger-save-multiple-documents-to-database-via-outputbinding"></a>Desencadenador HTTP, guardar varios documentos en la base de datos mediante OutputBinding

En el ejemplo siguiente se muestra una función de Java que escribe varios documentos en Cosmos DB a través de un parámetro de salida ```OutputBinding<T>```. En este ejemplo, el parámetro ```outputItem``` se anota con ```@CosmosDBOutput```, no con la firma de la función. El parámetro de salida, ```outputItem``` tiene una lista de objetos ```ToDoItem``` como su tipo de parámetro de plantilla. El uso de ```OutputBinding<T>``` permite que la función aproveche el enlace para escribir los documentos en Cosmos DB, mientras que también permite devolver un valor diferente para el autor de llamada de la función, como un documento JSON o XML.

```java
    @FunctionName("WriteMultipleDocsOutputBinding")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBOutput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            OutputBinding<List<ToDoItem>> outputItem,
            final ExecutionContext context) {

        // Parse query parameter
        String query = request.getQueryParameters().get("desc");
        String name = request.getBody().orElse(query);

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());

        // Generate documents
        List<ToDoItem> items = new ArrayList<>();

        for (int i = 0; i < 5; i ++) {
          // Generate random ID
          final int id = Math.abs(new Random().nextInt());

          // Create ToDoItem
          ToDoItem item = new ToDoItem(String.valueOf(id), name);

          items.add(item);
        }

        // Set outputItem's value to the list of POJOs to be saved
        outputItem.setValue(items);
        context.getLogger().info("Document to be saved: " + items);

        // return a different document to the browser or calling client.
        return request.createResponseBuilder(HttpStatus.OK)
                      .body("Documents created successfully.")
                      .build();
    }
```

En la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime), utilice la anotación `@CosmosDBOutput` en los parámetros que se van a escribir en Cosmos DB.  El tipo de parámetro de anotación debe ser ```OutputBinding<T>```, donde T es un tipo nativo de Java o un POJO.

---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [CosmosDB](https://github.com/Azure/azure-webjobs-sdk-extensions/tree/dev/test/WebJobs.Extensions.CosmosDB.Tests).

El constructor del atributo toma el nombre de la base de datos y el nombre de la colección. Para información sobre esos valores y otras propiedades que puede configurar, consulte [Salida: configuración](#configuration). A continuación, se muestra un ejemplo del atributo `CosmosDB` en una signatura de método:

```csharp
    [FunctionName("QueueToDocDB")]
    public static void Run(
        [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
        [CosmosDB("ToDoList", "Items", Id = "id", ConnectionStringSetting = "myCosmosDB")] out dynamic document)
    {
        ...
    }
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

La anotación `CosmosDBOutput` está disponible para escribir datos en Cosmos DB. Puede aplicar la anotación a la función o a un parámetro individual de la función. Cuando se usa en el método de la función, el valor devuelto de la función es lo que se escribe en Cosmos DB. Si utiliza la anotación con un parámetro, el tipo del parámetro se debe declarar como `OutputBinding<T>`, donde `T` es un tipo nativo de Java o un POJO.

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `CosmosDB`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type**     | N/D | Se debe establecer en `cosmosDB`.        |
|**direction**     | N/D | Se debe establecer en `out`.         |
|**name**     | N/D | Nombre del parámetro de enlace que representa al documento en la función.  |
|**databaseName** | **DatabaseName**|Base de datos que contiene la colección en la que se ha creado el documento.     |
|**collectionName** |**CollectionName**  | Nombre de la colección en la que se ha creado el documento. |
|**createIfNotExists**  |**CreateIfNotExists**    | Valor booleano que indica si la colección se ha creado si no existía. El valor predeterminado es *false* porque las colecciones nuevas se crean con rendimiento reservado, lo que afecta el costo. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/)para obtener más información.  |
|**partitionKey**|**PartitionKey** |Cuando el valor de `CreateIfNotExists` es true, define la ruta de la clave de partición para la colección creada.|
|**collectionThroughput**|**CollectionThroughput**| Cuando el valor de `CreateIfNotExists` es true, define el [rendimiento](../cosmos-db/set-throughput.md) de la colección creada.|
|**connectionStringSetting**    |**ConnectionStringSetting** |Nombre de la configuración de aplicación que contiene la cadena de conexión de Azure Cosmos DB.        |
|**preferredLocations**| **PreferredLocations**| (Opcional) Defina las ubicaciones preferidas (regiones) para las cuentas de base de datos con replicación geográfica en el servicio de Azure Cosmos DB. Los valores deben estar separados por comas. Por ejemplo, "Este de EE. UU., Centro-sur de EE. UU. Norte de Europa". |
|**useMultipleWriteLocations**| **UseMultipleWriteLocations**| (Opcional) Cuando se establece en `true` junto con `PreferredLocations`, puede aprovechar [escrituras de varias regiones](../cosmos-db/how-to-manage-database-account.md#configure-multiple-write-regions) en el servicio Azure Cosmos DB. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Uso

De forma predeterminada, cuando se escribe en el parámetro de salida de la función, se crea un documento en la base de datos. Este documento tiene un GUID generado automáticamente como identificador de documento. Puede especificar el identificador de documento del documento de salida si especifica la propiedad `id` en el objeto JSON pasado al parámetro de salida.

> [!Note]
> Cuando especifica el identificador de un documento existente, se sobrescribe con el nuevo documento de salida.

## <a name="exceptions-and-return-codes"></a>Excepciones y códigos de retorno

| Enlace | Referencia |
|---|---|
| CosmosDB | [Códigos de error de CosmosDB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb) |

<a name="host-json"></a>

## <a name="hostjson-settings"></a>configuración de host.json

En esta sección se describen las opciones de configuración globales disponibles para este enlace en la versión 2.x. Para obtener más información acerca de las opciones de configuración globales de la versión 2.x, consulte la [referencia de host.json para Azure Functions, versión 2.x](functions-host-json.md).

```json
{
    "version": "2.0",
    "extensions": {
        "cosmosDB": {
            "connectionMode": "Gateway",
            "protocol": "Https",
            "leaseOptions": {
                "leasePrefix": "prefix1"
            }
        }
    }
}
```

|Propiedad  |Valor predeterminado | Descripción |
|---------|---------|---------|
|GatewayMode|Puerta de enlace|Modo de conexión que usa la función al conectarse al servicio de Azure Cosmos DB. Las opciones son `Direct` y `Gateway`|
|Protocolo|Https|Protocolo de conexión que usa la función al conectarse al servicio de Azure Cosmos DB.  Lea [aquí para obtener una explicación de los dos modos](../cosmos-db/performance-tips.md#networking)|
|leasePrefix|N/D|Prefijo de concesión que se usará en todas las funciones de una aplicación.|

## <a name="next-steps"></a>Pasos siguientes

- [Ejecución de una función cuando se crea o modifica un documento de Azure Cosmos DB (desencadenador)](./functions-bindings-cosmosdb-v2-trigger.md)
- [Lectura de un documento de Azure Cosmos DB (enlace de entrada)](./functions-bindings-cosmosdb-v2-input.md)
